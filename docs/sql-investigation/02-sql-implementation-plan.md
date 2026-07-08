# SQL Implementation Plan

## 1. Purpose

This document turns the approved SQL Query Plan into an implementation roadmap for Microsoft Access SQL. It clarifies query layers, dependencies, execution order, validation expectations, and Power BI handoff.

SQL implementation should follow the business questions, not the other way around. Each query should exist because it supports a specific analytical question, visual, or business decision.

Implementation has been completed and validated in [03-sql-validation-report.md](03-sql-validation-report.md).

## 2. Implementation Principles

- Business question first, SQL second.
- Use saved Access queries for reusable staging and summary logic.
- Keep query grain clear: shipment-level queries should not mix with aggregate outputs.
- Avoid unsupported or overly complex Access SQL patterns when a staging query is clearer.
- Preserve null handling for cost, weight, and POD fields.
- Do not create artificial checkpoint compliance because checkpoint-level SLA targets do not exist.
- Do not create weighted priority scores in Access SQL.
- Use SQL outputs as evidence for Power BI visuals and recommendations.

## 3. Query Layer Architecture

### Layer 1 — Base / Direct Investigation Queries

These queries can be written directly from source tables or create reusable staging outputs.

- `qry_01_overall_shipment_performance`
- `qry_02_delay_exposure_by_service_region`
- `qry_03a_checkpoint_segment_staging`
- `qry_04_incident_issue_bucket_summary`
- `qry_06_csi_by_delay_band`
- `qry_07_cost_per_move_by_service_level`
- `qry_08_pod_timeliness_summary`

### Layer 2 — Derived / Summary Queries

These queries depend on Layer 1 and turn staging outputs into business-level summaries.

- `qry_03b_checkpoint_segment_summary`
  - Depends on `qry_03a_checkpoint_segment_staging`

### Layer 3 — Business Prioritization Query

These queries combine multiple evidence streams.

- `qry_05_corrective_action_priority_segments`
  - Depends on `tbl_Shipments`
  - Depends on `tbl_Incidents` or shipment-level incident fields
  - Depends on `qry_03a_checkpoint_segment_staging` for shipment-level checkpoint duration evidence
  - May reference `qry_04_incident_issue_bucket_summary` for context, but should not rely on it as the direct join source because `qry_04` is grouped by `Issue_Bucket`

This query should aggregate all evidence at the same target grain: `Origin_Facility` x `Destination_Region`.

## 4. Query Dependency Map

| Query | Layer | Depends On | Output Grain | Power BI Use |
|---|---|---|---|---|
| `qry_01_overall_shipment_performance` | Layer 1 | `tbl_Shipments`, `tbl_SLA_Targets` | Delivery month / overall network | Overview KPI and trend |
| `qry_02_delay_exposure_by_service_region` | Layer 1 | `tbl_Shipments`, `tbl_SLA_Targets` | Service level x region x delivery month, if applicable | Delay exposure matrix / heat map |
| `qry_03a_checkpoint_segment_staging` | Layer 1 | `tbl_Checkpoints`, `tbl_Shipments` | Shipment level | Drill-through and staging |
| `qry_03b_checkpoint_segment_summary` | Layer 2 | `qry_03a_checkpoint_segment_staging` | Service level x region x on-time flag | Checkpoint bottleneck visuals |
| `qry_04_incident_issue_bucket_summary` | Layer 1 | `tbl_Incidents`, `tbl_Shipments` | Issue bucket | Incident root-cause visuals |
| `qry_05_corrective_action_priority_segments` | Layer 3 | `tbl_Shipments`, `tbl_Incidents`, `qry_03a_checkpoint_segment_staging` | Origin group x destination region | Corrective action priority table / matrix |
| `qry_06_csi_by_delay_band` | Layer 1 | `tbl_Shipments`, `tbl_CSI_Scores` | Delay band | CSI impact visuals |
| `qry_07_cost_per_move_by_service_level` | Layer 1 | `tbl_Shipments` | Service level | Cost per move visuals |
| `qry_08_pod_timeliness_summary` | Layer 1 | `tbl_Shipments` | Service level x region | Data timeliness visuals |

## 5. Recommended Build Order

1. `qry_01_overall_shipment_performance`
   - Builds the overall performance baseline and confirms core shipment fields are usable.
2. `qry_03a_checkpoint_segment_staging`
   - Builds the most technically risky Access query early because checkpoint pivot/self-join logic is harder to debug late in the process.
3. `qry_07_cost_per_move_by_service_level`
   - Validates null handling for cost fields early because cost data is partially captured.
4. `qry_08_pod_timeliness_summary`
   - Tests POD lag rules and preserves unusual POD lag values for visibility.
5. `qry_02_delay_exposure_by_service_region`
   - Extends the overall performance logic into service level and region segments.
6. `qry_04_incident_issue_bucket_summary`
   - Summarizes incident evidence needed for later prioritization.
7. `qry_06_csi_by_delay_band`
   - Adds customer experience analysis after delay fields are validated.
8. `qry_03b_checkpoint_segment_summary`
   - Aggregates checkpoint transition durations from `qry_03a` after shipment-level staging is validated.
9. `qry_05_corrective_action_priority_segments`
   - Comes last because it combines delay, incident, and shipment-level checkpoint duration evidence.

Building `qry_03a_checkpoint_segment_staging` early reduces late-stage implementation risk while still allowing `qry_03b_checkpoint_segment_summary` to be created after the staging query is validated.

For `qry_03a`, prefer a self-join approach using `Checkpoint_Type` filters for Origin Scan, Gateway Scan, and Destination Scan because it is easier to debug and reuse as a source for `qry_03b` and `qry_05`. Avoid relying on an Access crosstab query unless necessary.

## 6. Planned SQL Artifact Structure

Documentation stays in:

```text
docs/sql-investigation/
```

Actual SQL query files should be stored separately in:

```text
sql/
├── 01-overall-shipment-performance.sql
├── 02-delay-exposure-by-service-region.sql
├── 03a-checkpoint-segment-staging.sql
├── 03b-checkpoint-segment-summary.sql
├── 04-incident-issue-bucket-summary.sql
├── 05-corrective-action-priority-segments.sql
├── 06-csi-by-delay-band.sql
├── 07-cost-per-move-by-service-level.sql
└── 08-pod-timeliness-summary.sql
```

File numbering does not have to equal build order. Filenames follow business question order, while implementation follows dependency and difficulty.

## 7. Validation Approach

Each SQL query should be validated after implementation using:

- Imported Access table row count checks against final cleaned Excel / Power Query outputs
- Row count checks
- Null handling checks
- Rate denominator checks
- Join coverage checks
- Duplicate checks where relevant
- Query grain validation
- Cross-checks between Power BI totals and Access query outputs

Validation examples:

- Before writing analytical queries, confirm imported Access table row counts match the final cleaned Excel / Power Query outputs.
- For `tbl_Checkpoints`, confirm each `Shipment_ID` has exactly three checkpoint records before creating `qry_03a`.
- Confirm checkpoint types are not duplicated per shipment.
- `qry_03a` should return one row per shipment with complete checkpoint timestamps.
- `qry_03b` should aggregate only from `qry_03a`.
- `qry_04` incident share should sum to approximately 100% across issue buckets.
- `qry_05` should aggregate checkpoint evidence from `qry_03a` at `Origin_Facility` x `Destination_Region` grain and should not join directly to `qry_03b`.
- `qry_07` should calculate cost metrics only from captured cost records.
- `qry_08` should preserve negative and high POD lag exceptions instead of correcting them.

## 8. Power BI Handoff

The following Access queries are intended as Power BI-ready outputs:

- `qry_01_overall_shipment_performance` for overview KPI and trend visuals.
- `qry_02_delay_exposure_by_service_region` for delay exposure matrix / heat map visuals.
- `qry_03b_checkpoint_segment_summary` for checkpoint bottleneck visuals.
- `qry_04_incident_issue_bucket_summary` for incident root-cause visuals.
- `qry_05_corrective_action_priority_segments` for corrective action priority table / matrix visuals.
- `qry_06_csi_by_delay_band` for CSI impact visuals.
- `qry_07_cost_per_move_by_service_level` for cost per move visuals.
- `qry_08_pod_timeliness_summary` for data timeliness visuals.

`qry_03a_checkpoint_segment_staging` may be used for drill-through or validation, but it is not necessarily a main dashboard table.

## 9. Scope Boundaries

The following will not be done in Access SQL:

- No final business recommendations yet.
- No weighted priority score in Access.
- No artificial checkpoint SLA/compliance calculation.
- No imputation of missing cost or weight.
- No correction of unusual POD lag values inside SQL.
- No causal claims from simulated relationships.
- Source table schemas are considered frozen after the analytical data model phase; SQL implementation should not change the structure of `tbl_Shipments`, `tbl_SLA_Targets`, `tbl_Checkpoints`, `tbl_Incidents`, or `tbl_CSI_Scores`.

## 10. Next Step

The next step is to implement the planned Microsoft Access SQL queries one by one according to the build order, then validate the outputs before connecting to Power BI.
