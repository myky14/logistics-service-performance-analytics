# SQL Query Plan

This document bridges the approved SQL Investigation Framework and the completed Microsoft Access SQL implementation. It translates each business question into an implemented query reference with a clear purpose, expected output, and Power BI use.

Actual query syntax is maintained in the SQL implementation artifacts and Microsoft Access database.

## 1. What is the overall shipment performance?

### Business Question

What is the overall shipment performance?

### Query Name

`qry_01_overall_shipment_performance`

### Purpose

Establish the overall service quality baseline for the shipment network.

### Tables Used

- `tbl_Shipments`
- `tbl_SLA_Targets`

### Join Logic

Join `tbl_Shipments` to `tbl_SLA_Targets` using `Service_Level` and `Destination_Region` where SLA target comparison is needed.

### Filters / Conditions

- Include all shipment records.
- Preserve null values for cost, weight, and POD fields.
- Calculate SLA-related metrics only where a matching SLA target is available.

### Grouping Level

- Overall Network
- Delivery Month (for trend reporting)

### Output Fields

- Delivery month
- Total shipments
- On-time shipments
- Delayed shipments
- On-time rate
- Delay rate
- Average transit time
- Average delay days
- Average POD lag
- SLA compliance rate, if SLA target is joinable

### Metric Logic

Use shipment-level delivery fields to count total, on-time, and delayed shipments. Calculate rates as share of total shipments. Compare actual transit time against SLA target days where the SLA lookup can be joined. Derive delivery month from `Actual_Delivery_Date` to enable trend reporting in Power BI.

### How the Result Will Be Used in Power BI

The output will support KPI cards, delivery-month trend visuals, a high-level performance summary, and baseline context for later drilldowns.

### Expected Business Interpretation

The result should show whether the simulated network has measurable service quality risk before moving into segment-level analysis.

### Notes / Limitations

SLA compliance depends on successful matching between shipment service level, destination region, and the SLA target table.

## 2. Which service levels or regions have the highest delay exposure?

### Business Question

Which service levels or regions have the highest delay exposure?

### Query Name

`qry_02_delay_exposure_by_service_region`

### Purpose

Identify service level and destination region combinations with higher delay exposure.

### Tables Used

- `tbl_Shipments`
- `tbl_SLA_Targets`

### Join Logic

Join `tbl_Shipments` to `tbl_SLA_Targets` using `Service_Level` and `Destination_Region` for SLA target comparison.

### Filters / Conditions

- Include all shipments with service level and destination region values.
- Keep segments visible even when performance is mixed.
- Calculate SLA compliance only when target days are available.

### Grouping Level

- `Service_Level`
- `Destination_Region`

### Output Fields

- Delivery month
- Service level
- Destination region
- Shipment count
- Delayed shipment count
- Delay rate
- Average delay days
- SLA compliance rate
- Average transit time

### Metric Logic

Group shipment outcomes by service level and region. Calculate delay rate and average delay days within each segment. Compare actual transit time against SLA target days when available.

### How the Result Will Be Used in Power BI

The output will support a matrix, clustered bar chart, or heat map showing delay exposure by service and region.

### Expected Business Interpretation

The result should identify which service commitments or destination markets may require deeper investigation.

### Notes / Limitations

Segments with low shipment volume should be interpreted carefully before assigning priority.

## 3. Which operational checkpoint contributes most to shipment delay?

### Business Question

Which operational checkpoint contributes most to shipment delay?

### Query 3A Name

`qry_03a_checkpoint_segment_staging`

### Purpose

Create a shipment-level staging query that pivots checkpoint rows into one row per shipment and calculates checkpoint transition durations.

### Tables Used

- `tbl_Checkpoints`
- `tbl_Shipments`

### Join Logic

Join checkpoint records to shipments using `Shipment_ID`. Use a pivot-style aggregation or self-join pattern to place Origin Scan, Gateway Scan, and Destination Scan timestamps on one shipment row.

### Filters / Conditions

- Include shipments with available Origin Scan, Gateway Scan, and Destination Scan timestamps.
- Do not calculate checkpoint-level SLA metrics because checkpoint-level targets do not exist.
- Preserve shipment-level on-time status for comparison.

### Grouping Level

Shipment level.

### Output Fields

- Shipment_ID
- Service_Level
- Destination_Region
- On_Time_Flag
- Origin_Scan_Timestamp
- Gateway_Scan_Timestamp
- Destination_Scan_Timestamp
- Origin_To_Gateway_Duration
- Gateway_To_Destination_Duration
- Longest_Checkpoint_Segment

### Metric Logic

Use a pivot-style aggregation or self-join pattern to place Origin Scan, Gateway Scan, and Destination Scan timestamps on one shipment row. Then calculate elapsed time between Origin to Gateway and Gateway to Destination.

### How the Result Will Be Used in Power BI

This query can support shipment-level drill-through or serve as the staging input for summary analysis.

### Expected Business Interpretation

The result should show shipment-level checkpoint transition timing and identify the longest checkpoint segment per shipment.

### Notes / Limitations

This query should not calculate group averages.

### Query 3B Name

`qry_03b_checkpoint_segment_summary`

### Purpose

Summarize checkpoint transition duration by business segment to identify operational bottlenecks.

### Tables Used

- `qry_03a_checkpoint_segment_staging`

### Join Logic

Use `qry_03a_checkpoint_segment_staging` as the input query. No additional table join is required unless later segmentation fields are added.

### Filters / Conditions

- Include records with valid checkpoint transition durations.
- Preserve on-time status for comparison between delayed and on-time shipments.

### Grouping Level

- `Service_Level`
- `Destination_Region`
- `On_Time_Flag`
- Checkpoint transition segment

### Output Fields

- Service_Level
- Destination_Region
- On_Time_Flag
- Average origin-to-gateway duration
- Average gateway-to-destination duration
- Shipment count
- Dominant / longest average checkpoint segment

### Metric Logic

Aggregate the staging query from 3A. Compare average segment durations to identify which checkpoint transition contributes more to delay risk.

### How the Result Will Be Used in Power BI

This query supports segment duration bar charts, bottleneck ranking, and matrix visuals.

### Expected Business Interpretation

The result should show which checkpoint transition contributes most to delay risk by service level, region, and on-time status.

### Notes / Limitations

This query depends on `qry_03a_checkpoint_segment_staging`.

## 4. What are the most common incident issue buckets?

### Business Question

What are the most common incident issue buckets?

### Query Name

`qry_04_incident_issue_bucket_summary`

### Purpose

Summarize incident categories associated with delayed shipments.

### Tables Used

- `tbl_Incidents`
- `tbl_Shipments`

### Join Logic

Join incidents to shipments using `Shipment_ID`.

### Filters / Conditions

- Treat `tbl_Incidents` as delayed-shipment incident records.
- Do not interpret incident share as a share of all shipments unless separately calculated.

### Grouping Level

- Issue bucket

### Output Fields

- Issue bucket
- Incident count
- Share of incident records
- Average delay days by issue bucket
- Delayed shipment count by issue bucket

### Metric Logic

Count incident records by issue bucket. Calculate each issue bucket as a share of total incident records. Note that this is not a share of all shipments.

### How the Result Will Be Used in Power BI

The output will support a ranked incident bar chart, root-cause summary, and issue bucket table.

### Expected Business Interpretation

The result should show which incident categories are most visible among delayed shipments.

### Notes / Limitations

Because `tbl_Incidents` exists only for delayed shipments, this query describes the distribution of recorded issue buckets among delayed shipments, not incident rate across the full shipment population.

## 5. Which origin groups or regions should be prioritized for corrective action?

### Business Question

Which origin groups or regions should be prioritized for corrective action?

### Query Name

`qry_05_corrective_action_priority_segments`

### Purpose

Identify origin groups and destination regions with combined delay, incident, and operational bottleneck risk.

### Tables Used

- `tbl_Shipments`
- `tbl_Incidents`
- `tbl_Checkpoints`

### Join Logic

Join incidents and checkpoint summaries to shipments using `Shipment_ID`. This query depend on `qry_03b_checkpoint_segment_summary` for checkpoint duration patterns and `qry_04_incident_issue_bucket_summary` or another incident aggregation query for issue bucket summaries.

### Filters / Conditions

- Use `Origin_Facility` as a proxy analytical grouping field derived from the source vendor field.
- Do not treat origin groups as confirmed real logistics facilities.
- Include segments with enough shipment volume to support interpretation.

### Grouping Level

- `Origin_Facility`
- `Destination_Region`

### Output Fields

- Origin group
- Destination region
- Shipment count
- Delay rate
- Average delay days
- Incident count
- Dominant issue bucket, if feasible
- Checkpoint segment duration patterns, if feasible

### Metric Logic

Output component metrics only: shipment count, delay rate, average delay days, incident count, dominant issue bucket, and checkpoint duration patterns where available. Priority tiering should be handled later in Power BI using transparent DAX rules or visual sorting, not as a weighted Access SQL score.

### How the Result Will Be Used in Power BI

The output will support a priority matrix, heat map, or ranked corrective-action table.

### Expected Business Interpretation

The result should identify high-risk origin groups or regions that deserve review before broad network-level actions are considered.

### Notes / Limitations

This query should be written after Query 3A, Query 3B, and Query 4. Do not create a weighted priority score in Access SQL because it would require assumptions about metric weighting and normalization. This query provides evidence for prioritization, not a final ranking by itself.

## 6. How does transit delay affect CSI score?

### Business Question

How does transit delay affect CSI score?

### Query Name

`qry_06_csi_by_delay_band`

### Purpose

Assess whether customer satisfaction differs across delay bands.

### Tables Used

- `tbl_Shipments`
- `tbl_CSI_Scores`

### Join Logic

Join CSI records to shipments using `Shipment_ID`.

### Filters / Conditions

- Include shipments with valid CSI scores.
- Assign shipments into delay bands:
  - On time / early
  - 1-3 days late
  - 4-7 days late
  - 8+ days late

### Grouping Level

- Delay band

### Output Fields

- Delay band
- Shipment count
- Average CSI
- Minimum CSI
- Maximum CSI
- Average delay days

### Metric Logic

Classify shipments into delay bands using transit delay days. Calculate CSI summary metrics and average delay days within each band.

### How the Result Will Be Used in Power BI

The output will support a line chart, scatter-style summary, or bar chart comparing CSI by delay band.

### Expected Business Interpretation

The result should show whether CSI score declines as delay severity increases in the simulated model.

### Notes / Limitations

This analysis supports pattern interpretation within the model and should not be presented as real-world causal proof.

## 7. How does cost per move vary by service level?

### Business Question

How does cost per move vary by service level?

### Query Name

`qry_07_cost_per_move_by_service_level`

### Purpose

Compare captured shipment cost patterns by service level.

### Tables Used

- `tbl_Shipments`

### Join Logic

No join required unless later analysis adds related dimensions.

### Filters / Conditions

- Do not treat null shipment cost as zero.
- Calculate cost metrics only for shipments where `Shipment_Cost_USD` is available.
- Calculate average cost per KG only where both `Shipment_Cost_USD` and `Weight_KG` are valid.

### Grouping Level

- `Service_Level`

### Output Fields

- Service level
- Shipment count
- Cost-captured shipment count
- Cost capture rate
- Average cost per move
- Total captured shipment cost
- Optional average cost per KG

### Metric Logic

Count all shipments and cost-captured shipments separately. Calculate cost capture rate and average cost per move using only valid cost records.

### How the Result Will Be Used in Power BI

The output will support a service-level cost bar chart, cost capture indicator, and service mix comparison.

### Expected Business Interpretation

The result should show how captured shipment cost varies by service level and whether missing cost records affect interpretation.

### Notes / Limitations

Cost and weight fields have partial availability, so cost metrics should include capture-rate context.

## 8. How timely is proof of delivery recorded after shipment delivery?

### Business Question

How timely is proof of delivery recorded after shipment delivery?

### Query Name

`qry_08_pod_timeliness_summary`

### Purpose

Evaluate POD recording timeliness after physical delivery.

### Tables Used

- `tbl_Shipments`

### Join Logic

No join required.

### Filters / Conditions

- Late POD = `POD_Lag_Days` greater than 1.
- Negative POD lag = `POD_Lag_Days` less than 0.
- High POD lag = `POD_Lag_Days` greater than 7.
- Preserve unusual POD lag values for data quality visibility.

### Grouping Level

- `Service_Level`
- `Destination_Region`

### Output Fields

- Service level
- Destination region
- Shipment count
- Average POD lag
- Late POD count
- Late POD rate
- Negative POD lag count
- High POD lag count
- Data timeliness risk category

### Metric Logic

Use `POD_Lag_Days` to classify late, negative, and high POD lag records. Summarize timeliness patterns by service level and region.

### How the Result Will Be Used in Power BI

The output will support a KPI card for average POD lag, bar chart by service level or region, distribution chart, and exception table.

### Expected Business Interpretation

The result should show whether POD recording delays create data timeliness risk and whether that risk is concentrated in specific segments.

### Notes / Limitations

Negative and unusually high POD lag values should be treated as source-driven exceptions for investigation, not automatically corrected in SQL.
