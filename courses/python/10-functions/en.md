# 10. Functions

> 🎯 **Goals**: define your own functions with `def` and understand **parameters** and **return values**; tell `return` and `print` apart; use default arguments, keyword arguments and multiple return values; understand **local** and **global variables**; write modern Python functions with **type hints** and docstrings; learn that "functions are values" and use `lambda`; meet **recursion**; avoid the "mutable default argument" trap.
>
> ⏱ **Time**: 75 minutes　|　📋 **You need**: lessons 6–9

## 10.1 Why do we need functions?

You've already used plenty of functions: `print()`, `len()`, `input()`, `sorted()`… Each one **packages up** a ready-made piece of work and **gives it a name**, so using it only takes the name.

Writing your own functions has three benefits:

1. **No repetition**: write the code once and call it everywhere; to change it, change one place.
2. **Readability**: `if is_prime(n):` is much easier to understand than a big loop.
3. **Testability**: check each function on its own, so problems are easy to locate.

## 10.2 Defining and calling

```python
def greet(name):                  # def function_name(parameter):
    print(f"Hello, {name}!")      # the function body: indented 4 spaces

greet("Tom")                      # call it: function_name(argument)
greet("Amy")
```

- `def` is short for **define**;
- `name` is a **parameter**: the value passed in when the function is called goes into this variable;
- the body must be **indented**, just like with `if` and `for`;
- defining a function **doesn't run** the body; it runs only when the function is **called**. So define first, call afterwards.

## 10.3 Return values: `return`

A function can "hand back" a result to the place that called it:

```python
def area(width, height):
    return width * height          # hand back the result; the function ends here

a = area(3, 4)                     # a return value can be stored in a variable
print(a)                           # 12
print(area(2, 5) + area(1, 1))     # or used directly in a calculation: 11
```

### `return` versus `print`

This is what beginners mix up most often:

```python
def add_print(a, b):
    print(a + b)        # only shows it on the screen

def add_return(a, b):
    return a + b        # hands the result back

x = add_print(1, 2)     # shows 3 on the screen, but x gets None
y = add_return(1, 2)    # shows nothing, but y gets 3
print("x =", x)
print("y =", y)
```

`print` is for **people** to read; `return` is for the **program** to use. A function with no `return` (or just a bare `return`) returns `None`.

## 10.4 Ways to pass arguments

```python
def power(base, exp=2):                 # exp has a default: 2 is used if you don't pass it
    return base ** exp

print(power(5))                          # 25: uses the default
print(power(5, 3))                       # 125: arguments by position
print(power(exp=3, base=2))              # 8: arguments by name (keyword arguments), in any order

def describe(name, age, city="London"):
    return f"{name}, {age}, lives in {city}"

print(describe("Tom", 12))
print(describe("Amy", 13, city="Paris"))
```

Rules: **parameters with defaults must come after those without**. When calling, positional arguments come before keyword arguments.

## 10.5 Returning several values

Separate several values with commas and you actually return a **tuple**, which you can unpack directly (lesson 8):

```python
def min_max(nums):
    return min(nums), max(nums)

lo, hi = min_max([3, 9, 1, 7])
print(lo, hi)            # 1 9

def divide(a, b):
    return a // b, a % b         # quotient and remainder

q, r = divide(17, 5)
print(f"17 = 5 × {q} + {r}")
```

## 10.6 Local and global variables

Variables created **inside** a function are **local**: they exist only in that function and vanish when it ends:

```python
def f():
    secret = 42           # a local variable
    return secret

print(f())
# print(secret)           # this would be a NameError: the outside can't see variables inside the function
```

Variables created **outside** any function are **global**, and a function can **read** them. But if you **assign** to such a name inside a function, Python assumes you're creating a local variable with the same name:

```python
count = 0                 # a global variable

def show():
    print("count =", count)    # just reading it: fine

show()
```

```python norun
count = 0

def increase():
    count = count + 1     # trying to change the global variable…
    return count

increase()
```

```text
UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
```

You could declare `global count` to change the global variable, but **the better way is: pass values in through parameters and out through `return`**:

```python
def increase(count):
    return count + 1

count = 0
count = increase(count)
count = increase(count)
print(count)              # 2
```

Then the function depends only on its parameters, which makes it easier to understand and test.

## 10.7 The modern style: type hints and docstrings

Modern Python code usually includes **type hints** and a **docstring**:

```python
def bmi(weight_kg: float, height_m: float) -> float:
    """Body mass index: BMI = weight / height²."""
    return weight_kg / height_m ** 2

print(f"{bmi(60, 1.70):.1f}")
print(bmi.__doc__)        # the docstring is stored in __doc__; in an interactive session help(bmi) shows more
```

- `weight_kg: float` says "this parameter should be a decimal number", and `-> float` says "it returns a decimal number";
- the triple-quoted string is the docstring, describing what the function does.

> [!NOTE]
> Type hints are **not** enforced by Python — passing the wrong type still runs. They're written for people and tools: editors use them for suggestions, and tools like `mypy` and `pyright` can catch type errors early (we'll use one in lesson 15).

## 10.8 Functions are values

Functions, like numbers and strings, can be stored in variables and passed to other functions:

```python
def double(x):
    return x * 2

f = double                     # no parentheses: this is the function itself, not a call
print(f(21))                   # 42

def apply_twice(func, value):
    return func(func(value))

print(apply_twice(double, 3))  # 12
```

A small function used only once can be written on one line with **`lambda`**: `lambda parameters: expression`. Its most common use is telling `sorted`, `max` and `min` what to compare by (you saw this in lesson 9):

```python
words = ["banana", "kiwi", "apple", "fig"]
print(sorted(words, key=len))                    # sort by length
print(sorted(words, key=lambda w: w[-1]))        # sort by the last letter

students = [("Tom", 88), ("Amy", 95), ("Bob", 72)]
print(max(students, key=lambda s: s[1]))         # the highest score
```

## 10.9 Recursion: a function calling itself

A function can **call itself**; this is **recursion**. Take the factorial, `n! = n × (n−1)!` with `0! = 1`:

```python
def factorial(n: int) -> int:
    if n == 0:                    # the stopping condition: essential, or it calls itself forever
        return 1
    return n * factorial(n - 1)   # turn the problem into a smaller problem of the same kind

print(factorial(5))               # 5 × 4 × 3 × 2 × 1 = 120
```

The calls unfold like this:

```text
factorial(3)
= 3 * factorial(2)
= 3 * (2 * factorial(1))
= 3 * (2 * (1 * factorial(0)))
= 3 * (2 * (1 * 1))
= 6
```

Recursion is elegant for problems where "a big problem contains smaller copies of itself" (like folders inside folders). But forget the stopping condition, or recurse too deeply (Python's default limit is about 1000 levels), and you get a `RecursionError`. Many recursive functions can also be rewritten as loops.

## 10.10 A trap: mutable default arguments

```python
def add_item(item, box=[]):        # dangerous! the default list is created only once, when the function is defined
    box.append(item)
    return box

print(add_item("apple"))           # ['apple']
print(add_item("banana"))          # ['apple', 'banana']  ← last time's "apple" is still there!
```

A default value is evaluated only **once, when the function is defined**, and every call shares that same list. The correct way is to use `None` as the default:

```python
def add_item(item, box=None):
    if box is None:
        box = []                   # a new list on every call
    box.append(item)
    return box

print(add_item("apple"))
print(add_item("banana"))
```

**Never use mutable things like lists, dictionaries or sets as default values.**

## 10.11 Common mistakes

```python norun
def area(w, h):
    return w * h

print(area(3))
```

```text
TypeError: area() missing 1 required positional argument: 'h'
```

Wrong number of arguments: the definition has 2 parameters but the call gave only 1.

```python norun
print(square(4))

def square(x):
    return x * x
```

```text
NameError: name 'square' is not defined
```

The call comes before the definition. Move the definition above the call.

```python norun
def square(x):
    x * x              # forgot return

print(square(4) + 1)
```

```text
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

The function has no `return`, so it returned `None`, and `None + 1` is an error.

## 10.12 Summary

- `def name(parameters):` defines a function; the indented part is its body; define first, call afterwards.
- `return` hands a result back to the caller and ends the function; with no `return` the result is `None`. `print` only displays.
- Default arguments `def f(x, y=2)`; keyword arguments `f(y=3, x=1)`; returning several values gives a tuple you can unpack.
- Variables inside a function are local; to change outside values, pass them in as parameters and out with `return` rather than using `global`.
- Type hints `def f(x: int) -> str:` and docstrings make code clearer.
- Functions are values and can be passed around; `lambda` writes small one-line functions, often for `sorted(key=...)`.
- Recursion = a function calling itself; it must have a stopping condition.
- Don't use mutable objects as default arguments; use `None` instead.

## 10.13 Exercises

<!-- exercise: max3 -->
### Exercise 1: The largest of three

Complete the function `max3(a, b, c)` so it returns the largest of the three numbers (**don't** use the built-in `max`; compare with `if`). The program reads a line with three integers and prints the result of `max3`.

```text
Sample input: 3 9 4
Sample output: 9
```

<details>
<summary>Hint</summary>

Start with `biggest = a`, compare it with `b` and then `c`, update `biggest` whenever you find something bigger, and finally `return biggest`.
</details>

<!-- exercise: is-prime -->
### Exercise 2: Is it prime?

Complete the function `is_prime(n)`: return `True` if `n` is prime and `False` otherwise. (A prime is an integer greater than 1 that is divisible only by 1 and itself.) The program reads an integer and prints `yes` if it's prime, `no` otherwise.

```text
Sample input: 97
Sample output: yes
```

<details>
<summary>Hint</summary>

Return `False` straight away for numbers below 2; then check `for d in range(2, int(n ** 0.5) + 1):` and return `False` if anything divides evenly; if the loop finishes without that, return `True`.
</details>

<!-- exercise: grade -->
### Exercise 3: Grades

Complete the function `grade(score)` to turn a score into a grade: 90 and above `A`, 80–89 `B`, 70–79 `C`, 60–69 `D`, below 60 `F`. Read a line of scores and print the corresponding grades, separated by spaces.

```text
Sample input: 95 82 60 59
Sample output: A B D F
```

<details>
<summary>Hint</summary>

Use `if / elif / else` inside the function, checking from the highest grade down; in the main program, `print(" ".join(grade(s) for s in scores))`.
</details>

<!-- exercise: factorial -->
### Exercise 4: Factorial by recursion

Complete the function `factorial(n)` **using recursion**. Read an integer `n` (0 ≤ n ≤ 20) and print `n!`.

```text
Sample input: 5
Sample output: 120
```

<details>
<summary>Hint</summary>

Stopping condition `if n == 0: return 1`, otherwise `return n * factorial(n - 1)`.
</details>

<!-- exercise: scope-predict -->
### Exercise 5: Predict the output

```python
x = 10

def f(x):
    x = x + 1
    return x

y = f(x)
print(x, y)

def g():
    return

print(g())
```

<details>
<summary>Answer and explanation</summary>

The `x` inside `f` is a parameter (a local variable); changing it doesn't affect the outer `x`, so the output is `10 11`. `g` has a bare `return`, so it returns `None`.
</details>

<!-- exercise: mutable-default -->
### Exercise 6: A mutable default argument

What does this program print?

```python
def add(item, box=[]):
    box.append(item)
    return box

add(1)
print(add(2))
```

- `[2]`
- `[1, 2]`
- an error
- `None`

<details>
<summary>Answer and explanation</summary>

`[1, 2]`. The default list is created only once, so the 1 from the first call is still in it. Use `box=None` and create a new list inside the function.
</details>

## 10.14 Further reading

- [The Python Tutorial · 4.8 Defining Functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)
- [The typing module: type hints](https://docs.python.org/3/library/typing.html)
- Next lesson: [Modules and files](../11-modules-files/en.md)
