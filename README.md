# E-commerce performance dashboard

A Power BI dashboard for an **Ecommerce Manager** who reviews results weekly, monthly, and yearly. It covers sales, profitability, customers, inventory, delivery, returns, and customer satisfaction. The project is saved in **PBIP format**: the report (PBIR) and semantic model (TMDL) are plain text under `dashboard/`.

This README describes the dashboard **as built**. `dashboard/DASHBOARD_INSTRUCTION.md` holds the metric definitions, business rules, and a page-by-page description of every visual.

## Open the dashboard
1. Open `dashboard/dashboard.pbip` in Power BI Desktop.
2. Refresh the data if Desktop asks.

### Data path
Each table loads its CSV through Power Query using an absolute path:

```
D:\projects\vintelligence2026_ai_agent_powerbi_dashboard\data\<file>.csv
```

If you clone the repository somewhere else, update the `File.Contents(...)` path in all 10 table partitions. You can do this in Power Query or in `dashboard/dashboard.SemanticModel/definition/tables/*.tmdl`.

## Data sources
All 13 CSVs are in `data/`, and the model loads 10 of them. Power Query promotes headers and sets column types. On some tables it also drops columns that no visual or measure uses. It doesn't filter, deduplicate, or replace values. ID columns are typed as whole numbers (`Int64`).

| File | Model table | Role |
|---|---|---|
| orders.csv | Orders | Fact: one row per order |
| order_items.csv | OrderItems | Fact: order × product line |
| payments.csv | – | Not loaded (payment method comes from `orders.csv`) |
| shipments.csv | Shipments | Fact: shipment records |
| returns.csv | Returns | Fact: returned lines |
| reviews.csv | Reviews | Fact: product reviews |
| inventory.csv | Inventory | Monthly product stock snapshot |
| sales.csv | – | Not loaded: daily benchmark revenue series |
| web_traffic.csv | – | Not loaded: daily traffic, from 2013-01-01 |
| customers.csv | Customers | Dimension |
| geography.csv | Geography | Dimension |
| products.csv | Products | Dimension |
| promotions.csv | Promotions | Dimension (filters OrderItems through the primary promotion ID) |

The model also contains a calculated `Date` table covering 2012-01-01 to 2022-12-31. It is marked as the model's date table and has month-start and week-start (Monday) columns. Power BI's auto date/time option is on, so Desktop adds hidden `LocalDateTable_*` tables.

## Report pages
The report has **seven pages**, all 1920 px wide. Executive Overview opens first.

| # | Page (tab name) | Header title | Size | View |
|---|---|---|---|---|
| 1 | Executive Overview | Executive Overview | 1920×1750 | Fit to page |
| 2 | Sales & Profitability | Sales and Profitability | 1920×2780 | Fit to page |
| 3 | Customers & Market | Customers and Market Performance | 1920×2240 | Fit to page |
| 4 | Inventory | Inventory Performance | 1920×3460 | Fit to page |
| 5 | Shipment | Delivery and Logistics | 1920×2460 | Fit to width |
| 6 | Returns & Product Quality | Returns and Product Quality | 1920×2240 | Fit to width |
| 7 | Customer Satisfaction | Customer Satisfaction | 1920×2240 | Fit to width |

One hidden page, **Category Tooltip** (320×240), is the report-page tooltip of the six category bar charts (Net Sales YoY %, discount rate, products by stock risk, shipping cost, refund amount, negative review rate). Hovering a category shows its name and six cards: Net Sales, Net Sales YoY, Gross Margin, Return Rate (closed window), Negative Review Rate and Stockout Rate (latest snapshot, ignoring the stock-risk segment). There is no drill-through page.

Every page has the same layout:
- **A header band.** It shows the brand line, the page title with a one-line note on how to read the page, a *Data through* card showing the last date in the data, and a **↺ Reset filters** button.
- **Four dropdown slicers**, synced across all seven pages: **Reporting period** (Year-Month), **Product category**, **Product**, and **Region**.
- **One or two rows of KPI cards.**
- **A story told in numbered sections.** Each section opens with a heading that names it and the question it answers. Executive Overview and Sales and Profitability have three sections and the other pages have four. On the domain pages the sections go from the trend, to what drives it, to where to act, and finally to the detail table.

| # | Page | KPI cards | Sections |
|---|---|---|---|
| 1 | **Executive Overview** | Net Sales, Net Sales Growth YoY and MoM, Gross Margin, Total Orders, Return Rate (orders 31+ days old); Inventory Turnover, Stockout Rate, Fulfilled Rate, Avg Order-to-Delivery Days, Transit Within 3 Days (diagnostic) | 1 Sales performance: Net Sales vs prior year by month; Net Sales YoY % by year → 2 Operations health: inventory turnover by month against its period average; stockout rate by month; order share by status → 3 Category scorecard: one row per category, with a Watch flag. Totals only, with no product-level visuals |
| 2 | **Sales and Profitability** | Net Sales, Gross Sales, Total Orders, Average Order Value, Promoted Sales Share, Top 20% Product Sales Share; Net Sales Growth YoY and MoM, Discount Rate, Gross Profit Growth YoY, Order Growth YoY, Gross Margin | 1 Sales trend and growth: Net Sales vs prior year by month; Net Sales, gross profit and order growth by year; selectable sales metrics by month, using the Trend Metric list (opens on Gross Profit and Discount Amount) → 2 Category and mix: Net Sales YoY % by category; sales, profit and promotion mix by category; Net Sales by a selectable dimension (Breakdown Dimension list, opens on Segment); discount rate by category → 3 Product and promotion detail: a catalog-cost profitability table and a promotion table, each with its own search box |
| 3 | **Customers and Market Performance** | Active Customers, New Customers, Repeat Customer Rate, Revenue per Active Customer, Repeat Buyers (All-Time). When no period is selected, the first three show the latest 12 months | 1 Customer base trend: new and repeat customers by month; repeat customer rate against its period average → 2 Who our customers are: customers by acquisition channel, customers by age group, revenue per customer by product segment → 3 Market and buying behaviour: Net Sales by region, Average Order Value by payment method, Net Sales by order source, order frequency by device type → 4 City detail: customer value by city, with a city search box |
| 4 | **Inventory Performance** | Fill Rate, Stockout Rate, Current Stock (units in/out in the subtitle), Inventory Value (vs prior year in the subtitle), Days of Supply, Inventory Turnover; Products at Shortage Risk, Products with Excess Stock, Excess Stock Value (each with a share in the subtitle) | 1 Service level: fill and stockout rate by month and by ABC class → 2 Stock movement: inbound, outbound and units on hand by month; turnover by month; inventory value by month and cover band → 3 Stock status and risk: products by month and by category, split into Shortage risk / Excess stock / Healthy → 4 Category and product detail: category health table, inventory value by ABC class and cover band, and planning actions by product (▲ Replenish / ▼ Reduce excess / ● Healthy) with a product search box |
| 5 | **Delivery and Logistics** | Shipment Cost, Cost per Shipment, Premium-Fee Shipments ($10+ fee), Fulfilled Rate, Cancellation Rate, Avg Order-to-Delivery Days (the subtitle splits days to ship and days in transit) | 1 Shipping cost: cost per shipment and premium-fee share by month; cost per shipment by basket size and by category → 2 Order fulfilment: order status share by year; cancellation rate by payment method → 3 Delivery speed: lead time by month; shipments by transit days → 4 Region and city detail: shipping cost and fulfilment by region; delivery lead time by city, with a city search box |
| 6 | **Returns and Product Quality** | Return Rate, Returned Orders, Refund Amount, Refund per Returned Order, Refund Rate, Average Days to Return. Both rates use orders 31+ days old | 1 Return and refund trend: return and refund rate by month; refund amount by year → 2 Why products come back: return reason share; return lines by days to return; return rate by payment method → 3 Where refunds concentrate: refund amount by category; returns and refunds by region → 4 Product detail: returns and refunds by product, with a product search box |
| 7 | **Customer Satisfaction** | Average Rating, Positive Review Rate, Negative Review Rate, Review Count, Review Coverage, Average Days to Review | 1 Satisfaction trend: positive and negative review rate by review month; review coverage by year → 2 Rating mix and feedback timing: rating share by star; reviews by days to review; negative review rate by transit days → 3 Where dissatisfaction concentrates: negative review rate by category; satisfaction by region → 4 Product detail: rating, review mix and return rate by product (blank below 10 reviews), with a product search box |

**Growth colours.** On Executive Overview and Sales and Profitability, the growth cards and charts (Net Sales YoY and MoM, Gross Profit YoY, Order Growth YoY) show ▲/▼ arrows, green for up and red for down. Colour always repeats what the arrow already says. The category scorecard marks above-average stockout and return rates red. Gross profit below zero is red on the Sales tables.

**Date basis.** Every visual follows the **order date** except the Customer Satisfaction sentiment trend, which uses the review date. Inventory is a monthly snapshot: its stock measures take the latest snapshot in context and never sum across dates. Each visual's subtitle states its date basis. The 3-day transit split is shown only as a diagnostic, because the data has no promised delivery date.

**Parameters.** Two field-parameter slicers on Sales and Profitability change what a chart draws. They don't filter the data.

| Slicer | Effect | Default |
|---|---|---|
| Trend Metric | Chooses the metrics on the monthly trend: Net Sales, Gross Sales, Gross Profit, COGS, Discount Amount, Refund Amount | Gross Profit and Discount Amount |
| Breakdown Dimension | Chooses the dimension of the Net Sales bar chart: Category, Segment, Size, Color, Region, City, District, Promotion Type | Segment |

The model also has four threshold and scenario parameters. None of them has a slicer on any page, so each stays at its default:

| Parameter | Default |
|---|---|
| Low Stock Threshold | 20 units |
| Transit Day Threshold | 3 days |
| Return Reduction | 0% |
| Minimum Reviews | 10 |

**Search.** Every detail table has a search box just above it: product, promotion or city. Type part of a name and tick the match. Only that table is filtered.

## KPIs shown on the report
- **Sales:** Net Sales, Gross Sales, Total Orders, Average Order Value, Units Sold, Discount Amount, Discount Rate, COGS, Gross Profit, Gross Margin %, Promoted Sales Share, Top 20% Product Sales Share, Net Sales Prior Year, Net Sales / Gross Profit / Total Orders YoY %, Net Sales MoM %
- **Customers:** Active, New and Repeat Customers and Repeat Customer Rate (each as a `… KPI` card version that falls back to the latest 12 months), Distinct Customers, Revenue per Customer, Order Frequency, All-Time Repeat Customer Rate
- **Inventory:** Fill Rate (Latest), Stockout Rate, Current Stock, Inventory Value, Days of Supply, Inventory Turnover, Inbound / Outbound Stock, Shortage-Risk Products, Excess-Stock Products, Excess Units, Excess Stock Value, Excess Value Share, Planning Action; columns Cover Band, Stock Risk and ABC Class
- **Delivery:** Shipment Cost, Shipment Cost per Shipment, Premium-Fee Shipment Share, Fulfilled Rate, Cancellation Rate, Order Status Share, Average Days to Ship, Average Delivery Time, Average Order-to-Delivery Days, Transit Within 3 Days (diagnostic), Shipment Records; columns Orders[Basket Size], Shipments[Transit Days]
- **Returns:** Return Rate (Closed Window), Refund Rate (Closed Window), Returned Orders, Refund Amount, Refund per Returned Order, Average Days to Return, Return Reason Share, Return Lines, Return Rate for Product Orders; columns Returns[Days to Return], Returns[Return Reason Label]
- **Satisfaction:** Average Rating, Positive / Negative Review Rate (also "by Review Date" and "Min Reviews" versions), Review Count, Review Coverage, Rating Share, Average Days to Review, Negative Review Rate by Shipment

Growth measures live in `Date.tmdl`. YoY compares like-for-like dates, and MoM compares the latest month with the one before it. The first year of data has no YoY value. Web-traffic, benchmark-revenue, and payment-value measures aren't built, and `payments.csv`, `sales.csv`, and `web_traffic.csv` aren't loaded.

Some measures stay in the model but no visual shows them:
- Most `… vs PY Label` / `… vs PY Color` measures. Only `Inventory Value vs PY Label` is shown. `Total Orders vs PY Label` is bound to the hidden subtitle of the *Data through* card.
- The scenario measures `Net Sales After Returns (Scenario)` and `Refund Saved (Scenario)`.
- The plain `Return Rate` / `Refund Rate`.
- The "by Ship Date" and "by Return Date" variants.
- `Sell-Through Rate`.

They are listed in `DASHBOARD_INSTRUCTION.md` §6.4.

## Important assumptions
- The CSV files are the source of truth and aren't modified.
- Sales and order metrics come from `orders.csv` and `order_items.csv`. **Net Sales** is after discounts and before refunds. `discount_amount` is a line total, so it is never multiplied by quantity.
- Profitability uses the current catalog COGS from `products.csv`, not historical cost.
- `sales.csv` is a separate benchmark series. It isn't loaded and must not be blended with order revenue.
- The model is a star schema with one conformed `Date` table. The order date is the active date for orders and everything linked to them, and the inventory snapshot date is active for `Inventory`. The ship, delivery, return, and review dates connect through inactive relationships, which a measure turns on when it needs them.
- Order-level measures (Total Orders, Distinct Customers, order status, shipments) respond to the Product category and Product slicers. They do this through `CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH)` inside each measure, and they count the orders that contain the selected products. The model relationship itself stays single-direction.
- Returns arrive up to 31 days after the order, so the return and refund rates count only orders placed 31+ days before the last date in the data.
- Low stock means stock on hand at or below 20 units. A positive review is 4–5 stars, a negative one is 1–2 stars.
- Promoted Sales Share counts order lines with a primary promotion ID. Top 20% Product Sales Share ranks only the products that sold in the current filters.

## Known limitations
These describe the current build.
- **Shipment measures follow the order.** Shipment Cost, Average Delivery Time, and Transit Within 3 Days are filtered by order date, Region, and product. Under a product filter, Shipment Cost is the full shipping fee of each order that contains the product. It isn't split across the products in the order.
- **The Region slicer doesn't apply to inventory.** Inventory connects to Products and to the snapshot date only. Inventory measures show the latest snapshot inside the selected period.
- Inbound Stock (units received) and Outbound Stock (units sold) add up the monthly flows of every snapshot in the period.
- Promotion attribution is limited. 61% of order lines have a blank `promo_id`, and only the primary promotion ID is related to `Promotions`. The second promotion ID is filled on 0.03% of lines and isn't loaded.
- Negative Review Rate by Transit Days is a comparison, not proof that transit time causes the rating.
- The data is synthetic and may not represent a deployed ERP environment.
