# Transformation

Two AWS ETL services were used deliberately across the two datasets, to gain hands-on experience with both approaches and understand where each fits.

| Dataset | Service | Why |
|---|---|---|
| `support_logs` | **AWS Lambda** | Files are small, processing is naturally event-driven, no infrastructure to manage — a lightweight serverless fit for log parsing |
| `support_tickets` | **AWS Glue** | Purpose-built for ETL on structured datasets; supports both visual and script-based development; integrates with the broader AWS analytics ecosystem |

Both paths converge on the same output format and destination: cleaned, standardized **Parquet** files in the S3 Processed Zone.

## Subfolders

- [`support-log-transformation/`](support-log-transformation/) — AWS Lambda function code and logic for parsing raw `.log` files, plus local dev/exploration notebooks
- [`support-tickets-transformation/`](support-tickets-transformation/) — Lambda function that triggers the AWS Glue ETL job for ticket data

## Why Parquet

Columnar storage was chosen for the Processed Zone specifically for:
- Reduced storage footprint vs. raw CSV/TXT
- Improved query performance (column pruning, predicate pushdown)
- Native compatibility with both AWS Athena and Amazon Redshift's `COPY` command

## End-to-end transformation flow

```
New File Uploaded (Raw Zone)
        ↓
S3 Event Notification
        ↓
Transformation Service (Lambda for logs / Glue for tickets)
        ↓
Amazon S3 Processed Zone (Parquet)
```

For ticket data specifically, the S3 event triggers a lightweight Lambda function whose sole job is to *start the Glue ETL job* — Glue then performs the actual transformation. See [`support-tickets-transformation/README.md`](support-tickets-transformation/README.md) for details.
