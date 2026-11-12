# 10. 指针

> 🎯 **本课目标**：理解内存和**地址**；会定义指针、用 `&` 取地址、用 `*` 访问指针指向的变量；用指针参数写出真正能交换两个变量的 `swap`，让函数「返回」多个结果；认识 `NULL` 和野指针；理解指针和数组的关系（指针运算、`a[i]` 就是 `*(a + i)`）；用指针处理字符串（`strchr`、`strtol`），知道字符串字面量不能修改；了解 `void *`、函数指针，用标准库的 `qsort` 排序。
>
> ⏱ **预计用时**：110 分钟　｜　📋 **前置知识**：本课程第 1～9 课

指针是 C 语言最有特色、也最让初学者紧张的部分。其实它只有一个核心概念：**地址**。慢慢来，每个例子都运行一下、改一改。

## 10.1 内存和地址

可以把内存想象成一条很长很长的街道，街上每栋房子（每个**字节**）都有一个门牌号，这个门牌号就叫**地址**。每个变量都住在某个地址上，`int` 占 4 栋连着的房子，`double` 占 8 栋。

```c
#include <stdio.h>

int main(void) {
    int age = 20;
    double price = 9.5;
    printf("age 的值是 %d，地址是 %p\n", age, (void *)&age);
    printf("price 的值是 %.1f，地址是 %p\n", price, (void *)&price);
    printf("age 占 %zu 字节，price 占 %zu 字节\n", sizeof age, sizeof price);
    return 0;
}
```

- `&age` 读作「age 的地址」，`&` 叫**取地址运算符**。你在 `scanf("%d", &age)` 里早就见过它了。
- `%p` 以十六进制输出地址；按规定要先转换成 `(void *)` 类型（10.8 节解释 `void *`）。
- 每次运行，地址很可能都不一样——这是操作系统为了安全故意打乱的，不用在意具体的数值。

## 10.2 指针变量

**指针**就是「存放地址的变量」。

```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int *p = &x;                  // p 是「指向 int 的指针」，里面存着 x 的地址

    printf("x = %d\n", x);
    printf("p 里存的地址：%p\n", (void *)p);
    printf("*p = %d\n", *p);      // *p：顺着这个地址找到的那个 int，也就是 x

    *p = 25;                      // 通过指针修改 x
    printf("x = %d\n", x);        // 25

    int y = 7;
    p = &y;                       // 指针可以改为指向别的变量
    *p += 1;
    printf("y = %d\n", y);        // 8
    return 0;
}
```

```text
       x                    p
    ┌──────┐          ┌──────────┐
    │  25  │ ◀─────── │ x 的地址  │
    └──────┘          └──────────┘
```

| 写法 | 意思 |
|------|------|
| `int *p;` | 定义一个指针 `p`，它可以存放某个 `int` 变量的地址 |
| `&x` | 取 `x` 的地址 |
| `*p` | **解引用**：`p` 指向的那个变量本身，可以读也可以写 |

`*` 有两种用法，别搞混：**定义时**的 `int *p` 表示「p 是指针」；**使用时**的 `*p` 表示「p 指向的东西」。

> [!TIP]
> `int *p` 和 `int* p` 是一样的，但推荐前一种写法。因为 `int* a, b;` 里只有 `a` 是指针，`b` 只是一个普通的 `int`！定义两个指针要写 `int *a, *b;`。

指针的类型要和它指向的变量一致：`int` 的地址存在 `int *` 里，`double` 的地址存在 `double *` 里。

## 10.3 指针作为函数参数

第 7 课里的 `swap` 失败了，因为函数拿到的是副本。现在把**地址**传进去：

```c
#include <stdio.h>

void swap(int *a, int *b) {        // 参数是两个地址
    int t = *a;
    *a = *b;
    *b = t;
}

void min_max(const int arr[], int n, int *min, int *max) {   // 用指针「返回」两个结果
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
    swap(&p, &q);                       // 传的是地址
    printf("p = %d, q = %d\n", p, q);   // p = 2, q = 1：成功了！

    int data[] = {4, -2, 9, 0};
    int lo = 0, hi = 0;
    min_max(data, 4, &lo, &hi);
    printf("最小 %d，最大 %d\n", lo, hi);
    return 0;
}
```

C 仍然是值传递——只不过这次复制的是**地址**。副本和原来的地址指向同一个地方，所以通过 `*a` 修改的就是 `main` 里的 `p`。

现在 `scanf("%d", &x)` 也就清楚了：`scanf` 要把读到的数存进 `x`，所以需要 `x` 的地址。

## 10.4 `NULL` 和野指针

**没有初始化的指针**里是一个垃圾地址，叫**野指针**。通过它读写，就是往一个随机的地方写数据——可能崩溃，也可能悄悄改坏别的数据：

```c norun
int *p;          // 没有初始化：野指针
*p = 5;          // 未定义行为！
```

规则很简单：

- 指针定义时就初始化：要么指向一个变量（`&x`），要么设成 **`NULL`**——一个特殊的「空地址」，表示「哪儿也不指」；
- 不确定指针是否有效时，用之前先检查 `if (p != NULL)`；
- 通过 `NULL` 读写会立刻崩溃（`Segmentation fault`）。打开内存检查（第 8 课）后，它会告诉你是哪一行。

很多函数用返回 `NULL` 表示「没找到」或「失败」：

```c
#include <stdio.h>

int *find(int arr[], int n, int target) {   // 找到就返回那个元素的地址，找不到返回 NULL
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
        *p = 80;                     // 通过指针直接修改数组里的那个元素
    }
    if (find(data, 3, 42) == NULL) {
        printf("没找到 42\n");
    }
    printf("%d %d %d\n", data[0], data[1], data[2]);   // 3 80 5
    return 0;
}
```

> [!WARNING]
> 函数**不能返回自己的局部变量的地址**：函数一结束，局部变量就消失了，那个地址变成了「悬空指针」。上面的 `find` 返回的是调用者的数组里的元素地址，这是可以的。GCC 发现返回局部变量地址时会警告 `function returns address of local variable`。

## 10.5 指针和数组

数组和指针关系非常密切：

```c
#include <stdio.h>

int main(void) {
    int a[5] = {10, 20, 30, 40, 50};
    int *p = a;                       // 数组名自动变成首元素的地址，等于 &a[0]

    printf("%d %d\n", *p, *(p + 1));  // 10 20：p + 1 指向下一个元素
    printf("%d %d\n", a[3], *(a + 3));// a[i] 就是 *(a + i)

    for (int *q = a; q < a + 5; q++) {   // 用指针遍历数组
        printf("%d ", *q);
    }
    printf("\n");

    int *r = &a[4];
    printf("r 和 p 相隔 %td 个元素\n", r - p);                      // 4
    printf("sizeof a = %zu，sizeof p = %zu\n", sizeof a, sizeof p);   // 20 和 8
    return 0;
}
```

- **数组名在大多数表达式里会自动变成首元素的地址**。所以 `int *p = a;` 不用写 `&`，`scanf("%s", name)` 也不用写 `&`。
- **指针运算**：`p + 1` 不是地址加 1，而是**前进一个元素**（对 `int *` 来说前进 4 个字节）。两个指向同一数组的指针相减，得到相隔的元素个数（用 `%td` 输出）。
- `a[i]` 只是 `*(a + i)` 的简写。
- 函数参数 `int a[]` 和 `int *a` **完全一样**——这就是第 8 课里函数内 `sizeof a` 得到 8 的原因：那里的 `a` 其实是个指针。

但数组不等于指针：数组是一整块存放数据的内存，`sizeof a` 是整个数组的大小，而且 `a = p;`、`a++` 都是不允许的；指针只是一个存放地址的变量（通常 8 字节），可以随时改为指向别处。

## 10.6 指针和字符串

字符串函数大量使用指针。`strchr`、`strstr` 返回的就是指向找到的位置的指针：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(void) {
    char text[] = "name=Ada;lang=C";
    char *eq = strchr(text, '=');              // 指向第一个 '='
    if (eq != NULL) {
        printf("'=' 的下标是 %td\n", eq - text);   // 指针相减：4
        printf("它后面是 %s\n", eq + 1);           // 从下一个字符开始的字符串
    }

    const char *greeting = "hello";            // 指向字符串字面量：只能读，不能改
    printf("%s 的第一个字母是 %c\n", greeting, greeting[0]);

    // strtol：把字符串转成整数，并通过 end 告诉你在哪里停下
    const char *input = "123abc";
    char *end;
    long value = strtol(input, &end, 10);      // 10 表示十进制
    printf("读到 %ld，剩下 \"%s\"\n", value, end);
    if (end == input) {
        printf("一个数字也没读到\n");
    } else if (*end != '\0') {
        printf("数字后面还有别的字符\n");
    }
    return 0;
}
```

- `strtol(字符串, &end, 进制)` 比 `atoi` 严谨：`end` 指向第一个没被转换的字符。`end == input` 说明一个数字都没有；`*end == '\0'` 说明整个字符串都是数字。还有对应小数的 `strtod`。
- **字符串字面量不能修改**。`"hello"` 存放在只读的内存里，下面的代码会崩溃：

```c norun
char *s = "hello";
s[0] = 'J';          // 段错误！
```

| 写法 | 含义 |
|------|------|
| `char s[] = "hello";` | **数组**：把字面量**复制**一份到数组里，可以随便修改 |
| `const char *s = "hello";` | **指针**：指向只读的字面量。加上 `const`，万一写了 `s[0] = 'J'`，编译器会直接报错 |

## 10.7 `const` 和指针

函数参数里的 `const` 是一种承诺：

- `const int *p`（或第 8 课的 `const int a[]`）：**不能通过 `p` 修改**它指向的数据。读可以，写会编译报错 `assignment of read-only location`。
- `int *const p`：`p` **自己**不能再改为指向别处，但可以通过它修改数据。这个较少用到。

经验：函数只读取指针指向的数据时，就给参数加上 `const`。调用者一看就知道「我的数据不会被改」，编译器也会帮你检查。

## 10.8 `void *`、函数指针和 `qsort`

### 函数指针

函数也有地址，存放函数地址的变量叫**函数指针**：

```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int mul(int a, int b) {
    return a * b;
}

int main(void) {
    int (*op)(int, int) = add;     // op：指向「接收两个 int、返回 int」的函数
    printf("%d\n", op(3, 4));      // 7
    op = mul;                      // 换成另一个函数
    printf("%d\n", op(3, 4));      // 12
    return 0;
}
```

写法看起来有点怪：`int (*op)(int, int)` 中，`(*op)` 说明 `op` 是指针，后面的 `(int, int)` 和前面的 `int` 说明它指向什么样的函数。函数名本身（不带括号）就代表函数的地址。

### 用 `qsort` 排序

标准库 `<stdlib.h>` 的 `qsort` 能排序**任何类型**的数组，而且比冒泡排序快得多。因为它不知道元素是什么类型，所以需要你提供一个**比较函数**，告诉它两个元素谁在前：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int compare_ints(const void *a, const void *b) {
    int x = *(const int *)a;          // 先把 void * 转成 int *，再取值
    int y = *(const int *)b;
    return (x > y) - (x < y);         // x < y 返回 -1，相等返回 0，x > y 返回 1
}

int compare_words(const void *a, const void *b) {
    return strcmp((const char *)a, (const char *)b);
}

int main(void) {
    int nums[] = {42, 7, 19, -3, 7};
    int n = sizeof nums / sizeof nums[0];
    qsort(nums, n, sizeof nums[0], compare_ints);    // 数组、个数、每个元素的大小、比较函数
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

- **`void *`** 是「通用指针」：可以存放任何类型数据的地址，但不知道那里是什么类型，所以**不能直接用 `*` 取值**，要先转换成具体的指针类型，比如 `(const int *)a`。
- 比较函数的格式是固定的：`int 名字(const void *a, const void *b)`。`a` 应该排在 `b` 前面就返回负数，后面就返回正数，相等返回 0。
- 为什么不直接写 `return x - y;`？当 `x`、`y` 很大或很小时，相减可能**溢出**（第 3 课），得到错误的符号。`(x > y) - (x < y)` 永远不会溢出。
- 想从大到小排序，把比较的结果反过来即可，比如 `return compare_ints(b, a);`。

## 10.9 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|----------|
| `int *p; *p = 1;`（野指针） | 崩溃或改坏数据 | 定义时就初始化为 `&x` 或 `NULL` |
| 通过 `NULL` 读写 | 段错误 | 使用前检查 `if (p != NULL)` |
| 返回局部变量的地址 | 悬空指针 | 返回值本身，或让调用者传指针进来，或用 `malloc`（第 11 课） |
| 忘了 `&` 或多写了 `*` | 警告 `makes pointer from integer without a cast` | 分清 `x`、`&x`、`p`、`*p` |
| `int* a, b;` | `b` 不是指针 | `int *a, *b;` |
| 修改字符串字面量 | 段错误 | 要修改就用 `char s[] = "..."`；不修改就写 `const char *` |
| 在函数里用 `sizeof` 求数组参数的长度 | 得到指针的大小（8） | 把长度作为参数传进去 |

## 10.10 小结

- 每个变量都有地址；`&x` 取地址，`%p` 输出地址。
- `int *p = &x;` 定义指向 `x` 的指针；`*p` 就是 `x` 本身，可以读写。
- 传指针给函数，函数就能修改调用者的变量（`swap`），也能「返回」多个结果。
- 指针要初始化；`NULL` 表示「不指向任何东西」，用前检查；不要返回局部变量的地址。
- 数组名会变成首元素地址；`p + 1` 前进一个元素；`a[i]` 就是 `*(a + i)`；函数参数 `int a[]` 就是 `int *a`。
- `strchr`/`strstr` 返回指针；`strtol` 用 `end` 指针报告停在哪里；字符串字面量只读，用 `const char *` 指向它。
- `const int *p` 承诺不通过 `p` 修改数据。
- `void *` 是通用指针，要先转换类型才能取值；函数名就是函数的地址；`qsort` + 比较函数 `int cmp(const void *a, const void *b)` 可以排序任何数组。

## 10.11 练习

<!-- exercise: swap-ptr -->
### 练习 1：真正的交换

补全函数 `void swap(int *a, int *b)`，交换两个指针指向的整数。`main` 已经写好：读入两个整数，交换后输出。

```text
样例输入：3 8
样例输出：8 3
```

<details>
<summary>提示</summary>

和 10.3 节一样：用一个临时变量 `int t = *a;`。
</details>

<!-- exercise: split-time -->
### 练习 2：拆分时间

写函数 `void split_time(int total, int *h, int *m, int *s)`，把总秒数拆成时、分、秒，通过指针「返回」。输入总秒数（0 ≤ total < 1000000），按 `时:分:秒` 输出，分和秒都占两位。

```text
样例输入：3661
样例输出：1:01:01
```

<details>
<summary>提示</summary>

`*h = total / 3600;`，`*m = total % 3600 / 60;`，`*s = total % 60;`。输出用 `printf("%d:%02d:%02d\n", h, m, s);`。
</details>

<!-- exercise: qsort-desc -->
### 练习 3：从大到小

输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数，用 `qsort` 把它们**从大到小**排序后输出，用空格隔开。

```text
样例输入：
5
3 -1 4 1 5
样例输出：5 4 3 1 -1
```

<details>
<summary>提示</summary>

写一个比较函数，在 `x > y` 时返回负数（让大的排在前面）：`return (x < y) - (x > y);`。
</details>

<!-- exercise: count-substr -->
### 练习 4：子串出现了几次

第一行输入一段英文，第二行输入一个单词。输出这个单词在这段文字里出现了几次（**允许重叠**，区分大小写）。

```text
样例输入：
banana
ana
样例输出：2
```

<details>
<summary>提示</summary>

用 `strstr` 反复查找：找到一次就计数，然后从找到的位置的**下一个字符**开始继续找：`p = strstr(p + 1, word);`，直到返回 `NULL`。两行都用 `fgets` 读，并去掉 `'\n'`。
</details>

<!-- exercise: reverse-ptr -->
### 练习 5：用两个指针反转字符串

写函数 `void reverse(char *s)`，用两个指针（一个从头、一个从尾往中间走）把字符串原地反转。输入一个英文单词，输出反转后的结果。

```text
样例输入：pointer
样例输出：retniop
```

<details>
<summary>提示</summary>

`char *left = s;`、`char *right = s + strlen(s) - 1;`，当 `left < right` 时交换 `*left` 和 `*right`，然后 `left++`、`right--`。注意空字符串的情况。
</details>

<!-- exercise: ptr-predict -->
### 练习 6：读代码，写输出

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
<summary>答案与解析</summary>

`p` 一开始指向 `a[1]`，把它改成 20；`p++` 后指向 `a[2]`，加 10 变成 13。`p - a` 是 2。`q` 指向 `x`，`*q = *q * 2` 就是 `x = x * 2`。输出：

```text
1 20 13 4
2
10
```
</details>

<!-- exercise: ptr-value -->
### 练习 7：哪个是 3？

```c norun
int x = 3;
int *p = &x;
```

下面哪个表达式的值是 `3`？

- `p`
- `*p`
- `&p`
- `&x`

<details>
<summary>答案与解析</summary>

`*p` 是 `p` 指向的变量，也就是 `x`，值为 3。`p` 和 `&x` 都是 `x` 的地址，`&p` 是指针变量 `p` 自己的地址。
</details>

## 10.12 延伸阅读

- [cppreference：指针声明](https://zh.cppreference.com/w/c/language/pointer)、[qsort](https://zh.cppreference.com/w/c/algorithm/qsort)、[strtol](https://zh.cppreference.com/w/c/string/byte/strtol)
- [Python Tutor（选 C 语言）](https://pythontutor.com/c.html)——把本课的代码贴进去，一步一步地看指针指向哪里
- 下一课：[动态内存](../11-memory/zh-CN.md)
