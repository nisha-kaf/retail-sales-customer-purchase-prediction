# Retail Sales & Customer Purchase Prediction

BCA 6th Semester Internship Project — Group A
20-Day Data Analytics & Machine Learning Student Project

## Project Overview

This project analyzes two years of transactional data from a UK-based online retailer and builds a machine
learning system to predict which customers are likely to become **high-value** in a future period, based
only on their earlier purchase history. The project follows the complete data analytics pipeline required
by the instructor's manual: problem definition, raw data, data dictionary, cleaning, NumPy/Pandas analysis,
exploratory data analysis, Matplotlib visualization, feature engineering, machine learning, evaluation,
business insights, and recommendations.

## Business Problem

A retail business wants to understand its sales performance and identify which orders and customers are
likely to be high-value, so it can prioritize retention, loyalty and marketing effort where it matters most.

## Objectives

- Understand overall sales performance: revenue trends, top products, top countries, order patterns.
- Build a transparent, reproducible definition of a "high-value customer."
- Predict, from a customer's historical purchase behavior alone, whether they will become high-value in a
  later period — without leaking future information into the prediction.
- Compare a baseline, Logistic Regression, and Random Forest Classifier on this task.
- Translate the findings into practical, evidence-based business recommendations.

## Dataset

- **File:** `online_retail_II.xlsx`
- **Sheets:** `Year 2009-2010` (525,461 rows), `Year 2010-2011` (541,910 rows) — identical schema, merged
  into 1,067,371 rows total.
- **Columns:** `Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID, Country`
- **Date range:** 2009-12-01 to 2011-12-09 (~24 months)
- **Countries:** 43 unique countries across both sheets (40 in 2009-2010, 38 in 2010-2011); predominantly United Kingdom (83.7% of valid-sales revenue)

## Data Source

**Dataset name:** Online Retail II
**Description:** Transaction-level data from a UK-based, non-store online retailer specializing in unique
all-occasion gift items, primarily selling to wholesale customers.
**Known limitations:**
- ~22.8% of rows have no `Customer ID` and cannot be attributed to a specific customer.
- The dataset predates current retail conditions (2009–2011) and does not reflect present-day pricing,
  behavior, or channel mix.
- No demographic, profit-margin, or customer-satisfaction data is included.

The uploaded workbook itself does not embed a citation or license field. The two sheet names
(`Year 2009-2010`, `Year 2010-2011`) and column structure match the publicly known "Online Retail II"
dataset. We are documenting exactly what is verifiable from the file; we are not asserting a specific
external URL or publisher beyond what the file itself shows.

## Technologies Used

Python, NumPy, Pandas, Matplotlib, Scikit-learn, Jupyter Notebook, openpyxl

## Project Workflow

```
Problem -> Data Source -> Raw Data -> Data Dictionary -> Data Cleaning -> NumPy/Pandas ->
Exploratory Data Analysis -> Matplotlib Visualization -> Feature Engineering -> Machine Learning ->
Model Evaluation -> Business Insights -> Recommendations
```

## Data Cleaning

Cleaning was rule-based and fully documented (see `notebooks/02_cleaning_and_eda.ipynb`, Section 3).
Nothing was silently deleted — every excluded row is flagged with a reason in
`data/cleaned/online_retail_cleaned_flagged.csv`.

| Rule | Rows affected |
|---|---|
| Exact duplicate rows removed | 34,335 |
| Cancelled invoices (flagged, excluded from sales analysis) | 19,104 |
| Adjustment/bad-debt invoices (flagged, excluded) | 6 |
| Non-product stock codes — postage, fees, manual entries (flagged, excluded) | 5,524 |
| Invalid (≤0) quantity (flagged, excluded) | 22,496 |
| Invalid (≤0) price (flagged, excluded) | 6,019 |
| Missing Customer ID (flagged, excluded from customer-level analysis) | 235,151 |
| **Valid completed sales used for analysis** | **776,624 rows (72.8% of merged data)** |

## Exploratory Data Analysis

12 business questions were answered directly from the cleaned data (monthly revenue trend, top products,
top countries, average order value, top customers, purchase timing by day/hour, quantity-revenue
relationship, purchase frequency vs. spend, and more). Full detail: `notebooks/02_cleaning_and_eda.ipynb`.

**Headline numbers:**
- Total revenue (valid completed sales): **17,073,078.29**
- Unique customers (valid sales): **5,861**
- Unique products (valid sales): **4,624**

## Machine Learning

**Target:** `HighValue` — whether a customer's spending in a later "target period" (2010-12-01 onward)
reaches the 75th percentile of target-period spend, calculated only among customers active in the earlier
"feature period."

**Leakage prevention:** All predictive features (`HistTotalSpend`, `HistNumOrders`, `HistAvgOrderValue`,
`HistNumUniqueProducts`, `HistRecencyDays`, `HistPurchaseFrequency`, `HistLifespanDays`, `HistNumItems`,
`IsUK`) are computed **only** from the feature period (before the cutoff). The label is computed **only**
from the target period. No feature is derived from the same window used to build the label.

**Models:** Logistic Regression and Random Forest Classifier, compared against a majority-class baseline.

## Model Evaluation

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| Baseline (always "not high-value") | 0.750 | 0.000 | 0.000 | 0.000 |
| Logistic Regression | 0.804 | 0.591 | 0.707 | 0.644 |
| Random Forest | 0.831 | 0.644 | 0.722 | **0.681** |

Random Forest performs best across all four metrics. Both trained models substantially outperform the
baseline on precision, recall, and F1 — the baseline cannot identify a single high-value customer despite
its deceptively high accuracy, which is exactly why accuracy alone is the wrong metric for this problem.

**Top predictive features (Random Forest):** `HistTotalSpend`, `HistNumItems`, `HistNumOrders` — historical
spending behavior, not country, drives the prediction.

## Key Insights

- Revenue is strongly seasonal, peaking around October–November each year.
- A small set of products and a small set of customers account for a disproportionate share of revenue.
- The business is heavily UK-concentrated, both in revenue and customer count.
- About 25% of customers active in a given period go on to become high-value in the following period.
- Historical spend/order-count features are far more predictive of future high-value status than country.

## Recommendations

| Finding | Interpretation | Recommendation |
|---|---|---|
| Small set of products drives most revenue | These items are core demand drivers | Prioritize inventory availability and promotion for top-revenue products |
| Revenue peaks Oct–Nov | Strong seasonal, likely holiday-driven demand | Plan seasonal marketing and staffing/inventory buildup ahead of this window |
| ~25% of customers become high-value | A meaningful, identifiable minority drives disproportionate value | Use the trained model to flag likely high-value customers early and prioritize them for loyalty/retention programs |
| Historical spend/frequency are the strongest predictors | Past engagement is the best available signal of future value | Focus retention efforts on customers showing early signs of increasing order frequency or spend, rather than on demographic/geographic targeting |
| Business is UK-concentrated | International markets are comparatively under-penetrated | Evaluate targeted expansion or marketing in secondary markets, informed by further country-level analysis |

## Project Structure

```
project-name/
├── README.md
├── data/
│   ├── raw/                 online_retail_II.xlsx (untouched original)
│   ├── cleaned/              flagged full dataset, valid-sales subset, full-period customer features
│   └── model_ready/          leakage-safe historical features + high-value label
├── notebooks/
│   ├── 01_data_profiling.ipynb
│   ├── 02_cleaning_and_eda.ipynb
│   └── 03_machine_learning.ipynb
├── src/
├── outputs/
│   ├── charts/                17 Matplotlib PNGs
│   └── model_results/         model comparison table (CSV)
├── report/
│   └── final_report.pdf
├── presentation/
│   └── final_presentation.pptx
└── requirements.txt
```

## Team Members & Contribution Plan

Nisha Kafle · Soniya Pandey · Susmita Balami · Swikriti Bhusal · Pratish Shakya


| Role | Focus | Should be able to explain in viva |
|---|---|---|
| Nisha Kafle — Member 1, Project Lead / Business Analyst | Problem statement, objectives, KPIs, coordination | Why this business problem matters; how objectives map to the analysis |
| Soniya Pandey — Member 2, Data Collection & Quality | Source documentation, data-quality report (Notebook 1) | Every data-quality issue found and why each cleaning rule was chosen |
| Susmita Balami — Member 3, Python Data Analyst | Cleaning, merging, feature engineering (Notebook 2) | How the two sheets were merged; how each derived variable is calculated |
| Swikriti Bhusal — Member 4, Visualization Analyst | 12 EDA questions and Matplotlib charts (Notebook 2) | What each chart shows and its business interpretation |
| Pratish Shakya — Member 5, Machine Learning Analyst | Target design, leakage prevention, modeling (Notebook 3) | Why the time-split prevents leakage; what each metric means for this business problem |

Every member should be able to explain the full pipeline end-to-end, not just their own section.

## How to Run

1. Install dependencies: `pip install -r requirements.txt`
2. Run the notebooks in order:
   - `notebooks/01_data_profiling.ipynb`
   - `notebooks/02_cleaning_and_eda.ipynb`
   - `notebooks/03_machine_learning.ipynb`
3. Each notebook reads from `data/` and writes its outputs back into `data/` and `outputs/`.
4. Two large cleaned files (`online_retail_cleaned_flagged.csv`, `online_retail_valid_sales.csv`) are not stored in GitHub because they exceed its 100 MB file limit. They are recreated when you run Notebook 2, which must be run before Notebook 3.

## Requirements

See `requirements.txt`. Only libraries actually used in the notebooks are listed.

## Limitations

- Public/historical dataset (2009–2011); does not reflect current customer behavior.
- ~22.8% of rows had no Customer ID and were excluded from customer-level analysis.
- No demographic, profit-margin, or satisfaction data available.
- The high-value threshold (75th percentile) is a business choice, not an absolute definition; a different
  choice would change the class balance and reported metrics.
- Prediction and feature importance indicate association, not causation.
- Model performance depends on the specific historical/target period split used.
