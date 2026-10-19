# Project Organization

In the shell lecture, we created the skeleton of vgrank with a single script, `vgrank/main.py`. One file is enough for a few lines of code, but your project will grow to hundreds of lines: loading the data, cleaning it, computing statistics, drawing plots, presenting the results. If everything lives in one file (or in one long notebook), the code becomes hard to read, to fix and to reuse, and, at the exam, hard to explain.

A well-organized project gives you:

- **Clarity:** you (and your instructor) can find things quickly.
- **Maintainability:** when you find a bug, you fix it in one place, without breaking the rest.
- **Reusability:** the same function can be used by a notebook, a script and a web app.

This lecture covers how to split your code into modules, how to write functions and classes, how to document them, how to keep a consistent style, and how to find bugs.

## Project Structure

This is the structure of the final version of [vgrank](https://github.com/sergiopicascia/vgrank):

```
vgrank/
├── README.md                # what the project is about, and how to run it
├── requirements.txt         # the libraries needed by the project
├── .gitignore
├── data/
│   └── games.csv            # the dataset
├── vgrank/                  # the package: all the logic of the project
│   ├── __init__.py
│   ├── api.py               # requests to the RAWG API
│   ├── models.py            # the Game class
│   ├── scoring.py           # categories, weights and overall score
│   ├── collection.py        # the GameCollection class: load, save, rank
│   ├── analysis.py          # statistics
│   └── plots.py             # charts
├── notebooks/
│   └── vgrank_walkthrough.ipynb   # the presentation of the project
├── app.py                   # the web app (see the Streamlit lecture)...
└── pages/                   # ...and its pages
```

It's the skeleton that we created in the shell lecture, grown up:

- the **root** folder is the Git repository, with the files of the previous lectures: `README.md`, `requirements.txt` and `.gitignore`;
- `data/` contains the datasets (large files stay out of Git, and the README explains how to download them);
- `vgrank/` is the **package**, with the same name as the project: it contains all the logic, split into modules;
- `notebooks/` (or `app.py` and `pages/`, for a web app) **presents** the project: it imports the functions of the package, and shows their results.

The most important idea is the separation between **logic** (the package) and **presentation** (the notebook or the app). The package doesn't show anything: its functions return values (a number, a DataFrame, a figure), and the notebook decides what to display. This way, the same function works in a notebook, in a script and in a web app, and the notebook stays short and readable. As the README says, the notebook or the app should *use* the code of your project, not duplicate it.

> **NOTE**: there is no single correct structure, and you don't need to copy the modules of vgrank. But a package with a few modules, a `data/` folder and a notebook (or an app) is all your project needs.

## Modules and Packages

A **module** is a `.py` file; a **package** is a folder of modules, containing a file named `__init__.py`, which tells Python that the folder is a package. `__init__.py` can be empty, or contain a docstring that describes the package.

#### Splitting the Code

Give each module **one responsibility**, and name it after it. In vgrank:

- `api.py` talks to the RAWG API;
- `models.py` defines how a game is represented (the `Game` class);
- `scoring.py` contains the rules of the overall score;
- `collection.py` loads, saves and ranks the games;
- `analysis.py` computes statistics, and `plots.py` draws charts.

Your project can follow the steps of your analysis: e.g., `data.py` to download, load and clean the data, `analysis.py` for the computations, `plots.py` for the charts. When a module grows too much, or does two unrelated things, split it.

> **NOTE**: don't give your files the name of a library (e.g., `pandas.py`, `random.py`, `csv.py`): Python would import your file instead of the library, with confusing errors like `AttributeError: module 'pandas' has no attribute 'read_csv'`.

#### Imports

In the modules of the package, import the other modules with their full name, starting from the name of the package (**absolute imports**). Put all the imports at the top of the module, in three groups separated by a blank line: the standard library, the external libraries, your own modules. For instance, this is the beginning of `vgrank/collection.py`:

```python
import os

import pandas as pd

from vgrank.models import GAME_COLUMNS, game_from_row
from vgrank.scoring import add_overall_column, rank_games, validate_scores
```

> **NOTE**: you may also find *relative imports*, like `from .models import Game`, where the dot means "the package of this module". They work too, but absolute imports are easier to read, and they are the ones recommended by PEP 8.

#### What Goes in a Module

When a module is imported, Python runs it from top to bottom. The modules of the package should therefore only **define** things (functions, classes and constants), and not run the analysis, print results or draw plots: that's the job of the notebook.

**Constants** are values that don't change while the program runs: define them once, at the top of the module, with an `UPPER_SNAKE_CASE` name. The categories and the weights of vgrank are defined in `scoring.py`, and used everywhere else:

```python
# In vgrank/scoring.py
CATEGORIES = ["gameplay", "story", "graphics", "sound", "replayability"]

WEIGHTS = {
    "gameplay": 0.30,
    "story": 0.20,
    "graphics": 0.20,
    "sound": 0.10,
    "replayability": 0.20,
}
```

To change a weight, you change one line, instead of looking for `0.3` in all your files. The same goes for the paths of your data files, the URL of an API, or the thresholds of your analysis.

#### Running Your Code

Python looks for the modules to import in a list of folders, called `sys.path`: the folder of the script that you run (for a notebook, the folder of the notebook), the standard library, and the libraries installed in your environment. This has a consequence that surprises everyone the first time. Suppose that `vgrank/main.py` imports a module of the package:

```python
# In vgrank/main.py
from vgrank.scoring import CATEGORIES

print(CATEGORIES)
```

```bash
# From the root of the project, with the environment active
python vgrank/main.py
# ModuleNotFoundError: No module named 'vgrank'
```

Python looks for the package `vgrank` in the folder of the script, `vgrank/`, which doesn't contain it. If you run the script as a module of the package instead, with `-m` and without `.py`, Python looks in the current folder, i.e. the root of the project:

```bash
python -m vgrank.main
# ['gameplay', 'story', 'graphics', 'sound', 'replayability']
```

In general, the code that *runs* things lives outside the package: in the notebook, or in a script in the root of the project (like `app.py` in vgrank), which you run from the root. As your project grows, `main.py` will probably disappear: its functions move to the modules, and the code that runs the analysis moves to the notebook.

The root also matters for your data: relative paths, like `data/games.csv`, are relative to the current working directory, so they only work when you run your code from the root. Notebooks are a special case: see [below](#notebooks-for-presentation).

## Functions and Classes

#### Functions

Most of your code will be functions. A good function:

- does **one thing**, and its name says what (usually with a verb: `load_games`, `compute_overall`, `plot_scores`);
- receives its inputs as **parameters**, instead of reading variables defined elsewhere (constants aside);
- **returns** its result, instead of printing it;
- is **short**: if you need to scroll to read it, it probably does more than one thing.

```python
# Easy to reuse: the input is a parameter, the output is a return value
def average(values):
    return sum(values) / len(values)
```

#### A Brief Introduction to Classes

Object-Oriented Programming (OOP) bundles some data, and the functions that operate on them, into single units called **objects**.

- **Class:** a blueprint for creating objects. In vgrank, the `Game` class defines what all the rated games have in common.
- **Object** (or **instance**): a specific entity created from a class. If `Game` is the blueprint, The Witcher 3, with its scores, is an object.
- **Attribute:** a piece of data of an object, like its `title`, its release date or its `scores`.
- **Method:** a function that belongs to an object, and works on its attributes, like `overall()` or `release_year()`.

```python
# In vgrank/models.py (simplified)
from vgrank.scoring import overall_score


class Game:
    """A video game rated by the user."""

    def __init__(self, title, released, scores):
        self.title = title
        self.released = released  # the release date, e.g. "2015-05-18"
        self.scores = scores  # a dictionary: category -> score

    def overall(self):
        """Return the weighted overall score of the game."""
        return overall_score(self.scores)

    def release_year(self):
        """Return the release year as an integer, or None if unknown."""
        if not self.released:
            return None
        return int(self.released[:4])
```

The `__init__` method runs when you create an object, and stores the data in its attributes. `self` is the object itself: it's the first parameter of every method, but you don't pass it, Python does it for you.

```python
witcher = Game(
    "The Witcher 3: Wild Hunt",
    "2015-05-18",
    {"gameplay": 9, "story": 10, "graphics": 9, "sound": 10, "replayability": 7},
)

witcher.title  # 'The Witcher 3: Wild Hunt'
witcher.release_year()  # 2015
witcher.overall()  # 8.9, i.e. Game.overall(witcher)
```

Use a class when some data and the operations on them belong together: a game and its scores (`Game`), or the collection of games, with the methods to load, save and rank it (`GameCollection`). Don't write classes just for the sake of it: `analysis.py` and `plots.py` are made of plain functions that receive a DataFrame, and that's perfectly fine. Most data science projects need few classes, if any.

## Documentation

#### Docstrings

A **docstring** is a string right below the `def` (or `class`) line, which explains what the function does, its parameters and its return value. Unlike comments, Python keeps docstrings: `help(overall_score)` prints them, and VS Code and PyCharm show them when you hover on the name of the function.

There are a few conventions for writing them. We use the **NumPy style**, the one of the documentation of `numpy`, `pandas` and `scipy`:

```python
def overall_score(scores, weights=WEIGHTS):
    """
    Compute the weighted average of the category scores.

    Parameters
    ----------
    scores : dict
        Mapping ``category -> score``.
    weights : dict, optional
        Mapping ``category -> weight``. Defaults to ``WEIGHTS``.

    Returns
    -------
    float
        The overall score, rounded to two decimals.
    """
```

- If a function raises errors on purpose, list them in a `Raises` section (see `validate_scores` in `vgrank/scoring.py`).
- Short functions can have a one-line docstring, like `"""Return the release year as an integer, or None if unknown."""`.
- Classes describe their attributes in an `Attributes` section, and each module starts with a docstring that describes what it contains (see `vgrank/models.py`).

> **NOTE**: the other popular convention is the Google style (`Args:`, `Returns:`). Both are fine: pick one, and use it everywhere.

#### Comments

Docstrings say what a function does; comments explain the code inside it. Good comments explain **why**, since the code already says *what*:

```python
# Useless: it repeats the code
count = count + 1  # add 1 to count

# Useful: it explains a choice
# Standardise the scores, so that a category that is rated high
# for every game does not always win
z_scores = stats.zscore(score_matrix, axis=0, ddof=0)
```

> **NOTE**: you'll also meet **type hints**, which declare the types of parameters and return values: `def overall_score(scores: dict, weights: dict = WEIGHTS) -> float:`. Python doesn't check them when it runs the code, and you don't have to use them (vgrank doesn't). If you do, use the modern syntax, `list[str]` and `str | None`, rather than `List` and `Optional` from the `typing` module, which you'll find in older code.

## Style

#### PEP 8

[PEP 8](https://peps.python.org/pep-0008/) is the official style guide for Python code: following it makes your code easy to read for anyone who knows Python (your instructor included). The main rules:

- **Indentation:** 4 spaces per level, never tabs.
- **Line length:** PEP 8 says at most 79 characters; many projects (and Ruff, by default) accept up to 88.
- **Names:**
    - `snake_case` for variables, functions and modules (`overall_score`, `scoring.py`);
    - `PascalCase` for classes (`GameCollection`);
    - `UPPER_SNAKE_CASE` for constants (`CATEGORIES`, `MAX_SCORE`).
- **Spaces** around operators and after commas (`x = a + b`, `f(a, b)`), but not around the `=` of keyword arguments (`round(x, ndigits=2)`).
- **Blank lines:** two around top-level functions and classes, one between methods.
- **Imports:** at the top of the file, grouped as we saw above.

Beyond the rules, choose names that explain the code: `games`, `mean_score` and `release_year` tell the reader what they contain, `df2`, `x` and `tmp` don't.

#### Linters and Formatters

You don't need to check these rules by hand. Two kinds of tools do it for you:

- a **linter** finds style problems and likely bugs, e.g. a library that you import but never use, or a variable that you use but never defined;
- a **formatter** rewrites your code in a consistent style, automatically.

[Ruff](https://docs.astral.sh/ruff/) is both, and it's very fast (it comes from the same developers as `uv`). Install it in your environment, and run it from the root of the project:

```bash
pip install ruff

# Look for problems in all the Python files of the project
ruff check .

# Fix the problems that can be fixed automatically (e.g., unused imports)
ruff check --fix .

# Format all the Python files of the project
ruff format .
```

In VS Code, the [Ruff extension](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff) underlines the problems while you type, and can format the file whenever you save it.

> **NOTE**: `flake8` (linter) and `black` (formatter) are also valid alternatives.

## Notebooks for Presentation

A notebook is a great way to present your project: it combines text, code, tables and plots, and it can guide the audience through your analysis step by step. In your project, the notebook is a **client** of your package, not its container: it imports the functions of the package, calls them, and shows their results. If you are writing a long function in a cell, ask yourself whether it belongs to the package.

#### Importing Your Package

Jupyter runs a notebook from its own folder, `notebooks/`, which doesn't contain the package: `from vgrank.collection import GameCollection` fails with `ModuleNotFoundError: No module named 'vgrank'`. The vgrank notebook solves it in its first cell, by adding the root of the project to `sys.path`:

```python
import os
import sys

# Jupyter runs the notebook from its folder (notebooks/),
# so the root of the project is its parent
NOTEBOOK_DIR = os.getcwd()
PROJECT_ROOT = os.path.dirname(NOTEBOOK_DIR)

# Let Python find the vgrank package, which lives in the root of the project
if PROJECT_ROOT not in sys.path:
    sys.path.append(PROJECT_ROOT)
```

For the same reason, the paths of the data files start from the root of the project:

```python
from vgrank.collection import GameCollection

csv_path = os.path.join(PROJECT_ROOT, "data", "games.csv")
collection = GameCollection(csv_path=csv_path).load()
ranking = collection.ranking()
ranking[["rank", "title", "overall"]].head()
```

> **NOTE**: the kernel of the notebook must be the Python of your environment: see the environments lecture.

#### Editing the Package While Using the Notebook

Python imports each module only once: if you change a function in `vgrank/analysis.py`, the notebook keeps using the old version until you restart the kernel. To reload your modules automatically whenever they change, run this in the first cell of the notebook:

```python
%load_ext autoreload
%autoreload 2
```

The lines starting with `%` are *magic commands* of Jupyter, not Python code.

#### Before Presenting (or Pushing)

You can run the cells of a notebook in any order, and the kernel remembers the variables of the cells that you have changed or deleted: a notebook can work on your computer, and fail for anyone else. Before presenting your project, or pushing the notebook, restart the kernel and run all the cells from top to bottom (*Kernel → Restart Kernel and Run All Cells* in Jupyter Lab, *Restart* and then *Run All* in VS Code): this is what your instructor will do.

## Debugging

**Debugging** is the systematic process of finding and fixing errors in your code. Let's practice with this function, which should compute the average overall score of the games released from a given year on. Save it as `average.py`, and run it with `python average.py`:

```python
def average_score_since(games, first_year):
    """Average overall score of the games released in first_year or later."""
    total = 0
    count = 0
    for game in games:
        if game["year"] >= first_year:
            total += game["overall"]
            count += 1
    return total / len(games)


games = [
    {"title": "Hades", "year": 2020, "overall": 9.3},
    {"title": "Celeste", "year": 2018, "overall": 8.6},
    {"title": "Hollow Knight", "year": "2017", "overall": 9.0},
    {"title": "Tetris", "year": 1984, "overall": 7.5},
]

print(average_score_since(games, 2015))
print(average_score_since([], 2015))
```

It contains three bugs. Let's find them.

#### Reading the Traceback

When an error stops your program, Python prints a **traceback**:

```text
Traceback (most recent call last):
  File "/home/student/projects/vgrank/average.py", line 19, in <module>
    print(average_score_since(games, 2015))
          ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^
  File "/home/student/projects/vgrank/average.py", line 6, in average_score_since
    if game["year"] >= first_year:
       ^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: '>=' not supported between instances of 'str' and 'int'
```

Read it **from the bottom up**:

1. The last line says **what** went wrong: a `TypeError`, because `>=` can't compare a string (`str`) with an integer (`int`).
2. The lines above say **where**: line 6 of `average.py`, in the function `average_score_since`. The `^` characters underline the code that failed.
3. Further up, you find **who called** that function: line 19, in the main code of the script (`<module>`).

When the error comes from inside a library (e.g., pandas), the traceback goes on with the files of the library: look for the last lines that come from *your* files, which is usually where the problem is.

#### Print Debugging

The error says that one of the years is a string. Which one? The most basic debugging technique is to add some `print()` calls, to inspect the variables while the program runs:

```python
    for game in games:
        print(game["title"], game["year"], type(game["year"]))
        if game["year"] >= first_year:
```

```text
Hades 2020 <class 'int'>
Celeste 2018 <class 'int'>
Hollow Knight 2017 <class 'str'>
Traceback (most recent call last):
...
```

The year of Hollow Knight is a string, `"2017"`: a typical problem of data read from a file. The right place for the fix is the data, not the comparison: data are cleaned when they are loaded (we'll see how with pandas). Here, we correct the list (`"year": 2017`), and remove the `print`.

#### Using a Debugger

We could have found the same bug without touching the code. A **debugger** lets you pause your program at specific lines, called **breakpoints**, and inspect all its variables. In VS Code, click to the left of the number of the line `if game["year"] >= first_year:`: a red dot appears, the breakpoint. Then start the debugger (`F5`, choosing *Python Debugger* and *Python File*): the program stops before running that line, and the *Variables* panel shows `game`, `total` and `count`. Press *Continue* (`F5` again): the loop goes on, and stops at the next game. At the third stop, `game` is `{'title': 'Hollow Knight', 'year': '2017', 'overall': 9.0}`: there's the bug.

Notebooks can be debugged too: in VS Code, choose *Debug Cell* from the menu next to the *Run* button of a cell. PyCharm offers the same tools (*Run → Debug*).

#### Silent Bugs

With the year fixed, the script runs:

```text
6.725
Traceback (most recent call last):
  ...
ZeroDivisionError: division by zero
```

The first result comes with no error, but it's wrong: the three games released from 2015 on have scores 9.3, 8.6 and 9.0, so their average can't be 6.725. The function divides by `len(games)`, the number of *all* the games (4), instead of `count`, the number of games it summed (3): the right result is 26.9 / 3 ≈ 8.97. This is a **silent bug**: no error message, just a wrong number. You only catch it by checking the result against one computed by hand, or at least by asking yourself whether it's plausible.

The second call shows an **edge case**: when no game is selected (here, because the list is empty), `count` is 0, and the division fails. What should the function do in this case? It's up to you: here, we raise an error with a clear message, as vgrank does when a score is not valid.

```python
def average_score_since(games, first_year):
    """Average overall score of the games released in first_year or later."""
    total = 0
    count = 0
    for game in games:
        if game["year"] >= first_year:
            total += game["overall"]
            count += 1
    if count == 0:
        raise ValueError(f"No games released in {first_year} or later")
    return total / count
```

```text
8.966666666666667
Traceback (most recent call last):
  ...
ValueError: No games released in 2015 or later
```

Three problems, each with its own signs: a **crash** tells you where to look; a **silent bug** only shows up if you check the results; an **edge case** only if you think about unusual inputs (empty data, missing values, a single row). We'll meet all three again in the code written by LLMs.

#### Getting Help

When you don't understand an error:

- search for its last line (without the paths of your computer): someone has probably had the same problem, e.g. on Stack Overflow or in the issues of the library on GitHub;
- read the official documentation of the functions involved, for the version that you are using (the documentation of pandas, for instance, has a version selector);
- ask an LLM: see the lecture on coding with LLMs.

## Useful Resources

- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [The NumPy docstring style](https://numpydoc.readthedocs.io/en/latest/format.html)
- [Modules](https://docs.python.org/3/tutorial/modules.html) and [Classes](https://docs.python.org/3/tutorial/classes.html), from the official Python tutorial
- [Ruff](https://docs.astral.sh/ruff/), and its [extension for VS Code](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)
- [The autoreload extension](https://ipython.readthedocs.io/en/stable/config/extensions/autoreload.html) of Jupyter
- [Python Debugging in VS Code](https://code.visualstudio.com/docs/python/debugging)
- [vgrank](https://github.com/sergiopicascia/vgrank), the example project of the course
