# Unit 2 Revision — Data Wrangling, Cleaning & Preparation

**Summary**: Progressive study notes for Unit 2 of 21CSS303T. Full code blocks for every major topic.

**Sources**: `Unit II - Data Science.pptx.pdf`

**Last updated**: 2026-05-12

#DS

---

## 1. What Is Data Wrangling?

Data wrangling = transforming raw, messy data into a clean, analysis-ready format.

**Use cases**: fraud detection, customer behaviour analysis, recommendation systems.

**Tools**: OpenRefine (GUI cleaning), Tabula (PDF→table), Google DataPrep (cloud), Plotly (visual QA).

---

### Step 1 — Discover: audit the raw data before touching it

```python
import pandas as pd
import numpy as np

raw = pd.DataFrame({
    'StudentID': [101, 102, 103, 104, 105, 102],
    'Name':      ['Alice', 'Bob', None, 'Diana', '  Eve  ', 'Bob'],
    'Age':       [20.0, 22.0, None, 19.0, 21.0, 22.0],
    'Score':     [85.0, 90.0, 78.0, None, 92.0, 90.0],
    'Grade':     ['A', 'A', 'B', None, 'A', 'A']
})

print(raw)
#    StudentID     Name   Age  Score Grade
# 0        101    Alice  20.0   85.0     A
# 1        102      Bob  22.0   90.0     A
# 2        103     None   NaN   78.0     B
# 3        104    Diana  19.0    NaN  None
# 4        105  '  Eve  '  21.0   92.0     A
# 5        102      Bob  22.0   90.0     A   ← duplicate!

print(raw.shape)           # (6, 5)

print(raw.dtypes)
# StudentID      int64
# Name          object
# Age          float64
# Score        float64
# Grade         object

print(raw.isnull().sum())
# Name     1
# Age      1
# Score    1
# Grade    1

print(raw.duplicated().sum())  # 1

print(raw.describe())
#        StudentID    Age   Score
# count        6.0    5.0     5.0
# mean       102.8   20.8    87.0
# min        101.0   19.0    78.0
# max        105.0   22.0    92.0
```

What Discover tells you: 1 null in each column, 1 duplicate row, `Eve` has leading/trailing spaces.

---

### Step 2 — Structure: put data in the right shape

```python
df = raw.copy()
df.set_index('StudentID', inplace=True)

print(df)
#               Name   Age  Score Grade
# StudentID
# 101          Alice  20.0   85.0     A
# 102            Bob  22.0   90.0     A
# 103           None   NaN   78.0     B
# 104          Diana  19.0    NaN  None
# 105          Eve    21.0   92.0     A
# 102            Bob  22.0   90.0     A
```

`StudentID` is an identifier, not a measurement — it belongs as the row label, not a data column.

---

### Step 3 — Clean: fix everything found in Discover

```python
df.drop_duplicates(inplace=True)

df['Name'] = df['Name'].str.strip()
df['Name'].fillna('Unknown', inplace=True)

df['Age'].fillna(df['Age'].mean(), inplace=True)       # mean = 20.5
df['Score'].fillna(df['Score'].median(), inplace=True)  # median = 87.5
df['Grade'].fillna(df['Grade'].mode()[0], inplace=True) # mode()[0] = 'A'

print(df)
#               Name   Age  Score Grade
# StudentID
# 101          Alice  20.0   85.0     A
# 102            Bob  22.0   90.0     A
# 103        Unknown  20.5   78.0     B   ← Name filled, Age filled
# 104          Diana  19.0   87.5     A   ← Score filled, Grade filled
# 105            Eve  21.0   92.0     A   ← spaces stripped

print(df.isnull().sum().sum())  # 0
```

Use mean for numeric columns without outliers. Use median when outliers are present (a single 0 score would drag the mean down). Use `mode()[0]` for categoricals — `mode()` returns a Series, `[0]` extracts the value.

---

### Step 4 — Enrich: derive new columns and join external data

```python
df['Pass_Fail']  = df['Score'].apply(lambda x: 'Pass' if x >= 80 else 'Fail')
df['Score_Rank'] = df['Score'].rank(ascending=False).astype(int)

departments = pd.DataFrame({
    'StudentID':  [101, 102, 103, 104, 105],
    'Department': ['CS', 'IT', 'CS', 'IT', 'CS']
})
df = df.merge(departments.set_index('StudentID'),
              left_index=True, right_index=True, how='left')

print(df)
#               Name   Age  Score Grade Pass_Fail  Score_Rank Department
# StudentID
# 101          Alice  20.0   85.0     A      Pass           4         CS
# 102            Bob  22.0   90.0     A      Pass           2         IT
# 103        Unknown  20.5   78.0     B      Fail           5         CS
# 104          Diana  19.0   87.5     A      Pass           3         IT
# 105            Eve  21.0   92.0     A      Pass           1         CS
```

---

### Step 5 — Validate: confirm the cleaned data meets your rules

```python
print("Any nulls?",             df.isnull().sum().sum())            # 0
print("Score in range 0–100?",  df['Score'].between(0,100).all())   # True
print("Age realistic 15–30?",   df['Age'].between(15,30).all())     # True
print("Valid grades?",          df['Grade'].isin(['A','B','C','D','F']).all())  # True
print("Unique index?",          df.index.is_unique)                 # True
```

If any line returns `False`, go back to Step 3. Validate catches bugs your cleaning introduced.

---

### Step 6 — Publish: save for downstream use

```python
df.to_csv('clean_students.csv')
# df.to_parquet('clean_students.parquet')  # compressed, faster for big data
# df.to_sql('students', conn)              # push to a database

df.info()
# <class 'pandas.core.frame.DataFrame'>
# Index: 5 entries, 101 to 105
# Data columns (total 7 columns):
#  0   Name        5 non-null  object
#  1   Age         5 non-null  float64
#  2   Score       5 non-null  float64
#  3   Grade       5 non-null  object
#  4   Pass_Fail   5 non-null  object
#  5   Score_Rank  5 non-null  int64
#  6   Department  5 non-null  object
# 5 rows × 7 columns, 0 nulls
```

---

## 2. The Core Pipeline: Clean → Transform → Merge

```python
import pandas as pd

df1 = pd.DataFrame({
    'ID':   [1, 2, 3, 4],
    'Name': ['Alice', None, 'Bob', 'Carol'],
    'Age':  [25, 30, None, 22]
})
df2 = pd.DataFrame({
    'ID':    [1, 2, 3, 5],
    'Score': [85, 90, 78, 92]
})

# --- CLEAN ---
df1['Name'].fillna('Unknown', inplace=True)
df1['Age'].fillna(df1['Age'].mean(), inplace=True)

# --- TRANSFORM ---
df1['Age_Category'] = df1['Age'].apply(lambda x: 'Young' if x < 30 else 'Old')
df1['Name'] = df1['Name'].str.upper()

# --- MERGE ---
result = pd.merge(df1, df2, on='ID', how='left')
print(result)
#    ID     Name        Age Age_Category  Score
# 0   1    ALICE  25.000000        Young   85.0
# 1   2  UNKNOWN  25.666667        Young   90.0  ← Name filled, Age filled with mean
# 2   3      BOB  25.000000        Young   78.0
# 3   4    CAROL  22.000000        Young    NaN  ← ID 4 not in df2, Score = NaN
```

Always clean before merging — NaN in a key column creates mismatched rows and NaN propagates silently into the result.

---

## 3. Combining Datasets

### `pd.merge()` — SQL-style joins on a column

```python
import pandas as pd

df1 = pd.DataFrame({'ID': [1,2,3],   'Name':  ['Alice','Bob','Carol']})
df2 = pd.DataFrame({'ID': [2,3,4],   'Score': [85, 90, 78]})

inner = pd.merge(df1, df2, on='ID', how='inner')
print(inner)
#    ID   Name  Score
# 0   2    Bob     85
# 1   3  Carol     90
# only IDs that exist in BOTH tables

left = pd.merge(df1, df2, on='ID', how='left')
print(left)
#    ID   Name  Score
# 0   1  Alice    NaN   ← ID 1 not in df2
# 1   2    Bob   85.0
# 2   3  Carol   90.0

right = pd.merge(df1, df2, on='ID', how='right')
print(right)
#    ID   Name  Score
# 0   2    Bob     85
# 1   3  Carol     90
# 2   4    NaN     78   ← ID 4 not in df1

outer = pd.merge(df1, df2, on='ID', how='outer')
print(outer)
#    ID   Name  Score
# 0   1  Alice    NaN
# 1   2    Bob   85.0
# 2   3  Carol   90.0
# 3   4    NaN   78.0   ← all rows from both, NaN where no match
```

---

### `pd.merge()` on index instead of a column

```python
merged = pd.merge(df1.set_index('ID'), df2.set_index('ID'),
                  left_index=True, right_index=True, how='outer')
print(merged)
#     Name  Score
# ID
# 1   Alice    NaN
# 2     Bob   85.0
# 3   Carol   90.0
# 4     NaN   78.0
```

---

### `pd.concat()` — stack DataFrames row-wise or column-wise

```python
df_a = pd.DataFrame({'ID': [1,2], 'Name': ['Alice','Bob']})
df_b = pd.DataFrame({'ID': [3,4], 'Name': ['Carol','Diana']})

row_stack = pd.concat([df_a, df_b], ignore_index=True)
print(row_stack)
#    ID   Name
# 0   1  Alice
# 1   2    Bob
# 2   3  Carol
# 3   4  Diana

scores = pd.DataFrame({'Score': [85, 90]})
col_stack = pd.concat([df_a, scores], axis=1)
print(col_stack)
#    ID   Name  Score
# 0   1  Alice     85
# 1   2    Bob     90
```

---

### `combine_first()` — fill NaN from a backup DataFrame

```python
df_primary = pd.DataFrame({'Score': [85, None,  78]})
df_backup  = pd.DataFrame({'Score': [99,   90, 100]})

result = df_primary.combine_first(df_backup)
print(result)
#    Score
# 0   85.0   ← kept from primary
# 1   90.0   ← NaN in primary → filled from backup
# 2   78.0   ← kept from primary
```

Primary values are always kept. Only NaN slots are filled from the backup.

---

### `join()` — index-based merge shorthand

```python
joined = df1.set_index('ID').join(df2.set_index('ID'), how='left')
print(joined)
#       Name  Score
# ID
# 1    Alice    NaN
# 2      Bob   85.0
# 3    Carol   90.0
```

`join()` is just `merge()` with `left_index=True, right_index=True` — a shortcut when both DataFrames share the same index.

---

### When to use which

| Method | Key use |
|---|---|
| `merge()` | Join on a shared column (like SQL) |
| `concat()` | Stack more rows or more columns |
| `combine_first()` | Patch NaN gaps using a backup source |
| `join()` | Index-based merge shorthand |

---

## 4. Reshaping: Pivot, Melt, Stack, Unstack

### `pivot()` — long → wide

```python
import pandas as pd

df = pd.DataFrame({
    'Date':  ['Jan','Jan','Feb','Feb'],
    'City':  ['Delhi','Mumbai','Delhi','Mumbai'],
    'Sales': [100, 200, 150, 250]
})

wide = df.pivot(index='Date', columns='City', values='Sales')
print(wide)
# City   Delhi  Mumbai
# Date
# Feb      150     250
# Jan      100     200
```

Each unique `City` value becomes its own column. Rows are indexed by `Date`.

---

### `melt()` — wide → long (undo a pivot)

```python
long = wide.reset_index().melt(id_vars='Date',
                               var_name='City',
                               value_name='Sales')
print(long)
#   Date    City  Sales
# 0  Feb   Delhi    150
# 1  Jan   Delhi    100
# 2  Feb  Mumbai    250
# 3  Jan  Mumbai    200
```

`id_vars` = columns to keep as-is. Everything else gets melted into `var_name` + `value_name`.

---

### `stack()` — columns → inner row index

```python
stacked = wide.stack()
print(stacked)
# Date  City
# Feb   Delhi     150
#       Mumbai    250
# Jan   Delhi     100
#       Mumbai    200
# dtype: int64

print(type(stacked))   # <class 'pandas.core.series.Series'>
```

`stack()` folds the column level down into a new inner row index. Result is a Series with a MultiIndex.

---

### `unstack()` — inner row index → columns (inverse of stack)

```python
unstacked = stacked.unstack()
print(unstacked)
# City   Delhi  Mumbai
# Date
# Feb      150     250
# Jan      100     200
```

Exactly reverses `stack()` — back to the wide pivot shape.

---

### Mental model

```
pivot()   / unstack()  →  WIDER  (more columns, fewer rows)
melt()    / stack()    →  LONGER (more rows, fewer columns)
```

---

## 5. Missing Data Handling

### Detecting missing values

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'Name':  ['Alice', None, 'Bob', None, 'Carol'],
    'Age':   [25, np.nan, 30, np.nan, 22],
    'Score': [85, 90, np.nan, 78, 92]
})

print(df.isnull())
#     Name    Age  Score
# 0  False  False  False
# 1   True   True  False
# 2  False  False   True
# 3   True   True  False
# 4  False  False  False

print(df.isnull().sum())
# Name     2
# Age      2
# Score    1

print(df.isnull().sum().sum())  # 5  ← total nulls in the whole DataFrame
```

---

### Deletion strategies

```python
print(df.dropna())                 # listwise — drop any row with ANY null
#     Name   Age  Score
# 0  Alice  25.0   85.0
# 4  Carol  22.0   92.0

print(df.dropna(subset=['Name']))  # only drop if Name is null
#     Name   Age  Score
# 0  Alice  25.0   85.0
# 2    Bob  30.0    NaN
# 4  Carol  22.0   92.0

print(df.dropna(how='all'))        # drop only if EVERY column is null
# all 5 rows kept — none are fully null
```

---

### Imputation — numeric columns

```python
df['Age'].fillna(df['Age'].mean(), inplace=True)
# row 1 Age → 25.666  (mean of 25, 30, 22)
# row 3 Age → 25.666

df['Score'].fillna(df['Score'].median(), inplace=True)
# row 2 Score → 87.5  (median of 85, 90, 78, 92)
```

Use **mean** when no outliers. Use **median** when outliers are present — a single extreme value won't shift the median.

---

### Imputation — categorical columns

```python
df['Name'].fillna(df['Name'].mode()[0], inplace=True)
# rows 1 and 3 Name → 'Alice'  (most frequent)
```

`mode()` returns a **Series** — `[0]` picks the actual value out of it.

---

### Forward fill and backward fill

```python
s = pd.Series([1, np.nan, np.nan, 4, 5])

print(s.ffill())
# 0    1.0
# 1    1.0   ← copied from 1 behind
# 2    1.0   ← copied from 1 behind
# 3    4.0
# 4    5.0

print(s.bfill())
# 0    1.0
# 1    4.0   ← copied from 4 ahead
# 2    4.0   ← copied from 4 ahead
# 3    4.0
# 4    5.0
```

---

### Fill multiple columns at once

```python
df.fillna({'Age': df['Age'].mean(),
           'Score': df['Score'].median(),
           'Name': 'Unknown'}, inplace=True)
#       Name        Age  Score
# 0    Alice  25.000000   85.0
# 1  Unknown  25.666667   90.0
# 2      Bob  30.000000   87.5
# 3  Unknown  25.666667   78.0
# 4    Carol  22.000000   92.0
```

---

### Choosing the right strategy

| Situation | Strategy |
|---|---|
| Less than 5% missing, random pattern | `dropna()` |
| Numeric, no outliers | `fillna(mean)` |
| Numeric, outliers present | `fillna(median)` |
| Categorical | `fillna(mode()[0])` |
| Time-ordered data | `ffill()` or `bfill()` |
| Drop only if specific column is null | `dropna(subset=['col'])` |

---

## 6. Data Transformation Techniques

The 6 types — know the name, what it does, and one example each.

| Technique | What it does | Code hint |
|---|---|---|
| **Smoothing** | Replace noisy values with local estimates | `rolling(window=3).mean()` |
| **Attribute Construction** | Derive new columns from existing ones | `df['BMI'] = weight / height²` |
| **Generalization** | Replace detail with a higher-level concept | city → state via `.map()` |
| **Aggregation** | Summarise rows into totals/averages | `groupby('Month')['Sales'].sum()` |
| **Discretization** | Continuous → categorical bins | `pd.cut()`, `pd.qcut()` |
| **Normalization** | Rescale to a common range or distribution | Min-Max, Z-score |

---

### Smoothing

```python
import pandas as pd

df = pd.DataFrame({'Day': range(1,8),
                   'Sales': [200, 220, 180, 300, 190, 210, 250]})

df['Smoothed'] = df['Sales'].rolling(window=3).mean()
print(df)
#    Day  Sales    Smoothed
# 0    1    200         NaN   ← window not full yet
# 1    2    220         NaN
# 2    3    180  200.000000   ← avg(200, 220, 180)
# 3    4    300  233.333333
# 4    5    190  223.333333
# 5    6    210  233.333333
# 6    7    250  216.666667
```

---

### Attribute Construction

```python
df = pd.DataFrame({'Weight_kg': [70, 80, 60],
                   'Height_m':  [1.75, 1.80, 1.65],
                   'First': ['Alice','Bob','Carol'],
                   'Last':  ['Smith','Jones','Lee'],
                   'Price': [100, 200, 150],
                   'Quantity': [3, 2, 5]})

df['BMI']       = df['Weight_kg'] / (df['Height_m'] ** 2)
df['Full_Name'] = df['First'] + ' ' + df['Last']
df['Revenue']   = df['Price'] * df['Quantity']

print(df[['Full_Name', 'BMI', 'Revenue']])
#    Full_Name        BMI  Revenue
# 0  Alice Smith  22.857143      300
# 1    Bob Jones  24.691358      400
# 2    Carol Lee  22.038567      750
```

---

### Generalization

```python
df = pd.DataFrame({'City': ['Delhi','Mumbai','Chennai','Kolkata','Delhi']})

city_to_state = {'Delhi':'Delhi NCR', 'Mumbai':'Maharashtra',
                 'Chennai':'Tamil Nadu', 'Kolkata':'West Bengal'}
df['State'] = df['City'].map(city_to_state)

print(df)
#       City        State
# 0    Delhi    Delhi NCR
# 1   Mumbai  Maharashtra
# 2  Chennai   Tamil Nadu
# 3  Kolkata  West Bengal
# 4    Delhi    Delhi NCR
```

Generalization reduces cardinality — fewer unique values make grouping and modeling easier.

---

### Aggregation

```python
df = pd.DataFrame({'Date':  ['2024-01-01','2024-01-02','2024-01-03',
                              '2024-02-01','2024-02-02'],
                   'Sales': [100, 150, 130, 200, 180]})

df['Month'] = pd.to_datetime(df['Date']).dt.to_period('M')
monthly = df.groupby('Month')['Sales'].sum()
print(monthly)
# Month
# 2024-01    380
# 2024-02    380
# Freq: M, Name: Sales, dtype: int64
```

---

## 7. Binning (Discretization)

### `pd.cut()` — equal-width bins

```python
import pandas as pd

df = pd.DataFrame({'Age': [5, 14, 19, 25, 31, 42, 55, 63, 72, 85]})

df['Age_EW'] = pd.cut(df['Age'],
                       bins=[0, 18, 35, 60, 100],
                       labels=['Child','Young','Middle','Senior'])
print(df[['Age','Age_EW']])
#    Age  Age_EW
# 0    5   Child
# 1   14   Child
# 2   19   Young
# 3   25   Young
# 4   31   Young
# 5   42  Middle
# 6   55  Middle
# 7   63  Senior
# 8   72  Senior
# 9   85  Senior
```

---

### `pd.qcut()` — equal-frequency bins

```python
df['Age_EF'] = pd.qcut(df['Age'], q=4, labels=['Q1','Q2','Q3','Q4'])
print(df[['Age','Age_EF']])
#    Age Age_EF
# 0    5     Q1
# 1   14     Q1
# 2   19     Q1
# 3   25     Q2
# 4   31     Q2
# 5   42     Q3
# 6   55     Q3
# 7   63     Q4
# 8   72     Q4
# 9   85     Q4
```

---

### Frequency analysis after binning

```python
print(df['Age_EW'].value_counts())
# Age_EW
# Young     3
# Senior    3
# Child     2
# Middle    2

print(df['Age_EW'].value_counts(normalize=True).round(2))
# Age_EW
# Young     0.3
# Senior    0.3
# Child     0.2
# Middle    0.2
```

---

### `pd.cut` vs `pd.qcut`

| | `pd.cut` | `pd.qcut` |
|---|---|---|
| Controls | Value range per bin | Count per bin |
| Bins may have | Unequal counts | Unequal ranges |
| Use when | You have meaningful breakpoints (0–18, 18–60…) | You need balanced class sizes |

---

## 8. Normalization vs Standardization

### Min-Max Normalization → scales to [0, 1]

```python
import pandas as pd

df = pd.DataFrame({'Score': [50, 80, 60, 90, 70, 40, 100, 30]})

df['Normalized'] = (df['Score'] - df['Score'].min()) / \
                   (df['Score'].max() - df['Score'].min())
print(df[['Score','Normalized']])
#    Score  Normalized
# 0     50    0.285714
# 1     80    0.714286
# 2     60    0.428571
# 3     90    0.857143
# 4     70    0.571429
# 5     40    0.142857
# 6    100    1.000000   ← max → 1.0
# 7     30    0.000000   ← min → 0.0
```

---

### Z-score Standardization → mean=0, std=1

```python
df['Standardized'] = (df['Score'] - df['Score'].mean()) / df['Score'].std()
print(df[['Score','Standardized']])
#    Score  Standardized
# 0     50     -0.612372
# 1     80      0.612372
# 2     60     -0.204124
# 3     90      1.020621
# 4     70      0.204124
# 5     40     -1.020621
# 6    100      1.428869
# 7     30     -1.428869

print(f"mean: {df['Standardized'].mean():.2f}")  # mean: 0.00
print(f"std:  {df['Standardized'].std():.2f}")   # std:  1.00
```

---

### Comparison table

| | Normalization (Min-Max) | Standardization (Z-score) |
|---|---|---|
| Formula | `(x − min) / (max − min)` | `(x − mean) / std` |
| Output range | 0 to 1 | Unbounded (mean=0, std=1) |
| Sensitive to outliers | Yes — min/max shift drastically | No |
| Use for | Neural nets, KNN, distance-based | Gaussian-assumption algorithms, when outliers present |

---

## 9. String Manipulation

### Basic string methods

```python
text = "  Hello, Data Science World!  "

print(repr(text.strip()))   # 'Hello, Data Science World!'
print(repr(text.lstrip()))  # 'Hello, Data Science World!  '
print(repr(text.rstrip()))  # '  Hello, Data Science World!'
print(text.upper())         # '  HELLO, DATA SCIENCE WORLD!  '
print(text.lower())         # '  hello, data science world!  '
print(text.title())         # '  Hello, Data Science World!  '

print(text.strip().find("Data"))           # 7
print(text.replace("World", "Python"))     # '  Hello, Data Science Python!  '
print(text.strip().startswith("Hello"))    # True
```

---

### Split and join

```python
csv = "apple,banana,cherry"
fruits = csv.split(",")
print(fruits)           # ['apple', 'banana', 'cherry']
print("|".join(fruits)) # apple|banana|cherry

sentence = "one two three"
print(" ".join(sentence.split()[::-1]))  # three two one
```

---

### Property checks

```python
print("hello".isalpha())    # True
print("123".isdigit())      # True
print("hello123".isalnum()) # True
print("   ".isspace())      # True
print("hello123".isalpha()) # False  ← has digits
```

---

### Pandas vectorised string ops (on a Series)

```python
import pandas as pd

s = pd.Series(["  Alice ", "BOB  ", None, "carol"])

print(s.str.strip())
# 0    Alice
# 1      BOB
# 2     None
# 3    carol

print(s.str.lower())
# 0    alice
# 1      bob
# 2     None
# 3    carol

print(s.str.contains("ali", case=False, na=False))
# 0     True
# 1    False
# 2    False
# 3    False

print(s.str.replace("BOB", "Bob", regex=False))
# 0    Alice
# 1      Bob
# 2     None
# 3    carol

print(s.str.len())
# 0    8.0   ← includes spaces in original
# 1    5.0
# 2    NaN
# 3    5.0
```

---

### f-string formatting

```python
name, age, score = "Jake", 21, 95.5
print(f"Student: {name} | Age: {age} | Score: {score:.1f}")
# Student: Jake | Age: 21 | Score: 95.5
```

---

## 10. Summarizing Data

### Centrality

```python
import pandas as pd

df = pd.DataFrame({'Score': [70, 85, 90, 60, 78, 92, 55, 88, 74, 95]})

print(df['Score'].mean())    # 78.7
print(df['Score'].median())  # 81.5
print(df['Score'].mode()[0]) # 55  (every value unique → first value returned)
```

---

### Dispersion

```python
print(df['Score'].std())                        # 13.77
print(df['Score'].var())                        # 189.57
print(df['Score'].max() - df['Score'].min())    # 40  (range)

Q1 = df['Score'].quantile(0.25)   # 71.0
Q3 = df['Score'].quantile(0.75)   # 89.5
print(Q3 - Q1)                    # 18.5  (IQR)
```

---

### Full summary with describe()

```python
print(df['Score'].describe())
# count    10.000000
# mean     78.700000
# std      13.768321
# min      55.000000
# 25%      71.000000
# 50%      81.500000
# 75%      89.500000
# max      95.000000
```

---

### Skewness

```python
# Manual skewness check: compare mean vs median
# mean=78.7 > median=81.5 → not strongly skewed
# If mean > median → right-skewed (long right tail)
# If mean < median → left-skewed (long left tail)
# If mean ≈ median → symmetric
```

---

## 11. Outlier Detection

### IQR method

```python
import pandas as pd

data = [10,12,11,13,12,14,11,13,12,10,11,12,13,14,10,500,600,12,11,13]
df = pd.DataFrame({'Value': data})

Q1  = df['Value'].quantile(0.25)   # 11.0
Q3  = df['Value'].quantile(0.75)   # 13.0
IQR = Q3 - Q1                      # 2.0

lower = Q1 - 1.5 * IQR             # 8.0
upper = Q3 + 1.5 * IQR             # 16.0

outliers = df[(df['Value'] < lower) | (df['Value'] > upper)]
clean    = df[(df['Value'] >= lower) & (df['Value'] <= upper)]

print(f"IQR fences: {lower} — {upper}")  # IQR fences: 8.0 — 16.0
print(outliers)
#     Value
# 15    500
# 16    600
```

---

### Z-score method

```python
df['z'] = (df['Value'] - df['Value'].mean()) / df['Value'].std()

outliers_z = df[df['z'].abs() > 3]
print(outliers_z)
#     Value         z
# 16    600  3.210474   ← z > 3, confirmed outlier

# 500 has z ≈ 2.6 — flagged by IQR but not Z-score (threshold is 3)
```

---

### IQR vs Z-score

| | IQR | Z-score |
|---|---|---|
| Based on | Quartiles (median family) | Mean + std (Gaussian) |
| Best for | Skewed data | Normally distributed data |
| Threshold | 1.5 × IQR beyond Q1/Q3 | \|z\| > 3 |
| Catches more outliers | Yes (lower threshold) | No (stricter) |

---

### Global vs Contextual outliers

- **Global**: isolated extreme value anywhere in the dataset (e.g., 600 in a list of 10–14)
- **Contextual**: normal globally but anomalous given context (e.g., 30°C is fine in summer, outlier in winter)

---

## Quick Reference Card

```
Wrangling:   Discover → Structure → Clean → Enrich → Validate → Publish
Missing:     isnull() / fillna(mean/median/mode()[0]) / dropna()
Merge:       pd.merge(df1, df2, on='col', how='inner/left/right/outer')
Concat:      pd.concat([df1,df2], ignore_index=True)   # rows
             pd.concat([df1,df2], axis=1)               # columns
Pivot:       df.pivot(index=, columns=, values=)        # long → wide
Melt:        df.melt(id_vars=, var_name=, value_name=)  # wide → long
Stack:       df.stack()    # columns → row index
Unstack:     df.unstack()  # row index → columns
Binning:     pd.cut(s, bins=[...], labels=[...])        # equal-width
             pd.qcut(s, q=4, labels=[...])              # equal-frequency
Normalize:   (x - min) / (max - min)  →  [0, 1]
Standardize: (x - mean) / std         →  mean=0, std=1
Outlier IQR: lower = Q1 - 1.5*IQR  |  upper = Q3 + 1.5*IQR
Outlier Z:   |z| > 3
String:      .strip() .lower() .upper() .split() .join() .replace() .find()
```

---

**See also**: [[data-wrangling]] · [[data-transformation]] · [[missing-data-handling]] · [[pandas]] · [[unit2-exam-roadmap]]
