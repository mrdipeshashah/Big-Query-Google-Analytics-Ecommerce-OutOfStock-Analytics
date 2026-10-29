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
| **Page 1** | **OOS Revenue Share (%)** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_revenue ELSE 0 END) / SUM(total_revenue)` | Percent | Proportion of total site revenue generated by sessions that were exposed to an out-of-stock item but recovered via cross-sell conversions. | 
| **Page 1** | **OOS Abandonment Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Proportion of OOS sessions that dropped off completely without purchasing. | 
| **Page 1** | **OOS Recovered Revenue** | `SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_revenue ELSE 0 END)` | Currency | Total revenue saved/recovered from sessions exposed to an out-of-stock PDP. | 
| **Page 1** | **Channel Grouping** | *(See Section 3 Regex CASE Statement)* | Text | Categorizes traffic acquisition sources into high-level default channels. | 
| **Page 1** | **Channel OOS Exposure Rate (%)** | `SUM(oos_pdp_sessions) / SUM(pdp_sessions)` | Percent | Channel-level metric measuring proportion of incoming traffic exposed to stockouts. | 
| **Page 1** | **Channel OOS Abandonment Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Abandoned (No Purchase)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Rate at which out-of-stock traffic from a specific channel drops off without buying. | 
| **Page 1** | **Channel OOS Recovery Rate (%)** | `IFNULL(SUM(CASE WHEN session_type = 'OOS Recovered (Purchased)' THEN total_sessions ELSE 0 END) / SUM(oos_pdp_sessions), 0)` | Percent | Rate at which out-of-stock traffic from a specific channel cross-sells to alternative items. | 
| **Page 2** | **OOS PDP Views** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total product detail page (PDP) views spent on out-of-stock SKUs. | 
| **Page 2** | **In-Stock PDP Views** | `SUM(CASE WHEN stock_status = 'instock' THEN total_pdp_views ELSE 0 END)` | Integer | Total PDP views spent on available in-stock SKUs. | 
| **Page 2** | **OOS Share of Views (%)** | `SUM(CASE WHEN stock_status != 'instock' THEN total_pdp_views ELSE 0 END) / SUM(total_pdp_views)` | Percent | Percentage of total PDP page views exposed to stockouts. | 
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

# 📊 E-commerce Out-of-Stock (OOS) Demand & Ghost Demand Calculator

An interactive financial and operational supply chain calculator connecting GA4 front-end e-commerce session telemetry with back-end inventory planning.

---

## 📌 Overview & Core Purpose

Traditional inventory management systems (ERP/WMS) only record completed sales transactions. When a high-demand SKU goes out of stock, sales drop to zero—causing forecasting algorithms to treat the stockout period as zero demand.

This Google Sheets model bridges **Merchandising Recovery (Point 1)** and **Supply Chain Optimization (Point 4)** by taking out-of-stock session telemetry from Looker Studio / BigQuery to:
1. **Quantify Net Revenue Leakage (£):** Calculate actual revenue lost after accounting for cross-sell recoveries on substitute products.
2. **Surface Uncaptured "Ghost Demand":** Convert unfulfilled PDP browsing velocity into an annualized demand baseline.
3. **Correct Economic Order Quantity (EOQ):** Prevent under-purchasing on future purchase orders by factoring ghost demand into reorder calculations.

---

## 🎨 Color-Coding Architecture

To ensure operational clarity, the sheet uses strict visual segmentation:
* 🟧 **Orange Headers (Columns B to I):** Raw inputs and parameters pulled directly from GA4/Looker Studio or supply chain master data.
* 🟥 **Red Headers (Columns J to O):** Fully automated outputs calculated via dynamic spreadsheet formulas.

---

## 📖 Parameter & Metric Glossary

| Parameter / Metric Name | Type | Example Value | Spreadsheet Formula Logic | Description & Operational Detail |
| :--- | :--- | :--- | :--- | :--- |
| **SKU Name** | Input | Modern Leather Jacket | *Manual Text* | Primary product title or SKU identifier evaluated for out-of-stock performance. |
| **Stockout Duration (Days)** | Input | 14 | *Manual Number* | Total consecutive calendar days the SKU had zero available warehouse inventory. |
| **Total OOS PDP Sessions** | Input | 1,822 | *Manual Number* | High-intent customer sessions that loaded this specific SKU's out-of-stock product detail page (pulled from Looker Studio Page 2). |
| **Baseline In-Stock CVR (%)** | Input | 3.50% | *Manual Percentage* | Historical conversion rate ($CVR$) achieved by this product during active in-stock periods. |
| **Average Order Value (AOV)** | Input | £85.00 | *Manual Currency* | Average total cart revenue generated per completed transaction for this SKU/category. |
| **OOS Recovery Rate (%)** | Input | 28.00% | *Manual Percentage* | Percentage of OOS sessions that converted on an alternative available product on the site (pulled from Looker Studio Page 1). |
| **Ordering Cost per PO (S)** | Input | £150.00 | *Manual Currency* | Fixed administrative, freight setup, receiving, and processing cost per batch purchase order. |
| **Annual Holding Cost per Unit (H)** | Input | £12.00 | *Manual Currency* | Annual warehouse storage, insurance, capital, and handling cost per unit held in inventory. |
| **Gross Lost Demand (Units)** | Formula | 64 | `=Sessions * CVR` | Total unfulfilled unit demand calculated by multiplying OOS sessions by the baseline conversion rate. |
| **Gross Lost Revenue (£)** | Formula | £5,420.45 | `=Gross_Lost_Units * AOV` | Total gross revenue opportunity exposed to the stockout before accounting for cross-sell recovery. |
| **Recovered Revenue (£)** | Formula | £1,517.73 | `=Gross_Lost_Revenue * Recovery_Rate` | Revenue saved by redirecting OOS visitors to purchase alternative available products (**Point 1**). |
| **Net Revenue Leakage (£)** | Formula | £3,902.72 | `=Gross_Lost_Revenue - Recovered_Revenue` | **True Net Loss:** Financial revenue permanently lost to customer bounce and abandonment. |
| **Annualized Ghost Demand (Units)** | Formula | 1,663 | `=(Gross_Lost_Units / Duration_Days) * 365` | **Ghost Demand:** Annualized uncaptured unit demand baseline derived from stockout browsing velocity (**Point 4**). |
| **Adjusted Reorder Quantity (EOQ)** | Formula | 204 | `=SQRT((2 * Ghost_Demand * S) / H)` | **Corrected EOQ:** Optimized batch reorder quantity balancing batch setup costs against holding costs to prevent repeat stockouts. |

---

## 🧮 Mathematical Proof & Step-by-Step Calculation

Using the **Modern Leather Jacket** sample data ($1,822$ OOS Sessions over 14 days, $3.50\%$ CVR, $£85.00$ AOV, $28.00\%$ Recovery Rate, $S = £150.00$, $H = £12.00$):

1. **Gross Lost Demand (Units):**
   $$1,822 \text{ sessions} \times 0.035 = 63.77 \approx \mathbf{64 \text{ units}}$$
2. **Gross Lost Revenue (£):**
   $$63.77 \text{ units} \times £85.00 = \mathbf{£5,420.45}$$
3. **Recovered Revenue (£):**
   $$£5,420.45 \times 0.28 = \mathbf{£1,517.73}$$
4. **Net Revenue Leakage (£):**
   $$£5,420.45 - £1,517.73 = \mathbf{£3,902.72}$$
5. **Annualized Ghost Demand (Units/Yr):**
   $$\left(\frac{63.77 \text{ units}}{14 \text{ days}}\right) \times 365 = 1,662.64 \approx \mathbf{1,663 \text{ units/year}}$$
6. **Adjusted Economic Order Quantity (EOQ):**
   $$\text{EOQ} = \sqrt{\frac{2 \times 1,662.64 \times 150}{12}} = \sqrt{41,566} \approx \mathbf{204 \text{ units per batch}}$$
