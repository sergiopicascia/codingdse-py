# Introduction to the Shell

The shell is a program that takes commands from your keyboard and gives them to the operating system to perform. It's a **C**ommand **L**ine **I**nterface (CLI), as opposed to a **G**raphical **U**ser **I**nterface (GUI) like your desktop, folders, and mouse.

For your Python project, you will need to perform tasks that are more convenient to execute through the shell:

*   **Version Control with Git**: you'll manage your code's history using commands like `git init`, `git commit`, and `git push`.
*   **Python Virtual Environments**: you'll create isolated environments for your project's libraries (e.g., `python -m venv myenv`, `source myenv/bin/activate`).
*   **Package Management**: you'll install libraries using `pip` (e.g., `pip install pandas`).
*   **Running Your Code**: you'll execute your Python scripts directly (e.g., `python main.py`).
*   **Working on Servers**: most web servers don't have a GUI. If you are going to deploy your project on a web server, you'll need to use the shell via a tool like `ssh`.

The shell you use depends on your OS:

*   **Linux & macOS:** you'll use a shell like `bash` or `zsh` inside the "Terminal" application. The commands are virtually identical.
*   **Windows:** the default is PowerShell. While powerful, its syntax is different from the Unix-based shells used in macOS and Linux (which are the standard on web servers). If you want, you can install the **Windows Subsystem for Linux (WSL)**. It gives you a full Linux environment directly within Windows. This will make your experience consistent with macOS/Linux users and the servers your code will eventually run on.

To see which shell you are currently using, you can run the following command, which prints the name of the current process:

```bash
$ echo $0
```

## A First Look at the Shell

When you open your terminal, you'll see something like this:

```bash
my-laptop:~/Projects$
```

Let's break it down:

*   `my-laptop`: the **hostname** of your machine.
*   `:`: a separator.
*   `~/Projects`: the **C**urrent **W**orking **D**irectory (CWD). `~` is a shortcut for your home directory.
*   `$`: the prompt symbol. This indicates the shell is ready for a command.
    *   A `$` or `%` means you are a regular user.
    *   A `#` would mean you are the "superuser" or `root`, who has administrative privileges.

### Running Your First Programs

These are simple, built-in programs.

```bash
# Get the current date and time
$ date

# Display a calendar for the current month
$ cal
```

### Programs with Arguments

Most programs accept **arguments**, which are extra pieces of information you pass to them. Arguments are separated by spaces.

```bash
# The string "hello world" is a single argument because of the quotes
$ echo "hello world"
hello world

# Here, echo receives two separate arguments, "hello" and "world"
$ echo hello world
hello world

# The backslash \ "escapes" the space, telling the shell to treat it as a literal character
# and not a separator. The result is a single argument.
$ echo Hello\ World
Hello World
```

> **NOTE**: spaces are special characters. They separate commands from arguments. If you want a space to be *part of* an argument, you must quote it or escape it.

### Finding Programs

How does the shell know what `echo` or `date` is? It looks for them in a list of specific directories. This list is stored in an **environment variable** called `PATH`.

```bash
# Print the value of the PATH variable. Notice the $ to access a variable's value.
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

# Find out the exact location of a program
$ which echo
/bin/echo
```

When you type `date`, the shell searches through each directory in `$PATH` (separated by colons) until it finds an executable file named `date`.

## Navigating the Filesystem

The filesystem organizes all the data on your computer in a tree-like structure of files and directories.

### Paths

A **path** is the address of a file or directory.

*   In Unix (macOS/Linux), path components are separated by a forward slash `/`.
*   In Windows, they use a backslash `\`.
*   The very top of the filesystem tree is called the **root**, denoted by `/`.

**Absolute vs. Relative Paths**

*   **Absolute Path:** A full address, starting from the root `/`. It's unambiguous.
    *   Example: `/home/student/projects/my_project/main.py`
*   **Relative Path:** An address relative to your current working directory.
    *   Example: If you are in `/home/student/projects/`, the relative path to `main.py` is `my_project/main.py`.

### Essential Navigation Commands

*   `pwd` (Print Working Directory): Shows you where you are right now.

    ```bash
    $ pwd
    /home/student/projects
    ```

*   `cd` (Change Directory): Moves you to a different directory.

    ```bash
    # Move to an absolute path
    $ cd /etc

    # Move to a directory inside your current one
    $ cd /home/student/projects
    $ cd my_project
    ```

### Special Directory Shortcuts

*   `.` (a single dot): Refers to the **current** directory.
*   `..` (two dots): Refers to the **parent** directory (the one above your current one).
*   `~` (tilde): Refers to your **home** directory.
*   `-` (hyphen, with `cd`): Refers to the **previous** directory you were in.

```bash
# Go to your home directory
$ cd ~

# Go "up" one level to the parent directory
$ cd ..

# Go back to where you were before
$ cd -
```

### Getting Information with `ls`

`ls` (List): Lists the contents of a directory.

```bash
# List contents of the current directory
$ ls

# List contents of a different directory
$ ls /home/student
```

You can pass **flags** (or options) to commands to modify their behavior. Flags usually start with a hyphen `-`.

*   `ls -l`: provides a **l**ong listing with more details.
*   `ls -a`: shows **a**ll files, including hidden ones (which start with a `.`).
*   `ls -R`: lists files **R**ecursively, showing the contents of all subdirectories.

You can combine flags: `ls -la`

```bash
$ ls -l
drwxr-xr-x 2 student staff   64 Oct 26 10:30 my_project
-rw-r--r-- 1 student staff 3412 Oct 26 10:31 notes.txt
```

*   `d` at the beginning means it's a **d**irectory. `-` means it's a file.
*   `rwx`: the permissions for **r**ead, **w**rite, and e**x**ecute.

### Getting Help

*   `--help`: most commands accept a `--help` flag for brief usage info.
*   `man` (**man**ual): gives you the full manual page for a command. (Press `q` to quit).

```bash
$ ls --help
$ man ls
```

## Creating and Manipulating Files & Directories

*   `mkdir` (**m**a**k**e **dir**ectory): creates a new directory.

    ```bash
    $ mkdir my_new_project
    ```

*   `touch`: creates a new, empty file.

    ```bash
    $ touch my_new_project/main.py
    ```

*   `cp` (**c**o**p**y): copies a file or directory. `cp <source> <destination>`

    ```bash
    # Copy a file
    $ cp my_new_project/main.py my_new_project/main_backup.py
    ```

*   `mv` (**m**o**v**e): moves or renames a file or directory. `mv <source> <destination>`

    ```bash
    # Rename a file
    $ mv notes.txt important_notes.md

    # Move a file into a directory
    $ mv important_notes.md my_new_project/
    ```

*   `rm` (**r**e**m**ove): permanently deletes a file.
*   `rmdir` (**r**e**m**ove **dir**ectory): deletes an **empty** directory.

    ```bash
    $ rm my_new_project/main_backup.py
    $ rmdir old_project
    ```

> **NOTE:** `rm` does not have a recycle bin. Once a file is gone, it's gone. The command `rm -r` deletes a directory and everything inside it. `rm -rf` will do this forcefully without prompting. Triple-check your command before running `rm -r`.

### Reading Files

*   `cat` (con**cat**enate): dumps the entire content of a file to the screen. Good for small files.
*   `less`: lets you view a file one page at a time. Use arrow keys to navigate and `q` to quit. Best for large files.

```bash
$ cat important_notes.md
$ less /var/log/syslog
```

### I/O Redirection: `>` and `<`

By default, a command's output goes to your screen (standard output) and it takes input from your keyboard (standard input). Redirection lets you change that.

*   `>` (output redirection): rewires the output of a command to a file. Overwrites the file if it exists.
*   `>>` (append output): appends the output to the end of a file.
*   `<` (input redirection): Rewires the input of a command to come from a file.

```bash
# Write "Hello" to a new file, overwriting it if it exists
$ echo "Hello from the shell" > hello.txt
$ cat hello.txt
Hello from the shell

# Add another line to the end of the file
$ echo "This is a new line" >> hello.txt
$ cat hello.txt
Hello from the shell
This is a new line

# Use the contents of hello.txt as input for cat, and send its output to hello2.txt
# This is a clumsy way of copying a file!
$ cat < hello.txt > hello2.txt
```

### Pipes `|`

A pipe `|` lets you take the output of one command and use it as the input for another command, chaining them together.

*   `tail -n 5`: Shows the last 5 lines of a file.
*   `head -n 5`: Shows the first 5 lines of a file.

```bash
# List all files in the root directory in long format, then pipe that output
# to the 'tail' command to only see the last line.
$ ls -l / | tail -n 1
```

When you don't know whether a command has been successfully executed or not, you can check the exit code of the last command.

```bash
$ ls non_existent_directory
ls: cannot access 'non_existent_directory': No such file or directory

# Check the exit code of the last command
$ echo $?
2
```

An exit code of `0` means success. Any other number means an error occurred.

### Wildcards (Globbing)

*   `*`: Matches **any number** of characters.
*   `?`: Matches **any single** character.

```bash
# List all files ending with .py
$ ls *.py

# List files named 'data' followed by a single character and then .csv (e.g., data1.csv, dataX.csv)
$ ls data?.csv
```

### Searching: `find` and `grep`

*   `find`: searches for files and directories in a directory hierarchy.

    ```bash
    # Find all directories named 'src' starting from the current directory
    $ find . -name "src" -type d

    # Find all Python files modified in the last 24 hours
    $ find . -name "*.py" -mtime -1
    ```

*   `grep`: searches for a pattern of text *inside* files.

    ```bash
    # Search for the word "import" in all python files in the current directory
    $ grep "import" *.py

    # List all processes running and pipe it to grep to find our python script
    $ ps aux | grep "python"
    ```

## Running Scripts & Customizing Your Shell

### Variables

You can store information in variables.

```bash
# Assignment: no spaces around the '='
$ PROJECT_NAME="MyAwesomeProject"

# Accessing the value: use a '$'
$ echo "Starting project: $PROJECT_NAME"
Starting project: MyAwesomeProject
```

### Running a Python Script

Let's take this Python script, save it as `reverser.py`:

```python
#!/usr/bin/env python3
# The line above is called a "shebang". It tells the shell what program to use to run this script.

import sys

# sys.argv is a list of command-line arguments.
# sys.argv[0] is the script name itself.
for arg in reversed(sys.argv[1:]):
    print(arg)
```

**How to run it:**

1.  **Make it executable:** right now, the OS sees this as a plain text file. We need to give it "execute" permission.

    ```bash
    # chmod (change mode) +x (add executable permission)
    $ chmod +x reverser.py
    ```

2.  **Run it:** since the current directory (`.`) is usually not in your `$PATH` for security reasons, you have to tell the shell exactly where it is.

    ```bash
    # Run the script in the current directory and pass it some arguments
    $ ./reverser.py hello world 2023
    2023
    world
    hello
    ```

## Quality of Life: Shortcuts and Tools

### Essential Shortcuts

*   `Ctrl + C`: **SIGINT**. kills the currently running foreground program. If your script is stuck in an infinite loop, this is your escape hatch.
*   `Ctrl + L`: clears the terminal screen.
*   `Up Arrow` / `Down Arrow`: cycle through your command history.
*   `Ctrl + R`: **R**everse search. Start typing part of an old command and it will find it for you. Hit Enter to execute or an arrow key to edit.
*   `Tab`: **autocomplete**. The single most useful shortcut. Type the first few letters of a file or command and hit Tab. The shell will try to complete it for you.
*   `history`: shows a list of your previously executed commands.

### Text Editors

You will need to edit files from the command line.

*   `nano`: A very simple, beginner-friendly editor. The commands are listed at the bottom of the screen (`^` means `Ctrl`). `Ctrl+X` to exit.
*   `vim` / `emacs`: Extremely powerful but have a steep learning curve. Feel free to explore them later.

For now, `nano` is your best bet.

```bash
$ nano my_file.txt
```

### Working on Remote Machines

When you need to connect to a server:

*   `ssh` (**s**ecure **sh**ell): log into a remote machine.

    ```bash
    $ ssh username@server-address.com
    ```

*   `scp` (**s**ecure **c**o**p**y): copy files between your computer and a remote machine.

    ```bash
    # Copy a local file to the remote machine
    $ scp my_script.py username@server-address.com:/home/username/
    ```