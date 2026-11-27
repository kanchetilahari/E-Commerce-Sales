# Data Reconciliation

This document describes the reconciliation checks applied in the E-Commerce Sales Data Engineering Pipeline to verify data consistency, record counts, and revenue accuracy across the Bronze, Silver and Gold layers.

> **Principle:** Data reconciliation ensures that records and business metrics remain consistent across all pipeline layers. Every valid, rejected or excluded record should be accounted for, and Gold revenue calculations should match the validated Silver source data.

## 1. Purpose of Reconciliation

The main purpose of reconciliation is to verify that data is correctly transferred and transformed between the different layers of the Medallion Architecture.

The reconciliation process helps to:

- Verify source record counts against Bronze records.
- Check valid and rejected records in Silver.
- Identify missing or duplicate records.
- Validate delivered-order revenue.
- Compare Gold aggregation totals.
- Ensure accurate business reporting.

## 2. Reconciliation by Layer

### Bronze: Source-to-Bronze Reconciliation

The Bronze layer stores raw e-commerce order records ingested from the source CSV using Auto Loader.

| Rule ID | Reconciliation Check | Expected Result |
|---|---|---|
| BRZ-REC-01 | Compare source CSV row count with Bronze table count | Counts should match |
| BRZ-REC-02 | Verify original source columns | All source columns should be present |
| BRZ-REC-03 | Check ingestion timestamp | Should not be NULL |
| BRZ-REC-04 | Check source file name | Should be populated |
| BRZ-REC-05 | Verify incremental ingestion | New files should be detected |

**Expected result:** Bronze should preserve the original source records without business-level modifications.

### Silver: Bronze-to-Silver Reconciliation

The Silver layer cleans and validates Bronze records and separates valid and rejected records.

| Rule ID | Reconciliation Check | Expected Result |
|---|---|---|
| SLV-REC-01 | Compare Bronze records with Silver records | All records should be accounted for |
| SLV-REC-02 | Count valid records | Records should pass R1–R6 |
| SLV-REC-03 | Count rejected records |
