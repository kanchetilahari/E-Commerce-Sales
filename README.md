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
| Total Records | Verify from source dataset |
| Total Columns | Verify from source dataset |
| Data Domain | E-Commerce / Sales Analytics |
| Processing Platform | Azure Databricks |

The dataset contains order, customer, product, category, quantity, price, city, payment-mode, and order-status-related information.

It is used to demonstrate data ingestion, cleaning, validation, sales calculations, aggregation, and reporting.

## Repository Structure

```text
E-Commerce-Sales-Data-Engineering/
│
├── README.md
│
├── Dataset/
│   └── ecommerce_orders.csv
│
├── Bronze/
│   └── bronze_transformation.sql
│
├── Silver/
│   └── silver_transformation.sql
│
├── Gold/
│   └── gold_transformation.sql
│
├── Dashboard/
│   ├── Sales_Overview/
│   ├── City_and_Category_Analysis/
│   └── Product_and_Payment_Analysis/
│
├── Images/
│   ├── Architecture/
│   ├── Pipeline/
│   └── Dashboard/
│
└── Documentation/
    ├── data_quality_rules.md
    ├── data_reconciliation.md
    └── review_questions.md
```

## Layer Details

### Bronze Layer

- Reads the raw CSV file from a Unity Catalog Volume.
- Stores original sales records.
- Maintains the source data for traceability.
- Preserves raw values for future investigation and reprocessing.
- Provides the starting point for Silver transformations.

**Bronze table:** `bronze_orders`

### Silver Layer

- Removes unnecessary leading and trailing spaces.
- Standardizes text values.
- Identifies missing and invalid values.
- Validates order IDs.
- Checks order dates.
- Validates quantity values.
- Identifies zero or negative unit prices.
- Validates required fields.
- Checks allowed order-status values.
- Calculates order amount using quantity multiplied by unit price.
- Separates clean and rejected records.

**Silver tables:**

- `silver_orders`
- `silver_orders_rejected`

### Gold Layer

Creates business-ready aggregated tables for reporting.

- `gold_city_sales`: Sales analysis by city.
- `gold_category_sales`: Sales analysis by product category.
- `gold_product_sales`: Product-level sales performance.
- `gold_payment_analysis`: Sales analysis by payment mode.
- Gold order-status summary: Understand delivered, shipped, pending, cancelled, and returned orders.

**Revenue calculation:**

`Order Amount = Quantity × Unit Price`

For revenue analysis, the project uses Delivered orders as the revenue population, according to the defined business rule.

## Data Quality Rules

The project applies six main validation rules to improve the accuracy and consistency of sales records.

| Rule | Treatment |
|---|---|
| Order ID validation | Identify missing or empty order IDs |
| Order date validation | Identify invalid order dates |
| Quantity validation | Check that quantity is greater than zero |
| Unit price validation | Check that unit price is greater than zero |
| Required fields validation | Identify missing important business fields |
| Order status validation | Allow only Delivered, Shipped, Pending, Cancelled, and Returned |

Additional cleaning activities include text standardization, data-type conversion, duplicate investigation, and order amount calculation.

## Reconciliation

Data reconciliation checks whether records are correctly processed across Bronze, Silver, and Gold layers.

| Check | Expected Result |
|---|---|
| Source CSV records | Verify against source dataset |
| Bronze records | Match source records |
| Silver clean records | Match valid records |
| Silver rejected records | Match rejected records |
| Remaining records | Investigate any unexplained differences |
| Gold aggregation totals | Verify against Silver |
| Revenue reconciliation | Gold revenue should match delivered-order revenue from Silver |

**Record reconciliation formula:**

`Bronze Records = Silver Clean Records + Silver Rejected Records + Separately Excluded Records`

**Revenue reconciliation formula:**

`Delivered Revenue = SUM(Quantity × Unit Price)`

The Gold city, category, product, and payment-mode revenue totals should agree with the delivered-order revenue calculated from Silver when they use the same population and calculation rules.

## Gold Tables

The Gold layer contains business-ready analytical tables for sales reporting.

| Gold Table | Business Purpose |
|---|---|
| `gold_city_sales` | Understand sales performance across cities |
| `gold_category_sales` | Compare sales across product categories |
| `gold_product_sales` | Identify product-level sales performance |
| `gold_payment_analysis` | Understand customer payment preferences |
| Gold order-status summary | Analyze different order statuses |

## Dashboard

The project includes an interactive Databricks AI/BI Dashboard for analyzing e-commerce sales performance.

### 1. Sales Overview

- Total sales
- Total orders
- Total quantity sold
- Average order amount
- Overall sales insights

### 2. City and Category Analysis

- City-wise sales analysis
- Category-wise sales comparison
- Sales performance across different locations
- Category-level business insights

### 3. Product and Payment Analysis

- Product-level sales performance
- Payment-mode comparison
- Order-status analysis
- Delivered, shipped, pending, cancelled, and returned order insights

## How to Run

1. Upload the e-commerce sales CSV file to a Unity Catalog Volume.
2. Create or select the catalog and schema.
3. Configure the source file path in the Bronze SQL.
4. Add the Bronze, Silver, and Gold SQL files to your Lakeflow Spark Declarative Pipeline.
5. Configure the target catalog and schema.
6. Run the pipeline.
7. Verify the Bronze and Silver tables.
8. Validate clean and rejected records using SQL queries.
9. Validate the Gold tables using reconciliation queries.
10. Create an AI/BI Dashboard using the Gold tables.
11. Use Databricks Genie to explore sales data through natural-language questions.

## Business Requirements Coverage

| ID | Requirement | Implementation |
|---|---|---|
| BR-01 | Raw data ingestion | Bronze table |
| BR-02 | Data cleaning | Silver layer |
| BR-03 | Data validation | Silver validation rules |
| BR-04 | Duplicate investigation | Silver layer |
| BR-05 | Order amount calculation | Silver layer |
| BR-06 | City-wise sales analysis | `gold_city_sales` |
| BR-07 | Category-wise sales analysis | `gold_category_sales` |
| BR-08 | Product-level sales analysis | `gold_product_sales` |
| BR-09 | Payment-mode analysis | `gold_payment_analysis` |
| BR-10 | Order-status analysis | Gold order-status summary |
| BR-11 | Business reporting | AI/BI Dashboard |
| BR-12 | Data reconciliation | Reconciliation checks |

## Key Learnings

- Understanding Medallion Architecture.
- Loading raw data into Azure Databricks.
- Working with Unity Catalog and Volumes.
- Building data transformations using SQL.
- Cleaning and validating sales records.
- Handling invalid and rejected records.
- Understanding duplicate investigation.
- Calculating order amounts and revenue.
- Creating Gold analytical tables.
- Performing data reconciliation.
- Developing interactive AI/BI dashboards.
- Exploring sales data using Databricks Genie.

## Future Improvements

- Implement automated daily data ingestion using Auto Loader.
- Add advanced data-quality monitoring.
- Improve duplicate detection rules.
- Introduce customer and product dimension tables.
- Add more sales KPIs to dashboards.
- Schedule automated pipeline refreshes.
- Integrate additional e-commerce datasets.
- Add automated alerts for pipeline failures.
- Improve incremental processing and schema evolution.

## License

This project is created for educational and portfolio purposes.
