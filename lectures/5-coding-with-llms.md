# Coding with LLMs

Large Language Models (LLMs), like ChatGPT, Claude, Gemini or GitHub Copilot, can write a working Python function in a few seconds, explain an error message, or turn a vague idea into a plot. You can use them for your project, within the rules of the course (see the [FAQ](../README.md#frequently-asked-questions) in the README):

- use them as **assistants**: to explain code and errors, to suggest other ways to approach a problem, to find issues in your code, to write template code;
- **don't** use them to generate entire scripts or the core logic of your analysis;
- **test and validate** any code that you use, whether it comes from an LLM or from the web;
- **understand every line** of your project: at the exam, you will be asked to explain your code and to justify your choices.

Why these rules? Because the goal of the project is that *you* learn to code, and you learn to code by writing code, like you learn maths by solving exercises. Delegating the thinking gets you working code, but no learning.

Used well, an LLM is a tutor that is always available, never tires of your questions, and can explain anything with an example built on your data. Used badly, it's a ghostwriter whose work you will have to defend at the exam.

> **NOTE**: this lecture is not about a specific tool: models change every few months, the principles don't.

## What LLMs Are Good (and Bad) At

An LLM is trained on huge amounts of text and code from the web, to continue a text in a plausible way. When you ask for code, it doesn't look it up and doesn't run it (unless the tool does it for you): it writes the code that *looks* most plausible. That's why it's so fluent, and also why it can be confidently wrong.

| LLMs are good at... | ...but they don't know |
|---|---|
| Common tasks, with thousands of examples online: reading a CSV, a `groupby`, a bar chart | Your data: they haven't seen it, unless you show it |
| Explaining code, error messages and concepts, with examples | Your computer: your OS, your Python version, which libraries are installed, where your files are |
| Suggesting libraries, functions and alternative approaches | The latest versions: they learned from old code, and may use functions that changed or no longer exist |
| Fixing an error, when you give them the whole error message | Their own limits: rather than saying "I don't know", they may invent functions, parameters and even libraries |
| Writing repetitive code: docstrings, plot formatting, a `.gitignore` | What your analysis needs: which rows to drop, how to rank, which chart answers your question |

Three more things to keep in mind:

- **They tend to agree with you.** Ask "is this code correct?", and you'll often get a yes. Ask "find the bugs in this code" instead.
- **They are not deterministic.** The same question can get different answers: if two answers disagree, it's a sign that you should check.
- **They only know the conversation.** A new chat usually starts from scratch: the decisions about your project belong in your code and in your README, not in a chat.

## The Tools

LLMs come in three forms:

- **Chats** (ChatGPT, Claude, Gemini, Microsoft Copilot, Le Chat, DeepSeek, ...): you ask in the browser or in an app, and you copy into your project only what you decide to copy. This is the setting of this lecture, and the one we recommend for your project.
- **Assistants in the IDE**, like GitHub Copilot in VS Code (free for verified students, through [GitHub Education](https://github.com/education)) or the AI assistant of PyCharm. They suggest the next lines while you type (*completions*), and have a chat panel that sees your files. Completions are handy for repetitive code, but they write the code *before* you have thought about it: while you are learning, consider turning them off (VS Code lets you do it from the Copilot menu in the status bar), and use the chat panel instead.
- **Agents** (the agent mode of Copilot, Cursor, Claude Code, Codex, Gemini CLI, ...): they read and edit your files on their own, and run commands on your computer. They can build a whole project in a few minutes, which is exactly what this course asks you not to do: don't let them write your project. If you use them for anything else, read every command they want to run before allowing it.

> **NOTE**: what you paste in a chat is sent to the servers of a company. Never paste API keys, passwords or your `.env` file, and avoid personal data. To show your data, a few rows are enough (see below).

## Asking Good Questions

#### A Naive Request

In vgrank, `ranking` is the DataFrame returned by `collection.ranking()`, with one row per game. Suppose that we want the average overall score of each genre, and we ask:

```text
Write Python code to compute the average score for each genre.
```

A typical answer:

```python
average_by_genre = df.groupby("genre")["score"].mean()
```

The model had to guess everything: the name of the DataFrame, the names of the columns, the format of the data. With the names of vgrank, the code runs without errors:

```python
ranking.groupby("genres")["overall"].mean()
```

```text
genres
Action, Adventure                7.866667
Action, Adventure, Simulation    6.400000
Action, RPG                      8.075000
Action, RPG, Indie               9.300000
...
```

But the result is wrong. A game can have several genres, stored in a single string (`"Action, RPG"`), so the code groups the games by their *combination* of genres: the output has 10 rows, but the data contains 8 genres. No error message will ever warn you: only looking at the output does.

#### The Ingredients of a Good Request

A good request has four ingredients:

- **Context:** who you are, what you are working on, with which tools. "I'm learning Python in a data science course; my project ranks video games; I use Python 3.13 and pandas 3.0, in a notebook." The model will write simpler code, explain it better, and target your versions.
- **Goal:** what goes in, what comes out, what success looks like. Not "analyze the genres", but "a function that receives the ranking and returns one row per genre, with the average overall score and the number of games".
- **Constraints:** what the code must (or must not) do: which libraries (only pandas, no new libraries), which structure (one function, with a docstring, that returns the result instead of printing it), how complex (simple code, that a beginner can read).
- **Material:** what the model can't know, unless you show it: a sample of your data, the types of the columns, your current code, the whole error message, the versions of your libraries, an example of the result you expect.

The material is the most underrated ingredient, and often the most important: with a few rows of data, the model *sees* that the genres are separated by commas, instead of guessing. This is how you collect it:

| To show... | ...run this, and paste the output |
|---|---|
| a sample of the data | `print(df.head().to_string())` |
| the type of each column | `print(df.dtypes)` |
| the size of the data | `print(df.shape)` |
| an error | nothing to run: copy the whole traceback, from `Traceback` to the last line |
| your versions | `python --version` and `pip show pandas` |

Before sending a request, ask yourself: *if a classmate received this message, would they have everything they need to help me?*

#### A Better Request

```text
I'm learning Python in a data science course. In my project (Python 3.13, pandas 3.0),
I have a DataFrame `ranking`, with one row per video game. These are its first rows:

                      title                     genres  overall
0                     Hades         Action, RPG, Indie      9.3
1             Hollow Knight  Platformer, Indie, Action      9.0
2  The Witcher 3: Wild Hunt                Action, RPG      8.9
3     Red Dead Redemption 2          Action, Adventure      8.6
4                  Portal 2            Shooter, Puzzle      8.2

Write a function `score_by_genre(ranking)` that returns a DataFrame with one row per
genre, and two columns: `mean_overall`, the average overall score of the games of that
genre, and `n_games`, the number of games of that genre. A game with two genres counts
for both. Sort the genres from the best to the worst.

Use only pandas. Write a NumPy-style docstring, and don't print anything: return the
result. Keep the code simple, and explain each step.
```

A typical answer:

```python
def score_by_genre(ranking):
    """
    Average overall score for each genre.

    Parameters
    ----------
    ranking : pandas.DataFrame
        One row per game, with the columns ``genres`` and ``overall``.

    Returns
    -------
    pandas.DataFrame
        One row per genre, with the columns ``mean_overall`` and ``n_games``,
        sorted from the best genre to the worst.
    """
    # One row for each (game, genre) pair: "Action, RPG" becomes two rows
    genres = ranking.assign(genre=ranking["genres"].str.split(", ")).explode("genre")

    # Average score and number of games of each genre
    scores = genres.groupby("genre")["overall"]
    result = scores.agg(mean_overall="mean", n_games="count")

    return result.sort_values("mean_overall", ascending=False)
```

The code is short, but it uses two methods that you may not know yet, `explode` and `agg`. Before using it, you must understand it (ask the model to show you what each step produces on a three-row example, and look the methods up in the documentation), and check it.

> **NOTE**: here we asked for a whole function, to show how much the ingredients of a request change the answer. But a function like this is part of the core logic of an analysis: in your project, use the same ingredients to ask for explanations and hints (see [Using LLMs to Learn](#using-llms-to-learn)), and write it yourself.

#### Working in Small Steps

The first answer is rarely the final one. Work in a loop, one small change at a time: **ask, read, run, check, commit**.

- **One problem at a time.** "Write the whole analysis of the genres, with plots and statistics" returns more code than you can check. Ask for one function, check it, then move on.
- **Describe what you see.** "It doesn't work" gives the model nothing to work with. "The result has 10 rows, but there are 8 genres: the games with several genres are grouped together" points it to the problem. Show the result you expected, and the one you got.
- **Limit the changes.** LLMs like to "improve" code that you didn't ask about. Say what must not change: "Modify only `score_by_genre`, and keep the rest of the code as it is."
- **Same chat, or a new one?** Stay in the same chat while you work on the same problem: the model remembers the code and the decisions taken. Start a new chat, with a new and complete request, when the conversation goes in circles: e.g., when the model proposes again a solution that you already rejected, or forgets your constraints.

Git is your safety net:

```bash
# Before pasting: start from a clean state, with everything committed
git status

# After pasting: see exactly what changed
git diff

# Not convinced? Throw the changes away...
git restore vgrank/analysis.py

# ...or, once checked, commit them
git add vgrank/analysis.py
git commit -m "Add average score by genre"
```

## Checking the Answer

#### Three Kinds of Errors

In the project organization lecture, we met crashes and silent bugs. Code written by an LLM has both, plus a third kind of error.

1. **The code crashes.** A `KeyError`, a `TypeError`, an `AttributeError`: the easy ones, because the traceback tells you where to look. Give the model the whole traceback, the code and your versions. An error on a function of a library may mean that the model used a function that doesn't exist (anymore): `df.append(...)`, common in old code, fails since pandas 2 with `AttributeError: 'DataFrame' object has no attribute 'append'`. Check the documentation of *your* version.

2. **The code runs, but the result is wrong.** The dangerous ones, because nothing warns you: the genres grouped by combination, an average divided by the wrong number, a filter that drops more rows than expected. Outdated code can fail silently too: LLMs learned from years of code written for older versions of pandas, and you may get lines like this one:

    ```python
    games["metacritic"].fillna(0, inplace=True)
    ```

    In pandas 3, it doesn't change `games` at all: it only shows a `ChainedAssignmentError` warning (so, read the warnings too!). In pandas 3, you write `games["metacritic"] = games["metacritic"].fillna(0)`.

3. **The code does what you asked, but you asked the wrong thing.** The code is correct; the analysis isn't. Even the fixed line above is questionable: the games without a Metacritic score now count as 0, so with three games scored 93, 89 and missing, the average drops from 91 to 60.7. Or take the *Rate My Recipe* proposal: ask for "the recipes sorted by their average rating", and you'll get perfect code, which puts a recipe with a single 5-star review above one with 500 reviews averaging 4.9. The model won't tell you, unless you ask: "Which assumptions did you make? What are the limitations of this approach?"

#### How to Check

- **Try a tiny example, whose result you computed by hand:**

    ```python
    sample = pd.DataFrame(
        {
            "title": ["A", "B", "C"],
            "genres": ["Action, RPG", "RPG", "Puzzle"],
            "overall": [8.0, 6.0, 9.0],
        }
    )
    score_by_genre(sample)
    ```

    ```text
            mean_overall  n_games
    genre
    Puzzle           9.0        1
    Action           8.0        1
    RPG              7.0        2
    ```

    RPG has two games, and (8 + 6) / 2 = 7: correct.

- **Try the edge cases:** empty data, missing values, a single row. A game without genres, for instance, silently disappears from the result of `score_by_genre`: probably fine, but now you know it, and you can say it at the exam.
- **Look at the output, and count:** `df.head()`, `len(df)`, `value_counts()`. How many rows before and after each step? As many genres as you expected?
- **Check plausibility:** remember the debugging example of the previous lecture? An average of 6.7 for three games, all scored above 8.5, can't be right.
- **Read the code, line by line:** if a line isn't clear, ask the model to explain it, and check the documentation of the functions (`help(pd.DataFrame.explode)`, or the website, with the version selector).

#### Decisions Are Yours

Some choices are not about code, but about your analysis: which data to use, how to handle missing values and duplicates, how to count a game with two genres, how to rank recipes with few reviews, which chart answers your question. An LLM will happily make these choices for you, silently. Ask it for the options, with their pros and cons (it's very good at that), but then choose yourself, and write down why: at the exam, "why did you drop these rows?" is a typical question, and "the LLM did it" is not an answer.

## Using LLMs to Learn

These requests make the model work as a tutor, not as a ghostwriter. Copy them, and fill in the brackets.

- **Explain an error, and let you fix it:**

    ```text
    I'm a beginner. This code: [code] gives this error: [whole traceback].
    Explain what the error means, and why it happens here. Don't fix the code:
    give me a hint, I want to fix it myself.
    ```

- **Explain some code**, e.g. a function of vgrank:

    ```text
    Explain this function line by line, for a beginner. Then show me what each step
    produces on a DataFrame with three rows. [code]
    ```

- **Give you hints, not solutions:**

    ```text
    I want to [goal] with pandas. Don't write the code: tell me which functions
    I should look up in the documentation, and in which order.
    ```

- **Review your code, without rewriting it:**

    ```text
    Review this function of my project. List possible bugs, edge cases (empty data,
    missing values) and style problems (PEP 8, docstring). Don't rewrite it:
    I'll fix it myself. [code]
    ```

- **Compare alternatives:**

    ```text
    What are two or three ways to [goal] in pandas? Compare them for a beginner:
    readability, speed, pitfalls.
    ```

- **Prepare the exam:**

    ```text
    I'm preparing the oral exam of my project. This is a function I wrote: [code]
    Ask me five questions that an examiner could ask about it, one at a time,
    and tell me whether my answers are correct and complete.
    ```

## When the LLM Is Not Enough

- When it keeps proposing fixes that don't work, stop asking: reduce the problem to a tiny example, read the error and the documentation, and write a new, complete request. Or ask a human.
- When your problem is a concept (what does `groupby` do? what is a z-score?), a chat is great for quick clarifications, but a systematic source (the official tutorial, a book chapter, the lecture notes) gives you a more solid understanding.
- The official documentation describes the version that you use: when the LLM and the documentation disagree, the documentation wins.
