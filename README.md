# Bank Marketing Campaign: Predicting Term Deposit Subscription

Predicting which clients are likely to subscribe to a term deposit *before* they're called, so a bank's contact center can prioritize its limited calling capacity toward the highest-propensity clients.

## Results

| Metric | Value |
|---|---|
| **Top-20% Model Lift** (target: 2.0x) | **3.52x** |
| Top-scored 20% conversion rate | 39.68% |
| Baseline conversion rate | 11.27% |
| Best model (deployable, no leakage) | Random Forest — F1: 0.514, ROC-AUC: 0.812 |

The model lets the bank reach the same number of conversions while calling a much smaller share of its client list — or convert far more clients for the same number of calls.

## The Business Problem

Outbound telemarketing is expensive per call, and only ~11% of contacted clients subscribe. Instead of calling everyone, this project builds a propensity model to score and rank clients before they're dialed.

## Data

[UCI Bank Marketing Dataset](https://archive.ics.uci.edu/dataset/222/bank+marketing) (`bank-additional-full.csv`) — 41,188 telemarketing contacts from a Portuguese retail bank (2008–2010), 21 variables.

## Approach

**Data cleaning**
- Removed 12 duplicate rows
- Mode-imputed low-missingness categorical columns (job, marital, education, housing, loan)
- Kept `default`'s "unknown" as its own category (20.9% missing — too much to safely impute)
- Converted the `pdays` sentinel value (999 = "never contacted") into an explicit flag + cleaned distance
- Winsorized outliers in `campaign` and `duration` at the 99th percentile

**Feature engineering**
- `age_group`, `duration_min`, `prior_success`, `quarter`, `total_contacts`

**Leakage-aware modeling**
`duration` (call length) is only known *after* a call ends — using it would be leakage in a real deployment. Every model was trained twice: once with `duration` (a benchmark/ceiling) and once without (the honest, deployable version).

| Feature Set | Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| Without duration | **Random Forest** | 0.872 | 0.448 | 0.601 | **0.513** | **0.812** |
| Without duration | Logistic Regression | 0.829 | 0.356 | 0.643 | 0.459 | 0.800 |
| Without duration | Decision Tree | 0.826 | 0.351 | 0.641 | 0.454 | 0.780 |
| Without duration | Linear SVM | 0.897 | 0.629 | 0.203 | 0.306 | 0.800 |

Random Forest was selected for deployment — best F1 and ROC-AUC among deployable models, capturing non-linear interactions between economic indicators, prior-contact history, and demographics.

## Key Findings

- **Prior relationship with the bank dominates** — a past campaign success is the strongest signal in the data, far more than any demographic attribute.
- **Timing matters as much as targeting** — lower-volume months (March, September, December) convert far better than the bank's high-volume summer push.
- **Life-stage edges convert best** — students (31.4%) and retirees (25.3%) subscribe at 2–3x the rate of core working-age clients.
- **Macroeconomic conditions matter** — higher interest rates and stronger employment correlate with *lower* subscription rates.

## Dashboard

Interactive 3-tab Tableau dashboard (Overview & KPI → Deep-Dive Drivers → Model Performance): **[Tableau Public link]**

![Dashboard screenshot](dashboard/screenshot.png)

## Tech Stack

`Python` · `pandas` · `scikit-learn` · `Tableau` — data cleaning, feature engineering, classification modeling, and dashboard design.

## Repo Structure

```
├── notebooks/          # data cleaning, EDA, modeling
├── dashboard/           # Tableau workbook + screenshot
├── report/              # full written report
└── README.md
```

## Full Report

See [`report/Project_Report.pdf`](report/Project_Report.pdf) for the complete writeup, including the KPI justification, AI-assisted insight generation process, and detailed business recommendations.
