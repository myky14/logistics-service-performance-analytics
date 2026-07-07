# Column Selection

## Purpose

This document explains how each original column was reviewed and selected for the Service Quality Analytics project.

The source dataset comes from a health-commodity supply chain context. For this project, selected fields are reframed into an express logistics / courier service quality scenario for learning and portfolio purposes.

## Column Review

| Original Column | Business Meaning | Courier Reframe | Keep? | Final Field Name | Reason / Dashboard Impact |
|---|---|---|---|---|---|
| ID | Unique delivery/shipment record ID | Shipment identifier | Yes | Shipment_ID | Primary key for all tables and relationships |
| Project Code | Procurement/project reference | Not relevant | No | - | Procurement context, not needed for service quality analytics |
| PQ # | Purchase quotation reference | Not relevant | No | - | Procurement workflow field, not useful for dashboard |
| PO / SO # | Purchase/sales order reference | Not relevant | No | - | Order admin detail, not needed for KPI analysis |
| ASN/DN # | Shipment notice/delivery note | Not used | No | - | Could identify documents, but does not support selected KPIs |
| Country | Destination country | Destination country | Yes | Destination_Country | Needed for region grouping, SLA, route performance |
| Managed By | Program owner | Not relevant | No | - | Internal source-program field, not courier service quality |
| Fulfill Via | Fulfillment route | Routing type | Yes | Routing_Type | Used to compare Direct Line-haul vs Via Regional Hub |
| Vendor INCO Term | Shipping responsibility term | Shipping term | Yes | Shipping_Term | Useful context for cost/routing analysis |
| Shipment Mode | Transport mode | Service level | Yes | Service_Level | Core dimension for SLA, transit time, incidents, dashboard slicing |
| PQ First Sent to Client Date | Procurement quotation date | Not used | No | - | Mixed with placeholders; not reliable for KPI calculation |
| PO Sent to Vendor Date | Purchase order date | Not used | No | - | High placeholder rate; procurement-focused, not core service KPI |
| Scheduled Delivery Date | Planned delivery date | Committed delivery date | Yes | Committed_Delivery_Date | Used to calculate on-time performance |
| Delivered to Client Date | Actual delivery date | Actual delivery date | Yes | Actual_Delivery_Date | Used to calculate delay / transit performance |
| Delivery Recorded Date | Delivery confirmation date | POD confirmation date | Yes | POD_Confirmation_Date | Used for data timeliness / POD lag |
| Product Group | Product category | Cargo category | Yes | Cargo_Category | Used to simplify shipment/cargo segmentation |
| Sub Classification | Product sub-category | Not used | No | - | Too specific to health dataset; adds noise |
| Vendor | Supplier/vendor | Origin facility | Yes | Origin_Facility | Proxy for origin facility / possible root-cause grouping |
| Item Description | Detailed commodity name | Not used | No | - | Too domain-specific and high-cardinality |
| Molecule/Test Type | Medical product attribute | Not used | No | - | Health-specific, not courier-focused |
| Brand | Product brand | Not used | No | - | Health/product-specific, not needed for service analytics |
| Dosage | Product dosage | Not used | No | - | Health-specific and has missing values |
| Dosage Form | Product form | Not used | No | - | Health-specific, not relevant to courier KPIs |
| Unit of Measure (Per Pack) | Product unit measure | Not used | No | - | Product packaging detail, not central to service quality |
| Line Item Quantity | Quantity shipped | Quantity | Yes | Quantity | Useful for shipment volume context |
| Line Item Value | Shipment/item value | Declared value | Yes | Declared_Value_USD | Useful for value-at-risk / shipment profile |
| Pack Price | Product pack price | Not used | No | - | Product pricing detail, not needed for service KPI |
| Unit Price | Product unit price | Not used | No | - | Product pricing detail, not needed for service KPI |
| Manufacturing Site | Manufacturer location | Not used | No | - | Health manufacturing context; not part of courier model |
| First Line Designation | Health program indicator | Not used | No | - | Health-specific field, not useful for logistics dashboard |
| Weight (Kilograms) | Shipment weight | Shipment weight | Yes | Weight_KG | Needed for shipment profile and possible cost analysis |
| Freight Cost (USD) | Freight cost | Shipment cost | Yes | Shipment_Cost_USD | Needed for Cost/Move KPI; only valid numeric values used |
| Line Item Insurance (USD) | Insurance amount | Insurance amount | Yes | Insurance_USD | Optional cost/value context; keep null if missing |
| Load_Date | Upload/system metadata | Not used | No | - | Nearly empty technical column |
| Source_System | Upload/system metadata | Not used | No | - | Nearly empty technical column |

## Final Shipments Table

The final `Shipments` table will keep the following fields:

1. Shipment_ID
2. Destination_Country
3. Destination_Region
4. Routing_Type
5. Shipping_Term
6. Service_Level
7. Origin_Facility
8. Cargo_Category
9. Committed_Delivery_Date
10. Actual_Delivery_Date
11. POD_Confirmation_Date
12. Quantity
13. Declared_Value_USD
14. Weight_KG
15. Is_Weight_Captured
16. Shipment_Cost_USD
17. Is_Cost_Captured
18. Insurance_USD
19. Transit_Time_Actual_Days
20. On_Time_Flag
21. POD_Lag_Days

## Notes

- Raw data must not be edited directly.
- Removed columns are excluded because they are procurement-specific, health-product-specific, or not relevant to service quality analytics.
- Weight and freight cost contain text placeholders, so numeric values will be converted while invalid placeholders remain null.
- This project does not claim to use real DHL or courier operational data.