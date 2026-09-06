# Screenshots

Console-level evidence for every stage of the pipeline — included so reviewers can verify the pipeline was actually built and run in AWS, not just designed on paper.

## Contents

| Screenshot | Description |
|---|---|
| `s3_bucket.png` | Amazon S3 bucket structure showing Raw and Processed zones for both datasets |
| `s3_events.png` | S3 Event Notification configuration used to trigger downstream Lambda/Glue jobs automatically |
| `lambda_functions.png` | AWS Lambda functions used for log transformation, Glue job triggering, and Redshift loading |
| `glue_visual_etl.png` | AWS Glue Visual ETL workflow used to design the support ticket transformation |
| `athena_querying.png` | Querying and validating processed datasets using AWS Athena |
| `redshift_editor.png` | Amazon Redshift Serverless tables and SQL query editor |
| `cloudwatch_logs.png` | Amazon CloudWatch Logs used to monitor pipeline execution and troubleshoot issues |
| `iam_roles.png` | IAM roles and permissions configured per service relationship |

Power BI dashboard screenshots live in [`../analytics/`](../analytics/) alongside the `.pbix` file, and the architecture diagrams live in [`../architecture/`](../architecture/).

Each image is referenced from the main [README](../README.md#pipeline-walkthrough) and the relevant folder-level README across this repo.

## A note on redaction

The original screenshots showed the author's real AWS account ID and IAM username in several places (console header, S3 bucket name, IAM role ARN in the Redshift `COPY` command). Those regions have been blacked out in the images above before publishing — the rest of each screenshot (query results, resource names, job configuration, etc.) is untouched and reflects the actual pipeline run.
