# Unit 3 Revision — Data Visualization

**Summary**: Progressive study notes for Unit 3 of 21CSS303T. Code snippets for every major visualization topic using Matplotlib and Seaborn.

**Pre-Req**: [[matplotlib]] · [[seaborn-visualization]]

**Sources**: `Unit III - Data Science.pptx.pdf`

**Last updated**: 2026-05-12

#DS

---

## 1. Static vs Interactive Visualization

| | Static | Interactive |
|---|---|---|
| Library | Matplotlib, Seaborn | Plotly, Bokeh, Altair |
| Output | Fixed image (PNG, PDF) | HTML with zoom/hover/filter |
| Use for | Reports, papers, exams | Dashboards, exploration |
| Performance | Fast, low memory | Heavier, browser-based |

Static is the exam default — Matplotlib/Seaborn. Interactive is Plotly.

---

## 2. Matplotlib — Core Components

Every Matplotlib figure has this hierarchy:

```
Figure  →  one canvas
  └── Axes   →  one plot (can have multiple per Figure)
        ├── x-axis, y-axis
        ├── Title
        ├── Labels
        └── Legend
```

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()          # creates Figure + one Axes
ax.plot([1,2,3], [4,5,6])         # draw on the Axes
ax.set_title('My Plot')
ax.set_xlabel('X')
ax.set_ylabel('Y')
plt.show()
```

---

## 3. Customized Bar Chart (Matplotlib)

The most common long-answer question for Unit 3.

```python
import matplotlib.pyplot as plt

categories = ['Electronics', 'Clothing', 'Food', 'Books']
sales      = [23000, 15000, 31000, 8000]
colors     = ['steelblue', 'coral', 'green', 'orange']

fig, ax = plt.subplots(figsize=(8, 5))

bars = ax.bar(categories, sales, color=colors, edgecolor='black', width=0.6)

# Labels on top of each bar
for bar, val in zip(bars, sales):
    ax.text(bar.get_x() + bar.get_width() / 2,
            bar.get_height() + 300,
            f'{val:,}',
            ha='center', va='bottom', fontsize=10)

ax.set_title('Sales by Category — Q1 2024', fontsize=14, fontweight='bold')
ax.set_xlabel('Category', fontsize=12)
ax.set_ylabel('Sales (₹)', fontsize=12)
ax.set_ylim(0, 35000)
ax.legend(['Sales'], loc='upper right')

plt.tight_layout()
plt.savefig('bar_chart.png', dpi=150)
plt.show()
# → Produces a bar chart with 4 coloured bars, value labels on top,
#   bold title, axis labels, legend, saved as PNG
```

---

## 4. Subplots — Multiple Plots in One Figure

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig, axes = plt.subplots(1, 2, figsize=(12, 4))   # 1 row, 2 columns

# Left plot
axes[0].plot(x, np.sin(x), color='blue', label='sin(x)')
axes[0].plot(x, np.cos(x), color='red',  label='cos(x)')
axes[0].set_title('Trig Functions')
axes[0].set_xlabel('x')
axes[0].set_ylabel('y')
axes[0].legend()

# Right plot
axes[1].bar(['A','B','C'], [10, 25, 15], color='coral')
axes[1].set_title('Bar Chart')
axes[1].set_xlabel('Category')
axes[1].set_ylabel('Count')

plt.suptitle('Dashboard — 2 Plots', fontsize=14)
plt.tight_layout()
plt.savefig('subplots.png', dpi=150)
plt.show()
# → Left: overlapping sine and cosine lines with legend
#   Right: 3-bar bar chart
#   Both share one figure with a common title
```

Use subplots when you need to compare multiple plots side by side in one view.

---

## 5. Annotations, Font Styles, Color Themes

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 5))

x = [1, 2, 3, 4, 5]
y = [10, 24, 18, 35, 28]

ax.plot(x, y, marker='o', color='steelblue', linewidth=2)

# Annotation — point to the maximum value
max_idx = y.index(max(y))
ax.annotate(f'Peak: {max(y)}',
            xy=(x[max_idx], max(y)),
            xytext=(x[max_idx] - 1, max(y) + 3),
            arrowprops=dict(arrowstyle='->', color='red'),
            fontsize=11, color='red')

# Font styling
ax.set_title('Sales Trend', fontsize=16, fontfamily='serif', fontweight='bold')
ax.set_xlabel('Month', fontsize=12, fontstyle='italic')
ax.set_ylabel('Sales', fontsize=12)

# Color theme (dark background)
plt.style.use('seaborn-v0_8-darkgrid')

plt.tight_layout()
plt.show()
# → Line plot with an arrow annotation pointing to peak value (35 at x=4)
```

---

## 6. savefig()

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot([1,2,3], [4,5,6])

# Save before show() — show() clears the figure
plt.savefig('output.png', dpi=150, bbox_inches='tight')
# dpi=150      → resolution (dots per inch); 72=screen, 300=print
# bbox_inches  → 'tight' removes excess whitespace around the figure

plt.savefig('output.pdf')   # vector format, scales without pixelation
plt.show()
```

Always call `savefig()` **before** `show()` — `show()` resets the figure.

---

## 7. Seaborn — Scatter Plot (day vs tip)

Classic exam question — "Prepare a scatterplot for day (x-axis) vs tip (y-axis)".

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic scatter: day vs tip
axes[0].scatter(tips['day'], tips['tip'], alpha=0.6, color='steelblue')
axes[0].set_xlabel('Day')
axes[0].set_ylabel('Tip ($)')
axes[0].set_title('Day vs Tip')

# Seaborn scatter with hue (relationship on different axis)
sns.scatterplot(data=tips, x='day', y='tip',
                hue='sex', style='time',
                ax=axes[1])
axes[1].set_title('Day vs Tip (by Sex & Time)')

plt.tight_layout()
plt.show()
# → Left:  plain scatter showing tip amounts for each day
# → Right: same scatter with colour by sex and marker by time of day
```

---

## 8. Histogram with KDE Overlay

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# Seaborn histogram + KDE
sns.histplot(data=tips, x='total_bill', kde=True,
             bins=20, color='steelblue', ax=axes[0])
axes[0].set_title('Total Bill Distribution (Histogram + KDE)')

# KDE only
sns.kdeplot(data=tips, x='total_bill', fill=True,
            color='coral', ax=axes[1])
axes[1].set_title('Total Bill — KDE Only')

plt.tight_layout()
plt.show()
# → Left:  bars showing frequency with a smooth KDE curve on top
# → Right: filled KDE curve showing the probability density
```

KDE (Kernel Density Estimate) is the smooth curve version of a histogram — shows the shape of the distribution without bin-size dependency.

---

## 9. Box Plot by Group

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Box plot: tip distribution by day
sns.boxplot(data=tips, x='day', y='tip',
            palette='Set2', ax=axes[0])
axes[0].set_title('Tip Distribution by Day')

# With individual points overlaid (stripplot)
sns.boxplot(data=tips, x='day', y='tip', palette='Set2', ax=axes[1])
sns.stripplot(data=tips, x='day', y='tip',
              color='black', alpha=0.3, jitter=True, ax=axes[1])
axes[1].set_title('Box Plot + Data Points')

plt.tight_layout()
plt.show()
# → Each box shows Q1, median, Q3, whiskers (1.5×IQR), and outlier dots
```

Box plot anatomy:
```
      ─── upper whisker (Q3 + 1.5×IQR)
 ┌───────┐
 │  75%  │  ← Q3
 ├───────┤
 │ median│  ← Q2 (50%)
 ├───────┤
 │  25%  │  ← Q1
 └───────┘
      ─── lower whisker (Q1 − 1.5×IQR)
      ●   ← outlier (beyond whisker)
```

---

## 10. Pair Plot

```python
import seaborn as sns
import matplotlib.pyplot as plt

iris = sns.load_dataset('iris')

sns.pairplot(iris, hue='species', diag_kind='kde',
             plot_kws={'alpha': 0.6})
plt.suptitle('Iris Dataset — Pair Plot', y=1.02)
plt.show()
# → Grid of scatter plots for every pair of numeric columns
#   Diagonal: KDE for each feature
#   Off-diagonal: scatter coloured by species
```

**Pair plot vs Scatter plot:**
| | Scatter plot | Pair plot |
|---|---|---|
| Shows | Relationship between 2 variables | All pairwise relationships at once |
| Number of plots | 1 | n × n (one per column pair) |
| Use when | Focused on one relationship | Exploring which features correlate |

---

## 11. Univariate Visualization — Numerical Data

Univariate = one variable at a time.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

data = np.random.normal(loc=50, scale=10, size=200)

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Histogram
axes[0].hist(data, bins=20, color='steelblue', edgecolor='white')
axes[0].set_title('Histogram')
axes[0].set_xlabel('Value')
axes[0].set_ylabel('Frequency')
# → Bars showing how often values fall in each range

# KDE (smooth distribution)
sns.kdeplot(data, fill=True, color='coral', ax=axes[1])
axes[1].set_title('KDE Plot')
# → Smooth bell-curve shape (normal distribution here)

# Box plot (single variable)
axes[2].boxplot(data, vert=True)
axes[2].set_title('Box Plot')
# → Box from Q1 to Q3, whiskers to 1.5×IQR, dots for outliers

plt.tight_layout()
plt.show()
```

| Chart | Best for |
|---|---|
| Histogram | Frequency distribution, bin-level detail |
| KDE | Smooth shape of distribution, comparing distributions |
| Box plot | Spotting outliers, comparing spread across groups |

---

## 12. Univariate Visualization — Categorical Data

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Count plot (bar chart of category frequencies)
sns.countplot(data=tips, x='day', palette='Set2', ax=axes[0])
axes[0].set_title('Count by Day')
# → Bars showing how many rows belong to each day

# Pie chart
day_counts = tips['day'].value_counts()
axes[1].pie(day_counts, labels=day_counts.index,
            autopct='%1.1f%%', startangle=90,
            colors=sns.color_palette('Set2'))
axes[1].set_title('Day Distribution (Pie)')
# → Pie chart showing proportion of visits per day

# Horizontal bar chart
axes[2].barh(day_counts.index, day_counts.values, color='coral')
axes[2].set_title('Day Count (Horizontal Bar)')
# → Easy to read when category labels are long

plt.tight_layout()
plt.show()
```

---

## 13. Compare Groups — Charts

"Demonstrate the different set of graphs used to compare values between groups."

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# 1. Grouped bar chart
sns.barplot(data=tips, x='day', y='tip', hue='sex',
            palette='Set1', ax=axes[0][0])
axes[0][0].set_title('Grouped Bar — Tip by Day & Sex')

# 2. Box plot by group
sns.boxplot(data=tips, x='day', y='tip', hue='time',
            palette='Set2', ax=axes[0][1])
axes[0][1].set_title('Box Plot — Tip by Day & Time')

# 3. Violin plot
sns.violinplot(data=tips, x='day', y='tip',
               palette='muted', ax=axes[0][2])
axes[0][2].set_title('Violin Plot — Tip Distribution by Day')

# 4. Strip plot (jittered points)
sns.stripplot(data=tips, x='day', y='tip',
              hue='sex', jitter=True, alpha=0.7, ax=axes[1][0])
axes[1][0].set_title('Strip Plot — Individual Points')

# 5. Swarm plot (non-overlapping points)
sns.swarmplot(data=tips, x='day', y='tip',
              hue='sex', ax=axes[1][1])
axes[1][1].set_title('Swarm Plot')

# 6. Line plot with confidence interval
sns.lineplot(data=tips, x='size', y='tip',
             hue='sex', ax=axes[1][2])
axes[1][2].set_title('Line Plot — Tip by Party Size')

plt.tight_layout()
plt.show()
```

| Chart | When to use |
|---|---|
| Grouped bar | Compare mean/sum across 2 categorical dimensions |
| Box plot | Compare spread and outliers across groups |
| Violin | Box plot + KDE — shows full distribution shape |
| Strip/Swarm | See individual data points per group |
| Line + CI | Trend over a continuous or ordinal variable |

---

## 14. Distribution Charts

"Show the different charts to visualise how data points' values are distributed."

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

data = np.concatenate([np.random.normal(40, 5, 150),
                       np.random.normal(70, 8, 100)])  # bimodal

fig, axes = plt.subplots(1, 4, figsize=(18, 4))

# 1. Histogram
axes[0].hist(data, bins=25, color='steelblue', edgecolor='white')
axes[0].set_title('Histogram')

# 2. KDE
sns.kdeplot(data, fill=True, color='coral', ax=axes[1])
axes[1].set_title('KDE Plot')

# 3. Histogram + KDE combined
sns.histplot(data, kde=True, bins=25, color='purple', ax=axes[2])
axes[2].set_title('Histogram + KDE')

# 4. ECDF (Empirical Cumulative Distribution)
sns.ecdfplot(data, color='green', ax=axes[3])
axes[3].set_title('ECDF')
axes[3].set_ylabel('Proportion ≤ x')

plt.tight_layout()
plt.show()
# → Histogram: frequency bars
# → KDE: smooth density curve, reveals bimodal shape clearly
# → Histogram+KDE: both overlaid
# → ECDF: S-curve showing what fraction of data is below each value
```

---

## 15. Multi-Plot Dashboard

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig = plt.figure(figsize=(14, 10))
fig.suptitle('Tips Dataset — Analysis Dashboard', fontsize=16, fontweight='bold')

# Plot 1: Distribution of total bill
ax1 = fig.add_subplot(2, 2, 1)
sns.histplot(tips['total_bill'], kde=True, color='steelblue', ax=ax1)
ax1.set_title('Total Bill Distribution')

# Plot 2: Tip by day (box)
ax2 = fig.add_subplot(2, 2, 2)
sns.boxplot(data=tips, x='day', y='tip', palette='Set2', ax=ax2)
ax2.set_title('Tip by Day')

# Plot 3: Total bill vs tip scatter
ax3 = fig.add_subplot(2, 2, 3)
sns.scatterplot(data=tips, x='total_bill', y='tip',
                hue='sex', alpha=0.7, ax=ax3)
ax3.set_title('Bill vs Tip')

# Plot 4: Count by day
ax4 = fig.add_subplot(2, 2, 4)
sns.countplot(data=tips, x='day', palette='Set1', ax=ax4)
ax4.set_title('Visit Count by Day')

plt.tight_layout()
plt.savefig('dashboard.png', dpi=150)
plt.show()
# → 2×2 grid: histogram, box plot, scatter, count plot
#   All using the same dataset, giving a complete picture at once
```

Multi-plot dashboards help decision-making by combining distribution, relationship, and comparison views in one frame.

---

## 16. 3D Surface Plot

```python
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

x = np.linspace(-5, 5, 60)
y = np.linspace(-5, 5, 60)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))   # ripple function

fig = plt.figure(figsize=(10, 6))
ax = fig.add_subplot(111, projection='3d')

surf = ax.plot_surface(X, Y, Z, cmap='viridis', edgecolor='none', alpha=0.9)
fig.colorbar(surf, ax=ax, shrink=0.5, label='Z value')

ax.set_title('3D Surface Plot — sin(√(x²+y²))', fontsize=13)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
ax.view_init(elev=30, azim=60)   # rotate the view angle

plt.tight_layout()
plt.savefig('3d_surface.png', dpi=150)
plt.show()
# → Coloured 3D surface resembling ripples on water
#   Color darkens toward 0 (trough), brightens at peaks
```

---

## 17. Visualization for Identifying Patterns and Anomalies

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

np.random.seed(42)
df = pd.DataFrame({
    'Date':  pd.date_range('2024-01', periods=50, freq='D'),
    'Sales': np.concatenate([
        np.random.normal(200, 20, 45),   # normal sales
        [500, 480, 510, 490, 520]         # anomalous spike at end
    ])
})

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Line plot — spot trend and spikes
axes[0].plot(df['Date'], df['Sales'], color='steelblue', linewidth=1.5)
axes[0].axhline(df['Sales'].mean(), color='red', linestyle='--', label='Mean')
axes[0].axhline(df['Sales'].mean() + 3*df['Sales'].std(),
                color='orange', linestyle='--', label='+3σ threshold')
axes[0].set_title('Sales Over Time — Anomaly Detection')
axes[0].legend()

# Box plot — outliers visible as dots beyond whiskers
axes[1].boxplot(df['Sales'], vert=True, patch_artist=True)
axes[1].set_title('Box Plot — Outlier Dots Visible')
axes[1].set_ylabel('Sales')

plt.tight_layout()
plt.show()
# → Left:  time series with mean line and +3σ threshold
#          spikes at end of series visually cross the orange line
# → Right: box plot shows dots for anomalous values above upper whisker
```

---

## 18. Seaborn vs Matplotlib

| | Matplotlib | Seaborn |
|---|---|---|
| Level | Low-level (full control) | High-level (built on Matplotlib) |
| Default style | Plain, requires manual styling | Attractive defaults |
| Statistical plots | Manual | Built-in (violin, pair plot, KDE, reg plot) |
| Works with | NumPy arrays, lists | Pandas DataFrames (native) |
| Code length | More verbose | Shorter |
| Customization | Maximum | Medium (can drop to Matplotlib) |
| Best for | Full control, custom layouts | Quick EDA, statistical visualization |

```python
# Same bar chart — Matplotlib vs Seaborn
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

df = pd.DataFrame({'Day': ['Mon','Tue','Wed'], 'Sales': [200, 350, 280]})

# Matplotlib — manual
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
axes[0].bar(df['Day'], df['Sales'], color='steelblue')
axes[0].set_title('Matplotlib')

# Seaborn — one line, prettier defaults
sns.barplot(data=df, x='Day', y='Sales', palette='Set2', ax=axes[1])
axes[1].set_title('Seaborn')

plt.tight_layout()
plt.show()
```

---

## Quick Reference Card

```
Figure/Axes:   fig, ax = plt.subplots(rows, cols, figsize=(w,h))
Save:          plt.savefig('file.png', dpi=150, bbox_inches='tight')
                         — always BEFORE plt.show()
Annotate:      ax.annotate('text', xy=(x,y), xytext=(tx,ty), arrowprops={...})
Style:         plt.style.use('seaborn-v0_8-darkgrid')

Seaborn plots:
  Bar:         sns.barplot(data=df, x=, y=, hue=)
  Count:       sns.countplot(data=df, x=)
  Scatter:     sns.scatterplot(data=df, x=, y=, hue=, style=)
  Box:         sns.boxplot(data=df, x=, y=, palette=)
  Violin:      sns.violinplot(data=df, x=, y=)
  Histogram:   sns.histplot(data=df, x=, kde=True, bins=)
  KDE:         sns.kdeplot(data=df, x=, fill=True)
  Pair plot:   sns.pairplot(df, hue=, diag_kind='kde')
  ECDF:        sns.ecdfplot(data=df, x=)

3D:            from mpl_toolkits.mplot3d import Axes3D
               ax = fig.add_subplot(111, projection='3d')
               ax.plot_surface(X, Y, Z, cmap='viridis')
```

---

**See also**: [[matplotlib]] · [[seaborn-visualization]] · [[data-science-overview]]
