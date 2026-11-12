# 5. Input and interaction

> 🎯 **Goals**: read user input with `input()`; understand that input is **always a string** and convert it with `int()` and `float()`; read several values from one line; write your first truly interactive programs; learn how input is provided when exercises are checked.
>
> ⏱ **Time**: 35 minutes　|　📋 **You need**: lessons 2–4

## 5.1 input(): letting your program listen

So far all the data has been written into the program. To **interact** with a user we need input. `input()` **pauses the program**, waits for the user to type a line and press Enter, then hands that line to the program:

```python
name = input("What's your name? ")
print(f"Hello, {name}!")
```

The program shows `What's your name?` and waits. In the learning app, **type into the input box below the output** (e.g. `Alice`) and press Enter; the program continues:

```text
What's your name? Alice
Hello, Alice!
```

The text inside the parentheses is the **prompt**, shown first to tell the user what to type. You can leave it out: `name = input()`.

> [!TIP]
> If you know what input the program will need, you can type it into the input box and press Enter **before** clicking "▶ Run"; the program receives it as soon as it starts.

## 5.2 Input is always a string

This is **very important** and the most common beginner trap:

```python norun
age = input("How old are you? ")
print(age + 1)
```

Type `18` and you get:

```text
TypeError: can only concatenate str (not "int") to str
```

Why? Because `input()` **always returns a string** — even if the user types digits, you get the text `"18"`, not the number `18`. To calculate, **convert** it first:

```python
age = int(input("How old are you? "))
print(f"Next year you'll be {age + 1}")
```

`int(input(...))` is a **nested call**: the inner `input()` runs first and gives text, which the outer `int()` turns into an integer.

| Type you need | Write |
|---------------|-------|
| text | `s = input()` |
| integer | `n = int(input())` |
| decimal | `x = float(input())` |

## 5.3 A simple calculator

```python
a = float(input("First number: "))
b = float(input("Second number: "))
print(f"{a} + {b} = {a + b}")
print(f"{a} - {b} = {a - b}")
print(f"{a} × {b} = {a * b}")
print(f"{a} ÷ {b} = {a / b:.4f}")
```

Run it and enter `7` and `2`. We use `float` instead of `int` so decimals work too. (If the second number is 0 the last line raises `ZeroDivisionError` — after the next lesson you'll be able to handle that with `if`.)

## 5.4 Several values on one line

Sometimes the user types several numbers on one line separated by spaces, like `3 4 5`. Split the line and convert each part:

```python input="3 4 5"
line = input("Three sides of a triangle (separated by spaces): ")
parts = line.split()          # "3 4 5" → ['3', '4', '5']
a = int(parts[0])
b = int(parts[1])
c = int(parts[2])
print("Perimeter:", a + b + c)
```

A shorter way uses `map`, which applies a function to every element:

```python input="3 4 5"
a, b, c = map(int, input().split())
print(a + b + c)
```

`map(int, ...)` calls `int()` on every piece produced by `split()`, and multiple assignment hands them to `a`, `b` and `c`. This is one of **the most common** ways to read input in programming exercises — remember it!

## 5.5 Cleaning up input

Users often type extra spaces by accident. `strip()` removes whitespace at the ends and `lower()` unifies the case, making your program more forgiving:

```python
answer = input("Do you like Python? (yes/no) ").strip().lower()
print(f"Your answer: {answer}")
```

Typing `  YES ` still gives `yes`.

> [!NOTE]
> `int()` itself ignores surrounding whitespace: `int(" 42 ")` is `42`. But `int("4 2")` (a space inside) and `int("42.0")` are errors.

## 5.6 Input in coding exercises

For coding exercises, the checker **provides the input automatically** and compares your output with the expected output **exactly**. Therefore:

> [!IMPORTANT]
> In coding exercises, **don't put a prompt inside `input()`**. The prompt is printed too, mixes into your output and makes the answer "wrong". Write `n = int(input())`, not `n = int(input("Enter: "))`.

Exercises describe input and output like this:

```text
Input: two lines, one integer each
Output: their sum

Sample input:     Sample output:
3                 8
5
```

**Each line** of the sample input corresponds to one `input()` call.

## 5.7 A complete example: temperature conversion

```python
c = float(input())          # degrees Celsius
f = c * 9 / 5 + 32
print(f"{c:.1f}°C = {f:.1f}°F")
```

Input `36.6` gives `36.6°C = 97.9°F`.

## 5.8 Common mistakes

### Forgetting to convert

```python norun
a = input()
b = input()
print(a + b)     # input 3 and 5 gives 35!
```

No error, but the result is wrong: adding two **strings** joins them. Remember `int(input())`.

### Input that can't be converted

```python norun
n = int(input())     # the user typed "abc" or "3.5"
```

```text
ValueError: invalid literal for int() with base 10: 'abc'
```

Lesson 12 shows how to handle this gracefully with `try/except`.

### Reading more lines than there are

```text
EOFError: EOF when reading a line
```

EOF means "end of file" — the input ran out. The program tried to read another line but there was none. Check that the number of `input()` calls matches the number of input lines, or whether several values on one line should be read with a single `split()`.

## 5.9 Summary

- `input("prompt")` pauses and reads one line; in the app, type into the input box.
- `input()` **always returns a string**; convert with `int()` / `float()` before calculating.
- Several values on a line: `a, b = map(int, input().split())`.
- `strip()` and `lower()` make input handling more forgiving.
- In coding exercises, **no prompt** in `input()`; each sample input line is one `input()` call.

## 5.10 Exercises

<!-- exercise: greet -->
### Exercise 1: Greeting

Read a name and print `Hello, name!`.

```text
Sample input: Alice
Sample output: Hello, Alice!
```

<details>
<summary>Hint</summary>

`name = input()`, then `print(f"Hello, {name}!")`. Remember: no prompt inside `input()`.
</details>

<!-- exercise: sum-two -->
### Exercise 2: Sum of two numbers

Read two lines, one integer each, and print their sum.

```text
Sample input:
3
5
Sample output:
8
```

<details>
<summary>Hint</summary>

`a = int(input())`, `b = int(input())`, `print(a + b)`.
</details>

<!-- exercise: rectangle-line -->
### Exercise 3: Two numbers on one line

Read **one line** with the width and height of a rectangle (integers separated by a space) and print the area and the perimeter, separated by a space.

```text
Sample input: 4 7
Sample output: 28 22
```

<details>
<summary>Hint</summary>

`w, h = map(int, input().split())`
</details>

<!-- exercise: celsius -->
### Exercise 4: Temperature conversion

Read a temperature in Celsius (possibly a decimal) and print it in Fahrenheit with **one decimal place**. Formula: `F = C × 9 / 5 + 32`.

```text
Sample input: 36.6
Sample output: 97.9
```

<details>
<summary>Hint</summary>

`c = float(input())`, and print with `print(f"{f:.1f}")`.
</details>

<!-- exercise: input-type -->
### Exercise 5: The type of input

The user types `42`. What is the type of `x` in `x = input()`?

- `int`
- `float`
- `str`
- it depends on what the user typed

<details>
<summary>Answer and explanation</summary>

`str`. `input()` always returns a string, even when it looks like a number. Convert with `int(x)` when you need a number.
</details>

## 5.11 Further reading

- [Python docs · input()](https://docs.python.org/3/library/functions.html#input)
- [Python docs · map()](https://docs.python.org/3/library/functions.html#map)
- Next lesson: [Conditions](../06-conditions/en.md)
