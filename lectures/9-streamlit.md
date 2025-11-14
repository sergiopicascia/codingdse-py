# Web Applications with Streamlit

Streamlit is an open-source Python library that makes it easy to create and share custom web apps. A Streamlit app is just a Python script. There is no hidden state, no callbacks, and no need to write HTML, CSS, or JavaScript.

Every time a user interacts with a widget (e.g., moves a slider), the script is re-run from top to bottom. This simple execution model makes it easy to reason about your application's logic.

## Getting Started

First, install Streamlit in your virtual environment.

```shell
pip install streamlit
```

Now, let's create our first app. Create a file named `app.py`:

```python
# app.py
import streamlit as st

st.title("Hello, Streamlit!")
st.header("This is a header")
st.write("This is a simple web app built with Python.")
```

To run the app, open your terminal, navigate to the file's directory, and run:

```shell
streamlit run app.py
```

Your web browser will open with your new application.

## Displaying Information

For our dashboard, we will use a simple CSV file of stock data. Please create a file named `stocks.csv` inside a `data/` directory with the following content:

```csv
Date,Ticker,Open,High,Low,Close,Volume
2022-01-03,AAPL,177.83,182.88,177.71,182.01,104487900
2022-01-04,AAPL,182.63,182.94,179.12,179.70,99310400
2022-01-05,AAPL,179.61,180.17,174.64,174.92,94537600
2022-01-03,GOOGL,144.47,145.42,143.76,145.07,33464000
2022-01-04,GOOGL,145.56,146.40,143.82,144.41,29194000
2022-01-05,GOOGL,144.25,144.25,137.52,137.65,58228000
2022-01-03,MSFT,335.35,338.00,329.78,334.75,28865100
2022-01-04,MSFT,334.83,335.20,326.12,329.01,32674300
2022-01-05,MSFT,325.86,326.07,315.98,316.38,40054300
```

Streamlit provides a variety of commands to display content on the screen.

*   `st.title()`, `st.header()`, `st.subheader()`: For adding titles and section headers.
*   `st.write()`: A magic command that can render almost anything: text, markdown, numbers, dataframes, and plots.
*   `st.dataframe()`: An interactive table for displaying dataframes.
*   `st.metric()`: For displaying a key performance indicator with an optional delta.
*   `st.pyplot()`: For displaying Matplotlib figures.

Let's build the first **static** version of our dashboard.

```python
# app.py
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# --- Page Configuration ---
st.set_page_config(
    page_title="Stock Market Dashboard",
    page_icon="💹",
    layout="wide"
)

# --- Data Loading ---
def load_data():
    df = pd.read_csv('data/stocks.csv')
    df['Date'] = pd.to_datetime(df['Date'])
    df = df.set_index('Date')
    return df

stock_df = load_data()

# --- Static Dashboard ---
st.title("💹 Stock Market Dashboard")

st.header("Apple (AAPL) Stock Prices")
st.write("A static view of Apple's stock data from our dataset.")

# Filter data for a specific stock
aapl_df = stock_df[stock_df['Ticker'] == 'AAPL']

# Display metrics
latest_close = aapl_df['Close'].iloc[-1]
prev_close = aapl_df['Close'].iloc[-2]
delta = latest_close - prev_close
st.metric(label="Latest Closing Price (AAPL)", value=f"${latest_close:.2f}", delta=f"{delta:.2f}")

# Display a plot
st.subheader("Price History")
fig, ax = plt.subplots()
ax.plot(aapl_df['Close'])
ax.set_title("AAPL Closing Price")
ax.set_xlabel("Date")
ax.set_ylabel("Price (USD)")
plt.xticks(rotation=45)
st.pyplot(fig)

# Display raw data
st.subheader("Raw Data")
st.dataframe(aapl_df)
```

Run `streamlit run app.py`. You now have a clean, static report about Apple's stock.

## Interactive Loop & Widgets

Streamlit's execution model is simple: **every time a user interacts with a widget, the script is re-run from top to bottom.** The value of the widget is assigned to a variable in that run.

Common widgets include:

*   `st.button()`: A clickable button.
*   `st.selectbox()`: A dropdown menu.
*   `st.slider()`: A numerical slider.
*   `st.date_input()`: A date picker.
*   `st.text_input()`: A text input box.
*   `st.checkbox()`: A checkbox.

Let's make our dashboard interactive. We want the user to select the stock ticker they are interested in.

```python
# (Keep the top part of the script from before)

st.title("💹 Stock Market Dashboard")

# --- Interactive Sidebar ---
st.sidebar.header("User Input")

# Get unique tickers from the dataframe
ticker_list = stock_df['Ticker'].unique()
selected_ticker = st.sidebar.selectbox("Select a Stock Ticker", ticker_list)

# --- Main Page Content ---
st.header(f"{selected_ticker} Stock Prices")
st.write(f"A dynamic view of {selected_ticker}'s stock data.")

# Filter data for the selected stock
ticker_df = stock_df[stock_df['Ticker'] == selected_ticker]

# Display metrics
# (Add logic to handle if there's less than 2 days of data)
if len(ticker_df) > 1:
    latest_close = ticker_df['Close'].iloc[-1]
    prev_close = ticker_df['Close'].iloc[-2]
    delta = latest_close - prev_close
    st.metric(label=f"Latest Closing Price ({selected_ticker})", value=f"${latest_close:.2f}", delta=f"{delta:.2f}")
else:
    st.info("Not enough data for metric calculation.")

# Display a plot
st.subheader("Price History")
fig, ax = plt.subplots()
ax.plot(ticker_df['Close'])
ax.set_title(f"{selected_ticker} Closing Price")
ax.set_xlabel("Date")
ax.set_ylabel("Price (USD)")
plt.xticks(rotation=45)
st.pyplot(fig)

# Display raw data with a checkbox toggle
if st.sidebar.checkbox("Show Raw Data"):
    st.subheader("Raw Data")
    st.dataframe(ticker_df)
```

Now, when you run the app, you'll see a sidebar. When you select a new ticker from the dropdown, the script reruns, `selected_ticker` gets the new value, and the entire page updates accordingly.

## Performance and Caching

Notice that our `load_data()` function is called every single time we interact with a widget. For our small CSV, this is fine. But what if loading the data involved a slow database query or a heavy computation? The app would feel sluggish.

Streamlit provides a powerful solution: **caching**. The `@st.cache_data` decorator tells Streamlit to store the result of a function in a local cache. When the function is called again with the *exact same inputs*, Streamlit skips executing the function and returns the stored result instantly.

Let's apply it to our data loading function.

```python
import time # To simulate a slow function

@st.cache_data # The magic decorator
def load_data():
    # Simulate a slow data loading process
    time.sleep(3)
    df = pd.read_csv('data/stocks.csv')
    df['Date'] = pd.to_datetime(df['Date'])
    df = df.set_index('Date')
    return df

# ... rest of the app ...
stock_df = load_data()
```

Now, run the app. The first time it loads, it will pause for 3 seconds. But every subsequent interaction (changing the ticker, checking the box) will be instantaneous because the `stock_df` is being retrieved from the cache.

## Multi-Page Applications

As your app grows, you might want to organize it into multiple pages. Streamlit makes this incredibly simple.

1.  In the same directory as your `app.py` script, create a new folder named `pages`.
2.  Any `.py` file you place inside the `pages` folder will automatically become a page in your app's sidebar navigation.
3.  The filenames are used to name the pages (e.g., `01_Home.py` becomes "Home").

Let's restructure our app:

**Directory Structure:**
```
.
├── data/
│   └── stocks.csv
├── app.py              (This will be our home page)
└── pages/
    └── 01_📈_Stock_Analysis.py
    └── 02_📊_Company_Comparison.py
```

**`app.py` (Home Page):**
```python
import streamlit as st

st.set_page_config(page_title="Stock Dashboard Home")

st.title("Welcome to the Interactive Stock Dashboard!")
st.write("Use the navigation on the left to explore different analyses.")
st.sidebar.success("Select an analysis page above.")
```

**`pages/01_📈_Stock_Analysis.py`:**
*This will be the main interactive script we just built.*

**`pages/02_📊_Company_Comparison.py`:**
*A new page for a different type of visualization.*
```python
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# Using the cached function from the main app is tricky.
# For simplicity, we define it here again.
# In larger apps, you would put this in a shared utility module.
@st.cache_data
def load_data():
    df = pd.read_csv('data/stocks.csv')
    df['Date'] = pd.to_datetime(df['Date'])
    return df

stock_df = load_data()

st.title("Company Comparison")

# Multi-select widget
selected_tickers = st.multiselect("Select Tickers to Compare", stock_df['Ticker'].unique())

if selected_tickers:
    # Filter data for selected tickers
    comparison_df = stock_df[stock_df['Ticker'].isin(selected_tickers)]
    
    # Plot closing prices
    st.subheader("Closing Price Comparison")
    fig, ax = plt.subplots(figsize=(10, 6))
    for ticker in selected_tickers:
        subset = comparison_df[comparison_df['Ticker'] == ticker]
        ax.plot(subset['Date'], subset['Close'], label=ticker)
    
    ax.legend()
    ax.set_xlabel("Date")
    ax.set_ylabel("Closing Price (USD)")
    st.pyplot(fig)
else:
    st.warning("Please select at least one ticker to compare.")
```

Now, when you run `streamlit run app.py`, you will see a fully-fledged multi-page application.

## Persisting Information with Session State

As we've learned, a Streamlit script reruns from top to bottom on every interaction. This simple model is powerful but has a limitation: variables are reset on each run. What if you need to remember something across reruns, such as a user's choice or the result of a previous action?

This is where **Session State** comes in. `st.session_state` is a dictionary-like object that persists across script reruns for the duration of a user's browser session.

*   It's unique to each user session.
*   It behaves like a Python dictionary. You can read, write, and delete keys.
*   It's the standard way to create stateful, multi-step applications.

Let's enhance our **Stock Analysis** page. We will add a feature that allows users to mark a stock as their "favorite." This choice should be remembered as they navigate the app or change other settings on the page.

Modify the `pages/01_📈_Stock_Analysis.py` file to include this new functionality.

```python
# pages/01_📈_Stock_Analysis.py
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import time

# --- (The load_data function remains the same) ---
@st.cache_data
def load_data():
    time.sleep(1) # Simulate a slightly slower load
    df = pd.read_csv('data/stocks.csv')
    df['Date'] = pd.to_datetime(df['Date'])
    df = df.set_index('Date')
    return df

stock_df = load_data()

# --- Initialize session state ---
# This is crucial. We initialize the 'favorite_ticker' key if it's not already present.
if 'favorite_ticker' not in st.session_state:
    st.session_state.favorite_ticker = None

st.title("📈 Single Stock Analysis")

# --- Display the favorite stock prominently if it exists ---
if st.session_state.favorite_ticker:
    st.info(f"Your favorite stock is: **{st.session_state.favorite_ticker}**")

# --- Interactive Sidebar ---
st.sidebar.header("User Input")
ticker_list = stock_df["Ticker"].unique().tolist()

# Determine the default index for the selectbox
default_index = 0
if st.session_state.favorite_ticker in ticker_list:
    # If a favorite is set and it's a valid ticker, find its index
    default_index = ticker_list.index(st.session_state.favorite_ticker)

# Create the selectbox with the dynamically set default index
selected_ticker = st.sidebar.selectbox(
    "Select a Stock Ticker", ticker_list, index=default_index
)

# --- Button to set the favorite stock, using session state ---
if st.sidebar.button("Set as Favorite"):
    st.session_state.favorite_ticker = selected_ticker
    st.sidebar.success(f"{selected_ticker} has been set as your favorite!")

# --- Main Page Content ---
st.header(f"{selected_ticker} Stock Prices")
ticker_df = stock_df[stock_df['Ticker'] == selected_ticker]

# --- (The rest of the page logic for metrics, plot, and raw data remains the same) ---
if len(ticker_df) > 1:
    latest_close = ticker_df['Close'].iloc[-1]
    prev_close = ticker_df['Close'].iloc[-2]
    delta = latest_close - prev_close
    st.metric(label=f"Latest Closing Price ({selected_ticker})", value=f"${latest_close:.2f}", delta=f"${delta:.2f}")
else:
    st.info("Not enough data for metric calculation.")

st.subheader("Price History")
fig, ax = plt.subplots()
ax.plot(ticker_df['Close'])
ax.set_title(f"{selected_ticker} Closing Price")
ax.set_xlabel("Date")
ax.set_ylabel("Price (USD)")
plt.xticks(rotation=45)
st.pyplot(fig)

if st.sidebar.checkbox("Show Raw Data"):
    st.subheader("Raw Data")
    st.dataframe(ticker_df)
```

How `session_state` works:

Of course. Here is the rewritten "How it works" section, which now reflects the more sophisticated use of `session_state` to both store data and control widget defaults.

---

### How it works: The Session State Lifecycle

This example demonstrates the complete lifecycle of state management in a Streamlit application, which can be broken down into four key stages:

1.  **Initialization:** The first time a user's session runs the script, the check `if 'favorite_ticker' not in st.session_state:` evaluates to `True`. This line creates the `favorite_ticker` key and sets its initial value to `None`. This block is ignored on every subsequent rerun for that session, preventing the user's stored choice from ever being reset.

2.  **Modification:** A user's action, in this case, clicking the "Set as Favorite" button, is the trigger to *write* to the session state. The line `st.session_state.favorite_ticker = selected_ticker` directly updates the value associated with our key. This is how the application "saves" the user's preference.

3.  **Persistence:** The advantage of `session_state` is that the value `'AAPL'` (or whichever ticker was chosen) is now stored for the entire user session. When the script reruns, whether because the user selects a different ticker, checks a box, or even navigates to another page and returns, the value of `st.session_state.favorite_ticker` is preserved.

4.  **Retrieval and Application:** On every rerun, the application *reads* the value from `st.session_state` to alter its own behavior. We see this in two places:
    *   **Direct Display:** The line `st.info(f"Your favorite stock is: ...")` reads the state to provide direct feedback to the user.
    *   **Dynamic Widget Configuration:** Before rendering the `st.selectbox`, the code checks if the stored `favorite_ticker` exists in our list of options. If it does, it calculates the corresponding `index` and passes it to the widget. This makes the selectbox automatically default to the user's saved preference.