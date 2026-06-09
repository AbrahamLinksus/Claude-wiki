# Data Science Overview

**Summary**: A complete roadmap and topic map for the SRM Data Science course (21CSS303T), covering the full pipeline from data acquisition through visualization using Python.

**Sources**: `Unit I- Data Science.pptx.pdf`, `Unit II - Data Science.pptx.pdf`, `Unit III - Data Science.pptx.pdf`, `Question Bank_Data Science.docx`

**Last updated**: 2026-05-11

#DS

---

## Roadmap

Work through topics in this order for progressive understanding:

1. [[data-science-process]] — What data science is, the 3 V's of Big Data, and the 6-step DS process
2. [[data-storage-hierarchy]] — Where data lives: Databases → Data Marts → DWH → Data Lakes; open data providers
3. [[data-preparation]] — Cleansing, transformation, combining; the core data prep cycle
4. [[missing-data-handling]] — Detecting and fixing missing values and outliers before modeling
5. [[numpy]] — Fast numerical arrays in Python; foundation for all numerical work
6. [[pandas]] — Series and DataFrames; the primary tool for structured data manipulation
7. [[data-wrangling]] — Combining and reshaping datasets; the full wrangling pipeline
8. [[data-transformation]] — Cleaning, binning, standardization, string ops, and summarizing
9. [[matplotlib]] — Building and customizing plots; the base visualization library
10. [[seaborn-visualization]] — Statistical visualizations + 3D plots built on Matplotlib

---

## Quick Reference

| Topic | Key Tool | Key Concept |
|---|---|---|
| Data Process | statsmodels | 6-step cycle |
| Storage | SQL / Spark | DWH vs Data Lake |
| Missing Data | pandas `fillna` | Imputation vs Deletion |
| NumPy | `np.ndarray` | 30-45× faster than lists |
| Pandas | `pd.DataFrame` | Label-aligned operations |
| Wrangling | `merge/concat/pivot` | Combine + reshape |
| Transformation | `pd.cut`, `StandardScaler` | Binning + normalization |
| Visualization | `matplotlib`, `seaborn` | 2D and 3D plots |
