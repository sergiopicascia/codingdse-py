# Data Input/Output

Every analysis starts by loading some data, and ends by saving or showing its results. The data can come from a file (a CSV, JSON or Excel file), from a database or from the web, and each source can fail in its own way: a missing file, a wrong encoding, a server that doesn't answer.

This lecture covers how to read and write files (plain text, CSV and JSON), how to handle the errors that come with them, and how to get data from the web: downloading files, calling APIs and, as a last resort, scraping web pages.

## Files

A file is a sequence of bytes. In a **text file** (`.txt`, `.csv`, `.json`, `.py`), the bytes represent characters, according to an **encoding**; a **binary file** (an image, an Excel or a ZIP file) can only be read by a program, or a library, that knows its format.

#### Opening a File

The built-in function `open()` takes the path of a file and a **mode**:

| Mode | Meaning |
|---|---|
| `"r"` | read (the default); raises a `FileNotFoundError` if the file doesn't exist |
| `"w"` | write; creates the file, or **overwrites** it if it exists (like `>` in the shell) |
| `"a"` | append; creates the file, or adds to its end (like `>>` in the shell) |
| `"b"` | binary, combined with the others (`"rb"`, `"wb"`), for files that are not text |

Always open files with the `with` statement: at the end of the block, Python closes the file, even if an error occurs inside it. Until a file is closed, what you write may not be on the disk yet.

```python
# Write a text file (the data/ folder must already exist)
with open("data/backlog.txt", "w", encoding="utf-8") as f:
    f.write("Hades\n")
    f.write("Celeste\n")  # write() doesn't add the newline: you do

# Add a line at the end of the file
with open("data/backlog.txt", "a", encoding="utf-8") as f:
    f.write("Hollow Knight\n")
```

There are two common ways to read it:

```python
# Read the whole file into a single string
with open("data/backlog.txt", encoding="utf-8") as f:
    content = f.read()  # 'Hades\nCeleste\nHollow Knight\n'

# Read it line by line: this also works for files larger than your memory
with open("data/backlog.txt", encoding="utf-8") as f:
    for line in f:
        print(line.strip())  # each line ends with "\n": strip() removes it
```

```text
Hades
Celeste
Hollow Knight
```

#### Encodings

An encoding maps each character to one or more bytes: in **UTF-8**, the standard of the web, `é` takes two bytes. Without the `encoding` parameter, `open()` uses the encoding of your operating system: UTF-8 on macOS and Linux, but usually `cp1252` on Windows (with the Python versions of this course). A file written with one encoding and read with another shows garbled characters:

```python
with open("data/pokemon.txt", "w", encoding="utf-8") as f:
    f.write("Pokémon\n")

with open("data/pokemon.txt", encoding="cp1252") as f:  # the default on Windows
    print(f.read())  # PokÃ©mon
```

In the opposite case, a `cp1252` file read as UTF-8, the file can't be read at all: `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 3: invalid continuation byte`.

**Always pass `encoding="utf-8"`** to `open()`: your code will work in the same way on every computer, including your instructor's. If a dataset uses another encoding (its documentation should say so), pass that one instead, e.g. `encoding="latin-1"`.

#### Paths

As we saw in the project organization lecture, a relative path like `data/backlog.txt` starts from the current working directory: it works when you run your code from the root of the project, but not from a notebook, which runs in `notebooks/`. The `os` module of the standard library builds and checks paths for you:

```python
import os

path = os.path.join("data", "backlog.txt")  # data\backlog.txt on Windows
os.path.exists(path)  # True if the file (or the folder) exists

# Create a folder (and the missing folders above it), unless it already exists
os.makedirs(os.path.join("data", "raw"), exist_ok=True)
```

Writing into a folder that doesn't exist raises a `FileNotFoundError`, just like reading a file that doesn't exist: create the folder first. In a notebook, build the paths from the root of the project, as the vgrank notebook does: `os.path.join(PROJECT_ROOT, "data", "games.csv")`.

> **NOTE**: you may also meet `pathlib`, a more recent way to handle paths: `Path("data") / "backlog.txt"`. It works as well as `os.path`, which is the one used by vgrank.

## Structured Data

Plain text has no structure, but most data do: a table has rows and columns, a game has a list of platforms and a score for each category. Two text formats store this structure: **CSV**, for tables, and **JSON**, for nested data. The standard library reads and writes both; for tables, pandas (the next lecture) is even simpler.

#### CSV

A CSV (Comma-Separated Values) file stores a table, one row per line, with the values separated by commas; the first line is usually the **header**, with the names of the columns. These are the header and the first row of `data/games.csv` in vgrank:

```text
game_id,title,released,genres,platforms,metacritic,image_url,gameplay,story,graphics,sound,replayability
3328,The Witcher 3: Wild Hunt,2015-05-18,"Action, RPG","PC, PlayStation 4, Xbox One, Nintendo Switch, PlayStation 5, Xbox Series S/X",92.0,https://media.rawg.io/media/games/618/618c2031a07bbff6b4f611f10b6bcdbc.jpg,9,10,9,10,7
```

The values that contain a comma, like the genres and the platforms, are wrapped in double quotes. That's why you shouldn't split the lines yourself:

```python
with open("data/games.csv", encoding="utf-8") as f:
    header = f.readline()
    first_game = f.readline()

print(len(header.split(",")), len(first_game.split(",")))  # 12 18
```

12 columns, but 18 values: `split` also cuts the genres and the platforms. The `csv` module handles the quotes for you:

```python
import csv

with open("data/games.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        print(row["title"], "|", row["genres"], "|", row["gameplay"])
```

```text
The Witcher 3: Wild Hunt | Action, RPG | 9
Grand Theft Auto V | Action, Adventure | 8
Portal 2 | Shooter, Puzzle | 10
...
```

`csv.DictReader` reads the header, and returns each row as a dictionary, column → value (`csv.reader` returns each row as a list instead, header included). Open CSV files with `newline=""`, both to read and to write them, as the documentation of the `csv` module recommends: otherwise, the values that contain a newline are not read correctly, and on Windows an empty line appears after each row that you write.

A CSV file only contains text: every value you read is a string, and a missing value is an empty string. Converting them is up to you:

```python
with open("data/games.csv", newline="", encoding="utf-8") as f:
    gameplay = [row["gameplay"] for row in csv.DictReader(f)]

gameplay[:3]  # ['9', '8', '10']
sum(gameplay)  # TypeError: unsupported operand type(s) for +: 'int' and 'str'
sum([int(score) for score in gameplay]) / len(gameplay)  # 8.4375
```

To write a CSV file, `csv.DictWriter` takes the names of the columns, and writes a row for each dictionary:

```python
top_games = [
    {"title": "Hades", "overall": 9.3},
    {"title": "Hollow Knight", "overall": 9.0},
]

with open("data/top_games.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["title", "overall"])
    writer.writeheader()
    writer.writerows(top_games)
```

> **NOTE**: not all CSV files use commas. Excel, with Italian settings, saves CSV files with semicolons, since the comma separates the decimals (`24,99`): `csv.reader(f, delimiter=";")` reads them, and so does pandas (see below). Before reading a file, look at its first lines (e.g., with `head` in the shell).

#### JSON

JSON (JavaScript Object Notation) is the format of most web APIs, and of data that don't fit in a table. It looks a lot like Python:

| JSON | Python |
|---|---|
| object: `{"title": "Hades"}` | `dict` |
| array: `["PC", "Switch"]` | `list` |
| string: `"Hades"` | `str` |
| number: `10`, `9.3` | `int`, `float` |
| `true`, `false` | `True`, `False` |
| `null` | `None` |

The `json` module writes Python objects to a file (`json.dump`), and reads them back (`json.load`):

```python
import json

rating = {
    "title": "Hades",
    "scores": {
        "gameplay": 10,
        "story": 9,
        "graphics": 9,
        "sound": 10,
        "replayability": 10,
    },
    "platforms": ["PC", "Nintendo Switch"],
    "finished": True,
    "notes": None,
}

with open("data/hades.json", "w", encoding="utf-8") as f:
    json.dump(rating, f, indent=2)  # indent makes the file readable
```

This is `data/hades.json`: `True` became `true`, and `None` became `null`.

```json
{
  "title": "Hades",
  "scores": {
    "gameplay": 10,
    "story": 9,
    "graphics": 9,
    "sound": 10,
    "replayability": 10
  },
  "platforms": [
    "PC",
    "Nintendo Switch"
  ],
  "finished": true,
  "notes": null
}
```

Reading it gives back the same dictionary:

```python
with open("data/hades.json", encoding="utf-8") as f:
    rating = json.load(f)

rating["scores"]["story"]  # 9
```

`json.dumps` and `json.loads` (with an *s*, for *string*) do the same with strings, instead of files.

> **NOTE**: JSON has fewer types than Python: tuples come back as lists, and the keys of a dictionary always come back as strings (`{1: "a"}` becomes `{"1": "a"}`). By default, `json.dump` also escapes the non-ASCII characters (`"Pok\u00e9mon"`): add `ensure_ascii=False` to keep them readable.

#### Tables with pandas

In your project, you'll read most tables with pandas, which we'll see in the next lecture. As a preview:

```python
import pandas as pd

games = pd.read_csv("data/games.csv")
games[["title", "genres", "gameplay"]].head(3)
```

```text
                      title             genres  gameplay
0  The Witcher 3: Wild Hunt        Action, RPG         9
1        Grand Theft Auto V  Action, Adventure         8
2                  Portal 2    Shooter, Puzzle        10
```

`read_csv` handles the quotes, converts the numbers (`gameplay` is a column of integers: `games["gameplay"].mean()` gives 8.4375 straight away), and turns the empty values into missing values (`NaN`). It also writes tables: `games.to_csv("data/games_backup.csv", index=False)`; without `index=False`, the row numbers end up in the file, as an extra column with no name.

With the right parameters, `read_csv` and the other `read_` functions load many other files:

- `pd.read_csv(path, sep=";", decimal=",")`: the semicolon CSV files of Excel, with Italian settings;
- `pd.read_csv(path, encoding="latin-1")`: a file that is not in UTF-8 (pandas reads UTF-8 by default);
- `pd.read_csv(url)`: a file on the web (see [below](#downloading-a-dataset));
- `pd.read_json(path)`: a JSON file with a list of records; for nested data, load it with `json`, and flatten it with `pd.json_normalize(data)`;
- `pd.read_excel(path, sheet_name="Sheet1")`: a sheet of an Excel file (it needs `pip install openpyxl`);
- `pd.read_sql(query, connection)`: the result of a SQL query on a database, e.g. a SQLite file, through the `sqlite3` module of the standard library:

```python
import sqlite3

connection = sqlite3.connect("data/games.db")
games.to_sql("games", connection, if_exists="replace", index=False)  # write a table
query = "SELECT title, metacritic FROM games WHERE metacritic >= 95"
best = pd.read_sql(query, connection)  # Portal 2, Red Dead Redemption 2, ...
connection.close()
```

Remember: whenever you install a new library, add it to your `requirements.txt`.

## Handling Errors

Loading data is where most things go wrong: a missing file, a folder that doesn't exist, a JSON file that isn't valid, a server that doesn't answer. By default, an error stops your program with a traceback, as we saw in the debugging section of the project organization lecture. With `try` and `except`, you decide what happens instead.

#### `try` and `except`

```python
import json

try:
    with open("data/settings.json", encoding="utf-8") as f:
        settings = json.load(f)
except FileNotFoundError:
    settings = {}  # no settings saved yet: use the defaults
```

Python runs the `try` block. If one of its lines raises a `FileNotFoundError`, Python jumps to the `except` block, and goes on from there, instead of stopping; any other error still stops the program, with its traceback. These are the errors that you'll meet most often while loading data:

| Error | When |
|---|---|
| `FileNotFoundError` | the file, or the folder where you write it, doesn't exist |
| `PermissionError` | you can't open the file: e.g., on Windows, it's open in Excel |
| `UnicodeDecodeError` | the file was written with another encoding |
| `json.JSONDecodeError` | the text is not valid JSON |
| `KeyError` | a key or a column is missing, e.g. `row["title"]` |
| `ValueError` | a value can't be converted, e.g. `int("n/a")` |

An `except` can catch several errors, listed in a tuple, and give you the error itself with `as`, e.g. to show its message:

```python
try:
    with open("data/hades.json", encoding="utf-8") as f:
        rating = json.load(f)
except (FileNotFoundError, json.JSONDecodeError) as error:
    print(f"Could not load the rating: {error}")
```

If the file contains `{'title': 'Hades'}` (Python, not JSON: JSON wants double quotes), this prints `Could not load the rating: Expecting property name enclosed in double quotes: line 1 column 2 (char 1)`.

#### What to Do with an Error

Catching an error is easy; deciding what to do with it is the real work.

- **Go on with a default**, when the situation is normal. In vgrank, a missing or empty `games.csv` just means that no game has been rated yet:

    ```python
    # In vgrank/collection.py: the load method of GameCollection (simplified)
    def load(self):
        """Read the games from the CSV file: no file, or an empty one, means no games."""
        self.games = []

        if not os.path.exists(self.csv_path):
            return self

        try:
            table = pd.read_csv(self.csv_path)
        except pd.errors.EmptyDataError:  # the file exists, but it's empty
            return self

        for _, row in table.iterrows():
            self.games.append(game_from_row(row))
        return self
    ```

- **Stop, with a clear message**, when your program can't go on: say what went wrong, and how to fix it. You `raise` the error yourself, as we saw in the project organization lecture:

    ```python
    def load_books(path):
        """Load the books dataset, which must be downloaded first."""
        if not os.path.exists(path):
            raise FileNotFoundError(f"{path} not found: run download_books() first")
        return pd.read_csv(path)
    ```

- **Never hide an error.** An `except` that catches every error and does nothing turns a crash into a silent bug:

    ```python
    # Don't: every error disappears, even a typo in the path
    try:
        games = pd.read_csv(path)
    except Exception:
        pass
    ```

    The program goes on without the data, and fails later, far from the cause (here, with a `NameError` as soon as you use `games`), or never fails, and gives wrong results. Catch only the errors that you expect, and that you know how to handle.

## Data from the Web

Most of the data of the project proposals are on the web: files to download, like the datasets on GitHub, or APIs to query, like Open-Meteo and the World Bank.

#### Requests and Responses

When you open a web page, your browser (the **client**) sends a **request** to a computer on the internet (the **server**), which sends back a **response**. Your code can do the same, with the protocol of the web, **HTTP** (**HTTPS** is its encrypted version). A request has:

- a **URL**, the address of the resource, which can end with some **query parameters**, after a `?` and separated by `&`: `https://archive-api.open-meteo.com/v1/archive?latitude=45.46&longitude=9.19`;
- a **method**: `GET` asks for a resource (a page, a file, some data), `POST` sends data to the server (e.g., a form). To get data, you'll almost always use `GET`;
- some optional **headers**, with extra information, like who you are, or your API key.

The response has a **status code**, which says how it went, and a **body**: an HTML page, a file, or some data, usually in JSON.

| Code | Meaning |
|---|---|
| `200` | OK |
| `400` | Bad Request: e.g., a missing or invalid parameter |
| `401`, `403` | Unauthorized, Forbidden: a missing or invalid API key, or no permission |
| `404` | Not Found: a wrong URL |
| `429` | Too Many Requests: slow down |
| `500`, `503` | a problem of the server: try again later |

In Python, requests are sent with the `requests` library. We installed it in vgrank in the environments lecture; otherwise, run `pip install requests`, and add it to your `requirements.txt`.

#### Downloading a Dataset

Most datasets are just files on the web: on GitHub (copy the URL of the *Raw* button of the file), or on the website of their source. As the README says, don't commit large datasets to your repository: write a function that downloads them, and explain in the README how to run it.

```python
import os

import requests

BOOKS_URL = "https://raw.githubusercontent.com/zygmuntz/goodbooks-10k/master/books.csv"
BOOKS_PATH = os.path.join("data", "books.csv")


def download_file(url, path):
    """Download a file from the web, unless it is already on disk."""
    if os.path.exists(path):
        return
    response = requests.get(url, timeout=30)
    response.raise_for_status()
    os.makedirs(os.path.dirname(path), exist_ok=True)
    with open(path, "wb") as f:
        f.write(response.content)


download_file(BOOKS_URL, BOOKS_PATH)
```

- `requests.get` sends a `GET` request, and returns the response. Always set a `timeout`, in seconds: without it, a server that doesn't answer blocks your program forever.
- `raise_for_status()` raises an error if the status code is an error (`4xx` or `5xx`): otherwise, you would save an error page as if it were your dataset.
- `response.content` is the body, as bytes, written as it is in binary mode (`"wb"`); `response.text` is the body as a string.
- The file is downloaded only once: the next runs, and your notebook, read it from the disk. List it in your `.gitignore` (e.g., `data/books.csv`, or a whole folder, like `data/raw/`).

The goodbooks-10k dataset is the one of the *Page Turners* proposal: `pd.read_csv(BOOKS_PATH)` loads its 10,000 books. In your project, functions like `download_file` belong to a module of the package (e.g., `data.py`), and the notebook calls them.

> **NOTE**: `pd.read_csv(BOOKS_URL)` also reads a CSV file directly from the web: handy for a quick look, but it downloads the file again at every run. Some sources, like Kaggle, require an account: download their files by hand, and explain in the README where to get them, and where to put them.

#### APIs

An **API** (Application Programming Interface) lets your code ask a server for exactly the data it needs, usually in JSON, instead of a whole web page. Each API has its documentation, which lists its **endpoints** (the URLs of the different resources) and their parameters. Before scraping a website, always look for an API, or for a dataset to download: it's faster, more reliable, and it's what the data provider wants you to use.

Let's ask the [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) (the one of the *Weather Report* proposal, which doesn't need a key) for the weather in Milan, at the beginning of July 2025:

```python
import pandas as pd
import requests

ARCHIVE_URL = "https://archive-api.open-meteo.com/v1/archive"

params = {
    "latitude": 45.46,
    "longitude": 9.19,
    "start_date": "2025-07-01",
    "end_date": "2025-07-05",
    "daily": "temperature_2m_max,precipitation_sum",
    "timezone": "Europe/Rome",
}

response = requests.get(ARCHIVE_URL, params=params, timeout=10)
response.status_code  # 200
response.url
```

```text
https://archive-api.open-meteo.com/v1/archive?latitude=45.46&longitude=9.19&start_date=2025-07-01&end_date=2025-07-05&daily=temperature_2m_max%2Cprecipitation_sum&timezone=Europe%2FRome
```

`requests` builds the URL from the `params` dictionary, and encodes the special characters for you (`,` becomes `%2C`, `/` becomes `%2F`). The body is JSON: `response.json()` converts it into Python objects.

```python
data = response.json()
list(data)
```

```text
['latitude', 'longitude', 'generationtime_ms', 'utc_offset_seconds', 'timezone', 'timezone_abbreviation', 'elevation', 'daily_units', 'daily']
```

To find your way in a response, print its keys, or read the documentation. Here, `data["daily"]` contains a list of values for each variable (and `data["daily_units"]` their units: °C and mm), which pandas turns into a table:

```python
weather = pd.DataFrame(data["daily"])
weather
```

```text
         time  temperature_2m_max  precipitation_sum
0  2025-07-01                35.2                0.9
1  2025-07-02                34.4                1.0
2  2025-07-03                35.1                0.1
3  2025-07-04                36.1                0.0
4  2025-07-05                29.8               14.9
```

A heatwave, and then a storm. When something is wrong with a request, the API answers with an error code, and usually explains the problem in the body. With a typo in the name of a variable (`"daily": "temperature_max"`), `response.status_code` is `400`, and `response.json()` is:

```text
{'error': True, 'reason': 'Invalid value: Cannot initialize ForecastVariableDaily from invalid String value temperature_max'}
```

while `response.raise_for_status()` raises `requests.exceptions.HTTPError: 400 Client Error: Bad Request for url: ...`.

A few more things you'll meet:

- **Rate limits.** Free APIs limit the number of requests (per minute, per day): above the limit, you get a `429`. Ask for many data with each request (e.g., a whole year, rather than a request per day), pause between requests (`time.sleep(1)`), and save the responses to a file, so that you don't download them again at every run.
- **Pagination.** APIs that return long lists split them into pages: the documentation explains how to ask for the next one (e.g., a `page` parameter, or a `next` URL in the response), and you request them in a loop, until there are no more pages.
- **Client libraries.** Some sources have a Python library that sends the requests for you, like `wbgapi` for the World Bank, and `fastf1` for Formula 1 (see the project proposals). If there is one, use it.

#### API Keys

Many APIs, like the [RAWG API](https://rawg.io/apidocs) where vgrank searches its games, require a **key**: a secret string that identifies you, which you get by registering on their website. The key must never be written in your code, and thus never end up on GitHub: anyone could use it in your name. Store it in a file named `.env`, in the root of the project, and list `.env` in your `.gitignore` (see the Git lecture):

```text
RAWG_API_KEY=your_key_here
```

Then, read it with the `python-dotenv` library (`pip install python-dotenv`):

```python
import os

from dotenv import load_dotenv

load_dotenv()  # reads the variables written in .env
api_key = os.environ.get("RAWG_API_KEY")  # None, if the variable is not set
```

`load_dotenv` copies the variables of `.env` into the **environment variables** of your program (like `PATH`, in the shell lecture), and `os.environ.get` reads them. Then, pass the key as the documentation of the API says: RAWG wants it as a parameter named `key` (without it, the answer is a `401`).

```python
params = {"key": api_key, "search": "hades", "page_size": 5}
response = requests.get("https://api.rawg.io/api/games", params=params, timeout=10)
```

How can others run your project, then? Commit a file named `.env.example`, with the names of the variables but not their values, and explain in the README how to get a key and create `.env`, as in the *Setup* section of the [vgrank README](https://github.com/sergiopicascia/vgrank#setup).

> **NOTE**: `load_dotenv()` looks for `.env` in the folder of the code that calls it (in a notebook, in the working directory), and then in the folders above it: with `.env` in the root of the project, both the modules of the package and the notebook find it. And remember the lecture on coding with LLMs: never paste your keys in a chat.

#### When the Network Fails

A request can fail in many ways: no connection, a server that doesn't answer, a wrong key, too many requests. `requests` raises errors that you can catch: they all derive from `requests.exceptions.RequestException`, and the most common are `ConnectionError` (no connection, or a wrong address), `Timeout` (no answer within the timeout) and `HTTPError` (raised by `raise_for_status()`). This is how vgrank searches games on RAWG:

```python
# In vgrank/api.py (simplified)
def search_games(query, max_results=10):
    """Search games by name on RAWG."""
    params = {"key": get_api_key(), "search": query, "page_size": max_results}

    try:
        response = requests.get(BASE_URL, params=params, timeout=TIMEOUT_SECONDS)
        response.raise_for_status()  # turns HTTP errors (401, 500, ...) into exceptions
    except requests.exceptions.Timeout:
        raise ApiError("The RAWG API did not answer in time, please retry")
    except requests.exceptions.RequestException as error:
        raise ApiError(f"Could not reach the RAWG API: {error}")

    data = response.json()
    return [parse_game(item) for item in data.get("results", [])]
```

Python checks the `except` blocks in order, and runs the first one that matches: the specific errors go first, since a `Timeout` is also a `RequestException`. `ApiError` is an error defined by vgrank, with `class ApiError(Exception)`: the web app catches it, and shows its message to the user, instead of crashing. `get_api_key` raises it too, with the instructions to get a key, when `RAWG_API_KEY` is not set. Have a look at the whole [`vgrank/api.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/api.py), which also handles an answer that is not valid JSON.

## Web Scraping

When the data are only on a web page, with no API and no dataset to download, you can **scrape** them: download the HTML of the page, and extract the data from it. HTML (HyperText Markup Language) describes the structure of a page with nested **tags**, like `<h2>` (a heading), `<a href="...">` (a link) or `<span class="...">`: the attributes `class` and `id` are what you'll use to find your data.

#### Before You Scrape

- **Look for an API, or a dataset, first**: scraping is slower, and it breaks whenever the website changes its layout.
- **Read the rules**: the terms of service of the website, and its `robots.txt` file (e.g., `https://www.python.org/robots.txt`), which says which pages automated programs may visit.
- **Be gentle**: pause between requests (`time.sleep(1)`), and download each page only once, saving it to a file.
- **Identify yourself**, with the `User-Agent` header: many websites reject the anonymous requests of scripts. Wikipedia, for instance, answers `403 Forbidden` to the default `User-Agent` of `requests`.
- **Mind the data**: personal data (names, profiles, reviews signed by real people) are protected by the GDPR, even when anyone can see them, and the content of a website may be copyrighted: don't republish it.

#### Tables

If the data are in an HTML table, like in many Wikipedia pages, pandas extracts it for you (it needs `pip install lxml`):

```python
from io import StringIO

import pandas as pd
import requests

HEADERS = {"User-Agent": "vgrank (student project; your.name@example.com)"}
URL = "https://en.wikipedia.org/wiki/List_of_best-selling_video_games"

response = requests.get(URL, headers=HEADERS, timeout=10)
response.raise_for_status()

# The tables of the page that contain the text "Sales"
tables = pd.read_html(StringIO(response.text), match="Sales")
best_selling = tables[0]
best_selling[["Rank", "Title", "Sales (millions)"]].head()
```

```text
   Rank                  Title  Sales (millions)
0  1[a]                 Tetris             520.0
1  1[a]              Minecraft             425.0
2     2     Grand Theft Auto V             230.0
3     3  Red Dead Redemption 2              87.0
4     4         Wii Sports [b]              82.9
```

`read_html` returns a list, with a DataFrame for each table. It wants a path, a URL or a file: `StringIO` turns the text of the response into a file in memory. Scraped data usually need some cleaning, like the footnotes `[a]` and `[b]` here: we'll see how with pandas.

#### Other Data: BeautifulSoup

For anything else, **BeautifulSoup** (`pip install beautifulsoup4`) parses the HTML of the page, and lets you search its elements. Let's collect the latest offers of the [Python Job Board](https://www.python.org/jobs/). First, look at the page in the browser: right-click on the title of a job, and choose *Inspect*. Each job is an `<li>` element, in a list with the class `list-recent-jobs` (simplified):

```html
<li>
  <h2 class="listing-company">
    <span class="listing-company-name">
      <a href="/jobs/1234/">Data Engineer</a><br/>
      Example Ltd
    </span>
    <span class="listing-location">
      <a href="/jobs/location/milan-italy/">Milan, Italy</a>
    </span>
  </h2>
  <span class="listing-posted">Posted: <time datetime="2026-10-14">14 October 2026</time></span>
</li>
```

With the same `HEADERS` as above:

```python
from bs4 import BeautifulSoup

response = requests.get("https://www.python.org/jobs/", headers=HEADERS, timeout=10)
response.raise_for_status()
soup = BeautifulSoup(response.text, "html.parser")

links = soup.select(".list-recent-jobs h2 a")
len(links)  # 50
```

`select` takes a **CSS selector**: `.list-recent-jobs` matches the elements with that class, and `h2 a` the links inside an `<h2>`, inside them. But the page shows 25 jobs, and we got 50 links: the selector also matches the links to the locations (the second link is the location of the first job). Another silent bug: always count your results. A more precise selector, applied to each job, extracts its fields:

```python
rows = []
for item in soup.select(".list-recent-jobs li"):
    rows.append(
        {
            "title": item.select_one(".listing-company-name a").get_text(strip=True),
            "location": item.select_one(".listing-location a").get_text(strip=True),
            "posted": item.select_one("time")["datetime"][:10],
        }
    )

jobs = pd.DataFrame(rows)
jobs.head(3)
```

```text
                              title               location      posted
0    Software Engineer - Full Stack  Remote, United States  2026-10-14
1  Junior Machine Learning Engineer      Remote, Worldwide  2026-10-14
2         Machine Learning Engineer      Remote, Worldwide  2026-10-14
```

`select_one` returns the first matching element (or `None`, if nothing matches); `get_text(strip=True)` extracts its text, and `["datetime"]` the value of one of its attributes. The jobs change every day, so your output will be different.

#### Dynamic Pages

Many websites are **dynamic**: the HTML sent by the server is almost empty, and JavaScript, running in the browser, downloads the data and builds the page. `requests` doesn't run JavaScript, so it doesn't see those data. [quotes.toscrape.com](https://quotes.toscrape.com/), a website made for practising scraping, has a static and a dynamic version of the same page:

```python
for url in ["https://quotes.toscrape.com/", "https://quotes.toscrape.com/js/"]:
    response = requests.get(url, timeout=10)
    soup = BeautifulSoup(response.text, "html.parser")
    print(url, len(soup.select(".quote")))
```

```text
https://quotes.toscrape.com/ 10
https://quotes.toscrape.com/js/ 0
```

In the browser, both show 10 quotes. You have two options:

- **Find where the data come from.** Open the developer tools of the browser (*Inspect*), go to the *Network* tab, filter by *Fetch/XHR*, and reload the page: you'll often find the JSON that JavaScript downloads, at a URL that you can request with `requests`, like an API. Otherwise, look at the source of the page (right-click, *View Page Source*): the data may be written inside a `<script>`, as in the dynamic version of quotes.toscrape.com.
- **Control a real browser**, with [Selenium](https://www.selenium.dev/documentation/webdriver/getting_started/) or [Playwright](https://playwright.dev/python/): they open the page in Chrome or Firefox, run its JavaScript, and let you find the elements, much like BeautifulSoup. They are slower and more complex, so keep them as a last resort. Since version 4.6, Selenium only needs the browser to be installed: the *driver* that controls it is downloaded automatically.

## Useful Resources

- [Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files) and [Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html), from the official Python tutorial
- The [`csv`](https://docs.python.org/3/library/csv.html), [`json`](https://docs.python.org/3/library/json.html) and [`sqlite3`](https://docs.python.org/3/library/sqlite3.html) modules
- [IO tools](https://pandas.pydata.org/docs/user_guide/io.html), from the pandas documentation
- [Requests: Quickstart](https://requests.readthedocs.io/en/latest/user/quickstart/)
- [HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status), from MDN
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) and [RAWG API](https://rawg.io/apidocs)
- [Beautiful Soup documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [Inspect network activity](https://developer.chrome.com/docs/devtools/network), in the Chrome DevTools
- [Selenium](https://www.selenium.dev/documentation/webdriver/getting_started/) and [Playwright for Python](https://playwright.dev/python/)
- [`vgrank/api.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/api.py) and [`vgrank/collection.py`](https://github.com/sergiopicascia/vgrank/blob/main/vgrank/collection.py), the I/O code of vgrank
