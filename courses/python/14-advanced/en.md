# 14. Advanced features

> 🎯 **Goals**: master a set of tools that make Python code shorter and more "Pythonic": built-ins like `enumerate`, `zip`, `any` and `all`; generator expressions and **`yield` generators** (memory-saving "lazy" computation); `*args`, `**kwargs` and star unpacking; `match` structural pattern matching; **decorators** and `@cache`; the walrus operator `:=`; modern type hints.
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: lessons 8–13

Nothing in this lesson is "required before you can write programs", but all of it shows up constantly in other people's code, open-source projects and interview questions. Learn it and your code becomes shorter and clearer — and you'll be able to read much more of other people's Python.

## 14.1 Handy built-in functions

```python
names = ["Tom", "Amy", "Bob"]
scores = [88, 95, 72]

for i, name in enumerate(names, start=1):        # the index and the element together (seen in lesson 8)
    print(i, name)

for name, score in zip(names, scores):           # pair up several lists "like a zip"
    print(f"{name}: {score}")

print(dict(zip(names, scores)))                  # two lists straight into a dictionary

nums = [3, -1, 4, 0, 5]
print(any(n < 0 for n in nums))                  # is any number negative? True
print(all(n >= 0 for n in nums))                 # are all of them non-negative? False
print(sum(n for n in nums if n > 0))             # sum of the positives: 12
print(max(names, key=len), min(scores))          # the longest name, the lowest score
print(list(reversed(nums)), sorted(nums, reverse=True))
```

| Function | What it does |
|----------|--------------|
| `enumerate(seq, start=0)` | gives (index, element) pairs |
| `zip(a, b, ...)` | pairs up several sequences, stopping at the shortest |
| `any(...)` / `all(...)` | true if any / all of the items are true |
| `sum`, `min`, `max` | total, smallest, largest (`min`/`max` accept `key=`) |
| `sorted`, `reversed` | a new sorted list, iteration in reverse |
| `map(f, seq)`, `filter(f, seq)` | apply f to every element, keep matching ones (comprehensions are usually used instead) |

## 14.2 Generator expressions: not building the whole list at once

Swap a list comprehension's square brackets for parentheses and you get a **generator expression**: it doesn't compute all the elements immediately, but **one at a time, as they're needed**:

```python
import sys

squares_list = [n * n for n in range(1_000_000)]     # creates a million numbers at once and stores them all
squares_gen = (n * n for n in range(1_000_000))      # just something that "can compute them on demand"

print(sys.getsizeof(squares_list), "bytes")          # several MB
print(sys.getsizeof(squares_gen), "bytes")           # a couple of hundred bytes
print(sum(squares_gen))                               # adds them up as they're computed, never storing a million numbers
```

As the only argument of a function, one set of parentheses can be dropped: `sum(n * n for n in range(10))`. The `any(n < 0 for n in nums)` in the previous section is a generator expression too.

> [!NOTE]
> A generator can be **looped over only once**. Use it again afterwards and it's empty. If you need the values several times, use a list.

## 14.3 `yield`: writing your own generators

Any function containing `yield` becomes a **generator function**: calling it doesn't run it straight away but returns a generator; each time you ask for the next value, the function runs up to the next `yield`, hands over a value and **pauses** there until asked again.

```python
def countdown(n):
    while n > 0:
        yield n           # hand over n, then pause here
        n -= 1
    print("Lift-off!")

for x in countdown(3):    # a for loop keeps asking the generator for the next value
    print(x)
```

Generators suit situations where data is "produced one item after another": reading a big file line by line, infinite sequences, processing data in batches…

```python
def fibonacci():                     # an infinite Fibonacci sequence: no memory problem, one value at a time
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

for i, f in enumerate(fibonacci()):
    if i >= 10:
        break
    print(f, end=" ")
print()

def read_numbers(lines):             # a "pipeline" cleaning data one item at a time
    for line in lines:
        line = line.strip()
        if line and not line.startswith("#"):
            yield int(line)

data = ["10", "# comment", "", "20", " 30 "]
print(sum(read_numbers(data)))       # 60
```

## 14.4 Stars: `*args`, `**kwargs` and unpacking

```python
def average(*nums):                  # *nums collects any number of positional arguments into a tuple
    return sum(nums) / len(nums)

print(average(1, 2, 3), average(10, 20))

def profile(name, **info):           # **info collects any number of keyword arguments into a dict
    print(name, info)

profile("Tom", age=12, city="London")

scores = [90, 85, 77]
print(average(*scores))              # in a call, * "unpacks" a list into separate arguments
options = {"sep": " | ", "end": "!\n"}
print("a", "b", "c", **options)      # ** unpacks a dict into keyword arguments

first, *rest = [1, 2, 3, 4]          # when unpacking, * collects "the rest"
print(first, rest)
*init, last = "abcd"
print(init, last)
merged = [*scores, 100, *[1, 2]]     # merge lists
both = {**{"a": 1}, **{"b": 2}}      # merge dicts (d1 | d2 works too)
print(merged, both)
```

## 14.5 `match`: structural pattern matching

Python 3.10 added the `match` statement. It's not just shorthand for several `if`s — it can take data apart and match it **by its structure**:

```python
def handle(command: str) -> str:
    match command.split():
        case ["quit"]:
            return "bye"
        case ["say", *words]:                 # first word is say; collect the rest in words
            return " ".join(words)
        case ["move", x, y]:                  # exactly three words
            return f"moving to ({int(x)}, {int(y)})"
        case ["move", *_]:
            return "move needs two coordinates"
        case []:
            return "(empty command)"
        case _:                               # _ matches anything: like else
            return f"unknown command: {command}"

for cmd in ["say hello world", "move 3 4", "move 1", "", "quit", "jump"]:
    print(f"{cmd!r:17} → {handle(cmd)}")
```

`case` can also match constants, the shape of dictionaries and attributes of classes, and add conditions with `if` (`case [x, y] if x == y:`). For handling commands or picking apart JSON data, `match` can make code much clearer.

## 14.6 Decorators: "adding features" to functions

`@dataclass` and `@property` from lesson 13 are **decorators**: an `@name` written above a function or class definition that "wraps" it to add extra behaviour.

Since functions are values (lesson 10), you can write a function that "takes a function and returns a new function":

```python
import time
from functools import wraps

def timer(func):                             # takes a function
    @wraps(func)                             # keep the original function's name and docstring
    def wrapper(*args, **kwargs):            # the new function: accepts any arguments
        start = time.perf_counter()
        result = func(*args, **kwargs)       # call the original function
        print(f"{func.__name__} took {time.perf_counter() - start:.4f} s")
        return result
    return wrapper                           # return the new function

@timer                                       # same as slow_sum = timer(slow_sum)
def slow_sum(n):
    return sum(range(n))

print(slow_sum(1_000_000))
```

Writing `@timer` above `def` is the same as running `slow_sum = timer(slow_sum)` after the definition. From then on, calling `slow_sum` actually calls `wrapper`, which adds timing around the original function.

### A useful built-in decorator: `@cache`

The recursive Fibonacci function becomes astonishingly slow as `n` grows, because the same values are recomputed over and over. `functools.cache` **remembers** the result of every call and returns it straight away for the same arguments:

```python
from functools import cache

@cache
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))      # instant; without @cache it wouldn't finish before the universe ends
```

This trick of "remembering results already computed" is called **memoization**, and it's the basis of dynamic programming.

## 14.7 The walrus operator `:=`

`:=` assigns to a variable **in the middle of an expression** (it looks like a walrus's eyes and tusks, hence the name):

```python
import re

text = "order A1234, total 56 dollars"
if (m := re.search(r"\d+", text)) is not None:     # assign to m first, then test it
    print("first number found:", m.group())

data = [3, 8, 1, 9, 4]
big = [y for x in data if (y := x * 10) > 40]       # works in comprehensions too
print(big)
```

It saves some repeated computation and temporary variables — but don't overuse it, or code becomes hard to read.

## 14.8 Modern type hints

Since Python 3.9 you can use the built-in types directly for container types, and since 3.10 `|` means "or":

```python
def word_lengths(words: list[str]) -> dict[str, int]:
    return {w: len(w) for w in words}

def find(names: list[str], target: str) -> int | None:     # the index if found, otherwise None
    return names.index(target) if target in names else None

print(word_lengths(["hi", "hello"]))
print(find(["a", "b"], "b"), find(["a", "b"], "z"))
```

| Hint | Meaning |
|------|---------|
| `list[int]` | a list of integers |
| `dict[str, float]` | a dict with string keys and decimal values |
| `tuple[int, str]` | a tuple of an integer then a string |
| `int \| None` | an integer or `None` |
| `Callable[[int], str]` (from `collections.abc`) | a function taking an integer and returning a string |

## 14.9 Summary

- `enumerate`, `zip`, `any`, `all`, `sum/min/max(key=)`, `sorted` and `reversed` make loops shorter.
- Generator expressions `(x for x in ...)` and `yield` generators "compute only when needed", saving memory and allowing infinite sequences; they can be looped over only once.
- `*args` collects positional arguments and `**kwargs` keyword arguments; in calls, `*` and `**` unpack lists and dicts; `first, *rest = ...` unpacks.
- `match / case` matches data by structure; `_` matches anything.
- A decorator `@deco` is the same as `f = deco(f)`; `@cache` remembers a function's results.
- `:=` assigns inside an expression; type hints can be written `list[int]`, `dict[str, int]`, `int | None`.

## 14.10 Exercises

<!-- exercise: evens-gen -->
### Exercise 1: Write a generator

Complete the generator function `evens(n)` so it produces, one after another, every even number from 0 to `n` (including `n`). The program reads `n` and prints the generated numbers joined with spaces.

```text
Sample input: 9
Sample output: 0 2 4 6 8
```

<details>
<summary>Hint</summary>

`for i in range(0, n + 1, 2): yield i`
</details>

<!-- exercise: star-average -->
### Exercise 2: Any number of arguments

Complete the function `stats(*nums)` so it returns a tuple `(minimum, maximum, average)`. The program reads a line of integers, calls it as `stats(*nums)` and prints the minimum, maximum and average (the average with 2 decimal places).

```text
Sample input: 4 8 15 16 23 42
Sample output: 4 42 18.00
```

<details>
<summary>Hint</summary>

Inside the function, `nums` is a tuple: `return min(nums), max(nums), sum(nums) / len(nums)`.
</details>

<!-- exercise: match-commands -->
### Exercise 3: Handling commands with match

Use `match` to process commands until `quit`:

- `add a b`: print `a + b`;
- `mul a b`: print `a * b`;
- `neg a`: print `-a`;
- any other command: print `unknown`;
- `quit`: end the program (printing nothing).

All numbers are integers.

```text
Sample input:
add 2 3
neg 7
pow 2 3
mul 4 5
quit
Sample output:
5
-7
unknown
20
```

<details>
<summary>Hint</summary>

`match input().split():`, then `case ["add", a, b]: print(int(a) + int(b))` and so on, ending with `case _: print("unknown")`. Leave the loop with `break` on `quit`.
</details>

<!-- exercise: big-fib -->
### Exercise 4: Memoization

Read `n` (0 ≤ n ≤ 300) and print the `n`th Fibonacci number (`fib(0) = 0`, `fib(1) = 1`). Use **recursion + `@cache`**.

```text
Sample input: 90
Sample output: 2880067194370816120
```

<details>
<summary>Hint</summary>

`from functools import cache` and put `@cache` on the recursive function. For `n` up to a few hundred, the recursion is only a few hundred levels deep — within Python's default limit of about 1000.
</details>

<!-- exercise: gen-predict -->
### Exercise 5: Predict the output

```python
def gen():
    print("start")
    yield 1
    print("middle")
    yield 2

g = gen()
print("created")
print(next(g))
print(next(g))
```

<details>
<summary>Answer and explanation</summary>

Calling `gen()` **doesn't start running** the function body, so `created` is printed first. The first `next(g)` runs up to the first `yield`: it prints `start` and hands over 1. The second continues from where it paused: it prints `middle` and hands over 2. The output is `created`, `start`, `1`, `middle`, `2`.
</details>

<!-- exercise: decorator-meaning -->
### Exercise 6: What does a decorator mean?

Which line is this equivalent to?

```python norun
@timer
def work():
    ...
```

- `work = timer`
- `work = timer(work)`
- `timer = work(timer)`
- `work()` runs once right after it's defined

<details>
<summary>Answer and explanation</summary>

`@timer` is the same as writing `work = timer(work)` after the definition: the original `work` is replaced by the new function `timer` returns.
</details>

## 14.11 Further reading

- [The Python Tutorial · 9.9 Generators](https://docs.python.org/3/tutorial/classes.html#generators)
- [PEP 636: Structural Pattern Matching tutorial](https://peps.python.org/pep-0636/)
- [functools documentation](https://docs.python.org/3/library/functools.html), [itertools documentation](https://docs.python.org/3/library/itertools.html)
- Next lesson: [Packages and modern tooling](../15-packages-tooling/en.md)
