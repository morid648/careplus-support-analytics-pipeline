# Support Tickets Ingestion

Incremental ingestion of `support_tickets` records from a MySQL source database into the S3 Raw Zone.

## What's included

| File | Description |
|---|---|
| `support_tickets_ingestion_to_S3.ipynb` | Extracts ticket records for the next un-ingested date from MySQL and uploads them to S3 as CSV |
| `date_tracker.txt` | Plaintext tracker recording the last successfully ingested date (`2025-06-30` — one day before the dataset starts) |
| `sample.env` | Template for required environment variables — copy to `.env` and fill in your own values |

> The source MySQL database itself (`careplus_support_db.sql`) is course-provided data and is not included here — see [`../../sample-data/support-tickets/`](../../sample-data/support-tickets/) for a small synthetic seed script in the same schema.

## How it works

1. Connects to MySQL via SQLAlchemy using credentials from `db_config` (host, port, user, password, database).
2. Reads `date_tracker.txt` to determine the last ingested date, and computes the next date to pull.
3. Queries `support_tickets` for records created on that date.
4. Converts the result to CSV in-memory and uploads it to `s3://<bucket>/support-tickets/raw/` using `boto3`.
5. Updates `date_tracker.txt` with the newly ingested date.

## Configuration

Before running, update in the notebook:
- `db_config` — your MySQL host, port, username, password, and database name (defaults to local `root`/`root` for a local dev MySQL instance)
- `S3_BUCKET` / `S3_PREFIX` — your target S3 bucket and raw-zone prefix

And in `.env` (copied from `sample.env`):
- `AWS_ACCESS_KEY`, `SECRET_KEY`, `REGION`

## How to run

1. Load the sample seed data (or your own) into a local/remote MySQL instance: see [`../../sample-data/support-tickets/careplus_support_db_sample.sql`](../../sample-data/support-tickets/careplus_support_db_sample.sql).
2. Copy `sample.env` → `.env` and fill in AWS credentials.
3. Update `db_config` and `S3_BUCKET` in the notebook to match your setup.
4. Run the notebook. Each run ingests one new day's worth of tickets, per `date_tracker.txt`.
