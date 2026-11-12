# 6. Conditions

> 🎯 **Goals**: write conditions with comparison and logical operators; use `if`, `elif` and `else` to make a program act differently in different situations; understand what **indentation** means in Python; meet conditional expressions and the `match` statement.
>
> ⏱ **Time**: 45 minutes　|　📋 **You need**: lessons 2–5

## 6.1 Programs need to make decisions

Life is full of decisions: **if** it rains, take an umbrella; **if** you score at least 60 you pass, **otherwise** you resit. Programs are the same: they run different code depending on the situation. This is a **condition** (or branch).

```python
temperature = 35
if temperature > 30:
    print("It's hot! Drink plenty of water.")
print("Have a nice day")
```

Output:

```text
It's hot! Drink plenty of water.
Have a nice day
```

Change `temperature` to `20` and run again — the first message disappears.

## 6.2 Comparison operators

A condition evaluates to a **boolean**: `True` or `False`.

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `==` | equal | `3 == 3` | `True` |
| `!=` | not equal | `3 != 4` | `True` |
| `>` | greater than | `5 > 7` | `False` |
| `<` | less than | `5 < 7` | `True` |
| `>=` | greater than or equal | `7 >= 7` | `True` |
| `<=` | less than or equal | `8 <= 7` | `False` |

```python
age = 18
print(age >= 18)        # True
print(age == 20)        # False
print("abc" == "abc")   # True: strings can be compared too
print("Abc" == "abc")   # False: different case
```

> [!WARNING]
> **`=` assigns, `==` compares.** `if age = 18:` is a syntax error. Beginners in every language make this mistake.

Python also allows **chained comparisons**, just like maths:

```python
x = 5
print(0 < x < 10)      # True: x is between 0 and 10
```

## 6.3 The shape of an if statement

```text
if condition:
    code run when the condition is true
    (any number of lines)
code after it (always runs)
```

Two key points:

1. `if condition` **must end with a colon** `:`.
2. The code that belongs to the `if` **must be indented** (always 4 spaces). Indented lines form a **block**; going back to the previous indentation ends the block.

```python
score = 95
if score >= 90:
    print("Excellent!")      # belongs to the if
    print("Keep it up")      # also belongs to the if
print("Score recorded")      # not part of the if; always runs
```

> [!IMPORTANT]
> In C, Java and JavaScript, blocks are marked with braces `{ }`; in Python a block is **defined entirely by indentation**. That's why Python is strict about it: every line in a block needs exactly the same number of leading spaces. The app's editor indents automatically when you press Enter, and <kbd>Tab</kbd> inserts 4 spaces.

## 6.4 else: otherwise

```python
age = int(input())
if age >= 18:
    print("adult")
else:
    print("minor")
```

`else` also ends with a colon; its block runs when the condition is **false**. Exactly **one** of the two branches always runs.

### Odd or even

```python
n = int(input())
if n % 2 == 0:
    print("even")
else:
    print("odd")
```

## 6.5 elif: several conditions

When there are more than two cases, use `elif` (short for "else if"):

```python
score = int(input())
if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 70:
    print("C")
elif score >= 60:
    print("D")
else:
    print("F")
```

Python checks the conditions **from top to bottom**, **runs the first one that is true and skips all the rest**. With input `85`: `85 >= 90` is false → `85 >= 80` is true → prints `B`, and nothing else is checked.

> [!TIP]
> Because only the first true branch runs, `elif score >= 80` doesn't need to be `elif 80 <= score < 90` — if we got there, `score >= 90` was already false. But **order matters**: put `score >= 60` first and a score of 95 would get a D!

## 6.6 Logical operators: and, or, not

Combine several conditions:

| Operator | Meaning | Example |
|----------|---------|---------|
| `and` | both must be true | `age >= 18 and has_ticket` |
| `or` | at least one is true | `day == "Saturday" or day == "Sunday"` |
| `not` | the opposite | `not is_raining` |

```python
age = 20
has_ticket = True
if age >= 18 and has_ticket:
    print("You may enter")

day = "Sunday"
if day == "Saturday" or day == "Sunday":
    print("Have a nice weekend!")

is_raining = False
if not is_raining:
    print("Let's go for a walk")
```

### A classic: leap years

A year is a leap year if it's divisible by 4 **and** not by 100, **or** divisible by 400:

```python
year = int(input())
if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
    print("leap year")
else:
    print("common year")
```

Try 2024 (leap), 1900 (common), 2000 (leap). The parentheses make the logic clear.

> [!WARNING]
> Beginners often write `if day == "Saturday" or "Sunday":` — that's **wrong**! It means `(day == "Saturday") or ("Sunday")`, and a non-empty string like `"Sunday"` always counts as true, so the condition is always true. Write both sides in full: `day == "Saturday" or day == "Sunday"`, or `day in ("Saturday", "Sunday")`.

## 6.7 Nested conditions

An `if` can contain another `if`; each level adds 4 more spaces:

```python
age = int(input())
member = input()      # yes or no
if age < 12:
    print("Child ticket: free")
else:
    if member == "yes":
        print("Member ticket: 5")
    else:
        print("Standard ticket: 8")
```

Deep nesting gets hard to read. Often `elif` or `and` flattens it:

```python
age = int(input())
member = input()
if age < 12:
    print("Child ticket: free")
elif member == "yes":
    print("Member ticket: 5")
else:
    print("Standard ticket: 8")
```

## 6.8 Truthiness: more than True and False

The condition after `if` doesn't have to be a comparison — any value works. Python treats it as `bool(value)` (lesson 2): `0`, `0.0`, `""`, `None` and empty lists `[]` are "falsy"; everything else is "truthy".

```python
name = input()
if name:                 # same as if name != ""
    print(f"Hello, {name}")
else:
    print("You didn't type anything")
```

## 6.9 Conditional expressions: a one-line if

A simple either/or fits on one line:

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)
```

The form is `value1 if condition else value2`. Good for simple cases; use a normal `if` for anything complex.

## 6.10 The match statement

Python 3.10 added `match`, handy when you do different things for different values of one thing:

```python
command = input()
match command:
    case "start":
        print("Game started")
    case "pause" | "stop":          # | means "or"
        print("Paused")
    case "quit":
        print("Bye!")
    case _:                          # _ matches everything else
        print("Unknown command")
```

`match` can do much more (it can take apart lists, dictionaries and other structures — "structural pattern matching"); you'll see it again. For simple decisions `if/elif` is perfectly fine.

## 6.11 Common mistakes

```python norun
if x = 5:
```

```text
SyntaxError: invalid syntax. Maybe you meant '==' or ':=' instead of '='?
```

Compare with `==`. Python's message even guesses what you meant.

```python norun
if x > 5
    print("big")
```

```text
SyntaxError: expected ':'
```

The colon is missing.

```python norun
if x > 5:
print("big")
```

```text
IndentationError: expected an indented block after 'if' statement on line 1
```

The indentation is missing.

```python norun
if x > 5:
    print("big")
     print("very big")
```

```text
IndentationError: unexpected indent
```

Inconsistent indentation within one block (one extra space on the second line).

## 6.12 Summary

- Comparison operators `== != > < >= <=` give `True`/`False`; `=` assigns, `==` compares.
- `if condition:` + an **indented block**; `else:` for the other case; `elif` for more cases — **only the first true branch runs, top to bottom**.
- `and`, `or`, `not` combine conditions; write full conditions on both sides of `or`.
- Any value can be a condition: `0`, `""`, `None` and empty containers are falsy.
- `value1 if condition else value2` is a one-line choice; `match/case` branches on a value.

## 6.13 Exercises

<!-- exercise: odd-even -->
### Exercise 1: Odd or even

Read an integer and print `even` if it's even, otherwise `odd`.

```text
Sample input: 7
Sample output: odd
```

<details>
<summary>Hint</summary>

`n % 2 == 0` means even.
</details>

<!-- exercise: grade -->
### Exercise 2: Grades

Read a score from 0 to 100 and print the grade: 90 and above `A`, 80–89 `B`, 70–79 `C`, 60–69 `D`, below 60 `F`.

```text
Sample input: 85
Sample output: B
```

<details>
<summary>Hint</summary>

Use `if / elif / else`, checking from the highest grade down.
</details>

<!-- exercise: leap -->
### Exercise 3: Leap years

Read a year and print `leap` if it's a leap year, otherwise `common`. Rule: divisible by 4 and not by 100, or divisible by 400.

```text
Sample input: 1900
Sample output: common
```

<details>
<summary>Hint</summary>

`(y % 4 == 0 and y % 100 != 0) or y % 400 == 0`
</details>

<!-- exercise: max3 -->
### Exercise 4: The largest of three

Read three integers on one line and print the largest. **Use `if`**, not `max()` (afterwards you can check with `max()`).

```text
Sample input: 3 9 4
Sample output: 9
```

<details>
<summary>Hint 1</summary>

Assume the first is the largest: `biggest = a`; then compare with `b` and `c` in turn and update `biggest` when you find something larger.
</details>

<details>
<summary>Hint 2</summary>

```python
a, b, c = map(int, input().split())
biggest = a
if b > biggest:
    biggest = b
# now compare c...
```
</details>

<!-- exercise: predict-if -->
### Exercise 5: Predict the output

```python
x = 15
if x > 10:
    print("A")
elif x > 5:
    print("B")
if x % 3 == 0 and x % 5 == 0:
    print("C")
else:
    print("D")
print("E" if x < 10 else "F")
```

<details>
<summary>Answer and explanation</summary>

`x > 10` is true, so `A`, and the `elif` is skipped; the next `if` is a new statement: 15 is divisible by both 3 and 5, so `C`; finally `x < 10` is false, so `F`.
</details>

<!-- exercise: or-trap -->
### Exercise 6: The or trap

With `day = "Monday"`, what does the condition in `if day == "Saturday" or "Sunday":` evaluate to?

- `False`, because Monday isn't the weekend
- `True`, because `"Sunday"` on its own counts as true
- an error
- it depends on the computer's language settings

<details>
<summary>Answer and explanation</summary>

`True`. It's read as `(day == "Saturday") or "Sunday"`, and a non-empty string is always truthy. Write `day == "Saturday" or day == "Sunday"`.
</details>

## 6.14 Further reading

- [Python tutorial · 4.1 if statements](https://docs.python.org/3/tutorial/controlflow.html#if-statements)
- [Python tutorial · 4.7 match statements](https://docs.python.org/3/tutorial/controlflow.html#match-statements)
- Next lesson: [Loops](../07-loops/en.md)
