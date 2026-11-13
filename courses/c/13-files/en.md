# 13. Files

> 🎯 **Goals**: open files with `fopen` and close them with `fclose`, and tell the modes `"r"`, `"w"`, `"a"` and friends apart; read and write text files with `fprintf`, `fputs`, `fgetc`, `fgets` and `fscanf`; see that `stdin`, `stdout` and `stderr` are "files" too, and write functions that work with any `FILE *`; handle errors such as "can't open the file" (`perror`, `strerror(errno)`); read formatted data such as CSV; read and write binary files with `fwrite` and `fread`; take command-line arguments with `argc` and `argv`.
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: lessons 1–12 of this course

While a program runs, its variables live in memory, and they're gone as soon as it ends. To **keep** data — game saves, mark sheets, settings — you write it to a **file**.

## 13.1 Open, write, close, read

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("hello.txt", "w");       // open a file for writing; create it if it doesn't exist
    if (f == NULL) {                         // NULL if it couldn't be opened
        perror("hello.txt");                 // print the reason
        return 1;
    }
    fprintf(f, "Hello, file!\n");            // just like printf, with one extra first argument
    fprintf(f, "1 + 1 = %d\n", 1 + 1);
    fclose(f);                               // close: makes sure the data really reaches the file

    f = fopen("hello.txt", "r");             // open it again, this time for reading
    if (f == NULL) {
        perror("hello.txt");
        return 1;
    }
    char line[100];
    while (fgets(line, sizeof line, f) != NULL) {   // read line by line; NULL at the end
        printf("read: %s", line);
    }
    fclose(f);
    return 0;
}
```

- **`FILE *`** is the standard library's "file pointer", standing for an open file. `fopen` returns one, and every reading or writing function after that needs it.
- **Always check whether `fopen` returned `NULL`**: a missing file, missing permissions or a wrong path all make it fail, and reading or writing through `NULL` crashes the program.
- **`fclose` when you're done.** Written data first collects in a buffer in memory and only reaches the disk on `fclose`; forget it and the file may be empty or incomplete.
- The file name `"hello.txt"` is a **relative path**, relative to the program's "current directory" while it runs. In this tutorial's app, every run happens in a temporary folder.

| Mode | What it does |
|------|--------------|
| `"r"` | read. The file must already exist |
| `"w"` | write. Creates the file if it doesn't exist; **if it does, its contents are erased** |
| `"a"` | append. What you write goes at the end of the file |
| `"r+"` / `"w+"` / `"a+"` | both read and write (`"w+"` erases the file too) |
| add `b`, as in `"rb"`, `"wb"` | binary mode (section 13.5) |
| add `x`, as in `"wx"` | **fail** if the file already exists, so nothing is overwritten by accident (since C11) |

## 13.2 `stdin`, `stdout` and `stderr` are files too

As soon as a program starts, three `FILE *`s are already open:

| Name | What it is |
|------|------------|
| `stdin` | standard input: the keyboard, or the input box in the learning app |
| `stdout` | standard output: where `printf` writes |
| `stderr` | standard error: meant for error messages |

`printf(...)` is really `fprintf(stdout, ...)`, and `scanf(...)` is `fscanf(stdin, ...)`. So a function that takes a `FILE *` parameter can **handle files and keyboard input alike**:

```c input="apple\nbanana\ncherry"
#include <stdio.h>

int count_lines(FILE *in) {              // works with any FILE *
    int lines = 0;
    int ch;
    while ((ch = fgetc(in)) != EOF) {    // fgetc: read one character; EOF at the end
        if (ch == '\n') {
            lines++;
        }
    }
    return lines;
}

int main(void) {
    FILE *f = fopen("abc.txt", "w");
    if (f == NULL) {
        perror("abc.txt");
        return 1;
    }
    fputs("a\nb\nc\nd\n", f);            // fputs: write a string
    fclose(f);

    f = fopen("abc.txt", "r");
    if (f == NULL) {
        perror("abc.txt");
        return 1;
    }
    printf("abc.txt has %d lines\n", count_lines(f));
    fclose(f);

    printf("standard input has %d lines\n", count_lines(stdin));
    fprintf(stderr, "this sentence went to stderr\n");
    return 0;
}
```

Writing error messages to `stderr` has two benefits: even when the output is redirected to a file (`./main > out.txt`), errors still appear on screen; and they don't get mixed into the normal output — the learning app only compares `stdout` when checking answers.

The everyday reading and writing functions:

| Write | Read | Unit |
|-------|------|------|
| `fputc(c, f)` | `fgetc(f)` (returns `EOF` at the end) | one character |
| `fputs(s, f)` | `fgets(buf, size, f)` (returns `NULL` at the end) | a line / a string |
| `fprintf(f, format, ...)` | `fscanf(f, format, ...)` (returns how many values it read) | formatted data |
| `fwrite(p, size, n, f)` | `fread(p, size, n, f)` (returns how many items it read) | a block of bytes (binary) |

## 13.3 Handling errors

When opening fails, the global variable **`errno`** (in `<errno.h>`) holds an error number, `strerror(errno)` (in `<string.h>`) turns it into a sentence, and `perror("prefix")` prints "prefix: explanation" straight to `stderr`.

Many programs fall back to default settings when their settings file is missing, instead of quitting:

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void) {
    int volume = 50;                                  // the default
    FILE *f = fopen("settings.txt", "r");
    if (f == NULL) {
        printf("couldn't read settings.txt (%s), using the defaults\n", strerror(errno));
    } else {
        if (fscanf(f, "%d", &volume) != 1) {          // check fscanf's return value
            printf("settings.txt is malformed, using the defaults\n");
            volume = 50;
        }
        fclose(f);
    }
    printf("volume: %d\n", volume);
    return 0;
}
```

If the file is essential, print an error message and end the program instead: `return EXIT_FAILURE;` (the "failure" exit code defined in `<stdlib.h>`, usually 1).

### Reaching the end of a file: don't use `while (!feof(f))`

```c norun
while (!feof(f)) {                  // the wrong way!
    fgets(line, sizeof line, f);
    printf("%s", line);             // the last line gets handled twice
}
```

`feof(f)` only becomes true **after a read has already failed**, so this loop goes round once too often, handling the leftovers of the previous read again. The right way is to **check the reading function's return value** directly: `while (fgets(...) != NULL)`, `while (fscanf(...) == 1)`, `while ((ch = fgetc(f)) != EOF)`.

## 13.4 Reading formatted data: CSV

**CSV** (comma-separated values) is the most common format for tables, and Excel opens it too:

```text
name,score
Ada,95
Linus,88
Grace,91
```

The most robust way to read it: read a whole line with `fgets`, then pick the data out of that line with `sscanf`. That way one malformed line doesn't disturb the lines after it:

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("scores.csv", "w");
    if (f == NULL) {
        perror("scores.csv");
        return 1;
    }
    fputs("name,score\nAda,95\nLinus,88\nGrace,91\n", f);
    fclose(f);

    f = fopen("scores.csv", "r");
    if (f == NULL) {
        perror("scores.csv");
        return 1;
    }
    char line[200];
    if (fgets(line, sizeof line, f) == NULL) {         // skip the first line (the header)
        fclose(f);
        return 0;
    }
    char name[32];
    int score = 0, count = 0;
    long total = 0;
    while (fgets(line, sizeof line, f) != NULL) {
        if (sscanf(line, "%31[^,],%d", name, &score) == 2) {   // %[^,]: read up to the comma
            printf("%-6s %d\n", name, score);
            total += score;
            count++;
        }
    }
    fclose(f);
    if (count > 0) {
        printf("average %.2f\n", (double)total / count);
    }
    return 0;
}
```

`%31[^,]` is a **scanset**: `[^,]` means "any character except a comma", so it reads up to the comma (at most 31 characters), and spaces in names are fine. Similarly, `%[^\n]` reads to the end of the line.

## 13.5 Binary files

`fwrite` writes the bytes in memory to the file **exactly as they are**, and `fread` reads them back the same way. Saving an array of structs takes one line:

```c
#include <stdio.h>

typedef struct {
    char name[16];
    int level;
    double hp;
} Player;

int main(void) {
    Player team[3] = {{"Ada", 12, 98.5}, {"Bob", 7, 40.0}, {"Cy", 20, 150.25}};
    FILE *f = fopen("save.dat", "wb");
    if (f == NULL) {
        perror("save.dat");
        return 1;
    }
    size_t written = fwrite(team, sizeof team[0], 3, f);    // 3 items of sizeof team[0] bytes each
    fclose(f);
    printf("wrote %zu saves of %zu bytes each\n", written, sizeof team[0]);

    Player loaded[3];
    f = fopen("save.dat", "rb");
    if (f == NULL) {
        perror("save.dat");
        return 1;
    }
    size_t got = fread(loaded, sizeof loaded[0], 3, f);    // returns how many items were read
    fclose(f);
    for (size_t i = 0; i < got; i++) {
        printf("%-4s Lv.%-3d HP %.2f\n", loaded[i].name, loaded[i].level, loaded[i].hp);
    }
    return 0;
}
```

**Text or binary?**

- Binary files are small and fast to read and write, but people can't read them; and the size of `int`, the order of bytes and struct padding (lesson 12) can differ between computers and compilers, so another machine may not be able to read them.
- Text files (CSV, JSON…) are a little bigger, but people can read them, other programs can process them, and they work everywhere.
- To exchange data with other people and programs, prefer text formats; binary is fine for temporary data only your own program reads.

## 13.6 Command-line arguments

When you run a program in a terminal, you can add arguments after it, as in `./wordcount essay.txt -v`. `main` can receive them like this:

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("%d arguments in all\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    return 0;
}
```

- `argc` (argument count) is the number of arguments, and `argv` (argument vector) is an array of strings.
- `argv[0]` is the program's own name (path); the real arguments start at `argv[1]`. Running `./wordcount essay.txt -v` gives an `argc` of 3.
- The learning app runs your code without arguments, so you'll only see an `argc` of 1. Try adding arguments when you compile and run in a terminal.

Many command-line tools start like this:

```c norun
if (argc < 2) {
    fprintf(stderr, "usage: %s filename\n", argv[0]);
    return EXIT_FAILURE;
}
FILE *f = fopen(argv[1], "r");
```

## 13.7 Common mistakes

| Mistake | What happens | Do this instead |
|---------|--------------|-----------------|
| not checking what `fopen` returned | crashes when the file can't be opened | `if (f == NULL) { perror(...); ... }` |
| forgetting `fclose` | incomplete data; too many open files | pair every `fopen` with an `fclose` |
| opening an important file with `"w"` | its contents are erased | append with `"a"`; use `"wx"` to avoid overwriting |
| `while (!feof(f))` | the last item is handled twice | check the reading function's return value |
| `fopen("data.txt", 'r')` | a warning: single quotes make a character, not a string | `"r"` |
| `fprintf("...", x)` without the file | a warning: the first argument has the wrong type | `fprintf(f, "...", x)` |
| the file "can't be found" | relative paths are relative to the current directory | check the current directory, or use a full path |

## 13.8 Summary

- `FILE *f = fopen(name, mode);`, check for `NULL`, and `fclose(f);` when done.
- Modes: `"r"` read, `"w"` write (erasing), `"a"` append; add `b` for binary and `x` to avoid overwriting.
- Text I/O: `fputc`/`fgetc`, `fputs`/`fgets`, `fprintf`/`fscanf`; `stdin`/`stdout`/`stderr` are `FILE *`s too, and error messages go to `stderr`.
- Explain failures with `perror` or `strerror(errno)`.
- Reaching the end: check the reading function's return value; don't use `while (!feof(f))`.
- CSV: `fgets` for a line plus `sscanf` for the data; `%[^,]` reads up to a comma.
- Binary: `fwrite`/`fread` copy bytes as they are — fast but not portable.
- `int main(int argc, char *argv[])` receives command-line arguments; `argv[0]` is the program's name.

## 13.9 Exercises

<!-- exercise: wc-lite -->
### Exercise 1: Counting lines, words and bytes

Write a function `void count(FILE *in, long *lines, long *words, long *bytes)` that counts the **lines** (the number of newlines), **words** (runs of non-whitespace characters) and **bytes** (all characters, newlines included) in `in`. `main` counts the input with `count(stdin, ...)` and prints `lines words bytes`. (This is exactly what Linux's `wc` command does.)

```text
Sample input:
hello world
  C is fun
Sample output: 2 5 23
```

<details>
<summary>Hint</summary>

Read character by character with `fgetc`, doing `bytes++` for each and `lines++` for each `'\n'`. Use a variable `in_word` for "currently inside a word": on a non-whitespace character when `in_word` is false, do `words++` and set `in_word` to true; on whitespace (`isspace`), set it to false.
</details>

<!-- exercise: file-roundtrip -->
### Exercise 2: Into a file and back out

Read `n` (1 ≤ n ≤ 1000) and `n` integers. Write them to the file `numbers.txt` (one per line) and close it; then open it again, read with `fscanf` until nothing more can be read, and print how many numbers were read, their sum and their maximum, separated by spaces.

```text
Sample input:
4
3 -1 10 2
Sample output: 4 14 10
```

<details>
<summary>Hint</summary>

Writing: `fprintf(f, "%d\n", x);`. Reading: `while (fscanf(f, "%d", &x) == 1) { ... }`. Check both `fopen`s for `NULL`, and `fclose` both times.
</details>

<!-- exercise: csv-top -->
### Exercise 3: A CSV mark sheet

Read a CSV table from standard input: the first line is the header `name,score`, and each following line is `name,mark` (names are at most 30 characters and may contain spaces; there's at least 1 data line). Print the average mark (2 decimal places) and the name of the person with the highest mark (the first one if there's a tie), separated by a space.

```text
Sample input:
name,score
Ada,95
Linus,88
Grace,91
Sample output: 91.33 Ada
```

<details>
<summary>Hint</summary>

Follow section 13.4, with `stdin` instead of a file: `fgets(line, sizeof line, stdin)`. Update the best mark only on strictly greater (`>`), so that on a tie the first one stays.
</details>

<!-- exercise: grep-lite -->
### Exercise 4: Finding lines that contain a word

The first line is the word to look for; every following line (until the end of the input) is text. Print every text line containing the word as `line number: content` (numbering the text lines from 1). If there are none, print `not found`.

```text
Sample input:
cat
the cat sat
a dog ran
concatenate
Sample output:
1: the cat sat
3: concatenate
```

<details>
<summary>Hint</summary>

First read the word with `fgets` and remove its `'\n'`; then loop with `fgets` over the remaining lines, remove the `'\n'` and test with `strstr(line, word) != NULL`. A variable remembers whether anything was found.
</details>

<!-- exercise: file-predict -->
### Exercise 5: Predict the output

```c
#include <stdio.h>

void show(const char *name) {
    FILE *f = fopen(name, "r");
    if (f == NULL) {
        return;
    }
    char line[50];
    while (fgets(line, sizeof line, f) != NULL) {
        printf("%s", line);
    }
    fclose(f);
}

int main(void) {
    FILE *f = fopen("demo.txt", "w");
    if (f == NULL) {
        return 1;
    }
    fputs("one\n", f);
    fclose(f);

    f = fopen("demo.txt", "a");
    if (f == NULL) {
        return 1;
    }
    fputs("two\n", f);
    fclose(f);
    show("demo.txt");
    printf("---\n");

    f = fopen("demo.txt", "w");
    if (f == NULL) {
        return 1;
    }
    fputs("three\n", f);
    fclose(f);
    show("demo.txt");
    return 0;
}
```

<details>
<summary>Answer and explanation</summary>

`"w"` creates the file and writes `one`; `"a"` appends `two`, so the first display shows two lines. Opening with `"w"` again erases the contents, leaving only `three`. The output:

```text
one
two
---
three
```
</details>

<!-- exercise: open-w -->
### Exercise 6: The `"w"` mode

What happens when you open a file that **already exists and has contents** in `"w"` mode?

- new text is appended after the old contents
- the old contents are erased and writing starts from the beginning
- `fopen` fails and returns `NULL`
- the file is opened read-only

<details>
<summary>Answer and explanation</summary>

`"w"` empties an existing file (its length becomes 0). To append, use `"a"`; to avoid overwriting an existing file, use `"wx"`, which makes `fopen` fail if the file exists.
</details>

## 13.10 Further reading

- [cppreference: file input/output](https://en.cppreference.com/w/c/io) (`fopen`, `fgets`, `fscanf`, `fread` and more)
- [RFC 4180: the CSV format](https://www.rfc-editor.org/rfc/rfc4180) — how to write fields containing commas or quotes
- Next lesson: [Multi-file programs and C23](../14-multifile-c23/en.md)
