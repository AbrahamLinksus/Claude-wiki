# Data Wrangling

**Summary**: Data wrangling (data munging) is the process of transforming and mapping raw data into a usable format, including combining datasets with merge/concat/join and reshaping with pivot/melt/stack.

**Pre-Req**: [[pandas]], [[data-preparation]]

**Sources**: `Unit II - Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## What Is Data Wrangling?

Data wrangling = converting raw, messy data into a structured, clean form ready for analysis. Also called **data munging**. (source: Unit II - Data Science.pptx.pdf)

---

## The 6-Step Wrangling Cycle

```
1. Discovery    → understand the data
2. Organization → structure it logically
3. Cleaning     → fix errors and inconsistencies
4. Enrichment   → add/derive new features
5. Validation   → verify correctness
6. Publishing   → make it available for analysis
```

(source: Unit II - Data Science.pptx.pdf)

---

## Large Data Challenges & Libraries

When data doesn't fit in memory, standard pandas/numpy may be insufficient.

**Problems**:
- Memory limits — dataset > RAM
- I/O bottlenecks — reading massive files
- CPU starvation — single-threaded computation

**Python libraries for large data**:
| Library | Purpose |
|---|---|
| **Dask** | Parallel DataFrames / arrays, lazy evaluation |
| **Blaze** | Translates NumPy/Pandas expressions to various backends |
| **Numba** | JIT-compiles Python/NumPy to machine code |
| **Theano** | GPU-accelerated mathematical expressions |
| **Bcolz** | Compressed columnar storage |
| **Cython** | Compile Python-like code to C extensions |

---

## Combining Datasets

### `pd.merge()` — SQL-style Joins
Joins on a common column (like a foreign key).

```python
# Inner join — only matching rows
merged = pd.merge(df1, df2, on='ID', how='inner')

# Left outer join — all from df1, matching from df2
merged = pd.merge(df1, df2, on='ID', how='left')

# Right outer join
merged = pd.merge(df1, df2, on='ID', how='right')

# Full outer join — all rows from both
merged = pd.merge(df1, df2, on='ID', how='outer')

# Join on index
merged = pd.merge(df1, df2, left_index=True, right_index=True, how='outer')
```

### `pd.concat()` — Stacking
Stack DataFrames vertically (add rows) or horizontally (add columns).

```python
# Vertical stack (add rows)
result = pd.concat([df1, df2], ignore_index=True)

# Horizontal stack (add columns)
result = pd.concat([df1, df2], axis=1)
```

### `df.join()` — Index-based Merge
Simpler syntax for merging on index.

```python
joined = df1.join(df2, how='outer')
```

### `combine_first()` — Fill Missing Values
Element-wise: use df2's values wherever df1 has NaN.

```python
combined = df1.combine_first(df2)
```

### When to Use Which

| Scenario | Use |
|---|---|
| Stacking rows/columns | `concat` |
| SQL-style join on column | `merge` |
| Join on index (simpler) | `join` |
| Fill NaN from another DF | `combine_first` |

(source: Unit II - Data Science.pptx.pdf)

---

## Reshaping Datasets

### `pivot()` / `pivot_table()` — Long → Wide
Turns unique values in a column into new columns.

```python
pivot_df = df.pivot(index='Date', columns='City', values='Sales')
```

### `melt()` — Wide → Long
Opposite of pivot: collapses columns back into rows.

```python
melted = pivot_df.reset_index().melt(id_vars='Date', var_name='City', value_name='Sales')
```

### `stack()` — Columns → Row Index
Moves the innermost column level into the row index.

```python
stacked = pivot_df.stack()    # columns become row multi-index
```

### `unstack()` — Row Index → Columns
Reverses `stack()`.

```python
unstacked = stacked.unstack()
```

### Relationship Summary
```
Wide DataFrame
   ↓ stack()           ↑ unstack()
Multi-index Series
   ↓ (equivalent)      ↑
   pivot ↔ melt (Long/Wide conversion)
```

(source: Unit II - Data Science.pptx.pdf)
