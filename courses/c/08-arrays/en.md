# 8. Arrays

> 🎯 **Goals**: store a group of values of the same type in an **array**; define and initialise arrays, access elements by index and walk through them with loops; understand that C **doesn't check bounds**, what that leads to, and how to catch out-of-bounds bugs with the "memory check"; master the everyday array algorithms (sum, maximum and minimum, search, reverse, copy, bubble sort); pass arrays to functions (together with their length) and see why functions can change them; use two-dimensional arrays.
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: lessons 1–7 of this course

## 8.1 Why arrays?

Storing the marks of 5 students in 5 variables `s1`, `s2`… is clumsy but possible; but 100 students? 1000? An **array** is "a row of boxes of the same type": one name plus an **index** (a number) gets you any box:

```c
#include <stdio.h>

int main(void) {
    int scores[5] = {90, 85, 77, 60, 95};   // 5 ints in a row

    printf("first: %d\n", scores[0]);       // indices start at 0
    printf("last: %d\n", scores[4]);        // the length is 5, so the last index is 4

    scores[2] = 80;                         // change one element
    for (int i = 0; i < 5; i++) {           // handle every element with a loop
        printf("scores[%d] = %d\n", i, scores[i]);
    }
    return 0;
}
```

```text
index:      0    1    2    3    4
         ┌────┬────┬────┬────┬────┐
scores   │ 90 │ 85 │ 80 │ 60 │ 95 │
         └────┴────┴────┴────┴────┘
```

- Definition: `type name[length];`, e.g. `double prices[100];`, `char letters[26];`;
- indices start at **0**: an array of length n has indices **0 to n-1**;
- each element `scores[i]` works just like an ordinary variable: you can assign to it, compute with it and pass it to `printf`;
- the index can be any integer expression: `scores[i]`, `scores[i + 1]`, `scores[n - 1]`;
- the elements sit **right next to each other** in memory, which makes indexing very fast.

`for (int i = 0; i < n; i++)` is the most common loop in C (lesson 6) precisely because it visits every index of an array of length n.

## 8.2 Initialising arrays

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};        // every value given
    int b[5] = {1, 2};                 // only the first two; the rest become 0
    int c[5] = {0};                    // the usual way to zero everything
    int d[] = {7, 8, 9};               // no length: the compiler counts 3
    int e[5] = {[1] = 10, [3] = 30};   // initialise chosen indices; the rest are 0

    printf(" a  b  c  e\n");
    for (int i = 0; i < 5; i++) {
        printf("%2d %2d %2d %2d\n", a[i], b[i], c[i], e[i]);
    }
    printf("d has %zu elements\n", sizeof d / sizeof d[0]);
    return 0;
}
```

- `sizeof d` is the size of the whole array in bytes (3 × 4 = 12) and `sizeof d[0]` the size of one element (4); dividing gives the **number of elements**. This trick only works **in the function where the array is defined** (section 8.6 shows why).
- **A local array that isn't initialised is full of garbage**, just like an uninitialised variable (lesson 2). Writing `int a[100] = {0};` is a good habit.
- It's best to give the length a **name**, so changing it means changing one place:

```c
#include <stdio.h>

#define N 5                        // the preprocessor replaces every N below with 5

int main(void) {
    int squares[N];
    for (int i = 0; i < N; i++) {
        squares[i] = i * i;
    }
    for (int i = 0; i < N; i++) {
        printf("%d ", squares[i]);
    }
    printf("\n");
    return 0;
}
```

`#define N 5` is a **macro**: before compiling, the preprocessor replaces every `N` in the code with `5`. By convention, macro names are in capitals. (C23 also allows `constexpr int N = 5;`, but older compilers don't support it.)

## 8.3 C doesn't check bounds!

This is one of the most important differences between C and Python. The loop below goes one step too far and reads `a[5]`, which doesn't exist:

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};
    for (int i = 0; i <= 5; i++) {     // wrong: i reaches 5
        printf("%d ", a[i]);
    }
    printf("\n");
    return 0;
}
```

Python would immediately raise an `IndexError`; C **says nothing**: it simply reads the memory after the array and prints some meaningless number (which may differ from run to run). This is **undefined behaviour** — anything can happen:

- you read a garbage value, and the program "seems fine" while giving wrong results;
- you **write** past the array and silently change some other variable; the program may be stopped with `*** stack smashing detected ***`;
- you touch memory that doesn't belong to the program, and it crashes with `Segmentation fault`.

Many famous security holes ("buffer overflows") come from exactly this. So:

- write loops as `i < n`, not `i <= n`;
- check any index that comes from input before using it: `if (k >= 0 && k < N) { ... }`;
- let the **memory check** catch out-of-bounds accesses for you.

### Turning on the memory check

GCC and Clang come with **AddressSanitizer**. When compiling in a terminal, add `-fsanitize=address,undefined -g`:

```bash
gcc -fsanitize=address,undefined -g main.c -o main
./main
```

In this tutorial's desktop app, open **Tools → Settings…** and tick "**C/C++ memory check (AddressSanitizer)**" (MinGW on Windows doesn't support it; the app then skips it automatically). Run the example above again, and the program stops at the moment of the out-of-bounds access and tells you the line:

```text
main.c:6:24: runtime error: index 5 out of bounds for type 'int [5]'
```

> [!TIP]
> When writing C, **keep the memory check on** all the time. Programs run a bit slower, but memory bugs that "happen now and then and are hard to reproduce" turn into "an error right away, with a line number". It becomes even more useful in lessons 10 and 11, with pointers and dynamic memory.

## 8.4 Reading a list of values

Many problems give their input as "first a count n, then n numbers". The simplest approach is an array that's **big enough**:

```c input="5\n3 8 1 9 4"
#include <stdio.h>

#define MAX_N 1000                     // supports at most 1000 numbers

int main(void) {
    int a[MAX_N];
    int n = 0;
    scanf("%d", &n);
    if (n < 1 || n > MAX_N) {          // avoid going out of bounds: check n first
        printf("n must be between 1 and %d\n", MAX_N);
        return 1;
    }
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);            // a[i] is just an int variable, so it needs & too
    }

    long long sum = 0;
    int max_i = 0;                     // remember the *index* of the maximum, not just its value
    for (int i = 0; i < n; i++) {
        sum += a[i];
        if (a[i] > a[max_i]) {
            max_i = i;
        }
    }
    printf("sum %lld, average %.2f\n", sum, (double)sum / n);
    printf("the largest is number %d: %d\n", max_i + 1, a[max_i]);
    return 0;
}
```

Note: the array's own size `MAX_N` is fixed; `n` just says "the first n boxes are in use".

> [!NOTE]
> You may see code that reads n and then writes `int a[n];`. That's a **variable-length array** (VLA). GCC and Clang support it, but it's an "optional feature" of the C standard that Microsoft's compiler doesn't have, and a large n can blow the stack (lesson 7). This course uses a "fixed maximum length"; lesson 11 introduces `malloc`, which allocates exactly as much memory as needed.

## 8.5 Everyday array algorithms

```c
#include <stdio.h>

#define N 6

int main(void) {
    int a[N] = {4, 8, 15, 16, 23, 42};

    // linear search: compare one by one from start to end
    int target = 16, pos = -1;         // -1 means "not found"
    for (int i = 0; i < N; i++) {
        if (a[i] == target) {
            pos = i;
            break;                     // stop at the first match
        }
    }
    printf("index of %d: %d\n", target, pos);

    // reverse in place: swap a[i] and a[N-1-i], only going halfway
    for (int i = 0; i < N / 2; i++) {
        int t = a[i];
        a[i] = a[N - 1 - i];
        a[N - 1 - i] = t;
    }

    // copy: only element by element
    int b[N];
    for (int i = 0; i < N; i++) {
        b[i] = a[i];
    }

    for (int i = 0; i < N; i++) {
        printf("%d ", b[i]);
    }
    printf("\n");
    return 0;
}
```

- If the reversing loop went through the whole array (`i < N`), each pair would be swapped twice and everything would end up back where it started.
- Arrays **can't be assigned as a whole**: `b = a;` gives the error `assignment to expression with array type`, and `==` can't compare the contents of two arrays either. Copy with a loop, or with `memcpy(b, a, sizeof a);` from `<string.h>`.

## 8.6 Arrays and functions

When you pass an array to a function, **pass its length too**:

```c
#include <stdio.h>

void print_array(const int a[], int n) {   // const: promises not to change the array
    for (int i = 0; i < n; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void double_all(int a[], int n) {          // no const: it changes the array
    for (int i = 0; i < n; i++) {
        a[i] *= 2;
    }
}

int sum_array(const int a[], int n) {
    int s = 0;
    for (int i = 0; i < n; i++) {
        s += a[i];
    }
    return s;
}

int main(void) {
    int nums[] = {1, 2, 3, 4};
    int n = sizeof nums / sizeof nums[0];  // the length can only be worked out like this here
    print_array(nums, n);
    double_all(nums, n);
    print_array(nums, n);                  // 2 4 6 8: main's array really was changed!
    printf("sum: %d\n", sum_array(nums, n));
    return 0;
}
```

Two things differ from lesson 7's "pass by value":

1. **The function can change the caller's array.** C doesn't copy the whole array (it could be huge); it only passes "where the array is" — the **address** of its first element. So `double_all` changes `main`'s `nums` itself. If a function shouldn't change the array, put `const` before the parameter: an accidental change then becomes a compile error.
2. **The function doesn't know how long the array is.** The parameter `int a[]` is really just an address (lesson 10 calls it a "pointer"), so `sizeof a` gives the size of an address (usually 8), not of the array. GCC warns:

```text
warning: 'sizeof' on array function parameter 'a' will return size of 'int *'
```

That's why the length must always be passed as a separate parameter. Also, a function **can't return an array**; to "produce" a list of values, let the caller prepare an array and pass it in for the function to fill (like `double_all`).

## 8.7 Sorting: bubble sort

**Sorting** is the classic algorithm problem. The idea of **bubble sort**: go from left to right comparing neighbours, swapping them if the left one is bigger. After one pass, the biggest number has "bubbled up" to the right end; repeat for the rest, until a pass makes no swaps at all:

```c
#include <stdio.h>

void print_array(const int a[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void bubble_sort(int a[], int n) {
    for (int pass = 0; pass < n - 1; pass++) {
        bool swapped = false;
        for (int i = 0; i < n - 1 - pass; i++) {   // the last `pass` numbers are already in place
            if (a[i] > a[i + 1]) {
                int t = a[i];
                a[i] = a[i + 1];
                a[i + 1] = t;
                swapped = true;
            }
        }
        printf("after pass %d: ", pass + 1);
        print_array(a, n);
        if (!swapped) {
            break;                                 // no swaps: already sorted
        }
    }
}

int main(void) {
    int a[] = {5, 1, 4, 2, 8};
    bubble_sort(a, 5);
    return 0;
}
```

- The inner loop stops at `n - 1 - pass`: since it reads `a[i + 1]`, `i` may go no further than the second-to-last element, or it would go out of bounds; and the last `pass` numbers are already in their final places.
- Bubble sort makes about n²/2 comparisons: about 500,000 for 1000 numbers, which is still quick; but about 500 billion for a million numbers, far too slow. Real programs use the standard library's much faster `qsort` — we'll use it once we've learned pointers (lesson 10).

## 8.8 Two-dimensional arrays

Tables, game boards, images and matrices are all "rows × columns" data, stored in **two-dimensional arrays**:

```c
#include <stdio.h>

#define ROWS 3
#define COLS 4

int main(void) {
    int m[ROWS][COLS] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12},
    };
    m[1][2] = 70;                      // row 1, column 2 (both counted from 0)

    for (int r = 0; r < ROWS; r++) {
        int row_sum = 0;
        for (int c = 0; c < COLS; c++) {
            printf("%4d", m[r][c]);
            row_sum += m[r][c];
        }
        printf("  | sum %d\n", row_sum);
    }
    return 0;
}
```

- `m[r][c]`: row first, then column. `int m[3][4]` can be seen as "an array of 3 elements, each of which is an array of 4 ints".
- In memory, a 2D array is stored **one row after another**: `m[0][0] m[0][1] m[0][2] m[0][3] m[1][0] …`.
- Walk through it with two loops: the outer one over rows, the inner one over columns (nested loops, lesson 6).

When passing a 2D array to a function, the compiler must know the **number of columns** (it needs it to work out where `m[r][c]` is). You can pass the numbers of rows and columns as parameters, written before the array parameter:

```c
#include <stdio.h>

void print_matrix(int rows, int cols, const int m[rows][cols]) {
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            printf("%3d", m[r][c]);
        }
        printf("\n");
    }
}

int main(void) {
    int table[3][3] = {0};
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            table[r][c] = (r + 1) * (c + 1);   // a corner of the times table
        }
    }
    print_matrix(3, 3, table);
    return 0;
}
```

(Using parameters as lengths, as in `const int m[rows][cols]`, needs GCC or Clang; you can also fix the number of columns: `const int m[][3]`.)

## 8.9 Summary

- Arrays: `type name[length]`, indices **0 to n-1**; `int a[5] = {0};` zeroes everything, `int d[] = {...}` lets the compiler count.
- `sizeof a / sizeof a[0]` gives the number of elements, only in the function that defines the array; name lengths with constants like `#define N 5`.
- **C doesn't check bounds**: going out of bounds is undefined behaviour — garbage values, silently corrupted data or a crash. Loop with `i < n`, and turn on the memory check (`-fsanitize=address,undefined`).
- "Read n, then n numbers": use a fixed maximum length and check n.
- Everyday algorithms: sum, maximum and minimum (remember the index), linear search (return -1 if not found), reverse (go halfway), copy (element by element), bubble sort.
- Arrays can't be assigned with `=` or compared with `==`.
- Passing arrays to functions: pass the length too; the function **can change** the original array, so add `const` if it shouldn't; inside the function `sizeof` can't tell the array's length.
- 2D arrays `m[row][column]` are stored row by row and walked with two loops; functions must be told the number of columns.

## 8.10 Exercises

<!-- exercise: reverse-array -->
### Exercise 1: Backwards

Read a positive integer `n` (at most 1000), then `n` integers, and print them in reverse order, separated by spaces.

```text
Sample input:
5
1 2 3 4 5
Sample output: 5 4 3 2 1
```

<details>
<summary>Hint</summary>

Read everything into an array first, then print with `for (int i = n - 1; i >= 0; i--)`. Make sure there's no space before the first number: for example, print a space first whenever `i < n - 1`.
</details>

<!-- exercise: array-stats -->
### Exercise 2: Maximum, minimum and average

Read `n` (1 ≤ n ≤ 1000) and `n` integers, and print the maximum, the minimum and the average (2 decimal places), separated by spaces.

```text
Sample input:
4
3 -2 10 5
Sample output: 10 -2 4.00
```

<details>
<summary>Hint</summary>

Start both the maximum and the minimum at `a[0]`, not 0 — otherwise all-negative (or all-positive) input goes wrong. For the average, use `(double)sum / n`.
</details>

<!-- exercise: above-average -->
### Exercise 3: Above average

Read `n` (1 ≤ n ≤ 1000) and `n` integers, and print how many of them are **strictly greater** than the average.

```text
Sample input:
5
1 2 3 4 10
Sample output: 1
```

<details>
<summary>Hint</summary>

You need the average before you can compare, so all the numbers must be stored in an array. To avoid decimals, rewrite `a[i] > sum / n` as `a[i] * n > sum` (using `long long`).
</details>

<!-- exercise: sort-numbers -->
### Exercise 4: Sorting

Read `n` (1 ≤ n ≤ 1000) and `n` integers, and print them sorted from smallest to largest, separated by spaces.

```text
Sample input:
6
5 3 9 1 3 7
Sample output: 1 3 3 5 7 9
```

<details>
<summary>Hint</summary>

Write `bubble_sort` as in section 8.7 (without the printing after each pass).
</details>

<!-- exercise: transpose -->
### Exercise 5: Transposing a matrix

Read the number of rows `r` and columns `c` (both from 1 to 10), then `r` rows of `c` integers each. Print its **transpose**: row i becomes column i, so the output has `c` rows of `r` numbers each, separated by spaces.

```text
Sample input:
2 3
1 2 3
4 5 6
Sample output:
1 4
2 5
3 6
```

<details>
<summary>Hint</summary>

Read into `int m[10][10];`. When printing, let the outer loop go over the original columns `j` and the inner loop over the original rows `i`, printing `m[i][j]`.
</details>

<!-- exercise: array-predict -->
### Exercise 6: Predict the output

```c
#include <stdio.h>

void change(int x, int a[]) {
    x = 100;
    a[0] = 100;
}

int main(void) {
    int x = 1;
    int a[3] = {1, 2};
    change(x, a);
    printf("%d %d %d %d\n", x, a[0], a[1], a[2]);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`x` is passed by value, so the function changes a copy and the outer `x` is still 1; the array is passed as an address, so `a[0]` really does become 100; `a[2]` had no initial value given, so it is 0. The output is `1 100 2 0`.
</details>

<!-- exercise: array-index -->
### Exercise 7: Valid indices

Which indices are valid for `int a[10];`?

- 1 to 10
- 0 to 9
- 0 to 10
- any integer; C checks automatically

<details>
<summary>Answer and explanation</summary>

An array of length 10 has indices 0 to 9. C does **not** check bounds; accessing `a[10]` is undefined behaviour.
</details>

## 8.11 Further reading

- [cppreference: array declaration](https://en.cppreference.com/w/c/language/array), [array initialization](https://en.cppreference.com/w/c/language/array_initialization)
- [AddressSanitizer documentation](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- [VisuAlgo: sorting animations](https://visualgo.net/en/sorting) — watch bubble sort swap step by step
- Next lesson: [Strings](../09-strings/en.md)
