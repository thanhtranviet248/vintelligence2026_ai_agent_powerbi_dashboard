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

The formulas below use semantic-model names where they were supplied. For additional metrics without confirmed table or column names, formulas are expressed as calculation logic rather than executable DAX.

## 2.1. Implementation Status

**Core:** Reported as already implemented in the supplied requirements. Implementation has not been independently verified.

**Planned:** Not yet implemented, but the required fields are identified in the supplied source descriptions.

**Conditional:** Requires additional fields, confirmed relationships, or an agreed business rule before implementation.

**Important:** A metric should not be labelled Core if part of its calculation is still Planned. For example, revenue-based product performance may be Core, while gross-profit-based product performance remains Planned.

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

Unless stated otherwise, sales measures use the eligible-order population defined in
2.2. Exclude invalid quantities, prices, or dates from numeric measures and surface them
as data-quality exceptions. Do not mix order-date and return-date totals in the same trend.

| Metric | Definition | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Gross Sales | Value of eligible order lines before discounts | `SUMX(OrderItems, Quantity * UnitPrice)` | order_items.csv | Core |
| Discount Amount | Total discount applied to eligible units | `SUMX(OrderItems, Quantity * DiscountAmount)` only if discount is per unit; otherwise sum the line value | order_items.csv | Planned |
| Net Sales | Gross sales less discounts, before refunds | `SUMX(OrderItems, Quantity * (UnitPrice - DiscountAmount))` when discount is confirmed per unit; otherwise use the approved line-level rule | order_items.csv | Conditional |
| Discount Rate | Discount value as a share of gross sales | `DIVIDE([Discount Amount], [Gross Sales])` | order_items.csv | Planned |
| Units Sold | Quantity on eligible order lines | `SUM(OrderItems[Quantity])` | order_items.csv | Core |
| Total Orders | Distinct eligible orders; show all statuses separately | `DISTINCTCOUNT(Orders[OrderID])` using the agreed status filter | orders.csv | Core |
| Average Order Value (AOV) | Net sales per eligible order | `DIVIDE([Net Sales], [Total Orders])`; available only when Net Sales is confirmed | orders.csv, order_items.csv | Conditional |
| COGS | Cost of eligible units sold using the confirmed cost basis | `SUMX(OrderItems, Quantity * RELATED(Products[COGS]))` | order_items.csv, products.csv | Planned |
| Gross Profit | Net sales less COGS | `[Net Sales] - [COGS]` | order_items.csv, products.csv | Planned |
| Gross Margin % | Gross profit as a share of net sales | `DIVIDE([Gross Profit], [Net Sales])` | order_items.csv, products.csv | Planned |
| Revenue Growth (YoY) | Net-sales change versus the equivalent prior-year period | `DIVIDE([Net Sales] - [Prior-Year Net Sales], [Prior-Year Net Sales])`; blank when Net Sales is unconfirmed or the denominator is zero | Date, order_items.csv | Conditional |
| Order Growth (YoY) | Eligible-order change versus the equivalent prior-year period | `DIVIDE([Total Orders] - [Prior-Year Orders], [Prior-Year Orders])`; blank when the denominator is zero | Date, orders.csv | Core |
| Product / Category / Channel Performance | Compare net sales, units, orders, and profit by the selected dimension | Apply the same eligible-order and date context to each measure | orders.csv, order_items.csv, products.csv | Conditional |
| Contribution % | Selected product or category share of net sales within the retained report context | `DIVIDE([Net Sales], [Net Sales] with only the selected product/category filter removed)` | order_items.csv, products.csv | Conditional |
| Catalog Benchmarks | Catalog price and cost references, not realized sales or profit | `AVERAGE(Products[Price])`, `AVERAGE(Products[COGS])`; label as catalog values | products.csv | Core |

Growth measures must compare complete, equivalent periods and must not be shown as
meaningful when the comparison period is incomplete or has no denominator.

## 2.4. Customers and Market Performance Metrics

| Metric | Definition / requirement | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Distinct Customers | Unique customers with at least one eligible order in the selected period | `DISTINCTCOUNT(Orders[CustomerID])` | orders.csv | Core |
| Revenue per Customer | Net sales per distinct buying customer | `DIVIDE([Net Sales], [Distinct Customers])` | orders.csv, order_items.csv | Core |
| Customer Segments | Compare customers, orders, units, and net sales by acquisition channel, age group, and gender | Reuse the corresponding measures by customer attributes; acquisition channel remains distinct from order channel | customers.csv, orders.csv | Core |
| Customer Geography | Compare customers, orders, and net sales by mapped region and city | Use `Geography` for ZIP attributes and report unmatched ZIPs separately | geography.csv, customers.csv, orders.csv | Core |
| New Customers | Customers whose first eligible order occurs in the selected period | Identify the minimum eligible order date per customer, then count customers whose first date is in the period | customers.csv, orders.csv | Planned |
| Signup Cohort | Customers whose signup date falls in the selected period; do not label this as new purchasing customers | `DISTINCTCOUNT(Customers[CustomerID])` by `SignupDate`, where a signup date is available | customers.csv | Planned |
| Orders by Sales Channel | Eligible order count split by order source and device type | `[Total Orders]` by `Orders[OrderSource]` / `Orders[DeviceType]` | orders.csv | Core |
| Repeat Customer / Order Frequency | Distinguish first-time from returning purchasing customers | Classify customers by first eligible order date; calculate `DIVIDE([Total Orders], [Distinct Customers])` | orders.csv | Planned |

## 2.5. Inventory Performance Metrics

Inventory measures use the product-by-month-end snapshot grain. They must not be
interpreted as daily balances or summed across snapshot dates. “Current” means the
latest available snapshot in the selected context; if no snapshot is selected, use the
latest snapshot date and display that date.

| Metric | Definition | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Stock on Hand | Units on hand at the selected/latest snapshot | Sum `StockOnHand` for one snapshot date per product | inventory.csv | Core |
| Inventory Value | Selected-snapshot stock valued at catalog price | `SUMX(Inventory, StockOnHand * RELATED(Products[Price]))` at one snapshot date | inventory.csv, products.csv | Core |
| Stock In / Stock Out | Units received / recorded as sold in the snapshot record | Sum the relevant snapshot fields; do not describe them as daily flows | inventory.csv | Core |
| Low-Stock / Out-of-Stock Products | Products at or below 20 units / at zero units in one snapshot | Distinct products filtered by `StockOnHand <= 20` or `= 0` | inventory.csv | Core |
| Stockout Days | Recorded stockout days in the snapshot period | `AVERAGE(Inventory[StockoutDays])`; state that it is an average of source values | inventory.csv | Planned |
| Stockout / Overstock Rate | Share of product snapshots flagged stockout / overstock | Flagged snapshot rows divided by all valid snapshot rows in the same context | inventory.csv | Planned |
| Reorder Candidates | Products flagged for reorder in one snapshot | Distinct products where `ReorderFlag = 1` | inventory.csv | Planned |
| Days of Supply | Recorded expected days of supply at the snapshot | `AVERAGE(Inventory[DaysOfSupply])`; do not derive without a confirmed demand-rate rule | inventory.csv | Planned |
| Fill Rate / Sell-Through Rate | Source-provided fulfillment and movement ratios | Prefer aggregated numerator/denominator measures; use averages only when source denominators are unavailable and label them as simple averages | inventory.csv | Planned |
| Average Stock on Hand | Average snapshot balance across product-date rows | `AVERAGE(Inventory[StockOnHand])`; not a daily average and not a days-of-inventory measure | inventory.csv | Core |

## 2.6. Delivery and Logistics Performance Metrics

Transit time is a measured elapsed duration. It is not, by itself, proof that an
order was late. A true late-delivery KPI requires a promised delivery date or an
approved service-level target. The 3-day rule below is therefore a diagnostic
threshold unless the business explicitly approves it as the service level.

| Metric | Definition / requirement | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Shipment Cost | Total recorded shipping fees for the selected shipment population | `SUM(Shipments[ShippingFee])` | shipments.csv | Core |
| Average Transit Time | Average elapsed days from ship to delivery for valid delivered shipments | Average `DATEDIFF(ShipDate, DeliveryDate, DAY)` after excluding missing or negative intervals | shipments.csv | Core |
| Transit Within 3 Days | Delivered shipments completed within the current 3-day diagnostic threshold | Delivered shipments with `DATEDIFF(ShipDate, DeliveryDate, DAY) <= 3` divided by delivered shipments | shipments.csv | Core |
| Late Delivery Rate | Shipments delivered after the approved promise or service-level target | Delivered shipments with `DeliveryDate > PromisedDate` (or above approved target) divided by delivered shipments | shipments.csv, confirmed SLA field | Conditional |
| Order-to-Delivery Lead Time | Elapsed days from order date to delivery for valid delivered orders | Average `DATEDIFF(OrderDate, DeliveryDate, DAY)` with one order-to-shipment record per order | orders.csv, shipments.csv | Planned |
| Order Status Counts / Mix | Counts and shares by the source order status | `[Total Orders]` by `OrderStatus`; mix uses the same denominator with only status filtering removed | orders.csv | Core |
| Fulfillment Rate | Share of eligible orders whose source status is delivered | `DIVIDE([Delivered Orders], [Total Orders])`; do not infer delivery from a nonblank shipment alone | orders.csv | Core |

Use ship date for shipment trends, delivery date for delivery trends, and order date
for order-status trends. Missing dates and unmatched orders must be reported.

## 2.7. Returns and Product Quality Metrics

Return activity uses return date. Reconciliation to sales uses the associated order
cohort, and must not be interpreted as same-period profitability unless the periods are
explicitly aligned. Return-rate denominators must use the corresponding eligible-order
population; for a delivered-order diagnostic, state that denominator separately.

| Metric | Definition | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Returned Orders | Distinct eligible orders with at least one recorded return | `DISTINCTCOUNT(Returns[OrderID])` | returns.csv, orders.csv | Core |
| Return Quantity | Total units recorded as returned | `SUM(Returns[ReturnQuantity])` | returns.csv | Core |
| Refund Amount | Total recorded refund value | `SUM(Returns[RefundAmount])`; do not treat as a sales reversal unless the data rule confirms it | returns.csv | Core |
| Return Rate | Share of eligible orders with at least one return | `DIVIDE([Returned Orders], [Total Orders])` with matching order context | returns.csv, orders.csv | Core |
| Net Sales After Returns | Net sales less recorded refunds for the aligned order cohort | `[Net Sales] - [Refund Amount]`; label the cohort and period basis | order_items.csv, returns.csv | Planned |
| Returns by Reason | Returned quantity and refund value by recorded reason | `[Return Quantity]` / `[Refund Amount]` by `ReturnReason`; retain unknown reasons | returns.csv | Core |
| Returns by Product / Category | Returns split by SKU or product category | Apply the product relationship and show quantity, refund value, and returned orders | returns.csv, products.csv | Core |

Show return measures by return period, product/category, customer segment, linked
promotion, and reason. Use reasons to distinguish potential quality issues, delivery
problems, customer preference, and other causes; do not infer cause from the reason alone.

## 2.8. Customer Satisfaction Metrics

Review measures use review date for trends and count submitted reviews, not customers,
unless explicitly labelled otherwise. Product comparisons should include review count so
small samples are not presented as equally representative.

| Metric | Definition | Formula / logic | Source | Status |
| --- | --- | --- | --- | --- |
| Average Rating | Mean valid star rating across submitted reviews | `AVERAGE(Reviews[Rating])` after validating the rating range | reviews.csv | Core |
| Review Count | Number of submitted reviews | `COUNTROWS(Reviews)` or distinct `ReviewID` where duplicates are possible | reviews.csv | Core |
| Positive / Negative Review Rate | Share of valid reviews rated 4–5 / 1–2 | Numerator filtered by rating band divided by `[Review Count]` | reviews.csv | Core |
| Rating by Product / Category | Average rating and review count by product or category | `[Average Rating]` and `[Review Count]` by product/category | reviews.csv, products.csv | Core |
| Rating and Return Diagnostic | Compare rating and return measures for the same product/category and aligned context | `[Average Rating]`, `[Review Count]`, and `[Return Rate]`; suppress or flag low-review samples | reviews.csv, returns.csv, products.csv | Planned |

Use ratings with delivery, returns, and sales to identify patterns for investigation.
Written review text can support qualitative inspection but is not a numeric KPI.

## 2.9. Cross-Domain Diagnostics

Cross-domain views are exploratory comparisons, not causal models. They must disclose
the date basis, denominator, sample size, and any incomplete or unmatched data.

| Diagnostic | Requirement / definition | Formula or view | Source | Status |
| --- | --- | --- | --- | --- |
| Sales and Order Trend | Compare value, volume, and units over equivalent weekly, monthly, or yearly periods | `[Net Sales]`, `[Total Orders]`, `[Units Sold]`, and `[AOV]` by the selected Date period | orders.csv, order_items.csv | Core |
| Stock Availability and Sales Movement | Compare stock flags or stockout days with observed units and net sales | Inventory snapshot measures alongside `[Units Sold]` / `[Net Sales]` by product and aligned period; no lost-sales estimate | inventory.csv, order_items.csv, products.csv | Planned |
| Delivery and Returns | Compare return outcomes across transit-time or approved late-delivery groups | Return rate by derived transit group, using a clearly stated delivered-order denominator | shipments.csv, returns.csv, orders.csv | Planned |
| Promotion Performance | Compare order lines with and without a promotion reference | `[Net Sales]`, `[Units Sold]`, `[Discount Amount]`, and return measures by promotion presence; blank references are an unknown group, not proof of no promotion | order_items.csv, promotions.csv, returns.csv | Planned |
| Rating and Operational Experience | Compare ratings with transit, return, and fulfillment measures | Product/category-level view including review count and aligned date context; interpret as association only | reviews.csv, shipments.csv, returns.csv | Planned |

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

## 3.2. Power BI Preparation Rules

- Import identifier columns such as `order_id`, `customer_id`, `product_id`, `zip`,
  and `promo_id` as text where leading zeros or nonnumeric values must be preserved.
- Convert all source date columns to Date types. Use one marked Date table with a
  continuous date column and Year, Month Number, Month, Quarter, and Year-Month
  attributes. Sort Year-Month by `Year * 100 + Month Number`.
- Keep numeric amounts as decimal numbers and quantities, flags, installments, and
  ratings as appropriate whole-number types. Do not round source amounts before
  aggregation.
- Preserve blanks in `promo_id`, `promo_id_2`, and
  `promotions[applicable_category]`. A blank promotion reference is an unknown or
  unrecorded promotion state, not evidence that no promotion applied.
- Do not remove the 16 repeated `order_id + product_id` combinations in
  `order_items.csv` without a business rule. Treat each source row as a separate
  line unless the source owner confirms that they are duplicates to be merged.
- Validate nonnegative quantities, prices, costs, refunds, fees, and rates. Do not
  silently coerce invalid values to zero; expose rejected or anomalous rows in a
  data-quality view.
- Keep `sales.csv` separate from `order_items.csv`. Use it as a daily benchmark and
  reconcile differences rather than combining both tables into one revenue fact.
- Treat `inventory.csv` as a snapshot fact. Select one snapshot date for stock
  balances and do not sum `stock_on_hand` across dates.
- Treat `web_traffic.csv` as a date-and-source aggregate. It can support traffic
  trends and source comparisons, but it cannot attribute sessions to orders without
  a session or campaign key.

## 3.3. Data-Quality Checks

- Confirm uniqueness of the declared keys: all checked dimension and transaction
  keys are unique except the documented order-item composite. Do not declare
  `order_id + product_id` a primary key.
- Confirm that all checked child keys match their parents: order items, payments,
  shipments, returns, and reviews match orders; product references match products;
  customer references match customers; and order/customer ZIPs match geography.
- Record coverage counts in the model documentation. Shipments cover 566,067 of
  646,945 orders, so missing shipment rows are not equivalent to undelivered
  orders. Reviews cover a subset of orders and customers.
- Validate date ordering: shipment delivery must not precede shipment, and shipment
  must not precede the order when both dates are present. Exclude invalid intervals
  from lead-time measures and report them.
- Validate the monthly snapshot calendar and display the selected snapshot date.
  Do not interpolate monthly inventory into weekly or daily balances.
- Preserve the distinct order statuses found in the source: `delivered`,
  `cancelled`, `returned`, `shipped`, `paid`, and `created`. Grouping statuses for
  a KPI requires an explicit business rule.

# 4. RELATIONSHIPS

Use single-direction dimension-to-fact filtering unless a specific diagnostic requires
another design. Avoid direct relationships that create ambiguous paths.

- Customers[CustomerID] -> Orders[CustomerID]
- Geography[Zip] -> Orders[Zip]
- Geography[Zip] -> Customers[Zip]
- Orders[OrderID] -> OrderItems[OrderID]
- Orders[OrderID] -> Payments[OrderID]
- Orders[OrderID] -> Shipments[OrderID]
- Orders[OrderID] -> Returns[OrderID]
- Orders[OrderID] -> Reviews[OrderID]
- Products[ProductID] -> OrderItems[ProductID]
- Products[ProductID] -> Returns[ProductID]
- Products[ProductID] -> Reviews[ProductID]
- Products[ProductID] -> Inventory[ProductID]
- Date[Date] -> Orders[OrderDate] (active)
- Date[Date] -> Sales[Date] (active benchmark date)
- Date[Date] -> WebTraffic[Date] (active)
- Date[Date] -> Shipments[ShipDate] (inactive role)
- Date[Date] -> Shipments[DeliveryDate] (inactive role)
- Date[Date] -> Returns[ReturnDate] (inactive role)
- Date[Date] -> Reviews[ReviewDate] (inactive role)
- Date[Date] -> Inventory[SnapshotDate] (inactive role)

Use the appropriate inactive date role explicitly for shipment, delivery, return,
review, and inventory trends. Do not create a direct Reviews-to-Customers relationship
because Reviews already connect to customers through Orders.

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

# 6. DASHBOARD DESIGN

## 6.1. Design Objective and Page Count

Build **seven report pages**. The report must use a consistent top-down reading order:
context and controls first, headline results second, trends and comparisons third,
and diagnostic detail last. Do not compress content to fit a default canvas. Use a
custom page size of approximately **1,920 x 1,400 pixels** and allow additional
vertical space when a page needs more detail. Every page must remain readable at
normal browser zoom without overlapping visuals.

The page sequence is:

1. Executive Overview
2. Sales and Profitability
3. Customers and Market Performance
4. Inventory Performance
5. Delivery and Logistics
6. Returns and Product Quality
7. Customer Satisfaction

Cross-domain diagnostics and data-quality caveats are embedded in the seven analytical
pages. Each page must expose relevant sample sizes, date-basis notes, source-coverage
warnings, and interpretation guidance so users do not need a separate definitions page.

## 6.2. Common Page Structure

Use the following top-down structure on every analytical page unless a page-specific
layout below states otherwise:

### Row 1: Page context

- Page title on the left, using the same title as the navigation label.
- One-sentence purpose statement below the title.
- Selected reporting period, last data date, and refresh timestamp on the right.
- A small status note when the page uses a special date role, snapshot date, cohort,
  or incomplete source population.

### Row 2: Filters and navigation

- Persistent page navigation: Executive Overview, Sales, Customers, Inventory,
  Delivery, Returns, Satisfaction, Diagnostics, and Data Quality.
- Report-level slicers: reporting period, year, month, product category, product,
  customer segment, sales channel, and geography where relevant.
- Page-specific slicers beside the common slicers rather than in a separate hidden
  area.
- Use dropdown slicers for high-cardinality fields and horizontal button slicers for
  short status or category lists.
- Display the active filter state clearly and provide a Reset Filters action.

### Row 3: Headline KPIs

- Use four to six KPI cards, ordered from the most important business outcome to
  supporting operational measures.
- Each card must show the current value, unit, comparison period where meaningful,
  and a clear blank or unavailable state when its denominator is invalid.
- Use consistent number formats: currency for monetary values, whole numbers for
  units and orders, one decimal place for durations, and percentages for rates.
- Do not use colour as the only indication of status; pair it with text, an icon, or
  a reference line.

### Row 4: Main time trend

- Place the primary weekly, monthly, or yearly trend across most of the canvas width.
- Use line or clustered-column charts for trends and show the selected period against
  an equivalent comparison period when the comparison is valid.
- Label the date basis in the visual subtitle, such as “by Order Date,” “by Return
  Date,” or “by Snapshot Date.”
- Do not mix incompatible date roles in one trend.

### Row 5: Comparisons and drivers

- Use two or three aligned visuals for product, category, channel, customer, region,
  status, reason, or threshold comparisons.
- Prefer sorted horizontal bars, column charts, decomposition trees, or diagnostic
  comparison charts
  over crowded legends.
- Keep the same measure, unit, and filter context within a visual.
- Show the denominator or sample size when a rate, average, or rating could be
  misleading.

### Row 6: Detail and action

- Place a matrix or detail table at the bottom with the dimensions most useful for
  investigation.
- Use conditional formatting only when the rule is defined and the underlying
  measure is valid.
- Use clear chart subtitles, caveat notes, and detail-table labels to describe
  observed patterns without claiming causation.
- Use drillthrough from summary visuals to the relevant detail page or table where
  the model supports it.

## 6.3. Page Specifications

### Page 1: Executive Overview

**Purpose:** Give the Supply Chain Manager a single starting point for the selected
period and identify where deeper investigation is required.

Top-down content:

1. Filters: period, year/month, category, sales channel, and region.
2. KPI cards: Net Sales, Total Orders, Units Sold, AOV, Gross Profit or COGS when
   available, and Fulfillment Rate.
3. Main trend: Net Sales and Total Orders by month, with separate axes only when
   clearly labelled.
4. Business balance: a row of compact visuals for category sales, stockout products,
   transit within three days, return rate, and average rating.
5. Action table: top products or categories requiring attention, including sales,
   stock status, delivery, returns, and rating indicators.

Do not present a combined health score unless its weights and thresholds are approved.

### Page 2: Sales and Profitability

**Purpose:** Explain changes in sales value, order volume, discounting, and
profitability.

Top-down content:

1. Filters: period, product/category, customer segment, sales channel, order status,
   and promotion presence.
2. KPI cards: Gross Sales, Discount Amount, Net Sales, Total Orders, Units Sold, and
   AOV. Add COGS, Gross Profit, and Gross Margin only when the cost basis is confirmed.
3. Main trend: Net Sales, Orders, Units, and AOV by the selected period using separate
   visuals when scales would obscure the trend.
4. Driver visuals: sales and gross profit by category/product, sales by order source
   and device, and discount rate by category or period.
5. Detail matrix: product, category, units, orders, gross sales, discounts, net
   sales, COGS, profit, and margin, with unavailable measures clearly marked.

Keep catalog price and COGS benchmarks visually separate from realized sales measures.

### Page 3: Customers and Market Performance

**Purpose:** Show who purchases, where purchasing occurs, and how customer segments
perform.

Top-down content:

1. Filters: period, acquisition channel, age group, gender, region, city, order
   source, and device type.
2. KPI cards: Distinct Customers, New Customers, Repeat Customers or Repeat Rate,
   Total Orders, Revenue per Customer, and AOV.
3. Main trend: Distinct Customers, Orders, and Net Sales by month.
4. Segment visuals: customers and revenue by acquisition channel, age group, gender,
   region, and city. Use a map only when geographic density is readable; otherwise
   use ranked bars.
5. Cohort/detail matrix: first eligible order period, customer count, orders, net
   sales, revenue per customer, and order frequency.

Keep acquisition channel and order sales channel in separate visuals and labels.

### Page 4: Inventory Performance

**Purpose:** Monitor stock availability and movement at the supported monthly
snapshot grain.

Top-down content:

1. Snapshot selector: snapshot date, year/month, category, segment, and product.
   Display the selected snapshot date prominently.
2. KPI cards: Stock on Hand, Inventory Value, Stock In, Stock Out, Low-Stock
   Products, and Out-of-Stock Products.
3. Main trend: Stock on Hand, Stockout Days, and flagged-product counts by monthly
   snapshot date. Do not interpolate snapshots into daily or weekly values.
4. Driver visuals: stock status by category, reorder candidates, days of supply,
   fill rate, and sell-through rate.
5. Action table: product, category, snapshot date, stock on hand, stockout days,
   days of supply, fill rate, reorder flag, and overstock flag.

Use one snapshot balance per product in totals. Do not sum stock balances across
multiple snapshot dates.

### Page 5: Delivery and Logistics

**Purpose:** Monitor shipment progress, transit duration, shipping cost, and order
status without confusing transit time with a contractual late-delivery result.

Top-down content:

1. Filters: order date, ship date, delivery date, order status, device, order source,
   product/category, and geography.
2. KPI cards: Shipment Cost, Shipped Orders, Delivered Orders, Average Transit Time,
   Transit Within 3 Days, and Fulfillment Rate.
3. Main trend: shipment count, delivery count, and average transit time by ship or
   delivery month, with the selected date basis stated.
4. Driver visuals: order-status mix, transit-time distribution, transit threshold by
   region/channel, and shipping cost by channel or product category.
5. Detail table: order, order date, ship date, delivery date, transit days, status,
   shipping fee, customer geography, and return indicator.

Do not label the three-day threshold as “on time” or “late” unless an approved
service-level rule exists.

### Page 6: Returns and Product Quality

**Purpose:** Identify the products, reasons, and segments associated with returns and
refunds.

Top-down content:

1. Filters: return date, order cohort, product/category, customer segment, promotion
   presence, return reason, and geography.
2. KPI cards: Returned Orders, Return Quantity, Refund Amount, Return Rate, and Net
   Sales After Returns when the cohort is aligned.
3. Main trend: returned orders, return quantity, and refund amount by return month.
4. Driver visuals: return rate by product/category, return quantity and refund amount
   by reason, and return rate by customer segment or channel.
5. Action table: product, category, return reason, returned orders, return quantity,
   refund amount, return rate, and linked delivery or rating indicators.

Keep return-date activity trends separate from order-cohort reconciliation visuals.
Use return reasons as investigation categories, not as proof of causation.

### Page 7: Customer Satisfaction

**Purpose:** Evaluate ratings and review volume alongside sales, delivery, and return
patterns.

Top-down content:

1. Filters: review date, product/category, customer segment, region, order source,
   and rating band.
2. KPI cards: Average Rating, Review Count, Positive Review Rate, Negative Review
   Rate, and the share of products below the approved rating threshold.
3. Main trend: Average Rating and Review Count by review month.
4. Driver visuals: rating distribution, average rating by product/category, review
   volume by category, and rating versus return rate or transit time.
5. Detail table: product, category, average rating, review count, positive rate,
   negative rate, return rate, and delivery indicators.

Flag low-review samples and avoid ranking products by rating alone. Written review
titles may be shown for qualitative inspection but must not be treated as a numeric KPI.

### Integrated cross-domain and data-quality perspectives

Cross-domain comparisons and data-quality caveats are distributed across the seven
analytical pages. Use scatter views for relationships such as product value versus
rating, stock availability versus sell-through, transit versus shipping cost, and
return rate versus refund impact. Surface sample counts, date roles, missing-record
warnings, and metric assumptions in chart subtitles, caveat notes, and detail tables.

## 6.4. Visual and Interaction Standards

- Use a restrained colour system: one primary colour for current performance, one
  comparison colour, and semantic warning colours only for defined thresholds.
- Use consistent titles in the form “Measure by Dimension” and include the date basis
  or snapshot basis in the subtitle.
- Place legends above or inside the visual only when they remain readable. Prefer
  direct labels for two-series comparisons.
- Avoid decorative visuals, three-dimensional charts, gauges without an approved
  target, and pie charts with many categories.
- Enable cross-highlighting only when the interaction preserves the denominator and
  date context. Use drillthrough for detailed order, product, or exception analysis.
- Add accessible alt text, meaningful tab order, keyboard-accessible navigation, and
  sufficient contrast. Do not communicate a result through colour alone.
- Keep slicers and visual interactions synchronized across pages only when they have
  the same semantic meaning. A return date filter must not silently behave as an
  order date filter.
- Every page must include a visible reset-filters action and a clear indication when
  a metric is unavailable because its required field, denominator, or business rule
  has not been confirmed.
