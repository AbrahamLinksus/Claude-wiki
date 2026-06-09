# Data Science Process

**Summary**: Defines data science, explains the 3 V's of Big Data, and walks through the 6-step data science process from setting a research goal to automation.

**Sources**: `Unit I- Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## What Is Data Science?

Data Science is the discipline of extracting actionable insights from raw data using statistics, programming, and domain knowledge. It sits at the intersection of math/statistics, computing, and subject expertise.

**Big Data is characterized by 3 V's:**
- **Volume** — massive amounts of data (TB to PB scale)
- **Variety** — structured (tables), semi-structured (JSON/XML), unstructured (text/images)
- **Velocity** — data generated and consumed at high speed (streams, real-time feeds)

---

## The 6-Step Data Science Process

```
1. Setting the Research Goal
2. Retrieving Data
3. Data Preparation
4. Data Exploration
5. Data Modeling
6. Presentation & Automation
```

### Step 1 — Setting the Research Goal
Define what question you are answering. A clear goal determines what data you need and how you evaluate success.

### Step 2 — Retrieving Data
Collect data from internal stores ([[data-storage-hierarchy]]) or external sources (APIs, open data, web scraping). (source: Unit I- Data Science.pptx.pdf)

### Step 3 — Data Preparation
Clean, transform, and combine raw data so it is ready for analysis. See [[data-preparation]] and [[missing-data-handling]]. (source: Unit I- Data Science.pptx.pdf)

### Step 4 — Data Exploration
Summarize and visualize data to understand distributions, correlations, and anomalies before modeling. Tools: [[pandas]], [[matplotlib]], [[seaborn-visualization]]. (source: Unit I- Data Science.pptx.pdf)

### Step 5 — Data Modeling
Build statistical or machine learning models. Example: OLS linear regression using `statsmodels`.

```python
import statsmodels.api as sm
lm = sm.OLS(target, predictors)
result = lm.fit()
result.summary()   # R-squared, p-values, coefficients
```

Key outputs to interpret:
- **R-squared** — proportion of variance explained (0 to 1; higher is better)
- **p-value** — statistical significance of each predictor (< 0.05 = significant)
- **Coefficient** — effect size of each predictor on the target

(source: Unit I- Data Science.pptx.pdf)

### Step 6 — Presentation & Automation
Communicate findings via dashboards or reports; automate the pipeline to re-run on new data. (source: Unit I- Data Science.pptx.pdf)

---

## Facets of Data

Data can be characterized along several dimensions examined during exploration:
- **Centrality**: mean, median, mode — where the center of the data lies
- **Dispersion**: standard deviation, variance, range — how spread out the data is
- **Distribution shape**: skewness (asymmetry), kurtosis (tail weight)

See [[data-transformation]] for detailed summarizing techniques.
