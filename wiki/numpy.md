# NumPy

**Summary**: NumPy (Numerical Python) is the foundation of scientific computing in Python, providing fast N-dimensional arrays and a large library of mathematical functions.

**Pre-Req**: Basic Python (lists, loops, functions)

**Sources**: `Unit I- Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## What Is NumPy?

NumPy provides the `ndarray` — an N-dimensional, homogeneous array that is **~30–45× faster** than Python lists for numerical operations. Speed comes from:
- Contiguous memory layout
- Vectorized C/Fortran under the hood
- Avoids Python's per-element overhead

```python
import numpy as np
```

(source: Unit I- Data Science.pptx.pdf)

---

## Array Creation

### From Python structures
```python
np.array([1, 2, 3])                   # 1D from list
np.array([[1, 2], [3, 4]])            # 2D from nested list
np.array([1, 1, 1], ndmin=10)         # force minimum 10 dimensions
```

### Placeholder functions
| Function | Output |
|---|---|
| `np.zeros((2, 4))` | All zeros |
| `np.ones((3, 6))` | All ones |
| `np.full((2, 2), 3)` | Filled with constant 3 |
| `np.empty((2, 3))` | Uninitialized (garbage values) |
| `np.eye(4)` | 4×4 identity-like (configurable diagonal) |
| `np.identity(4)` | 4×4 true identity matrix |
| `np.arange(20)` | 0–19, like `range()` |
| `np.linspace(0, 5, 10)` | 10 equally spaced points from 0 to 5 |
| `np.random.rand(3, 3)` | Uniform random [0, 1) |

### Reshaping
```python
arr = np.arange(12)
arr.reshape(4, 3)         # 1D → 2D (4 rows × 3 cols)
arr.reshape(2, 3, 2)      # 1D → 3D
arr.reshape(-1)            # flatten to 1D
```
**Rule**: total elements must be equal before and after reshape. `ValueError` if not.

(source: Unit I- Data Science.pptx.pdf)

---

## `identity()` vs `eye()`

| | `np.identity(n)` | `np.eye(R, C, k)` |
|---|---|---|
| Shape | Always square (n×n) | Flexible (R rows, C cols) |
| Diagonal | Always main diagonal (k=0) | Configurable via `k` |
| Use case | True identity matrix | Custom diagonal position |

```python
np.eye(4)          # 4×4, main diagonal
np.eye(3, 2)       # 3×2 matrix
np.eye(3, 3, 1)    # diagonal shifted up 1
np.eye(3, 2, -1)   # diagonal shifted down 1
np.identity(4)     # 4×4 identity
```

(source: Unit I- Data Science.pptx.pdf)

---

## Array Attributes

```python
arr.ndim        # number of dimensions
arr.shape       # tuple: (rows, cols, ...)  e.g. (2,4)
arr.dtype       # data type
arr.itemsize    # bytes per element
arr.size        # total number of elements
```

---

## Indexing and Slicing

```python
arr[3]           # element at index 3
arr[2:5]         # slice from index 2 to 4
arr[::2]         # every other element

# 2D
arr[0, 1]        # row 0, col 1
arr[1, :]        # full row 1
arr[:, 2]        # full column 2
```

### Checking dimensions
```python
a = np.array(5)          # 0D  → a.ndim = 0
b = np.array([1,2])      # 1D  → b.ndim = 1
c = np.array([[1,2]])    # 2D  → c.ndim = 2
d = np.array([[[1]]])    # 3D  → d.ndim = 3
```

---

## Copying Arrays

Three methods — **behavior differs critically**:

| Method | Type | Change to original affects copy? |
|---|---|---|
| `copy_arr = org_arr` | Assignment (reference) | **Yes** — same object |
| `np.copy(arr)` | Deep copy | No |
| `np.empty_like(arr)` | Shape/dtype copy (uninit) | No |

```python
# Assignment — both point to same data
copy_arr = org_arr
org_arr[1, 2] = 13      # copy_arr also changes!

# True copy
safe_copy = np.copy(org_arr)
```

(source: Unit I- Data Science.pptx.pdf)

---

## Iterating Arrays

```python
# 1D — iterate elements
for x in arr: print(x)

# 2D — iterate rows
for row in arr: print(row)

# 2D — iterate elements (nested loop)
for row in arr:
    for val in row: print(val)

# 3D — iterate 2D slices
for x in arr: print(x)

# Any dimension — iterate all scalars with nditer
for x in np.nditer(arr): print(x)
```

`np.nditer()` is the clean way to iterate every scalar regardless of array dimensionality. (source: Unit I- Data Science.pptx.pdf)

---

## Flattening

```python
arr.reshape(-1)     # flatten to 1D (returns view if possible)
arr.flatten()       # always returns a copy
arr.ravel()         # returns view if possible

---

## Transpose

Swaps rows and columns of a 2D array (generalizes to N-D).

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])   # shape (2, 3)
arr.T                                      # shape (3, 2)
np.transpose(arr)                          # equivalent
```

(source: Question Bank_Data Science.docx)

---

## Sorting Arrays

### Basic Sort
```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
np.sort(arr)                   # sorted copy (ascending)
np.sort(arr)[::-1]             # descending
arr.sort()                     # in-place sort
```

### `np.argsort()` — Sort by Returning Indices
Returns the indices that would sort the array. Use when you need to reorder a paired array.

```python
heights = np.array([170, 155, 182, 161])
student_ids = np.array([101, 102, 103, 104])

sort_idx = np.argsort(heights)        # [1, 3, 0, 2] — ascending by height
print(student_ids[sort_idx])           # student IDs in order of increasing height
print(heights[sort_idx])               # sorted heights

# To get indices describing sort order by multiple columns, use lexsort
```

**Exam question**: "Construct a NumPy program to sort the student id with increasing height" (source: Question Bank_Data Science.docx)

### `np.lexsort()` — Sort by Multiple Keys

`lexsort` sorts by the *last* key first, then breaks ties using earlier keys. Pass keys in **reverse priority** order.

```python
# Sort students by last name, then first name (tiebreak)
first = np.array(['Alice', 'Bob', 'Alice', 'Charlie'])
last  = np.array(['Smith', 'Jones', 'Jones', 'Smith'])

# Last key = primary sort = last name; first key = tiebreak = first name
idx = np.lexsort((first, last))    # sort by last, tiebreak by first
print(idx)                          # [1, 2, 0, 3] → Bob Jones, Alice Jones, Alice Smith, Charlie Smith

# To get the sorted data
sorted_first = first[idx]
sorted_last  = last[idx]
```

**Exam question**: "Write a NumPy program to sort pairs of first name and last name and return their indices" (source: Question Bank_Data Science.docx)

```python
# Sort by multiple numeric columns
# E.g., sort by column 1 first, then column 0 as tiebreak
col0 = arr[:, 0]
col1 = arr[:, 1]
idx = np.lexsort((col0, col1))   # primary = col1, tiebreak = col0
sorted_arr = arr[idx]
print("Integer indices that describe sort order:", idx)
print("Sorted data:\n", sorted_arr)
```

**Exam question**: "Print the integer indices that describe the sort order by multiple columns and the sorted data" (source: Question Bank_Data Science.docx)

### Sort Complex Array (Real Part First, Then Imaginary)

```python
arr = np.array([3+4j, 1+2j, 3+1j, 2+5j])

# np.sort on complex sorts lexicographically — real first, then imaginary
sorted_arr = arr[np.lexsort((arr.imag, arr.real))]
print(sorted_arr)
# Output: [1+2j, 2+5j, 3+1j, 3+4j]
```

**Exam question**: "Construct a NumPy program to sort a given complex array using the real part first, then the imaginary part" (source: Question Bank_Data Science.docx)

---

## Array Comparison

### `np.array_equal()` — Exact Equality

```python
a = np.random.rand(3, 3)
b = a.copy()

np.array_equal(a, b)    # True — same shape and all values equal
np.array_equal(a, a*2)  # False

# Check if two randomly generated arrays are equal
x = np.random.randint(0, 10, size=(3, 3))
y = np.random.randint(0, 10, size=(3, 3))
print("Arrays are equal:", np.array_equal(x, y))
```

### `np.allclose()` — Approximate Equality (Floats)

```python
a = np.array([1.0, 2.0000001])
b = np.array([1.0, 2.0])
np.allclose(a, b)           # True — within default tolerance (rtol=1e-5, atol=1e-8)
np.array_equal(a, b)        # False — exact comparison fails
```

Use `allclose` when comparing floating-point results where rounding errors are expected.

**Exam question**: "Write a NumPy program to check two random arrays are equal or not" (source: Question Bank_Data Science.docx)

---

## NumPy vs Pandas

| Feature | NumPy | Pandas |
|---|---|---|
| Primary structure | `ndarray` (N-D array) | `Series` (1D), `DataFrame` (2D) |
| Data type | Homogeneous (one dtype) | Heterogeneous (mixed dtypes) |
| Index | Integer position only | Labelled index (string, datetime, etc.) |
| Missing values | Manual masking | Native NaN handling |
| Operations | Vectorized math | Data manipulation + SQL-like ops |
| File I/O | Limited | Rich (`read_csv`, `read_json`, `read_sql`) |
| Performance | Faster for pure math | Slightly slower, more features |
| Use case | Numerical computation, ML matrices | Tabular data, cleaning, analysis |

**Key insight**: Pandas DataFrames are built on NumPy arrays internally. NumPy is the engine; Pandas is the interface for structured data.

**Exam question**: "Compare the use of NumPy and Pandas in handling structured data" (source: Question Bank_Data Science.docx)
```
