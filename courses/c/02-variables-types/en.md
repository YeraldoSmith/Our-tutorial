# 2. Variables and types

> 🎯 **Goals**: **declare** variables in C (always stating the type); meet the basic types `int`, `long long`, `double`, `char` and `bool` and how many bytes they take; print each type with `printf` **format specifiers** (`%d`, `%f`, `%c`…) and control width and decimal places; understand that "characters are small integers"; define constants with `const`; know that "an uninitialised variable holds garbage"; meet the fixed-size integer types of `<stdint.h>`.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lesson 1 of this course

## 2.1 Declaring variables: say the type first

In Python, `age = 12` creates a variable. In C, a variable **must be declared first**, stating its **type**:

```c
#include <stdio.h>

int main(void) {
    int age = 12;              // declare an integer variable age with initial value 12
    double height = 1.52;      // a decimal number (double-precision floating point)
    char grade = 'A';          // one character, in single quotes

    printf("age: %d\n", age);
    printf("height: %f m\n", height);
    printf("grade: %c\n", grade);

    age = age + 1;             // assignment: change the value (the type can't change)
    printf("next year: %d\n", age);
    return 0;
}
```

Why state the type? Because C must know **at compile time** how much memory each variable needs and which instructions to compute with. Once declared, a variable's type **can never change** — you can't store an integer and later a string in it (this is **static typing**).

You can declare several variables of the same type at once: `int a = 1, b = 2, c;`.

## 2.2 Basic types

| Type | Meaning | Typical size | Example |
|------|---------|--------------|---------|
| `int` | integer | 4 bytes, about ±2.1 billion | `int count = 42;` |
| `long long` | big integer | 8 bytes, about ±9.2 × 10¹⁸ | `long long big = 9000000000LL;` |
| `double` | decimal number (double precision) | 8 bytes, about 15–16 significant digits | `double pi = 3.14159;` |
| `float` | decimal number (single precision) | 4 bytes, about 6–7 significant digits | `float f = 2.5f;` |
| `char` | one character (really a 1-byte small integer) | 1 byte | `char c = 'x';` |
| `bool` | true / false | 1 byte | `bool ok = true;` |

> [!NOTE]
> `bool`, `true` and `false` are keywords in the **C23** standard and can be used directly. Older standards (C99–C17) need `#include <stdbool.h>` first. This course uses C23 (the `-std=c2x` option); add that line if an older compiler complains.

`sizeof` tells you how many bytes a type or variable takes:

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    printf("char:      %zu bytes\n", sizeof(char));
    printf("int:       %zu bytes, range %d to %d\n", sizeof(int), INT_MIN, INT_MAX);
    printf("long long: %zu bytes, max %lld\n", sizeof(long long), LLONG_MAX);
    printf("float:     %zu bytes, about %d significant digits\n", sizeof(float), FLT_DIG);
    printf("double:    %zu bytes, about %d significant digits\n", sizeof(double), DBL_DIG);
    printf("bool:      %zu bytes\n", sizeof(bool));
    return 0;
}
```

`<limits.h>` and `<float.h>` define the limits of each type. Print the result of `sizeof` with `%zu`.

> [!TIP]
> **Which type?** Use `int` for integers in general; `long long` for values that may exceed 2.1 billion (big amounts of money in cents, timestamps); `double` for decimals (`float` is too imprecise unless you need to save memory).

## 2.3 printf format specifiers

The first argument of `printf` is a **format string**; `%d`, `%f` and so on are "placeholders" replaced by the following arguments in order:

| Specifier | For | Example → output |
|-----------|-----|------------------|
| `%d` | `int` | `printf("%d", 42)` → `42` |
| `%lld` | `long long` | `printf("%lld", 9000000000LL)` → `9000000000` |
| `%f` | `double` (6 decimals by default) | `printf("%f", 3.14)` → `3.140000` |
| `%.2f` | 2 decimal places | `printf("%.2f", 3.14159)` → `3.14` |
| `%e` | scientific notation | `printf("%e", 12345.678)` → `1.234568e+04` |
| `%c` | `char` | `printf("%c", 'A')` → `A` |
| `%s` | a string | `printf("%s", "hi")` → `hi` |
| `%x` | hexadecimal integer | `printf("%x", 255)` → `ff` |
| `%zu` | the result of `sizeof` | `printf("%zu", sizeof(int))` → `4` |
| `%%` | a percent sign | |

You can also control **width** and **alignment** — perfect for neat tables:

```c
#include <stdio.h>

int main(void) {
    printf("[%5d]\n", 42);       // width 5, right-aligned: [   42]
    printf("[%-5d]\n", 42);      // left-aligned:           [42   ]
    printf("[%05d]\n", 42);      // padded with zeros:      [00042]
    printf("[%8.3f]\n", 3.14159);// width 8, 3 decimals:    [   3.142]
    printf("[%-6s|%6s]\n", "ab", "cd");

    printf("%-8s%6s%8s\n", "item", "qty", "price");
    printf("%-8s%6d%8.2f\n", "apple", 3, 4.5);
    printf("%-8s%6d%8.2f\n", "banana", 12, 0.75);
    return 0;
}
```

> [!NOTE]
> `printf` counts width in **bytes**. Letters outside English (Chinese, Cyrillic…) take 2–3 bytes each in UTF-8, so columns containing them won't line up exactly; for aligned tables, use ASCII headers or compute the padding yourself.

> [!WARNING]
> **The specifier must match the type.** `printf("%d", 3.14)` reads a `double` as if it were an `int` and prints a nonsense number (this is "undefined behaviour"). Fortunately, with `-Wall` the compiler warns: `format '%d' expects argument of type 'int', but argument 2 has type 'double'`. **Always fix warnings.**

## 2.4 Characters are small integers

Computers have no "letters", only numbers. A `char` really stores an integer from 0 to 127 (or −128 to 127), and the **ASCII table** maps each number to a character: `'A'` is 65, `'a'` is 97, `'0'` is 48.

```c
#include <stdio.h>

int main(void) {
    char c = 'A';
    printf("the code of %c is %d\n", c, c);  // the same value: %c shows a character, %d a number
    printf("%c\n", c + 2);                   // 'A' + 2 = 67 = 'C'
    printf("%c\n", 'a' - 'A' + 'G');         // uppercase to lowercase: g
    char digit = '7';
    printf("the number of the character '7': %d\n", digit - '0');   // '7' - '0' = 55 - 48 = 7
    return 0;
}
```

> [!NOTE]
> A `char` holds a single byte, so only English letters, digits and common symbols fit. Chinese characters, Cyrillic letters and so on take 2–4 bytes in UTF-8 and don't fit in one `char`; store such text in **strings** (lesson 9).

## 2.5 Constants

A value that never changes is a **constant**. There are several ways to write one:

```c
#include <stdio.h>

#define MAX_STUDENTS 40              // a preprocessor macro: every MAX_STUDENTS is replaced with 40 before compiling

int main(void) {
    const double PI = 3.14159265;    // const: this variable can't be changed afterwards
    double r = 2.0;
    printf("area: %.2f\n", PI * r * r);
    printf("at most %d students\n", MAX_STUDENTS);
    // PI = 3;                       // uncomment for an error: assignment of read-only variable 'PI'
    return 0;
}
```

- `const`: an ordinary variable, but read-only. Modern C prefers this style because it has a type the compiler can check.
- `#define`: plain text replacement done by the **preprocessor** (the "preprocess" step from lesson 1); no type; names are written in capitals by convention.
- C23 also adds `constexpr` for "constants computable at compile time", e.g. `constexpr int SIZE = 10;`.

**Literals** have their own rules: `10` is an `int`, `10LL` a `long long`, `3.14` a `double`, `3.14f` a `float`, `'x'` a character and `"x"` a string (not the same thing!). Integers can also be written in hexadecimal `0xFF` or binary `0b1010` (C23), and long numbers can be split with single quotes: `1'000'000` (C23).

## 2.6 Naming rules

- only letters, digits and underscores, and **not starting with a digit**: `score2` ✓, `2score` ✗;
- case matters: `count` and `Count` are two different variables;
- keywords can't be used: `int`, `return`, `if`, `while`…;
- convention: variables and functions in lowercase with underscores (`total_score`), macro constants in capitals (`MAX_SIZE`).

## 2.7 Uninitialised variables: garbage values

In Python you can't use a variable that was never assigned. C lets you declare a variable **without an initial value**, but then it contains **whatever happened to be left in that memory** — a "garbage value":

```c norun
#include <stdio.h>

int main(void) {
    int x;                  // no initial value
    printf("%d\n", x);      // could be 0, could be 32765, may differ on every run
    return 0;
}
```

```text
warning: 'x' is used uninitialized [-Wuninitialized]
```

This is one of the most common bugs in C programs. **Make it a habit: always give variables an initial value when you declare them.**

## 2.8 Overflow: integers have limits

Every integer type has a range. What happens beyond it?

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    unsigned char small = 255;          // unsigned: no sign, only 0–255
    small = small + 1;
    printf("255 + 1 = %d\n", small);    // 0: unsigned overflow "wraps around" to 0

    long long big = INT_MAX;            // a bigger type doesn't overflow
    big = big + 1;
    printf("INT_MAX + 1 = %lld\n", big);
    return 0;
}
```

- **unsigned** integers wrap around on overflow like a car's odometer — this behaviour is well defined;
- overflowing a **signed** integer (an ordinary `int`) is **undefined behaviour**: the C standard doesn't say what happens, so the program may produce strange numbers or even stranger bugs. **Avoid it**: estimate how big your data can get and use `long long` when needed.

In 1996 the European Ariane 5 rocket exploded 40 seconds after launch because converting a 64-bit floating-point number to a 16-bit integer overflowed.

## 2.9 Fixed-size integers: `<stdint.h>`

The size of `int` can differ between platforms (4 bytes on almost every computer, but only 2 on some microcontrollers). When you need an **exact size** (file formats, network data, hardware registers), use `<stdint.h>`:

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    int8_t   a = -100;          // exactly 8 bits (1 byte), signed
    uint8_t  b = 200;           // exactly 8 bits, unsigned: 0–255
    int32_t  c = 2000000000;    // exactly 32 bits
    uint64_t d = 18000000000000000000u;   // exactly 64 bits, unsigned
    printf("%d %d %" PRId32 " %" PRIu64 "\n", a, b, c, d);
    printf("%zu %zu %zu %zu\n", sizeof a, sizeof b, sizeof c, sizeof d);
    return 0;
}
```

`PRId32` and `PRIu64` are format specifiers from `<inttypes.h>` that are correct on every platform.

## 2.10 Summary

- C variables must **declare a type** first: `int`, `long long`, `double`, `float`, `char`, `bool`; the type can't change afterwards.
- `sizeof` gives the number of bytes (print it with `%zu`); `<limits.h>` and `<float.h>` contain the ranges.
- `printf` specifiers: `%d`, `%lld`, `%f`, `%.2f`, `%c`, `%s`, `%x`, `%zu`; `%5d`, `%-5d`, `%05d` control width and alignment; **specifiers must match types**.
- A `char` is a small integer (ASCII): `'A'` is 65, `'7' - '0'` is 7.
- Constants: `const` (preferred), `#define`, C23's `constexpr`.
- **Initialise when you declare**, or you get garbage; signed overflow is undefined behaviour; use `<stdint.h>` when you need exact sizes.

## 2.11 Exercises

<!-- exercise: profile -->
### Exercise 1: A profile

Declare three variables in the program — `int age = 12;`, `double height = 1.523;`, `char grade = 'B';` — and print them **using format specifiers** (height with 2 decimal places):

```text
Sample output: Age: 12, Height: 1.52 m, Grade: B
```

<details>
<summary>Hint</summary>

`printf("Age: %d, Height: %.2f m, Grade: %c\n", age, height, grade);`
</details>

<!-- exercise: sizes -->
### Exercise 2: Sizes of types

Use `sizeof` to print the number of bytes of `char`, `int`, `double` and `long long`, separated by spaces.

```text
Sample output: 1 4 8 8
```

<details>
<summary>Hint</summary>

`printf("%zu %zu %zu %zu\n", sizeof(char), sizeof(int), sizeof(double), sizeof(long long));`
</details>

<!-- exercise: ascii -->
### Exercise 3: Characters and codes

Print two things separated by a space: the ASCII code of the character `'Z'`, and the character whose code is `97`.

```text
Sample output: 90 a
```

<details>
<summary>Hint</summary>

The same value prints as a number with `%d` and as a character with `%c`: `printf("%d %c\n", 'Z', 97);`
</details>

<!-- exercise: table -->
### Exercise 4: An aligned table

Use widths to print this table (name left-aligned in 6 columns, quantity right-aligned in 4, price right-aligned in 8 with 2 decimal places):

```text
apple    3    4.50
kiwi    12    0.75
```

<details>
<summary>Hint</summary>

`printf("%-6s%4d%8.2f\n", "apple", 3, 4.5);`
</details>

<!-- exercise: format-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

int main(void) {
    int a = 7;
    double b = 2.5;
    printf("%d %.1f|%5.2f|%-4d|\n", a, b, b * 2, a);
    printf("%c%c\n", 'a' + 1, 'A' + 25);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

First line: `%d` prints `7`; `%.1f` prints `2.5`; `%5.2f` prints `5.00`, which is only 4 characters in a width of 5, so one space is added in front; `%-4d` left-aligns `7` in 4 columns, adding 3 spaces after it. So: `7 2.5| 5.00|7   |`. Second line: `'a' + 1` is `b` and `'A' + 25` is `Z`, so it prints `bZ`.
</details>

<!-- exercise: uninit -->
### Exercise 6: A variable without an initial value

After `int x;`, what does `printf("%d", x);` print?

- always 0
- an unpredictable "garbage value" that may differ each time
- the compiler refuses to compile it
- the program always crashes

<details>
<summary>Answer and explanation</summary>

C doesn't initialise local variables automatically; they hold whatever was left in that memory. The compiler (with `-Wall`) usually just warns. So always initialise when you declare.
</details>

## 2.12 Further reading

- [cppreference: arithmetic types](https://en.cppreference.com/w/c/language/arithmetic_types)
- [cppreference: printf format specification](https://en.cppreference.com/w/c/io/fprintf)
- [The ASCII table](https://en.cppreference.com/w/c/language/ascii)
- Next lesson: [Operators and expressions](../03-operators/en.md)
