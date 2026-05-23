# FNB DataQuest — Building Interpretable Credit Models

> **Student:** Sbonokuhle Myeni · `MYNSBO001` · mynsbo001@myuct.ac.za
---

## At a Glance

| Metric | Value |
|---|---|
| Dataset size (clean) | 120 358 records |
| Original variables | 26 |
| Engineered features | 35+ |
| AUC | **80.21%** |
| Gini Coefficient | **60.43%** |
| Balanced Accuracy | **72.88%** |
| Optimal Threshold | 0.4876 (Youden's J) |

---

## Overview

Credit risk modelling assists lending institutions in evaluating the likelihood that a borrower will default on a loan. In regulated banking environments, predictive accuracy must be balanced against model **interpretability** — lending decisions must remain transparent and explainable to satisfy regulatory scrutiny and enable adverse-action notices.

This project developed an interpretable credit-default prediction model using a simulated FNB loan application dataset of over 120 000 observations. The study explored the trade-off between interpretability and performance by applying **logistic regression** enhanced with:

- **Weight of Evidence (WoE)** transformation
- **Information Value (IV)** feature selection
- **Elastic Net** regularisation

Extensive EDA and data preprocessing were carried out to handle missing values, outliers, inconsistent categorical labels, and class imbalance before model fitting.

---

## Dataset

The raw dataset consisted of **120 960 observations** across **26 variables**, covering demographic, financial, behavioural, and credit-related information:

| Category | Variables |
|---|---|
| Demographic | `age`, `region`, `months_at_current_address` |
| Financial | `annual_income`, `dti_ratio`, `loan_amount`, `interest_rate`, `total_revolving_balance` |
| Credit behaviour | `credit_utilisation_pct`, `num_delinquencies_2yr`, `num_hard_inquiries_6mo`, `pct_accounts_current`, `months_since_last_delinquency` |
| Account history | `num_open_accounts`, `months_since_oldest_account`, `employment_length_years` |
| Application | `loan_purpose`, `application_date`, `application_dow`, `home_ownership` |
| Digital | `phone_verified`, `email_domain_type` |
| Target | `default_flag` (1 = default, 0 = no default) |

**Class distribution:** ~84% non-default · ~16% default (imbalanced)

---

## Modelling Pipeline

### Step 1: Data Cleaning

- Removed `applicant_id_hash` (unique identifier, not predictive)
- Removed **602 duplicate records** → 120 358 remaining
- Standardised inconsistent categorical labels (e.g. `RENT` / `RENTING` / `rent`) by uppercasing and stripping special characters
- Unified four mixed date formats (`YYYY-MM-DD`, `MM/DD/YYYY`, `DD/MM/YYYY`, `DD-Mon-YYYY`) into a single `YYYY-MM-DD` standard

---

### Step 2: Missing Value Treatment

| Variable | Missing | % | Treatment |
|---|---|---|---|
| `annual_income` | 8 638 | 7.18% | Median imputation |
| `employment_length_years` | 3 711 | 3.08% | Median imputation |
| `num_open_accounts` | 2 423 | 2.01% | Median imputation |
| `months_since_last_delinquency` | 60 068 | **49.91%** | MNAR — binary missingness indicator created |

`months_since_last_delinquency` was treated as **Missing Not At Random (MNAR)** — the absence of a delinquency record is itself informative (the borrower likely has no delinquency history). A binary indicator variable was created rather than imputing directly.

Median formula used:

$$x_i = \begin{cases} x_i & \text{if observed} \\ \tilde{x} & \text{if missing} \end{cases}$$

---

### Step 3:  Outlier Removal

Tukey fence method applied to `annual_income` and `loan_amount`:

$$\text{Lower Fence} = Q_1 - 1.5 \times IQR \qquad \text{Upper Fence} = Q_3 + 1.5 \times IQR$$

This removed extreme values responsible for the long right tails and resulted in an **~8% reduction in records**.

---

### Step 4: Exploratory Data Analysis

**Key findings from univariate analysis:**
- `annual_income`, `loan_amount`, and `total_revolving_balance` are strongly right-skewed → log transforms needed
- `num_delinquencies_2yr` and `num_hard_inquiries_6mo` are zero-inflated count variables
- `months_since_last_delinquency` exhibits a zero-inflated structure with a long tail

**Key findings from bivariate analysis (vs `default_flag`):**
- Most categorical variables show only minor differences in default proportions
- `home_ownership` (renters default more) and `loan_purpose` (medical/small business loans default more) show slightly stronger patterns
- Numerical variables show clearer separation: `credit_utilisation_pct`, `interest_rate`, `dti_ratio`, `num_delinquencies_2yr`, and `num_hard_inquiries_6mo` are the most informative

**Key findings from correlation analysis:**
- Most variables show weak correlations → limited multicollinearity
- Strongest relationship: `age` ↔ `months_since_oldest_account` (r = 0.95)
- Notable pairs: `interest_rate` ↔ `dti_ratio` (0.43), `credit_utilisation_pct` ↔ `total_revolving_balance` (0.38)

---

### Step 5: Feature Engineering

Over 35 features were engineered to improve predictive power:

| Feature | Type | Description |
|---|---|---|
| `log_annual_income` | Log transform | Reduces right skew in income |
| `log_loan_amount` | Log transform | Reduces right skew in loan size |
| `log_total_revolving_balance` | Log transform | Reduces right skew in revolving balance |
| `log_employment_length_years` | Log transform | Stabilises employment duration |
| `monthly_income` | Derived | Annual income ÷ 12 |
| `credit_history_ratio` | Derived | Oldest account age ÷ borrower age |
| `dti_risk` | Binary flag | DTI > 40% threshold indicator |
| `risk_flag` | Categorical | Risk category based on delinquency behaviour |
| `delinquency_band` | Ordinal | None / Low / Moderate / High |
| `revolving_to_income` | Ratio | Revolving balance ÷ annual income |
| `income_per_emp_yr` | Ratio | Income ÷ employment length |
| `repayment_stability` | Ratio | Account repayment consistency relative to delinquency |
| `inquiries_per_account` | Ratio | Hard inquiries ÷ open accounts |
| `util_x_dti` | Interaction | Credit utilisation × DTI ratio |
| `interest_burden` | Interaction | Interest rate × DTI ratio |
| `utilisation_band` | Ordinal | Low / Moderate / High credit utilisation |
| `exposure_score` | Composite | Loan amount + revolving balance |

---

### Step 6: Information Value & WoE Encoding

IV was computed **on the training set only** to prevent data leakage. Features with IV < 0.02 were dropped.

#### IV Thresholds

| IV Range | Predictive Strength |
|---|---|
| < 0.02 | Useless — dropped |
| 0.02 – 0.10 | Weak |
| 0.10 – 0.30 | Medium |
| 0.30 – 0.50 | Strong |
| > 0.50 | Suspicious (possible leakage) |

#### Top Features by IV

| Feature | IV | Strength |
|---|---|---|
| `repayment_stability` | 0.578 | ⚠️ Suspicious |
| `interest_rate` | 0.534 | ⚠️ Suspicious |
| `log_loan_amount` / `loan_amount` | 0.526 | ⚠️ Suspicious |
| `exposure_score` | 0.470 | 🟢 Strong |
| `total_revolving_balance` | 0.444 | 🟢 Strong |
| `risk_score_raw` | 0.399 | 🟢 Strong |
| `ever_delinquent` | 0.349 | 🟢 Strong |
| `num_delinquencies_2yr` | 0.343 | 🟢 Strong |
| `delinquency_band` | 0.340 | 🟢 Strong |
| `months_since_oldest_account` | 0.283 | 🔵 Medium |
| `util_x_dti` | 0.274 | 🔵 Medium |
| `annual_income` | 0.221 | 🔵 Medium |
| `credit_utilisation_pct` | 0.157 | 🔵 Medium |
| `dti_ratio` | 0.130 | 🔵 Medium |

**WoE formula for bin i:**

$$\text{WoE}_i = \ln\left(\frac{\text{Distribution of Events}_i}{\text{Distribution of Non-Events}_i}\right)$$

WoE encoding converts all features onto a common log-odds scale, enforces monotone score functions, and handles both numeric and categorical variables uniformly.

---

### Step 7: Elastic Net Logistic Regression

Elastic Net regularisation combines L1 (Lasso) and L2 (Ridge) penalties:

$$\min_{\beta} \left\{ \sum_{i=1}^{n}(y_i - \hat{y}_i)^2 + \lambda_1 \sum_{j=1}^{p}|\beta_j| + \lambda_2 \sum_{j=1}^{p}\beta_j^2 \right\}$$

- **L1** shrinks some coefficients to exactly zero (feature selection)
- **L2** stabilises correlated predictors (prevents blow-up)
- Optimal **λ = 0.001443** via 10-fold cross-validation
- Class weights applied to address the 84% / 16% imbalance
- Optimal classification threshold **0.4876** selected by maximising Youden's J statistic

---

## Results

### Performance Metrics

| Metric | Value | Notes |
|---|---|---|
| **AUC** | **80.21%** | Exceeds the industry benchmark of 75% |
| **Gini Coefficient** | **60.43%** | Industry benchmark ≥ 50%; target met |
| **Balanced Accuracy** | **72.88%** | Reliable under class imbalance |
| Accuracy | 71.94% | Less meaningful under imbalance |
| Sensitivity (Recall) | 74.26% | 74% of defaulters correctly identified |
| Specificity | 71.49% | 71% of non-defaulters correctly identified |
| Precision (PPV) | 33.38% | Low — driven by class imbalance |
| F1 Score | 46.06% | Reflects precision–recall trade-off |

### Interpretation

The model achieves **strong rank-ordering performance** with AUC > 0.80 and Gini > 0.60, meeting industry benchmarks for credit scorecards. Sensitivity and specificity are well balanced at the optimal threshold. The lower precision is expected given the 84/16 class split and is not unusual for credit scoring applications. Further threshold calibration or cost-sensitive learning could improve precision if business requirements demand fewer false positives.

---

## Regulatory Considerations

Although this dataset is simulated, the following features would draw scrutiny from real-world financial regulators (CFPB · FCA · FSCA):

| Feature | Regulatory Risk |
|---|---|
| `age` | Age discrimination laws (e.g. US Equal Credit Opportunity Act) prohibit using age as a primary credit factor for applicants over 40 |
| `region` | Regional proxies can correlate with race, ethnicity, or socio-economic status — constitutes indirect discrimination (redlining) |
| `phone_verified` | Digital-divide proxy — unverified phones correlate with lower-income demographics; potential disparate impact |
| `email_domain_type` | Free vs corporate email may proxy income or occupation — possible protected-class proxy |

> **Best practice:** All model features should be subject to a **Disparate Impact Analysis** before production deployment to ensure no disproportionate effect on protected classes, even where discrimination is unintentional.

---

## Project Structure

```
fnb-dataquest/
├── data/
│   ├── raw/                   # Original dataset
│   └── processed/             # Cleaned & engineered dataset
├── notebooks/
│   ├── 01_eda.Rmd             # Exploratory data analysis
│   ├── 02_cleaning.Rmd        # Data cleaning & imputation
│   ├── 03_feature_eng.Rmd     # Feature engineering
│   └── 04_modelling.Rmd       # IV, WoE, model training & evaluation
├── outputs/
│   ├── figures/               # All EDA and model plots
│   └── model/                 # Saved model object & coefficients
├── DataQuest-2026-Report.pdf  # Full project report
└── README.md
```

---

## Tech Stack

| Tool | Purpose |
|---|---|
| R | Core language |
| `tidyverse` | Data manipulation & visualisation |
| `glmnet` | Elastic Net logistic regression |
| `scorecard` | WoE binning & IV computation |
| `ggplot2` | Visualisations |
| `caret` / `pROC` | Model evaluation & ROC analysis |
| `lubridate` | Date parsing & standardisation |

---

## Key References

- Hand, D.J. & Henley, W.E. (1997). Statistical classification methods in consumer credit scoring. *Journal of the Royal Statistical Society*.
- Siddiqi, N. (2006). *Credit Risk Scorecards: Developing and Implementing Intelligent Credit Scoring*. Wiley.
- Zou, H. & Hastie, T. (2005). Regularization and variable selection via the Elastic Net. *Journal of the Royal Statistical Society, Series B*.
- Youden, W.J. (1950). Index for rating diagnostic tests. *Cancer, 3*(1), 32–35.

---
