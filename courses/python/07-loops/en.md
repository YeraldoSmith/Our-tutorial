# 7. Loops

> 🎯 **Goals**: repeat code with `while` and `for`; use `range()` fluently; master the loop patterns "accumulate", "count" and "find the maximum"; control loops with `break` and `continue`; write nested loops (e.g. a multiplication table); avoid endless loops and off-by-one errors.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lesson 6

## 7.1 Why loops?

Suppose you want to print 1 to 5:

```python
print(1)
print(2)
print(3)
print(4)
print(5)
```

What about 1 to 1000? Copying 1000 lines is absurd. A **loop** runs the same code **many times**. Tireless repetition is exactly what computers are best at.

## 7.2 The while loop: keep going while a condition holds

```python
i = 1
while i <= 5:
    print(i)
    i += 1
print("done")
```

Step by step:

```text
i = 1 → 1 <= 5 true  → print 1, i becomes 2
i = 2 → 2 <= 5 true  → print 2, i becomes 3
  ...
i = 5 → 5 <= 5 true  → print 5, i becomes 6
i = 6 → 6 <= 5 false → leave the loop, print "done"
```

`while` looks like `if`: `while condition:` plus an indented block. The difference: `if` runs once, `while` **repeats until the condition becomes false**.

### Beware of endless loops

```python norun
i = 1
while i <= 5:
    print(i)
    # forgot i += 1
```

`i` stays 1, the condition stays true, and the program prints 1 **forever** — an **endless (infinite) loop**. Click "■ Stop" in the app, or press <kbd>Ctrl</kbd>+<kbd>C</kbd> in a terminal.

> [!TIP]
> When writing a `while` loop, ask three questions: ① where does the loop variable start? ② under what condition does it continue? ③ does the loop variable **move towards the end** on every pass?

### while suits "I don't know how many times"

For example, ask until the answer is correct:

```python input="abc\npython"
password = input()
while password != "python":
    print("Wrong password, try again")
    password = input()
print("Logged in!")
```

## 7.3 The for loop: one item at a time

A `for` loop **handles each item of a group in turn**:

```python
for ch in "abc":
    print(ch)
```

This prints three lines: `a`, `b`, `c`. On each pass, the variable `ch` gets the next character.

### range(): a sequence of numbers

`for` is most often used with `range()` to "repeat n times" or "go through some numbers":

```python
for i in range(5):
    print(i)
```

This prints `0 1 2 3 4` (one per line). Note: **`range(5)` is 0 to 4, not including 5** — "start included, stop excluded", like slices.

Three forms of `range`:

| Form | Numbers |
|------|---------|
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(1, 6)` | 1, 2, 3, 4, 5 |
| `range(0, 10, 2)` | 0, 2, 4, 6, 8 (step 2) |
| `range(10, 0, -1)` | 10, 9, 8, …, 1 (counting down) |

```python
for i in range(1, 6):
    print(f"Pass number {i}")
```

> [!NOTE]
> `for i in range(1, 6)` does exactly what the `while` example in 7.2 did, but it's shorter and harder to get wrong (no forgotten `i += 1`). **When you know the number of repetitions, prefer `for`.**

## 7.4 Classic loop patterns

### Pattern 1: accumulate

Compute 1 + 2 + … + 100:

```python
total = 0                  # ① before the loop: a variable for the result, starting at 0
for i in range(1, 101):
    total += i             # ② add the current number each time
print(total)               # ③ after the loop, use the result: 5050
```

### Pattern 2: multiply up

Compute 5 factorial, 5! = 1 × 2 × 3 × 4 × 5:

```python
result = 1                 # for multiplication start at 1, not 0!
for i in range(1, 6):
    result *= i
print(result)              # 120
```

### Pattern 3: count

How many letters `o` are in a sentence?

```python
text = "hello world, good morning"
count = 0
for ch in text:
    if ch == "o":
        count += 1
print(count)               # 5
```

### Pattern 4: find the maximum

Read 5 numbers and find the largest:

```python
biggest = int(input())      # take the first number as the current maximum
for _ in range(4):          # read the remaining 4
    n = int(input())
    if n > biggest:
        biggest = n
print(biggest)
```

When the loop variable isn't needed, it's conventionally called `_`.

### Pattern 5: read n numbers

Many problems give a count n followed by n numbers:

```python
n = int(input())
total = 0
for _ in range(n):
    total += int(input())
print(total)
```

## 7.5 break and continue

- **`break`**: **leave the whole loop** immediately.
- **`continue`**: skip the rest of this pass and **go straight to the next one**.

```python
for i in range(1, 11):
    if i == 6:
        break           # stop at 6
    print(i, end=" ")
print()
# output: 1 2 3 4 5

for i in range(1, 11):
    if i % 2 == 0:
        continue        # skip even numbers
    print(i, end=" ")
print()
# output: 1 3 5 7 9
```

### while True + break: "do it, and quit when..."

```python input="hello\nquit"
while True:
    cmd = input()
    if cmd == "quit":
        break
    print(f"You typed: {cmd}")
print("Program finished")
```

`while True` means "loop forever"; the `break` inside ends it. This is the most common shape for interactive programs.

### Find the first number that fits

```python
n = 100
while True:
    n += 1
    if n % 7 == 0:
        break
print("First multiple of 7 above 100:", n)    # 105
```

## 7.6 Nested loops

A loop inside a loop: for each pass of the outer loop, the inner loop runs completely.

```python
for row in range(1, 4):
    for col in range(1, 5):
        print(f"({row},{col})", end=" ")
    print()          # new line after each row
```

```text
(1,1) (1,2) (1,3) (1,4)
(2,1) (2,2) (2,3) (2,4)
(3,1) (3,2) (3,3) (3,4)
```

### A multiplication table

```python
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f"{j}×{i}={i * j}", end="\t")
    print()
```

The inner range `range(1, i + 1)` depends on the outer `i`: one entry in row 1, two in row 2… so the output is a triangle. `\t` is a tab that lines things up.

### Drawing shapes

```python
n = 4
for i in range(1, n + 1):
    print("*" * i)
```

```text
*
**
***
****
```

String multiplication `"*" * i` draws i stars in one go.

## 7.7 for…else (good to know)

A loop can have an `else` that runs **when the loop finished normally (not interrupted by `break`)**. It's handy for searching:

```python
n = 29
for d in range(2, n):
    if n % d == 0:
        print(n, "is not prime; divisible by", d)
        break
else:
    print(n, "is prime")
```

## 7.8 Common mistakes

### Off-by-one

You want 1 to 10 but write:

```python
for i in range(1, 10):
    print(i)      # stops at 9!
```

`range` excludes the stop value; write `range(1, 11)`. Being "one too few or one too many" is so common in every language that it has a name: the **off-by-one error**. When checking a loop, **look closely at the first and the last pass**.

### The accumulator in the wrong place

```python
for i in range(1, 4):
    total = 0          # ❌ reset on every pass
    total += i
print(total)           # 3, not 6
```

Initialise **before** the loop.

### Endless loops

The variable in a `while` condition never changes inside the loop, or changes in the wrong direction (`i -= 1` instead of `i += 1`). If a program seems "stuck", suspect this first. When checking answers, the app reports "Time limit exceeded".

## 7.9 Summary

- `while condition:` repeats while the condition is true; make sure it eventually becomes false.
- `for variable in group:` handles items one by one; `range(start, stop, step)` excludes stop.
- Known count → `for`; unknown count (until something happens) → `while`.
- Patterns: accumulate (start 0), multiply (start 1), count, maximum, read n numbers.
- `break` leaves the loop, `continue` jumps to the next pass; `while True` + `break` is the usual interactive loop.
- Nested loops: one outer pass = a full inner loop; watch for off-by-one.

## 7.10 Exercises

<!-- exercise: sum-to-n -->
### Exercise 1: Sum from 1 to n

Read a positive integer n and print 1 + 2 + … + n.

```text
Sample input: 100
Sample output: 5050
```

<details>
<summary>Hint</summary>

`total = 0`, then `for i in range(1, n + 1): total += i`.
</details>

<!-- exercise: factorial -->
### Exercise 2: Factorial

Read a non-negative integer n (0 ≤ n ≤ 20) and print n!. By definition 0! = 1.

```text
Sample input: 5
Sample output: 120
```

<details>
<summary>Hint</summary>

`result = 1`, then multiply by 1 to n. For n = 0 the loop doesn't run at all and the result is 1, as required.
</details>

<!-- exercise: fizzbuzz -->
### Exercise 3: FizzBuzz

Read a positive integer n and print 1 to n, one per line — except: multiples of 3 print `Fizz`, multiples of 5 print `Buzz`, and multiples of both print `FizzBuzz`.

```text
Sample input: 5
Sample output:
1
2
Fizz
4
Buzz
```

<details>
<summary>Hint</summary>

Check "multiple of both" (that is, of 15) first, then 3 and 5 — the order matters.
</details>

<!-- exercise: count-digits -->
### Exercise 4: Count the digits

Read a positive integer and, **using a while loop** (no `len`), count how many digits it has.

```text
Sample input: 12345
Sample output: 5
```

<details>
<summary>Hint</summary>

Each `n //= 10` removes the last digit; count until `n` becomes 0.
</details>

<!-- exercise: triangle -->
### Exercise 5: Number triangle

Read a positive integer n and print n lines; line i holds the numbers 1 to i separated by spaces.

```text
Sample input: 4
Sample output:
1
1 2
1 2 3
1 2 3 4
```

<details>
<summary>Hint 1</summary>

Outer loop `for i in range(1, n + 1)`, inner loop prints 1 to i.
</details>

<details>
<summary>Hint 2</summary>

To avoid a trailing space, build a list of strings and use `" ".join(...)`; or print with `end=" "` — the checker ignores trailing spaces, so that works too.
</details>

<!-- exercise: predict-loop -->
### Exercise 6: Predict the output

```python
total = 0
for i in range(10, 0, -3):
    total += i
    print(i, end=" ")
print()
print(total)
```

<details>
<summary>Answer and explanation</summary>

`range(10, 0, -3)` gives 10, 7, 4, 1 (the next, -2, is past the stop value 0). The sum is 10 + 7 + 4 + 1 = 22. The trailing space on the first line doesn't matter to the checker.
</details>

<!-- exercise: range-len -->
### Exercise 7: How many numbers?

How many numbers does `range(3, 15, 3)` produce?

- 3
- 4
- 5
- 12

<details>
<summary>Answer and explanation</summary>

3, 6, 9, 12 — four numbers. 15 is the stop value and is excluded.
</details>

## 7.11 Further reading

- [Python tutorial · 4.2 for statements, 4.3 range()](https://docs.python.org/3/tutorial/controlflow.html#for-statements)
- [Python Tutor](https://pythontutor.com/): visualises your code step by step — great for watching how variables change inside loops.
- Next lesson: [Lists and tuples](../08-lists-tuples/en.md)
