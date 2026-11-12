# 2. Computer basics: files, folders and typing

> 🎯 **Goals**: understand files, folders, extensions and paths; turn on file extensions; avoid "smart" and full-width punctuation; learn the names of the keyboard symbols and the shortcuts programmers use.
>
> ⏱ **Time**: 30 minutes　|　📋 **You need**: lesson 1

Many people get stuck not because programming is hard, but because some everyday computer ideas were never explained to them. This lesson lays that foundation.

## 2.1 Files and folders

Everything on a computer — photos, documents, music, programs — is stored in **files**. Files live in **folders** (also called **directories**), and folders can contain other folders, forming a tree:

```text
C:\Users\alex\                  ← your user folder (Windows)
├── Desktop\
├── Documents\
│   ├── CV.docx
│   └── code\                    ← let's make a folder for code
│       ├── hello.py
│       └── game\
│           └── main.py
└── Downloads\
```

> [!TIP]
> Create a folder called `code` inside "Documents" right now and keep all your practice code there. **Use only letters, digits and dashes in folder names** — no spaces or accented/non-Latin letters. Some programming tools break on paths with spaces or special characters.

## 2.2 Extensions: a file's "type label"

The part of a file name after the last dot is the **extension**. It tells the computer what kind of file it is:

| Extension | Type |
|-----------|------|
| `.txt` | plain text |
| `.docx` | Word document |
| `.jpg` `.png` | images |
| `.exe` | Windows program |
| `.py` | Python source code |
| `.c` `.h` | C source code |
| `.cpp` | C++ source code |
| `.java` | Java source code |
| `.rs` | Rust source code |
| `.go` | Go source code |
| `.js` `.ts` | JavaScript / TypeScript source code |
| `.md` | Markdown (the lessons of this course are .md files) |

### Always turn on "show extensions"

Windows and macOS **hide** extensions by default. That causes a classic problem: you think your file is `hello.py`, but it is really `hello.py.txt`, and it won't run.

- **Windows 11**: File Explorer → "View" at the top → "Show" → tick "File name extensions".
- **Windows 10**: File Explorer → "View" → tick "File name extensions".
- **macOS**: Finder → menu "Finder" → "Settings" → "Advanced" → tick "Show all filename extensions".
- **Linux**: most file managers show them already.

## 2.3 Code is plain text

Code files are **plain text**: just characters, no fonts, colours or bold. So:

- ✅ Write code in a **code editor**: this course's app, VS Code, Notepad (Windows), TextEdit (macOS, switched to "plain text" mode).
- ❌ **Don't** use Word, Pages or Google Docs for code — they add hidden formatting and turn straight quotes `"` into curly ones `“”`, which breaks code.

> [!NOTE]
> Text files are saved with an "encoding" that decides how characters become 0s and 1s. **Always choose UTF-8** — it can represent every writing system in the world and is the default of every modern tool. Lesson 6 explains how it works.

## 2.4 Paths: a file's address

A **path** is the full address of a file, written from the outermost folder down to the file:

- Windows: `C:\Users\alex\Documents\code\hello.py` (separated by **backslashes** `\`, starting with a drive letter `C:`)
- macOS: `/Users/alex/Documents/code/hello.py` (separated by **slashes** `/`)
- Linux: `/home/alex/code/hello.py`

There are two kinds:

- **Absolute path**: the full address from the very top, like the examples above.
- **Relative path**: an address relative to the folder you're "in" right now. If you are in `code`, then `game/main.py` means `code/game/main.py`; `..` means "the folder one level up".

You'll use paths a lot in the next lesson, about the terminal.

## 2.5 Smart quotes and full-width symbols ⚠️

Programming languages only understand plain **ASCII** punctuation: `( ) , ; : " ' . ! ?`. Two kinds of look-alike characters cause endless trouble:

- **Smart (curly) quotes** `“ ” ‘ ’` — produced automatically by word processors, chat apps and many web pages.
- **Full-width symbols** `（ ），；：` — produced by Chinese, Japanese and Korean input methods. They are as wide as a CJK character.

Compare (look closely at the brackets and quotes):

```text
✅ print("Hello")        ← plain brackets and quotes
❌ print(“Hello”)        ← curly quotes
❌ print（"Hello"）      ← full-width brackets
❌ print("Hello")；      ← full-width semicolon
```

How to avoid them:

1. Type code in a code editor, not a word processor.
2. When copying code from a web page or a chat message, check the quotes.
3. If you use an East-Asian input method, switch to English mode while typing code (often just <kbd>Shift</kbd>).
4. When an error appears, read the app's explanation — it recognises these characters.

> [!IMPORTANT]
> Any language **can** appear in two places: inside quoted text (`"Hello"`, `"Привет"`) and in comments (`# a comment`). Everything else — keywords, names, symbols — is plain ASCII.

## 2.6 Keyboard symbols programmers use

Programming uses many symbols you rarely type otherwise. Knowing their names also helps when reading documentation:

| Symbol | Name | Typical key (US layout) |
|--------|------|-------------------------|
| `( )` | parentheses (round brackets) | <kbd>Shift</kbd> + <kbd>9</kbd> / <kbd>0</kbd> |
| `[ ]` | square brackets | <kbd>[</kbd> <kbd>]</kbd> |
| `{ }` | curly braces | <kbd>Shift</kbd> + <kbd>[</kbd> / <kbd>]</kbd> |
| `< >` | angle brackets | <kbd>Shift</kbd> + <kbd>,</kbd> / <kbd>.</kbd> |
| `"` | double quote | <kbd>Shift</kbd> + <kbd>'</kbd> |
| `'` | single quote / apostrophe | <kbd>'</kbd> |
| `` ` `` | backtick | key left of <kbd>1</kbd> |
| `;` | semicolon | <kbd>;</kbd> |
| `:` | colon | <kbd>Shift</kbd> + <kbd>;</kbd> |
| `=` | equals sign | <kbd>=</kbd> |
| `_` | underscore | <kbd>Shift</kbd> + <kbd>-</kbd> |
| `#` | hash / pound | <kbd>Shift</kbd> + <kbd>3</kbd> |
| `\` | backslash | <kbd>\</kbd> |
| `/` | slash | <kbd>/</kbd> |
| `\|` | pipe / vertical bar | <kbd>Shift</kbd> + <kbd>\</kbd> |
| `&` | ampersand | <kbd>Shift</kbd> + <kbd>7</kbd> |
| `*` | asterisk | <kbd>Shift</kbd> + <kbd>8</kbd> |
| `%` | percent | <kbd>Shift</kbd> + <kbd>5</kbd> |
| `~` | tilde | <kbd>Shift</kbd> + <kbd>`</kbd> |
| `^` | caret | <kbd>Shift</kbd> + <kbd>6</kbd> |

(Other keyboard layouts put some of these elsewhere.)

## 2.7 Shortcuts you must know

Shortcuts make you much faster. Windows/Linux use <kbd>Ctrl</kbd>; macOS uses <kbd>⌘ Command</kbd>:

| Action | Windows / Linux | macOS |
|--------|-----------------|-------|
| Copy | <kbd>Ctrl</kbd>+<kbd>C</kbd> | <kbd>⌘</kbd>+<kbd>C</kbd> |
| Paste | <kbd>Ctrl</kbd>+<kbd>V</kbd> | <kbd>⌘</kbd>+<kbd>V</kbd> |
| Cut | <kbd>Ctrl</kbd>+<kbd>X</kbd> | <kbd>⌘</kbd>+<kbd>X</kbd> |
| Undo | <kbd>Ctrl</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Z</kbd> |
| Redo | <kbd>Ctrl</kbd>+<kbd>Y</kbd> or <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> |
| Select all | <kbd>Ctrl</kbd>+<kbd>A</kbd> | <kbd>⌘</kbd>+<kbd>A</kbd> |
| Save | <kbd>Ctrl</kbd>+<kbd>S</kbd> | <kbd>⌘</kbd>+<kbd>S</kbd> |
| Find | <kbd>Ctrl</kbd>+<kbd>F</kbd> | <kbd>⌘</kbd>+<kbd>F</kbd> |
| Comment / uncomment | <kbd>Ctrl</kbd>+<kbd>/</kbd> | <kbd>⌘</kbd>+<kbd>/</kbd> |
| Indent / unindent | <kbd>Tab</kbd> / <kbd>Shift</kbd>+<kbd>Tab</kbd> | same |

In this course's app you also have: <kbd>F5</kbd> run, <kbd>Ctrl</kbd>+<kbd>Enter</kbd> check answer, <kbd>Alt</kbd>+<kbd>←</kbd>/<kbd>→</kbd> previous/next lesson.

## 2.8 Try it

This program prints some common symbols. Run it, then **type it again yourself** in the editor (no copy-paste) to practise:

```python
print("parentheses ( )  brackets [ ]  braces { }")
print("quotes \" '  semicolon ;  colon :  equals =")
print("A path: C:\\Users\\code\\hello.py")
```

Look at line three: to put one backslash `\` inside a string you write two, `\\` (the "escape sequences" from lesson 1).

## 2.9 Summary

- Data is stored in **files**, files live in **folders**, folders form a tree.
- The **extension** is a file's type; turn on "show extensions" to avoid `hello.py.txt`.
- Code is **plain text**: use a code editor, not a word processor; save as **UTF-8**.
- A **path** is a file's address, absolute or relative; Windows uses `\`, macOS/Linux use `/`.
- Code needs plain ASCII punctuation — beware of smart quotes and full-width symbols.
- Learn the symbol names and the shortcuts; typing code becomes much smoother.

## 2.10 Exercises

<!-- exercise: extension -->
### Exercise 1: Extensions

What extension does a Python source file normally have?

- `.exe`
- `.txt`
- `.py`
- `.docx`

<details>
<summary>Answer and explanation</summary>

`.py`. `.exe` is a Windows program, `.txt` plain text and `.docx` a Word document.
</details>

<!-- exercise: half-width -->
### Exercise 2: Which line runs?

Which line uses **only plain ASCII symbols** and therefore runs?

- `print（"OK"）`
- `print(“OK”)`
- `print("OK")`
- `print("OK")；`

<details>
<summary>Answer and explanation</summary>

The third. The first has full-width brackets, the second curly quotes, the fourth a full-width semicolon at the end.
</details>

<!-- exercise: symbols -->
### Exercise 3: Symbol practice

Type it yourself (no copying) — write a program that prints:

```text
(a + b) * [c - d] / {e}
```

<details>
<summary>Hint</summary>

Put the whole line inside one pair of double quotes and pass it to `print`.
</details>

<!-- exercise: backslash -->
### Exercise 4: Print a Windows path

Write a program that prints:

```text
C:\code\hello.py
```

<details>
<summary>Hint</summary>

Inside a string, write `\` as `\\`, e.g. `"C:\\code"`.
</details>

## 2.11 Further reading

- [The Missing Semester of Your CS Education](https://missing-semester.csail.mit.edu/) (MIT): shells, editors, Git and other tools every programmer uses.
- Next lesson: [The terminal (command line)](../03-terminal/en.md)
