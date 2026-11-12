# 2. 变量与类型

> 🎯 **本课目标**：学会在 C 里**声明**变量（必须写明类型）；认识 `int`、`long long`、`double`、`char`、`bool` 等基本类型和它们占多少字节；用 `printf` 的**格式说明符**（`%d`、`%f`、`%c`……）输出各种类型，并控制宽度和小数位数；明白「字符就是小整数」；会用 `const` 定义常量；知道「未初始化的变量里是垃圾值」；认识 `<stdint.h>` 里固定大小的整数类型。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：本课程第 1 课

## 2.1 声明变量：必须先说清楚类型

在 Python 里，`age = 12` 就创建了一个变量。在 C 里，变量**必须先声明**，并且要写明它的**类型**：

```c
#include <stdio.h>

int main(void) {
    int age = 12;              // 声明一个整数变量 age，初始值是 12
    double height = 1.52;      // 小数（双精度浮点数）
    char grade = 'A';          // 一个字符，用单引号

    printf("年龄：%d\n", age);
    printf("身高：%f 米\n", height);
    printf("等级：%c\n", grade);

    age = age + 1;             // 赋值：改变变量的值（类型不能变）
    printf("明年：%d 岁\n", age);
    return 0;
}
```

为什么要写类型？因为 C 要在**编译时**就知道每个变量需要多少内存、该用什么指令来计算。变量一旦声明，类型就**不能再改变**——不能先存整数、后存字符串（这叫**静态类型**）。

可以一次声明多个同类型的变量：`int a = 1, b = 2, c;`。

## 2.2 基本类型

| 类型 | 含义 | 典型大小 | 例子 |
|------|------|----------|------|
| `int` | 整数 | 4 字节，约 ±21 亿 | `int count = 42;` |
| `long long` | 大整数 | 8 字节，约 ±922 亿亿 | `long long big = 9000000000LL;` |
| `double` | 小数（双精度浮点数） | 8 字节，约 15～16 位有效数字 | `double pi = 3.14159;` |
| `float` | 小数（单精度） | 4 字节，约 6～7 位有效数字 | `float f = 2.5f;` |
| `char` | 一个字符（其实是 1 字节的小整数） | 1 字节 | `char c = 'x';` |
| `bool` | 真 / 假 | 1 字节 | `bool ok = true;` |

> [!NOTE]
> `bool`、`true`、`false` 在 **C23** 标准里是关键字，直接就能用。旧的标准（C99～C17）需要先 `#include <stdbool.h>`。本课程使用 C23（编译选项 `-std=c2x`），旧编译器不支持时加上那一行即可。

用 `sizeof` 可以查看一个类型或变量占多少字节：

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    printf("char:      %zu 字节\n", sizeof(char));
    printf("int:       %zu 字节，范围 %d ~ %d\n", sizeof(int), INT_MIN, INT_MAX);
    printf("long long: %zu 字节，最大 %lld\n", sizeof(long long), LLONG_MAX);
    printf("float:     %zu 字节，约 %d 位有效数字\n", sizeof(float), FLT_DIG);
    printf("double:    %zu 字节，约 %d 位有效数字\n", sizeof(double), DBL_DIG);
    printf("bool:      %zu 字节\n", sizeof(bool));
    return 0;
}
```

`<limits.h>` 和 `<float.h>` 里定义了各种类型的极限值。`sizeof` 的结果要用 `%zu` 输出。

> [!TIP]
> **用哪个类型？** 整数一般用 `int`；可能超过 21 亿的（比如以「分」为单位的大笔金额、时间戳）用 `long long`；小数一般用 `double`（`float` 精度太低，除非是为了省内存）。

## 2.3 printf 的格式说明符

`printf` 的第一个参数是**格式字符串**，里面的 `%d`、`%f` 等是「占位符」，会被后面的参数依次替换：

| 说明符 | 用于 | 例子 → 输出 |
|--------|------|-------------|
| `%d` | `int` | `printf("%d", 42)` → `42` |
| `%lld` | `long long` | `printf("%lld", 9000000000LL)` → `9000000000` |
| `%f` | `double`（默认 6 位小数） | `printf("%f", 3.14)` → `3.140000` |
| `%.2f` | 保留 2 位小数 | `printf("%.2f", 3.14159)` → `3.14` |
| `%e` | 科学计数法 | `printf("%e", 12345.678)` → `1.234568e+04` |
| `%c` | `char` | `printf("%c", 'A')` → `A` |
| `%s` | 字符串 | `printf("%s", "hi")` → `hi` |
| `%x` | 十六进制整数 | `printf("%x", 255)` → `ff` |
| `%zu` | `sizeof` 的结果 | `printf("%zu", sizeof(int))` → `4` |
| `%%` | 百分号本身 | |

还可以控制**宽度**和**对齐**，非常适合输出整齐的表格：

```c
#include <stdio.h>

int main(void) {
    printf("[%5d]\n", 42);       // 宽度 5，右对齐：[   42]
    printf("[%-5d]\n", 42);      // 左对齐：       [42   ]
    printf("[%05d]\n", 42);      // 用 0 补齐：    [00042]
    printf("[%8.3f]\n", 3.14159);// 宽度 8，3 位小数：[   3.142]
    printf("[%-6s|%6s]\n", "ab", "cd");

    printf("%-8s%6s%8s\n", "item", "qty", "price");
    printf("%-8s%6d%8.2f\n", "apple", 3, 4.5);
    printf("%-8s%6d%8.2f\n", "banana", 12, 0.75);
    return 0;
}
```

> [!NOTE]
> `printf` 的宽度按**字节**计算，而一个汉字在 UTF-8 里占 3 个字节、在屏幕上占 2 个字符宽，所以含中文的列很难对齐。需要对齐的表格，表头先用英文，或者自己计算要补多少空格。

> [!WARNING]
> **说明符必须和类型匹配**。`printf("%d", 3.14)` 把一个 `double` 当成 `int` 来读，结果是一个莫名其妙的数（这是「未定义行为」）。好在加了 `-Wall` 的编译器会警告：`format '%d' expects argument of type 'int', but argument 2 has type 'double'`。**看到警告一定要改。**

## 2.4 字符就是小整数

计算机里没有「字母」，只有数字。`char` 类型存的其实是一个 0～127（或 -128～127）的整数，每个数字按照 **ASCII 表**对应一个字符：`'A'` 是 65，`'a'` 是 97，`'0'` 是 48。

```c
#include <stdio.h>

int main(void) {
    char c = 'A';
    printf("%c 的编码是 %d\n", c, c);       // 同一个值，用 %c 显示成字符，用 %d 显示成数字
    printf("%c\n", c + 2);                   // 'A' + 2 = 67 = 'C'
    printf("%c\n", 'a' - 'A' + 'G');         // 大写变小写：g
    char digit = '7';
    printf("字符 '7' 对应的数字：%d\n", digit - '0');   // '7' - '0' = 55 - 48 = 7
    return 0;
}
```

> [!NOTE]
> 一个 `char` 只能存一个字节，所以只能放英文字母、数字和常见符号。汉字、俄文字母在 UTF-8 里要占 2～4 个字节，不能放进一个 `char`。中文等文字要用**字符串**（第 9 课）来存。

## 2.5 常量

值不会改变的量叫**常量**。有几种写法：

```c
#include <stdio.h>

#define MAX_STUDENTS 40              // 预处理器宏：编译前把 MAX_STUDENTS 全部替换成 40

int main(void) {
    const double PI = 3.14159265;    // const：这个变量之后不能再修改
    double r = 2.0;
    printf("面积：%.2f\n", PI * r * r);
    printf("最多 %d 人\n", MAX_STUDENTS);
    // PI = 3;                       // 取消注释会报错：assignment of read-only variable 'PI'
    return 0;
}
```

- `const`：普通变量，但只读。现代 C 更推荐这种写法，因为它有类型，编译器能检查。
- `#define`：**预处理器**做的纯文本替换（第 1 课的「预处理」那一步），没有类型，名字习惯全部大写。
- C23 还新增了 `constexpr`，表示「编译时就能算出来的常量」，比如 `constexpr int SIZE = 10;`。

**字面量**的写法也有讲究：`10` 是 `int`，`10LL` 是 `long long`，`3.14` 是 `double`，`3.14f` 是 `float`，`'x'` 是字符，`"x"` 是字符串（两者不一样！）。整数还可以写成十六进制 `0xFF`、二进制 `0b1010`（C23）；长数字可以用单引号分隔：`1'000'000`（C23）。

## 2.6 变量命名规则

- 只能包含字母、数字和下划线，**不能以数字开头**：`score2` ✓，`2score` ✗；
- 区分大小写：`count` 和 `Count` 是两个不同的变量；
- 不能使用关键字：`int`、`return`、`if`、`while`……；
- 习惯：变量和函数用小写加下划线（`total_score`），宏常量全部大写（`MAX_SIZE`）。

## 2.7 未初始化的变量：垃圾值

在 Python 里，没赋值的变量根本不能用。C 允许你声明一个变量而**不给初始值**，但这时它里面是**之前留在那块内存里的随便什么数**——叫做「垃圾值」：

```c norun
#include <stdio.h>

int main(void) {
    int x;                  // 没有初始值
    printf("%d\n", x);      // 可能是 0，也可能是 32765，每次运行都可能不同
    return 0;
}
```

```text
warning: 'x' is used uninitialized [-Wuninitialized]
```

这是 C 程序里最常见的 bug 之一。**养成习惯：声明变量时总是给一个初始值。**

## 2.8 溢出：整数也有尽头

每种整数类型都有范围。超出范围会发生什么？

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    unsigned char small = 255;          // unsigned：无符号，只能存 0～255
    small = small + 1;
    printf("255 + 1 = %d\n", small);    // 0：无符号整数溢出会「绕回」0

    long long big = INT_MAX;            // 用更大的类型就不会溢出
    big = big + 1;
    printf("INT_MAX + 1 = %lld\n", big);
    return 0;
}
```

- **无符号**整数（`unsigned`）溢出时会像汽车里程表一样「绕回」，这是规定好的行为；
- **有符号**整数（普通的 `int`）溢出是**未定义行为**（undefined behavior）：C 标准不规定结果，程序可能得到奇怪的数，也可能出现更离奇的错误。**要避免它**：估计一下数据会有多大，必要时用 `long long`。

1996 年，欧洲的阿丽亚娜 5 号火箭发射 40 秒后爆炸，原因就是把一个 64 位浮点数转换成 16 位整数时发生了溢出。

## 2.9 固定大小的整数：`<stdint.h>`

`int` 的大小在不同的平台上可能不同（绝大多数电脑上是 4 字节，但一些单片机上只有 2 字节）。需要**确切大小**时（比如处理文件格式、网络数据、硬件寄存器），使用 `<stdint.h>`：

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    int8_t   a = -100;          // 正好 8 位（1 字节）有符号
    uint8_t  b = 200;           // 正好 8 位无符号：0～255
    int32_t  c = 2000000000;    // 正好 32 位
    uint64_t d = 18000000000000000000u;   // 正好 64 位无符号
    printf("%d %d %" PRId32 " %" PRIu64 "\n", a, b, c, d);
    printf("%zu %zu %zu %zu\n", sizeof a, sizeof b, sizeof c, sizeof d);
    return 0;
}
```

`PRId32`、`PRIu64` 是 `<inttypes.h>` 提供的格式说明符，保证在任何平台上都正确。

## 2.10 小结

- C 的变量必须**先声明类型**：`int`、`long long`、`double`、`float`、`char`、`bool`；类型声明后不能改变。
- `sizeof` 查看字节数（用 `%zu` 输出）；`<limits.h>`、`<float.h>` 里有范围。
- `printf` 格式说明符：`%d`、`%lld`、`%f`、`%.2f`、`%c`、`%s`、`%x`、`%zu`；`%5d`、`%-5d`、`%05d` 控制宽度和对齐；**说明符必须和类型匹配**。
- `char` 就是小整数（ASCII）：`'A'` 是 65，`'7' - '0'` 是 7。
- 常量：`const`（推荐）、`#define`、C23 的 `constexpr`。
- **声明时就初始化**，否则是垃圾值；有符号整数溢出是未定义行为；需要确切大小时用 `<stdint.h>`。

## 2.11 练习

<!-- exercise: profile -->
### 练习 1：个人信息

在程序里声明三个变量：`int age = 12;`、`double height = 1.523;`、`char grade = 'B';`，然后**用格式说明符**输出（身高保留 2 位小数）：

```text
样例输出：Age: 12, Height: 1.52 m, Grade: B
```

<details>
<summary>提示</summary>

`printf("Age: %d, Height: %.2f m, Grade: %c\n", age, height, grade);`
</details>

<!-- exercise: sizes -->
### 练习 2：类型的大小

用 `sizeof` 输出 `char`、`int`、`double`、`long long` 四种类型的字节数，用空格隔开。

```text
样例输出：1 4 8 8
```

<details>
<summary>提示</summary>

`printf("%zu %zu %zu %zu\n", sizeof(char), sizeof(int), sizeof(double), sizeof(long long));`
</details>

<!-- exercise: ascii -->
### 练习 3：字符和编码

输出两样东西，用空格隔开：字符 `'Z'` 的 ASCII 编码，以及编码 `97` 对应的字符。

```text
样例输出：90 a
```

<details>
<summary>提示</summary>

同一个值用 `%d` 输出是数字，用 `%c` 输出是字符：`printf("%d %c\n", 'Z', 97);`
</details>

<!-- exercise: table -->
### 练习 4：对齐的表格

用宽度控制输出下面的表格（名字左对齐占 6 格，数量右对齐占 4 格，价格右对齐占 8 格、保留 2 位小数）：

```text
apple    3    4.50
kiwi    12    0.75
```

<details>
<summary>提示</summary>

`printf("%-6s%4d%8.2f\n", "apple", 3, 4.5);`
</details>

<!-- exercise: format-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

int main(void) {
    int a = 7;
    double b = 2.5;
    printf("%d %.1f|%5.2f|%-4d|\n", a, b, b * 2, a);
    printf("%c%c\n", 'a' + 1, 'A' + 25);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

第一行：`%d` 输出 `7`；`%.1f` 输出 `2.5`；`%5.2f` 输出的 `5.00` 只有 4 个字符，宽度是 5，所以前面补 1 个空格；`%-4d` 让 `7` 左对齐占 4 格，后面补 3 个空格。所以是 `7 2.5| 5.00|7   |`。第二行：`'a' + 1` 是 `b`，`'A' + 25` 是 `Z`，输出 `bZ`。
</details>

<!-- exercise: uninit -->
### 练习 6：没有初始值的变量

`int x;` 之后直接 `printf("%d", x);`，会输出什么？

- 一定是 0
- 一个不可预测的「垃圾值」，每次可能不同
- 编译器会拒绝编译
- 程序一定会崩溃

<details>
<summary>答案与解析</summary>

C 不会自动初始化局部变量，里面是那块内存原来留下的任意值。编译器（加了 `-Wall`）通常只给出警告。所以声明时一定要初始化。
</details>

## 2.12 延伸阅读

- [cppreference：算术类型](https://zh.cppreference.com/w/c/language/arithmetic_types)
- [cppreference：printf 格式说明](https://zh.cppreference.com/w/c/io/fprintf)
- [ASCII 码表](https://zh.cppreference.com/w/c/language/ascii)
- 下一课：[运算符与表达式](../03-operators/zh-CN.md)
