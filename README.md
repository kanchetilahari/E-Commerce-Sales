# E-Commerce-Sales

# E-Commerce Sales Data Engineering Pipeline: Medallion Architecture with Lakeflow Spark Declarative Pipelines

An end-to-end e-commerce sales data engineering project built using **Azure Databricks**, **Lakeflow Spark Declarative Pipelines (SQL)**, and **Medallion Architecture** (Bronze → Silver → Gold).

This project transforms raw e-commerce sales data into clean, structured, and business-ready analytical tables. It includes data ingestion, data cleaning, validation, data-quality tracking, reconciliation, and business reporting.

## Table of Contents

- Business Problem
- Architecture
- Pipeline Lineage
- Tech Stack
- Dataset
- Repository Structure
- Layer Details
- Data Quality Rules
- Reconciliation
- Gold Tables
- Dashboard
- How to Run
- Business Requirements Coverage
- Key Learnings
- Future Improvements

## Business Problem

E-commerce platforms generate large amounts of sales data from different customers, products, categories, cities, payment modes, and order statuses.

Raw sales data may contain missing values, inconsistent text, duplicate records, invalid dates, zero or negative quantities, negative unit prices, and invalid order statuses.

Manual data cleaning makes sales reporting difficult and less reliable.

**Goal:** Build a repeatable data pipeline that preserves raw sales data, cleans and validates records, and creates trusted Gold tables for sales analysis and business reporting.

## Architecture

`ecommerce_orders.csv` (Unity Catalog Volume) → **Bronze** (raw data) → **Silver** (cleaned and validated data) → **Gold** (business aggregations) → Databricks AI/BI Dashboard / Genie

| Layer | Purpose | Key Actions |
|---|---|---|
| Bronze | Preserve source data | Load raw CSV and maintain original records |
| Silver | Improve data quality | Clean text, validate records, identify duplicates, separate rejected records |
| Gold | Support business analysis | Create city, category, product, payment-mode, and order-status summaries |

## Pipeline Lineage

The pipeline processes e-commerce sales data through three layers using Lakeflow Spark Declarative Pipelines.

| Object | Type | Description |
|---|---|---|
| `bronze_orders` | Bronze table | Raw sales records |
| `silver_orders` | Silver table | Cleaned and validated sales records |
| `silver_orders_rejected` | Silver table | Rejected sales records |
| `gold_city_sales` | Gold table | Sales summary by city |
| `gold_category_sales` | Gold table | Sales summary by category |
| `gold_product_sales` | Gold table | Product-level sales summary |
| `gold_payment_analysis` | Gold table | Sales analysis by payment mode |
| Gold order-status summary | Gold table | Order-status analysis |

*Confirm the exact Gold table names against your implemented pipeline before publishing.*

## Tech Stack

- Azure Databricks
- Lakeflow Spark Declarative Pipelines
- SQL
- Unity Catalog
- Unity Catalog Volumes
- Delta Lake
- Databricks AI/BI Dashboards
- Databricks Genie

## Dataset

The project uses an e-commerce sales dataset in CSV format.

| Dataset Information | Details |
|---|---|
| File Name | ecommerce_orders.csv |
| File Format | CSV |
| Total
