# 7. Functions

> 🎯 **Goals**: package a piece of work as a **function**, understanding parameters, return values and `void`; see why functions must be "declared before use" and write **function prototypes**; understand that C passes arguments **by value** (functions get copies); tell local variables, global variables and `static` local variables apart; write recursive functions (factorial, Towers of Hanoi) and learn about the **call stack** and stack overflow.
>
> ⏱ **Time**: 80 minutes　|　📋 **You need**: lessons 1–6 of this course

## 7.1 Defining and calling functions

We've been using functions written by others all along — `printf`, `scanf`, `sqrt`… and `main` is a function too. Now let's write our own:

```c
#include <stdio.h>

int square(int x) {          // return type  name(parameter type parameter name)
    return x * x;            // return: hand the result back to the caller and end the function
}

void greet(void) {           // void: returns nothing; void in the parameters: takes no arguments
    printf("Hello!\n");
}

int main(void) {
    greet();
    int a = square(5);       // call square, passing 5 as x, and get 25
    printf("%d\n", a);
    printf("%d\n", square(3) + square(4));   // return values can be used directly: 25
    return 0;
}
```

| Part | Meaning |
|------|---------|
| `int` | the **return type**: the function hands back an `int`. If it returns nothing, write `void` |
| `square` | the function's name; same naming rules as variables |
| `(int x)` | the **parameter list**: each parameter has a type; separate several with commas, e.g. `(int a, double b)` |
| `return x * x;` | return the result. A `void` function can use a bare `return;` to finish early, or omit it |

As in Python, functions avoid repeated code and make programs easier to read and test. Unlike Python, **parameters and return values must state their types**.

## 7.2 Declare before use: function prototypes

The C compiler reads code **from top to bottom**. If `main` calls a function written after `main`, the compiler doesn't know it yet when it reads the call:

```c norun
#include <stdio.h>

int main(void) {
    printf("%d\n", square(4));     // square is unknown here
    return 0;
}

int square(int x) {
    return x * x;
}
```

```text
warning: implicit declaration of function 'square' [-Wimplicit-function-declaration]
```

(Newer compilers report an error.) There are two fixes: move the definition before the call; or write a **function prototype** (declaration) earlier — just the "signature", no body, ending with a semicolon:

```c
#include <stdio.h>

int square(int x);                 // a prototype: tells the compiler "this function exists"
double average(int a, int b);

int main(void) {
    printf("%d %.1f\n", square(4), average(3, 4));
    return 0;
}

int square(int x) {                // the definitions can come afterwards
    return x * x;
}

double average(int a, int b) {
    return (a + b) / 2.0;          // divide by 2.0 for decimal division (lesson 3)
}
```

Many C programs are laid out like this: prototypes for every function first, then `main`, then the function definitions. Multi-file programs put the prototypes in **header files** (`.h`) — `#include <stdio.h>` actually includes the prototypes of `printf` and friends. Lesson 14 covers this.

## 7.3 Pass by value: functions get copies

When a function is called, the arguments' **values are copied** into the parameters. Changing a parameter inside the function doesn't affect the variable outside:

```c
#include <stdio.h>

void try_to_change(int n) {
    n = 100;                       // only the copy changes
    printf("inside the function n = %d\n", n);
}

void swap(int a, int b) {          // trying to swap two numbers…
    int t = a;
    a = b;
    b = t;
}

int main(void) {
    int x = 5;
    try_to_change(x);
    printf("outside x = %d\n", x); // still 5

    int p = 1, q = 2;
    swap(p, q);
    printf("p = %d, q = %d\n", p, q);   // still 1 and 2: the swap failed!
    return 0;
}
```

For a function to change variables outside it (a real `swap`) or to "return several values", you pass in the variables' **addresses** — exactly why `scanf("%d", &x)` needs the `&`. Once we've learned pointers in lesson 10, we'll write a `swap` that works.

## 7.4 Local, global and static variables

```c
#include <stdio.h>

int total_calls = 0;               // a global variable: outside every function, usable by all of them

int next_id(void) {
    static int id = 0;             // a static local variable: initialised once, keeps its value after the function ends
    id++;
    total_calls++;
    return id;
}

int main(void) {
    int a = next_id();
    int b = next_id();
    int c = next_id();
    printf("%d %d %d\n", a, b, c);          // 1 2 3
    printf("called %d times in total\n", total_calls);
    return 0;
}
```

| Kind | Written | Usable by | Lives for |
|------|---------|-----------|-----------|
| local variable | inside a function (or block) | that function only | until the function ends |
| `static` local variable | inside a function, with `static` | that function only | the whole run; the value is kept |
| global variable | outside every function | every function | the whole run |

> [!TIP]
> **Use global variables as little as possible.** Any function can change them, so in a big program it becomes very hard to tell who changed what, and when. Whatever can be passed through parameters and return values should be.

## 7.5 Recursion: a function calling itself

As in Python (Python course lesson 10), C functions can call themselves:

```c
#include <stdio.h>

long long factorial(int n) {
    if (n <= 1) {                  // the stopping condition
        return 1;
    }
    return n * factorial(n - 1);   // turn the problem into a smaller one of the same kind
}

int main(void) {
    for (int i = 0; i <= 20; i += 5) {
        printf("%d! = %lld\n", i, factorial(i));
    }
    return 0;
}
```

One of the most beautiful examples of recursion is the **Towers of Hanoi**: move n discs from peg A to peg C, one at a time, never putting a bigger disc on a smaller one. The idea: move the top n−1 discs to B, move the biggest to C, then move the n−1 discs from B to C — and "moving n−1 discs" is the same problem again:

```c
#include <stdio.h>

void hanoi(int n, char from, char via, char to) {
    if (n == 0) {
        return;
    }
    hanoi(n - 1, from, to, via);         // the top n-1: from → via
    printf("%c -> %c\n", from, to);      // the biggest: from → to
    hanoi(n - 1, via, from, to);         // those n-1: via → to
}

int main(void) {
    hanoi(3, 'A', 'B', 'C');             // 3 discs take 7 moves
    return 0;
}
```

### The call stack

Each time a function is called, the computer sets aside a block of memory on the **stack** holding that call's parameters, local variables and "where to go back to when finished"; the block is released when the function returns. While `factorial(4)` recurses, the stack holds four layers at once: `factorial(4)`, `factorial(3)`, `factorial(2)` and `factorial(1)`.

The stack is limited in size (usually a few MB). If a recursive function **forgets its stopping condition**, or recurses too deeply, the stack runs out — a **stack overflow** — and the program crashes, usually with `Segmentation fault`. The famous programmers' Q&A site Stack Overflow is named after it.

## 7.6 Habits of good functions

- **One function, one job**, with a name that says what it does: `is_prime`, `read_scores`, `print_report`;
- keep functions short; if one grows past a screen, consider splitting it;
- take data through parameters and hand results back through return values; rely little on globals;
- separate "computing" from "input and output": `int max3(int a, int b, int c)` only computes and leaves printing to `main` — such functions are the easiest to test and reuse.

## 7.7 Summary

- Function definition: `return_type name(type parameter, ...) { ... return value; }`; use `void` when nothing is returned and `(void)` for no parameters.
- C reads top to bottom: functions must be **declared before use**, e.g. with an earlier prototype `int square(int x);`.
- Arguments are **passed by value**: changing a parameter doesn't affect the outside variable; changing outside variables needs addresses (lesson 10).
- Local variables live only in their function; `static` locals keep their value between calls; use few globals.
- Recursion needs a stopping condition; each call takes stack space, and too deep a recursion overflows the stack.

## 7.8 Exercises

<!-- exercise: max3-c -->
### Exercise 1: The largest of three

Complete the function `int max3(int a, int b, int c)` so it returns the largest of the three numbers. `main` is already written: it reads three integers and prints the result of `max3`.

```text
Sample input: 3 9 4
Sample output: 9
```

<details>
<summary>Hint</summary>

`int m = a; if (b > m) m = b; if (c > m) m = c; return m;`
</details>

<!-- exercise: gcd -->
### Exercise 2: Greatest common divisor and least common multiple

Write a function `int gcd(int a, int b)` using **Euclid's algorithm**: while `b` isn't 0, keep replacing `(a, b)` with `(b, a % b)`; the final `a` is the answer. Read two positive integers and print their GCD and LCM, separated by a space.

```text
Sample input: 12 18
Sample output: 6 36
```

<details>
<summary>Hint</summary>

LCM = `a / gcd(a, b) * b` (divide before multiplying to avoid overflow). `long long` is safer.
</details>

<!-- exercise: count-primes -->
### Exercise 3: Primes in a range

Write a function `bool is_prime(int n)` and use it to count the primes in the closed range `[a, b]`. Read `a b` and print the count.

```text
Sample input: 10 30
Sample output: 6
```

<details>
<summary>Hint</summary>

In `is_prime`: return `false` below 2; `for (int d = 2; d * d <= n; d++)` return `false` if `d` divides `n`; return `true` at the end.
</details>

<!-- exercise: hanoi -->
### Exercise 4: Towers of Hanoi

Read the number of discs `n` (1 ≤ n ≤ 10) and, using the method from section 7.5, print every move needed to take them from `A` to `C` (via `B`), in the format `A -> C`.

```text
Sample input: 2
Sample output:
A -> B
A -> C
B -> C
```

<details>
<summary>Hint</summary>

Follow the `hanoi` function from section 7.5 and call `hanoi(n, 'A', 'B', 'C');` in `main`.
</details>

<!-- exercise: func-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

void swap(int a, int b) {
    int t = a;
    a = b;
    b = t;
}

int counter(void) {
    static int n = 0;
    n += 10;
    return n;
}

int main(void) {
    int x = 1, y = 2;
    swap(x, y);
    printf("%d %d\n", x, y);
    counter();
    counter();
    printf("%d\n", counter());
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`swap` only swaps copies, so `x` and `y` are unchanged: `1 2`. The `n` in `counter` is `static`, so each call adds 10 to the previous value: 20 after the first two calls, and the third returns 30. The output is `1 2` and `30`.
</details>

<!-- exercise: prototype -->
### Exercise 6: Why write prototypes?

- to make functions run faster
- the C compiler reads top to bottom, so before calling a function defined later, a prototype must tell it the function's name, parameters and return type
- a prototype is the function's definition, so no body is needed afterwards
- only `void` functions need prototypes

<details>
<summary>Answer and explanation</summary>

A prototype lets the compiler check the number and types of arguments as soon as it sees a call. Alternatively, put the function's definition before the call and no prototype is needed.
</details>

## 7.9 Further reading

- [cppreference: function declarations](https://en.cppreference.com/w/c/language/function_declaration), [storage duration and linkage (static)](https://en.cppreference.com/w/c/language/storage_duration)
- Next lesson: [Arrays](../08-arrays/en.md)
