# 5. Reading error messages and debugging

> 🎯 **Goals**: stop fearing red error text; find "which file, which line, what problem" in any message; know the 40 most common words in error messages; debug with print statements and by halving the code; search, ask questions and use AI assistants the right way.
>
> ⏱ **Time**: 40 minutes　|　📋 **You need**: lesson 1

## 5.1 Mindset: errors are your friends

Beginners who see a screen full of red often think "maybe programming isn't for me". In reality:

- **Every programmer gets errors every day.** Writing code is a loop of "write → error → fix".
- An error message is the compiler or interpreter **helping you**: it says where the problem is, and sometimes how to fix it.
- The scary bugs are the ones that **don't** produce an error but give a wrong result — so an error is actually good news.

> [!TIP]
> Treat an error message as a **clue**, not as criticism. You're the detective: start from the clue and track down the problem.

## 5.2 The anatomy of an error message

Formats differ a little between languages, but they all tell you **where** (file, line, column), **what kind** and **details**.

### Python: read from the bottom up

```python norun
def average(numbers):
    return sum(numbers) / len(numbers)

print(average([]))
```

```text
Traceback (most recent call last):
  File "main.py", line 4, in <module>
    print(average([]))
          ~~~~~~~^^^^
  File "main.py", line 2, in average
    return sum(numbers) / len(numbers)
           ~~~~~~~~~~~~~^~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

- **The last line matters most**: `ZeroDivisionError: division by zero` — the error type is "division by zero".
- Above it is the **traceback**: how the program got to the error, step by step. The lowest `File "main.py", line 2` is the line that actually failed; `line 4` above it is where that code was called from.
- The `~~~^^^` marks show the exact spot.

So, for Python: **read the last line (what went wrong), then the last `File` (where).**

### C / C++: file:line:column: level: description

```text
main.c:5:20: error: expected ';' before 'return'
    5 |     printf("hi\n")
      |                   ^
      |                   ;
    6 |     return 0;
```

Read it as: file `main.c`, line `5`, column `20`, an `error`: "a `;` was expected before return". The compiler even draws where the semicolon should go.

> [!IMPORTANT]
> Compiled languages often report **many errors at once**, but later ones are frequently a chain reaction caused by the first. **Always fix the first error first**, then compile again.

### Errors versus warnings

- **error**: must be fixed, otherwise the program can't compile or run.
- **warning**: the program runs, but the compiler thinks something is **probably wrong** (e.g. a variable that's never used). Don't ignore warnings — many bugs hide there.

## 5.3 Words you'll see in error messages

Error messages use very "template-like" English. With these words you can decode most of them:

| Word | Meaning | Example |
|------|---------|---------|
| error | something that must be fixed | `SyntaxError` |
| warning | probably a problem | `warning: unused variable` |
| syntax | the grammar of the language | `invalid syntax` |
| invalid | not allowed | `invalid character` |
| expected | something should be here | `expected ':'` |
| unexpected | something should NOT be here | `unexpected indent` |
| missing | something is absent | `missing return statement` |
| undefined / not defined | unknown name | `name 'x' is not defined` |
| undeclared | used before being declared | `'x' undeclared` |
| declared | introduced (a name) | `declared and not used` |
| unused | never used | `unused import` |
| type | kind of value | `TypeError` |
| mismatch / mismatched | doesn't fit | `mismatched types` |
| incompatible | can't be combined | `incompatible types` |
| cannot / can't | impossible | `cannot find symbol` |
| convert | change a value to another type | `cannot convert string to int` |
| index | position in a list | `IndexError` |
| out of range / out of bounds | position doesn't exist | `index out of range` |
| null / nil / None | "nothing", an empty value | `NullPointerException` |
| reference | a name pointing to something | `undefined reference` |
| symbol | a name (variable, function, class) | `cannot find symbol` |
| scope | the region where a name is visible | `not declared in this scope` |
| argument | value passed in a call | `takes 1 positional argument but 2 were given` |
| parameter | name in a function definition | |
| return | give back a result | `return type` |
| statement | one instruction | `unreachable statement` |
| expression | something that produces a value | `expected expression` |
| token | one "word" or symbol of code | `unexpected token` |
| literal | a value written directly | `unterminated string literal` |
| unterminated / never closed | not finished | `'(' was never closed` |
| indent / indentation | spaces at the start of a line | `IndentationError` |
| attribute | a property or method | `has no attribute 'x'` |
| module | a file/library of code | `No module named 'numpy'` |
| permission denied | not allowed by the system | |
| not found | doesn't exist | `file not found` |
| overflow | number too big for its type | `integer overflow` |
| exception | an error raised while running | `Exception in thread "main"` |
| panic | (Rust/Go) the program crashed | `thread 'main' panicked` |
| segmentation fault | illegal memory access | |
| deprecated | old, should no longer be used | `deprecated function` |
| traceback / stack trace | the chain of calls leading to an error | |

> [!TIP]
> When the app recognises an error, it shows a plain-language explanation below it. Read it, then compare with the original message — the vocabulary will soon feel familiar.

## 5.4 Debugging techniques

**Debugging** means finding and fixing mistakes (bugs) in a program. The most useful techniques:

### 1. Read, think, then change

Don't change code randomly the moment you see an error. Instead:

1. Read the message: **what error? which line?**
2. Look at that line (and the **line before** — many syntax errors are a missing symbol on the previous line).
3. Think: what did I **expect**? What **actually** happened?
4. Make **one** small change and run again.

### 2. Print debugging: make the program tell you what it's doing

If the result is wrong but there's no error, print the values of variables at key points:

```python
price = 100
discount = 20
final = price - price * discount / 100
print("DEBUG: final =", final)     # what did we actually get?
print("To pay:", final)
```

Seeing intermediate results shows which step goes wrong. Remove the debug prints when you're done.

### 3. Halve the code

If the program is long and you have no idea where the problem is, **comment out** the second half and run it. Still broken? The problem is in the first half. Fixed? It's in the second half. Repeat on the bad half. Each step rules out half the code.

### 4. Rubber duck debugging

Explain your code **line by line to someone** — or to a rubber duck on your desk. While explaining "what this line does", you'll often suddenly see the mistake. It works so well that programmers gave it a name: rubber duck debugging.

### 5. Make a minimal example

Copy the failing code and **delete everything unrelated** to the problem, until the smallest possible program still shows the error. This often reveals the cause on its own — and if not, it's the perfect thing to show when asking for help.

### 6. Use a debugger

A **debugger** lets the program **pause** at a line (a breakpoint), step through one line at a time and inspect every variable. VS Code and other editors have one built in (click left of a line number to set a breakpoint, press <kbd>F5</kbd> to start). Learn it once your programs get bigger.

## 5.5 Searching and asking for help

### Searching

1. Copy the **key, general part** of the error; remove your own file names, variable names and paths. Search for `ZeroDivisionError: division by zero`, not `main.py line 2 average`.
2. Add the language name: `python ZeroDivisionError division by zero`.
3. Good places: **Stack Overflow**, official documentation, GitHub issues.

### Asking

A good question lets someone help you in a minute. Include:

1. **what you're trying to do**;
2. **your code** (a minimal example, as text in a code block — not a screenshot);
3. **the complete error message** (copied as text);
4. **what you already tried**;
5. your environment: operating system and language version.

### Using AI assistants well

ChatGPT, Claude, Gemini, DeepSeek and other assistants can be great for learning — if used well:

- ✅ Ask them to **explain** errors, concepts and code: "Why does this code give this error? Explain step by step."
- ✅ Ask for **hints** or practice problems rather than finished answers.
- ✅ After writing code yourself, ask for a **review**: "What could be improved?"
- ❌ **Don't copy answers** without understanding — you lose the practice and won't manage next time.
- ⚠️ AI can be confidently wrong. **Always run and check** the code and compare with official documentation.

## 5.6 Practice: fix this program

This program has 3 mistakes. Run it and fix them one by one, guided by the errors (exercise 3 is the same program, so you can check your fix there):

```python norun
print("Receipt")
apple = 3
banana = 5
print("Apples:" apple)
print("Bananas:", Banana)
print("Total:", apple + banana
```

## 5.7 Summary

- Errors help you; the scary bugs are wrong results without errors.
- Read Python errors **from the bottom up**: last line = what, last `File` = where.
- Compiler messages look like "file:line:column: level: description"; **fix the first error first**.
- Errors must be fixed; take warnings seriously too.
- Techniques: read-think-change, print debugging, halving, rubber duck, minimal example, debugger.
- Search without personal details; ask with goal, code, full error and what you tried.
- Use AI to explain and to get hints, never copy blindly, always verify.

## 5.8 Exercises

<!-- exercise: read-traceback -->
### Exercise 1: Read a Python error

Which line does this error point to?

```text
Traceback (most recent call last):
  File "main.py", line 7, in <module>
    total = add(1, "2")
  File "main.py", line 3, in add
    return a + b
           ~~^~~
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

- Line 1
- Line 3: a number and text can't be added
- Line 7: the function name is misspelled
- You can't tell from the message

<details>
<summary>Answer and explanation</summary>

The last `File` is `line 3`, and `TypeError` says an `int` (integer) and a `str` (string) can't be added with `+`. The root cause is the text `"2"` passed on line 7, but the failure happens on line 3.
</details>

<!-- exercise: first-error -->
### Exercise 2: Which first?

A C compiler reports 12 errors at once. What's the best approach?

- Start with the last error
- Fix the first error, then compile again
- Fix all 12, then compile
- Switch to another compiler

<details>
<summary>Answer and explanation</summary>

Later errors are often a chain reaction of the first. Fix the first, recompile, and many of the others usually disappear.
</details>

<!-- exercise: fix-receipt -->
### Exercise 3: Fix the receipt

Fix the program from 5.6 so that it prints:

```text
Receipt
Apples: 3
Bananas: 5
Total: 8
```

<details>
<summary>Hint 1</summary>

Line 4: a comma is missing between the two things to print.
</details>

<details>
<summary>Hint 2</summary>

Line 5: the variable's capitalisation doesn't match; line 6: a closing parenthesis is missing.
</details>

<!-- exercise: word-meaning -->
### Exercise 4: Error vocabulary

What does the message `expected ')'` mean?

- There's an extra closing parenthesis
- A closing parenthesis should be here (it's missing)
- The parenthesis is full-width
- The expected output is a parenthesis

<details>
<summary>Answer and explanation</summary>

"Expected" means "I expected to see this here", i.e. a `)` is **missing**. The opposite, "unexpected", means something appears where it **shouldn't**.
</details>

## 5.9 Further reading

- [Python tutorial · Errors and exceptions](https://docs.python.org/3/tutorial/errors.html)
- [How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html) — the classic essay on asking good questions.
- [Rubber duck debugging (Wikipedia)](https://en.wikipedia.org/wiki/Rubber_duck_debugging)
- Next lesson: [How computers store numbers and text](../06-numbers-and-text/en.md)
