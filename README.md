# COVID-19 Global Trend Analysis & Forecasting
**Project Ref:** JohnsHopkins_COVID19_Analysis · DataMites AI Capstone 01  
**Domain:** Time Series · Epidemiological Data Analysis · Forecasting  
**Data Source:** Johns Hopkins University CSSE COVID-19 Repository  
**Models:** ARIMA · Facebook Prophet

---

## Project Summary

Built a complete COVID-19 data analysis and forecasting pipeline using Johns Hopkins time-series data for 188 countries. The project identifies the hardest-hit nations, visualizes epidemic trajectories, and forecasts daily new cases for the UAE using classical (ARIMA) and modern (Prophet) time-series models.

---

## Key Results

### Global Snapshot (as of September 21, 2020)

| Country | Confirmed Cases | Deaths |
|---------|-----------------|--------|
| United States | 6,856,884 | 199,865 |
| India | 5,487,580 | 87,882 |
| Brazil | 4,558,040 | 137,272 |
| Russia | 1,105,048 | 19,420 |
| Colombia | 770,435 | 24,397 |

### UAE Forecasting Model Comparison

| Model | RMSE | MAE | Winner |
|-------|------|-----|--------|
| ARIMA (5, 1, 0) | 284.48 | 233.17 | |
| **Prophet** | **210.30** | **172.26** | **Best Model** |

> Prophet outperformed ARIMA by **26% on RMSE** — its built-in weekly and yearly seasonality captured COVID reporting patterns that ARIMA's non-seasonal specification missed.

---

## Dataset

- **Source:** Johns Hopkins University CSSE (public repository)
- **Files:** `time_series_covid19_confirmed_global.csv` · `time_series_covid19_deaths_global.csv`
- **Date Range:** January 22, 2020 → September 21, 2020
- **Merged Records:** 64,904 rows (country × date combinations)
- **Countries:** 188 unique country/region entries
- **Missing Values:** Province/State filled with "Unknown"; none in core metrics

---

## Pipeline

```
Raw CSV (confirmed + deaths, global, Jan–Sep 2020)
     ↓
Load & Melt (wide → long format, date columns → rows)
     ↓
Merge confirmed + deaths on [Country, Province, Date]
     ↓
Date parsing (explicit format '%m/%d/%y' → avoids FutureWarning)
     ↓
EDA
  • Top 10 countries by confirmed cases & deaths
  • Cumulative curves for US, India, Brazil, UAE
     ↓
UAE Isolation → Daily New Cases calculation
     ↓
Train / Test Split (last 30 days = test)
     ↓
┌───────────────────┬──────────────────────────────────────────┐
│  ARIMA (5, 1, 0)  │  Prophet (daily + weekly + yearly)       │
│  statsmodels      │  daily_seasonality=True, weekly=True      │
│  RMSE: 284.48     │  RMSE: 210.30  ← best model              │
└───────────────────┴──────────────────────────────────────────┘
     ↓
Historical + Forecast visualization
```

---

## EDA Highlights

- **US, India, Brazil** dominated both confirmed cases and death counts by a wide margin
- **UAE** curve showed steady upward growth with visible weekly reporting cycles (lower case counts on weekends)
- Cumulative death-to-case ratio varied significantly across countries, reflecting healthcare capacity, testing, and reporting differences
- Prophet detected the **weekly seasonality pattern** in UAE's daily new cases — a known artifact of weekend under-reporting

---

## Model Details

### ARIMA (5, 1, 0)
- Fitted on UAE daily new cases (`diff()` of cumulative confirmed)
- Order selected by manual specification; d=1 for stationarity
- Limitations: no seasonality component → missed weekly reporting cycles → higher error

### Prophet
- Facebook's additive decomposition model
- `daily_seasonality=True`, `weekly_seasonality=True`, `yearly_seasonality=True`
- Handles missing data and multiple seasonality levels automatically
- Predictions aligned to test index with `.index` reassignment to correct NaN alignment bug

---

## Key Engineering Decisions

| Decision | Why |
|----------|-----|
| Explicit date format `'%m/%d/%y'` | Avoided `FutureWarning` from ambiguous auto-parsing |
| `.asfreq('D')` on UAE series | Ensured monotonic daily index for ARIMA — resolved `ValueWarning` |
| `predictions.index = test_data.index` | Fixed NaN alignment when merging ARIMA + Prophet predictions |
| Last 30 days as test set | Clinically meaningful evaluation window; Prophet needs training history |
| `class_weight` not applicable | Regression task on count data — no class imbalance issue |

---

## Government & Public Health Recommendations

Based on the analysis:

1. **Deploy rolling forecasting** — Weekly retraining of Prophet with new case data enables 2–4 week ahead projections for resource planning (ICU beds, ventilators, testing kits)
2. **Target high-burden countries** — US, Brazil, India required differentiated intervention strategies due to scale differences
3. **Standardize reporting** — Weekend reporting gaps introduce systematic error; standardized daily reporting would improve all forecast models
4. **Confidence intervals** — For policy decisions, add Prophet's built-in uncertainty intervals (`forecast['yhat_lower']` / `'yhat_upper'`)
5. **Extend with exogenous variables** — Incorporate vaccination rates, mobility indices, policy stringency for ARIMAX/Prophet with regressors

---

## Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Wide-format date columns (248 date columns) | `.melt()` to convert to long format |
| `UserWarning` from ambiguous date parsing | Specified `format='%m/%d/%y'` explicitly |
| ARIMA `ValueWarning` — non-monotonic index | `.sort_index()` + `.asfreq('D')` before fitting |
| Prophet predictions returning NaN on merge | Reassigned test index to Prophet output Series |
| COVID reporting gaps (weekends) | Prophet's weekly seasonality component handled this naturally |

---

## Technologies

`Python` · `Pandas` · `NumPy` · `statsmodels` · `Facebook Prophet` · `scikit-learn` · `Matplotlib` · `Seaborn`

---

## Files

| File | Description |
|------|-------------|
| `JohnsHopkins_COVID19_Analysis_capstone_01.ipynb` | Full analysis notebook |

---

*Project completed as part of the DataMites Certified AI Expert Programme.*  
*Author: Reema Hussain Mohamed Sulthan · PharmD · AI Expert (IABAC)*
