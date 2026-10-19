# Python Environments

Imagine you have two projects:
- Project A was written last year with version 2 of `pandas`.
- Project B needs version 3 of `pandas`, which changed some behaviours of the previous one (e.g., text columns now have their own data type).

If you install libraries globally, you can only have one version of `pandas` installed at a time. When you work on Project B, you'd have to upgrade `pandas`, which could break Project A!

This is where **virtual environments** come in.

A **Python virtual environment** is an isolated, self-contained directory that has its own `python` command and its own set of installed libraries.

Why use them?

- **Dependency Isolation:** Each project gets its own private set of libraries, preventing version conflicts.
- **Reproducibility:** You can create a list of a project's exact dependencies, making it easy for others (or your future self) to set up the project.
- **Cleanliness:** Keeps your global Python installation clean and minimal.

> **Rule of thumb**: Every Python project should have its own virtual environment.

## Which Python?

Before creating an environment, check which version of Python you have:

```bash
python3 --version
# Python 3.13.4
```

For this course, use **Python 3.12, 3.13 or 3.14**. Older versions can't install the latest releases of `numpy` and `scipy`, while a brand-new version (like 3.15, due in October 2026) takes a few weeks to be supported by all the libraries we use. If needed, install Python 3.14 from [python.org](https://www.python.org/downloads/).

> **NOTE**: as we saw in the shell lecture, depending on your OS and installation, the interpreter may be called `python3` or `python`. On Windows, use `python` (or `py`) instead of `python3` in the commands below. On macOS, the `python3` that comes with the *Command Line Developer Tools* is usually too old for this course.

## Creating an Environment with `venv`

Python comes with a built-in module for creating virtual environments called `venv`.

#### Create the Virtual Environment
Navigate to your project's root directory. The standard practice is to create the environment in a folder named `.venv`: the leading dot keeps it hidden, like `.git`.

```bash
# In your project's root directory
cd ~/projects/vgrank

# python3 -m venv <name_of_environment_folder>
python3 -m venv .venv
```
This command creates a `.venv` folder containing:

- `pyvenv.cfg`: a small configuration file, recording which Python installation the environment is based on;
- `bin/` (`Scripts\` on Windows): the `python` and `pip` commands of the environment, and the scripts to activate it. On macOS and Linux, `python` is just a link to the interpreter of your Python installation;
- `lib/` (`Lib\` on Windows): the `site-packages` folder, where your libraries will be installed. For now, it contains little more than `pip`.

The standard library is not copied either: the environment reuses the one of your Python installation. This makes a virtual environment **disposable**: you can't move or copy it to another folder or computer, but you can delete it (`rm -rf .venv`) and recreate it at any time.

> **Crucially, make sure that `.venv/` is listed in your `.gitignore` file!** You should never commit your virtual environment to Git: it is large, it only works on your computer, and it can be recreated from your `requirements.txt` (see below). Since Python 3.13, `venv` also puts a `.gitignore` inside `.venv`, but older versions don't.

> **NOTE**: don't confuse `.venv` (the virtual environment) with `.env`, a file used by some projects (like vgrank) to store secrets, such as API keys. Neither of them should end up on GitHub.

> **NOTE**: on Ubuntu (also in WSL), the command may fail, saying that `ensurepip` is not available. Install the missing package, as suggested by the error message (usually `sudo apt install python3-venv`), and run the command again.

#### Activate the Environment
Creating the environment isn't enough; you need to "activate" it to start using it, in every new terminal where you work on the project. The command differs slightly by operating system.

-   **On macOS/Linux (bash/zsh):**
    ```bash
    source .venv/bin/activate
    ```
-   **On Windows (PowerShell/Command Prompt):**
    ```powershell
    .venv\Scripts\activate
    ```
-   **On Windows (Git Bash):**
    ```bash
    source .venv/Scripts/activate
    ```

When the environment is active, your shell prompt will change, usually by adding `(.venv)` at the beginning. This is your visual cue that you're working inside the environment.

> **NOTE**: on Windows, PowerShell may refuse to activate the environment, with an error saying that *running scripts is disabled on this system*. Allow it (only once) with the following command, then activate the environment again:
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

#### How Activation Works
Activating an environment simply puts its `bin` folder (`Scripts` on Windows) at the beginning of your `PATH` (remember the shell lecture?): when you type `python` or `pip`, the shell finds the ones of the environment first. You can check it with `which`:

```bash
# Before the activation: the Python of your system
which python3
# /usr/bin/python3

source .venv/bin/activate

# After the activation: the Python of the environment
which python
# /home/student/projects/vgrank/.venv/bin/python
```

On Windows, use `Get-Command python` (PowerShell) or `where python` (Command Prompt) instead of `which`. Inside an active environment, simply use `python` and `pip`, whatever your operating system.

#### Deactivate the Environment
When you're done working, you can deactivate it.

```bash
deactivate
```

## Managing Packages with `pip`

`pip` is Python's package installer: it downloads libraries from the [Python Package Index (PyPI)](https://pypi.org/). When a virtual environment is active, `pip` will install packages *inside that environment only*.

-   `pip install <package>`: installs the latest version of a package, together with its dependencies. To install a specific version, use `pip install <package>==<version>`.
-   `pip list`: shows all the packages installed in the current environment, with their versions.
-   `pip show <package>`: shows information about an installed package, such as its version and its dependencies.
-   `pip uninstall <package>`: removes a package.

Let's install the first libraries of vgrank:

```bash
# With the environment active...
pip install pandas requests
pip list
```

You'll see `pandas` and `requests`, but also their own dependencies (e.g., `numpy`), which `pip` installed automatically. None of them is visible outside the environment.

> **NOTE**: if `pip install` fails with `error: externally-managed-environment`, you are installing outside a virtual environment, and your system protects its own Python (this happens, e.g., on recent versions of Ubuntu, or with Python installed through Homebrew on macOS). Activate your environment and try again.

## Reproducibility with `requirements.txt`

How do you share your project's dependencies with a collaborator (or with your instructor)? You create a `requirements.txt` file in the root of your project.

#### Write the File
`requirements.txt` lists the libraries that your code imports, one per line, each one pinned with `==` to the version you are using (check it with `pip list`). For vgrank, at this point, it looks like this (your versions may be more recent):

```text
pandas==3.0.6
requests==2.34.2
```

You don't need to list the dependencies of your libraries (e.g., `numpy` for `pandas`): `pip` installs them automatically. List them only if your code imports them directly. As your project grows, so does the file: the final [`requirements.txt` of vgrank](https://github.com/sergiopicascia/vgrank/blob/main/requirements.txt) lists 8 libraries.

> **You should always commit your `requirements.txt` file to Git**, and update it whenever you add (or remove) a library.

```bash
git add requirements.txt
git commit -m "Add pandas and requests to requirements"
git push
```

#### What about `pip freeze`?
`pip freeze` lists all the packages installed in the current environment, with their exact versions, in the same format of `requirements.txt`. That's why you will often see the file generated with `pip freeze > requirements.txt` (the redirection we saw in the shell lecture). However, this lists **everything**: the dependencies of your dependencies, the libraries you only tried, and even packages needed only on your operating system. With just `pandas` and `requests` installed, the output of `pip freeze` is already:

```text
certifi==2026.7.22
charset-normalizer==3.5.2
idna==3.20
numpy==2.5.3
pandas==3.0.6
python-dateutil==2.9.0.post0
requests==2.34.2
six==1.17.0
urllib3==2.8.0
```

The environment of vgrank, for instance, contains more than 120 packages, while its `requirements.txt` needs 8 lines. Use `pip freeze` or `pip list` to check the versions, and write `requirements.txt` yourself.

#### Install from the File
When someone else clones your project, they can recreate the same environment with a few commands.

```bash
# First, create and activate their own virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Then, install all dependencies from the file
pip install -r requirements.txt
```

These are the setup instructions you should write in the `README.md` of your project, together with the Python version it needs: see the *Setup* section of the [vgrank README](https://github.com/sergiopicascia/vgrank).

> **NOTE**: before submitting your project, test your instructions as if you were your instructor: clone your repository into a new folder, create a fresh environment, install the requirements and run your code. If a library is missing from `requirements.txt`, you'll find out now, and not during the exam.

## Jupyter Notebooks and Environments

A **Jupyter notebook** is a file, with extension `.ipynb`, made of **cells**: each cell contains either Python code, or text written in Markdown (the same format as `README.md`). You run the code one cell at a time, and its output (a value, a table, a plot or an error) appears right below the cell; the variables defined by a cell remain available to the cells that you run afterwards. This makes notebooks handy to explore your data step by step, and to present your analysis, mixing code, results and explanations: have a look at the [notebook of vgrank](https://github.com/sergiopicascia/vgrank/blob/main/notebooks/vgrank_walkthrough.ipynb). You can work on a notebook in the browser, with Jupyter Lab, or in VS Code.

A notebook runs its code in a **kernel**, a Python process connected to it. To use the libraries of your environment, the kernel must run the Python of `.venv`: otherwise, you'll get errors like `ModuleNotFoundError: No module named 'pandas'`, even though `pandas` is installed in your environment.

-   **Jupyter Lab** (in the browser): install it in your environment (`pip install jupyterlab`), then start it from the project folder, with the environment active, by running `jupyter lab`. It will use the Python of the environment.
-   **VS Code**: open the notebook, click *Select Kernel* (top right) and choose *Python Environments → .venv*. The environment needs the `ipykernel` package: if it is missing, VS Code offers to install it (or you can run `pip install ipykernel`).

If you present your project with a notebook, remember to add `jupyterlab` (or `ipykernel`) to your `requirements.txt`, as vgrank does.

## IDE Integration

When you open a project, VS Code and PyCharm will usually detect the `.venv` folder. Make sure they use the Python interpreter of the environment (`.venv/bin/python`, or `.venv\Scripts\python.exe` on Windows): this ensures that when you run or debug your code, it uses the packages from your virtual environment.

-   **VS Code:** run *Python: Select Interpreter* from the Command Palette. The *Python: Create Environment* command can also create the `.venv` for you, and install the libraries listed in `requirements.txt`. The integrated terminal activates the selected environment automatically.
-   **PyCharm:** configure it in *Settings → Project → Python Interpreter*.

## Beyond `venv` and `pip`

In this course, we use `venv` and `pip`, which come with Python. You may also come across:

-   **conda** (with Anaconda or Miniconda), popular in data science, which manages both environments and packages with its own commands (e.g., `conda create`, `conda activate`, `conda install`);
-   **pyenv** (on Windows, its port **pyenv-win**), which installs several versions of Python side by side, and selects one for each project: e.g., `pyenv install 3.14` installs Python 3.14, and `pyenv local 3.14`, run in the root of the project, selects it for that folder, saving the version in a `.python-version` file. Then, `python3 -m venv .venv` creates the environment with the selected version;
-   **Poetry**, which manages the environment and the dependencies of a project together: `poetry add pandas` installs `pandas` and records it in the `pyproject.toml` file, while the `poetry.lock` file keeps the exact versions of all the installed packages, so that `poetry install` can recreate the same environment on another computer;
-   **uv**, a recent and much faster replacement for `pip` and `venv` (e.g., `uv venv`, `uv pip install -r requirements.txt`), which can also install Python itself (like pyenv) and manage the dependencies of a project (like Poetry).

Whatever tool you use, your project must provide a `requirements.txt` file, and the instructions to install it.

## Useful Resources

- [Virtual Environments and Packages](https://docs.python.org/3/tutorial/venv.html), from the official Python tutorial
- [Installing Packages with pip and venv](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/)
- [Python Environments in VS Code](https://code.visualstudio.com/docs/python/environments)
- [Jupyter Notebooks in VS Code](https://code.visualstudio.com/docs/datascience/jupyter-notebooks)
- [Virtual Environments in PyCharm](https://www.jetbrains.com/help/pycharm/creating-virtual-environment.html)
