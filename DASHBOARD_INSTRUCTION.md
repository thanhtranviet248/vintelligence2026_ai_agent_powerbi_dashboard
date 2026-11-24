> **How to read this document.** Sections 1, 2.2, 3.1, and 5 are the original design intent and business rules. New measures and visuals should still follow them. Everything else (implementation status, data preparation, relationships, and the page design in section 6) describes the dashboard **as built** in `dashboard.Report/` and `dashboard.SemanticModel/`. Where the build departs from a rule, the as-built sections say so. Improvements to the build will be discussed separately.

# 1. PURPOSES OF THE DASHBOARD

## 1.1. Primary User

Ecommerce Manager (the report header reads "VinTelligence · E-commerce Performance"), working with the supply chain, fulfilment and customer-service teams. The original spec named the Supply Chain Manager; the supply-chain pages (Inventory, Shipment, Returns & Product Quality) still serve that role.

## 1.2. Objectives and Scope

The dashboard provides an integrated view of supply chain performance and its relationship with sales and customer outcomes. It should help the Ecommerce Manager:

- Monitor sales, revenue, gross profit, and demand trends.
- Compare performance across products, categories, customer segments, sales channels, and geographic markets.
- Identify inventory shortages, excess stock, slow-moving products, and potential fulfillment bottlenecks.
- Evaluate shipment progress, delivery lead times, shipping charges, and delivery reliability.
- Identify products and operational issues associated with returns, refunds, and poor customer ratings.
- Prioritize areas for further investigation and operational improvement.

Reporting should support weekly, monthly, and yearly analysis. Daily reporting and real-time operational monitoring are outside the dashboard’s scope. However, underlying transaction dates may still be required to calculate lead times and assign records to reporting periods.
Inventory reporting is limited to the frequency of the available snapshots. Monthly inventory snapshots must not be presented as weekly stock balances.

## 1.3. Key Business Questions

Sales and profitability: Track units sold, order volume, gross sales, discounts, net sales, COGS, gross profit, and gross margin over time. Key questions include:

- Which products, categories, channels, and customer segments generate the most revenue and gross profit?
- Where are sales or margins improving or declining?
- How do discount levels and promotional activity relate to sales volume and profitability?
- Are changes in revenue associated with changes in order volume, average order value, or product mix?

Customers and market performance: Evaluate customer purchasing activity and commercial performance across customer segments, acquisition channels, sales channels, and geographic markets. Key questions include:

- Which customer segments, regions, and cities contribute the most customers, orders, and revenue?
- Which segments or markets are growing or declining?
- How do order frequency and revenue per customer differ across segments?
- What proportion of purchasing customers are first-time versus returning buyers?
- Which acquisition channels attract customers who subsequently generate orders and revenue?

Customer acquisition channel and order sales channel should remain separate: one describes how the customer was acquired, while the other describes where an order was placed.

Inventory performance: Monitor stock availability, stock receipts, sales movement, and fulfillment indicators at the product and inventory-snapshot level. Key questions include:

- Which products are out of stock, below the low-stock threshold, or flagged for replenishment?
- Which products have excess inventory or low sales movement?
- Where do stock shortages coincide with strong demand or low fill rates?
- How effectively is available inventory converted into sales?
- Which products require investigation because inventory availability may be constraining sales?
- Potential lost sales should not be presented as a measured value unless unmet demand or another approved estimation method is available.

Delivery and logistics performance: Track order progression from placement through shipment and delivery. Key questions include:

- How long does it take to dispatch and deliver orders?
- What proportion of orders have been shipped, delivered, cancelled, or remain open?
- Which products, channels, or geographic markets have longer delivery times?
- Which shipments are late relative to a promised delivery date, where that date is available?
- How do delivery performance and shipping charges relate to returns and customer ratings?
- A long delivery lead time does not automatically mean a delivery is late. Identifying late deliveries requires a promised delivery date or an agreed service-level target.

Returns and product quality: Track returned quantities, refund values, return rates, and return reasons. Key questions include:

- Which products and categories have the highest return rates and refund values?
- What reasons are most frequently recorded for returns?
- How do returns vary by customer segment, promotion, channel, and period?
- Are particular products repeatedly associated with quality-related return reasons?
- How much do refunds reduce revenue after discounts?
- Return reasons should be used to distinguish potential product-quality issues from delivery problems, customer preference, or other causes.

Customer satisfaction: Review product ratings and written feedback alongside sales, delivery, and return performance. Key questions include:

- Which products receive consistently high or low ratings?
- How do ratings change over time?
- Do low ratings coincide with long delivery times, frequent returns, or recurring complaints?
- How representative are the ratings, given the number of reviews received?
- These comparisons should identify patterns for investigation rather than imply that one factor necessarily caused another.


# 2. METRICS

The dashboard must support weekly, monthly, and yearly comparisons wherever the underlying data allows. Metrics should be filterable by the relevant product, category, customer segment, sales channel, geography, and reporting period.

The tables below describe the semantic model **as built**. Formulas are the DAX in `dashboard.SemanticModel/definition/tables/*.tmdl`, shortened where needed. Measure names appear in backticks. Section 6 lists the pages where each measure is shown.

## 2.1. Implementation Status

**Built:** a measure exists in the model. Where the measure is on a page, the Status cell names that page. "Model only" means no visual uses it yet.

**Built (differs):** the measure exists, but its logic departs from the rule in 2.2 or from the definition column. The note says how.

**Not built:** the model has no measure for it. Usually a required field (for example a promised delivery date) or a business rule is missing.

Page abbreviations (page tab names in brackets): **EO** Executive Overview · **SP** Sales and Profitability ("Sales & Profitability") · **CM** Customers and Market Performance ("Customers & Market") · **IP** Inventory Performance ("Inventory") · **DL** Delivery and Logistics ("Shipment") · **RQ** Returns and Product Quality ("Returns & Product Quality") · **CS** Customer Satisfaction.

## 2.2. Common Calculation Rules

**Reporting calendar:** Use a shared date dimension with consistent week, month, and year definitions. Confirm whether reporting uses calendar or fiscal periods and define the week start day.

**Date basis:** Use order date for sales and purchasing activity; snapshot date for inventory; shipment or delivery date for logistics activity; return or refund date for return activity; and review date for customer feedback. Clearly label exceptions.

**Order eligibility:** Define which order statuses qualify for sales, units, and AOV. Use the same eligible-order population for each related numerator and denominator. Show all-order status monitoring separately.

**Discounts:** `DiscountAmount` is a **line-level total**, not a per-unit value: `Quantity × UnitPrice − DiscountAmount` equals the order's payment value for every order. The measures sum it as is and never multiply it by quantity.

**Revenue terminology:** Net Sales means sales after discounts but before refunds. Show refunds separately and use a separate measure for revenue after refunds.

**Cost basis:** The proposed COGS formula uses the product-master cost. Confirm whether this is historical transaction cost or current catalog cost, and label the result accordingly.

**Snapshot measures:** Do not sum stock balances across multiple snapshot dates. Use a selected snapshot, a period-end snapshot, or an explicitly defined average.

**Percentage measures:** Calculate percentages from aggregated numerators and denominators where available. Clearly label simple averages of source percentages.

**Zero denominators:** Use DIVIDE() or equivalent logic to return blank where no meaningful denominator exists.

**Period comparisons:** Compare equivalent periods. Flag incomplete weeks, months, and years rather than comparing them silently with complete periods.

**Dimension filtering:** Confirm that filters reach all relevant fact tables. Do not assume an orders-table measure is automatically filtered correctly by a product selection.

**Missing data:** Exclude invalid records from calculations where necessary, but report missing dates, missing costs, and unmatched records as data-quality exceptions.

## 2.3. Sales and Profitability Metrics

The sales measures don't apply an order-status filter. They count every order line and order in context, whatever the status (see `Total Orders`). The model has no data-quality exclusions for invalid quantities, prices, or dates. Sales trends use the order date.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Gross Sales | Value of order lines before discounts | `SUMX(OrderItems, Quantity * UnitPrice)` | order_items.csv | Built `Gross Sales`: SP |
| Discount Amount | Total discount. `DiscountAmount` is a line total | `SUM(OrderItems[DiscountAmount])` | order_items.csv | Built `Discount Amount`: SP |
| Net Sales | Gross sales less discounts, before refunds | `[Gross Sales] - [Discount Amount]` | order_items.csv | Built `Net Sales`: EO, SP, CM |
| Discount Rate | Discount value as a share of gross sales | `DIVIDE([Discount Amount], [Gross Sales])` | order_items.csv | Built `Discount Rate`: SP |
| Units Sold | Quantity on order lines | `SUM(OrderItems[Quantity])` | order_items.csv | Built `Units Sold`: SP |
| Total Orders | Distinct orders | `DISTINCTCOUNT(Orders[OrderID])` with `CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH)`, so product filters reach it. All statuses, no eligibility filter. The only status-specific count is `Delivered Orders` | orders.csv | Built (differs): no eligible-order rule. EO, SP, CM, DL |
| Average Order Value (AOV) | Net sales per order | `DIVIDE([Net Sales], [Total Orders])` | orders.csv, order_items.csv | Built `Average Order Value`: SP, CM (by payment method) |
| COGS | Cost of units sold at **current catalog cost** | `SUMX(OrderItems, Quantity * RELATED(Products[COGS]))` | order_items.csv, products.csv | Built `COGS`: SP (titled as a catalog-cost estimate) |
| Gross Profit | Net sales less COGS | `[Net Sales] - [COGS]` | order_items.csv, products.csv | Built `Gross Profit`: SP |
| Gross Margin % | Gross profit as a share of net sales | `DIVIDE([Gross Profit], [Net Sales])` | order_items.csv, products.csv | Built `Gross Margin %`: EO, SP |
| Product / Category / Channel Performance | Net sales, units, and orders by dimension | Existing measures by `Products[Category]`, `Orders[OrderSource]`, `Orders[DeviceType]`, `Orders[PaymentMethod]` | orders.csv, order_items.csv, products.csv | Built as visuals: category on SP; order source, device type, and payment method on CM; product and geography attributes through the Breakdown Dimension list on SP |
| Promoted Net Sales / Share | Net sales from order lines that carry a primary promotion ID, and their share of net sales | `CALCULATE([Net Sales], KEEPFILTERS(OrderItems[PromoID] <> ""))`; `DIVIDE([Promoted Net Sales], [Net Sales])`. The second promotion ID isn't counted | order_items.csv | Built `Promoted Net Sales` (model only), `Promoted Sales Share`: SP |
| Top 20% Product Sales Share | Share of net sales earned by the top 20% of selling products in the current filters | Products with net sales > 0 only; product count × 0.2 rounded up; `TOPN` keeps ties at the cut-off. Blank when nothing sold | order_items.csv, products.csv | Built `Top 20% Product Sales Share`: SP |

### Growth measures (in `Date.tmdl`)

All growth measures use the order date and compare like-for-like days: each date in context is matched with the same date 12 months earlier (`EDATE(..., -12)`), applied through `TREATAS` after `REMOVEFILTERS('Date')`. Dates whose prior-year date falls before the first order (2012-07-04) are dropped, so the first year returns blank rather than a false +100%. With no date filter the measures default to the latest 12 months of data against the 12 months before. Two hidden helpers, `Data Start Date` and `Data End Date`, return the first and last order date ignoring filters.

| Metric | Formula / logic (as built) | Status |
| --- | --- | --- |
| Net Sales Prior Year | `[Net Sales]` on the prior-year matching dates | Built `Net Sales Prior Year`: EO, SP |
| Net Sales / Gross Profit / Total Orders YoY % | `DIVIDE(Current - Prior, Prior)` on matching dates | Built `Net Sales YoY %` (EO, SP), `Gross Profit YoY %` (SP), `Total Orders YoY %` (SP) |
| Net Sales MoM % | Latest month in context (capped at the last order date) against the calendar month before. Blank when the prior month starts before the first order month | Built `Net Sales MoM %`: EO, SP |

`Date[Year]` (a whole-number column) is the axis for the yearly growth columns. 2012 is blank and 2013 covers only July–December, because those are the only 2013 dates with a 2012 match.

## 2.4. Customers and Market Performance Metrics

| Metric | Definition / requirement | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Distinct Customers | Unique customers with at least one order in context | `DISTINCTCOUNT(Orders[CustomerID])` with `CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH)`, all statuses. `New Customers`, `Repeat Customers`, and the all-time rates build their customer list the same way | orders.csv | Built `Distinct Customers`: CM |
| Revenue per Customer | Net sales per distinct customer | `DIVIDE([Net Sales], [Distinct Customers])` | orders.csv, order_items.csv | Built `Revenue per Customer`: CM |
| Customer Segments | Customers, orders, and net sales by acquisition channel, age group, and gender | Existing measures by `Customers[AcquisitionChannel]`, `[AgeGroup]`, `[Gender]`. "Customers by Acquisition Channel / Age Group" use `Distinct Customers`, so they count customers who ordered in the filtered context | customers.csv, orders.csv | Built as visuals: CM |
| Customer Geography | Net sales by region and city | `[Net Sales]` by `Geography[Region]` through `Orders[Zip]`; the CM "Customer Value by City" table adds customers, orders, order frequency and net sales by `Geography[City]`. No unmatched-ZIP report | geography.csv, orders.csv | Built (partial): CM |
| New Customers | Customers ordering in the selected period whose first-ever order falls in that period | Customers in context (same list as `Distinct Customers`) whose first order date, `CALCULATE(MIN(Orders[OrderDate]), ALLEXCEPT(Orders, Orders[CustomerID]))`, lies between `MIN` and `MAX` of `'Date'[Date]`. The first order is taken across all products, regions, and statuses. It is computed inside the measure; there is no first-order-date column | orders.csv | Built `New Customers`: CM |
| Repeat Customers (period) | Customers ordering in the period whose first-ever order came earlier | Same as `New Customers`, with first order date `< MIN('Date'[Date])`. New + Repeat = `Distinct Customers`. `Repeat Customer Rate` = `DIVIDE([Repeat Customers], [Distinct Customers])`, which follows the period, unlike the all-time rate below | orders.csv | Built `Repeat Customers`, `Repeat Customer Rate`: CM |
| Repeat Rate (all-time) | Share of customers with more than one order across all dates | `All-Time Repeat Customers` (customers with an order count above 1 under `ALL('Date')`) divided by `Distinct Customers` under `ALL('Date')` | orders.csv | Built `All-Time Repeat Customer Rate`: CM (card "Repeat Buyers (All-Time)"); `All-Time Repeat Customers` is its helper. It ignores the period slicer |
| Order Frequency | Orders per customer | `DIVIDE([Total Orders], [Distinct Customers])` | orders.csv | Built `Order Frequency`: CM |
| Signup Cohort | Customers by signup date | – | customers.csv | Not built. `SignupDate` is hidden and linked only to an auto date table |
| Sales by Channel | Buying behaviour by payment method, order source, and device type | `[Average Order Value]` by `Orders[PaymentMethod]`, `[Net Sales]` by `Orders[OrderSource]`, `[Order Frequency]` by `Orders[DeviceType]`, each with `[Total Orders]` in the tooltip | orders.csv, order_items.csv | Built as visuals: CM |
| Customer KPI measures | Card values that fall back to the latest 12 months when no date is filtered (all-time New Customers would equal every customer, and the all-time period repeat rate is blank) | `Active Customers KPI`, `New Customers KPI`, `Repeat Customer Rate KPI` (Orders), `Revenue per Active Customer KPI` (OrderItems); `Repeat Customer Rate Period Average` draws the dashed average line | orders.csv, order_items.csv | Built: CM |

## 2.5. Inventory Performance Metrics

Inventory measures use the product-by-month-end snapshot grain. Stock balances take the latest snapshot in context (`VAR LatestSnapshot = MAX(Inventory[SnapshotDate])`) and never sum across dates. Inventory is related to `Products` and, through `Inventory[SnapshotDate]`, to `Date` (see section 4). The Reporting-period slicer filters it by snapshot date. The Region slicer doesn't reach it. "Current" therefore means the latest snapshot inside the selected period. The model doesn't display the snapshot date it used.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Current Stock | Units on hand at the latest snapshot | Sum of `StockOnHand` where `SnapshotDate = LatestSnapshot` | inventory.csv | Built `Current Stock`: IP |
| Inventory Value | Latest-snapshot stock valued at catalog price | `SUMX` of `StockOnHand * RELATED(Products[Price])` at `LatestSnapshot` | inventory.csv, products.csv | Built `Inventory Value`: IP |
| Inbound Stock / Outbound Stock | Units received / sold in the snapshot records | `SUM(Inventory[UnitsReceived])` / `SUM(Inventory[UnitsSold])` across the rows in context | inventory.csv | Built `Inbound Stock`, `Outbound Stock`: IP |
| Low Stock | A product at or below the low-stock threshold at the latest snapshot. The threshold comes from the Low Stock Threshold parameter; no page has its slicer, so it stays at the default of 20 | `StockOnHand <= [Low Stock Threshold Value]` at `LatestSnapshot`, tested inside `Stock Status` and `Planning Action` (there is no separate low-stock count). Stock on hand never falls below 3 in the data, so no product is ever out of stock | inventory.csv | Built inside `Stock Status`, `Planning Action`: IP. `Products in Snapshot` is the denominator of the IP share measures and appears in the IP category scorecard |
| Stockout Rate | Share of products flagged stockout at the latest snapshot. The flag means at least one stockout day in the snapshot month, not zero stock | `DIVIDE(SUM(Inventory[StockoutFlag]), COUNTROWS(Inventory))` where `SnapshotDate = LatestSnapshot` | inventory.csv | Built `Stockout Rate`: EO, IP (`Stockout Rate Color` marks above-average rows red in the scorecards). `Category Stockout Rate` (same measure with the `Inventory[Stock Risk]` filter removed): Category Tooltip |
| Inventory Turnover | Annualised cost of goods sold over average stock at cost | `DIVIDE(SUMX(Inventory, UnitsSold * RELATED(Products[COGS])), AVERAGEX(VALUES(SnapshotDate), stock at cost)) * DIVIDE(12, snapshot count)`. Stock is averaged across the snapshots in context, never summed. Over all data it is about 1.1×, falling from about 3.1× in 2012 to 0.6–0.7× in 2020–2022 | inventory.csv, products.csv | Built `Inventory Turnover`: EO, IP (`Inventory Turnover Period Average` draws the average line) |
| Stockout Days, Days of Supply, Fill Rate, Reorder Flag | Source values per product and snapshot; Days of Supply is recomputed from units | `Stockout Days (Latest)` and `Fill Rate (Latest)` (average) at the latest snapshot. `Days of Supply` is weighted: `DIVIDE(SUM(StockOnHand), SUM(UnitsSold)) * 30` at `LatestSnapshot`; `Units Sold (Latest Month)` is `SUM(UnitsSold)` there. `ReorderFlag` is 0 on every source row and is not used | inventory.csv | Built: IP |
| Overstock | Products carrying the source `overstock_flag` (column `OverstockFlag`, hidden) at the latest snapshot. About 80% of products are flagged at the last snapshot, and many are also flagged stockout in the same month | `Excess Units` (stock above 90 days of cover: `StockOnHand − 3 × UnitsSold`, floored at 0, flagged products only), `Excess Stock Value` (excess units × list price, as Inventory Value), `Excess Value Share` (÷ `Inventory Value`). In the source, the flag is set exactly when `days_of_supply > 90`, and `days_of_supply = stock_on_hand ÷ units_sold × 30`; so the flag covers a product's whole stock, and the excess measures count only the part above the 90-day line. The snapshot is fixed before the flag filter. `Excess-Stock Products` and `Shortage-Risk Products` count products at the latest snapshot by `Inventory[Stock Risk]`; their `… Label` measures (and `Excess Value Share Label`) are the IP card subtitles, and read `Excess-Stock Product Share` / `Shortage-Risk Product Share` (÷ `Products in Snapshot`) | inventory.csv, products.csv | Built `Excess-Stock Products`, `Shortage-Risk Products`, `Excess Units`, `Excess Stock Value`, `Excess Value Share`: IP. The two `… Product Share` measures are helpers of the labels |
| Cover Band, ABC Class, Planning Action | Planning segmentation (section 6.3.4) | Calculated columns `Inventory[Cover Band]` (+ hidden `Cover Band Order`), `Inventory[Stock Risk]` (+ hidden `Stock Risk Order`) and `Products[ABC Class]`; measures `Planning Action` (+ hidden `Planning Action Color`), `Stock Status` (▼ Low stock / ! Stockout days / OK, + `Stock Status Color`) and `Stock Flow Label` (Units on Hand card subtitle) | inventory.csv, products.csv | Built: IP |

## 2.6. Delivery and Logistics Performance Metrics

Transit time is a measured elapsed duration. It is not, by itself, proof that an order was late. A true late-delivery KPI requires a promised delivery date or an approved service-level target. The 3-day rule below is therefore a diagnostic threshold, and the report labels it "(Diagnostic)".

`Shipments` is related to `Orders` (section 4), and the base shipment measures add `CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH)`. They therefore follow order date, Region, order status, and product filters. No shipment measure uses the ship-date role. Under a product filter, a shipment measure covers every order that contains the product, and `Shipment Cost` is the full order fee, not an allocated share.

| Metric | Definition / requirement | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Shipment Cost | Total recorded shipping fees | `SUM(Shipments[ShippingFee])` | shipments.csv | Built `Shipment Cost`: DL (card, region table) |
| Shipment Records | Distinct orders with a shipment row | `DISTINCTCOUNT(Shipments[OrderID])` | shipments.csv | Built `Shipment Records`: DL |
| Fulfilled Rate | Orders with status delivered, paid, or returned as a share of non-cancelled orders | `DIVIDE(CALCULATE([Total Orders], KEEPFILTERS(Orders[OrderStatus] IN {"delivered", "paid", "returned"})), CALCULATE([Total Orders], KEEPFILTERS(Orders[OrderStatus] <> "cancelled")))` | orders.csv | Built `Fulfilled Rate` (in the `Shipments` table): EO, DL |
| Average Transit Time | Average days from ship to delivery for valid delivered shipments | `AVERAGEX` of `DATEDIFF(ShipDate, DeliveryDate, DAY)` where both dates are present and delivery ≥ ship | shipments.csv | Built `Average Delivery Time`: DL (lead-time line; the city table shows it under the header "Average Transit Time") |
| Transit Within 3 Days | Delivered shipments completed within 3 days (diagnostic) | Shipments with transit ≤ 3 days divided by `Delivered Shipments` (helper: shipments with both dates and delivery ≥ ship) | shipments.csv | Built `Transit Within 3 Days`: EO, DL |
| Late Delivery Rate | Shipments delivered after a promised date | – | needs a promised-date / SLA field | Not built (no promised date in source) |
| Order-to-Delivery Lead Time | Days from order date to delivery, and its first leg (order to ship) | `Average Days to Ship` = `AVERAGE(Shipments[Days to Ship])`. `Average Order-to-Delivery Days` = average of `Days to Ship + Transit Days` over shipments where both are present. Both use the order-level `CROSSFILTER`. `Lead Time Split Label` (DL card subtitle) splits the total into days to ship and days in transit | orders.csv, shipments.csv | Built `Average Order-to-Delivery Days`: EO, DL. `Average Days to Ship`, `Lead Time Split Label`: DL |
| Shipping Cost Ratios | Fee per shipped order, and fees as a share of the net sales of the same orders | `DIVIDE([Shipment Cost], [Shipment Records])`. `Shipment Cost % of Net Sales` divides by `[Net Sales]` restricted (`TREATAS`) to orders with a shipment record. The fee belongs to the whole order, so read the percentage without product filters | shipments.csv, order_items.csv | Built `Shipment Cost per Shipment`, `Shipment Cost % of Net Sales`: DL |
| Premium-Fee Shipment Share | Share of shipments with a fee of 10 or more | `DIVIDE(CALCULATE([Shipment Records], KEEPFILTERS(Shipments[ShippingFee] >= 10)), [Shipment Records])` | shipments.csv | Built: DL |
| Cancellation Rate | Cancelled orders as a share of all orders | `DIVIDE(CALCULATE([Total Orders], KEEPFILTERS(Orders[OrderStatus] = "cancelled")), [Total Orders])`, shown by `Orders[PaymentMethod]` | orders.csv | Built: DL |
| Order Status Share | Share of orders by source order status | `DIVIDE([Total Orders], CALCULATE([Total Orders], ALL(Orders[OrderStatus])))` by `Orders[OrderStatus]` | orders.csv | Built: EO (bar), DL (column by `Date[Year]` × status) |

No measure uses the delivery-date role yet (`Shipments[DeliveryDate]` → `Date` is inactive and unused). The lead-time measures read the calculated columns `Shipments[Transit Days]` and `Shipments[Days to Ship]` and follow the order date.

## 2.7. Returns and Product Quality Metrics

All return measures follow the order-date path through `Returns[OrderID] → Orders`; none uses the return-date role. The page rates are the closed-window variants (see below), so the latest month isn't under-counted.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Returned Orders | Distinct orders with at least one return | `DISTINCTCOUNT(Returns[OrderID])` | returns.csv | Built `Returned Orders`: RQ |
| Return Quantity | Units recorded as returned | `SUM(Returns[ReturnQuantity])` | returns.csv | Built `Return Quantity`: RQ |
| Refund Amount | Recorded refund value | `SUM(Returns[RefundAmount])` | returns.csv | Built `Refund Amount`: RQ, SP, tooltip |
| Return Rate for Product Orders | Return rate with a denominator that respects product filters | `DIVIDE([Returned Orders], DISTINCTCOUNT(OrderItems[OrderID]))` | returns.csv, order_items.csv | Built: EO, RQ (and inside `Return Rate (Min Reviews)` on CS) |
| Return Reason Share | Share of returned orders by reason | Returned orders with `ALL(Returns[ReturnReason])` as the denominator | returns.csv | Built `Return Reason Share`: RQ |
| Return Rate / Refund Rate (Closed Window) | Share of orders with at least one return (all-status denominator), and refunds as a share of net sales, limited to orders that have had time to be returned | `DIVIDE([Returned Orders], [Total Orders])` and `DIVIDE([Refund Amount], [Net Sales])`, each part wrapped in `CALCULATE(..., KEEPFILTERS(Orders[OrderDate] <= [Data End Date] - 31))`; 31 days is the longest order-to-return gap | returns.csv, orders.csv, order_items.csv | Built `Return Rate (Closed Window)`: EO, RQ. `Refund Rate (Closed Window)`: RQ |
| Return Lines | Return records (order × product rows) | `COUNTROWS(Returns)` | returns.csv | Built: RQ |
| Average Days to Return | Days from order to return | `AVERAGE(Returns[Days to Return])`, column `INT(Returns[ReturnDate] - RELATED(Orders[OrderDate]))` | returns.csv, orders.csv | Built: RQ |
| Refund per Returned Order | Average refund per returned order | `DIVIDE([Refund Amount], [Returned Orders])` | returns.csv | Built: RQ |
| Returns by Reason | Share of returned orders by recorded reason | `[Return Reason Share]` by `Returns[Return Reason Label]` | returns.csv | Built as visuals: RQ |
| Returns by Product / Category | Returns by product or category | Existing measures by `Products[Category]` / `[ProductName]` | returns.csv, products.csv | Built as visuals: RQ |

## 2.8. Customer Satisfaction Metrics

The satisfaction trend uses the review date through `Positive Review Rate by Review Date` and `Negative Review Rate by Review Date`; every other review measure follows the order date. Review measures count submitted reviews, not customers.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Average Rating | Mean star rating | `AVERAGE(Reviews[Rating])`; no range validation | reviews.csv | Built `Average Rating`: EO, CS |
| Review Count | Number of submitted reviews | `COUNTROWS(Reviews)` | reviews.csv | Built `Review Count`: CS |
| Positive / Negative Review Rate | Share of reviews rated ≥ 4 / ≤ 2 | Rating-band count divided by `[Review Count]` | reviews.csv | Built `Positive Review Rate`, `Negative Review Rate`: CS |
| Positive / Negative Review Rate by Review Date | Sentiment shares by the date the review was written | `CALCULATE([… Review Rate], USERELATIONSHIP('Date'[Date], Reviews[ReviewDate]), CROSSFILTER('Date'[Date], Orders[OrderDate], NONE))` | reviews.csv | Built: CS (satisfaction trend) |
| Negative Review Rate by Shipment | Negative review rate by shipment attributes | `CALCULATE([Negative Review Rate], CROSSFILTER(Shipments[OrderID], Orders[OrderID], BOTH))` | reviews.csv, shipments.csv | Built: CS ("Negative Review Rate by Transit Days") |
| Rating Share | Share of reviews at each star rating | `DIVIDE([Review Count], CALCULATE([Review Count], REMOVEFILTERS(Reviews[Rating], Reviews[Rating Label])))` | reviews.csv | Built: CS |
| Reviewed Orders / Review Coverage | Delivered orders with at least one review, and their share of delivered orders | `CALCULATE(DISTINCTCOUNT(Reviews[OrderID]), KEEPFILTERS(Orders[OrderStatus] = "delivered"))`; `DIVIDE([Reviewed Orders], [Delivered Orders])` | reviews.csv, orders.csv | Built: CS (`Delivered Orders` is the denominator) |
| Average Days to Review | Days from order to review | `AVERAGE(Reviews[Days to Review])`, column `INT(Reviews[ReviewDate] - RELATED(Orders[OrderDate]))` | reviews.csv, orders.csv | Built: CS |
| Rating by Product / Category | Sentiment and review count by product or category | `[Negative Review Rate]` by `Products[Category]` (CS bar), the CS product table, and average rating in the EO category scorecard | reviews.csv, products.csv | Built as visuals: EO, CS |
| Rating and Return Diagnostic | Rating and return measures side by side | The CS product table (`p7-table-product-rating`) shows the `(Min Reviews)` measures: review count, average rating, positive and negative review rate, and `Return Rate (Min Reviews)`, so products under the threshold show blanks | reviews.csv, returns.csv, products.csv | Built: CS |
| Rating with a Minimum Sample | Rating, sentiment and return measures shown only when the review count reaches the Minimum Reviews parameter (1–50; no page has its slicer, so it stays at 10) | `IF([Review Count] >= [Minimum Reviews Value], [Average Rating])`, and the same test for the other measures (`Return Rate (Min Reviews)` wraps `[Return Rate for Product Orders]`). Blank otherwise, so thinly reviewed products drop out of the table | reviews.csv, returns.csv | Built `Average Rating (Min Reviews)`, `Review Count (Min Reviews)`, `Positive Review Rate (Min Reviews)`, `Negative Review Rate (Min Reviews)`, `Return Rate (Min Reviews)`: CS |
| Rating by Shipment Attribute | Rating and review count with shipment columns (such as Transit Days) allowed to filter reviews through the order | `CALCULATE([Average Rating], CROSSFILTER(Shipments[OrderID], Orders[OrderID], BOTH))`, and the same for `[Review Count]`. Use only on visuals grouped by a `Shipments` column | reviews.csv, shipments.csv | Built `Average Rating by Shipment`, `Review Count by Shipment`: CS, as tooltips on "Negative Review Rate by Transit Days" |

## 2.9. Cross-Domain Diagnostics

Cross-domain views are exploratory comparisons, not causal models. The report has no scatter charts. Cross-domain comparisons appear as side-by-side columns in tables.

| Diagnostic | Requirement / definition | As built | Status |
| --- | --- | --- | --- |
| Sales and Order Trend | Value, volume, and units over equivalent periods | EO shows Net Sales against Net Sales Prior Year by `Date[Month Start]`. On SP the reader picks the metrics (Net Sales, Gross Sales, Gross Profit, COGS, Discount Amount, Refund Amount) from the Trend Metric slicer; with nothing selected the chart shows Gross Profit and Discount Amount. No orders, units, or AOV trend | Built (partial) |
| Stock Availability and Sales Movement | Stock flags alongside units and net sales | IP shows Inbound Stock, Outbound Stock and Units on Hand by month (stock movement section), and stock-risk product counts by month. Stock flags aren't set against net sales | Built (partial) |
| Delivery and Returns | Return outcomes by transit group | Not built for returns. CS "Negative Review Rate by Transit Days" compares satisfaction across transit durations instead | Not built |
| Promotion Performance | Lines with vs without a promotion reference | `Promotions` filters `OrderItems` through `PromoID`. SP shows `Promoted Sales Share` and the table "Sales, Discount, and Margin by Promotion". Lines without a promotion appear as the blank row | Built: SP |
| Rating and Operational Experience | Ratings next to sales, return, and stock measures | The EO category scorecard (`p1-table-category-summary`) puts net sales, net sales YoY, gross margin, inventory turnover, stockout rate, fulfilled rate, order-to-delivery days, product-order return rate and average rating side by side per category, with `Category Watch Flag` (⚠ text) and `Return Rate Color` / `Stockout Rate Color` (red cells) marking the categories to watch. There is no product-level exception table | Built (partial) |

## 2.10. Helper Measures (not bound to any visual)

The `Sales`, `WebTraffic`, and `Payments` tables were removed. Every measure that no visual bound and no other measure referenced was then deleted, so each measure left in the model is either on a page or feeds one that is. The helpers below are not bound to a visual directly:

| Table | Helper | Feeds |
| --- | --- | --- |
| `Date` | `Data Start Date`, `Data End Date` | Growth measures (first-year cut-off, latest-12-months default), closed-window return rates, the data-through label |
| `OrderItems` | `Promoted Net Sales` | `Promoted Sales Share` |
| `Orders` | `All-Time Repeat Customers` | `All-Time Repeat Customer Rate` |
| `Inventory` | `Excess-Stock Product Share`, `Shortage-Risk Product Share` | The `Excess-Stock Product Label` / `Shortage-Risk Product Label` card subtitles |
| `Shipments` | `Delivered Shipments` | `Transit Within 3 Days` |
| `Low Stock Threshold` | `Low Stock Threshold Value` | `Stock Status`, `Planning Action` |
| `Minimum Reviews` | `Minimum Reviews Value` | The `(Min Reviews)` measures |

Deleted as unused (no visual, no reference): every `… vs PY Color` measure and every `… vs PY Label` except `Total Orders vs PY Label` (hidden `hdr-data-through` text) and `Inventory Value vs PY Label` (Inventory Value card); `Gross Sales YoY %` and `Gross Sales YoY Color`; `All-Time First-Time Customer Rate` / `Customers`; `Low-Stock Products`, `Low-Stock Product Share`, `Overstock Products`, `Overstock Product Share`, `Sell-Through Rate`, `Units on Hand Label`; `Shipment Cost by Ship Date`, `Transit Within 3 Days by Ship Date`, `Transit Within Threshold`; `Return Rate`, `Refund Rate`, `Return Rate (Received Orders)`, `Return Quantity by Return Date`, `Refund Amount by Return Date`, `Refund Saved (Scenario)`, `Net Sales After Returns (Scenario)`; `Average Rating by Review Date`, `Review Count by Review Date`; `Transit Day Threshold Value` and `Return Reduction Value`.

Deleted earlier as duplicates after a data check: `Return Amount` (identical to `Refund Amount`), `Average Transit Time` (alias of `Average Delivery Time`), `Product-Linked Shipment Orders` / `Product-Linked Average Transit Time` (equal to the base measures since the order-level `CROSSFILTER`), and `Fulfillment Rate` (delivered ÷ all orders = 79.9%, easily confused with `Fulfilled Rate` = 96.4%; `Delivered Orders` stays because `Review Coverage` uses it). Two measures exist only for the Category Tooltip page: `Selected Category` (`Products`) and `Category Stockout Rate` (`Inventory`).

# 3. PREPARE THE DATA

## 3.1. Source Files and Grains

The following row counts and grains were checked against the CSV files in `data`. A
fact table's row count must not be used as a count of orders, products, or customers
unless its grain supports that interpretation.

| Fact/Dimension | File | Rows | Grain / coverage | Key columns | Information |
| --- | --- | ---: | --- | --- | --- |
| Fact | orders.csv | 646,945 | One row per order; 2012-07-04 to 2022-12-31 | `order_id` | Order date, customer, ZIP, status, payment method, device, and source. |
| Fact | order_items.csv | 714,669 | One row per order-product line; 16 repeated `order_id + product_id` combinations | `order_id`, `product_id` | Quantity, unit price, discount, and two promotion references (only the first is loaded). |
| Fact | shipments.csv | 566,067 | At most one row per shipped order; not all orders have a shipment row | `order_id` | Ship date, delivery date, and shipping fee. |
| Fact | payments.csv | 646,945 | One row per order | `order_id` | Payment method, payment value, and installments. Not loaded into the model. |
| Fact | returns.csv | 39,939 | One row per return record; an order or product can occur more than once | `return_id` | Return date, reason, quantity, and refund amount. |
| Fact | reviews.csv | 113,551 | One row per review; an order, product, or customer can have multiple reviews | `review_id` | Review date, rating, title, and related order/product/customer. |
| Fact | web_traffic.csv | 3,652 | One row per date and traffic source; 2013-01-01 to 2022-12-31 | `date`, `traffic_source` | Sessions, visitors, page views, bounce rate, and session duration. Not loaded into the model. |
| Dimension | customers.csv | 121,930 | One row per customer | `customer_id` | ZIP, city, signup date, gender, age group, and acquisition channel. |
| Fact / benchmark | sales.csv | 3,833 | One row per day; 2012-07-04 to 2022-12-31 | `Date` | Daily Revenue and COGS benchmark, separate from order-line revenue. Not loaded into the model. |
| Fact | inventory.csv | 60,247 | One row per product and month-end snapshot; 126 snapshot dates | `product_id`, `snapshot_date` | Stock, receipt, movement, stockout, supply, fill, and reorder fields. |
| Dimension | geography.csv | 39,948 | One row per ZIP | `zip` | City, region, and district mapping. |
| Dimension | products.csv | 2,412 | One row per SKU | `product_id` | Product attributes, catalog price, and COGS. |
| Dimension | promotions.csv | 50 | One row per promotion | `promo_id` | Promotion dates, type, value, channel, applicability, and minimum order value. |

The model loads 10 of the 13 files. `payments.csv`, `sales.csv`, and `web_traffic.csv` aren't loaded, because no visual used them. Payment method comes from `orders.csv`.

## 3.2. Power BI Preparation (as built)

Each of the 10 CSV tables is an import partition with the same three Power Query steps:

```m
Source = Csv.Document(File.Contents("D:\projects\vintelligence2026_ai_agent_powerbi_dashboard\data\<file>.csv"),
                      [Delimiter = ",", Columns = n, Encoding = 65001, QuoteStyle = QuoteStyle.Csv]),
#"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
#"Changed Type"     = Table.TransformColumnTypes(#"Promoted Headers", { ... })
```

- **Paths are absolute.** There is no shared folder parameter. Moving the repository means editing every partition.
- **No row-level cleaning.** Power Query doesn't filter, deduplicate, or replace values. Blank promotion IDs and the 16 repeated `order_id + product_id` rows in `order_items.csv` are kept as they are.
- **Unused columns are dropped.** Six tables add a fourth step, `#"Removed Unused Columns" = Table.RemoveColumns(#"Changed Type", {…})`, for columns that no visual, measure, or relationship uses: Customers (Zip, City), Inventory (SellThroughRate, year, month), OrderItems (PromoID2), Returns (ReturnID), Reviews (ReviewID, CustomerID, ReviewTitle), and Promotions (DiscountValue, ApplicableCategory, PromoChannel, stackable_flag, min_order_value). To bring one back, take it out of that step's list and add a `column` to the table.
- **Identifier columns** (`order_id`, `customer_id`, `product_id`, `zip`) are typed `Int64`, not text. The promotion ID (`promo_id`) is text.
- **Dates** are typed `date`. Amounts and rates are decimal numbers. Quantities, flags, and ratings are whole numbers. Source amounts aren't rounded.
- **`Date` table** is a calculated table:
  `ADDCOLUMNS(CALENDAR(DATE(2012,1,1), DATE(2022,12,31)), "Year-Month", FORMAT(…, "YYYY-MM"), "Year-Month Sort", Year * 100 + Month, "Month Start", first day of the month, "Year", YEAR([Date]))`. It is marked as a date table (`dataCategory: Time`, `Date[Date]` is the key). `Month Start` is a true date for continuous trend axes; `Year` is used by the yearly columns (refund amount, review coverage, order status share, gross profit growth). `Month Number`, `Month Name`, `Quarter`, and `Week Start` don't exist; add them to the `ADDCOLUMNS` expression if a weekly or quarterly view is needed.
- **Auto date/time is on** (`__PBI_TimeIntelligenceEnabled = 1`). Desktop creates hidden `LocalDateTable_*` tables for date columns without a relationship to `Date`. These are `Customers[SignupDate]`, `Promotions[StartDate]`, and `Promotions[EndDate]`.
- **Hidden columns.** Columns that only measures or relationships use are hidden in report view: keys (`OrderID`, `ProductID`, `CustomerID`, `Zip`, `PromoID`), raw amounts and quantities (for example `Quantity`, `UnitPrice`, `DiscountAmount`, `RefundAmount`, `ShippingFee`, `Price`, `COGS`), the role-playing dates (`OrderDate`, `ShipDate`, `DeliveryDate`, `ReturnDate`, `ReviewDate`), and `Date[Year-Month Sort]`. Every measure has a format string.
- **Calculated columns:** `Shipments[Transit Days]` (ship to delivery) and `Shipments[Days to Ship]` (order to ship, hidden). Both are blank when a date is missing or the dates are out of order.
- **What-if parameter tables** (calculated with `GENERATESERIES`, not related to any table). Each has one column. `Low Stock Threshold` and `Minimum Reviews` also have a `… Value` measure that returns the selected value, or the default when nothing or more than one value is selected. **No page has a slicer for any of them**, so every measure below runs at the default:

  | Table | Range | Default | Used by |
  | --- | --- | --- | --- |
  | `Low Stock Threshold` | 5–100, step 5 | 20 | `Low Stock Threshold Value` → `Stock Status`, `Planning Action` |
  | `Minimum Reviews` | 1–50 | 10 | `Minimum Reviews Value` → `Average Rating (Min Reviews)`, `Review Count (Min Reviews)`, `Positive Review Rate (Min Reviews)`, `Negative Review Rate (Min Reviews)`, `Return Rate (Min Reviews)` |
  | `Transit Day Threshold` | 1–7 | 3 | Nothing (column only; its measures were deleted as unused). `Transit Within 3 Days` hard-codes 3 days |
  | `Return Reduction` | 0–50%, step 5% | 0% | Nothing (column only; the scenario measures were deleted as unused) |

- **Field-parameter tables** (calculated, built with `NAMEOF`). `Trend Metric` switches a chart between Net Sales, Gross Sales, Gross Profit, COGS, Discount Amount, and Refund Amount. `Breakdown Dimension` switches an axis between Category, Segment, Size, Color, Region, City, District, and Promotion Type. Both are on SP, the only page with parameter slicers. Renaming any of those measures or columns means updating the parameter table as well.
- `inventory.csv` is loaded at snapshot grain. Its own product attributes (`ProductName`, `Category`, `Segment`) are kept as columns and are used by some IP visuals.

Rules for future changes: keep blanks blank. Don't coerce invalid values to zero. Don't merge the repeated order-item rows. If `sales.csv` is loaded again, keep it as its own table and don't blend it into order revenue.

## 3.3. Data-Quality Checks

The checks below were run against the CSVs when the model was designed. The report has **no data-quality page or exception visuals**, and measures don't exclude invalid rows. The only exception is the transit measures, which skip missing or negative intervals.

- Declared keys are unique, except the documented order-item composite. `order_id + product_id` isn't a primary key.
- Child keys match their parents: order items, payments, shipments, returns, and reviews match orders. Product references match products. Customer references match customers. Order and customer ZIPs match geography.
- Shipments cover 566,067 of 646,945 orders, so a missing shipment row isn't the same as an undelivered order. Reviews cover a subset of orders and customers.
- A delivery must not precede its shipment. The transit measures enforce this; nothing else does.
- Inventory has 126 monthly snapshot dates. It must not be interpolated into weekly or daily balances.
- The source order statuses are `delivered`, `cancelled`, `returned`, `shipped`, `paid`, and `created`. Statuses are grouped only inside measures: `Delivered Orders` and `Fulfilled Rate`.

# 4. RELATIONSHIPS

All relationships are many-to-one, single direction (dimension filters fact), as defined in `relationships.tmdl`. None is bidirectional in the model. Order-level measures switch `OrderItems[OrderID] → Orders[OrderID]` to both directions inside the measure with `CROSSFILTER(..., BOTH)`.

**Active**

| From (many) | To (one) |
| --- | --- |
| Orders[CustomerID] | Customers[CustomerID] |
| Orders[Zip] | Geography[Zip] |
| OrderItems[OrderID] | Orders[OrderID] |
| OrderItems[ProductID] | Products[ProductID] |
| OrderItems[PromoID] | Promotions[PromoID] |
| Returns[OrderID] | Orders[OrderID] |
| Returns[ProductID] | Products[ProductID] |
| Reviews[OrderID] | Orders[OrderID] |
| Reviews[ProductID] | Products[ProductID] |
| Shipments[OrderID] | Orders[OrderID] |
| Inventory[ProductID] | Products[ProductID] |
| Inventory[SnapshotDate] | Date[Date] |
| Orders[OrderDate] | Date[Date] |

**Inactive** (turned on inside measures with `USERELATIONSHIP`)

| From (many) | To (one) | Used by |
| --- | --- | --- |
| Shipments[ShipDate] | Date[Date] | – |
| Shipments[DeliveryDate] | Date[Date] | – |
| Returns[ReturnDate] | Date[Date] | – |
| Reviews[ReviewDate] | Date[Date] | `Positive Review Rate by Review Date`, `Negative Review Rate by Review Date` |

**Not related:** Customers → Geography (customer ZIP isn't linked), Reviews → Customers (omitted on purpose), and the second promotion reference (`promo_id_2`, filled on 0.03% of lines, isn't loaded). Lines with a blank `PromoID` (61% of lines) fall under the blank promotion.

Only the review-date role is used today. The ship, delivery and return relationships stay in the model for future date-role measures. Date-role measures follow one pattern. They activate the role-playing relationship and switch off the order-date path, so that order date doesn't filter the result as well:

```dax
CALCULATE(
    [Negative Review Rate],
    USERELATIONSHIP('Date'[Date], Reviews[ReviewDate]),
    CROSSFILTER('Date'[Date], Orders[OrderDate], NONE)
)
```

**Resulting filter behaviour** (the four report slicers are `Date[Year-Month]`, `Products[Category]`, `Products[ProductName]`, `Geography[Region]`):

| Slicer | Orders, OrderItems, Returns, Reviews | Inventory | Shipments | Customers |
| --- | --- | --- | --- | --- |
| Reporting period | Yes, by order date | Yes, by snapshot date | Yes, by order date | No |
| Product category / Product | Yes. Order-level measures count the orders that contain the selected products (`CROSSFILTER ... BOTH` in the measure) | Yes | Yes (orders containing the products) | No |
| Region | Yes, through Orders[Zip] | No | Yes, through Orders | No |

Use the appropriate inactive date role explicitly for shipment, delivery, return, and review trends. Don't create a direct Reviews-to-Customers relationship, because Reviews already connect to customers through Orders.

# 5. ASSUMPTIONS AND CAVEATS

- The model reflects the supplied source files and does not invent missing shipments,
  demand, promised delivery dates, or customer attribution.
- `order_items.csv` is the primary transaction-level revenue fact. `sales.csv` is a
  daily Revenue/COGS benchmark and may differ because its calculation basis is not
  established by the source files. It is no longer loaded into the model.
- The source contains no promised delivery date or explicit service-level target.
  A three-day transit split is a diagnostic threshold, not a confirmed late-delivery
  KPI.
- `shipments.csv` is incomplete relative to orders. A missing shipment row means that
  shipment data is unavailable; it must not be classified as pending, cancelled, or
  late without an order-status or service rule.
- Promotion references are sparse (`promo_id_2` is almost entirely blank and isn't loaded) and are not
  guaranteed to identify one promotion per order line. Promotion analysis is
  directional and must retain an unknown/unrecorded group.
- `web_traffic.csv` begins on 2013-01-01 while core commerce and sales data begins on
  2012-07-04. Do not compare traffic and commerce trends before the traffic start date.
  The file isn't loaded into the model.
- Inventory is monthly and includes product attributes in addition to the product
  relationship. Use the Products dimension for shared product filtering and inspect
  attribute mismatches rather than assuming both copies are identical.
- Return refunds are recorded at return-row grain and cannot be treated as same-period
  sales reversals without aligning them to the associated order cohort.
- `reviews.csv` includes `customer_id`, but the column isn't loaded and a direct Reviews-to-Customers relationship is
  intentionally omitted to avoid an ambiguous filter path. Use the order relationship
  or a deliberate bridge only if the model design requires it.

# 6. DASHBOARD DESIGN (AS BUILT)

## 6.1. Report Structure

- **Pages:** 8 in `pageOrder`: 7 analytical pages and the hidden Category Tooltip page (section 6.3.8). Executive Overview is the active (opening) page. The tab names (`displayName`) are:

  | # | Tab name | Page ID | Display option | Canvas |
  | --- | --- | --- | --- | --- |
  | 1 | Executive Overview | `9f7b1d2d-…f712` | Fit to page | 1920×1750 |
  | 2 | Sales & Profitability | `2d466881-…34d5` | Fit to page | 1920×2780 |
  | 3 | Customers & Market | `a1f2c3d4-…5d6e` | Fit to page | 1920×2240 |
  | 4 | Inventory | `5c7bc72d-…68d16b1ac7af` | Fit to page | 1920×3460 |
  | 5 | Shipment | `fcd9a8b8-…183f6a` | Fit to width | 1920×2460 |
  | 6 | Returns & Product Quality | `b2f3c4d5-…5d6e7f` | Fit to width | 1920×2240 |
  | 7 | Customer Satisfaction | `cb86d456-…1ef3` | Fit to width | 1920×2240 |
  | – | Category Tooltip (hidden) | `c3e4d5f6-…6d7e` | Actual size | 320×240 |

  Customers & Market, Returns & Product Quality and Customer Satisfaction are 2240 high because every story step opens with a 40-high section heading and row D adds a search box (y 1756, h 80) above its detail table (y 1842); Shipment is 2460 because its row D stacks a region table, a search box and the city table. They share one storytelling grid (section 6.2).
- **Theme:** `purple-slicer`, registered as both the base theme and the custom theme. Colours come from the theme, not from per-visual formatting.
- **Navigation:** page tabs only. There is no drill-through page, no page-navigator buttons and no bookmarks.
- **Tooltips:** the six category bar charts (`p2-bar-yoy-category`, `p2-bar-discount-rate` (SP), `p4-bar-risk-category` (IP), `p5-bar-cost-category` (Shipment), `p6-bar-refund-category` (RQ), `p7-bar-rating-category` (CS)) use the Category Tooltip page (section 6.3.8). Every other visual shows the default tooltip plus any extra tooltip fields listed in section 6.3.
- **Parameters:** the only parameter slicers are the two field-parameter slicers on Sales & Profitability (Trend Metric, Breakdown Dimension). No page has a what-if slicer, so Low Stock Threshold and Minimum Reviews stay at their defaults (20 and 10). The Transit Day Threshold and Return Reduction tables are column-only and feed no measure.
- **Reset filters:** every page has one (`hdr-reset-filters`, *Clear all slicers*), next to `hdr-data-through`.
- **Alt text:** set on every visual except the shared header and slicers. It describes the chart type, the measure, and the dimension.
- **Subtitles:** set on every chart and table except the header and slicers, in grey 10 pt. Some KPI cards have no subtitle (section 6.3 lists them per page). Trend axes use `Date[Month Start]`, a true date, so the axis is continuous. Each one states the date basis: "order date" on almost every visual, "review date" on `p7-line-sentiment-month` only, and "Snapshot month" or "Latest snapshot" plus "Region not applied" on the inventory visuals. No visual or measure uses ship date or return date. Wherever the 3-day transit share appears it is labelled diagnostic, not an SLA.
- **Report-level filters:** none.
- **Search boxes:** every detail table has a search slicer just above it (`…-search-<field>`: a dropdown copied from `filter-3-productname` with `selfFilterEnabled` on, no sync group, w 420 h 80). Type part of a name, tick the match, and only that table filters: the page's `visualInteractions` give each search slicer **NoFilter** on every other visual. Add the same entries when adding a visual to a page with a search box.
- **Not built from the original design:** a Diagnostics page, a Data Quality page, and scatter-chart cross-domain views.

## 6.2. Shared Layout

Every analytical page uses the same grid. The x positions run from 40 to 1880.

| Row | y | Height | Contents |
| --- | --- | --- | --- |
| Header | 16 | – | `hdr-band` (background band), `hdr-brand`, `hdr-title` (page title at 22pt and a one-line purpose at 10pt, no padding, so both lines fit the 64-high box) |
| Slicers | 110 | – | Four dropdown slicers, left to right (below) |
| KPI cards | 254 | 132 | 6 cards 290 wide at x = 40, 350, 660, 970, 1280, 1590, named `kpi-*` (Customers & Market has 5 cards 352 wide). A second row at y = 406 on Executive Overview (5 × 352), Sales & Profitability (6 × 290) and Inventory (3 × 600) |

Executive Overview and Sales and Profitability use their own storytelling grid below the cards (sections 6.3.1 and 6.3.2). The domain pages all follow the same four-step story, so a reader always finds the same kind of answer in the same place. Every page below the KPI cards is split into numbered sections, each opened by a section heading: a text box 1840×40 at x = 40 (copied from `p4-section-service`, named `p<N>-section-<subject>`) with "N · Title" in Segoe UI Semibold 15 pt navy and the question the section answers in 11 pt grey. The heading sits 10 below the previous section and its visuals start 50 below the heading. On the four domain pages the headings are at y = 416, 896, 1296 and 1706:

| Row | y | Height | Story step | Typical layout |
| --- | --- | --- | --- | --- |
| A | 466 | 400 | **What is happening over time?** The main trend | Trend line (w 1220) + a supporting trend or yearly column (w 600). Shipment: 900 + 450 + 450 |
| B | 946 | 320 | **Why / what drives it?** Drivers and mix | Three visuals 600 wide at x = 40, 660, 1280. Shipment: 1220 + 600 |
| C | 1346 | 330 | **Where to act?** Category or region comparison | Category bar (w 600) + region table (w 1220) on Returns and Customer Satisfaction; 1220 + 600 on Shipment; four 445-wide bars on Customers & Market |
| D | 1756 | 80 + 380 | **Which records?** Detail at the bottom | Search box (y 1756, h 80), then the full-width detail table (y 1842, h 380), ending at y = 2222. Shipment: full-width region table (1756, h 180), search box (1966), city table (2052, h 380); page height 2460 |

Gaps between visuals are 20. Customers & Market's four row C bars sit at x = 40, 505, 970, 1435.

**Slicers.** All four are dropdowns. Each has its own sync group, so a selection carries across all seven pages.

| Visual | Title | Field |
| --- | --- | --- |
| `filter-1-year-month` | Reporting period | `Date[Year-Month]` |
| `filter-2-category` | Product category | `Products[Category]` |
| `filter-3-productname` | Product | `Products[ProductName]` |
| `filter-4-region` | Region | `Geography[Region]` |

Section 4 shows which tables each slicer reaches.

**Naming.** Every visual folder name matches its content (cards `kpi-<subject>`, other visuals `p<N>-<type>-<subject>`). `N` is the page's position in `pageOrder` (p1 Executive Overview … p7 Customer Satisfaction); tooltip-page visuals use `tt-`. The section-heading template `p4-section-service` lives on Inventory. The tables below give each folder name next to its displayed title.

## 6.3. Pages

### 6.3.1. Executive Overview (`9f7b1d2d-…f712`)

Purpose: totals only, with no product-level detail. The page reads top to bottom as a story in three labelled sections (1 sales performance, 2 operations health, 3 category scorecard): how big is the business, is it growing and how much comes back → how hard is stock working and is the network delivering → how sales compare with last year → how turnover, stock availability and the order mix move → which category needs attention.

Header extras (every page): `hdr-reset-filters` is a *Clear all slicers* button (actionButton, 1150, 28, 150×40) and `hdr-data-through` is a transparent card showing `Data Through Label` (1316, 24, 176×48).

Card subtitles: Net Sales, Net Sales YoY and Net Sales MoM have text subtitles (definition and date basis), and Transit Within 3 Days says "Diagnostic only, no promised date in the data (order date)". Gross Margin, Total Orders, Return Rate, Inventory Turnover, Stockout Rate, Fulfilled Rate and Avg Order-to-Delivery Days have **no subtitle** (a Desktop save removed their `… vs PY Label` bindings, and the unbound label measures have since been deleted).

Reading aids: `Net Sales YoY %` and `Net Sales MoM %` use the format string `▲ 0.0%;▼ 0.0%;0.0%`, so the sign never depends on colour, and are coloured green when up and red when down (`Net Sales YoY Color`, `Net Sales MoM Color`). The scorecard's Watch column and the red Stockout/Return cells say when a category value is a problem.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI 1 (254, 132) | `kpi-net-sales`, `kpi-net-sales-yoy`, `kpi-net-sales-mom`, `kpi-gross-margin`, `kpi-total-orders`, `kpi-return-rate` | Cards (6, w 290) | Net Sales · Net Sales Growth YoY · Net Sales Growth MoM · Gross Margin · Total Orders · Return Rate | Measures of the same names (order date), except Return Rate, which is `Return Rate (Closed Window)` like the Returns page. Gross Sales is on the Sales page only |
| KPI 2 (406, 132) | `kpi-inventory-turnover`, `kpi-stockout-rate`, `kpi-fulfilled-rate`, `kpi-order-to-delivery-days`, `kpi-transit-3-days` | Cards (5, w 352) | Inventory Turnover (x per year) · Stockout Rate · Fulfilled Rate · Avg Order-to-Delivery Days · Transit Within 3 Days (diagnostic) | Inventory cards use the snapshot date and ignore Region. The 3-day card is labelled diagnostic, not an SLA |
| Section labels (568, 1048, 1428; h 40) | `p1-section-sales-performance`, `p1-section-operations-health`, `p1-section-category-scorecard` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Sales performance · 2 · Operations health · 3 · Category scorecard | Heading plus the question the section answers; alt text set |
| 1 Sales (618, 400) | `p1-line-net-sales-vs-py` | Line (w 1220) | Net Sales vs Prior Year by Month | `Net Sales` (solid navy) and `Net Sales Prior Year` (dashed grey) by `Date[Month Start]` |
| 1 Sales | `p1-col-net-sales-yoy` | Column (w 600) | Net Sales YoY % by Year | `Net Sales YoY %` by `Date[Year]`, labels with ▲/▼, bars coloured by `Net Sales YoY Color` (green up, red down), value axis hidden. 2012 blank, 2013 is July–December only |
| 2 Operations (1098, 300) | `p1-line-inventory-turnover` | Line | Inventory Turnover by Month | `Inventory Turnover` and `Inventory Turnover Period Average` (dashed) by `Date[Month Start]` (snapshot date) |
| 2 Operations | `p1-line-stockout-rate` | Line | Stockout Rate by Month | `Stockout Rate` by `Date[Month Start]` (snapshot date). Delivery lead time by month is on the Delivery page |
| 2 Operations | `p1-bar-orderstatus` | Bar | Order Share by Status | `Order Status Share` by `Orders[OrderStatus]`, sorted descending, `Total Orders` in the tooltip |
| 3 Scorecard (1478, 252) | `p1-table-category-summary` | Table (w 1840) | Category Scorecard by Category | Category; Watch (`Category Watch Flag`: ⚠ names falling sales, stockouts or returns above the all-category rate, else ✓); Net Sales (data bars), Net Sales YoY % (▲ green / ▼ red), Gross Margin %, Inventory Turnover, Stockout Rate and Return Rate for Product Orders (red when above the all-category rate), Fulfilled Rate, Average Order-to-Delivery Days, Average Rating. Fixed column widths |

### 6.3.2. Sales & Profitability (`2d466881-…34d5`)

Purpose: revenue, growth, discounting, and margin (order date), told in three labelled sections (1 sales trend and growth, 2 category and mix, 3 product and promotion detail): size → growth → trend against last year → which categories drive growth → what drives sales and discounts → profitability and promotion detail.

Header extras: the same `hdr-reset-filters` and `hdr-data-through` as Executive Overview, at the same positions.

Reading aids: the four growth cards use the format `▲ 0.0%;▼ 0.0%;0.0%` and colour the value by direction, green up and red down (`Net Sales YoY Color`, `Net Sales MoM Color`, `Gross Profit YoY Color`, `Total Orders YoY Color`). Gross Sales YoY was dropped as a card because it repeats Net Sales YoY; the Discount Rate card explains the gap between gross and net instead. Every card except Gross Margin % has a text subtitle giving its definition and date basis; Gross Margin % has none. Sales currency measures show whole dollars (`$#,0`); Average Order Value and Revenue per Customer keep cents. Negative gross profit is red (`Gross Profit Color`) and also shown in parentheses or with a minus sign.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI 1 (254, 132) | `kpi-net-sales`, `kpi-gross-sales`, `kpi-total-orders`, `kpi-aov`, `kpi-promoted-sales-share`, `kpi-top20-product-share` | Cards (6) | Net Sales · Gross Sales · Total Orders · Average Order Value · Promoted Sales Share · Top 20% Product Sales Share | Measures of the same names |
| KPI 2 (406, 132) | `kpi-net-sales-yoy`, `kpi-net-sales-mom`, `kpi-discount-rate`, `kpi-gross-profit-yoy`, `kpi-total-orders-yoy`, `kpi-gross-margin` | Cards (6) | Net Sales Growth YoY · Net Sales Growth MoM · Discount Rate · Gross Profit Growth YoY · Order Growth YoY · Gross Margin | Growth measures from `Date.tmdl`, `Discount Rate` (discount ÷ gross sales) and `Gross Margin %` |
| Section labels (568, 1448, 2188; h 40) | `p2-section-sales-trend`, `p2-section-category-mix`, `p2-section-product-promotion` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Sales trend and growth · 2 · Category and mix · 3 · Product and promotion detail | Heading plus the question the section answers; alt text set |
| 1 Trend (618, 400) | `p2-line-net-sales-vs-py` | Line (w 1220) | Net Sales vs Prior Year by Month | `Net Sales` (solid navy) and `Net Sales Prior Year` (dashed grey) by `Date[Month Start]` |
| 1 Trend | `p2-col-growth-yoy` | Column (w 600) | Net Sales, Gross Profit and Order Growth YoY by Year | `Net Sales YoY %` (navy), `Gross Profit YoY %` (teal), `Total Orders YoY %` (grey) by `Date[Year]`. Legend on top, no data labels (values in the tooltip) |
| 1 Trend (1038, 380) | `p2-trend-net-sales` | Line (w 1540) | Selected Sales Metrics by Month | The metrics picked in the Trend Metric list, by `Date[Month Start]`. Opens on Gross Profit and Discount Amount (Net Sales already has its own trend above). Each metric keeps a fixed colour |
| 1 Trend | `p2-slicer-trend-metric` | Slicer (list, multi-select) | Trend Metric | Field parameter `Trend Metric`: Net Sales, Gross Sales, Gross Profit, COGS, Discount Amount, Refund Amount |
| 2 Mix (1498, 320) | `p2-bar-yoy-category` | Bar (w 600) | Net Sales YoY % by Category | `Net Sales YoY %` by Category (▲/▼ labels, bars coloured by `Net Sales YoY Color`, value axis hidden) |
| 2 Mix | `p2-table-category` | Table (w 1220) | Net Sales, Profit and Promotion Mix by Category | Category; Net Sales (data bars), Gross Profit, Gross Margin %, Units Sold, Average Order Value, Promoted Sales Share. Sorted by Net Sales descending. YoY sits in the bar beside it, so the table carries no growth columns |
| 2 Mix (1838, 320) | `p2-bar-breakdown` | Bar (w 920) | Net Sales by Selected Dimension | `Net Sales` by the chosen dimension, value axis hidden (data labels). Opens on Segment |
| 2 Mix | `p2-slicer-breakdown` | Slicer (list, single select) | Breakdown Dimension | Field parameter `Breakdown Dimension`: Category, Segment, Size, Color, Region, City, District, Promotion Type. Opens on Segment |
| 2 Mix | `p2-bar-discount-rate` | Bar (w 600) | Discount Rate by Category | `Discount Rate` by Category, value axis hidden (data labels) |
| 3 Detail (2238, 80) | `p2-search-product`, `p2-search-promotion` | Search slicers (w 420) | Search product (filters the profitability table) · Search promotion (filters the promotion table) | `Products[ProductName]` and `Promotions[PromoName]`, each filtering only the table below it |
| 3 Detail (2324, 434) | `p2-detail-profitability` | Table (w 1005) | Catalog-Cost Profitability Estimate (Not Historical Cost) | ProductName, Category; Gross Sales, Discount Amount, Net Sales, COGS, Gross Profit, Gross Margin % |
| 3 Detail | `p2-table-promotion` | Table (w 815) | Sales, Discount, and Margin by Promotion | PromoType, PromoName; Net Sales, Discount Amount, Discount Rate, Units Sold, Gross Margin %. Sorted by Net Sales. The blank row is lines with no promotion |

The page is 2780 high and ends at y = 2758.

### 6.3.3. Customers & Market (`a1f2c3d4-…5d6e`)

Story: how big is the customer base and are we keeping customers → who the customers are → where and how they buy → detail. Order date throughout.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI (254, 132) | `kpi-active-customers`, `kpi-new-customers`, `kpi-repeat-rate`, `kpi-revenue-per-customer`, `kpi-repeat-rate-alltime` | Cards (5, w 352, same x grid as Executive Overview row 2) | Active Customers · New Customers · Repeat Customer Rate · Revenue per Active Customer · Repeat Buyers (All-Time) | `Active Customers KPI`, `New Customers KPI`, `Repeat Customer Rate KPI`, `Revenue per Active Customer KPI`, `All-Time Repeat Customer Rate`. Active Customers, New Customers and Repeat Customer Rate have no subtitle; Revenue per Active Customer and Repeat Buyers (All-Time) have text subtitles (definition, period fallback) |
| Section labels (416, 896, 1296, 1706; h 40) | `p3-section-customer-base`, `p3-section-customer-profile`, `p3-section-market-buying`, `p3-section-city-detail` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Customer base trend · 2 · Who our customers are · 3 · Market and buying behaviour · 4 · City detail | Heading plus the question the section answers; alt text set |
| A (466, 400) | `p3-line-new-repeat` | Line (w 1220) | New and Repeat Customers by Month | `New Customers` (navy), `Repeat Customers` (teal) by `Date[Month Start]`, legend on top |
| A | `p3-line-repeat-rate` | Line (w 600) | Repeat Customer Rate by Month | `Repeat Customer Rate` (teal) and `Repeat Customer Rate Period Average` (dashed grey) by `Date[Month Start]` |
| B (946, 320) — who | `p3-col-channel` | Column | Customers by Acquisition Channel | `Distinct Customers` by `Customers[AcquisitionChannel]`, sorted descending, value axis hidden |
| B | `p3-bar-age` | Bar | Customers by Age Group | `Distinct Customers` by `Customers[AgeGroup]`, sorted by age group (18-24 first), value axis hidden |
| B | `p3-bar-segment` | Bar | Revenue per Customer by Product Segment | `Revenue per Customer` by `Products[Segment]`, sorted descending, value axis hidden |
| C (1346, 330) — where and how | `p3-bar-region-sales` | Bar (w 445) | Net Sales by Region | `Net Sales` by `Geography[Region]` |
| C | `p3-bar-payment` | Bar (w 445) | Average Order Value by Payment Method | `Average Order Value` by `Orders[PaymentMethod]`, `Total Orders` in the tooltip |
| C | `p3-bar-source` | Bar (w 445) | Net Sales by Order Source | `Net Sales` by `Orders[OrderSource]`, `Total Orders` in the tooltip |
| C | `p3-bar-device` | Bar (w 445) | Order Frequency by Device Type | `Order Frequency` by `Orders[DeviceType]`, `Total Orders` in the tooltip. Each row C bar answers a different question (where the money is, basket size, channel, loyalty) and is sorted descending with the value axis hidden (data labels) |
| D (1756, 80) | `p3-search-city` | Search slicer (w 420) | Search city (filters the table below) | `Geography[City]` |
| D (1842, 380) | `p3-detail-city` | Table (w 1840) | Customer Value by City | Region, City; Distinct Customers, Total Orders, Order Frequency, Net Sales, Revenue per Customer, Average Order Value. Sorted by Net Sales descending. Channel and age group are already charted in row B, so the table goes to city level instead |

Header extras: the same `hdr-reset-filters` and `hdr-data-through` as Executive Overview, at the same positions.

Reading aids: with no period selected, a customer count over all dates makes New Customers equal every customer and leaves the repeat rate blank (nobody ordered before the first date). So the four period cards use `… KPI` measures that fall back to the latest 12 months of data when `Date` is unfiltered; the charts and the table use the base measures and follow the slicer as is. First-time buyers (all-time) are the complement of Repeat Buyers (All-Time), so they have no card or measure of their own.

### 6.3.4. Inventory (`5c7bc72d-…68d16b1ac7af`)

Story (planner's view), in four labelled sections: 1 service level — are we filling demand (fill rate, stockout rate) and is the A class served best → 2 stock movement — units in, units out and units on hand by month, how fast stock turns, and how inventory value spreads across cover bands → 3 stock status and risk — how many products are at shortage risk or carry excess, over time and by category → 4 detail — category health scorecard, value by ABC class and cover band, and one planning action per product. Inventory follows the snapshot date and ignores Region (section 2.5).

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI row 1 (254, 132) | `kpi-fill-rate`, `kpi-stockout-rate`, `kpi-units-on-hand`, `kpi-inventory-value`, `kpi-days-of-supply`, `kpi-inventory-turnover` | Cards (6, w 290) | Fill Rate · Stockout Rate · Units on Hand · Inventory Value · Days of Supply · Inventory Turnover | Latest snapshot (turnover over the snapshots in the period). Fill Rate, Stockout Rate, Days of Supply and Turnover have no subtitle. Units on Hand (`Current Stock`) shows `Stock Flow Label` (units in / out over the period); Inventory Value shows `Inventory Value vs PY Label` in neutral grey |
| KPI row 2 (406, 132) | `kpi-shortage-risk`, `kpi-excess-products`, `kpi-excess-value` | Cards (3, w 600) | Products at Shortage Risk · Products with Excess Stock · Excess Stock Value | Latest snapshot. Subtitles `Shortage-Risk Product Label`, `Excess-Stock Product Label` (share of stocked products plus the rule) and `Excess Value Share Label` |
| Section labels (568, 1008, 1868, 2328; h 40) | `p4-section-service`, `p4-section-movement`, `p4-section-risk`, `p4-section-detail` | Text boxes (w 1840) | 1 · Service level · 2 · Stock movement · 3 · Stock status and risk · 4 · Category and product detail | Heading plus the question the section answers; alt text set |
| 1 Service (618, 360) | `p4-line-stock-health` | Line (x 40, w 1220) | Fill Rate and Stockout Rate by Month | `Fill Rate (Latest)` (soft teal), `Stockout Rate` (soft coral) by `Date[Month Start]` (snapshot month), legend on top |
| 1 Service | `p4-bar-service-abc` | Clustered bar (x 1280, w 600) | Fill Rate and Stockout Rate by ABC Class | `Fill Rate (Latest)` (soft teal), `Stockout Rate` (soft coral) by `Products[ABC Class]`, data labels on; tooltip `Products in Snapshot` |
| 2 Movement (1058, 360) | `p4-line-stock-flow` | Line (x 40, w 1220) | Inbound, Outbound and Units on Hand by Month | `Inbound Stock` (soft blue), `Outbound Stock` (soft teal) on the left axis and `Current Stock` (soft amber) on the secondary axis, by `Date[Month Start]` (snapshot month) |
| 2 Movement | `p4-line-turnover` | Line (x 1280, w 600) | Inventory Turnover by Month | `Inventory Turnover` (soft teal) and `Inventory Turnover Period Average` (dashed grey) by `Date[Month Start]`, legend on top |
| 2 Movement (1438, 400) | `p4-col-value-cover-band` | Stacked column (w 1840) | Inventory Value by Month and Cover Band | `Inventory Value` by `Date[Month Start]` (snapshot month), stacked by `Inventory[Cover Band]` (Under 30 days coral, 30–90 days soft teal, 90–180 days pale amber, 180–365 days soft orange, Over 365 days muted brown; the legend names each band) |
| 3 Risk (1918, 380) | `p4-col-risk-month` | Stacked column (x 40, w 1220) | Products by Month and Stock Risk | `Products in Snapshot` by `Date[Month Start]`, stacked by `Inventory[Stock Risk]` (Shortage risk coral, Excess stock soft orange, Healthy soft teal; legend names each status) |
| 3 Risk | `p4-bar-risk-category` | Stacked bar (x 1280, w 600) | Products by Category and Stock Risk | `Products in Snapshot` by `Products[Category]`, stacked by `Inventory[Stock Risk]`, sorted by products descending. Its subtitle invites the reader to hover for the Category Tooltip (section 6.3.8) |
| 4 Detail (2378, 400) | `p4-table-category` | Table (x 40, w 1220) | Inventory Health by Category | Category; Inventory Value (data bars), Days of Supply, Inventory Turnover, Fill Rate (Latest), Stockout Rate (red when above the all-category rate, `Stockout Rate Color`), Excess Value Share. Sorted by Inventory Value descending |
| 4 Detail | `p4-bar-abc-cover-band` | Stacked bar (x 1280, w 600) | Inventory Value by ABC Class and Cover Band | `Inventory Value` by `Products[ABC Class]`, stacked by `Inventory[Cover Band]` (same band colours); tooltips `Products in Snapshot`, `Days of Supply` |
| 4 Detail (2798, 80) | `p4-search-product` | Search slicer (x 40, w 420, above the table) | Search product | `Inventory[ProductName]` |
| 4 Detail (2898, 520) | `p4-detail-stock` | Table (w 1840) | Planning Actions by Product | Category, ProductName, ABC Class; `Planning Action` (▲ Replenish / ▼ Reduce excess / ● Healthy, coloured by `Planning Action Color`), `Stock Status`, Current Stock, `Units Sold (Latest Month)`, `Days of Supply`, `Stockout Days (Latest)`, `Fill Rate (Latest)`, `Excess Units`, `Excess Stock Value` (data bars). One row per product at its latest snapshot, sorted by Excess Stock Value descending |

Cover bands, ABC classes and planning actions: `Inventory[Cover Band]` buckets each snapshot row by the source `DaysOfSupply` (Under 30, 30–90, 90–180, 180–365, Over 365 days; blank when the source is blank), sorted by the hidden `Cover Band Order`. `Inventory[Stock Risk]` (sorted by hidden `Stock Risk Order`) applies the Planning Action rule per snapshot row: Shortage risk at ≤ 20 units or under 30 days of cover (`StockOnHand ÷ UnitsSold × 30`), Excess stock when overstock-flagged and `StockOnHand > 3 × UnitsSold`, otherwise Healthy; `Shortage-Risk Products` / `Excess-Stock Products` count products by it at the latest snapshot (0, not blank, when none) and `… Product Share` divides by `Products in Snapshot`. `Products[ABC Class]` ranks products by usage at cost (`UnitsSold × COGS`) over the last 12 snapshot months: A is the first 80% of cumulative usage, B the next 15%, C the rest, and *No recent sales* has no usage. `Planning Action` is ▲ Replenish when stock is at or below the low-stock threshold (20 units) or covers under 30 days (a product with no units sold has no cover and is not counted as short), ▼ Reduce excess when `Excess Units` > 0, otherwise ● Healthy; it is blank above product level. The low-stock cards and the low-stock threshold slicer were removed; `Low Stock Threshold Value` uses its default of 20 units.

Chart palette: the charts on this page use a softer palette set per visual (teal #5FA89D, coral #E07B6F, soft orange #F2A766, pale amber #F3CF7A, muted brown #A8786C, soft blue #7096C8, amber #D4A24C, grey #98A2B3) instead of the saturated status colours; cards and tables keep the theme colours. Every colour is still paired with a legend or label.

Header extras: the same `hdr-reset-filters` and `hdr-data-through` as Executive Overview, at the same positions.

### 6.3.5. Shipment (`fcd9a8b8-…183f6a`)

Story (shipping and fulfilment manager's view), in four labelled sections: 1 shipping cost — is cost per shipment falling, and what drives the fee (basket size, category) → 2 order fulfilment — how the order-status mix moves year by year and which payment methods cancel most → 3 delivery speed — the lead-time trend and the spread of transit days → 4 region and city detail. Speed is flat in this data (about 1.5 days to ship + 4.5 in transit, the same in every region and year), so the page leads with cost and fulfilment, where the variation is. The 3-day transit share stays only as a diagnostic column in the city table.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI (254, 132) | `kpi-shipment-cost`, `kpi-cost-per-shipment`, `kpi-premium-fee-share`, `kpi-fulfillment-rate`, `kpi-cancellation-rate`, `kpi-order-to-delivery-days` | Cards (6, w 290) | Shipment Cost · Shipment Cost per Shipment · Premium-Fee Shipments · Fulfilled Rate · Cancellation Rate · Average Order-to-Delivery Days | Measures of the same names (order date); Fulfilled Rate is the same measure as on Executive Overview (folder `kpi-fulfillment-rate`, no subtitle). The order-to-delivery card subtitle is `Lead Time Split Label` (days to ship + days in transit); the other four cards have text subtitles |
| Section labels (416, 896, 1296, 1706; h 40) | `p5-section-shipping-cost`, `p5-section-fulfilment`, `p5-section-delivery-speed`, `p5-section-region-city` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Shipping cost · 2 · Order fulfilment · 3 · Delivery speed · 4 · Region and city detail | Heading plus the question the section answers; alt text set |
| 1 Cost (466, 400) | `p5-line-cost-premium` | Line (x 40, w 900) | Cost per Shipment and Premium-Fee Share by Month | `Shipment Cost per Shipment` (soft blue) and, on the secondary axis, `Premium-Fee Shipment Share` (soft orange) by `Date[Month Start]` (order date) |
| 1 Cost | `p5-bar-cost-basket` | Bar (x 960, w 450) | Cost per Shipment by Basket Size | `Shipment Cost per Shipment` by `Orders[Basket Size]` (1 unit … 11+ units, sorted by `Basket Size Order`); tooltips `Shipment Records`, `Premium-Fee Shipment Share` |
| 1 Cost | `p5-bar-cost-category` | Bar (x 1430, w 450) | Cost per Shipment by Category | `Shipment Cost per Shipment` by `Products[Category]` (orders containing the category), highest first |
| 2 Fulfilment (946, 320) | `p5-col-status-year` | 100% stacked column (x 40, w 1220) | Order Status Share by Year | `Order Status Share` by `Date[Year]`, stacked by `Orders[OrderStatus]` (delivered teal, shipped soft blue, paid pale amber, created soft orange, returned muted brown, cancelled coral; the legend names each status); tooltip `Total Orders` |
| 2 Fulfilment | `p5-bar-cancel-payment` | Bar (x 1280, w 600) | Cancellation Rate by Payment Method | `Cancellation Rate` by `Orders[PaymentMethod]`, highest first (coral); tooltip `Total Orders` |
| 3 Speed (1346, 330) | `p5-line-lead-time` | Line (x 40, w 1220) | Delivery Lead Time by Month | `Average Days to Ship`, `Average Delivery Time`, `Average Order-to-Delivery Days` by `Date[Month Start]` (order date) |
| 3 Speed | `p5-col-transit-days` | Column (x 1280, w 600) | Shipments by Transit Days | `Shipment Records` by `Shipments[Transit Days]` |
| 4 Detail (1756, 180) | `p5-table-cost-region` | Table (w 1840) | Shipping Cost and Fulfilment by Region | Region; Shipment Records, Shipment Cost, Shipment Cost per Shipment, Premium-Fee Shipment Share, Shipment Cost % of Net Sales, Fulfilled Rate, Cancellation Rate, Average Order-to-Delivery Days |
| 4 Detail (1966, 80) | `p5-search-city` | Search slicer (w 420) | Search city (filters the table below) | `Geography[City]` |
| 4 Detail (2052, 380) | `p5-detail-city` | Table (w 1840) | Delivery Lead Time by City | Region, City; Shipment Records, Shipment Cost per Shipment, Premium-Fee Shipment Share, Average Days to Ship, Average Transit Time (column header on `Average Delivery Time`), Average Order-to-Delivery Days, Transit Within 3 Days (diagnostic) |

Every visual uses order date. Shipment measures respond to Region, status, and product through the order (section 4). `Premium-Fee Shipment Share` counts shipments with `ShippingFee >= 10`: fees fall into two tiers (about $0–3 and $20–32), so $10 separates them. `Cancellation Rate` is cancelled orders ÷ `Total Orders`. `Orders[Basket Size]` bands the units in each order (sum of `OrderItems[Quantity]`, computed in the hidden `Basket Size Order` with `ALLEXCEPT(Orders, Orders[OrderID])` to avoid a circular dependency). No measure uses the ship-date role, and the `Transit Day Threshold` table feeds no measure.

### 6.3.6. Returns & Product Quality (`b2f3c4d5-…5d6e7f`)

Story (returns manager's view), in four labelled sections: 1 return and refund trend — are the return rate and refund rate moving, and how big is the refund bill each year → 2 why products come back — the reason mix, how many days after the order returns arrive, and which payment methods return most → 3 where refunds concentrate — category and region → 4 product detail. Returns arrive 5–31 days after the order (mean about 18), so a plain return rate under-counts the latest month; the page's rates therefore use `… (Closed Window)` measures that only count orders placed at least 31 days before the data end date.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI (254, 132) | `kpi-return-rate`, `kpi-returned-orders`, `kpi-refund-amount`, `kpi-refund-per-return`, `kpi-refund-rate`, `kpi-avg-days-to-return` | Cards (6, w 290) | Return Rate · Returned Orders · Refund Amount · Refund per Returned Order · Refund Rate · Average Days to Return | `Return Rate (Closed Window)`, `Returned Orders`, `Refund Amount`, `Refund per Returned Order`, `Refund Rate (Closed Window)`, `Average Days to Return` (order date). Each card has a short text subtitle with its definition and "· order date" |
| Section labels (416, 896, 1296, 1706; h 40) | `p6-section-return-refund`, `p6-section-why-products`, `p6-section-where-returns`, `p6-section-product-detail` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Return and refund trend · 2 · Why products come back · 3 · Where refunds concentrate · 4 · Product detail | Heading plus the question the section answers; alt text set |
| 1 Trend (466, 400) | `p6-line-return-refund-rate` | Line (x 40, w 1220) | Return Rate and Refund Rate by Month | `Return Rate (Closed Window)` (coral) and `Refund Rate (Closed Window)` (soft blue) by `Date[Month Start]` (order date) |
| 1 Trend | `p6-col-refund-year` | Column (x 1280, w 600) | Refund Amount by Year | `Refund Amount` by `Date[Year]` (muted brown); tooltips `Returned Orders`, `Refund Rate (Closed Window)` |
| 2 Why (946, 320) | `p6-bar-reason-share` | Bar (x 40, w 600) | Return Reason Share by Reason | `Return Reason Share` by `Returns[Return Reason Label]`, largest first (soft orange); tooltip `Return Quantity` |
| 2 Why | `p6-col-days-to-return` | Column (x 660, w 600) | Return Lines by Days to Return | `Return Lines` by `Returns[Days to Return]` (teal) |
| 2 Why | `p6-bar-return-rate-payment` | Bar (x 1280, w 600) | Return Rate by Payment Method | `Return Rate (Closed Window)` by `Orders[PaymentMethod]`, highest first (coral); tooltips `Total Orders`, `Returned Orders` |
| 3 Where (1346, 330) | `p6-bar-refund-category` | Bar (x 40, w 600) | Refund Amount by Category | `Refund Amount` by `Products[Category]`, largest first (muted brown); tooltips `Refund per Returned Order`, `Return Rate (Closed Window)` |
| 3 Where | `p6-table-returns-region` | Table (x 660, w 1220) | Returns and Refunds by Region | Region; Total Orders, Returned Orders, Return Rate (Closed Window), Refund Amount, Refund Rate (Closed Window), Refund per Returned Order, Average Days to Return. Sorted by Refund Amount descending |
| 4 Detail (1756, 80) | `p6-search-product` | Search slicer (w 420) | Search product (filters the table below) | `Products[ProductName]` |
| 4 Detail (1842, 380) | `p6-detail-product` | Table (w 1840) | Returns and Refunds by Product | ProductName, Category; Returned Orders, Return Quantity, Refund Amount, Refund per Returned Order, Return Rate for Product Orders. Sorted by Refund Amount descending |

Every visual uses order date. `Returns[Days to Return]` (`ReturnDate` minus the order date via `RELATED`) and `Returns[Return Reason Label]` (readable reason names) are calculated columns; `Return Reason Share` removes filters from both reason columns. The earlier return-month refund line, return rate by size, refund by reason × category, return rate by category and return rate by region were replaced: the return rate is about 5.6% in every size, category and region, so those charts showed no variation, while the payment-method, reason and timing views do. The open-window `Return Rate` / `Refund Rate`, `Return Rate (Received Orders)`, the return-date variants and the Return Reduction scenario measures were deleted as unused.

### 6.3.7. Customer Satisfaction (`cb86d456-…1ef3`)

Story (customer experience view), in four labelled sections: 1 satisfaction trend — is sentiment improving, and is the share of delivered orders that get a review holding up → 2 rating mix and feedback timing — how ratings spread across stars, how many days after the order reviews arrive, and whether slower delivery brings more negative reviews → 3 where dissatisfaction concentrates — category and region → 4 product detail. Ratings are nearly flat across every customer, order and delivery attribute (average 3.92–3.96, negative share 12–14%), so the page leads with sentiment shares and review coverage, which do move, and uses the product table as the place to act.

| Row (y, height) | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI (254, 132) | `kpi-avg-rating`, `kpi-positive-review-rate`, `kpi-negative-review-rate`, `kpi-review-count`, `kpi-review-coverage`, `kpi-avg-days-to-review` | Cards (6, w 290) | Average Rating · Positive Review Rate · Negative Review Rate · Review Count · Review Coverage · Average Days to Review | Measures of the same names (order date). Each card has a short text subtitle with its definition and "· order date" |
| Section labels (416, 896, 1296, 1706; h 40) | `p7-section-sentiment-trend`, `p7-section-rating-mix`, `p7-section-where-unhappy`, `p7-section-product-detail` | Text boxes (w 1840, h 40, copied from `p4-section-service`) | 1 · Satisfaction trend · 2 · Rating mix and feedback timing · 3 · Where dissatisfaction concentrates · 4 · Product detail | Heading plus the question the section answers; alt text set |
| 1 Trend (466, 400) | `p7-line-sentiment-month` | Line (x 40, w 1220) | Positive and Negative Review Rate by Month | `Positive Review Rate by Review Date` (teal) and `Negative Review Rate by Review Date` (coral) by `Date[Month Start]` (review date) |
| 1 Trend | `p7-col-coverage-year` | Column (x 1280, w 600) | Review Coverage by Year | `Review Coverage` by `Date[Year]` (soft blue, categorical axis); tooltips `Reviewed Orders`, `Delivered Orders` |
| 2 Mix (946, 320) | `p7-bar-rating-share` | Bar (x 40, w 600) | Rating Share by Star Rating | `Rating Share` by `Reviews[Rating Label]`, 5 ★ first (teal → soft blue → pale amber → soft orange → coral); tooltip `Review Count` |
| 2 Mix | `p7-col-days-to-review` | Column (x 660, w 600) | Review Count by Days to Review | `Review Count` by `Reviews[Days to Review]` (teal) |
| 2 Mix | `p7-col-neg-rate-transit` | Column (x 1280, w 600) | Negative Review Rate by Transit Days | `Negative Review Rate by Shipment` by `Shipments[Transit Days]` (coral); tooltips `Review Count by Shipment`, `Average Rating by Shipment` |
| 3 Where (1346, 330) | `p7-bar-rating-category` | Bar (x 40, w 600) | Negative Review Rate by Category | `Negative Review Rate` by `Products[Category]`, highest first (coral); tooltips `Review Count`, `Average Rating` |
| 3 Where | `p7-table-satisfaction-region` | Table (x 660, w 1220) | Satisfaction and Review Coverage by Region | Region; Review Count, Review Coverage, Average Rating, Positive Review Rate, Negative Review Rate, Average Days to Review. Sorted by Negative Review Rate descending |
| 4 Detail (1756, 80) | `p7-search-product` | Search slicer (w 420) | Search product (filters the table below) | `Products[ProductName]` |
| 4 Detail (1842, 380) | `p7-table-product-rating` | Table (w 1840) | Rating, Reviews and Returns by Product | ProductName, Category; Average Rating (Min Reviews), Review Count (Min Reviews), Positive Review Rate (Min Reviews), Negative Review Rate (Min Reviews), Return Rate (Min Reviews). Lowest rating first; products with fewer than 10 reviews (the `Minimum Reviews` default; no slicer on the page) stay blank in every column |

Cards, coverage, mix, timing, category, region and product views use order date; the trend line uses review date. `Reviews[Days to Review]` (`ReviewDate` minus the order date via `RELATED`) and `Reviews[Rating Label]` (`5 ★` … `1 ★`) are calculated columns; `Rating Share` removes filters from both rating columns. The earlier average-rating line, the review-count donut, the review-count-by-month column, average rating by region and the age-group table were replaced: the average rating and the negative share hardly differ by month, region, age group or any other attribute, while the sentiment shares, review coverage (falling from about 21% of delivered orders in 2012 to about 18% in 2022), review timing and product-level ratings do. `Average Rating by Review Date` and `Review Count by Review Date` were deleted as unused.

### 6.3.8. Category Tooltip (`c3e4d5f6-…6d7e`, hidden)

Report-page tooltip (`type: Tooltip`, `pageBinding` Pod1, 320×240, actual size, hidden in view mode). It is not in the page tabs; it opens when the reader hovers a category on `p2-bar-yoy-category`, `p2-bar-discount-rate` (SP), `p4-bar-risk-category` (IP), `p5-bar-cost-category` (Shipment), `p6-bar-refund-category` (RQ), `p7-bar-rating-category` (CS). Every card follows the hovered category plus the page's slicers.

| Position | Folder | Visual | Title | Fields |
| --- | --- | --- | --- | --- |
| Heading (6, 4; 308×42) | `tt-card-category` | Card, title off | – | `Selected Category` (`Products`): the hovered category, or "All categories" |
| Row 1 (y 52; 98×88 each) | `tt-card-net-sales`, `tt-card-net-sales-yoy`, `tt-card-gross-margin` | Cards | Net Sales · Net Sales YoY · Gross Margin | `Net Sales`, `Net Sales YoY %`, `Gross Margin %` |
| Row 2 (y 146) | `tt-card-return-rate`, `tt-card-negative-review-rate`, `tt-card-stockout-rate` | Cards | Return Rate · Negative Reviews · Stockout Rate | `Return Rate (Closed Window)`, `Negative Review Rate`, `Category Stockout Rate` (`Inventory`: `Stockout Rate` with the `Inventory[Stock Risk]` filter removed, so the stacked risk bar shows the category's rate, not the segment's) |

## 6.4. Known Issues in the Build

The final build has these known issues. They are recorded here, not fixed. Measures in the model that no visual binds are listed in section 2.10.

| Issue | Where | Effect |
| --- | --- | --- |
| KPI cards without a subtitle | EO: Gross Margin, Total Orders, Return Rate, Inventory Turnover, Stockout Rate, Fulfilled Rate, Order-to-Delivery Days. SP: Gross Margin. CM: Active, New and Repeat Customers. IP: Fill Rate, Stockout Rate, Days of Supply, Turnover. Shipment: `kpi-fulfillment-rate` | The last Desktop save dropped their subtitles, including most `… vs PY Label` bindings. The unbound label measures were later deleted; only `Total Orders vs PY Label` and `Inventory Value vs PY Label` remain |
| Hidden binding | `hdr-data-through` | `Total Orders vs PY Label` is bound only inside this hidden text visual |
| Filter path | Region slicer → `Inventory` | `Inventory` connects only to `Products` and the snapshot date, so the Region slicer doesn't filter inventory visuals |

## 6.5. Design Rules for Future Changes

These rules come from the original design. New and edited visuals should follow them:

- Titles take the form "Measure by Dimension". Put the date basis (order, ship, return, review, or snapshot date) in the subtitle.
- Never use colour alone to carry meaning. Pair it with labels, icons, or text.
- Don't use 3D charts, gauges without a target, or pies or donuts with many slices.
- Give every visual alt text, and add a reset-filters action to every page.
- Keep the shared header and slicer positions identical across pages. Copy an existing visual of the same type rather than writing new JSON.
- Label the transit thresholds (fixed 3-day and parameter-driven) as diagnostic, never as an SLA or "on-time".
- No page has a what-if slicer today, so those measures run at their defaults. If one is added, state the default in its title, because the measure falls back to it when nothing is selected.
