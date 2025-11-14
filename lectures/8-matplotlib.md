# Data Visualization with Matplotlib

## Matplotlib Fundamentals

First, let's install the library.
```shell
pip install matplotlib
```

To effectively use Matplotlib, you must understand its object hierarchy. Almost every element of a plot is a configurable object.

![Matplotlib Anatomy](https://matplotlib.org/stable/_images/anatomy.png)
*(Image from the official Matplotlib documentation)*

Here are the most critical components:
*   **Figure:** The top-level container for everything. It's the overall window or page that everything is drawn on.
*   **Axes:** This is what you might think of as the actual "plot". It is the region of the image with the data space. A single `Figure` can contain multiple `Axes` (subplots).
*   **Axis:** These are the number-line-like objects that control the graph limits and ticks (the marks on the axis). Each `Axes` has an X-Axis and a Y-Axis.
*   **Artist:** Everything you see on the figure is an artist, including the `Figure`, `Axes`, and `Axis` objects, as well as text, lines, and patches.

## Plotting Interfaces

Matplotlib has two primary ways of being used.

#### a) The `pyplot` (State-Based) Interface
This is a collection of functions that operate on a "current" figure and axes. It's convenient for quick, interactive plotting.

```python
import matplotlib.pyplot as plt

x = [1, 2, 3]
y = [4, 7, 11]

plt.plot(x, y)
plt.xlabel("X Axis")
plt.ylabel("Y Axis")
plt.title("My First Plot")
plt.show()
```

While simple, this approach can become confusing when you have multiple plots or want fine-grained control.

#### b) The Object-Oriented (OO) Interface (Recommended)
This is the best practice. We explicitly create `Figure` and `Axes` objects and then call methods on them. This gives us full control over our canvas.

```python
import matplotlib.pyplot as plt

x = [1, 2, 3]
y = [4, 7, 11]

# Create a Figure and a set of subplots (in this case, just one)
# `fig` is the Figure object, `ax` is the Axes object
fig, ax = plt.subplots()

# Use the Axes object to plot
ax.plot(x, y)

# Use the Axes object's methods to customize
ax.set_xlabel("X Axis")
ax.set_ylabel("Y Axis")
ax.set_title("Plot Title")

plt.show()
```
From now on, we will exclusively use the **Object-Oriented** approach.

## Basic Plot Types

Let's load the classic Iris dataset, which contains measurements for three species of iris flowers.

```python
import pandas as pd
from sklearn.datasets import load_iris

# Load the dataset
iris = load_iris()

# Create a Pandas DataFrame for easier manipulation
data = pd.DataFrame(data=iris.data, columns=iris.feature_names)
data['target'] = iris.target
data['species'] = data['target'].map({0: iris.target_names[0], 1: iris.target_names[1], 2: iris.target_names[2]})

print(data.head())
```

#### Line Plot (`ax.plot`)
Line plots are excellent for showing the relationship between ordered points or tracking a value's evolution over time.

```python
import numpy as np

# Create some smooth data for a better line plot example
x_sin = np.linspace(0, 10, 100)
y_sin = np.sin(x_sin)

fig, ax = plt.subplots()
ax.plot(x_sin, y_sin)
ax.set_title("A Sine Wave")
ax.set_xlabel("X Value")
ax.set_ylabel("sin(X)")
plt.show()
```

#### Scatter Plot (`ax.scatter`)
Scatter plots are ideal for visualizing the relationship between two numerical variables to identify correlations or clusters.

```python
# Let's visualize the relationship between sepal length and sepal width
fig, ax = plt.subplots(figsize=(8, 6)) # figsize controls the figure size in inches

ax.scatter(data["sepal length (cm)"], data["sepal width (cm)"])
ax.set_xlabel("Sepal Length (cm)")
ax.set_ylabel("Sepal Width (cm)")
ax.set_title("Sepal Length vs. Sepal Width")
plt.show()
```

## Plot Customization

A default plot is good, but a great plot is customized for clarity and impact.

#### Colors, Markers, and Line Styles

```python
fig, ax = plt.subplots(figsize=(10, 6))

# Use the `c` or `color` argument for color
# Use `marker` to control the point shape
# Use `s` to control the marker size
# Use `alpha` for transparency, useful for overlapping points
ax.scatter(data["sepal length (cm)"], data["sepal width (cm)"],
           color="darkblue", marker="x", s=50, alpha=0.6)

ax.set_title("Customized Scatter Plot")
plt.show()
```

#### Labels, Titles, and Legends

A legend is crucial when plotting multiple groups. Let's color our scatter plot by species.

```python
fig, ax = plt.subplots(figsize=(10, 6))
colors = {"setosa": "red", "versicolor": "green", "virginica": "blue"}

for species_name in data['species'].unique():
    subset = data[data['species'] == species_name]
    ax.scatter(subset["sepal length (cm)"], subset["sepal width (cm)"],
               c=colors[species_name],
               label=species_name) # The label is used by the legend

ax.set_xlabel("Sepal Length (cm)")
ax.set_ylabel("Sepal Width (cm)")
ax.set_title("Sepal Dimensions by Species")
ax.legend() # This command displays the legend
plt.show()
```

## Statistical Plot Types

#### Bar Plot (`ax.bar`, `ax.barh`)
Bar plots compare a numerical value across different categories. Let's find the average sepal length for each species.

```python
# Prepare the data
avg_sepal_length = data.groupby("species")["sepal length (cm)"].mean()

fig, ax = plt.subplots()

# Vertical Bar Plot
ax.bar(avg_sepal_length.index, avg_sepal_length.values, color=["red", "green", "blue"])
ax.set_ylabel("Average Sepal Length (cm)")
ax.set_title("Average Sepal Length per Iris Species")
plt.show()

# Horizontal Bar Plot
fig, ax = plt.subplots()
ax.barh(avg_sepal_length.index, avg_sepal_length.values, color=["#ff9999", "#99ff99", "#9999ff"])
ax.set_xlabel("Average Sepal Length (cm)")
ax.set_title("Average Sepal Length per Iris Species")
plt.show()
```

#### Histogram (`ax.hist`)
Histograms show the distribution of a single numerical variable by dividing the data into "bins" and counting the observations in each bin.

```python
fig, ax = plt.subplots()

# Plot a histogram of sepal length for all species
ax.hist(data["sepal length (cm)"], bins=20, edgecolor="black")
ax.set_xlabel("Sepal Length (cm)")
ax.set_ylabel("Frequency")
ax.set_title("Distribution of Sepal Lengths")
plt.show()
```

#### Box Plot (via Pandas)
Box plots are excellent for summarizing a distribution. They show the median, quartiles, and potential outliers in a compact way. Pandas has a convenient wrapper for Matplotlib.

```python
# The box represents the interquartile range (IQR), from Q1 to Q3.
# The line inside the box is the median (Q2).
# The "whiskers" typically extend to 1.5 * IQR from the box.
# Points beyond the whiskers are considered potential outliers.
fig, ax = plt.subplots(figsize=(8, 6))

data.boxplot("sepal length (cm)", by="species", ax=ax)
ax.set_title("Sepal Length Distribution by Species")
ax.set_ylabel("Sepal Length (cm)")
fig.suptitle('') # Suppress the automatic title from pandas
plt.show()
```

## Multiple Subplots

The true power of the OO interface shines when creating figures with multiple plots.

```python
# Create a figure with a 1x2 grid of axes
fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(12, 5))

# Plot on the first axes
axes[0].scatter(data["sepal length (cm)"], data["sepal width (cm)"])
axes[0].set_title("Sepal Length vs. Width")
axes[0].set_xlabel("Sepal Length (cm)")
axes[0].set_ylabel("Sepal Width (cm)")


# Plot on the second axes
axes[1].scatter(data["petal length (cm)"], data["petal width (cm)"])
axes[1].set_title("Petal Length vs. Width")
axes[1].set_xlabel("Petal Length (cm)")
axes[1].set_ylabel("Petal Width (cm)")


# Improve layout
plt.tight_layout()
plt.show()
```

## Beyond Matplotlib

While Matplotlib is powerful, other libraries offer simpler interfaces for complex plots or add new capabilities.

#### Seaborn: High-Level Statistical Visualization
Seaborn is built on top of Matplotlib and is designed to create beautiful, informative statistical plots with very little code.

```shell
pip install seaborn
```

Notice how the complex `for` loop we wrote for the species-colored scatter plot becomes a single line in Seaborn using the `hue` parameter.

```python
import seaborn as sns

# Use a pre-built Seaborn style
sns.set_theme(style="whitegrid")

# Create the plot
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(data=data,
                x="sepal length (cm)",
                y="sepal width (cm)",
                hue="species",  # Automatically colors by this column and creates a legend
                s=60,           # Marker size
                ax=ax)

ax.set_title("Sepal Dimensions by Species (Seaborn)")
plt.show()
```

#### Plotly: Interactive Visualizations
Plotly creates fully interactive, web-ready plots. You can zoom, pan, and hover over data points to see more information.

```shell
pip install plotly
```

```python
import plotly.express as px

# Plotly Express is a high-level interface for creating figures
fig = px.scatter(data,
                 x="sepal length (cm)",
                 y="sepal width (cm)",
                 color="species", # Automatically handles colors and legend
                 title="Interactive Scatter Plot of Iris Data (Plotly)",
                 hover_data=['petal length (cm)', 'petal width (cm)']) # Show this data on hover

fig.show()
```