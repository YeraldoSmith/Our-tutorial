# 11. Dynamic memory

> 🎯 **Goals**: understand the areas of a program's memory (stack, heap, globals) and when you need **dynamic memory**; request memory with `malloc`, give it back with `free`, and check for `NULL`; get zeroed memory with `calloc`, make arrays "grow" with `realloc`, and write a **dynamic array** that holds any amount of data; have functions return dynamically allocated memory and make clear "who frees it"; simulate a dynamic 2D array with a 1D one; recognise memory leaks, use after free, double free and similar bugs, and find them with the memory check.
>
> ⏱ **Time**: 100 minutes　|　📋 **You need**: lessons 1–10 of this course, especially lesson 10, "Pointers"

## 11.1 Why dynamic memory?

So far, all our arrays have had a **fixed size**:

```c norun
#define MAX_N 1000
int a[MAX_N];          // wasteful for 10 numbers, not enough for a million
```

And there are more problems:

- Local variables live on the **stack**, which is usually only a few MB. Put `int big[10000000];` (about 40 MB) inside a function and the program crashes as soon as it runs (stack overflow, lesson 7).
- Local variables disappear when their function returns, so a function can't return the address of a local array (lesson 10).
- Often you only learn how much data there is **while the program runs** — "read until the end of the input", for example.

The solution is to request memory **dynamically** from the **heap**: ask for exactly as much as you need, and give it back yourself when you're done. A program's memory is laid out roughly like this:

```text
┌──────────────────────────────┐
│ stack                        │ local variables, parameters: freed automatically on return; small (a few MB)
│              ↓               │
│                              │
│              ↑               │
│ heap                         │ from malloc: you request it and you free it; large (up to several GB)
├──────────────────────────────┤
│ global and static variables  │ exist for the whole run
├──────────────────────────────┤
│ program code, string literals│ read-only
└──────────────────────────────┘
```

## 11.2 `malloc` and `free`

```c input="5\n3 8 1 9 4"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n = 0;
    if (scanf("%d", &n) != 1 || n <= 0) {
        return 1;
    }

    int *a = malloc(n * sizeof *a);      // request room for n ints
    if (a == NULL) {                     // NULL when there isn't enough memory: always check
        printf("out of memory\n");
        return 1;
    }

    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);              // used exactly like an array
    }
    long long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
    printf("%d numbers, sum %lld\n", n, sum);

    free(a);                             // done: give it back to the system
    a = NULL;                            // good habit: prevents accidental later use
    return 0;
}
```

- **`malloc(bytes)`** requests a block of memory on the heap and returns its address (of type `void *`, which C converts to `int *` automatically — no cast needed).
- **`n * sizeof *a`**: `*a` is an `int`, so `sizeof *a` is the size of one element. The advantage: if you later change `a` to a `double *`, this line needn't change.
- Memory from `malloc` holds **garbage**, just like uninitialised local variables.
- When there isn't enough memory, `malloc` returns **`NULL`**, so check before using the result.
- **`free(pointer)`** gives the memory back. Every block from `malloc` must be `free`d, and **only once**. After `free` the pointer still exists, but you mustn't read or write through it; setting it to `NULL` is a good habit (`free(NULL)` does nothing, so it's safe).

## 11.3 `calloc` and `realloc`: arrays that grow

| Function | What it does |
|----------|--------------|
| `malloc(bytes)` | request memory; it holds garbage |
| `calloc(count, size of each)` | request memory and **zero all of it**, e.g. `calloc(n, sizeof *a)` |
| `realloc(p, new size in bytes)` | **resize** the memory `p` points to, keeping its contents; it may move to a new address, so use its return value |
| `free(p)` | give the memory back |

With `realloc` you can write a **dynamic array**: start with a little space, and **double** the capacity whenever it fills up. This program reads any number of numbers:

```c input="5 12 -3 8 0 7 21"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int capacity = 2;                         // start with room for 2
    int count = 0;                            // how many are in use
    int *data = malloc(capacity * sizeof *data);
    if (data == NULL) {
        return 1;
    }

    int x;
    while (scanf("%d", &x) == 1) {
        if (count == capacity) {              // full: double the capacity
            capacity *= 2;
            int *bigger = realloc(data, capacity * sizeof *data);
            if (bigger == NULL) {             // on failure the old data is still there: free it
                free(data);
                return 1;
            }
            data = bigger;
            printf("(capacity grown to %d)\n", capacity);
        }
        data[count++] = x;                    // store in data[count], then add 1 to count
    }

    printf("read %d numbers: ", count);
    for (int i = 0; i < count; i++) {
        printf("%d ", data[i]);
    }
    printf("\n");
    free(data);
    return 0;
}
```

- **Why the temporary variable `bigger`?** With `data = realloc(data, ...)`, a failure returns `NULL` and the address of the old block is lost, so it can never be `free`d — a memory leak.
- **Why double instead of adding 1 each time?** `realloc` sometimes has to move all the data somewhere new, which takes time. With doubling, a million numbers need only about 20 resizes. Python lists and C++'s `vector` do the same thing internally.

## 11.4 Returning dynamic memory from functions

A function's local variables vanish when it returns, but memory from `malloc` stays until it's `free`d. So a function can hand it back to its caller:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Returns a new string: s repeated `times` times.
// The caller must free the result; returns NULL on failure.
char *repeat(const char *s, int times) {
    size_t len = strlen(s);
    char *result = malloc(len * times + 1);       // +1 for the '\0'
    if (result == NULL) {
        return NULL;
    }
    for (int i = 0; i < times; i++) {
        memcpy(result + i * len, s, len);         // copy s's len bytes into the i-th slot
    }
    result[len * times] = '\0';
    return result;
}

// Returns a new array {0, 1, 4, 9, ...} of n squares. The caller must free it.
int *squares(int n) {
    int *a = malloc(n * sizeof *a);
    if (a == NULL) {
        return NULL;
    }
    for (int i = 0; i < n; i++) {
        a[i] = i * i;
    }
    return a;
}

int main(void) {
    char *text = repeat("ab", 3);
    if (text != NULL) {
        printf("%s\n", text);          // ababab
        free(text);
    }

    int *sq = squares(6);
    if (sq != NULL) {
        for (int i = 0; i < 6; i++) {
            printf("%d ", sq[i]);
        }
        printf("\n");
        free(sq);
    }
    return 0;
}
```

The key is making clear **who frees it** ("ownership"). By convention, the function's comment says "the caller must `free` the result". `memcpy(dest, src, bytes)` (in `<string.h>`) copies a block of memory byte by byte. By the way, C23's standard library also has `strdup(s)`, which returns a `malloc`ed copy of `s`.

## 11.5 Dynamic 2D arrays

The simplest and most portable approach: request one 1D block of `rows × cols` and work out the index yourself — row `r`, column `c` is at `r * cols + c`:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int rows = 3, cols = 4;
    int *grid = malloc(rows * cols * sizeof *grid);
    if (grid == NULL) {
        return 1;
    }
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            grid[r * cols + c] = r * 10 + c;    // row r, column c
        }
    }
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            printf("%3d", grid[r * cols + c]);
        }
        printf("\n");
    }
    free(grid);
    return 0;
}
```

It's the same idea as lesson 8's "a 2D array is stored one row after another", except that we compute the index ourselves. (GCC and Clang also support `int (*m)[cols] = malloc(rows * sizeof *m);`, after which you can write `m[r][c]` directly.)

## 11.6 Memory bugs, and how to find them

Bugs with dynamic memory are sneaky: the program often "seems fine" and only goes wrong mysteriously later. Turn on the **memory check** (Tools → Settings…, lesson 8) and most of them are caught on the spot:

| Bug | Example | Memory check report |
|-----|---------|---------------------|
| **memory leak**: forgetting `free` | `malloc` in a function, no `free` before returning | `LeakSanitizer: detected memory leaks` |
| **use after free** | `p[0] = 1;` after `free(p);` | `heap-use-after-free` |
| **double free** | the same pointer `free`d twice | `attempting double-free` |
| **out of bounds** | `malloc(4 * sizeof *p)` but accessing `p[4]` | `heap-buffer-overflow` |
| **freeing memory that didn't come from `malloc`** | `int x; free(&x);` | `attempting free on address which was not malloc()-ed` |

```c norun
void process(void) {
    int *tmp = malloc(1000 * sizeof *tmp);
    // ...use tmp...
    return;              // forgot free(tmp): each call loses about 4 KB
}
```

When a program ends, the operating system takes back all its memory, so a short program that leaks a little "seems OK". But long-running programs such as servers and games pile up leaks until memory runs out. Build good habits from the start:

- whenever you write a `malloc`, decide right away where the `free` goes;
- set pointers to `NULL` after `free`;
- store `realloc`'s result in a temporary variable first;
- keep the memory check on while writing C.

## 11.7 Summary

- Stack: local variables, freed automatically, small; heap: from `malloc`, freed by hand with `free`, large.
- `T *p = malloc(n * sizeof *p);`, check for `NULL`, and when done `free(p); p = NULL;`.
- `calloc` requests zeroed memory; `realloc` resizes (store the result in a temporary first); a dynamic array doubles its capacity when full.
- Functions can return `malloc`ed memory — say clearly who `free`s it.
- Dynamic 2D arrays: one `rows * cols` block, indexed with `r * cols + c`.
- Common bugs: leaks, use after free, double free, out of bounds, freeing non-heap memory — find them with the memory check.

## 11.8 Exercises

<!-- exercise: reverse-unknown -->
### Exercise 1: No idea how many

Read integers (at least 1; how many isn't known in advance and could be a million) until the end of the input, and print them in reverse order, separated by spaces.

```text
Sample input: 1 2 3 4 5
Sample output: 5 4 3 2 1
```

<details>
<summary>Hint</summary>

Write a doubling dynamic array as in section 11.3, then print backwards from `count - 1`. Don't forget to `free`.
</details>

<!-- exercise: merge-sorted -->
### Exercise 2: Merging two sorted arrays

The first line has `n` followed by `n` integers in ascending order; the second line has `m` followed by `m` integers in ascending order (1 ≤ n, m ≤ 100000). Merge them into one ascending array and print it, separated by spaces. All three arrays must come from `malloc`.

```text
Sample input:
3 1 4 9
4 2 3 10 12
Sample output: 1 2 3 4 9 10 12
```

<details>
<summary>Hint</summary>

Keep two indices `i` and `j`, one into each array; each time, put the smaller of the two elements into the result and move that index on. When one array runs out, copy the rest of the other.
</details>

<!-- exercise: lines-reverse -->
### Exercise 3: Lines in reverse

Read lines of text (each at most 200 characters; how many isn't known in advance) until the end of the input, and print them in **reverse order**.

```text
Sample input:
first line
second line
third line
Sample output:
third line
second line
first line
```

<details>
<summary>Hint</summary>

Use a doubling `char **lines` array whose elements each point to one line. For each line (read with `fgets` into a temporary `char buf[202]`, with the `'\n'` removed), `malloc(strlen(buf) + 1)` bytes and `strcpy` the line into them. Finally print backwards, and **`free` each line first, then `free(lines)`**.
</details>

<!-- exercise: matrix-mul -->
### Exercise 4: Matrix multiplication

The first line holds `n m p` (each from 1 to 50); then `n` rows of `m` integers give matrix A, then `m` rows of `p` integers give matrix B. Print C = A × B (`n` rows of `p` numbers, separated by spaces), where row i, column j of C is the sum of the products of the matching elements of row i of A and column j of B.

```text
Sample input:
2 3 2
1 2 3
4 5 6
7 8
9 10
11 12
Sample output:
58 64
139 154
```

<details>
<summary>Hint</summary>

Store all three matrices the 1D way from section 11.5: row i, column k of A is `a[i * m + k]`. `c[i * p + j] = Σ a[i * m + k] * b[k * p + j]` (k from 0 to m-1).
</details>

<!-- exercise: mem-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int *a = calloc(4, sizeof *a);
    if (a == NULL) {
        return 1;
    }
    a[1] = 5;
    int *b = realloc(a, 6 * sizeof *b);
    if (b == NULL) {
        free(a);
        return 1;
    }
    a = b;
    a[5] = 9;
    printf("%d %d %d %d %d\n", a[0], a[1], a[2], a[3], a[5]);
    free(a);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`calloc` zeroes all 4 `int`s, then `a[1] = 5`. `realloc` grows the block to 6, keeping the first 4 values (the new `a[4]` and `a[5]` hold garbage, which is why the program doesn't print `a[4]`), and `a[5]` is set to 9. The output is `0 5 0 0 9`.
</details>

<!-- exercise: mem-safe -->
### Exercise 6: Which one has no memory bug?

- `int *p = malloc(4 * sizeof *p); p[4] = 1; free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); p = NULL; free(p);`
- `int x = 0; int *p = &x; free(p);`

<details>
<summary>Answer and explanation</summary>

The third is correct: `p` is set to `NULL` after `free`, and `free(NULL)` does nothing. The first goes out of bounds (only `p[0]`–`p[3]` exist), the second frees twice, and the fourth frees memory that didn't come from `malloc`.
</details>

## 11.9 Further reading

- [cppreference: dynamic memory management](https://en.cppreference.com/w/c/memory) (`malloc`, `calloc`, `realloc`, `free`)
- [AddressSanitizer documentation](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- Next lesson: [Structs](../12-structs/en.md)
