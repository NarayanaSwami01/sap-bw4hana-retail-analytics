# sap-bw4hana-retail-analytics
Sales and Inventory Project - SAP BW4/HANA
# SAP BW/4HANA Retail Sales & Inventory Analytics

A self-directed SAP BW/4HANA project building a complete data warehouse solution for retail sales and inventory reporting — from flat-file source data through InfoObjects, DataSources, DataStore Objects (Advanced), Transformations, a CompositeProvider, BEx queries, and a process chain.

Built using Eclipse ADT and SAP GUI on a company development system.

---

## Project Overview

| | |
|---|---|
| **Domain** | Retail sales and inventory management |
| **Source Data** | 5 flat files (Customer, Product, Store masters; Sales and Inventory transactions) |
| **Platform** | SAP BW/4HANA (Eclipse ADT + SAP GUI) |
| **Scope** | InfoObjects → DataSources → ADSOs → Transformations → CompositeProvider → BEx Queries → Process Chain |
| **Status** | Phases 1–4 complete; Phase 5 (documentation) in progress |

---

## Architecture

![Data Flow Diagram](full_data_flow_diagram.png)

**Pipeline:** 5 CSV source files → 5 DataSources → 2 Standard ADSOs (`Y_SALES`, `Y_INVNTRY`) → 1 CompositeProvider (`Y_CPSALESINV`, Union) → 3 BEx queries → Analysis for Office / RSRT reporting.

---

## Object Summary

| Object Type | Count |
|---|---|
| Characteristics | 22 |
| Key Figures | 14 |
| DataSources | 5 |
| DataStore Objects (Advanced) | 2 |
| CompositeProvider | 1 |
| BEx Queries | 3 |
| Process Chain | 1 |
| **Total** | **48** |

Full technical details — every object's technical name, description, and role — are in [`Day21_Documentation.docx`](./Day21_Documentation.docx).

---

## Key Reports

### Sales Performance (`Y_CPSALESINV_Q0001`)
Transaction-level sales analysis with calculated Profit, Profit Margin, Total Cost, and Discount Percentage.

### Inventory Status (`Y_CPSALESINV_Q0002`)
Stock movement tracking with a 3-tier exception (Red/Yellow/Green) flagging products approaching or below their reorder level.

### Executive Dashboard (`Y_CPSALESINV_Q0003`)
Region/Store rollup of headline KPIs with a date-range selection variable.

---

## Notable Engineering Decisions

This project intentionally documents trade-offs rather than hiding them. Full detail in the Business Rules section of the Day 21 documentation; summary below.

| Decision | Why |
|---|---|
| **Profit & Profit Margin calculated at query level (Calculated Key Figures), not in the Transformation** | An AMDP/HANA SQLScript routine was attempted first but blocked by a credentials/authorization issue outside project control. Query-level CKFs avoid ABAP entirely and are a standard BW pattern for this kind of cross-entity calculation. |
| **High Value Order Flag not implemented** | Same credentials blocker as above (required an ABAP routine). |
| **Gross Revenue calculated, not sourced** | The actual source data doesn't include a Gross Revenue column — it's derived at load time from Quantity × Unit Price. |
| **Process chain built and activated, but not executed end-to-end** | All DataSources use a Local Workstation adapter, which requires an interactive session — incompatible with unattended background job execution. Fixing this requires application-server file upload access not available on this system. |

### A bug worth mentioning

During Executive Dashboard testing, Profit for one store came back as **−167,675,543** against ~1.2 million in revenue. Root cause: the Profit formula (`Quantity × Unit Cost`) was correct at transaction grain but broke once rolled up to Store level — BW was computing `SUM(Quantity) × SUM(Unit Cost)` across many different products instead of summing each transaction's already-correct profit. Fixed by setting **Exception Aggregation = Summation** with **Reference Characteristic = Order ID**, forcing the calculation to happen at the finest grain first. This is documented as a general modeling principle: any CKF multiplying a transactional quantity by a master-data rate needs this setting explicitly, since BW's default is not safe for that pattern.

---

## Testing

12 formal test cases covering record count reconciliation, calculated field accuracy (at both transaction and aggregated grain), load repeatability, master data quality, exception logic, and CompositeProvider integrity. 11 passed outright; 1 partial (process chain execution — see above). Full test log in [`Phase4_Test_Case_Log.docx`](./Phase4_Test_Case_Log.docx).

---

## Repository Structure

```
├── README.md
├── full_data_flow_diagram.png
├── Day21_Documentation.docx          # Business Rules & Object List
├── Phase4_Test_Case_Log.docx         # 12 formal test cases
├── source_files/
│   ├── Customer_Master.csv
│   ├── Product_Master.csv
│   ├── Store_Master.csv
│   ├── Sales_Transaction.csv
│   └── Inventory_Transaction.csv
└── screenshots/
    ├── infoobjects/
    ├── datasources/
    ├── adso/
    ├── compositeprovider/
    ├── queries/
    └── process_chain/
```

---

## Skills Demonstrated

SAP BW/4HANA data modeling (InfoObjects, master data attribute design), Eclipse ADT and SAP GUI development, DataSource and ETL configuration, DataStore Object (Advanced) design, Transformation logic including Formula rules and attempted AMDP/SQLScript routines, CompositeProvider union modeling, BEx Query Designer (Calculated Key Figures, Exceptions, Variables), process chain design, root-cause debugging of a real aggregation defect, and structured technical documentation.

---

## Author

Built as a self-directed learning project. Not affiliated with or representing any employer's production systems or data.
