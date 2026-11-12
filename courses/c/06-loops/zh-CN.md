# 6. 循环

> 🎯 **本课目标**：用 `while`、`do … while`、`for` 让代码重复执行；用 `break` 提前结束循环、用 `continue` 跳过一次；写出嵌套循环（乘法表、图案）；掌握求和、计数、找最大值、拆分数字等常用套路；学会发现「死循环」和「差一错误」。
>
> ⏱ **预计用时**：75 分钟　｜　📋 **前置知识**：本课程第 3～5 课

## 6.1 while 循环

```c
#include <stdio.h>

int main(void) {
    int i = 1;
    while (i <= 5) {             // 条件为真就执行循环体，然后再检查条件
        printf("第 %d 次\n", i);
        i++;                     // 别忘了改变 i，否则条件永远为真
    }
    printf("循环结束时 i = %d\n", i);
    return 0;
}
```

执行过程：检查条件 → 为真就执行花括号里的代码 → 回到开头再检查……直到条件为假，跳出循环。

**死循环**：如果条件永远为真（比如忘了写 `i++`），循环就永远不会结束。在终端里按 **Ctrl+C** 可以强制结束程序；学习程序里点「停止」按钮。

## 6.2 for 循环

「从几数到几」是最常见的循环，`for` 把三个部分写在一行里：

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 5; i++) {
        printf("%d ", i);
    }
    printf("\n");

    for (int i = 10; i > 0; i -= 3) {    // 倒着数，每次减 3
        printf("%d ", i);
    }
    printf("\n");
    return 0;
}
```

```text
for (初始化; 条件; 每次循环后做的事) {
    循环体
}
```

它和下面的 `while` 完全等价：

```c norun
int i = 1;            // 初始化（只做一次）
while (i <= 5) {      // 条件
    printf("%d ", i); // 循环体
    i++;              // 每次循环后做的事
}
```

在 `for` 的括号里声明的变量（`int i`）只在循环里有效，循环结束就不存在了。

## 6.3 do … while：至少执行一次

`do … while` 先执行循环体，**再**检查条件，所以循环体至少执行一次。最适合「一直问到输入正确为止」：

```c input="-3\n150\n42"
#include <stdio.h>

int main(void) {
    int age = 0;
    do {
        printf("请输入年龄（0～120）：");
        scanf("%d", &age);
    } while (age < 0 || age > 120);       // 注意最后的分号
    printf("\n年龄是 %d\n", age);
    return 0;
}
```

## 6.4 break 和 continue

- **`break`**：立刻跳出整个循环；
- **`continue`**：跳过这一次循环剩下的部分，直接开始下一次。

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 10; i++) {
        if (i % 3 == 0) {
            continue;              // 跳过 3 的倍数
        }
        if (i > 8) {
            break;                 // 超过 8 就结束
        }
        printf("%d ", i);
    }
    printf("\n");                  // 输出：1 2 4 5 7 8
    return 0;
}
```

一个常见的写法是「先无限循环，满足条件时 `break`」：

```c input="5 12 -1 7"
#include <stdio.h>

int main(void) {
    int sum = 0, x = 0;
    while (true) {
        scanf("%d", &x);
        if (x < 0) {
            break;                 // 读到负数就停止
        }
        sum += x;
    }
    printf("和是 %d\n", sum);      // 17
    return 0;
}
```

## 6.5 循环的常用套路

**求和与计数**：

```c
#include <stdio.h>

int main(void) {
    int sum = 0, count = 0;
    for (int i = 1; i <= 100; i++) {
        if (i % 7 == 0) {
            sum += i;              // 累加
            count++;               // 计数
        }
    }
    printf("1～100 里 7 的倍数有 %d 个，和是 %d\n", count, sum);
    return 0;
}
```

**拆分一个整数的每一位**：`% 10` 取出最后一位，`/ 10` 去掉最后一位：

```c
#include <stdio.h>

int main(void) {
    int n = 2025;
    int digits = 0, digit_sum = 0, reversed = 0;
    while (n > 0) {
        int last = n % 10;         // 最后一位
        digit_sum += last;
        reversed = reversed * 10 + last;
        n /= 10;                   // 去掉最后一位
        digits++;
    }
    printf("%d 位，各位之和 %d，倒过来是 %d\n", digits, digit_sum, reversed);
    return 0;
}
```

**判断质数**：

```c
#include <stdio.h>

int main(void) {
    int n = 97;
    bool prime = n >= 2;
    for (int d = 2; d * d <= n; d++) {    // 只需要检查到 √n
        if (n % d == 0) {
            prime = false;
            break;                         // 找到一个因数就够了
        }
    }
    printf("%d %s质数\n", n, prime ? "是" : "不是");
    return 0;
}
```

## 6.6 嵌套循环

循环里面还可以有循环。外层每执行一次，内层就完整地执行一遍。九九乘法表：

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 9; i++) {
        for (int j = 1; j <= i; j++) {
            printf("%d×%d=%-3d", j, i, i * j);
        }
        printf("\n");
    }
    return 0;
}
```

用循环画图案：

```c
#include <stdio.h>

int main(void) {
    int n = 4;
    for (int row = 1; row <= n; row++) {
        for (int s = 0; s < n - row; s++) {
            printf(" ");                  // 先输出空格
        }
        for (int k = 0; k < 2 * row - 1; k++) {
            printf("*");                  // 再输出星号
        }
        printf("\n");
    }
    return 0;
}
```

> [!TIP]
> 画图案的诀窍：先找出「第 row 行有几个空格、几个星号」，写成和 `row` 有关的式子（比如 `n - row` 和 `2 * row - 1`），再用循环输出。

注意：`break` 只能跳出**它所在的那一层**循环。

## 6.7 差一错误

循环次数多一次或少一次，是最常见的 bug，叫**差一错误**（off-by-one error）：

```c norun
for (int i = 0; i <= 10; i++) { ... }   // 执行 11 次（0 到 10）
for (int i = 0; i < 10; i++) { ... }    // 执行 10 次（0 到 9）← C 里最常见的写法
for (int i = 1; i <= 10; i++) { ... }   // 执行 10 次（1 到 10）
```

拿不准的时候，用很小的数（比如 1、2、3）在纸上走一遍，或者在循环里 `printf` 一下 `i` 看看。

## 6.8 小结

- `while (条件) { ... }`：先检查后执行；`do { ... } while (条件);`：至少执行一次。
- `for (初始化; 条件; 更新) { ... }`：适合「从几数到几」；`for` 里声明的变量只在循环内有效。
- `break` 跳出当前循环，`continue` 跳到下一次；`while (true) { ... if (...) break; }` 是常用写法。
- 常用套路：累加、计数、找最大值、`% 10` 和 `/ 10` 拆分数字、`d * d <= n` 判断质数。
- 嵌套循环：外层每走一步，内层走一整遍；画图案先找出每行的规律。
- 小心死循环（Ctrl+C 结束）和差一错误（`<` 还是 `<=`）。

## 6.9 练习

<!-- exercise: sum-n -->
### 练习 1：1 加到 n

输入一个正整数 `n`，输出 `1 + 2 + … + n` 的和。（`n` 最大为 100000，注意结果可能超过 `int` 的范围。）

```text
样例输入：100
样例输出：5050
```

<details>
<summary>提示</summary>

用 `long long sum = 0;` 累加，输出用 `%lld`。
</details>

<!-- exercise: times-table -->
### 练习 2：乘法表的一行

输入一个整数 `n`，输出 `n × 1` 到 `n × 9`，每行一个，格式是 `n x i = 积`。

```text
样例输入：3
样例输出：
3 x 1 = 3
3 x 2 = 6
3 x 3 = 9
3 x 4 = 12
3 x 5 = 15
3 x 6 = 18
3 x 7 = 21
3 x 8 = 24
3 x 9 = 27
```

<details>
<summary>提示</summary>

`for (int i = 1; i <= 9; i++) printf("%d x %d = %d\n", n, i, n * i);`
</details>

<!-- exercise: digits -->
### 练习 3：数位

输入一个非负整数，输出它有几位数和各位数字之和，空格隔开。（0 算 1 位。）

```text
样例输入：2025
样例输出：4 9
```

<details>
<summary>提示</summary>

用 `do { ... } while (n > 0);`，这样 0 也会被算作 1 位。
</details>

<!-- exercise: primes -->
### 练习 4：质数列表

输入一个整数 `n`（`n ≥ 2`），输出 2 到 `n` 之间的所有质数，用空格隔开。

```text
样例输入：20
样例输出：2 3 5 7 11 13 17 19
```

<details>
<summary>提示</summary>

外层循环 `k` 从 2 到 `n`，内层用 6.5 节的方法判断 `k` 是不是质数。输出时注意第一个数前面不要有空格：可以用一个变量记录「是不是第一个」。
</details>

<!-- exercise: pyramid -->
### 练习 5：金字塔

输入 `n`，输出一个 `n` 行的星号金字塔（第 `row` 行先输出 `n - row` 个空格，再输出 `2 × row - 1` 个星号）。

```text
样例输入：3
样例输出：
  *
 ***
*****
```

<details>
<summary>提示</summary>

照着 6.6 节的程序写，把 `n` 改成用 `scanf` 读入。
</details>

<!-- exercise: loop-predict -->
### 练习 6：读代码，写输出

```c
#include <stdio.h>

int main(void) {
    int i = 0;
    while (i < 10) {
        i += 3;
        if (i == 6) {
            continue;
        }
        printf("%d ", i);
    }
    printf("\n");
    for (int j = 5; j > 0; j--) {
        if (j == 2) break;
        printf("%d", j);
    }
    printf("\n");
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`while`：i 依次变成 3（输出）、6（`continue` 跳过输出）、9（输出）、12（输出，然后 12 < 10 不成立，结束）。第一行是 `3 9 12 `。`for`：j 为 5、4、3 时输出，j 为 2 时 `break`。第二行是 `543`。
</details>

<!-- exercise: off-by-one -->
### 练习 7：循环几次？

`for (int i = 0; i <= 10; i++)` 的循环体执行几次？

- 9 次
- 10 次
- 11 次
- 无限次

<details>
<summary>答案与解析</summary>

i 取 0, 1, 2, …, 10，一共 11 个值。想要 10 次，通常写 `i < 10`。
</details>

## 6.10 延伸阅读

- [cppreference：for 循环](https://zh.cppreference.com/w/c/language/for)、[while 循环](https://zh.cppreference.com/w/c/language/while)
- 下一课：[函数](../07-functions/zh-CN.md)
