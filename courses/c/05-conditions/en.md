# 5. Conditions

> 🎯 **Goals**: use `if`, `else if` and `else` to make programs act differently depending on the situation; understand why **curly braces** are a must; handle "pick one of many" with `switch`, understanding `break` and "fall-through"; use the ternary operator `? :`; recognise classic traps such as "an extra semicolon after `if`".
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lessons 3–4 of this course (comparisons and logic, scanf)

## 5.1 The if statement

```c input="75"
#include <stdio.h>

int main(void) {
    int score = 0;
    printf("Score: ");
    scanf("%d", &score);
    if (score >= 60) {
        printf("You passed!\n");
    }
    printf("the end\n");
    return 0;
}
```

- the condition goes in **parentheses**, which are required;
- when the condition is true (non-zero), the code in the braces runs; when false, it's skipped;
- unlike Python, it's the `{ }`, not indentation, that decides which code belongs to the `if`.

## 5.2 else and else if

```c input="85"
#include <stdio.h>

int main(void) {
    int score = 0;
    scanf("%d", &score);
    if (score >= 90) {
        printf("excellent\n");
    } else if (score >= 80) {
        printf("good\n");
    } else if (score >= 60) {
        printf("pass\n");
    } else {
        printf("fail\n");
    }
    return 0;
}
```

The conditions are checked from top to bottom, and **only the first branch that's true runs**; the rest are skipped. So order matters: put `score >= 60` first and even 90 would count as just a "pass".

## 5.3 Always write the braces

If an `if` controls only one statement, C lets you leave out the braces. But this causes lots of bugs:

```c norun
if (balance < price)
    printf("insufficient funds\n");
    return 1;                // looks like part of the if, but isn't! It always runs
```

Without braces, the `if` controls only **the one statement right after it**. The second line is indented but isn't inside the `if` — the program runs `return 1;` every time. With `-Wall`, the compiler warns:

```text
warning: this 'if' clause does not guard... [-Wmisleading-indentation]
```

In 2014 a famous security hole in Apple's systems ("goto fail") happened exactly like this: one extra line without braces made the SSL certificate check get skipped. **Make it a habit: always write braces after `if`, `else`, `for` and `while`.**

## 5.4 A classic trap: a semicolon after if

```c norun
if (x > 100);              // look at that semicolon!
{
    printf("x is big\n");  // runs whatever x is
}
```

The semicolon after `if (x > 100)` is itself an "empty statement", and that's what the `if` controls; the braces below are just an ordinary block that always runs. The compiler warns `suggest braces around empty body in an 'if' statement`.

## 5.5 Nesting and combining conditions

An `if` can contain another `if`:

```c input="2024"
#include <stdio.h>

int main(void) {
    int year = 0;
    scanf("%d", &year);
    // leap year: divisible by 4 but not by 100, or divisible by 400
    if (year % 4 == 0) {
        if (year % 100 != 0 || year % 400 == 0) {
            printf("%d is a leap year\n", year);
        } else {
            printf("%d is not a leap year\n", year);
        }
    } else {
        printf("%d is not a leap year\n", year);
    }
    return 0;
}
```

Deep nesting gets hard to read. Often the conditions can be combined into one with logical operators:

```c norun
bool leap = (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
```

## 5.6 switch: pick one of many

When choosing between many cases based on **the value of one integer (or character)**, `switch` is clearer than a long chain of `else if`:

```c input="3"
#include <stdio.h>

int main(void) {
    int day = 0;
    scanf("%d", &day);
    switch (day) {
        case 1:
            printf("Monday\n");
            break;
        case 2:
            printf("Tuesday\n");
            break;
        case 3:
            printf("Wednesday\n");
            break;
        case 6:
        case 7:                        // 6 and 7 share the same code
            printf("weekend!\n");
            break;
        default:                       // runs when nothing above matched
            printf("another day\n");
            break;
    }
    return 0;
}
```

- the value in `switch (expression)` must be an **integer or character** (not a decimal, not a string);
- each `case` is followed by a constant; execution starts there when it equals the expression's value;
- **`break` leaves the `switch`**. Forget it and the program **carries on into the next `case`'s code** — this is **fall-through**;
- `case 6:` above has nothing after it, deliberately falling through so 6 and 7 run the same code.

> [!WARNING]
> Forgetting `break` is the most common `switch` mistake. With `-Wextra` the compiler warns `this statement may fall through`. If the fall-through is intentional, C23 lets you write `[[fallthrough]];` to say so.

```c
#include <stdio.h>

int main(void) {
    char op = '*';
    int a = 6, b = 7;
    switch (op) {
        case '+': printf("%d\n", a + b); break;
        case '-': printf("%d\n", a - b); break;
        case '*': printf("%d\n", a * b); break;
        case '/':
            if (b != 0) {
                printf("%d\n", a / b);
            } else {
                printf("can't divide by 0\n");
            }
            break;
        default:  printf("unknown operator %c\n", op); break;
    }
    return 0;
}
```

## 5.7 The ternary operator `? :`

`condition ? value1 : value2` gives value1 if the condition is true, otherwise value2. Handy for short "either/or" choices:

```c
#include <stdio.h>

int main(void) {
    int a = 7, b = 12;
    int bigger = (a > b) ? a : b;
    printf("the bigger one is %d\n", bigger);

    int n = 3;
    printf("%d is %s\n", n, (n % 2 == 0) ? "even" : "odd");
    printf("found %d file%s\n", n, n == 1 ? "" : "s");
    return 0;
}
```

Don't nest ternary operators deeply — at that point an `if` is clearer.

## 5.8 Summary

- `if (condition) { ... } else if (condition) { ... } else { ... }`: checked top to bottom; only the first true branch runs.
- **Always write braces**: without them an `if` controls only one statement; indentation means nothing to C.
- A semicolon after `if (...);` disables the condition.
- `switch` picks one case based on an integer or character; end each `case` with `break` or it falls through; `default` handles everything else.
- `condition ? a : b` is a short either/or.

## 5.9 Exercises

<!-- exercise: sign -->
### Exercise 1: Positive, negative or zero

Read an integer and print `positive` if it's positive, `negative` if it's negative and `zero` if it's zero.

```text
Sample input: -5
Sample output: negative
```

<details>
<summary>Hint</summary>

`if (n > 0) ... else if (n < 0) ... else ...`
</details>

<!-- exercise: grade -->
### Exercise 2: Grades

Read a score from 0 to 100 and print its grade: 90 and above `A`, 80–89 `B`, 70–79 `C`, 60–69 `D`, below 60 `F`.

```text
Sample input: 85
Sample output: B
```

<details>
<summary>Hint</summary>

Check `score >= 90`, `score >= 80`… from the highest down.
</details>

<!-- exercise: leap -->
### Exercise 3: Leap years

Read a year and print `leap` if it's a leap year, `common` otherwise. (A year divisible by 4 but not by 100, or divisible by 400, is a leap year.)

```text
Sample input: 1900
Sample output: common
```

<details>
<summary>Hint</summary>

`if ((y % 4 == 0 && y % 100 != 0) || y % 400 == 0)`
</details>

<!-- exercise: calc -->
### Exercise 4: A switch calculator

Read a calculation of the form `integer operator integer` (the operator is one of `+`, `-`, `*`, `/`, with spaces around it) and print the result (division is integer division). If the divisor is 0 or the operator is unknown, print `error`.

```text
Sample input: 6 * 7
Sample output: 42
```

<details>
<summary>Hint</summary>

`scanf("%d %c %d", &a, &op, &b);` — the space before `%c` skips whitespace. Then `switch (op)`.
</details>

<!-- exercise: braces-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    if (x > 10)
        printf("A\n");
        printf("B\n");
    if (x > 3) printf("C\n"); else printf("D\n");
    switch (x) {
        case 5: printf("E\n");
        case 6: printf("F\n"); break;
        default: printf("G\n");
    }
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

The first `if` has no braces and controls only `printf("A\n")`; `x > 10` is false, so A isn't printed, but B isn't controlled by the `if` and is printed anyway. `x > 3` is true, so C is printed. The `switch` starts at `case 5`, prints E, has no `break`, falls through into `case 6` and prints F, then `break`s. The output is the four lines B, C, E, F.
</details>

<!-- exercise: semicolon -->
### Exercise 6: An extra semicolon

What does `if (x > 100); { printf("big\n"); }` do?

- prints big only when x is greater than 100
- prints big whatever x is
- fails to compile
- never prints big

<details>
<summary>Answer and explanation</summary>

The semicolon in `if (x > 100);` is an empty statement, and that's all the `if` controls; the braces after it are an independent block that always runs. The compiler warns `suggest braces around empty body`.
</details>

## 5.10 Further reading

- [cppreference: if statement](https://en.cppreference.com/w/c/language/if), [switch statement](https://en.cppreference.com/w/c/language/switch)
- The story of the "goto fail" bug (search for Apple goto fail SSL bug)
- Next lesson: [Loops](../06-loops/en.md)
