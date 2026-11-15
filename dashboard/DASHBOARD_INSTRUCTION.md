> **How to read this document.** Sections 1, 2.2, 3.1, and 5 are the original design intent and business rules. New measures and visuals should still follow them. Everything else (implementation status, data preparation, relationships, and the page design in section 6) describes the dashboard **as built** in `dashboard.Report/` and `dashboard.SemanticModel/`. Where the build departs from a rule, the as-built sections say so. Improvements to the build will be discussed separately.

# 1. PURPOSES OF THE DASHBOARD

## 1.1. Primary User

Supply Chain Manager and the Supply Chain Department.

## 1.2. Objectives and Scope

The dashboard provides an integrated view of supply chain performance and its relationship with sales and customer outcomes. It should help the Supply Chain Manager:

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

Page abbreviations: **EO** Executive Overview · **SP** Sales and Profitability · **CM** Customers and Market Performance · **IP** Inventory Performance · **DL** Delivery and Logistics · **RQ** Returns and Product Quality · **CS** Customer Satisfaction.

## 2.2. Common Calculation Rules

**Reporting calendar:** Use a shared date dimension with consistent week, month, and year definitions. Confirm whether reporting uses calendar or fiscal periods and define the week start day.

**Date basis:** Use order date for sales and purchasing activity; snapshot date for inventory; shipment or delivery date for logistics activity; return or refund date for return activity; and review date for customer feedback. Clearly label exceptions.

**Order eligibility:** Define which order statuses qualify for sales, units, and AOV. Use the same eligible-order population for each related numerator and denominator. Show all-order status monitoring separately.

**Discounts:** The supplied formulas assume DiscountAmount is a per-unit discount. Confirm this before implementation. If it is a line-level discount, do not multiply it by quantity.

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
| Discount Amount | Total discount, treating `DiscountAmount` as per unit | `SUMX(OrderItems, Quantity * DiscountAmount)` | order_items.csv | Built `Discount Amount`: SP |
| Net Sales | Gross sales less discounts, before refunds | `SUMX(OrderItems, Quantity * (UnitPrice - DiscountAmount))` | order_items.csv | Built `Net Sales`: EO, SP, CM |
| Discount Rate | Discount value as a share of gross sales | `DIVIDE([Discount Amount], [Gross Sales])` | order_items.csv | Built `Discount Rate`: SP |
| Units Sold | Quantity on order lines | `SUM(OrderItems[Quantity])` | order_items.csv | Built `Units Sold`: SP |
| Total Orders | Distinct orders | `DISTINCTCOUNT(Orders[OrderID])`, all statuses, no eligibility filter. Status-specific counts: `Delivered Orders`, `Shipped Orders` (shipped + delivered), `Pending Orders` (created + paid), `Cancelled Orders`, `Returned Status Orders` | orders.csv | Built (differs): no eligible-order rule. SP, CM, DL |
| Average Order Value (AOV) | Net sales per order | `DIVIDE([Net Sales], [Total Orders])` | orders.csv, order_items.csv | Built `Average Order Value`: SP |
| COGS | Cost of units sold at **current catalog cost** | `SUMX(OrderItems, Quantity * RELATED(Products[COGS]))` | order_items.csv, products.csv | Built `COGS`: SP (titled as a catalog-cost estimate) |
| Gross Profit | Net sales less COGS | `[Net Sales] - [COGS]` | order_items.csv, products.csv | Built `Gross Profit`: EO, SP |
| Gross Margin % | Gross profit as a share of net sales | `DIVIDE([Gross Profit], [Net Sales])` | order_items.csv, products.csv | Built `Gross Margin %`: SP |
| Revenue Growth (YoY) | Net-sales change versus the same period a year earlier | Inline `CALCULATE([Net Sales], DATEADD('Date'[Date], -1, YEAR))`; no separate prior-year measure; no completeness check | Date, order_items.csv | Built twice, identically: `Revenue Growth` and `Year-over-Year Growth`. Model only |
| Month-over-Month Growth | Net-sales change versus the previous month | Inline `DATEADD('Date'[Date], -1, MONTH)` | Date, order_items.csv | Built `Month-over-Month Growth`. Model only |
| Order Growth (YoY) | Order change versus the same period a year earlier | Inline prior-year `[Total Orders]` | Date, orders.csv | Built `Order Growth`. Model only |
| Product / Category / Channel Performance | Net sales, units, and orders by dimension | Existing measures by `Products[Category]`, `Orders[OrderSource]`, `Orders[DeviceType]`, `Payments[PaymentMethod]` | orders.csv, order_items.csv, products.csv, payments.csv | Built as visuals: SP |
| Contribution % | Category share of net sales | `DIVIDE([Net Sales], CALCULATE([Net Sales], ALL(Products[Category])))` | order_items.csv, products.csv | Built `Contribution %`. Model only |
| Catalog Benchmarks | Catalog price and cost references, not realized values | `AVERAGE(Products[Price])`, `AVERAGE(Products[COGS])`, product count | products.csv | Built `Average Product Price`, `Average COGS`, `Products in Catalog`. Model only |

Growth measures should compare complete, equivalent periods. The built growth measures don't check whether a period is complete.

## 2.4. Customers and Market Performance Metrics

| Metric | Definition / requirement | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Distinct Customers | Unique customers with at least one order in context | `DISTINCTCOUNT(Orders[CustomerID])`, all statuses | orders.csv | Built `Distinct Customers`: CM |
| Revenue per Customer | Net sales per distinct customer | `DIVIDE([Net Sales], [Distinct Customers])` | orders.csv, order_items.csv | Built `Revenue per Customer`: CM |
| Customer Segments | Customers, orders, and net sales by acquisition channel, age group, and gender | Existing measures by `Customers[AcquisitionChannel]`, `[AgeGroup]`, `[Gender]`. "Customers by Acquisition Channel / Age Group" count `Customers[CustomerID]` rows, so date, category, and region filters don't reach them | customers.csv, orders.csv | Built as visuals: CM |
| Customer Geography | Net sales by region | `[Net Sales]` by `Geography[Region]` through `Orders[Zip]`. No city view and no unmatched-ZIP report | geography.csv, orders.csv | Built (partial): CM |
| New Customers | Customers whose first order falls in the selected period | First `Orders[OrderDate]` per customer (all dates, all statuses) falls inside the current Date range | customers.csv, orders.csv | Built `New Customers`. Model only |
| Repeat Customers (period) | Customers in the period whose first order came earlier | First order date < `MIN('Date'[Date])` | orders.csv | Built `Repeat Customers`. Model only |
| First-Time / Repeat Rate (all-time) | Share of customers with exactly one order, or more than one, across all dates | Order count per customer under `ALL('Date')`, divided by `Distinct Customers` under `ALL('Date')` | orders.csv | Built `All-Time First-Time Customer Rate` (CM) and `All-Time Repeat Customer Rate` (EO, CM). These ignore the period slicer |
| Order Frequency | Orders per customer | `DIVIDE([Total Orders], [Distinct Customers])` | orders.csv | Built `Order Frequency`: CM |
| Signup Cohort | Customers by signup date | – | customers.csv | Not built. `SignupDate` is linked only to an auto date table |
| Orders by Sales Channel | Order count by order source and device type | `[Total Orders]` by `Orders[OrderSource]` / `Orders[DeviceType]` | orders.csv | Built as visuals: CM |

## 2.5. Inventory Performance Metrics

Inventory measures use the product-by-month-end snapshot grain. Stock balances take the latest snapshot in context (`VAR LatestSnapshot = MAX(Inventory[SnapshotDate])`) and never sum across dates. Inventory is related only to `Products` (see section 4), so the Reporting-period and Region slicers don't filter it. "Current" therefore means the latest snapshot in the data unless a visual puts `SnapshotDate` in context. The model doesn't display the snapshot date it used.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Current Stock | Units on hand at the latest snapshot | Sum of `StockOnHand` where `SnapshotDate = LatestSnapshot` | inventory.csv | Built `Current Stock`: EO, IP |
| Inventory Value | Latest-snapshot stock valued at catalog price | `SUMX` of `StockOnHand * RELATED(Products[Price])` at `LatestSnapshot` | inventory.csv, products.csv | Built `Inventory Value`: IP |
| Stock In / Stock Out | Units received / sold in the snapshot records | `SUM(Inventory[UnitsReceived])` / `SUM(Inventory[UnitsSold])` across the rows in context | inventory.csv | Built `Stock In`, `Stock Out`: IP |
| Low-Stock / Out-of-Stock Products | Products with ≤ 20 / 0 units at the latest snapshot | Distinct products at `LatestSnapshot` with `StockOnHand <= 20` / `= 0` | inventory.csv | Built `Low-Stock Products`, `Out-of-Stock Products`: IP |
| Reorder Candidates | Products flagged for reorder at the latest snapshot | Distinct products at `LatestSnapshot` with `ReorderFlag = 1` | inventory.csv | Built `Reorder Candidates`. Model only |
| Stockout / Overstock Rate | Share of snapshot rows flagged stockout / overstock | `DIVIDE(SUM(flag), COUNTROWS(Inventory))` over **all snapshot rows in context**, not the latest snapshot only | inventory.csv | Built `Stockout Rate` (EO), `Overstock Rate` (model only) |
| Stockout Days | Recorded stockout days | `AVERAGE(Inventory[StockoutDays])` (simple average of source values) | inventory.csv | Built `Stockout Days`. Model only; IP's detail table shows the raw column |
| Days of Supply | Recorded days of supply | `AVERAGE(Inventory[DaysOfSupply])` | inventory.csv | Built `Days of Supply`. Model only; raw column on IP |
| Fill Rate / Sell-Through Rate | Source-provided ratios | `AVERAGE(Inventory[FillRate])` / `AVERAGE(Inventory[SellThroughRate])` (simple averages) | inventory.csv | Built `Fill Rate` (model only; raw column on IP), `Sell-Through Rate` (IP) |
| Average Stock on Hand | Average balance across product-snapshot rows | `AVERAGE(Inventory[StockOnHand])` | inventory.csv | Built `Average Stock on Hand`. Model only |
| Snapshot Dates | Number of snapshot dates in context | `DISTINCTCOUNT(Inventory[SnapshotDate])` | inventory.csv | Built `Snapshot Dates`. Model only |

## 2.6. Delivery and Logistics Performance Metrics

Transit time is a measured elapsed duration. It is not, by itself, proof that an order was late. A true late-delivery KPI requires a promised delivery date or an approved service-level target. The 3-day rule below is therefore a diagnostic threshold, and the report labels it "(Diagnostic)".

`Shipments` has no active relationship (section 4). Shipment measures without a "by Ship Date" or "Product-Linked" variant therefore ignore every slicer and every row/axis dimension.

| Metric | Definition / requirement | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Shipment Cost | Total recorded shipping fees | `SUM(Shipments[ShippingFee])`. Date-aware variant: `Shipment Cost by Ship Date` (`USERELATIONSHIP` on ShipDate + `CROSSFILTER(..., OrderDate, NONE)`) | shipments.csv | Built: DL (card, region table); by-ship-date variant on DL trend |
| Shipment Records | Distinct orders with a shipment row | `DISTINCTCOUNT(Shipments[OrderID])` | shipments.csv | Built `Shipment Records`: DL |
| Shipment Coverage Rate | Shipment records as a share of orders | `DIVIDE([Shipment Records], [Total Orders])`. Only the denominator responds to filters | shipments.csv, orders.csv | Built (differs: unfiltered numerator) `Shipment Coverage Rate`: EO |
| Average Transit Time | Average days from ship to delivery for valid delivered shipments | `AVERAGEX` of `DATEDIFF(ShipDate, DeliveryDate, DAY)` where both dates are present and delivery ≥ ship | shipments.csv | Built `Average Delivery Time`, plus the alias `Average Transit Time`: DL |
| Transit Within 3 Days | Delivered shipments completed within 3 days (diagnostic) | Shipments with transit ≤ 3 days divided by `Delivered Shipments`. Date-aware variant: `Transit Within 3 Days by Ship Date` | shipments.csv | Built: DL |
| Product-Linked Shipment Measures | Shipment measures restricted to orders in the current product context | `TREATAS(VALUES(OrderItems[OrderID]), Shipments[OrderID])` | shipments.csv, order_items.csv | Built `Product-Linked Shipment Orders`, `Product-Linked Average Transit Time`: EO |
| Late Delivery Rate | Shipments delivered after a promised date | – | needs a promised-date / SLA field | Not built (no promised date in source) |
| Order-to-Delivery Lead Time | Days from order date to delivery | – | orders.csv, shipments.csv | Not built |
| Order Status Counts / Mix | Counts and shares by source order status | `[Total Orders]` by `Orders[OrderStatus]`. Mix: `DIVIDE([Total Orders], CALCULATE([Total Orders], ALL(Orders[OrderStatus])))` | orders.csv | Built: EO, DL (counts). `Order Status Mix` is model only |
| Fulfillment Rate | Share of orders with status delivered | `DIVIDE([Delivered Orders], [Total Orders])` | orders.csv | Built `Fulfillment Rate`: DL |

No measure uses the delivery-date role yet (`Shipments[DeliveryDate]` → `Date` is inactive and unused).

## 2.7. Returns and Product Quality Metrics

Return trends use the return date through `… by Return Date` measures. The other return measures follow the order-date path through `Returns[OrderID] → Orders`.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Returned Orders | Distinct orders with at least one return | `DISTINCTCOUNT(Returns[OrderID])` | returns.csv | Built `Returned Orders`: RQ |
| Return Quantity | Units recorded as returned | `SUM(Returns[ReturnQuantity])`. Date-aware variant: `Return Quantity by Return Date` | returns.csv | Built: RQ |
| Refund Amount | Recorded refund value | `SUM(Returns[RefundAmount])`. Date-aware variant: `Refund Amount by Return Date` | returns.csv | Built twice, identically: `Refund Amount` (RQ) and `Return Amount` (RQ reason donut and category table) |
| Return Rate | Share of orders with at least one return | `DIVIDE([Returned Orders], [Total Orders])`, all-status denominator. Product filters don't reach the denominator | returns.csv, orders.csv | Built `Return Rate`: EO, RQ |
| Return Rate for Product Orders | Return rate with a denominator that respects product filters | `DIVIDE([Returned Orders], DISTINCTCOUNT(OrderItems[OrderID]))` | returns.csv, order_items.csv | Built: EO, RQ, CS |
| Return Reason Share | Share of returned orders by reason | Returned orders with `ALL(Returns[ReturnReason])` as the denominator | returns.csv | Built `Return Reason Share`: RQ |
| Net Sales After Returns | Net sales less refunds | `[Net Sales] - [Refund Amount]`; no cohort alignment | order_items.csv, returns.csv | Built. Model only |
| Returns by Reason | Return values by recorded reason | Existing measures by `Returns[ReturnReason]` | returns.csv | Built as visuals: RQ |
| Returns by Product / Category | Returns by product or category | Existing measures by `Products[Category]` / `[ProductName]` | returns.csv, products.csv | Built as visuals: RQ |

## 2.8. Customer Satisfaction Metrics

Review trends use the review date through `… by Review Date` measures. Review measures count submitted reviews, not customers.

| Metric | Definition | Formula / logic (as built) | Source | Status |
| --- | --- | --- | --- | --- |
| Average Rating | Mean star rating | `AVERAGE(Reviews[Rating])`; no range validation. Date-aware variant: `Average Rating by Review Date` | reviews.csv | Built: EO, CS |
| Review Count | Number of submitted reviews | `COUNTROWS(Reviews)`. Date-aware variant: `Review Count by Review Date` | reviews.csv | Built: CS |
| Reviewed Orders / Coverage | Orders with a review, and their share of orders | `DISTINCTCOUNT(Reviews[OrderID])`; `DIVIDE(..., [Total Orders])` | reviews.csv, orders.csv | Built `Reviewed Orders`, `Review Order Coverage Rate`. Model only |
| Positive / Negative Review Rate | Share of reviews rated ≥ 4 / ≤ 2 | Rating-band count divided by `[Review Count]` | reviews.csv | Built `Positive Review Rate`, `Negative Review Rate`: CS |
| Low Rating Products | Products with an average rating < 3 | Count of products whose `[Average Rating] < 3` | reviews.csv, products.csv | Built `Low Rating Products`. Model only |
| Rating by Product / Category | Average rating and review count by product or category | Existing measures by `Products[Category]` / `[ProductName]` | reviews.csv, products.csv | Built as visuals: EO, CS |
| Rating and Return Diagnostic | Rating and return measures side by side | CS "Product Satisfaction Detail" shows rating, review count, positive rate, and product-order return rate. Low-review samples aren't suppressed or flagged | reviews.csv, returns.csv, products.csv | Built (partial): CS |

## 2.9. Cross-Domain Diagnostics

Cross-domain views are exploratory comparisons, not causal models. The report has no scatter charts. Cross-domain comparisons appear as side-by-side columns in tables.

| Diagnostic | Requirement / definition | As built | Status |
| --- | --- | --- | --- |
| Sales and Order Trend | Value, volume, and units over equivalent periods | Net Sales trend by Year-Month on EO (with Gross Sales and Gross Profit) and SP. No orders, units, or AOV trend | Built (partial) |
| Stock Availability and Sales Movement | Stock flags alongside units and net sales | Not built. IP compares Stock In with Stock Out only | Not built |
| Delivery and Returns | Return outcomes by transit group | – | Not built |
| Promotion Performance | Lines with vs without a promotion reference | – (`Promotions` isn't related to any fact) | Not built |
| Rating and Operational Experience | Ratings next to transit, return, and stock measures | EO "Category Health Signals" and "Product Exception Detail" tables: stockout rate, product-order return rate, average rating, product-linked transit time | Built (partial) |

## 2.10. Other Measures in the Model (not on any page)

| Area | Measures | Notes |
| --- | --- | --- |
| Web traffic | `Total Sessions`, `Users or Visitors`, `Page Views`, `Average Session Duration`, `Conversion Rate` = `DIVIDE([Total Orders], [Total Sessions])` | `WebTraffic` begins 2013-01-01 |
| Sales benchmark | `Benchmark Revenue`, `Benchmark COGS`, `Benchmark Gross Profit` | Separate daily series; not blended with order revenue |
| Payments | `Total Payment Value` | |

# 3. PREPARE THE DATA

## 3.1. Source Files and Grains

The following row counts and grains were checked against the CSV files in `data`. A
fact table's row count must not be used as a count of orders, products, or customers
unless its grain supports that interpretation.

| Fact/Dimension | File | Rows | Grain / coverage | Key columns | Information |
| --- | --- | ---: | --- | --- | --- |
| Fact | orders.csv | 646,945 | One row per order; 2012-07-04 to 2022-12-31 | `order_id` | Order date, customer, ZIP, status, payment method, device, and source. |
| Fact | order_items.csv | 714,669 | One row per order-product line; 16 repeated `order_id + product_id` combinations | `order_id`, `product_id` | Quantity, unit price, discount, and two promotion references. |
| Fact | shipments.csv | 566,067 | At most one row per shipped order; not all orders have a shipment row | `order_id` | Ship date, delivery date, and shipping fee. |
| Fact | payments.csv | 646,945 | One row per order | `order_id` | Payment method, payment value, and installments. |
| Fact | returns.csv | 39,939 | One row per return record; an order or product can occur more than once | `return_id` | Return date, reason, quantity, and refund amount. |
| Fact | reviews.csv | 113,551 | One row per review; an order, product, or customer can have multiple reviews | `review_id` | Review date, rating, title, and related order/product/customer. |
| Fact | web_traffic.csv | 3,652 | One row per date and traffic source; 2013-01-01 to 2022-12-31 | `date`, `traffic_source` | Sessions, visitors, page views, bounce rate, and session duration. |
| Dimension | customers.csv | 121,930 | One row per customer | `customer_id` | ZIP, city, signup date, gender, age group, and acquisition channel. |
| Fact / benchmark | sales.csv | 3,833 | One row per day; 2012-07-04 to 2022-12-31 | `Date` | Daily Revenue and COGS benchmark, separate from order-line revenue. |
| Fact | inventory.csv | 60,247 | One row per product and month-end snapshot; 126 snapshot dates | `product_id`, `snapshot_date` | Stock, receipt, movement, stockout, supply, fill, and reorder fields. |
| Dimension | geography.csv | 39,948 | One row per ZIP | `zip` | City, region, and district mapping. |
| Dimension | products.csv | 2,412 | One row per SKU | `product_id` | Product attributes, catalog price, and COGS. |
| Dimension | promotions.csv | 50 | One row per promotion | `promo_id` | Promotion dates, type, value, channel, applicability, and minimum order value. |

## 3.2. Power BI Preparation (as built)

Each of the 13 tables is an import partition with the same three Power Query steps:

```m
Source = Csv.Document(File.Contents("D:\projects\vintelligence2026_ai_agent_powerbi_dashboard\data\<file>.csv"),
                      [Delimiter = ",", Columns = n, Encoding = 65001, QuoteStyle = QuoteStyle.Csv]),
#"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
#"Changed Type"     = Table.TransformColumnTypes(#"Promoted Headers", { ... })
```

- **Paths are absolute.** There is no shared folder parameter. Moving the repository means editing every partition.
- **No row-level cleaning.** Power Query doesn't filter, deduplicate, or replace values. Blank promotion IDs and the 16 repeated `order_id + product_id` rows in `order_items.csv` are kept as they are.
- **Identifier columns** (`order_id`, `customer_id`, `product_id`, `zip`) are typed `Int64`, not text. Promotion IDs (`promo_id`, `promo_id_2`) are text.
- **Dates** are typed `date`. Amounts and rates are decimal numbers. Quantities, flags, and ratings are whole numbers. Source amounts aren't rounded.
- **`Date` table** is a calculated table:
  `ADDCOLUMNS(CALENDAR(DATE(2012,1,1), DATE(2023,12,31)), "Year", …, "Month Number", …, "Month Name", …, "Quarter", "Q" & …, "Year-Month", FORMAT(…, "YYYY-MM"), "Year-Month Sort", Year * 100 + Month)`.
  It isn't marked as a date table and has no week column.
- **Auto date/time is on** (`__PBI_TimeIntelligenceEnabled = 1`). Desktop creates hidden `LocalDateTable_*` tables for date columns without a relationship to `Date`. These are `Customers[SignupDate]`, `Promotions[StartDate]`, and `Promotions[EndDate]`.
- `sales.csv` is loaded as its own table (`Sales`) and isn't combined with `OrderItems`. `inventory.csv` is loaded at snapshot grain. Its own product attributes (`ProductName`, `Category`, `Segment`) are kept as columns and are used by some IP visuals.

Rules for future changes: keep blanks blank. Don't coerce invalid values to zero. Don't merge the repeated order-item rows. Don't blend `Sales` into order revenue.

## 3.3. Data-Quality Checks

The checks below were run against the CSVs when the model was designed. The report has **no data-quality page or exception visuals**, and measures don't exclude invalid rows. The only exception is the transit measures, which skip missing or negative intervals.

- Declared keys are unique, except the documented order-item composite. `order_id + product_id` isn't a primary key.
- Child keys match their parents: order items, payments, shipments, returns, and reviews match orders. Product references match products. Customer references match customers. Order and customer ZIPs match geography.
- Shipments cover 566,067 of 646,945 orders, so a missing shipment row isn't the same as an undelivered order. Reviews cover a subset of orders and customers.
- A delivery must not precede its shipment. The transit measures enforce this; nothing else does.
- Inventory has 126 monthly snapshot dates. It must not be interpolated into weekly or daily balances.
- The source order statuses are `delivered`, `cancelled`, `returned`, `shipped`, `paid`, and `created`. The status-group measures (`Shipped Orders`, `Pending Orders`) are the only groupings in the model.

# 4. RELATIONSHIPS

All relationships are many-to-one, single direction (dimension filters fact), as defined in `relationships.tmdl`. None is bidirectional.

**Active**

| From (many) | To (one) |
| --- | --- |
| Orders[CustomerID] | Customers[CustomerID] |
| Orders[Zip] | Geography[Zip] |
| OrderItems[OrderID] | Orders[OrderID] |
| OrderItems[ProductID] | Products[ProductID] |
| Payments[OrderID] | Orders[OrderID] |
| Returns[OrderID] | Orders[OrderID] |
| Returns[ProductID] | Products[ProductID] |
| Reviews[OrderID] | Orders[OrderID] |
| Reviews[ProductID] | Products[ProductID] |
| Inventory[ProductID] | Products[ProductID] |
| Orders[OrderDate] | Date[Date] |
| Sales[Date] | Date[Date] |
| WebTraffic[Date] | Date[Date] |

**Inactive** (turned on inside measures with `USERELATIONSHIP`)

| From (many) | To (one) | Used by |
| --- | --- | --- |
| Shipments[OrderID] | Orders[OrderID] | – (the Product-Linked measures use `TREATAS` instead) |
| Shipments[ShipDate] | Date[Date] | `Shipment Cost by Ship Date`, `Transit Within 3 Days by Ship Date` |
| Shipments[DeliveryDate] | Date[Date] | – |
| Returns[ReturnDate] | Date[Date] | `Return Quantity by Return Date`, `Refund Amount by Return Date` |
| Reviews[ReviewDate] | Date[Date] | `Review Count by Review Date`, `Average Rating by Review Date` |
| Inventory[SnapshotDate] | Date[Date] | – |

**Not related:** Customers → Geography (customer ZIP isn't linked), Reviews → Customers (omitted on purpose), and `Promotions` (not linked to any fact).

Date-role measures follow one pattern. They activate the role-playing relationship and switch off the order-date path, so that order date doesn't filter the result as well:

```dax
CALCULATE(
    [Shipment Cost],
    USERELATIONSHIP('Date'[Date], Shipments[ShipDate]),
    CROSSFILTER('Date'[Date], Orders[OrderDate], NONE)
)
```

**Resulting filter behaviour** (the four report slicers are `Date[Year-Month]`, `Products[Category]`, `Products[ProductName]`, `Geography[Region]`):

| Slicer | Orders, OrderItems, Payments, Returns, Reviews | Inventory | Shipments | Customers |
| --- | --- | --- | --- | --- |
| Reporting period | Yes, by order date | No | Only "by Ship Date" measures | No |
| Product category / Product | OrderItems, Returns, Reviews: yes. Orders and Payments: **no** (`Total Orders`, `Distinct Customers`, order status, and `Fulfillment Rate` don't change) | Yes | Only "Product-Linked" measures | No |
| Region | Yes, through Orders[Zip] | No | No | No |

Use the appropriate inactive date role explicitly for shipment, delivery, return, review, and inventory trends. Don't create a direct Reviews-to-Customers relationship, because Reviews already connect to customers through Orders.

# 5. ASSUMPTIONS AND CAVEATS

- The model reflects the supplied source files and does not invent missing shipments,
  demand, promised delivery dates, or customer attribution.
- `order_items.csv` is the primary transaction-level revenue fact. `sales.csv` is a
  daily Revenue/COGS benchmark and may differ because its calculation basis is not
  established by the source files.
- The source contains no promised delivery date or explicit service-level target.
  A three-day transit split is a diagnostic threshold, not a confirmed late-delivery
  KPI.
- `shipments.csv` is incomplete relative to orders. A missing shipment row means that
  shipment data is unavailable; it must not be classified as pending, cancelled, or
  late without an order-status or service rule.
- Promotion references are sparse (`promo_id_2` is almost entirely blank) and are not
  guaranteed to identify one promotion per order line. Promotion analysis is
  directional and must retain an unknown/unrecorded group.
- `web_traffic.csv` begins on 2013-01-01 while core commerce and sales data begins on
  2012-07-04. Do not compare traffic and commerce trends before the traffic start date.
- Inventory is monthly and includes product attributes in addition to the product
  relationship. Use the Products dimension for shared product filtering and inspect
  attribute mismatches rather than assuming both copies are identical.
- Return refunds are recorded at return-row grain and cannot be treated as same-period
  sales reversals without aligning them to the associated order cohort.
- Reviews include `customer_id`, but the direct Reviews-to-Customers relationship is
  intentionally omitted to avoid an ambiguous filter path. Use the order relationship
  or a deliberate bridge only if the model design requires it.

# 6. DASHBOARD DESIGN (AS BUILT)

## 6.1. Report Structure

- **Pages:** 8 in `pageOrder`. There are seven analytical pages plus a blank leftover `Page 1` (1920×1080, Fit to page, no visuals). Executive Overview is the active (opening) page.
- **Canvas:** analytical pages are **1920×1570**, display option **Fit to width**.
- **Theme:** `purple-slicer`, registered as both the base theme and the custom theme. Colours come from the theme, not from per-visual formatting.
- **Navigation:** none. There are no page-navigator buttons, bookmarks, or drill-through pages. Users move between pages with the page tabs.
- **Reset filters:** not built. No page has a reset button or bookmark.
- **Alt text:** not set on any visual.
- **Subtitles:** empty on every visual. The date basis is stated in the title instead (for example "by Ship Month" or "by Return Month").
- **Report-level filters:** none.
- **Not built from the original design:** a Diagnostics page, a Data Quality page, scatter-chart cross-domain views, and a sixth KPI card per page.

## 6.2. Shared Layout

Every analytical page uses the same grid. The x positions run from 40 to 1880.

| Row | y | Height | Contents |
| --- | --- | --- | --- |
| Header | 16 | – | `hdr-band` (background band), `hdr-brand`, `hdr-title` (page title and a one-line purpose) |
| Slicers | 110 | – | Four dropdown slicers, left to right (below) |
| KPI cards | 254 | 132 | 4 cards (5 on Executive Overview), named `kpi-*` |
| Row 2 | 406 | 430 | Main trend, plus one or two comparison visuals |
| Row 3 | 856 | ≈314 | Breakdown visuals and a summary table |
| Row 4 | 1200 | 280 | Detail table (wide, left) and a ranked bar chart (right). Ends at y = 1480 |

**Slicers.** All four are dropdowns. Each has its own sync group, so a selection carries across all seven analytical pages.

| Visual | Title | Field |
| --- | --- | --- |
| `filter-1-year-month` | Reporting period | `Date[Year-Month]` |
| `filter-2-category` | Product category | `Products[Category]` |
| `filter-3-productname` | Product | `Products[ProductName]` |
| `filter-4-region` | Region | `Geography[Region]` |

Section 4 shows which tables each slicer reaches.

**Naming.** Visual folder names were copied between pages and often don't match the content. For example, `kpi-net-sales` on Executive Overview shows Shipment Coverage. The tables below give each folder name next to its displayed title.

## 6.3. Pages

### 6.3.1. Executive Overview (`9f7b1d2d-…f712`)

Purpose: a one-page health check across sales, stock, returns, delivery, and satisfaction.

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-net-sales` | Card | Shipment Coverage | `Shipment Coverage Rate` |
| KPI | `kpi-total-orders` | Card | Stockout Rate | `Stockout Rate` |
| KPI | `kpi-distinct-customers` | Card | Return Rate | `Return Rate` |
| KPI | `kpi-aov` | Card | Average Rating | `Average Rating` |
| KPI | `kpi-otd` | Card | Repeat Customer Rate | `All-Time Repeat Customer Rate` |
| 2 | `p1-trend-net-sales` | Line (w 1180) | Portfolio Net Sales Trend by Month | Net Sales, Gross Sales, Gross Profit by `Date[Year-Month]` |
| 2 | `p1-col-orderstatus` | Column | Orders by Status | `Total Orders` by `Orders[OrderStatus]` |
| 3 | `p1-signal-map` | Bar | Average Rating by Category | `Average Rating` by Category |
| 3 | `p1-bar-region` | Bar | Stockout Risk by Category | `Stockout Rate` by Category |
| 3 | `p1-table-segment` | Table | Category Health Signals | Category; Stockout Rate, Return Rate for Product Orders, Average Rating, Product-Linked Average Transit Time |
| 4 | `p1-detail-product` | Table | Product Exception Detail: Stock, Returns, Rating, and Transit | Product-level version of the category health table |
| 4 | `p1-bar-gross-profit` | Bar | Return Rate by Category | `Return Rate for Product Orders` by Category |

### 6.3.2. Sales and Profitability (`2d466881-…34d5`)

Purpose: revenue, discounting, and margin by channel and category (order date).

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-*` | Cards | Net Sales · Gross Sales · Total Orders · Average Order Value | Measures of the same names |
| 2 | `p2-trend-net-sales` | Line (full width) | Net Sales Trend by Month | `Net Sales` by `Date[Year-Month]` |
| 3 | `p2-col-device` | Column | Net Sales by Device Type | by `Orders[DeviceType]` |
| 3 | `p2-col-source` | Column | Net Sales by Order Source | by `Orders[OrderSource]` |
| 3 | `p2-bar-payment` | Bar | Net Sales by Payment Method | by `Payments[PaymentMethod]` |
| 3 | `p2-table-category` | Table | Category Performance | Category; Net Sales, Units Sold, Average Order Value |
| 4 | `p2-detail-profitability` | Table | Catalog-Cost Profitability Estimate (Not Historical Cost) | Category, ProductName; Gross Sales, Discount Amount, Net Sales, COGS, Gross Profit, Gross Margin % |
| 4 | `p2-bar-discount-rate` | Bar | Discount Rate by Category | `Discount Rate` by Category |

### 6.3.3. Customers and Market Performance (`a1f2c3d4-…5d6e`)

Purpose: customer base, buyer behaviour, channels, and regional sales. This page has no time trend.

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-distinct-customers` | Card | Distinct Customers | `Distinct Customers` |
| KPI | `kpi-revenue-per-customer` | Card | Revenue per Customer | `Revenue per Customer` |
| KPI | `kpi-avg-rating` | Card | First-Time Buyer Rate (All-Time) | `All-Time First-Time Customer Rate` |
| KPI | `kpi-positive-review-rate` | Card | Repeat Buyer Rate (All-Time) | `All-Time Repeat Customer Rate` |
| 2 | `p5-col-channel` | Column | Customers by Acquisition Channel | Customer count by `Customers[AcquisitionChannel]` |
| 2 | `p5-donut-age` | Donut | Customers by Age Group | Customer count by `Customers[AgeGroup]` |
| 2 | `p5-donut-rating` | Donut | Orders by Device Type | `Total Orders` by `Orders[DeviceType]` |
| 3 | `p5-bar-rating-category` | Bar | Orders by Sales Source | `Total Orders` by `Orders[OrderSource]` |
| 3 | `p3-segment-map` | Bar | Revenue per Customer by Segment | by `Products[Segment]` |
| 3 | `p5-table-agegroup` | Table | Customer Segment Summary | Customer measures by segment |
| 4 | `p3-detail-customer` | Table | Customer Segment Detail | AcquisitionChannel, AgeGroup, Gender; Distinct Customers, Total Orders, Order Frequency, Net Sales |
| 4 | `p3-bar-region-sales` | Bar | Net Sales by Region | `Net Sales` by `Geography[Region]` |

The `p5-*` folders were copied from Customer Satisfaction and repurposed for customer visuals. Their names still refer to ratings.

### 6.3.4. Inventory Performance (`5c7bc72d-…68d16b1ac7af`)

Purpose: stock position, movement, and risk (latest snapshot; see section 2.5).

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-*` | Cards | Current Stock · Inventory Value · Low-Stock Products · Out-of-Stock Products | Measures of the same names |
| 2 | `p4-trend-stock` | Line | Stock In vs Stock Out Trend | `Stock In`, `Stock Out` by `Inventory[SnapshotDate]` |
| 2 | `p4-inventory-map` | Bar | Sell-Through Rate by Category | `Sell-Through Rate` by Category |
| 3 | `p4-bar-segment` | Bar | Inventory Value by Segment | `Inventory Value` by Segment |
| 3 | `p4-col-lowstock` | Column | Low & Out-of-Stock Products by Category | `Low-Stock Products`, `Out-of-Stock Products` |
| 3 | `p4-table-category` | Table | Inventory Summary by Category | Current Stock, Inventory Value, Stock In, Stock Out |
| 4 | `p4-detail-stock` | Table | Inventory Exception Detail | Raw columns: ProductName, SnapshotDate, StockOnHand, StockoutDays, DaysOfSupply, FillRate, ReorderFlag |
| 4 | `p4-bar-stock-category` | Bar | Current Stock by Category | `Current Stock` by Category |

The trend axis uses `Inventory[SnapshotDate]` directly, not the `Date` table.

### 6.3.5. Delivery and Logistics (`fcd9a8b8-…183f6a`)

Purpose: fulfilment, transit time, and shipping cost. The trends use ship date. The 3-day split is labelled "(Diagnostic)".

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-shipment-cost` | Card | Shipment Cost | `Shipment Cost` |
| KPI | `kpi-otd` | Card | Fulfillment Rate | `Fulfillment Rate` |
| KPI | `kpi-return-rate` | Card | Transit Within 3 Days (Diagnostic) | `Transit Within 3 Days` |
| KPI | `kpi-refund-amount` | Card | Average Transit Time | `Average Transit Time` |
| 2 | `p3-trend-shipcost` | Line | Shipment Cost by Ship Month | `Shipment Cost by Ship Date` by Year-Month |
| 2 | `p3-trend-otd` | Line | Transit Within 3 Days by Ship Month (Diagnostic) | `Transit Within 3 Days by Ship Date` by Year-Month |
| 3 | `p3-donut-reason` | Bar | Transit Within 3 Days by Region (Diagnostic) | by `Geography[Region]` |
| 3 | `p3-col-orderstatus` | Column | Orders by Status | `Total Orders` by `Orders[OrderStatus]` |
| 3 | `p3-table-returns-category` | Table | Shipment Volume, Transit, and Cost by Region | Region; Shipment Records, Average Transit Time, Shipment Cost |
| 4 | `p5-detail-delivery` | Table | Delivery Status Detail | OrderStatus; Total Orders, Fulfillment Rate, Shipment Records, Average Transit Time, Transit Within 3 Days |
| 4 | `p5-logistics-map` | Bar | Average Transit Time by Region | by `Geography[Region]` |

Shipment measures don't respond to Region or Status (section 4), so the region and status breakdowns repeat the same shipment value on every row.

### 6.3.6. Returns and Product Quality (`b2f3c4d5-…5d6e7f`)

Purpose: return volume, value, reasons, and category exposure. The trends use return date.

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-shipment-cost` | Card | Return Quantity | `Return Quantity` |
| KPI | `kpi-otd` | Card | Returned Orders | `Returned Orders` |
| KPI | `kpi-return-rate` | Card | Return Rate | `Return Rate` |
| KPI | `kpi-refund-amount` | Card | Refund Amount | `Refund Amount` |
| 2 | `p3-trend-shipcost` | Line | Refund Amount by Return Month | `Refund Amount by Return Date` by Year-Month |
| 2 | `p3-trend-otd` | Line | Returned Quantity by Return Month | `Return Quantity by Return Date` by Year-Month |
| 3 | `p3-donut-reason` | Donut | Returns by Reason | `Return Amount` by `Returns[ReturnReason]` |
| 3 | `p3-col-orderstatus` | Column | Returned Orders by Product Category | `Returned Orders` by Category |
| 3 | `p3-table-returns-category` | Table | Returns by Product Category | Returned Orders, Return Quantity, Return Amount |
| 4 | `p6-detail-returns` | Table | Return Detail by Reason | ReturnReason; Returned Orders, Return Quantity, Refund Amount, Return Reason Share |
| 4 | `p6-quality-map` | Bar | Return Rate by Category | `Return Rate for Product Orders` by Category |

This page and Delivery and Logistics share folder names (`p3-*`, `kpi-shipment-cost`, …) because one was copied from the other.

### 6.3.7. Customer Satisfaction (`cb86d456-…1ef3`)

Purpose: rating levels, rating mix, and weak categories and products. The trends use review date.

| Row | Visual (folder) | Type | Title | Content |
| --- | --- | --- | --- | --- |
| KPI | `kpi-distinct-customers` | Card | Review Count | `Review Count` |
| KPI | `kpi-revenue-per-customer` | Card | Negative Review Rate | `Negative Review Rate` |
| KPI | `kpi-avg-rating` | Card | Average Rating | `Average Rating` |
| KPI | `kpi-positive-review-rate` | Card | Positive Review Rate | `Positive Review Rate` |
| 2 | `p5-col-channel` | Column | Review Count by Review Month | `Review Count by Review Date` by Year-Month |
| 2 | `p5-donut-age` | Donut | Review Count by Customer Age Group | by `Customers[AgeGroup]` |
| 2 | `p5-donut-rating` | Donut | Rating Distribution | Review Count by `Reviews[Rating]` |
| 3 | `p5-bar-rating-category` | Bar | Negative Review Rate by Category | `Negative Review Rate` by Category |
| 3 | `p5-col-gender` | **Line** | Average Rating by Review Month | `Average Rating by Review Date` by Year-Month |
| 3 | `p5-table-agegroup` | Table | Satisfaction by Customer Age Group | Rating measures by `Customers[AgeGroup]` |
| 4 | `p7-detail-rating` | Table | Product Satisfaction Detail | Category, ProductName; Average Rating, Review Count, Positive Review Rate, Return Rate for Product Orders |
| 4 | `p7-experience-map` | Bar | Average Rating by Category | `Average Rating` by Category |

## 6.4. Design Rules for Future Changes

These rules come from the original design. New and edited visuals should follow them:

- Titles take the form "Measure by Dimension". Put the date basis (order, ship, return, review, or snapshot date) in the subtitle.
- Never use colour alone to carry meaning. Pair it with labels, icons, or text.
- Don't use 3D charts, gauges without a target, or pies or donuts with many slices.
- Give every visual alt text, and add a reset-filters action to every page.
- Keep the shared header and slicer positions identical across pages. Copy an existing visual of the same type rather than writing new JSON.
- Label the 3-day transit threshold as diagnostic, never as an SLA or "on-time".

