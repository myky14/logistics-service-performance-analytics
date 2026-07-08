# Power BI Dashboard Design

## 1. Purpose

This document defines the dashboard design before visuals are built in Power BI. It connects validated SQL outputs to business questions, dashboard pages, and expected decision support.

## 2. Dashboard Objective

The dashboard should help operations stakeholders monitor performance, identify delay hotspots, diagnose bottlenecks and root causes, understand customer impact, review cost visibility, and prioritize corrective actions.

## 3. Dashboard Pages

### Page 1 - Executive Overview

Purpose:

Give a high-level service quality baseline.

Suggested visuals:

- KPI cards: Total Shipments, On-Time Rate, Delay Rate, Avg Transit Delay, Avg CSI Score (0-100), Avg POD Lag
- Trend chart by `Delivery_Month`
- Service level performance summary
- Region performance summary
- Footer note: "Simulated/reframed dataset - see Data Quality notes for limitations."

Source queries:

- `qry_01_overall_shipment_performance`
- `qry_02_delay_exposure_by_service_region`
- `qry_06_csi_by_delay_band`
- `qry_08_pod_timeliness_summary`

Business questions supported:

- What is the overall shipment performance?
- Which service levels or regions have the highest delay exposure?

### Page 2 - Delay & Operational Bottlenecks

Purpose:

Identify where delays concentrate and which checkpoint transition contributes most.

Suggested visuals:

- Heat map: `Service_Level` x `Destination_Region` delay rate
- Segment duration comparison: Origin-to-Gateway vs Gateway-to-Destination
- Bottleneck ranking by service level/region
- Table of high-volume delayed segments

Source queries:

- `qry_02_delay_exposure_by_service_region`
- `qry_03b_checkpoint_segment_summary`
- `qry_03a_checkpoint_segment_staging` only for drill-through or validation

Business questions supported:

- Which service levels or regions have the highest delay exposure?
- Which operational checkpoint contributes most to shipment delay?

### Page 3 - Root Cause & Customer Impact

Purpose:

Connect incident patterns with customer satisfaction impact.

Suggested visuals:

- Ranked bar chart: incident count by issue bucket
- Severity chart: average delay days by issue bucket
- Column chart: Avg CSI Score (0-100) by Delay Band

Source queries:

- `qry_04_incident_issue_bucket_summary`
- `qry_06_csi_by_delay_band`

Business questions supported:

- What are the most common incident issue buckets?
- How does transit delay affect CSI score?

### Page 4 - Corrective Action & Data Quality

Purpose:

Prioritize follow-up segments and monitor data timeliness/cost visibility.

Suggested visuals:

- POD timeliness by service level/region
- Negative / high POD lag exception cards or table
- Cost per move by service level
- Cost capture rate by service level
- Corrective action priority table using `qry_05`
- Notes card for data limitations:
  - blank `Destination_Region` retained and treated as unclassified
  - `Origin_Facility` is a proxy
  - cost and weight partially captured
  - CSI is modeled on a 0-100 scale
  - simulated model, no causal claims

Avg POD Lag should be paired with exception visibility because an average can hide unusual negative or high values.

Source queries:

- `qry_05_corrective_action_priority_segments`
- `qry_07_cost_per_move_by_service_level`
- `qry_08_pod_timeliness_summary`

Business questions supported:

- Which origin groups or regions should be prioritized for corrective action?
- How does cost per move vary by service level?
- How timely is proof of delivery recorded after shipment delivery?

## 4. Suggested Filters / Slicers

- Delivery month
- Service level
- Destination region
- Origin group
- On-time flag
- Issue bucket
- Data timeliness risk category
- Cost capture status / cost captured indicator, if available in model

## 5. Key Measures / Calculations

- Total Shipments
- On-Time Rate
- Delay Rate
- Avg Transit Delay
- Avg CSI Score (0-100)
- Avg POD Lag
- Negative POD Lag Count
- High POD Lag Count
- Optional Negative / High POD Lag Exception Rate
- Cost Capture Rate
- Avg Cost per Move
- Late POD Rate
- Optional priority tier using transparent DAX rule, not weighted Access SQL

## 6. Design Principles

- Business story first, visuals second.
- Every visual should answer a business question.
- Do not overload pages with too many charts.
- Always pair rate metrics with volume.
- Treat low-volume segments cautiously.
- Avoid overclaiming causal relationships.
- Keep limitations visible but concise.

## 7. Navigation

Use page navigation buttons or a simple guided flow:

```text
Overview -> Bottlenecks -> Root Cause & Customer Impact -> Corrective Action & Data Quality
```

## 8. Power BI Build Sequence

1. Connect to Access queries.
2. Validate totals against Access.
3. Create model relationships if needed.
4. Build measures.
5. Build Page 1 overview.
6. Build Page 2 bottleneck view.
7. Build Page 3 root cause/customer view.
8. Build Page 4 corrective action/data quality view.
9. Cross-check dashboard totals.
10. Prepare final screenshots and insights.

## 9. Expected Business Story

The dashboard should tell a clear story:

```text
Overview -> Delay Hotspots -> Operational Bottlenecks -> Root Causes -> Customer Impact -> Cost/Data Timeliness -> Corrective Action
```

## 10. Limitations to Display or Document

- Simulated/reframed dataset.
- Public supply chain data, not real courier company data.
- Synthetic checkpoint, incident, and CSI tables are deterministic.
- Cost and weight partial capture.
- Blank `Destination_Region` values are retained from the current analytical dataset and should be treated as unclassified segments in reporting.
- Blank `Destination_Region` values should be clearly labeled or explained in dashboard notes instead of interpreted as a named region.
- `Origin_Facility` is a proxy.
- CSI is modeled on 0-100 scale.
