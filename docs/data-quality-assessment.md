# Data Quality Assessment

## Dataset Overview

- Source dataset: SCMS Delivery History Dataset
- Rows: 10,324
- Original columns: 35

## Data Profiling Summary

The dataset was profiled in Power Query before transformation and dashboard modeling.

![Power Query ETL](../screenshots/powerquery-etl.png)

Key findings:

- No duplicate `Shipment_ID` values were found.
- No parsing errors were found.
- One corrupted encoding value in `Country` was corrected to `Côte d'Ivoire`.
- A BOM issue in the ID header was corrected.
- `Shipment Mode` contains missing values.
- `Weight (Kilograms)` and `Freight Cost (USD)` contain placeholder text instead of numeric values.

## Cleaning Actions Performed

- Removed 20 non-business columns.
- Renamed business columns to the final analytics schema.
- Reframed business values into an express logistics context.
- Converted `Weight_KG` and `Shipment_Cost_USD` to numeric values while preserving placeholders as null.
- Created `Is_Weight_Captured`.
- Created `Is_Cost_Captured`.
- Converted delivery date fields to Date.
- Added `Transit_Time_Actual_Days`.
- Added `On_Time_Flag`.
- Added `POD_Lag_Days`.
- Added `Destination_Region` using a lookup table merge instead of nested IF statements.

## Data Quality Decisions

The raw dataset remains unchanged so the original source data can be referenced or reprocessed if needed.

Null values were preserved instead of being replaced with zero because missing weight or cost is different from a true zero value. Preserving nulls prevents misleading averages, totals, and cost-per-shipment calculations.

A lookup table and merge were used for `Destination_Region` because this approach is easier to maintain than nested IF statements. Region assignments can be reviewed and updated in one place without rewriting transformation logic.

Business terminology was standardized for an express logistics scenario so the final table supports service quality analysis while remaining traceable to the original source fields.

## Known Limitations

- The dataset originally belongs to a healthcare supply chain domain.
- Some `POD_Lag_Days` values are negative or unusually large and were intentionally preserved because they originate from the source data.
- Weight and shipment cost are only available for approximately 60% of shipments.

## Phase 2 Summary

- Four analytical tables were generated: `SLA_Targets`, `Checkpoints`, `Incidents`, and `CSI_Scores`.
- Synthetic data follows deterministic business rules.
- No random functions are used.
- All generated datasets remain refreshable through Power Query.

## SQL Validation Note

- During SQL validation, checkpoint timestamp sequence issues were detected and corrected at the Power Query generation layer.
- Final checkpoint records follow `Origin Scan <= Gateway Scan <= Destination Scan`.

## Dashboard Readiness

- Microsoft Access SQL outputs were validated before use in Power BI.
- Final dashboard visuals preserve the documented data quality limitations.
- Cost Capture Rate and Late POD Rate are shown with exception context to avoid overreading partial or unusual records.
