# 6. Loops

> 🎯 **Goals**: repeat code with `while`, `do … while` and `for`; end a loop early with `break` and skip one round with `continue`; write nested loops (times tables, patterns); master common patterns — summing, counting, finding the maximum, taking numbers apart digit by digit; learn to spot "infinite loops" and "off-by-one errors".
>
> ⏱ **Time**: 75 minutes　|　📋 **You need**: lessons 3–5 of this course

## 6.1 The while loop

```c
#include <stdio.h>

int main(void) {
    int i = 1;
    while (i <= 5) {             // while the condition is true, run the body, then check again
        printf("round %d\n", i);
        i++;                     // don't forget to change i, or the condition stays true forever
    }
    printf("after the loop i = %d\n", i);
    return 0;
}
```

How it runs: check the condition → if true, run the code in the braces → go back and check again… until the condition is false and the loop ends.

**Infinite loops**: if the condition is always true (say you forgot `i++`), the loop never ends. In a terminal, **Ctrl+C** forces the program to stop; in the learning app, click "Stop".

## 6.2 The for loop

"Count from here to there" is the most common loop, and `for` puts its three parts on one line:

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 5; i++) {
        printf("%d ", i);
    }
    printf("\n");

    for (int i = 10; i > 0; i -= 3) {    // count down, 3 at a time
        printf("%d ", i);
    }
    printf("\n");
    return 0;
}
```

```text
for (initialisation; condition; done after each round) {
    body
}
```

It is exactly equivalent to this `while`:

```c norun
int i = 1;            // initialisation (done once)
while (i <= 5) {      // condition
    printf("%d ", i); // body
    i++;              // done after each round
}
```

A variable declared inside the `for` parentheses (`int i`) exists only within the loop and disappears when it ends.

## 6.3 do … while: runs at least once

`do … while` runs the body first and checks the condition **afterwards**, so the body always runs at least once. Perfect for "keep asking until the input is valid":

```c input="-3\n150\n42"
#include <stdio.h>

int main(void) {
    int age = 0;
    do {
        printf("Enter your age (0-120): ");
        scanf("%d", &age);
    } while (age < 0 || age > 120);       // note the semicolon at the end
    printf("\nage is %d\n", age);
    return 0;
}
```

## 6.4 break and continue

- **`break`**: leave the whole loop immediately;
- **`continue`**: skip the rest of this round and start the next.

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 10; i++) {
        if (i % 3 == 0) {
            continue;              // skip multiples of 3
        }
        if (i > 8) {
            break;                 // stop once past 8
        }
        printf("%d ", i);
    }
    printf("\n");                  // prints: 1 2 4 5 7 8
    return 0;
}
```

A common style is "loop forever, and `break` when a condition is met":

```c input="5 12 -1 7"
#include <stdio.h>

int main(void) {
    int sum = 0, x = 0;
    while (true) {
        scanf("%d", &x);
        if (x < 0) {
            break;                 // stop at the first negative number
        }
        sum += x;
    }
    printf("the sum is %d\n", sum);  // 17
    return 0;
}
```

## 6.5 Common loop patterns

**Summing and counting**:

```c
#include <stdio.h>

int main(void) {
    int sum = 0, count = 0;
    for (int i = 1; i <= 100; i++) {
        if (i % 7 == 0) {
            sum += i;              // accumulate
            count++;               // count
        }
    }
    printf("there are %d multiples of 7 in 1-100; their sum is %d\n", count, sum);
    return 0;
}
```

**Taking an integer apart digit by digit**: `% 10` takes the last digit, `/ 10` removes it:

```c
#include <stdio.h>

int main(void) {
    int n = 2025;
    int digits = 0, digit_sum = 0, reversed = 0;
    while (n > 0) {
        int last = n % 10;         // the last digit
        digit_sum += last;
        reversed = reversed * 10 + last;
        n /= 10;                   // remove the last digit
        digits++;
    }
    printf("%d digits, digit sum %d, reversed %d\n", digits, digit_sum, reversed);
    return 0;
}
```

**Checking for a prime**:

```c
#include <stdio.h>

int main(void) {
    int n = 97;
    bool prime = n >= 2;
    for (int d = 2; d * d <= n; d++) {    // only need to check up to √n
        if (n % d == 0) {
            prime = false;
            break;                         // one divisor is enough
        }
    }
    printf("%d is %sprime\n", n, prime ? "" : "not ");
    return 0;
}
```

## 6.6 Nested loops

A loop can contain another loop. Each time the outer loop goes round once, the inner loop runs all the way through. A times table:

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 9; i++) {
        for (int j = 1; j <= i; j++) {
            printf("%dx%d=%-3d", j, i, i * j);
        }
        printf("\n");
    }
    return 0;
}
```

Drawing a pattern with loops:

```c
#include <stdio.h>

int main(void) {
    int n = 4;
    for (int row = 1; row <= n; row++) {
        for (int s = 0; s < n - row; s++) {
            printf(" ");                  // spaces first
        }
        for (int k = 0; k < 2 * row - 1; k++) {
            printf("*");                  // then stars
        }
        printf("\n");
    }
    return 0;
}
```

> [!TIP]
> The trick to patterns: work out "how many spaces and how many stars are on row `row`" as formulas in `row` (such as `n - row` and `2 * row - 1`), then print them with loops.

Note: `break` only leaves **the loop it's in**, not the outer ones.

## 6.7 Off-by-one errors

Running a loop one time too many or too few is the most common bug of all, the **off-by-one error**:

```c norun
for (int i = 0; i <= 10; i++) { ... }   // runs 11 times (0 to 10)
for (int i = 0; i < 10; i++) { ... }    // runs 10 times (0 to 9) ← the most common C idiom
for (int i = 1; i <= 10; i++) { ... }   // runs 10 times (1 to 10)
```

When unsure, walk through it on paper with tiny numbers (1, 2, 3), or `printf` the value of `i` inside the loop.

## 6.8 Summary

- `while (condition) { ... }` checks first; `do { ... } while (condition);` runs at least once.
- `for (init; condition; update) { ... }` suits "count from here to there"; variables declared in a `for` exist only in the loop.
- `break` leaves the current loop, `continue` jumps to the next round; `while (true) { ... if (...) break; }` is a common style.
- Common patterns: accumulating, counting, finding the maximum, `% 10` and `/ 10` to take digits apart, `d * d <= n` to test for primes.
- Nested loops: each step of the outer loop runs the inner loop completely; for patterns, find the rule for each row first.
- Watch out for infinite loops (Ctrl+C stops them) and off-by-one errors (`<` or `<=`?).

## 6.9 Exercises

<!-- exercise: sum-n -->
### Exercise 1: Adding 1 to n

Read a positive integer `n` and print the sum `1 + 2 + … + n`. (`n` is at most 100000, so the result may not fit in an `int`.)

```text
Sample input: 100
Sample output: 5050
```

<details>
<summary>Hint</summary>

Accumulate in `long long sum = 0;` and print with `%lld`.
</details>

<!-- exercise: times-table -->
### Exercise 2: One row of the times table

Read an integer `n` and print `n × 1` to `n × 9`, one per line, in the format `n x i = product`.

```text
Sample input: 3
Sample output:
3 x 1 = 3
3 x 2 = 6
3 x 3 = 9
3 x 4 = 12
3 x 5 = 15
3 x 6 = 18
3 x 7 = 21
3 x 8 = 24
3 x 9 = 27
```

<details>
<summary>Hint</summary>

`for (int i = 1; i <= 9; i++) printf("%d x %d = %d\n", n, i, n * i);`
</details>

<!-- exercise: digits -->
### Exercise 3: Digits

Read a non-negative integer and print how many digits it has and the sum of its digits, separated by a space. (0 counts as 1 digit.)

```text
Sample input: 2025
Sample output: 4 9
```

<details>
<summary>Hint</summary>

Use `do { ... } while (n > 0);` so that 0 also counts as 1 digit.
</details>

<!-- exercise: primes -->
### Exercise 4: A list of primes

Read an integer `n` (`n ≥ 2`) and print every prime from 2 to `n`, separated by spaces.

```text
Sample input: 20
Sample output: 2 3 5 7 11 13 17 19
```

<details>
<summary>Hint</summary>

Loop `k` from 2 to `n` and test each `k` with the method from section 6.5. Make sure there's no space before the first number: a variable can remember whether it's the first one.
</details>

<!-- exercise: pyramid -->
### Exercise 5: A pyramid

Read `n` and print an `n`-row pyramid of stars (row `row` has `n - row` spaces followed by `2 × row - 1` stars).

```text
Sample input: 3
Sample output:
  *
 ***
*****
```

<details>
<summary>Hint</summary>

Follow the program in section 6.6, reading `n` with `scanf`.
</details>

<!-- exercise: loop-predict -->
### Exercise 6: Predict the output

```c
#include <stdio.h>

int main(void) {
    int i = 0;
    while (i < 10) {
        i += 3;
        if (i == 6) {
            continue;
        }
        printf("%d ", i);
    }
    printf("\n");
    for (int j = 5; j > 0; j--) {
        if (j == 2) break;
        printf("%d", j);
    }
    printf("\n");
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`while`: i becomes 3 (printed), 6 (`continue` skips printing), 9 (printed), 12 (printed, then 12 < 10 is false and the loop ends). Line one is `3 9 12 `. `for`: j = 5, 4, 3 are printed, and at j = 2 it `break`s. Line two is `543`.
</details>

<!-- exercise: off-by-one -->
### Exercise 7: How many rounds?

How many times does the body of `for (int i = 0; i <= 10; i++)` run?

- 9
- 10
- 11
- forever

<details>
<summary>Answer and explanation</summary>

i takes the values 0, 1, 2, …, 10 — 11 values in all. For 10 rounds, the usual way is `i < 10`.
</details>

## 6.10 Further reading

- [cppreference: for loop](https://en.cppreference.com/w/c/language/for), [while loop](https://en.cppreference.com/w/c/language/while)
- Next lesson: [Functions](../07-functions/en.md)
