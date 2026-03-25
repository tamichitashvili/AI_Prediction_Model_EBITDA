# EBITDA Estimation for Dutch Companies

A machine learning pipeline that estimates EBITDA for Dutch companies using UK company financials as training data. Built for use in Google Colab.

## Overview

Dutch company datasets (sourced from Orbis/BvD) typically have EBITDA missing for the majority of companies. This notebook trains regression models on UK companies — where EBITDA is consistently reported — and applies the best-performing model to estimate EBITDA for Dutch companies where it is unavailable.

**Training data:** UK companies (Orbis extract, ~95k companies, EBITDA always present)  
**Validation data:** Dutch companies where EBITDA is known (~5.5k companies, used as a cross-country hold-out)  
**Prediction target:** Dutch companies with missing EBITDA (~109k companies)

## Models Compared

| Model | Notes |
|---|---|
| Ridge Regression | Interpretable baseline |
| Random Forest | Robust to outliers, minimal tuning required |
| XGBoost | Gradient boosting with early stopping; typically best performer |
| Neural Network (MLP) | 3-layer network (256→128→64), captures non-linear interactions |

The best model is selected automatically based on R² on the Dutch validation set and used for final predictions.

## Features Engineered

- **Financial levels:** revenue, total assets, net income, shareholders funds, employees (latest year + 3 historical years)
- **Growth rates:** year-over-year changes in revenue, assets, employees, net income
- **Efficiency ratios:** net margin, asset turnover, equity ratio, revenue per employee
- **Log-transforms** on skewed monetary variables
- **NACE sector dummies** (top 20 two-digit divisions)
- **Region dummies** (top 20 regions)

## Outputs

- `NL_EBITDA_predictions_<ModelName>.xlsx` — predicted EBITDA for Dutch companies where it was missing
- `NL_companies_EBITDA_complete.xlsx` — full Dutch dataset with EBITDA filled in (known + predicted), with a `ebitda_source` column indicating provenance

## How to Run

### 1. Open in Google Colab
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

Go to [colab.research.google.com](https://colab.research.google.com) → **File → Open notebook → GitHub**, paste this repo URL.

### 2. Prepare your data files

You will need the following files from Orbis/Bureau van Dijk:

| File | Description |
|---|---|
| `20230112_Consolidated_Orbis_extract_UK_companies_v1.xlsb` | UK companies extract (training data) |
| `20230112_Consolidated_Orbis_extract_Dutch_companies_v1.xlsb` | Dutch companies extract (prediction target) |

> ⚠️ These data files are **not included** in this repository due to their size.

### 3. Run the notebook

Execute all cells in order. When prompted in **Step 2**, upload both `.xlsb` files. The notebook will handle everything else automatically, and prompt you to download the output files at the end.

## Data Requirements

The notebook expects Orbis extracts with the following fields per company:

- Company name, BvD ID, Orbis ID
- NACE Rev. 2 code (4-digit) and description
- Region in country
- Number of employees (latest year + 3 historical)
- Number of current directors & managers
- Operating revenue / Turnover (latest year + 3 historical), m EUR
- P/L for period (net income) (latest year + 3 historical), m EUR
- Other shareholders funds (latest year + 3 historical), m EUR
- Total assets (latest year + 3 historical), m EUR
- EBITDA (latest year + 3 historical), m EUR — must be present for UK, may be missing for NL

See `20231201_Data_dictionary_Orbis_extracts.xlsx` for full field descriptions.

## Repository Structure

```
.
├── EBITDA_Estimation_NL_Companies.ipynb   # Main notebook
├── README.md                              # This file
└── .gitignore                             # Excludes data files
```

## Requirements

All dependencies are installed automatically in the first cell of the notebook:

```
pyxlsb       # Read .xlsb Excel binary files
xgboost      # Gradient boosting
shap         # Model explainability
scikit-learn # ML utilities, Random Forest, Ridge, MLP
pandas, numpy, matplotlib, seaborn
```

## Results Interpretation

- **R²** — proportion of EBITDA variance explained (higher is better; 1.0 = perfect)
- **MAE** — mean absolute error in millions EUR (lower is better)
- **RMSE** — root mean squared error in millions EUR (lower is better, penalises large errors more)

Models are evaluated on both the UK test set (20% hold-out) and the Dutch validation set. The Dutch validation performance is the primary selection criterion, as it reflects true cross-country generalisation.

## Caveats

- EBITDA predictions are estimates based on observable financial and structural features. They should be used as indicative figures, not as substitutes for audited accounts.
- The model is trained on UK companies; differences in Dutch accounting standards, industry mix, or company size distribution may introduce systematic bias.
- Very small companies (revenue < 0.1m EUR) and conglomerates with complex group structures may have lower prediction accuracy.

