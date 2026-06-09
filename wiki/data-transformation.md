# Data Transformation

**Summary**: Covers all data transformation techniques used to prepare data for modeling: cleaning steps, smoothing/generalization/normalization, string manipulation, statistical summarizing, binning, and standardization.

**Pre-Req**: [[data-preparation]], [[pandas]]

**Sources**: `Unit II - Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Data Cleaning Steps

Before transformation, clean the dataset: (source: Unit II - Data Science.pptx.pdf)

1. **Remove unwanted observations** — drop irrelevant rows, duplicates
2. **Fix structural errors** — typos, wrong data types, inconsistent categories
3. **Manage outliers** — cap, remove, or flag (see [[missing-data-handling]])
4. **Handle missing data** — delete or impute (see [[missing-data-handling]])

---

## Transformation Techniques

### Smoothing
Removes noise from data to reveal underlying patterns and trends. Highlights important features and helps predict patterns.

### Attribute Construction
Build new features from existing ones to simplify or enrich the dataset.
Example: derive `BMI` from `height` and `weight`.

### Generalization
Convert detailed data into a higher-level, abstract representation.

**4 types:**
| Type | Example |
|---|---|
| Attribute Generalization | Ages → age groups (20-30, 30-40) |
| Hierarchical Generalization | "Toyota Corolla" → "Toyota" → "Car" |
| Numeric Generalization | Exact salaries → salary ranges |
| Text Generalization | "John" → "Person" |

### Aggregation
Summarize multiple records into a single summary value (e.g., monthly sales from daily records).
```python
df.groupby('Month')['Sales'].sum()
```

### Discretization (Binning)
Convert continuous numeric data into discrete categories. See **Binning** section below.

### Normalization
Scale numeric features to a standard range. See **Standardization** section below.

(source: Unit II - Data Science.pptx.pdf)

---

## String Manipulation

Strings in Python are immutable sequences of characters.

```python
# Case manipulation
s.upper()         # ALL CAPS
s.lower()         # all lowercase
s.title()         # Title Case
s.capitalize()    # First letter cap
s.swapcase()      # flip case

# Search and replace
s.find('sub')     # returns index or -1
s.index('sub')    # like find but raises ValueError
s.replace('old', 'new')

# Split and join
s.split(',')      # split on delimiter
','.join(lst)     # join list with delimiter

# Whitespace
s.strip()         # remove both ends
s.lstrip()        # left only
s.rstrip()        # right only

# Formatting
f"Hello {name}"        # f-string
"Hello {}".format(x)   # format()

# Checking
s.isalpha()    # only letters
s.isdigit()    # only digits
s.isalnum()    # letters + digits
s.isspace()    # only whitespace
s.islower()    # all lowercase
s.isupper()    # all uppercase

# Reverse words
" ".join(text.split()[::-1])
```

(source: Unit II - Data Science.pptx.pdf)

---

## Summarizing Data

Reduces complexity by extracting patterns, trends, and key statistics.

### Centrality — where is the center?
- **Mean** — average; sensitive to outliers
- **Median** — middle value; robust to outliers
- **Mode** — most frequent value

### Dispersion — how spread out?
- **Standard Deviation** — average distance from the mean
- **Variance** — std dev squared
- **Range** — max − min

### Sample Distribution — what shape?
- **Histogram** — graphical distribution of a numerical variable
- **Tally** — simple counting frequency
- **Skewness** — asymmetry of the distribution (+ = right tail, − = left tail)
- **Kurtosis** — "tailedness" (high = heavy tails, peaked center)

(source: Unit II - Data Science.pptx.pdf)

---

## Binning (Discretization)

Binning = grouping continuous values into discrete intervals (bins). Used in preprocessing, feature engineering, and visualization to reduce complexity.

```python
pd.cut(df['Age'], bins=[0, 20, 40, 60, 100], labels=['Teen','Young','Middle','Senior'])
```

### Types of Binning

**Equal-Width Binning**
Divides range into equal-sized intervals.
- Example: Ages [22, 25, 30, 35, 40] → bins [20-30], [30-40]

**Equal-Frequency Binning (Quantile Binning)**
Each bin has approximately the same number of values.
- Example: 10 salaries split into 5 bins of 2 each

**Custom Binning**
Manually defined bin edges based on domain knowledge.
- Example: income → Low/Medium/High based on business rules

(source: Unit II - Data Science.pptx.pdf)

---

## Standardization

Scaling numerical data so features contribute equally to ML models.

**Why standardize?**
- Equalizes feature importance (prevents large-magnitude features dominating)
- Required by distance-based algorithms (KNN, SVM)
- Speeds gradient descent convergence

### Z-Score Normalization
Transforms to mean = 0, standard deviation = 1.

```
z = (x - mean) / standard_deviation
```

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
df_scaled = scaler.fit_transform(df)
```

### Min-Max Normalization
Scales to a specific range, typically [0, 1].

```
x_scaled = (x - min(x)) / (max(x) - min(x))
```

```python
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler()
df_scaled = scaler.fit_transform(df)
```

### When to Use Which
| Method | Use when |
|---|---|
| Z-Score | Algorithms assuming normal distribution (PCA, LDA, linear regression) |
| Min-Max | Need exact [0,1] range; no extreme outliers |

(source: Unit II - Data Science.pptx.pdf)

---

## Standardization vs Normalization — Distinguished

These terms are often confused. They are **two different things**:

| | Standardization (Z-score) | Normalization (Min-Max) |
|---|---|---|
| Formula | `z = (x - mean) / std` | `x' = (x - min) / (max - min)` |
| Output range | Unbounded (mean=0, std=1) | [0, 1] (or custom range) |
| Preserves outliers | Yes (outliers remain far from mean) | Compresses outliers into range |
| Sensitive to outliers | Less sensitive | Very sensitive (outlier shifts min/max) |
| Required by | PCA, SVM, K-means, logistic regression | Neural networks, KNN with Euclidean distance |

**Key distinction**: Standardization does NOT bound the output range; normalization does. Use standardization when the algorithm assumes zero-mean input; use normalization when a bounded scale is required.

**Exam question**: "Define and distinguish between standardization and normalization" (source: Question Bank_Data Science.docx)

---

## Binning with Frequency Analysis

After binning, analyze the frequency of each bin to understand distribution.

```python
import pandas as pd

ages = [22, 25, 30, 35, 40, 18, 28, 32, 55, 45, 19, 38]
df = pd.DataFrame({'Age': ages})

# Equal-width binning
df['Age_Group'] = pd.cut(df['Age'],
    bins=[0, 20, 30, 40, 60],
    labels=['Teen', 'Young', 'Middle', 'Senior']
)

# Frequency analysis
freq = df['Age_Group'].value_counts().sort_index()
print(freq)

# Relative frequency (proportion)
print(freq / len(df))

# Equal-frequency (quantile) binning
df['Age_Quartile'] = pd.qcut(df['Age'], q=4,
    labels=['Q1', 'Q2', 'Q3', 'Q4'])
print(df['Age_Quartile'].value_counts())

# Custom binning
df['Income_Level'] = pd.cut(df['Income'],
    bins=[0, 30000, 70000, float('inf')],
    labels=['Low', 'Medium', 'High']
)
```

**Exam question**: "Classify a numeric column into bins (e.g., age groups) and analyze frequency" (source: Question Bank_Data Science.docx)

---

## Why Data Transformation Is Critical Before Modeling

1. **Scale uniformity**: ML algorithms like KNN, SVM, gradient descent converge faster and perform better when features are on the same scale. Without normalization, a feature with range [0, 10000] dominates one with range [0, 1].

2. **Distribution shape**: Some algorithms (linear regression, PCA) assume approximately normal distributions. Transformations (log, Box-Cox) fix skewed distributions.

3. **Categorical encoding**: ML models require numeric inputs. Categorical variables must be encoded (one-hot, label encoding) before training.

4. **Outlier impact**: Untreated outliers bias model parameters (especially mean-based ones like linear regression).

5. **Missing values**: Models cannot process NaN — imputation or deletion is mandatory.

6. **Redundant features**: Correlated features add noise. Feature selection/construction improves model generalization.

**Exam question**: "Why is data transformation a critical step before modeling?" (source: Question Bank_Data Science.docx)

---

## Data Cleaning — Full Code Example

```python
import pandas as pd

df = pd.read_csv('data.csv')

# 1. Remove duplicate records
df.drop_duplicates(inplace=True)

# 2. Replace missing values
df['Age'].fillna(df['Age'].mean(), inplace=True)
df['City'].fillna('Unknown', inplace=True)

# 3. String operations — clean text columns
df['Name'] = df['Name'].str.strip()           # remove whitespace
df['Name'] = df['Name'].str.lower()           # to lowercase
df['City'] = df['City'].str.title()           # Title Case
df['Email'] = df['Email'].str.replace(' ', '') # remove spaces in email

# 4. Fix structural errors — standardize categories
df['Gender'] = df['Gender'].replace({
    'M': 'Male', 'm': 'Male', 'F': 'Female', 'f': 'Female'
})

# 5. Check result
print(df.info())
print(df.describe())
```

**Exam question**: "Perform data cleaning: replacing missing values, removing duplicate records, performing string operations" (source: Question Bank_Data Science.docx)
