# 1. PURPOSES OF THE DASHBOARD
## 1.1. Who will use the dashboard?
- The main user of the dashboard is the Supply Chain Manager
## 1.2. Which decisions should the dashboard support?
- The dashboard aims to provide the Supply Chain Manager a general view of the supply chain performance, including sales/orders, customer and their satisfaction, inventory, shipments, and returns.
- The Supply Chain Manager can base on the dashboard to monitor commercial performance, identify operational bottlenecks, and understand the factors influencing revenue, profitability, inventory efficiency, customer satisfaction, shipments, and returns. 
## 1.3. How often will those decisions be made?
- The dashboard only aims to provide general information by weekly, monthly, and yearly basis. It does not need to go detail to daily.
## 1.4. What are key concerns of the Supply Chain Manager?
- Sales and profitability: Track sales (units sold, revenuve, and order volume), discounts/promotions, COGS and gross profit over time. Identify high-performing/low-performing products, categories, channels, customers.
- Inventory performance: Monitor stock availability, inbound inventory, sales movement, and fill metrics at product and monthly snapshot level. Detect potential stockout risks, excess inventory, slow-moving products, and products where inventory constraints may be limiting sales.
- Customers and market performance: 
- Delivery and logistics performance: Measure shipment progress from order date to shipping and delivery date, including delivery lead time, shipping fees, and order status. Highlight delayed deliveries or shipping-related issues that may affect customer satisfaction and returns.
- Returns and product quality: Track return quantity, refund value, return rate, and return reasons by product, category, customer segment, promotion, and period. The goal is to identify products or operational issues that drive refunds and reduce net revenue.
- Customer satisfaction: Review star ratings and written review data to assess customer sentiment toward products. Combine review ratings with delivery performance, return behavior, and product sales to identify likely drivers of positive or negative customer experience.

# 2. METRICS
The dashboard must let the Supply Chain Manager review performance by week, month, and year; daily detail is out of scope. Metrics should be filterable and comparable by the relevant product, category, customer segment, sales channel, geography, and period. Use the formulas below as metric definitions; table and column names refer to the semantic model described in Sections 3-4. Noted: **Status: Core** means the measure is already implemented. **Status: Planned** means it can be calculated from the listed source data but is not yet implemented as a semantic-model measure.
## 2.1. Sales & profitability
| Metric | Definition | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Gross Sales | Value of order items before discounts | `SUMX(OrderItems, OrderItems[Quantity] * OrderItems[UnitPrice])` | order_items.csv (`quantity`, `unit_price`) | Core |
| Discount Amount | Total discount value applied to sold units | `SUMX(OrderItems, OrderItems[Quantity] * OrderItems[DiscountAmount])` | order_items.csv (`quantity`, `discount_amount`) | Planned |
| Net Sales | Revenue after per-unit discounts; primary revenue measure | `SUMX(OrderItems, OrderItems[Quantity] * (OrderItems[UnitPrice] - OrderItems[DiscountAmount]))` | order_items.csv (`quantity`, `unit_price`, `discount_amount`) | Core |
| Discount Rate | Discount value as a share of gross sales | `DIVIDE([Discount Amount], [Gross Sales])` | order_items.csv | Planned |
| Units Sold | Total quantity sold | `SUM(OrderItems[Quantity])` | order_items.csv (`quantity`) | Core |
| Total Orders | Distinct order count; show status mix separately | `DISTINCTCOUNT(Orders[OrderID])` | orders.csv (`order_id`) | Core |
| Average Order Value (AOV) | Net sales per order | `DIVIDE([Net Sales], [Total Orders])` | order_items.csv, orders.csv | Core |
| COGS | Product cost for units sold | `SUMX(OrderItems, OrderItems[Quantity] * RELATED(Products[COGS]))` | order_items.csv (`quantity`), products.csv (`cogs`) | Planned |
| Gross Profit | Net sales less COGS | `[Net Sales] - [COGS]` | order_items.csv, products.csv | Planned |
| Gross Margin % | Gross profit as a share of net sales | `DIVIDE([Gross Profit], [Net Sales])` | order_items.csv, products.csv | Planned |
| Revenue Growth (YoY) | Percentage change in net sales versus the comparable prior-year period | `DIVIDE([Net Sales] - CALCULATE([Net Sales], DATEADD(Date[Date], -1, YEAR)), CALCULATE([Net Sales], DATEADD(Date[Date], -1, YEAR)))` | Date, order_items.csv | Core |
| Order Growth (YoY) | Percentage change in order count versus the comparable prior-year period | `DIVIDE([Total Orders] - CALCULATE([Total Orders], DATEADD(Date[Date], -1, YEAR)), CALCULATE([Total Orders], DATEADD(Date[Date], -1, YEAR)))` | Date, orders.csv | Core |
| Like-for-Like Store Sales Growth (YoY) | Percentage change in net sales for the same stores operating in both the current and comparable prior-year periods; excludes the effect of store openings and closures | `DIVIDE([Current-Period Net Sales for Comparable Stores] - [Prior-Year Net Sales for Comparable Stores], [Prior-Year Net Sales for Comparable Stores])`; calculate both period totals over the intersection of stores eligible in both periods | Date, store-level sales/orders, store dimension with opening and closure dates | Planned |
| Product / Category / Channel Performance | Compare net sales, units, orders, and gross profit across products, categories, and order channels | `[Net Sales]`, `[Units Sold]`, `[Total Orders]`, `[Gross Profit]` by the relevant dimension | order_items.csv, orders.csv, products.csv | Core |
| Contribution % | Selected product/category's share of net sales, retaining other report filters | `DIVIDE([Net Sales], CALCULATE([Net Sales], REMOVEFILTERS(Products[Category])))` (for category contribution) | order_items.csv, products.csv | Core |
| Catalog Benchmarks | Average catalog price and COGS; use for product comparison, not as realized sales or profit | `AVERAGE(Products[Price])`, `AVERAGE(Products[COGS])` | products.csv (`price`, `cogs`) | Core |
## 2.2. Inventory performance
| Metric | Definition | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Current Stock | Units on hand for the selected/latest product snapshot | `SUM(Inventory[StockOnHand])` in the selected snapshot context | inventory.csv (`stock_on_hand`) | Core |
| Inventory Value | Selected-snapshot stock valued at catalog price | `SUMX(Inventory, Inventory[StockOnHand] * RELATED(Products[Price]))` | inventory.csv (`stock_on_hand`), products.csv (`price`) | Core |
| Stock In | Units received during the selected snapshot period | `SUM(Inventory[UnitsReceived])` | inventory.csv (`units_received`) | Core |
| Stock Out | Units sold during the selected snapshot period | `SUM(Inventory[UnitsSold])` | inventory.csv (`units_sold`) | Core |
| Low-Stock Products | Distinct products at or below the 20-unit threshold in the selected snapshot | `DISTINCTCOUNT` of products where `Inventory[StockOnHand] <= 20` | inventory.csv (`stock_on_hand`) | Core |
| Out-of-Stock Products | Distinct products with zero stock in the selected snapshot | `DISTINCTCOUNT` of products where `Inventory[StockOnHand] = 0` | inventory.csv (`stock_on_hand`) | Core |
| Stockout Days | Average days out of stock in the snapshot period | `AVERAGE(Inventory[StockoutDays])` | inventory.csv (`stockout_days`) | Planned |
| Stockout / Overstock Rate | Share of product snapshots flagged as stockout / overstock, within the selected snapshot period | `DIVIDE(CALCULATE(COUNTROWS(Inventory), Inventory[StockoutFlag] = 1), COUNTROWS(Inventory))`; use `OverstockFlag` for overstock rate | inventory.csv (`stockout_flag`, `overstock_flag`) | Planned |
| Reorder Candidates | Products flagged for reorder in the selected snapshot | `DISTINCTCOUNT` of products where `Inventory[ReorderFlag] = 1` | inventory.csv (`reorder_flag`) | Planned |
| Days of Supply | Average expected days current stock will last at the recorded sales rate | `AVERAGE(Inventory[DaysOfSupply])` | inventory.csv (`days_of_supply`) | Planned |
| Fill Rate | Average share of demand fulfilled from available stock | `AVERAGE(Inventory[FillRate])` | inventory.csv (`fill_rate`) | Planned |
| Sell-Through Rate | Average share of received stock sold | `AVERAGE(Inventory[SellThroughRate])` | inventory.csv (`sell_through_rate`) | Planned |
| Average Stock on Hand | Average stock balance across product snapshots in the selected period; not a days-of-inventory measure | `AVERAGE(Inventory[StockOnHand])` | inventory.csv (`stock_on_hand`) | Core |
## 2.3. Customers and market performance
| Metric | Definition / requirement | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Distinct Customers | Unique customers with orders in the selected period | `DISTINCTCOUNT(Orders[CustomerID])` | orders.csv (`customer_id`) | Core |
| Revenue per Customer | Net sales per distinct buying customer | `DIVIDE([Net Sales], [Distinct Customers])` | order_items.csv, orders.csv | Core |
| Customer Segments | Compare customer count, orders, and net sales by acquisition channel, age group, and gender | `[Distinct Customers]`, `[Total Orders]`, `[Net Sales]` by customer attributes | customers.csv, orders.csv | Core |
| Customer Geography | Compare customer count, orders, and net sales by region and city | `[Distinct Customers]`, `[Total Orders]`, `[Net Sales]` by `Geography[Region]` / `Geography[City]` | geography.csv, customers.csv, orders.csv | Core |
| New Customers | Count customers whose signup date is in the selected period | `DISTINCTCOUNT(Customers[CustomerID])` filtered by `Customers[SignupDate]` in the selected period | customers.csv (`signup_date`) | Planned |
| Orders by Sales Channel | Order count split by order source and device type | `[Total Orders]` by `Orders[OrderSource]` / `Orders[DeviceType]` | orders.csv (`order_source`, `device_type`) | Core |
## 2.4. DELIVERY AND LOGISTICS PERFORMANCE

| Metric | Definition / requirement | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Shipment Cost | Total shipping fees | `SUM(Shipments[ShippingFee])` | shipments.csv (`shipping_fee`) | Core |
| Average Transit Time | Average days from ship date to delivery date for delivered shipments | `AVERAGEX(FILTER(Shipments, NOT ISBLANK(Shipments[DeliveryDate])), DATEDIFF(Shipments[ShipDate], Shipments[DeliveryDate], DAY))` | shipments.csv (`ship_date`, `delivery_date`) | Core |
| On-Time Delivery Rate | Share of delivered shipments delivered within 3 days of ship date; treat 3 days as the current reporting threshold | `DIVIDE(CALCULATE(DISTINCTCOUNT(Shipments[OrderID]), FILTER(Shipments, NOT ISBLANK(Shipments[DeliveryDate]) && DATEDIFF(Shipments[ShipDate], Shipments[DeliveryDate], DAY) <= 3)), CALCULATE(DISTINCTCOUNT(Shipments[OrderID]), FILTER(Shipments, NOT ISBLANK(Shipments[DeliveryDate]))))` | shipments.csv (`ship_date`, `delivery_date`) | Core |
| Late Delivery Rate | Share of delivered shipments taking more than 3 days from ship to delivery | `DIVIDE(CALCULATE(DISTINCTCOUNT(Shipments[OrderID]), FILTER(Shipments, NOT ISBLANK(Shipments[DeliveryDate]) && DATEDIFF(Shipments[ShipDate], Shipments[DeliveryDate], DAY) > 3)), CALCULATE(DISTINCTCOUNT(Shipments[OrderID]), FILTER(Shipments, NOT ISBLANK(Shipments[DeliveryDate]))))` | shipments.csv (`ship_date`, `delivery_date`) | Planned |
| Order-to-Delivery Lead Time | Average days from order date through delivery for delivered orders | Average `DATEDIFF(Orders[OrderDate], Shipments[DeliveryDate], DAY)` over delivered orders | orders.csv (`order_date`), shipments.csv (`delivery_date`) | Planned |
| Order Status Counts | Count orders by delivered, shipped, pending (created/paid), and cancelled status | `[Total Orders]` filtered by `Orders[OrderStatus]` | orders.csv (`order_status`) | Core |
| Fulfillment Rate | Share of orders reaching delivered status | `DIVIDE(CALCULATE([Total Orders], Orders[OrderStatus] = "delivered"), [Total Orders])` | orders.csv (`order_status`) | Core |
| Order Status Mix | Share of orders represented by each status | `DIVIDE([Total Orders], CALCULATE([Total Orders], REMOVEFILTERS(Orders[OrderStatus])))` | orders.csv (`order_status`) | Core |

Report shipment progress, transit time, shipping fees, and order status over time, and
allow the manager to identify delayed deliveries and investigate their relationship to
customer satisfaction and returns.

## 2.5. RETURNS AND PRODUCT QUALITY

| Metric | Definition / requirement | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Returned Orders | Distinct orders with at least one return | `DISTINCTCOUNT(Returns[OrderID])` | returns.csv (`order_id`) | Core |
| Return Quantity | Total units returned | `SUM(Returns[ReturnQuantity])` | returns.csv (`return_quantity`) | Core |
| Refund Amount | Total recorded refund value | `SUM(Returns[RefundAmount])` | returns.csv (`refund_amount`) | Core |
| Return Rate | Share of orders with at least one return | `DIVIDE([Returned Orders], [Total Orders])` | returns.csv, orders.csv | Core |
| Net Sales After Returns | Net sales less recorded refunds | `[Net Sales] - [Refund Amount]` | order_items.csv, returns.csv | Planned |
| Returns by Reason | Return quantity and refund amount split by reason | `[Return Quantity]` / `[Refund Amount]` by `Returns[ReturnReason]` | returns.csv (`return_reason`) | Core |
| Returns by Product / Category | Return quantity and refund amount split by SKU or category | `[Return Quantity]` / `[Refund Amount]` by product/category | returns.csv, products.csv | Core |

Show return quantity, refund value, and return rate by return period, product/category,
customer segment, promotion (where linked), and reason. Use the return date for return
activity trends; use the associated order cohort when reconciling refunds to order sales.
These views should identify recurring product-quality or operational issues that reduce
net revenue.

## 2.6. CUSTOMER SATISFACTION

| Metric | Definition / requirement | Formula | Source | Status |
| --- | --- | --- | --- | --- |
| Average Rating | Mean star rating across submitted reviews | `AVERAGE(Reviews[Rating])` | reviews.csv (`rating`) | Core |
| Review Count | Number of submitted reviews | `COUNTROWS(Reviews)` | reviews.csv (`review_id`) | Core |
| Positive Review Rate | Share of reviews rated 4 or 5 stars | `DIVIDE(CALCULATE(COUNTROWS(Reviews), Reviews[Rating] >= 4), [Review Count])` | reviews.csv (`rating`) | Core |
| Negative Review Rate | Share of reviews rated 1 or 2 stars | `DIVIDE(CALCULATE(COUNTROWS(Reviews), Reviews[Rating] <= 2), [Review Count])` | reviews.csv (`rating`) | Core |
| Rating by Product / Category | Average rating and review count by product or category | `AVERAGE(Reviews[Rating])`, `COUNTROWS(Reviews)` by product/category | reviews.csv, products.csv | Core |
| Rating and Return Diagnostic | Compare ratings and return rate by product/category to highlight possible quality issues; use as a diagnostic, not proof of causation | `[Average Rating]` and `[Return Rate]` by product/category | reviews.csv, returns.csv, products.csv | Planned |

Use review ratings with delivery performance, returns, and sales to investigate likely
drivers of positive or negative customer experience. Written review text may support
qualitative inspection but is not a numeric KPI.

## 2.7. CROSS-DOMAIN DIAGNOSTICS

| Diagnostic | Requirement / definition | Formula or view | Source | Status |
| --- | --- | --- | --- | --- |
| Sales and Order Trend | Compare net sales, order count, and units over the same weekly, monthly, or yearly period to distinguish changes in order volume from changes in value per order | `[Net Sales]`, `[Total Orders]`, `[Units Sold]` by `Date` period | orders.csv, order_items.csv | Core |
| Stockout Risk to Sales | Identify products with stockout flags/days alongside their sales movement; do not present a lost-revenue estimate without a validated demand model | Inventory stockout measures and `[Net Sales]` / `[Units Sold]` by product and period | inventory.csv, order_items.csv, products.csv | Planned |
| Returns and Delivery Diagnostic | Compare return rate for delivered orders above and within the 3-day transit threshold | `[Return Rate]` by derived on-time/late shipment group | shipments.csv, returns.csv, orders.csv | Planned |
| Promotion Performance | Compare net sales, units, discount amount, and return rate for order lines with and without a promotion reference; interpret directionally because promotion references may be blank or incomplete | `[Net Sales]`, `[Units Sold]`, `[Discount Amount]`, `[Return Rate]` by promotion presence | order_items.csv (`promo_id`, `promo_id_2`), promotions.csv, returns.csv | Planned |

# 3. PREPARE THE DATA

## 3.1. SOURCES FILES AND GRAINS

| Fact/Dimension | File | Grain | Key columns | Information |
| --- | --- | --- | --- | --- |
| Fact | orders.csv | 1 row per order | order_id | Transaction header with order date, status, payment, device, and source. |
| Fact | order_items.csv | 1 row per order-product line | order_id + product_id | Sales quantity, price, discount, and promotion references. |
| Fact | shipments.csv | 1 row per order | order_id | Shipping and delivery dates and shipping fee. |
| Fact | payments.csv | 1 row per order | order_id | Payment method and value. |
| Fact | returns.csv | 1 row per return line | return_id | Product return reasons, quantities, and refund values. |
| Fact | reviews.csv | 1 row per review | review_id | Customer product reviews and star ratings. |
| Fact | web_traffic.csv | 1 row per day/source | date + traffic_source | Daily sessions, visitors, page views, and conversion indicators. |
| Dimension | customers.csv | 1 row per customer | customer_id | Demographic and acquisition metadata for buyers. |
| Fact | sales.csv | 1 row per day | Date | Daily summary of revenue and COGS used as a trend benchmark. |
| Fact | inventory.csv | 1 row per product snapshot per month end | product_id + snapshot_date | Stock position, inbound volume, sales, and fill metrics. |
| Dimension | geography.csv | 1 row per zip code | zip | Geographic mapping for customer and order ZIP codes. |
| Dimension | products.csv | 1 row per SKU | product_id | Product catalog including category, segment, price, and COGS. |
| Dimension | promotions.csv | 1 row per promotion | promo_id | Promotion metadata with type, channel, and applicability. |


## 3.2. INFERRED SCHEMA AND QUALITY NOTES

- No duplicate primary keys were detected in the source files.
- All date columns were valid ISO dates and converted to Power BI Date types.
- Numeric fields were clean and mostly complete; `promo_id` and `promo_id_2` contain blanks by design.
- `customers.csv`, `orders.csv`, `products.csv`, `returns.csv`, `reviews.csv`, and `shipments.csv` all share plausible join keys (`customer_id`, `product_id`, `order_id`, `zip`).
- `inventory.csv` is a monthly snapshot table and is not a transactional fact table; it operates at product-by-date grain.
- `web_traffic.csv` is daily aggregated with no direct join to orders other than date-based analysis.
- The synthetic data spans 2012-07-04 through 2022-12-31 for core commerce tables.
- The `Date[Year-Month]` column is sorted by a hidden `Year-Month Sort` column
  (`Year * 100 + Month Number`) rather than `Month Number` alone, so multi-year
  trend visuals render in true chronological order instead of grouping by
  calendar month across years.

# 4. RELATIONSHIPS

- Customers[CustomerID] -> Orders[CustomerID]
- Geography[Zip] -> Orders[Zip]
- Geography[Zip] -> Customers[Zip]
- Orders[OrderID] -> OrderItems[OrderID]
- Orders[OrderID] -> Payments[OrderID]
- Orders[OrderID] -> Shipments[OrderID] (inactive alternate date role)
- Orders[OrderID] -> Returns[OrderID]
- Orders[OrderID] -> Reviews[OrderID]
- Products[ProductID] -> OrderItems[ProductID]
- Products[ProductID] -> Returns[ProductID]
- Products[ProductID] -> Reviews[ProductID]
- Products[ProductID] -> Inventory[ProductID]
- Date[Date] -> Orders[OrderDate]
- Date[Date] -> Shipments[ShipDate] (inactive)
- Date[Date] -> Shipments[DeliveryDate] (inactive)
- Date[Date] -> Returns[ReturnDate] (inactive)
- Date[Date] -> Reviews[ReviewDate] (inactive)
- Date[Date] -> Inventory[SnapshotDate] (inactive)
- Date[Date] -> WebTraffic[Date]


# 5. ASSUMPTIONS AND CAVEATS

- The model is intentionally designed to reflect the source data rather than inventing downstream facts.
- `order_items.csv` is the primary revenue fact; `sales.csv` acts as a daily summary benchmark rather than a system-of-record revenue source.
- Promotion data is available but not strongly linked to orders because the promotion columns in `order_items.csv` are partly blank and not always one-to-one with the promotion table.
- `web_traffic.csv` is date-based and can support trend analysis and traffic-source segmentation but not direct order-attribution without a separate session identifier.
- Reviews retain their source `CustomerID` column, but the direct Reviews-to-Customers relationship is intentionally omitted because Reviews already connect to Customers through Orders; this avoids an ambiguous filter path.
