# Customer Churn & Retention Analysis

**A Power BI dashboard for customer retention, RFM segmentation, and cohort-based churn analysis built from raw transaction data into an executive-ready analytics product.**

---

## Overview

Working from two raw CSVs, 350 customer records and ~12,000 transaction line items spanning 24 months, I built a full star-schema data model, a percentile-based RFM segmentation engine, and monthly/quarterly cohort retention analysis, wired into a 3-page executive dashboard.

The most interesting part of the build wasn't the visuals, it was a bug. My RFM segmentation was only ever producing 4 of 11 designed customer segments, despite the DAX running without errors. The cause: `RANKX` using `DENSE` ranking silently compresses its output range on columns with heavy duplication, one field had only 37 distinct values across 350 customers, so a percentile-bucketing formula could never mathematically reach its lowest tiers. Switching to standard ranking fixed it instantly. Finding it meant questioning a result that *looked* plausible rather than trusting that "it ran, so it's correct", the throughline of this whole project.

---

## Dashboard Preview

| Executive Overview | RFM Segmentation | Cohort Retention |
|---|---|---|
| ![Executive Overview](images/Screenshot%202026-08-02%20001122.png) | ![RFM Segmentation](images/Screenshot%202026-08-02%20001140.png) | ![Cohort Retention](images/Screenshot%202026-08-02%20001154.png) |

---

## What's Inside

**Data Model** : Star schema (`DimCustomers`, `FactTransactions`, `DimDate`), with one relationship intentionally kept inactive and activated selectively via `RELATED()` for cohort calculations.

**RFM Segmentation** : Percentile-based Recency/Frequency/Monetary scoring via `RANKX`, classified into 11 segments (Champions, Loyal Customers, At Risk, Hibernating, Lost, etc.) through a single ordered `SWITCH(TRUE(), ...)` expression.

**Cohort Retention** : Monthly and Quarterly retention matrices using `DATEDIFF`-based cohort index logic, with color-scale heatmaps showing retention decay over a customer's lifetime.

**Dashboard** : 3 pages (Executive Overview, RFM Segmentation, Cohort Retention), with a custom theme (`dashboard-theme.json`) built on a documented pixel-level grid and color system.

---

## Key DAX

```DAX
R_Score = 
VAR RankAsc = RANKX ( DimCustomers, DimCustomers[Days Since Last Purchase], , ASC )
VAR PctPosition = DIVIDE ( RankAsc, COUNTROWS ( DimCustomers ) )
RETURN
SWITCH ( TRUE(), PctPosition <= 0.2, 5, PctPosition <= 0.4, 4, PctPosition <= 0.6, 3, PctPosition <= 0.8, 2, 1 )
```

```DAX
CohortIndexMonth = DATEDIFF ( RELATED ( DimCustomers[First Purchase Date] ), FactTransactions[InvoiceDate], MONTH )
```

---

## Tech Stack

Power Query (M) · DAX · Power BI Desktop · Star schema data modeling

---

## Repository Structure

```
Customer-Churn-Retention-Analysis/
├── README.md
├── Customer Churn & Retention Analysis.pbix          # Full Power BI report
├── customers.csv
├── transactions.csv
├── dashboard-theme.json
└── images/
```

---

## How to Use

1. Clone this repository
2. Open `Customer Churn & Retention Analysis.pbix` in Power BI Desktop
3. If prompted, repoint the data source to your local `customers.csv` / `transactions.csv`
4. Apply the theme via **View → Themes → Browse for themes** → `dashboard-theme.json`

---

## Author

Built by [Akshit Kumar Sahoo](https://github.com/AKS-19)