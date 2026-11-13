# 14. 多文件程序、预处理器和 C23

> 🎯 **本课目标**：把程序拆分成多个 `.c` 源文件和 `.h` 头文件，理解「声明放头文件、定义放源文件」；写头文件保护；用 `static` 隐藏模块内部的函数和变量，用 `extern` 共享全局变量；了解编译的几个阶段，看懂链接错误；用好预处理器（带参数的宏和它的陷阱、条件编译、`assert`）；了解 Makefile 和 CMake；系统地认识 **C23** 的新特性。最后看看学完 C 之后可以往哪里走。
>
> ⏱ **预计用时**：100 分钟　｜　📋 **前置知识**：本课程第 1～13 课

## 14.1 为什么要拆成多个文件

到目前为止，每个程序都只有一个文件。真实的项目动辄几万、几百万行，全写在一个文件里，没人能看得过来，改一行也要把整个程序重新编译。所以要把程序按功能拆成**模块**：每个模块一个 `.c` 文件（实现）加一个 `.h` 文件（接口）。

下面是一个三个文件的小项目。学习程序一次只能运行一个文件，所以这几段代码请在自己电脑上新建文件夹来试（编译命令在 14.3 节）。

```c norun
// geometry.h —— 模块的「接口」：告诉别人这个模块提供什么
#ifndef GEOMETRY_H
#define GEOMETRY_H

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b);
double triangle_area(Point a, Point b, Point c);

#endif
```

```c norun
// geometry.c —— 模块的「实现」
#include <math.h>
#include "geometry.h"

static double cross(Point o, Point a, Point b) {    // static：只在本文件内使用
    return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
}

double distance(Point a, Point b) {
    return hypot(a.x - b.x, a.y - b.y);
}

double triangle_area(Point a, Point b, Point c) {
    return fabs(cross(a, b, c)) / 2;
}
```

```c norun
// main.c —— 使用这个模块
#include <stdio.h>
#include "geometry.h"

int main(void) {
    Point a = {0, 0}, b = {4, 0}, c = {0, 3};
    printf("AB = %.1f\n", distance(a, b));
    printf("面积 = %.1f\n", triangle_area(a, b, c));
    return 0;
}
```

- **头文件（`.h`）只放声明**：函数原型、类型定义（`struct`、`typedef`、`enum`）、宏、`extern` 变量声明。谁要用这个模块，就 `#include` 它的头文件。
- **源文件（`.c`）放定义**：函数体、全局变量的定义。源文件也要包含自己的头文件，这样编译器能检查声明和定义是否一致。
- `#include "geometry.h"` 用**双引号**：先在当前文件夹里找；`#include <stdio.h>` 用**尖括号**：只在系统的头文件目录里找。

## 14.2 头文件保护、`static` 和 `extern`

### 头文件保护

`#include` 其实就是把头文件的内容**原样复制**到这里。如果一个 `.c` 文件间接地把 `geometry.h` 包含了两次，`Point` 就会被定义两次，编译器报错 `redefinition of ...`。上面头文件开头和结尾的三行就是**头文件保护**（include guard）：

```c norun
#ifndef GEOMETRY_H      // 如果还没定义过 GEOMETRY_H……
#define GEOMETRY_H      // ……就定义它，并处理下面的内容
/* 头文件的内容 */
#endif                  // 第二次包含时，GEOMETRY_H 已经定义过，整段被跳过
```

每个头文件都要写。宏名一般用文件名的大写形式。很多编译器也支持更短的 `#pragma once`，效果一样（但它不是 C 标准的一部分）。

### `static`：模块的私有部分

在函数外面，`static` 的意思是「**只在本文件里可见**」。上面的 `cross` 是 `geometry.c` 内部的辅助函数，加上 `static`，别的文件就看不到它，也不会和别的文件里同名的函数冲突。这和面向对象语言里的「私有」很像：头文件里公开的是接口，`static` 的是实现细节。

（注意：函数**里面**的 `static` 局部变量是另一个意思——值在多次调用之间保留，第 7 课讲过。）

### `extern`：共享全局变量

如果确实需要一个被多个文件共用的全局变量，在**头文件**里用 `extern` 声明它，在**某一个**源文件里定义它：

```c norun
// config.h
extern int verbose;           // 声明：这个变量在别处定义

// config.c
#include "config.h"
int verbose = 0;              // 定义：只能有一处
```

如果把 `int verbose = 0;` 直接写进头文件，每个包含它的 `.c` 文件都会有一份定义，链接时报错 `multiple definition of 'verbose'`。不过还是那句话：全局变量能少用就少用。

## 14.3 编译、链接和链接错误

在终端里编译多文件程序，把所有 `.c` 文件都列出来：

```bash
gcc -std=c23 -Wall -Wextra -g main.c geometry.c -o app -lm
./app
```

（GCC 14 以前的版本用 `-std=c2x`。`-lm` 链接数学库，`hypot`、`fabs` 在里面。）这一条命令其实分好几步完成：

```text
main.c ──预处理──▶ 展开 #include、宏 ──编译──▶ main.o ─┐
                                                      ├──链接──▶ app（可执行文件）
geometry.c ──预处理──▶ ……         ──编译──▶ geometry.o ─┘   ↑ 还会链接标准库
```

1. **预处理**：处理所有 `#` 开头的指令——把头文件内容贴进来、展开宏、处理条件编译。
2. **编译**：把每个 `.c` 文件**单独**翻译成机器码，得到目标文件（`.o`）。编译 `main.c` 时，编译器只知道 `distance` 的原型，不知道它的实现。
3. **链接**：链接器把所有 `.o` 文件和标准库拼在一起，把每个函数调用和它的定义对上号。

所以有两类不同的错误：**编译错误**（语法、类型问题，会指出文件和行号）和**链接错误**（编译都通过了，但拼装时找不到或找到多个定义）：

| 链接错误 | 原因 |
|----------|------|
| `undefined reference to 'distance'` | 编译命令里漏了 `geometry.c`；或者函数名拼错了；或者只写了原型没写函数体 |
| `multiple definition of 'verbose'` | 变量或函数的**定义**写进了头文件，被多个 `.c` 文件包含 |
| `undefined reference to 'sqrt'`（某些系统上） | 忘了加 `-lm` |

分开编译的好处是：只改了 `geometry.c`，就只需要重新编译它，再重新链接——大项目能节省大量时间。下面的构建工具就是帮你自动做这件事的。

## 14.4 构建工具：Make 和 CMake

每次手敲一长串编译命令很麻烦。**Make** 根据一个叫 `Makefile` 的文件，只重新编译改过的部分：

```makefile
CC = gcc
CFLAGS = -std=c2x -Wall -Wextra -g

app: main.o geometry.o
	$(CC) main.o geometry.o -o app -lm

main.o: main.c geometry.h
	$(CC) $(CFLAGS) -c main.c

geometry.o: geometry.c geometry.h
	$(CC) $(CFLAGS) -c geometry.c
```

在终端里输入 `make` 就会构建 `app`。每条规则是「目标: 依赖」，下一行（**必须用 Tab 缩进**）是生成目标的命令；依赖的文件比目标新时，才会重新执行。

更大的项目一般用 **CMake**，它能为不同的系统和编译器（包括 Windows 上的 Visual Studio）生成构建文件：

```cmake
cmake_minimum_required(VERSION 3.20)
project(geometry C)
set(CMAKE_C_STANDARD 23)
add_executable(app main.c geometry.c)
target_link_libraries(app m)
```

然后执行 `cmake -B build` 和 `cmake --build build`。（在 Windows 上去掉最后一行，那里的数学函数不需要单独链接。）

## 14.5 预处理器

### 带参数的宏和它的陷阱

`#define` 不仅能定义常量，还能定义「带参数的宏」。但宏只是**文字替换**，不是函数，很容易出错：

```c
#include <stdio.h>

#define SQUARE_BAD(x) x * x
#define SQUARE(x) ((x) * (x))           // 每个参数、整个结果都加上括号
#define MAX(a, b) ((a) > (b) ? (a) : (b))

static inline int square(int x) {       // 更好的选择：inline 函数
    return x * x;
}

int main(void) {
    printf("%d\n", SQUARE_BAD(1 + 2));  // 变成 1 + 2 * 1 + 2，结果是 5！
    printf("%d\n", SQUARE(1 + 2));      // ((1 + 2) * (1 + 2)) = 9
    printf("%d\n", MAX(3, 7));          // 7
    printf("%d\n", square(1 + 2));      // 9：函数先算出参数，再计算
    return 0;
}
```

- 宏的每个参数、整个表达式都要**加括号**。
- 参数可能被计算两次：`MAX(i++, j)` 会让 `i` 加两次。
- 所以能用函数就用函数，`static inline` 函数一样快，而且有类型检查；能用 `const` 或 `constexpr` 定义常量，就别用 `#define`。

### 条件编译和调试信息

```c
#include <stdio.h>
#include <assert.h>

#define DEBUG 1

int average(const int a[], int n) {
    assert(n > 0);                       // 断言：n 必须大于 0，否则程序立刻停下并报告
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
#if DEBUG
    printf("[调试] %s 第 %d 行：sum = %ld\n", __func__, __LINE__, sum);
#endif
    return (int)(sum / n);
}

int main(void) {
    int data[] = {3, 5, 10};
    printf("平均：%d\n", average(data, 3));
    return 0;
}
```

- `#if` / `#ifdef` / `#else` / `#endif`：根据条件决定**编译哪些代码**。常用于调试代码、针对不同操作系统的代码（`#ifdef _WIN32`）。
- `__FILE__`、`__LINE__`、`__func__`：当前的文件名、行号和函数名，适合输出调试信息。
- **`assert(条件)`**（`<assert.h>`）：条件为假时，打印出错的文件、行号和条件，然后中止程序。它用来检查「绝不应该发生」的情况，是很好的自我检查工具。正式发布时用 `-DNDEBUG` 编译，所有 `assert` 都会被去掉。把 `average(data, 3)` 改成 `average(data, 0)` 试试看。

## 14.6 C23：现代的 C

C 语言的标准大约每隔几年更新一次：C89、C99、C11、C17，以及最新的 **C23**。本课程从第 1 课起就在用 C23（编译选项 `-std=c2x`，较新的编译器也可以写 `-std=c23`）。把它的主要新特性汇总一下：

```c
#include <stdio.h>

constexpr int MAX_PLAYERS = 4;                   // 编译时常量，可以用作数组长度
enum Color : unsigned char { RED, GREEN, BLUE }; // 指定枚举的底层类型

[[nodiscard]] int checked_half(int x, bool *ok) {   // [[nodiscard]]：返回值不应被忽略
    *ok = (x % 2 == 0);
    return x / 2;
}

int main(void) {
    int scores[MAX_PLAYERS] = {};            // {}：空初始化，全部为 0
    auto total = 0;                          // auto：根据初始值推断类型（这里是 int）
    int *p = nullptr;                        // nullptr：真正的空指针常量
    bool ok = false;                         // bool、true、false 是关键字了
    int million = 1'000'000;                 // 数字分隔符
    int flags = 0b1010'0101;                 // 二进制字面量
    typeof(million) copy = million;          // typeof：取一个表达式的类型
    static_assert(sizeof(int) >= 4);         // 编译时断言（C23 可以省略消息）
    enum Color c = BLUE;

    int half = checked_half(10, &ok);
    printf("%d %d %d %d\n", scores[0], total, half, ok);
    printf("%d %d %d\n", million, flags, copy);
    printf("p %s空指针，c 占 %zu 字节\n", p == nullptr ? "是" : "不是", sizeof c);
    return 0;
}
```

| 特性 | 说明 |
|------|------|
| `bool`、`true`、`false` | 成为关键字，不再需要 `<stdbool.h>` |
| `nullptr` | 空指针常量，比 `NULL` 更安全（`NULL` 可能只是整数 0） |
| `constexpr` | 编译时常量，比 `#define` 安全，比 `const` 用途更广 |
| `auto`、`typeof` | 让编译器推断类型、取得表达式的类型 |
| `{}` 空初始化 | `int a[10] = {};` 全部清零 |
| `0b1010`、`1'000'000` | 二进制字面量、数字分隔符 |
| `[[nodiscard]]`、`[[maybe_unused]]`、`[[deprecated]]` | 标准的「属性」，帮编译器发现问题 |
| `enum E : 类型` | 指定枚举的底层整数类型 |
| `static_assert(条件)` | 编译时检查，消息可以省略 |
| `<stdckdint.h>` | `ckd_add(&r, a, b)` 等：带溢出检查的整数运算 |
| `<stdbit.h>` | `stdc_count_ones` 等位操作函数 |
| `#embed "file.bin"` | 把一个文件的内容直接嵌入数组 |
| `strdup`、`memccpy` | 进入了标准库 |

（后面三行比较新：`<stdckdint.h>` 和 `<stdbit.h>` 需要 GCC 14 以上，`#embed` 需要 GCC 15 或 Clang 19 以上。如果你的编译器报错说不认识 `nullptr`、`bool` 之类的写法，说明它太旧了，请升级到 GCC 13+ 或 Clang 16+。）

## 14.7 学完 C 之后

恭喜你学完了 C 语言课程！你已经掌握了变量、控制流、函数、数组、字符串、指针、动态内存、结构体、文件和多文件程序——这正是理解计算机如何工作的基础。接下来可以：

- **做项目**：命令行记账本、通讯录、文本冒险游戏、简单的 shell、迷宫求解器、图片格式转换器（BMP 格式很简单）……做项目是巩固知识最好的办法。
- **学数据结构和算法**：栈、队列、哈希表、二叉树、图、排序和搜索——用 C 亲手实现一遍，理解会深刻得多。
- **学本教程的其他课程**：C++ 在 C 的基础上加了类、模板和丰富的标准库；Rust 用编译期检查解决了 C 的内存安全问题；汇编课会带你看到 C 代码最终变成的机器指令。
- **读好书**：Jens Gustedt 的《Modern C》（作者是 C 标准委员会成员，内容覆盖 C23，有免费的英文电子版）；Robert C. Seacord 的《Effective C》；经典的《C 程序设计语言》（K&R）。

## 14.8 小结

- 模块 = `.h`（声明：原型、类型、宏、`extern`）+ `.c`（定义）；源文件包含自己的头文件。
- 每个头文件都写头文件保护（`#ifndef X_H` / `#define X_H` / `#endif`，或 `#pragma once`）。
- 文件作用域的 `static` 让函数和变量只在本文件可见；共享全局变量：头文件 `extern` 声明，一个源文件定义。
- 编译分预处理、编译、链接；`undefined reference` 和 `multiple definition` 是链接错误。
- 多文件编译：`gcc main.c geometry.c -o app`；用 Make 或 CMake 自动构建。
- 宏是文字替换：参数和结果都加括号，能用函数和 `constexpr` 就别用宏；`#if` 条件编译，`assert` 检查不应发生的情况。
- C23：`bool`、`nullptr`、`constexpr`、`auto`、`typeof`、`{}`、`0b`、`'` 分隔符、属性、`static_assert` 等。

## 14.9 练习

<!-- exercise: stack-module -->
### 练习 1：栈模块

学习程序只能运行一个文件，所以这道题把一个「模块」的三部分写在同一个文件里：接口（原型）、实现、使用。请补全实现部分的四个函数，用 `static` 的数组 `items` 和计数器 `top` 实现一个**栈**（后放进去的先拿出来）：`stack_push` 放入一个数（栈满时忽略）；`stack_pop` 取出并返回最后放入的数，栈空时返回 -1；`stack_empty` 判断是否为空；`stack_size` 返回元素个数。`main` 已经写好，它读入 `push x`、`pop`、`size` 这样的命令。

```text
样例输入：push 3 push 5 pop size pop pop
样例输出：
5
1
3
-1
```

<details>
<summary>提示</summary>

`push`：`if (top < STACK_MAX) { items[top] = value; top++; }`。`pop`：栈空返回 -1，否则 `top--; return items[top];`。`top` 本身就是元素个数。
</details>

<!-- exercise: find-negative -->
### 练习 2：用 `nullptr` 表示「没找到」

写函数 `int *find_negative(int a[], int n)`：返回数组里第一个负数的地址，没有负数时返回 `nullptr`。输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数，如果找到了，输出它的下标和值（下标用指针相减求出），用空格隔开；否则输出 `none`。

```text
样例输入：
5
4 0 -7 2 -1
样例输出：2 -7
```

<details>
<summary>提示</summary>

找到后：`int *p = find_negative(a, n); if (p != nullptr) printf("%td %d\n", p - a, *p);`。
</details>

<!-- exercise: to-binary -->
### 练习 3：二进制

输入一个整数 `x`（0 ≤ x < 2³¹），输出它的二进制表示（没有前导 0；`x` 为 0 时输出 `0`）和其中 1 的个数，用空格隔开。

```text
样例输入：10
样例输出：1010 2
```

<details>
<summary>提示</summary>

从最高位往最低位检查：`for (int bit = 30; bit >= 0; bit--)`，用 `(x >> bit) & 1` 取出这一位；遇到第一个 1 之前的 0 不输出。也可以先用 `x % 2`、`x / 2` 从低位往高位取，存进数组后倒着输出。
</details>

<!-- exercise: macro-predict -->
### 练习 4：读代码，写输出

```c
#include <stdio.h>

#define DOUBLE_BAD(x) x + x
#define DOUBLE(x) ((x) + (x))
#define SQUARE_BAD(x) x * x

int main(void) {
    printf("%d\n", 3 * DOUBLE_BAD(2));
    printf("%d\n", 3 * DOUBLE(2));
    printf("%d\n", SQUARE_BAD(1 + 2));
    return 0;
}
```

<details>
<summary>答案与解析</summary>

宏只是文字替换。`3 * DOUBLE_BAD(2)` 变成 `3 * 2 + 2`，等于 8；`3 * DOUBLE(2)` 变成 `3 * ((2) + (2))`，等于 12；`SQUARE_BAD(1 + 2)` 变成 `1 + 2 * 1 + 2`，等于 5。输出：

```text
8
12
5
```
</details>

<!-- exercise: link-error -->
### 练习 5：链接错误

`area` 函数定义在 `geometry.c` 里，`main.c` 包含了 `geometry.h` 并调用 `area`。用 `gcc main.c -o app` 编译时，报错 `undefined reference to 'area'`。原因是什么？

- `geometry.h` 没有写头文件保护
- 编译命令里漏掉了 `geometry.c`
- `main.c` 没有 `#include <stdio.h>`
- `area` 函数应该写成 `static`

<details>
<summary>答案与解析</summary>

`undefined reference` 是链接错误：`main.c` 编译通过了，但链接时找不到 `area` 的定义，因为 `geometry.c` 根本没参与编译。应该写 `gcc main.c geometry.c -o app`。如果把 `area` 写成 `static`，它只在 `geometry.c` 里可见，反而会出同样的错。
</details>

<!-- exercise: header-guard -->
### 练习 6：头文件保护

头文件开头的 `#ifndef GEOMETRY_H` / `#define GEOMETRY_H` 和结尾的 `#endif` 有什么用？

- 让头文件编译得更快
- 防止同一个头文件在一个源文件里被包含多次，导致类型等被重复定义
- 允许在头文件里写函数的定义
- 禁止其他文件包含这个头文件

<details>
<summary>答案与解析</summary>

第一次包含时 `GEOMETRY_H` 还没定义，内容被处理，同时定义了 `GEOMETRY_H`；之后再被包含时，`#ifndef` 条件为假，整段内容被跳过，就不会出现 `redefinition` 错误。
</details>

## 14.10 延伸阅读

- [cppreference：预处理器](https://zh.cppreference.com/w/c/preprocessor)、[C23 新特性一览（英文）](https://en.cppreference.com/w/c/23)
- [Modern C（Jens Gustedt，英文，免费）](https://gustedt.gitlabpages.inria.fr/modern-c/)
- [CMake 官方教程（英文）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)
- 返回：[C 语言课程目录](../)
