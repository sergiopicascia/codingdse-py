# Version Control with Git

A **Version Control System (VCS)** is a tool that helps you:

- **Keep track of every change** made to your project over time.
- **Create a coherent history** of your project, like a time machine.
- **Collaborate** with others on the same codebase without overwriting each other's work.
- **Experiment** with new features in isolation without breaking the main project.
- **Revert** to a previous working state if you introduce a bug.

## Git

**Git** is the most popular, modern **Distributed Version Control System**.

- **Version Control System:** It tracks the history of your files.
- **Distributed:** Unlike older systems, every developer has a full copy of the project's history on their local machine. This makes it fast and allows for offline work.

Git thinks of your project history as a series of **snapshots**. Each time you save your work (called a **commit**), Git takes a picture of what all your files look like at that moment and stores a reference to that snapshot. These snapshots form a graph, allowing you to navigate the history of your project.

## Setup

Before you use Git, you need to install it and configure it.

**Installation:**
Visit the official [Git website](https://git-scm.com/downloads) and download the installer for your operating system.

**Configuration:**
After installing, open your shell (Terminal, Git Bash, etc.) and introduce yourself to Git. This information will be attached to every commit you make.

```bash
# Set your name for all repositories on this machine
git config --global user.name "Your Name"

# Set your email for all repositories on this machine
git config --global user.email "youremail@example.com"
```

This is a one-time setup.

## A Local Repository

Let's create our first project and track it with Git.

#### Initialize a Repository
Navigate to your project folder (or create a new one) and run `git init`.

```bash
# Create a new directory for our project
mkdir my-python-project
cd my-python-project

# Tell Git to start tracking this directory
git init
```
This command creates a hidden sub-directory named `.git`. This is where Git stores all the history and metadata for your project. **Never manually delete or edit files in the `.git` directory!**

#### The Three States of a File
In Git, your files can be in one of three main states:

1.  **Working Directory:** All the files you can see and edit in your project folder.
2.  **Staging Area (or Index):** A "drafting" area where you prepare the changes for your next snapshot (commit).
3.  **Repository (.git directory):** Where Git permanently stores your committed snapshots.

![Git States Diagram](https://git-scm.com/book/en/v2/images/areas.png)

The command `git status` is your best friend. It tells you the current state of your files.

```bash
# Let's create a file
echo "print('Hello, World!')" > main.py

# Now, check the status
git status
```
Git will report that `main.py` is an "untracked file". Git sees the file but isn't tracking its changes yet.

#### Staging Changes with `git add`
To include changes in your next commit, you first need to add them to the Staging Area. This is done with the `git add` command.

```bash
# Add a specific file to the staging area
git add main.py

# You can also add all modified/new files in the current directory
# git add .
```
Now, run `git status` again. You'll see that `main.py` is now listed under "Changes to be committed".

#### Committing Changes with `git commit`
A **commit** is the act of taking the snapshot of your Staging Area and permanently saving it to your repository's history. Each commit has a unique ID and a descriptive message.

**Good commit messages are crucial.** A good message explains *why* a change was made.

- The first line should be a short summary (50 characters or less).
- Leave a blank line.
- Provide a more detailed explanation if necessary.

```bash
# Commit the staged changes with a message
git commit -m "Initial commit: Add main.py with a simple print statement"
```

#### Viewing History with `git log`
You can view the history of commits using `git log`.

```bash
# Show the full commit history
git log

# A more concise and graphical view of the history
git log --all --graph --decorate --oneline
```
This will show you a list of all your commits, with their unique IDs, author, date, and message.

#### Checking Differences with `git diff`
What if you've made changes but haven't staged them yet? `git diff` shows you the exact lines that have changed.

```bash
# Let's modify our file
echo "# A comment" >> main.py

# See the difference between your working directory and the last commit
git diff
```
This will show you the new line that was added.

## Branching and Merging

So far, we've been working on a single timeline, which by default is called `master` or `main`.

A **branch** is an independent line of development. You create branches to work on new features or fix bugs without disturbing the stable code on the `main` branch.

#### Creating and Switching Branches
Let's say we want to add a new feature. We'll create a new branch for it.

```bash
# Create a new branch called 'add-greeting-feature'
git branch add-greeting-feature

# Switch to the new branch to start working on it
git checkout add-greeting-feature

# --- OR, do both in one command ---
# git checkout -b add-greeting-feature
```

Now you are on the `add-greeting-feature` branch. Any commits you make here will *not* affect the `main` branch until you decide to merge them.

Let's make a change.

```bash
# Edit main.py to add a new function
# (You can use a text editor for this)
# Let's say main.py now looks like this:
#
# def greet(name):
#     print(f"Hello, {name}!")
#
# print("Hello, World!")
# greet("Alice")

# Now, stage and commit this change on our feature branch
git add main.py
git commit -m "FEAT: Add greet function"
```

#### Merging Branches
Our feature is complete! Now we want to incorporate it into our main codebase. We do this by **merging**.

1.  First, switch back to the branch you want to merge *into* (our `main` branch).
2.  Then, run the `git merge` command with the name of the branch you want to merge *from*.

```bash
# Switch back to the main branch
git checkout main

# Merge the changes from our feature branch into main
git merge add-greeting-feature
```
If you now look at `main.py` on the `main` branch, you'll see it contains the new `greet` function.

#### Merge Conflicts
Sometimes, Git can't automatically merge changes. This happens if you and someone else (or you on two different branches) changed the **same line of code in the same file**. This is a **merge conflict**.

When this happens, Git will stop the merge and mark the file with conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).

Your job is to:

1.  Open the conflicted file.
2.  Edit it to resolve the differences. Remove the conflict markers. Decide which version to keep, or combine them.
3.  Save the file.
4.  `git add` the resolved file.
5.  `git commit` to finalize the merge.

## Working with Remotes (GitHub)

So far, everything has been on your local machine. To collaborate, you need a central place to share your repository. This is where services like **GitHub**, GitLab, or Bitbucket come in. They host your Git repositories in the cloud.

A **remote** is a reference to a repository hosted on another server.

#### Starting a Project on GitHub
This is the most common workflow for joining an existing project or starting a new one.

1.  **On GitHub:** Create a new repository. GitHub will give you a URL, like `https://github.com/your-username/my-python-project.git`.
2.  **On your local machine:** Use `git clone` to download a full copy of the repository.

```bash
# Clone the repository from GitHub to your machine
git clone https://github.com/your-username/my-python-project.git

# This automatically creates a folder named 'my-python-project'
cd my-python-project
```

This automatically sets up a remote named `origin` that points to the GitHub URL.

#### Pushing an Existing Local Repo to GitHub

1.  **On GitHub:** Create a *new, empty* repository (do not initialize it with a README or license). Copy its URL.
2.  **On your local machine:**
    - Navigate to your existing project folder (the one where you ran `git init`).
    - Link your local repo to the remote one on GitHub.
    - Push your `main` branch to the remote.

```bash
# Add a remote named 'origin' pointing to your GitHub repo
git remote add origin https://github.com/your-username/my-python-project.git

# Rename the default branch to 'main' (modern standard)
git branch -M main

# Push your main branch to the origin remote
# The -u flag sets the upstream branch for future pushes
git push -u origin main
```

#### Push, Fetch, Pull

-   `git push`: Sends your committed changes from your local repository to the remote repository.
    ```bash
    # After committing your changes locally...
    git push origin main
    ```
-   `git fetch`: Downloads changes from the remote repository but *does not* merge them into your working files. It just updates your local `.git` database.
-   `git pull`: This is a combination of `git fetch` and `git merge`. It downloads changes from the remote and immediately tries to merge them into your current branch. This is the most common way to update your local repository.

```bash
# Before you start working, get the latest changes from the team
git pull origin main
```

## Important Files

-   `.gitignore`: There are certain files you *don't* want to track with Git. These include temporary files, compiled code, and secret keys. Create a file named `.gitignore` in your project's root and list the files/folders to ignore. For Python, this is essential!

    ```text
    # .gitignore

    # Python virtual environment
    venv/
    .venv/

    # Python bytecode and caches
    __pycache__/
    *.pyc

    # IDE-specific folders
    .vscode/
    .idea/
    ```

-   `README.md`: A markdown file that explains what your project is, how to install it, and how to use it. This is the first thing people see on GitHub.

-   `LICENSE`: A file that specifies how others are allowed to use your code (e.g., MIT, Apache 2.0).