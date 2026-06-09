# Missing Data Handling

**Summary**: Covers detection, deletion, and imputation of missing values, plus outlier types and detection methods using boxplots and IQR.

**Pre-Req**: [[data-preparation]], basic pandas

**Sources**: `Unit I- Data Science.pptx.pdf`, `Unit II - Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Detecting Missing Values

```python
df.isnull().sum()          # count NaN per column
df.isnull().any(axis=1)    # which rows have any NaN
```

(source: Unit I- Data Science.pptx.pdf)

---

## Handling Missing Values: Deletion

Remove records or columns containing missing data.

### Listwise Deletion (complete case analysis)
Drop entire rows with any missing value.
```python
df.dropna()           # drop rows with any NaN
df.dropna(axis=1)     # drop columns with any NaN
```
**Trade-off**: simple but loses potentially useful data.

### Pairwise Deletion
Use all available data for each calculation; different analyses use different subsets. Used in correlation matrices — don't drop the whole row, just exclude the pair when either value is missing.

(source: Unit I- Data Science.pptx.pdf)

---

## Handling Missing Values: Imputation

Replace missing values with estimated ones instead of removing.

### Mean / Median / Mode
```python
df['Age'].fillna(df['Age'].mean(), inplace=True)
df['Salary'].fillna(df['Salary'].median(), inplace=True)
df['City'].fillna(df['City'].mode()[0], inplace=True)
```

### Forward / Backward Fill
```python
df.fillna(method='ffill')   # use previous valid value
df.fillna(method='bfill')   # use next valid value
```

### Constant Fill
```python
df.fillna(0)
df.fillna('Unknown')
```

### KNN Imputation
Replaces missing value with the mean of the k nearest neighbors based on other features. More accurate but computationally expensive.

### Regression Imputation
Builds a regression model using non-missing features to predict the missing value.

### Multiple Imputation
Creates multiple complete datasets, analyzes each, then pools the results. Gold standard for rigorous statistical work.

(source: Unit I- Data Science.pptx.pdf)

---

## Outliers

### Definition
Outlier = a data point that deviates significantly from the rest. Also called: abnormality, discordant, deviant, anomaly. (source: Unit II - Data Science.pptx.pdf)

### Types of Outliers

**Global Outliers**
- Isolated data points far from the main body of data
- Easy to identify and remove
- Example: a person with age = 200

**Contextual Outliers**
- Unusual only within a specific context; normal in another
- Harder to identify; requires domain knowledge
- Example: 35°C temperature is normal in summer, outlier in winter

### Detecting Outliers — Boxplot (5-Number Summary)

```
Min  Q1  Median  Q3  Max
 |---|----|----|---|
      IQR = Q3 - Q1

Lower fence: Q1 - 1.5 × IQR
Upper fence: Q3 + 1.5 × IQR
Points outside the fences = outliers
```

```python
import seaborn as sns
sns.boxplot(data=df, showmeans=True, whis=1.5)
```

### Detecting Outliers — Z-Score
```
z = (x - mean) / std_dev
|z| > 3  →  potential outlier
```

### Anomalies vs Outliers
- **Outlier**: statistical definition — far from the norm
- **Anomaly**: semantically significant deviation — rare instance with features that differ from most data; an anomaly in one dataset may be normal in another (source: Unit II - Data Science.pptx.pdf)

---

## Outlier Detection — Full Code

### Method 1: IQR Method

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({'Score': [10, 12, 11, 200, 13, 9, 14, 11, 198, 12]})

Q1 = df['Score'].quantile(0.25)
Q3 = df['Score'].quantile(0.75)
IQR = Q3 - Q1

lower_fence = Q1 - 1.5 * IQR
upper_fence = Q3 + 1.5 * IQR

# Boolean mask of outliers
outliers = (df['Score'] < lower_fence) | (df['Score'] > upper_fence)
print("Outliers:\n", df[outliers])

# Remove outliers
df_clean = df[~outliers]
print("Clean data:\n", df_clean)
```

**How it works**: IQR = Q3 − Q1. Any value below `Q1 − 1.5×IQR` or above `Q3 + 1.5×IQR` is flagged as an outlier. The boxplot whiskers use this same rule.

### Method 2: Z-Score Method

```python
from scipy import stats
import pandas as pd
import numpy as np

df = pd.DataFrame({'Score': [10, 12, 11, 200, 13, 9, 14, 11, 198, 12]})

z_scores = np.abs(stats.zscore(df['Score']))

# Flag rows where |z| > 3
outliers = z_scores > 3
print("Outlier indices:", df.index[outliers].tolist())
print("Outlier values:\n", df[outliers])

# Without scipy
z_manual = (df['Score'] - df['Score'].mean()) / df['Score'].std()
outliers_manual = np.abs(z_manual) > 3
df_clean = df[~outliers_manual]
```

**How it works**: Z-score measures how many standard deviations a value is from the mean. |z| > 3 is the conventional threshold for outliers (~0.3% of data in a normal distribution).

### IQR vs Z-Score Comparison

| Feature | IQR Method | Z-Score Method |
|---|---|---|
| Assumption | None (non-parametric) | Assumes normal distribution |
| Sensitivity | Robust to extreme outliers | Affected by extreme values |
| Threshold | Q1/Q3 ± 1.5×IQR | \|z\| > 2 or > 3 |
| Best for | Skewed data, robust analysis | Normally distributed data |

**Exam question**: "Implement outlier detection using IQR and Z-score methods" (source: Question Bank_Data Science.docx)

---

## Impact of Improper Outlier Handling

- **Leaving outliers in**: distorts mean, inflates std dev, biases regression coefficients, degrades ML model accuracy
- **Removing too aggressively**: loses valid rare events (fraud detection, medical abnormalities); introduces selection bias
- **Best practice**: investigate before removing — determine if outlier is a data error or a genuine rare event

**Exam question**: "How does improper handling of outliers affect model accuracy?" (source: Question Bank_Data Science.docx)
