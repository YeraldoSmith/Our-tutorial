# 9. 字符串

> 🎯 **本课目标**：明白 C 的字符串就是「以 `'\0'` 结尾的字符数组」；会输出、读入字符串（读单词用 `scanf`，读整行用 `fgets`），解决「残留换行符」问题；会用 `<string.h>` 里的常用函数（`strlen`、`strcpy`、`strcat`、`strcmp`、`strstr`），知道它们的危险以及更安全的 `snprintf`；用 `<ctype.h>` 逐个处理字符；在字符串和数字之间转换；理解汉字在 UTF-8 里占好几个字节；使用字符串数组。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：本课程第 1～8 课

## 9.1 字符串就是以 `'\0'` 结尾的字符数组

C 没有专门的「字符串类型」。字符串就是一个 `char` 数组，最后放一个特殊的字符 `'\0'`（数值为 0 的字符，叫**空字符**）表示「到这里结束」：

```c
#include <stdio.h>

int main(void) {
    char a[6] = {'H', 'e', 'l', 'l', 'o', '\0'};   // 一个一个字符地写
    char b[] = "Hello";                            // 同样的内容：编译器自动在末尾加 '\0'

    printf("%s %s\n", a, b);                       // %s 输出字符串
    printf("b 占 %zu 个字节\n", sizeof b);         // 6：5 个字母 + 1 个 '\0'

    for (int i = 0; b[i] != '\0'; i++) {           // 一个一个字符地走，直到 '\0'
        printf("[%c]", b[i]);
    }
    printf("\n");

    b[0] = 'J';                                    // 字符数组里的字符可以修改
    printf("%s\n", b);                             // Jello
    b[2] = '\0';                                   // 在中间放一个 '\0'……
    printf("%s\n", b);                             // Je：字符串到这里就「结束」了
    return 0;
}
```

```text
         b[0] b[1] b[2] b[3] b[4] b[5]
        ┌────┬────┬────┬────┬────┬────┐
"Hello" │ H  │ e  │ l  │ l  │ o  │ \0 │
        └────┴────┴────┴────┴────┴────┘
```

- `"Hello"`（双引号）是字符串，`'H'`（单引号）是一个字符——它们完全不同（第 2 课）。
- `printf("%s", b)`、`strlen(b)` 等所有字符串函数，都是**从头一直读到 `'\0'`** 为止。所以 `'\0'` 至关重要：没有它，函数就会一直读到数组外面去（越界，第 8 课）。
- 因此存放 n 个字符的字符串，数组至少要 **n + 1** 个 `char`。

## 9.2 读入字符串

**读一个单词**用 `scanf("%Ns", 数组)`（第 4 课）：读到空格或换行为止；`N` 写「数组长度 − 1」，防止输入太长写出界；数组名前**不加** `&`。

**读一整行**（可以包含空格）用 `fgets`：

```c input="20\nAda Lovelace"
#include <stdio.h>
#include <string.h>

int main(void) {
    int age = 0;
    char name[50];

    printf("年龄：");
    scanf("%d", &age);

    int ch;                                        // 丢掉这一行剩下的部分（包括换行符）
    while ((ch = getchar()) != '\n' && ch != EOF) {
    }

    printf("\n全名：");
    if (fgets(name, sizeof name, stdin) == NULL) { // 最多读 49 个字符；读不到时返回 NULL
        return 1;
    }
    name[strcspn(name, "\n")] = '\0';              // 去掉末尾的换行符

    printf("\n%s，%d 岁\n", name, age);
    return 0;
}
```

这段代码里有三个要点：

1. **`fgets(数组, 大小, stdin)`** 最多读「大小 − 1」个字符，绝不会写出界。它会把行末的**换行符 `'\n'` 也读进来**，所以通常紧接着用 `name[strcspn(name, "\n")] = '\0';` 把它去掉（`strcspn` 返回第一个 `'\n'` 的位置；没有换行时返回字符串长度，这一句也是安全的）。
2. **残留的换行符**：`scanf("%d", &age)` 读完 `20` 就停了，后面的换行符还留在输入里。如果不处理，`fgets` 会立刻读到这个换行符，得到一个**空行**。那个 `while` 循环用 `getchar()` 一个一个字符地读，直到把这一行剩下的东西（包括换行符）都丢掉。
3. `ch` 要定义成 `int` 而不是 `char`：`getchar()` 在输入结束时返回 `EOF`（通常是 -1），它不是任何一个字符，需要用 `int` 才能和普通字符区分开。

> [!WARNING]
> 你可能在老书里见过 `gets(name)`。它不检查长度，输入一长就写出界，**已经从 C 语言里删除了**。永远用 `fgets`。

## 9.3 `<string.h>`：字符串函数

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[50] = "hello";
    printf("长度 %zu，数组大小 %zu\n", strlen(s), sizeof s);   // 5 和 50

    char t[50];
    strcpy(t, s);                          // 复制：t = s 是不行的（数组不能整体赋值）
    strcat(t, ", world");                  // 拼接：接到 t 的末尾
    printf("%s\n", t);                     // hello, world

    if (strcmp(s, "hello") == 0) {         // 比较内容：相等时返回 0
        printf("s 的内容是 hello\n");
    }
    if (strcmp("apple", "banana") < 0) {   // 负数：前一个按字典序排在前面
        printf("apple 排在 banana 前面\n");
    }
    if (strstr(t, "world") != NULL) {      // 找子串：找不到返回 NULL
        printf("t 里有 world\n");
    }
    if (strchr(t, ',') != NULL) {          // 找一个字符
        printf("t 里有逗号\n");
    }
    return 0;
}
```

| 函数 | 作用 |
|------|------|
| `strlen(s)` | 字符串长度，**不算** `'\0'` |
| `strcpy(d, s)` | 把 `s` 复制到 `d`（`d` 必须足够大！） |
| `strcat(d, s)` | 把 `s` 接到 `d` 的末尾（`d` 必须足够大！） |
| `strcmp(a, b)` | 比较：相等返回 0；`a` 排在前面返回负数，排在后面返回正数 |
| `strncmp(a, b, n)` | 只比较前 `n` 个字符，比如判断开头：`strncmp(s, "http", 4) == 0` |
| `strchr(s, c)` / `strstr(s, t)` | 查找字符 `c` / 子串 `t`；找不到返回 `NULL`（返回值其实是一个指针，第 10 课细讲） |
| `snprintf(d, size, 格式, ...)` | 像 `printf` 一样，但「打印」到字符串 `d` 里，最多写 `size - 1` 个字符 |

### `strcpy` 和 `strcat` 很危险

它们**不知道目标数组有多大**，只管一直写。目标放不下时就会写出界——这正是「缓冲区溢出」漏洞的来源。GCC 能发现一部分这样的错误：

```text
warning: '__builtin_memcpy' writing 8 bytes into a region of size 4 overflows the destination
```

拼接、格式化字符串时，更安全的办法是 `snprintf`：它最多写 `size - 1` 个字符，再补上 `'\0'`，放不下就**截断**，绝不越界：

```c
#include <stdio.h>

int main(void) {
    char first[] = "Grace", last[] = "Hopper";
    char full[50];
    snprintf(full, sizeof full, "%s %s", first, last);   // 空间足够
    printf("[%s]\n", full);

    char tiny[8];
    snprintf(tiny, sizeof tiny, "%s %s", first, last);   // 只放得下 7 个字符
    printf("[%s]\n", tiny);                              // [Grace H]：被安全地截断了
    return 0;
}
```

（编译时 GCC 会对第二个 `snprintf` 发出 `output may be truncated` 的警告——它提醒你「这里会被截断」，这正是我们想演示的。）

## 9.4 比较字符串：不能用 `==`

```c norun
char answer[10] = "yes";
if (answer == "yes") { ... }          // 错！
```

```text
warning: comparison with string literal results in unspecified behavior [-Waddress]
```

`answer == "yes"` 比较的是两个字符串**在内存里的位置**，而不是内容，结果几乎总是「不相等」。比较内容**一定要用 `strcmp`**：

```c norun
if (strcmp(answer, "yes") == 0) { ... }   // 对
```

同理，复制用 `strcpy`（或 `snprintf`），不能用 `=`；`strcmp` 相等时返回的是 **0**，别写成 `if (strcmp(a, b))`——那表示「不相等」。

## 9.5 逐个处理字符：`<ctype.h>`

`<ctype.h>` 提供了判断和转换单个字符的函数：

```c
#include <stdio.h>
#include <ctype.h>

int main(void) {
    char s[] = "Hello, World 2026!";
    int letters = 0, digits = 0, spaces = 0, others = 0;

    for (int i = 0; s[i] != '\0'; i++) {
        unsigned char c = (unsigned char)s[i];   // ctype 函数要求传入 unsigned char 范围的值
        if (isalpha(c)) {
            letters++;
        } else if (isdigit(c)) {
            digits++;
        } else if (isspace(c)) {
            spaces++;
        } else {
            others++;
        }
        s[i] = (char)toupper(c);                 // 转成大写；不是字母的保持不变
    }
    printf("%s\n", s);
    printf("字母 %d，数字 %d，空白 %d，其他 %d\n", letters, digits, spaces, others);
    return 0;
}
```

| 函数 | 判断 / 作用 |
|------|-------------|
| `isalpha(c)` | 是英文字母吗 |
| `isdigit(c)` | 是数字 `0`～`9` 吗 |
| `isalnum(c)` | 是字母或数字吗 |
| `isspace(c)` | 是空白（空格、`\t`、`\n` 等）吗 |
| `isupper(c)` / `islower(c)` | 是大写 / 小写字母吗 |
| `toupper(c)` / `tolower(c)` | 转成大写 / 小写（不是字母就原样返回） |

> [!NOTE]
> 为什么要先转成 `unsigned char`？在很多平台上 `char` 是有符号的，汉字等非英文字符的字节会变成**负数**，而把负数传给 `isalpha` 等函数是未定义行为。转换一下就安全了。

字符就是小整数（第 2 课），所以可以对字符做算术。数字字符 `'0'`～`'9'` 的编码是连续的，`c - '0'` 就是数字的值；字母也是连续的，`'a' + 2` 就是 `'c'`：

```c
#include <stdio.h>

int main(void) {
    char number[] = "9527";
    int sum = 0;
    for (int i = 0; number[i] != '\0'; i++) {
        sum += number[i] - '0';          // '9' - '0' = 9
    }
    printf("各位数字之和：%d\n", sum);    // 23

    for (char c = 'a'; c <= 'e'; c++) {
        printf("%c", c - 'a' + 'A');     // 小写变大写的「手工版」
    }
    printf("\n");
    return 0;
}
```

## 9.6 字符串和数字互相转换

- **字符串 → 数字**：用 `sscanf`，它和 `scanf` 一样，只是从字符串里读。它的返回值告诉你成功读到了几个值。
- **数字 → 字符串**：用 `snprintf`。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char text[] = "42 3.5";
    int n = 0;
    double x = 0;
    if (sscanf(text, "%d %lf", &n, &x) == 2) {        // 读到了 2 个值才算成功
        printf("%d %.1f\n", n * 2, x * 2);
    }
    if (sscanf("abc", "%d", &n) != 1) {
        printf("\"abc\" 不是整数\n");
    }

    char date[20];
    snprintf(date, sizeof date, "%d-%02d-%02d", 2026, 3, 7);
    printf("%s，长度 %zu\n", date, strlen(date));      // 2026-03-07，长度 10
    return 0;
}
```

你也许会见到 `atoi("42")`。它用起来简单，但遇到 `"abc"` 会悄悄返回 0，没法知道出了错；更严谨的 `strtol` 要用到指针，第 10 课再介绍。

## 9.7 汉字和 UTF-8

`strlen` 数的是**字节数**，不是「字数」。现在通用的 **UTF-8** 编码里，英文字母占 1 个字节，大部分汉字占 3 个字节，表情符号占 4 个字节：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[] = "C语言";
    printf("strlen = %zu\n", strlen(s));               // 7 = 1 + 3 + 3

    int chars = 0;
    for (int i = 0; s[i] != '\0'; i++) {
        if (((unsigned char)s[i] & 0xC0) != 0x80) {   // 不是「后续字节」，就是一个新字符的开始
            chars++;
        }
    }
    printf("字符数 = %d\n", chars);                     // 3
    return 0;
}
```

UTF-8 的规则：一个字符的第一个字节之后的「后续字节」，二进制都以 `10` 开头（`0x80`～`0xBF`）。`& 0xC0` 取出最高两位（位运算，第 3 课），不等于 `0x80` 的字节就是一个新字符的开头。

所以处理中文时要小心：按字节反转字符串会把汉字拆坏；`char s[4]` 只放得下 1 个汉字；`toupper` 之类的函数只对英文字母有用。

## 9.8 字符串数组

多个字符串可以放进一个**二维字符数组**，每一行是一个字符串：

```c
#include <stdio.h>
#include <string.h>

#define COUNT 4
#define LEN 20

int main(void) {
    char names[COUNT][LEN] = {"Carol", "alice", "Bob", "Dave"};

    // 冒泡排序（第 8 课）：比较用 strcmp，交换用 strcpy
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

结果里 `alice` 排在了最后！因为 `strcmp` 按字符的**编码**比较，所有大写字母（`A`～`Z` 是 65～90）都排在小写字母（`a`～`z` 是 97～122）前面。想忽略大小写，可以先把两个字符串都转成小写再比较。

## 9.9 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|----------|
| `char s[5] = "Hello";` 没给 `'\0'` 留位置 | 字符串没有结尾，函数读出界 | 长度至少 n + 1，或写 `char s[] = "Hello";` |
| `s = "abc";` | 编译错误：数组不能整体赋值 | `strcpy(s, "abc");` 或 `snprintf` |
| `if (s == "abc")` | 比较的是位置，几乎总是假 | `if (strcmp(s, "abc") == 0)` |
| `scanf("%s", s);` 不写长度 | 输入太长时写出界 | `scanf("%49s", s);` |
| `fgets` 之后忘了去掉 `'\n'` | 比较、输出时多一个换行 | `s[strcspn(s, "\n")] = '\0';` |
| `scanf("%d")` 之后直接 `fgets` | 读到一个空行 | 先用 `getchar` 循环丢掉这一行剩下的部分 |
| `strcpy` / `strcat` 目标太小 | 缓冲区溢出 | 算好大小，或用 `snprintf` |

## 9.10 小结

- C 的字符串是以 `'\0'` 结尾的 `char` 数组；`"..."` 会自动加 `'\0'`；存 n 个字符要 n + 1 的空间。
- 读单词：`scanf("%49s", s)`；读整行：`fgets(s, sizeof s, stdin)`，再去掉 `'\n'`；`scanf` 和 `fgets` 混用时，先丢掉残留的换行符。
- `<string.h>`：`strlen`、`strcpy`、`strcat`、`strcmp`（相等返回 0）、`strncmp`、`strchr`、`strstr`；`strcpy`/`strcat` 不检查大小，拼接和格式化优先用 `snprintf`。
- 比较内容用 `strcmp`，不能用 `==`；赋值用 `strcpy`，不能用 `=`。
- `<ctype.h>`：`isalpha`、`isdigit`、`isspace`、`toupper`……参数先转 `unsigned char`；`c - '0'` 得到数字字符的值。
- 字符串 ↔ 数字：`sscanf`（检查返回值）和 `snprintf`。
- UTF-8 里一个汉字通常占 3 个字节，`strlen` 数的是字节。
- 字符串数组：`char names[N][LEN]`。

## 9.11 练习

<!-- exercise: char-classes -->
### 练习 1：字符分类统计

输入一行英文（可能包含空格），分别统计其中的**字母**、**数字**、**空格**和**其他字符**的个数，用空格隔开输出。（行末的换行符不算。）

```text
样例输入：Hello World 123!
样例输出：10 3 2 1
```

<details>
<summary>提示</summary>

用 `fgets` 读一行，去掉 `'\n'`，再用 `isalpha`、`isdigit` 和 `c == ' '` 判断每个字符。
</details>

<!-- exercise: palindrome -->
### 练习 2：回文

输入一个英文单词（不超过 100 个字母），如果它正着读和倒着读一样（**不区分大小写**），输出 `yes`，否则输出 `no`。

```text
样例输入：Level
样例输出：yes
```

<details>
<summary>提示</summary>

用两个下标 `i = 0` 和 `j = strlen(s) - 1` 从两头往中间走，比较 `tolower(s[i])` 和 `tolower(s[j])`。
</details>

<!-- exercise: caesar -->
### 练习 3：凯撒密码

第一行输入一个整数 `k`（0 ≤ k ≤ 25），第二行输入一句英文。把每个字母向后移动 `k` 位（`z` 后面回到 `a`），大写字母仍是大写、小写仍是小写，其他字符不变，输出加密后的句子。

```text
样例输入：
3
Hello, World!
样例输出：Khoor, Zruog!
```

<details>
<summary>提示</summary>

小写字母：`'a' + (c - 'a' + k) % 26`，大写字母同理。别忘了 `scanf` 读完 `k` 之后要丢掉残留的换行符，再用 `fgets` 读句子。
</details>

<!-- exercise: word-count -->
### 练习 4：数单词

输入一行英文，单词之间用一个或多个空格隔开（行首、行尾也可能有空格），输出单词的个数。空行输出 `0`。

```text
样例输入：  the quick  brown fox 
样例输出：4
```

<details>
<summary>提示</summary>

数「单词的开头」：当前字符不是空格，并且它是第一个字符或者前一个字符是空格时，计数加 1。
</details>

<!-- exercise: title-case -->
### 练习 5：首字母大写

输入一行英文，把每个单词的第一个字母变成大写，其余字母变成小写，输出结果。单词之间用空格隔开。

```text
样例输入：hello wORLD from c
样例输出：Hello World From C
```

<details>
<summary>提示</summary>

和练习 4 一样判断「单词的开头」：开头用 `toupper`，其他位置用 `tolower`。
</details>

<!-- exercise: str-predict -->
### 练习 6：读代码，写输出

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
<summary>答案与解析</summary>

`strlen` 是 3（不算 `'\0'`），`sizeof` 是整个数组 10。把 `s[1]` 改成 `'\0'` 后，字符串在那里就结束了，只剩 `a`，长度 1。`"cat"` 和 `"car"` 前两个字符相同，第三个 `t` 比 `r` 大，所以 `strcmp` 返回正数，`> 0` 为真，输出 1。输出：

```text
3 10
a 1
1
```
</details>

<!-- exercise: str-compare -->
### 练习 7：比较字符串

要判断字符数组 `a` 和 `b` 里的字符串内容是否相同，应该怎么写？

- `if (a == b)`
- `if (strcmp(a, b) == 0)`
- `if (strcmp(a, b))`
- `if (strlen(a) == strlen(b))`

<details>
<summary>答案与解析</summary>

`strcmp` 在两个字符串相同时返回 0。`a == b` 比较的是位置；`if (strcmp(a, b))` 恰好反了，表示「不相同」；长度相同不代表内容相同。
</details>

## 9.12 延伸阅读

- [cppreference：空终止字节字符串](https://zh.cppreference.com/w/c/string/byte)（`<string.h>` 和 `<ctype.h>` 的所有函数）
- [UTF-8（维基百科）](https://zh.wikipedia.org/wiki/UTF-8)——看看 UTF-8 是怎样用 1～4 个字节表示一个字符的
- 下一课：[指针](../10-pointers/zh-CN.md)
