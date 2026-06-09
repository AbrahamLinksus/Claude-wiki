# Unit 1 — Exam Roadmap (Question Bank vs Slides)

**Summary**: Cross-reference of Unit 1 question bank against slide topics, with priority tiers, coverage gaps, and all required code.

**Sources**: `Unit I- Data Science.pptx.pdf`, `Question Bank_Data Science.docx`

**Last updated**: 2026-05-11

#DS

---

## Coverage Map

| QB Question                                         | In Slides?      | Priority | Type |
| --------------------------------------------------- | --------------- | -------- | ---- |
| Define Data Science + key components                | ✅               | Medium   | SA   |
| NumPy arrays vs Python lists                        | ✅               | Medium   | SA   |
| Purpose of `eye()`, `zeros()`, `ones()`             | ✅               | High     | SA   |
| Data alignment in Pandas + why useful               | ✅               | Medium   | SA   |
| `reindex()` vs `drop()`                             | ✅               | Medium   | SA   |
| Series / DataFrames handle missing data             | ✅               | Medium   | SA   |
| 2D NumPy array: slicing + **transpose** + reshaping | ✅ / ⚠️          | **HIGH** | LA   |
| Read CSV → stats → missing → rank → sort            | ⚠️ partial      | **HIGH** | LA   |
| Select entries by multiple conditions               | ✅               | **HIGH** | LA   |
| Web API → DataFrame                                 | ❌ not in slides | **HIGH** | LA   |
| Web scraping (requests + BeautifulSoup)             | ❌ not in slides | **HIGH** | LA   |
| Compare NumPy vs Pandas                             | ✅               | Medium   | LA   |
| Open Data Sources vs Web APIs                       | ⚠️ partial      | Medium   | LA   |
| Index hierarchy + MultiIndex                        | ✅               | Medium   | LA   |
| Sort student IDs by increasing height               | ❌ not in slides | **HIGH** | LA   |
| Check two random arrays equal                       | ❌ not in slides | Medium   | LA   |
| Illustrate Data Science Process                     | ✅               | **HIGH** | LA   |
| Sort complex array (real first, then imaginary)     | ❌ not in slides | **HIGH** | LA   |
| Illustrate Facets of Data                           | ✅               | Medium   | LA   |
| Data Preparation + Exploration with examples        | ✅               | **HIGH** | LA   |
| Compare `eye()` vs `identity()`                     | ✅               | **HIGH** | LA   |
| Sort first/last name pairs — return indices         | ❌ not in slides | **HIGH** | LA   |

**Legend**: ✅ = fully in slides | ⚠️ = partially covered | ❌ = NOT in slides (extra content required)

---

## Critical Gaps — What Is NOT in the Slides

These 6 topics appear in the question bank but were **not taught in the Unit 1 slides**. They will almost certainly appear in exams.

1. `np.argsort()` — sort and return integer indices
2. `np.lexsort()` — sort by multiple keys (multi-column, name pairs)
3. Sort complex arrays by real part then imaginary part
4. `np.array_equal()` / `np.allclose()` — check equality
5. `requests` library + Web API → DataFrame
6. `BeautifulSoup` + Web scraping → DataFrame
7. `pd.read_csv()` + `df.describe()` + `df.info()` (CSV workflow — slides used statsmodels directly)

---

## Priority Tier 1 — Must Know Cold

### T1-A: NumPy Sorting (4 questions — most repeated topic)

**`np.argsort()` — sort and get indices**
```python
import numpy as np

# Sort student IDs by increasing height
student_ids = np.array([101, 102, 103, 104, 105])
heights     = np.array([170, 155, 182, 161, 175])

sort_idx = np.argsort(heights)           # indices that sort heights ascending
print("Sort indices:", sort_idx)          # [1 3 0 4 2]
print("Sorted IDs:  ", student_ids[sort_idx])  # [102 104 101 105 103]
print("Sorted heights:", heights[sort_idx])    # [155 161 170 175 182]
```

**`np.lexsort()` — sort by multiple keys (last key = primary)**
```python
# Sort by last name first, then first name as tiebreak
first = np.array(['Alice', 'Bob',   'Alice',   'Charlie'])
last  = np.array(['Smith', 'Jones', 'Jones',   'Smith'  ])

# lexsort: LAST key in tuple = PRIMARY sort key
idx = np.lexsort((first, last))      # sort by last; tiebreak by first
print("Sorted indices:", idx)        # [1, 2, 0, 3]
print("Sorted names:")
for i in idx:
    print(f"  {first[i]} {last[i]}")
# Bob Jones, Alice Jones, Alice Smith, Charlie Smith
```

**Multi-column sort — print indices + sorted data**
```python
data = np.array([[3, 2],
                 [1, 5],
                 [3, 1],
                 [2, 4]])

# Sort by column 1 (primary), then column 0 (tiebreak)
idx = np.lexsort((data[:, 0], data[:, 1]))
print("Integer indices describing sort order:", idx)
print("Sorted data:\n", data[idx])
```

**Sort complex array — real part first, then imaginary**
```python
arr = np.array([3+4j, 1+2j, 3+1j, 2+5j, 1+8j])

# lexsort: last key = primary = real part
idx = np.lexsort((arr.imag, arr.real))
sorted_arr = arr[idx]
print("Sorted complex array:", sorted_arr)
# [1+2j, 1+8j, 2+5j, 3+1j, 3+4j]
```

---

### T1-B: 2D NumPy Array Operations (Long Answer Q1)

Must demonstrate: **creation → slicing → transpose → reshape** in one script.

```python
import numpy as np

# Create 2D array
arr = np.array([[1, 2, 3, 4],
                [5, 6, 7, 8],
                [9, 10, 11, 12]])
print("Original array:\n", arr)
print("Shape:", arr.shape)         # (3, 4)
print("Dimensions:", arr.ndim)     # 2

# --- Slicing ---
print("\nFirst row:       ", arr[0])           # [1 2 3 4]
print("Last column:     ", arr[:, -1])         # [4 8 12]
print("Subarray [1:, 1:]:\n", arr[1:, 1:])    # 2×3 block bottom-right
print("Element [1,2]:   ", arr[1, 2])          # 7

# --- Transpose ---
transposed = arr.T
print("\nTransposed (4×3):\n", transposed)
print("Transposed shape:", transposed.shape)   # (4, 3)

# --- Reshape ---
reshaped_2d = arr.reshape(4, 3)
print("\nReshaped to (4,3):\n", reshaped_2d)

reshaped_3d = arr.reshape(2, 2, 3)
print("\nReshaped to (2,2,3):\n", reshaped_3d)

flat = arr.reshape(-1)
print("\nFlattened:", flat)
```

---

### T1-C: Pandas CSV Workflow (Long Answer Q2)

```python
import pandas as pd

# Step 1 — Read
df = pd.read_csv('students.csv')

# Step 2 — Explore
print(df.head())
print(df.info())         # dtypes, non-null counts
print(df.describe())     # count, mean, std, min, Q1, median, Q3, max

# Step 3 — Handle missing values
print(df.isnull().sum())
df['Score'].fillna(df['Score'].mean(), inplace=True)
df['Grade'].fillna(df['Grade'].mode()[0], inplace=True)
df.dropna(subset=['Name'], inplace=True)      # drop if Name is missing

# Step 4 — Rank
df['Rank'] = df['Score'].rank(ascending=False, method='min')

# Step 5 — Sort
df_sorted = df.sort_values(by='Score', ascending=False)
print(df_sorted[['Name', 'Score', 'Rank']].head(10))
```

---

### T1-D: Multiple Condition Selection (Long Answer Q3)

```python
import pandas as pd

data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'Age':  [28, 22, 35, 25, 32],
    'City': ['Delhi', 'Mumbai', 'Delhi', 'Chennai', 'Mumbai'],
    'Salary': [55000, 28000, 90000, 45000, 72000]
}
df = pd.DataFrame(data)

# AND — both conditions true
high_earners_delhi = df[(df['City'] == 'Delhi') & (df['Salary'] > 50000)]

# OR
young_or_rich = df[(df['Age'] < 25) | (df['Salary'] > 70000)]

# NOT
not_mumbai = df[~(df['City'] == 'Mumbai')]

# isin
metro_cities = df[df['City'].isin(['Delhi', 'Mumbai'])]

# between (inclusive)
mid_age = df[df['Age'].between(25, 32)]

# query string syntax
result = df.query("Age > 25 and Salary > 50000")

print("High earners in Delhi:\n", high_earners_delhi)
print("\nYoung or rich:\n", young_or_rich)
print("\nisin example:\n", metro_cities)
```

---

### T1-E: Web API → DataFrame (Long Answer Q4)

```python
import requests
import pandas as pd

# Example: COVID-19 data from a public API
url = "https://disease.sh/v3/covid-19/countries"
response = requests.get(url)

# Check success
if response.status_code == 200:
    data = response.json()       # list of dicts
    df = pd.DataFrame(data)

    # Select relevant columns
    df = df[['country', 'cases', 'deaths', 'recovered', 'active']]

    print(df.head())
    print(df.describe())

    # Save to CSV
    df.to_csv('covid_data.csv', index=False)
else:
    print("Request failed:", response.status_code)
```

---

### T1-F: Web Scraping → DataFrame (Long Answer Q5)

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

url = "https://example.com/data-table"
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# --- Method 1: pandas read_html (fastest) ---
tables = pd.read_html(url)        # finds all <table> tags
df = tables[0]                    # take first table
print(df.head())

# --- Method 2: manual BeautifulSoup ---
table = soup.find('table')
headers = [th.text.strip() for th in table.find_all('th')]

rows = []
for tr in table.find_all('tr')[1:]:    # skip header row
    cols = [td.text.strip() for td in tr.find_all('td')]
    if cols:
        rows.append(cols)

df = pd.DataFrame(rows, columns=headers)
print(df.head())
```

---

### T1-G: Data Science Process (Long Answer Q11)

Full answer structure — see [[data-science-process]].

**Key points to state:**
- DS = extracting insights from data using statistics + programming + domain knowledge
- **6 steps**: Research Goal → Retrieve Data → Prepare → Explore → Model → Present/Automate
- **Big Data 3 V's**: Volume (scale), Variety (formats), Velocity (speed)

**Model execution code (exam-expected):**
```python
import statsmodels.api as sm
import pandas as pd

df = pd.read_csv('data.csv')
target     = df['SalePrice']
predictors = df[['Area', 'Bedrooms', 'Age']]
predictors = sm.add_constant(predictors)    # add intercept term

model  = sm.OLS(target, predictors)
result = model.fit()
print(result.summary())
# Key outputs: R-squared, Adj. R-squared, p-values, coefficients
```

---

### T1-H: `eye()` vs `identity()` (SA Q3, LA Q15 — appears twice)

```python
import numpy as np

# identity() — always square, always main diagonal
a = np.identity(4)
print(a)
# [[1. 0. 0. 0.]
#  [0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]]

# eye() — flexible: non-square, diagonal offset
print(np.eye(4))          # 4×4 main diagonal (same as identity(4))
print(np.eye(3, 2))       # 3 rows, 2 cols
print(np.eye(3, 3, 1))    # 3×3, diagonal shifted UP by 1
print(np.eye(3, 2, -1))   # 3×2, diagonal shifted DOWN by 1
```

**Comparison table for answer:**

| | `np.identity(n)` | `np.eye(R, C, k)` |
|---|---|---|
| Shape | Always square (n×n) | Flexible (R×C) |
| Diagonal position | Fixed at main diagonal (k=0) | Configurable via `k` |
| Can be non-square | ❌ | ✅ |
| Use case | True identity matrix | Any diagonal 1s matrix |

---

## Priority Tier 2 — Know Well

### T2-A: Data Alignment in Pandas (SA Q4)

```python
import pandas as pd
import numpy as np

# Series alignment — operations align by index label, not position
s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20, 30], index=['b', 'c', 'd'])

print(s1 + s2)
# a    NaN   ← 'a' not in s2
# b    12.0
# c    23.0
# d    NaN   ← 'd' not in s1

# DataFrame alignment
df1 = pd.DataFrame({'A': [1,2,3], 'B': [4,5,6], 'C': [7,8,9]})
df2 = pd.DataFrame({'A': [10,11], 'B': [12,13], 'D': [14,15]})

df1_aligned, df2_aligned = df1.align(df2, fill_value=np.nan)
# Both DataFrames now have columns A, B, C, D — missing cells = NaN
```

**Why it's useful**: Prevents silent index-position misalignment bugs; ensures operations between two datasets match on meaning, not position.

---

### T2-B: `reindex()` vs `drop()` (SA Q5)

```python
import pandas as pd

df = pd.DataFrame({'Name': ['Alice', 'Bob', 'Charlie'],
                   'Age':  [25, 30, 35]}, index=['A', 'B', 'C'])

# reindex() — create a new index; missing labels → NaN; can ADD new labels
df_reindexed = df.reindex(['A', 'B', 'D', 'E'])
print(df_reindexed)
# A  Alice  25.0
# B  Bob    30.0
# D  NaN    NaN   ← new label, gets NaN
# E  NaN    NaN

# drop() — REMOVE specific rows or columns
df_no_age = df.drop('Age', axis='columns')      # remove column
df_no_B   = df.drop('B',   axis='index')        # remove row by label
```

**Key difference:**
- `reindex()` — *redefines* the index structure; can expand or reorder; NaN fills gaps
- `drop()` — *removes* specific labels; always shrinks the DataFrame

---

### T2-C: Series + DataFrame Missing Data Handling (SA Q6)

```python
import pandas as pd
import numpy as np

# --- Series NaN ---
s = pd.Series([1, np.nan, 3, np.nan, 5])
print(s.isnull())          # [False, True, False, True, False]
print(s.dropna())          # removes NaN rows
print(s.fillna(s.mean()))  # fill NaN with mean
print(s.fillna(method='ffill'))  # forward fill

# --- DataFrame NaN ---
df = pd.DataFrame({'A': [1, np.nan, 3],
                   'B': [np.nan, 2, 3],
                   'C': [1, 2, np.nan]})

print(df.isnull().sum())        # NaN count per column
df.dropna()                     # drop any row with NaN
df.dropna(axis=1)               # drop any column with NaN
df.dropna(thresh=2)             # keep rows with at least 2 non-NaN values
df.fillna(0)                    # fill all NaN with 0
df['A'].fillna(df['A'].mean(), inplace=True)  # column-specific fill
```

**Key insight**: Series alignment in operations produces NaN for unmatched indices — this is intentional (fail-safe), not a bug.

---

### T2-D: Array Equality Check (Long Answer Q10)

```python
import numpy as np

# Exact equality — all elements must match exactly
a = np.array([[1, 2], [3, 4]])
b = np.array([[1, 2], [3, 4]])
c = np.array([[1, 2], [3, 5]])

print(np.array_equal(a, b))   # True
print(np.array_equal(a, c))   # False

# With random arrays
x = np.random.randint(0, 5, size=(3, 3))
y = np.random.randint(0, 5, size=(3, 3))
print("Are x and y equal?", np.array_equal(x, y))

# Approximate equality (floating point)
p = np.array([1.0, 2.0000001, 3.0])
q = np.array([1.0, 2.0,       3.0])
print(np.allclose(p, q))          # True  (within tolerance)
print(np.array_equal(p, q))       # False (exact)
```

---

### T2-E: Index Hierarchy / MultiIndex (Long Answer Q8)

```python
import pandas as pd

# Creating MultiIndex from arrays
index = pd.MultiIndex.from_arrays(
    [['Science', 'Science', 'Arts', 'Arts'],
     ['2022', '2023', '2022', '2023']],
    names=['Department', 'Year']
)
df = pd.DataFrame({'Students': [120, 130, 90, 95],
                   'Pass_Rate': [0.85, 0.88, 0.78, 0.82]},
                  index=index)

print(df)
#                         Students  Pass_Rate
# Department Year
# Science    2022          120       0.85
#            2023          130       0.88
# Arts       2022           90       0.78
#            2023           95       0.82

# Accessing
print(df.loc['Science'])              # all Science rows
print(df.loc[('Science', '2022')])    # specific row
print(df.xs('2022', level='Year'))    # all departments for 2022

# Creating from existing columns
df_flat = pd.DataFrame({
    'Dept': ['Sci', 'Sci', 'Arts'], 'Year': [2022, 2023, 2022],
    'Count': [120, 130, 90]
})
df_multi = df_flat.set_index(['Dept', 'Year'])
print(df_multi)

# Flatten back
df_flat_again = df_multi.reset_index()
```

**Significance**: MultiIndex represents naturally nested/hierarchical data (department→year, city→month) in one DataFrame without data duplication. Enables efficient cross-sectional queries with `.xs()`.

---

### T2-F: Facets of Data + Data Preparation + Exploration (LA Q13, Q14)

**Facets of Data** — dimensions to examine during exploration:

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv('data.csv')

# Centrality
print("Mean:  ", df['Price'].mean())
print("Median:", df['Price'].median())
print("Mode:  ", df['Price'].mode()[0])

# Dispersion
print("Std Dev:", df['Price'].std())
print("Variance:", df['Price'].var())
print("Range:  ", df['Price'].max() - df['Price'].min())

# Distribution shape
print("Skewness:", df['Price'].skew())
print("Kurtosis:", df['Price'].kurt())

# Visualize
fig, axes = plt.subplots(1, 3, figsize=(15, 4))
sns.histplot(df['Price'], kde=True, ax=axes[0])
axes[0].set_title('Distribution (Histogram + KDE)')

sns.boxplot(y=df['Price'], ax=axes[1])
axes[1].set_title('Boxplot (Outliers + IQR)')

df['Category'].value_counts().plot(kind='bar', ax=axes[2])
axes[2].set_title('Category Frequency')

plt.tight_layout()
plt.show()
```

**Data Preparation pipeline (for exam answer):**
```python
# 1. Load
df = pd.read_csv('raw_data.csv')

# 2. Inspect
print(df.shape)
print(df.dtypes)
print(df.isnull().sum())
print(df.describe())

# 3. Clean
df.drop_duplicates(inplace=True)
df['Age'].fillna(df['Age'].median(), inplace=True)
df['City'] = df['City'].str.strip().str.title()

# 4. Outlier check
Q1 = df['Salary'].quantile(0.25)
Q3 = df['Salary'].quantile(0.75)
IQR = Q3 - Q1
df = df[~((df['Salary'] < Q1 - 1.5*IQR) | (df['Salary'] > Q3 + 1.5*IQR))]

# 5. Transform
df['Salary_Scaled'] = (df['Salary'] - df['Salary'].min()) / \
                       (df['Salary'].max() - df['Salary'].min())

# 6. Explore
print(df.describe())
sns.pairplot(df[['Age', 'Salary', 'Experience']])
plt.show()
```

---

## Priority Tier 3 — Short Answer Recall

### T3-A: NumPy Arrays vs Python Lists (SA Q2)

| Feature | Python List | NumPy Array |
|---|---|---|
| Speed | Slow (Python loop) | ~30–45× faster |
| Type | Heterogeneous (mixed) | Homogeneous (one dtype) |
| Memory | More (Python object overhead) | Less (compact C array) |
| Operations | Require loops | Vectorized (element-wise) |
| Dimensions | Nested lists (messy) | Native N-D arrays |
| Math ops | Manual | Built-in (`+`, `-`, `*`, `np.sqrt`) |

```python
import numpy as np, time

# Speed comparison
lst = list(range(1000000))
arr = np.arange(1000000)

t1 = time.time(); result = [x**2 for x in lst]; t2 = time.time()
t3 = time.time(); result = arr**2; t4 = time.time()

print(f"List: {t2-t1:.4f}s | NumPy: {t4-t3:.4f}s")
```

---

### T3-B: `zeros()`, `ones()`, `eye()` — Purpose (SA Q3)

```python
import numpy as np

# zeros() — initialize arrays before filling (avoids garbage values)
w = np.zeros((3, 4))          # weight matrix initialized to 0

# ones() — initialization, or as mask/multiplier
bias = np.ones((1, 5))        # bias vector

# eye() — identity matrix for linear algebra
I = np.eye(4)                 # 4×4 identity
# A @ I == A (multiplying by identity = no change)
# Also used for: transformations, diagonal matrices in ML

# full() — custom constant fill
grid = np.full((3, 3), 7)     # 3×3 matrix of 7s
```

**Exam answer**: `zeros()` initializes arrays before filling (safe default); `ones()` creates unit vectors or masks; `eye()` creates identity matrices essential in linear algebra operations (matrix inversion, transformations).

---

## Complete Quick Reference

```
TOPIC                           → FUNCTION/METHOD
─────────────────────────────────────────────────────
Sort + get indices              → np.argsort()
Sort by multiple keys           → np.lexsort((key2, key1))  ← last = primary
Sort complex (real→imag)        → np.lexsort((arr.imag, arr.real))
Check equality                  → np.array_equal() / np.allclose()
Transpose                       → arr.T  or  np.transpose(arr)
Shape                           → arr.shape
Reshape                         → arr.reshape(r, c) — total elements must match
Flatten                         → arr.reshape(-1) / arr.flatten()
Iterate all scalars             → for x in np.nditer(arr)

Read CSV                        → pd.read_csv('file.csv')
Explore stats                   → df.describe() / df.info()
Filter (AND/OR/NOT)             → df[(cond1) & (cond2)]
Rank                            → df['col'].rank(method='min')
Sort                            → df.sort_values(by='col', ascending=False)
Align two DFs                   → df1.align(df2, fill_value=np.nan)
Reindex                         → df.reindex(new_index)
Drop                            → df.drop('col', axis='columns')
MultiIndex access               → df.loc['key'] / df.xs(val, level='col')
Pivot table                     → pd.pivot_table(df, values=, index=, aggfunc=)
Group + aggregate               → df.groupby('col').agg(['mean','min','max'])
Web API                         → requests.get(url).json() → pd.DataFrame()
Web scraping                    → pd.read_html(url) / BeautifulSoup
```
