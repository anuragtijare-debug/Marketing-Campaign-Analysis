# Marketing Campaign Performance Dashboard (Power BI)

An interactive Power BI dashboard that compares marketing campaign performance across regions and channels (Digital vs Traditional), focusing on **spend, revenue and ROI**.

![Dashboard](dashboard/dashboard.png)

## Business Problem
The marketing team runs 8 campaign types across 6 regions, but had no clear view of which campaigns and regions actually turn spend into revenue efficiently. Most campaigns are Digital, yet it was unclear whether that is where the budget works hardest.

## Goal
Give the team a single ROI-first view to compare campaigns and regions, spot misallocated budget, and decide where the next marketing dollar should go.

## Dataset
| File | Rows | Description |
|---|---|---|
| `marketing_campaign_performance.csv` | 1,000 | Spend, impressions, clicks, conversions, revenue, ROI per campaign record |
| `marketing_campaign_details.csv` | 8 | Campaign type (Digital / Traditional), average spend, average ROI, total campaigns |
| `region_performance.csv` | 6 | Spend, revenue and average ROI by region |

Raw files are in `data/raw/`, cleaned and renamed versions in `data/cleaned/`.

## Data Preparation
- Checked all 3 files: **0 nulls, 0 duplicates**, data types verified
- Renamed columns for clarity (e.g. `Spend` -> `Total_Spend`, `Type` -> `Campaign_Type`)

## Data Model
- `Marketing_Campaign_Details[Campaign_Name]` -> `Marketing_Campaign_Performance[Campaign_Name]` (One-to-Many)
- `Region_Performance[Region]` -> `Marketing_Campaign_Performance[Region]` (One-to-Many)

## DAX Measures
Total Impressions, Total Clicks, Total Conversions, Total Revenue, Total Spend, Total ROI, Average ROI, Best Campaign. Full code in [`docs/DAX_Measures.md`](docs/DAX_Measures.md).

## Dashboard Visuals
- KPI cards: Total Revenue, Total Spend, Average ROI, Best Campaign
- Stacked column: Total Spend vs Total Revenue by Region
- Combo chart: Total Spend and Average ROI by Campaign
- Bar chart: Total ROI by Campaign
- Pie chart: Campaigns by Type
- Gauge: Total ROI vs Average ROI
- Slicers: Region, Campaign Type

## Key Insights
| Metric | Value |
|---|---|
| Total Revenue | $42.54M |
| Total Spend | $25.69M |
| Average ROI | 67.63% |
| Total ROI | 65.56% |
| Best Campaign | Radio Ads (73.88% ROI) |

- **Traditional campaigns outperform Digital:** Traditional ROI is ~71.8% vs ~62.3% for Digital.
- **Budget mismatch:** Digital is 65.5% of campaigns and ~$16.9M of spend, yet returns less per dollar.
- **Top 3 campaigns by ROI** are all Traditional: Radio Ads, TV Commercials, Billboards. Weakest are Influencer Marketing (59.28%) and Email Marketing (58.74%).
- **Regions:** Africa has the best ROI (~70.2%), Oceania the weakest (~60.6%).
- Influencer Marketing has the highest *average* ROI (~110%) but a low *total* ROI, so a few outlier campaigns inflate the average.

## Recommendations
1. Shift 10-15% of budget from Email and Influencer Marketing into Radio Ads and TV Commercials.
2. Audit Influencer Marketing campaign by campaign and scale only the top performers.
3. Review Oceania's campaign mix instead of cutting its budget across the board.

## Project Structure
```
Marketing-Campaign-Analysis/
├── README.md
├── dashboard/
│   └── dashboard.png
├── data/
│   ├── raw/
│   └── cleaned/
├── docs/
│   ├── DAX_Measures.md
│   └── PowerBI_Build_Guide.md
└── .gitignore
```

## Tools
Power BI Desktop, DAX, Python (pandas) for data checks.
