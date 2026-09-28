# Python Exercise Classes
Resources for the Python module exercise classes of the *Coding for Data Science and Data Management* course, A.Y. 2026/27.

Contacts: [sergio.picascia@unimi.it](mailto:sergio.picascia@unimi.it), [stefano.montanelli@unimi.it](mailto:stefano.montanelli@unimi.it)

## Exam Modality
As an alternative to the written exam of the Python module, it is possible to submit a project.

**The project is individual**: group projects are not allowed.

The project only covers the Python module. The exams of R and data management are exclusively written and can be only taken in the ordinary exam sessions.

A list of possible traces for the Python project will be presented in the Python lab class of **October 15th, 2026**. A student can propose a project idea not in the list. The proposal must be submitted by email to Prof. Montanelli and Dr. Picascia after the publication of official traces.

The final version of the project must be submitted by **November 30th, 2026** (firm deadline, do not ask for extensions). Registration and submission modalities will be communicated during the course. Project updates after the deadline will be not considered in the evaluation.
The students that submit a project will be called for a discussion on **December 1st, 2nd or 3rd, 2026**.

After the discussion, the student receives a grade that is valid as a result of the Python module.
If not satisfied with the result, a student must immediately reject the project grade and take the written exam of the Python module in the ordinary exam sessions, starting from **December 9th, 2026**.

Students not interested in submitting a project must take the written exam in any ordinary exam session.

**The project can be submitted only in this session**, i.e. by the deadline of November 30th, 2026. After this deadline, only written exams in ordinary sessions are possible for all the course modules.

## Calendar

- Thu, Oct 15th, 16:30-18:30 - *Course and Projects Introduction*
- Mon, Oct 19th, 14:30-18:30 - *Project Organisation*
- Thu, Oct 29th, 16:30-18:30 - *Data Input/Output*
- Mon, Nov 2nd, 16:30-18:30 - *Data Manipulation*
- Mon, Nov 9th, 16:30-18:30 - *Scientific Computing*
- Mon, Nov 16th, 16:30-18:30 - *Data Visualisation*
- Mon, Nov 23rd, 16:30-18:30 - *Dashboard*

**NOTE**: this is a tentative outline and may be subject to changes.

#### Important Dates

- Registration for the project evaluation will be open for a couple of weeks in **November**: dates and registration form will be communicated during the course.
- The deadline for submitting the final version of the project, i.e. the last GitHub commit, is **November 30th, 2026, 23:59 CET** (Milan local time).
- Project evaluations will take place on:
  - **December 1st, 2026**, 14:30-16:30
  - **December 2nd, 2026**, 14:30-18:30
  - **December 3rd, 2026**, 16:30-18:30

## The Exam

The evaluation consists of a short presentation (**~5 minutes**) of your project, followed by some questions about both the code of your project and the topics covered during the course.

The project must be presented either as a **Jupyter notebook** or as a **Streamlit web application**. Beyond the results of your analysis, the project should demonstrate your ability to organise a software project (versioning, file organisation, instructions and documentation) and to use Python and its libraries (e.g. `pandas`, `numpy`, `matplotlib`).

For an example of a complete project, have a look at [**vgrank**](https://github.com/sergiopicascia/vgrank).

## Evaluation Criteria

The project will be evaluated based on the following criteria, each designed to assess a key skill covered in the course.

#### 1. GitHub Usage
This criterion assesses your ability to use Git and GitHub for version control, a fundamental practice in modern software and data science development. The project should be hosted in a GitHub repository with: a clear and informative `README.md` file, explaining what the project is about, listing important information (e.g., data sources, libraries), and providing clear instructions on how to setup and run the code; a well-configured `.gitignore` file to exclude unnecessary files; a meaningful commit history, with frequent and atomic commits, having descriptive messages.

#### 2. Project Organization
This criterion evaluates the structure and quality of your codebase. A well-organized project should: be logically organized into modules/scripts; have functions and classes with clear docstrings explaining their purpose, parameters, and return values; respect Python style guide, using consistent naming conventions, proper indentation, and clear variable names; provide a `requirements.txt` file, listing all the necessary Python libraries and their versions.

#### 3. Input/Output
This criterion assesses your ability to work with data from external sources. The project should successfully load data from at least one external source (e.g., CSV, JSON, Excel, a database, or an API). The chosen data source should be appropriate for the analytical goals of the project. The code should handle data loading cleanly, addressing possible errors. The project may also demonstrate output capabilities by saving or displaying results, such as a summary table or generated plots.

#### 4. Data Manipulation
This criterion focuses on the core task of handling data, evaluating your ability to use a data manipulation library like `pandas` to clean, transform, reshape, and prepare your data for the analysis and visualization stages.

#### 5. Scientific Computing
This criterion assesses your ability to perform numerical and statistical computations on your data, using libraries like `numpy` or `scipy` to extract quantitative insights, perform calculations, and prepare data for more complex modeling.

#### 6. Visualisation
This criterion evaluates your ability to communicate your findings visually. This involves creating clear and informative plots that reveal patterns, trends, and insights using a visualisation library, such as `matplotlib` or `seaborn`.

#### 7. Presentation
This criterion evaluates how your work is presented, either through a Jupyter notebook or a web application. The notebook or the application should use the code of your project (not duplicate it), and guide the audience through your analysis in a clear way. Wrapping your analysis in an interactive web application, using a framework like `streamlit`, is not mandatory but it is appreciated.

## Project Proposals

You can choose among one of the following proposals, or propose your own: in the second case, please contact Prof. Montanelli and Dr. Picascia describing your proposal for approval.

**NOTE**: the project proposals are only ideas. The actual implementation is up to you. Thus, you are free to change part of the proposals, as long as the above criteria are still met in the final implementation.

#### 1. Powering the Planet
How do countries produce and consume their energy, and what does it cost the planet? This project invites you to explore decades of global data on energy and emissions to study the energy transition. You could analyze how the energy mix (coal, oil, gas, nuclear, renewables) of different countries has evolved, how CO₂ emissions relate to GDP and population, or which countries managed to grow their economy while reducing their emissions. Create compelling visualizations, such as stacked area charts of the energy mix or scatter plots of emissions versus wealth, to tell a story about the energy transition. Data for this project can be sourced from the [**Our World in Data**](https://ourworldindata.org/) repositories on [energy](https://github.com/owid/energy-data) and [CO₂ emissions](https://github.com/owid/co2-data), optionally enriched with indicators from the [**World Bank Open Data**](https://data.worldbank.org/) portal, e.g. via its [Python API](https://pypi.org/project/wbgapi/).

#### 2. Weather Report
Is the climate really changing where we live? This project involves fetching and processing decades of historical weather data for a set of cities of your choice, to uncover long-term trends and anomalies. You could investigate how average temperatures have changed over time, whether heatwaves or heavy rainfall have become more frequent, or how the seasons differ between cities in different regions. Create insightful charts, like temperature anomaly plots, heatmaps of monthly averages, or comparisons between cities, to visualize how the climate has evolved. Historical weather data from 1940 onwards can be retrieved for free, without registration, from the [**Open-Meteo Historical Weather API**](https://open-meteo.com/en/docs/historical-weather-api).

#### 3. Around the Globe Quiz
The purpose of the project is to automatically create multiple-choice quizzes about geography, such as capitals, flags, languages, currencies, population, or neighbouring countries. Each quiz must consist of one question and four possible answers, only one of which must be correct. The program that generates the quizzes must also implement a criterion to generate the possible answers proportional to the difficulty of the desired quiz, e.g. wrong answers that are more similar to the correct one make the quiz harder. The program must also allow a human player to answer the quiz and must measure their performance with an overall score that takes into account the difficulty of each individual question.
Country data can be obtained from the [**World countries**](https://github.com/mledoze/countries) dataset, and enriched with statistics (e.g., population, GDP, life expectancy) from the [**World Bank API**](https://datahelpdesk.worldbank.org/knowledgebase/articles/889392-about-the-indicators-api-documentation).

#### 4. Page Turners
What makes a book loved by readers? This project explores a large collection of books and user ratings to find out. You will analyze factors like author, publication year, number of ratings, and tags (genres) to see how they connect to the popularity and the appreciation of a book. Investigate whether some genres are consistently rated higher, whether older books are rated differently than recent ones, or how the ratings of an author evolve across their works. Visualize your findings with engaging plots that compare genres, show rating distributions, or track authors over time. A comprehensive dataset for this analysis, with 10,000 books and 6 million ratings, is [**goodbooks-10k**](https://github.com/zygmuntz/goodbooks-10k).

#### 5. Box, Box
Step into the role of a Formula 1 data engineer by analyzing lap-by-lap timing data from real races. This project moves beyond final race results to uncover what happens during a race. You could compare the pace of drivers and teams, study tyre strategies and degradation, analyze the impact of pit stops, or see how a season unfolds race after race. Visualize race dynamics with lap time plots, position charts, or telemetry comparisons between drivers. The objective is to use data to understand how races are won (and lost). Timing and telemetry data are available through the [**FastF1**](https://docs.fastf1.dev/) Python library.

#### 6. Douze Points
Is the Eurovision Song Contest only about songs? This project explores the history of the contest through its voting data. You could investigate whether some countries systematically exchange points (e.g. neighbours or countries with cultural ties), how the introduction of televoting changed the results, how the jury and the public disagree, or which countries have been the most successful over the years. Visualize your findings with voting matrices, heatmaps, maps or time-series plots. Data about contestants and votes of every edition since 1956 is available in the [**Eurovision Song Contest dataset**](https://github.com/EurovisionAPI/dataset).

#### 7. Rate My Recipe
The [Food.com](https://www.food.com/) website hosts a huge collection of recipes, which users review and rate with a score between 0 and 5. The [**Food.com Recipes and Interactions**](https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions) dataset contains more than 180,000 recipes and 700,000 reviews. The purpose of the project is to produce a ranking of recipes in order of user liking. It should be noted that in order to obtain an adequate ranking it is necessary to consider not only the ratings, but also the fact that different recipes may be associated with very different numbers of reviews. For a discussion of this point see, for example, [How Not To Sort By Average Rating](https://www.evanmiller.org/how-not-to-sort-by-average-rating.html). The project should propose its own strategy for sorting the recipes, also arguing its appropriateness compared to a naive ranking based on the average rating, and could explore how the ranking changes across categories (e.g., tags, preparation time, number of ingredients).

#### 8. Island Hopper
Consider the [**OurAirports**](https://ourairports.com/data/) dataset, describing airports all over the world. Assume that you have a small plane, which can only land at large and medium airports, and that has a predefined maximum range of kilometers. Each flight takes 1 hour every 500 km, plus 1 hour for refuelling at each stop; in addition, some time is needed for customs whenever you land in a different country than the one you departed from. Starting from Milan Malpensa (or any other airport of your choice), is it possible to reach Sydney? How many stops and how much time are needed at minimum? And which is the farthest airport that you can reach with at most 10 stops?

## Frequently Asked Questions

##### *I don't know which project to pick...*
Do not choose a project because it seems 'easier' or because your friends are picking that one. It will become a burden. Go for something that you like, a hobby, an interest. Have fun in building it: it is an opportunity for you to get experience in writing code.

##### *Can I work on the project together with a colleague?*
No, the project is individual: each student must develop and present their own project.

##### *When should I start working on my project?*
*The best time to start was yesterday, and the second-best time is now.*
Programming requires practice and building a good project requires time and consistency. Start soon and do a little bit every week and you won't find any difficulties in delivering a good project.

##### *The project proposal says that I need to do X, Y and Z; what if I want to change Y with A?*
You are free to change part of the proposals, as long as the nature of the project is maintained, i.e. that the project criteria are still met in the final implementation. If you are unsure about that, please get in touch with the instructors.

##### *Am I restricted to using Pandas, NumPy, and Matplotlib, or can I use other libraries like Plotly, SciPy, or Scikit-learn?*
You can use other libraries, as long as they perform similar operations compared to the ones suggested.

##### *My dataset is very large and my machine is not able to process it. How should I handle it?*
It is not mandatory to use an entire dataset. You can use a subset of it, either sampled randomly or with a certain criterion, as long as the project is still coherent without the left-out data.

##### *My dataset is too large to be uploaded on GitHub. What should I do?*
Do not commit large data files to your repository: exclude them using the `.gitignore` file. Instead, explain in your `README.md` where and how to download the data, or, even better, write a script that downloads it automatically.

##### *What is the expected scope or complexity of the project? Am I supposed to build a machine learning model?*
The project will be evaluated on the quality of the code. Therefore, it is not necessary to build complex statistical or machine learning models: this is out of the scope of this course.

##### *How much analysis is considered "enough"? Is one interesting chart sufficient?*
The bare amount of operations/plots/computations is not relevant by itself; the evaluation will consider the quality of their implementation and their appropriateness for the project.

##### *Should I present my project with a Jupyter notebook or a Streamlit app?*
Both are accepted. A Streamlit app is not mandatory, but it is appreciated, since it makes your analysis interactive. In both cases, the notebook or the app should import and use the code of your project, rather than containing all the logic itself.

##### *How do I officially submit my project?*
You should invite the instructor as collaborator to your GitHub repository: in this way, if you want, you can keep your repository private. Later during the course, the instructor will ask you to fill in an online form in order to register for the project evaluation.

##### *When is the evaluation taking place? How will it happen?*
The evaluation will take place on December 1st, 2nd and 3rd, 2026. You will be assigned to one of these dates after the registrations are closed, depending on the number of registrations received. The evaluation will most likely take place in presence; if it will be held online, you will receive further instructions in due time.

##### *What is the format of the project presentation? What should I focus on during the evaluation?*
During the evaluation you will be asked to briefly (5-10 min) present your project, either through a Jupyter notebook or a web application. Then, you will be asked some questions about both your code and the topics covered during the course.

##### *What if I cannot attend the evaluation on the specified date?*
The instructor will communicate the calendar after the registrations are closed. If you cannot attend the evaluation on the time slot assigned to you, please contact one of your colleagues in another time slot to arrange a swap. Then communicate the swap to the instructor.

##### *What if I miss the submission deadline or the evaluation?*
The project can only be submitted and discussed in this session. If you miss it, you will have to take the written exam of the Python module in the ordinary exam sessions, or submit a project in the next academic year.

##### *During the lectures we answer Wooclap questions. Are they graded?*
No. The Wooclap sessions are only meant to check whether the topics of the previous lecture are clear, both for you and for the instructor. They do not affect your evaluation in any way.

##### *What about the usage of Large Language Models (LLMs) for code generation? Which is the policy on this regard?*
You are permitted to use AI-powered tools like LLMs (e.g. ChatGPT, Claude, Gemini) to assist you with your project. We will discuss how to use them effectively during the course; however, we have some recommendations.

We encourage you to use these tools as assistants: for instance, to suggest alternative ways to approach a problem, to identify some possible issues within your code or to generate template code; do not use them to generate entire scripts or the core logic of your analysis.

You must test, debug, and validate any code you use, either generated by a LLM or taken from online sources, e.g. StackOverflow. These code snippets can sometimes be buggy, inefficient, use outdated practices, or are logically flawed.

Most importantly, **you must fully understand what every part of your code does**. During the project presentation, you will be expected to explain your code, justify your design choices, and answer questions about its implementation. Answering these questions correctly is mandatory for passing the exam.
