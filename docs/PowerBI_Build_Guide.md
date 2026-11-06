# Marketing Campaign Performance — Power BI Build Guide

This guide walks through recreating the dashboard in Power BI Desktop using the cleaned CSVs provided alongside this file: `Marketing_Campaign_Performance_Clean.csv`, `Marketing_Campaign_Details_Clean.csv`, `Region_Performance_Clean.csv`.

---

## Task 1 — Data Preparation

**What was checked in the source files:** all three CSVs came in with 0 nulls, 0 duplicate rows, and correct implicit types (numbers as numbers, text as text), so cleaning was limited to renaming for clarity and confirming types on import.

**Column renames applied:**

| File | Original | Renamed |
|---|---|---|
| Marketing_Campaign_Performance | CampaignID | Campaign_ID |
| | CampaignName | Campaign_Name |
| | Spend | Total_Spend |
| | Impressions | Total_Impressions |
| | Clicks | Total_Clicks |
| | Conversions | Total_Conversions |
| | Revenue | Total_Revenue |
| Marketing_Campaign_Details | CampaignName | Campaign_Name |
| | Type | Campaign_Type |
| | AverageSpend | Average_Spend |
| | AverageROI | Average_ROI |
| | TotalCampaigns | Total_Campaigns |
| Region_Performance | TotalSpend | Total_Spend |
| | TotalRevenue | Total_Revenue |
| | AverageROI | Average_ROI |

**In Power BI Desktop:**
1. `Get Data → Text/CSV` → import all three cleaned CSVs.
2. In Power Query Editor, for each table confirm data types: `Campaign_ID` → Whole Number; `Total_Spend`, `Total_Revenue`, `ROI`, `Average_ROI` → Decimal Number; `Total_Impressions/Clicks/Conversions/Total_Campaigns` → Whole Number; everything else → Text.
3. Use **Remove Duplicates** and **Remove Blank Rows** on each table as a safety check (none should be removed — the source data is already clean).
4. `Close & Apply`.

---

## Task 2 — Data Modeling

Go to the **Model** view and create these relationships:

| From | To | Cardinality | Direction |
|---|---|---|---|
| Marketing_Campaign_Details[Campaign_Name] | Marketing_Campaign_Performance[Campaign_Name] | One-to-Many | Single |
| Region_Performance[Region] | Marketing_Campaign_Performance[Region] | One-to-Many | Single |

Marketing_Campaign_Performance is the **fact table** (1,000 rows, one row per campaign execution); Details and Region are **dimension tables**. Filter flows from the "one" side (Details, Region) down into the "many" side (Performance) — this is what lets the Region and Campaign Type slicers filter the fact table correctly.

---

## Task 3 — DAX Measures

Create a new **measures table** (Modeling → New Table → name it `_Measures`) and add these, or place them directly on Marketing_Campaign_Performance as instructed:

```DAX
Total Impressions = SUM(Marketing_Campaign_Performance[Total_Impressions])

Total Clicks = SUM(Marketing_Campaign_Performance[Total_Clicks])

Total Conversions = SUM(Marketing_Campaign_Performance[Total_Conversions])

Total Revenue = SUM(Marketing_Campaign_Performance[Total_Revenue])

Total Spend = SUM(Marketing_Campaign_Performance[Total_Spend])

Total ROI = 
DIVIDE(
    [Total Revenue] - [Total Spend],
    [Total Spend],
    0
)

Average ROI = AVERAGE(Marketing_Campaign_Performance[ROI])

Best Campaign = 
VAR CampaignROI =
    ADDCOLUMNS(
        VALUES(Marketing_Campaign_Performance[Campaign_Name]),
        "@ROI",
        VAR s = CALCULATE([Total Spend])
        VAR r = CALCULATE([Total Revenue])
        RETURN DIVIDE(r - s, s, 0)
    )
VAR TopRow = TOPN(1, CampaignROI, [@ROI], DESC)
RETURN
    MAXX(TopRow, Marketing_Campaign_Performance[Campaign_Name])
```

Notes:
- `Total ROI` is a **calculated ratio** ((Revenue − Spend) / Spend) evaluated at whatever filter context is active — this is what makes it respond correctly to slicers and to being broken out by Region or Campaign.
- `Average ROI` averages the **row-level ROI column** that's already in the source data (a simple average of ratios), which is intentionally a different number than `Total ROI` (a ratio of sums) — that contrast is exactly what the gauge chart visualizes.
- `Best Campaign` returns whichever campaign has the highest Total ROI under current filters, so it updates live as slicers change.

With this data: **Total Revenue ≈ $42.5M, Total Spend ≈ $25.7M, Average ROI ≈ 67.6%, Total ROI ≈ 65.6%, Best Campaign = Radio Ads** (highest Total ROI at ~73.9%, despite Influencer Marketing having the highest *Average* ROI at ~110% — a good example of the two ROI measures disagreeing and why both are on the dashboard).

---

## Task 4 — Visualizations

Build these on one report page, referencing the preview in `Marketing_Campaign_Dashboard_Preview.html`:

| Visual | Type | Fields |
|---|---|---|
| Total Spend vs Total Revenue by Region | Stacked column chart | Axis: `Region_Performance[Region]`; Values: `[Total Spend]`, `[Total Revenue]` |
| Total Spend & Average ROI by Campaign | Line and stacked/clustered column chart (combo) | Axis: `Marketing_Campaign_Details[Campaign_Name]`; Column: `[Total Spend]`; Line: `[Average ROI]` (secondary axis) |
| Total ROI by Campaign | Bar chart (horizontal) | Axis: `Campaign_Name`; Values: `[Total ROI]`; sort descending; conditional color (green ≥0, red <0) |
| Number of Campaigns by Campaign Type | Pie chart | Legend: `Campaign_Type`; Values: `SUM(Total_Campaigns)` |
| Total ROI vs Average ROI | Gauge | Value: `[Total ROI]`; Target/Max: `[Average ROI]` (or a fixed max like 1.5) |
| KPI cards | Card visuals ×4 | `[Total Revenue]`, `[Total Spend]`, `[Average ROI]`, `[Best Campaign]` |

**Slicers:** add a slicer visual bound to `Region_Performance[Region]` and another to `Marketing_Campaign_Details[Campaign_Type]`; set both to multi-select list format and pin them at the top of the canvas so they act as global filters over every visual on the page.

---

## Dashboard Design Guidelines

- **Canvas:** Format → Page background → dark navy/charcoal (e.g. `#0D1119`), 100% transparency off.
- **Text:** set default text color to a light gray/white (`#E9EDF6`) in Format → General → all visuals, so titles and data labels stay readable.
- **Consistency:** pick one font (e.g. Segoe UI) and one weight for titles across all visuals; set it once, then use the **Format Painter** (Home → Format Painter, or right-click a visual → Copy/paste formatting) to apply it everywhere instead of re-styling each visual by hand.
- **Depth:** turn on **Shadow** and a subtle **Border** (1px, slightly lighter than the canvas) on every visual container under Format → Effects, so cards read as distinct panels rather than blending into the background.
- **Spacing:** align visuals to a grid (View → Show gridlines / Snap to grid) and keep consistent padding between panels — the HTML preview uses 14px gutters as a reference.
- **Color coding:** keep Spend and Revenue on consistent colors across every chart (e.g. indigo for Spend, teal for Revenue) so the eye doesn't have to re-learn the legend per visual.

---

## Files in this package

- `Marketing_Campaign_Performance_Clean.csv`, `Marketing_Campaign_Details_Clean.csv`, `Region_Performance_Clean.csv` — cleaned, renamed source data ready to import.
- `Marketing_Campaign_Dashboard_Preview.html` — interactive preview of the finished layout (KPI cards, all 5 charts, working Region/Type slicers) so you can validate the design before building it natively in Power BI Desktop.
- `PowerBI_Build_Guide.md` — this document.

**One limitation worth flagging:** I can't produce a native `.pbix` file directly (Power BI Desktop isn't available in this environment), so the HTML file is a functional stand-in for design/QA, and this guide gives you the exact relationships, DAX, and visual field-wells to reproduce it in Power BI itself.
