# Matplotlib

**Summary**: Matplotlib is Python's foundational 2D plotting library. This page covers its component model, subplots, axis/tick/label/legend control, annotations, and saving figures.

**Pre-Req**: [[numpy]], basic Python

**Sources**: `Unit III - Data Science.pptx.pdf`

**Last updated**: 2026-05-11

#DS

---

## Introduction

Matplotlib was created by **John D. Hunter in 2003** to provide MATLAB-like plotting in Python. It is the base library that [[seaborn-visualization]] and most other Python viz tools are built on.

```python
import matplotlib.pyplot as plt
```

(source: Unit III - Data Science.pptx.pdf)

---

## Component Model

```
Figure
└── Axes (one or more)
    ├── x-axis / y-axis
    ├── tick marks + tick labels
    ├── axis labels
    ├── title
    ├── legend
    ├── annotations
    └── grid
```

- **Figure** — the top-level container (the window/canvas)
- **Axes** — the actual plot area (one Figure can have multiple Axes)

---

## Subplots

```python
# Create a single subplot (nrows, ncols, index)
plt.subplot(1, 2, 1)    # row 1, col 2, position 1

# Create grid of subplots — returns figure + array of Axes
fig, axes = plt.subplots(nrows=3, ncols=4, figsize=(10, 10))

# Super title over all subplots
fig.suptitle('Overall Title')
```

(source: Unit III - Data Science.pptx.pdf)

---

## Controlling Axes

```python
plt.xlim(0, 10)          # set x-axis limits
plt.ylim(-1, 1)          # set y-axis limits

ax = plt.gca()
ax.get_xlim()            # retrieve current limits
ax.set_xlim(0, 10)       # set via axes object
```

---

## Controlling Ticks

```python
plt.xticks([0, 1, 2, 3], ['a', 'b', 'c', 'd'], rotation=45)
plt.yticks([0, 0.5, 1.0])
```

---

## Labels and Legends

```python
plt.xlabel('X Axis Label')
plt.ylabel('Y Axis Label')
plt.title('Plot Title')

# Legend — requires label= in each plot call
plt.plot(x, y, label='Series A')
plt.legend(loc='upper right')   # loc options: best, upper left, lower right, etc.
```

---

## Annotations

Add text with optional arrow pointing to a data point.

```python
plt.annotate(
    text='Peak',
    xy=(x_data, y_data),          # where arrow points TO
    xytext=(x_text, y_text),      # where text appears
    arrowprops=dict(
        arrowstyle='->',
        color='black'
    ),
    bbox=dict(boxstyle='round', fc='yellow')
)

# Plain text on plot (no arrow)
plt.text(x, y, f"({x}, {y})", ha="center", va="bottom")

# Text as axes-relative coordinate (0–1 scale)
ax.text(0.5, 0.5, "center", horizontalalignment='center',
        verticalalignment='center', transform=ax.transAxes)
```

(source: Unit III - Data Science.pptx.pdf)

---

## Saving Figures

```python
plt.savefig('plot.png')            # PNG
plt.savefig('plot.pdf')            # PDF (vector)
plt.savefig('plot.svg')            # SVG (vector)
plt.savefig('plot.jpg', dpi=300)   # JPEG with resolution
```

**Formats**: PNG, JPEG, PDF, SVG

---

## Common Plot Types (Matplotlib)

```python
plt.plot(x, y)                  # line plot
plt.scatter(x, y)               # scatter plot
plt.bar(categories, values)     # bar chart
plt.hist(data, bins=10)         # histogram
plt.hist(data, density=True)    # normalized histogram (area = 1)
```

---

## Full Example

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, y, label='sin(x)', color='blue')
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_title('Sine Wave')
ax.legend(loc='upper right')
ax.grid(True)
plt.savefig('sine.png', dpi=150)
plt.show()
```
