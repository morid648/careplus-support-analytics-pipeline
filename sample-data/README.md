# Sample Data

The original datasets used to build and run this pipeline — a MySQL dump (`careplus_support_db.sql`, ~90k rows) and 31 days of raw `.log` files — are **course-provided content** (from a Codebasics data engineering exercise) and are **not redistributed in this public repository**.

This folder contains a small, entirely **synthetic** stand-in dataset instead: fabricated records, generated to match the exact schema, column types, and file formats of the original data, so that the notebooks in `ingestion/` and `transformation/` can still be pointed at real files and run end-to-end.

## What's included

| Path | Description |
|---|---|
| `support-tickets/support_tickets_sample.csv` | 12 fabricated ticket records in the `support_tickets` schema |
| `support-tickets/careplus_support_db_sample.sql` | A minimal `CREATE TABLE` + `INSERT` script that seeds the same 12 fabricated tickets into a local MySQL database — a lightweight stand-in for the full course DB dump |
| `support-logs/support_logs_sample.log` | 20 fabricated log entries in the exact same `.log` text format as the original data, referencing the same sample ticket IDs above |

## Notes on the sample data

- All ticket IDs, timestamps, agent names, IP addresses, and log content are randomly generated — none of it reflects real users, real support interactions, or the original course dataset's actual values.
- The log file was validated against the same regex pattern used in `transformation/support-log-transformation/` — all 20 sample entries parse cleanly.
- This sample is deliberately small (12 tickets / 20 log lines) — enough to demonstrate the pipeline mechanics (ingestion, parsing, transformation, loading), not to reproduce dashboard-scale volumes. The numbers shown in `analytics/careplus_ticket_insights_dashboard.png` and `analytics/careplus_support_logs_dashboard.png` were produced against the original (excluded) dataset, not this sample.

## How to use it

- **Tickets**: run `support-tickets/careplus_support_db_sample.sql` against a local MySQL instance to create and populate `careplus_support_db`, then point `ingestion/support-tickets/support_tickets_ingestion_to_S3.ipynb` at it.
- **Logs**: copy `support-logs/support_logs_sample.log` into a local `day-wise-logs-data/` folder (renaming it to match the date your `log_date_tracker.txt` expects next), then run `ingestion/support-logs/support_logs_ingestion_to_S3.ipynb`.
