# Loan Default Risk: Data Cleaning, Exploratory Analysis & Interactive Dashboard

**Tools:** Python (pandas, matplotlib, seaborn), Tableau Public
**Dataset:** ~32,600 anonymized personal loan applications — customer demographics, loan terms, credit grade, and current default status

**Live interactive dashboard:** [Add Tableau Public link]

## Overview

This project cleans and explores a raw, messy loan-level dataset to understand what actually drives default risk. The focus wasn't just on producing charts — it was on treating every cleaning decision as something worth documenting, and questioning *why* data was missing rather than just filling gaps automatically. That second habit surfaced the most important finding in the whole analysis.

## Process

### 1. Cleaning (Python / pandas)

Starting from the raw file (32,586 rows, 13 columns), I:

- Converted currency-formatted fields (`loan_amnt` stored as `"£35,000.00"`, `customer_income` with comma separators) to numeric types
- Removed 6 exact duplicate rows and dropped 4 rows with a missing target value (`Current_loan_status`), since there's no reasonable way to impute the outcome itself
- Caught and removed implausible records that a simple `.describe()` wouldn't flag on its own — applicants aged 3 and 144, and an employment history longer than the applicant's own age minus 14 years
- Removed a handful of extreme income/loan-amount outliers (income up to £6,000,000, one loan of £3,500,000) that were multiple orders of magnitude beyond the rest of the distribution
- Imputed missing interest rates using the median rate **per loan grade** (rather than one global median), since grade is a direct driver of rate

### 2. Catching a Data Leakage Signal

`historical_default` (whether the applicant had defaulted before) was missing for 64% of rows — too much to drop, and too much to impute a guessed value for two-thirds of the dataset. Rather than pick one of those two flawed options, I kept missing values as an explicit `"Unknown"` category and checked whether the three groups (`Y` / `N` / `Unknown`) behaved sensibly.

They didn't. **Every row where the field was "Unknown" had a current default rate of exactly 0%, while both populated categories (Y and N) showed high default rates (37% and 80%).** That's not what a randomly-missing field looks like — it strongly suggests the field was only ever recorded when there was a reason to check someone's default history, meaning its *absence* was quietly leaking the outcome.

Rather than treat this as a modeling feature, I flagged it as a data governance issue: this field would need to go back to whoever owns the source system before it's used in any predictive model, since using it as-is would badly overstate a model's real accuracy.

### 3. Exploratory Analysis (Python)

With that caveat noted, the rest of the analysis focused on the remaining, reliably-populated fields:

- Default rate by loan grade, loan purpose, and home ownership
- Interest rate and loan-to-income ratio, compared across defaulted vs. non-defaulted loans
- Income vs. loan amount, colored by outcome

### 4. Interactive Dashboard (Tableau Public)

The cleaned dataset was rebuilt as an interactive Tableau dashboard so the same findings could be explored by a non-technical audience (e.g. a loan officer) without needing to read code. The dashboard includes:

- KPI tiles for overall default rate and total applicants
- Default rate by loan grade and by home ownership, with click-to-filter enabled on both charts
- Default rate by loan purpose
- An income-vs-loan-amount scatter plot colored by outcome
- A loan purpose filter that cross-applies to every chart at once

**View it live:** [Add Tableau Public link]

## Key Findings

- **Loan grade is by far the strongest predictor of default** — a clean, monotonic climb from 10% (Grade A) to 72% (Grade E)
- **`historical_default`'s missingness pattern likely leaks the target** — a data quality/governance finding, not just a cleaning step (see above)
- **Renters default roughly 5x more often than homeowners** (31% vs. 6%), and **debt consolidation loans default most by purpose** (30%) — both plausible proxies for existing financial strain
- **Loan-to-income ratio and interest rate both move with default risk in the expected direction** (medians of 0.23 and 13.6% for defaults vs. 0.13 and 10.6% for non-defaults), and alongside grade would likely anchor a simple risk-scoring model
- Overall default rate across the cleaned dataset: **21%**

## Skills Demonstrated

- Data cleaning and validation on a real-world-messy dataset: currency parsing, duplicate detection, implausible-value detection using cross-field logic (not just single-column outlier checks)
- Recognizing and diagnosing a data leakage risk from a missingness pattern, rather than defaulting to a standard imputation strategy
- Exploratory data analysis and visualization (matplotlib/seaborn): grouped bar charts, boxplots, scatterplots
- Building an interactive, filterable dashboard in Tableau Public, including cross-filtering across multiple charts from a single click or dropdown
- Translating statistical patterns into plain-language, decision-relevant findings and a clearly scoped "next step" (a risk-scoring model, with leakage explicitly excluded)

*Full analysis, code, and charts in `loan_default_analysis.ipynb`.*
