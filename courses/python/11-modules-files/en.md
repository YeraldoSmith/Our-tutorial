# 11. Modules and files

> 🎯 **Goals**: use Python's built-in "toolbox" (the standard library) with `import`; split your own code into modules and understand `if __name__ == "__main__":`; read and write text files with `with open(...)`, and **always remember `encoding="utf-8"`**; handle paths with `pathlib`; read and write CSV tables and JSON data, so your data survives after the program closes.
>
> ⏱ **Time**: 75 minutes　|　📋 **You need**: lessons 8–10

## 11.1 `import`: using ready-made tools

Python comes with hundreds of **modules**, together called the **standard library**. A module is a `.py` file full of ready-made functions and variables. `import` "brings it in":

```python
import math

print(math.sqrt(16))       # square root: 4.0
print(math.pi)             # pi
print(math.ceil(2.1))      # round up: 3
```

There are several ways to write `import`:

```python
import math                       # use it as math.sqrt
from math import sqrt, pi         # take out sqrt and pi directly; no math. prefix needed
import datetime as dt             # give it a shorter nickname

print(sqrt(2), pi)
print(dt.date.today().year >= 2024)
```

> [!TIP]
> `import math` is usually the recommended style: seeing `math.sqrt` tells you where the function comes from. **Don't** write `from math import *` (bringing in everything) — names easily clash with your own variables.

### Commonly used standard-library modules

| Module | Purpose | Example |
|--------|---------|---------|
| `math` | maths functions | `math.sqrt(2)`, `math.floor(3.7)` |
| `random` | random numbers | `random.randint(1, 6)`, `random.choice(lst)` |
| `datetime` | dates and times | `datetime.date.today()` |
| `statistics` | statistics | `statistics.mean(nums)`, `statistics.median(nums)` |
| `collections` | more data structures | `Counter`, `defaultdict`, `deque` |
| `pathlib` | file paths | `Path("data.txt").exists()` |
| `json`, `csv` | data formats | see the second half of this lesson |
| `time` | timing, pausing | `time.sleep(1)`, `time.perf_counter()` |
| `os`, `sys` | the operating system and interpreter | `sys.version`, `os.getcwd()` |

```python
import random
import statistics
from datetime import date, timedelta

random.seed(1)                                   # fix the random seed: the same result every run
dice = [random.randint(1, 6) for _ in range(10)]
print("10 dice rolls:", dice)
print("mean:", statistics.mean(dice), " median:", statistics.median(dice))

d = date(2025, 1, 1)
print(d + timedelta(days=100))                   # the date 100 days later
print(d.isoweekday())                            # day of the week (1 = Monday … 7 = Sunday)
```

To find out what a module offers, check the [Python documentation](https://docs.python.org/3/library/index.html) or type `help(math)` in an interactive session.

## 11.2 Writing your own modules

As a program grows, you can split it into several files. Suppose a folder contains two files:

```python norun
# file geometry.py
PI = 3.14159

def circle_area(r):
    return PI * r * r
```

```python norun
# file main.py
import geometry

print(geometry.circle_area(2))
print(geometry.PI)
```

When you run `main.py`, `import geometry` finds `geometry.py` in the same folder and lets you use its functions. **A module's name is its file name without `.py`**, so avoid spaces and non-English letters in file names, and **don't reuse standard-library names** (don't call your file `math.py` or `random.py`, or `import random` will import your own file instead).

### `if __name__ == "__main__":`

Every module has a special variable `__name__`: when you run the file directly it is `"__main__"`; when another file `import`s it, it's the module's name.

```python
def main():
    print("the program starts here")

print("__name__ is:", __name__)

if __name__ == "__main__":      # runs only when this file is run directly
    main()
```

Put "the things to do when run directly" inside `if __name__ == "__main__":`, and the file **can be run directly or imported by other files without running anything by accident**. This is the standard way to write Python programs.

> [!NOTE]
> The learning app and the website run one file at a time, so this lesson's multi-file examples are marked as not runnable. To try them, create a folder on your computer, put both files in it, and run `python main.py` in a terminal (the terminal was covered in part 1 lesson 3).

## 11.3 Reading and writing files

Data in variables is gone when the program ends. To **keep it**, write it to a **file**.

### Writing a file

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("line one\n")            # write doesn't add a newline: add \n yourself
    f.write("line two\n")
    print("line three", file=f)      # print can write to a file too (with a newline)

print("done")
```

- `open(filename, mode, encoding=...)` opens a file and gives you a **file object** `f`;
- mode `"w"`: write (**an existing file is emptied**); `"a"`: append to the end; `"r"`: read (the default);
- **`with ... as f:`**: the file is **closed automatically** when the indented block ends, even if an error happens. Always use `with`.

### Reading a file

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("apple 5\nbanana 3\norange 8\n")

with open("notes.txt", encoding="utf-8") as f:       # the default mode is "r"
    content = f.read()                                 # read everything at once (one string)
print(repr(content))

with open("notes.txt", encoding="utf-8") as f:
    for line in f:                                     # read line by line (most common, saves memory)
        name, count = line.split()
        print(f"{name}: {count}")
```

When reading line by line, each line ends with a newline `\n`; `line.strip()` removes it (`split()` above removes it automatically).

### Always write `encoding="utf-8"`!

Files store **bytes**, which are converted to text using an **encoding** (UTF-8 was covered in the tokenization lesson). If you don't specify one, Python uses **the operating system's default encoding** — often GBK on Chinese Windows, cp1251 on Russian Windows, UTF-8 on macOS and Linux. The result: a program that works fine on one computer shows garbled text or crashes on another:

```text
UnicodeDecodeError: 'gbk' codec can't decode byte 0xa7 in position 2: illegal multibyte sequence
```

**Make it a habit: write `encoding="utf-8"` every time you `open` a text file.** (Under PEP 686, Python 3.15 uses UTF-8 by default; but many computers still have older versions, so keep writing it.)

## 11.4 `pathlib`: the modern way to handle paths

Different systems write paths differently: Windows uses `C:\Users\tom\data.txt`, macOS/Linux use `/home/tom/data.txt`. With `pathlib` you don't have to worry:

```python
from pathlib import Path

folder = Path("my_data")
folder.mkdir(exist_ok=True)                      # create a folder (no error if it exists)
file = folder / "hello.txt"                      # join paths with / — correct on every system
file.write_text("Hello, pathlib!\n", encoding="utf-8")   # write a file in one line
print(file.read_text(encoding="utf-8"))          # read a file in one line
print(file.exists(), file.name, file.suffix)     # True hello.txt .txt
print(sorted(p.name for p in folder.glob("*.txt")))   # find every .txt file in the folder
```

> [!NOTE]
> A **relative path** like `"notes.txt"` is relative to the **current working directory**, which isn't necessarily the folder your `.py` file is in. If the program can't find a file, first `print(Path.cwd())` to see where you are.

## 11.5 CSV: tabular data

**CSV** (comma-separated values) is the simplest table format, and spreadsheet programs like Excel open it. One record per line, columns separated by commas:

```python
import csv

rows = [["name", "english", "maths"], ["Tom", 90, 85], ["Amy", 95, 92]]
with open("scores.csv", "w", newline="", encoding="utf-8") as f:
    csv.writer(f).writerows(rows)

with open("scores.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):              # each row becomes a dict keyed by the header
        total = int(row["english"]) + int(row["maths"])
        print(row["name"], "total", total)
```

- open CSV files with `newline=""` to avoid extra blank lines on Windows;
- everything read back is a **string** — convert with `int()` or `float()` before calculating;
- for Excel to show non-English text correctly, you can write the file with `encoding="utf-8-sig"`.

## 11.6 JSON: saving structured data

The "lists + dictionaries" data from the last lesson can be saved as a **JSON** file as-is. JSON is the most widely used data format on the web, and almost every programming language reads and writes it:

```python
import json

data = {"name": "Tom", "age": 12, "hobbies": ["football", "drawing"], "member": True}

with open("profile.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)   # write to the file

with open("profile.json", encoding="utf-8") as f:
    print(f.read())                                     # see what the file looks like

with open("profile.json", encoding="utf-8") as f:
    loaded = json.load(f)                               # read it back: a dict again
print(loaded["hobbies"][0], type(loaded))

text = json.dumps(data, ensure_ascii=False)             # dumps / loads convert to and from strings
print(text)
print(json.loads(text) == data)
```

- `ensure_ascii=False` keeps non-English text (Chinese, Russian…) **as it is**; otherwise it becomes escapes like `\u5c0f\u660e`;
- `indent=2` lays it out with indentation for people to read;
- Python's `True`/`False`/`None` are written `true`/`false`/`null` in JSON.

## 11.7 Common mistakes

```python norun
with open("no_such_file.txt", encoding="utf-8") as f:
    print(f.read())
```

```text
FileNotFoundError: [Errno 2] No such file or directory: 'no_such_file.txt'
```

The file name is misspelled, or the file isn't in the current working directory. Check with `Path("...").exists()` first, or `print(Path.cwd())` to see the current directory.

```python norun
import random

def random():          # your own function has the same name as a module
    return 4

print(random.randint(1, 6))
```

```text
AttributeError: 'function' object has no attribute 'randint'
```

The name `random` was overwritten by your own function. Don't give your variables, functions or files the same names as modules.

```python norun
with open("data.txt", "w") as f:
    f.write(123)
```

```text
TypeError: write() argument must be str, not int
```

`write` only accepts strings: write `f.write(str(123))` (and don't forget `encoding="utf-8"`).

## 11.8 Summary

- `import module`, `from module import name`, `import module as nickname`; the standard library includes `math`, `random`, `datetime`, `statistics`, `pathlib`, `json`, `csv` and more.
- Your own `.py` files are modules; don't reuse standard-library names; `if __name__ == "__main__":` separates "run directly" from "imported".
- Files: `with open(name, mode, encoding="utf-8") as f:`; `"r"` reads, `"w"` overwrites, `"a"` appends; `for line in f` reads line by line.
- **Always write `encoding="utf-8"`**, or text may be garbled on other systems.
- `pathlib.Path` handles paths on every platform: `/` to join, `read_text`, `write_text`, `exists`, `glob`.
- CSV with the `csv` module (use `newline=""`); JSON with `json.dump/load` (use `ensure_ascii=False`).

## 11.9 Exercises

<!-- exercise: stats -->
### Exercise 1: Statistics with the standard library

Read a line of numbers and use the `statistics` module to print their **mean** and **median**, each with 2 decimal places, separated by a space.

```text
Sample input: 3 1 4 1 5 9 2 6
Sample output: 3.88 3.50
```

<details>
<summary>Hint</summary>

`import statistics`, then `statistics.mean(nums)` and `statistics.median(nums)`.
</details>

<!-- exercise: weekday -->
### Exercise 2: Day of the week, day of the year

Read a date in `YYYY-MM-DD` format and print which day of the week it is (1 = Monday, 7 = Sunday) and which day of the year it is, separated by a space.

```text
Sample input: 2025-03-01
Sample output: 6 60
```

<details>
<summary>Hint</summary>

`from datetime import date`; `d = date.fromisoformat(text)`; `d.isoweekday()` is the day of the week and `d.timetuple().tm_yday` the day of the year.
</details>

<!-- exercise: to-json -->
### Exercise 3: Turn it into JSON

The first line is a name and the second an age (an integer). Put them into a dictionary `{"name": name, "age": age}` and print it with `json.dumps` (non-English letters must appear as they are).

```text
Sample input:
Tom
12
Sample output: {"name": "Tom", "age": 12}
```

<details>
<summary>Hint</summary>

`json.dumps(data, ensure_ascii=False)`. Convert the age with `int()` first, so the output shows `12` rather than `"12"`.
</details>

<!-- exercise: file-roundtrip -->
### Exercise 4: Write it to a file and read it back

The first line is an integer `n`, followed by `n` lines of text. Write these `n` lines to the file `lines.txt` (remember `encoding="utf-8"`), then **open the file again and read it back**, printing how many lines it has and the longest line (if several are equally long, the first one).

```text
Sample input:
3
apple
melon and grapes
banana
Sample output:
3
melon and grapes
```

<details>
<summary>Hint</summary>

Add `"\n"` to each line when writing; when reading, `lines = [line.rstrip("\n") for line in f]`, and the longest line is `max(lines, key=len)`.
</details>

<!-- exercise: why-encoding -->
### Exercise 5: Why specify the encoding?

The main reason to always write `encoding="utf-8"` when opening text files is:

- to make files smaller
- so the program reads and writes non-English text (Chinese, Russian…) without garbling or errors on every operating system (Chinese Windows, for example)
- to read files faster
- files can't be opened without it

<details>
<summary>Answer and explanation</summary>

Without it, Python uses the system's default encoding — often GBK on Chinese Windows or cp1251 on Russian Windows. A file written in UTF-8 and read there with another encoding comes out garbled or raises `UnicodeDecodeError`.
</details>

<!-- exercise: why-with -->
### Exercise 6: What `with` does

Why is `with open(...) as f:` better than plain `f = open(...)`?

- it can open bigger files
- the file is closed automatically when the block ends, even if an error happens in the middle
- it adds `encoding="utf-8"` automatically
- it backs up the file automatically

<details>
<summary>Answer and explanation</summary>

`with` guarantees the file is closed. Until a file is closed, what you wrote may not actually be saved to disk, and it ties up system resources. It doesn't choose the encoding for you.
</details>

## 11.10 Further reading

- [The Python Standard Library](https://docs.python.org/3/library/index.html)
- [The Python Tutorial · 7.2 Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [pathlib documentation](https://docs.python.org/3/library/pathlib.html), [json documentation](https://docs.python.org/3/library/json.html)
- Next lesson: [Exceptions](../12-exceptions/en.md)
