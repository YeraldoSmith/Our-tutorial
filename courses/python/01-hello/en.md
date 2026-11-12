# 1. Hello, Python!

> 🎯 **Goals**: write and run your first Python program; understand `print`, strings and comments; learn to read the two most common error messages.
>
> ⏱ **Time**: 30 minutes　|　📋 **You need**: nothing (just typing)

## 1.1 What is Python, and why start with it?

**Python** is a programming language. A programming language is an agreed way for people to talk to computers: we write instructions following some rules, and the computer carries them out one by one.

Why do so many people learn Python first?

- **It reads like English**: `print("Hello")` means "print: Hello". It barely needs explaining.
- **Few rules**: no piles of braces and semicolons, no need to declare types up front.
- **Used everywhere**: web back-ends, data analysis, automating office work, artificial intelligence (large language models like ChatGPT are trained with Python), science, small games…
- **Huge community**: whatever problem you hit, someone has already asked about it.

> [!NOTE]
> This course uses **Python 3** (the newest stable version is 3.14). Some old tutorials online use Python 2, which looks different (for example `print "hello"` without parentheses). Python 2 was retired in 2020 — **don't learn it**.

## 1.2 Your first program

Type this line into the code editor on the right (or click "▶ Try it" above the code box):

```python
print("Hello, World!")
```

Now click **▶ Run** (shortcut <kbd>F5</kbd>). The "Output" area below shows:

```text
Hello, World!
```

🎉 Congratulations, you've written your first program! When programmers learn a new language, the very first thing they do is make the computer say "Hello, World!" — a tradition going back to 1978.

### What exactly does this line do?

Let's take it apart:

```text
print ( "Hello, World!" )
  │   │        │        │
  │   │        │        └─ closing parenthesis: end of the argument
  │   │        └────────── what to show (a piece of text, called a "string")
  │   └─────────────────── opening parenthesis: start telling print what to show
  └─────────────────────── function name: print
```

- `print` is a **function** that comes with Python. A function is like a little machine: you put something in, it does a job for you. The job of `print` is to **show what's inside the parentheses on the screen**.
- What you put inside the parentheses is called an **argument**.
- The **double quotes** around `"Hello, World!"` tell Python: "this is text — show it as it is, don't treat it as a command". Quoted text is called a **string**.

### Any language works inside the quotes

```python
print("Hello, world!")
print("I am learning Python.")
print("你好，世界！ Привет, мир!")
```

This shows three lines. A program runs **from top to bottom, one line at a time**: first line 1, then line 2, then line 3.

> [!TIP]
> Inside the quotes you can write anything — any language, even emoji. But the **quotes and parentheses themselves** must be plain ASCII symbols. If your keyboard has a Chinese/Japanese input mode, full-width symbols like `（` and `“` are the most common beginner mistake (more below).

## 1.3 Strings: single or double quotes

A string can use double quotes or single quotes — they mean exactly the same:

```python
print("I like programming")
print('Me too')
```

Why two kinds? It's handy when the text itself contains a quote:

```python
print("It's a good day.")          # the text has ' so we wrap it in "
print('She said: "Coding is fun"')  # the text has " so we wrap it in '
```

### New lines and special characters

To start a new line inside a string, write `\n` (backslash + n, for "new line"):

```python
print("Line one\nLine two\nLine three")
```

Output:

```text
Line one
Line two
Line three
```

Combinations that start with a backslash and have a special meaning are called **escape sequences**. Common ones:

| You write | Meaning |
|-----------|---------|
| `\n` | new line |
| `\t` | tab (a stretch of space, for lining things up) |
| `\\` | an actual backslash `\` |
| `\"` | a double quote inside a double-quoted string |

### Multi-line text: triple quotes

For a long piece of text, wrap it in three quotes `"""` and just press Enter inside:

```python
print("""Roses are red,
Violets are blue,
Python is easy,
And fun to use too.""")
```

## 1.4 Printing several things at once

Put several things inside `print`, separated by **commas**; a space is added between them automatically:

```python
print("I am", 18, "years old")
```

Output:

```text
I am 18 years old
```

Notice that `18` has no quotes — it's a **number**, not text. `print` can show both.

You can also let Python do maths:

```python
print(1 + 2)
print("1 + 2")
print("A day has", 24 * 60, "minutes")
```

Output:

```text
3
1 + 2
A day has 1440 minutes
```

See the difference? Without quotes, Python **calculates** `1 + 2` and gets `3`; with quotes, `"1 + 2"` is text and is shown as it is. In Python, `*` means "multiply".

### Changing the separator and the ending

`print` has two useful settings:

- `sep`: what to put between items (a space by default).
- `end`: what to add at the very end (by default `\n`, a new line).

```python
print("2026", "11", "12", sep="-")
print("No new line", end="")
print(", continued on the same line")
```

Output:

```text
2026-11-12
No new line, continued on the same line
```

## 1.5 Comments: notes for humans

Anything after a `#` is a **comment**. Python ignores it completely; it's an explanation for people:

```python
# This is a comment: the next line prints a greeting
print("Hi!")  # a comment can also follow code

# print("this line won't run because of the #")
```

Two main uses of comments:

1. **Explaining code**: tell others (and yourself in three months) what the code does and why.
2. **Switching code off temporarily**: while debugging, put `#` in front of a line so it doesn't run.

> [!TIP]
> Select some lines in the editor and press <kbd>Ctrl</kbd> + <kbd>/</kbd> (<kbd>⌘</kbd> + <kbd>/</kbd> on a Mac) to comment or uncomment them all at once.

## 1.6 How does a program run?

When you click "Run", this happens behind the scenes:

```text
your code (main.py)
      │
      ▼
Python interpreter: read a line → understand it → do it → read the next line…
      │
      ▼
results appear on the screen
```

Python is an **interpreted** language: a program called the "interpreter" (the Python you installed) reads your code line by line and executes it. When you learn C or Rust later, you'll meet the other approach — **compiled**: the whole program is first translated into a file the machine can run directly, and then that file is run.

In this course's app you just click "▶ Run". In your own terminal you can also run a file like this:

```console
python3 hello.py
```

(On Windows it's usually `python hello.py` or `py hello.py`; the "Start here" course explains this step by step.)

## 1.7 Common mistakes (read this!)

Making mistakes is normal; professional programmers make dozens every day. What matters is **reading the error message**. It's in English, but the format is always the same:

```text
  File "main.py", line 1
    print("Hello")
                 ^
SyntaxError: unterminated string literal (detected at line 1)
```

Read it as: **which file (main.py), which line (line 1), where (the `^`), what kind of error (SyntaxError) and the reason.**

### Mistake 1: full-width punctuation

```python norun
print("Hello"）
```

The closing bracket `）` is a full-width character (common with Chinese/Japanese keyboards), and Python doesn't understand it. **Brackets, quotes, commas and colons in code must be typed with an English keyboard layout.**

> [!WARNING]
> Curly "smart quotes" `“ ”` copied from a word processor or a web page cause the same problem. Code must use straight quotes `" "`.

### Mistake 2: typos and capital letters

```python norun
Print("Hello")
```

```text
NameError: name 'Print' is not defined
```

Python is **case-sensitive**: `print` and `Print` are completely different names. Python only knows lowercase `print`.

### Mistake 3: forgetting the quotes

```python norun
print(Hello)
```

```text
NameError: name 'Hello' is not defined
```

Without quotes, Python thinks `Hello` is a "name" (a variable, next lesson) and looks it up. It can't find it, so it complains. Text always needs quotes.

### Mistake 4: unpaired quotes or brackets

```python norun
print("Hello)
```

```text
SyntaxError: unterminated string literal (detected at line 1)
```

"Unterminated string literal" means "a string that never ends" — the closing quote is missing.

> [!TIP]
> When your code fails in this course's app, a "💬 What does this mean?" box under the error explains it in plain words. Read it, then go back and find the problem.

## 1.8 Summary

- `print(...)` shows what's inside the parentheses.
- Text in quotes is a **string**; single and double quotes both work; triple quotes allow several lines.
- `\n` is a new line; `sep` and `end` control the separator and the ending of `print`.
- `1 + 2` without quotes is calculated; `"1 + 2"` with quotes is shown as it is.
- `#` starts a comment, which Python ignores.
- Symbols in code must be plain ASCII; Python is **case-sensitive**.
- Read error messages: file, line, `^` position, error type and reason.

## 1.9 Exercises

<!-- exercise: hello-world -->
### Exercise 1: Hello, World

Write a program that prints exactly this line (capitals, comma and exclamation mark included):

```text
Hello, World!
```

<details>
<summary>Hint</summary>

Use `print` and put the text inside a pair of double quotes.
</details>

<!-- exercise: three-lines -->
### Exercise 2: Introduce yourself in three lines

Use **three** `print` calls to show these three lines:

```text
My name is Alex
I am learning Python
Today is lesson 1
```

> Want a challenge? Try it with just **one** `print` and `\n`.

<details>
<summary>Hint</summary>

Each `print` prints one line. Don't add extra spaces — answers are compared character by character.
</details>

<!-- exercise: math-print -->
### Exercise 3: Let Python do the maths

A year has 365 days and a day has 24 hours. Make Python **calculate** (don't work it out yourself) how many hours are in a year, and print:

```text
A year has 8760 hours
```

<details>
<summary>Hint 1</summary>

`print` can take several items separated by commas; it puts a space between them.
</details>

<details>
<summary>Hint 2</summary>

Multiplication is `*`: `print("A year has", 365 * 24, "hours")`
</details>

<!-- exercise: sep-end -->
### Exercise 4: A date with sep

Use **one** `print` to join the numbers `2026`, `11` and `12` with slashes:

```text
2026/11/12
```

<details>
<summary>Hint</summary>

`print(a, b, c, sep="/")`
</details>

<!-- exercise: predict-1 -->
### Exercise 5: Read the code, predict the output

Don't run it — think first. What does this program print?

```python
print("1 + 1")
print(1 + 1)
print("A", "B", sep="")
```

<details>
<summary>Answer and explanation</summary>

Line 1 has quotes, so it prints `1 + 1` as it is; line 2 has none, so it calculates `2`; line 3 uses `sep=""`, meaning nothing goes between the items: `AB`.
</details>

<!-- exercise: choice-error -->
### Exercise 6: Spot the mistake

Which line **runs correctly**?

- `print（"Hello"）`
- `Print("Hello")`
- `print("Hello")`
- `print(Hello)`

<details>
<summary>Answer and explanation</summary>

The third. The first uses full-width brackets; the second has a capital `P`; the fourth has no quotes, so Python treats `Hello` as a variable name.
</details>

## 1.10 Further reading

- [The official Python tutorial](https://docs.python.org/3/tutorial/index.html): chapter 3, "An Informal Introduction to Python", complements this lesson.
- [learn-python](https://github.com/trekhleb/learn-python): a popular GitHub playground of Python examples with explanations.
- Next lesson: [Variables and data types](../02-variables/en.md) — making a program remember things.
