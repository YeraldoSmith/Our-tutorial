# 12. Exceptions

> 🎯 **Goals**: tell "syntax errors" apart from runtime "exceptions"; catch exceptions with `try / except` so your program doesn't crash when the user types something wrong; use `else`, `finally` and `as e`; report errors from your own functions with `raise`; know the common exception types and how they're related; know when to use exceptions and when to use `if`; avoid "catch everything" `except:` clauses.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lesson 5 of the starter course (reading error messages); lessons 10 and 11 of this course

## 12.1 Two kinds of errors

In lesson 5 of the starter course you learned to read error messages. Python's errors come in roughly two kinds:

1. **Syntax errors** (`SyntaxError`, `IndentationError`): the code doesn't follow Python's grammar, so **the program never starts running**. The only fix is to change the code.
2. **Exceptions**: the grammar is fine, but **something goes wrong when a particular line runs** — dividing by 0, turning `"abc"` into an integer, opening a file that doesn't exist…

```python norun
age = int(input("How old are you? "))     # the user typed "twelve"
print(age + 1)
```

```text
Traceback (most recent call last):
  File "main.py", line 1, in <module>
    age = int(input("How old are you? "))
ValueError: invalid literal for int() with base 10: 'twelve'
```

`ValueError` here is an exception. By default an exception **stops the program at once** and prints a traceback. But many exceptions are predictable (users always type things wrong), and we'd like the program to **handle them itself** instead of crashing.

## 12.2 `try / except`: catching an exception

```python input="twelve"
text = input("How old are you? ")
try:
    age = int(text)                      # code that might fail goes inside try
    print("Next year you'll be", age + 1)
except ValueError:                       # if a ValueError happens, run this instead
    print(f'"{text}" is not a whole number; please type digits, e.g. 12')
print("the program keeps running")
```

What happens:

1. the code inside `try` runs first;
2. if **nothing** goes wrong, the `except` part is skipped;
3. if a `ValueError` happens, the rest of the `try` block is **skipped** and execution jumps into `except ValueError:`;
4. either way, the code after it keeps running.

A common pattern: **keep asking until the input is valid**:

```python input="abc\n-5\n12"
while True:
    try:
        age = int(input("Enter your age: "))
        if age < 0:
            print("age can't be negative")
            continue
        break                                # valid input: leave the loop
    except ValueError:
        print("please enter a whole number")
print("your age is", age)
```

## 12.3 Catching different exceptions

A piece of code may raise several kinds of exception, and you can handle each separately:

```python
def divide(a_text, b_text):
    try:
        a, b = float(a_text), float(b_text)
        return a / b
    except ValueError:
        return "please enter numbers"
    except ZeroDivisionError:
        return "the divisor can't be 0"

print(divide("10", "4"))
print(divide("10", "abc"))
print(divide("10", "0"))
```

To handle several exceptions the same way, put them in a tuple: `except (ValueError, ZeroDivisionError):`.

### Getting the details: `as e`

```python
try:
    nums = [1, 2, 3]
    print(nums[10])
except IndexError as e:
    print("something went wrong:", e)   # e is the exception object; printing it gives the error message
    print("exception type:", type(e).__name__)
```

## 12.4 `else` and `finally`

The complete structure looks like this:

```python
def read_number(text):
    try:
        n = int(text)
    except ValueError:
        print(f'  "{text}": not a whole number')
    else:                               # runs only if try raised nothing
        print(f'  "{text}": got {n}')
    finally:                            # always runs at the end, error or not
        print("  -- done")

read_number("42")
read_number("4.2")
```

- `else` runs only when **no exception** happened. Putting "things that shouldn't happen after an error" here, instead of stuffing them into `try`, keeps `try` around just the line or two that can actually fail.
- `finally` **always runs**, even if `try` contains a `return` or raises an exception nobody catches. It's used for "cleaning up", such as closing files or network connections. (Files opened with `with` close automatically, so you need it less often.)

## 12.5 Raising your own exceptions: `raise`

When someone passes unreasonable arguments to your function, the best response is often not to `print` a message and return an odd value, but to **raise an exception** so the caller knows something went wrong:

```python
def set_age(age: int) -> int:
    if not 0 <= age <= 150:
        raise ValueError(f"age must be between 0 and 150, got {age}")
    return age

for a in (12, -3, 200):
    try:
        print("set:", set_age(a))
    except ValueError as e:
        print("failed:", e)
```

`raise ExceptionType("message")` stops the current function at once and hands the exception to whoever called it; if nobody catches it, the program stops and prints a traceback.

### Custom exceptions

You can define your own exception types to make errors clearer (`class` comes properly in the next lesson; just copy this pattern for now):

```python
class InsufficientFunds(Exception):     # inherits from Exception
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFunds(f"balance is {balance}, can't withdraw {amount}")
    return balance - amount

try:
    print(withdraw(100, 30))
    print(withdraw(100, 300))
except InsufficientFunds as e:
    print("withdrawal failed:", e)
```

## 12.6 Common exception types

| Exception | When it happens | Example |
|-----------|-----------------|---------|
| `ValueError` | right type, unsuitable value | `int("abc")` |
| `TypeError` | wrong type | `"1" + 1` |
| `ZeroDivisionError` | dividing by 0 | `5 / 0` |
| `IndexError` | index out of range | `[1, 2][5]` |
| `KeyError` | key not in the dictionary | `{}["a"]` |
| `AttributeError` | the object has no such attribute or method | `"abc".push(1)` |
| `NameError` | using a name that isn't defined | `print(x)` (x undefined) |
| `FileNotFoundError` | the file doesn't exist | `open("missing.txt")` |
| `KeyboardInterrupt` | the user pressed Ctrl+C | a running program is interrupted |

These exceptions form a "family tree": almost all are "descendants" of `Exception`, `FileNotFoundError` is a descendant of `OSError`, and so on — `except OSError:` catches `FileNotFoundError`, `PermissionError` and their relatives.

## 12.7 Don't write "catch everything" `except`

```python norun
try:
    result = calculate(data)
except:                     # dangerous! catches every exception, including typos in names and Ctrl+C
    print("something went wrong")
```

Written like this, **real bugs get swallowed silently**: you misspell a variable name (`NameError`) and the program only says "something went wrong", with no clue where; when the user presses Ctrl+C to quit, that gets blocked too.

Principles:

1. **catch only exceptions you expect and know how to handle**, such as `except ValueError:`;
2. if you really need a safety net (a server mustn't quit because one request failed), at least write `except Exception as e:` and **log** the error instead of silently ignoring it;
3. never write `except: pass`.

## 12.8 Exceptions or `if`?

Both styles are common:

```python
d = {"a": 1}

# check first, then act (Look Before You Leap)
if "b" in d:
    print(d["b"])
else:
    print("no b")

# act first, handle the error if it happens (Easier to Ask Forgiveness than Permission)
try:
    print(d["b"])
except KeyError:
    print("no b")
```

Rule of thumb: **use `if` for situations that are easy to check in advance and happen often**; **use `try` for situations that are hard to check in advance (will the file be deleted just before you open it? will the network drop?) or that happen rarely**. The Python community leans towards the second style, but neither is wrong.

## 12.9 `assert`: checking things that "should never happen"

`assert condition, "message"` raises an `AssertionError` if the condition is false. It checks **your program's own assumptions** to catch bugs:

```python
def average(nums):
    assert len(nums) > 0, "average() needs at least one number"
    return sum(nums) / len(nums)

print(average([1, 2, 3]))
try:
    average([])
except AssertionError as e:
    print("AssertionError:", e)
```

> [!WARNING]
> `assert` is for catching bugs — **don't** use it to check user input: running Python with `-O` skips every `assert`. Check user input with `if` + `raise ValueError(...)`. The place you'll use `assert` most is in **tests** (pytest, lesson 15).

## 12.10 Summary

- Syntax errors stop the program from starting; **exceptions** happen while it runs and stop it unless handled.
- `try:` holds code that may fail, `except Type:` handles it; `as e` gives the exception object; `else` runs when nothing failed; `finally` always runs.
- Report errors from your own functions with `raise ValueError("message")`; inherit from `Exception` to define your own exceptions.
- Common exceptions: `ValueError`, `TypeError`, `ZeroDivisionError`, `IndexError`, `KeyError`, `FileNotFoundError`…
- Catch only the exceptions you expect; don't write a bare `except:`, and never `except: pass`.
- Use `if` for things easy to check, `try` for things hard to check; `assert` checks the program's own assumptions, not user input.

## 12.11 Exercises

<!-- exercise: safe-int -->
### Exercise 1: Reading integers safely

The first line is an integer `n`, followed by `n` lines, each something a user typed. If it's an integer, print twice its value; otherwise print `invalid`. Use `try / except`.

```text
Sample input:
3
21
abc
-4
Sample output:
42
invalid
-8
```

<details>
<summary>Hint</summary>

Inside the loop: `try: print(int(text) * 2)`, `except ValueError: print("invalid")`.
</details>

<!-- exercise: safe-divide -->
### Exercise 2: Safe division

The first line is an integer `n`, followed by `n` lines, each containing two things `a b`. Print `a / b` with 2 decimal places; if `b` is 0 print `division by zero`; if either isn't a number print `invalid`.

```text
Sample input:
3
10 4
1 0
x 2
Sample output:
2.50
division by zero
invalid
```

<details>
<summary>Hint</summary>

Use two `except` clauses: `except ZeroDivisionError:` and `except ValueError:`. `float("x")` raises `ValueError`.
</details>

<!-- exercise: raise-age -->
### Exercise 3: Reporting errors with raise

Complete the function `check_age(age)`: if `age` is below 0 or above 150, raise `ValueError("age must be between 0 and 150")`; otherwise return `age`. The main program is already written: it reads some integers and prints `ok` for valid ones and the exception's message for invalid ones.

```text
Sample input: 12 -1 151 150
Sample output:
ok
age must be between 0 and 150
age must be between 0 and 150
ok
```

<details>
<summary>Hint</summary>

`if age < 0 or age > 150: raise ValueError("age must be between 0 and 150")`. The message must match exactly.
</details>

<!-- exercise: finally-order -->
### Exercise 4: Predict the output

```python
def f(x):
    try:
        print("start")
        return 10 / x
    except ZeroDivisionError:
        print("oops")
        return None
    finally:
        print("done")

print(f(2))
print(f(0))
```

<details>
<summary>Answer and explanation</summary>

`f(2)`: prints `start` and is about to return 5.0, but `finally` runs first and prints `done`; then the outer `print` shows `5.0`. `f(0)`: prints `start`, the division by 0 jumps to `except`, which prints `oops`, then `finally` prints `done`, and finally the outer `print` shows `None`.
</details>

<!-- exercise: bare-except -->
### Exercise 5: Why not write `except:`?

- because it runs slowly
- because it catches every exception, including bugs from misspelled names and Ctrl+C, so problems get silently hidden
- because Python doesn't allow it
- because it only catches `ValueError`

<details>
<summary>Answer and explanation</summary>

A bare `except:` catches everything, so real errors (like `NameError`) are treated as "normal" and become very hard to find; even the `KeyboardInterrupt` from Ctrl+C gets blocked. Catch only the specific exceptions you expect.
</details>

<!-- exercise: which-exception -->
### Exercise 6: Which exception?

Which exception does `int("3.14")` raise?

- `TypeError`
- `ValueError`
- `SyntaxError`
- no error; the result is 3

<details>
<summary>Answer and explanation</summary>

`ValueError`: the argument is a string (the type is fine), but `"3.14"` isn't how an integer is written. To get 3, write `int(float("3.14"))`.
</details>

## 12.12 Further reading

- [The Python Tutorial · 8. Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html)
- [The built-in exception hierarchy](https://docs.python.org/3/library/exceptions.html#exception-hierarchy)
- Next lesson: [Classes and objects](../13-classes/en.md)
