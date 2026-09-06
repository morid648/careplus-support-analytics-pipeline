# Architecture

The end-to-end pipeline architecture diagram for this project.

## Contents

| File | Description |
|---|---|
| `aws_project_pipeline.svg` | Full pipeline diagram: source systems → S3 (Raw) → Lambda/Glue transformation → S3 (Processed, Parquet) → Athena validation → Amazon Redshift Serverless → Power BI |
| `detailed_architecture_diagram.png` | Layer-by-layer breakdown of the same pipeline (Data Sources → Ingestion → Storage → Transformation → Validation → Warehouse → Automation → Reporting), showing the specific AWS services and S3 folder structure used in each layer |

## How to read the diagram

- **Support Logs** (raw `.log` files) are uploaded to an S3 Raw bucket, transformed by **AWS Lambda** (parsing + cleaning via Python/Pandas/Regex), and written to S3 Processed as Parquet.
- **Support Tickets** (extracted from MySQL) are uploaded to an S3 Raw bucket as CSV, then transformed by **AWS Glue** (Visual ETL → script-based job), and written to S3 Processed as Parquet.
- Both processed Parquet outputs are registered as tables by **AWS Glue Crawlers** and queried directly by **AWS Athena** for validation — before anything is trusted for warehouse loading.
- Validated data is loaded into **Amazon Redshift Serverless** via a Lambda function running SQL `COPY` operations.
- **Power BI** connects directly to Redshift for the final reporting layer.

`detailed_architecture_diagram.png` breaks the same flow into eight explicit layers (Data Sources, Ingestion, Storage, Transformation, Analytics Validation, Data Warehouse, Automation, Reporting), and additionally shows the `careplus-storage` bucket's internal folder structure (`support-logs/raw|processed`, `support-tickets/raw|processed`) and which Lambda function handles which automation step.

## Notes

- The diagram documents the actual AWS resources configured in a personal AWS account for this project.
- No live AWS endpoints, account IDs, or credentials are referenced in this diagram or elsewhere in this repo — see the main [README](../README.md) for how secrets were handled.
