# Scientific Computing with NumPy

Counting, filtering and grouping take you a long way, and pandas does them well. But some questions need more numerical work: a weighted average across the columns of a table, a standardisation, the distance between two places, a simulation, a statistical test.

NumPy is the foundation of scientific computing in Python: pandas stores its numeric columns in NumPy arrays, and SciPy, scikit-learn and matplotlib are all built on it.

This lecture covers the arrays of NumPy, and how to compute with them without loops: vectorization, indexing, aggregations, broadcasting and the matrix product. Then, random numbers and simulations, and the statistical functions of SciPy.

## Arrays

As in the pandas lecture, the examples run from the root of vgrank (in a notebook, build the paths from `PROJECT_ROOT`), and start from the ranking:

```python
import numpy as np
import pandas as pd

from vgrank.collection import GameCollection
from vgrank.scoring import CATEGORIES, WEIGHTS

ranking = GameCollection().load().ranking()
```

#### From Lists to Arrays

The overall score of a game is the weighted average of its scores in the five categories. With Python lists, it takes a loop:

```python
hades = [10, 8, 9, 9, 10]  # gameplay, story, graphics, sound, replayability
weights = [0.3, 0.2, 0.2, 0.1, 0.2]

total = 0
for score, weight in zip(hades, weights):
    total += score * weight
total  # 9.3
```

Lists don't do arithmetic: `hades * weights` raises `TypeError: can't multiply sequence by non-int of type 'list'`, `hades + hades` joins the two lists, and `hades * 2` repeats the list twice. NumPy arrays do: their operations work **element by element**, without loops.

```python
import numpy as np

hades = np.array([10, 8, 9, 9, 10])
weights = np.array([0.3, 0.2, 0.2, 0.1, 0.2])

hades * weights  # array([3. , 1.6, 1.8, 0.9, 2. ])
(hades * weights).sum()  # np.float64(9.3)
```

`np` is the conventional alias of NumPy, as `pd` is for pandas. NumPy is installed together with pandas but, as we saw in the environments lecture, if your code imports it, list it in your `requirements.txt`.

#### Creating Arrays

The **array** (`ndarray`, for *N-dimensional array*) is the core of NumPy: a grid of values, all of the same type, with any number of dimensions. A DataFrame becomes an array with `to_numpy()`:

```python
scores = ranking[CATEGORIES].to_numpy()  # one row per game, one column per category
scores[:3]
```

```text
array([[10,  8,  9,  9, 10],
       [10,  8,  9, 10,  8],
       [ 9, 10,  9, 10,  7]])
```

```python
scores.shape  # (16, 5): 16 rows (the games) and 5 columns (the categories)
scores.ndim  # 2 dimensions: a matrix
scores.size  # 80 values
scores.dtype  # dtype('int64'): the type of the values
```

The array keeps the values, but not the labels: the titles of the games and the names of the categories are gone. Keep them aside, in the same order:

```python
titles = ranking["title"].to_numpy()
weights = np.array([WEIGHTS[category] for category in CATEGORIES])
```

`weights` is built in the order of `CATEGORIES`, as in `vgrank/scoring.py`: we'll see why it matters in [Broadcasting](#broadcasting). `titles` has the dtype `object`, an array of Python strings: NumPy is made for numbers, and for text pandas is the better tool.

Arrays can also be created from scratch:

```python
np.zeros(5)  # array([0., 0., 0., 0., 0.])
np.ones((2, 3))  # a matrix of ones, with 2 rows and 3 columns: the shape is a tuple
np.full(5, 0.2)  # array([0.2, 0.2, 0.2, 0.2, 0.2]): five equal weights
np.arange(1, 17)  # the integers from 1 to 16: as in range, the end is excluded
np.linspace(0, 10, 5)  # 5 evenly spaced values from 0 to 10, both included
```

`np.arange` numbers the ranking in `rank_games` (`vgrank/scoring.py`), and `np.linspace` creates the points of a curve, for a plot (here, `array([ 0. ,  2.5,  5. ,  7.5, 10. ])`). `reshape` changes the shape and keeps the values (`np.arange(6).reshape(2, 3)` is a matrix with two rows, `0 1 2` and `3 4 5`), while `ravel` puts all the values in one dimension: `scores.ravel()` contains all the 80 scores, e.g. for a histogram.

#### Types

All the values of an array have the same type, its `dtype`: that's what makes arrays compact and fast. When you mix types, NumPy picks one that fits all the values: `np.array([1, 2.5])` is an array of floats, and `np.array([1, "2"])` an array of strings (`array(['1', '2'], dtype='<U21')`), on which arithmetic doesn't work, like the numbers with footnotes of the pandas lecture. `astype` converts an array to another type.

The most common types are `int64` (integers), `float64` (decimal numbers) and `bool` (`True` and `False`). The smaller types of the pandas lecture, like `int8`, come from NumPy, and so does their problem: `np.array([300]).astype(np.int8)` gives `array([44], dtype=int8)`, without any error.

Floats are approximations, as in plain Python:

```python
0.1 + 0.2  # 0.30000000000000004
0.1 + 0.2 == 0.3  # False
np.isclose(0.1 + 0.2, 0.3)  # np.True_
```

Don't compare floats with `==`: `np.isclose`, or `np.allclose` for whole arrays, allow a tiny tolerance. For instance, `np.isclose(weights.sum(), 1)` checks that the weights sum to 1.

> **NOTE**: the single values of an array, and the results of aggregations, show their type: `np.float64(9.3)`, `np.int64(8)`, `np.True_`. They behave like Python numbers, with one exception that you'll meet: the `json` module can't write a `np.int64` (`TypeError: Object of type int64 is not JSON serializable`). Convert them with `int()` and `float()`, as `overall_score` does in `vgrank/scoring.py` (`round(float(...), 2)`), or a whole array with `.tolist()`.

## Vectorization

#### Operations on Whole Arrays

Arithmetic and comparisons work element by element, between an array and a number, or between two arrays with the same shape. Let's compare my scores with the critics', as `compare_with_metacritic` does in `vgrank/analysis.py`:

```python
overall = ranking["overall"].to_numpy()
metacritic = ranking["metacritic"].to_numpy()

difference = overall - metacritic / 10  # both from 0 to 10
difference
```

```text
array([ 0. ,  0.2, -0.3, -1. , -1.3, -1.5, -1.1, -0.8, -1.5, -1. , -1. ,
       -1.2, -0.9, -2.6, -1.1, -1.9])
```

```python
difference > 0  # an array of booleans: True where I'm more generous than the critics
(difference > 0).sum()  # np.int64(1): True counts as 1, False as 0
difference.mean().round(2)  # np.float64(-1.06)
np.abs(difference).max()  # np.float64(2.5999999999999996): 2.6, for a float
```

I'm harsher than the critics: only Hollow Knight gets more from me than from them, and on average I give one point less.

NumPy has its own version of the mathematical functions, which work on whole arrays, element by element (they are called **universal functions**, or *ufuncs*): `np.abs`, `np.round`, `np.sqrt`, `np.exp`, `np.log`, `np.log10`, `np.sin`, `np.cos`, `np.minimum`, `np.maximum`, `np.clip`, and many others. The functions of the `math` module only work on single numbers: `math.sqrt(np.array([4, 9]))` raises `TypeError: only 0-dimensional arrays can be converted to Python scalars`, while `np.sqrt(np.array([4, 9]))` gives `array([2., 3.])`.

#### Formulas

With universal functions, a formula written for single numbers works for whole arrays too. Several project proposals need the distance between two places, from their coordinates: between cities (*Weather Report*), countries (*Around the Globe Quiz*) or airports (*Island Hopper*). On the surface of the Earth, it's given by the *haversine formula*:

```python
EARTH_RADIUS_KM = 6371


def distance_km(lat1, lon1, lat2, lon2):
    """Great-circle distance, in km, between points given in degrees."""
    lat1, lon1 = np.radians(lat1), np.radians(lon1)
    lat2, lon2 = np.radians(lat2), np.radians(lon2)
    a = (
        np.sin((lat2 - lat1) / 2) ** 2
        + np.cos(lat1) * np.cos(lat2) * np.sin((lon2 - lon1) / 2) ** 2
    )
    return 2 * EARTH_RADIUS_KM * np.arcsin(np.sqrt(a))
```

The function works with single numbers, which makes it easy to test (Milan–Rome is about 480 km), and with arrays, e.g. with the cities of the pandas lecture:

```python
distance_km(45.46, 9.19, 41.89, 12.48)  # np.float64(476.9677728322604)

latitude = np.array([45.46, 41.89, 38.12])  # Milan, Rome, Palermo
longitude = np.array([9.19, 12.48, 13.36])
distance_km(45.46, 9.19, latitude, longitude).round()  # from Milan to each city
```

```text
array([  0., 477., 886.])
```

The coordinates of Milan are single numbers, and NumPy uses them with each element of the arrays: it's broadcasting, which we'll see below.

#### Why It's Fast

In the pandas lecture, `apply` took seconds, where the vectorized version took milliseconds. Let's compare NumPy with a Python loop, on the overall scores of a million ratings:

```python
import time

many_scores = np.tile(scores, (62_500, 1))  # the 16 games, repeated: 1,000,000 rows
many_rows = many_scores.tolist()  # the same data, as a list of lists
weight_list = weights.tolist()

start = time.perf_counter()
slow = []
for row in many_rows:
    total = 0
    for score, weight in zip(row, weight_list):
        total += score * weight
    slow.append(total)
print(f"loop: {time.perf_counter() - start:.3f} s")

start = time.perf_counter()
fast = (many_scores * weights).sum(axis=1)  # axis=1: the sum of each row
print(f"vectorized: {time.perf_counter() - start:.3f} s")
```

```text
loop: 0.451 s
vectorized: 0.014 s
```

The results are the same, but the vectorized version is about 30 times faster (on my laptop). Three reasons:

- **One type.** A list can contain anything, so, for each element, Python checks its type and looks for the right operation; an array has a single dtype, checked once.
- **Contiguous memory.** The values of an array are stored one after the other, as raw numbers; a list stores references to Python objects, scattered in memory.
- **Compiled loops.** The loop over the elements still happens, but in compiled C code, instead of Python.

The speed comes only from the operations on whole arrays: a `for` loop over the elements of an array is as slow as a loop over a list (actually, even slower). It's the same rule as in pandas: when you're about to write a loop over the values of an array, look for a function that does the same on the whole array.

## Indexing

#### Positions and Slices

Indexing works as with lists, with a position (or a slice) for each dimension, separated by commas:

```python
scores[0]  # the first row, Hades: array([10,  8,  9,  9, 10])
scores[0, 1]  # row 0, column 1: the story score of Hades, np.int64(8)
scores[-1]  # the last row: Minecraft
scores[:3, :2]  # the first 3 rows, and the first 2 columns
scores[:, 0]  # all the rows, column 0: the gameplay score of every game
```

It's like `iloc` in pandas: an array has positions, not labels. To select a category by its name, find its position first: `scores[:, CATEGORIES.index("story")]`.

#### Views and Copies

A slice doesn't copy the data: it's a **view** on the same memory as the original array. This makes slices fast, even on huge arrays, but modifying a view modifies the original array. Let's try:

```python
podium = scores[:3]
podium[0, 0] = 0
```

```text
ValueError: assignment destination is read-only
```

`to_numpy()` may also return a view, which shares its data with the DataFrame. Such arrays are read-only: otherwise, modifying the array would modify the DataFrame, against the rules of Copy-on-Write. `copy()` creates an independent array, with its own data (`to_numpy(copy=True)` does the same, straight away):

```python
my_scores = scores.copy()
podium = my_scores[:3]  # a view on the first 3 rows of my_scores
podium[0, 0] = 0
my_scores[0]  # array([ 0,  8,  9,  9, 10]): the original changed too!
```

It's the opposite of pandas, where a selection behaves as a copy. When you want to modify a part of an array without touching the original, take a copy: `scores[:3].copy()`. Selections with masks and with lists of positions (see below) always return copies.

#### Boolean Masks

Filters work as in pandas: a comparison gives an array of booleans (a **mask**), which selects the elements where it's `True`.

```python
gameplay = scores[:, 0]
story = scores[:, 1]

titles[gameplay == 10]
```

```text
array(['Hades', 'Hollow Knight', 'Portal 2'], dtype=object)
```

```python
titles[(gameplay >= 9) & (story <= 6)]  # great gameplay, weak story
```

```text
array(['Elden Ring', 'Stardew Valley', 'DOOM (2016)'], dtype=object)
```

The rules are the same as in pandas: `&`, `|` and `~`, with each condition in parentheses. A mask on the rows selects whole rows (`scores[gameplay == 10]` contains the 3 rows of those games), while a mask with the same shape as the array selects single values, and returns them in one dimension: `scores[scores < 5]` gives `array([4, 4, 2])`, the scores below 5.

`np.where(condition, a, b)` chooses element by element: `a` where the condition is `True`, `b` elsewhere, e.g. `np.where(gameplay == 10, "must play", "")`, a label for each game. And `np.isin` checks which elements are in a list: `np.isin(titles, ["Hades", "Celeste"])`.

#### Sorting

`np.sort` returns the sorted values, and `np.argsort` their **positions**, in the order that sorts them. An array of positions selects the elements in that order:

```python
order = np.argsort(difference)  # from the lowest difference to the highest
order
```

```text
array([13, 15,  5,  8,  4, 11, 14,  6,  9,  3, 10, 12,  7,  2,  0,  1])
```

```python
titles[order[:2]], difference[order[:2]]  # where I disagree the most with the critics
```

```text
(array(['Grand Theft Auto IV', 'Minecraft'], dtype=object), array([-2.6, -1.9]))
```

The game with the lowest difference is in position 13 (Grand Theft Auto IV), the next one in position 15 (Minecraft), and so on. With `argsort`, you can sort several arrays in the same order (`titles[order]`, `scores[order]`), as `sort_values` does with the rows of a DataFrame. The order is ascending: for the descending order, sort the opposite values, `np.argsort(-difference)`.

## Aggregations

#### Axes

Methods like `sum`, `mean`, `min` and `max` reduce an array to a single number or, with `axis`, to one number per row or per column:

```python
scores.mean()  # np.float64(7.9): the average of all the 80 scores
scores.mean(axis=0)  # one average per column, i.e. per category
```

```text
array([8.4375, 7.    , 8.    , 8.6875, 7.375 ])
```

```python
scores.mean(axis=1)  # one average per row, i.e. per game (without weights)
```

```text
array([9.2, 9. , 9. , 8.8, 8. , 8. , 8.2, 8. , 8. , 8. , 7.6, 7.6, 7. ,
       7. , 6.8, 6.2])
```

The axis is the dimension that disappears: `scores` has shape `(16, 5)`, its mean along axis 0 (the rows) has shape `(5,)`, and its mean along axis 1 (the columns) has shape `(16,)`. It's the same `axis` as in pandas, where `sum(axis=1)` computed the sum of each row.

Other aggregations are `sum`, `min`, `max`, `std`, `argmin`, `argmax` and `cumsum` (methods of the array), and `np.median` and `np.percentile` (functions: e.g., `np.percentile(scores, 90, axis=0)`). On booleans, `sum` counts the `True`, `mean` gives their proportion, while `any` and `all` check whether at least one, or all of them, are `True`:

```python
(scores >= 9).sum(axis=0)  # how many 9s and 10s in each category
```

```text
array([7, 3, 7, 9, 4])
```

```python
titles[(scores < 5).any(axis=1)]  # the games with at least one score below 5
```

```text
array(['DOOM (2016)', "Don't Starve", 'Minecraft'], dtype=object)
```

#### Weighted Averages

`np.average` computes weighted averages, and it's how `add_overall_column` computes the overall scores of all the games, in `vgrank/scoring.py`:

```python
np.average(scores, axis=1, weights=weights)  # one weighted average per row
np.allclose(np.average(scores, axis=1, weights=weights), overall)  # True
```

Since the weights sum to 1, the result is the same as `(scores * weights).sum(axis=1)`, the vectorized version of our first loop. `np.average` divides by the sum of the weights, so it also works with weights that don't sum to 1, like `[3, 2, 2, 1, 2]`.

#### Where the Maximum Is

`max` returns the highest value, `argmax` its position. Along axis 1, `argmax` returns, for each game, the position of the category with the highest score, and an array of names turns the positions into names:

```python
category_names = np.array(CATEGORIES)
category_names[scores.argmax(axis=1)][:8]
```

```text
array(['gameplay', 'gameplay', 'story', 'story', 'gameplay', 'gameplay',
       'sound', 'replayability'], dtype='<U13')
```

With ties, `argmax` returns the first position: Hollow Knight has a 10 for gameplay and for sound, and gets gameplay. But is this really where each game stands out? I give high scores to the sound of almost every game (8.69 on average), so Grand Theft Auto V gets sound, with a 9. That's why `strongest_category`, in `vgrank/analysis.py`, standardises the scores first: we'll do it in [Broadcasting](#standardising).

#### Standard Deviation

```python
scores.std(axis=0).round(2)  # array([1.  , 2.12, 1.37, 1.16, 1.73])
ranking[CATEGORIES].std().round(2)  # gameplay 1.03, story 2.19, graphics 1.41, ...
```

Same data, different results! By default, NumPy divides by *n*, the number of values (`ddof=0`): it's the standard deviation of these values. pandas divides by *n* − 1 (`ddof=1`): it's the *sample* standard deviation, which estimates the variability of a larger population from a sample of it (my 16 games, out of all the games that I could play). With `ddof`, you choose: `scores.std(axis=0, ddof=1)` gives the same result as pandas. The difference shrinks as the data grow (here, it's about 3%), but it's there: choose one, and say which. vgrank uses both: pandas in the summary of the categories (`category_summary`), and `ddof=0` for the z-scores of `strongest_category` (there, the choice doesn't matter: it changes all the z-scores by the same factor, and not the category where a game stands out).

#### Missing Values

```python
metacritic_scores = np.array([93.0, 89.0, np.nan])  # the third one is missing
metacritic_scores.mean()  # np.float64(nan)
np.nanmean(metacritic_scores)  # np.float64(91.0)
```

These are the three games of the lecture on coding with LLMs. `np.nan` (*Not a Number*) is the missing value of NumPy, and it's contagious: any computation that involves it gives `nan`. pandas skips missing values by default; NumPy doesn't. Use the `nan` versions of the functions (`np.nanmean`, `np.nansum`, `np.nanmax`, `np.nanstd`, ...), or drop the missing values first. `nan` isn't equal to anything, not even to itself (`np.nan == np.nan` is `False`): find it with `np.isnan`, e.g. `metacritic_scores[~np.isnan(metacritic_scores)]`. And `nan` is a float: an array of integers can't contain it, which is why pandas turns a column of integers with missing values into floats.

## Broadcasting

#### The Rules

Operations between arrays with different shapes follow the rules of **broadcasting**. We've already used it: in `scores * weights`, a matrix with shape `(16, 5)` is multiplied by an array with shape `(5,)`, and each row of the matrix is multiplied by the weights:

```python
(scores * weights)[:3]
```

```text
array([[3. , 1.6, 1.8, 0.9, 2. ],
       [3. , 1.6, 1.8, 1. , 1.6],
       [2.7, 2. , 1.8, 1. , 1.4]])
```

NumPy compares the shapes starting from the last dimension. Two dimensions are compatible when they are equal, or when one of them is 1 (a missing dimension counts as 1): the array with size 1 is repeated along that dimension, without copying any data.

```text
scores    16 x 5
weights        5    ->  1 x 5, repeated for each of the 16 rows
result    16 x 5
```

A single number is the simplest case: `scores / 10` divides all the values. When the shapes are not compatible, you get an error: `scores * np.array([0.5, 0.5])` raises `ValueError: operands could not be broadcast together with shapes (16,5) (2,)`.

#### Positions, Not Labels

In the pandas lecture, `games[CATEGORIES] * weights` matched the names of the columns with the labels of the weights. Arrays have no labels: the first weight multiplies the first column, whatever it contains. Suppose that your weights come from a dictionary written in another order:

```python
my_weights = {
    "story": 0.2,
    "gameplay": 0.3,
    "graphics": 0.2,
    "sound": 0.1,
    "replayability": 0.2,
}
wrong = np.array(list(my_weights.values()))  # the weight of story comes first
(scores * wrong).sum(axis=1)[:3]  # array([9.1, 8.8, 9. ]), not 9.3, 9.0 and 8.9
```

No error and no `NaN`, just wrong scores: it's the NumPy version of the silent bug of the pandas lecture. Build your arrays in a fixed order, the one of your constants, `np.array([my_weights[category] for category in CATEGORIES])`, as `vgrank/scoring.py` does.

#### Standardising

A **z-score** says how far a value is from the average of its column, in standard deviations: *z* = (*x* − mean) / std. A z-score of 1 means one standard deviation above the average of the category, 0 exactly average. With broadcasting, the z-scores of all the categories take one line:

```python
z_scores = (scores - scores.mean(axis=0)) / scores.std(axis=0)
z_scores[:3].round(2)
```

```text
array([[ 1.57,  0.47,  0.73,  0.27,  1.52],
       [ 1.57,  0.47,  0.73,  1.13,  0.36],
       [ 0.56,  1.41,  0.73,  1.13, -0.22]])
```

`scores.mean(axis=0)` and `scores.std(axis=0)` have shape `(5,)`, a value per category, and they are broadcast over the 16 rows. The 9 of Hades for sound is only 0.27 standard deviations above my average for sound. The category where each game stands out is the one with the highest z-score, as in `strongest_category`, which computes the same z-scores with SciPy (`stats.zscore(score_matrix, axis=0, ddof=0)`, see [below](#scipy)):

```python
pd.DataFrame(
    {
        "title": titles,
        "highest score": category_names[scores.argmax(axis=1)],
        "stands out in": category_names[z_scores.argmax(axis=1)],
    }
).head(8)
```

```text
                      title  highest score  stands out in
0                     Hades       gameplay       gameplay
1             Hollow Knight       gameplay       gameplay
2  The Witcher 3: Wild Hunt          story          story
3     Red Dead Redemption 2          story       graphics
4                  Portal 2       gameplay       gameplay
5                Elden Ring       gameplay       graphics
6        Grand Theft Auto V          sound          story
7            Stardew Valley  replayability  replayability
```

Grand Theft Auto V doesn't stand out for its sound, a 9 like many other games, but for its story: an 8, when my average for story is 7.

What if all the games have the same score in a category? Its standard deviation is 0, and the division gives `nan`. It's an edge case that happens with few data, e.g. with only Hades and Hollow Knight, which have the same scores for gameplay, story and graphics:

```python
first_two = scores[:2]
(first_two - first_two.mean(axis=0)) / first_two.std(axis=0)
```

```text
RuntimeWarning: invalid value encountered in divide
array([[nan, nan, nan, -1.,  1.],
       [nan, nan, nan,  1., -1.]])
```

NumPy doesn't raise an error, only a **warning**: the code goes on, with `nan` in the results, so read the warnings! `strongest_category` handles this case with `np.nan_to_num`, which replaces `nan` with 0: in a category where all the games are equal, no game stands out.

#### Rows and Columns

Since the shapes are compared from the right, an array with a value for each **row** can't be broadcast as it is. Let's compare each score with the average of its game:

```python
scores - scores.mean(axis=1)
```

```text
ValueError: operands could not be broadcast together with shapes (16,5) (16,)
```

The last dimensions, 5 and 16, don't match. The averages must be a column, with shape `(16, 1)`: `keepdims=True` keeps the dimension that the aggregation removes.

```python
game_means = scores.mean(axis=1, keepdims=True)  # shape (16, 1)
(scores - game_means)[:3]
```

```text
array([[ 0.8, -1.2, -0.2, -0.2,  0.8],
       [ 1. , -1. ,  0. ,  1. , -1. ],
       [ 0. ,  1. ,  0. ,  1. , -2. ]])
```

`(16, 5)` and `(16, 1)` are compatible: the column of averages is repeated for each of the 5 categories. To turn a 1D array into a column, you can also add a dimension with `np.newaxis`: `scores.mean(axis=1)[:, np.newaxis]` has shape `(16, 1)`.

#### Distances

Broadcasting a single row compares every game with that row. Which games are the most similar to Hades? The distance between two games is the length of the difference between their scores (the Euclidean distance, i.e. Pythagoras' theorem in 5 dimensions):

```python
differences = scores - scores[0]  # (16, 5) minus (5,): each game minus Hades
distances = np.sqrt((differences**2).sum(axis=1))  # or np.linalg.norm(..., axis=1)
nearest = np.argsort(distances)[1:3]  # position 0 is Hades itself, at distance 0
titles[nearest], distances[nearest].round(2)
```

```text
(array(['Hollow Knight', 'Grand Theft Auto V'], dtype=object), array([2.24, 3.  ]))
```

If you liked Hades, try Hollow Knight. With `np.newaxis`, broadcasting computes all the pairs at once: an array with shape `(3, 1)` against one with shape `(3,)` gives a `(3, 3)` matrix. Back to the distances between the cities:

```python
distance_km(
    latitude[:, np.newaxis], longitude[:, np.newaxis], latitude, longitude
).round()
```

```text
array([[  0., 477., 886.],
       [477.,   0., 426.],
       [886., 426.,   0.]])
```

The row is the city of departure, the column the city of arrival. With the airports of *Island Hopper*, it's the table of all the possible flights, from which you keep those within the range of your plane.

## Linear Algebra

#### Matrix Product

`@` is the **matrix product**. Between a matrix and a vector, it computes, for each row of the matrix, the sum of its products with the vector: exactly our weighted sum.

```python
scores @ weights  # (16, 5) @ (5,) -> (16,): the overall score of each game
```

```text
array([9.3, 9. , 8.9, 8.6, 8.2, 8.1, 8.1, 8.1, 7.9, 7.8, 7.5, 7.4, 7. ,
       6.9, 6.8, 6.4])
```

It's also the fastest way: on the million ratings of [Why It's Fast](#why-its-fast), `many_scores @ weights` takes 0.002 seconds, since it doesn't create the intermediate matrix of the products. Between two matrices, the inner dimensions must match: `(16, 5) @ (5, 3)` gives a `(16, 3)` matrix. What if I cared more about the story? Let's compare three sets of weights, one per row:

```python
profiles = np.array(
    [
        [0.3, 0.2, 0.2, 0.1, 0.2],  # mine
        [0.2, 0.2, 0.2, 0.2, 0.2],  # all equal
        [0.1, 0.5, 0.2, 0.1, 0.1],  # story first
    ]
)
by_profile = (scores @ profiles.T).round(2)  # (16, 5) @ (5, 3) -> (16, 3)
pd.DataFrame(by_profile, index=titles, columns=["mine", "equal", "story first"]).head()
```

```text
                          mine  equal  story first
Hades                      9.3    9.2          8.7
Hollow Knight              9.0    9.0          8.6
The Witcher 3: Wild Hunt   8.9    9.0          9.4
Red Dead Redemption 2      8.6    8.8          9.4
Portal 2                   8.2    8.0          8.3
```

`.T` is the **transpose**, which swaps rows and columns: the profiles, one per row, become one per column, as the matrix product requires (`scores @ profiles` raises `ValueError: matmul: Input operand 1 has a mismatch in its core dimension 0, ...`). With the story first, The Witcher 3 and Red Dead Redemption 2 overtake Hades. Remember the difference: `*` multiplies element by element (with broadcasting), `@` is the matrix product.

#### Beyond the Product

`np.linalg` contains the other operations of linear algebra: `norm` (the length of vectors, as in the distances above), `solve` (systems of linear equations), `inv` (the inverse of a matrix), `eig` (eigenvalues and eigenvectors), `lstsq` (least squares). Square matrices also appear in the project proposals: in the voting matrix of *Douze Points*, with the points given by each country (the rows) to each other country (the columns), `votes.T` swaps who gives and who receives, so `votes - votes.T` shows the imbalance between each pair of countries.

## Random Numbers

#### Pseudo-Randomness

Simulations, random samples, the wrong answers of a quiz: many computations need random numbers. Computers produce them with an algorithm, a **pseudo-random number generator**: its numbers look random, but they are determined by a starting value, the **seed**. Same seed, same numbers: that's what makes a random computation **reproducible**, so that you (and your instructor) get the same results at every run, and you can debug it.

```python
rng = np.random.default_rng(42)  # a generator, with 42 as its seed

rng.integers(1, 11, size=5)  # 5 integers from 1 to 10: the end is excluded
rng.random(3)  # 3 floats between 0 and 1
rng.normal(loc=8, scale=1, size=3)  # 3 values from a normal distribution
rng.choice(titles)  # a random game
rng.choice(titles, size=3, replace=False)  # 3 different games
```

```text
array([1, 8, 7, 5, 5])
array([0.69736803, 0.09417735, 0.97562235])
array([8.1278404 , 7.68375741, 7.98319884])
'Grand Theft Auto IV'
array(['The Elder Scrolls V: Skyrim', 'Cyberpunk 2077',
       'Grand Theft Auto V'], dtype=object)
```

Run the cell again, and you get the same numbers: the generator starts again from the same seed. Within a run, each call continues the sequence, so two calls of `rng.integers(1, 11, size=5)` give two different arrays. Without a seed, `np.random.default_rng()` starts from an unpredictable one, and the numbers change at every run.

`rng.permutation(array)` returns the elements in a random order: with `choice`, that's all you need for the *Around the Globe Quiz*, e.g. three different wrong answers, `rng.choice(capitals, size=3, replace=False)`, shuffled together with the right one. pandas accepts a seed too, or a generator: `ranking.sample(n=3, random_state=42)`, as in the pandas lecture.

> **NOTE**: in many examples online, you'll find the legacy functions of NumPy, with a global seed: `np.random.seed(42)`, then `np.random.rand(3)` or `np.random.randint(1, 11, size=5)`. They still work, but they give different numbers, and the documentation recommends generators for new code. The `random` module of the standard library (`random.seed(42)`, `random.choice(...)`) works well for single values, without NumPy.

#### Simulations

A **simulation** repeats a random experiment many times, and looks at the distribution of the results. Here's a question for vgrank: my ranking depends on the weights of the categories, which I chose. Would the ranking be very different with other weights? Let's draw 10,000 random sets of weights, and count how many times each game comes first:

```python
def first_places(scores, n_simulations=10_000, seed=42):
    """Count how many times each game ranks first, with random weights."""
    rng = np.random.default_rng(seed)
    random_weights = rng.random((n_simulations, scores.shape[1]))
    random_weights = random_weights / random_weights.sum(axis=1, keepdims=True)
    overall = scores @ random_weights.T  # one column for each set of weights
    winners = overall.argmax(axis=0)  # the position of the best game, in each column
    return np.bincount(winners, minlength=len(scores))


share = first_places(scores) / 10_000
pd.Series(share, index=titles).head()
```

```text
Hades                       0.6000
Hollow Knight               0.0776
The Witcher 3: Wild Hunt    0.1961
Red Dead Redemption 2       0.1262
Portal 2                    0.0001
dtype: float64
```

Step by step:

- `rng.random((10_000, 5))` draws 10,000 rows of 5 random numbers, and dividing each row by its sum (with `keepdims=True`, to broadcast a column) makes the weights of each row sum to 1;
- the matrix product computes the overall score of the 16 games with each set of weights: 160,000 scores, in a `(16, 10_000)` matrix, without a loop;
- `argmax(axis=0)` finds the best game in each column, and `np.bincount` counts how many times each position appears.

Hades comes first with 60% of the random weights, and only the first five games of my ranking ever come first (Portal 2 only once; the other games, not shown, never do): the top of my ranking doesn't depend much on the choice of the weights. The seed is a parameter of the function, with a default value: the results are reproducible, and you can check that they don't depend on the seed, by changing it. Arguments like this one (how much do the results depend on my choices?) make a project convincing, at the exam too.

## SciPy

NumPy provides arrays and the basic operations; **SciPy** (`pip install scipy`) provides algorithms built on them, organised in modules: statistics, optimisation, interpolation, distances, and more. For your project, the most useful one is `scipy.stats`, which vgrank imports as `from scipy import stats`.

#### Correlation

Do my scores agree with the critics'? A **correlation** coefficient measures the relationship between two variables, from −1 (one decreases when the other increases) to 1 (they increase together); 0 means no relationship.

```python
from scipy import stats

stats.pearsonr(overall, metacritic)
```

```text
PearsonRResult(statistic=np.float64(0.59195148210881), pvalue=np.float64(0.015704083848088014))
```

```python
result = stats.spearmanr(overall, metacritic)
result.statistic, result.pvalue
```

```text
(np.float64(0.553664104299795), np.float64(0.026077925511530427))
```

Pearson's coefficient measures a *linear* relationship; Spearman's computes the same coefficient on the ranks of the values (first, second, ...), so it measures whether the two variables grow together, even if not along a line, and it's less sensitive to extreme values. For scores and rankings, Spearman's is a natural choice: it's what `compare_with_metacritic` uses.

The **p-value** answers a question: if there were no relationship between my scores and the critics', how likely would it be to observe a correlation at least as strong as this one, just by chance? Here, 2.6%: unlikely. By convention, below 0.05 a result is called *statistically significant*. But the p-value doesn't measure how strong the relationship is (the coefficient does), and with few data, like my 16 games, even a fairly strong correlation can have a large p-value. And, as always, a correlation doesn't imply that one variable causes the other.

> **NOTE**: NumPy (`np.corrcoef`) and pandas (`corr()`, as in `category_correlation` in `vgrank/analysis.py`, and `corr(method="spearman")`) compute the coefficients too; SciPy adds the p-value.

#### Trends

Do I prefer recent games? `linregress` fits a straight line through the points (the *least squares* line), and returns its slope, with a p-value:

```python
year = pd.to_datetime(ranking["released"]).dt.year.to_numpy()
trend = stats.linregress(year, overall)
print(f"slope: {trend.slope:.3f} points per year, p-value: {trend.pvalue:.3f}")
```

```text
slope: 0.098 points per year, p-value: 0.079
```

About one point per decade: the more recent a game, the higher my score. But the p-value is 0.079: with 16 games, a slope like this one could also come out by chance. The same function gives the warming in °C per year of a city, in *Weather Report*, or the degradation of the tyres in seconds per lap, in *Box, Box*. `np.polyfit(year, overall, 1)` returns the same slope and intercept, without the p-value.

#### Comparing Groups

In the pandas lecture, we found that the best-sellers in my ranking have an average score slightly lower than the other games (7.82 against 7.94), and we said that, with 9 and 7 games, the difference meant little. A **t-test** quantifies it:

```python
# with_sales, from the pandas lecture: the rows of ranking, with the best-sellers marked
best_seller = with_sales["best_seller"].to_numpy()
stats.ttest_ind(overall[best_seller], overall[~best_seller])
```

```text
TtestResult(statistic=np.float64(-0.276505483824491), pvalue=np.float64(0.7862000316654131), df=np.float64(14.0))
```

The p-value is 0.79: if best-sellers and other games had the same average score, a difference as large as ours would be very common. The t-test assumes that the values are roughly normally distributed; with few values, or with ratings on a scale from 1 to 5, the Mann–Whitney U test, `stats.mannwhitneyu`, is a safer choice (here, its p-value is 0.83).

#### More SciPy

A few more modules that may be useful in your project:

- `scipy.stats` also has the probability distributions (`stats.norm`, `stats.binom`, ...), with their densities and quantiles, and many more tests (e.g., `chi2_contingency` for tables of counts);
- `scipy.optimize` finds the minimum of a function, or the parameters of a curve that fit your data (`curve_fit`);
- `scipy.interpolate` estimates values between known points, e.g. to fill a gap in a time series;
- `scipy.spatial` computes the distances between all the pairs of points (`distance.cdist`) and finds the nearest neighbours of a point (`KDTree`), e.g. the airports within range, in *Island Hopper*;
- `scipy.sparse` stores matrices that are mostly zeros: the users × books matrix of goodbooks-10k has more than 500 million cells (53,424 users, 10,000 books), but only 6 million ratings.

Libraries for statistical models (statsmodels) and machine learning (scikit-learn) are built on NumPy and SciPy too, but, as the [FAQ](../README.md#frequently-asked-questions) says, your project doesn't need them: the quality of your code and the appropriateness of your analysis matter more.

## Useful Resources

- [NumPy: the absolute basics for beginners](https://numpy.org/doc/stable/user/absolute_beginners.html), and the pages of the NumPy user guide on [indexing](https://numpy.org/doc/stable/user/basics.indexing.html), [copies and views](https://numpy.org/doc/stable/user/basics.copies.html) and [broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html)
- [Random sampling](https://numpy.org/doc/stable/reference/random/index.html), from the NumPy documentation
- [Statistics (`scipy.stats`)](https://docs.scipy.org/doc/scipy/tutorial/stats.html), from the SciPy user guide
- [Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/), by Jake VanderPlas, chapter 2, and [Python for Data Analysis](https://wesmckinney.com/book/numpy-basics.html), by Wes McKinney, chapter 4 (both free online)
- [`vgrank/scoring.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/scoring.py) and [`vgrank/analysis.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/analysis.py), the NumPy and SciPy code of vgrank
