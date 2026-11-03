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
The report uses a 1920x1700 canvas and seven pages, following the design in
`dashboard/DASHBOARD_INSTRUCTION.md`:

1. **Executive Overview** - Sales, order, customer, fulfillment, category, region,
   and product summaries.
2. **Sales and Profitability** - Sales trends, discounting, order source, device,
   payment method, and product/category profitability.
3. **Customers and Market Performance** - Buyer frequency, all-time first-time and
   repeat buyer rates, acquisition channel, order source, device, and geography.
4. **Inventory Performance** - Latest stock snapshot, inventory value, stockout and
   overstock flags, replenishment, and inventory exceptions.
5. **Delivery and Logistics** - Shipment coverage, shipping cost, transit duration,
   order status, and regional delivery diagnostics. The three-day split is not an
   on-time KPI because no promised delivery date or service target is available.
6. **Returns and Product Quality** - Return quantity, refund values, reasons,
   category/product performance, and return activity by return month.
7. **Customer Satisfaction** - Review volume and ratings by review month, product,
   and customer segment, with sample counts.
8. **Cross-domain perspectives** - Product, category, inventory, delivery, returns, and satisfaction relationships are embedded in the analytical pages as diagnostic scatter views.

Shipment, return, and review trends use their own date roles. Inventory is monthly
snapshot data and is never presented as daily or weekly stock.

## Key KPIs
- Net Sales
- Gross Sales
- Total Orders
- Distinct Customers
- Units Sold
- Average Order Value
- Revenue per Customer
- Shipment Cost
- Fulfillment Rate
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
