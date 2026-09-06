# Changelog

All notable changes to this project are documented in this file.

## [Unreleased]
### Planned
- Workflow orchestration with Apache Airflow
- CI/CD deployment pipeline
- Infrastructure as Code (Terraform) for all AWS resources
- Formal data quality validation framework
- Real-time streaming ingestion
- Medallion architecture (Bronze/Silver/Gold) restructuring
- Automated Power BI refresh scheduling

## [1.0.0] — Initial Release
### Added
- Incremental ingestion notebooks for `support_tickets` (MySQL → S3) and `support_logs` (flat files → S3), using date-tracker files
- AWS Lambda transformation for support logs (Python/Pandas/Regex parsing → Parquet), with local exploration and standalone testing notebooks
- AWS Lambda trigger for the AWS Glue transformation job for support tickets (Visual ETL → script-based Glue job → Parquet)
- AWS Glue Crawlers for schema discovery on processed Parquet data
- AWS Athena validation query set (`athena_queries.sql`)
- Amazon Redshift Serverless warehouse with incremental `COPY`-based loading via a Lambda function using `psycopg2`
- Power BI dashboard (`Careplus_Insights.pbix`) connected directly to Redshift, with validated incremental refresh
- Full repository documentation (architecture diagram, folder-level READMEs, screenshot placeholders)
- Synthetic sample dataset (`sample-data/`) standing in for the excluded course dataset, so the pipeline can be run end-to-end without the original data
