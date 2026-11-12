# 8. Lists and tuples

> 🎯 **Goals**: keep a group of values in a list and add, remove, change, find and sort them; loop over lists with `for` and `enumerate`; get started with list comprehensions; understand the "two names, one list" trap; meet tuples and unpacking.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lessons 4 and 7

## 8.1 Why lists?

To store the scores of a class of 40 students you can't create 40 variables `score1`, `score2`… A **list** keeps many values in **one** variable, in order:

```python
scores = [88, 92, 75, 64, 99]
names = ["Alice", "Bob", "Carol"]
mixed = [1, "two", 3.0, True]      # different types are allowed (usually not a good idea)
empty = []                         # an empty list
print(scores)
print(len(scores))                 # 5: number of elements
```

Lists use **square brackets** `[ ]` with **commas** between the elements.

## 8.2 Indexes and slices: just like strings

Indexes, negative indexes and slices work **exactly** as for strings in lesson 4:

```python
fruits = ["apple", "banana", "orange", "grape", "melon"]
print(fruits[0])       # apple
print(fruits[-1])      # melon
print(fruits[1:3])     # ['banana', 'orange']
print(fruits[::-1])    # reversed
print("orange" in fruits)  # True
```

## 8.3 Lists can be changed

Unlike strings, **lists are mutable**: you can change their elements directly:

```python
scores = [88, 92, 75]
scores[2] = 80         # change the third element to 80
print(scores)          # [88, 92, 80]
```

### Adding elements

```python
nums = [1, 2, 3]
nums.append(4)         # add one at the end: [1, 2, 3, 4]
nums.insert(0, 0)      # insert at index 0: [0, 1, 2, 3, 4]
nums.extend([5, 6])    # add several at the end: [0, 1, 2, 3, 4, 5, 6]
print(nums)
```

### Removing elements

```python
nums = [10, 20, 30, 20, 40]
nums.remove(20)        # remove the first 20: [10, 30, 20, 40]
last = nums.pop()      # remove and return the last: last = 40
first = nums.pop(0)    # remove and return index 0: first = 10
del nums[0]            # delete by index
print(nums, last, first)
nums.clear()           # empty the list
print(nums)            # []
```

### Finding and counting

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
print(nums.index(5))   # 4: index of the first 5
print(nums.count(1))   # 2: how many 1s
print(min(nums), max(nums), sum(nums))   # 1 9 31
print(sum(nums) / len(nums))             # average 3.875
```

### Sorting

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
nums.sort()                  # sort in place (changes nums itself)
print(nums)                  # [1, 1, 2, 3, 4, 5, 6, 9]
nums.sort(reverse=True)      # largest first
print(nums)

data = [3, 1, 2]
new = sorted(data)           # returns a new sorted list; the original is unchanged
print(data, new)             # [3, 1, 2] [1, 2, 3]

words = ["banana", "Apple", "cherry"]
print(sorted(words))                  # ['Apple', 'banana', 'cherry']: capitals sort first
print(sorted(words, key=str.lower))   # ignore case
print(sorted(words, key=len))         # by length
```

> [!IMPORTANT]
> `nums.sort()` **changes the list** and returns `None`; `sorted(nums)` **leaves it alone** and returns a new list. Beginners often write `nums = nums.sort()` and end up with `nums` being `None`!

### Cheat sheet

| Operation | Code |
|-----------|------|
| add at the end | `lst.append(x)` |
| insert | `lst.insert(i, x)` |
| combine | `lst.extend(other)` or `lst + other` |
| remove by value | `lst.remove(x)` |
| remove and return by index | `lst.pop(i)` (the last if no i) |
| index of / count | `lst.index(x)` / `lst.count(x)` |
| sort | `lst.sort()` / `sorted(lst)` |
| reverse | `lst.reverse()` / `lst[::-1]` |
| length, min, max, sum | `len`, `min`, `max`, `sum` |

## 8.4 Looping over a list

```python
names = ["Alice", "Bob", "Carol"]
for name in names:
    print(f"Hello, {name}")
```

When you need **the index and the value**, use `enumerate`:

```python
names = ["Alice", "Bob", "Carol"]
for i, name in enumerate(names, start=1):
    print(f"{i}. {name}")
```

```text
1. Alice
2. Bob
3. Carol
```

To go through two lists side by side, use `zip`:

```python
names = ["Alice", "Bob", "Carol"]
scores = [88, 95, 72]
for name, score in zip(names, scores):
    print(f"{name}: {score} points")
```

## 8.5 Building a list in a loop

The most common pattern: start with an empty list and `append` inside a loop:

```python
n = int(input())
numbers = []
for _ in range(n):
    numbers.append(int(input()))
print("You entered:", numbers)
print("Maximum:", max(numbers))
```

If the n numbers are on **one line**, use `split` + `map` from lesson 5 and turn the result into a list with `list()`:

```python
numbers = list(map(int, input().split()))
print(numbers)
```

### Filtering

```python
scores = [88, 45, 92, 59, 75, 30]
passed = []
for s in scores:
    if s >= 60:
        passed.append(s)
print(passed)        # [88, 92, 75]
```

## 8.6 List comprehensions

"Loop + filter + append" is so common that Python has a compact form for it, the **list comprehension**:

```python
scores = [88, 45, 92, 59, 75, 30]
passed = [s for s in scores if s >= 60]
print(passed)                          # [88, 92, 75]

squares = [x * x for x in range(1, 6)]
print(squares)                         # [1, 4, 9, 16, 25]

words = ["hello", "world"]
print([w.upper() for w in words])      # ['HELLO', 'WORLD']
```

Read it as `[what to compute  for x in where  if condition]` — "for each x (that passes the condition), compute a new value and put it in a new list".

> [!TIP]
> Comprehensions are great for **simple** transformations and filters. For complex logic a normal `for` loop is clearer.

## 8.7 Nested lists: tables and matrices

A list's elements can themselves be lists, representing a two-dimensional table:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]
print(matrix[1])        # [4, 5, 6]: the second row
print(matrix[1][2])     # 6: row 2, column 3

for row in matrix:
    for value in row:
        print(value, end=" ")
    print()
```

## 8.8 The trap: two names for one list

```python
a = [1, 2, 3]
b = a          # b and a refer to the SAME list
b.append(4)
print(a)       # [1, 2, 3, 4] — a changed too!
```

Lesson 2 said variables are labels. `b = a` doesn't copy the list; it sticks a second label on **the same list**. A change through either name is visible through the other:

```text
a ──┐
    ├──► [1, 2, 3, 4]
b ──┘
```

For an independent copy, use `copy()` or a slice:

```python
a = [1, 2, 3]
b = a.copy()       # or b = a[:] or b = list(a)
b.append(4)
print(a, b)        # [1, 2, 3] [1, 2, 3, 4]
```

> [!NOTE]
> Numbers and strings are immutable, so `y = x; y += 1` never affects `x` — `y += 1` makes `y` refer to a new number. Only mutable objects (lists, and the dictionaries and sets of the next lesson) show this "change one, both change" behaviour.

## 8.9 Tuples: lists that can't change

A **tuple** is like a list but uses **round brackets** and **can't be changed** after creation:

```python
point = (3, 4)
print(point[0])        # 3
colors = ("red", "green", "blue")
print(len(colors), "green" in colors)
```

```python norun
point[0] = 5           # TypeError: 'tuple' object does not support item assignment
```

When to use a tuple? When a group of values **shouldn't change**: coordinates `(x, y)`, an RGB colour `(255, 128, 0)`, a date `(2026, 11, 12)`. Immutability is a safety feature: nobody can modify it by accident.

### Packing and unpacking

You've used tuples already:

```python
x, y = 3, 4        # 3, 4 on the right is a tuple (packing); the left side unpacks it
x, y = y, x        # the swap trick from lesson 2 works because of tuples
```

```python
point = (3, 4)
x, y = point
print(x, y)

first, *rest = [1, 2, 3, 4]   # * collects all remaining elements
print(first, rest)            # 1 [2, 3, 4]
```

> [!NOTE]
> A one-element tuple needs a comma: `(5,)`. `(5)` is just the number 5 in parentheses.

## 8.10 Common mistakes

```python norun
nums = [1, 2, 3]
print(nums[3])
```

```text
IndexError: list index out of range
```

A list of length 3 has indexes 0, 1, 2.

```python norun
nums = [3, 1, 2]
nums = nums.sort()
print(nums[0])
```

```text
TypeError: 'NoneType' object is not subscriptable
```

`sort()` returns `None`, which overwrote `nums`. Write just `nums.sort()`, or `nums = sorted(nums)`.

```python norun
nums = [1, 2, 3]
nums.remove(5)
```

```text
ValueError: list.remove(x): x not in list
```

The value isn't in the list. Check with `if 5 in nums:` first.

## 8.11 Summary

- A list `[a, b, c]` keeps values in order; indexes, slices, `in` and `len` work as for strings.
- Lists are **mutable**: `append`, `insert`, `extend`, `remove`, `pop`, `del`, `sort`…
- `lst.sort()` sorts in place and returns `None`; `sorted(lst)` returns a new list; `key=` chooses what to sort by.
- Looping: `for x in lst`; with indexes `enumerate`; side by side `zip`.
- List comprehensions `[expression for x in sequence if condition]`.
- `b = a` doesn't copy a list; `b = a.copy()` does.
- Tuples `(a, b)` can't change; packing and unpacking make multiple assignment and swapping possible.

## 8.12 Exercises

<!-- exercise: average -->
### Exercise 1: Average score

The first line contains an integer n, the second line n integers separated by spaces. Print their average with 2 decimal places.

```text
Sample input:
4
90 85 77 60
Sample output:
78.00
```

<details>
<summary>Hint</summary>

`n = int(input())`, `nums = list(map(int, input().split()))`; the average is `sum(nums) / n`.
</details>

<!-- exercise: sort-desc -->
### Exercise 2: Largest first

Read a line of integers separated by spaces, sort them from largest to smallest and print them separated by spaces.

```text
Sample input: 3 1 4 1 5 9 2 6
Sample output: 9 6 5 4 3 2 1 1
```

<details>
<summary>Hint</summary>

Sort with `sort(reverse=True)`; to print, convert each number to a string and use `" ".join(...)`. Or write `print(*nums)` — the `*` spreads the list into separate arguments.
</details>

<!-- exercise: unique -->
### Exercise 3: Remove duplicates

Read a line of words separated by spaces, remove duplicates **keeping the order of first appearance**, and print what's left.

```text
Sample input: apple banana apple cherry banana
Sample output: apple banana cherry
```

<details>
<summary>Hint</summary>

Start with an empty list `result`; for each word, `append` it if `word not in result`.
</details>

<!-- exercise: evens-squared -->
### Exercise 4: A list comprehension

Read a line of integers; with a **list comprehension**, keep the even numbers and print their squares separated by spaces.

```text
Sample input: 1 2 3 4 5 6
Sample output: 4 16 36
```

<details>
<summary>Hint</summary>

`[x * x for x in nums if x % 2 == 0]`
</details>

<!-- exercise: alias-predict -->
### Exercise 5: Predict the output

```python
a = [1, 2, 3]
b = a
c = a.copy()
b.append(4)
c.append(5)
print(a)
print(len(b), len(c))
```

<details>
<summary>Answer and explanation</summary>

`b` is the same list as `a`, so after `b.append(4)` `a` is `[1, 2, 3, 4]`; `c` is a copy and only it gets the 5. So `a` prints as `[1, 2, 3, 4]`, `b` has length 4 and `c` also 4 (`[1, 2, 3, 5]`).
</details>

<!-- exercise: sort-return -->
### Exercise 6: What sort returns

After `x = [3, 1, 2].sort()`, what is `x`?

- `[1, 2, 3]`
- `[3, 1, 2]`
- `None`
- an error

<details>
<summary>Answer and explanation</summary>

`None`. `sort()` sorts in place and returns nothing. For a new sorted list use `sorted([3, 1, 2])`.
</details>

## 8.13 Further reading

- [Python tutorial · 5.1 More on lists](https://docs.python.org/3/tutorial/datastructures.html)
- [Sorting HOW TO](https://docs.python.org/3/howto/sorting.html)
- Next lesson: [Dictionaries and sets](../09-dicts-sets/en.md)
