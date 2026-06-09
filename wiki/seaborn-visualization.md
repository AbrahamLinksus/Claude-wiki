# Seaborn and 3D Visualization

**Summary**: Seaborn is a statistical visualization library built on Matplotlib. This page covers scatter, line, histogram, boxplot, pair plot, word cloud, and 3D plots (scatter, line, surface) using mpl_toolkits.

**Pre-Req**: [[matplotlib]], [[numpy]]

**Sources**: `Unit III - Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Seaborn Overview

Seaborn is built on [[matplotlib]] and provides:
- Higher-level API for statistical plots
- Better default aesthetics
- Built-in support for pandas DataFrames
- Automatic grouping by categorical variables (`hue=`)

```python
import seaborn as sns
import matplotlib.pyplot as plt
```

(source: Unit III - Data Science.pptx.pdf)

---

## 2D Plot Types

### Scatter Plot
```python
sns.scatterplot(x="col_x", y="col_y", data=df)
sns.scatterplot(x="day", y="tip", data=tips_df)
```
Use: relationship between two continuous variables.

### Line Plot
```python
sns.lineplot(x=x_values, y=y_values)
```
Use: trends over time or ordered data.

### Histogram
```python
plt.hist(data, density=True)    # density=True normalizes to probability
```
Use: distribution of a single numeric variable.

### Boxplot
Shows the 5-number summary + outliers.

```
    Min   Q1  Median  Q3   Max
     |-----|----|----|-----|
           IQR = Q3 - Q1
     *                        * ← outliers (beyond 1.5×IQR)
```

```python
sns.boxplot(data=df)
sns.boxplot(data=df, showmeans=True, whis=1.5)  # whis controls fence multiplier
```
Use: comparing distributions, identifying outliers, univariate and bivariate analysis.

### Pair Plot
Plots every pairwise combination of variables in a DataFrame. Diagonal shows individual distributions.

```python
sns.pairplot(df)                # all columns
sns.pairplot(df, hue='class')   # color by category
```
Use: quick overview of all bivariate relationships in a dataset.

(source: Unit III - Data Science.pptx.pdf)

---

## Subplot Grid

```python
figure, axes = plt.subplots(3, 4, figsize=(10, 10))
# axes is a 3×4 array of Axes objects
sns.scatterplot(ax=axes[0, 0], data=df, x='x', y='y')
```

---

## Word Cloud

Visualizes text frequency — more frequent words appear larger.

```python
from wordcloud import WordCloud

wordcloud = WordCloud(width=800, height=400, background_color="white")
wordcloud.generate(text)

plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()
```

(source: Unit III - Data Science.pptx.pdf)

---

## 3D Plots (`mpl_toolkits.mplot3d`)

### Setup
```python
from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt
import numpy as np
```

### 3D Scatter Plot
```python
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

x = [1, 2, 3, 4, 5]
y = [2, 4, 3, 5, 1]
z = [10, 15, 20, 25, 30]

ax.scatter(x, y, z, c='blue', s=100)
ax.set_xlabel('X-axis')
ax.set_ylabel('Y-axis')
ax.set_zlabel('Z-axis')
ax.view_init(elev=20, azim=45)    # set viewing angle
plt.title('3D Scatter Plot')
plt.show()
```

### 3D Line Plot
```python
fig = plt.figure(figsize=(6, 5))
ax = fig.add_subplot(projection='3d')

ax.plot3D(x, y, z)
plt.title('3D Line Plot')
plt.show()
```

### 3D Surface Plot
Best for visualizing mathematical functions z = f(x, y).

```python
x = np.linspace(0, 5, 10)
y = np.linspace(0, 5, 10)
X, Y = np.meshgrid(x, y)      # create 2D coordinate grid

def f(x, y):
    return x**2 + y**2         # the function to visualize

Z = f(X, Y)

fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(111, projection='3d')

surf = ax.plot_surface(X, Y, Z,
    cmap='viridis',
    linewidth=0,
    antialiased=True)

ax.set_xlabel('X-axis')
ax.set_ylabel('Y-axis')
ax.view_init(elev=20, azim=45)
plt.title('3D Surface Plot')
plt.show()
```

**Key parameters:**
- `cmap` — colormap: `'viridis'`, `'plasma'`, `'coolwarm'`, etc.
- `view_init(elev, azim)` — `elev` = elevation angle (up/down), `azim` = azimuth (rotation)
- `np.meshgrid(x, y)` — creates 2D grids from 1D arrays; required for surface plots

(source: Unit III - Data Science.pptx.pdf)

---

## Seaborn vs Matplotlib

| Feature | Matplotlib | Seaborn |
|---|---|---|
| Level | Low-level, full control | High-level, statistical focus |
| Default style | Basic | Polished |
| DataFrame support | Manual | Built-in (`data=df`) |
| Statistical plots | Manual | Native (boxplot, pairplot) |
| 3D plots | Yes (mpl_toolkits) | No |
| Use case | Custom control | Quick statistical exploration |

---

## Static vs Interactive Visualizations

| Feature | Static | Interactive |
|---|---|---|
| Rendering | Image (PNG, PDF, SVG) | HTML/JS (browser) |
| User interaction | None | Zoom, hover, filter, click |
| Libraries | Matplotlib, Seaborn | Plotly, Bokeh, Altair, Dash |
| Use case | Reports, papers, print | Dashboards, exploratory analysis |
| File size | Small | Larger (includes JS) |

```python
# Static (Matplotlib/Seaborn)
import matplotlib.pyplot as plt
plt.savefig('chart.png')

# Interactive (Plotly)
import plotly.express as px
fig = px.scatter(df, x='day', y='tip', color='sex')
fig.show()   # opens in browser with hover/zoom
```

**Exam question**: "What is the difference between static and interactive visualizations?" (source: Question Bank_Data Science.docx)

---

## Customized Bar Chart (Matplotlib)

```python
import matplotlib.pyplot as plt
import numpy as np

categories = ['Q1', 'Q2', 'Q3', 'Q4']
sales = [15000, 22000, 18000, 27000]
target = [18000, 20000, 20000, 25000]

x = np.arange(len(categories))
width = 0.35

fig, ax = plt.subplots(figsize=(9, 5))

bars1 = ax.bar(x - width/2, sales,  width, label='Actual Sales',  color='steelblue')
bars2 = ax.bar(x + width/2, target, width, label='Target',         color='orange')

# Labels and title
ax.set_xlabel('Quarter')
ax.set_ylabel('Revenue (USD)')
ax.set_title('Quarterly Sales vs Target')

# Axis control — custom ticks
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.set_ylim(0, 32000)

# Legend
ax.legend(loc='upper left')

# Add value labels on bars
for bar in bars1:
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 200,
            f'{bar.get_height():,}', ha='center', va='bottom', fontsize=9)

ax.grid(axis='y', linestyle='--', alpha=0.5)
plt.tight_layout()
plt.savefig('sales_chart.png', dpi=150)
plt.show()
```

**Exam question**: "Create a customized bar chart showing sales data using Matplotlib: add labels, title, legend, and axis control" (source: Question Bank_Data Science.docx)

---

## Histogram with KDE Overlay

KDE (Kernel Density Estimate) is a smooth curve showing the estimated probability distribution.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

data = np.random.normal(loc=50, scale=10, size=200)

fig, ax = plt.subplots(figsize=(8, 5))

# Seaborn histplot with KDE
sns.histplot(data, kde=True, ax=ax, color='steelblue',
             bins=20, stat='density')

ax.set_xlabel('Value')
ax.set_ylabel('Density')
ax.set_title('Histogram with KDE Overlay')
plt.tight_layout()
plt.show()

# Alternative using distplot (older API)
# sns.distplot(data, bins=20, kde=True)
```

**Exam question**: "Generate: histogram with KDE overlay using Seaborn" (source: Question Bank_Data Science.docx)

---

## Box Plot for Comparing Distributions

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Distribution by category
sns.boxplot(x='day', y='total_bill', data=tips, ax=axes[0])
axes[0].set_title('Total Bill by Day')

# With hue (grouping variable)
sns.boxplot(x='day', y='total_bill', hue='sex', data=tips, ax=axes[1])
axes[1].set_title('Total Bill by Day and Sex')

plt.tight_layout()
plt.show()
```

**Exam question**: "Generate: box plot for comparing distributions using Seaborn" (source: Question Bank_Data Science.docx)

---

## Scatterplot — Day vs Tip

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic scatter: day on x, tip on y
sns.scatterplot(x='day', y='tip', data=tips, ax=axes[0])
axes[0].set_xlabel('Day')
axes[0].set_ylabel('Tip')
axes[0].set_title('Day vs Tip')

# With relationship on a second axis — regression line
sns.regplot(x='total_bill', y='tip', data=tips, ax=axes[1])
axes[1].set_title('Total Bill vs Tip (with regression)')

plt.tight_layout()
plt.show()
```

**Exam question**: "Prepare a scatterplot for day (x-axis) vs tip (y-axis) and also represent a relationship between X and Y on a different axis" (source: Question Bank_Data Science.docx)

---

## Styling Plots — Annotations, Font, Color Themes

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

# Use a built-in style
plt.style.use('seaborn-v0_8-darkgrid')

fig, ax = plt.subplots(figsize=(9, 5))
ax.plot(x, y, color='#E84393', linewidth=2, label='sin(x)')

# Font customization
ax.set_title('Styled Sine Plot', fontsize=16, fontweight='bold', color='navy')
ax.set_xlabel('x', fontsize=13)
ax.set_ylabel('sin(x)', fontsize=13)

# Annotation with style
ax.annotate('Peak',
    xy=(np.pi/2, 1), xytext=(2, 1.2),
    arrowprops=dict(arrowstyle='->', color='black', lw=1.5),
    fontsize=11, color='darkred',
    bbox=dict(boxstyle='round,pad=0.3', fc='lightyellow', ec='orange')
)

ax.legend(fontsize=11)
plt.tight_layout()
plt.savefig('styled_plot.png', dpi=150)
plt.show()
```

**Exam question**: "Style a plot with annotations, font styles, and color themes" (source: Question Bank_Data Science.docx)

---

## Which Plot for Which Data Type

### Univariate — One Variable

| Data Type | Plot | Library |
|---|---|---|
| Numerical (continuous) | Histogram | `plt.hist()` / `sns.histplot()` |
| Numerical (continuous) | KDE plot | `sns.kdeplot()` |
| Numerical (continuous) | Boxplot | `sns.boxplot()` |
| Categorical | Bar chart | `sns.countplot()` |
| Categorical | Pie chart | `plt.pie()` |

```python
# Univariate — numerical
sns.histplot(df['Age'], kde=True)
sns.kdeplot(df['Age'])
sns.boxplot(y=df['Age'])

# Univariate — categorical
sns.countplot(x='Gender', data=df)
df['City'].value_counts().plot(kind='bar')
```

**Exam question**: "Illustrate in detail about Univariate visualization for Numerical data" and "Predict the different Univariate visualization for categorical data" (source: Question Bank_Data Science.docx)

### Bivariate — Two Variables

| Variables | Plot | When to use |
|---|---|---|
| Num × Num | Scatter | Correlation, trend |
| Num × Num | Line | Ordered/time data |
| Num × Num | Regression (regplot) | Relationship + CI |
| Cat × Num | Boxplot / Violin | Compare distributions |
| Cat × Num | Bar chart (mean) | Group averages |
| Cat × Cat | Heatmap / Crosstab | Frequency of combinations |

```python
# Bivariate scatter
sns.scatterplot(x='height', y='weight', data=df)

# Violin plot — richer than boxplot (shows distribution shape)
sns.violinplot(x='day', y='tip', data=tips)

# Heatmap — correlation matrix
import seaborn as sns
corr = df.corr()
sns.heatmap(corr, annot=True, cmap='coolwarm', fmt='.2f')
```

**Exam question**: "Compare univariate and bivariate graphs" (source: Question Bank_Data Science.docx)

---

## Comparing Values Between Groups

Plots that show differences across categories:

```python
import matplotlib.pyplot as plt
import seaborn as sns

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(2, 3, figsize=(14, 8))

# 1. Bar chart — mean value per group
tips.groupby('day')['total_bill'].mean().plot(kind='bar', ax=axes[0,0], color='steelblue')
axes[0,0].set_title('Mean Bill by Day')

# 2. Grouped bar chart
tips.groupby(['day','sex'])['total_bill'].mean().unstack().plot(kind='bar', ax=axes[0,1])
axes[0,1].set_title('Bill by Day & Sex')

# 3. Boxplot — distribution per group
sns.boxplot(x='day', y='total_bill', data=tips, ax=axes[0,2])
axes[0,2].set_title('Bill Distribution by Day')

# 4. Violin plot — distribution shape
sns.violinplot(x='day', y='tip', data=tips, ax=axes[1,0])
axes[1,0].set_title('Tip Distribution by Day')

# 5. Strip plot — all points
sns.stripplot(x='day', y='tip', data=tips, jitter=True, ax=axes[1,1])
axes[1,1].set_title('All Tips by Day')

# 6. Count plot — frequency
sns.countplot(x='day', data=tips, ax=axes[1,2])
axes[1,2].set_title('Count by Day')

plt.tight_layout()
plt.show()
```

**Exam question**: "Demonstrate the different set of graphs that are used to compare values between groups using Matplotlib" (source: Question Bank_Data Science.docx)

---

## Visualizing Data Distribution

Plots that show how data values are distributed:

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

data = np.random.normal(50, 15, 500)
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# 1. Histogram — frequency bins
axes[0,0].hist(data, bins=25, color='steelblue', edgecolor='white')
axes[0,0].set_title('Histogram')

# 2. Histogram + KDE
sns.histplot(data, kde=True, ax=axes[0,1], color='coral')
axes[0,1].set_title('Histogram + KDE')

# 3. KDE only — smooth density
sns.kdeplot(data, ax=axes[1,0], fill=True, color='green')
axes[1,0].set_title('KDE Plot')

# 4. Boxplot — summary statistics
axes[1,1].boxplot(data, vert=True, patch_artist=True)
axes[1,1].set_title('Boxplot')

plt.tight_layout()
plt.show()
```

**Exam question**: "Show the different charts to visualize about how data points' values are distributed" (source: Question Bank_Data Science.docx)

---

## Multi-Plot Dashboards

When multiple visualizations are shown together, decision-making improves because:
- Patterns become visible across dimensions simultaneously
- Outliers in one chart can be cross-referenced in another
- Filters on one chart can propagate to others (interactive dashboards)

**Example real-world scenario**: A sales manager dashboard showing — simultaneously — regional sales trends (line), product category breakdown (bar), outlier orders (boxplot), and customer distribution (map). Spotting a spike in the line chart immediately links to a specific region in the map.

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle('Sales Dashboard', fontsize=16)

# Panel 1 — trend
axes[0,0].plot(dates, revenue)
axes[0,0].set_title('Revenue Trend')

# Panel 2 — category breakdown
axes[0,1].bar(categories, values)
axes[0,1].set_title('By Category')

# Panel 3 — distribution
sns.histplot(df['order_value'], ax=axes[1,0], kde=True)
axes[1,0].set_title('Order Value Distribution')

# Panel 4 — correlation
sns.heatmap(df[['revenue','units','returns']].corr(),
            annot=True, ax=axes[1,1])
axes[1,1].set_title('Feature Correlation')

plt.tight_layout()
plt.savefig('dashboard.png', dpi=150)
```

**Exam question**: "Describe a real-world scenario where multi-plot dashboards improve decision-making" (source: Question Bank_Data Science.docx)

---

## Visualization for Pattern and Anomaly Detection

- **Scatter plots** — reveal clusters, outliers, and non-linear relationships
- **Boxplots** — flag statistical outliers using IQR fences
- **Heatmaps** — show unusually high/low correlations
- **Time series line plots** — expose spikes, dips, and seasonal patterns
- **Pair plots** — reveal which variable pairs show anomalous behavior
- **KDE + histogram** — show bimodal distributions or heavy tails

```python
# Scatter with outlier highlighting
z = np.abs(stats.zscore(df['value']))
df['is_outlier'] = z > 3

sns.scatterplot(x='feature1', y='value', hue='is_outlier',
                palette={True: 'red', False: 'blue'}, data=df)
```

**Exam question**: "How can data visualization help in identifying patterns and anomalies?" (source: Question Bank_Data Science.docx)
