# Python Environments

Imagine you have two projects:
- Project A needs version 1.0 of a library called `requests`.
- Project B needs version 2.0 of the same `requests` library.

If you install libraries globally, you can only have one version of `requests` installed at a time. When you work on Project B, you'd have to upgrade `requests`, which would break Project A!

This is where **virtual environments** come in.

A **Python virtual environment** is an isolated, self-contained directory that has its own Python interpreter and its own set of installed libraries.

Why use them?

- **Dependency Isolation:** Each project gets its own private set of libraries, preventing version conflicts.
- **Reproducibility:** You can create a list of a project's exact dependencies, making it easy for others (or your future self) to set up the project.
- **Cleanliness:** Keeps your global Python installation clean and minimal.

> **Rule of thumb**: Every Python project should have its own virtual environment.

## Creating an Environment with `venv`

Python comes with a built-in module for creating virtual environments called `venv`.

#### Create the Virtual Environment
Navigate to your project's root directory. The standard practice is to create a folder named `venv` (or `.venv`) for the environment.

```bash
# In your project's root directory
# python3 -m venv <name_of_environment_folder>
python3 -m venv venv
```
This command creates a `venv` folder containing a copy of the Python interpreter and its standard libraries.

> **Crucially, add `venv/` to your `.gitignore` file immediately!** You should never commit your virtual environment to Git.

#### Activate the Environment
Creating the environment isn't enough; you need to "activate" it to start using it. The command differs slightly by operating system.

-   **On macOS/Linux (bash/zsh):**
    ```bash
    source venv/bin/activate
    ```
-   **On Windows (Command Prompt/PowerShell):**
    ```bash
    .\venv\Scripts\activate
    ```

When the environment is active, your shell prompt will change, usually by adding `(venv)` at the beginning. This is your visual cue that you're working inside the environment.

#### Deactivate the Environment
When you're done working, you can deactivate it.

```bash
deactivate
```

## Managing Packages with `pip`

`pip` is Python's package installer. When a virtual environment is active, `pip` will install packages *inside that environment only*.

-   `pip install <package_name>`: Installs a package.

    ```bash
    # With the venv active...
    pip install requests
    pip install numpy
    ```
    
-   `pip list`: Shows all the packages installed in the current environment.

    ```bash
    pip list
    ```

You'll see `requests`, `numpy`, and their own dependencies listed here, but they won't be present if you `deactivate` the environment and run `pip list` again.

## Reproducibility with `requirements.txt`

How do you share your project's dependencies with a collaborator? You create a `requirements.txt` file.

-   **To generate the file:** `pip freeze` lists all installed packages and their exact versions. We can redirect this output to a file.

    ```bash
    pip freeze > requirements.txt
    ```
    
    Your `requirements.txt` file will look something like this:
    
    ```text
    certifi==2022.9.24
    charset-normalizer==2.1.1
    idna==3.4
    numpy==1.23.4
    requests==2.28.1
    urllib3==1.26.12
    ```
    
> **You should always commit your `requirements.txt` file to Git.**

-   **To install from the file:** When someone else clones your project, they can recreate the exact same environment with one command.

    ```bash
    # First, create and activate their own virtual environment
    python3 -m venv venv
    source venv/bin/activate

    # Then, install all dependencies from the file
    pip install -r requirements.txt
    ```

## IDE Integration

Modern code editors like VS Code and PyCharm have excellent integration for Git and Python environments.

-   **Git:** They provide a graphical user interface (GUI) to stage changes, commit, push, pull, and manage branches. This can be a great way to visualize what's happening.
-   **Python Environments:** When you open a project, your IDE will usually detect the `venv` folder. You can then select the Python interpreter from within that `venv` (`venv/bin/python`). This ensures that when you run or debug your code, it uses the packages from your virtual environment. In VS Code, this is done via the "Select Interpreter" command. In PyCharm, it's configured in the project settings.