# 9. Dictionaries and sets

> 🎯 **Goals**: use **dictionaries** to look data up by "name" instead of by "position"; add, remove, change, find and loop over dictionary entries; use a dictionary to **count** things (word frequencies); understand why lists can't be dictionary keys; use **sets** to remove duplicates and find intersections and unions; see why lookups in dictionaries and sets are much faster than in lists.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lessons 7 and 8

## 9.1 Why do we need dictionaries?

Say you're building a phone book. With a list, finding Amy's number means comparing names one by one from start to end. But real phone books, dictionaries and glossaries are **looked up by name directly**.

A Python **dictionary** (dict) is exactly that: it stores many "**key** → **value**" pairs, and you can get a value straight from its key.

```python
phone = {"Tom": "555-0101", "Amy": "555-0199", "Bob": "555-0123"}
print(phone["Amy"])         # look up by key: 555-0199
print(len(phone))           # 3 key-value pairs
print(phone)
```

Dictionaries use **curly braces** `{ }`; each pair is written `key: value`, and pairs are separated by commas.

## 9.2 Getting values: `[]` and `get`

```python
ages = {"Tom": 12, "Amy": 13}
print(ages["Tom"])                  # 12
print(ages.get("Amy"))              # 13
print(ages.get("Bob"))              # None: a missing key gives None, no error
print(ages.get("Bob", 0))           # 0: you can also give a default value
print("Bob" in ages)                # False: in checks the "keys"
```

Using `dict[key]` with a key that **doesn't exist** is an error:

```python norun
ages = {"Tom": 12}
print(ages["Bob"])
```

```text
KeyError: 'Bob'
```

So when you're not sure a key exists, check with `in` first, or use `get`.

## 9.3 Adding, changing, deleting

Dictionaries are **mutable**:

```python
stock = {"apple": 5, "banana": 3}
stock["orange"] = 10        # new key: adds a pair
stock["apple"] = 8          # existing key: changes its value
stock["banana"] += 2        # add 2 to the old value
print(stock)

del stock["orange"]         # delete a pair
n = stock.pop("banana")     # delete it and return its value
print(n, stock)
```

> [!NOTE]
> **Keys never repeat** in a dictionary: assigning to an existing key just overwrites its value. Values can repeat — two people can be the same age.

## 9.4 Looping over a dictionary

```python
scores = {"English": 90, "Maths": 85, "Science": 92}

for subject in scores:                      # looping directly gives the keys
    print(subject)

for subject, score in scores.items():       # .items(): keys and values together (most common)
    print(f"{subject}: {score} points")

print(list(scores.keys()))                  # all the keys
print(list(scores.values()))                # all the values
print("total:", sum(scores.values()))
```

A dictionary **remembers insertion order** (since Python 3.7): entries come out in the order they were added.

## 9.5 Counting with a dictionary

"Count how many times each thing appears" is one of the most common tasks in programming, and dictionaries are made for it:

```python
text = "the cat and the dog and the bird"
counts = {}
for word in text.split():
    counts[word] = counts.get(word, 0) + 1     # start from 0 if unseen, then add 1
print(counts)

# sort by count, most frequent first
for word, n in sorted(counts.items(), key=lambda kv: kv[1], reverse=True):
    print(word, n)
```

`counts.get(word, 0) + 1` is the classic counting idiom: the first time a word appears, `get` returns 0. `sorted(..., key=lambda kv: kv[1])` sorts by the **second item** of each pair (the count); `lambda` is a tiny throwaway function, explained properly in the next lesson on functions.

Python's standard library also has a dedicated counting tool, `Counter`:

```python
from collections import Counter

counts = Counter("the cat and the dog and the bird".split())
print(counts.most_common(2))      # the 2 most common: [('the', 3), ('and', 2)]
print(Counter("hello"))           # it can count letters too
```

## 9.6 Dictionary comprehensions

Like list comprehensions (lesson 8):

```python
squares = {n: n * n for n in range(1, 6)}
print(squares)                            # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

prices = {"apple": 5, "melon": 20, "grapes": 12}
cheap = {k: v for k, v in prices.items() if v < 15}
print(cheap)                              # keep only the cheap ones
```

## 9.7 Putting them together: dicts in lists, lists in dicts

Real data is often nested. For example, information about a class of students:

```python
students = [
    {"name": "Tom", "age": 12, "hobbies": ["football", "drawing"]},
    {"name": "Amy", "age": 13, "hobbies": ["piano"]},
]
for s in students:
    print(f"{s['name']} ({s['age']}) likes: {', '.join(s['hobbies'])}")

students[1]["hobbies"].append("swimming")     # get the dict, then the list inside it, then change it
print(students[1])
```

This "lists + dictionaries" structure is almost identical to **JSON**, the most common data format on the web; lesson 11 shows how to save it to a file.

## 9.8 What can be a key?

Keys must be **unchangeable** (hashable) values: numbers, strings and tuples are fine; changeable values like lists and dictionaries are not.

```python
distance = {("London", "Paris"): 344, ("London", "Berlin"): 932}   # a tuple key: "a pair of cities"
print(distance[("London", "Paris")])
```

```python norun
bad = {["London", "Paris"]: 344}
```

```text
TypeError: unhashable type: 'list'
```

> [!NOTE]
> Why? A dictionary decides where to store a key using its "hash" (a number computed from its contents). If a key's contents could change, its hash would change too, and it could never be found again.

## 9.9 Sets: a bag of unique things

A **set** also uses curly braces, but has values only, no keys. Two properties: **no duplicates** and **no order**.

```python
tags = {"python", "tutorial", "python", "coding"}
print(tags)                       # the duplicate "python" is kept only once (order may differ from what you wrote)
print(len(tags))                  # 3

nums = [3, 1, 3, 2, 1, 3]
print(set(nums))                  # remove duplicates: {1, 2, 3}
print(sorted(set(nums)))          # remove duplicates and sort: [1, 2, 3]

tags.add("beginner")
tags.discard("coding")            # remove (no error if it isn't there)
print("tutorial" in tags)         # True
```

> [!WARNING]
> An empty set is written `set()`. `{}` is an **empty dictionary**!

The most useful thing about sets is **set operations**:

```python
a = {"Tom", "Amy", "Bob"}         # entered the maths contest
b = {"Amy", "Bob", "Lily"}        # entered the English contest
print(a & b)        # intersection: entered both
print(a | b)        # union: entered at least one
print(a - b)        # difference: maths only
print(a ^ b)        # symmetric difference: entered exactly one
```

## 9.10 Why are dictionaries and sets so fast to search?

To evaluate `x in lst`, Python compares elements one by one from the start; with a million elements that can take a million comparisons. Dictionaries and sets are stored as **hash tables**: the position is computed straight from the hash, so a lookup takes about **one step** however many elements there are:

```python
import time

n = 200_000
lst = list(range(n))
st = set(lst)

start = time.perf_counter()
for _ in range(200):
    (n - 1) in lst               # the number we want is at the very end
t_list = time.perf_counter() - start

start = time.perf_counter()
for _ in range(200):
    (n - 1) in st
t_set = time.perf_counter() - start

print(f"list: {t_list:.4f} s   set: {t_set:.6f} s")
```

So **when you need to ask "is it in there?" many times, use a set or a dictionary, not a list**.

## 9.11 Common mistakes

```python norun
counts = {}
counts["apple"] += 1
```

```text
KeyError: 'apple'
```

The key doesn't exist yet, so there's no old value to add 1 to. Use `counts["apple"] = counts.get("apple", 0) + 1`.

```python norun
d = {"a": 1, "b": 2, "c": 3}
for k in d:
    if d[k] < 2:
        del d[k]
```

```text
RuntimeError: dictionary changed size during iteration
```

You can't delete entries while looping over the dictionary. Loop over a copy instead: `for k in list(d):`.

```python norun
s = {}
s.add(1)
```

```text
AttributeError: 'dict' object has no attribute 'add'
```

`{}` is an empty dictionary; an empty set is `set()`.

## 9.12 Summary

- A dictionary `{key: value}` looks values up by key; `d[k]` raises `KeyError` for a missing key, `d.get(k, default)` doesn't.
- Adding and changing are both `d[k] = v`; delete with `del d[k]` or `d.pop(k)`; `in` checks keys.
- Loop with `for k, v in d.items()`; dictionaries keep insertion order.
- Counting: `d[x] = d.get(x, 0) + 1`, or `collections.Counter`; sort with `sorted(d.items(), key=...)`.
- Keys must be unchangeable (numbers, strings, tuples); lists can't be keys.
- A set `{a, b}` has unique elements and no order; `set()` is the empty set; `& | - ^` are intersection, union, difference and symmetric difference.
- Lookups in dictionaries and sets take about one step — much faster than in lists.

## 9.13 Exercises

<!-- exercise: word-count -->
### Exercise 1: Word frequencies

Read a line of English words (separated by spaces). In the order each word **first appears**, print one word per line with the number of times it occurs, separated by a space.

```text
Sample input: the cat and the dog
Sample output:
the 2
cat 1
and 1
dog 1
```

<details>
<summary>Hint</summary>

Count with `counts[w] = counts.get(w, 0) + 1`; dictionaries loop in insertion order.
</details>

<!-- exercise: top-word -->
### Exercise 2: The most common word

Read a line of words and print the word that occurs most often and its count, separated by a space. If several words share the highest count, print the one that comes **first alphabetically**.

```text
Sample input: b a c a b
Sample output: a 2
```

<details>
<summary>Hint</summary>

Count first, then use `min(counts, key=lambda w: (-counts[w], w))`: compare counts first (negated so bigger comes first), and the word itself on a tie.
</details>

<!-- exercise: phonebook -->
### Exercise 3: Phone book

The first line is an integer `n`; each of the next `n` lines has a name and a phone number separated by a space. Then comes an integer `m` and `m` lines, each a name to look up. For each lookup print the number, or `not found` if the name isn't in the book.

```text
Sample input:
2
Tom 123
Amy 456
3
Amy
Bob
Tom
Sample output:
456
not found
123
```

<details>
<summary>Hint</summary>

Read the `n` lines into a dictionary — `name, number = input().split()`, `book[name] = number` — and answer each lookup with `book.get(name, "not found")`.
</details>

<!-- exercise: common -->
### Exercise 4: Numbers in common

Read two lines of integers (separated by spaces) and print the numbers that appear **in both lines**, in increasing order, without duplicates, separated by spaces. If there are none, print `none`.

```text
Sample input:
1 2 3 4 4
3 4 5 4
Sample output: 3 4
```

<details>
<summary>Hint</summary>

Turn each line into a set, take the intersection `&`, then `sorted` it.
</details>

<!-- exercise: dict-predict -->
### Exercise 5: Predict the output

```python
d = {"a": 1}
d["b"] = 2
d["a"] += 10
print(d)
print(len(d), "c" in d, d.get("c", 0))
```

<details>
<summary>Answer and explanation</summary>

`d["b"] = 2` adds a new key; `d["a"] += 10` turns 1 into 11. So the first line is `{'a': 11, 'b': 2}`, then `2 False 0`: length 2, no key `"c"`, and `get` returns the default 0.
</details>

<!-- exercise: valid-key -->
### Exercise 6: Which can be a key?

Which of these values can be a dictionary key?

- `[1, 2]`
- `(1, 2)`
- `{1: 2}`
- `{1, 2}`

<details>
<summary>Answer and explanation</summary>

Only the tuple `(1, 2)`. Lists, dictionaries and sets can all be changed, so they can't be keys (you'd get `TypeError: unhashable type`).
</details>

## 9.14 Further reading

- [The Python Tutorial · 5.4 Sets, 5.5 Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#sets)
- [The collections module: Counter, defaultdict and more](https://docs.python.org/3/library/collections.html)
- Next lesson: [Functions](../10-functions/en.md)
