# Support Tickets Transformation (AWS Glue)

Transforms raw ticket CSV files into cleaned, structured Parquet using AWS Glue — triggered by a lightweight Lambda function that starts the Glue job whenever a new ticket file lands in the Raw Zone.

## What's included

| File | Description |
|---|---|
| `automate_support_tickets_ETL.ipynb` | Lambda handler that fires on an S3 event and starts the `automate_etl_support_tickets` Glue job run, passing the uploaded file's S3 path as a job argument |

> The Glue job itself (`automate_etl_support_tickets`) was authored using AWS Glue's **Visual ETL** interface and then converted into a script-based job within the AWS Glue console — it isn't a local notebook file and so isn't included here. See the main [README](../../README.md#challenges--design-decisions) for why this staged approach (visual first, then script-based) was used.

## How the transformation works

1. **Trigger**: an S3 event fires when a new ticket CSV is uploaded to the Raw Zone.
2. **Lambda**: `automate_support_tickets_ETL.ipynb`'s `lambda_handler` extracts the bucket and object key from the event, builds the full `s3://` path, and calls `glue.start_job_run()` for the `automate_etl_support_tickets` Glue job, passing the input file path as the `--input_file_path` argument.
3. **Glue job**: performs the actual transformation — parsing, cleansing, validation, data-type standardization, duplicate removal, and conversion to Parquet — and writes the output to the S3 Processed Zone.

## Why Glue (and why a Lambda trigger in front of it)

AWS Glue is purpose-built for ETL on structured, tabular datasets, supports both visual and script-based development, and integrates natively with Glue Crawlers, Athena, and Redshift. A small Lambda function in front of it keeps the trigger mechanism consistent with the log-transformation path (both start from an S3 event) while letting Glue handle the heavier, more structured transformation work.
