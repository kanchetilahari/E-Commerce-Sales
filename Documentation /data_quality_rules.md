# Data Quality Rules

This document describes the data-quality (DQ) rules applied in the E-Commerce Sales Data Engineering Pipeline, how each issue is treated, and where it is enforced.

> **Principle:** Raw e-commerce sales data is preserved in the Bronze layer. Data issues are identified, cleaned, standardized, flagged, or rejected according to documented rules. Gold tables contain business-ready aggregated data for trusted reporting and analytics.

## 1. Known Issues in the Source Data

The source file (`ecommerce_orders.csv`) contains e-commerce order information with 11 expected columns.

The pipeline is designed to handle the following data-quality issues:

| # | Issue | Example |
|---|---|---|
| 1 | Leading/trailing whitespace | `" Hyderabad "` instead of `"Hyderabad"` |
| 2 | Inconsistent text formatting | `"delivered"` instead of `"Delivered"` |
| 3 | Missing values | NULL or blank customer ID |
| 4 | Invalid dates | Incorrect order date format |
| 5 | Invalid quantity | Zero or negative quantity |
| 6 | Invalid unit price | Zero or negative unit price |
| 7 | Missing order ID | Empty or NULL order ID |
| 8 | Missing required fields | Missing product name, category or city |
| 9 | Invalid order status | Status outside the accepted values |

## 2. Rules by Layer

### Bronze: Preserve Raw Data

The Bronze layer stores the original e-commerce order data without applying business-level cleaning.

| Rule ID | Rule | Treatment |
|---|---|---|
| BRZ-01 | Preserve original source values | No business-level modifications |
| BRZ-02 | Store source order records | Load raw CSV data using Auto Loader |
| BRZ-03 | Maintain source record count | Validate against source CSV |
| BRZ-04 | Preserve original data for auditing | Retain raw records |
| BRZ-05 | Capture ingestion timestamp | Add ingestion metadata |
| BRZ-06 | Capture source file information | Store source file name |

### Silver: Clean, Standardize and Validate

The Silver layer performs data cleaning, standardization, type conversion and business validation.

| Rule ID | Dimension | Rule | Treatment | Action |
|---|---|---|---|---|
| SLV-01 | Standardization | Remove leading and trailing spaces | TRIM() | Fix |
| SLV-02 | Standardization | Standardize customer, city, product and category text | Consistent text formatting | Fix |
| SLV-03 | Completeness | Identify missing or blank required values | NULL and blank checks | Reject |
| SLV-04 | Date validity | Order date must be a valid date | Date validation | Reject |
| SLV-05 | Business validity | Quantity must be greater than zero | Range validation | Reject |
| SLV-06 | Business validity | Unit price must be greater than zero | Price validation | Reject |
| SLV-07 | Status validation | Order status must belong to accepted values | Status classification | Reject |
| SLV-08 | Type conversion | Convert order date, quantity and unit price into correct data types | DATE, INT, DECIMAL | Fix |
| SLV-09 | Derived values | Calculate order amount using quantity × unit price | Calculation | Derive |
| SLV-10 | Record separation | Separate valid and rejected records | Validation conditions | Separate |

**Accepted order statuses:**
- Delivered
- Shipped
- Pending
- Cancelled
- Returned

**Required fields:**
- order_id
- order_date
- customer_id
- product_name
- category
- city
- quantity
- unit_price
- payment_mode
- order_status

Note: According to the current project specification, customer_name is allowed to be empty.

### Gold: Business-Ready Data

The Gold layer contains aggregated datasets used for business reporting and dashboard development.

| Rule ID | Rule | Treatment |
|---|---|---|
| GLD-01 | Delivered-order revenue analysis uses only Delivered records | Filter |
| GLD-02 | City-wise sales summaries use validated delivered orders | Aggregate |
| GLD-03 | Category-wise sales summaries calculate sales and units sold | Aggregate |
| GLD-04 | Product-wise sales summaries provide product performance insights | Aggregate |
| GLD-05 | Payment analysis summarizes sales by payment mode | Aggregate |
| GLD-06 | Order summary includes valid records across order statuses | Aggregate |
| GLD-07 | Overall sales summary provides business KPIs | Aggregate |
| GLD-08 | Gold revenue is checked against delivered-order records | Reconcile |

Note: `gold_order_summary` and `gold_sales_summary` are planned tables in the current implementation documentation and must be completed and validated.

## 3. Flag vs. Reject

| | **Flag** | **Reject / Exclude** |
|---|---|---|
| Meaning | Record is retained but marked for investigation | Record is separated from clean reporting data |
| Use when | A value needs investigation | A record violates a defined business validation rule |
| Example | A suspicious value requiring review | Missing order ID or invalid quantity |
| Visible in | Silver quality checks | `silver_orders_rejected` |
| Reversible | Yes | Yes, because raw data remains in Bronze |

In this project, the six business validation rules determine whether a record is accepted into `silver_orders` or separated into `silver_orders_rejected`.

## 4. Data Quality Validation Columns

The following checks describe the quality information maintained during Silver processing.

| Column / Check | Type | Description |
|---|---|---|
| order_id validation | Boolean | Identifies missing order IDs |
| order_date validation | Boolean | Identifies invalid order dates |
| quantity validation | Boolean | Identifies zero or negative quantities |
| unit_price validation | Boolean | Identifies zero or negative prices |
| Required fields validation | Boolean | Identifies missing required information |
| order_status validation | Boolean | Identifies unsupported order statuses |
| order_amount | Decimal(18,2) | Stores calculated order amount |
| rejection_reasons | String / Array | Describes why a record failed validation |
| Clean record | Boolean | Indicates whether the record passes all validation rules |

*Note: These are logical quality checks. Use the actual column names from your implemented Silver tables in the final pipeline.*

## 5. Pipeline Expectations

The Silver layer performs validation before records are used for Gold reporting.

Example SQL validation queries:

```sql
-- Check missing order IDs
SELECT *
FROM silver_orders_typed
WHERE order_id IS NULL OR TRIM(order_id) = '';

-- Check invalid order dates
SELECT *
FROM silver_orders_typed
WHERE order_date IS NULL;

-- Check invalid quantities
SELECT *
FROM silver_orders_typed
WHERE quantity IS NULL OR quantity <= 0;

-- Check invalid unit prices
SELECT *
FROM silver_orders_typed
WHERE unit_price IS NULL OR unit_price <= 0;

-- Check missing required fields
SELECT *
FROM silver_orders_typed
WHERE customer_id IS NULL
   OR product_name IS NULL
   OR category IS NULL
   OR city IS NULL
   OR payment_mode IS NULL;

-- Check invalid order statuses
SELECT *
FROM silver_orders_typed
WHERE order_status NOT IN (
    'Delivered',
    'Shipped',
    'Pending',
    'Cancelled',
    'Returned'
);
```

*Note: These are example validation queries. Replace table names with the fully qualified names used in your Databricks catalog and schema.*

## 6. Investigating Suspicious Values

When an invalid e-commerce order record is identified:

1. Identify the affected record in the Silver layer.
2. Compare the record with its original Bronze data.
3. Check order ID, customer information, product details and order date.
4. Validate quantity and unit price.
5. Check whether the order status belongs to the accepted values.
6. Determine the appropriate cleaning or rejection rule.
7. Store invalid records in `silver_orders_rejected` with rejection reasons.
8. Verify that Gold sales summaries are not affected by rejected records.

## 7. Data Quality Issue Summary

The following counts should be filled after running the corresponding validation queries against the actual pipeline output.

| Issue | Rule ID | Count | Treatment |
|---|---|---:|---|
| Missing order IDs | R1 | TODO | Rejected |
| Invalid order dates | R2 | TODO | Rejected |
| Invalid quantities | R3 | TODO | Rejected |
| Invalid unit prices | R4 | TODO | Rejected |
| Missing required fields | R5 | TODO | Rejected |
| Invalid order statuses | R6 | TODO | Rejected |
| Duplicate records | Pending | TODO | Investigate |
| Clean records | SLV-10 | TODO | Retained |
| Rejected records | SLV-10 | TODO | Separated |

*Note: Duplicate handling is pending in the current implementation. The actual order grain must be confirmed before applying deduplication. Do not assume that order_id alone is unique.*

## 8. Assumptions and Decisions

- Bronze data is preserved as the original source.
- Auto Loader is used for incremental CSV ingestion.
- Text standardization is performed in Silver.
- Blank values are normalized to NULL.
- Order dates are converted into DATE format.
- Quantity is converted into INTEGER.
- Unit price is converted into DECIMAL(12,2).
- Order amount is calculated using quantity multiplied by unit price.
- Invalid records are separated into `silver_orders_rejected`.
- Customer name is allowed to be empty.
- Duplicate handling requires confirmation of the source data grain.
- Gold revenue analysis uses Delivered orders only.
- Other valid order statuses remain available for order lifecycle analysis.
- Final issue counts must be verified against actual pipeline output.

## 9. Conclusion

Data quality rules help maintain accurate, complete and consistent e-commerce sales information.

The Bronze layer preserves the original raw data, the Silver layer cleans and validates order records, and the Gold layer provides business-ready summaries for sales analytics and dashboard reporting.

By implementing these rules, the E-Commerce Sales Data Engineering Pipeline improves data reliability, maintains traceability and supports trustworthy business decisions.
