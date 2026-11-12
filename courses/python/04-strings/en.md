# 4. Strings

> 🎯 **Goals**: pick out any part of a string with indexes and slices; master `len`, `in` and a dozen of the most useful string methods; understand that strings can't be changed in place; format text fluently with f-strings.
>
> ⏱ **Time**: 45 minutes　|　📋 **You need**: lessons 2 and 3

Working with text is one of the most common programming tasks: user names, analysing articles, generating reports, cleaning data… This lesson covers strings (`str`) thoroughly.

## 4.1 A string is a sequence of characters

A string is **characters in order**. Each character has a position number, its **index**, **starting at 0**:

```text
 string:   P   y   t   h   o   n
 index:    0   1   2   3   4   5
 negative:-6  -5  -4  -3  -2  -1
```

Square brackets `[index]` pick one character:

```python
word = "Python"
print(word[0])     # P: the first character
print(word[3])     # h
print(word[-1])    # n: negative indexes count from the end; -1 is the last
print(word[-2])    # o
print(len(word))   # 6: the length (number of characters)
```

> [!IMPORTANT]
> Indexes **start at 0**, so in a string of length 6 the last character has index 5 (= length − 1). Every mainstream language does this. It trips everyone up at first; practice makes it natural.

An index that doesn't exist is an error:

```python norun
word = "Python"
print(word[6])
```

```text
IndexError: string index out of range
```

## 4.2 Slices: taking a piece

`string[start:stop]` takes the part from `start` up to **but not including** `stop`:

```python
word = "Python"
print(word[0:2])    # Py: indexes 0, 1
print(word[2:5])    # tho: indexes 2, 3, 4
print(word[:3])     # Pyt: no start = from the beginning
print(word[3:])     # hon: no stop = to the end
print(word[-3:])    # hon: the last three characters
print(word[:])      # Python: the whole string
```

A good way to remember: **slice positions are the gaps between characters**:

```text
   | P | y | t | h | o | n |
   0   1   2   3   4   5   6
word[2:5] cuts from gap 2 to gap 5 → "tho"
```

### Step

A third number is the **step** (how far to jump each time):

```python
digits = "0123456789"
print(digits[::2])     # 02468: every second character
print(digits[1::2])    # 13579
print(digits[::-1])    # 9876543210: a step of -1 goes backwards!
```

`s[::-1]` is the classic way to reverse a string.

Unlike indexing, **slicing never raises an error** when out of range; you just get what exists: `"abc"[1:100]` is `"bc"`.

## 4.3 Strings can't be changed

Once created, a string's characters **can't be modified**:

```python norun
word = "Python"
word[0] = "J"
```

```text
TypeError: 'str' object does not support item assignment
```

To "change" one, build a new string:

```python
word = "Python"
word = "J" + word[1:]
print(word)     # Jython
```

This property is called **immutability**. Every string method below **returns a new string** and leaves the original alone:

```python
name = "alice"
upper_name = name.upper()
print(name)          # alice: the original is unchanged
print(upper_name)    # ALICE
```

## 4.4 Joining, repeating and containment

```python
first = "Hello"
second = "World"
print(first + ", " + second + "!")   # join: Hello, World!
print("ha" * 3)                       # repeat: hahaha
print("-" * 20)                       # a dividing line
print("ell" in first)                 # True: is it contained?
print("xyz" not in first)             # True
```

## 4.5 Useful string methods

A **method** is a function that belongs to a value, called as `value.method()`. Strings have dozens; here are the most useful.

### Upper and lower case

```python
s = "hello World"
print(s.upper())        # HELLO WORLD
print(s.lower())        # hello world
print(s.title())        # Hello World: capitalise every word
print(s.capitalize())   # Hello world: only the first letter
```

### Removing whitespace

User input often comes with extra spaces or newlines:

```python
s = "   python   \n"
print("[" + s.strip() + "]")    # [python]: remove whitespace at both ends
print("[" + s.lstrip() + "]")   # left only
print("[" + s.rstrip() + "]")   # right only
```

### Finding and replacing

```python
s = "I like cats. Cats are cute."
print(s.find("cats"))           # 7: index of the first occurrence
print(s.find("dogs"))           # -1: not found
print(s.count("ats"))           # 2: how many times
print(s.replace("cats", "dogs"))   # I like dogs. Cats are cute.
print(s.startswith("I"))        # True
print(s.endswith("."))          # True
```

`replace` and `find` are **case-sensitive**: `"Cats"` wasn't replaced.

### Splitting and joining

This pair is used constantly:

```python
sentence = "apple banana cherry"
words = sentence.split()        # split on whitespace
print(words)                    # ['apple', 'banana', 'cherry']

date = "2026-11-12"
parts = date.split("-")         # split on "-"
print(parts)                    # ['2026', '11', '12']

print("/".join(parts))          # 2026/11/12: join with "/"
print(" + ".join(["1", "2", "3"]))   # 1 + 2 + 3
```

`split` returns a **list** (a group of values in square brackets — lesson 8). `join` does the reverse: it glues a group of strings together with a separator.

### Checking the contents

```python
print("2026".isdigit())    # True: only digits
print("abc".isalpha())     # True: only letters
print("abc123".isalnum())  # True: letters or digits
print("   ".isspace())     # True: only whitespace
print("Hello".islower())   # False
```

### Cheat sheet

| Method | Does |
|--------|------|
| `s.upper()` / `s.lower()` | upper / lower case |
| `s.strip()` | remove whitespace at both ends (`strip("x")` removes given characters) |
| `s.replace(a, b)` | replace every a with b |
| `s.find(x)` | index of the first x, or -1 |
| `s.count(x)` | number of occurrences of x |
| `s.split(sep)` | split into a list (whitespace if no sep) |
| `sep.join(list)` | join the strings in the list with sep |
| `s.startswith(x)` / `s.endswith(x)` | starts / ends with x? |
| `s.isdigit()` / `s.isalpha()` | only digits / only letters? |
| `s.center(w)` / `s.ljust(w)` / `s.rjust(w)` | centre / left / right in width w |
| `s.zfill(w)` | pad with zeros on the left to width w |

Methods can be **chained**:

```python
raw = "  Hello, PYTHON!  "
print(raw.strip().lower().replace("!", "?"))   # hello, python?
```

## 4.6 More f-string formatting

You've used f-strings in lessons 2 and 3. A few more useful tricks:

```python
name = "Bob"
score = 87.456
print(f"{name:>10}|")          # right-aligned in width 10
print(f"{name:<10}|")          # left-aligned
print(f"{name:^10}|")          # centred
print(f"{name:*^10}")          # centred, padded with *: ***Bob****
print(f"{score:8.2f}|")        # width 8, two decimals
print(f"{score=}")             # debugging trick: prints score=87.456
```

A tidy table:

```python
print(f"{'Item':<8}{'Qty':>4}{'Price':>8}")
print(f"{'Apple':<8}{3:>4}{4.5:>8.2f}")
print(f"{'Banana':<8}{12:>4}{0.99:>8.2f}")
```

### Escapes and raw strings

Remember the escape sequences from lesson 1: `\n` newline, `\t` tab, `\\` backslash, `\"` quote. If a string contains many backslashes (Windows paths, regular expressions), put an `r` in front for a **raw string**, where backslashes are not escapes:

```python
print("C:\\new\\table")     # a normal string needs double backslashes
print(r"C:\new\table")      # raw string: what you see is what you get
```

## 4.7 Looping over a string (preview)

To do something with **each character**, use a `for` loop (lesson 7 explains it):

```python
for ch in "abc":
    print(ch.upper())
```

This prints `A`, `B`, `C` on three lines — just to make it look familiar.

## 4.8 Summary

- Indexes start at **0**; `s[-1]` is the last character; `len(s)` is the length.
- Slices `s[start:stop:step]` include start and exclude stop; `s[::-1]` reverses.
- Strings are **immutable**; every method returns a new string.
- `+` joins, `*` repeats, `in` tests containment.
- Key methods: `upper/lower`, `strip`, `replace`, `find/count`, `split/join`, `startswith/endswith`, `isdigit/isalpha`.
- f-strings: `{x:>10}` alignment, `{x:.2f}` decimals, `{x=}` debugging; `r"..."` is a raw string.

## 4.9 Exercises

<!-- exercise: reverse -->
### Exercise 1: Reverse a string

`s = "stressed"`. Print it backwards:

```text
desserts
```

<details>
<summary>Hint</summary>

A slice with step `-1`: `s[::-1]`.
</details>

<!-- exercise: slice-predict -->
### Exercise 2: Predict the output

```python
s = "Hello, World"
print(s[0], s[-1])
print(s[7:12])
print(s[:5].upper())
print(len(s))
```

<details>
<summary>Answer and explanation</summary>

`s[0]` is `H`, `s[-1]` is `d`; indexes 7 to 11 are `World`; the first five characters `Hello` in upper case are `HELLO`; with the comma and the space there are 12 characters.
</details>

<!-- exercise: clean-name -->
### Exercise 3: Clean up a name

`raw = "   aLiCe SMITH  "`. Remove the spaces at both ends and capitalise each word, printing:

```text
Alice Smith
```

<details>
<summary>Hint</summary>

`raw.strip().title()`: remove whitespace, then fix the case.
</details>

<!-- exercise: date-format -->
### Exercise 4: Convert a date

`date = "2026-11-12"` (year-month-day). Print it as day/month/year:

```text
12/11/2026
```

<details>
<summary>Hint 1</summary>

`parts = date.split("-")` gives `['2026', '11', '12']`; `parts[0]` is the year and `parts[2]` the day.
</details>

<details>
<summary>Hint 2</summary>

`print(parts[2] + "/" + parts[1] + "/" + parts[0])`, or `"/".join([parts[2], parts[1], parts[0]])`.
</details>

<!-- exercise: immutable -->
### Exercise 5: Immutability

`s = "cat"`. After running `s.upper()` and then `print(s)`, what is printed?

- `CAT`
- `cat`
- an error
- `Cat`

<details>
<summary>Answer and explanation</summary>

`cat`. `upper()` returned a new string `"CAT"`, but we didn't keep it; `s` itself is unchanged. To change `s`, write `s = s.upper()`.
</details>

<!-- exercise: count-vowels -->
### Exercise 6: Count vowels

`text = "programming is fun"`. Count how many times the letters `a`, `i`, `o` and `u` appear and print the **total**:

```text
5
```

<details>
<summary>Hint</summary>

`text.count("a") + text.count("i") + ...`
</details>

## 4.10 Further reading

- [Python docs · String methods](https://docs.python.org/3/library/stdtypes.html#string-methods): the complete list.
- [Python tutorial · 3.1.2 Text](https://docs.python.org/3/tutorial/introduction.html#text)
- Next lesson: [Input and interaction](../05-input/en.md)
