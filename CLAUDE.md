# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

This repo holds a Power BI e-commerce / supply-chain dashboard saved in **PBIP format**, so the whole report and semantic model are plain text. The primary audience is a **Supply Chain Manager** reviewing results weekly, monthly, and yearly (not daily). The build tooling is Power BI Desktop and nothing else: there is no package manager, build script, linter, or test suite.

```
README.md                      Project overview, KPIs, assumptions (partly stale — see Known issues)
data/*.csv                     13 source CSVs (committed). Loaded by the semantic model.
forecast/sample_submission.csv Placeholder for a future forecasting task — OUT OF SCOPE, don't touch
dashboard/
  dashboard.pbip               Entry point — open this in Power BI Desktop
  DASHBOARD_INSTRUCTION.md     Original design/metric spec (background reference)
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
- `definition/pages/pages.json` holds `pageOrder` and `activePage`. A page folder that isn't listed in `pageOrder` won't appear. A listed ID with no folder breaks the load.
- Pages live at `definition/pages/<pageId>/page.json`, and each page's visuals at `.../visuals/<visualName>/visual.json` (schema `visualContainer/2.12.0`). The visual folder name must match the visual's `name` property. Each visual has:
  - `position`: x, y, z, width, height, tabOrder.
  - `visual.query.queryState`: projections that reference model fields as `Entity` (table) + `Property` (column or measure). Renaming a measure or column in the model breaks every visual that references it. Grep the report for the old name and update every match.
- When you create a new page or visual, copy an existing one of the same type and change it. Don't write the JSON from scratch. Give new pages a fresh unique ID and add it to `pageOrder`.

Current pages, in `pageOrder`:

| Page ID | Display name |
|---|---|
| `9f7b1d2d-…f712` | Executive Overview (activePage) |
| `2d466881-…34d5` | Sales and Profitability |
| `a1f2c3d4-…5d6e` | Customers and Market Performance |
| `5c7bc72d-…68d16b1ac7af` | Inventory Performance |
| `fcd9a8b8-…183f6a` | Delivery and Logistics |
| `b2f3c4d5-…5d6e7f` | Returns and Product Quality |
| `cb86d456-…1ef3` | Customer Satisfaction |
| `8e30a6d6bf64860c8c71` | Page 1 (blank leftover, 1920×1080) |

**Layout conventions (as built):**
- Analytical pages are **1920×1570** with `FitToWidth`.
- Every page carries the same shared header and slicer visuals: `hdr-band`, `hdr-brand`, `hdr-title`, `filter-1-year-month`, `filter-2-category`, `filter-3-productname`, `filter-4-region`. Keep their positions identical across pages.
- KPI cards are named `kpi-<subject>`. Other visuals are named `p<N>-<type>-<subject>`, e.g. `p3-line-net-sales`. The `N` prefixes are inconsistent today. For new visuals, use the page's existing prefix.
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
| Payments | 1 per order — 646,945 | |
| Shipments | 566,067 | A missing shipment row ≠ undelivered |
| Returns | 39,939 | |
| Reviews | 113,551 | |
| Inventory | product × monthly snapshot — 60,247 (126 snapshot dates) | **Snapshot table, never sum across dates** |
| Sales | daily benchmark — 3,833 | Separate benchmark series; don't blend with OrderItems revenue |
| WebTraffic | daily — 3,652 (from 2013-01-01) | |
| Customers / Geography / Products / Promotions | dimensions — 121,930 / 39,948 / 2,412 / 50 | Blank promo IDs are valid: keep them |

**Date relationships:** the active ones are `Orders[OrderDate]`, `Inventory[SnapshotDate]`, `Sales[Date]`, and `WebTraffic[Date]`. The inactive role-playing ones are ShipDate, DeliveryDate, ReturnDate, and ReviewDate. Measures that analyse by any other date use this pattern (the `… by Ship Date`, `… by Return Date`, and `… by Review Date` variants):

```dax
measure 'Shipment Cost by Ship Date' =
    CALCULATE(
        [Shipment Cost],
        USERELATIONSHIP('Date'[Date], Shipments[ShipDate]),
        CROSSFILTER('Date'[Date], Orders[OrderDate], NONE)   -- stop the order-date path from also filtering
    )
```

How the build differs from the spec (these are deliberate as-built facts):
- `Shipments[OrderID] → Orders[OrderID]` is **active**, so shipment measures follow order date, Region, and status.
- Order-level measures (`Total Orders`, `Distinct Customers`, the customer counts, `Total Payment Value`, and the base shipment measures) wrap their aggregation in `CALCULATE(..., CROSSFILTER(OrderItems[OrderID], Orders[OrderID], BOTH))` so product slicers reach them. The relationship stays single-direction in the model. Follow the same pattern for new order-level measures.
- There is **no** Customers → Geography relationship.
- ID columns are typed `Int64` in Power Query (the spec says text).

## Measure and business rules

- **Net Sales** = after discounts, before refunds. Use `Net Sales After Returns` for the post-refund view.
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

`README.md` and `DASHBOARD_INSTRUCTION.md` §6 now describe the build, including these issues. See the per-page tables in §6.3 for the mapping from folder name to displayed visual.

- The blank `Page 1` (`8e30a6d6bf64860c8c71`) is still in `pageOrder`.
- **Visual folder names don't match their content.** Folders were copied between pages and repurposed:
  - On Executive Overview, all five `kpi-*` folders show something else. For example, `kpi-net-sales` shows "Shipment Coverage".
  - On Customers and Market Performance (`a1f2…`), the `p5-*` folders hold customer and order visuals, not satisfaction visuals.
  - Delivery and Logistics and Returns and Product Quality share the folder names `kpi-shipment-cost`, `kpi-otd`, `kpi-return-rate`, `kpi-refund-amount`, and `p3-*`.
  - On Customer Satisfaction, `p5-col-gender` is a line chart, "Average Rating by Review Month".
  - Always check a visual's title and projections, not its folder name.
- **Filter paths:** the Region slicer doesn't reach `Inventory`, which connects only to `Products` and the snapshot date. The "Product-Linked" shipment measures now duplicate the base measures.
- No page has a reset-filters action. Every non-header, non-slicer visual has alt text and a date-basis subtitle, and new visuals need both.
- There are duplicate measures: `Return Amount` / `Refund Amount`, `Revenue Growth` / `Year-over-Year Growth`, and `Average Transit Time` (an alias of `Average Delivery Time`).

## Git

- Work on `main` unless told otherwise, and commit only when asked.
- Use descriptive commit messages (e.g. "Add return-rate trend to Returns page"), not the "update" messages in the past history.
- A Desktop save can touch many files. Before committing, check the diff for unintended changes.
