# Data Preparation

**Summary**: Covers the three core stages of data preparation — cleansing, transformation, and combining — that convert raw source data into analysis-ready form.

**Pre-Req**: [[data-science-process]], basic Python

**Sources**: `Unit I- Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Overview

Data preparation (Step 3 of the [[data-science-process]]) transforms raw, messy data into a clean, consistent dataset suitable for exploration and modeling. It has three phases:

```
Raw Data → [Cleansing] → [Transformation] → [Combining] → Analysis-ready Data
```

---

## 1. Data Cleansing

Removes errors and inconsistencies that would corrupt analysis.

**Common cleansing tasks:**
- Remove duplicate records
- Fix structural errors (typos, inconsistent capitalization, wrong data types)
- Handle missing values — see [[missing-data-handling]]
- Manage outliers — see [[missing-data-handling]]

(source: Unit I- Data Science.pptx.pdf)

---

## 2. Data Transformation

Converts data into a more useful form for modeling.

Key transformation techniques (detailed in [[data-transformation]]):
- **Smoothing** — remove noise from data
- **Attribute Construction** — create new features from existing ones
- **Generalization** — convert detailed data to higher-level categories
- **Aggregation** — summarize multiple records into one
- **Discretization / Binning** — convert continuous values into discrete categories
- **Normalization / Standardization** — scale values to a common range

---

## 3. Combining Data

Merging multiple datasets into one for analysis. Covered in detail in [[data-wrangling]].

Quick reference:

| Method | Use case |
|---|---|
| `pd.merge()` | SQL-like join on common column |
| `pd.concat()` | Stack rows or columns |
| `df.join()` | Index-based merge |
| `combine_first()` | Fill NaN from another DataFrame |

---

## Data Exploration Link

After preparation, Step 4 ([[data-science-process]]) involves exploring the clean data:
- Summary statistics: mean, median, std (see [[data-transformation]])
- Visualizations: distributions, correlations (see [[matplotlib]], [[seaborn-visualization]])
- Outlier detection via boxplots (see [[missing-data-handling]])
