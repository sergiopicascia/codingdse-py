# Working with Local Files

At the most basic level, a file on your computer is just a sequence of bytes. Python provides simple, powerful tools for interacting with these files.

#### The `open()` function

The core function for file operations is `open()`. It takes a file path and a *mode* as its primary arguments.

```python
# open(file_path, mode)
f = open('my_file.txt', 'w')
```

**Common Modes:**

*   `'r'`: **Read** (default). Throws an error if the file doesn't exist.
*   `'w'`: **Write**. Creates the file if it doesn't exist. **Overwrites the entire file if it does exist.**
*   `'a'`: **Append**. Creates the file if it doesn't exist. Adds new content to the end of the file.
*   `'b'`: **Binary mode** (e.g., `'rb'`, `'wb'`). For non-text files like images or executables.
*   `'+'`: **Read and Write** (e.g., `'r+'`).

#### The `with` statement

When you open a file, you are creating a connection to it. It's crucial to **close** that connection when you're done to free up system resources and ensure your changes are saved.

You *could* do this manually:

```python
# The manual (and less safe) way
f = open('my_file.txt', 'w')
f.write('Hello, world!')
f.close() # You must remember to do this!
```

What if an error happens before `f.close()`? The file might be left open or corrupted.

The **best practice** in Python is to use the `with` statement, which automatically handles closing the file, even if errors occur.

```python
# The Pythonic way
with open('my_file.txt', 'w') as f:
    # The file is open inside this block
    f.write('This is the first line.\n')
    f.write('This is the second line.\n')
# The file is automatically closed once we exit the block
```

#### Writing and Appending to Files

```python
# Writing to a file (overwrites existing content)
lines_to_write = ['First line.', 'Second line.', 'Third line.']

with open('data/my_notes.txt', 'w') as f:
    for line in lines_to_write:
        f.write(line + '\n') # f.write() does not add newlines automatically

print("Wrote to my_notes.txt")

# Appending to a file (adds to the end)
with open('data/my_notes.txt', 'a') as f:
    f.write('This line was appended.\n')

print("Appended to my_notes.txt")
```

#### Reading from Files

There are a few ways to read data from a file.

```python
# Assuming 'data/my_notes.txt' exists from the previous step

# 1. Reading the entire file into one string
with open('data/my_notes.txt', 'r') as f:
    content = f.read()
    print("--- Reading entire file ---")
    print(content)

# 2. Reading line by line (memory efficient for large files)
with open('data/my_notes.txt', 'r') as f:
    print("--- Reading line by line ---")
    for line in f:
        print(line.strip())

# 3. Reading all lines into a list of strings
with open('data/my_notes.txt', 'r') as f:
    lines = f.readlines()
    print("--- Reading into a list ---")
    print(lines) # Note that lines will contain the newline characters '\n'
```

## Handling Structured Data: CSV and JSON

Plain text is simple, but most data has a structure. Two of the most common formats are CSV (for tabular data) and JSON (for nested data). Python's standard library has modules to handle both.

#### CSV (Comma-Separated Values)

CSV files are used for spreadsheet-like data. Each line is a row, and commas separate the columns.

While you *could* just `.split(',')` a line, the built-in `csv` module is much better because it correctly handles edge cases like commas within a quoted field.

**Example: Writing to a CSV**

```python
import csv

# Our data: a list of lists
header = ['name', 'department', 'year_born']
employees = [
    ['Alice', 'Engineering', 1990],
    ['Bob', 'Marketing', 1985],
    ['Charlie', 'Engineering', 1992]
]

with open('data/employees.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(header)      # Write the header row
    writer.writerows(employees)  # Write all the data rows
```

> **NOTE**: `newline=''` is a required argument to prevent blank rows from being written on some systems.

**Example: Reading from a CSV**

```python
import csv

with open('data/employees.csv', 'r') as f:
    reader = csv.reader(f)
    header = next(reader) # Skips the header row
    print(f"Header: {header}")
    for row in reader:
        # Each row is a list of strings
        print(f"{row[0]} works in {row[1]} and was born in {row[2]}.")
```

#### JSON (JavaScript Object Notation)

JSON is the standard for data exchange on the web. It's human-readable and maps very nicely to Python's dictionaries and lists.

*   JSON `object` -> Python `dict`
*   JSON `array` -> Python `list`
*   JSON `string` -> Python `str`
*   JSON `number` -> Python `int` or `float`

**Example: Writing a Python Dictionary to a JSON file**

```python
import json

# Our data: a Python dictionary
employee_data = {
    "name": "Alice",
    "department": "Engineering",
    "year_born": 1990,
    "skills": ["Python", "Data Analysis", "Git"]
}

with open('data/alice.json', 'w') as f:
    # json.dump() writes the object to a file-like object
    # indent=4 makes it human-readable
    json.dump(employee_data, f, indent=4)
```

**Example: Reading a JSON file into a Python Dictionary**

```python
import json

with open('data/alice.json', 'r') as f:
    # json.load() reads from a file-like object and parses it
    data = json.load(f)

print(type(data))
print(data)
print(f"Alice's second skill is: {data['skills'][1]}")
```

> Also useful: `json.dumps()` to convert a Python object to a JSON string, and `json.loads()` to parse a JSON string into a Python object.

# Accessing Data from the Web

Before we can grab data from the internet, we need a basic model of how it works.

*   **Client-Server Model**: When you visit a website, your browser (the **client**) sends a **request** to a computer on the internet (the **server**). The server finds the requested resource and sends a **response** back to your browser. Our Python script will act as the client.

*   **HTTP/HTTPS**: This is the protocol, or set of rules, for communication between clients and servers. `HTTPS` is just the secure version.

*   **HTTP Verbs**: These tell the server what kind of action the client wants to perform. The two most common are:
    *   `GET`: "Please give me this resource." (e.g., loading a webpage, fetching API data).
    *   `POST`: "Here is some data for you to process." (e.g., submitting a login form).

*   **HTML (HyperText Markup Language)**: The code that structures a webpage. It's a set of tags like `<p>` (paragraph), `<h1>` (heading), and `<a>` (link) that tell the browser what to display. We "scrape" data by parsing this HTML.

## APIs

An **API (Application Programming Interface)** is a structured way for programs to talk to each other. A web API is like a "menu" that a server provides for clients. Instead of ordering a full, messy webpage (HTML), you can ask for just the data you need, usually in a clean format like JSON.

**Always check for an API before you try to scrape a website!** It's more reliable, faster, and it's what the provider wants you to use.

#### The `requests` Library

The `requests` library is the de-facto standard for making HTTP requests in Python. It's not in the standard library, so we need to install it.

```shell
# In your terminal, with your virtual environment activated
pip install requests
```

**Example: Using `requests` to get data from a public API**

Let's use the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) API, a free service for testing.

```python
import requests
import json

# The URL for the resource we want (the 'todos' endpoint)
url = "https://jsonplaceholder.typicode.com/todos/1"

# Make a GET request
response = requests.get(url)

# Check the status of the response
print(f"Status Code: {response.status_code}") # 200 means OK!

# The API returns JSON, so we can use the built-in .json() method
# to automatically parse the response content into a Python dictionary
if response.status_code == 200:
    data = response.json()
    print("\n--- Response Data ---")
    print(json.dumps(data, indent=2))
    print(f"\nThe task is: '{data['title']}'")
    print(f"Is it completed? {'Yes' if data['completed'] else 'No'}")
else:
    print(f"Error: Failed to fetch data. Status code: {response.status_code}")

```

## Web Scraping

Sometimes, the data you want is visible on a webpage, but there's no public API to get it. In this case, we can write a script to **scrape** the data directly from the HTML of the page.

#### Ethical Considerations
1.  **Check `robots.txt`**: Most websites have a file at `www.example.com/robots.txt` that outlines rules for automated bots. Respect it.
2.  **Rate Limit**: Don't hammer a server with hundreds of requests per second. Add delays (`time.sleep()`) between requests to be polite.
3.  **Identify Yourself**: Set a `User-Agent` in your request headers to identify your script. It's more honest than pretending to be a normal browser.
4.  **Check Terms of Service**: Some sites explicitly forbid scraping in their ToS.
5.  **Don't Re-publish**: Be careful with copyrighted data.

#### Static vs. Dynamic Websites

*   **Static Site**: The server sends the full HTML, and the browser just displays it. We can often use `requests` to get the HTML and a library like `BeautifulSoup` to parse it.
*   **Dynamic Site**: The initial HTML is just a shell. **JavaScript** runs in the browser *after* the page loads to fetch data and build the page content. `requests` won't work here because it can't run JavaScript.

#### The `selenium` Library

For dynamic sites, we need a tool that can control a real web browser. That's what `selenium` does. It automates browser actions, so it sees exactly what a user would see.

**Setup:**

1.  Install the library:
    ```
    pip install selenium
    ```
2.  Download a **WebDriver**. This is the executable that `selenium` uses to control your browser. For Chrome, search for "chromedriver". Make sure its version matches your Chrome browser's version. Place the executable in your project directory or on your system's PATH.

**Example: Using `selenium` to find Python job titles on python.org**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service

# --- Setup ---
# Path to your chromedriver executable
# For simplicity, let's assume it's in the same directory.
# A better practice is to use a webdriver manager library.
SERVICE = Service('./chromedriver') # Update this path
driver = webdriver.Chrome(service=SERVICE)

# The URL we want to scrape
url = "https://www.python.org/jobs/"

# --- Main Logic ---
try:
    # 1. Open the URL
    driver.get(url)

    # 2. Wait for the page to load
    # A simple wait. For real projects, use WebDriverWait for reliability.
    time.sleep(3)

    # 3. Find elements by a selector
    # We can use the browser's "Inspect Element" tool to find selectors.
    # In this case, job titles are inside <h2> tags within an element with class 'list-recent-jobs'.
    job_listings = driver.find_elements(By.CSS_SELECTOR, ".list-recent-jobs h2 a")

    print(f"Found {len(job_listings)} job titles:")

    # 4. Extract the text from the elements
    for job in job_listings:
        title = job.text
        link = job.get_attribute('href')
        print(f"- {title} ({link})")

finally:
    # 5. VERY IMPORTANT: Close the browser
    driver.quit()
```

This example shows the basic workflow: `get` the page, `find` the elements you want, and `extract` the data from them.