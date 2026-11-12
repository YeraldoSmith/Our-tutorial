# 3. 运算符与表达式

> 🎯 **本课目标**：掌握 C 的算术运算，特别是**整数除法**和取余 `%` 的规则；学会用**类型转换**得到小数结果；理解运算符的优先级；会用 `+=`、`++` 等简写；掌握比较运算和逻辑运算（以及「短路」）；避开 `=` 和 `==` 的经典陷阱；用 `<math.h>` 做数学计算；明白为什么 `0.1 + 0.2 != 0.3`；初步认识位运算。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：本课程第 1、2 课

## 3.1 算术运算

```c
#include <stdio.h>

int main(void) {
    int a = 17, b = 5;
    printf("%d + %d = %d\n", a, b, a + b);
    printf("%d - %d = %d\n", a, b, a - b);
    printf("%d * %d = %d\n", a, b, a * b);
    printf("%d / %d = %d\n", a, b, a / b);      // 整数除法：3，小数部分直接丢掉
    printf("%d %% %d = %d\n", a, b, a % b);     // 取余数：2
    return 0;
}
```

### 整数除法：最常见的「意外」

在 C 里，**两个整数相除，结果还是整数**，小数部分被直接截掉（向 0 取整），不会四舍五入：

| 表达式 | 结果 | 说明 |
|--------|------|------|
| `7 / 2` | `3` | 不是 3.5 |
| `-7 / 2` | `-3` | 向 0 截断（不是 -4） |
| `7 % 3` | `1` | 余数 |
| `-7 % 3` | `-1` | 余数的正负号跟被除数一样 |
| `7.0 / 2` | `3.5` | 只要有一个是小数，就做小数除法 |

这和 Python 很不一样（Python 的 `7 / 2` 是 3.5，`-7 // 2` 是 -4）。

`%` 取余非常有用：判断奇偶（`n % 2 == 0`）、取个位数（`n % 10`）、循环计数（`i % 7` 是星期几）……

## 3.2 类型转换

想让两个整数相除得到小数，需要先把其中一个**转换**成 `double`：

```c
#include <stdio.h>

int main(void) {
    int total = 17, count = 5;
    double wrong = total / count;               // 先做整数除法得到 3，再变成 3.0
    double right = (double)total / count;       // 先把 total 变成 17.0，再做小数除法
    printf("%.2f %.2f\n", wrong, right);        // 3.00 3.40

    double price = 9.99;
    int yuan = (int)price;                      // 小数转整数：直接截掉小数部分（不是四舍五入）
    printf("%d\n", yuan);                       // 9
    return 0;
}
```

`(类型)值` 叫**强制类型转换**（cast）。注意 `wrong` 那一行：等号右边先算完 `total / count`（整数除法，结果 3），**之后**才赋值给 `double` 变量——这时小数部分早就丢了。

C 也会自动做一些转换：`int` 和 `double` 一起运算时，`int` 会自动变成 `double`；把 `double` 赋值给 `int` 变量时会截断小数（加了 `-Wextra` 的编译器有时会警告）。

## 3.3 优先级

和数学一样，先乘除后加减，有括号先算括号：

```c
#include <stdio.h>

int main(void) {
    printf("%d\n", 2 + 3 * 4);       // 14
    printf("%d\n", (2 + 3) * 4);     // 20
    printf("%d\n", 10 - 4 - 3);      // 3：同级从左往右
    printf("%d\n", 2 * 3 % 4);       // 2：* / % 同级，从左往右：6 % 4
    return 0;
}
```

C 的运算符多达几十种，优先级表很长。**不确定的时候就加括号**——括号不会让程序变慢，却能让代码更清楚。

## 3.4 赋值的简写和自增自减

```c
#include <stdio.h>

int main(void) {
    int score = 10;
    score += 5;      // score = score + 5    → 15
    score -= 3;      // score = score - 3    → 12
    score *= 2;      // score = score * 2    → 24
    score /= 5;      // score = score / 5    → 4（整数除法）
    score %= 3;      // score = score % 3    → 1
    printf("%d\n", score);

    int n = 5;
    n++;             // 自增：n = n + 1 → 6
    n--;             // 自减：n = n - 1 → 5
    printf("%d\n", n);

    int a = 5;
    int b = a++;     // 后置：先把 a 原来的值（5）给 b，再把 a 加 1
    int c = ++a;     // 前置：先把 a 加 1（变成 7），再把新值给 c
    printf("a=%d b=%d c=%d\n", a, b, c);   // a=7 b=5 c=7
    return 0;
}
```

> [!WARNING]
> 不要在同一个表达式里对同一个变量做多次修改，比如 `i = i++ + ++i;`。这在 C 里是**未定义行为**，不同编译器的结果可能完全不同。`++` 最好单独写成一句，或者只用在 `for` 循环里（第 6 课）。

## 3.5 比较运算

比较的结果是**真（1）**或**假（0）**：

| 运算符 | 意思 |
|--------|------|
| `==` | 等于（两个等号！） |
| `!=` | 不等于 |
| `<`、`>` | 小于、大于 |
| `<=`、`>=` | 小于等于、大于等于 |

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    printf("%d %d %d\n", x == 5, x != 5, x > 3);    // 1 0 1
    bool adult = x >= 18;
    printf("%d\n", adult);                           // 0
    return 0;
}
```

在 C 里，**0 表示假，任何非 0 的值都表示真**。`bool` 类型只是让意思更清楚。

### 经典陷阱：`=` 和 `==`

```c norun
int x = 5;
if (x = 0) {           // 本想比较 x 是否等于 0，却写成了赋值！
    printf("x 是 0\n");
}
```

`x = 0` 是**赋值**：把 0 存进 `x`，整个表达式的值是 0（假），所以 `if` 永远不执行，而且 `x` 被改掉了。加了 `-Wall` 的编译器会提醒：

```text
warning: suggest parentheses around assignment used as truth value [-Wparentheses]
```

## 3.6 逻辑运算

| 运算符 | 意思 | 例子 |
|--------|------|------|
| `&&` | 并且（两边都真才真） | `age >= 6 && age <= 12` |
| `\|\|` | 或者（有一边真就真） | `day == 6 \|\| day == 7` |
| `!` | 非（取反） | `!finished` |

```c
#include <stdio.h>

int main(void) {
    int age = 10, day = 6;
    printf("%d\n", age >= 6 && age <= 12);     // 1：是小学生年龄
    printf("%d\n", day == 6 || day == 7);      // 1：是周末
    printf("%d\n", !(age > 18));               // 1

    int count = 0;
    if (count != 0 && 100 / count > 5) {      // 短路：count 是 0 时，后半部分根本不会计算
        printf("平均数大于 5\n");
    }
    printf("没有除以 0\n");
    return 0;
}
```

**短路求值**：`&&` 左边是假，整个结果一定是假，右边就**不再计算**；`||` 左边是真，右边也不再计算。上面的例子正是利用这一点，避免了除以 0。

> [!NOTE]
> 不能像数学那样写 `6 <= age <= 12`！C 会先算 `6 <= age`（得到 0 或 1），再拿这个 0 或 1 和 12 比较，结果几乎总是真。要写成 `age >= 6 && age <= 12`。

## 3.7 数学函数：`<math.h>`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    printf("%.4f\n", sqrt(2.0));        // 平方根
    printf("%.1f\n", pow(2, 10));       // 2 的 10 次方：1024.0
    printf("%.1f\n", fabs(-3.5));       // 绝对值（小数用 fabs，整数用 abs，后者在 <stdlib.h>）
    printf("%.1f %.1f %.1f\n", round(2.5), floor(-2.5), ceil(2.1));   // 四舍五入、向下取整、向上取整
    double pi = acos(-1.0);             // 标准 C 没有 π 常量，可以这样算出来
    printf("%.6f\n", pi);
    printf("%.4f\n", sin(pi / 6));      // 三角函数用弧度：sin(30°) = 0.5
    return 0;
}
```

> [!TIP]
> 在 Linux 上用 gcc 编译使用了 `<math.h>` 的程序，命令最后要加 `-lm`（链接数学库）：`gcc main.c -o main -lm`。学习程序已经自动加上了。

## 3.8 小数不精确：`0.1 + 0.2 != 0.3`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    double s = 0.1 + 0.2;
    printf("%.17f\n", s);                       // 0.30000000000000004
    printf("%d\n", s == 0.3);                   // 0：不相等！
    printf("%d\n", fabs(s - 0.3) < 1e-9);       // 1：差距足够小，就认为相等
    return 0;
}
```

计算机用二进制存小数，而 0.1 在二进制里是无限循环小数，只能存一个近似值（就像 1/3 在十进制里只能写成 0.333…）。所以：

- **不要用 `==` 比较两个小数**，而是看它们的差是否足够小；
- **钱不要用小数存**：用整数存「分」，比如 12.50 元存成 1250。

## 3.9 位运算（初步）

C 可以直接操作整数的每一个**二进制位**。这在嵌入式、网络协议、图像处理中很常用：

```c
#include <stdio.h>

int main(void) {
    int a = 13;   // 二进制 1101
    int b = 6;    // 二进制 0110
    printf("%d\n", a & b);    // 按位与：0100 = 4
    printf("%d\n", a | b);    // 按位或：1111 = 15
    printf("%d\n", a ^ b);    // 按位异或：1011 = 11
    printf("%d\n", 1 << 5);   // 左移 5 位：100000 = 32，相当于乘以 2⁵
    printf("%d\n", 40 >> 2);  // 右移 2 位：相当于除以 4 → 10
    printf("%d\n", (a >> 2) & 1);   // 取出 a 的第 2 位（从 0 数起）：1
    return 0;
}
```

别把 `&`（按位与）和 `&&`（逻辑与）、`|` 和 `||` 弄混。本课程的「汇编」课会更深入地讲二进制。

## 3.10 小结

- `+ - * / %`；**整数相除结果是整数**（向 0 截断）；`%` 的符号跟被除数。
- 要小数结果：`(double)a / b`；小数转整数会截断。
- 优先级：先乘除后加减，不确定就加括号。
- `+=`、`-=`、`*=`……；`++`、`--`（前置先加后用，后置先用后加）；不要在一个表达式里多次修改同一个变量。
- 比较 `== != < > <= >=` 结果是 1 或 0；**`=` 是赋值，`==` 才是比较**；0 为假，非 0 为真。
- 逻辑 `&& || !` 会短路；区间判断要写 `a <= x && x <= b`。
- `<math.h>`：`sqrt`、`pow`、`fabs`、`round`、`floor`、`ceil`、三角函数（Linux 编译加 `-lm`）。
- 小数有误差：比较用 `fabs(a - b) < 1e-9`，钱用整数存。
- 位运算 `& | ^ ~ << >>` 直接操作二进制位。

## 3.11 练习

<!-- exercise: divide -->
### 练习 1：整数除法与小数除法

程序里已经有 `int a = 17, b = 5;`。输出三个数，用空格隔开：整数商、余数、精确的商（保留 2 位小数）。

```text
样例输出：3 2 3.40
```

<details>
<summary>提示</summary>

`printf("%d %d %.2f\n", a / b, a % b, (double)a / b);`
</details>

<!-- exercise: clock -->
### 练习 2：秒数换算

程序里已经有 `int total = 3725;`（秒）。用 `/` 和 `%` 把它换算成「时:分:秒」的格式输出，分和秒都要两位数。

```text
样例输出：1:02:05
```

<details>
<summary>提示</summary>

小时 = `total / 3600`；分钟 = `total % 3600 / 60`；秒 = `total % 60`。输出用 `%d:%02d:%02d`。
</details>

<!-- exercise: circle -->
### 练习 3：圆

程序里已经有 `double r = 2.5;`。输出圆的面积和周长，保留 2 位小数，空格隔开。π 用 `acos(-1.0)` 计算。

```text
样例输出：19.63 15.71
```

<details>
<summary>提示</summary>

`#include <math.h>`，`double pi = acos(-1.0);`，面积 `pi * r * r`，周长 `2 * pi * r`。
</details>

<!-- exercise: bits -->
### 练习 4：位运算

输出 `13 & 6`、`13 | 6`、`13 ^ 6`、`1 << 5` 四个值，空格隔开。

```text
样例输出：4 15 11 32
```

<details>
<summary>提示</summary>

直接写在 `printf` 的参数里就行，不需要额外的变量。
</details>

<!-- exercise: ops-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

int main(void) {
    printf("%d %d %d\n", 7 / 2, -7 / 2, 7 % -3);
    printf("%.1f %d\n", 7 / 2 * 1.0, (int)3.99);
    int a = 5;
    int b = a++;
    printf("%d %d\n", a, b);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

第一行：`7 / 2` 是 3；`-7 / 2` 向 0 截断是 -3；`7 % -3` 的符号跟被除数 7，是 1。第二行：`7 / 2` 先算出整数 3，再乘 1.0 得 3.0；`(int)3.99` 截断为 3。第三行：`b = a++` 先把 5 给 b，再把 a 变成 6。所以输出 `3 -3 1`、`3.0 3`、`6 5`。
</details>

<!-- exercise: assign-trap -->
### 练习 6：`=` 还是 `==`？

`int x = 5; if (x = 0) { ... }` 执行后会怎样？

- `if` 里的代码执行，`x` 仍然是 5
- `if` 里的代码不执行，而且 `x` 变成了 0
- 编译失败
- `if` 里的代码执行，`x` 变成了 0

<details>
<summary>答案与解析</summary>

`x = 0` 是赋值，`x` 被改成 0，表达式的值是 0（假），所以 `if` 里的代码不执行。比较要写 `x == 0`。
</details>

## 3.12 延伸阅读

- [cppreference：C 运算符优先级](https://zh.cppreference.com/w/c/language/operator_precedence)
- [cppreference：<math.h> 常用数学函数](https://zh.cppreference.com/w/c/numeric/math)
- [0.30000000000000004.com](https://0.30000000000000004.com/)：各种语言里的小数误差
- 下一课：[输入：scanf](../04-input/zh-CN.md)
