# 15. Packages and modern tooling

> 🎯 **Goals**: know where to find third-party **packages** and how to install them; understand why **virtual environments** are essential; use the modern tool **uv** to create projects and manage dependencies and Python versions; read a `pyproject.toml`; keep code style consistent and find problems with **ruff**; catch bugs early with a type checker; write automated tests with **pytest**; debug with `breakpoint()` and `logging`. These are the tools you use every day on "real projects".
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: starter course lessons 3, 4 and 7 (the terminal, installing tools, Git); lessons 11 and 12 of this course

So far our programs have only used what comes with Python. Real projects almost always use **third-party packages** written by others, and need a set of tools to keep the code in good shape. Most of this lesson happens in the **terminal** (starter course lesson 3); lines starting with `$` in the code blocks are terminal commands, not Python.

## 15.1 Third-party packages and PyPI

**PyPI** (the Python Package Index, <https://pypi.org>) is Python's official "app store", with over six hundred thousand free packages. Some of the most popular:

| Package | Purpose |
|---------|---------|
| `requests`, `httpx` | talking to websites and web APIs |
| `numpy`, `pandas` | numerical computing, analysing tabular data |
| `matplotlib` | charts and plots |
| `fastapi`, `flask`, `django` | websites and web services |
| `pillow` | image processing |
| `rich` | colourful, pretty text and tables in the terminal |
| `pytest` | testing |
| `torch`, `transformers` | deep learning and large models (see this project's LLM tutorial) |

Once installed, you use them like the standard library — just `import`:

```python norun
# install first in a terminal: uv add requests (or pip install requests)
import requests

r = requests.get("https://api.github.com")
print(r.status_code)          # 200 means success
print(r.json()["current_user_url"])
```

## 15.2 Virtual environments: a separate "toolbox" per project

Install every package into your system's Python and trouble soon follows: project A needs version 1.0 of a package and project B needs 2.0, so they conflict; and nobody can tell which packages each project actually depends on.

A **virtual environment** gives each project its **own folder** (usually called `.venv`) where that project's packages are installed, so projects don't interfere with each other.

With Python's built-in `venv` and `pip`:

```bash
$ python -m venv .venv                 # create a virtual environment in the current folder
$ source .venv/bin/activate            # activate it (macOS / Linux)
$ .venv\Scripts\activate               # activate it (Windows)
(.venv) $ pip install requests         # once activated, pip installs into this environment only
(.venv) $ pip freeze > requirements.txt   # write down the list of dependencies
(.venv) $ deactivate                   # leave the virtual environment
```

When it's active, `(.venv)` appears at the start of the prompt.

## 15.3 uv: the modern all-in-one tool

**uv** (made by Astral, written in Rust) combines "installing Python, creating virtual environments, installing packages, locking versions and running programs" in one tool, and it's 10–100 times faster than pip. It has become the first choice for many new projects, and this course recommends it too (starter course lesson 4 shows how to install it).

```bash
$ uv init my-app             # create a new project (with pyproject.toml, a README and more)
$ cd my-app
$ uv add requests rich       # add dependencies: creates .venv, installs them, records them in pyproject.toml and uv.lock
$ uv add --dev pytest ruff   # tools only needed while developing (testing, checking code)
$ uv run main.py             # run the program in the project's virtual environment (no manual activation)
$ uv run pytest              # run the tests
$ uv remove rich             # remove a dependency
$ uv python install 3.13     # install a specific Python version
$ uv sync                    # on another computer: recreate exactly the same environment from uv.lock
```

The benefits: **no manual activation** (`uv run` uses the project's environment automatically), and `uv.lock` records the **exact version** of every package, so anyone (including you a year later) can get an identical environment with `uv sync`.

### `pyproject.toml`: the project's "ID card"

The `pyproject.toml` that `uv init` creates is the standard configuration file for Python projects. It looks roughly like this:

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "My first project"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32",
    "rich>=14.0",
]

[dependency-groups]
dev = ["pytest>=8", "ruff>=0.12"]

[tool.ruff]
line-length = 100
```

`>=2.32` in `dependencies` means "2.32 or newer". Version numbers are usually `major.minor.patch` (**semantic versioning**): a new patch number only fixes bugs; a new minor number adds features while old code keeps working; a new major number may contain incompatible changes.

### A typical project layout

```text
my-app/
├── pyproject.toml        project configuration and dependencies
├── uv.lock               exactly locked versions (managed by uv; don't edit by hand)
├── README.md             what the project is about
├── .gitignore            tells Git not to commit .venv, __pycache__ etc.
├── src/
│   └── my_app/
│       ├── __init__.py   this file makes the folder a "package"
│       └── core.py
└── tests/
    └── test_core.py      test code
```

`.gitignore` should contain at least `.venv/` and `__pycache__/`: the virtual environment can be recreated at any time with `uv sync`, so it needn't and shouldn't be committed to Git (starter course lesson 7).

## 15.4 ruff: code style and static checks

The Python community has a style guide, **PEP 8**: indent with 4 spaces, write variable names in `snake_case` (lowercase with underscores) and class names in `CamelCase`, put spaces around operators… A shared style makes everyone's code equally pleasant to read.

**ruff** (also from Astral) does two things **automatically**:

```bash
$ uv run ruff format .       # format: rewrite the code into a consistent style
$ uv run ruff check .        # check: find likely bugs and poor practices
$ uv run ruff check --fix .  # fix whatever can be fixed automatically
```

For example, for this code:

```python norun
import os, sys
def Add(a,b):
    unused = 1
    if a == None:
        return b
    return a+b
```

`ruff check` points out that several modules are imported on one line, that `os` and `sys` are imported but unused, that the variable `unused` is assigned but never used, and that `== None` should be `is None`; `ruff format` adds the missing spaces and blank lines. (The function name `Add` breaks PEP 8's naming rules; that check only runs if the `N` rules are enabled in the configuration.)

> [!TIP]
> Install the **Ruff** extension in VS Code and your files are formatted on save, with problems highlighted right in the editor.

## 15.5 Type checking: catching bugs before running

As lesson 10 said, Python doesn't enforce type hints. A **type checker** reads your code and hints and finds type errors **without running the program**:

```python norun
def total_price(prices: list[float]) -> float:
    return sum(prices)

print(total_price(["3.5", "2"]))      # passes a list of strings
```

```bash
$ uv run mypy main.py
main.py:4: error: List item 0 has incompatible type "str"; expected "float"
```

Common type checkers are **mypy**, **pyright** (built into VS Code's Pylance extension) and Astral's newer **ty**. The bigger the project, the more useful type checking becomes.

## 15.6 Automated testing: pytest

After writing a function, you try a few examples by hand to see whether it works. **Automated tests** turn those examples into code, so every future change can re-check them all with one command — making sure you didn't break what used to work.

**pytest** is the most popular testing tool, and it's very simple: files whose names start with `test_`, functions whose names start with `test_`, and `assert` inside:

```python norun
# file src/my_app/core.py
def slugify(title: str) -> str:
    """Turn a title into a URL-friendly form: lowercase, spaces become hyphens."""
    return "-".join(title.lower().split())
```

```python norun
# file tests/test_core.py
from my_app.core import slugify

def test_basic():
    assert slugify("Hello World") == "hello-world"

def test_extra_spaces():
    assert slugify("  Learn   Python  ") == "learn-python"

def test_empty():
    assert slugify("") == ""
```

```bash
$ uv run pytest
============================= test session starts ==============================
collected 3 items

tests/test_core.py ...                                                   [100%]

============================== 3 passed in 0.01s ===============================
```

Each dot `.` is a passing test. When a test fails, pytest shows clearly "what was expected and what was actually received".

The learning app and the website run one file at a time, so pytest can't be used directly there. The standard library's **unittest** lets you try the same idea in a single file:

```python
import unittest

def slugify(title: str) -> str:
    return "-".join(title.lower().split())

class TestSlugify(unittest.TestCase):
    def test_basic(self):
        self.assertEqual(slugify("Hello World"), "hello-world")

    def test_extra_spaces(self):
        self.assertEqual(slugify("  Learn   Python  "), "learn-python")

    def test_empty(self):
        self.assertEqual(slugify(""), "")

unittest.main(argv=["tests"], exit=False, verbosity=2)    # results are printed to the error output (stderr)
```

> [!NOTE]
> **Writing the tests first** is a common practice too (test-driven development, TDD): write tests saying "how I want the function to behave", watch them fail, then write code until they pass. The automatic checking of every coding exercise in this course is, in fact, a set of tests.

## 15.7 Debugging: `breakpoint()` and `logging`

There are better ways to debug than sprinkling `print` everywhere:

**1. Breakpoints**: write `breakpoint()` in your code and the program pauses there in the `pdb` debugger, where you can inspect variables and step through line by line:

```python norun
def average(nums):
    total = sum(nums)
    breakpoint()            # pauses here; type p total to see a variable, n for the next line, c to continue, q to quit
    return total / len(nums)
```

Editors such as VS Code and PyCharm have graphical debuggers: click left of a line number to set a breakpoint and press F5 to start debugging — much more intuitive.

**2. Logging**: the `logging` module is better than `print` for recording what a program is doing — messages have levels (debug, info, warning, error) and timestamps, and in production one setting switches off the debug messages:

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
log = logging.getLogger("shop")

def buy(item, money):
    log.debug("about to buy %s", item)        # the level is INFO, so DEBUG messages are hidden
    if money < 10:
        log.warning("not enough money for %s: only %d", item, money)
        return False
    log.info("bought %s", item)
    return True

buy("book", 30)
buy("toy", 5)
```

## 15.8 Summary

- Third-party packages live on PyPI; install with `uv add name` or `pip install name`, then `import`.
- **One virtual environment per project** (`.venv`) avoids version conflicts; don't commit `.venv` to Git.
- **uv**: `uv init` creates a project, `uv add` adds dependencies, `uv run` runs code, `uv sync` restores the environment; dependencies are recorded in `pyproject.toml` and exact versions locked in `uv.lock`.
- **ruff format** enforces a consistent style (PEP 8), **ruff check** finds problems; **mypy / pyright / ty** check types.
- **pytest**: files and functions starting with `test_` + `assert`; `uv run pytest` runs every test at once.
- Debug with `breakpoint()` or your editor's debugger; record what's happening with `logging`.

## 15.9 Exercises

<!-- exercise: version-compare -->
### Exercise 1: Comparing version numbers

Read a line with two version numbers (e.g. `1.10.0 1.9.3`) and print the **newer** one, or `equal` if they're the same. A version number is several integers separated by `.`, compared **numerically** (`1.10` is newer than `1.9`); when the lengths differ, missing parts count as 0 (`1.2` equals `1.2.0`).

```text
Sample input: 1.10.0 1.9.3
Sample output: 1.10.0
```

<details>
<summary>Hint</summary>

Turn each version into a list of integers — `[int(x) for x in v.split(".")]` — pad the shorter one with 0s to the same length, then compare the lists with `>` (Python compares element by element).
</details>

<!-- exercise: snake-case -->
### Exercise 2: PEP 8 style names

PEP 8 says variable and function names should be `snake_case`. Read a camelCase name and turn every capital letter into "an underscore + the lowercase letter". (The first letter of the input is always lowercase.)

```text
Sample input: getUserName
Sample output: get_user_name
```

<details>
<summary>Hint</summary>

Loop over the characters: `if ch.isupper(): result += "_" + ch.lower()`, otherwise add the character as it is.
</details>

<!-- exercise: fix-clamp -->
### Exercise 3: Make the tests pass

The starter code has a function `clamp(x, lo, hi)` (it should keep `x` between `lo` and `hi`: return `lo` if `x` is smaller than `lo`, `hi` if it's larger than `hi`, and `x` otherwise) and a few tests written with `assert`. But the function is wrong and the tests fail. Fix the function so the program prints `all tests passed`. **Don't change the tests.**

```text
Sample output: all tests passed
```

<details>
<summary>Hint</summary>

Run it and see which `assert` fails (the traceback gives the line number), then think about what that input should produce. One correct version is `return max(lo, min(x, hi))`.
</details>

<!-- exercise: why-venv -->
### Exercise 4: Why use a virtual environment?

- to make Python run faster
- so each project has its own separate packages and versions without conflicts, and each project's dependencies are clearly recorded
- packages can't be installed without one
- to make code safer against viruses

<details>
<summary>Answer and explanation</summary>

A virtual environment isolates each project's packages: project A on version 1.0 and project B on 2.0 don't conflict; together with `pyproject.toml` / `uv.lock`, others can recreate exactly the same environment.
</details>

<!-- exercise: tool-roles -->
### Exercise 5: Who does what

Which option matches **every** tool with its main job correctly?

- ruff: runs tests; pytest: formats code; uv: checks types
- uv: manages dependencies and virtual environments; ruff: formats and checks code; pytest: runs tests
- uv: draws charts; ruff: accesses the network; pytest: installs Python
- pytest: manages dependencies; uv: checks code style; ruff: runs programs

<details>
<summary>Answer and explanation</summary>

uv handles projects, dependencies, virtual environments and Python versions; ruff handles code style and static checks; pytest handles automated tests.
</details>

## 15.10 Further reading

- [uv documentation](https://docs.astral.sh/uv/), [Ruff documentation](https://docs.astral.sh/ruff/)
- [pytest documentation](https://docs.pytest.org/)
- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [Python Packaging User Guide](https://packaging.python.org/)
- Next lesson: [Project: a command-line expense tracker](../16-project/en.md)
