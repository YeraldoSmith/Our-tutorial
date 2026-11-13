# 14. Multi-file programs, the preprocessor and C23

> 🎯 **Goals**: split a program into several `.c` source files and `.h` headers, with "declarations in headers, definitions in source files"; write include guards; hide a module's internal functions and variables with `static` and share global variables with `extern`; understand the stages of compilation and read linker errors; use the preprocessor well (macros with parameters and their traps, conditional compilation, `assert`); meet Makefiles and CMake; get a complete picture of **C23**'s new features. Finally, see where to go after C.
>
> ⏱ **Time**: 100 minutes　|　📋 **You need**: lessons 1–13 of this course

## 14.1 Why split into several files?

So far every program has been a single file. Real projects run to tens of thousands or millions of lines; nobody could find their way around one giant file, and changing one line would mean recompiling everything. So programs are split by purpose into **modules**: each module is a `.c` file (the implementation) plus a `.h` file (the interface).

Here's a small project of three files. The learning app runs one file at a time, so try these in a new folder on your own computer (the compile command is in section 14.3).

```c norun
// geometry.h — the module's "interface": tells others what the module offers
#ifndef GEOMETRY_H
#define GEOMETRY_H

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b);
double triangle_area(Point a, Point b, Point c);

#endif
```

```c norun
// geometry.c — the module's "implementation"
#include <math.h>
#include "geometry.h"

static double cross(Point o, Point a, Point b) {    // static: used only in this file
    return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
}

double distance(Point a, Point b) {
    return hypot(a.x - b.x, a.y - b.y);
}

double triangle_area(Point a, Point b, Point c) {
    return fabs(cross(a, b, c)) / 2;
}
```

```c norun
// main.c — uses the module
#include <stdio.h>
#include "geometry.h"

int main(void) {
    Point a = {0, 0}, b = {4, 0}, c = {0, 3};
    printf("AB = %.1f\n", distance(a, b));
    printf("area = %.1f\n", triangle_area(a, b, c));
    return 0;
}
```

- **Headers (`.h`) hold only declarations**: function prototypes, type definitions (`struct`, `typedef`, `enum`), macros, `extern` variable declarations. Whoever uses the module `#include`s its header.
- **Source files (`.c`) hold definitions**: function bodies and the definitions of global variables. A source file also includes its own header, so the compiler can check that declarations and definitions agree.
- `#include "geometry.h"` with **double quotes** looks in the current folder first; `#include <stdio.h>` with **angle brackets** looks only in the system's header directories.

## 14.2 Include guards, `static` and `extern`

### Include guards

`#include` really just **pastes the header's contents** in place. If a `.c` file ends up including `geometry.h` twice (indirectly), `Point` gets defined twice and the compiler complains `redefinition of ...`. The three lines at the start and end of the header above are an **include guard**:

```c norun
#ifndef GEOMETRY_H      // if GEOMETRY_H hasn't been defined yet...
#define GEOMETRY_H      // ...define it, and process what follows
/* the header's contents */
#endif                  // the second time, GEOMETRY_H is defined and the whole thing is skipped
```

Every header needs one; the macro name is usually the file name in capitals. Many compilers also accept the shorter `#pragma once`, which does the same job (but isn't part of standard C).

### `static`: a module's private parts

Outside a function, `static` means "**visible only in this file**". `cross` above is a helper inside `geometry.c`; with `static`, other files can't see it, and it can't clash with a function of the same name elsewhere. It's much like "private" in object-oriented languages: the header publishes the interface, and the `static` parts are implementation details.

(Note that a `static` local variable **inside** a function means something else — its value is kept between calls, as lesson 7 showed.)

### `extern`: sharing a global variable

If you really need a global variable shared by several files, declare it with `extern` in a **header** and define it in **exactly one** source file:

```c norun
// config.h
extern int verbose;           // declaration: this variable is defined elsewhere

// config.c
#include "config.h"
int verbose = 0;              // definition: only in one place
```

Put `int verbose = 0;` straight into the header instead, and every `.c` file that includes it gets its own definition, so linking fails with `multiple definition of 'verbose'`. Still, as always: use as few global variables as you can.

## 14.3 Compiling, linking and linker errors

To compile a multi-file program in a terminal, list every `.c` file:

```bash
gcc -std=c23 -Wall -Wextra -g main.c geometry.c -o app -lm
./app
```

(Before GCC 14, use `-std=c2x`. `-lm` links the maths library, which contains `hypot` and `fabs`.) That one command actually runs in several steps:

```text
main.c ──preprocess──▶ #include and macros expanded ──compile──▶ main.o ─┐
                                                                         ├──link──▶ app (executable)
geometry.c ──preprocess──▶ ...                      ──compile──▶ geometry.o ─┘   ↑ plus the standard library
```

1. **Preprocessing**: handle every line starting with `#` — paste in headers, expand macros, process conditional compilation.
2. **Compiling**: translate each `.c` file **on its own** into machine code, producing an object file (`.o`). While compiling `main.c`, the compiler only knows `distance`'s prototype, not its body.
3. **Linking**: the linker joins all the `.o` files and the standard library, matching every function call with its definition.

So there are two different kinds of error: **compile errors** (syntax and type problems, with a file and line number) and **linker errors** (everything compiled, but when putting it together a definition is missing or appears more than once):

| Linker error | Cause |
|--------------|-------|
| `undefined reference to 'distance'` | `geometry.c` was left out of the compile command; or the name is misspelt; or there's only a prototype and no body |
| `multiple definition of 'verbose'` | a variable or function **definition** was put in a header included by several `.c` files |
| `undefined reference to 'sqrt'` (on some systems) | `-lm` is missing |

Compiling separately pays off: change only `geometry.c`, and only it needs recompiling before relinking — a huge time saver in big projects. The build tools below do this for you automatically.

## 14.4 Build tools: Make and CMake

Typing long compile commands every time gets old. **Make** reads a file called `Makefile` and rebuilds only what changed:

```makefile
CC = gcc
CFLAGS = -std=c2x -Wall -Wextra -g

app: main.o geometry.o
	$(CC) main.o geometry.o -o app -lm

main.o: main.c geometry.h
	$(CC) $(CFLAGS) -c main.c

geometry.o: geometry.c geometry.h
	$(CC) $(CFLAGS) -c geometry.c
```

Type `make` in a terminal and it builds `app`. Each rule is "target: dependencies", and the line below (which **must be indented with a Tab**) is the command that produces the target; it runs only when a dependency is newer than the target.

Larger projects usually use **CMake**, which generates build files for different systems and compilers (including Visual Studio on Windows):

```cmake
cmake_minimum_required(VERSION 3.20)
project(geometry C)
set(CMAKE_C_STANDARD 23)
add_executable(app main.c geometry.c)
target_link_libraries(app m)
```

Then run `cmake -B build` and `cmake --build build`. (On Windows, drop the last line: the maths functions there don't need a separate library.)

## 14.5 The preprocessor

### Macros with parameters, and their traps

`#define` can define more than constants: it can define "macros with parameters". But a macro is just **text substitution**, not a function, which makes mistakes easy:

```c
#include <stdio.h>

#define SQUARE_BAD(x) x * x
#define SQUARE(x) ((x) * (x))           // brackets around every parameter and around the whole result
#define MAX(a, b) ((a) > (b) ? (a) : (b))

static inline int square(int x) {       // the better choice: an inline function
    return x * x;
}

int main(void) {
    printf("%d\n", SQUARE_BAD(1 + 2));  // becomes 1 + 2 * 1 + 2, which is 5!
    printf("%d\n", SQUARE(1 + 2));      // ((1 + 2) * (1 + 2)) = 9
    printf("%d\n", MAX(3, 7));          // 7
    printf("%d\n", square(1 + 2));      // 9: a function works out its argument first
    return 0;
}
```

- **Bracket** every parameter and the whole expression in a macro.
- Arguments may be evaluated twice: `MAX(i++, j)` increments `i` twice.
- So use functions where you can — `static inline` functions are just as fast and their types are checked; and define constants with `const` or `constexpr` rather than `#define` where you can.

### Conditional compilation and debugging output

```c
#include <stdio.h>
#include <assert.h>

#define DEBUG 1

int average(const int a[], int n) {
    assert(n > 0);                       // assertion: n must be positive, or the program stops and reports it
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
#if DEBUG
    printf("[debug] %s line %d: sum = %ld\n", __func__, __LINE__, sum);
#endif
    return (int)(sum / n);
}

int main(void) {
    int data[] = {3, 5, 10};
    printf("average: %d\n", average(data, 3));
    return 0;
}
```

- `#if` / `#ifdef` / `#else` / `#endif` decide **which code gets compiled**. They're used for debugging code and for code specific to an operating system (`#ifdef _WIN32`).
- `__FILE__`, `__LINE__` and `__func__` give the current file name, line number and function name — handy in debugging output.
- **`assert(condition)`** (`<assert.h>`): if the condition is false, it prints the file, line and condition, then aborts the program. It checks for things that "should never happen" and is a great self-check. For a release build, compile with `-DNDEBUG` and every `assert` disappears. Try changing `average(data, 3)` to `average(data, 0)`.

## 14.6 C23: modern C

The C standard is revised every few years: C89, C99, C11, C17 and now **C23**. This course has used C23 since lesson 1 (the option `-std=c2x`; newer compilers also accept `-std=c23`). Here are its main new features in one place:

```c
#include <stdio.h>

constexpr int MAX_PLAYERS = 4;                   // a compile-time constant, usable as an array length
enum Color : unsigned char { RED, GREEN, BLUE }; // choose the enum's underlying type

[[nodiscard]] int checked_half(int x, bool *ok) {   // [[nodiscard]]: the result shouldn't be ignored
    *ok = (x % 2 == 0);
    return x / 2;
}

int main(void) {
    int scores[MAX_PLAYERS] = {};            // {}: empty initialisation, all zero
    auto total = 0;                          // auto: type inferred from the initialiser (int here)
    int *p = nullptr;                        // nullptr: a real null pointer constant
    bool ok = false;                         // bool, true and false are keywords now
    int million = 1'000'000;                 // digit separators
    int flags = 0b1010'0101;                 // binary literals
    typeof(million) copy = million;          // typeof: the type of an expression
    static_assert(sizeof(int) >= 4);         // compile-time assertion (C23 lets you omit the message)
    enum Color c = BLUE;

    int half = checked_half(10, &ok);
    printf("%d %d %d %d\n", scores[0], total, half, ok);
    printf("%d %d %d\n", million, flags, copy);
    printf("p is %sa null pointer, c takes %zu byte\n", p == nullptr ? "" : "not ", sizeof c);
    return 0;
}
```

| Feature | What it means |
|---------|---------------|
| `bool`, `true`, `false` | keywords now; `<stdbool.h>` isn't needed |
| `nullptr` | the null pointer constant, safer than `NULL` (which may be just the integer 0) |
| `constexpr` | compile-time constants: safer than `#define`, more useful than `const` |
| `auto`, `typeof` | let the compiler infer a type, or get an expression's type |
| `{}` empty initialisation | `int a[10] = {};` zeroes everything |
| `0b1010`, `1'000'000` | binary literals, digit separators |
| `[[nodiscard]]`, `[[maybe_unused]]`, `[[deprecated]]` | standard "attributes" that help the compiler spot problems |
| `enum E : type` | choose an enum's underlying integer type |
| `static_assert(condition)` | a compile-time check; the message is optional |
| `<stdckdint.h>` | `ckd_add(&r, a, b)` and friends: integer arithmetic that detects overflow |
| `<stdbit.h>` | bit functions such as `stdc_count_ones` |
| `#embed "file.bin"` | embed a file's contents directly in an array |
| `strdup`, `memccpy` | now part of the standard library |

(The last few are newer: `<stdckdint.h>` and `<stdbit.h>` need GCC 14 or later, and `#embed` GCC 15 or Clang 19. If your compiler complains that it doesn't know `nullptr`, `bool` or similar, it's too old: upgrade to GCC 13+ or Clang 16+.)

## 14.7 After C

Congratulations on finishing the C course! You've mastered variables, control flow, functions, arrays, strings, pointers, dynamic memory, structs, files and multi-file programs — the foundations of understanding how computers work. Next, you could:

- **Build projects**: a command-line budget book, an address book, a text adventure, a simple shell, a maze solver, an image converter (the BMP format is simple)… Projects are the best way to make knowledge stick.
- **Learn data structures and algorithms**: stacks, queues, hash tables, binary trees, graphs, sorting and searching — implementing them yourself in C makes them sink in deeply.
- **Take this tutorial's other courses**: C++ adds classes, templates and a rich standard library on top of C; Rust solves C's memory-safety problems with compile-time checks; the assembly course shows the machine instructions your C code finally becomes.
- **Read good books**: *Modern C* by Jens Gustedt (a member of the C standards committee; it covers C23 and is free online); *Effective C* by Robert C. Seacord; and the classic *The C Programming Language* (K&R).

## 14.8 Summary

- Module = `.h` (declarations: prototypes, types, macros, `extern`) + `.c` (definitions); a source file includes its own header.
- Give every header an include guard (`#ifndef X_H` / `#define X_H` / `#endif`, or `#pragma once`).
- File-scope `static` keeps functions and variables private to their file; shared globals are declared `extern` in a header and defined in one source file.
- Compilation means preprocessing, compiling and linking; `undefined reference` and `multiple definition` are linker errors.
- Compile several files with `gcc main.c geometry.c -o app`; automate builds with Make or CMake.
- Macros are text substitution: bracket parameters and results, and prefer functions and `constexpr`; use `#if` for conditional compilation and `assert` for things that should never happen.
- C23: `bool`, `nullptr`, `constexpr`, `auto`, `typeof`, `{}`, `0b`, `'` separators, attributes, `static_assert` and more.

## 14.9 Exercises

<!-- exercise: stack-module -->
### Exercise 1: A stack module

The learning app runs one file at a time, so this exercise puts a module's three parts in one file: the interface (prototypes), the implementation and the user. Complete the four functions in the implementation part, using the `static` array `items` and counter `top` to make a **stack** (last in, first out): `stack_push` adds a number (ignored when full); `stack_pop` removes and returns the most recently added number, or -1 when the stack is empty; `stack_empty` says whether it's empty; `stack_size` returns how many items it holds. `main` is already written; it reads commands such as `push x`, `pop` and `size`.

```text
Sample input: push 3 push 5 pop size pop pop
Sample output:
5
1
3
-1
```

<details>
<summary>Hint</summary>

`push`: `if (top < STACK_MAX) { items[top] = value; top++; }`. `pop`: return -1 if empty, otherwise `top--; return items[top];`. `top` itself is the number of items.
</details>

<!-- exercise: find-negative -->
### Exercise 2: `nullptr` for "not found"

Write a function `int *find_negative(int a[], int n)` that returns the address of the first negative number in the array, or `nullptr` if there is none. Read `n` (1 ≤ n ≤ 1000) and `n` integers; if a negative number is found, print its index (worked out by subtracting pointers) and its value, separated by a space; otherwise print `none`.

```text
Sample input:
5
4 0 -7 2 -1
Sample output: 2 -7
```

<details>
<summary>Hint</summary>

Once found: `int *p = find_negative(a, n); if (p != nullptr) printf("%td %d\n", p - a, *p);`.
</details>

<!-- exercise: to-binary -->
### Exercise 3: Binary

Read an integer `x` (0 ≤ x < 2³¹) and print its binary representation (without leading zeros; `0` for zero) and the number of 1s in it, separated by a space.

```text
Sample input: 10
Sample output: 1010 2
```

<details>
<summary>Hint</summary>

Check from the highest bit down: `for (int bit = 30; bit >= 0; bit--)`, taking each bit with `(x >> bit) & 1`, and skip the zeros before the first 1. Alternatively, collect bits from the lowest up with `x % 2` and `x / 2` into an array, then print it backwards.
</details>

<!-- exercise: macro-predict -->
### Exercise 4: Predict the output

```c
#include <stdio.h>

#define DOUBLE_BAD(x) x + x
#define DOUBLE(x) ((x) + (x))
#define SQUARE_BAD(x) x * x

int main(void) {
    printf("%d\n", 3 * DOUBLE_BAD(2));
    printf("%d\n", 3 * DOUBLE(2));
    printf("%d\n", SQUARE_BAD(1 + 2));
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

Macros are just text substitution. `3 * DOUBLE_BAD(2)` becomes `3 * 2 + 2`, which is 8; `3 * DOUBLE(2)` becomes `3 * ((2) + (2))`, which is 12; `SQUARE_BAD(1 + 2)` becomes `1 + 2 * 1 + 2`, which is 5. The output:

```text
8
12
5
```
</details>

<!-- exercise: link-error -->
### Exercise 5: A linker error

The function `area` is defined in `geometry.c`; `main.c` includes `geometry.h` and calls `area`. Compiling with `gcc main.c -o app` fails with `undefined reference to 'area'`. Why?

- `geometry.h` has no include guard
- `geometry.c` was left out of the compile command
- `main.c` doesn't `#include <stdio.h>`
- `area` should have been `static`

<details>
<summary>Answer and explanation</summary>

`undefined reference` is a linker error: `main.c` compiled fine, but at link time there's no definition of `area`, because `geometry.c` never took part. The command should be `gcc main.c geometry.c -o app`. Making `area` `static` would hide it inside `geometry.c` and cause the very same error.
</details>

<!-- exercise: header-guard -->
### Exercise 6: Include guards

What are `#ifndef GEOMETRY_H` / `#define GEOMETRY_H` at the start of a header and `#endif` at its end for?

- they make the header compile faster
- they stop the same header being included more than once in a source file, which would define types and so on twice
- they allow function definitions in the header
- they stop other files from including the header

<details>
<summary>Answer and explanation</summary>

The first time, `GEOMETRY_H` isn't defined yet, so the contents are processed and `GEOMETRY_H` gets defined; on any later inclusion, the `#ifndef` condition is false and everything is skipped, so there's no `redefinition` error.
</details>

## 14.10 Further reading

- [cppreference: the preprocessor](https://en.cppreference.com/w/c/preprocessor), [what's new in C23](https://en.cppreference.com/w/c/23)
- [Modern C (Jens Gustedt, free)](https://gustedt.gitlabpages.inria.fr/modern-c/)
- [The official CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)
- Back to: [the C course](../)
