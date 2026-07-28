# Building the WIM Dashboard in Power BI Desktop — Step by Step

Data source: `WIM_Dashboard_DataSource.xlsx` (the `Metrics` table)

## 1. Install and open Power BI Desktop
1. Download from the Microsoft Store or powerbi.microsoft.com (free for Desktop).
2. Open Power BI Desktop → you land on a blank report canvas.

## 2. Connect to your Excel file
1. **Home** ribbon → **Get Data** → **Excel workbook**.
2. Browse to `WIM_Dashboard_DataSource.xlsx` → **Open**.
3. In the Navigator window, check the box next to **Metrics** (the table, not the whole file) → **Transform Data**.
4. In Power Query Editor, confirm column types (click the icon at the top of each column header):
   - `Month` → Date
   - `Category`, `Metric`, `Unit` → Text
   - `Value` → Decimal Number
5. **Home** → **Close & Apply**.

## 3. Add a Date table
Even with one date column, a dedicated date table makes month-level filtering and sorting reliable.
1. **Modeling** ribbon → **New Table**. Paste:
   ```
   DateTable = CALENDAR(MIN(Metrics[Month]), MAX(Metrics[Month]))
   ```
2. With `DateTable` selected → **Table tools** → **Mark as Date Table** → pick the `Date` column.
3. **Model view** (left sidebar) → drag from `DateTable[Date]` to `Metrics[Month]` to create a relationship (one-to-many, DateTable → Metrics).

## 4. Build the DAX measures
**Modeling** → **New Measure**, one at a time. This pattern pulls the value for a specific metric, at the latest month present in the data:

```
Team Members = CALCULATE(SUM(Metrics[Value]), Metrics[Metric]="Team Members", Metrics[Month]=MAX(Metrics[Month]))
```

Create one measure per KPI card, changing only the metric name:

| Measure name | Metric filter to use |
|---|---|
| Team Members | "Team Members" |
| Releases Supported | "Releases Supported" |
| Scrum Boards | "Scrum Boards" |
| User Stories | "User Stories" |
| Test Cases Designed | "Test Cases Designed (Cumulative)" |
| Test Cases Executed | "Test Cases Executed (Cumulative)" |
| Automation Designed | "Automation Designed (Cumulative)" |
| Automation Executed | "Automation Executed (Cumulative)" |
| Automation Coverage % | "Automation Coverage %" |
| Defects | "Defects (Cumulative)" |
| AI Adoption % | "AI Adoption %" |
| AI Enabled Resources | "AI-Enabled Resources" |
| Total Resources | "Total Resources" |
| Hours Saved | "Hours Saved (Quarterly)" |
| Productivity Gain % | "Productivity Gain %" |
| AI Use Cases | "AI Use Cases Implemented" |

For percentage measures, format the measure as a percentage: select it in the **Fields** pane → **Measure tools** → **Format** → **Percentage**.

For the trend charts, you don't need separate measures — you'll filter the visual directly to the "(Monthly)" metric rows (step 6).

## 5. Apply the red theme
1. **View** ribbon → **Themes** → **Browse for themes**.
2. Use the attached `wim_dashboard_theme.json` (or paste its contents into a new file) and select it.
3. This sets the report's default colors to the red palette used in the mockup.

## 6. Build each page

### Page 1 — Executive Summary
1. Rename **Page 1** to "Executive Summary" (double-click the tab).
2. Add 4 **Card** visuals across the top for: Automation Coverage %, Defects, AI Adoption %, Hours Saved. Resize them larger than other cards — this is your hero row.
3. Below, add a **Text box** with a one-line written insight (e.g. call out June's numbers) — update it manually each month, or skip if you'd rather keep the page numbers-only.
4. Add two rows of smaller **Card** visuals grouped by category: Team & Delivery (Team Members, Releases Supported, Scrum Boards, User Stories), then Quality & Automation (Test Cases Designed/Executed, Automation Designed/Executed). Use **Text box** headers above each row to label the groups.
5. Add a **Clustered bar chart**: Axis = none needed (use 4 separate cards instead) — or, for a single combined chart, add a **Clustered column chart** with `Metrics[Metric]` on the X-axis and `Metrics[Value]` on Y, filtered (visual-level filter) to just the 4 cumulative metrics.
6. Add a **Line chart**: X-axis = `DateTable[Date]`, Y-axis = `Metrics[Value]`, Legend = `Metrics[Metric]`, filtered to `Metric` = "Automation Executed (Monthly)" OR "Test Cases Executed (Monthly)".

### Page 2 — Quality
1. New page, rename "Quality".
2. 4 cards: Defects, Test Cases Designed, Test Cases Executed, User Stories.
3. **Column chart**: X = `DateTable[Date]`, Y = `Metrics[Value]`, filtered to `Metric` = "Defects Logged (Monthly)".
4. **Line chart**: X = `DateTable[Date]`, Y = `Metrics[Value]`, filtered to `Metric` = "Test Cases Executed (Monthly)".

### Page 3 — Automation
1. New page, rename "Automation".
2. 3 cards: Automation Coverage %, Automation Designed, Automation Executed.
3. **Column chart**: monthly Automation Executed trend (same filter pattern as above).
4. **Donut chart**: two-row helper table for Automated (60) vs Manual (40) — easiest to add as a small manual table (**Enter data**) since it's a single snapshot ratio, not a Metrics row.

### Page 4 — AI Usage
1. New page, rename "AI Usage".
2. 5 cards: AI Adoption %, AI Enabled Resources (as "25 / 76" — combine with a text box), Hours Saved, Productivity Gain %, AI Use Cases.
3. **Bar chart** (horizontal): Axis = `Metrics[Metric]`, Value = `Metrics[Value]`, filtered to the three AI-artifact metrics (Manual Test Cases Generated with AI, Automation Scripts Created with AI, Automation Scripts Maintained with AI).
4. **Donut chart**: AI-Enabled Resources (25) vs remaining (Total Resources − AI-Enabled Resources = 51) — small manual table again, or a measure `Non-AI Resources = [Total Resources] - [AI Enabled Resources]`.

## 7. Add slicers
On each page (or a shared filter pane), add a **Slicer** visual bound to `DateTable[Date]` so viewers can narrow the period. If you later add an LOB or Application column to your Excel source, add a second slicer for that.

## 8. Sync filters across pages (optional but recommended)
**View** ribbon → **Sync slicers** → check the slicer → tick which pages it should apply to (Sync + Visible). This makes your month/LOB filter behave consistently across all four pages, like a single control panel.

## 9. Format for a management audience
1. **Insert** → **Text box** for the report title on each page, or use the built-in page header.
2. **Format** pane on each visual → turn off unnecessary titles/borders for a cleaner look; keep consistent card font sizes.
3. **View** → **Page view** → **Fit to page**, so it displays cleanly when presented full-screen.

## 10. Publish and refresh
1. **Home** → **Publish** → sign in with your work account → select a workspace.
2. In the Power BI Service (app.powerbi.com), open the dataset → **Settings** → **Scheduled refresh**:
   - If the Excel file lives on OneDrive/SharePoint, you can point Power BI directly at that cloud path and set a daily/weekly refresh — no manual re-upload needed.
   - If it's a local file, you'll need the **On-premises data gateway** for scheduled refresh, or just re-publish manually after each update.
3. To present live to management: open the report in the Service and use **Present** mode, or **Export** → **PowerPoint**/**PDF** for a static snapshot beforehand.

## Maintenance loop going forward
1. Add new rows to the `Metrics` tab in Excel each month (see the Instructions tab in that workbook).
2. Save the file.
3. In Power BI Desktop: **Home** → **Refresh** (or let the scheduled refresh handle it if published).
4. Every card and chart updates automatically — no visual needs to be rebuilt.
