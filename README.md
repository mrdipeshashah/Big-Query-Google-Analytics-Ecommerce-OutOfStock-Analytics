# GA4 & BigQuery Out-of-Stock (OOS) Analytics Dashboard — Looker Studio Master Documentation

This document serves as the master technical reference for the **Looker Studio Out-of-Stock (OOS) Analytics Dashboard**. It details the underlying methodology, business logic, single-source channel groupings, and all calculated field definitions across both **Page 1: Session-Level OOS Impact & Acquisition Attribution** and **Page 2: PDP-Level Demand & SKU Exposure Analysis**.

---

## 1. Methodology & Strategic Context

### 1.1 Objective & Executive Value
Standard e-commerce reporting tracks macro conversion rates and total site traffic but routinely obscures the hidden revenue leakages caused by inventory stockouts. When high-intent shoppers reach an out-of-stock (OOS) Product Detail Page (PDP), standard conversion metrics fail to isolate whether the user abandoned the session entirely or pivoted to purchase an alternative item.

This dashboard quantifies inventory friction in two core dimensions:
1. **Session-Level Funnel Impact & Revenue Leakage (Page 1):** Measures macro user behavior, identifying how out-of-stock PDP exposure impacts session outcomes (Abandoned vs. Recovered) and mapping these friction points directly back to paid/organic acquisition channels to prevent wasted ad spend.
2. **PDP-Level Demand & SKU Inventory Friction (Page 2):** Isolates shopper demand intensity at the product and category level, measuring view rates, repeat browsing friction (`Views per Session Index`), and inventory exposure across stock states (`instock`, `outofstock`, `onbackorder`).

---

### 1.2 Data Model Architecture & Granularity
The dashboard is powered by two distinct underlying BigQuery aggregated datasets derived from raw GA4 event data:

```
[ GA4 Raw Event Stream ]
           │
           ├──► 1. Session-Level Aggregation (Page 1 Data Source)
           │       • Granularity: Session + Date + Channel + Campaign + SKU Journey Outcome
           │       • Primary Metrics: total_sessions, pdp_sessions, oos_pdp_sessions, total_revenue
           │
           └──► 2. Item/PDP-Level Aggregation (Page 2 Data Source)
                   • Granularity: Item ID + Item Name + Item Category + Date + Stock Status
                   • Primary Metrics: total_pdp_views, pdp_sessions, stock_status
```

---

### 1.3 Key Behavioral Definitions & Journey Taxonomy
To eliminate ambiguity across cross-functional teams (Merchandising, Performance Marketing, Operations), sessions and product views are classified using strict logical rules:

* **Session Outcome Categories (`session_type`):**
  * **Pure In-Stock PDP Session:** Sessions that viewed at least one PDP, where all viewed items were fully in-stock.
  * **Non-PDP Session:** Sessions that browsed the store (home page, category pages) but never reached a single PDP.
  * **OOS Abandoned (No Purchase):** High-intent sessions exposed to one or more out-of-stock PDPs that dropped off without making a purchase. Represents direct revenue leakage.
  * **OOS Recovered (Purchased):** High-intent sessions exposed to an out-of-stock PDP that successfully cross-sold or converted on an alternative in-stock product. Represents recovered opportunity.

* **Inventory Friction Indices:**
  * **OOS Exposure Rate (%):** The probability of a product-intent session encountering an out-of-stock page (`SUM(oos_pdp_sessions) / SUM(pdp_sessions)`).
  * **OOS Recovery Rate (%):** The effectiveness of site cross-selling and alternative recommendations when stockouts occur (`OOS Recovered Sessions / Total OOS PDP Sessions`).
  * **Views per Session Index:** Indicates repeat view behavior on product pages (`SUM(total_pdp_views) / SUM(pdp_sessions)`). High values on out-of-stock items signal elevated customer frustration and unfulfilled demand.

---

## 2. Master Calculated Fields Reference Table

Below is the unified reference table for all custom calculated fields used across both pages of the dashboard.

| Page No | Metric / Field Name | Looker Studio Formula | Data Type | Description | 
| ----- | ----- | ----- | ----- | ----- | 
| **Page 1** | **PDP Session Share (%)** | `SUM(pdp_sessions) / SUM(total_sessions)` | Percent | Percentage of total site sessions that reached at least one PDP page. | 
| **Page 1** | **OOS PDP Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Percentage of PDP sessions that encountered an out-of-stock product page. | 
| **Page 1** | **Overall Site CVR (%)** | `SUM(purchasing_sessions) / SUM(total_sessions)` | Percent | Macro conversion rate across all site visits. | 
| **Page 1** | **PDP CVR (%)** | `SUM(purchasing_sessions) / SUM(pdp_sessions)` | Percent | Conversion rate specifically for sessions that viewed a product page. | 
| **Page 1** | **AOV (Average Order Value)** | `SUM(total_revenue) / SUM(total_purchases)` | Currency | Average revenue generated per completed purchase transaction. | 
| **Page 1** | **OOS Recovery Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Proportion of OOS sessions that successfully cross-sold or converted on an alternative item. | 
| **Page 1** | **OOS Abandonment Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Proportion of OOS sessions that dropped off completely without purchasing. | 
| **Page 1** | **OOS Recovered Revenue** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_revenue ELSE 0 END)` | Currency | Total revenue saved/recovered from sessions exposed to an out-of-stock PDP. | 
| **Page 1** | **Channel Grouping** | *(See Section 3 Regex CASE Statement)* | Text | Categorizes traffic acquisition sources into high-level default channels. | 
| **Page 1** | **Channel OOS Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Channel-level metric measuring proportion of incoming traffic exposed to stockouts. | 
| **Page 1** | **Channel OOS Abandonment Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Rate at which out-of-stock traffic from a specific channel drops off without buying. | 
| **Page 1** | **Channel OOS Recovery Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Rate at which out-of-stock traffic from a specific channel cross-sells to alternative items. | 
| **Page 2** | **OOS PDP Views** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total product detail page (PDP) views spent on out-of-stock SKUs. | 
| **Page 2** | **In-Stock PDP Views** | `SUM(CASE WHEN stock_status = 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total PDP views spent on available in-stock SKUs. | 
| **Page 2** | **OOS Share of Views (%)** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END) / SUM(total_pdp_views)` | Percent | Percentage of total PDP page views exposed to stockouts. | 
| **Page 2** | **OOS PDP Sessions** | `SUM(CASE WHEN stock_status != 'instock' THEN pdp_sessions ELSE 0 END)` | Integer | Distinct sessions exposed to out-of-stock SKUs. | 
| **Page 2** | **OOS Session Share (%)** | `SUM(CASE WHEN stock_status != 'instock' THEN pdp_sessions ELSE 0 END) / SUM(pdp_sessions)` | Percent | Proportion of high-intent PDP sessions encountering unavailable inventory. | 
| **Page 2** | **Views per Session Index** | `SUM(total_pdp_views) / SUM(pdp_sessions)` | Decimal | Average PDP page views per session (measures shopper browsing intensity and repeated friction). | 

---

## 3. Default Channel Grouping Field Logic

To classify traffic channels cleanly in Looker Studio without SQL pre-processing, create a custom field named **`Channel Grouping`** using the following formula:

```text
CASE
  WHEN REGEXP_MATCH(source_medium, '(?i)\\(direct\\) / \\(none\\)') THEN 'Direct'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / (cpc|ppc|paidsearch)') THEN 'Paid Search'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / (paidsocial|paid_social)') THEN 'Paid Social'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / (paidvideo|paid_video)') THEN 'Paid Video'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / (organicsocial|social)') THEN 'Organic Social'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / organic') THEN 'Organic Search'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / (email|newsletter)') THEN 'Email'
  WHEN REGEXP_MATCH(source_medium, '(?i).* / referral') THEN 'Referral'
  ELSE 'Unassigned / Other'
END
```

---

## 4. Page 1 Architecture: Session-Level OOS Impact & Acquisition

### Scorecard Banner (Overall Performance)
* **Total Sessions:** `SUM(total_sessions)`
* **PDP Sessions:** `SUM(pdp_sessions)` *(Sub-metric: `PDP Session Share (%)`)*
* **OOS PDP Sessions:** `SUM(oos_pdp_sessions)` *(Sub-metric: `OOS PDP Exposure Rate (%)`)*
* **Total Revenue:** `SUM(total_revenue)`
* **OOS Recovery Rate:** `OOS Recovery Rate (%)`

### Charts
1. **SESSION JOURNEY DISTRIBUTION & STOCKOUT OUTCOMES**
   * **Type:** Horizontal 100% Stacked Bar Chart
   * **Dimension:** `event_date` (Granularity: `Year Month`)
   * **Breakdown Dimension:** `session_type`
   * **Metric:** `SUM(total_sessions)`
   * **Style Note:** Y-axis labels set to Hidden to eliminate margin overflow.

2. **INVENTORY FRICTION & OOS EXPOSURE TREND**
   * **Type:** Dual-Line Time Series Chart
   * **Dimension:** `event_date`
   * **Metric 1 (Left Axis):** `OOS PDP Exposure Rate (%)`
   * **Metric 2 (Right Axis):** `OOS Recovery Rate (%)`

3. **OOS VOLUME v REVENUE RECOVERY TREND**
   * **Type:** Dual-Axis Combo Chart (Column + Line)
   * **Dimension:** `event_date`
   * **Metric 1 (Left Axis / Columns):** `SUM(oos_pdp_sessions)`
   * **Metric 2 (Right Axis / Line):** `OOS Recovered Revenue`

### Data Tables
1. **PRODUCTS IMPACTED BY STOCKOUTS: RECOVERY & LEAKAGE ANALYSIS**
   * **Dimensions:** `item_name`, `session_type`
   * **Metrics:** `SUM(total_sessions)`, `SUM(purchasing_sessions)`, `PDP CVR (%)`, `SUM(total_revenue)`
   * **Sort:** `SUM(total_sessions)` DESC

2. **ACQUISITION CHANNEL OOS FRICTION & LEAKAGE ANALYSIS**
   * **Dimensions:** `Channel Grouping` (Primary), `source_medium` (Drill-down), `campaign` (Drill-down)
   * **Metrics:**
     1. `SUM(pdp_sessions)`
     2. `SUM(oos_pdp_sessions)`
     3. `Channel OOS Exposure Rate (%)`
     4. `Channel OOS Abandonment Rate (%)`
     5. `Channel OOS Recovery Rate (%)`
     6. `OOS Recovered Revenue`
   * **Sort:** `SUM(pdp_sessions)` DESC

---

## 5. Page 2 Architecture: PDP Demand & Inventory Exposure

### Scorecard Banner (Overall Performance)
* **Total PDP Views:** `SUM(total_pdp_views)`
* **OOS PDP Views:** `OOS PDP Views` *(Sub-metric: `OOS Share of Views (%)`)*
* **OOS PDP Sessions:** `OOS PDP Sessions` *(Sub-metric: `OOS Session Share (%)`)*
* **PDP Views per Session:** `Views per Session Index`

### Visual Charts
1. **DEMAND BREAKDOWN BY STOCK STATUS SPLIT BY ITEM CATEGORY**
   * **Type:** Horizontal 100% Stacked Bar Chart
   * **Dimension:** `item_category`
   * **Breakdown Dimension:** `stock_status` (`instock`, `outofstock`, `onbackorder`)
   * **Metric:** `SUM(total_pdp_views)`

2. **DAILY OOS PDP DEMAND TREND**
   * **Type:** Time Series Line Chart
   * **Dimension:** `event_date`
   * **Metric 1 (Left Axis):** `Views per Session Index`
   * **Metric 2 (Right Axis):** `OOS PDP Views`

3. **OOS PDP VIEWS v SESSION INDEX**
   * **Type:** Time Series Line Chart
   * **Dimension:** `event_date`
   * **Metric 1 (Left Axis):** `OOS PDP Sessions`
   * **Metric 2 (Right Axis):** `Views per Session Index`

4. **CATEGORY STOCKOUT EXPOSURE & HIGH INTENT TRAFFIC IMPACT**
   * **Type:** Dual-Axis Combo Chart (Stacked Column + Line)
   * **Dimension:** `item_category`
   * **Metric 1 (Left Axis / Column):** `OOS PDP Sessions`
   * **Metric 2 (Left Axis / Column):** `In-Stock PDP Views`
   * **Metric 3 (Right Axis / Line):** `OOS Share of Views (%)`

### Data Table
* **SKU LEVEL DEMAND & INVENTORY EXPOSURE RANKING**
  * **Dimensions:** `item_name`
  * **Metrics:**
    1. `SUM(total_pdp_views)` *(PDP Views)*
    2. `SUM(pdp_sessions)` *(PDP Sessions)*
    3. `Views per Session Index`
    4. `OOS PDP Views`
    5. `OOS Share of Views (%)`
  * **Sort:** `SUM(total_pdp_views)` DESC
