# Looker Studio Calculated Fields - Page 1: Session-Level OOS Impact

| Metric Name | Looker Studio Formula | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **PDP Session Share (%)** | `SUM(pdp_sessions) / SUM(total_sessions)` | Percent | Percentage of total site sessions that reached at least one PDP page. |
| **OOS PDP Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Percentage of PDP sessions that encountered an out-of-stock product page. |
| **Overall Site CVR (%)** | `SUM(purchasing_sessions) / SUM(total_sessions)` | Percent | Macro conversion rate across all site visits. |
| **PDP CVR (%)** | `SUM(purchasing_sessions) / SUM(pdp_sessions)` | Percent | Conversion rate specifically for sessions that viewed a product page. |
| **AOV (Average Order Value)** | `SUM(total_revenue) / SUM(total_purchases)` | Currency | Average revenue generated per completed purchase transaction. |
| **OOS Recovery Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Proportion of OOS sessions that successfully cross-sold or converted on an alternative item. |
| **OOS Abandonment Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Proportion of OOS sessions that dropped off completely without purchasing. |
| **OOS Recovered Revenue** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_revenue ELSE 0 END)` | Currency | Total revenue saved/recovered from sessions exposed to an out-of-stock PDP. |


# Page 2: SKU-Level PDP Demand (2.0) — Looker Studio Calculated Fields

This document details the custom calculated fields and visual architecture for **Page 2: SKU-Level PDP Demand** of the Out-of-Stock (OOS) Analytics Dashboard.

---

## Data Source & Schema Context
* **Source Table:** `sku_level_pdp_demand`
* **Granularity:** Daily SKU-level event aggregations (`event_date`, `item_id`, `item_name`, `stock_status`, etc.)

---

## 1. Calculated Fields Reference Table

| Calculated Field Name | Looker Studio Formula | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **OOS PDP Views** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total product detail page (PDP) views spent on out-of-stock SKUs. |
| **In-Stock PDP Views** | `SUM(CASE WHEN stock_status = 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total PDP views spent on available in-stock SKUs. |
| **OOS Share of Views (%)** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END) / SUM(total_pdp_views)` | Percent | Percentage of total PDP page views exposed to stockouts. |
| **OOS PDP Sessions** | `SUM(CASE WHEN stock_status != 'instock' THEN pdp_sessions ELSE 0 END)` | Integer | Distinct sessions exposed to out-of-stock SKUs. |
| **OOS Session Share (%)** | `SUM(CASE WHEN stock_status != 'instock' THEN pdp_sessions ELSE 0 END) / SUM(pdp_sessions)` | Percent | Proportion of high-intent PDP sessions encountering unavailable inventory. |
| **Views per Session Index** | `SUM(total_pdp_views) / SUM(pdp_sessions)` | Decimal (2 dec) | Average PDP page views per session (measures shopper browsing intensity and repeated friction). |

---

## 2. Dashboard Component Architecture

### Scorecards Section (Top Banner)
* **Total PDP Views:** `SUM(total_pdp_views)`
* **OOS PDP Views:** `OOS PDP Views` *(Sub-metric: `OOS Share of Views (%)`)*
* **PDP Sessions Exposed to OOS:** `OOS PDP Sessions` *(Sub-metric: `OOS Session Share (%)`)*
* **PDP Views Per Session:** `Views per Session Index`

### Visual Charts
1. **Demand Breakdown by Stock Status**
   * **Type:** Horizontal 100% Stacked Bar Chart
   * **Dimension:** `item_category` (or `item_name`)
   * **Breakdown Dimension:** `stock_status`
   * **Metric:** `SUM(total_pdp_views)`

2. **Daily OOS PDP Demand Trend**
   * **Type:** Dual-Axis Time Series Combo Chart
   * **Dimension:** `event_date`
   * **Left Y-Axis (Bars):** `OOS PDP Views`
   * **Right Y-Axis (Line):** `Views per Session Index`

### Main Diagnostic Table
* **Title:** `SKU Level Demand & Inventory Exposure Ranking`
* **Dimensions:** `item_id`, `item_name`, `item_category`
* **Metrics:** 
  1. `SUM(total_pdp_views)`
  2. `SUM(pdp_sessions)`
  3. `Views per Session Index`
  4. `OOS PDP Views`
  5. `OOS Share of Views (%)`
* **Default Sort:** `SUM(total_pdp_views)` DESC

# Page 1: Session-Level OOS Impact & Channel Attribution — Looker Studio Calculated Fields

This document details all custom calculated fields, BigQuery schemas, and visual components for **Page 1: Session-Level OOS Impact & Channel Attribution** of the Out-of-Stock (OOS) Analytics Dashboard.

---

## 1. Calculated Fields Reference Table

| Metric / Field Name | Looker Studio Formula | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **PDP Session Share (%)** | `SUM(pdp_sessions) / SUM(total_sessions)` | Percent | Percentage of total site sessions that reached at least one PDP page. |
| **OOS PDP Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Percentage of PDP sessions that encountered an out-of-stock product page. |
| **Overall Site CVR (%)** | `SUM(purchasing_sessions) / SUM(total_sessions)` | Percent | Macro conversion rate across all site visits. |
| **PDP CVR (%)** | `SUM(purchasing_sessions) / SUM(pdp_sessions)` | Percent | Conversion rate specifically for sessions that viewed a product page. |
| **AOV (Average Order Value)** | `SUM(total_revenue) / SUM(total_purchases)` | Currency | Average revenue generated per completed purchase transaction. |
| **OOS Recovery Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Proportion of OOS sessions that successfully cross-sold or converted on an alternative item. |
| **OOS Abandonment Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Proportion of OOS sessions that dropped off completely without purchasing. |
| **OOS Recovered Revenue** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_revenue ELSE 0 END)` | Currency | Total revenue saved/recovered from sessions exposed to an out-of-stock PDP. |
| **Channel OOS Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Channel-level metric measuring proportion of incoming traffic exposed to stockouts. |
| **Channel OOS Abandonment Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Rate at which out-of-stock traffic from a specific acquisition channel drops off without buying. |
| **Channel OOS Recovery Rate (%)** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions)` | Percent | Rate at which out-of-stock traffic from a specific acquisition channel cross-sells to alternative items. |

---

## 2. Dashboard Component Architecture

### Scorecards Section (Top Banner)
* **Total Sessions:** `SUM(total_sessions)`
* **PDP Sessions:** `SUM(pdp_sessions)` *(Sub-metric: `PDP Session Share (%)`)*
* **OOS PDP Sessions:** `SUM(oos_pdp_sessions)` *(Sub-metric: `OOS PDP Exposure Rate (%)`)*
* **Total Revenue:** `SUM(total_revenue)`
* **OOS Recovery Rate:** `OOS Recovery Rate (%)`

### Visual Charts
1. **SESSION JOURNEY DISTRIBUTION & STOCKOUT OUTCOMES**
   * **Type:** Horizontal 100% Stacked Bar Chart
   * **Dimension:** `event_date` (Granularity: `Year Month`)
   * **Breakdown Dimension:** `session_type`
   * **Metric:** `SUM(total_sessions)`

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

### Tables Section

#### Table 1: SKU Product Breakdown
* **Title:** `PRODUCTS IMPACTED BY STOCKOUTS: RECOVERY & LEAKAGE ANALYSIS`
* **Dimensions:** `item_id`, `item_name`, `item_category`, `session_type`
* **Metrics:** `SUM(total_sessions)`, `SUM(purchasing_sessions)`, `PDP CVR (%)`, `SUM(total_revenue)`
* **Component Filter:** `Include session_type Contains OOS`

#### Table 2: Channel Acquisition Breakdown
* **Title:** `ACQUISITION CHANNEL OOS FRICTION & LEAKAGE ANALYSIS`
* **Dimensions:** `source_medium`, `campaign`
* **Metrics:**
  1. `SUM(pdp_sessions)` *(Total PDP Visits)*
  2. `SUM(oos_pdp_sessions)` *(OOS PDP Visits)*
  3. `Channel OOS Exposure Rate (%)`
  4. `Channel OOS Abandonment Rate (%)`
  5. `Channel OOS Recovery Rate (%)`
  6. `OOS Recovered Revenue`
* **Default Sort:** `SUM(oos_pdp_sessions)` DESC
* **Component Filter:** `Include oos_pdp_sessions > 0`
