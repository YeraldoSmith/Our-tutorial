# 13. 文件

> 🎯 **本课目标**：用 `fopen` 打开文件、`fclose` 关闭文件，分清 `"r"`、`"w"`、`"a"` 等打开模式；用 `fprintf`、`fputs`、`fgetc`、`fgets`、`fscanf` 读写文本文件；明白 `stdin`、`stdout`、`stderr` 也是「文件」，写出对任何 `FILE *` 都适用的函数；处理「文件打不开」之类的错误（`perror`、`strerror(errno)`）；读取 CSV 这样的格式化数据；用 `fwrite`、`fread` 读写二进制文件；接收命令行参数 `argc`、`argv`。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：本课程第 1～12 课

程序运行时的变量都放在内存里，程序一结束就没了。想把数据**保存下来**——游戏存档、成绩表、配置——就要写进**文件**。

## 13.1 打开、写入、关闭、读取

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("hello.txt", "w");       // 打开文件准备写；不存在就创建
    if (f == NULL) {                         // 打开失败时返回 NULL
        perror("hello.txt");                 // 打印失败的原因
        return 1;
    }
    fprintf(f, "你好，文件！\n");             // 和 printf 一样，只是多了第一个参数
    fprintf(f, "1 + 1 = %d\n", 1 + 1);
    fclose(f);                               // 关闭：确保数据真正写进了文件

    f = fopen("hello.txt", "r");             // 再以「读」的方式打开
    if (f == NULL) {
        perror("hello.txt");
        return 1;
    }
    char line[100];
    while (fgets(line, sizeof line, f) != NULL) {   // 一行一行地读，读完了返回 NULL
        printf("读到：%s", line);
    }
    fclose(f);
    return 0;
}
```

- **`FILE *`** 是标准库定义的「文件指针」，代表一个打开的文件。`fopen` 返回它，之后的读写函数都要用它。
- **一定要检查 `fopen` 是否返回 `NULL`**：文件不存在、没有权限、路径写错，都会打开失败。拿 `NULL` 去读写，程序会崩溃。
- **用完要 `fclose`**。写文件时数据会先放在内存的缓冲区里，`fclose` 时才真正写进磁盘；忘了关闭，文件可能是空的或不完整的。
- 文件名 `"hello.txt"` 是**相对路径**，相对于程序运行时的「当前目录」。在本教程的学习程序里，每次运行都在一个临时文件夹里进行。

| 模式 | 作用 |
|------|------|
| `"r"` | 读。文件必须已经存在 |
| `"w"` | 写。文件不存在就创建；**已经存在就清空原有内容** |
| `"a"` | 追加。写入的内容接在文件末尾 |
| `"r+"` / `"w+"` / `"a+"` | 既能读又能写（`"w+"` 同样会清空文件） |
| 加上 `b`，如 `"rb"`、`"wb"` | 二进制模式（13.5 节） |
| 加上 `x`，如 `"wx"` | 文件已经存在时**打开失败**，防止不小心覆盖（C11 起） |

## 13.2 `stdin`、`stdout`、`stderr` 也是文件

程序一启动，就已经有三个打开的 `FILE *`：

| 名字 | 是什么 |
|------|--------|
| `stdin` | 标准输入：键盘，或者学习程序里的输入框 |
| `stdout` | 标准输出：`printf` 写到这里 |
| `stderr` | 标准错误：专门用来输出错误信息 |

`printf(...)` 其实就是 `fprintf(stdout, ...)`，`scanf(...)` 就是 `fscanf(stdin, ...)`。因此，把 `FILE *` 作为参数的函数，**既能处理文件，也能处理键盘输入**：

```c input="apple\nbanana\ncherry"
#include <stdio.h>

int count_lines(FILE *in) {              // 对任何 FILE * 都适用
    int lines = 0;
    int ch;
    while ((ch = fgetc(in)) != EOF) {    // fgetc：读一个字符；读完了返回 EOF
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
    fputs("a\nb\nc\nd\n", f);            // fputs：写一个字符串
    fclose(f);

    f = fopen("abc.txt", "r");
    if (f == NULL) {
        perror("abc.txt");
        return 1;
    }
    printf("abc.txt 有 %d 行\n", count_lines(f));
    fclose(f);

    printf("标准输入有 %d 行\n", count_lines(stdin));
    fprintf(stderr, "这句话写到了 stderr\n");
    return 0;
}
```

错误信息写到 `stderr` 有两个好处：即使输出被重定向到文件里（`./main > out.txt`），错误信息仍然会显示在屏幕上；而且它不会混进正常的输出——学习程序判题时也只比较 `stdout`。

常用的读写函数：

| 写 | 读 | 单位 |
|----|----|------|
| `fputc(c, f)` | `fgetc(f)`（读完返回 `EOF`） | 一个字符 |
| `fputs(s, f)` | `fgets(buf, size, f)`（读完返回 `NULL`） | 一行 / 一个字符串 |
| `fprintf(f, 格式, ...)` | `fscanf(f, 格式, ...)`（返回成功读到的个数） | 格式化数据 |
| `fwrite(p, size, n, f)` | `fread(p, size, n, f)`（返回读到的个数） | 一块字节（二进制） |

## 13.3 处理错误

打开失败时，全局变量 **`errno`**（在 `<errno.h>` 里）记录着错误的编号，`strerror(errno)`（在 `<string.h>` 里）把它变成一句说明，`perror("前缀")` 则直接把「前缀: 说明」输出到 `stderr`。

很多程序在找不到配置文件时会改用默认设置，而不是直接退出：

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void) {
    int volume = 50;                                  // 默认设置
    FILE *f = fopen("settings.txt", "r");
    if (f == NULL) {
        printf("没有读到 settings.txt（%s），使用默认设置\n", strerror(errno));
    } else {
        if (fscanf(f, "%d", &volume) != 1) {          // 检查 fscanf 的返回值
            printf("settings.txt 的格式不对，使用默认设置\n");
            volume = 50;
        }
        fclose(f);
    }
    printf("音量：%d\n", volume);
    return 0;
}
```

（学习程序里会显示 `No such file or directory`，也就是「没有这个文件或目录」。）如果文件是必需的，就输出错误信息并结束程序：`return EXIT_FAILURE;`（`<stdlib.h>` 里定义的「失败」退出码，通常是 1）。

### 读到文件末尾：别用 `while (!feof(f))`

```c norun
while (!feof(f)) {                  // 错误的写法！
    fgets(line, sizeof line, f);
    printf("%s", line);             // 最后一行会被处理两次
}
```

`feof(f)` 只有在**某次读取已经失败之后**才变成真，所以这个循环会多转一圈，用上一次留下的内容再处理一遍。正确做法是**直接检查读取函数的返回值**：`while (fgets(...) != NULL)`、`while (fscanf(...) == 1)`、`while ((ch = fgetc(f)) != EOF)`。

## 13.4 读取格式化数据：CSV

**CSV**（逗号分隔值）是最常见的表格数据格式，Excel 也能打开它：

```text
name,score
Ada,95
Linus,88
Grace,91
```

最稳妥的读法是：用 `fgets` 读一整行，再用 `sscanf` 从这一行里取数据。这样某一行格式不对，也不会影响后面的行：

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
    if (fgets(line, sizeof line, f) == NULL) {         // 跳过第一行（表头）
        fclose(f);
        return 0;
    }
    char name[32];
    int score = 0, count = 0;
    long total = 0;
    while (fgets(line, sizeof line, f) != NULL) {
        if (sscanf(line, "%31[^,],%d", name, &score) == 2) {   // %[^,]：读到逗号为止
            printf("%-6s %d\n", name, score);
            total += score;
            count++;
        }
    }
    fclose(f);
    if (count > 0) {
        printf("平均 %.2f\n", (double)total / count);
    }
    return 0;
}
```

`%31[^,]` 叫**扫描集**：`[^,]` 表示「除了逗号以外的任何字符」，所以它会一直读到逗号为止（最多 31 个字符），名字里有空格也没关系。类似地，`%[^\n]` 读到行尾为止。

## 13.5 二进制文件

`fwrite` 把内存里的字节**原样**写进文件，`fread` 再原样读回来。保存一个结构体数组只需要一行：

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
    size_t written = fwrite(team, sizeof team[0], 3, f);    // 3 个元素，每个 sizeof team[0] 字节
    fclose(f);
    printf("写入了 %zu 个存档，每个 %zu 字节\n", written, sizeof team[0]);

    Player loaded[3];
    f = fopen("save.dat", "rb");
    if (f == NULL) {
        perror("save.dat");
        return 1;
    }
    size_t got = fread(loaded, sizeof loaded[0], 3, f);    // 返回实际读到的元素个数
    fclose(f);
    for (size_t i = 0; i < got; i++) {
        printf("%-4s Lv.%-3d HP %.2f\n", loaded[i].name, loaded[i].level, loaded[i].hp);
    }
    return 0;
}
```

**文本文件和二进制文件怎么选？**

- 二进制文件小、读写快，但人看不懂；而且 `int` 的大小、字节的排列顺序、结构体的填充（第 12 课）在不同的电脑和编译器上可能不一样，换台机器就可能读不出来。
- 文本文件（CSV、JSON……）稍大一些，但人能直接看、能用别的程序处理，跨平台也没问题。
- 要和别人、别的程序交换数据时，优先用文本格式；只给自己的程序读写的临时数据，可以用二进制。

## 13.6 命令行参数

在终端里运行程序时，可以在后面跟上参数，比如 `./wordcount essay.txt -v`。`main` 函数可以这样接收它们：

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("一共 %d 个参数\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    return 0;
}
```

- `argc`（argument count）是参数的个数，`argv`（argument vector）是一个字符串数组。
- `argv[0]` 是程序自己的名字（路径），真正的参数从 `argv[1]` 开始。运行 `./wordcount essay.txt -v` 时，`argc` 是 3。
- 学习程序运行代码时不带参数，所以你只会看到 `argc` 为 1。在终端里编译运行时可以试试带上参数。

很多命令行工具都这样开头：

```c norun
if (argc < 2) {
    fprintf(stderr, "用法：%s 文件名\n", argv[0]);
    return EXIT_FAILURE;
}
FILE *f = fopen(argv[1], "r");
```

## 13.7 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|----------|
| 不检查 `fopen` 的返回值 | 文件打不开时程序崩溃 | `if (f == NULL) { perror(...); ... }` |
| 忘了 `fclose` | 数据没写完整；打开的文件太多 | 每个 `fopen` 都配一个 `fclose` |
| 用 `"w"` 打开了重要的文件 | 原有内容被清空 | 追加用 `"a"`；防止覆盖用 `"wx"` |
| `while (!feof(f))` | 最后一项被处理两次 | 检查读取函数的返回值 |
| `fopen("data.txt", 'r')` | 警告：单引号是字符，不是字符串 | `"r"` |
| `fprintf("...", x)` 忘了文件参数 | 警告：第一个参数类型不对 | `fprintf(f, "...", x)` |
| 文件「找不到」 | 相对路径是相对于当前目录的 | 确认当前目录，或者用完整路径 |

## 13.8 小结

- `FILE *f = fopen(名字, 模式);` 检查 `NULL`；用完 `fclose(f);`。
- 模式：`"r"` 读、`"w"` 写（清空）、`"a"` 追加，加 `b` 为二进制、加 `x` 防覆盖。
- 文本读写：`fputc`/`fgetc`、`fputs`/`fgets`、`fprintf`/`fscanf`；`stdin`/`stdout`/`stderr` 也是 `FILE *`，错误信息写到 `stderr`。
- 出错时用 `perror` 或 `strerror(errno)` 说明原因。
- 读到末尾：检查读取函数的返回值，不要用 `while (!feof(f))`。
- CSV：`fgets` 读一行 + `sscanf` 取数据，`%[^,]` 读到逗号为止。
- 二进制：`fwrite`/`fread` 原样读写字节，快但不跨平台。
- `int main(int argc, char *argv[])` 接收命令行参数，`argv[0]` 是程序名。

## 13.9 练习

<!-- exercise: wc-lite -->
### 练习 1：数行、单词和字节

写函数 `void count(FILE *in, long *lines, long *words, long *bytes)`，统计 `in` 里的**行数**（换行符的个数）、**单词数**（连续的非空白字符算一个单词）和**字节数**（所有字符，包括换行符）。`main` 用 `count(stdin, ...)` 统计输入，输出 `行数 单词数 字节数`。（这正是 Linux 的 `wc` 命令做的事。）

```text
样例输入：
hello world
  C is fun
样例输出：2 5 23
```

<details>
<summary>提示</summary>

用 `fgetc` 一个一个字符地读，每读一个 `bytes++`；遇到 `'\n'` 就 `lines++`。用一个变量 `in_word` 记住「当前是否在单词里」：遇到非空白字符而 `in_word` 为假时，`words++` 并把 `in_word` 设为真；遇到空白（`isspace`）就设为假。
</details>

<!-- exercise: file-roundtrip -->
### 练习 2：存进文件，再读出来

输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数。把它们写进文件 `numbers.txt`（每行一个），关闭文件；再重新打开它，用 `fscanf` 一直读到读不出为止，输出读到的个数、总和、最大值，用空格隔开。

```text
样例输入：
4
3 -1 10 2
样例输出：4 14 10
```

<details>
<summary>提示</summary>

写：`fprintf(f, "%d\n", x);`。读：`while (fscanf(f, "%d", &x) == 1) { ... }`。两次 `fopen` 都要检查 `NULL`，两次都要 `fclose`。
</details>

<!-- exercise: csv-top -->
### 练习 3：CSV 成绩表

从标准输入读入一个 CSV 表格：第一行是表头 `name,score`，之后每行是 `名字,成绩`（名字不超过 30 个字符，可能含空格；至少有 1 行数据）。输出平均分（保留 2 位小数）和最高分的人的名字（同分取先出现的），用空格隔开。

```text
样例输入：
name,score
Ada,95
Linus,88
Grace,91
样例输出：91.33 Ada
```

<details>
<summary>提示</summary>

照着 13.4 节写，只是把文件换成 `stdin`：`fgets(line, sizeof line, stdin)`。最高分用严格大于 `>` 来更新，这样同分时保留先出现的。
</details>

<!-- exercise: grep-lite -->
### 练习 4：查找包含某个词的行

第一行是要找的词，之后的每一行（直到输入结束）是一段文本。输出所有包含这个词的文本行，格式是 `行号: 内容`（行号从文本的第一行开始数，从 1 开始）。如果一行都没有，输出 `not found`。

```text
样例输入：
cat
the cat sat
a dog ran
concatenate
样例输出：
1: the cat sat
3: concatenate
```

<details>
<summary>提示</summary>

先用 `fgets` 读第一行并去掉 `'\n'`；然后循环 `fgets` 读其余各行，去掉 `'\n'` 后用 `strstr(line, word) != NULL` 判断。用一个变量记住有没有找到过。
</details>

<!-- exercise: file-predict -->
### 练习 5：读代码，写输出

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
<summary>答案与解析</summary>

`"w"` 创建文件并写入 `one`；`"a"` 在末尾追加 `two`，所以第一次显示两行。再用 `"w"` 打开时，原有内容被清空，只剩 `three`。输出：

```text
one
two
---
three
```
</details>

<!-- exercise: open-w -->
### 练习 6：`"w"` 模式

用 `"w"` 模式打开一个**已经存在、里面有内容**的文件，会发生什么？

- 新内容追加在原有内容的后面
- 原有内容被清空，从头开始写
- `fopen` 失败，返回 `NULL`
- 文件以只读方式打开

<details>
<summary>答案与解析</summary>

`"w"` 会把已有的文件清空（长度变成 0）。要追加用 `"a"`；不想覆盖已有文件，可以用 `"wx"`，文件已存在时 `fopen` 会失败。
</details>

## 13.10 延伸阅读

- [cppreference：文件输入/输出](https://zh.cppreference.com/w/c/io)（`fopen`、`fgets`、`fscanf`、`fread` 等）
- [RFC 4180：CSV 格式（英文）](https://www.rfc-editor.org/rfc/rfc4180)——字段里有逗号或引号时该怎么写
- 下一课：[多文件程序和 C23](../14-multifile-c23/zh-CN.md)
