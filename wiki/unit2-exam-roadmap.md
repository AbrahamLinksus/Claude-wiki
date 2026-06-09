# Unit 2 Exam Roadmap — Data Wrangling, Cleaning & Preparation

**Summary**: QB-mapped priority guide for Unit 2 of 21CSS303T. Every question ranked by exam weight with full code answers for Tier 1.

**Sources**: `Unit II - Data Science.pptx.pdf`, `Question Bank_Data Science.docx`

**Last updated**: 2026-05-12

#DS

---

## Coverage Map

| QB Question | In Slides? | Priority | Type |
|---|---|---|---|
| Key steps in the data wrangling process | Yes — 6-step cycle | **T1** | Short |
| Purpose of pivoting and reshaping | Yes — pivot/melt/stack/unstack | **T1** | Short |
| Common strategies for missing data | Yes — deletion + 6 imputation methods | **T1** | Short |
| Standardization vs normalization | Yes — both formulas with examples | **T1** | Short |
| String manipulation in data cleaning | Yes — full section with code | **T1** | Short |
| Binning, summarizing, classifying | Yes — pd.cut, qcut, centrality/dispersion | **T1** | Short |
| Data cleaning code (fillna, drop_duplicates, string ops) | Yes | **T1** | Long |
| Merge two datasets using merge() and concat() | Yes — all join types | **T1** | Long |
| Reshape using pivot(), melt(), stack() | Yes | **T1** | Long |
| Outlier detection using IQR and Z-score | Yes — both with full code | **T1** | Long |
| Classify column into bins and analyze frequency | Yes — pd.cut + value_counts | **T1** | Long |
| Why data transformation is critical before modeling | Yes | **T2** | Long |
| How improper outlier handling affects model accuracy | Yes | **T2** | Long |
| Compare methods for classifying and summarizing | Partial | **T2** | Long |
| Business days from a date (BusinessDay/MonthEnd) | In pandas.md (Unit 1 expansion) | **T2** | Long |
| Pivot table: manager-wise purchase count + mean | In pandas.md (Unit 1 expansion) | **T2** | Long |
| groupby: mean/min/max by customer_id | In pandas.md (Unit 1 expansion) | **T2** | Long |

---

## Critical Gaps

None — all Unit 2 QB questions are covered in the slides or prior pandas.md expansion. The business days and groupby questions appeared in the Unit 1 question bank expansion session; their answers live in [[pandas]].

---

## Priority Tier 1 — Must Know Cold

### 1. Data Wrangling: The 6-Step Process (Short)

The 6 steps of data wrangling:
1. **Discover** — understand the data structure and domain
2. **Structure** — reshape into the format needed for analysis
3. **Clean** — fix errors, handle missing values, remove duplicates
4. **Enrich** — combine with other datasets or derived features
5. **Validate** — confirm quality and correctness post-clean
6. **Publish** — make the clean data available for downstream use

Tools: OpenRefine (interactive cleaning), Tabula (PDF tables), Google DataPrep (cloud), Plotly (exploration)

---

### 2. Pivoting and Reshaping (Short)

| Operation | Purpose |
|---|---|
| `pivot()` | Wide format — rows → columns (one value per cell) |
| `melt()` | Long format — columns → rows (unpivot) |
| `stack()` | Move column level → row level (inner index) |
| `unstack()` | Move row level → column level |

Use pivoting when you need a cross-tab view; use melting when you need a flat, tidy dataset for modeling.

---

### 3. Missing Data Strategies (Short)

**Deletion:**
- Listwise (complete-case) — drop any row with a NaN; safe only when missing data is random and < 5%
- Pairwise — use all available data per calculation; preserves more data

**Imputation:**
- Mean/median/mode — fast, no data loss; distorts distribution
- KNN — uses similar rows to estimate; slower but smarter
- Regression — predict missing value from other columns
- Multiple imputation — creates several filled datasets, averages results

Choose based on: % missing, data type (numerical vs categorical), pattern (MCAR/MAR/MNAR).

---

### 4. Standardization vs Normalization (Short)

| | Normalization (Min-Max) | Standardization (Z-score) |
|---|---|---|
| Formula | `(x - min) / (max - min)` | `(x - mean) / std_dev` |
| Output range | 0 to 1 | mean=0, std=1 (unbounded) |
| Use when | Algorithm needs values in [0,1] (Neural nets, KNN) | Algorithm assumes Gaussian; outliers present |
| Sensitive to outliers | Yes — min/max shift drastically | No — robust to outliers |

---

### 5. String Manipulation (Short + Examples)

```python
text = "  Hello, World!  "

# Case
text.upper()        # "  HELLO, WORLD!  "
text.lower()        # "  hello, world!  "
text.title()        # "  Hello, World!  "

# Strip whitespace
text.strip()        # "Hello, World!"
text.lstrip()       # "Hello, World!  "
text.rstrip()       # "  Hello, World!"

# Search and replace
text.find("World")         # 8
text.replace("World", "Python")

# Split and join
words = "apple,banana,cherry".split(",")   # ['apple','banana','cherry']
"-".join(words)                            # 'apple-banana-cherry'

# Property checks
"hello".isalpha()    # True
"123".isdigit()      # True
"abc123".isalnum()   # True

# Reverse words
" ".join("hello world".split()[::-1])     # "world hello"

# f-string formatting
name, age = "Jake", 21
f"Name: {name}, Age: {age}"
```

---

### 6. Binning, Summarizing, Classifying (Short)

**Binning** = discretizing continuous values into categories
- Equal-width: `pd.cut(df['Age'], bins=3)` — same range per bin
- Equal-frequency: `pd.qcut(df['Age'], q=4)` — same count per bin

**Summarizing:**
- Centrality: mean, median, mode
- Dispersion: std, variance, range, IQR
- Distribution: skewness, kurtosis, histogram

**Classifying** in cleaning context = grouping records into meaningful categories (e.g., age → Young/Middle/Old using cut)

---

### 7. Data Cleaning — Full Pipeline (Long)

```python
import pandas as pd

data = {
    'Name': ['Alice', None, 'Bob', 'Bob', '  Carol  '],
    'Age':  [25, None, 30, 30, 22],
    'City': ['Delhi', 'Mumbai', None, 'Mumbai', 'Chennai']
}
df = pd.DataFrame(data)

# 1. Inspect
print(df.isnull().sum())
print(df.duplicated().sum())

# 2. Handle missing values
df['Name'].fillna('Unknown', inplace=True)
df['Age'].fillna(df['Age'].mean(), inplace=True)
df['City'].fillna(df['City'].mode()[0], inplace=True)

# 3. Remove duplicates
df.drop_duplicates(inplace=True)

# 4. String operations
df['Name'] = df['Name'].str.strip()          # remove whitespace
df['Name'] = df['Name'].str.lower()          # lowercase
df['City'] = df['City'].str.replace(' ', '_')  # replace spaces

print(df)
```

---

### 8. Merge and Concat (Long)

```python
import pandas as pd

df1 = pd.DataFrame({'ID': [1,2,3], 'Name': ['Alice','Bob','Carol']})
df2 = pd.DataFrame({'ID': [2,3,4], 'Score': [85, 90, 78]})

# --- MERGE (SQL-style joins) ---
inner = pd.merge(df1, df2, on='ID', how='inner')   # only matching IDs
left  = pd.merge(df1, df2, on='ID', how='left')    # all of df1
right = pd.merge(df1, df2, on='ID', how='right')   # all of df2
outer = pd.merge(df1, df2, on='ID', how='outer')   # all rows both sides

# Merge on index
merged_idx = pd.merge(df1.set_index('ID'), df2.set_index('ID'),
                      left_index=True, right_index=True, how='outer')

# --- CONCAT (row/column stacking) ---
row_stack = pd.concat([df1, df1], ignore_index=True)   # row-wise (axis=0)
col_stack = pd.concat([df1, df2], axis=1)              # column-wise (axis=1)

# --- COMBINE_FIRST (fill NaN from another DF) ---
df_a = pd.DataFrame({'A': [1, None, 3]})
df_b = pd.DataFrame({'A': [10, 20, 30]})
combined = df_a.combine_first(df_b)   # df_a values kept; NaN filled from df_b

# --- JOIN (index-based merge shortcut) ---
joined = df1.set_index('ID').join(df2.set_index('ID'), how='left')
```

| Method | Key use | Axis |
|---|---|---|
| `merge()` | SQL join on column or index | horizontal |
| `concat()` | Stack DataFrames | row or column |
| `combine_first()` | Fill NaN from backup DF | horizontal |
| `join()` | Index-based merge shorthand | horizontal |

---

### 9. Pivot, Melt, Stack (Long)

```python
import pandas as pd

data = {'Date':  ['2024-01','2024-01','2024-02','2024-02'],
        'City':  ['Delhi','Mumbai','Delhi','Mumbai'],
        'Sales': [100, 200, 150, 250]}
df = pd.DataFrame(data)

# --- PIVOT: long → wide ---
pivot_df = df.pivot(index='Date', columns='City', values='Sales')
#         Delhi  Mumbai
# 2024-01  100    200
# 2024-02  150    250

# --- MELT: wide → long (undo pivot) ---
melted = pivot_df.reset_index().melt(id_vars='Date',
                                     var_name='City',
                                     value_name='Sales')

# --- STACK: columns → inner row index ---
stacked = pivot_df.stack()
# Date     City
# 2024-01  Delhi     100
#          Mumbai    200
# dtype: int64

# --- UNSTACK: inner row index → columns (inverse of stack) ---
unstacked = stacked.unstack()   # back to pivot_df shape
```

---

### 10. Outlier Detection — IQR and Z-score (Long)

```python
import pandas as pd
import numpy as np

data = [10, 12, 13, 12, 14, 300, 11, 13, 12, 15, 400, 13]
df = pd.DataFrame({'Value': data})

# === IQR METHOD ===
Q1 = df['Value'].quantile(0.25)
Q3 = df['Value'].quantile(0.75)
IQR = Q3 - Q1

lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

outliers_iqr = df[(df['Value'] < lower) | (df['Value'] > upper)]
clean_iqr    = df[(df['Value'] >= lower) & (df['Value'] <= upper)]

print("IQR bounds:", lower, upper)
print("Outliers (IQR):\n", outliers_iqr)

# === Z-SCORE METHOD ===
df['z_score'] = (df['Value'] - df['Value'].mean()) / df['Value'].std()

outliers_z = df[df['z_score'].abs() > 3]
clean_z    = df[df['z_score'].abs() <= 3]

print("\nOutliers (Z-score):\n", outliers_z)
```

| | IQR | Z-score |
|---|---|---|
| Based on | Quartiles (median family) | Mean + std (Gaussian) |
| Best for | Skewed data | Normally distributed data |
| Threshold | 1.5 × IQR beyond Q1/Q3 | \|z\| > 3 |
| Outlier type | Global outliers | Global outliers |

---

### 11. Binning a Column + Frequency Analysis (Long)

```python
import pandas as pd

df = pd.DataFrame({'Age': [5, 22, 35, 48, 60, 73, 19, 31, 55, 67, 82, 14]})

# Equal-width bins (pd.cut) — each bin covers same value range
df['Age_Group'] = pd.cut(df['Age'],
                         bins=[0, 18, 35, 60, 100],
                         labels=['Child', 'Young', 'Middle', 'Senior'])

# Equal-frequency bins (pd.qcut) — each bin has same number of records
df['Age_Quartile'] = pd.qcut(df['Age'], q=4,
                              labels=['Q1','Q2','Q3','Q4'])

# Frequency analysis
print(df['Age_Group'].value_counts())
print(df['Age_Group'].value_counts(normalize=True) * 100)  # percentages

# Cross-tab
print(pd.crosstab(df['Age_Group'], df['Age_Quartile']))
```

---

## Priority Tier 2 — Know Well

### Why Data Transformation is Critical Before Modeling

1. **Algorithm assumptions** — most ML algorithms assume features are on similar scales; unscaled features make distance-based algorithms (KNN, SVM) biased toward large-magnitude features.
2. **Convergence** — gradient descent converges faster when features are normalized.
3. **Outlier sensitivity** — raw data with extreme values skews model weights.
4. **Categorical encoding** — algorithms need numbers; text labels must be transformed.
5. **Interpretability** — standardized features allow comparing coefficient magnitudes.

### How Improper Outlier Handling Affects Accuracy

- **Keeping outliers** → skews mean, inflates variance, distorts regression coefficients, misleads distance-based algorithms.
- **Removing legitimate extremes** → loses rare but important signal (fraud, disease).
- **Clipping incorrectly** → creates artificial boundary effects at the clip value.
- **Rule**: always understand *why* a value is extreme before removing it.

### Compare Methods for Classifying and Summarizing

| Method | Type | Best for |
|---|---|---|
| `value_counts()` | Frequency table | Categorical distributions |
| `pd.cut()` | Equal-width binning | Even range partitioning |
| `pd.qcut()` | Equal-frequency binning | Equal-population partitioning |
| `groupby().agg()` | Group summaries | Multi-stat per group |
| `pivot_table()` | Cross-tabular summary | 2D aggregation |
| `describe()` | Statistical summary | Quick numeric profiling |

### Business Days Code

```python
import pandas as pd
from pandas.tseries.offsets import BusinessDay, BusinessMonthEnd

date = pd.Timestamp('2024-01-10')

print(date + 1 * BusinessDay())    # next business day
print(date + 2 * BusinessDay())    # 2 business days later
print(date + 3 * BusinessDay())    # 3 business days later
print(date + BusinessMonthEnd())   # next business month end
```

### Pivot Table — Manager-wise Purchase

```python
import pandas as pd

data = {
    'Order Number':    [70001,70009,70002,70004,70007,70005,70008,70010,70003,70012,70011,70013],
    'Purchase Amount': [150.50,270.65,65.26,110.50,948.50,2400.60,5760.00,1983.43,2480.40,250.45,75.29,3045.60],
    'Manager ID':      [5002,5005,5001,5003,5002,5001,5001,5006,5003,5002,5007,5001]
}
df = pd.DataFrame(data)

result = df.pivot_table(index='Manager ID',
                        values='Purchase Amount',
                        aggfunc=['count', 'mean'])
print(result)
```

### GroupBy — Mean/Min/Max by Customer

```python
import pandas as pd

data = {
    'Order Number':    [70001,70009,70002,70004,70007,70005,70008,70010,70003,70012,70011,70013],
    'Purchase Amount': [150.50,270.65,65.26,110.50,948.50,2400.60,5760.00,1983.43,2480.40,250.45,75.29,3045.60],
    'Customer ID':     [3005,3001,3002,3009,3005,3007,3002,3004,3009,3008,3003,3002]
}
df = pd.DataFrame(data)

result = df.groupby('Customer ID')['Purchase Amount'].agg(['mean','min','max'])
print(result)
```

---

## Priority Tier 3 — Short Recall Only

| Topic | Key fact |
|---|---|
| 6 transformation techniques | Smoothing, Attribute Construction, Generalization, Aggregation, Discretization, Normalization |
| 4 data generalization types | Attribute (drop detail), Hierarchical (city→state), Numeric (range→label), Text (keyword→concept) |
| Global vs Contextual outlier | Global = isolated extreme (easy). Contextual = normal globally but anomalous in context (e.g., 30°C in winter) |
| Data wrangling tools | OpenRefine, Tabula, Google DataPrep, Plotly |
| Data smoothing | Replace noisy values with local averages (bin means, regression, clustering) |
| Data aggregation | Combining detailed data into summary form (daily → monthly sales) |

---

## Quick Reference

```
Data Wrangling steps:  Discover → Structure → Clean → Enrich → Validate → Publish
Missing data options:  dropna() | fillna(mean) | fillna(median) | fillna(mode()[0]) | ffill() | bfill()
Merge types:           inner | left | right | outer
Reshape ops:           pivot (long→wide) | melt (wide→long) | stack (col→row) | unstack (row→col)
Outlier bounds:        IQR: Q1-1.5*IQR to Q3+1.5*IQR  |  Z-score: |z| > 3
Normalization:         (x - min) / (max - min)  →  [0,1]
Standardization:       (x - mean) / std          →  mean=0, std=1
Binning:               pd.cut (equal width)  |  pd.qcut (equal frequency)
```

---

**See also**: [[data-wrangling]] · [[data-transformation]] · [[missing-data-handling]] · [[pandas]] · [[unit2-revision]]
