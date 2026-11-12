# 7. Git and GitHub

> 🎯 **Goals**: understand what version control is for; install and configure Git; use `init`, `status`, `add`, `commit`, `log` and `diff`; learn what branches are; use GitHub to download (clone) and update (pull) this course and to upload (push) your own code.
>
> ⏱ **Time**: 50 minutes　|　📋 **You need**: lesson 3 (terminal)

## 7.1 Why version control?

You've probably seen a folder like this:

```text
essay.docx
essay_edited.docx
essay_edited2.docx
essay_final.docx
essay_final_REALLY_final.docx
```

With code it gets worse: you change one thing and something else breaks, but you can't remember what you changed; several people work on the same project and overwrite each other…

A **version control system** solves this. It's a "time machine" that records every change — who, when, what and why. You can look at history, compare versions, go back to any point, and collaborate without overwriting each other.

**Git** is by far the most widely used version control system, created in 2005 by Linus Torvalds (the author of Linux). **GitHub** is a website that hosts Git repositories (GitLab and others are similar). This course lives on GitHub.

> [!NOTE]
> In short: **Git is a tool** (software on your computer), **GitHub is a website** (a place to store and share repositories).

## 7.2 Install and configure Git

| System | Command |
|--------|---------|
| Windows | `winget install -e --id Git.Git` (or download from <https://git-scm.com/>; the default options are fine) |
| macOS | `xcode-select --install` already includes git; or `brew install git` |
| Ubuntu/Debian | `sudo apt install git` |

Check:

```console
git --version
```

Before the first use, tell Git who you are (it's recorded in every commit):

```console
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

## 7.3 Key ideas: repository, staging, commit

```text
  working directory        staging area              repository (history)
 (files you're editing) ──git add──► (what will be committed) ──git commit──► (a series of snapshots)
```

- **Repository** (repo): a project folder managed by Git. A hidden `.git` folder inside stores the whole history.
- **Commit**: a **snapshot** of the project at one moment, with a short message.
- **Staging area**: where you choose which changes go into the next commit — you can commit only some of them.

## 7.4 Hands on: your first repository

Follow along in a terminal (one command per line):

```console
mkdir my-first-repo
cd my-first-repo
git init
```

`git init` turns the current folder into a Git repository. Now create a file `hello.py` inside it with:

```python
print("version one")
```

Check the status:

```console
git status
```

Git reports one **untracked** file, `hello.py`. Stage and commit it:

```console
git add hello.py
git commit -m "Add hello.py"
```

The text after `-m` is the commit message. Good messages are **short and specific** — "Fix division by zero", "Add login page" — not "changed stuff".

### Change it again

Edit `hello.py` to:

```python
print("version two")
print("I learned Git!")
```

See exactly what changed:

```console
git diff
```

Red lines starting with `-` were removed, green lines starting with `+` were added. Commit:

```console
git add hello.py
git commit -m "Update greeting"
```

> [!TIP]
> `git add .` stages every change in the current folder at once (`.` means "this folder").

### Look at the history

```console
git log --oneline
```

```text
a1b2c3d (HEAD -> main) Update greeting
9f8e7d6 Add hello.py
```

Each commit has a unique id (a hash such as `a1b2c3d`). To see a file as it was in an old commit:

```console
git show 9f8e7d6:hello.py
```

To throw away uncommitted changes and restore the last committed version:

```console
git restore hello.py
```

## 7.5 .gitignore: files you don't want recorded

Some files don't belong in a repository: compiled programs, temporary files, passwords and keys. Create a text file called `.gitignore` in the repository's top folder with one rule per line:

```text
# build output
*.exe
*.o
build/
# Python caches
__pycache__/
# virtual environments
.venv/
# secrets — never commit!
.env
```

> [!WARNING]
> **Never commit passwords or API keys to GitHub.** Even if you delete them later they stay in the history, and public repositories are scanned automatically. If it happens, revoke the key on the service's website immediately and create a new one.

## 7.6 Branches: parallel universes

A **branch** lets you try something new without disturbing the main line:

```text
main:      A ── B ── C ─────────── F (merge)
                      \           /
feature:               D ── E ───
```

```console
git switch -c new-feature     # create and switch to branch new-feature
# ...edit, add, commit...
git switch main               # back to the main branch
git merge new-feature         # bring the changes in
```

While learning it's perfectly fine to work only on `main`; just know branches exist.

## 7.7 GitHub: download this course

### Option 1: download a ZIP (no Git needed)

Open <https://github.com/YeraldoSmith/Our-tutorial>, click the green **Code** button → **Download ZIP**, and unzip. Downside: to update you must download again.

### Option 2: git clone (recommended)

```console
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
```

`clone` downloads the whole repository including its history. Whenever the course is updated, run this inside the folder:

```console
git pull
```

The learning app reads lessons from this folder, so it updates too.

## 7.8 GitHub: upload your own code

1. Create an account: <https://github.com/signup>.
2. Click **+** (top right) → **New repository**, enter a name (e.g. `my-first-repo`), click **Create repository**.
3. Following the page's instructions, run inside your local repository:

   ```console
   git remote add origin https://github.com/YOUR-NAME/my-first-repo.git
   git push -u origin main
   ```

4. The first push asks you to sign in. GitHub no longer accepts account passwords for pushing; install the **GitHub CLI** (`winget install -e --id GitHub.cli` or `brew install gh`) and run `gh auth login` to sign in through the browser. On Windows, the Git Credential Manager that comes with Git also opens a browser sign-in window.

From then on, after each change: `git add .` → `git commit -m "message"` → `git push`.

### Open source: forks and pull requests

Want to fix a typo in someone else's project, such as this course?

1. Click **Fork** on the project page to copy it to your account;
2. make and commit the change in your copy;
3. click **Contribute → Open pull request** to ask the authors to merge it.

That's the basic open-source workflow.

## 7.9 Cheat sheet

| Command | Does |
|---------|------|
| `git init` | make the current folder a repository |
| `git clone URL` | download a remote repository |
| `git status` | show the current state (changed files) |
| `git add file` / `git add .` | stage changes |
| `git commit -m "message"` | commit |
| `git log --oneline` | show history |
| `git diff` | show unstaged changes |
| `git restore file` | discard working-directory changes |
| `git switch -c name` | create and switch to a branch |
| `git pull` | fetch and merge the latest changes |
| `git push` | upload local commits |

## 7.10 Summary

- Version control is a time machine: it records every change, lets you inspect, compare and go back, and supports teamwork.
- **Git** is the tool, **GitHub** a website hosting repositories.
- Basic loop: edit → `git add` → `git commit -m "..."`; inspect with `git status`, `git diff`, `git log`.
- `.gitignore` excludes files; **never commit passwords or keys**.
- `git clone` downloads, `git pull` updates, `git push` uploads.

## 7.11 Exercises

<!-- exercise: commit-meaning -->
### Exercise 1: What is a commit?

In Git, what is a **commit**?

- Uploading files to GitHub
- A snapshot of the project at one moment, with a message
- Deleting a file
- Creating a folder

<details>
<summary>Answer and explanation</summary>

A commit is a snapshot stored in your local repository. Uploading to GitHub is the job of `git push`.
</details>

<!-- exercise: clone -->
### Exercise 2: Download a repository

Which command downloads a repository from GitHub, including all of its history?

- `git init`
- `git clone`
- `git commit`
- `git log`

<details>
<summary>Answer and explanation</summary>

`git clone URL`. Afterwards, use `git pull` to get later updates.
</details>

<!-- exercise: secret -->
### Exercise 3: What shouldn't be committed?

Which file should you **definitely not** commit to a public GitHub repository?

- `README.md`
- `main.py`
- `.env` containing an API key
- `.gitignore`

<details>
<summary>Answer and explanation</summary>

Files like `.env` that hold passwords or keys belong in `.gitignore`. Once public, anyone can use your key.
</details>

<!-- exercise: workflow -->
### Exercise 4: The daily routine

Write a program that prints, in order, the three commands to upload changes to GitHub (use the message `update`):

```text
git add .
git commit -m "update"
git push
```

<details>
<summary>Hint</summary>

The second line contains double quotes; wrap the whole string in single quotes: `print('git commit -m "update"')`.
</details>

## 7.12 Further reading

- [Pro Git](https://git-scm.com/book/en/v2): the official free book; the first three chapters cover daily use.
- [Learn Git Branching](https://learngitbranching.js.org/): learn branching through animations and puzzles — highly recommended.
- [GitHub Docs: Hello World](https://docs.github.com/en/get-started/start-your-journey/hello-world)
- Next lesson: [How to learn programming well](../08-how-to-learn/en.md)
