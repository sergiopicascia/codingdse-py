# Data Manipulation with pandas

Data rarely come ready for the analysis: numbers stored as text, footnotes and missing values, several values in a single cell, information split across two tables. Before you can analyse and visualise them, you'll need to clean, transform and reshape them.

**pandas** is the most widely used Python library for tabular data: tables with rows and columns, like those of a spreadsheet or of a database. It reads and writes many formats, from CSV to Excel and SQL, and it selects, cleans, transforms, combines and summarises data with a few lines of code.

This lecture covers the two data structures of pandas, `Series` and `DataFrame`, and how to explore and select their data; then the index and alignment, cleaning text, numbers and dates, how modifications work (Copy-on-Write), transforming columns without loops, grouping, reshaping, combining tables, time series and large datasets.

## Series and DataFrames

The examples run from the root of vgrank (in a notebook, build the paths from `PROJECT_ROOT`, as we saw in the project organization lecture), and use some of the data of the Data I/O lecture: the best-selling video games scraped from Wikipedia, the weather from Open-Meteo, and the goodbooks-10k dataset. pandas is not part of the standard library: install it with `pip install pandas` (vgrank has it in its `requirements.txt`), and import it as `pd`, as everybody does:

```python
import pandas as pd
```

#### Series

A **Series** is a sequence of values of the same type, each with a **label**: like a column of a spreadsheet, with a name for each row. It can be created from a dictionary, whose keys become the labels:

```python
metacritic = pd.Series({"Hades": 93, "Hollow Knight": 88, "The Witcher 3": 92})
metacritic
```

```text
Hades            93
Hollow Knight    88
The Witcher 3    92
dtype: int64
```

The labels, on the left, form the **index** of the Series, and the values are on the right; at the bottom, there's the type of the values, the **dtype**: `int64` means integers. From a list, `pd.Series([93, 88, 92])`, the labels are the positions: 0, 1 and 2.

A Series works like a dictionary, where each value has its label, but its operations apply to all the values at once, without loops:

```python
metacritic["Hollow Knight"]  # 88
metacritic / 10  # a new Series: 9.3, 8.8, 9.2
metacritic > 90  # a Series of booleans: True, False, True
metacritic.mean()  # 91.0
metacritic.idxmax()  # 'Hades', the label of the highest value
```

#### DataFrames

A **DataFrame** is a table: a set of columns, each one a Series, which share the same index. It can be created from a dictionary with a list of values for each column:

```python
podium = pd.DataFrame(
    {
        "title": ["Hades", "Hollow Knight", "The Witcher 3: Wild Hunt"],
        "released": ["2020-09-17", "2017-02-23", "2015-05-18"],
        "metacritic": [93.0, 88.0, 92.0],
        "gameplay": [10, 10, 9],
    }
)
podium
```

```text
                      title    released  metacritic  gameplay
0                     Hades  2020-09-17        93.0        10
1             Hollow Knight  2017-02-23        88.0        10
2  The Witcher 3: Wild Hunt  2015-05-18        92.0         9
```

These are the first three games of my ranking. The index numbers the rows from 0, and each column has its own dtype (`podium.dtypes` lists them). A column is selected by its name, in square brackets, and it's a Series: `podium["title"]`. A list of dictionaries, one per row, with the names of the columns as keys, works too: that's how `to_dataframe`, in `vgrank/collection.py`, builds the table of the games, from the `to_dict` of each `Game`.

> **NOTE**: the documentation and many examples online call their DataFrames `df`. That's fine for a function that works on any table, but in your analysis, prefer names that say what a table contains, like `games` or `ratings`: in a notebook with five tables, `df`, `df2` and `df_clean` don't.

#### Exploring a Table

Most of the time, a DataFrame comes from a file: `pd.read_csv` reads a CSV file (the Data I/O lecture lists the functions for the other formats, like `read_json` and `read_excel`). Let's read the rated games of vgrank:

```python
games = pd.read_csv("data/games.csv")
games.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 16 entries, 0 to 15
Data columns (total 12 columns):
 #   Column         Non-Null Count  Dtype
---  ------         --------------  -----
 0   game_id        16 non-null     int64
 1   title          16 non-null     str
 2   released       16 non-null     str
 3   genres         16 non-null     str
 4   platforms      16 non-null     str
 5   metacritic     16 non-null     float64
 6   image_url      16 non-null     str
 7   gameplay       16 non-null     int64
 8   story          16 non-null     int64
 9   graphics       16 non-null     int64
 10  sound          16 non-null     int64
 11  replayability  16 non-null     int64
dtypes: float64(1), int64(6), str(5)
memory usage: 4.3 KB
```

`info()` is the first thing to run on a new dataset: the number of rows, and the name, the type and the number of non-missing values of each column (16 out of 16: no missing values here, which pandas would show as `NaN`, *Not a Number*). Integers are `int64`, decimal numbers `float64`, text is `str`: we'll meet other types later, for dates and categories. `games.shape` gives the size of the table, `(16, 12)`: 16 rows and 12 columns.

To look at the data, `head()` shows the first 5 rows (`head(10)`, the first 10), `tail()` the last ones, and `sample(5)` five random rows: in a notebook, they're displayed as a table below the cell. `describe()` computes the main statistics of the numeric columns: the number of values, the mean, the standard deviation, the minimum, the quartiles (50% is the median) and the maximum:

```python
games.describe().round(1)  # rounded to one decimal, to make it readable
```

```text
        game_id  metacritic  gameplay  story  graphics  sound  replayability
count      16.0        16.0      16.0   16.0      16.0   16.0           16.0
mean    45861.3        89.4       8.4    7.0       8.0    8.7            7.4
std    100409.1         5.7       1.0    2.2       1.4    1.2            1.8
min        28.0        79.0       7.0    2.0       5.0    6.0            5.0
25%      3309.2        85.8       8.0    6.0       7.0    8.0            6.0
50%      5069.0        90.5       8.0    7.5       8.0    9.0            7.5
75%     20242.5        94.2       9.0    8.0       9.0   10.0            8.2
max    326243.0        96.0      10.0   10.0      10.0   10.0           10.0
```

`describe()` doesn't know what the columns mean: the statistics of `game_id` make no sense, since an identifier is not a quantity. The others tell a story: my gameplay scores are high, and close to each other (a standard deviation of 1.0), while the story scores are the lowest on average, and the most spread out, from 2 to 10.

Each statistic also has its own method: `mean()`, `median()`, `min()`, `max()`, `sum()`, `std()`, and `count()`, which counts the values that are not missing. On a Series, they return a single value (`games["gameplay"].mean()` is 8.4375); on a DataFrame, a value for each column. `value_counts()` counts how many times each value appears, from the most common:

```python
games["gameplay"].value_counts()
```

```text
gameplay
8     6
9     4
10    3
7     3
Name: count, dtype: int64
```

Six games got an 8 for gameplay, and none less than 7.

## Selecting Data

#### Columns

A name in square brackets selects a column, as a Series; a list of names selects those columns, in that order, as a DataFrame (hence the double brackets). Assigning to a column creates it, or replaces it if it already exists; and since operations work on whole columns, as with Series, one line computes the value of every row. Here, the Metacritic score of each game, on a 10-point scale like mine:

```python
games["metacritic_10"] = games["metacritic"] / 10
games[["title", "metacritic", "metacritic_10"]].head(3)
```

```text
                      title  metacritic  metacritic_10
0  The Witcher 3: Wild Hunt        92.0            9.2
1        Grand Theft Auto V        92.0            9.2
2                  Portal 2        95.0            9.5
```

To remove columns, `drop(columns=["image_url"])` returns the table without them; `rename` changes their names (see [Names and Types](#names-and-types)).

#### Rows

`loc` selects rows (and columns) by **label**, `iloc` by **position**, from 0, as in Python. Both take the rows first and, optionally, the columns, separated by a comma, and each of them can be a single label (or position), a list or a slice:

```python
games.loc[0, "title"]  # row label 0, column "title": 'The Witcher 3: Wild Hunt'
games.loc[0]  # row label 0, all the columns: a Series
games.loc[0:2, ["title", "metacritic"]]  # row labels 0 to 2, two columns: a DataFrame
games.iloc[-1, 1]  # last row, second column: 'Hollow Knight'
games.iloc[:3, :2]  # first three rows, first two columns
```

Here, the labels of the rows are their positions, so `loc` and `iloc` look alike, but they are not. A slice with `loc` includes its end, so `0:2` gives three rows, while a slice with `iloc` doesn't, as in Python. And once a table is sorted or filtered, the labels are no longer the positions (see [The Index](#the-index)).

#### Filters

A comparison on a column gives a boolean Series, with a `True` or a `False` for each row. Inside `[]` or `loc`, it works as a **filter**, which keeps the rows with `True`: `games[games["metacritic"] >= 95]` gives the four games with a Metacritic score of at least 95. Conditions are combined with `&` (and), `|` (or) and `~` (not):

```python
# The games with a 10 for gameplay, or with a Metacritic score of at least 95
mask = (games["gameplay"] == 10) | (games["metacritic"] >= 95)
games.loc[mask, ["title", "gameplay", "metacritic"]]
```

```text
                    title  gameplay  metacritic
2                Portal 2        10        95.0
3   Red Dead Redemption 2         8        96.0
5     Grand Theft Auto IV         7        95.0
10                  Hades        10        93.0
13             Elden Ring         9        96.0
15          Hollow Knight        10        88.0
```

Each condition needs its parentheses, since Python evaluates `&` and `|` before `==` and `>=`: without them, you get an error. Python's `and` and `or` don't work with Series either (`ValueError: The truth value of a Series is ambiguous`). A few shortcuts:

```python
games[games["title"].isin(["Hades", "Celeste"])]  # only Hades: Celeste isn't rated
games[games["metacritic"].between(80, 85)]  # both ends included
games[~games["genres"].str.contains("Action")]  # the games that are not Action games
games.query("metacritic >= 95 and story >= 9")  # a filter written as a string
games.nlargest(3, "metacritic")  # the 3 games with the highest Metacritic score
```

## The Index

#### Labels

`read_csv` labels the rows with numbers from 0, but any column with unique values can become the index, and its values can then be used with `loc`:

```python
by_id = games.set_index("game_id")
by_id.loc[274755, "title"]  # 'Hades'
```

`reset_index()` does the opposite, and turns the index back into a column. The index is not just a row number: it's how pandas matches rows, and it stays with each row through filters and sorting.

#### Alignment

When an operation involves two Series, or a DataFrame and a Series, pandas matches their **labels**, not their positions. This is how we can compute the overall score of every game, with the weights of `vgrank/scoring.py`:

```python
from vgrank.scoring import CATEGORIES, WEIGHTS

weights = pd.Series(WEIGHTS)
weighted = games[CATEGORIES] * weights  # each column times the weight with its label
weighted.head(3)
```

```text
   gameplay  story  graphics  sound  replayability
0       2.7    2.0       1.8    1.0            1.4
1       2.4    1.6       1.6    0.9            1.6
2       3.0    1.8       1.4    0.8            1.2
```

```python
games["overall"] = weighted.sum(axis=1).round(2)  # axis=1: the sum of each row
games[["title", "overall"]].head(3)
```

```text
                      title  overall
0  The Witcher 3: Wild Hunt      8.9
1        Grand Theft Auto V      8.1
2                  Portal 2      8.2
```

The weights sum to 1, so the weighted sum is the weighted average: the same result as `add_overall_column` in `vgrank/scoring.py`, which uses NumPy (the topic of the next lecture). The order of the weights doesn't matter, only their labels do. But when the labels don't match, alignment creates a silent bug:

```python
typo = pd.Series(
    {"gameplay": 0.3, "story": 0.2, "graphic": 0.2, "sound": 0.1, "replayability": 0.2}
)
(games[CATEGORIES] * typo).head(3)
```

```text
   gameplay  graphic  graphics  replayability  sound  story
0       2.7      NaN       NaN            1.4    1.0    2.0
1       2.4      NaN       NaN            1.6    0.9    1.6
2       3.0      NaN       NaN            1.2    0.8    1.8
```

`graphic` is not `graphics`: pandas keeps the labels of both sides, and puts `NaN` where one of them is missing. There's no error, and the sum makes it worse: `sum` skips the missing values, so the overall score of The Witcher 3 becomes 7.1, instead of 8.9. With `sum(axis=1, skipna=False)`, the result would be `NaN`, which at least you'd notice. Whenever you combine data, look for unexpected missing values (`isna().sum()`).

#### Sorting and Ranking

```python
ranking = games.sort_values(["overall", "title"], ascending=[False, True])
ranking[["title", "overall"]].head(3)
```

```text
                       title  overall
10                     Hades      9.3
15             Hollow Knight      9.0
0   The Witcher 3: Wild Hunt      8.9
```

Sorting keeps the labels: Hades is still row 10. `reset_index(drop=True)` numbers the rows again from 0 (`drop=True` throws the old index away, instead of keeping it as a column), and the position in the ranking is then the index plus 1: that's what `rank_games` does in `vgrank/scoring.py`. The `rank` method, instead, can give the same position to ties:

```python
ranking = ranking.reset_index(drop=True)
ranking["rank"] = ranking.index + 1  # as in vgrank: ties are sorted by title
ranking["rank_tied"] = (
    ranking["overall"].rank(ascending=False, method="min").astype(int)
)
ranking.loc[4:8, ["title", "overall", "rank", "rank_tied"]]
```

```text
                         title  overall  rank  rank_tied
4                     Portal 2      8.2     5          5
5                   Elden Ring      8.1     6          6
6           Grand Theft Auto V      8.1     7          6
7               Stardew Valley      8.1     8          6
8  The Elder Scrolls V: Skyrim      7.9     9          9
```

Three games share the 8.1, and with `method="min"` they all get the 6th place (`rank` returns floats, since other methods give the average position to ties: hence `astype(int)`). Remember that slices with `loc` include their end, the label 8, while slices with `iloc` don't, as in Python.

From now on, we'll use the ranking built by vgrank itself, which has the same `overall` and `rank` columns:

```python
from vgrank.collection import GameCollection

ranking = GameCollection().load().ranking()
```

## Cleaning Data

In the Data I/O lecture, we scraped the list of the best-selling video games from Wikipedia, and its values came with footnotes. Let's clean it: at the end, we'll use it to find out which of my games are best-sellers. Once again:

```python
from io import StringIO

import requests

HEADERS = {"User-Agent": "vgrank (student project; your.name@example.com)"}
URL = "https://en.wikipedia.org/wiki/List_of_best-selling_video_games"

response = requests.get(URL, headers=HEADERS, timeout=10)
response.raise_for_status()
best_selling = pd.read_html(StringIO(response.text), match="Sales")[0]
best_selling.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 51 entries, 0 to 50
Data columns (total 9 columns):
 #   Column            Non-Null Count  Dtype
---  ------            --------------  -----
 0   Rank              51 non-null     str
 1   Title             51 non-null     str
 2   Sales (millions)  51 non-null     float64
 3   Series            43 non-null     str
 4   Platform(s)       51 non-null     str
 5   Release year      51 non-null     str
 6   Developer(s)      51 non-null     str
 7   Publisher(s)      51 non-null     str
 8   Ref.              51 non-null     str
dtypes: float64(1), str(8)
memory usage: 7.6 KB
```

```python
best_selling[["Rank", "Title", "Release year"]].head()
```

```text
   Rank                  Title Release year
0  1[a]                 Tetris      1988[c]
1  1[a]              Minecraft      2011[d]
2     2     Grand Theft Auto V         2013
3     3  Red Dead Redemption 2         2018
4     4         Wii Sports [b]         2006
```

Three problems: the names of the columns, with spaces and parentheses, are hard to type; `Rank` and `Release year` should be numbers, but they are text, because of the footnotes (`1[a]`, `1988[c]`), which also appear in the titles (`Wii Sports [b]`); `Series` has 8 missing values. Wikipedia pages change, so your table may be slightly different.

#### Names and Types

```python
sales = best_selling.rename(
    columns={
        "Rank": "rank",
        "Title": "title",
        "Sales (millions)": "sales_millions",
        "Series": "series",
        "Release year": "year",
    }
)
sales = sales[["rank", "title", "sales_millions", "series", "year"]]
sales["year"].astype(int)
```

```text
ValueError: invalid literal for int() with base 10: '1988[c]'
```

`astype` converts a column to another type, and fails on the first value that it can't convert. That's good: it tells you that something must be cleaned first.

#### Text

The `.str` accessor applies the methods of strings to all the values of a column: `lower`, `strip`, `startswith`, `split`, slicing (`str[:4]`), and many others. Some of them, like `replace`, `contains`, `extract` and `split`, also accept **regular expressions**: patterns that describe a family of strings. `\[\w+\]` describes a footnote: an opening bracket (`\[`, since a bare `[` has a special meaning), one or more letters or digits (`\w+`), and a closing bracket.

```python
FOOTNOTE = r"\[\w+\]"  # e.g. [a], [c], [17]

for column in ["rank", "title", "year"]:
    sales[column] = sales[column].str.replace(FOOTNOTE, "", regex=True).str.strip()

sales["rank"] = sales["rank"].astype(int)
sales["year"] = sales["year"].astype(int)
sales.head()
```

```text
   rank                  title  sales_millions            series  year
0     1                 Tetris           520.0               NaN  1988
1     1              Minecraft           425.0         Minecraft  2011
2     2     Grand Theft Auto V           230.0  Grand Theft Auto  2013
3     3  Red Dead Redemption 2            87.0          Red Dead  2018
4     4             Wii Sports            82.9               Wii  2006
```

`strip` removes the space that preceded some footnotes, as in `Wii Sports [b]`. Write regular expressions as **raw strings**, with an `r` before the quotes: in a raw string, `\` is just a character, while in a normal string it starts an escape sequence, like `\n`. Test your patterns on a few examples first, e.g. on [regex101](https://regex101.com/) (with the *Python* flavor).

> **NOTE**: `str.replace` treats the pattern as plain text, unless you pass `regex=True`. When some values can't be converted, `pd.to_numeric(column, errors="coerce")` turns them into `NaN` instead of raising an error. Handy, but count how many `NaN` you created: each one is a value that you lost.

#### Missing Values

```python
sales.isna().sum()
```

```text
rank              0
title             0
sales_millions    0
series            8
year              0
dtype: int64
```

Eight games don't belong to a series, like Tetris and Stardew Valley. There's no universal rule for missing values: you can drop the rows (`dropna`), fill the values (`fillna`), or keep them, depending on what they mean and on what you'll do with them (remember the Metacritic example of the lecture on coding with LLMs). Here, a game without a series is a series of its own: `fillna` with a Series fills each missing value with the value with the same label, i.e. the title of the same game. This way, the total sales of each series, computed with `groupby` (see [Grouping](#grouping)), include the games without a series, which `groupby` would otherwise leave out:

```python
sales["series"] = sales["series"].fillna(sales["title"])
sales.groupby("series")["sales_millions"].sum().nlargest(3)
```

```text
series
Tetris              520.0
Minecraft           425.0
Grand Theft Auto    282.5
Name: sales_millions, dtype: float64
```

Two variants: `dropna(subset=["metacritic"])` drops the rows with a missing value in some columns only (as `compare_with_metacritic` does in `vgrank/analysis.py`), and `fillna({"series": "none", "year": 0})` fills each column with its own value.

> **NOTE**: a missing value in a column of integers turns the whole column into floats: in the books of goodbooks-10k, `original_publication_year` is `2008.0`, `1997.0`, and so on. The nullable types, `"Int64"` (with a capital *I*) and `"boolean"`, keep their type with missing values: `books["original_publication_year"].astype("Int64")`.

#### Duplicates and Checks

Duplicated rows are common, e.g. when two pages of an API overlap: `duplicated()` marks the rows that already appeared, and `drop_duplicates()` removes them; both accept `subset=` to compare only some columns. When you are done cleaning, check that the data are what you expect, so that a wrong assumption stops your code, instead of silently changing your results:

```python
if not sales["title"].is_unique:
    raise ValueError("Some titles appear more than once")
if not (sales["sales_millions"] > 0).all():
    raise ValueError("Sales must be positive")
```

## Copy-on-Write

What happens to a table when you modify a part of it? pandas follows a mechanism called **Copy-on-Write**, with a simple rule: a DataFrame or a Series obtained from another one (a selection, a filter, the result of a method) behaves as a **copy**, and modifying it never modifies the original. Under the hood, pandas copies the data only when one of the two is modified: hence the name.

```python
classics = games[games["metacritic"] >= 95]
classics["story"] = 0  # changes classics, but not games
```

This has two consequences.

#### Assign in One Step

Selecting some rows, and then assigning to the selection, is called **chained assignment**: it never works, since it modifies a temporary copy.

```python
# Don't: the filter creates a copy, which is modified, and then thrown away
games[games["title"] == "Minecraft"]["story"] = 3
```

```text
ChainedAssignmentError: A value is being set on a copy of a DataFrame or Series through chained assignment.
Such chained assignment never works to update the original DataFrame or Series, because the intermediate object on which we are setting values always behaves as a copy (due to Copy-on-Write).

Try using '.loc[row_indexer, col_indexer] = value' instead, to perform the assignment in a single step.
```

It's a warning, not an error: your code goes on, and Minecraft keeps its old score. Select the rows and the column in a single step, with `loc`:

```python
# Do: rows and column together
games.loc[games["title"] == "Minecraft", "story"] = 3
```

#### Methods Return a New Object

For the same reason, `inplace=True` doesn't work on a column, as we saw in the lecture on coding with LLMs:

```python
# Don't: fillna works on a copy of the column (another ChainedAssignmentError)
games["metacritic"].fillna(0, inplace=True)

# Do: assign the result back to the column
games["metacritic"] = games["metacritic"].fillna(0)
```

Most methods (`fillna`, `dropna`, `rename`, `sort_values`, `drop_duplicates`, ...) leave the original unchanged, and return a new object: assign it to a variable. Avoid `inplace=True` altogether: it's not faster, and it makes the code harder to follow.

This matters for your functions too. A function that receives a DataFrame receives the same object as the caller, not a copy (as with lists), and assigning a column modifies the DataFrame itself: a function that adds a column to its input modifies the DataFrame of the caller. That's why `add_overall_column` in `vgrank/scoring.py` starts with `result = games_df.copy()`. Alternatively, `assign` returns a new DataFrame, with the extra columns:

```python
def add_year(games_df):
    """Return a copy of games_df, with the release year as an extra column."""
    return games_df.assign(year=games_df["released"].str[:4].astype(int))
```

## Transforming Columns

Transformations create new columns from existing ones. In pandas, they work on whole columns at once, without loops.

#### Dates

Dates read from a file are strings. `pd.to_datetime` converts them into dates (dtype `datetime64`), which can be compared, subtracted, and split into their parts with the `.dt` accessor:

```python
released = pd.to_datetime(ranking["released"])

pd.DataFrame(
    {
        "title": ranking["title"],
        "year": released.dt.year,
        "weekday": released.dt.day_name(),
        "days_old": (pd.Timestamp("2026-11-05") - released).dt.days,
    }
).head(3)
```

```text
                      title  year   weekday  days_old
0                     Hades  2020  Thursday      2240
1             Hollow Knight  2017  Thursday      3542
2  The Witcher 3: Wild Hunt  2015    Monday      4189
```

Dates can also be compared with strings, like `released < "2012-01-01"`. A value that is not a date raises an error: `errors="coerce"` turns it into `NaT` (*Not a Time*, the missing value of dates), as `games_per_year` does in `vgrank/analysis.py`, since the release date of a game may be unknown. And beware of the format:

```python
pd.to_datetime(pd.Series(["05/11/2026"]))  # May 11th, 2026!
pd.to_datetime(pd.Series(["05/11/2026"]), format="%d/%m/%Y")  # November 5th, 2026
```

ISO dates (`2026-11-05`) are unambiguous. For any other format, pass it with `format` (or use `dayfirst=True`): otherwise, pandas reads `05/11` in the American way, month first, without any warning.

#### From Numbers to Categories

`pd.cut` splits the values of a column into intervals, with a label for each:

```python
verdict = pd.cut(
    ranking["overall"],
    bins=[0, 7, 8, 9, 10],
    labels=["ok", "good", "great", "masterpiece"],
)
verdict.value_counts(sort=False)
```

```text
overall
ok             4
good           4
great          7
masterpiece    1
Name: count, dtype: int64
```

Each interval includes its right end: (8, 9] is *great*, so Hollow Knight, with 9.0, is not a masterpiece (`right=False` includes the left ends instead). `pd.qcut` creates intervals with the same number of values, e.g. quartiles. The result has the `category` dtype: a fixed and ordered set of values, which `value_counts(sort=False)`, sorting and plots keep in their order, and which uses less memory than text (see [Large Datasets](#large-datasets)).

To replace some values with others, `map` takes a dictionary (e.g., `{"PS4": "PlayStation 4"}`), and returns `NaN` for the values that are not in the dictionary, while `replace` leaves them as they are.

#### Avoid Loops

`apply` calls a Python function on each value of a column (or, with `axis=1`, on each row of a DataFrame): it works with any function, but it's a loop in disguise. Let's compute the overall score of 100,000 games (the 16 games of the ranking, repeated 6,250 times with `concat`, which we'll see [later](#concatenate)), with `apply` and with the vectorized operations of [Alignment](#alignment):

```python
import time

many_games = pd.concat([ranking[CATEGORIES]] * 6250, ignore_index=True)  # 100,000 rows

start = time.perf_counter()
slow = many_games.apply(lambda row: (row * weights).sum(), axis=1)
print(f"apply: {time.perf_counter() - start:.3f} s")

start = time.perf_counter()
fast = (many_games * weights).sum(axis=1)
print(f"vectorized: {time.perf_counter() - start:.3f} s")
```

```text
apply: 2.478 s
vectorized: 0.007 s
```

The results are the same, but the vectorized version is hundreds of times faster (on my laptop, a loop with `iterrows` takes more than 10 seconds). Vectorized operations run their loop in compiled code: that's NumPy, the topic of the next lecture. Before writing `apply`, or a `for` loop on the rows, look for a method that does the same on the whole column: arithmetic and comparisons, `.str` and `.dt`, `cut`, `map`, `where`, `clip`, `groupby`... and keep `apply` for what no method can do, on small tables. vgrank, for instance, uses `iterrows` in `GameCollection.load`, where each row becomes a `Game` object: with a few dozen games, it doesn't matter.

#### Method Chaining

Since each method returns a new DataFrame, methods can be chained, one per line, inside parentheses: the chain reads like a recipe, from top to bottom, without intermediate variables.

```python
recent_best = (
    ranking.assign(year=released.dt.year)
    .query("year >= 2015")
    .nlargest(3, "overall")
    .loc[:, ["title", "year", "overall"]]
)
recent_best
```

```text
                      title  year  overall
0                     Hades  2020      9.3
1             Hollow Knight  2017      9.0
2  The Witcher 3: Wild Hunt  2015      8.9
```

The intermediate DataFrames of a chain have no name, so how can a step refer to a column created by a previous step? `query` accepts the names of the columns; `assign` and `loc` accept a function, like `lambda df: df["year"] >= 2015`, which receives the DataFrame of the current step; `pd.col("year")` does the same, and it's easier to read:

```python
(
    ranking.assign(year=pd.col("released").str[:4].astype(int))
    .loc[pd.col("year") >= 2015]
    .assign(vs_critics=pd.col("overall") - pd.col("metacritic") / 10)
    .loc[:, ["title", "year", "overall", "vs_critics"]]
    .head(3)
)
```

```text
                      title  year  overall  vs_critics
0                     Hades  2020      9.3         0.0
1             Hollow Knight  2017      9.0         0.2
2  The Witcher 3: Wild Hunt  2015      8.9        -0.3
```

`pipe` puts your own functions in the chain: `df.pipe(f, x=1)` is the same as `f(df, x=1)`, e.g. `ranking.pipe(analysis.score_by_genre, min_games=3)`. In a project, the preparation of a dataset can become a chain of small functions of your package, each with a docstring: `load_books().pipe(clean_books).pipe(add_decade)`. Don't overdo it, though: a long chain is hard to debug, since you can't see its intermediate results.

## Grouping

`groupby` splits the rows into groups, applies a function to each group, and combines the results (*split-apply-combine*). For the genres, there is a problem: the `genres` column contains all the genres of a game in one string, like `"Action, RPG"`, and grouping by it would create a group for each combination of genres.

#### One Row per Value

```python
genres = ranking.assign(genre=ranking["genres"].str.split(", ")).explode("genre")
genres[["title", "genre", "overall"]].head(5)
```

```text
           title       genre  overall
0          Hades      Action      9.3
0          Hades         RPG      9.3
0          Hades       Indie      9.3
1  Hollow Knight  Platformer      9.0
1  Hollow Knight       Indie      9.0
```

`str.split` turns each string into a list, and `explode` creates a row for each element of the list, repeating the other columns and the index labels: the 16 games become 36 (game, genre) pairs. It's the first step of `score_by_genre` in `vgrank/analysis.py`, which we met in the lecture on coding with LLMs.

#### Aggregating

`agg` with **named aggregations** computes several statistics at once, each with the name of its column: `name=(column, function)`.

```python
genre_stats = (
    genres.groupby("genre")
    .agg(
        n_games=("title", "size"),
        mean_overall=("overall", "mean"),
        best_game=("title", "first"),
    )
    .sort_values("mean_overall", ascending=False)
    .round(2)
)
genre_stats
```

```text
            n_games  mean_overall              best_game
genre
Platformer        2          8.40          Hollow Knight
Indie             4          8.30                  Hades
RPG               6          8.28                  Hades
Puzzle            1          8.20               Portal 2
Action           12          7.92                  Hades
Shooter           3          7.57               Portal 2
Adventure         6          7.43  Red Dead Redemption 2
Simulation        2          7.25         Stardew Valley
```

`first` takes the first value of each group: `groupby` keeps the order of the rows, and `ranking` is sorted by score, so it's the best game of the genre. `size` counts the rows, while `count` counts the values that are not missing. The averages of one or two games mean little: `genre_stats[genre_stats["n_games"] >= 3]` keeps the others, like the `min_games` parameter of `score_by_genre`. You can also aggregate many columns at once: `genres.groupby("genre")[CATEGORIES].mean()` gives the average score of each genre in each category.

#### Transforming within Groups

`agg` returns one row per group; `transform` returns one value per row, with the original index: the result of its group. It's what you need to compare each row with its group:

```python
genres["genre_mean"] = genres.groupby("genre")["overall"].transform("mean").round(2)
genres["vs_genre"] = (genres["overall"] - genres["genre_mean"]).round(2)
genres.loc[genres["genre"] == "RPG", ["title", "overall", "genre_mean", "vs_genre"]]
```

```text
                          title  overall  genre_mean  vs_genre
0                         Hades      9.3        8.28      1.02
2      The Witcher 3: Wild Hunt      8.9        8.28      0.62
5                    Elden Ring      8.1        8.28     -0.18
7                Stardew Valley      8.1        8.28     -0.18
8   The Elder Scrolls V: Skyrim      7.9        8.28     -0.38
11               Cyberpunk 2077      7.4        8.28     -0.88
```

Other methods work within each group too: `groupby("genre")["overall"].rank()` ranks the games of each genre, `groupby("genre").head(1)` keeps the first row of each genre, and `shift`, `diff` and `cumsum` start over in each group (e.g., the difference between consecutive laps of each driver, in *Box, Box*). `filter` keeps or drops whole groups: `genres.groupby("genre").filter(lambda group: len(group) >= 3)` keeps the rows of the genres with at least 3 games.

#### Several Keys

With a list of columns, the groups are their combinations, and the result has an index with several levels, a `MultiIndex`. `unstack` moves the last level of the index to the columns:

```python
genres["decade"] = genres["released"].str[:3] + "0s"  # "2015-05-18" -> "2010s"
genres.groupby(["genre", "decade"])["overall"].mean().unstack().round(2)
```

```text
decade      2000s  2010s  2020s
genre
Action        6.9   7.92   8.27
Adventure     6.9   7.54    NaN
Indie         NaN   7.97   9.30
Platformer    NaN   8.40    NaN
Puzzle        NaN   8.20    NaN
RPG           NaN   8.30   8.27
Shooter       NaN   7.57    NaN
Simulation    NaN   7.25    NaN
```

The combinations without games are `NaN`, and those with a single game (Hades is my only Indie game of the 2020s) deserve the same caution as above.

## Reshaping

#### Wide and Long

`ranking` is in **wide** format: one row per game, one column per category. Many tools prefer the **long** format, with one row per observation (the score of a game in a category): `groupby`, and plotting libraries like Plotly Express and seaborn (see the visualisation lecture). `melt` converts from wide to long, as `scores_long_format` does in `vgrank/analysis.py`:

```python
long_scores = ranking.melt(
    id_vars=["title"], value_vars=CATEGORIES, var_name="category", value_name="score"
)
long_scores.head(3)  # 16 games x 5 categories = 80 rows
```

```text
                      title  category  score
0                     Hades  gameplay     10
1             Hollow Knight  gameplay     10
2  The Witcher 3: Wild Hunt  gameplay      9
```

In long format, the category is a value, like the score: a single `groupby("category")` computes the statistics of all the categories, and a single plot draws all of them, like the box plot of the vgrank app. `pivot` goes back to wide:

```python
long_scores.pivot(index="title", columns="category", values="score").head(3)
```

```text
category        gameplay  graphics  replayability  sound  story
title
Cyberpunk 2077         7         9              5      9      8
DOOM (2016)            9         8              7     10      4
Don't Starve           7         8              8      7      4
```

`pivot` wants a single value for each pair (index, column), and raises `ValueError: Index contains duplicate entries, cannot reshape` otherwise. `pivot_table` aggregates the duplicates instead (with the mean, by default): `genres.pivot_table(index="genre", columns="decade", values="overall")` gives the table of the previous section. With `aggfunc="sum"`, it builds the voting matrix of *Douze Points*: the points that each country gave to each other country, over the years.

#### One Column per Value

Another way to handle multiple genres is **one-hot encoding**: a column for each genre, with 1 if the game has it, and 0 otherwise. `str.get_dummies` creates it from the strings:

```python
flags = ranking["genres"].str.get_dummies(sep=", ")
flags.head(3)
```

```text
   Action  Adventure  Indie  Platformer  Puzzle  RPG  Shooter  Simulation
0       1          0      1           0       0    1        0           0
1       1          0      1           1       0    0        0           0
2       1          0      0           0       0    1        0           0
```

`flags.sum()` counts the games of each genre, and the columns make the filters easy. The ranking page of the vgrank app (`pages/ranking.py`) filters the genres with `apply` and a function, `has_any_genre`; this is the vectorized version:

```python
wanted = ["RPG", "Indie"]
ranking.loc[flags[wanted].any(axis=1), "title"]  # the games with at least one of them
```

```text
0                           Hades
1                   Hollow Knight
2        The Witcher 3: Wild Hunt
5                      Elden Ring
7                  Stardew Valley
8     The Elder Scrolls V: Skyrim
11                 Cyberpunk 2077
14                   Don't Starve
Name: title, dtype: str
```

`any(axis=1)` is `True` for the rows with at least one value that is not zero (`all(axis=1)` would want both genres).

## Combining Tables

#### Merge

`merge` joins two tables, matching the values of one or more columns, the **keys** (like a `JOIN` in SQL). Let's add the sales of the best-sellers to my ranking:

```python
with_sales = ranking.merge(
    sales[["title", "sales_millions"]], on="title", how="left", validate="one_to_one"
)
with_sales["sales_millions"].notna().sum()  # 8
```

`how="left"` keeps all the rows of the left table, `ranking`, with `NaN` where `sales` has no match:

| `how` | Keeps |
|---|---|
| `"inner"` (the default) | only the keys that are in both tables |
| `"left"` | all the rows of the left table |
| `"right"` | all the rows of the right table |
| `"outer"` | all the rows of both tables |

So, 8 of my games are best-sellers. Or are they? Red Dead Redemption 2 is the fourth game of the Wikipedia list... With `indicator=True`, `merge` adds a column, `_merge`, which says where each row comes from: `"both"`, `"left_only"` or `"right_only"`.

```python
check = ranking.merge(sales, on="title", how="outer", indicator=True)
check.loc[check["_merge"] == "left_only", "title"].tolist()
```

```text
['DOOM (2016)', "Don't Starve", 'Hades', 'Hollow Knight', 'Ori and the Blind Forest', 'Portal 2', 'Prey (2017)', 'Red Dead Redemption 2']
```

```python
sales.loc[sales["title"].str.startswith("Red Dead"), "title"].tolist()
```

```text
['Red Dead Redemption\xa02']
```

The character before the 2 is not a space: it's `\xa0`, a *non-breaking space*, which Wikipedia uses to keep the 2 on the same line as the rest of the title. It looks the same, but it's a different character, and the keys of a merge must be identical. `str.normalize("NFKC")` converts these special characters (and other variants, like full-width letters) into their standard form:

```python
sales["title"] = sales["title"].str.normalize("NFKC")

with_sales = ranking.merge(
    sales[["title", "sales_millions"]], on="title", how="left", validate="one_to_one"
)
with_sales.loc[
    with_sales["sales_millions"].notna(), ["rank", "title", "overall", "sales_millions"]
]
```

```text
    rank                        title  overall  sales_millions
2      3     The Witcher 3: Wild Hunt      8.9            65.0
3      4        Red Dead Redemption 2      8.6            87.0
5      6                   Elden Ring      8.1            30.0
6      7           Grand Theft Auto V      8.1           230.0
7      8               Stardew Valley      8.1            50.0
8      9  The Elder Scrolls V: Skyrim      7.9            65.0
11    12               Cyberpunk 2077      7.4            40.0
13    14          Grand Theft Auto IV      6.9            25.0
15    16                    Minecraft      6.4           425.0
```

When the keys are text, normalise them on both sides before merging (spaces, special characters, upper and lower case), and always check how many rows matched. Even better, use identifiers, when both tables have them: that's why vgrank stores the `game_id` of each game on RAWG.

`validate` checks the relationship between the keys. With `"one_to_one"`, each title must appear at most once in each table: without this check, a title repeated in `sales` would silently duplicate a game of the ranking, and a sum or an average would count it twice. When the left table repeats its keys on purpose, like `genres`, with a row for each genre of a game, use `"many_to_one"`: with `"one_to_one"`, `merge` raises `MergeError: Merge keys are not unique in left dataset; not a one-to-one merge`.

Two more details: when the two tables have other columns with the same name, `merge` adds the suffixes `_x` and `_y` to them (choose better ones, with `suffixes=("_mine", "_wikipedia")`); to match on the index instead of a column, use `left_index=True` and `right_index=True`, or `join`.

So, do I like best-sellers?

```python
with_sales["best_seller"] = with_sales["sales_millions"].notna()
with_sales.groupby("best_seller")["overall"].agg(["size", "mean"]).round(2)
```

```text
             size  mean
best_seller
False           7  7.94
True            9  7.82
```

Slightly less than the others: my top two, Hades and Hollow Knight, are not on the list, and neither is Portal 2. With 9 and 7 games, though, the difference means little.

#### Concatenate

`concat` stacks tables with the same columns, one below the other (or, with `axis=1`, side by side, aligned on the index). It's what you need when your data come in pieces: one file per year, one API request per city, or per page. Let's download the weather of 2025 in three cities, from the Open-Meteo API of the Data I/O lecture:

```python
ARCHIVE_URL = "https://archive-api.open-meteo.com/v1/archive"
CITIES = {"Milan": (45.46, 9.19), "Rome": (41.89, 12.48), "Palermo": (38.12, 13.36)}

frames = []
for city, (latitude, longitude) in CITIES.items():
    params = {
        "latitude": latitude,
        "longitude": longitude,
        "start_date": "2025-01-01",
        "end_date": "2025-12-31",
        "daily": "temperature_2m_max,precipitation_sum",
        "timezone": "Europe/Rome",
    }
    response = requests.get(ARCHIVE_URL, params=params, timeout=10)
    response.raise_for_status()
    frame = pd.DataFrame(response.json()["daily"])
    frame["city"] = city
    frames.append(frame)

weather = pd.concat(frames, ignore_index=True)
weather.shape  # (1095, 4): 365 days x 3 cities
```

Collect the pieces in a list, and concatenate them once, at the end: `concat` copies all the data every time, so calling it inside the loop gets slower and slower as the table grows. `ignore_index=True` numbers the rows again, from 0: otherwise, each piece keeps its own labels (0 to 364), and the index has duplicates. The `city` column keeps track of where each row comes from. Rows work in the same way: to build a table one row at a time, collect dictionaries in a list, and create the DataFrame at the end, as `to_dataframe` does in `vgrank/collection.py`.

## Time Series

Weather, energy, lap times: many datasets of the project proposals are **time series**, values recorded over time. With dates in the index, pandas offers a few more tools:

```python
weather["time"] = pd.to_datetime(weather["time"])
temperature = weather.pivot(index="time", columns="city", values="temperature_2m_max")
temperature.loc["2025-07-01":"2025-07-05"]
```

```text
city        Milan  Palermo  Rome
time
2025-07-01   35.2     31.2  36.8
2025-07-02   34.4     31.8  36.7
2025-07-03   35.1     32.8  38.3
2025-07-04   36.1     33.2  37.7
2025-07-05   29.8     32.8  36.1
```

The heatwave of the Data I/O lecture, in three cities. With dates in the index (a `DatetimeIndex`), `loc` accepts dates as strings, even partial ones: `temperature.loc["2025-07"]` selects the whole month of July.

`resample` is a `groupby` on periods of time:

```python
temperature.resample("ME").mean().round(1)
```

```text
city        Milan  Palermo  Rome
time
2025-01-31    7.5     16.3  14.0
2025-02-28    9.7     15.5  14.8
2025-03-31   14.7     18.2  17.0
2025-04-30   19.5     19.4  20.1
2025-05-31   22.9     23.1  23.8
2025-06-30   31.5     29.3  33.9
2025-07-31   29.4     31.7  33.0
2025-08-31   28.8     30.7  32.8
2025-09-30   23.5     28.2  27.7
2025-10-31   18.0     23.6  22.6
2025-11-30   11.7     19.3  16.8
2025-12-31    9.0     17.0  13.7
```

`"ME"` (*month end*) groups by month, and labels each month with its last day; `"W"` groups by week, `"QE"` by quarter, `"YE"` by year. Any aggregation works: the total precipitation of each month would be a `resample("ME").sum()`.

`rolling` computes a statistic on a moving window of rows:

```python
temperature.rolling(7).mean().loc["2025-07-01":"2025-07-05"].round(1)
```

```text
city        Milan  Palermo  Rome
time
2025-07-01   35.0     31.1  37.0
2025-07-02   35.0     31.2  37.4
2025-07-03   35.1     31.4  37.8
2025-07-04   35.5     31.7  37.7
2025-07-05   34.6     32.0  37.3
```

Each value is the average of that day and of the 6 days before it: the weekly average smooths the daily noise, and shows the trend (the first 6 days of the year are `NaN`, since their window is incomplete). With a `DatetimeIndex`, the window can also be a duration, `rolling("7D")`, which works even when some days are missing. Other useful methods are `diff()`, the difference from the previous row, `shift()`, the values of the previous row, and `pct_change()`, the relative change. And comparisons work as usual:

```python
(temperature >= 30).sum()  # days with a maximum of at least 30 °C
```

```text
city
Milan      42
Palermo    57
Rome       88
dtype: int64
```

## Large Datasets

The 16 games of vgrank fit anywhere, but some datasets of the project proposals are large: goodbooks-10k has 6 million ratings, Food.com 700,000 reviews. pandas keeps the whole table in memory, so choose the types of your columns. The ratings of goodbooks-10k are in the same repository as its books: download them with the `download_file` function of the Data I/O lecture, from `https://raw.githubusercontent.com/zygmuntz/goodbooks-10k/master/ratings.csv` (72 MB).

```python
ratings = pd.read_csv("data/ratings.csv")
ratings.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 5976479 entries, 0 to 5976478
Data columns (total 3 columns):
 #   Column   Dtype
---  ------   -----
 0   user_id  int64
 1   book_id  int64
 2   rating   int64
dtypes: int64(3)
memory usage: 136.8 MB
```

```python
ratings = pd.read_csv(
    "data/ratings.csv",
    dtype={"user_id": "int32", "book_id": "int16", "rating": "int8"},
)
ratings.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 5976479 entries, 0 to 5976478
Data columns (total 3 columns):
 #   Column   Dtype
---  ------   -----
 0   user_id  int32
 1   book_id  int16
 2   rating   int8
dtypes: int16(1), int32(1), int8(1)
memory usage: 39.9 MB
```

By default, integers are `int64`, 8 bytes each, which hold numbers up to about 9 billion billions. A rating from 1 to 5 fits in a single byte: `int8` holds the numbers from -128 to 127, `int16` up to 32,767, `int32` up to about 2 billion. But be careful: a value out of range doesn't raise an error, it wraps around (300, read as `int8`, becomes 44). Check the maximum of each column first (`ratings.max()`), or let pandas choose the smallest type that holds the values, with `pd.to_numeric(ratings["rating"], downcast="integer")`.

A few more tips:

- **Read only what you need**: `usecols=["book_id", "rating"]` reads only some columns, `nrows=1000` only the first rows, for a quick look.
- **Use categories** for the text columns with few distinct values, repeated many times (countries, genres, languages): `dtype={"language_code": "category"}`.
- **Develop on a sample**: `ratings.sample(n=100_000, random_state=42)` takes 100,000 random rows, always the same ones, thanks to the seed (more on randomness in the next lecture). If your final analysis uses a subset of the data, say so in the README.
- **Save the clean data in Parquet**, a compressed binary format that keeps the dtypes: `ratings.to_parquet("data/ratings.parquet")` writes a file of 21 MB, instead of the 72 MB of the CSV, and `pd.read_parquet` reads it in 0.01 seconds, instead of 0.5 (on my laptop). It needs `pip install pyarrow` (already installed with Streamlit), which also speeds up the CSV files: `pd.read_csv(path, engine="pyarrow")`.
- **Process the file in pieces**, when it doesn't fit in memory: `for chunk in pd.read_csv(path, chunksize=1_000_000):` reads the table a million rows at a time, and `chunk` is a DataFrame that you can filter or aggregate, before combining the results.

Beyond pandas, [Polars](https://pola.rs/) and [DuckDB](https://duckdb.org/) handle much larger tables, on a single computer.

## Useful Resources

- [10 minutes to pandas](https://pandas.pydata.org/docs/user_guide/10min.html), a short tour of the basics, from the pandas documentation
- The [User Guide](https://pandas.pydata.org/docs/user_guide/index.html) of pandas, in particular: [Intro to data structures](https://pandas.pydata.org/docs/user_guide/dsintro.html), [Indexing and selecting data](https://pandas.pydata.org/docs/user_guide/indexing.html), [Copy-on-Write](https://pandas.pydata.org/docs/user_guide/copy_on_write.html), [Working with text data](https://pandas.pydata.org/docs/user_guide/text.html), [Group by](https://pandas.pydata.org/docs/user_guide/groupby.html), [Reshaping and pivot tables](https://pandas.pydata.org/docs/user_guide/reshaping.html), [Merge, join, concatenate and compare](https://pandas.pydata.org/docs/user_guide/merging.html), [Time series](https://pandas.pydata.org/docs/user_guide/timeseries.html) and [Scaling to large datasets](https://pandas.pydata.org/docs/user_guide/scale.html)
- [Python for Data Analysis](https://wesmckinney.com/book/), by Wes McKinney, the creator of pandas (free online)
- [Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html), from the Python documentation, and [regex101](https://regex101.com/), to test your patterns
- [`vgrank/analysis.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/analysis.py) and [`vgrank/scoring.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/scoring.py), the pandas code of vgrank
