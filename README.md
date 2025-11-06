# Python Exercise Classes
Resources for the Python module exercise classes of the *Coding for Data Science and Data Management* course.

Contacts: [sergio.picascia@unimi.it](mailto:sergio.picascia@unimi.it), [stefano.montanelli@unimi.it](mailto:stefano.montanelli@unimi.it)

EXAM REGISTRATION ARE OPEN. Please fill in this [form](https://forms.gle/fsiJq9dr5T7yXoM18) before November 24th to register for the exam.

## Exam Modality
As an alternative to the written exam of the Python module, it is possible to submit a project. 

Only individual project are possibile; group projects are not allowed.

The project only covers the Python module. The exams of R and data management are exclusively written and can be only taken in the ordinary exam sessions.

A list of possible traces for the Python project will be presented in the Python lab class of **September 29th, 2025**. A student can propose a project idea not in the list. The proposal must be submitted by email to prof. Montanelli and prof. Picascia (responsible of the Python lab) after the publication of official traces.

The final version of the project must be submitted by **November 30th, 2025** (firm deadline, do not ask for extensions). Modalities for the submission will be published in the diary of Python classes in due time. Project updates after the deadline will be not considered in the evaluation.
The students that submit a project will be called for a discussion that will be scheduled in early December. 

After the discussion, the student receives a grade that is valid as a result of the Python module. 
If not satisfied with the result, a student must immediately reject the project grade and take the written exam of the Python module in the ordinary exam sessions starting from December 10, 2025.

Students not interested in submitting a project must take the written exam in any ordinary exam session.

The submission of a project is possible only by the deadline of November 30, 2025. After this deadline, only written exams in ordinary sessions are possible for all the course modules.

## Calendar

- Sep 29th, 16:30-18:00 - *Projects Introduction, Shell*
- Oct 6th, 16:00-17:30 - *Git, Python Environments*
- Oct 13th, 16:00-17:30 - *Project Organisation*
- Oct 20th, 16:00-17:30 - *Data Input/Output*
- Oct 27th, 14:30-16:00 - *Data Manipulation*
- Nov 3rd, 16:00-17:30 - *Scientific Computing*
- Nov 10th, 16:00-17:30 - *Data Visualisation*
- Nov 17th, 16:00-17:30 - *Dashboard*

**NOTE**: this is a tentative outline and and may be subject to changes.

#### Important Dates

- Registration for the project evaluation will open on **November 3rd**, and will close on **November 24th**.
- The deadline for submitting the final version of the project, i.e. the last GitHub commit, is **November 30th, 23:59 CEST** (Milan local time).
- Project evaluations will start from **December 1st, 2025**.

## Evaluation Criteria

The project will be evaluated based on the following seven criteria. The total score is 33 points. Each component is designed to assess a key skill covered in the course.

#### 1. GitHub Usage (5 points)
This criterion assesses your ability to use Git and GitHub for version control, a fundamental practice in modern software and data science development. The project should be hosted in a GitHub repository with: a clear and informative `README.md` file, explaining what the project is about, listing important information (e.g., data sources, libraries), and providing clear instructions on how to setup and run the code; a well-configured `.gitignore` file to exclude unnecessary files; a meaningful commit history, with frequent and atomic commits, having descriptive messages. 

#### 2. Project Organization (5 points)
This criterion evaluates the structure and quality of your codebase. A well-organized project should: be logically organized into modules/scripts; have functions and classes with clear docstrings explaining their purpose, parameters, and return values; respect Python style guide, using consistent naming conventions, proper indentation, and clear variable names; provide a `requirements.txt` file, listing all the necessary Python libraries and their versions.

#### 3. Input/Output (5 points)
This criterion assesses your ability to work with data from external sources. The project should successfully load data from at least one external source (e.g., CSV, JSON, Excel, a database, or an API). The chosen data source should be appropriate for the analytical goals of the project. The code should handle data loading cleanly, addressing possible errors. The project may also demonstrate output capabilities by saving or displaying results, such as a summary table or generated plots.

#### 4. Data Manipulation (5 points)
This criterion focuses on the core task of handling data, evaluating your ability to use a data manipulation library like `pandas` to clean, transform, reshape, and prepare your data for the analysis and visualization stages.

#### 5. Scientific Computing (5 points)
This criterion assesses your ability to perform numerical and statistical computations on your data, using libraries like `numPy` or `sciPy` to extract quantitative insights, perform calculations, and prepare data for more complex modeling.

#### 6. Visualisation (5 points)
This criterion evaluates your ability to communicate your findings visually. This involves creating clear and informative plots that reveal patterns, trends, and insights using a visualisation library, such as `matplotlib` or `seaborn`.

#### 7. BONUS: Web Application (3 points)
This optional criterion offers a chance to earn bonus points by making your project interactive and accessible. By wrapping your analysis in a simple web application, using a framework like `streamlit`, you could create a dynamic dashboard that allows users to explore your findings in an engaging way.

## Project Proposals

You can choose among one of the following proposals, or propose your own: in the second case, please contact Prof. Montanelli and Picascia describing your proposal for approval.

**NOTE**: the project proposals are only ideas. The actual implementation is up to you. Thus, you are free to change part of the proposals, as long as the above criteria are still met in the final implementation.

#### 1. The Wealth of Nations
Dive into the intricate relationship between a country's economic prosperity and the well-being of its population. This project invites you to explore decades of global development data to uncover trends and correlations. You could analyze how indicators like GDP per capita relate to life expectancy, healthcare spending, or child mortality rates. Create compelling visualizations, such as time-series charts or global maps, to tell a story about global health and economics. The goal is to build an analytical project that reveals patterns in how nations thrive. Data for this project can be sourced directly from the [**World Bank Open Data**](https://data.worldbank.org/) portal or via its [Python API](https://pypi.org/project/wbgapi/).

#### 2. Market Pulse
Explore the world of financial markets by analyzing the behavior of different stock market sectors. This project involves fetching and processing historical stock price data to uncover performance trends and risk profiles. You could investigate which sectors are most volatile, how they correlate with each other, or how they perform during market shifts. Create insightful financial charts, like volatility plots or correlation heatmaps, to visualize market dynamics. The goal is to build a project that reveals the underlying structure and behavior of the stock market. Historical stock data can be easily accessed using the [`yfinance`](https://pypi.org/project/yfinance/) library, which pulls from the [**Yahoo Finance API**](https://finance.yahoo.com/).

#### 3. Who Wants to Be a Millionaire
The purpose of the project is to automatically create multiple-choice quizzes. The questions could be about any topic, or focus on one specific subjects, like movies, exploiting data from domain specific sources, e.g. [IMDb](https://www.imdb.com/). Each quiz must consist of one question and four possible answers, only one of which must be correct. The program that generates the quizzes must also implement a criterion to generate the possible answers proportional to the difficulty of the desired quiz. The program must also allow a human player to answer the quiz and must measure his/her performance with an overall score that takes into account the difficulty of each individual question.
IMDb data can be acquired through specific APIs (such as [Cinemagoer](https://cinemagoer.github.io)) or existing datasets, such as [IMDb Dataset](https://www.kaggle.com/datasets/ashirwadsangwan/imdb-dataset).

#### 4. Anatomy of a Blockbuster
What makes a movie a hit with audiences and critics? This project explores a vast dataset of films to find out. You will analyze factors like genre, budget, runtime, and release date to see how they connect to box office revenue and ratings. Investigate whether there's a "formula" for success or how the film industry has changed over the years. Visualize your findings with engaging plots that compare genres, show budget vs. revenue, or chart profitability. Your challenge is to use data to uncover the hidden trends behind cinematic triumphs. A comprehensive dataset for this analysis can be found in [**The Movies Dataset**](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset) on Kaggle.

#### 5. Moneyball FC
Step into the role of a modern football analyst by using advanced event data to scout for talented players. This project moves beyond simple goals and assists to uncover deeper performance metrics from professional matches. You could develop your own metrics, analyze player positioning with shot maps, or identify players who outperform expectations. Visualize player performance with professional-looking pitch plots and comparative scatter graphs. The objective is to use data to find the hidden gems and statistical standouts in the world of football. Professional-grade event data is available for free from the [**StatsBomb Open Data**](https://github.com/statsbomb/open-data) repository on GitHub.

#### 6. The Sound of Change
Explore the evolution of popular music by analyzing the "audio features" of chart-topping hits from different eras. Using Spotify's data, you can investigate how metrics like danceability, energy, and acousticness have changed over time. Your analysis could compare the sonic profiles of the 80s with the 2020s or track the rise of certain musical trends. Visualize these sonic shifts with compelling time-series plots or radar charts to show how our music has transformed. Data can be obtained by querying the **Spotify API** using a Python library like [`spotipy`](https://spotipy.readthedocs.io/en/2.25.1/) or from public datasets on Kaggle, e.g. [**Spotify Song Attributes**](https://www.kaggle.com/datasets/geomack/spotifyclassification).

#### 7. The Art of Play
The [BoardGameGeek (BGG)](https://boardgamegeek.com/) website provides various data and statistics on the enthusiastic board game hobby. BGG users comment on games and optionally assign them a liking score between 0 and 10. The [**BoardGameGeek Reviews**](https://www.kaggle.com/datasets/jvanelteren/boardgamegeek-reviews) dataset contains millions of board game reviews. The purpose of the project is to produce a ranking of games in order of user liking. It should be noted that in order to obtain an adequate ranking it is necessary to consider not only the votes, but also the fact that different games may be associated with very different numbers of comments and thus votes from individual users. For a discussion of this point see, for example, [How Not To Sort By Average Rating](https://www.evanmiller.org/how-not-to-sort-by-average-rating.html). The project should propose its own strategy for sorting the games, also arguing its appropriateness in relation to the official BGG ranking, available [online](https://boardgamegeek.com/browse/boardgame).

#### 8. Around the World
Consider the [**World Cities**](https://www.kaggle.com/datasets/max-mind/world-cities-database?select=worldcitiespop.csv) dataset describing some of the world's major cities. Assume that it is always possible to travel from each city to the 3 nearest cities and that such travel takes 2 hours to the nearest city, 4 hours to the second nearest city, and 8 hours to the third nearest city. In addition, the trip takes an additional 2 hours if the destination city is in another country than the starting city and an additional 2 hours if the destination city has more than 200,000 inhabitants. Starting in London and always traveling east, is it possible to travel around the world by returning to London in 80 days? How long does this take at a minimum?

## Frequently Asked Questions

##### *I don't know which project to pick...*
Do not choose a project because it seems 'easier' or because your friends are picking that one. It will become a burden. Go for something that you like, a hobby, an interest. Have fun in building it: it is an opportunity for you to get experience in writing code.

##### *When should I start working on my project?*
*The best time to start was yesterday, and the second-best time is now.*
Programming requires practice and building a good project requires time and consistency. Start soon and do a little bit every week and you won't find any difficulties in delivering a good project.

##### *The project proposal says that I need to do X, Y and Z; what if I want to change Y with A?*
You are free to change part of the proposals, as long as the nature of the project is maintained, i.e. that the project criteria are still met in the final implementation. If you are unsure about that, please get in touch with the instructors.

##### *Am I restricted to using Pandas, NumPy, and Matplotlib, or can I use other libraries like Plotly, SciPy, or Scikit-learn?*
You can use other libraries, as long as they perform similar operations compared to the ones suggested.

##### *My dataset is very large and my machine is not able to process it. How should I handle it?*
It is not mandatory to use an entire dataset. You can use a subset of it, either sampled randomly or with a certain criterion, as long as the project is still coherent without the left-out data.

##### *What is the expected scope or complexity of the project? Am I supposed to build a machine learning model?*
The project will be evaluated on the quality of the code. Therefore, it is not necessary to build complex statistical or machine learning models: this is out of the scope of this course. 

##### *How much analysis is considered "enough"? Is one interesting chart sufficient?*
The bare amount of operations/plots/computations is not relevant by itself; the evaluation will consider the quality of their implementation and their appropriateness for the project.

##### *How do I officially submit my project?*
You should invite the instructor as collaborator to your GitHub repository: in this way, if you want, you can keep your repository private. Later during the course, the instructor will ask you to sign an online form in order to register for the project evaluation.

##### *When is the evaluation taking place? How will it happen?*
The evaluation will take place after the submission deadline, from December 1, 2025 onwards. The precise date(s) will be confirmed later, depending on the amount of registration received. The evaluation will be conducted online in one or multiple days: you will receive further instructions and the relative Teams link later.

##### *What is the format of the project presentation? What should I focus on during the evaluation?*
During the evaluation you will be asked to briefly (~5 min) present your project, either through a Jupyter notebook or a web application. Then, in the remaining time (~5 min), you will be asked some questions about your code.

##### *What if I cannot attend the evaluation on the specified date?*
There will probably be two or more rounds of evaluation, on different dates. The instructor will communicate the calendar after the registrations will be closed. If you cannot attend the evaluation on the time slot assigned to you, please contact one of your colleague in another time slot to arrange a swap. Then communicate the swap to the instructor.

##### *What about the usage of Large Language Models (LLMs) for code generation? Which is the policy on this regard?*
You are permitted to use AI-powered tools like LLMs (e.g. ChatGPT, Claude, Gemini) to assist you with your project. However, we have some recommendation.

We encourage you to use these tools as assistants: for instance, to suggest alternative ways to approach a problem, to identify some possible issues within your code or to generate template code; do not use them to generate entire scripts or the core logic of your analysis.

You must test, debug, and validate any code you use, either generated by a LLM or taken from online sources, e.g. StackOverflow. These code snippets can sometimes be buggy, inefficient, use outdated practices, or are logically flawed.

Most importantly, **you must fully understand what every part of your code does**. During the project presentation, you will be expected to explain your code, justify your design choices, and answer questions about its implementation. Answering these questions correctly is mandatory for passing the exam.
