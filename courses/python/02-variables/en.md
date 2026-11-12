# 2. Variables and data types

> 🎯 **Goals**: store data in variables and change it; know the naming rules; meet integers, floats, strings, booleans and `None`; check types with `type()` and convert with `int()`, `str()` and friends.
>
> ⏱ **Time**: 40 minutes　|　📋 **You need**: lesson 1

## 2.1 Variables: giving data a name

In lesson 1 we printed data directly. Real programs need to **remember** data and use it later — a game remembers the player's score, name and health. That's what **variables** are for:

```python
score = 100
name = "Alice"
print(name, "has", score, "points")
```

Output:

```text
Alice has 100 points
```

`score = 100` means: **create the name `score` and make it refer to the value `100`**. From then on, whenever you write `score`, Python uses `100`.

Think of a variable as a **label stuck onto a value**:

```text
   score ──►  100
   name  ──►  "Alice"
```

> [!IMPORTANT]
> This `=` is **not "equals" from maths**; it's **assignment**: give the value on the right to the name on the left. Read it as "score **gets** 100". To test whether two things are equal you use two equals signs, `==` (lesson 6).

## 2.2 Changing a variable

A variable is called "variable" because its value can change:

```python
score = 100
print(score)
score = 120        # the label moves to a new value
print(score)
```

Output:

```text
100
120
```

More often you change it **based on its current value**:

```python
score = 100
score = score + 10    # first the right side: 100 + 10 = 110, then store it in score
print(score)
```

`score = score + 10` would be nonsense in maths but is completely normal in programming: **evaluate the right side, then assign to the left**. It's so common that Python has shorthands:

```python
score = 100
score += 10     # same as score = score + 10
score -= 30     # same as score = score - 30
score *= 2      # same as score = score * 2
print(score)    # (100 + 10 - 30) * 2 = 160
```

### Assigning one variable to another

```python
a = 5
b = a       # b also refers to 5
a = 8       # a moves to 8; b is unaffected
print(a, b)
```

This prints `8 5`. `b = a` gives `b` the value `a` had **at that moment**; changing `a` later doesn't change `b`.

## 2.3 Naming rules

You can't name a variable just anything. These **rules** must be followed (breaking them is an error):

1. only **letters, digits and the underscore `_`**;
2. **can't start with a digit**: `player1` is fine, `1player` is not;
3. **case-sensitive**: `Score` and `score` are different variables;
4. no Python **keywords** such as `if`, `for`, `class`, `True`, `None`.

And some **conventions** (not errors, but everybody follows them):

- **lowercase words joined by underscores**, called snake_case: `player_name`, `max_speed`.
- **meaningful names**: `total_price` is far better than `tp`, `a` or `x1`.
- values that never change (constants) in **UPPER CASE**: `PI = 3.14159`, `MAX_LEVEL = 99`.

| Name | Allowed? | Note |
|------|----------|------|
| `age` | ✅ | |
| `user_name` | ✅ | recommended style |
| `userName` | ✅ | works, but Python style prefers underscores |
| `_count` | ✅ | a leading underscore has a special meaning (later) |
| `2nd_place` | ❌ | starts with a digit |
| `my-name` | ❌ | `-` is read as minus |
| `class` | ❌ | a keyword |
| `возраст` | ✅ | Python 3 allows non-English letters, but it's **not recommended** |

> [!TIP]
> When naming something, ask yourself: in three months, will I know at a glance what this variable holds?

## 2.4 Data types

Every value in Python has a **type** that decides what you can do with it. The four basic ones:

| Type | Name | Examples | Used for |
|------|------|----------|----------|
| `int` | integer | `42`, `-7`, `0` | counting, ages, scores |
| `float` | floating-point number (decimal) | `3.14`, `-0.5`, `2.0` | prices, heights, temperatures |
| `str` | string (text) | `"hello"`, `'abc'`, `""` | names, messages |
| `bool` | boolean | `True`, `False` | yes/no, on/off |

`type()` tells you a value's type:

```python
print(type(42))
print(type(3.14))
print(type("hello"))
print(type(True))
```

Output:

```text
<class 'int'>
<class 'float'>
<class 'str'>
<class 'bool'>
```

### Things to watch

- `2` is an `int`, `2.0` a `float`, `"2"` a `str` — **they look alike but are different types**.
- `True` and `False` start with a **capital letter**; `true` gives a `NameError`.
- Underscores make big numbers readable: `population = 8_100_000_000` (Python ignores them).
- There's a special value **`None`** meaning "**nothing here**", of type `NoneType` — e.g. `winner = None` before anyone has won.

### Dynamic typing

In Python a variable has no fixed type; **the value has the type**. One variable can refer to an integer and later to a string:

```python
x = 10
print(type(x))
x = "ten"
print(type(x))
```

This is **dynamic typing** — convenient, but easy to get wrong, so a good habit is to keep one kind of thing in each variable. (Java, C and Rust are **statically typed**: a variable's type is fixed when it's declared. You'll see the difference later.)

## 2.5 Converting between types

You often need to convert. The most common case: **numbers and text can't be glued together directly**.

```python norun
age = 18
print("I am " + age + " years old")    # ❌ error
```

```text
TypeError: can only concatenate str (not "int") to str
```

A string can only be joined with another string using `+`. The fix is to **turn the number into a string**:

```python
age = 18
print("I am " + str(age) + " years old")
```

Common conversion functions:

| Function | Does | Example | Result |
|----------|------|---------|--------|
| `int(x)` | to integer | `int("42")`, `int(3.9)` | `42`, `3` (drops the fraction — **not** rounding) |
| `float(x)` | to float | `float("3.5")`, `float(2)` | `3.5`, `2.0` |
| `str(x)` | to string | `str(100)` | `"100"` |
| `bool(x)` | to boolean | `bool(0)`, `bool("hi")` | `False`, `True` |

```python
print(int("42") + 8)       # 50: the text "42" becomes a number first
print("42" + "8")          # 428: two strings joined
print(int(3.9))            # 3
print(round(3.9))          # 4: use round to round
```

Not everything converts: `int("abc")` and `int("3.5")` both raise `ValueError` because they don't look like whole numbers.

> [!NOTE]
> `bool()` rules: `0`, `0.0`, the empty string `""` and `None` become `False`; almost everything else is `True`. This matters for conditions in lesson 6.

## 2.6 Several assignments at once

```python
x, y, z = 1, 2, 3        # three variables in one go
print(x, y, z)

a = b = 0                # both are 0
print(a, b)
```

A very handy trick — **swapping two variables**:

```python
left = "apple"
right = "banana"
left, right = right, left
print(left, right)       # banana apple
```

Many languages need a temporary variable to swap; Python does it in one line.

## 2.7 f-strings: putting variables into text

Joining with `+` and `str()` is clumsy. Python has something better — **f-strings**: put an `f` before the string and write `{name}` to insert a variable:

```python
name = "Alice"
age = 18
height = 1.756
print(f"I'm {name}, {age} years old, {height} m tall")
print(f"Next year I'll be {age + 1}")      # any expression works inside the braces
print(f"Height about {height:.2f} m")      # :.2f means 2 decimal places
```

Output:

```text
I'm Alice, 18 years old, 1.756 m tall
Next year I'll be 19
Height about 1.76 m
```

f-strings are the most common way to format text in modern Python and appear in almost every lesson from now on. Lesson 4 covers more of what they can do.

## 2.8 Common mistakes

### Using a variable before assigning it

```python norun
print(total)
total = 100
```

```text
NameError: name 'total' is not defined
```

The program runs top to bottom; on line 1 `total` doesn't exist yet. **Assign first, use later.**

### Inconsistent spelling

```python norun
user_name = "Bob"
print(username)        # missing underscore
```

Also a `NameError`. Python doesn't guess; the name must match exactly.

### Adding a number to a string

```python norun
count = 5
print("Count: " + count)
```

`TypeError`. Use `str(count)`, or simply an f-string: `print(f"Count: {count}")`.

## 2.9 Summary

- `name = value` gives a value to a variable; `=` means **assignment**, not equality.
- Variables can be reassigned; `x += 1` is short for `x = x + 1`.
- Rules: letters/digits/underscores, not starting with a digit, case-sensitive, no keywords; conventionally `snake_case`, constants in UPPER CASE.
- Basic types: `int`, `float`, `str`, `bool`, plus `None` for "nothing"; check with `type()`.
- Convert with `int()`, `float()`, `str()`, `bool()`; `int()` truncates, `round()` rounds.
- `a, b = b, a` swaps; f-strings `f"…{var}…"` put values into text.

## 2.10 Exercises

<!-- exercise: rectangle -->
### Exercise 1: Rectangle area

Create two variables `width = 8` and `height = 5`, store the area in a variable `area`, and print it:

```text
40
```

<details>
<summary>Hint</summary>

`area = width * height`, then `print(area)`.
</details>

<!-- exercise: swap -->
### Exercise 2: Swap

In the starter code `a` is `1` and `b` is `2`. Don't change the first two lines; add one line that swaps them so the program prints:

```text
2 1
```

<details>
<summary>Hint</summary>

Python's multiple assignment: `a, b = b, a`.
</details>

<!-- exercise: valid-name -->
### Exercise 3: A valid name

Which of these is a **valid** Python variable name?

- `3d_model`
- `my-score`
- `max_speed`
- `for`

<details>
<summary>Answer and explanation</summary>

`max_speed`. `3d_model` starts with a digit, `my-score` contains a minus sign and `for` is a keyword.
</details>

<!-- exercise: predict-vars -->
### Exercise 4: Predict the output

```python
x = 10
y = x
x = x + 5
y *= 2
print(x, y)
print("x" + "y")
print(int("7") + int(2.9))
```

<details>
<summary>Answer and explanation</summary>

`y = x` gives `y` the value `10`; then `x` becomes `15` and `y` becomes `20`. `"x" + "y"` joins two strings: `xy`. `int("7")` is `7`, `int(2.9)` drops the fraction: `2`, total `9`.
</details>

<!-- exercise: fstring-card -->
### Exercise 5: Business card

You have `name = "Alice"`, `age = 20` and `city = "Paris"`. Use an **f-string** to print:

```text
Name: Alice, Age: 20, City: Paris
```

<details>
<summary>Hint</summary>

`print(f"Name: {name}, Age: {age}, City: {city}")`
</details>

<!-- exercise: types -->
### Exercise 6: Types

What is `type("3.14")`?

- `<class 'float'>`
- `<class 'int'>`
- `<class 'str'>`
- `<class 'bool'>`

<details>
<summary>Answer and explanation</summary>

It has quotes, so it's a string, `str`. Text that looks like a number is still text.
</details>

## 2.11 Further reading

- [Python tutorial · 3.1 Using Python as a calculator](https://docs.python.org/3/tutorial/introduction.html)
- [PEP 8 naming conventions](https://peps.python.org/pep-0008/#naming-conventions): Python's official style guide.
- Next lesson: [Numbers and arithmetic](../03-numbers-math/en.md)
