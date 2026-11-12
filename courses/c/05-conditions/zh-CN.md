# 5. 条件判断

> 🎯 **本课目标**：用 `if`、`else if`、`else` 让程序根据情况做不同的事；明白**花括号**为什么一定要写；用 `switch` 处理「多选一」，理解 `break` 和「贯穿」；会用三元运算符 `? :`；认识「`if` 后面多一个分号」等经典陷阱。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：本课程第 3、4 课（比较和逻辑运算、scanf）

## 5.1 if 语句

```c input="75"
#include <stdio.h>

int main(void) {
    int score = 0;
    printf("分数：");
    scanf("%d", &score);
    if (score >= 60) {
        printf("及格了！\n");
    }
    printf("程序结束\n");
    return 0;
}
```

- 条件写在**圆括号**里，括号必须有；
- 条件为真（非 0）时，执行花括号里的代码；为假时跳过；
- 不像 Python 那样靠缩进，而是靠 `{ }` 确定哪些代码属于 `if`。

## 5.2 else 和 else if

```c input="85"
#include <stdio.h>

int main(void) {
    int score = 0;
    scanf("%d", &score);
    if (score >= 90) {
        printf("优秀\n");
    } else if (score >= 80) {
        printf("良好\n");
    } else if (score >= 60) {
        printf("及格\n");
    } else {
        printf("不及格\n");
    }
    return 0;
}
```

从上往下依次检查，**只执行第一个成立的分支**，其余的全部跳过。所以顺序很重要：如果把 `score >= 60` 写在最前面，90 分也会被判成「及格」。

## 5.3 一定要写花括号

如果 `if` 后面只有一条语句，C 允许省略花括号。但这是很多 bug 的来源：

```c norun
if (balance < price)
    printf("余额不足\n");
    return 1;                // 看起来属于 if，其实不属于！无论如何都会执行
```

没有花括号时，`if` 只管**紧跟着的一条语句**。第二行虽然缩进了，但它不在 `if` 里面——程序每次都会执行 `return 1;`。编译器（加了 `-Wall`）会提醒：

```text
warning: this 'if' clause does not guard... [-Wmisleading-indentation]
```

2014 年，苹果系统里一个著名的安全漏洞（「goto fail」）就是这样产生的：多了一行没有花括号保护的语句，导致 SSL 证书检查被跳过。**养成习惯：`if`、`else`、`for`、`while` 后面永远写花括号。**

## 5.4 经典陷阱：if 后面的分号

```c norun
if (x > 100);              // 注意这个分号！
{
    printf("x 很大\n");    // 不管 x 是多少都会执行
}
```

`if (x > 100);` 后面的分号本身是一条「空语句」，`if` 管的就是它；下面的花括号成了一个普通的代码块，总会执行。编译器会警告 `suggest braces around empty body in an 'if' statement`。

## 5.5 嵌套和组合条件

`if` 里面可以再写 `if`：

```c input="2024"
#include <stdio.h>

int main(void) {
    int year = 0;
    scanf("%d", &year);
    // 闰年：能被 4 整除但不能被 100 整除，或者能被 400 整除
    if (year % 4 == 0) {
        if (year % 100 != 0 || year % 400 == 0) {
            printf("%d 是闰年\n", year);
        } else {
            printf("%d 不是闰年\n", year);
        }
    } else {
        printf("%d 不是闰年\n", year);
    }
    return 0;
}
```

嵌套太深会难以阅读。很多时候可以用逻辑运算把条件合并成一个：

```c norun
bool leap = (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
```

## 5.6 switch：多选一

当要根据**一个整数（或字符）的值**在多种情况中选一种时，`switch` 比一长串 `else if` 更清楚：

```c input="3"
#include <stdio.h>

int main(void) {
    int day = 0;
    scanf("%d", &day);
    switch (day) {
        case 1:
            printf("星期一\n");
            break;
        case 2:
            printf("星期二\n");
            break;
        case 3:
            printf("星期三\n");
            break;
        case 6:
        case 7:                        // 6 和 7 共用同一段代码
            printf("周末！\n");
            break;
        default:                       // 以上都不是时执行
            printf("其他日子\n");
            break;
    }
    return 0;
}
```

- `switch (表达式)` 后面的值只能是**整数或字符**（不能是小数，也不能是字符串）；
- 每个 `case` 后面是一个常量，要和表达式的值相等才会从这里开始执行；
- **`break` 跳出 `switch`**。如果忘了写，程序会**继续执行下一个 `case` 的代码**，这叫**贯穿**（fall through）；
- 上面 `case 6:` 后面什么都没写，就是故意利用贯穿，让 6 和 7 执行同样的代码。

> [!WARNING]
> 忘写 `break` 是 `switch` 最常见的错误。加了 `-Wextra` 的编译器会警告 `this statement may fall through`。如果是故意要贯穿，C23 可以写 `[[fallthrough]];` 来说明意图。

```c
#include <stdio.h>

int main(void) {
    char op = '*';
    int a = 6, b = 7;
    switch (op) {
        case '+': printf("%d\n", a + b); break;
        case '-': printf("%d\n", a - b); break;
        case '*': printf("%d\n", a * b); break;
        case '/':
            if (b != 0) {
                printf("%d\n", a / b);
            } else {
                printf("不能除以 0\n");
            }
            break;
        default:  printf("不认识的运算符 %c\n", op); break;
    }
    return 0;
}
```

## 5.7 三元运算符 `? :`

`条件 ? 值1 : 值2`：条件为真时得到值1，否则得到值2。适合写简短的「二选一」：

```c
#include <stdio.h>

int main(void) {
    int a = 7, b = 12;
    int bigger = (a > b) ? a : b;
    printf("较大的是 %d\n", bigger);

    int n = 3;
    printf("%d 是%s数\n", n, (n % 2 == 0) ? "偶" : "奇");
    int stock = 0;
    printf("%s\n", stock > 0 ? "有货" : "缺货");
    return 0;
}
```

不要把三元运算符嵌套得太复杂，那样还不如写 `if`。

## 5.8 小结

- `if (条件) { ... } else if (条件) { ... } else { ... }`：从上往下检查，只执行第一个成立的分支。
- **永远写花括号**：没有花括号时 `if` 只管一条语句；缩进对 C 没有意义。
- `if (...);` 后面的分号会让条件失效。
- `switch` 根据整数或字符的值多选一；每个 `case` 结尾写 `break`，否则会贯穿；`default` 处理其他情况。
- `条件 ? a : b` 是简短的二选一。

## 5.9 练习

<!-- exercise: sign -->
### 练习 1：正数、负数还是零

输入一个整数，是正数输出 `positive`，负数输出 `negative`，零输出 `zero`。

```text
样例输入：-5
样例输出：negative
```

<details>
<summary>提示</summary>

`if (n > 0) ... else if (n < 0) ... else ...`
</details>

<!-- exercise: grade -->
### 练习 2：等级

输入一个 0～100 的分数，输出等级：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。

```text
样例输入：85
样例输出：B
```

<details>
<summary>提示</summary>

从高到低依次判断 `score >= 90`、`score >= 80`……
</details>

<!-- exercise: leap -->
### 练习 3：闰年

输入一个年份，是闰年输出 `leap`，否则输出 `common`。（能被 4 整除但不能被 100 整除，或者能被 400 整除的是闰年。）

```text
样例输入：1900
样例输出：common
```

<details>
<summary>提示</summary>

`if ((y % 4 == 0 && y % 100 != 0) || y % 400 == 0)`
</details>

<!-- exercise: calc -->
### 练习 4：用 switch 做计算器

输入一个算式，格式是 `整数 运算符 整数`（运算符是 `+`、`-`、`*`、`/` 之一，前后有空格），输出结果（除法是整数除法）。如果除数是 0 或者运算符不认识，输出 `error`。

```text
样例输入：6 * 7
样例输出：42
```

<details>
<summary>提示</summary>

`scanf("%d %c %d", &a, &op, &b);`——`%c` 前面的空格会跳过空白。然后 `switch (op)`。
</details>

<!-- exercise: braces-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    if (x > 10)
        printf("A\n");
        printf("B\n");
    if (x > 3) printf("C\n"); else printf("D\n");
    switch (x) {
        case 5: printf("E\n");
        case 6: printf("F\n"); break;
        default: printf("G\n");
    }
    return 0;
}
```

<details>
<summary>答案与解析</summary>

第一个 `if` 没有花括号，只管 `printf("A\n")`；`x > 10` 为假，所以不输出 A，但 B 不受 `if` 控制，照样输出。`x > 3` 为真，输出 C。`switch` 从 `case 5` 开始，输出 E 后没有 `break`，贯穿到 `case 6` 输出 F，然后 `break`。所以输出是 B、C、E、F 四行。
</details>

<!-- exercise: semicolon -->
### 练习 6：多出来的分号

`if (x > 100); { printf("big\n"); }` 会怎样？

- 只有 x 大于 100 时才输出 big
- 不管 x 是多少都会输出 big
- 编译失败
- 永远不会输出 big

<details>
<summary>答案与解析</summary>

`if (x > 100);` 的分号是一条空语句，`if` 只管它；后面的花括号是独立的代码块，总会执行。编译器会警告 `suggest braces around empty body`。
</details>

## 5.10 延伸阅读

- [cppreference：if 语句](https://zh.cppreference.com/w/c/language/if)、[switch 语句](https://zh.cppreference.com/w/c/language/switch)
- 「goto fail」漏洞的故事（搜索 Apple goto fail SSL bug）
- 下一课：[循环](../06-loops/zh-CN.md)
