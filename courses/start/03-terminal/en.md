# 3. The terminal (command line)

> 🎯 **Goals**: open a terminal; understand the prompt; use `pwd`, `ls`, `cd`, `mkdir` and friends; run a program from the terminal; understand PATH and "command not found".
>
> ⏱ **Time**: 40 minutes　|　📋 **You need**: lesson 2 (files, folders, paths)

## 3.1 What is a terminal?

Normally we operate a computer by clicking icons — a **graphical user interface** (GUI). Programmers often use another way too: **typing commands** into a window; the computer runs them and prints the results as text. The window is a **terminal**, the program inside it that understands your commands is the **shell**, and this way of working is the **command line**.

Why learn it?

- Many developer tools exist **only** as command-line programs (compilers, Git, package managers…).
- Installation guides and documentation almost always give you commands to type.
- Once you're used to it, many tasks are much faster than clicking, and you can automate them with scripts.

> [!NOTE]
> This course's app runs your code for you, so **you can continue even if the terminal feels strange**. But you'll need it sooner or later, so it's worth getting comfortable now.

## 3.2 Opening a terminal

- **Windows 11**: search for "Terminal" in the Start menu. It opens **PowerShell** by default. On Windows 10, search for "PowerShell".
- **macOS**: press <kbd>⌘</kbd> + <kbd>Space</kbd> (Spotlight), type "Terminal", press Enter. The default shell is **zsh**.
- **Linux**: on most desktops press <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>. The default shell is usually **bash**.

You'll see a line like one of these:

```console
PS C:\Users\alex>                 ← Windows PowerShell
alex@MacBook ~ %                  ← macOS zsh
alex@ubuntu:~$                    ← Linux bash
```

That is the **prompt**: "I'm ready, type a command". It usually shows the **current folder** (`~` means your home folder).

> [!IMPORTANT]
> Commands in tutorials often start with `$` or `>`. That's the prompt — **don't type it**. If you see `$ python3 --version`, type only `python3 --version`.

## 3.3 Your first commands

A terminal is always "standing" in some folder, the **current (working) directory**. These commands look around and move:

| Purpose | macOS / Linux | Windows PowerShell |
|---------|---------------|--------------------|
| show which folder you're in | `pwd` | `pwd` (or `Get-Location`) |
| list the contents | `ls` | `ls` (or `dir`) |
| go into a folder | `cd name` | `cd name` |
| go up one level | `cd ..` | `cd ..` |
| go to your home folder | `cd ~` or `cd` | `cd ~` |
| create a folder | `mkdir name` | `mkdir name` |
| clear the screen | `clear` | `cls` (or `clear`) |
| show a file's contents | `cat file` | `cat file` (or `type`) |
| print some text | `echo hello` | `echo hello` |

Good news: PowerShell has aliases with the same names as macOS/Linux, so most of these work on all three systems.

### Follow along

Type each line, pressing Enter after each one, and watch what happens:

```console
pwd
ls
mkdir code
cd code
pwd
cd ..
pwd
```

You'll see your home folder's path, a list of its contents, a new `code` folder being created, the path ending in `code` after you go in, and then you're back up one level.

> [!TIP]
> If `mkdir code` says the folder already exists, you made it in lesson 2 — just `cd code`.

## 3.4 Two huge time-savers

1. **<kbd>Tab</kbd> completion**: type the first few letters of a folder name and press <kbd>Tab</kbd>; the terminal completes the rest. `cd Doc` + <kbd>Tab</kbd> becomes `cd Documents`. Faster, and no typos.
2. **<kbd>↑</kbd> for history**: the up arrow brings back earlier commands; edit and press Enter instead of retyping.

And one **lifesaver**: if a program hangs or won't stop (for example an endless loop), press <kbd>Ctrl</kbd> + <kbd>C</kbd> to interrupt it. In a terminal <kbd>Ctrl</kbd> + <kbd>C</kbd> is **not copy**! (Copy is usually <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>; on macOS it's <kbd>⌘</kbd> + <kbd>C</kbd>.)

## 3.5 Running programs from the terminal

Type a program's name to run it. Let's check whether Python is installed:

```console
python3 --version
```

On Windows it's usually:

```console
python --version
```

or `py --version`. If you see something like `Python 3.13.x`, it's installed. (If not, the next lesson shows how.)

### Run a Python file

1. In an editor (for example the app's "File → Save as"), save this code as `hello.py` in your `code` folder:

   ```python
   print("I ran this program from the terminal!")
   ```

2. In the terminal, go to that folder and run it:

   ```console
   cd ~/Documents/code
   python3 hello.py
   ```

   On Windows:

   ```console
   cd ~\Documents\code
   python hello.py
   ```

3. The terminal prints: `I ran this program from the terminal!`

The general form of a command is **program argument1 argument2 …**. `python3 hello.py` means "run the program python3 and give it the argument hello.py". Arguments starting with dashes, like `--version`, are **options** that change how a program behaves.

## 3.6 PATH: where does the terminal look for programs?

You type `python3` — how does the terminal know where Python is installed? Through an **environment variable** called **PATH**: a list of folders that the terminal searches, in order, for a program called `python3`.

```text
PATH = C:\Windows\System32 ; C:\Python313 ; C:\Users\alex\.cargo\bin ; ...
             │                    │                     │
      python here? no       found it! run it      (not searched any further)
```

If no folder contains it, you get an error:

```console
python3: command not found                                    ← macOS / Linux
python3 : The term 'python3' is not recognized as the name of a cmdlet, function, script file, or operable program.   ← Windows
```

Both mean "**I can't find that command**". Usual reasons:

1. The program **isn't installed**.
2. It's installed but **not on PATH** (when installing Python on Windows, tick "Add python.exe to PATH").
3. You just installed it but **didn't restart the terminal** — PATH is read only when the terminal starts. Close it and open a new one.
4. A **typo**, or the name differs on this system (`python` on Windows instead of `python3`).

To see what's in your PATH:

```console
echo $PATH          # macOS / Linux
echo $env:PATH      # Windows PowerShell
```

## 3.7 Careful with dangerous commands

Commands in a terminal have **no recycle bin**. Be very careful with these, and don't run them until you understand them:

- `rm` (delete files), `rm -r` (delete a folder and everything inside), and on Windows `del` and `Remove-Item`
- anything starting with `sudo` (runs as administrator)
- long commands copied from the internet that you don't understand

> [!WARNING]
> Never run a command someone sent you that you don't understand, especially ones containing `rm -rf`, `format` or `curl ... | sh`. When installing software, only use commands from the official website.

## 3.8 Summary

- The **terminal** is a window for typing commands, the **shell** interprets them; the prompt means "waiting for input" — don't type the `$` from tutorials.
- Basics: `pwd` where am I, `ls` what's here, `cd` go in, `cd ..` go up, `mkdir` new folder, `clear` clear screen.
- <kbd>Tab</kbd> completes, <kbd>↑</kbd> recalls history, <kbd>Ctrl</kbd>+<kbd>C</kbd> stops a program.
- Run programs as `program arguments`, e.g. `python3 hello.py`.
- "Command not found": not installed, not on PATH, terminal not restarted, or a typo.
- Deleting commands have no recycle bin; don't run commands you don't understand.

## 3.9 Exercises

<!-- exercise: cd-up -->
### Exercise 1: Up one level

Your current directory is `/home/alex/code/game` and you want to get to `/home/alex/code`. Which command?

- `cd game`
- `cd ..`
- `mkdir ..`
- `ls ..`

<details>
<summary>Answer and explanation</summary>

`cd ..` — `..` means the folder one level up. `ls ..` only lists that folder; it doesn't move you there.
</details>

<!-- exercise: stop -->
### Exercise 2: It won't stop

A program in your terminal is stuck in an endless loop. Which keys stop it?

- <kbd>Ctrl</kbd> + <kbd>S</kbd>
- <kbd>Ctrl</kbd> + <kbd>C</kbd>
- <kbd>Tab</kbd>
- <kbd>↑</kbd>

<details>
<summary>Answer and explanation</summary>

<kbd>Ctrl</kbd> + <kbd>C</kbd>. In a terminal it means "interrupt", not copy.
</details>

<!-- exercise: not-found -->
### Exercise 3: Command not found

You just installed Python and type `python --version` in a terminal that **was already open**, but it says the command isn't found. What should you try first?

- Reinstall the operating system
- Close the terminal and open a new one
- Type `PYTHON --VERSION` instead
- Delete the Python folder

<details>
<summary>Answer and explanation</summary>

A terminal reads PATH only when it starts. After an installer changes PATH you need a new terminal. If it still fails, check that "add to PATH" was ticked during installation.
</details>

<!-- exercise: commands -->
### Exercise 4: Write the commands

Let's use Python to "print" some terminal commands. Write a program that prints these three lines — they create a `project` folder, enter it and show the current location:

```text
mkdir project
cd project
pwd
```

<details>
<summary>Hint</summary>

Three `print` calls, one line each.
</details>

## 3.10 Further reading

- [The Missing Semester · The shell](https://missing-semester.csail.mit.edu/2020/course-shell/)
- [Microsoft: PowerShell 101](https://learn.microsoft.com/powershell/scripting/learn/ps101/01-getting-started)
- Next lesson: [Installing developer tools](../04-install-tools/en.md)
