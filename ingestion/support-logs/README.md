# Support Logs Ingestion

Incremental ingestion of raw backend `.log` files into the S3 Raw Zone.

## What's included

| File | Description |
|---|---|
| `support_logs_ingestion_to_S3.ipynb` | Reads the next un-ingested day's `.log` file from a local folder and uploads it to S3 |
| `log_date_tracker.txt` | Plaintext tracker recording the last successfully ingested log date (`2025-06-30` — one day before the dataset starts) |
| `sample.env` | Template for required environment variables — copy to `.env` and fill in your own values |

> The original 31 days of raw `.log` files are course-provided data and are not included here — see [`../../sample-data/support-logs/support_logs_sample.log`](../../sample-data/support-logs/support_logs_sample.log) for a small synthetic file in the identical log format.

## How it works

1. Reads `log_date_tracker.txt` to determine the last ingested date, and computes the next date to pull.
2. Looks for a file named `support_logs_<next_date>.log` inside the local `day-wise-logs-data/` folder.
3. Reads the file's raw text content and uploads it as-is to `s3://<bucket>/support-logs/raw/` using `boto3`.
4. Updates `log_date_tracker.txt` with the newly ingested date.

## Configuration

Before running, update in the notebook:
- `LOGS_FOLDER` — local folder containing the day-wise `.log` files (defaults to `day-wise-logs-data/`)
- `S3_BUCKET` / `S3_PREFIX` — your target S3 bucket and raw-zone prefix

And in `.env` (copied from `sample.env`):
- `AWS_ACCESS_KEY`, `SECRET_KEY`, `REGION`

## How to run

1. Place the sample log file (or your own) into a local `day-wise-logs-data/` folder, named `support_logs_YYYY-MM-DD.log` to match the date the tracker expects next — see [`../../sample-data/support-logs/`](../../sample-data/support-logs/).
2. Copy `sample.env` → `.env` and fill in AWS credentials.
3. Update `S3_BUCKET` in the notebook to match your setup.
4. Run the notebook. Each run ingests one new day's log file, per `log_date_tracker.txt`.
