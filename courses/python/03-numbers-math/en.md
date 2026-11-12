# 3. Numbers and arithmetic

> 🎯 **Goals**: master all of Python's arithmetic operators (`+ - * / // % **`) and their precedence; use `%` and `//` for conversion problems; use `round`, `abs`, `max`, `min` and the `math` module; display numbers nicely with f-strings.
>
> ⏱ **Time**: 40 minutes　|　📋 **You need**: lesson 2

## 3.1 Arithmetic operators

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `+` | add | `7 + 2` | `9` |
| `-` | subtract | `7 - 2` | `5` |
| `*` | multiply | `7 * 2` | `14` |
| `/` | divide | `7 / 2` | `3.5` |
| `//` | floor division (round down) | `7 // 2` | `3` |
| `%` | remainder (modulo) | `7 % 2` | `1` |
| `**` | power | `7 ** 2` | `49` |

```python
print(7 + 2, 7 - 2, 7 * 2)
print(7 / 2, 7 // 2, 7 % 2)
print(2 ** 10)
```

### `/` always gives a float

```python
print(6 / 2)     # 3.0, not 3
```

Even when the division is exact, `/` returns a `float`. Use `//` for a whole-number result.

### Mixing integers and floats

If either number is a `float`, the result is a `float`:

```python
print(3 + 2.0)    # 5.0
print(2 * 1.5)    # 3.0
```

## 3.2 Floor division `//` and remainder `%`

Remember school division: 7 ÷ 2 = 3 remainder 1. In Python the **quotient** is `//` and the **remainder** is `%`:

```text
7 // 2 = 3    (quotient)
7 %  2 = 1    (remainder)
check: 3 × 2 + 1 = 7
```

These two are extremely useful:

```python
n = 17
print(n % 2)        # 1 → odd (even numbers leave 0)
print(n % 10)       # 7 → the last digit
print(n // 10)      # 1 → everything except the last digit
```

### Classic use: converting time

Convert 3725 seconds into hours, minutes and seconds:

```python
total = 3725
hours = total // 3600            # an hour is 3600 seconds
minutes = total % 3600 // 60     # the leftover seconds, as minutes
seconds = total % 60
print(hours, "h", minutes, "min", seconds, "s")
```

Output: `1 h 2 min 5 s`.

`divmod(a, b)` gives the quotient and the remainder at once:

```python
q, r = divmod(3725, 3600)
print(q, r)      # 1 125
```

> [!NOTE]
> With negative numbers, `//` rounds **down** (towards the smaller number): `-7 // 2` is `-4`, not `-3`, and so `-7 % 2` is `1`. You rarely meet this day to day. C and Java treat negatives differently; we'll point it out there.

## 3.3 Operator precedence

As in maths, **multiplication and division come before addition and subtraction**, and **parentheses come first**:

```python
print(2 + 3 * 4)      # 14: 3 * 4 first
print((2 + 3) * 4)    # 20: the parentheses first
```

The full order (highest first):

```text
1. ( )                parentheses
2. **                 power
3. -x                 negation
4. *  /  //  %        multiply, divide, floor divide, remainder
5. +  -               add, subtract
```

Operators of the same level go left to right: `10 - 3 - 2` is `(10 - 3) - 2 = 5`. The one exception is `**`, which goes right to left: `2 ** 3 ** 2` is `2 ** 9 = 512`.

> [!TIP]
> When in doubt, **add parentheses**. Extra parentheses never hurt and make code clearer: `average = (a + b + c) / 3`.

A common trap:

```python
print(-2 ** 2)     # -4: 2 ** 2 first, then the minus
print((-2) ** 2)   # 4
```

## 3.4 Handy maths functions

**Built-in functions** you can use without importing anything:

```python
print(abs(-5))             # 5: absolute value
print(max(3, 8, 1))        # 8: largest
print(min(3, 8, 1))        # 1: smallest
print(round(3.14159, 2))   # 3.14: two decimal places
print(round(7.6))          # 8: round to a whole number
print(pow(2, 5))           # 32: same as 2 ** 5
```

### A small surprise in `round`

```python
print(round(2.5))   # 2
print(round(3.5))   # 4
```

When a value is exactly halfway (.5), Python's `round` goes to the **nearest even number** ("banker's rounding"), which keeps errors smaller when summing lots of data. If you need classic "round half up", the `decimal` module (later) gives full control.

## 3.5 The math module

More maths lives in the **`math` module**. A **module** is a toolbox of extra functions; `import` brings it in:

```python
import math

print(math.sqrt(16))       # 4.0: square root
print(math.pi)             # 3.141592653589793
print(math.floor(3.7))     # 3: round down
print(math.ceil(3.2))      # 4: round up
print(math.gcd(12, 18))    # 6: greatest common divisor
print(math.factorial(5))   # 120: 5×4×3×2×1
print(math.hypot(3, 4))    # 5.0: hypotenuse of a right triangle
```

Put `import math` at the top of the file, and write `math.` before each function name.

Example: area and circumference of a circle:

```python
import math

r = 5
area = math.pi * r ** 2
circumference = 2 * math.pi * r
print("Area:", area)
print("Circumference:", circumference)
```

## 3.6 Displaying numbers nicely

Those results have too many decimals. f-strings take a **format specification** after a colon:

```python
import math

r = 5
area = math.pi * r ** 2
print(f"Area: {area:.2f}")              # two decimals: 78.54
print(f"Population: {8100000000:,}")    # thousands separators: 8,100,000,000
print(f"Share: {0.256:.1%}")            # percentage: 25.6%
print(f"ID: {7:03d}")                   # pad with zeros to width 3: 007
print(f"[{42:>6}]")                     # right-aligned, width 6: [    42]
print(f"[{42:<6}]")                     # left-aligned: [42    ]
```

| Format | Meaning | Example |
|--------|---------|---------|
| `:.2f` | 2 decimal places | `3.14159` → `3.14` |
| `:,` | thousands separator | `1234567` → `1,234,567` |
| `:.1%` | percentage, 1 decimal | `0.256` → `25.6%` |
| `:03d` | integer, width 3, zero-padded | `7` → `007` |
| `:>6` / `:<6` / `:^6` | width 6, right/left/centre | |

> [!TIP]
> `:.2f` only changes how the number is **shown**; the variable keeps its full value. To really change the value, use `round()`.

## 3.7 A complete example: a BMI calculator

```python
weight = 68      # kilograms
height = 1.75    # metres

bmi = weight / height ** 2
print(f"Your BMI is {bmi:.1f}")
```

Output: `Your BMI is 22.2`. `height ** 2` happens first because power beats division.

## 3.8 Common mistakes

```python norun
print(10 / 0)
```

```text
ZeroDivisionError: division by zero
```

You can't divide by 0 (with `//` and `%` either). Once you know `if`, check the divisor first.

```python norun
print(math.sqrt(9))
```

```text
NameError: name 'math' is not defined
```

`import math` is missing.

```python norun
x = 5
print(2x)
```

```text
SyntaxError: invalid decimal literal
```

In maths `2x` means 2 times x, but in Python the multiplication sign **must be written**: `2 * x`.

## 3.9 Summary

- Operators: `+ - * /`, `//` floor division, `%` remainder, `**` power; `/` always returns a `float`.
- `//` and `%` are used for odd/even tests, extracting digits and converting units; `divmod` gives both.
- Precedence: parentheses > `**` > negation > `* / // %` > `+ -`; when unsure, use parentheses.
- Built-ins `abs`, `max`, `min`, `round`, `pow`; `round` sends .5 to the nearest even number.
- After `import math`: `math.sqrt`, `math.pi`, `math.floor`, `math.ceil` and more.
- f-string formats: `:.2f`, `:,`, `:.1%`, `:03d`, `:>6`.

## 3.10 Exercises

<!-- exercise: time-convert -->
### Exercise 1: Convert seconds

`total = 7384` is a number of seconds. Work out the hours, minutes and seconds and print the three numbers with **one** `print`, separated by spaces:

```text
2 3 4
```

<details>
<summary>Hint 1</summary>

Hours: `total // 3600`; seconds: `total % 60`.
</details>

<details>
<summary>Hint 2</summary>

Minutes: `total % 3600` gives the seconds left over after whole hours; then `// 60`.
</details>

<!-- exercise: circle -->
### Exercise 2: Circle area

The radius is `r = 3`. Use `math.pi` to calculate the area and print it with two decimals:

```text
28.27
```

<details>
<summary>Hint</summary>

`import math`; the area is `math.pi * r ** 2`; print with `print(f"{area:.2f}")`.
</details>

<!-- exercise: digits -->
### Exercise 3: Split a number

`n = 472`. Use `//` and `%` to get the hundreds, tens and units digits and print:

```text
4 7 2
```

<details>
<summary>Hint</summary>

Units: `n % 10`; tens: `n // 10 % 10`; hundreds: `n // 100`.
</details>

<!-- exercise: predict-ops -->
### Exercise 4: Predict the output

```python
print(17 // 5)
print(17 % 5)
print(2 + 3 * 2 ** 2)
print(10 / 4)
```

<details>
<summary>Answer and explanation</summary>

`17 // 5 = 3` and `17 % 5 = 2`; `2 ** 2 = 4`, `3 * 4 = 12`, `2 + 12 = 14`; `10 / 4 = 2.5`.
</details>

<!-- exercise: precedence -->
### Exercise 5: The average

You want the average of `a`, `b` and `c`. Which expression is correct?

- `a + b + c / 3`
- `(a + b + c) / 3`
- `a + b + (c / 3)`
- `(a + b) + c / 3`

<details>
<summary>Answer and explanation</summary>

Division binds tighter than addition, so the sum must be in parentheses first. Only `(a + b + c) / 3` is right.
</details>

## 3.11 Further reading

- [Python docs · the math module](https://docs.python.org/3/library/math.html)
- [Format specification mini-language](https://docs.python.org/3/library/string.html#format-specification-mini-language): everything you can write after the colon in an f-string.
- Next lesson: [Strings](../04-strings/en.md)
