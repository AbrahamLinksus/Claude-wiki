# Pandas

**Summary**: Pandas is the primary Python library for structured data manipulation, providing Series (1D) and DataFrame (2D) data structures with powerful indexing, alignment, and aggregation operations.

**Pre-Req**: [[numpy]], basic Python

**Sources**: `Unit I- Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## What Is Pandas?

Pandas is an open-source data manipulation and analysis library. Its two core structures:
- **Series** — 1D labelled array (like a single column)
- **DataFrame** — 2D tabular structure (like a spreadsheet or SQL table)

```python
import pandas as pd
```

(source: Unit I- Data Science.pptx.pdf)

---

## Series

A 1D array-like object where every element has a label (index).

```python
# From list (auto integer index)
s = pd.Series([1, 2, 3, 4, 5])

# From dictionary (keys become index)
s = pd.Series({'a': 1, 'b': 2, 'c': 3})

# From list with custom index
s = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
```

### Indexing
```python
s[0]        # by integer position
s['b']      # by label
```

### Vectorized Operations
```python
s_a = pd.Series([1, 2, 3])
s_b = pd.Series([4, 5, 6])
s_a + s_b   # element-wise: [5, 7, 9]
```

### Alignment
When operating on two Series with **different indices**, Pandas aligns by label — unmatched labels produce `NaN`:
```python
s_a = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s_b = pd.Series([4, 5, 6], index=['b', 'c', 'd'])
s_a + s_b
# a    NaN
# b    6.0
# c    8.0
# d    NaN
```

(source: Unit I- Data Science.pptx.pdf)

---

## DataFrame

A 2D table with labeled rows (index) and columns.

### Creation
```python
# From dictionary
data = {'Name': ['John', 'Alice'], 'Age': [25, 30]}
df = pd.DataFrame(data)

# From list of lists
data = [['John', 25], ['Alice', 30]]
df = pd.DataFrame(data, columns=['Name', 'Age'])

# With custom row index
df = pd.DataFrame(data, index=['A', 'B', 'C'])
```

### Indexing
```python
df['Name']            # access column (returns Series)
df.loc[0]             # row by label
df.iloc[1]            # row by integer position
df.at[0, 'Name']      # single element by label
df[df['Age'] > 30]    # filter by condition
```

### Column Operations
```python
df['Salary'] = [50000, 60000, 70000]         # add column
df[df['Salary'] > 60000]                     # filter
df.sort_values(by='Age', ascending=False)    # sort
```

### NaN Handling
```python
df.dropna()           # drop rows with any NaN
df.dropna(axis=1)     # drop columns with any NaN
df.fillna(0)          # fill with constant
```

### Grouping and Aggregation
```python
df.groupby('City')['Age'].mean()    # avg age per city
df.groupby('City')['Salary'].agg(['mean', 'min', 'max'])
```

(source: Unit I- Data Science.pptx.pdf)

---

## Indexing Deep Dive

### Index Features
- **Immutability**: once created, an index cannot be modified
- **Alignment**: operations between Series/DataFrames align by index label, not position
- **Flexibility**: integer, datetime, string, multi-level index types

### Reindexing
Create a new DataFrame with a different index — missing labels get `NaN`:
```python
df_new = df.reindex(['A', 'B', 'D', 'E'])
# D, E → NaN NaN (didn't exist in original)
```

### Drop
```python
df.drop('Age', axis='columns')   # remove column
df.drop(0, axis='index')         # remove row by label
```

---

## Data Alignment

Pandas aligns by label, not position:
```python
df1_aligned, df2_aligned = df1.align(df2, fill_value=np.nan)
# Both DataFrames get all columns; missing cells → NaN
```

(source: Unit I- Data Science.pptx.pdf)

---

## Rank

Assigns numeric ranks based on sorted order. Ties handled by averaging positions by default.

```python
df['default_rank'] = df['col'].rank()              # average rank for ties
df['max_rank']     = df['col'].rank(method='max')  # highest position for ties
df['na_bottom']    = df['col'].rank(na_option='bottom')  # NaN → highest rank
df['pct_rank']     = df['col'].rank(pct=True)      # rank / total non-NaN
```

**How default rank works for ties**: Fox(4) and Deer(4) are at positions 2 and 3 → rank = (2+3)/2 = 2.5

| Animal | Number_legs | default_rank | max_rank | na_bottom | pct_rank |
|---|---|---|---|---|---|
| Fox | 4.0 | 2.5 | 3.0 | 2.5 | 0.625 |
| Kangaroo | 2.0 | 1.0 | 1.0 | 1.0 | 0.250 |
| Deer | 4.0 | 2.5 | 3.0 | 2.5 | 0.625 |
| Spider | 8.0 | 4.0 | 4.0 | 4.0 | 1.000 |
| Snake | NaN | NaN | NaN | 5.0 | NaN |

(source: Unit I- Data Science.pptx.pdf)

---

## Sort

```python
df.sort_values(by=['Country'])                          # ascending (default)
df.sort_values(by=['Population'], ascending=False)      # descending
df.sort_index()                                         # sort by row index
df.sort_values(by=['col'], inplace=True)                # in-place
```

(source: Unit I- Data Science.pptx.pdf)

---

## Reading and Exploring Data

```python
import pandas as pd

# Read from CSV
df = pd.read_csv('data.csv')
df = pd.read_csv('data.csv', index_col=0)           # first col as index
df = pd.read_csv('data.csv', parse_dates=['date'])  # parse date column

# Basic exploration
df.head(5)           # first 5 rows
df.tail(5)           # last 5 rows
df.shape             # (rows, cols)
df.columns           # column names
df.dtypes            # data type per column
df.info()            # non-null counts + dtypes summary

# Basic statistics
df.describe()        # count, mean, std, min, Q1, median, Q3, max (numeric cols)
df['col'].mean()
df['col'].median()
df['col'].std()
df['col'].value_counts()    # frequency table
```

**Full exam workflow — read CSV, clean, rank, sort:**
```python
df = pd.read_csv('students.csv')
print(df.head())
print(df.describe())

df['Score'].fillna(df['Score'].mean(), inplace=True)
df.dropna(subset=['Name'], inplace=True)

df['Rank'] = df['Score'].rank(ascending=False)
df_sorted = df.sort_values(by='Score', ascending=False)
print(df_sorted)
```

**Exam question**: "Write a script to read a CSV file using Pandas and: display basic statistics, handle missing values, rank and sort by a specific column" (source: Question Bank_Data Science.docx)

---

## Multiple Condition Selection

Use `&` (AND), `|` (OR), `~` (NOT) with **parentheses required** around each condition.

```python
# AND
result = df[(df['Age'] > 25) & (df['City'] == 'Chicago')]

# OR
result = df[(df['Salary'] < 30000) | (df['Salary'] > 80000)]

# NOT
result = df[~(df['Department'] == 'HR')]

# isin() — match against a list
result = df[df['Country'].isin(['India', 'China', 'USA'])]

# between() — range check (inclusive)
result = df[df['Age'].between(25, 35)]

# query() — SQL-like string syntax
result = df.query("Age > 25 and Salary > 50000")

# Multiple combined
result = df[
    (df['Age'] > 25) &
    (df['Salary'] > 50000) &
    (df['City'].isin(['New York', 'Chicago']))
]
```

**Exam question**: "Use Pandas to select entries based on multiple conditions" (source: Question Bank_Data Science.docx)

---

## Data Acquisition

### Web API → DataFrame

```python
import requests
import pandas as pd

response = requests.get("https://api.example.com/data")
data = response.json()

df = pd.DataFrame(data)                  # if data is a list of records
df = pd.DataFrame(data['results'])       # if data is nested

print(df.head())
print(df.describe())
```

**Exam question**: "Demonstrate fetching data from a public Web API and load it into a DataFrame" (source: Question Bank_Data Science.docx)

### Web Scraping → DataFrame

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

url = "https://example.com/table-page"

# Method 1 — pandas direct HTML parse (easiest)
tables = pd.read_html(url)
df = tables[0]

# Method 2 — manual BeautifulSoup parse
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')
table = soup.find('table')
rows = table.find_all('tr')

data = []
for row in rows:
    cols = [td.text.strip() for td in row.find_all('td')]
    if cols:
        data.append(cols)

df = pd.DataFrame(data)
```

**Exam question**: "Implement basic web scraping using requests and BeautifulSoup to gather table data from a webpage" (source: Question Bank_Data Science.docx)

### Open Data Sources vs Web APIs

| Aspect | Open Data Sources | Web APIs |
|---|---|---|
| Access | Download files (CSV, JSON) | HTTP requests (GET/POST) |
| Format | Static snapshots | Dynamic, real-time |
| Rate limits | None | Often rate-limited |
| Auth | Usually none | Often requires API key |
| Examples | Data.gov, europa.eu | COVID API, Twitter API |
| Code | `pd.read_csv(url)` | `requests.get(url).json()` |

(source: Question Bank_Data Science.docx)

---

## MultiIndex (Index Hierarchy)

A hierarchical index with multiple levels — for multi-dimensional tabular data.

```python
# Create MultiIndex
arrays = [['A', 'A', 'B', 'B'], [2020, 2021, 2020, 2021]]
index = pd.MultiIndex.from_arrays(arrays, names=['Company', 'Year'])
df = pd.DataFrame({'Revenue': [100, 120, 200, 250]}, index=index)

# Output:
#                   Revenue
# Company Year
# A       2020      100
#         2021      120
# B       2020      200
#         2021      250

# Create from columns
df2 = df_flat.set_index(['Company', 'Year'])

# Access
df.loc['A']                        # all rows for company A
df.loc[('A', 2020)]               # specific row
df.xs(2020, level='Year')         # cross-section: all companies, year 2020

# Flatten back
df.reset_index()
```

**Significance**: Enables representing data with multiple grouping dimensions. Natural output of `groupby`, `stack`, and `pivot_table`.

**Exam question**: "Explain the concept of index hierarchy and its significance in multi-dimensional datasets" (source: Question Bank_Data Science.docx)

---

## Pivot Table

Aggregates data by grouping on rows/columns, equivalent to SQL GROUP BY or Excel PivotTables.

```python
pd.pivot_table(df,
    values='Purchase_Amount',
    index='Manager_ID',
    aggfunc={'Purchase_Amount': ['count', 'mean']}
)
```

**Exam example — manager-wise purchase count and mean:**
```python
data = {
    'Order_Number':    [70001, 70009, 70002, 70004, 70007, 70005, 70008],
    'Purchase_Amount': [150.50, 270.65, 65.26, 110.50, 948.50, 2400.60, 5760.00],
    'Customer_ID':     [3005, 3001, 3002, 3009, 3005, 3007, 3002],
    'Manager_ID':      [5002, 5005, 5001, 5003, 5002, 5001, 5001]
}
df = pd.DataFrame(data)

pivot = pd.pivot_table(df,
    values='Purchase_Amount',
    index='Manager_ID',
    aggfunc={'Purchase_Amount': ['count', 'mean']}
)
print(pivot)
```

**Exam question**: "Create a Pivot table — manager wise purchase count and mean value" (source: Question Bank_Data Science.docx)

---

## GroupBy with Multiple Aggregations

```python
# Mean, min, max of purchase amount grouped by customer
result = df.groupby('customer_id')['purch_amt'].agg(['mean', 'min', 'max'])
print(result)

# Multiple columns groupby
result = df.groupby(['City', 'Dept']).agg({
    'Salary': ['mean', 'min', 'max'],
    'Age': 'mean'
})

# Named aggregation (pandas >= 0.25)
result = df.groupby('City').agg(
    avg_salary=('Salary', 'mean'),
    max_age=('Age', 'max'),
    count=('Name', 'count')
)
```

**Exam question**: "Demonstrate a Pandas program to split a dataset, group by one column and get mean, min, and max values" (source: Question Bank_Data Science.docx)

---

## Business Dates

```python
import pandas as pd
from pandas.tseries.offsets import BusinessDay, BusinessMonthEnd

# Generate business day range
pd.bdate_range(start='2024-01-01', periods=5)

# Add N business days to a date
date = pd.Timestamp('2024-01-05')
print(date + 1 * BusinessDay())    # 1 business day forward
print(date + 2 * BusinessDay())    # 2 business days
print(date + 3 * BusinessDay())    # 3 business days

# Next business month end
print(date + BusinessMonthEnd(1))
```

**Exam question**: "Construct a Pandas program to calculate one, two, three business day(s) from a specified date. Also find the next business month end." (source: Question Bank_Data Science.docx)
