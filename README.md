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
