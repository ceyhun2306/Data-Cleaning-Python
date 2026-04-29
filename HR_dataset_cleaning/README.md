# HR Dataset — Data Cleaning Project

## Overview
This project demonstrates a professional-grade data cleaning pipeline
applied to a real-world HR dataset containing 10,350 employee records.

The goal is not simply to make the data "clean" — but to make every
cleaning decision transparent, justified, and analytically sound.

## Dataset
- **Domain:** Human Resources — employee records for an Azerbaijani company
- **Raw shape:** 10,350 rows × 21 columns
- **Final shape:** ~10,000 rows × 25+ columns
- **Features:** Demographics, salary, performance, training, tenure, contact info

## Cleaning Pipeline

### 1. Duplicate Detection & Removal
- **168 exact duplicate rows** removed — identical records caused by system export errors
- **350 duplicate `employee_id` values** resolved — same employee recorded twice
  with minor formatting differences (e.g. `"Finance"` vs `"finance"`)
- Strategy: for each duplicate group, kept the row with the most non-null values
  to preserve as much information as possible

### 2. Type Conversion
All columns were stored as raw strings or incorrect types:

| Column | Before | After | Problem |
|---|---|---|---|
| `salary` | str | float | Mixed formats: `"1661.29 AZN"`, `"$2071.57"` |
| `hire_date` | str | datetime | Unparsed string |
| `last_review_date` | str | datetime | Unparsed string |
| `is_active` | str | bool | 6 formats: `"Yes"`, `"1"`, `"True"`, `"No"`, `"0"`, `"False"` |
| `age` | float64 | Int64 | Should be whole number |

### 3. Categorical Standardization
| Column | Before | After |
|---|---|---|
| `gender` | 12 unique values | 2 — `Male` / `Female` |
| `department` | 21 unique values | 8 — title case standardized |
| `phone` | `"none"`, `"unknown"` as strings | Converted to `NaN` |

### 4. Outlier Analysis
All flagged values were investigated using IQR bounds and domain knowledge.
No rows were dropped — only the specific erroneous values were nullified.

| Column | Issue | Decision |
|---|---|---|
| `age` | min = -5, max = 999 | Values below 18 or above 80 → `NaN` |
| `salary` | `-500`, `0.01`, `999999` | Impossible/sentinel values → `NaN` |
| `bonus` | Negative values | Impossible in HR context → `NaN` |
| `training_hours` | max = 998 hours | Above IQR upper bound → `NaN` |
| `overtime_hours` | max = 796 hours | Above IQR upper bound → `NaN` |

### 5. Missing Data Analysis
Each column was classified as MCAR, MAR, or MNAR through statistical
investigation before any imputation decision was made.

| Column | Classification | Decision |
|---|---|---|
| `last_review_date` | MNAR | Keep as `NaT` — new employees have no review yet |
| `phone` | MCAR | Keep as `NaN` — contact field, not analytical |
| `email` | MCAR | Keep as `NaN` — contact field, not analytical |
| `city` | MCAR | Keep as `NaN` — guessing adds no value |
| `bonus` | MNAR | Keep as `NaN` — added `has_bonus` flag |
| `training_hours` | MAR | Imputed — grouped median by `department` + `position` |
| `satisfaction_rating` | MAR | Imputed — grouped median by `position` + `is_active` |
| `performance_score` | MAR | Imputed — grouped median by `position` + `is_active` |
| `age` | MCAR (entry error) | Imputed — grouped median by `position` |
| `salary` | MCAR (entry error) | Imputed — grouped median by `position` + `department` |

### 6. Skewness Treatment
`np.log1p` applied to significantly skewed columns (|skewness| > 1).
New `_log` suffixed columns created — originals preserved.

- `np.log1p` chosen over `np.log` to safely handle zero values
- Use `_log` columns for modeling, originals for reporting

## Tools Used
- Python, Pandas, NumPy
- Google Colab

## Key Decisions
- Duplicate `employee_id` rows were resolved by keeping the record
  with the most non-null values — not by blindly dropping either row
- Outliers were never removed based on statistics alone —
  each value was evaluated against domain knowledge
- Missing data was never imputed without first classifying
  the missingness type (MCAR / MAR / MNAR)
- Negative and sentinel values (`-500`, `999999`, `0.01`) were treated
  as data errors, not statistical outliers
