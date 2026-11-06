# DAX Measures

```DAX
Total Impressions = SUM(Marketing_Campaign_Performance[Total_Impressions])

Total Clicks = SUM(Marketing_Campaign_Performance[Total_Clicks])

Total Conversions = SUM(Marketing_Campaign_Performance[Total_Conversions])

Total Revenue = SUM(Marketing_Campaign_Performance[Total_Revenue])

Total Spend = SUM(Marketing_Campaign_Performance[Total_Spend])

Total ROI =
DIVIDE([Total Revenue] - [Total Spend], [Total Spend], 0)

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
