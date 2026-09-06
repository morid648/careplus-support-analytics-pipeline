# Ingestion

Incremental ingestion workflows that pull raw data from source systems into the Amazon S3 Raw Zone. Split into two independent pipelines — one per dataset.

## What's included

| Path | Description |
|---|---|
| `meta_data.txt` | Full column-level documentation for both `support_tickets` and `support_logs`, including the status-field behavior and the one-to-many relationship between the two tables |
| `support-tickets/` | MySQL → S3 ingestion notebook, date tracker, and env template |
| `support-logs/` | Flat-file `.log` → S3 ingestion notebook, date tracker, and env template |

See each subfolder's README for details on that pipeline.

## Datasets

### `support_tickets`
Customer support ticket records sourced from a MySQL database (`careplus_support_db`). One row per ticket.

| Column | Description |
|---|---|
| `ticket_id` | Unique identifier for a support ticket (e.g. `TCK0701011`) |
| `created_at` | When the ticket was logged by the user |
| `resolved_at` | Resolution timestamp — null if status ≠ Resolved |
| `agent` | Support agent assigned to the ticket |
| `priority` | Priority level: Low, Medium, High |
| `issue_category` | Type of issue reported (e.g. Bug Report, Login Issue, Feature Request) |
| `num_interactions` | Number of customer–agent interactions on the ticket |
| `status` | Current ticket status: Resolved, Open, or Escalated |
| `channel` | Submission channel: Email, Chat, Phone, Web Form |

### `support_logs`
Backend system log records generated during ticket processing — one row per backend event (API call, service hit, CPU sample) tied to a ticket.

| Column | Description |
|---|---|
| `timestamp` | When the backend event occurred |
| `log_level` | Severity level (INFO, DEBUG, WARN, ERROR) |
| `component` | Backend system component that generated the event |
| `ticket_id` | Associated support ticket — foreign key to `support_tickets` |
| `session_id` | Session identifier for the logged event |
| `ip` | Client/user IP address |
| `response_time` | Response time in milliseconds |
| `cpu` | CPU load at the time the event was logged |
| `event_type` | Event category |
| `error` | Error indicator (True/False) |
| `user_agent` | Browser or client tool used |
| `message` | Event description (e.g. `"event for TCK0701011"`) |
| `debug` | Internal debug message |

**Relationship**: `support_tickets` → `support_logs` via `ticket_id` — one ticket can generate many log events.

## Incremental ingestion pattern

Both pipelines follow the same date-tracker pattern to avoid reprocessing already-ingested data:

1. Read the last processed date from the pipeline's tracker file.
2. Compute the next date to ingest (`last_date + 1 day`).
3. Extract only that date's records from the source (MySQL query for tickets, or the matching `.log` file on disk for logs).
4. Upload the extracted data to the appropriate S3 Raw Zone prefix.
5. Update the tracker file with the newly processed date.

This simulates a daily production ingestion job without a full orchestration tool (see [Future Enhancements](../README.md#status--future-enhancements) for planned Airflow adoption).

## On the missing raw data

The original raw data for this project — a full MySQL dump (`careplus_support_db.sql`) and 31 days of `.log` files — is course-provided content and has intentionally **not** been included in this repository. See [`../sample-data/README.md`](../sample-data/README.md) for a small synthetic dataset in the identical schema/format that can be used to run these notebooks end-to-end.
