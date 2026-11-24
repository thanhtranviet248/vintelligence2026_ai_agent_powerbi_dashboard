# VinTelligence · E-commerce Performance Dashboard

## 1. Project overview

**Goal.** Give one integrated view of how the e-commerce business performs and how the supply chain behind it (stock, delivery, returns) affects sales and customer outcomes. The dashboard should help its user:

- monitor sales, gross profit and growth against last year;
- compare performance across categories, products, customer groups, channels and regions;
- spot stock shortages, excess stock and slow-moving products;
- follow shipping cost, order fulfilment and delivery lead time;
- find the products and categories behind returns, refunds and poor reviews;
- decide where to look next.

**User.** The **Ecommerce Manager**, working with the supply-chain, fulfilment and customer-service teams. Results are reviewed **weekly, monthly and yearly**; daily or real-time monitoring is out of scope.

**How it is read.** Executive Overview gives the headline. Each of the other six pages covers one domain and reads top to bottom as a short story: KPI cards → what is happening over time → what drives it → where to act → which records.

**Detailed specification.** [`DASHBOARD_INSTRUCTION.md`](DASHBOARD_INSTRUCTION.md) defines every metric, its business rules and every visual on every page.

## 2. Data sources

The data is 13 CSV files covering orders from 2012 to 2022. Ten of them are loaded into the model. Values are not filtered, deduplicated or replaced on load.

| File | Model table | Role | Grain · rows |
|---|---|---|---|
| orders.csv | Orders | Fact | One row per order · 646,945 |
| order_items.csv | OrderItems | Fact (sales, discount, COGS) | Order × product line · 714,669 |
| shipments.csv | Shipments | Fact (shipping fee, ship/delivery dates) | Shipment record · 566,067 |
| returns.csv | Returns | Fact (returned lines, refunds, reasons) | Returned line · 39,939 |
| reviews.csv | Reviews | Fact (star rating, review date) | Product review · 113,551 |
| inventory.csv | Inventory | Snapshot (stock on hand, units in/out, stockout days) | Product × month · 60,247 |
| customers.csv | Customers | Dimension (age group, acquisition channel) | Customer · 121,930 |
| geography.csv | Geography | Dimension (region, city, district) | Location · 39,948 |
| products.csv | Products | Dimension (category, segment, size, colour, catalog cost) | Product · 2,412 |
| promotions.csv | Promotions | Dimension (promotion name and type) | Promotion · 50 |
| payments.csv | – | Not loaded (payment method comes from orders) | – |
| sales.csv | – | Not loaded (separate daily benchmark revenue series) | – |
| web_traffic.csv | – | Not loaded (daily traffic) | – |

**Model.** A star schema with one shared **Date** table (2012–2022). Order date drives orders, sales, shipments, returns and reviews; the snapshot date drives inventory. Ship, delivery, return and review dates are also linked but inactive; only the review-sentiment trend switches to review date. Products and Promotions filter the order lines; Customers and Geography filter the orders.

**Key definitions.**
- Net Sales is after discounts and before refunds.
- Inventory is a monthly snapshot. Stock measures take the latest snapshot in the selected period and never add snapshots together.
- Low stock is 20 units or fewer.
- A positive review is 4–5 stars, a negative review 1–2 stars.

## 3. The dashboard

The report has **7 pages**, opening on Executive Overview, plus one hidden tooltip page.

**On every page:**
- a header with the page title (the same as the tab name), a *Data through* date and a **Reset filters** button;
- four dropdown slicers, synced across all pages: **Reporting period** (year-month), **Product category**, **Product**, **Region**;
- numbered section headings, each naming the question the section answers;
- a search box above each detail table (product, promotion or city). It filters only that table.

**Category tooltip.** Hover a category on any of the six category bar charts to see a mini scorecard for that category: Net Sales, Net Sales YoY, Gross Margin, Return Rate, Negative Review Rate and Stockout Rate.

**Growth arrows.** Growth values show ▲/▼, green for up and red for down. For "lower is better" metrics such as return or stockout rate, red means worse. Colour always repeats an arrow, label or text.

### 3.1. Executive Overview

Is the business growing, and is the operation keeping up? Totals only, with no product-level detail.

- **Key KPIs:** Net Sales · Net Sales Growth YoY · Net Sales Growth MoM · Gross Margin · Total Orders · Return Rate | Inventory Turnover · Stockout Rate · Fulfilled Rate · Avg Order-to-Delivery Days · Transit Within 3 Days (diagnostic)
- **Reading flow:**
  1. **Sales performance:** Net Sales against the prior year by month, then the YoY growth for each year.
  2. **Operations health:** inventory turnover against its period average, stockout rate by month, and the share of orders in each status.
  3. **Category scorecard:** one row per category with sales, growth, margin, stock, fulfilment, delivery, returns and rating. A **Watch** flag (⚠) names falling sales, stockouts or returns above the all-category rate.

### 3.2. Sales & Profitability

Where do revenue, growth and margin come from?

- **Key KPIs:** Net Sales · Gross Sales · Total Orders · Average Order Value · Promoted Sales Share · Top 20% Product Sales Share | Net Sales Growth YoY · Net Sales Growth MoM · Discount Rate · Gross Profit Growth YoY · Order Growth YoY · Gross Margin
- **Reading flow:**
  1. **Sales trend and growth:** Net Sales against the prior year by month; sales, profit and order growth by year; a monthly trend whose metrics the reader picks (**Trend Metric** list, which opens on Gross Profit and Discount Amount).
  2. **Category and mix:** Net Sales YoY % by category; sales, profit and promotion mix by category; Net Sales by a dimension the reader picks (**Breakdown Dimension** list, which opens on Segment); discount rate by category.
  3. **Product and promotion detail:** a product profitability table (at catalog cost) and a promotion table, each with its own search box.

### 3.3. Customers & Markets

How big is the customer base, who are the customers, and how do they buy?

- **Key KPIs:** Active Customers · New Customers · Repeat Customer Rate · Revenue per Active Customer · Repeat Buyers (All-Time). With no period selected, the first four cards show the latest 12 months.
- **Reading flow:**
  1. **Customer base trend:** new and repeat customers by month; repeat customer rate against its period average.
  2. **Who our customers are:** customers by acquisition channel and by age group; revenue per customer by product segment.
  3. **Market and buying behaviour:** Net Sales by region, average order value by payment method, Net Sales by order source, order frequency by device type.
  4. **City detail:** customer value by city, with a city search box.

### 3.4. Inventory

Are we filling demand, and where is stock short or excessive? The planner's view.

- **Key KPIs:** Fill Rate · Stockout Rate · Units on Hand · Inventory Value · Days of Supply · Inventory Turnover | Products at Shortage Risk · Products with Excess Stock · Excess Stock Value
- **Reading flow:**
  1. **Service level:** fill rate and stockout rate by month and by ABC class.
  2. **Stock movement:** units in, units out and units on hand by month; turnover by month; inventory value by month and days-of-cover band.
  3. **Stock status and risk:** products by month and by category, split into Shortage risk / Excess stock / Healthy.
  4. **Category and product detail:** a category health table, inventory value by ABC class and cover band, and a **planning action** for each product (▲ Replenish / ▼ Reduce excess / ● Healthy), with excess units and excess stock value.

### 3.5. Shipping & Delivery

What does shipping cost, are orders fulfilled, and how fast do they arrive?

- **Key KPIs:** Shipment Cost · Shipment Cost per Shipment · Premium-Fee Shipments ($10+ fee) · Fulfilled Rate · Cancellation Rate · Average Order-to-Delivery Days (split into days to ship and days in transit)
- **Reading flow:**
  1. **Shipping cost:** cost per shipment and premium-fee share by month; cost per shipment by basket size and by category.
  2. **Order fulfilment:** order status share by year; cancellation rate by payment method.
  3. **Delivery speed:** lead time by month; shipments by transit days.
  4. **Region and city detail:** a region table for cost and fulfilment, and a city lead-time table with a city search box.

Delivery speed hardly changes in the data (about 1.5 days to ship plus 4.5 days in transit everywhere), so the page leads with cost and fulfilment, where the variation is.

### 3.6. Returns & Product Quality

Are returns and refunds rising, why do products come back, and where do refunds concentrate?

- **Key KPIs:** Return Rate · Returned Orders · Refund Amount · Refund per Returned Order · Refund Rate · Average Days to Return
- **Reading flow:**
  1. **Return and refund trend:** return rate and refund rate by month; refund amount by year.
  2. **Why products come back:** share of each return reason, return lines by days to return, return rate by payment method.
  3. **Where refunds concentrate:** refund amount by category; a region table.
  4. **Product detail:** returns and refunds by product, with a product search box.

### 3.7. Customer Satisfaction

Is sentiment improving, and which products and categories disappoint customers?

- **Key KPIs:** Average Rating · Positive Review Rate · Negative Review Rate · Review Count · Review Coverage · Average Days to Review
- **Reading flow:**
  1. **Satisfaction trend:** positive and negative review rate by review month; review coverage (the share of delivered orders that get a review) by year.
  2. **Rating mix and feedback timing:** share of each star rating, reviews by days to review, negative review rate by transit days.
  3. **Where dissatisfaction concentrates:** negative review rate by category; a region table.
  4. **Product detail:** rating, review mix and return rate by product (blank for products with fewer than 10 reviews), with a product search box.

## 4. Limitations

**Data**
- The data is synthetic. Ratings and return rates are almost flat across regions, categories, sizes and customer groups, so the pages emphasise the views that do vary (trends, mixes, timing, product detail).
- There is no promised delivery date, so there is no on-time or late-delivery KPI. The 3-day transit share is a diagnostic only, not an SLA.
- Profitability uses each product's current catalog cost, not the historical cost at the time of sale.
- Promotion attribution is limited. 61% of order lines have no promotion, and only the primary promotion is linked. The second promotion ID (filled on 0.03% of lines) isn't loaded.
- Web traffic, payments and the benchmark sales series aren't loaded, so there are no conversion or payment-value metrics.

**Metric behaviour**
- **Returns:** returns arrive up to 31 days after the order. Return and refund rates therefore count only orders placed at least 31 days before the last date in the data, so the most recent month is never under-counted.
- **Shipments:** shipment measures follow the order. Under a product filter, shipment cost is the full fee of each order that contains the product; it isn't split across the products in the order.
- **Inventory and Region:** the Region slicer doesn't filter inventory, because stock is held by product, not by region.
- **Inventory flows:** units in and units out add up the monthly flows of every snapshot in the selected period.
- **Customer KPIs:** with no period selected, the customer KPI cards fall back to the latest 12 months; the charts and tables follow the slicer as is.
- **Reviews:** negative review rate by transit days is a comparison, not proof that slow delivery causes bad ratings.

**Report**
- Several KPI cards have no subtitle and no comparison to last year.
- The low-stock threshold (20 units) and minimum review count (10) have no slicer, so they always run at these defaults. The transit threshold is fixed at 3 days, and there is no return-reduction scenario.
- Every measure in the model feeds a visual or another measure; unused measures were removed (see `DASHBOARD_INSTRUCTION.md` §2.10).
- Navigation is by page tabs only. There are no drill-through pages or bookmarks.
