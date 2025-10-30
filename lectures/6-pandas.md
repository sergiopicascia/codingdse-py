# Data Manipulation with Pandas

## Introduction to Pandas

[**Pandas**](https://pandas.pydata.org/docs/) is an open-source Python library built on top of NumPy. It provides high-performance, easy-to-use data structures and data analysis tools.

**Why Pandas?**

*   **Tabular Data:** It is specifically designed to work with tabular (table-like) data, similar to a spreadsheet or a SQL table.
*   **Data Alignment:** Intelligent handling of data alignment means you don't have to worry about mismatched indexes when performing operations between datasets.
*   **Missing Data:** Provides a robust toolkit for handling missing data (`NaN`).
*   **Performance:** Core operations are written in C or Cython, making them extremely fast.
*   **Rich Functionality:** Offers a vast range of tools for data wrangling, grouping, merging, reshaping, and time-series analysis.

First, let's install it and set up our environment.

```shell
# In your terminal, with your virtual environment activated
pip install pandas
```

The conventional alias for pandas is `pd`.

```python
import pandas as pd
```

## Core Data Structures: `Series` and `DataFrame`

Pandas has two primary data structures.

#### `Series`
A `Series` is a one-dimensional labeled array, capable of holding any data type. It's like a single column in a spreadsheet.

```python
# Creating a Series from a list
genres = pd.Series(['Action', 'Comedy', 'Drama', 'Sci-Fi'])
print(genres)
```
**Output:**
```
0     Action
1     Comedy
2      Drama
3     Sci-Fi
dtype: object
```
Notice the two components: the **index** (0, 1, 2, 3) on the left and the **values** on the right.

#### `DataFrame`
A `DataFrame` is a two-dimensional labeled data structure with columns of potentially different types. It is the most commonly used pandas object and can be thought of as a spreadsheet, a SQL table, or a dictionary of `Series` objects.

```python
# Creating a DataFrame from a dictionary
data = {
    'title': ['Movie A', 'Movie B', 'Movie C'],
    'rating': [8.5, 7.9, 9.1],
    'year': [2017, 2018, 2017]
}
df = pd.DataFrame(data)
print(df)
```
**Output:**
```
     title  rating  year
0  Movie A     8.5  2017
1  Movie B     7.9  2018
2  Movie C     9.1  2017
```

## Data Ingestion

For our case study, we will use a custom CSV file. Please create a file named `movies.csv` inside your `data/` directory with the following content:

```csv
rank,title,genre,description,director,actors,year,runtime_minutes,rating,votes,revenue_millions,metascore
1,Guardians of the Galaxy,Action,"A group of intergalactic criminals are forced to work together to stop a fanatical warrior from taking control of the universe.",James Gunn,"Chris Pratt, Vin Diesel, Bradley Cooper, Zoe Saldana",2014,121,8.1,757074,333.13,76
2,Prometheus,Adventure,"Following clues to the origin of mankind, a team finds a structure on a distant moon, but they soon realize they are not alone.",Ridley Scott,"Noomi Rapace, Logan Marshall-Green, Michael Fassbender, Charlize Theron",2012,124,7.0,485820,126.46,65
3,Split,Horror,"Three girls are kidnapped by a man with a diagnosed 23 distinct personalities. They must try to escape before the apparent emergence of a frightful new 24th.",M. Night Shyamalan,"James McAvoy, Anya Taylor-Joy, Haley Lu Richardson, Betty Buckley",2016,117,7.3,157606,138.12,62
4,Sing,Animation,"In a city of humanoid animals, a hustling theater impresario's attempt to save his theater with a singing competition becomes grander than he anticipates.",Christophe Lourdelet,"Matthew McConaughey,Reese Witherspoon, Seth MacFarlane, Scarlett Johansson",2016,108,7.2,60545,,60
5,Suicide Squad,Action,"A secret government agency recruits some of the most dangerous incarcerated super-villains to form a defensive task force. Their first mission: save the world from the apocalypse.",David Ayer,"Will Smith, Jared Leto, Margot Robbie, Viola Davis",2016,123,6.2,393727,325.02,40
6,The Great Wall,Action,"European mercenaries searching for black powder become embroiled in the defense of the Great Wall of China against a horde of monstrous creatures.",Yimou Zhang,"Matt Damon, Tian Jing, Willem Dafoe, Andy Lau",2016,103,6.1,56036,45.13,42
7,La La Land,Comedy,"A jazz pianist falls for an aspiring actress in Los Angeles.",Damien Chazelle,"Ryan Gosling, Emma Stone, Rosemarie DeWitt, J.K. Simmons",2016,128,8.3,258682,151.06,93
8,The Lost City of Z,Action,"A true-life drama, centering on British explorer Major Percival Fawcett, who disappeared while searching for a mysterious city in the Amazon in the 1920s.",James Gray,"Charlie Hunnam, Robert Pattinson, Sienna Miller, Tom Holland",2016,141,7.1,7188,8.01,78
9,Passengers,Adventure,"A spacecraft traveling to a distant colony planet and transporting thousands of people has a malfunction in its sleep chambers. As a result, two passengers are awakened 90 years early.",Morten Tyldum,"Jennifer Lawrence, Chris Pratt, Michael Sheen,Laurence Fishburne",2016,116,7.0,192177,100.01,41
10,Fantastic Beasts and Where to Find Them,Adventure,"The adventures of writer Newt Scamander in New York's secret community of witches and wizards seventy years before Harry Potter reads his book in school.",David Yates,"Eddie Redmayne, Katherine Waterston, Alison Sudol,Dan Fogler",2016,133,7.5,232072,234.03,66
11,Hidden Figures,Drama,"The story of a team of female African-American mathematicians who served a vital role in NASA during the early years of the U.S. space program.",Theodore Melfi,"Taraji P. Henson, Octavia Spencer, Janelle Monáe,Kevin Costner",2016,127,7.8,93103,169.27,74
12,The Dark Knight,Action,"When the menace known as the Joker emerges from his mysterious past, he wreaks havoc and chaos on the people of Gotham, the Dark Knight must accept one of the greatest psychological and physical tests of his ability to fight injustice.",Christopher Nolan,"Christian Bale, Heath Ledger, Aaron Eckhart, Michael Caine",2008,152,9.0,1791916,533.32,82
13,Inception,Action,"A thief who steals corporate secrets through the use of dream-sharing technology is given the inverse task of planting an idea into the mind of a C.E.O.",Christopher Nolan,"Leonardo DiCaprio, Joseph Gordon-Levitt, Ellen Page, Tom Hardy",2010,148,8.8,2000000,292.58,74
```

Now, let's load this data into a DataFrame using `pd.read_csv()`.

```python
# Load the dataset
file_path = 'data/movies.csv'
movies_df = pd.read_csv(file_path)
```

## Exploratory Data Analysis

Before manipulating the data, we must first understand it. This is a critical first step in any data analysis project.

**`df.head(n)`**: View the first `n` rows of the DataFrame (default is 5).

```python
print(movies_df.head())
```

**`df.info()`**: Provides a concise summary of the DataFrame. This is arguably the most important inspection method. It shows the index dtype, column dtypes, non-null values, and memory usage.

```python
movies_df.info()
```
**Output Analysis:**
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 12 entries, 0 to 11
Data columns (total 12 columns):
 #   Column            Non-Null Count  Dtype
---  ------            --------------  -----
 0   rank              12 non-null     int64
 1   title             12 non-null     object 
 2   genre             12 non-null     object
 3   description       12 non-null     object
 4   director          12 non-null     object
 5   actors            12 non-null     object
 6   year              12 non-null     int64
 7   runtime_minutes   12 non-null     int64
 8   rating            12 non-null     float64
 9   votes             12 non-null     int64
 10  revenue_millions  11 non-null     float64
 11  metascore         12 non-null     int64
dtypes: float64(2), int64(5), object(5)
memory usage: 1.3+ KB
```
From `.info()`, we immediately identify a potential issue: `revenue_millions` has only 11 non-null values out of 12 entries. This indicates one missing value.

**`df.describe()`**: Generates descriptive statistics for the numerical columns.

```python
print(movies_df.describe())
```
This gives us the count, mean, standard deviation, min, max, and quartile values for each numerical column. It's a great way to get a feel for the distribution of the data.

**`df.shape`**: Returns a tuple representing the dimensionality (rows, columns).
```python
print(f"Dataset dimensions: {movies_df.shape}")
```

## Indexing

Pandas offers a rich set of methods for selecting subsets of your data.

#### Selecting a Single Column
This returns a `Series`.
```python
titles = movies_df['title']
print(titles.head())
```

#### Selecting Multiple Columns
Pass a list of column names. This returns a new `DataFrame`.
```python
subset = movies_df[['title', 'director', 'rating', 'revenue_millions']]
print(subset.head())
```

#### Row Selection: `.loc` and `.iloc`
*   **`.loc`** selects by **label** (index name, column name).
*   **`.iloc`** selects by **integer position**.

```python
# .loc: Select the row with index label 3
print(movies_df.loc[3])

# .iloc: Select the row at integer position 0 (the first row)
print(movies_df.iloc[0])

# Slicing with .loc (inclusive of the end label)
print(movies_df.loc[2:5]) # Rows with index 2, 3, 4, 5

# Slicing with .iloc (exclusive of the end position)
print(movies_df.iloc[2:5]) # Rows at position 2, 3, 4
```

#### Boolean Indexing

This is the most common and powerful way to filter data. We can pass a Series of `True`/`False` values to select rows that meet a certain condition.

**Find all movies released after 2015.**

```python
# 1. Create the boolean condition
condition = movies_df['year'] > 2015

# 2. Apply the condition to the DataFrame
recent_movies = movies_df[condition]
print(recent_movies[['title', 'year']])
```

**Find all 'Action' movies with a rating higher than 8.0.**
We can combine conditions using `&` (and) and `|` (or). Each condition must be wrapped in parentheses.

```python
high_rated_action = movies_df[
    (movies_df['genre'] == 'Action') & (movies_df['rating'] > 8.0)
]
print(high_rated_action[['title', 'genre', 'rating']])
```

## Data Cleaning

Real-world data is rarely clean. We already identified a missing value in the `revenue_millions` column.

#### Finding Missing Values
The `.isnull()` (or `.isna()`) method returns a boolean DataFrame. We can chain it with `.sum()` to get a count of `NaN`s per column.

```python
print(movies_df.isnull().sum())
```
This confirms one missing value in `revenue_millions`.

#### Handling Missing Values
There are two primary strategies:
1.  **Deletion:** Remove the rows or columns with missing data (`.dropna()`). This is viable if you have a large dataset and only a few missing values.
2.  **Imputation:** Fill the missing values with some other value (`.fillna()`). A common choice is the mean or median of the column.

Let's use imputation. We'll fill the missing revenue with the average revenue of all other movies.

```python
# Calculate the mean revenue
mean_revenue = movies_df['revenue_millions'].mean()
print(f"Mean revenue: {mean_revenue:.2f} million")

# Fill NaN values with the mean. `inplace=True` modifies the DataFrame directly.
movies_df['revenue_millions'].fillna(mean_revenue, inplace=True)

# Verify that the missing value is gone
print("\nAfter filling missing values:")
movies_df.info()
```

## Data Manipulation

#### Creating New Columns
We can create new columns based on calculations from existing ones. Let's create a `rating_per_million_votes` column to normalize the rating score.

```python
movies_df['rating_per_vote'] = movies_df['rating'] / (movies_df['votes'] / 1_000_000)
print(movies_df[['title', 'rating', 'votes', 'rating_per_vote']].head())
```

#### Applying Functions with `.apply()`
The `.apply()` method is extremely versatile. It can apply a function along an axis of the DataFrame. Let's create a column that categorizes a movie's rating.

```python
# 1. Define a function
def rate_category(rating):
    if rating >= 8.0:
        return 'Excellent'
    elif rating >= 7.0:
        return 'Good'
    else:
        return 'Average'

# 2. Apply the function to the 'rating' column
movies_df['rating_category'] = movies_df['rating'].apply(rate_category)

print(movies_df[['title', 'rating', 'rating_category']].head())
```

## Grouping and Aggregation (`groupby`)

The `groupby` operation follows a **Split-Apply-Combine** pattern:
1.  **Split:** The data is split into groups based on some criteria (e.g., by 'genre' or 'director').
2.  **Apply:** A function (e.g., `mean`, `sum`, `count`) is applied to each group independently.
3.  **Combine:** The results are combined into a new data structure.

**Which movie genres are the most profitable on average?**

```python
# Group by genre and calculate the mean of the revenue_millions for each group
genre_revenue = movies_df.groupby('genre')['revenue_millions'].mean()

# Sort the results to see the most profitable
sorted_genre_revenue = genre_revenue.sort_values(ascending=False)
print(sorted_genre_revenue)
```

**Who are the most consistently high-rated directors?**
We can group by 'director' and calculate multiple aggregate statistics at once using the `.agg()` method.

```python
director_analysis = movies_df.groupby('director').agg(
    movie_count=('title', 'count'),
    avg_rating=('rating', 'mean'),
    avg_revenue=('revenue_millions', 'mean')
).sort_values(by='avg_rating', ascending=False)

print(director_analysis)
```

## Basic Visualization

Pandas integrates with Matplotlib, allowing for quick and simple plotting directly from a DataFrame or Series.

**How have movie ratings evolved over the years?**
Let's plot the average rating per year.

```python
# For plotting, it's good practice to install matplotlib
# pip install matplotlib

import matplotlib.pyplot as plt

# Group by year and calculate the mean rating
yearly_avg_rating = movies_df.groupby('year')['rating'].mean()

yearly_avg_rating.plot(
    kind='bar', # Type of plot
    title='Average Movie Rating by Year',
    xlabel='Year',
    ylabel='Average Rating'
)

# Display the plot
plt.tight_layout()
plt.show()
```