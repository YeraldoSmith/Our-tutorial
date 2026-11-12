# 16. Project: a command-line expense tracker

> 🎯 **Goals**: bring together everything from this course to build a **genuinely useful** little program from scratch — a command-line expense tracker that records spending, lists records, totals them by category and deletes mistakes, keeping the data in a file so nothing is lost when you close and reopen it. Along the way, practise **breaking a problem down**, **designing data** and splitting a program into "data / storage / logic / interface" layers; finally turn it into a uv project with tests and think about how to extend it.
>
> ⏱ **Time**: 120 minutes　|　📋 **You need**: lessons 1–15 of this course

Congratulations on reaching the last lesson of the Python course! Every lesson so far taught "parts"; now we build a complete "machine". Take it step by step — every piece comes from an earlier lesson.

## 16.1 First decide what to build

Before writing code, write the requirements down in your own words. This matters far more than you might think:

```text
I want an expense tracker:
1. record an expense: amount, category (food, transport…), a note; the date and an ID are added automatically
2. list all expenses
3. totals by category: how much each category cost and its share, ideally with a simple bar chart
4. delete an expense entered by mistake
5. keep the data in a file so it's still there next time
6. don't crash on bad input — give a friendly message instead
```

Then think about how the program **talks to the user**. We'll build a "command-line interface": the program shows a prompt `>`, the user types a command, the program carries it out and waits for the next one:

```text
> add 35 food lunch
✓ recorded #1: food 35.00
> report
food         35.00  100.0%  ████████████████████
total 35.00
> quit
Bye!
```

## 16.2 Splitting the program into layers

A good habit: divide the program into layers by responsibility, each doing one job:

| Layer | Responsible for | Knowledge used |
|-------|-----------------|----------------|
| **data** | what one expense looks like | `@dataclass` (lesson 13) |
| **storage** | reading and writing the file | `json`, `pathlib`, exceptions (lessons 11, 12) |
| **logic** | adding, deleting, totalling | lists, dicts, functions, sorting (lessons 8–10) |
| **interface** | reading commands, showing results | `input`, `match`, f-strings (lessons 5, 14) |

The benefit: functions in the logic layer **neither read input nor print** — they take arguments and return results — so they're easy to test; and if you later replace the command-line interface with a web page or a GUI, only the interface layer changes.

## 16.3 The data layer: one expense

```python norun
from dataclasses import dataclass

@dataclass
class Expense:
    id: int
    day: str            # the date, e.g. "2025-03-01"
    category: str       # the category, e.g. "food"
    amount: float       # the amount
    note: str = ""      # a note
```

Each expense has an **ID**, which is how you say which one to delete.

## 16.4 The storage layer: saving to a JSON file

```python norun
import json
from dataclasses import asdict
from pathlib import Path

DATA_FILE = Path("expenses.json")

def load() -> list[Expense]:
    if not DATA_FILE.exists():                 # first run: no file yet
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):  # the file has been damaged
        print("⚠ the data file is damaged; starting with an empty book")
        return []

def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]       # dataclass → dict
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")
```

- `asdict(e)` turns a dataclass object into a dict, which can be saved as JSON;
- `Expense(**row)` does the reverse: it creates an object using the dict's key-value pairs as keyword arguments (`**` unpacking, lesson 14);
- loading handles two mishaps: the file doesn't exist, or its contents are damaged. **Programs should cope with "imperfect" situations.**

## 16.5 The logic layer: add, delete, total

```python norun
from datetime import date

def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("the amount must be greater than 0")
    new_id = max((e.id for e in expenses), default=0) + 1     # the largest existing ID + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item

def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False                                              # not found

def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount     # the counting idiom from lesson 9
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))  # biggest spending first
```

A few details:

- `max(..., default=0)`: `max` fails on an empty sequence, and `default=0` makes it return 0 instead;
- `add` **raises an exception** for an invalid amount (lesson 12) and leaves the interface layer to decide how to tell the user;
- `delete` returns `True` / `False` to tell the caller whether it succeeded.

## 16.6 The interface layer: reading commands, showing results

`while True` keeps reading commands, and `match` tells them apart (lesson 14). Here is the complete program — run it and it carries out the prepared commands below in order (when you run it in the learning app, you can type your own commands):

```python input="add 35 food lunch\nadd 12.5 transport metro\nadd 20 food bubble tea\nadd abc food\nlist\nreport\ndelete 2\nreport\nquit"
"""A command-line expense tracker: record every expense and total them by category."""
import json
from dataclasses import dataclass, asdict
from datetime import date
from pathlib import Path

DATA_FILE = Path("expenses.json")


# ---------- 1. Data: one expense ----------
@dataclass
class Expense:
    id: int
    day: str            # the date, e.g. "2025-03-01"
    category: str       # the category, e.g. "food"
    amount: float       # the amount
    note: str = ""      # a note


# ---------- 2. Storage: reading and writing the JSON file ----------
def load() -> list[Expense]:
    if not DATA_FILE.exists():
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):
        print("⚠ the data file is damaged; starting with an empty book")
        return []


def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")


# ---------- 3. Logic: add, delete, total (no input or printing, so easy to test) ----------
def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("the amount must be greater than 0")
    new_id = max((e.id for e in expenses), default=0) + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item


def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False


def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))


# ---------- 4. Interface: parsing commands, showing results ----------
HELP = """Commands:
  add AMOUNT CATEGORY [NOTE]   record an expense, e.g. add 35 food lunch
  list                         list every record
  report                       totals by category
  delete ID                    delete an expense
  help                         show this help
  quit                         quit"""


def show_list(expenses: list[Expense]) -> None:
    if not expenses:
        print("(no records yet)")
        return
    for e in expenses:
        print(f"#{e.id:<3} {e.day}  {e.category:<10} {e.amount:>8.2f}  {e.note}")


def show_report(expenses: list[Expense]) -> None:
    rows = by_category(expenses)
    if not rows:
        print("(no records yet)")
        return
    total = sum(amount for _, amount in rows)
    for category, amount in rows:
        bar = "█" * round(amount / rows[0][1] * 20)      # the biggest category gets 20 blocks
        print(f"{category:<10} {amount:>8.2f}  {amount / total:6.1%}  {bar}")
    print(f"total {total:.2f}")


def main() -> None:
    expenses = load()
    print(f"Expense tracker: {len(expenses)} records so far. Type help for the commands.")
    while True:
        try:
            line = input("> ")
        except EOFError:                    # end of input (e.g. Ctrl+D)
            break
        match line.split():
            case ["add", amount, category, *note]:
                try:
                    value = float(amount)
                except ValueError:
                    print(f'✗ "{amount}" is not a number')
                    continue
                try:
                    item = add(expenses, value, category, " ".join(note))
                except ValueError as e:
                    print("✗", e)
                    continue
                save(expenses)
                print(f"✓ recorded #{item.id}: {item.category} {item.amount:.2f}")
            case ["list"]:
                show_list(expenses)
            case ["report"]:
                show_report(expenses)
            case ["delete", expense_id] if expense_id.isdigit():
                if delete(expenses, int(expense_id)):
                    save(expenses)
                    print(f"✓ deleted #{expense_id}")
                else:
                    print(f"✗ there is no record #{expense_id}")
            case ["help"]:
                print(HELP)
            case ["quit"]:
                break
            case []:
                continue
            case _:
                print("✗ unknown command; type help to see how to use it")
    print("Bye!")


if __name__ == "__main__":
    main()
```

> [!NOTE]
> Every `add` and `delete` is followed immediately by `save`, so data isn't lost even if the program is closed halfway. On the website, `expenses.json` lives in the browser's temporary file system and disappears when you reload the page; in the learning app or on your own computer it's saved in the folder the program runs in and is still there next time.

## 16.7 Turning it into a "real project"

The tools from the last lesson come in handy now. In a terminal:

```bash
$ uv init ledger
$ cd ledger
$ uv add --dev pytest ruff
```

Save the program as `ledger.py`. Because the logic-layer functions neither read input nor print, testing them is easy:

```python norun
# file test_ledger.py
from ledger import add, by_category, delete

def test_add_assigns_increasing_ids():
    items = []
    a = add(items, 10, "food")
    b = add(items, 5, "transport")
    assert (a.id, b.id) == (1, 2)

def test_add_rejects_non_positive_amount():
    import pytest
    with pytest.raises(ValueError):
        add([], 0, "food")

def test_delete():
    items = []
    add(items, 10, "food")
    assert delete(items, 1) is True
    assert delete(items, 1) is False      # already deleted
    assert items == []

def test_by_category_sorted_by_amount():
    items = []
    add(items, 10, "transport")
    add(items, 30, "food")
    add(items, 5, "transport")
    assert by_category(items) == [("food", 30), ("transport", 15)]
```

```bash
$ uv run pytest          # run the tests
$ uv run ruff check .    # check the code
$ uv run ruff format .   # consistent formatting
$ uv run ledger.py       # run the program
```

### Command-line arguments instead of a prompt (optional)

Many command-line tools don't make you "enter the program and type commands"; you write everything in one go in the terminal: `python ledger.py add 35 food lunch`. The standard library's **`argparse`** handles such **command-line arguments** and generates `--help` text automatically:

```python norun
import argparse

parser = argparse.ArgumentParser(description="A command-line expense tracker")
sub = parser.add_subparsers(dest="command", required=True)
p_add = sub.add_parser("add", help="record an expense")
p_add.add_argument("amount", type=float)
p_add.add_argument("category")
p_add.add_argument("note", nargs="*")
sub.add_parser("list", help="list every record")
sub.add_parser("report", help="totals by category")

args = parser.parse_args()
if args.command == "add":
    ...   # call the logic layer's add, then save
```

Notice that the logic and storage layers **don't change at all** — only the interface layer is swapped. That's the payoff of layering.

## 16.8 Going further: ideas to try yourself

Once the basics work, try adding features on your own. When you're stuck, revisit the relevant lesson or the official documentation:

1. **Monthly totals**: `report 2025-03` totals March only (hint: `e.day.startswith(...)`).
2. **Budget warnings**: set a monthly budget and warn at 80%.
3. **CSV export**: an `export` command that writes the records to a CSV file spreadsheets can open (lesson 11).
4. **Editing a record**: `edit ID AMOUNT`.
5. **Nice tables**: draw coloured tables with the third-party package `rich` (`uv add rich`).
6. **Charts**: plot monthly spending as a bar chart with `matplotlib`.
7. **Several books**: choose the data file with a command-line argument.

## 16.9 What you've learned

Looking back over this course, you've mastered: variables and types, strings, input and output, conditions and loops, lists, tuples, dictionaries, sets, functions, modules, files, exceptions, classes, generators, decorators, `match`, virtual environments, uv, ruff, pytest… That's already enough to write many useful programs.

Where to go next:

| Direction | What to learn |
|-----------|---------------|
| data analysis | `pandas`, `matplotlib`, Jupyter |
| websites and APIs | FastAPI, Django |
| automation | processing files and spreadsheets, fetching web pages with `requests`, scheduled tasks |
| AI and large language models | this project's **LLM tutorial** (LLM-Tutorial): from the maths basics all the way to training your own GPT |
| a second language | the other courses here: **C** (how computers work underneath), **JavaScript** (the web), **Go** (servers), **Rust** (safe and fast)… |

Most important of all: **find a small project you genuinely want to build, and build it.** When you don't know something, look it up or ask — that's what every programmer does every day.

## 16.10 Exercises

<!-- exercise: parse-add -->
### Exercise 1: Parsing the add command

Read one command. If it has the form `add AMOUNT CATEGORY [NOTE…]` (at least 3 words, the first being `add`) and the amount is a **number greater than 0**, print `AMOUNT | CATEGORY | NOTE` (the amount with 2 decimal places; the note is the remaining words joined with spaces, or `-` if there's no note). If the amount isn't a number or isn't greater than 0, print `invalid amount`; if it isn't an add command or has too few words, print `invalid command`.

```text
Sample input: add 35 food lunch and coffee
Sample output: 35.00 | food | lunch and coffee
```

<details>
<summary>Hint</summary>

Start with `parts = line.split()` and check `len(parts) < 3 or parts[0] != "add"`; handle the amount with `try: amount = float(parts[1])`; the note is `" ".join(parts[3:]) or "-"`.
</details>

<!-- exercise: category-report -->
### Exercise 2: Totals by category

The first line is an integer `n`, followed by `n` lines each with a category and an amount. Print each category and its total (2 decimal places) from the **largest total to the smallest** (ties sorted by category name), then a final line `total GRAND_TOTAL`.

```text
Sample input:
3
food 35
bus 12.5
food 20
Sample output:
food 55.00
bus 12.50
total 67.50
```

<details>
<summary>Hint</summary>

Accumulate in a dict: `totals[c] = totals.get(c, 0) + amount`; sort with `sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))`.
</details>

<!-- exercise: text-bars -->
### Exercise 3: A text bar chart

Read lines each containing a label and a positive integer, ending with a line `end`. For each line print: the label (left-aligned to the width of the longest label), a space, a bar of `#` characters (length = `round(value / maximum × 10)`), a space, and the value.

```text
Sample input:
food 40
bus 20
fun 8
end
Sample output:
food ########## 40
bus  ##### 20
fun  ## 8
```

<details>
<summary>Hint</summary>

Read every line into a list first and compute `width = max(len(label) for label, _ in rows)` and the maximum value; then print with `f"{label:<{width}} {'#' * n} {value}"`.
</details>

<!-- exercise: why-layers -->
### Exercise 4: Why layers?

What is the **main** benefit of splitting a program into data, storage, logic and interface layers?

- the program runs faster
- each layer does one job: the logic can be tested on its own, and changing the interface (to a web page, say) doesn't touch the logic or storage
- the files get smaller
- Python requires it

<details>
<summary>Answer and explanation</summary>

Layers give each part a clear responsibility: the logic layer neither reads input nor prints, so it's easy to test; swapping the command-line interface for argparse, a web page or a GUI only changes the interface layer.
</details>

<!-- exercise: why-file -->
### Exercise 5: Why save to a file?

Why does the expense tracker save its data to `expenses.json` after every change?

- because data in variables is gone when the program ends; saving to a file lets you continue next time
- because files are faster to read and write than variables
- because Python lists can hold at most 100 elements
- because JSON files calculate the total automatically

<details>
<summary>Answer and explanation</summary>

Variables live in memory and vanish when the program ends; files live on disk, and `load()` reads them back at the next start. Saving right after every change means nothing is lost even if the program closes unexpectedly.
</details>

## 16.11 Further reading

- [argparse tutorial](https://docs.python.org/3/howto/argparse.html)
- [Rich: pretty tables and colour in the terminal](https://rich.readthedocs.io/)
- *Python Crash Course* (Eric Matthes): includes complete hands-on projects
- Congratulations on finishing the Python course! Pick your next course from the course list.
