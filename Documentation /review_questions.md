# Project Review Questions and Answers

Answers to eight important technical review questions from the E-Commerce Sales Data Engineering Pipeline, written in the context of this project using Lakeflow Spark Declarative Pipelines on Azure Databricks.

---

## 1. Why do we preserve raw sales data in Bronze instead of cleaning the source immediately?

The Bronze layer stores the original e-commerce sales data received from the source CSV file. Preserving raw data is important because:

- **Traceability:** Cleaned records in Silver can be compared with their original Bronze records.
- **Reprocessability:** If a cleaning rule is incorrect, we can update the rule and process the Bronze data again.
- **Auditability:** We can identify which records were changed, rejected, or retained.
- **Data protection:** Original source values remain available without permanent modification.
- **Flexibility:** New business rules can be applied without uploading the source file again.

If we clean the source immediately, the original values may be lost, making it difficult to investigate data-quality problems.

---

## 2. What is the difference between a data-quality flag and rejecting an order record?

| | Flag | Reject / Exclude |
|---|---|---|
| What happens | The record is retained and marked for investigation | The record is separated from the clean dataset |
| Use when | A value needs further investigation | A record violates a defined rejection rule |
| Example | A suspicious value that needs investigation | Missing order ID or invalid quantity |
| Risk | Suspicious records may enter reports if not filtered | Valid records may be lost if rules are too strict |
| Reversible | Yes | Yes, because Bronze data is preserved |

In this project, data validation is performed in the Silver layer. Valid records are stored in `silver_orders`, while rejected records are stored in `silver_orders_rejected`.

The six main validation rules check order ID, order date, quantity, unit price, required fields and order status.

The actual rejection conditions follow the implemented SQL and pipeline logic.

---

## 3. Why is selecting the correct duplicate key important in an e-commerce sales dataset?

An e-commerce sales dataset may contain multiple orders from the same customer because a customer can purchase different products at different times.

For example, the same customer may purchase:

- Laptop
- Mobile Phone
- Headphones

Therefore, using only the customer ID as a duplicate key could incorrectly remove valid orders.

Duplicate detection should use the actual order-level identifier or a combination of columns that uniquely identifies an order.

Possible fields include:

- Order ID
- Customer ID
- Product Name
- Order Date
- Order Details

Exact duplicates and business-level duplicates should be checked separately.

**Important:** The final duplicate key must be confirmed using the actual dataset columns and business requirements. Order ID alone should not be assumed to be unique without verification.

---

## 4. How would you investigate a missing order ID, invalid quantity, or negative unit price?

The investigation process includes:

1. **Identify:** Find the invalid records in the Silver layer.
2. **Compare:** Check the corresponding original records in Bronze.
3. **Validate order ID:** Check whether the order ID is missing or blank.
4. **Check quantity:** Verify whether the quantity is greater than zero.
5. **Check unit price:** Investigate whether the price is zero, negative or incorrectly entered.
6. **Document:** Record the reason for rejecting the affected record.
7. **Update:** Change the validation rule only when the business requirement supports it.

The main principle is to investigate unusual values before assuming how they should be corrected.

---

## 5. How can you prove that Gold sales totals are consistent with validated Silver data?

We can use SQL reconciliation queries to compare Silver records with Gold aggregations.

Important validation checks include:

- Compare delivered-order counts with Gold delivered-order counts.
- Compare city-level revenue with Silver delivered-order revenue.
- Compare category-level revenue with Silver delivered-order revenue.
- Compare product-level revenue with Silver delivered-order revenue.
- Compare payment-mode revenue with Silver delivered-order revenue.
- Check whether rejected records are excluded from trusted reporting.

Example SQL:

```sql
SELECT
    COUNT(*) AS silver_delivered_count,
    SUM(order_amount) AS silver_delivered_revenue
FROM silver_orders
WHERE order_status = 'Delivered';
```

To validate Gold revenue:

```sql
SELECT
    SUM(total_revenue) AS gold_city_revenue
FROM gold_city_sales;
```

The revenue totals should agree when both queries use the same delivered-order population and calculation rules.

**Note:** Replace table and column names with the actual names in your Databricks catalog and schema.

---

## 6. What would change if new sales files arrived every day?

If new e-commerce sales files arrive daily, the pipeline can be improved to support incremental data processing.

- **Ingestion:** Use Auto Loader to automatically detect and ingest new CSV files.
- **Bronze:** Store incoming records with source-file and ingestion metadata.
- **Deduplication:** Check duplicates across new and previously processed records.
- **Idempotency:** Prevent the same source file from being processed multiple times.
- **Scheduling:** Configure pipeline refreshes or triggers.
- **Monitoring:** Track failed pipeline runs and data-quality issues.
- **Schema evolution:** Handle new columns or changes in the incoming source structure according to the pipeline's schema policy.

This approach helps process new sales data without reprocessing the entire dataset unnecessarily.

---

## 7. What happens if the source sales dataset contains new columns or changes in its schema?

Schema evolution means handling changes in the structure of incoming data.

For example, the source sales CSV may introduce a new column called `discount`.

In this situation:

1. Identify the new column in the source file.
2. Check whether it is required for business reporting.
3. Review the existing Bronze and Silver schemas.
4. Configure the appropriate schema evolution behavior.
5. Update Silver transformations if the new column needs cleaning or validation.
6. Update Gold aggregations if the new field is required for business analysis.
7. Validate the pipeline after the schema change.

The main goal is to handle source schema changes without losing existing data or breaking the pipeline.

---

## 8. How do you ensure that the entire data pipeline is reliable and accurate?

We ensure pipeline reliability and accuracy through data-quality validation, reconciliation and pipeline monitoring.

Important checks include:

- **Bronze validation:** Verify that raw source records are preserved.
- **Silver validation:** Apply the six business validation rules.
- **Rejected record tracking:** Store invalid records separately with rejection reasons.
- **Gold validation:** Verify that business aggregations use the correct Silver records.
- **Record reconciliation:** Compare record counts between Bronze and Silver.
- **Revenue reconciliation:** Compare Gold revenue with delivered-order revenue.
- **Pipeline monitoring:** Check pipeline execution status and investigate failures.
- **Reprocessing:** Correct transformation issues and rerun the pipeline when necessary.

By performing these checks, we can maintain consistent data across all three layers and provide reliable sales information for business reporting.

---

## Conclusion

The E-Commerce Sales Data Engineering Pipeline demonstrates how raw sales data can be transformed into reliable business-ready information using Azure Databricks and Lakeflow Spark Declarative Pipelines.

The project focuses on raw data preservation, data-quality validation, record separation, duplicate investigation, incremental ingestion and reconciliation across Bronze, Silver and Gold layers.
