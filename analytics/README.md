# Analytics

Validation queries, a schema-check notebook, and the final Power BI reporting layer.

## What's included

| File | Description |
|---|---|
| `athena_queries.sql` | Validation and exploratory SQL queries run against Glue Crawler-generated tables in AWS Athena, before data is trusted for warehouse loading |
| `parquet_files_overview.ipynb` | Notebook used to independently inspect the processed Parquet outputs (schema, dtypes, row counts, sample rows) via `pyarrow` and `pandas` |
| `Careplus_Insights.pbix` | Power BI dashboard file, connected directly to Amazon Redshift Serverless |
| `careplus_ticket_insights_dashboard.png` | Screenshot of the **Careplus Ticket Insights** dashboard page |
| `careplus_support_logs_dashboard.png` | Screenshot of the **Careplus - Support Logs** dashboard page |

## Athena: validation before warehouse load

Before any processed data is loaded into Redshift, **AWS Glue Crawlers** scan the Parquet files in the S3 Processed Zone and register them as queryable tables (`support_tickets_processed`, `support_logs_processed`). **AWS Athena** then runs SQL directly against those tables — no warehouse required — to catch problems early.

`athena_queries.sql` covers:
- **Ticket load by channel** — Email, Chat, Phone, Web Form volumes, to inform staffing decisions
- **Ticket status breakdown** — Resolved / Open / Escalated counts
- **Event counts and CPU usage per user agent** — from `support_logs_processed`
- **Ticket trend by day** — daily created-ticket volume
- **DEBUG-level log event count** — a simple data-quality/sanity check

See [`../screenshots/athena_querying.png`](../screenshots/athena_querying.png) for a console view of a query like this running against the processed data.

## Schema validation notebook

`parquet_files_overview.ipynb` reads both processed Parquet outputs directly with `pyarrow.parquet` and `pandas`, printing the inferred schema, dtypes, and shape, and previewing sample rows — confirming column types and row counts matched expectations before wiring up the Glue Crawlers and Athena queries above.

## Power BI: two-page dashboard

The `.pbix` file connects live to Amazon Redshift and has two report pages:

### `Careplus Ticket Insights`

![Careplus Ticket Insights](careplus_ticket_insights_dashboard.png)

- **Headline KPIs** — total tickets, resolved/open/escalated counts, total agents, average interactions per ticket, average resolution time (minutes)
- **Tickets by channel & status** — stacked bar showing resolution/escalation rate per channel (Chat, Email, Phone, Web Form)
- **Tickets by agent** — workload distribution across the support team
- **Avg. resolution time by issue category and priority** — where tickets take longest to close
- **Ticket-level drill-through table** — per-ticket detail (agent, resolution time, issue category) for ad-hoc investigation

### `Careplus - Support Logs`

![Careplus Support Logs](careplus_support_logs_dashboard.png)

- **Headline KPIs** — total logs, logged tickets, average CPU (%), average response time (ms)
- **Day selector** — a date strip (01–07 July 2025 shown) to filter the whole page to a single day
- **Avg CPU (%) by timestamp** — intraday CPU trend line
- **Total logs by user agent** — event volume split by client (curl, Python-urllib, Mobile-Safari, PostmanRuntime, etc.)
- **Total logs by log level** — INFO / DEBUG / WARNING / ERROR breakdown
- **Total logs and logged tickets by response-time bucket** — where response times cluster (e.g. 1000–1499 ms) and how that maps to ticket volume

### Incremental refresh — validated

```
New Data → Amazon S3 → Transformation → Amazon Redshift → Power BI Refresh → Updated Dashboard
```

New records become queryable in Redshift once the incremental load completes. Refreshing the Power BI dataset then updates every visual on both pages with no changes to the report itself — confirming the pipeline works end-to-end, from raw ingestion through to a live dashboard.

## How to view

- Open `Careplus_Insights.pbix` in Power BI Desktop (requires a Redshift connection to refresh live; otherwise it opens with the last-refreshed cached data).
- Or view the two PNG screenshots above for a static look at each page without Power BI Desktop installed.
