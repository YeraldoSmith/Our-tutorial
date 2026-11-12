# 4. 输入：scanf

> 🎯 **本课目标**：用 `scanf` 读取用户输入的整数、小数、字符和单词；理解为什么变量前要加 `&`；记住「`scanf` 读 `double` 用 `%lf`」；检查 `scanf` 的**返回值**，应对用户输错；解决读字符时「残留的换行符」问题；会一直读到输入结束为止。
>
> ⏱ **预计用时**：50 分钟　｜　📋 **前置知识**：本课程第 2、3 课

## 4.1 第一个 scanf

`printf` 负责输出，`scanf` 负责输入。它们的格式说明符几乎一样：

```c input="12"
#include <stdio.h>

int main(void) {
    int age = 0;
    printf("你几岁了？");
    scanf("%d", &age);                 // 读一个整数，存进 age
    printf("明年你就 %d 岁了！\n", age + 1);
    return 0;
}
```

运行后，程序会停下来等你输入。输入 `12` 并按回车，程序继续执行。

### 为什么要加 `&`？

`&age` 读作「age 的**地址**」。每个变量都存放在内存的某个位置，这个位置有一个编号，就是地址。`scanf` 需要知道**把读到的数存到哪里**，所以要把地址告诉它。

```c
#include <stdio.h>

int main(void) {
    int age = 12;
    printf("age 的值是 %d\n", age);
    printf("age 的地址是 %p\n", (void *)&age);     // %p 输出地址（每次运行可能不同）
    return 0;
}
```

`printf` 只需要知道值，所以不加 `&`；`scanf` 要修改变量，所以必须加 `&`。第 10 课学指针时会彻底弄明白这件事。**忘了 `&` 是新手最常见的错误之一，程序通常会直接崩溃。**

## 4.2 一次读多个值

```c input="3 4"
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    printf("输入两个整数：");
    scanf("%d %d", &a, &b);
    printf("%d + %d = %d\n", a, b, a + b);
    return 0;
}
```

`%d` 会自动跳过前面的空格、Tab 和换行，所以用户输入 `3 4`、`3    4`，或者分两行输入 `3` 和 `4`，结果都一样。

## 4.3 scanf 的格式说明符

| 类型 | `scanf` 用 | `printf` 用 |
|------|-----------|-------------|
| `int` | `%d` | `%d` |
| `long long` | `%lld` | `%lld` |
| `double` | **`%lf`** | `%f` |
| `float` | `%f` | `%f` |
| `char` | `%c`（或 `" %c"`，见 4.5） | `%c` |
| 一个单词（字符串） | `%s`（**不加 `&`**） | `%s` |

> [!WARNING]
> **`scanf` 读 `double` 必须用 `%lf`**，用 `%f` 会读出错误的值；而 `printf` 输出 `double` 用 `%f`。这是 C 里一个很容易弄错的不对称之处。

```c input="1.75 60"
#include <stdio.h>

int main(void) {
    double height = 0, weight = 0;
    printf("身高（米）和体重（千克）：");
    scanf("%lf %lf", &height, &weight);
    printf("BMI = %.1f\n", weight / (height * height));
    return 0;
}
```

## 4.4 检查返回值：用户输错了怎么办？

如果程序要一个整数，用户却输入了 `abc`，会怎样？`scanf` **读不进去**，变量保持原来的值。好在 `scanf` 有一个**返回值**：成功读到了几个值。

```c input="abc"
#include <stdio.h>

int main(void) {
    int n = 0;
    printf("输入一个整数：");
    if (scanf("%d", &n) != 1) {          // 应该读到 1 个值
        printf("\n这不是整数！\n");
    } else {
        printf("你输入的是 %d\n", n);
    }
    return 0;
}
```

（`if … else` 下一课会详细讲。）真实的程序遇到无法处理的输入时，常常 `return 1;` 结束，用非 0 的退出码告诉操作系统「出错了」（第 1 课）。

| `scanf` 的返回值 | 意思 |
|------------------|------|
| 等于要读的个数 | 全部成功 |
| 比要读的个数少 | 只读到了一部分（后面的格式对不上） |
| `EOF`（通常是 -1） | 输入已经结束（没有更多内容可读） |

> [!TIP]
> **认真的程序都要检查 `scanf` 的返回值。** 加了 `-Wall` 的编译器有时会提醒「忽略了 scanf 的返回值」。本课程后面的练习为了简洁，常常假设输入总是正确的。

## 4.5 读字符：小心残留的换行符

```c input="5\nY"
#include <stdio.h>

int main(void) {
    int count = 0;
    char answer = 0;
    printf("买几个？");
    scanf("%d", &count);
    printf("确定吗？(Y/N) ");
    scanf(" %c", &answer);             // 注意 %c 前面有一个空格！
    printf("\n数量 %d，回答 %c\n", count, answer);
    return 0;
}
```

用户输入 `5` 后按了回车，回车产生的换行符 `\n` 还**留在输入里**。`%d` 会跳过空白，但 `%c` **不会**——它会老老实实地读到这个换行符。在 `%c` 前加一个空格 `" %c"`，意思是「先跳过所有空白，再读一个字符」，问题就解决了。

## 4.6 读一个单词

```c input="Tom"
#include <stdio.h>

int main(void) {
    char name[50];                     // 一个能放 49 个字符的「字符数组」（第 9 课细讲）
    printf("你的英文名：");
    scanf("%49s", name);               // 数组名本身就代表地址，所以不加 &
    printf("Hello, %s!\n", name);
    return 0;
}
```

- `%s` 读到**空白（空格、换行）为止**，所以只能读一个单词；读一整行要用 `fgets`（第 9 课）。
- `%49s` 里的 49 表示「最多读 49 个字符」，留一个位置给字符串结尾的 `\0`。**一定要写这个限制**，否则用户输入太长时会写坏内存——这是很多安全漏洞的来源。

## 4.7 一直读到输入结束

题目常常不告诉你一共有几个数，只说「读到输入结束为止」。利用 `scanf` 的返回值：

```c input="3 8 1 9 4"
#include <stdio.h>

int main(void) {
    int x = 0, count = 0;
    long long sum = 0;
    while (scanf("%d", &x) == 1) {       // 只要还能读到一个整数，就继续
        sum += x;
        count++;
    }
    printf("一共 %d 个数，和为 %lld\n", count, sum);
    return 0;
}
```

`while` 循环下一课才会详细讲，这里先记住这个写法。在终端里手动输入时，按 **Ctrl+D**（macOS / Linux）或 **Ctrl+Z 再按回车**（Windows）表示「输入结束」。

## 4.8 常见错误

```c norun
int age = 0;
scanf("%d", age);          // 忘了 &
```

```text
warning: format '%d' expects argument of type 'int *', but argument 2 has type 'int' [-Wformat=]
```

运行时多半会崩溃（`Segmentation fault`）。`int *` 表示「指向 int 的指针（地址）」，编译器在提醒你：这里要的是地址。

```c norun
double h = 0;
scanf("%f", &h);           // double 要用 %lf
```

```text
warning: format '%f' expects argument of type 'float *', but argument 2 has type 'double *' [-Wformat=]
```

> [!NOTE]
> 用微软 Visual Studio 编译时，可能会看到 `'scanf': This function or variable may be unsafe. Consider using scanf_s instead`（错误 C4996）。这是微软自己的规定，不是 C 标准的要求。可以在文件最开头加一行 `#define _CRT_SECURE_NO_WARNINGS`，或者使用 GCC / Clang。

## 4.9 小结

- `scanf("%d", &x)` 读一个整数存进 `x`；**`&` 表示地址**，`scanf` 要修改变量所以必须加（字符数组除外）。
- `scanf` 的说明符：`%d`、`%lld`、**`%lf`（double）**、`%c`、`%s`；`%d`、`%lf`、`%s` 会自动跳过空白。
- `scanf` 返回成功读到的个数，读到输入末尾返回 `EOF`；用 `if (scanf(...) != 1)` 检查输入是否正确。
- 读字符用 `" %c"`（前面加空格）跳过残留的换行；读单词用 `%49s` 限制长度。
- `while (scanf("%d", &x) == 1)` 一直读到输入结束。

## 4.10 练习

<!-- exercise: sum-two -->
### 练习 1：两个数

输入两个整数，输出它们的和、差、积，用空格隔开。

```text
样例输入：7 3
样例输出：10 4 21
```

<details>
<summary>提示</summary>

`scanf("%d %d", &a, &b);` 然后 `printf("%d %d %d\n", a + b, a - b, a * b);`
</details>

<!-- exercise: avg3 -->
### 练习 2：三个小数的平均数

输入三个小数，输出它们的平均数，保留 2 位小数。

```text
样例输入：1.5 2.5 4
样例输出：2.67
```

<details>
<summary>提示</summary>

用 `double` 变量和 `%lf` 读入：`scanf("%lf %lf %lf", &a, &b, &c);`
</details>

<!-- exercise: next-year -->
### 练习 3：名字和年龄

输入一个英文名字（一个单词）和年龄，输出 `名字 will be 年龄+1 next year.`。

```text
样例输入：Tom 12
样例输出：Tom will be 13 next year.
```

<details>
<summary>提示</summary>

`char name[50]; int age;`，`scanf("%49s %d", name, &age);`——注意 `name` 不加 `&`，`age` 要加。
</details>

<!-- exercise: until-eof -->
### 练习 4：读到结束为止

输入若干个整数（不知道有几个），输出它们的个数、总和以及最大值，空格隔开。（至少有一个数。）

```text
样例输入：3 8 1 9 4
样例输出：5 25 9
```

<details>
<summary>提示</summary>

用 `while (scanf("%d", &x) == 1)`；最大值可以先设为第一个读到的数，或者初始化为 `INT_MIN`（在 `<limits.h>` 里）。
</details>

<!-- exercise: scanf-return -->
### 练习 5：读代码，写输出

假设运行下面的程序时，输入是 `12 abc`：

```c
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    int n = scanf("%d %d", &a, &b);
    printf("%d %d %d\n", n, a, b);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`12` 成功读进 `a`；`abc` 不是整数，读 `b` 失败，`b` 保持原来的 0。`scanf` 返回成功读到的个数 1。输出 `1 12 0`。
</details>

<!-- exercise: why-amp -->
### 练习 6：为什么要加 `&`？

`scanf("%d", &age);` 里为什么要写 `&age` 而不是 `age`？

- `&` 表示「这是一个整数」
- `scanf` 要把读到的值存进变量，需要知道变量在内存里的地址，`&age` 就是 age 的地址
- `&` 只是一种习惯，不加也没关系
- `&` 让输入的数字自动加 1

<details>
<summary>答案与解析</summary>

`&` 是取地址运算符。只传 `age` 的话，`scanf` 拿到的只是一个值（比如 0），不知道该把结果写到哪里，程序往往会崩溃。
</details>

## 4.11 延伸阅读

- [cppreference：scanf](https://zh.cppreference.com/w/c/io/fscanf)
- 下一课：[条件判断](../05-conditions/zh-CN.md)
