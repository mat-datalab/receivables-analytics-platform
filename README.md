# Receivables Analytics Platform

An analytics platform for the accounts receivable department. The system collects data on customers, invoices, payments, past-due accounts, disputes, payment promises, and exchange rates, and then builds an analytics layer to monitor risk, cash collection, and process efficiency.

## Business Problem

Accounts receivable teams need reliable and well-structured data to monitor overdue invoices, customer payment behavior, disputes, and collection performance. In many organizations, this data is spread across multiple operational sources, such as invoice systems, payment records, dispute logs, collector assignments, and exchange rate tables.

Without a clean analytical layer, it is difficult to answer basic business questions such as which customers are driving overdue balances, how much cash is blocked by disputes, which invoices require follow-up, and how effective the collection process is over time.

This project simulates that problem by building an end-to-end analytics platform for receivables data.

## Project Goal

The goal of this project is to build a local data engineering pipeline that ingests receivables-related data, stores it in PostgreSQL, transforms it into analytical models, and prepares business-ready datasets for reporting and analysis.

The project focuses on practical Data Engineering concepts:

- generating and ingesting realistic source data,
- designing raw, staging, and mart layers,
- modeling invoices, customers, payments, disputes, promises to pay, collectors, and exchange rates,
- adding data quality checks,
- documenting technical decisions and trade-offs,
- preparing the project to be extended with orchestration, PySpark, and CI.

## Planned Architecture

The initial version of the project will use a local batch-processing architecture.

Data will be generated or extracted from simple source files and public APIs, then loaded into a PostgreSQL raw layer. From there, dbt models will transform the raw data into cleaned staging tables and analytical marts. Later iterations of the project will add Airflow orchestration, PySpark-based medallion processing, automated tests, and CI checks.

Planned data flow:

```text
Synthetic data / Public API
        |
        v
Raw CSV / JSON files
        |
        v
PostgreSQL raw schema
        |
        v
dbt staging models
        |
        v
dbt analytical marts
        |
        v
SQL analysis / reporting-ready tables
```

## Key Business Questions
1. Which customers have the highest overdue balance?
2. How much receivables are in 0-30, 31-60, 61-90, 90+ aging buckets?
3. Which customer segments pay late most often?
4. How much outstanding amount is blocked by disputes?
5. How often are promises to pay kept or broken?