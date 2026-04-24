# Data Cleaning Study Materials

A collection of notebooks covering core data cleaning concepts with practical Python examples.

---

## 📁 Notebooks Overview

| Notebook | Topic |
|---|---|
| `handling_skewness.ipynb` | Skewness & Kurtosis |
| `handling_outliers.ipynb` | Outlier Detection & Treatment |
| `handling_missing_data.ipynb` | Missing Data (MCAR / MAR / MNAR) |
| `handling_duplicated.ipynb` | Duplicate Detection & Removal |

---

## 1. Skewness & Kurtosis

### What is Skewness?
Skewness measures the asymmetry of a distribution — which side has the longer tail?

| Value | Meaning |
|---|---|
| Skewness = 0 | Perfectly symmetric (normal distribution) |
| Skewness > 0 | Right-skewed — mean is greater than median |
| Skewness < 0 | Left-skewed — mean is less than median |

### What is Kurtosis?
Kurtosis measures the "tail heaviness" of a distribution — how prone is the data to producing outliers compared to a normal distribution?

| Type | Kurtosis | Meaning |
|---|---|---|
| Mesokurtic | ≈ 3 | Normal distribution (baseline) |
| Leptokurtic | > 3 | Heavy tails, more outliers |
| Platykurtic | < 3 | Light tails, fewer outliers |

### Solutions

**For Skewness:**
- **Log transform** → right skew (income, prices)
- **Square root** → moderate skew
- **Box-Cox / Yeo-Johnson** → flexible, automatically adjusts

**For Kurtosis:**
- High kurtosis → Robust scaling, Winsorizing, transformations
- Low kurtosis → Usually not a problem

---

## 2. Outliers

### What is an Outlier?
An observation that lies an abnormal distance from other values in the dataset. Think of a 50-story skyscraper suddenly built in a quiet neighborhood of single-family homes — it belongs to a fundamentally different distribution.

### Why Do They Occur?

| Cause | Example |
|---|---|
| Data entry errors | Typing 1000 instead of 100 |
| Measurement errors | Faulty sensor reading -200°C |
| Sampling errors | A corporate transaction in an individual consumer dataset |
| Genuine rare events | A billionaire in an income survey |

### Detection Methods

**Visual:**
- Box Plot — points beyond IQR whiskers
- Scatter Plot — multivariate outliers
- Histogram / KDE — long tails

**Statistical:**
- **Z-Score:** `z = (x - μ) / σ` → flag if |Z| > 3
- **IQR Method:** `Lower = Q1 - 1.5*IQR`, `Upper = Q3 + 1.5*IQR`
- **Isolation Forest, DBSCAN** — for complex, high-dimensional data

### Treatment Strategies

| Strategy | When to Use | How |
|---|---|---|
| **Remove** | Confirmed error, no correct value available | Filter rows outside robust bounds |
| **Cap / Winsorize** | Suspected error but need to keep the row | Clip to percentiles (e.g., 5th/95th) |
| **Transform** | Genuine skewed distribution (income, counts) | Apply log, sqrt, Box-Cox |
| **Separate Modeling** | Outliers are the signal (fraud, anomaly detection) | Main model + dedicated anomaly model |

---

## 3. Missing Data

### Step 0: Diagnose Before You Impute

Understanding *why* values are missing determines *how* to handle them.

| Type | Definition | Example |
|---|---|---|
| **MCAR** | Missing Completely At Random — unrelated to any variable | Random survey questions skipped |
| **MAR** | Missing At Random — depends on observed data, not on the missing value itself | Younger women more likely to skip age field |
| **MNAR** | Missing Not At Random — depends on the missing value itself | High earners tend not to report income |

### Why It Matters
- MCAR → Simple imputation works
- MAR → Conditional imputation needed
- MNAR → Hardest case; any imputation introduces bias

### Common Imputation Strategies

| Strategy | Best For |
|---|---|
| Mean / Median / Mode | MCAR, low missingness |
| Forward / Backward Fill | Time series data |
| KNN Imputation | MAR with correlated features |
| Model-based Imputation | MAR with complex relationships |
| Flag + Fill | MNAR — add a binary indicator column |

---

## 4. Duplicates

### Types of Duplicates
- **Exact duplicates** — all columns identical
- **Partial duplicates** — key columns identical (same product, different records)
- **Fuzzy duplicates** — near-identical (e.g., "iPhone 14" vs "iphone14")

### Detection & Removal

```python
# Count exact duplicates
df.duplicated().sum()

# Remove duplicates — keep first, last, or none
df.drop_duplicates(keep='first', inplace=True)

# Partial duplicates — keep most recent record by key column
df.sort_values('date').drop_duplicates(subset=['id'], keep='last')
```

---

## 🛠 Tools Used

- **Python** — Pandas, NumPy, Scipy, Scikit-learn
- **Visualization** — Matplotlib, Seaborn
- **Environment** — Google Colab

---

## 📌 General Principles

1. **Investigate before you act** — never blindly drop or fill
2. **Document every decision** — justify why you chose a specific method
3. **Preserve the original** — always keep the raw column before transforming
4. **Visualize before and after** — confirm the treatment worked
5. **Apply domain knowledge** — ask whether a value makes sense in context
