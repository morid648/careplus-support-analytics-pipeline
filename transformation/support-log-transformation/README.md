# Support Log Transformation (AWS Lambda)

Parses raw backend `.log` files into cleaned, structured Parquet using Python, Pandas, and regex — triggered automatically by an S3 event whenever a new log file lands in the Raw Zone.

## What's included

| File | Description |
|---|---|
| `automate_support_log_ETL.ipynb` | The production Lambda handler — `lambda_handler(event, context)`, triggered by an S3 `PUT` event on the Raw Zone |
| `support_log_etl.ipynb` | Standalone (non-Lambda) version of the same transformation logic, used for isolated testing outside the Lambda runtime |
| `etl_logs_local.ipynb` | Local exploration notebook — regex development and iteration against a raw `.log` file on disk, used to build and validate the parsing pattern before wiring it into Lambda |

## How the transformation works

1. **Trigger**: an S3 event fires when a new `.log` file is uploaded to the Raw Zone; the event payload gives the bucket and object key.
2. **Read**: the raw log file is pulled from S3 and decoded as UTF-8 text.
3. **Split**: individual log entries are separated on the `---` delimiter.
4. **Parse**: each entry is matched against a regex pattern that extracts `timestamp`, `log_level`, `component`, `ticket_id`, `session_id`, `ip`, `response_time`, `cpu`, `event_type`, `error`, `user_agent`, `message`, `debug`, and a `trace_id` field.
5. **Clean**:
   - Drop the `trace_id` column (not part of the target schema).
   - Remove rows with a negative `response_time`.
   - Fix known log-level typos (`INF0` → `INFO`, `DEBG` → `DEBUG`, `warnING` → `WARNING`, `EROR` → `ERROR`).
   - Drop duplicate rows.
   - Cast `response_time` to int, `cpu` to float, `error` to boolean, and `timestamp` to a millisecond-precision datetime.
6. **Write**: the cleaned DataFrame is converted to Parquet in-memory (via `pyarrow`) and uploaded to `s3://<bucket>/support-logs/processed/<same_filename>.parquet`.

## Why Lambda

Log files are small and arrive one at a time, processing is naturally event-driven, and there's no infrastructure to provision or manage — a lightweight serverless fit compared to a managed ETL service like Glue.

## Development process

The regex pattern and cleaning steps were first built and validated interactively in `etl_logs_local.ipynb` against a real log file, then consolidated into `support_log_etl.ipynb`, and finally wrapped in the `lambda_handler` entry point in `automate_support_log_ETL.ipynb` for deployment.
