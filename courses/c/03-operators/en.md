# 3. Operators and expressions

> 🎯 **Goals**: master arithmetic in C, especially the rules of **integer division** and the remainder `%`; get decimal results with **type casts**; understand operator precedence; use shorthands like `+=` and `++`; master comparison and logical operators (including "short-circuiting"); avoid the classic `=` versus `==` trap; do maths with `<math.h>`; understand why `0.1 + 0.2 != 0.3`; get a first look at bitwise operators.
>
> ⏱ **Time**: 60 minutes　|　📋 **You need**: lessons 1–2 of this course

## 3.1 Arithmetic

```c
#include <stdio.h>

int main(void) {
    int a = 17, b = 5;
    printf("%d + %d = %d\n", a, b, a + b);
    printf("%d - %d = %d\n", a, b, a - b);
    printf("%d * %d = %d\n", a, b, a * b);
    printf("%d / %d = %d\n", a, b, a / b);      // integer division: 3, the fraction is simply dropped
    printf("%d %% %d = %d\n", a, b, a % b);     // remainder: 2
    return 0;
}
```

### Integer division: the most common "surprise"

In C, **dividing two integers gives an integer**: the fractional part is cut off (rounded towards 0), not rounded to the nearest:

| Expression | Result | Note |
|------------|--------|------|
| `7 / 2` | `3` | not 3.5 |
| `-7 / 2` | `-3` | truncated towards 0 (not -4) |
| `7 % 3` | `1` | remainder |
| `-7 % 3` | `-1` | the remainder takes the sign of the dividend |
| `7.0 / 2` | `3.5` | if either operand is a decimal, it's decimal division |

This is very different from Python (where `7 / 2` is 3.5 and `-7 // 2` is -4).

The remainder `%` is extremely useful: checking even/odd (`n % 2 == 0`), taking the last digit (`n % 10`), cycling counters (`i % 7` for the day of the week)…

## 3.2 Type conversion

To get a decimal result from two integers, **convert** one of them to `double` first:

```c
#include <stdio.h>

int main(void) {
    int total = 17, count = 5;
    double wrong = total / count;               // integer division gives 3 first, then it becomes 3.0
    double right = (double)total / count;       // total becomes 17.0 first, then decimal division
    printf("%.2f %.2f\n", wrong, right);        // 3.00 3.40

    double price = 9.99;
    int whole = (int)price;                     // decimal to integer: the fraction is cut off (not rounded)
    printf("%d\n", whole);                      // 9
    return 0;
}
```

`(type)value` is a **cast**. Look at the `wrong` line: the right-hand side `total / count` is computed first (integer division, result 3), and only **then** assigned to the `double` — by which time the fraction is long gone.

C also converts automatically in some cases: when an `int` and a `double` meet in an operation, the `int` becomes a `double`; assigning a `double` to an `int` variable truncates the fraction (the compiler sometimes warns with `-Wextra`).

## 3.3 Precedence

As in maths, multiplication and division come before addition and subtraction, and parentheses come first:

```c
#include <stdio.h>

int main(void) {
    printf("%d\n", 2 + 3 * 4);       // 14
    printf("%d\n", (2 + 3) * 4);     // 20
    printf("%d\n", 10 - 4 - 3);      // 3: same level goes left to right
    printf("%d\n", 2 * 3 % 4);       // 2: * / % share a level, left to right: 6 % 4
    return 0;
}
```

C has dozens of operators and a long precedence table. **When unsure, add parentheses** — they don't slow the program down and they make the code clearer.

## 3.4 Assignment shorthands, increment and decrement

```c
#include <stdio.h>

int main(void) {
    int score = 10;
    score += 5;      // score = score + 5    → 15
    score -= 3;      // score = score - 3    → 12
    score *= 2;      // score = score * 2    → 24
    score /= 5;      // score = score / 5    → 4 (integer division)
    score %= 3;      // score = score % 3    → 1
    printf("%d\n", score);

    int n = 5;
    n++;             // increment: n = n + 1 → 6
    n--;             // decrement: n = n - 1 → 5
    printf("%d\n", n);

    int a = 5;
    int b = a++;     // postfix: b gets a's old value (5), then a goes up by 1
    int c = ++a;     // prefix: a goes up by 1 first (to 7), then c gets the new value
    printf("a=%d b=%d c=%d\n", a, b, c);   // a=7 b=5 c=7
    return 0;
}
```

> [!WARNING]
> Don't modify the same variable more than once in one expression, as in `i = i++ + ++i;`. That's **undefined behaviour** in C, and different compilers may give completely different results. Put `++` in a statement of its own, or use it only in `for` loops (lesson 6).

## 3.5 Comparisons

A comparison gives **true (1)** or **false (0)**:

| Operator | Meaning |
|----------|---------|
| `==` | equal to (two equals signs!) |
| `!=` | not equal to |
| `<`, `>` | less than, greater than |
| `<=`, `>=` | less than or equal, greater than or equal |

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    printf("%d %d %d\n", x == 5, x != 5, x > 3);    // 1 0 1
    bool adult = x >= 18;
    printf("%d\n", adult);                           // 0
    return 0;
}
```

In C, **0 means false and any non-zero value means true**. The `bool` type just makes the meaning clearer.

### The classic trap: `=` versus `==`

```c norun
int x = 5;
if (x = 0) {           // meant to check whether x equals 0, but wrote an assignment!
    printf("x is 0\n");
}
```

`x = 0` is an **assignment**: it stores 0 in `x`, and the whole expression has the value 0 (false), so the `if` never runs — and `x` has been changed. With `-Wall`, the compiler warns:

```text
warning: suggest parentheses around assignment used as truth value [-Wparentheses]
```

## 3.6 Logical operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `&&` | and (true only if both sides are true) | `age >= 6 && age <= 12` |
| `\|\|` | or (true if either side is true) | `day == 6 \|\| day == 7` |
| `!` | not (negation) | `!finished` |

```c
#include <stdio.h>

int main(void) {
    int age = 10, day = 6;
    printf("%d\n", age >= 6 && age <= 12);     // 1: primary-school age
    printf("%d\n", day == 6 || day == 7);      // 1: a weekend
    printf("%d\n", !(age > 18));               // 1

    int count = 0;
    if (count != 0 && 100 / count > 5) {      // short-circuit: when count is 0, the right part is never evaluated
        printf("the average is above 5\n");
    }
    printf("no division by 0\n");
    return 0;
}
```

**Short-circuit evaluation**: if the left side of `&&` is false, the result must be false, so the right side **isn't evaluated**; if the left side of `||` is true, the right side isn't evaluated either. The example uses this to avoid dividing by 0.

> [!NOTE]
> You can't write `6 <= age <= 12` as in maths! C computes `6 <= age` first (giving 0 or 1) and then compares that 0 or 1 with 12, which is almost always true. Write `age >= 6 && age <= 12`.

## 3.7 Maths functions: `<math.h>`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    printf("%.4f\n", sqrt(2.0));        // square root
    printf("%.1f\n", pow(2, 10));       // 2 to the power 10: 1024.0
    printf("%.1f\n", fabs(-3.5));       // absolute value (fabs for decimals; abs for integers, in <stdlib.h>)
    printf("%.1f %.1f %.1f\n", round(2.5), floor(-2.5), ceil(2.1));   // round, round down, round up
    double pi = acos(-1.0);             // standard C has no π constant; this computes it
    printf("%.6f\n", pi);
    printf("%.4f\n", sin(pi / 6));      // trigonometry uses radians: sin(30°) = 0.5
    return 0;
}
```

> [!TIP]
> When compiling a program that uses `<math.h>` with gcc on Linux, add `-lm` (link the maths library) at the end: `gcc main.c -o main -lm`. The learning app adds it automatically.

## 3.8 Decimals are inexact: `0.1 + 0.2 != 0.3`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    double s = 0.1 + 0.2;
    printf("%.17f\n", s);                       // 0.30000000000000004
    printf("%d\n", s == 0.3);                   // 0: not equal!
    printf("%d\n", fabs(s - 0.3) < 1e-9);       // 1: the difference is tiny, so treat them as equal
    return 0;
}
```

Computers store decimals in binary, and 0.1 is an infinitely repeating fraction in binary, so only an approximation can be stored (just as 1/3 can only be written 0.333… in decimal). Therefore:

- **don't compare two decimals with `==`**; check whether their difference is small enough;
- **don't store money as decimals**: store whole cents as integers, e.g. 12.50 becomes 1250.

## 3.9 Bitwise operators (a first look)

C can work directly on the individual **binary bits** of integers. This is common in embedded systems, network protocols and image processing:

```c
#include <stdio.h>

int main(void) {
    int a = 13;   // binary 1101
    int b = 6;    // binary 0110
    printf("%d\n", a & b);    // bitwise AND: 0100 = 4
    printf("%d\n", a | b);    // bitwise OR:  1111 = 15
    printf("%d\n", a ^ b);    // bitwise XOR: 1011 = 11
    printf("%d\n", 1 << 5);   // shift left 5: 100000 = 32, i.e. multiply by 2⁵
    printf("%d\n", 40 >> 2);  // shift right 2: i.e. divide by 4 → 10
    printf("%d\n", (a >> 2) & 1);   // bit number 2 of a (counting from 0): 1
    return 0;
}
```

Don't confuse `&` (bitwise AND) with `&&` (logical AND), or `|` with `||`. This curriculum's "Assembly" course goes deeper into binary.

## 3.10 Summary

- `+ - * / %`; **integer divided by integer gives an integer** (truncated towards 0); `%` takes the sign of the dividend.
- For a decimal result: `(double)a / b`; converting a decimal to an integer truncates.
- Precedence: multiplication and division before addition and subtraction; add parentheses when unsure.
- `+=`, `-=`, `*=`…; `++`, `--` (prefix: change then use; postfix: use then change); never modify one variable twice in one expression.
- Comparisons `== != < > <= >=` give 1 or 0; **`=` assigns, `==` compares**; 0 is false, non-zero is true.
- Logical `&& || !` short-circuit; range checks must be written `a <= x && x <= b`.
- `<math.h>`: `sqrt`, `pow`, `fabs`, `round`, `floor`, `ceil`, trigonometry (add `-lm` on Linux).
- Decimals have errors: compare with `fabs(a - b) < 1e-9`; store money as integers.
- Bitwise `& | ^ ~ << >>` work directly on binary bits.

## 3.11 Exercises

<!-- exercise: divide -->
### Exercise 1: Integer and decimal division

The program already has `int a = 17, b = 5;`. Print three numbers separated by spaces: the integer quotient, the remainder, and the exact quotient (2 decimal places).

```text
Sample output: 3 2 3.40
```

<details>
<summary>Hint</summary>

`printf("%d %d %.2f\n", a / b, a % b, (double)a / b);`
</details>

<!-- exercise: clock -->
### Exercise 2: Converting seconds

The program already has `int total = 3725;` (seconds). Use `/` and `%` to print it as "hours:minutes:seconds", with two digits for minutes and seconds.

```text
Sample output: 1:02:05
```

<details>
<summary>Hint</summary>

Hours = `total / 3600`; minutes = `total % 3600 / 60`; seconds = `total % 60`. Print with `%d:%02d:%02d`.
</details>

<!-- exercise: circle -->
### Exercise 3: A circle

The program already has `double r = 2.5;`. Print the circle's area and circumference with 2 decimal places, separated by a space. Compute π as `acos(-1.0)`.

```text
Sample output: 19.63 15.71
```

<details>
<summary>Hint</summary>

`#include <math.h>`, `double pi = acos(-1.0);`, area `pi * r * r`, circumference `2 * pi * r`.
</details>

<!-- exercise: bits -->
### Exercise 4: Bitwise operations

Print the values of `13 & 6`, `13 | 6`, `13 ^ 6` and `1 << 5`, separated by spaces.

```text
Sample output: 4 15 11 32
```

<details>
<summary>Hint</summary>

Put them straight into the `printf` arguments; no extra variables needed.
</details>

<!-- exercise: ops-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

int main(void) {
    printf("%d %d %d\n", 7 / 2, -7 / 2, 7 % -3);
    printf("%.1f %d\n", 7 / 2 * 1.0, (int)3.99);
    int a = 5;
    int b = a++;
    printf("%d %d\n", a, b);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

Line one: `7 / 2` is 3; `-7 / 2` truncates towards 0 to -3; `7 % -3` takes the sign of the dividend 7, giving 1. Line two: `7 / 2` is computed as the integer 3 first, then times 1.0 gives 3.0; `(int)3.99` truncates to 3. Line three: `b = a++` gives b the 5, then makes a 6. So the output is `3 -3 1`, `3.0 3`, `6 5`.
</details>

<!-- exercise: assign-trap -->
### Exercise 6: `=` or `==`?

What happens after `int x = 5; if (x = 0) { ... }`?

- the code inside the `if` runs and `x` is still 5
- the code inside the `if` doesn't run, and `x` has become 0
- it fails to compile
- the code inside the `if` runs and `x` has become 0

<details>
<summary>Answer and explanation</summary>

`x = 0` is an assignment: `x` becomes 0 and the expression's value is 0 (false), so the code inside the `if` doesn't run. To compare, write `x == 0`.
</details>

## 3.12 Further reading

- [cppreference: C operator precedence](https://en.cppreference.com/w/c/language/operator_precedence)
- [cppreference: common mathematical functions](https://en.cppreference.com/w/c/numeric/math)
- [0.30000000000000004.com](https://0.30000000000000004.com/): floating-point errors in every language
- Next lesson: [Input: scanf](../04-input/en.md)
