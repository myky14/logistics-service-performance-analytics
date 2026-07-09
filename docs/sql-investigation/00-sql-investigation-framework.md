# SQL Investigation Framework

This document defines how SQL is used as an investigation tool for the Service Quality Analytics case study. Each implemented query answers a specific business question, supports a Power BI visual, and provides evidence for a potential business recommendation.

The framework avoids causal claims and keeps final interpretation tied to validated SQL investigation and dashboard evidence.

## Simulation & Interpretation Note

Some operational relationships in this project are intentionally modeled using deterministic business rules to simulate realistic service quality scenarios. SQL investigation is used to validate operational patterns within the analytical model.

The analysis should not claim causal findings from real courier company data. This project remains a simulated portfolio case based on a public dataset reframed for learning purposes.

## 1. What is the overall shipment performance?

### Business Question

What is the overall shipment performance?

### Business Hypothesis

Overall shipment performance is expected to show measurable service quality risk through delayed shipments, SLA gaps, or extended transit time.

### Why This Question Matters

The operations team needs a baseline view of delivery performance before prioritizing specific regions, service levels, checkpoints, or facilities.

### Required Tables

- `tbl_Shipments`
- `tbl_SLA_Targets`

### SQL Investigation Strategy

Profile shipment volume, delivery outcomes, transit time, delay exposure, and SLA compliance at the overall level. Compare actual delivery performance against service commitments where SLA targets are available.

### Key Metrics / Output Fields

- Shipment count
- On-time shipment count
- Delayed shipment count
- On-time rate
- Average transit time
- Average delay days
- SLA compliance flag or rate

### Expected Power BI Visualization

- KPI cards
- Performance trend
- Summary table

### Expected Business Insight

The analysis should show whether service quality risk exists at the overall network level and whether further root-cause analysis is needed.

### Potential Business Recommendation

Use the performance baseline to decide whether improvement efforts should focus on broad network performance or targeted operational segments.

## 2. Which service levels or regions have the highest delay exposure?

### Business Question

Which service levels or regions have the highest delay exposure?

### Business Hypothesis

Delay exposure may be concentrated in specific service levels or regions.

### Why This Question Matters

Service quality issues are rarely distributed evenly. Identifying high-risk service levels or regions helps the operations team prioritize investigation and action.

### Required Tables

- `tbl_Shipments`
- `tbl_SLA_Targets`

### SQL Investigation Strategy

Group shipment performance by `Service_Level` and `Destination_Region`. Compare shipment volume, delay rate, delay days, and SLA compliance across segments.

### Key Metrics / Output Fields

- Service level
- Destination region
- Shipment count
- Delayed shipment count
- Delay rate
- Average delay days
- SLA compliance rate

### Expected Power BI Visualization

- Matrix by service level and region
- Clustered bar chart
- Heat map

### Expected Business Insight

The analysis should identify whether delay exposure is concentrated in specific service commitments or destination markets.

### Potential Business Recommendation

Prioritize corrective action for the service level and region combinations with the highest delay exposure and meaningful shipment volume.

## 3. Which operational checkpoint contributes most to shipment delay?

### Business Question

Which operational checkpoint contributes most to shipment delay?

### Business Hypothesis

Gateway-to-destination movement may contribute the largest time gap for delayed shipments.

### Why This Question Matters

The operations team needs to know where shipment movement slows down across Origin Scan, Gateway Scan, and Destination Scan so process improvement can focus on the most relevant handoff.

### Required Tables

- `tbl_Checkpoints`
- `tbl_Shipments`

### SQL Investigation Strategy

Compare elapsed time between checkpoint events for each shipment:

- Origin Scan to Gateway Scan
- Gateway Scan to Destination Scan

Then summarize average segment duration by service level, destination region, and on-time status.

### Key Metrics / Output Fields

- Shipment ID
- Service level
- Destination region
- On-time flag
- Origin scan timestamp
- Gateway scan timestamp
- Destination scan timestamp
- Origin-to-gateway duration
- Gateway-to-destination duration
- Longest checkpoint segment
- Average segment duration

### Expected Power BI Visualization

- Segment duration bar chart
- Bottleneck ranking
- Matrix by service level and destination region

### Expected Business Insight

The analysis should show which checkpoint transition contributes most to shipment delay risk within the simulated operational model.

### Potential Business Recommendation

Focus process review, handoff controls, staffing review, or escalation routines on the checkpoint transition with the longest average duration and strongest link to delayed shipments.

## 4. What are the most common incident issue buckets?

### Business Question

What are the most common incident issue buckets?

### Business Hypothesis

Gateway congestion and customs delay may be the dominant incident drivers.

### Why This Question Matters

Incident classification helps separate delay symptoms from root causes. This allows the service quality team to address recurring operational issues instead of treating every delay as isolated.

### Required Tables

- `tbl_Incidents`
- `tbl_Shipments`

### SQL Investigation Strategy

Count incidents by issue bucket and compare incident categories within delayed shipments. Because `tbl_Incidents` is only created for delayed shipments, incident share should be interpreted as share among incident records or delayed shipments, not automatically as share of all shipments.

### Key Metrics / Output Fields

- Issue bucket
- Incident count
- Share of incidents among delayed shipments
- Average delay days by issue bucket
- Delayed shipment count by issue bucket

### Expected Power BI Visualization

- Ranked bar chart
- Incident category table
- Root-cause summary visual

### Expected Business Insight

The analysis should identify which incident categories are most frequently associated with service failures.

### Potential Business Recommendation

Prioritize root-cause action plans for recurring issue buckets with the strongest connection to delay exposure.

## 5. Which origin groups or regions should be prioritized for corrective action?

### Business Question

Which origin groups or regions should be prioritized for corrective action?

### Business Hypothesis

Certain origin groups or regions may require prioritized corrective action.

### Why This Question Matters

Corrective action should focus on operational areas with both service quality risk and enough shipment activity to justify intervention.

### Required Tables

- `tbl_Shipments`
- `tbl_Incidents`
- `tbl_Checkpoints`

### SQL Investigation Strategy

Compare delay rates, incident patterns, and checkpoint transition performance by origin group and destination region. Look for segments that combine high delay exposure with operational exception patterns.

`Origin_Facility` is used as a proxy field derived from the source dataset vendor field. It should be interpreted as an analytical grouping dimension, not as a confirmed real logistics facility.

### Key Metrics / Output Fields

- Origin group
- Destination region
- Shipment count
- Delay rate
- Average delay days
- Incident count
- Checkpoint duration patterns

### Expected Power BI Visualization

- Priority matrix
- Heat map
- Facility or region ranking

### Expected Business Insight

The analysis should identify the origin groups or regions that deserve management attention before broader network changes are considered.

### Potential Business Recommendation

Review high-risk origin groups or regions, supported by delay, incident, and checkpoint evidence.

## 6. How does transit delay affect CSI score?

### Business Question

How does transit delay affect CSI score?

### Business Hypothesis

Longer transit delays may reduce CSI scores.

### Why This Question Matters

Service quality should be evaluated not only by operational metrics, but also by its effect on customer experience.

### Required Tables

- `tbl_Shipments`
- `tbl_CSI_Scores`

### SQL Investigation Strategy

Compare CSI scores across transit delay bands. Review whether customer satisfaction decreases as delivery delay increases.

### Key Metrics / Output Fields

- Shipment ID
- Transit delay days
- Delay band
- CSI score
- Average CSI score by delay band

### Expected Power BI Visualization

- Scatter plot
- Line chart by delay band
- CSI summary cards

### Expected Business Insight

The analysis should show whether operational delay appears to be associated with lower customer satisfaction in the simulated model.

### Potential Business Recommendation

Use delay impact on CSI to prioritize customer recovery actions and escalation rules for shipments with higher customer experience risk.

## 7. How does cost per move vary by service level?

### Business Question

How does cost per move vary by service level?

### Business Hypothesis

Cost per move may vary significantly by service level.

### Why This Question Matters

Cost visibility helps the operations team understand whether service mix and performance outcomes should be reviewed together.

### Required Tables

- `tbl_Shipments`

### SQL Investigation Strategy

Compare shipment cost per move by service level. Preserve null cost records appropriately so missing cost data is not treated as zero.

### Key Metrics / Output Fields

- Service level
- Shipment count
- Cost-captured shipment count
- Average cost per move
- Median cost per move, if supported
- Cost capture flag

### Expected Power BI Visualization

- Column chart by service level
- Cost summary table
- Service mix comparison

### Expected Business Insight

The analysis should show whether cost per move differs by service level and whether cost data coverage affects interpretation.

### Potential Business Recommendation

Use cost-per-move patterns to support service mix review and cost-performance discussions, without claiming optimization until analysis is complete.

## 8. How timely is proof of delivery recorded after shipment delivery?

### Business Question

How timely is proof of delivery recorded after shipment delivery?

### Business Hypothesis

Some shipments may have delayed POD confirmation, creating data timeliness risk even when the shipment has already been physically delivered.

### Why This Question Matters

Data timeliness affects customer visibility, escalation handling, reporting accuracy, and the ability of customer-facing teams to communicate proactively.

### Required Tables

- `tbl_Shipments`

### SQL Investigation Strategy

Analyze `POD_Lag_Days`, calculated as `POD_Confirmation_Date - Actual_Delivery_Date`. Summarize average POD lag, late POD count, late POD rate, and unusual negative or high POD lag values by service level and destination region.

### Key Metrics / Output Fields

- Shipment ID
- Service level
- Destination region
- Actual delivery date
- POD confirmation date
- POD lag days
- Late POD flag
- Average POD lag
- Late POD rate
- Data timeliness risk category

### Expected Power BI Visualization

- KPI card for average POD lag
- Bar chart by service level or region
- Distribution chart for POD lag days
- Exception table for unusual POD lag values

### Expected Business Insight

The analysis should show whether data recording delays create visibility risk after delivery and whether the issue is concentrated in specific service levels or regions.

### Potential Business Recommendation

Improve POD recording discipline, monitor late POD exceptions, and create follow-up rules for shipments where confirmation is not recorded within the expected time window.
