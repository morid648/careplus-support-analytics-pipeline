# Warehouse (Amazon Redshift Serverless)

Table definitions and incremental loading logic for the Redshift Serverless data warehouse that serves the Power BI dashboard.

## What's included

| File | Description |
|---|---|
| `redshift_table_creation.sql` | DDL for both warehouse tables, the `COPY` commands used to load them from S3 Processed (Parquet), and simple validation queries |
| `incremental-data-loading-logs.ipynb` | Lambda function that runs the incremental `COPY` operation whenever new processed Parquet data lands in S3 |

## Tables

### `public.support_logs`
```sql
CREATE TABLE public.support_logs (
    timestamp       TIMESTAMP,
    log_level       VARCHAR(20),
    component       VARCHAR(100),
    ticket_id       VARCHAR(50),
    session_id      VARCHAR(50),
    ip              VARCHAR(45),
    response_time   BIGINT,
    cpu             DOUBLE PRECISION,
    event_type      VARCHAR(50),
    error           BOOLEAN,
    user_agent      VARCHAR(300),
    message         VARCHAR(1000),
    debug           VARCHAR(1000)
);
```

### `public.support_tickets`
```sql
CREATE TABLE public.support_tickets (
    ticket_id         VARCHAR(50),
    created_at        TIMESTAMP,
    resolved_at       TIMESTAMP,
    agent             VARCHAR(100),
    priority          VARCHAR(20),
    num_interactions  BIGINT,
    issue_category    VARCHAR(100),
    channel           VARCHAR(50),
    status            VARCHAR(20)
);
```

Full DDL, `COPY` commands, and validation `SELECT` statements are in [`redshift_table_creation.sql`](redshift_table_creation.sql).

## Loading pattern

An S3 event fires the `lambda_handler` in `incremental-data-loading-logs.ipynb` whenever a new processed Parquet file is confirmed. The function:

1. Extracts the bucket/key of the newly landed file from the S3 event.
2. Opens a `psycopg2` connection to Redshift Serverless.
3. Executes a `COPY` command that loads that specific Parquet file directly into the target table:

```sql
COPY public.support_logs
FROM 's3://<bucket>/<key>'
IAM_ROLE '<redshift-iam-role-arn>'
FORMAT AS PARQUET
REGION 'us-east-1';
```

4. Commits the transaction and closes the connection.

This keeps the warehouse incrementally up to date without manual loads or full table rebuilds — each new processed file triggers its own targeted load.

> **Note on credentials:** the original notebook had the Redshift host, IAM role ARN, and password filled in for the author's own Redshift Serverless workgroup. These have been replaced with empty strings / `<YOUR_REDSHIFT_PASSWORD>` placeholders in this repo — see `REDSHIFT_HOST`, `IAM_ROLE`, and `REDSHIFT_PASSWORD` at the top of the notebook.

## Why Redshift Serverless

Serverless was chosen to avoid managing warehouse infrastructure/capacity directly, while still getting a proper MPP SQL warehouse that Power BI can connect to natively and that supports fast `COPY`-based bulk loading straight from S3.

## Access control

A dedicated Redshift IAM role scopes access to only the S3 paths it needs to read from — see [`../screenshots/README.md`](../screenshots/README.md) for the IAM roles and Redshift editor screenshots.
