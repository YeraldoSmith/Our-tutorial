# 1. Your first C program

> 🎯 **Goals**: know why C is worth learning; write, compile and run your first C program and understand every line of it; understand what "compiling" really does (preprocess → compile → assemble → link); print text and special characters with `printf`; write comments; recognise the most common compiler errors.
>
> ⏱ **Time**: 50 minutes　|　📋 **You need**: the "Getting started" course (especially lesson 3 on the terminal, lesson 4 on installing a C compiler and lesson 5 on reading errors)

## 1.1 Why learn C?

C was born in 1972 and is older than almost any software you've used. Yet it's still one of the most important languages in the world:

- **operating systems**: the kernels of Linux, Windows and macOS are mainly written in C;
- **embedded devices**: the programs on the chips in washing machines, cars, routers and drones are mostly C;
- **the foundation of other languages**: the Python interpreter (CPython), the SQLite database, Redis and Git are all written in C;
- **understanding computers**: C puts you face to face with **memory**, **addresses** and **bytes**. Learn C and you'll understand how programs really run inside a computer, which makes every other language easier.

Compared with Python, C is more "wordy", makes you manage more details yourself, and is easier to get wrong — but those very details are what let you truly understand the machine. Don't worry; we'll go step by step.

## 1.2 Hello, World!

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    printf("Hello, C!\n");
    return 0;
}
```

Click "Run" (or compile and run it in a terminal, see section 1.3) and two lines appear. Line by line:

| Code | Meaning |
|------|---------|
| `#include <stdio.h>` | include the description (header file) of the "standard input/output" library, so we can use `printf`. `stdio` = **st**andar**d** **i**nput/**o**utput |
| `int main(void)` | define the **`main` function**: the program starts running here. `int` means it returns an integer at the end; `void` means it takes no arguments |
| `{ ... }` | the curly braces enclose the function body — the statements to run |
| `printf("...\n");` | print some text. `\n` means "new line". **Every statement ends with a semicolon `;`** |
| `return 0;` | `main` ends and returns 0 to the operating system, meaning "the program finished normally" |

> [!NOTE]
> Unlike Python, C marks blocks of code with **curly braces** and ends statements with **semicolons**; indentation is only for people, and the compiler doesn't care about it. Still, always indent neatly (4 spaces), or your code quickly becomes unreadable.

## 1.3 Compiling and running

Python is **interpreted**: the interpreter reads the code and runs it as it goes. C is **compiled**: a **compiler** first translates the source code into **machine code** the computer can run directly, producing an **executable file**, and then you run that file.

In the learning app, "Run" compiles and runs for you automatically. To do it in your own terminal, save the code as `hello.c` and:

```bash
$ gcc hello.c -o hello       # compile: turn hello.c into the executable hello
$ ./hello                    # run it (macOS / Linux)
$ hello.exe                  # run it (Windows)
```

- `gcc` is the compiler (on macOS, `gcc` is actually Clang, used the same way);
- `-o hello` names the output file; without it, the default is `a.out` (`a.exe` on Windows);
- adding `-Wall -Wextra` makes the compiler warn you about likely problems: `gcc -Wall -Wextra hello.c -o hello`. The learning app adds these two options by default.

### What does compiling actually do?

Behind that one `gcc` command, four things happen in order:

```text
hello.c ──① preprocess──► expand #include, remove comments
        ──② compile─────► translate into assembly language (hello.s)
        ──③ assemble────► translate into machine code (hello.o, an object file)
        ──④ link────────► join it with libraries (printf lives in the C standard library) ──► the executable hello
```

So:

- **compile-time errors** (syntax errors, misspelled names) are found in steps ② and ④, and no program is produced at all;
- the resulting executable **runs on its own**, without the source code or the compiler — which is why software is often shipped as just a `.exe`;
- compiled programs run very fast, a major reason C is still so widely used.

> [!TIP]
> If you're curious, `gcc -S hello.c` shows the generated assembly code (`hello.s`). This curriculum's "Assembly" course explains how to read it.

## 1.4 Printing with printf

`printf` doesn't start a new line by itself; write `\n` wherever you want one:

```c
#include <stdio.h>

int main(void) {
    printf("first line");
    printf(" still the first line\n");
    printf("second line\nthird line\n");
    return 0;
}
```

Combinations starting with a backslash, like `\n`, are **escape sequences**; they stand for characters that are awkward to type directly:

| Written as | Means |
|------------|-------|
| `\n` | new line |
| `\t` | tab (jump to the next alignment position) |
| `\\` | one backslash `\` |
| `\"` | one double quote `"` |
| `%%` | one percent sign `%` (a rule of `printf` itself, because `%` has a special job — see next lesson) |

```c
#include <stdio.h>

int main(void) {
    printf("Name\tAge\n");
    printf("Tom\t12\n");
    printf("He said: \"Hi\"\n");
    printf("The file is in C:\\Users\\me\n");
    printf("100%% done\n");
    return 0;
}
```

## 1.5 Comments

Comments are notes for people; the compiler ignores them:

```c
#include <stdio.h>

/* This is a multi-line comment:
   it can span many lines,
   starting with / and * and ending with * and /. */

int main(void) {
    // This is a single-line comment: from // to the end of the line
    printf("comments don't affect the program\n");   // can also follow code
    return 0;
}
```

## 1.6 The return value of `main`

`return 0;` hands 0 to the operating system, meaning "all good"; returning another number (such as 1) usually means "something went wrong". In a terminal you can see the previous program's return value (its **exit code**):

```bash
$ ./hello
$ echo $?                 # macOS / Linux: shows 0
$ echo %errorlevel%       # Windows Command Prompt
```

Scripts and other programs use the exit code to tell whether your program succeeded. Since C99, if `main` ends without a `return`, it behaves as `return 0;` — but writing it explicitly is a good habit.

## 1.7 The most common compiler errors

Compiler messages are in English but follow clear patterns. The learning app explains the cause in your language underneath.

**A missing semicolon**:

```c norun
#include <stdio.h>

int main(void) {
    printf("Hello\n")
    return 0;
}
```

```text
hello.c: In function 'main':
hello.c:4:22: error: expected ';' before 'return'
```

`hello.c:4:22` means "line 4, column 22". Note that the semicolon is missing at the end of line 4, but the compiler often only notices when it reads the next word (`return` on line 5). **Check the line reported and the line above it.**

**A misspelled function name, or a missing `#include`**:

```c norun
int main(void) {
    printf("Hello\n");
    return 0;
}
```

```text
hello.c:2:5: warning: implicit declaration of function 'printf' [-Wimplicit-function-declaration]
hello.c:1:1: note: include '<stdio.h>' or provide a declaration of 'printf'
```

The compiler doesn't know `printf` because `#include <stdio.h>` is missing. GCC 13 and earlier only give a **warning** and the program may even run; GCC 14 and later, and Clang, report an **error**. Either way, fix it — **don't ignore warnings**; they are often the source of bugs. Misspelling `printf` as `printF` or `pritnf` gives a similar message (and the `note` line kindly tells you which header to include).

**Non-English punctuation**:

```c norun
#include <stdio.h>

int main(void) {
    printf（"Hello\n"）；
    return 0;
}
```

```text
hello.c:4:5: error: 'printf\U0000ff08' undeclared (first use in this function); did you mean 'printf'?
```

`\U0000ff08` is the Unicode number of the full-width bracket `（` typed by Chinese and Japanese input methods; the compiler thinks you wrote a name "printf（". (Other compilers and versions phrase it differently, for example `stray '\357' in program`: "there's a byte in the program that shouldn't be there".) The same happens with curly quotes `“ ”` that word processors insert automatically, and with Cyrillic letters like `р` that look exactly like Latin `p`. C only accepts plain ASCII symbols `(`, `)`, `;`, `"` in code. Text **inside** strings may be in any language, but the code itself must use ASCII symbols.

## 1.8 Summary

- C is the high-level language closest to the hardware; operating systems, embedded devices and many interpreters are written in it.
- Every C program starts running at `int main(void) { ... }`; `printf` needs `#include <stdio.h>`.
- Statements end with `;`, blocks use `{ }`; indentation is for people.
- C is compiled: `gcc hello.c -o hello` compiles, `./hello` runs; compiling = preprocess → compile → assemble → link.
- `printf` doesn't add new lines; escape sequences `\n`, `\t`, `\\`, `\"`, plus `%%`.
- Comments: `// single line` and `/* multi-line */`. `return 0` means the program ended normally.
- For errors, look at the reported line and the one above it; semicolons, `#include` and non-ASCII punctuation are the most common culprits.

## 1.9 Exercises

<!-- exercise: hello-c -->
### Exercise 1: Hello, C

Write a program that prints one line: `Hello, C!`.

```text
Sample output: Hello, C!
```

<details>
<summary>Hint</summary>

Follow the program in section 1.2: `printf("Hello, C!\n");`, and don't forget `#include <stdio.h>` and `return 0;`.
</details>

<!-- exercise: triangle -->
### Exercise 2: A little triangle

Print this triangle of stars (4 lines):

```text
*
**
***
****
```

<details>
<summary>Hint</summary>

You can use 4 `printf`s, or just one: `printf("*\n**\n***\n****\n");`.
</details>

<!-- exercise: escapes -->
### Exercise 3: Escape characters

Print these three lines exactly:

```text
"C" is fun!
Path: C:\new
100% done
```

<details>
<summary>Hint</summary>

Write a double quote as `\"`; a backslash as `\\` (careful: `\n` would be a new line!); a percent sign inside `printf` as `%%`.
</details>

<!-- exercise: fix-errors -->
### Exercise 4: Fix the errors

The starter code has 3 mistakes that stop it from compiling properly. Find and fix them so the program prints:

```text
I can fix errors.
Compiler is my friend.
```

<details>
<summary>Hint</summary>

Run it and look at the first message, fix it, run again — one at a time. Check: is an `#include` missing? Are the semicolons plain ASCII? Is a function name misspelled?
</details>

<!-- exercise: printf-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

int main(void) {
    printf("A");
    printf("B\n");
    printf("C\nD");
    printf("\n");
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`printf` doesn't add new lines: `A` and `B` are on the same line; a new line follows `C`, `D` is on its own line, and a final new line ends it. So the output is three lines: `AB`, `C`, `D`.
</details>

<!-- exercise: return-zero -->
### Exercise 6: What `return 0` means

What does `return 0;` in `main` mean?

- the program prints the number 0
- the program ended normally, handing exit code 0 to the operating system
- the program runs again from the start
- it frees all memory, otherwise the program fails

<details>
<summary>Answer and explanation</summary>

`main`'s return value is the program's **exit code**, and 0 means success. It doesn't print anything on the screen.
</details>

## 1.10 Further reading

- [cppreference: the C reference](https://en.cppreference.com/w/c) (for looking up syntax and functions later)
- Brian W. Kernighan and Dennis M. Ritchie: *The C Programming Language* (K&R, the classic by C's creators)
- [Compiler Explorer](https://godbolt.org/): see the assembly your C code compiles to, in the browser
- Next lesson: [Variables and types](../02-variables-types/en.md)
