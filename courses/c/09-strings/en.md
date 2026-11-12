# 9. Strings

> 🎯 **Goals**: understand that a C string is "an array of `char` ending in `'\0'`"; print and read strings (`scanf` for a word, `fgets` for a whole line) and deal with the "leftover newline"; use the common functions in `<string.h>` (`strlen`, `strcpy`, `strcat`, `strcmp`, `strstr`), know their dangers and the safer `snprintf`; process characters one at a time with `<ctype.h>`; convert between strings and numbers; understand that non-English characters take several bytes in UTF-8; use arrays of strings.
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: lessons 1–8 of this course

## 9.1 A string is an array of char ending in `'\0'`

C has no special "string type". A string is just an array of `char` with a special character `'\0'` at the end (the character with value 0, called the **null character**), meaning "the string stops here":

```c
#include <stdio.h>

int main(void) {
    char a[6] = {'H', 'e', 'l', 'l', 'o', '\0'};   // written character by character
    char b[] = "Hello";                            // the same: the compiler adds '\0' at the end

    printf("%s %s\n", a, b);                       // %s prints a string
    printf("b takes %zu bytes\n", sizeof b);       // 6: 5 letters + 1 '\0'

    for (int i = 0; b[i] != '\0'; i++) {           // walk character by character up to '\0'
        printf("[%c]", b[i]);
    }
    printf("\n");

    b[0] = 'J';                                    // the characters in a char array can be changed
    printf("%s\n", b);                             // Jello
    b[2] = '\0';                                   // put a '\0' in the middle…
    printf("%s\n", b);                             // Je: the string "ends" there
    return 0;
}
```

```text
         b[0] b[1] b[2] b[3] b[4] b[5]
        ┌────┬────┬────┬────┬────┬────┐
"Hello" │ H  │ e  │ l  │ l  │ o  │ \0 │
        └────┴────┴────┴────┴────┴────┘
```

- `"Hello"` (double quotes) is a string; `'H'` (single quotes) is one character — they're completely different things (lesson 2).
- `printf("%s", b)`, `strlen(b)` and every other string function **read from the start until they reach `'\0'`**. That's why `'\0'` is vital: without it, functions keep reading past the end of the array (out of bounds, lesson 8).
- So a string of n characters needs an array of at least **n + 1** `char`s.

## 9.2 Reading strings

**To read one word**, use `scanf("%Ns", array)` (lesson 4): it stops at a space or newline; write "array length − 1" as `N` so that long input can't write past the end; and **no** `&` before the array name.

**To read a whole line** (spaces included), use `fgets`:

```c input="20\nAda Lovelace"
#include <stdio.h>
#include <string.h>

int main(void) {
    int age = 0;
    char name[50];

    printf("Age: ");
    scanf("%d", &age);

    int ch;                                        // throw away the rest of this line (newline included)
    while ((ch = getchar()) != '\n' && ch != EOF) {
    }

    printf("\nFull name: ");
    if (fgets(name, sizeof name, stdin) == NULL) { // reads at most 49 characters; NULL if nothing could be read
        return 1;
    }
    name[strcspn(name, "\n")] = '\0';              // remove the newline at the end

    printf("\n%s, %d years old\n", name, age);
    return 0;
}
```

Three points in this code:

1. **`fgets(array, size, stdin)`** reads at most "size − 1" characters and never writes past the end. It **also reads the newline `'\n'`** at the end of the line, so it's usually followed by `name[strcspn(name, "\n")] = '\0';` to remove it (`strcspn` returns the position of the first `'\n'`; if there's none, it returns the string's length, so the line is safe either way).
2. **The leftover newline**: `scanf("%d", &age)` stops right after `20`, leaving the newline in the input. Left alone, `fgets` would read that newline at once and get an **empty line**. The `while` loop reads characters one by one with `getchar()` until the rest of the line (newline included) has been thrown away.
3. `ch` must be an `int`, not a `char`: at the end of input, `getchar()` returns `EOF` (usually -1), which isn't any character, and an `int` is needed to tell it apart from real characters.

> [!WARNING]
> Old books may show `gets(name)`. It doesn't check the length, so long input writes past the end — it has been **removed from the C language**. Always use `fgets`.

## 9.3 `<string.h>`: string functions

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[50] = "hello";
    printf("length %zu, array size %zu\n", strlen(s), sizeof s);   // 5 and 50

    char t[50];
    strcpy(t, s);                          // copy: t = s doesn't work (arrays can't be assigned)
    strcat(t, ", world");                  // concatenate: append to the end of t
    printf("%s\n", t);                     // hello, world

    if (strcmp(s, "hello") == 0) {         // compare contents: 0 means equal
        printf("s contains hello\n");
    }
    if (strcmp("apple", "banana") < 0) {   // negative: the first comes earlier in dictionary order
        printf("apple comes before banana\n");
    }
    if (strstr(t, "world") != NULL) {      // find a substring: NULL if not found
        printf("t contains world\n");
    }
    if (strchr(t, ',') != NULL) {          // find a character
        printf("t contains a comma\n");
    }
    return 0;
}
```

| Function | What it does |
|----------|--------------|
| `strlen(s)` | the length of the string, **not counting** `'\0'` |
| `strcpy(d, s)` | copy `s` into `d` (`d` must be big enough!) |
| `strcat(d, s)` | append `s` to the end of `d` (`d` must be big enough!) |
| `strcmp(a, b)` | compare: 0 if equal; negative if `a` comes first, positive if it comes after |
| `strncmp(a, b, n)` | compare only the first `n` characters, e.g. to check a prefix: `strncmp(s, "http", 4) == 0` |
| `strchr(s, c)` / `strstr(s, t)` | find character `c` / substring `t`; `NULL` if not found (the result is really a pointer — lesson 10) |
| `snprintf(d, size, format, ...)` | like `printf`, but "prints" into the string `d`, writing at most `size - 1` characters |

### `strcpy` and `strcat` are dangerous

They **don't know how big the destination is**; they just keep writing. If the destination is too small, they write past its end — the origin of "buffer overflow" security holes. GCC can spot some of these mistakes:

```text
warning: '__builtin_memcpy' writing 8 bytes into a region of size 4 overflows the destination
```

For joining and formatting strings, the safer choice is `snprintf`: it writes at most `size - 1` characters plus the `'\0'`, and if the result doesn't fit it **truncates** it, never going out of bounds:

```c
#include <stdio.h>

int main(void) {
    char first[] = "Grace", last[] = "Hopper";
    char full[50];
    snprintf(full, sizeof full, "%s %s", first, last);   // plenty of room
    printf("[%s]\n", full);

    char tiny[8];
    snprintf(tiny, sizeof tiny, "%s %s", first, last);   // room for only 7 characters
    printf("[%s]\n", tiny);                              // [Grace H]: safely truncated
    return 0;
}
```

(When compiling, GCC warns that the second `snprintf`'s `output may be truncated` — it's pointing out that this will be cut short, which is exactly what we're demonstrating.)

## 9.4 Comparing strings: not with `==`

```c norun
char answer[10] = "yes";
if (answer == "yes") { ... }          // wrong!
```

```text
warning: comparison with string literal results in unspecified behavior [-Waddress]
```

`answer == "yes"` compares **where the two strings are in memory**, not their contents, and is almost always "not equal". To compare contents, **always use `strcmp`**:

```c norun
if (strcmp(answer, "yes") == 0) { ... }   // right
```

In the same way, copy with `strcpy` (or `snprintf`), not `=`; and remember `strcmp` returns **0** for equal strings, so don't write `if (strcmp(a, b))` — that means "not equal".

## 9.5 One character at a time: `<ctype.h>`

`<ctype.h>` has functions for testing and converting single characters:

```c
#include <stdio.h>
#include <ctype.h>

int main(void) {
    char s[] = "Hello, World 2026!";
    int letters = 0, digits = 0, spaces = 0, others = 0;

    for (int i = 0; s[i] != '\0'; i++) {
        unsigned char c = (unsigned char)s[i];   // ctype functions need values in the unsigned char range
        if (isalpha(c)) {
            letters++;
        } else if (isdigit(c)) {
            digits++;
        } else if (isspace(c)) {
            spaces++;
        } else {
            others++;
        }
        s[i] = (char)toupper(c);                 // to upper case; non-letters stay the same
    }
    printf("%s\n", s);
    printf("letters %d, digits %d, whitespace %d, others %d\n", letters, digits, spaces, others);
    return 0;
}
```

| Function | Tests / does |
|----------|--------------|
| `isalpha(c)` | is it an English letter? |
| `isdigit(c)` | is it a digit `0`–`9`? |
| `isalnum(c)` | is it a letter or a digit? |
| `isspace(c)` | is it whitespace (space, `\t`, `\n`…)? |
| `isupper(c)` / `islower(c)` | is it an upper-case / lower-case letter? |
| `toupper(c)` / `tolower(c)` | convert to upper / lower case (non-letters come back unchanged) |

> [!NOTE]
> Why convert to `unsigned char` first? On many platforms `char` is signed, so the bytes of non-English characters become **negative numbers**, and passing a negative number to `isalpha` and friends is undefined behaviour. The conversion makes it safe.

Characters are small integers (lesson 2), so you can do arithmetic with them. The codes of the digit characters `'0'`–`'9'` are consecutive, so `c - '0'` is the digit's value; letters are consecutive too, so `'a' + 2` is `'c'`:

```c
#include <stdio.h>

int main(void) {
    char number[] = "9527";
    int sum = 0;
    for (int i = 0; number[i] != '\0'; i++) {
        sum += number[i] - '0';          // '9' - '0' = 9
    }
    printf("sum of the digits: %d\n", sum);   // 23

    for (char c = 'a'; c <= 'e'; c++) {
        printf("%c", c - 'a' + 'A');     // a "hand-made" lower-to-upper conversion
    }
    printf("\n");
    return 0;
}
```

## 9.6 Converting between strings and numbers

- **String → number**: use `sscanf`, which works like `scanf` but reads from a string. Its return value tells you how many values it read successfully.
- **Number → string**: use `snprintf`.

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char text[] = "42 3.5";
    int n = 0;
    double x = 0;
    if (sscanf(text, "%d %lf", &n, &x) == 2) {        // success only if both values were read
        printf("%d %.1f\n", n * 2, x * 2);
    }
    if (sscanf("abc", "%d", &n) != 1) {
        printf("\"abc\" is not an integer\n");
    }

    char date[20];
    snprintf(date, sizeof date, "%d-%02d-%02d", 2026, 3, 7);
    printf("%s, length %zu\n", date, strlen(date));    // 2026-03-07, length 10
    return 0;
}
```

You may come across `atoi("42")`. It's simple, but given `"abc"` it quietly returns 0, so you can't tell something went wrong; the more careful `strtol` needs pointers, so it waits until lesson 10.

## 9.7 Non-English text and UTF-8

`strlen` counts **bytes**, not "characters". In **UTF-8**, today's standard encoding, English letters take 1 byte, accented letters such as é and Cyrillic letters 2, Chinese characters usually 3, and emoji 4:

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[] = "café";
    printf("strlen = %zu\n", strlen(s));               // 5 = 1 + 1 + 1 + 2

    int chars = 0;
    for (int i = 0; s[i] != '\0'; i++) {
        if (((unsigned char)s[i] & 0xC0) != 0x80) {   // not a "continuation byte", so a new character starts
            chars++;
        }
    }
    printf("characters = %d\n", chars);                 // 4
    return 0;
}
```

The UTF-8 rule: after a character's first byte, its "continuation bytes" all start with the bits `10` (`0x80`–`0xBF`). `& 0xC0` keeps the top two bits (bit operations, lesson 3), so any byte not equal to `0x80` after that starts a new character.

So be careful with non-English text: reversing a string byte by byte breaks such characters apart, `char s[3]` can't even hold "é" plus another letter and the `'\0'`, and functions like `toupper` only work on English letters.

## 9.8 Arrays of strings

Several strings fit in a **two-dimensional char array**, one string per row:

```c
#include <stdio.h>
#include <string.h>

#define COUNT 4
#define LEN 20

int main(void) {
    char names[COUNT][LEN] = {"Carol", "alice", "Bob", "Dave"};

    // bubble sort (lesson 8): compare with strcmp, swap with strcpy
    for (int pass = 0; pass < COUNT - 1; pass++) {
        for (int i = 0; i < COUNT - 1 - pass; i++) {
            if (strcmp(names[i], names[i + 1]) > 0) {
                char t[LEN];
                strcpy(t, names[i]);
                strcpy(names[i], names[i + 1]);
                strcpy(names[i + 1], t);
            }
        }
    }
    for (int i = 0; i < COUNT; i++) {
        printf("%s\n", names[i]);
    }
    return 0;
}
```

`alice` ends up last! `strcmp` compares characters by their **codes**, and all upper-case letters (`A`–`Z` are 65–90) come before lower-case ones (`a`–`z` are 97–122). To ignore case, convert both strings to lower case before comparing.

## 9.9 Common mistakes

| Mistake | What happens | Do this instead |
|---------|--------------|-----------------|
| `char s[5] = "Hello";` leaves no room for `'\0'` | the string has no end; functions read out of bounds | length at least n + 1, or `char s[] = "Hello";` |
| `s = "abc";` | compile error: arrays can't be assigned | `strcpy(s, "abc");` or `snprintf` |
| `if (s == "abc")` | compares locations; almost always false | `if (strcmp(s, "abc") == 0)` |
| `scanf("%s", s);` without a width | long input writes out of bounds | `scanf("%49s", s);` |
| forgetting to remove `'\n'` after `fgets` | an extra newline when comparing or printing | `s[strcspn(s, "\n")] = '\0';` |
| `fgets` straight after `scanf("%d")` | reads an empty line | first throw away the rest of the line with a `getchar` loop |
| `strcpy` / `strcat` into a destination that's too small | buffer overflow | work out the size, or use `snprintf` |

## 9.10 Summary

- A C string is a `char` array ending in `'\0'`; `"..."` adds the `'\0'` automatically; n characters need room for n + 1.
- Reading a word: `scanf("%49s", s)`; a whole line: `fgets(s, sizeof s, stdin)`, then remove the `'\n'`; when mixing `scanf` and `fgets`, throw away the leftover newline first.
- `<string.h>`: `strlen`, `strcpy`, `strcat`, `strcmp` (0 when equal), `strncmp`, `strchr`, `strstr`; `strcpy`/`strcat` don't check sizes, so prefer `snprintf` for joining and formatting.
- Compare contents with `strcmp`, not `==`; copy with `strcpy`, not `=`.
- `<ctype.h>`: `isalpha`, `isdigit`, `isspace`, `toupper`… — convert the argument to `unsigned char` first; `c - '0'` gives a digit character's value.
- Strings ↔ numbers: `sscanf` (check its return value) and `snprintf`.
- In UTF-8, non-English characters take 2–4 bytes; `strlen` counts bytes.
- Arrays of strings: `char names[N][LEN]`.

## 9.11 Exercises

<!-- exercise: char-classes -->
### Exercise 1: Counting kinds of characters

Read a line of English text (it may contain spaces) and print how many **letters**, **digits**, **spaces** and **other characters** it has, separated by spaces. (The newline at the end doesn't count.)

```text
Sample input: Hello World 123!
Sample output: 10 3 2 1
```

<details>
<summary>Hint</summary>

Read the line with `fgets`, remove the `'\n'`, then test each character with `isalpha`, `isdigit` and `c == ' '`.
</details>

<!-- exercise: palindrome -->
### Exercise 2: Palindromes

Read an English word (at most 100 letters). If it reads the same forwards and backwards (**ignoring case**), print `yes`; otherwise print `no`.

```text
Sample input: Level
Sample output: yes
```

<details>
<summary>Hint</summary>

Walk two indices `i = 0` and `j = strlen(s) - 1` from both ends towards the middle, comparing `tolower(s[i])` with `tolower(s[j])`.
</details>

<!-- exercise: caesar -->
### Exercise 3: Caesar cipher

The first line holds an integer `k` (0 ≤ k ≤ 25), the second an English sentence. Shift every letter forward by `k` places (wrapping from `z` back to `a`), keeping upper-case letters upper-case and lower-case lower-case, and leaving other characters unchanged. Print the encrypted sentence.

```text
Sample input:
3
Hello, World!
Sample output: Khoor, Zruog!
```

<details>
<summary>Hint</summary>

For lower-case letters: `'a' + (c - 'a' + k) % 26`, and the same for upper case. Don't forget to throw away the leftover newline after `scanf` reads `k`, before reading the sentence with `fgets`.
</details>

<!-- exercise: word-count -->
### Exercise 4: Counting words

Read a line of English in which words are separated by one or more spaces (there may also be spaces at the start and end), and print the number of words. For an empty line, print `0`.

```text
Sample input:   the quick  brown fox 
Sample output: 4
```

<details>
<summary>Hint</summary>

Count "starts of words": the current character isn't a space, and it's either the first character or the one before it is a space.
</details>

<!-- exercise: title-case -->
### Exercise 5: Title case

Read a line of English and print it with the first letter of each word in upper case and the other letters in lower case. Words are separated by spaces.

```text
Sample input: hello wORLD from c
Sample output: Hello World From C
```

<details>
<summary>Hint</summary>

Detect "starts of words" as in exercise 4: use `toupper` there and `tolower` everywhere else.
</details>

<!-- exercise: str-predict -->
### Exercise 6: Predict the output

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[10] = "abc";
    printf("%zu %zu\n", strlen(s), sizeof s);
    s[1] = '\0';
    printf("%s %zu\n", s, strlen(s));
    printf("%d\n", strcmp("cat", "car") > 0);
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`strlen` is 3 (not counting `'\0'`) and `sizeof` is the whole array, 10. After `s[1]` becomes `'\0'`, the string ends there, leaving just `a`, of length 1. `"cat"` and `"car"` share their first two characters, and the third, `t`, is greater than `r`, so `strcmp` returns a positive number, `> 0` is true, and 1 is printed. The output:

```text
3 10
a 1
1
```
</details>

<!-- exercise: str-compare -->
### Exercise 7: Comparing strings

How do you check whether the strings in the char arrays `a` and `b` have the same contents?

- `if (a == b)`
- `if (strcmp(a, b) == 0)`
- `if (strcmp(a, b))`
- `if (strlen(a) == strlen(b))`

<details>
<summary>Answer and explanation</summary>

`strcmp` returns 0 when the two strings are the same. `a == b` compares locations; `if (strcmp(a, b))` is exactly backwards, meaning "different"; and equal lengths don't mean equal contents.
</details>

## 9.12 Further reading

- [cppreference: null-terminated byte strings](https://en.cppreference.com/w/c/string/byte) (all of `<string.h>` and `<ctype.h>`)
- [UTF-8 (Wikipedia)](https://en.wikipedia.org/wiki/UTF-8) — see how UTF-8 uses 1–4 bytes per character
- Next lesson: [Pointers](../10-pointers/en.md)
