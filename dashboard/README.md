# E-commerce performance dashboard

This Power BI project analyzes the repository's retail and operations data across sales, service, inventory, reviews, and digital traffic.

## Data sources used
- customers.csv
- geography.csv
- products.csv
- orders.csv
- order_items.csv
- payments.csv
- shipments.csv
- returns.csv
- reviews.csv
- inventory.csv
- promotions.csv
- web_traffic.csv
- sales.csv

## Report pages
Each page uses a 1920x1080 canvas with a purple brand header, a KPI card row, and a
grid of charts/tables bound to the semantic model's measures.

1. **Executive Overview** - Net Sales/Orders/Customers/AOV/On-Time Delivery KPIs, a
   monthly Net Sales trend line, Net Sales by Category donut, Orders by Status column
   chart, Net Sales by Region bar chart, and a Segment Performance table.
2. **Sales and Orders** - Net Sales/Gross Sales/Orders/AOV KPIs, a full-width monthly
   Net Sales trend, and Net Sales broken down by Device Type, Order Source, and
   Payment Method, plus a Category Performance table.
3. **Shipping and Returns** - Shipment Cost/On-Time Delivery/Return Rate/Refund
   Amount KPIs, Shipment Cost and On-Time Delivery Rate trends by ship date, Returns
   by Reason donut, Orders by Status column chart, and a Returns by Product Category
   table.
4. **Inventory** - Current Stock/Inventory Value/Low-Stock/Out-of-Stock KPIs, a Stock
   In vs. Stock Out trend by snapshot date, Current Stock by Category donut,
   Inventory Value by Segment bar chart, Low & Out-of-Stock Products by Category
   column chart, and an Inventory Summary table.
5. **Customers and Reviews** - Distinct Customers/Revenue per Customer/Average
   Rating/Positive Review Rate KPIs, Customers by Acquisition Channel, Age Group,
   and Gender charts, a Rating Distribution donut, an Average Rating by Category bar
   chart, and a Customer Segment Summary table.
6. **Web Traffic and Conversion** - Total Sessions/Users/Conversion Rate/Average
   Session Duration KPIs, a Sessions & Visitors monthly trend, Sessions by Traffic
   Source donut, Page Views and Bounce Rate by Traffic Source charts, and a Monthly
   Web Performance table.

All charts and cards are bound to the measures defined in the semantic model (see
`DATA_MODEL.md`) and respect the model's relationship directions - for example,
Shipment-related measures are trended by `Shipments[ShipDate]` directly rather than
through the `Date` table, since the `Shipments` -> `Orders` relationship is inactive.

## Key KPIs
- Net Sales
- Gross Sales
- Total Orders
- Distinct Customers
- Units Sold
- Average Order Value
- Revenue per Customer
- Shipment Cost
- On-Time Delivery Rate
- Return Rate
- Current Stock
- Inventory Value
- Average Rating
- Total Sessions
- Conversion Rate

## Important assumptions
- The model assumes the CSV files are the source of truth and should not be modified.
- Sales and order metrics are driven by `orders.csv` and `order_items.csv`.
- Inventory and web traffic are treated as operational and digital trend tables rather than direct order-level transactions.
- Date-based measures use a single conformed Date table with alternate date roles for shipping and returns.

## Known limitations
- Promotion attribution is limited because a large share of `promo_id` fields in `order_items.csv` is blank.
- `web_traffic.csv` supports trend analysis by channel and date but not customer-level attribution unless extra session or campaign tables are added.
- The raw data is synthetic and may not represent a deployed ERP environment.

## Open the dashboard
1. Open `dashboard/dashboard.pbip` in Power BI Desktop.
2. Refresh the data if Power BI requests it.

## Data path note
- Power BI Desktop requires absolute file paths for these CSV partitions. The current project is configured for `D:\projects\VinTelligence2026\data`. If the repository is moved, update the CSV source paths in Power Query to the new absolute `data` folder.
