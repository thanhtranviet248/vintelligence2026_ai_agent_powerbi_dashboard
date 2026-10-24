# Dashboard Project File Guide

This guide describes the Power BI project in `dashboard.Report` and
`dashboard.SemanticModel`. The `.Report` folder defines the report pages,
visuals, layout, and theme. The `.SemanticModel` folder defines the data model,
relationships, measures, and imported data.

The report connects to the local semantic model through
`dashboard.Report/definition.pbir`. Visuals in the report bind to the entity,
column, and measure names defined in the semantic model.

## 1. Purpose of Each Folder

| Folder | Purpose |
| --- | --- |
| `dashboard.Report` | Power BI report artifact: page order, page settings, visual definitions, themes, resources, and local report settings. |
| `dashboard.Report/definition` | The authored PBIR report definition. This contains report metadata, pages, and each visual's JSON definition. |
| `dashboard.Report/definition/pages` | One subfolder per report page, plus `pages.json`, which determines visible page order and the active page. |
| `dashboard.Report/definition/pages/<page-id>/visuals` | One subfolder per visual on that page. Each `visual.json` defines the visual type, position, formatting, and model fields/measures it uses. |
| `dashboard.Report/StaticResources` | Theme and other report resources referenced by the report definition. |
| `dashboard.Report/StaticResources/RegisteredResources` | Report-specific resources, including the custom theme `purple-slicer.json`. |
| `dashboard.Report/StaticResources/SharedResources/BaseThemes` | Base theme JSON files used by the report. |
| `dashboard.Report/.pbi` | Per-user/local Power BI Desktop state. It is not the source of the report's visual or model logic. |
| `dashboard.SemanticModel` | Power BI semantic model artifact: tables, columns, measures, Power Query imports, and relationships. |
| `dashboard.SemanticModel/definition` | The authored TMDL model definition. |
| `dashboard.SemanticModel/definition/tables` | One TMDL file per model table. Each defines columns, data types, measures, and usually the Power Query partition that loads a CSV. |
| `dashboard.SemanticModel/definition/cultures` | Culture and localization metadata for model labels and formatting. |
| `dashboard.SemanticModel/.pbi` | Local Power BI Desktop model editor settings and cache. These are not business definitions. |

## 2. Files and Their Purposes

### 2.1. `dashboard.Report`

#### Root files

| File | Purpose | What to change |
| --- | --- | --- |
| `definition.pbir` | Report entry point. Its `datasetReference.byPath.path` points to `../dashboard.SemanticModel`. | Change the path only if the semantic model folder is renamed or moved. Keep it aligned with the actual folder structure. |
| `.platform` | Fabric/Power BI artifact metadata, including artifact type, display name, and logical ID. | Change display metadata when renaming the artifact; avoid changing the logical ID unless intentionally creating/rebinding an artifact. |
| `.pbi/localSettings.json` | Machine/user-specific Power BI Desktop settings. | Normally leave alone; it is not where report content or layout is authored. |

#### Report definition files

| File | Purpose | What to change |
| --- | --- | --- |
| `definition/report.json` | Report-level settings, theme references, resource packages, and report behavior such as enhanced tooltips and visual interactions defaults. | Update theme/resource registrations when adding or removing a theme/resource. Adjust report-level settings only when the intended behavior should apply across the report. |
| `definition/version.json` | PBIR report definition version metadata. | Normally maintained by Power BI Desktop; avoid hand-editing unless performing a deliberate format migration. |
| `definition/pages/pages.json` | `pageOrder` lists the page IDs shown in the report and in what order; `activePageName` sets the page opened by default. | Reorder page IDs, choose the initial page, or add/remove page IDs here in sync with page folders and page metadata. |

#### Pages

Each page folder contains `page.json` and a `visuals` folder. The page IDs are
stable identifiers; do not rename them casually because `pages.json` refers to
them.

| Page ID | Display name | Page definition |
| --- | --- | --- |
| `9f7b1d2d-6d9d-4cb7-b1f6-a9d4a0b2f712` | Executive Overview | `definition/pages/9f7b1d2d-6d9d-4cb7-b1f6-a9d4a0b2f712/page.json` |
| `2d466881-2b4b-4e8e-8b9c-f3c20a7f34d5` | Sales and Orders | `definition/pages/2d466881-2b4b-4e8e-8b9c-f3c20a7f34d5/page.json` |
| `fcd9a8b8-0d77-4a8f-8e54-6c7101183f6a` | Shipping and Returns | `definition/pages/fcd9a8b8-0d77-4a8f-8e54-6c7101183f6a/page.json` |
| `5c7bc72d-0718-4ad3-b959-68d16b1ac7af` | Inventory | `definition/pages/5c7bc72d-0718-4ad3-b959-68d16b1ac7af/page.json` |
| `cb86d456-7f72-4bba-b1d9-0cf2b39f1ef3` | Customers and Reviews | `definition/pages/cb86d456-7f72-4bba-b1d9-0cf2b39f1ef3/page.json` |
| `41c9dcef-b0c8-41b4-9ff2-c53f0d8d8e8b` | Web Traffic and Conversion | `definition/pages/41c9dcef-b0c8-41b4-9ff2-c53f0d8d8e8b/page.json` |

Each listed page's `page.json` defines the display name, canvas dimensions
(currently 1920 by 1080), display option, and any page-level settings. Change
the page dimensions or display name there. If adding a page, also add its ID to
`pages.json`.

There is also a page folder named
`definition/pages/8e30a6d6bf64860c8c71/page.json`, whose display name is `Page 1`.
It is not in `pages.json`'s page order and has no visuals, so it is currently an
unlisted/unused page. To make it a report page, give it a meaningful display
name and add its ID to `pageOrder`; otherwise it can be removed through Power BI
Desktop after confirming it is not needed.

#### Visual definition files

Each listed visual is stored at
`definition/pages/<page-id>/visuals/<visual-folder>/visual.json`. The individual
JSON file, rather than the folder name, is the visual definition. On every
complete page:

- `hdr-band/visual.json`, `hdr-title/visual.json`, and `hdr-brand/visual.json`
  create the purple title band, page title/subtitle, and brand label.
- `kpi-*/visual.json` files define KPI cards.
- `p*-*/visual.json` files define that page's charts and tables.

| Page | Visual folders and purpose |
| --- | --- |
| Executive Overview | `kpi-net-sales` (Net Sales card), `kpi-total-orders` (order-count card), `kpi-distinct-customers` (customer-count card), `kpi-aov` (average-order-value card), `kpi-otd` (on-time-delivery card), `p1-trend-net-sales` (monthly sales line chart), `p1-donut-category` (sales by product category), `p1-col-orderstatus` (orders by status), `p1-bar-region` (sales by region), `p1-table-segment` (segment summary). |
| Sales and Orders | `kpi-net-sales` (Net Sales card), `kpi-gross-sales` (Gross Sales card), `kpi-total-orders` (order-count card), `kpi-aov` (average-order-value card), `p2-trend-net-sales` (monthly sales line chart), `p2-col-device` (sales by device type), `p2-col-source` (sales by order source), `p2-bar-payment` (sales by payment method), `p2-table-category` (category summary). |
| Shipping and Returns | `kpi-shipment-cost` (shipping-cost card), `kpi-otd` (on-time-delivery card), `kpi-return-rate` (return-rate card), `kpi-refund-amount` (refund card), `p3-trend-shipcost` (shipping-cost trend), `p3-trend-otd` (on-time-delivery trend), `p3-donut-reason` (returns by reason), `p3-col-orderstatus` (orders by status), `p3-table-returns-category` (returns by product category). |
| Inventory | `kpi-current-stock` (current stock card), `kpi-inventory-value` (stock value card), `kpi-low-stock` (low-stock card), `kpi-out-of-stock` (out-of-stock card), `p4-trend-stock` (stock received vs. sold trend), `p4-donut-category` (stock by category), `p4-bar-segment` (inventory value by segment), `p4-col-lowstock` (low/out-of-stock products by category), `p4-table-category` (inventory summary). |
| Customers and Reviews | `kpi-distinct-customers` (customer-count card), `kpi-revenue-per-customer` (revenue-per-customer card), `kpi-avg-rating` (average-rating card), `kpi-positive-review-rate` (positive-review card), `p5-col-channel` (customers by acquisition channel), `p5-donut-age` (customers by age group), `p5-donut-rating` (rating distribution), `p5-bar-rating-category` (average rating by category), `p5-col-gender` (customers by gender), `p5-table-agegroup` (customer segment summary). |
| Web Traffic and Conversion | `kpi-total-sessions` (sessions card), `kpi-users` (visitor card), `kpi-conversion-rate` (conversion card), `kpi-avg-session` (session-duration card), `p6-trend-sessions` (monthly sessions and visitors trend), `p6-donut-source` (sessions by traffic source), `p6-col-pageviews` (page views by source), `p6-bar-bounce` (bounce rate by source), `p6-table-monthly` (monthly web performance). |

To change a visual's **position or size**, edit its `position` fields (`x`, `y`,
`width`, `height`) in its `visual.json`; coordinates use the page canvas.
To change a visual's **data**, edit its query projections under
`visual.query.queryState` (category/axis/values/series) and ensure the referenced
entity and property exist in the semantic model. To change a title, colors,
labels, axes, legend, or other formatting, update the corresponding
`visual.objects` or `visual.visualContainerObjects` entries.

### 2.2. Report resources

| File | Purpose | What to change |
| --- | --- | --- |
| `StaticResources/RegisteredResources/purple-slicer.json` | Custom theme palette; its `dataColors` array supplies the report's data-series colors. | Change palette hex values to restyle chart series colors. |
| `StaticResources/SharedResources/BaseThemes/purple-slicer.json` | Base theme resource referenced by `report.json`; provides the purple palette/base theme identity. | Change theme properties here only when intentionally changing the base theme. |
| `StaticResources/SharedResources/BaseThemes/Fluent2-CY26SU08.json` | Additional Fluent 2 base theme resource shipped with the project. | Use/select it through Power BI theme configuration if desired; do not assume it is active just because the file exists. |

The project does not currently register decorative page-background images in
`report.json`; the visible page design is built from report visuals.

## 3. `dashboard.SemanticModel`

### 3.1. Root and model files

| File | Purpose | What to change |
| --- | --- | --- |
| `.platform` | Fabric artifact metadata: semantic-model type, display name, and logical ID. | Change display metadata when renaming the artifact. Keep the logical ID stable for an existing artifact. |
| `definition.pbism` | Semantic-model definition metadata and settings. | Usually maintained by Power BI Desktop; modify only for deliberate model-format or settings changes. |
| `definition/database.tmdl` | Database compatibility level for the tabular model. | Change only when a compatibility-level migration is intended and supported. |
| `definition/model.tmdl` | Model-wide culture, engine options, Power BI Desktop metadata, query order, and references to all tables/culture. | Update table references when adding/removing model tables; preserve the other metadata unless there is a specific model-wide change. |
| `definition/relationships.tmdl` | Relationship graph between model tables, including active/inactive relationships. | Change keys, cardinality, or active state only when the data model requires it. Relationship changes affect filtering and measure results throughout the report. |
| `definition/cultures/en-US.tmdl` | Model culture metadata for English (United States). | Use when managing culture-specific metadata or translations. |
| `diagramLayout.json` | Saved layout for the model diagram in Power BI Desktop. It currently describes an empty `All tables` diagram. | Change only to adjust the visual arrangement of tables in the model diagram; it does not define report page layout. |
| `.pbi/editorSettings.json` | Local semantic-model editor preferences. | Usually leave unchanged; not business logic. |
| `.pbi/localSettings.json` | Machine/user-specific local model settings. | Usually leave unchanged; not model source logic. |
| `.pbi/cache.abf` | Local Analysis Services model cache used by Desktop. | Do not edit manually or use as the source of truth; Desktop regenerates it. |

### 3.2. Table definition files

Each `definition/tables/<Table>.tmdl` defines that model table's columns,
types/format strings, summarization behavior, measures (where present), and
partition/data-loading expression. Imported CSV table partitions currently use
Power Query M (`Csv.Document` and type-conversion steps). The `Date` table is a
calculated DAX table.

| File | Table purpose | Typical change points |
| --- | --- | --- |
| `tables/Customers.tmdl` | Customer dimension: IDs, ZIP/city, signup date, gender, age group, and acquisition channel. | Add/change customer attributes, data types, or its CSV import mapping. |
| `tables/Date.tmdl` | Conformed calendar table, including Year, Month Number/Name, Quarter, Year-Month, hidden Year-Month Sort, and revenue/order growth measures. | Change calendar range or calendar attributes; preserve chronological sort behavior for Year-Month. Add date measures here if following the existing model convention. |
| `tables/Geography.tmdl` | Geography dimension: ZIP, city, region, and district. | Add/change geographic attributes or CSV import mapping. |
| `tables/Inventory.tmdl` | Product-by-snapshot inventory facts, including stock on hand, received/sold units, product/category/segment fields, and stock/value measures. | Add inventory metrics/columns or adjust their aggregation logic. Respect the monthly snapshot grain when writing measures. |
| `tables/OrderItems.tmdl` | Order-line sales facts: order/product IDs, quantity, unit price, discount, and promotion IDs; includes Gross Sales, Net Sales, Units Sold, AOV, Revenue per Customer, and Contribution % measures. | Change revenue/discount formulas or line-level fields. This is the primary sales source for report revenue. |
| `tables/Orders.tmdl` | Order-header facts: order date, customer, ZIP, order status, payment method, device, and order source; includes order/customer/status and fulfillment measures. | Add order attributes or adjust order status/count logic. |
| `tables/Payments.tmdl` | Payment facts: order ID, payment method, payment value, installments; includes Total Payment Value. | Add payment measures or update import mappings/formatting. |
| `tables/Products.tmdl` | Product dimension: SKU, name, category, segment, size, color, price, and COGS; includes average price/COGS and catalog count measures. | Add product attributes or change price/COGS formatting and measures. |
| `tables/Promotions.tmdl` | Promotion metadata: ID/name/type/discount, dates, applicable category/channel, stackable flag, and minimum order value. | Add promotion attributes or change the promotion CSV import. Promotion attribution is limited by incomplete promotion IDs in order items. |
| `tables/Returns.tmdl` | Return facts: return/order/product IDs, date, reason, quantity, and refund amount; includes return count/rate/amount measures. | Add return measures or revise definitions; preserve distinction between returned orders, return lines, and returned units. |
| `tables/Reviews.tmdl` | Product/order/customer review facts: review date, rating, and title; includes rating/count/positive/negative review measures. | Add review metrics or adjust rating thresholds and review import mappings. |
| `tables/Shipments.tmdl` | Shipment facts: order ID, ship/delivery dates, shipping fee; includes shipment cost, on-time delivery rate, and average delivery time. | Add logistics metrics or change the delivery-time threshold. Note the shipment-to-order relationship is currently inactive. |
| `tables/WebTraffic.tmdl` | Daily traffic-source facts: sessions, visitors, page views, bounce rate, average session duration; includes traffic/session/conversion measures. | Add digital metrics or change traffic import mappings. Conversion is a date-level proxy, not session-level attribution. |

The current imported CSV paths are absolute paths under
`D:\projects\VinTelligence2026\data`. If the project moves, update each relevant
Power Query `File.Contents(...)` path in the corresponding table TMDL, or update
the source in Power BI Desktop and save the model. Do not modify the CSVs to
change model logic; change the TMDL/Power Query definition instead.

## 4. Common Dashboard Changes

| Desired change | Main files to edit |
| --- | --- |
| Rename a report page | That page's `page.json` (`displayName`). |
| Reorder pages or set the opening page | `definition/pages/pages.json`. |
| Add/remove a page | Add/remove its page folder; update `pages.json`; keep page IDs and metadata consistent. Power BI Desktop is preferred for page creation/removal. |
| Add/remove/change a chart or KPI | The target page's `visuals/<visual-name>/visual.json`; for a new visual, create a visual folder/file with a unique visual name and valid PBIR structure. |
| Move or resize a chart | The visual's `position` in its `visual.json`. |
| Change fields, measures, or chart axes | The visual's `visual.query.queryState` projections. Keep references synchronized with semantic-model names. |
| Change a visual title, colors, legend, labels, axes, or background | `visual.objects` and `visual.visualContainerObjects` in that visual's `visual.json`; shared data colors are in theme resources. |
| Change KPI/calculation logic | The measure definition in its table's `.tmdl` file. |
| Add a source column or change source parsing/types | The table `.tmdl` column and Power Query partition; refresh the model afterward. |
| Change a CSV location | The `File.Contents(...)` expression in that table's `.tmdl` file or the equivalent query in Power BI Desktop. |
| Change filtering behavior across a table/model | `definition/relationships.tmdl`, plus affected DAX measures and report visuals. Validate cross-filtering after changes. |
| Change report-wide theme or options | `definition/report.json` and the referenced theme resources. |
| Change the model diagram layout | `diagramLayout.json` (presentation only). |

## 5. Editing and Validation Guidance

- Prefer Power BI Desktop for creating or deleting pages/visuals and for
  relationship changes. Desktop writes the PBIR/TMDL structures and helps avoid
  schema/version mistakes.
- When editing JSON/TMDL directly, use the existing files as templates and
  preserve schema versions, model lineage tags, valid expression syntax, and
  unique visual names.
- When a visual field or measure is renamed in the semantic model, update all
  report visual projections that reference it.
- After changes, open `dashboard.pbip` in Power BI Desktop, refresh the model,
  and check each page for rendering, values, ordering, and cross-filter behavior.
- Treat `.pbi` settings and cache files as local/generated state; do not use them
  to implement reusable dashboard changes.
