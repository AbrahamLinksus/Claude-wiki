 # Unit 1 — Revision Notes

**Summary**: Progressive revision page built during study sessions. Covers key functions and patterns from Unit 1 with context, code, and output.

**Pre-Req**: [[numpy]], [[pandas]]

**Last updated**: 2026-05-11

#DS

---

## NumPy — Sorting

### `np.argsort()` — Sort by one key, get indices back

When you have two related arrays (e.g. student IDs and their heights) and want to reorder one by the values of the other, `argsort` gives you the **integer indices** that would sort the target array. You then use those indices to reorder both arrays.

```python
import numpy as np

student_ids = np.array([101, 102, 103, 104, 105])
heights     = np.array([170, 155, 182, 161, 175])

sort_idx = np.argsort(heights)
print(sort_idx)               # [1, 3, 0, 4, 2]
print(student_ids[sort_idx])  # [102, 104, 101, 105, 103]
print(heights[sort_idx])      # [155, 161, 170, 175, 182]
```

---

### `np.lexsort()` — Sort by multiple keys

Used when you need a **primary sort + tiebreaker**. Works alphabetically for strings, numerically for numbers.

**Critical rule — the LAST key in the tuple is the PRIMARY sort:**

```python
# Sort names: primary = last name, tiebreak = first name
first = np.array(['Alice', 'Bob',     'Alice',   'Charlie'])
last  = np.array(['Smith', 'Jones',   'Jones',   'Smith'  ])

idx = np.lexsort((first, last))   # last = primary
for i in idx:
    print(f"{first[i]} {last[i]}")
# Bob Jones
# Alice Jones
# Alice Smith
# Charlie Smith
```

Same trick works for complex arrays — sort by real part first, imaginary as tiebreak:

```python
arr = np.array([3+4j, 1+2j, 3+1j, 2+5j, 1+8j])

idx = np.lexsort((arr.imag, arr.real))   # real = primary
print(arr[idx])
# [1+2j, 1+8j, 2+5j, 3+1j, 3+4j]
```

---

## NumPy — 2D Array Operations

### Transpose

Transpose flips rows and columns. A `(3, 4)` array becomes `(4, 3)`. Use `arr.T`.

```python
arr = np.array([[1,  2,  3,  4],
                [5,  6,  7,  8],
                [9, 10, 11, 12]])

t = arr.T
print(t.shape)   # (4, 3)
print(t)
# [[ 1,  5,  9],
#  [ 2,  6, 10],
#  [ 3,  7, 11],
#  [ 4,  8, 12]]
```

---

### Reshape

Changes the shape of an array without changing its data. Total elements must stay the same (e.g. 3×4 = 12 = 4×3 = 2×2×3).

Use `-1` to let NumPy calculate a dimension automatically.

```python
print(arr.reshape(4, 3))      # (4, 3)  — 12 elements
print(arr.reshape(2, 2, 3))   # (2, 2, 3) — 3D
print(arr.reshape(-1))        # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]  — flattened
```

---

## Pandas — Handling Missing Data + Rank

### `fillna()` with `mean()` — for numeric columns

When a numeric column has missing values, fill them with the column's average so the row isn't lost.

```python
import pandas as pd

df['Score'].fillna(df['Score'].mean(), inplace=True)
# mean() returns a single float — no [0] needed
```

---

### `fillna()` with `mode()` — for categorical columns

`mode()` returns a **Series** of the most frequent values (can be more than one if there's a tie). You need `[0]` to extract the actual value from that Series.

```python
df['Grade'].fillna(df['Grade'].mode()[0], inplace=True)
#                                     ↑
#             mode() returns a Series — [0] picks the first (most frequent) value
```

| Method | Returns | Needs `[0]`? |
|---|---|---|
| `df['col'].mean()` | single float | No |
| `df['col'].mode()` | Series (1+ values) | Yes |

---

### `rank()` — assign rank by value

Ranks rows by a column's value. `ascending=False` gives rank 1 to the highest value. `method='min'` gives tied entries the lower rank number.

```python
df['Rank'] = df['Score'].rank(ascending=False, method='min')
# Student with highest score → Rank 1
# If two students tie for 2nd, both get Rank 2 (not 2 and 3)
```

---

## Pandas — Multiple Condition Selection

### The operators

When filtering a DataFrame on more than one condition, use `&` (AND), `|` (OR), `~` (NOT) instead of Python's `and`/`or`/`not`. **Always wrap each condition in parentheses** — without them Python's operator precedence breaks the expression silently.

```python
# WRONG — & binds tighter than >, produces wrong result
df[df['Age'] > 25 & df['Salary'] > 50000]

# CORRECT
df[(df['Age'] > 25) & (df['Salary'] > 50000)]
```

---

### All filtering methods

```python
import pandas as pd

data = {
    'Name':   ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'Age':    [28, 22, 35, 25, 32],
    'City':   ['Delhi', 'Mumbai', 'Delhi', 'Chennai', 'Mumbai'],
    'Salary': [55000, 28000, 90000, 45000, 72000]
}
df = pd.DataFrame(data)

# AND — both must be true
df[(df['City'] == 'Delhi') & (df['Salary'] > 50000)]
# Alice, 28, Delhi, 55000

# OR — at least one must be true
df[(df['Age'] < 25) | (df['Salary'] > 70000)]
# Bob (age 22), Charlie (90000), Eve (72000)

# NOT — reverse a condition
df[~(df['City'] == 'Mumbai')]
# Alice, Charlie, Diana

# isin() — match against a list of values (cleaner than chained ORs)
df[df['City'].isin(['Delhi', 'Mumbai'])]
# Alice, Bob, Charlie, Eve

# between() — range check, inclusive on both ends
df[df['Age'].between(25, 32)]
# Alice (28), Diana (25), Eve (32)

# query() — string syntax, readable for complex filters
df.query("Age > 25 and Salary > 50000")
# Alice, Charlie, Eve
```

---

### When to use which

| Method | Best for |
|---|---|
| `&` / `\|` / `~` | Standard filtering — most common in exams |
| `isin()` | Matching one column against multiple values |
| `between()` | Range checks — cleaner than two `&` conditions |
| `query()` | Complex multi-condition filters, readable syntax |

---

## Pandas — Web API → DataFrame

### The pattern

An API returns JSON over HTTP. The flow is always: request → check status → parse JSON → DataFrame.

```python
import requests
import pandas as pd

url = "https://disease.sh/v3/covid-19/countries"
response = requests.get(url)

if response.status_code == 200:       # 200 = success
    data = response.json()            # JSON → Python list of dicts
    df = pd.DataFrame(data)           # list of dicts → each dict = one row

    df = df[['country', 'cases', 'deaths', 'recovered', 'active']]
    print(df.head())
    df.to_csv('covid_data.csv', index=False)
else:
    print("Request failed:", response.status_code)
```

### What each step does

```
requests.get(url)      → sends HTTP GET to the server
response.status_code   → 200 = OK | 404 = not found | 500 = server error
response.json()        → parses JSON body into a Python list/dict
pd.DataFrame(data)     → list of dicts becomes a DataFrame (one dict = one row)
```

Always check `status_code` before calling `.json()` — if the request failed, `.json()` will crash or return an error, not data.

---

## Data Science Process

### What is Data Science

Data Science = extracting meaningful insights from data using **statistics + programming + domain knowledge**.

### Big Data — 3 V's

| V | Meaning | Example |
|---|---|---|
| **Volume** | Scale of data | Petabytes of transaction logs |
| **Variety** | Formats of data | CSV, images, JSON, video |
| **Velocity** | Speed of generation | Real-time stock prices |

### 6-Step Process

```
1. Research Goal     → define the question
2. Retrieve Data     → collect from databases, APIs, files
3. Prepare Data      → clean, handle missing, transform
4. Explore Data      → visualize, find patterns
5. Model             → train and evaluate ML/statistical model
6. Present/Automate  → report results, deploy pipeline
```

### OLS Model Code

OLS = Ordinary Least Squares — fits a linear regression model. `add_constant` adds the intercept term; without it the line is forced through the origin.

```python
import statsmodels.api as sm
import pandas as pd

df = pd.read_csv('data.csv')

target     = df['SalePrice']
predictors = df[['Area', 'Bedrooms', 'Age']]
predictors = sm.add_constant(predictors)   # adds intercept

model  = sm.OLS(target, predictors)
result = model.fit()
print(result.summary())
# Key outputs: R-squared, Adj. R-squared, p-values, coefficients
```

### Facets of Data

When exploring data, examine across these dimensions:
- **Centrality** — mean, median, mode
- **Dispersion** — std dev, variance, range
- **Distribution shape** — skewness, kurtosis, histogram

---

## NumPy — `eye()` vs `identity()`

### `np.identity(n)` — always square, always main diagonal

```python
import numpy as np

a = np.identity(4)
# [[1. 0. 0. 0.]
#  [0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]]
```

Only one argument — the size. Always n×n, 1s on main diagonal only.

---

### `np.eye(R, C, k)` — flexible

```python
np.eye(4)          # 4×4 main diagonal — same as identity(4)
np.eye(3, 2)       # 3 rows, 2 cols — non-square
# [[1. 0.]
#  [0. 1.]
#  [0. 0.]]

np.eye(3, 3, 1)    # diagonal shifted UP by 1
# [[0. 1. 0.]
#  [0. 0. 1.]
#  [0. 0. 0.]]

np.eye(3, 3, -1)   # diagonal shifted DOWN by 1
# [[0. 0. 0.]
#  [1. 0. 0.]
#  [0. 1. 0.]]
```

---

### Comparison table

| | `np.identity(n)` | `np.eye(R, C, k)` |
|---|---|---|
| Shape | Always square (n×n) | Flexible (R×C) |
| Diagonal position | Fixed — main diagonal only | Configurable via `k` |
| Non-square | ❌ | ✅ |
| Arguments | 1 (size) | Up to 3 (rows, cols, offset) |
| Use case | True identity matrix | Any diagonal-ones matrix |

**Key line**: `identity(n)` is a special case of `eye(n)` — `eye` is the more general function.

---

## Pandas — Data Alignment

When you operate on two Series or DataFrames, Pandas aligns by **index label**, not by position. Unmatched labels produce NaN — this is intentional, not a bug.

```python
import pandas as pd
import numpy as np

s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20, 30], index=['b', 'c', 'd'])

print(s1 + s2)
# a     NaN   ← 'a' not in s2
# b    12.0   ← matched by label
# c    23.0   ← matched by label
# d     NaN   ← 'd' not in s1
```

For DataFrames, use `.align()` to explicitly align before operating:

```python
df1 = pd.DataFrame({'A': [1,2,3], 'B': [4,5,6], 'C': [7,8,9]})
df2 = pd.DataFrame({'A': [10,11], 'B': [12,13], 'D': [14,15]})

df1_aligned, df2_aligned = df1.align(df2, fill_value=0)
# Both now have columns A, B, C, D — missing cells filled with 0
```

**Why it matters**: prevents silent bugs where two datasets with different ordering or row counts produce wrong results when added positionally.

---

## Pandas — `reindex()` vs `drop()`

These do opposite things. `reindex` restructures the index; `drop` removes from it.

### `reindex()` — redefine the index

Labels that exist carry over; new labels get NaN. Can also be used to reorder rows.

```python
import pandas as pd

df = pd.DataFrame({'Name': ['Alice', 'Bob', 'Charlie'],
                   'Age':  [25, 30, 35]}, index=['A', 'B', 'C'])

df.reindex(['A', 'B', 'D', 'E'])
#      Name   Age
# A   Alice  25.0
# B     Bob  30.0
# D     NaN   NaN   ← new label, no data
# E     NaN   NaN

df.reindex(['C', 'A', 'B'])     # reorder only — no NaN
```

### `drop()` — remove specific rows or columns

```python
df.drop('Age', axis='columns')        # remove a column
df.drop('B',   axis='index')          # remove a row by label
df.drop(['A', 'C'], axis='index')     # remove multiple rows
```

### Key difference

| | `reindex()` | `drop()` |
|---|---|---|
| Purpose | Redefine / reorder the index | Remove specific labels |
| Size effect | Can expand (adds NaN) or shrink | Always shrinks |
| Missing labels | Filled with NaN | Raises KeyError |

---

## Pandas — NaN Handling (Series + DataFrame)

### Series

```python
import pandas as pd
import numpy as np

s = pd.Series([1, np.nan, 3, np.nan, 5])

s.isnull()               # [False, True, False, True, False]
s.notnull()              # [True, False, True, False, True]

s.dropna()               # removes NaN → [1, 3, 5]
s.fillna(s.mean())       # fill with mean → [1, 3.0, 3, 3.0, 5]
s.fillna(method='ffill') # forward fill (see below)
s.fillna(method='bfill') # backward fill (see below)
```

### ffill vs bfill

`ffill` (forward fill) — looks **backward** to the last valid value and copies it forward into the gap.
`bfill` (backward fill) — looks **forward** to the next valid value and copies it backward into the gap.

```
Original:  [1, NaN, NaN, 4, 5]

ffill:     [1,  1,   1,  4, 5]
                ↑    ↑
           copied from the 1 behind them

bfill:     [1,  4,   4,  4, 5]
                ↑    ↑
           copied from the 4 ahead of them
```

Use `ffill` when past values are a reasonable estimate (e.g. last known price).
Use `bfill` when the next known value is a better estimate (e.g. filling survey gaps with the next response).

Real-world example — stock prices missing mid-week:

```
Index:  Mon   Tue   Wed   Thu   Fri
Price:  100   NaN   NaN   200   250

ffill:  100   100   100   200   250
        ← Monday's price fills forward until a new value appears

bfill:  100   200   200   200   250
              ← Thursday's price fills backward to cover the gap
```

### DataFrame

```python
df = pd.DataFrame({'A': [1, np.nan, 3],
                   'B': [np.nan, 2, 3],
                   'C': [1, 2, np.nan]})

df.isnull().sum()        # NaN count per column
# A    1  
# B    1
# C    1

df.dropna()              # drop rows with ANY NaN
df.dropna(axis=1)        # drop columns with ANY NaN
df.dropna(thresh=2)      # keep rows with at least 2 non-NaN values

df.fillna(0)             # fill all NaN with 0
df['A'].fillna(df['A'].mean(), inplace=True)  # column-specific
```

### `thresh` parameter

```
thresh=2 → keep row only if it has at least 2 non-NaN values

Row [1, NaN, NaN] → 1 non-NaN → dropped
Row [1, 2,   NaN] → 2 non-NaN → kept
Row [1, 2,   3  ] → 3 non-NaN → kept
```

---

## NumPy — Array Equality

### `np.array_equal()` — exact match

Returns `True` only if both arrays have the same shape and every element matches exactly.

```python
import numpy as np

a = np.array([[1, 2], [3, 4]])
b = np.array([[1, 2], [3, 4]])
c = np.array([[1, 2], [3, 5]])

np.array_equal(a, b)   # True
np.array_equal(a, c)   # False — element [1,1] differs
```

### `np.allclose()` — approximate match for floats

Floating point arithmetic often gives `2.0000000001` instead of `2.0`. `array_equal` would return `False` — `allclose` handles this with a tolerance.

```python
p = np.array([1.0, 2.0000001, 3.0])
q = np.array([1.0, 2.0,       3.0])

np.array_equal(p, q)   # False — not exactly equal
np.allclose(p, q)      # True  — within default tolerance
```

### With random arrays (exam question format)

```python
x = np.random.randint(0, 5, size=(3, 3))
y = np.random.randint(0, 5, size=(3, 3))

print("Are x and y equal?", np.array_equal(x, y))
```

### When to use which

| | `array_equal` | `allclose` |
|---|---|---|
| Use for | Integers, exact comparisons | Floats, computed results |
| Tolerance | None — exact only | Yes — `rtol=1e-5, atol=1e-8` by default |
