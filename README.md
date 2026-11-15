# E-commerce performance dashboard

A Power BI dashboard for a **Supply Chain Manager** who reviews results weekly, monthly, and yearly. It covers sales, profitability, customers, inventory, delivery, returns, and customer satisfaction. The project is saved in **PBIP format**: the report (PBIR) and semantic model (TMDL) are plain text under `dashboard/`.

This README describes the dashboard **as built**. `dashboard/DASHBOARD_INSTRUCTION.md` holds the metric definitions, business rules, and a page-by-page description of the build.

## Open the dashboard
1. Open `dashboard/dashboard.pbip` in Power BI Desktop.
2. Refresh the data if Desktop asks.

### Data path
Each table loads its CSV through Power Query using an absolute path:

```
D:\projects\vintelligence2026_ai_agent_powerbi_dashboard\data\<file>.csv
```

If you clone the repository somewhere else, update the `File.Contents(...)` path in all 13 table partitions. You can do this in Power Query or in `dashboard/dashboard.SemanticModel/definition/tables/*.tmdl`.

## Data sources
All 13 CSVs are in `data/`. Power Query only promotes headers and sets column types: it doesn't filter, deduplicate, or replace values. ID columns are typed as whole numbers (`Int64`).

| File | Model table | Role |
|---|---|---|
| orders.csv | Orders | Fact: one row per order |
| order_items.csv | OrderItems | Fact: order × product line |
| payments.csv | Payments | Fact: one row per order |
| shipments.csv | Shipments | Fact: shipment records |
| returns.csv | Returns | Fact: returned lines |
| reviews.csv | Reviews | Fact: product reviews |
| inventory.csv | Inventory | Monthly product stock snapshot |
| sales.csv | Sales | Daily benchmark revenue series |
| web_traffic.csv | WebTraffic | Daily traffic, from 2013-01-01 |
| customers.csv | Customers | Dimension |
| geography.csv | Geography | Dimension |
| products.csv | Products | Dimension |
| promotions.csv | Promotions | Dimension (not related to any fact) |

The model also contains a calculated `Date` table covering 2012-01-01 to 2023-12-31. Power BI's auto date/time option is on, so Desktop adds hidden `LocalDateTable_*` tables.

## Report pages
The report has eight pages. Seven are analytical, each 1920×1570 with *Fit to width*. The eighth, a blank `Page 1`, is left over from setup.

Every analytical page has the same layout:
- A header band showing the page title and a one-line purpose.
- Four dropdown slicers, synced across all seven pages: **Reporting period** (Year-Month), **Product category**, **Product**, and **Region**.
- A row of four or five KPI cards.
- Charts in rows below the KPI cards, ending in a detail table.

| # | Page | KPI cards | Main visuals |
|---|---|---|---|
| 1 | **Executive Overview** | Shipment Coverage, Repeat Customer Rate, Stockout Rate, Return Rate, Average Rating | Net/Gross Sales and Gross Profit by month; orders by status; stockout rate, return rate, and average rating by category; category health table; product exception detail |
| 2 | **Sales and Profitability** | Gross Sales, Net Sales, Total Orders, Average Order Value | Net Sales by month; Net Sales by order source, device, and payment method; discount rate by category; category table; catalog-cost profitability detail |
| 3 | **Customers and Market Performance** | Distinct Customers, First-Time Buyer Rate (all-time), Repeat Buyer Rate (all-time), Revenue per Customer | Customers by acquisition channel and age group; orders by device and sales source; revenue per customer by segment; Net Sales by region; customer segment tables |
| 4 | **Inventory Performance** | Current Stock, Inventory Value, Low-Stock Products, Out-of-Stock Products | Stock in vs stock out by snapshot date; sell-through rate, current stock, and low/out-of-stock products by category; inventory value by segment; inventory exception detail |
| 5 | **Delivery and Logistics** | Fulfillment Rate, Average Transit Time, Transit Within 3 Days (diagnostic), Shipment Cost | Transit-within-3-days and shipment cost by ship month; orders by status; transit diagnostics by region; delivery status detail |
| 6 | **Returns and Product Quality** | Returned Orders, Refund Amount, Return Rate, Return Quantity | Returned quantity and refund amount by return month; returns by reason and by category; return rate by category; return detail by reason |
| 7 | **Customer Satisfaction** | Average Rating, Review Count, Positive Review Rate, Negative Review Rate | Review count and average rating by review month; rating distribution; reviews by age group; negative review rate and average rating by category; product satisfaction detail |
| 8 | Page 1 | – | Blank |

**Date basis.** Most visuals follow the order date. The shipment trends use the ship date, the return trends use the return date, and the review trends use the review date. Each of these has its own "by … Date" measure, so the Reporting-period slicer filters it by that date. Inventory is a monthly snapshot: its stock measures take the latest snapshot in context and never sum across dates. The 3-day transit split is shown only as a diagnostic, because the data has no promised delivery date.

## KPIs shown on the report
- **Sales:** Gross Sales, Net Sales, Total Orders, Average Order Value, Units Sold, Discount Rate, COGS, Gross Profit, Gross Margin %
- **Customers:** Distinct Customers, All-Time First-Time / Repeat Customer Rate, Revenue per Customer, Order Frequency
- **Inventory:** Current Stock, Inventory Value, Low-Stock Products, Out-of-Stock Products, Stockout Rate, Stock In / Stock Out, Sell-Through Rate
- **Delivery:** Fulfillment Rate, Shipment Coverage Rate, Shipment Cost, Average Transit Time, Transit Within 3 Days (diagnostic), Shipment Records
- **Returns:** Returned Orders, Return Quantity, Refund Amount, Return Rate, Return Rate for Product Orders, Return Reason Share
- **Satisfaction:** Average Rating, Review Count, Positive / Negative Review Rate

Some measures exist in the model but no page uses them yet:
- Web traffic: Total Sessions, Conversion Rate, Page Views, and related measures.
- The `Sales` benchmark series.
- Payment value.
- The growth measures: YoY, MoM, and Order Growth.

## Important assumptions
- The CSV files are the source of truth and aren't modified.
- Sales and order metrics come from `orders.csv` and `order_items.csv`. **Net Sales** is after discounts and before refunds. *Net Sales After Returns* subtracts refunds.
- Profitability uses the current catalog COGS from `products.csv`, not historical cost.
- The `Sales` table is a separate benchmark series and isn't blended with order revenue.
- The model is a star schema with one conformed `Date` table. The order date is the active date. The ship, delivery, return, review, and snapshot dates connect through inactive relationships, which a measure turns on when it needs them.

## Known limitations
These describe the current build. Improvements will be discussed separately.
- **Shipments has no active relationship to the rest of the model.** Shipment Cost, Average Transit Time, and Transit Within 3 Days ignore the slicers, so the region breakdowns show the same value on every row. There are three exceptions:
  - The "by Ship Date" trend measures respond to the date slicer.
  - The "Product-Linked" measures respond to product filters.
  - Shipment Coverage Rate's denominator (Total Orders) does respond to filters.
- **Inventory is related only to Products.** The Reporting-period and Region slicers don't filter it, so Current Stock always shows the latest snapshot overall.
- **The Product category and Product slicers don't filter order-level measures** such as Total Orders, Distinct Customers, Fulfillment Rate, and order status. The "Return Rate for Product Orders" measure exists for that reason.
- Customer counts by acquisition channel and age group count customer records, so the date, category, and region slicers don't filter them.
- Promotion attribution is limited: many `promo_id` values are blank, and `Promotions` isn't related to any fact table.
- Visuals have no alt text, and pages have no reset-filters button.
- The data is synthetic and may not represent a deployed ERP environment.
