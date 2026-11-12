# 10. Pointers

> 🎯 **Goals**: understand memory and **addresses**; define pointers, take addresses with `&` and reach the variable a pointer points to with `*`; use pointer parameters to write a `swap` that really swaps, and to let functions "return" several results; know about `NULL` and wild pointers; understand how pointers and arrays relate (pointer arithmetic, `a[i]` is `*(a + i)`); handle strings with pointers (`strchr`, `strtol`) and know that string literals can't be changed; meet `void *` and function pointers, and sort with the standard library's `qsort`.
>
> ⏱ **Time**: 110 minutes　|　📋 **You need**: lessons 1–9 of this course

Pointers are C's most distinctive feature, and the one beginners worry about most. In fact there's only one core idea: the **address**. Take it slowly, and run and tweak every example.

## 10.1 Memory and addresses

Think of memory as a very, very long street where every house (every **byte**) has a house number: its **address**. Every variable lives at some address; an `int` takes 4 houses in a row, a `double` 8.

```c
#include <stdio.h>

int main(void) {
    int age = 20;
    double price = 9.5;
    printf("age is %d and lives at %p\n", age, (void *)&age);
    printf("price is %.1f and lives at %p\n", price, (void *)&price);
    printf("age takes %zu bytes, price takes %zu\n", sizeof age, sizeof price);
    return 0;
}
```

- `&age` reads "the address of age"; `&` is the **address-of operator**. You've used it for ages in `scanf("%d", &age)`.
- `%p` prints an address in hexadecimal; the rules say to convert it to `(void *)` first (section 10.8 explains `void *`).
- The addresses will probably change from run to run — the operating system shuffles them on purpose, for security. The actual numbers don't matter.

## 10.2 Pointer variables

A **pointer** is simply "a variable that holds an address".

```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int *p = &x;                  // p is a "pointer to int" holding x's address

    printf("x = %d\n", x);
    printf("the address in p: %p\n", (void *)p);
    printf("*p = %d\n", *p);      // *p: the int found at that address, i.e. x

    *p = 25;                      // change x through the pointer
    printf("x = %d\n", x);        // 25

    int y = 7;
    p = &y;                       // a pointer can be pointed somewhere else
    *p += 1;
    printf("y = %d\n", y);        // 8
    return 0;
}
```

```text
       x                    p
    ┌──────┐          ┌──────────────┐
    │  25  │ ◀─────── │ address of x │
    └──────┘          └──────────────┘
```

| Written | Meaning |
|---------|---------|
| `int *p;` | define a pointer `p` that can hold the address of an `int` variable |
| `&x` | the address of `x` |
| `*p` | **dereference**: the variable `p` points to, for reading or writing |

`*` has two uses — don't mix them up: **in a definition**, `int *p` says "p is a pointer"; **in an expression**, `*p` means "the thing p points to".

> [!TIP]
> `int *p` and `int* p` are the same, but prefer the first. In `int* a, b;` only `a` is a pointer; `b` is a plain `int`! Two pointers are written `int *a, *b;`.

A pointer's type must match the variable it points to: the address of an `int` goes in an `int *`, the address of a `double` in a `double *`.

## 10.3 Pointers as function parameters

Lesson 7's `swap` failed because the function got copies. Now let's pass **addresses** instead:

```c
#include <stdio.h>

void swap(int *a, int *b) {        // the parameters are two addresses
    int t = *a;
    *a = *b;
    *b = t;
}

void min_max(const int arr[], int n, int *min, int *max) {   // "return" two results through pointers
    *min = arr[0];
    *max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < *min) {
            *min = arr[i];
        }
        if (arr[i] > *max) {
            *max = arr[i];
        }
    }
}

int main(void) {
    int p = 1, q = 2;
    swap(&p, &q);                       // pass the addresses
    printf("p = %d, q = %d\n", p, q);   // p = 2, q = 1: it worked!

    int data[] = {4, -2, 9, 0};
    int lo = 0, hi = 0;
    min_max(data, 4, &lo, &hi);
    printf("min %d, max %d\n", lo, hi);
    return 0;
}
```

C still passes by value — it's just that this time the value copied is an **address**. The copy points to the same place as the original, so changing `*a` changes `main`'s `p`.

Now `scanf("%d", &x)` makes sense too: `scanf` has to store the number it reads into `x`, so it needs `x`'s address.

## 10.4 `NULL` and wild pointers

**An uninitialised pointer** holds a garbage address and is called a **wild pointer**. Reading or writing through it means writing to a random place — it may crash, or it may quietly corrupt other data:

```c norun
int *p;          // not initialised: a wild pointer
*p = 5;          // undefined behaviour!
```

The rules are simple:

- initialise pointers when you define them: either point them at a variable (`&x`) or set them to **`NULL`** — a special "empty address" meaning "points nowhere";
- when you're not sure a pointer is valid, check `if (p != NULL)` before using it;
- reading or writing through `NULL` crashes immediately (`Segmentation fault`). With the memory check on (lesson 8), you're told the line.

Many functions return `NULL` to mean "not found" or "failed":

```c
#include <stdio.h>

int *find(int arr[], int n, int target) {   // return the element's address if found, NULL if not
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return &arr[i];
        }
    }
    return NULL;
}

int main(void) {
    int data[] = {3, 8, 5};
    int *p = find(data, 3, 8);
    if (p != NULL) {
        *p = 80;                     // change that array element directly through the pointer
    }
    if (find(data, 3, 42) == NULL) {
        printf("42 not found\n");
    }
    printf("%d %d %d\n", data[0], data[1], data[2]);   // 3 80 5
    return 0;
}
```

> [!WARNING]
> A function **must not return the address of its own local variable**: the local variable disappears when the function ends, and the address becomes a "dangling pointer". The `find` above returns the address of an element in the caller's array, which is fine. GCC warns `function returns address of local variable` when it spots the mistake.

## 10.5 Pointers and arrays

Arrays and pointers are very closely related:

```c
#include <stdio.h>

int main(void) {
    int a[5] = {10, 20, 30, 40, 50};
    int *p = a;                       // the array name turns into the address of its first element, &a[0]

    printf("%d %d\n", *p, *(p + 1));  // 10 20: p + 1 points to the next element
    printf("%d %d\n", a[3], *(a + 3));// a[i] is *(a + i)

    for (int *q = a; q < a + 5; q++) {   // walk through the array with a pointer
        printf("%d ", *q);
    }
    printf("\n");

    int *r = &a[4];
    printf("r and p are %td elements apart\n", r - p);              // 4
    printf("sizeof a = %zu, sizeof p = %zu\n", sizeof a, sizeof p);  // 20 and 8
    return 0;
}
```

- **In most expressions, an array's name turns into the address of its first element.** That's why `int *p = a;` needs no `&`, and neither does `scanf("%s", name)`.
- **Pointer arithmetic**: `p + 1` doesn't add 1 to the address; it **moves on by one element** (4 bytes, for an `int *`). Subtracting two pointers into the same array gives the number of elements between them (printed with `%td`).
- `a[i]` is just shorthand for `*(a + i)`.
- The function parameters `int a[]` and `int *a` are **exactly the same** — which is why, in lesson 8, `sizeof a` inside a function gave 8: that `a` really is a pointer.

But an array isn't a pointer: an array is a whole block of memory holding the data, `sizeof a` is the size of the whole array, and `a = p;` and `a++` aren't allowed; a pointer is just a variable holding an address (usually 8 bytes), and it can be pointed elsewhere at any time.

## 10.6 Pointers and strings

String functions use pointers everywhere. `strchr` and `strstr` return a pointer to what they found:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(void) {
    char text[] = "name=Ada;lang=C";
    char *eq = strchr(text, '=');              // points to the first '='
    if (eq != NULL) {
        printf("'=' is at index %td\n", eq - text);   // subtracting pointers: 4
        printf("after it: %s\n", eq + 1);              // the string starting at the next character
    }

    const char *greeting = "hello";            // points to a string literal: read-only
    printf("the first letter of %s is %c\n", greeting, greeting[0]);

    // strtol: convert a string to an integer, and say through end where it stopped
    const char *input = "123abc";
    char *end;
    long value = strtol(input, &end, 10);      // 10 means decimal
    printf("read %ld, left over: \"%s\"\n", value, end);
    if (end == input) {
        printf("no digits at all\n");
    } else if (*end != '\0') {
        printf("there are other characters after the number\n");
    }
    return 0;
}
```

- `strtol(string, &end, base)` is more careful than `atoi`: `end` points to the first character that wasn't converted. `end == input` means there were no digits; `*end == '\0'` means the whole string was a number. There's also `strtod` for decimals.
- **String literals can't be changed.** `"hello"` lives in read-only memory, so this code crashes:

```c norun
char *s = "hello";
s[0] = 'J';          // segmentation fault!
```

| Written | Meaning |
|---------|---------|
| `char s[] = "hello";` | an **array**: the literal is **copied** into it, and you can change it freely |
| `const char *s = "hello";` | a **pointer** to the read-only literal. With `const`, accidentally writing `s[0] = 'J'` becomes a compile error |

## 10.7 `const` and pointers

`const` in a function parameter is a promise:

- `const int *p` (or lesson 8's `const int a[]`): you **can't change the data through `p`**. Reading is fine; writing is a compile error, `assignment of read-only location`.
- `int *const p`: `p` **itself** can't be pointed elsewhere, but you can change the data through it. This is less common.

Rule of thumb: when a function only reads the data a pointer points to, make the parameter `const`. Callers see at a glance that their data won't change, and the compiler checks it for you.

## 10.8 `void *`, function pointers and `qsort`

### Function pointers

Functions have addresses too, and a variable holding one is a **function pointer**:

```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int mul(int a, int b) {
    return a * b;
}

int main(void) {
    int (*op)(int, int) = add;     // op points to "a function taking two ints and returning an int"
    printf("%d\n", op(3, 4));      // 7
    op = mul;                      // switch to another function
    printf("%d\n", op(3, 4));      // 12
    return 0;
}
```

The syntax looks odd: in `int (*op)(int, int)`, `(*op)` says `op` is a pointer, and `(int, int)` and the leading `int` say what kind of function it points to. A function's name on its own (without parentheses) is the function's address.

### Sorting with `qsort`

The standard library's `qsort` (in `<stdlib.h>`) sorts arrays of **any type**, and far faster than bubble sort. Since it doesn't know what the elements are, you give it a **comparison function** that says which of two elements comes first:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int compare_ints(const void *a, const void *b) {
    int x = *(const int *)a;          // first turn the void * into an int *, then read the value
    int y = *(const int *)b;
    return (x > y) - (x < y);         // -1 if x < y, 0 if equal, 1 if x > y
}

int compare_words(const void *a, const void *b) {
    return strcmp((const char *)a, (const char *)b);
}

int main(void) {
    int nums[] = {42, 7, 19, -3, 7};
    int n = sizeof nums / sizeof nums[0];
    qsort(nums, n, sizeof nums[0], compare_ints);    // array, count, size of each element, comparison
    for (int i = 0; i < n; i++) {
        printf("%d ", nums[i]);
    }
    printf("\n");

    char words[][10] = {"pear", "apple", "fig", "banana"};
    qsort(words, 4, sizeof words[0], compare_words);
    for (int i = 0; i < 4; i++) {
        printf("%s ", words[i]);
    }
    printf("\n");
    return 0;
}
```

- **`void *`** is the "generic pointer": it can hold the address of any type of data, but it doesn't know the type, so you **can't use `*` on it directly** — convert it to a specific pointer type first, such as `(const int *)a`.
- The comparison function always has the form `int name(const void *a, const void *b)`. Return a negative number if `a` should come before `b`, a positive one if after, and 0 if they're equal.
- Why not just `return x - y;`? When `x` and `y` are very large or very small, the subtraction can **overflow** (lesson 3) and give the wrong sign. `(x > y) - (x < y)` never overflows.
- To sort from largest to smallest, reverse the comparison, e.g. `return compare_ints(b, a);`.

## 10.9 Common mistakes

| Mistake | What happens | Do this instead |
|---------|--------------|-----------------|
| `int *p; *p = 1;` (wild pointer) | crash or corrupted data | initialise to `&x` or `NULL` when defining it |
| reading or writing through `NULL` | segmentation fault | check `if (p != NULL)` first |
| returning the address of a local variable | dangling pointer | return the value, have the caller pass a pointer in, or use `malloc` (lesson 11) |
| a missing `&` or an extra `*` | warning `makes pointer from integer without a cast` | keep `x`, `&x`, `p` and `*p` straight |
| `int* a, b;` | `b` isn't a pointer | `int *a, *b;` |
| changing a string literal | segmentation fault | to change it, use `char s[] = "..."`; otherwise write `const char *` |
| using `sizeof` on an array parameter inside a function | gives the size of a pointer (8) | pass the length as a parameter |

## 10.10 Summary

- Every variable has an address; `&x` takes it, `%p` prints it.
- `int *p = &x;` defines a pointer to `x`; `*p` is `x` itself, for reading and writing.
- Passing pointers lets a function change the caller's variables (`swap`) and "return" several results.
- Initialise pointers; `NULL` means "points nowhere" — check before use; never return the address of a local variable.
- An array name turns into the address of its first element; `p + 1` moves on one element; `a[i]` is `*(a + i)`; the parameter `int a[]` is `int *a`.
- `strchr`/`strstr` return pointers; `strtol` reports where it stopped through the `end` pointer; string literals are read-only, so point to them with `const char *`.
- `const int *p` promises not to change the data through `p`.
- `void *` is the generic pointer and must be converted before use; a function's name is its address; `qsort` with a comparison `int cmp(const void *a, const void *b)` sorts any array.

## 10.11 Exercises

<!-- exercise: swap-ptr -->
### Exercise 1: A real swap

Complete the function `void swap(int *a, int *b)` so it swaps the two integers the pointers point to. `main` is already written: it reads two integers, swaps them and prints them.

```text
Sample input: 3 8
Sample output: 8 3
```

<details>
<summary>Hint</summary>

As in section 10.3: use a temporary variable, `int t = *a;`.
</details>

<!-- exercise: split-time -->
### Exercise 2: Splitting time

Write a function `void split_time(int total, int *h, int *m, int *s)` that splits a number of seconds into hours, minutes and seconds, "returning" them through the pointers. Read the total number of seconds (0 ≤ total < 1000000) and print it as `hours:minutes:seconds`, with two digits for the minutes and seconds.

```text
Sample input: 3661
Sample output: 1:01:01
```

<details>
<summary>Hint</summary>

`*h = total / 3600;`, `*m = total % 3600 / 60;`, `*s = total % 60;`. Print with `printf("%d:%02d:%02d\n", h, m, s);`.
</details>

<!-- exercise: qsort-desc -->
### Exercise 3: Largest first

Read `n` (1 ≤ n ≤ 1000) and `n` integers, sort them **from largest to smallest** with `qsort`, and print them separated by spaces.

```text
Sample input:
5
3 -1 4 1 5
Sample output: 5 4 3 1 -1
```

<details>
<summary>Hint</summary>

Write a comparison function that returns a negative number when `x > y` (so larger numbers come first): `return (x < y) - (x > y);`.
</details>

<!-- exercise: count-substr -->
### Exercise 4: How many times?

The first line is a piece of English text, the second a word. Print how many times the word occurs in the text (**overlaps allowed**, case-sensitive).

```text
Sample input:
banana
ana
Sample output: 2
```

<details>
<summary>Hint</summary>

Search repeatedly with `strstr`: each time it finds a match, count it and continue from the **next character** after the match's start, `p = strstr(p + 1, word);`, until it returns `NULL`. Read both lines with `fgets` and remove the `'\n'`.
</details>

<!-- exercise: reverse-ptr -->
### Exercise 5: Reversing a string with two pointers

Write a function `void reverse(char *s)` that reverses a string in place using two pointers, one moving in from the start and one from the end. Read an English word and print it reversed.

```text
Sample input: pointer
Sample output: retniop
```

<details>
<summary>Hint</summary>

`char *left = s;`, `char *right = s + strlen(s) - 1;`; while `left < right`, swap `*left` and `*right`, then `left++` and `right--`. Watch out for the empty string.
</details>

<!-- exercise: ptr-predict -->
### Exercise 6: Predict the output

```c
#include <stdio.h>

int main(void) {
    int a[] = {1, 2, 3, 4};
    int *p = a + 1;
    *p = 20;
    p++;
    *p += 10;
    printf("%d %d %d %d\n", a[0], a[1], a[2], a[3]);
    printf("%td\n", p - a);

    int x = 5;
    int *q = &x;
    *q = *q * 2;
    printf("%d\n", x);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`p` starts at `a[1]` and sets it to 20; after `p++` it points to `a[2]`, which gains 10 and becomes 13. `p - a` is 2. `q` points to `x`, so `*q = *q * 2` is `x = x * 2`. The output:

```text
1 20 13 4
2
10
```
</details>

<!-- exercise: ptr-value -->
### Exercise 7: Which one is 3?

```c norun
int x = 3;
int *p = &x;
```

Which expression has the value `3`?

- `p`
- `*p`
- `&p`
- `&x`

<details>
<summary>Answer and explanation</summary>

`*p` is the variable `p` points to, i.e. `x`, whose value is 3. `p` and `&x` are both the address of `x`, and `&p` is the address of the pointer variable `p` itself.
</details>

## 10.12 Further reading

- [cppreference: pointer declaration](https://en.cppreference.com/w/c/language/pointer), [qsort](https://en.cppreference.com/w/c/algorithm/qsort), [strtol](https://en.cppreference.com/w/c/string/byte/strtol)
- [Python Tutor (choose C)](https://pythontutor.com/c.html) — paste this lesson's code in and watch step by step where each pointer points
- Next lesson: [Dynamic memory](../11-memory/en.md)
