# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

This repo holds a Power BI e-commerce / supply-chain dashboard saved in **PBIP format**, so the whole report and semantic model are plain text. The primary audience is an **Ecommerce Manager** (see `README.md`) reviewing results weekly, monthly, and yearly (not daily). The build tooling is Power BI Desktop and nothing else: there is no package manager, build script, linter, or test suite.

```
README.md                      Project overview, pages, KPIs, assumptions (describes the final build)
DASHBOARD_INSTRUCTION.md       Metric catalogue and per-page design spec (describes the final build; §6.4 known issues)
data/*.csv                     13 source CSVs (committed). The semantic model loads 10 of them.
dashboard/
  dashboard.pbip               Entry point — open this in Power BI Desktop
  dashboard.pdf                PDF export of the report
  dashboard.Report/            PBIR report (pages, visuals, theme)
  dashboard.SemanticModel/     TMDL model (tables, measures, relationships)
```

## Source of truth

**The current build is the source of truth.** `DASHBOARD_INSTRUCTION.md` and `README.md` are background. Where they disagree with the build (canvas size, ID data types, relationships, data path), follow the build. When asked to tidy things up, update the docs to match the build rather than changing the build to match the docs. The spec's *business rules* (below) still describe how the existing measures are meant to behave. Follow them when writing new measures.

## Working with the files

Changes can be made in two ways: by editing the PBIR/TMDL text directly, or in Power BI Desktop. Keep both in mind:

- **Desktop must be closed (or the change reloaded) before you edit files.** Desktop doesn't watch the files. When it next saves, it overwrites the folder with its in-memory state, which silently discards edits made on disk. If you're unsure whether Desktop is open, ask.
- **Desktop saves rewrite files.** They reformat JSON, reorder properties, add `lineageTag`s and auto date tables, and may change schema versions. Expect noisy diffs after a Desktop save, and don't "fix" that formatting by hand.
- **There's no way to validate here.** After editing files, tell the user to open `dashboard/dashboard.pbip` in Desktop, check for load errors, and refresh. Keep JSON valid (no trailing commas or comments) and keep TMDL indentation (tabs) consistent.
- **Never commit** `.pbi/localSettings.json` or `.pbi/cache.abf`. Both are in `.gitignore`.

### Report (PBIR) — `dashboard/dashboard.Report/`

- `definition.pbir` points to the model through `datasetReference.byPath = "../dashboard.SemanticModel"`.
- `definition/report.json` sets the theme to **`purple-slicer`**. The same file exists in two places, `StaticResources/SharedResources/BaseThemes/purple-slicer.json` (base theme) and `StaticResources/RegisteredResources/purple-slicer.json` (custom theme). Change colours in the theme files, not visual by visual.
- `definition/pages/pages.json` holds `pageOrder` and `activePageName` (the opening page). A page folder that isn't listed in `pageOrder` won't appear. A listed ID with no folder breaks the load.
- Pages live at `definition/pages/<pageId>/page.json`, and each page's visuals at `.../visuals/<visualName>/visual.json` (schema `visualContainer/2.12.0`). The visual folder name must match the visual's `name` property. Each visual has:
  - `position`: x, y, z, width, height, tabOrder.
  - `visual.query.queryState`: projections that reference model fields as `Entity` (table) + `Property` (column or measure). Renaming a measure or column in the model breaks every visual that references it. Grep the report for the old name and update every match.
- When you create a new page or visual, copy an existing one of the same type and change it. Don't write the JSON from scratch. Give new pages a fresh unique ID and add it to `pageOrder`.

Current pages, in `pageOrder`:

| Page ID | Tab name | Size | Display option |
|---|---|---|---|
| `9f7b1d2d-…f712` | Executive Overview (`activePageName`) | 1920×1750 | FitToPage |
| `2d466881-…34d5` | Sales & Profitability | 1920×2780 | FitToPage |
| `a1f2c3d4-…5d6e` | Customers & Markets | 1920×2240 | FitToPage |
| `5c7bc72d-…68d16b1ac7af` | Inventory | 1920×3460 | FitToPage |
| `fcd9a8b8-…183f6a` | Shipping & Delivery | 1920×2460 | FitToWidth |
| `b2f3c4d5-…5d6e7f` | Returns & Product Quality | 1920×2240 | FitToWidth |
| `cb86d456-…1ef3` | Customer Satisfaction | 1920×2240 | FitToWidth |
| `c3e4d5f6-…6d7e` | Category Tooltip (hidden, `type: Tooltip`) | 320×240 | ActualSize |

The Category Tooltip page is the only hidden page (`visibility: HiddenInViewMode`, `pageBinding` type Tooltip). It holds `tt-card-category` (`Selected Category`) and six cards: `Net Sales`, `Net Sales YoY %`, `Gross Margin %`, `Return Rate (Closed Window)`, `Negative Review Rate`, `Category Stockout Rate`. There is no drill-through page; navigation is by page tabs only.

**Layout conventions (as built):**
- Sizes and display options are in the table above. Every page is split into numbered sections below the KPI cards, each opened by a section heading: a 1840×40 text box at x 40 named `p<N>-section-<subject>`, copied from `p4-section-service` ("N · Title" Semibold 15 pt #17365D + the question in 11 pt #667085, alt text "Section heading: …"); its visuals start 50 below it. The domain pages share one storytelling grid below the KPI cards (y = 254, h 132), with headings at 416, 896, 1296, 1706: row A main trend at y = 466 (h 400), row B drivers/mix at 946 (h 320), row C category/region comparison at 1346 (h 330), row D detail at 1756 (a search box, h 80, then the detail table at 1842, h 380; Shipping & Delivery puts a full-width region table at 1756, h 180, then the search box at 1966 and the city table at 2052), see `DASHBOARD_INSTRUCTION.md` §6.2. Inventory uses its own grid: two KPI rows (254: fill rate, stockout rate, units on hand, inventory value, days of supply, turnover, w 290; 406: shortage-risk products, excess-stock products, excess stock value, w 600), then four sections each opened by a 40-px text heading (`p4-section-*`): 1 service level at 568 (fill/stockout trend + by ABC class), 2 stock movement at 1008 (inbound/outbound/units on hand with units on hand on the secondary axis, turnover, then inventory value by month × `Inventory[Cover Band]` full width), 3 stock status and risk at 1868 (products by month and by category × `Inventory[Stock Risk]`), 4 detail at 2328 (category scorecard, value by ABC class × cover band, product search box at 2798 and the full-width `Planning Actions by Product` table at 2898, h 520, with `Planning Action` (▲ Replenish / ▼ Reduce excess / ● Healthy) and `Excess Units` / `Excess Stock Value` (stock above 90 days of cover for overstock-flagged products)). Shipping & Delivery's sections are 1 shipping cost (cost per shipment + premium-fee share trend, by `Orders[Basket Size]`, by category), 2 order fulfilment (`Order Status Share` by year × status, `Cancellation Rate` by payment method), 3 delivery speed (lead-time trend, transit-day distribution), 4 region and city detail. Returns' sections are 1 return and refund trend (closed-window return + refund rate by month, refund amount by year), 2 why products come back (reason share, return lines by `Returns[Days to Return]`, return rate by payment method), 3 where refunds concentrate (refund by category, region table), 4 product detail; its rates use `Return Rate (Closed Window)` / `Refund Rate (Closed Window)`, which only count orders placed 31+ days before `[Data End Date]` because returns arrive up to 31 days after the order. Customer Satisfaction's sections are 1 satisfaction trend (positive/negative review rate by review month, `Review Coverage` by year), 2 rating mix and feedback timing (`Rating Share` by `Reviews[Rating Label]`, reviews by `Reviews[Days to Review]`, negative rate by transit days), 3 where dissatisfaction concentrates (category bar, region table), 4 product detail; ratings barely vary by any attribute, so new visuals there should show sentiment shares, coverage or product-level detail rather than average rating by another dimension. On each page every visual answers a different question; don't add one that repeats a measure-by-dimension already shown. Executive Overview and Sales & Profitability carry two rows of 6 KPI cards (y = 254 and 406), then three sections (headings: Executive Overview 568, 1048, 1428; Sales & Profitability 568, 1448, 2188; see `DASHBOARD_INSTRUCTION.md` §6.3.1–6.3.2). Executive Overview shows totals only, with no product-level visuals; its second KPI row has 5 cards (w 352). Every page adds `hdr-reset-filters` and `hdr-data-through` to the header. The Customers & Markets KPI row has 5 cards (w 352); Customer Satisfaction has 6 (w 290). The Customers KPI cards use `… KPI` measures (`Orders.tmdl`, `OrderItems.tmdl`) that fall back to the latest 12 months when no date is filtered, because all-time New Customers equals every customer and the all-time repeat rate is blank. Place new visuals in the row that matches their story step, and when a section grows, shift everything below it (and the page height) down. The Inventory charts use a soft palette set per visual (teal #5FA89D, coral #E07B6F, soft orange #F2A766, pale amber #F3CF7A, muted brown #A8786C, soft blue #7096C8); reuse it for new Inventory charts.
- Trend axes use `Date[Month Start]` (a true date), not `Date[Year-Month]`. Yearly growth columns use `Date[Year]`. The Reporting-period slicer still uses `Date[Year-Month]`.
- Card subtitles that show change vs prior year use text measures `… vs PY Label` in `Date.tmdl` (same period logic, ▲/▼ arrow). Only `Total Orders vs PY Label` (inside `hdr-data-through`) and `Inventory Value vs PY Label` (`kpi-inventory-value`) remain; the rest, every `… vs PY Color` and `Gross Sales YoY %` / `Gross Sales YoY Color` were deleted as unused after a Desktop save unbound them. Many cards now have a literal-text subtitle or none (see `DASHBOARD_INSTRUCTION.md` §6.4). To add a new vs-PY subtitle, copy `Inventory Value vs PY Label`. `Net Sales YoY %` and `Net Sales MoM %` use the format `▲ 0.0%;▼ 0.0%;0.0%`. Conditional colours come from hidden `… Color` measures returning hex codes and must always repeat what an arrow or text already says. On Executive Overview and Sales & Profitability, arrows are green when the value went up and red when it went down (#067647 / #B42318) (`Net Sales YoY Color`, `Net Sales MoM Color`, `Gross Profit YoY Color`, `Total Orders YoY Color`). Negative metrics, where lower is better, must invert this so that ▲ is red and ▼ is green; no such growth-colour measure exists today, so give any new one that inversion. `Gross Profit Color` marks negative gross profit red. Sales currency measures use `$#,0;($#,0);$#,0` (AOV and Revenue per Customer keep cents). The scorecard's `Stockout Rate Color` and `Return Rate Color` instead mark above-average (bad) values red.
- Growth measures (`Net Sales Prior Year`, `… YoY %`, `Net Sales MoM %`) live in `Date.tmdl` and compare like-for-like dates via EDATE + TREATAS; reuse that pattern for new growth measures. `Inventory Turnover` (annualised, at cost) is in `Inventory.tmdl`.
- Every detail table has a search box above it, named `<prefix>-search-<field>` (product, promotion or city). It is a copy of `filter-3-productname` with the sync group and filterConfig removed, `objects.general.selfFilterEnabled = true`, alt text, w 420 h 80. Page-local filtering comes from `page.json` `visualInteractions`: one `{source, target, type: "NoFilter"}` entry from the search slicer to every other visual with a query except its own table. When you add a visual to such a page, add its NoFilter entry too.
- `Reviews.tmdl` has `Positive Review Rate (Min Reviews)`, `Negative Review Rate (Min Reviews)` and `Return Rate (Min Reviews)`, which go blank under the Minimum Reviews threshold like `Average Rating (Min Reviews)`.
- Parameter tables: what-if `Low Stock Threshold` and `Minimum Reviews` (no slicer on any page, so they stay at 20 and 10; each has a `… Value` measure that returns the default when nothing is selected; if a slicer is ever added, state the default in its title), what-if `Transit Day Threshold` and `Return Reduction` (column only: their measures were deleted as unused, so nothing reads them), and field parameters `Trend Metric` and `Breakdown Dimension` (the only parameter slicers, both on Sales & Profitability). The field-parameter tables reference measures and columns by `NAMEOF`, so a rename must update them too.
- The six category bar charts (`p2-bar-yoy-category`, `p2-bar-discount-rate` (SP), `p4-bar-risk-category` (IP), `p5-bar-cost-category` (Shipping & Delivery), `p6-bar-refund-category` (RQ), `p7-bar-rating-category` (CS)) bind `visualContainerObjects.visualTooltip` to the Category Tooltip page (`section` = `c3e4d5f6-a7b8-49c0-8d1e-2f3a4b5c6d7e`). Give a new bar chart by `Products[Category]` the same binding; other visuals keep the default tooltip.
- Every page carries the same shared header and slicer visuals: `hdr-band`, `hdr-brand`, `hdr-title`, `hdr-reset-filters`, `hdr-data-through`, `filter-1-year-month`, `filter-2-category`, `filter-3-productname`, `filter-4-region`. Keep their positions identical across pages. The title text box (40, 16, 1100×64) holds only the page's tab name in Segoe UI Semibold 22 pt white, with no purpose line; on Executive Overview it is named `ce53d977703e5a68ca69` (auto-named by Desktop) instead of `hdr-title`.
- KPI cards are named `kpi-<subject>`. Other visuals are named `p<N>-<type>-<subject>`, where `N` is the page's position in `pageOrder` (p1 Executive Overview … p7 Customer Satisfaction), e.g. `p5-line-lead-time`. Tooltip-page visuals use `tt-`. Exception: Inventory's section headings are `p4-section-*` and the shared section-heading template is `p4-section-service`.
- Row structure from top to bottom: header/context, slicers, a row of 4–6 KPI cards, the main trend, comparison/breakdown visuals, and a detail table at the bottom.

### Semantic model (TMDL) — `dashboard/dashboard.SemanticModel/definition/`

- `model.tmdl`, `database.tmdl` (compatibilityLevel 1606), `relationships.tmdl`, and `tables/<Table>.tmdl`.
- Measures live in the table they mainly aggregate (e.g. sales measures in `OrderItems.tmdl`, date-intelligence measures in `Date.tmdl`). Each measure has a `formatString` and a `lineageTag`. When adding a measure by hand, use a newly generated GUID for `lineageTag` and never reuse one.
- Every table loads from CSV through Power Query with an **absolute path**:
  `Csv.Document(File.Contents("D:\projects\vintelligence2026_ai_agent_powerbi_dashboard\data\<file>.csv"), …)`.
  If the repo lives anywhere else, every partition must be updated. Doing that through a shared parameter would be an improvement, but only make that change if asked.
- `LocalDateTable_*` / `DateTableTemplate_*` tables are Desktop's auto date/time tables. Don't edit them by hand.

## Data model

A star schema with a single conformed **`Date`** table.

| Table | Grain / rows | Notes |
|---|---|---|
| Orders | 1 per order — 646,945 | Status: delivered, cancelled, returned, shipped, paid, created |
| OrderItems | order × product line — 714,669 | 16 repeated order+product rows are legitimate; **don't dedupe** |
| Shipments | 566,067 | A missing shipment row ≠ undelivered |
| Returns | 39,939 | |
| Reviews | 113,551 | |
| Inventory | product × monthly snapshot — 60,247 (126 snapshot dates) | **Snapshot table, never sum across dates** |
| Customers / Geography / Products / Promotions | dimensions — 121,930 / 39,948 / 2,412 / 50 | Blank promo IDs are valid: keep them |

`payments.csv`, `sales.csv`, and `web_traffic.csv` are **not loaded** (their tables were removed as unused; payment method comes from `Orders[PaymentMethod]`). Columns no visual or measure used were either dropped in Power Query (a `Removed Unused Columns` step) or hidden. Check that a column is still loaded before using it, and extend that step's list if you need one back.

**Date relationships:** the active ones are `Orders[OrderDate]` and `Inventory[SnapshotDate]`. The inactive role-playing ones are ShipDate, DeliveryDate, ReturnDate, and ReviewDate. Only the review-date role is used today (`Positive Review Rate by Review Date`, `Negative Review Rate by Review Date`); no measure uses ship, delivery or return date. A measure that analyses by another date uses this pattern:

```dax
measure 'Negative Review Rate by Review Date' =
    CALCULATE(
        [Negative Review Rate],
        USERELATIONSHIP('Date'[Date], Reviews[ReviewDate]),
        CROSSFILTER('Date'[Date], Orders[OrderDate], NONE)   -- stop the order-date path from also filtering
    )
```

How the build differs from the spec (these are deliberate as-built facts):
- `Shipments[OrderID] → Orders[OrderID]` is **active**, so shipment measures follow order date, Region, and status.
- Order-level measures (`Total Orders`, `Distinct Customers`, the customer counts, and the base shipment measures) wrap their aggregation in `CALCULATE(..., CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH))` so product slicers reach them. The relationship stays single-direction in the model. Follow the same pattern for new order-level measures.
- There is **no** Customers → Geography relationship.
- `New Customers` and `Repeat Customers` work out each customer's first-ever order date inside the measure (`CALCULATE(MIN(Orders[OrderDate]), ALLEXCEPT(Orders, Orders[CustomerID]))`). There is no first-order-date column.
- ID columns are typed `Int64` in Power Query (the spec says text).

## Measure and business rules

- **Net Sales** = after discounts, before refunds. There is no post-refund sales measure (the scenario measures were deleted as unused); refunds are reported separately through `Refund Amount`.
- Always use `DIVIDE()` for ratios, never `/`.
- **Snapshots:** inventory measures take the latest snapshot in context (`VAR LatestSnapshot = MAX(Inventory[SnapshotDate])`) rather than summing across dates.
- Compare only equivalent periods (YoY / MoM against a matching span).
- Every measure in a given ratio uses the same eligible-order population.
- Don't coerce invalid or blank values to zero. A blank stays blank.
- The 3-day transit threshold (`Transit Within 3 Days`) is **diagnostic only**, because the data has no promised delivery date. Don't label it as an SLA or "on-time".
- Low stock means ≤ 20 units. A positive review is ≥ 4, a negative one ≤ 2, and a low-rated product < 3.
- Reuse existing measures and don't re-derive them. Check the table's `.tmdl` before adding a new one.

## Visual design rules

- Titles take the form **"Measure by Dimension"**, with the date basis (order date, ship date, etc.) in the subtitle.
- Never use colour alone to carry meaning. Pair it with labels, icons, or text.
- Don't use 3D charts, gauges without a target, or pies with many slices.
- Give every visual alt text, and keep a reset-filters action on each page.

## Known issues (leave alone unless asked)

`README.md` and `DASHBOARD_INSTRUCTION.md` §6 describe the final build; §6.4 lists these issues in full and §2.10 lists the helper measures no visual binds (every other unbound measure was deleted). See the per-page tables in §6.3 for each folder name and its displayed visual.

- Several KPI cards have no subtitle after the last Desktop save. `Total Orders vs PY Label` is bound only inside the hidden `hdr-data-through` text.
- **Filter paths:** the Region slicer doesn't reach `Inventory`, which connects only to `Products` and the snapshot date.
- Every non-header, non-slicer visual has alt text, and new visuals need alt text and a date-basis subtitle.
- The Shipping & Delivery card folder `kpi-fulfillment-rate` binds `Fulfilled Rate` (delivered, paid or returned ÷ non-cancelled). The old `Fulfillment Rate`, `Return Amount`, `Average Transit Time` and Product-Linked shipment measures were deleted as duplicates; the city table keeps the column header "Average Transit Time" through a `displayName` on `Average Delivery Time`.

## Git

- Work on `main` unless told otherwise, and commit only when asked.
- Use descriptive commit messages (e.g. "Add return-rate trend to Returns page"), not the "update" messages in the past history.
- A Desktop save can touch many files. Before committing, check the diff for unintended changes.
