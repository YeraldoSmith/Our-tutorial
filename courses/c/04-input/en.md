# 4. Input: scanf

> 🎯 **Goals**: read integers, decimals, characters and words typed by the user with `scanf`; understand why variables need an `&` in front; remember that "`scanf` reads a `double` with `%lf`"; check `scanf`'s **return value** to cope with typos; solve the "leftover newline" problem when reading characters; keep reading until the input ends.
>
> ⏱ **Time**: 50 minutes　|　📋 **You need**: lessons 2–3 of this course

## 4.1 Your first scanf

`printf` handles output, `scanf` handles input. Their format specifiers are almost the same:

```c input="12"
#include <stdio.h>

int main(void) {
    int age = 0;
    printf("How old are you? ");
    scanf("%d", &age);                 // read an integer and store it in age
    printf("Next year you'll be %d!\n", age + 1);
    return 0;
}
```

When it runs, the program stops and waits for you. Type `12` and press Enter, and it carries on.

### Why the `&`?

`&age` reads "the **address** of age". Every variable lives somewhere in memory, and that location has a number — its address. `scanf` needs to know **where to store** the number it reads, so you tell it the address.

```c
#include <stdio.h>

int main(void) {
    int age = 12;
    printf("the value of age is %d\n", age);
    printf("the address of age is %p\n", (void *)&age);     // %p prints an address (may differ each run)
    return 0;
}
```

`printf` only needs the value, so no `&`; `scanf` has to change the variable, so the `&` is required. Lesson 10 on pointers makes this completely clear. **Forgetting the `&` is one of the most common beginner mistakes, and the program usually crashes.**

## 4.2 Reading several values at once

```c input="3 4"
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    printf("Enter two integers: ");
    scanf("%d %d", &a, &b);
    printf("%d + %d = %d\n", a, b, a + b);
    return 0;
}
```

`%d` automatically skips spaces, tabs and newlines in front, so typing `3 4`, `3    4`, or `3` and `4` on two lines all give the same result.

## 4.3 scanf's format specifiers

| Type | in `scanf` | in `printf` |
|------|-----------|-------------|
| `int` | `%d` | `%d` |
| `long long` | `%lld` | `%lld` |
| `double` | **`%lf`** | `%f` |
| `float` | `%f` | `%f` |
| `char` | `%c` (or `" %c"`, see 4.5) | `%c` |
| a single word (string) | `%s` (**no `&`**) | `%s` |

> [!WARNING]
> **`scanf` must read a `double` with `%lf`** — `%f` reads a wrong value — while `printf` prints a `double` with `%f`. This asymmetry is a very common source of mistakes in C.

```c input="1.75 60"
#include <stdio.h>

int main(void) {
    double height = 0, weight = 0;
    printf("Height (m) and weight (kg): ");
    scanf("%lf %lf", &height, &weight);
    printf("BMI = %.1f\n", weight / (height * height));
    return 0;
}
```

## 4.4 Checking the return value: what if the user types something wrong?

If the program wants an integer and the user types `abc`, what happens? `scanf` **can't read it**, and the variable keeps its old value. Fortunately, `scanf` has a **return value**: how many values it read successfully.

```c input="abc"
#include <stdio.h>

int main(void) {
    int n = 0;
    printf("Enter an integer: ");
    if (scanf("%d", &n) != 1) {          // we expect to read 1 value
        printf("\nThat's not an integer!\n");
    } else {
        printf("You entered %d\n", n);
    }
    return 0;
}
```

(`if … else` comes properly in the next lesson.) Real programs often end with `return 1;` when they get input they can't handle, using a non-zero exit code to tell the operating system "something went wrong" (lesson 1).

| `scanf` returns | Meaning |
|-----------------|---------|
| the number of values requested | complete success |
| fewer than requested | only part was read (the rest didn't match the format) |
| `EOF` (usually -1) | the input has ended (nothing more to read) |

> [!TIP]
> **Serious programs always check `scanf`'s return value.** With `-Wall`, the compiler sometimes warns that the return value of scanf is ignored. To keep things short, the later exercises in this course often assume the input is always correct.

## 4.5 Reading characters: beware the leftover newline

```c input="5\nY"
#include <stdio.h>

int main(void) {
    int count = 0;
    char answer = 0;
    printf("How many? ");
    scanf("%d", &count);
    printf("Are you sure? (Y/N) ");
    scanf(" %c", &answer);             // note the space before %c!
    printf("\nquantity %d, answer %c\n", count, answer);
    return 0;
}
```

After typing `5`, the user pressed Enter, and the newline `\n` it produced **is still waiting in the input**. `%d` skips whitespace, but `%c` **doesn't** — it dutifully reads that newline. Putting a space before it, `" %c"`, means "skip all whitespace first, then read one character", which solves the problem.

## 4.6 Reading a word

```c input="Tom"
#include <stdio.h>

int main(void) {
    char name[50];                     // a "char array" holding up to 49 characters (details in lesson 9)
    printf("Your name: ");
    scanf("%49s", name);               // an array's name already is an address, so no &
    printf("Hello, %s!\n", name);
    return 0;
}
```

- `%s` reads **up to the next whitespace (space, newline)**, so it reads just one word; to read a whole line use `fgets` (lesson 9).
- The 49 in `%49s` means "read at most 49 characters", leaving room for the `\0` that ends a string. **Always write this limit**: otherwise input that's too long overwrites memory — the source of many security holes.

## 4.7 Reading until the input ends

Problems often don't say how many numbers there are, only "read until the end of input". Use `scanf`'s return value:

```c input="3 8 1 9 4"
#include <stdio.h>

int main(void) {
    int x = 0, count = 0;
    long long sum = 0;
    while (scanf("%d", &x) == 1) {       // keep going as long as another integer can be read
        sum += x;
        count++;
    }
    printf("%d numbers, sum %lld\n", count, sum);
    return 0;
}
```

`while` loops are explained in detail next lesson; for now just remember this pattern. When typing input by hand in a terminal, press **Ctrl+D** (macOS / Linux) or **Ctrl+Z then Enter** (Windows) to signal "end of input".

## 4.8 Common mistakes

```c norun
int age = 0;
scanf("%d", age);          // forgot the &
```

```text
warning: format '%d' expects argument of type 'int *', but argument 2 has type 'int' [-Wformat=]
```

At run time it will most likely crash (`Segmentation fault`). `int *` means "a pointer to an int (an address)" — the compiler is telling you an address is needed here.

```c norun
double h = 0;
scanf("%f", &h);           // a double needs %lf
```

```text
warning: format '%f' expects argument of type 'float *', but argument 2 has type 'double *' [-Wformat=]
```

> [!NOTE]
> Compiling with Microsoft Visual Studio, you may see `'scanf': This function or variable may be unsafe. Consider using scanf_s instead` (error C4996). That's Microsoft's own rule, not a requirement of the C standard. Add `#define _CRT_SECURE_NO_WARNINGS` as the very first line of the file, or use GCC / Clang.

## 4.9 Summary

- `scanf("%d", &x)` reads an integer into `x`; **`&` means "address of"**, and since `scanf` changes the variable it's required (except for char arrays).
- `scanf` specifiers: `%d`, `%lld`, **`%lf` (double)**, `%c`, `%s`; `%d`, `%lf` and `%s` skip whitespace automatically.
- `scanf` returns the number of values it read, or `EOF` at the end of input; check input with `if (scanf(...) != 1)`.
- Read a character with `" %c"` (leading space) to skip a leftover newline; read a word with `%49s` to limit its length.
- `while (scanf("%d", &x) == 1)` reads until the input ends.

## 4.10 Exercises

<!-- exercise: sum-two -->
### Exercise 1: Two numbers

Read two integers and print their sum, difference and product, separated by spaces.

```text
Sample input: 7 3
Sample output: 10 4 21
```

<details>
<summary>Hint</summary>

`scanf("%d %d", &a, &b);` then `printf("%d %d %d\n", a + b, a - b, a * b);`
</details>

<!-- exercise: avg3 -->
### Exercise 2: The average of three decimals

Read three decimal numbers and print their average with 2 decimal places.

```text
Sample input: 1.5 2.5 4
Sample output: 2.67
```

<details>
<summary>Hint</summary>

Read into `double` variables with `%lf`: `scanf("%lf %lf %lf", &a, &b, &c);`
</details>

<!-- exercise: next-year -->
### Exercise 3: Name and age

Read a name (one word) and an age, and print `NAME will be AGE+1 next year.`.

```text
Sample input: Tom 12
Sample output: Tom will be 13 next year.
```

<details>
<summary>Hint</summary>

`char name[50]; int age;`, `scanf("%49s %d", name, &age);` — note `name` has no `&` but `age` does.
</details>

<!-- exercise: until-eof -->
### Exercise 4: Read until the end

Read some integers (you don't know how many) and print how many there are, their sum and their maximum, separated by spaces. (There is at least one number.)

```text
Sample input: 3 8 1 9 4
Sample output: 5 25 9
```

<details>
<summary>Hint</summary>

Use `while (scanf("%d", &x) == 1)`; start the maximum as the first number read, or initialise it to `INT_MIN` (from `<limits.h>`).
</details>

<!-- exercise: scanf-return -->
### Exercise 5: Predict the output

Suppose the input to this program is `12 abc`:

```c
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    int n = scanf("%d %d", &a, &b);
    printf("%d %d %d\n", n, a, b);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`12` is read into `a`; `abc` isn't an integer, so reading `b` fails and `b` keeps its 0. `scanf` returns the number of values read, 1. The output is `1 12 0`.
</details>

<!-- exercise: why-amp -->
### Exercise 6: Why the `&`?

Why does `scanf("%d", &age);` use `&age` rather than `age`?

- `&` means "this is an integer"
- `scanf` must store the value it reads in the variable, so it needs the variable's address in memory, and `&age` is that address
- `&` is just a habit; it works fine without it
- `&` adds 1 to the number typed

<details>
<summary>Answer and explanation</summary>

`&` is the address-of operator. Given only `age`, `scanf` gets just a value (say 0) and doesn't know where to write the result, so the program often crashes.
</details>

## 4.11 Further reading

- [cppreference: scanf](https://en.cppreference.com/w/c/io/fscanf)
- Next lesson: [Conditions](../05-conditions/en.md)
