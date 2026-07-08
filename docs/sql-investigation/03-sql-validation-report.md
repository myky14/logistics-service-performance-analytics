# SQL Validation Report

## 1. Purpose

This report documents validation performed after SQL implementation and before Power BI dashboard development. It confirms that the Access SQL outputs are reliable enough to support dashboard visuals, interpretation, and later business recommendations.

## 2. Validation Scope

Completed SQL queries:

- `qry_01_overall_shipment_performance`: validates monthly overview performance.
- `qry_02_delay_exposure_by_service_region`: validates delay exposure by service level, region, and month.
- `qry_03a_checkpoint_segment_staging`: validates shipment-level checkpoint sequence and segment durations.
- `qry_03b_checkpoint_segment_summary`: validates checkpoint duration summaries by business segment.
- `qry_04_incident_issue_bucket_summary`: validates incident issue bucket distribution.
- `qry_05_corrective_action_priority_segments`: validates corrective action segment evidence.
- `qry_06_csi_by_delay_band`: validates CSI patterns by delay severity.
- `qry_07_cost_per_move_by_service_level`: validates captured cost metrics by service level.
- `qry_08_pod_timeliness_summary`: validates POD timeliness and exception visibility.

## 3. Validation Principles

- Validate query grain.
- Validate row counts.
- Validate join coverage.
- Validate null handling.
- Validate denominators for rates.
- Investigate abnormal values instead of hiding them.
- Fix source/data-model issues at the correct pipeline layer.
- Do not patch business logic in SQL just to make results look clean.

## 4. Query Validation Summary

| Query | Business Question | Key Validation Checks | Result | Notes |
|---|---|---|---|---|
| `qry_01_overall_shipment_performance` | What is the overall shipment performance? | Monthly grouping, delivery performance metrics, rate denominators | Passed | Validates monthly overview performance for Power BI trend and KPI visuals. |
| `qry_02_delay_exposure_by_service_region` | Which service levels or regions have the highest delay exposure? | Service level, region, month grouping, delay rate denominator, SLA join coverage | Passed | Supports service and region delay exposure analysis. |
| `qry_03a_checkpoint_segment_staging` | Which operational checkpoint contributes most to shipment delay? | One row per shipment, checkpoint timestamp sequence, negative duration count | Passed after source fix | Initial negative Gateway-to-Destination durations were resolved by regenerating `tbl_Checkpoints` in Power Query. |
| `qry_03b_checkpoint_segment_summary` | Which operational checkpoint contributes most to shipment delay? | Aggregation from `qry_03a`, segment duration summaries, business segment grain | Passed after source fix | Rerun after `tbl_Checkpoints` was re-imported. |
| `qry_04_incident_issue_bucket_summary` | What are the most common incident issue buckets? | Issue bucket grouping, incident denominator, share total | Passed | Incident shares sum to approximately 100% across issue buckets. |
| `qry_05_corrective_action_priority_segments` | Which origin groups or regions should be prioritized for corrective action? | Minimum shipment threshold, origin/region grain, no weighted score | Passed | Includes `HAVING Count(Shipment_ID) >= 30`; blank `Destination_Region` values remain as-is. |
| `qry_06_csi_by_delay_band` | How does transit delay affect CSI score? | Delay band grouping, CSI scale check, average CSI by band | Passed | CSI is modeled on a 0-100 scale and average CSI declines as delay severity increases in the simulated model. |
| `qry_07_cost_per_move_by_service_level` | How does cost per move vary by service level? | Cost capture denominator, null cost handling, captured cost totals | Passed | Cost metrics use captured cost records only; cost and weight nulls are not imputed. |
| `qry_08_pod_timeliness_summary` | How timely is proof of delivery recorded after shipment delivery? | POD lag flags, late/negative/high POD counts, exception preservation | Passed | Negative and high POD lag values are retained as data exceptions for visibility. |

## 5. Checkpoint Duration Issue and Resolution

During validation of `qry_03a_checkpoint_segment_staging`, negative Gateway-to-Destination duration values were found. Around 2,000 records initially had negative duration, where the Destination Scan timestamp appeared before the Gateway Scan timestamp.

The issue was traced to the generated `tbl_Checkpoints` source table, not to SQL join logic. The correct fix was applied in Power Query by regenerating `tbl_Checkpoints` so checkpoint sequence follows:

```text
Origin Scan <= Gateway Scan <= Destination Scan
```

After regeneration, `tbl_Checkpoints` was re-imported into Access and `qry_03a` and `qry_03b` were rerun. Validation confirmed the negative Gateway-to-Destination duration count became 0.

This was treated as a realistic QA finding: the abnormal result was investigated, traced to the correct pipeline layer, fixed upstream, and revalidated.

## 6. Data Quality Notes

- Blank `Destination_Region` values remain as-is and should be interpreted carefully in dashboard/reporting.
- CSI score is modeled on a 0-100 scale.
- Cost and weight fields have partial capture and nulls are preserved.
- POD lag exceptions are retained for visibility.
- `Origin_Facility` is a proxy grouping field derived from the source dataset, not a confirmed real logistics facility.

## 7. Power BI Readiness

SQL outputs are now ready to support:

- Overview performance page
- Delay exposure analysis
- Checkpoint bottleneck analysis
- Incident/root-cause analysis
- CSI/customer experience analysis
- Cost per move analysis
- Data timeliness analysis
- Corrective action priority view

## 8. Sign-Off

The SQL investigation layer is validated and ready for Power BI dashboard development.
