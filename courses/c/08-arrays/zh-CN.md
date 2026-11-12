# 8. 数组

> 🎯 **本课目标**：用**数组**存放一组同类型的数据；会定义、初始化数组，用下标访问元素，用循环遍历；明白 C **不检查越界**以及它的后果，学会用「内存检查」找出越界；掌握数组的常用算法（求和、最值、查找、反转、复制、冒泡排序）；把数组传给函数（要同时传长度），理解为什么函数能修改数组；使用二维数组。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：本课程第 1～7 课

## 8.1 为什么需要数组

要存 5 个学生的成绩，写 5 个变量 `s1`、`s2`……还凑合；要是 100 个、1000 个呢？**数组**就是「一排同类型的格子」，用一个名字加一个**下标**（编号）访问其中每一格：

```c
#include <stdio.h>

int main(void) {
    int scores[5] = {90, 85, 77, 60, 95};   // 5 个 int 排成一排

    printf("第一个：%d\n", scores[0]);       // 下标从 0 开始
    printf("最后一个：%d\n", scores[4]);     // 长度是 5，最后一个下标是 4

    scores[2] = 80;                          // 修改一个元素
    for (int i = 0; i < 5; i++) {            // 用循环处理每一个元素
        printf("scores[%d] = %d\n", i, scores[i]);
    }
    return 0;
}
```

```text
下标：      0    1    2    3    4
         ┌────┬────┬────┬────┬────┐
scores   │ 90 │ 85 │ 80 │ 60 │ 95 │
         └────┴────┴────┴────┴────┘
```

- 定义：`类型 名字[长度];`，比如 `double prices[100];`、`char letters[26];`；
- 下标从 **0** 开始，长度为 n 的数组，下标是 **0 到 n-1**；
- 每个元素 `scores[i]` 用起来就和普通变量一样：可以赋值、参与计算、传给 `printf`；
- 下标可以是任何整数表达式：`scores[i]`、`scores[i + 1]`、`scores[n - 1]`；
- 数组的元素在内存里**紧挨着**排列，这让按下标访问非常快。

`for (int i = 0; i < n; i++)` 是 C 里最常见的循环（第 6 课），正是因为它恰好走遍长度为 n 的数组的所有下标。

## 8.2 初始化数组

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};        // 全部给出
    int b[5] = {1, 2};                 // 只给前两个，其余自动为 0
    int c[5] = {0};                    // 常用写法：全部清零
    int d[] = {7, 8, 9};               // 不写长度：编译器数出来是 3
    int e[5] = {[1] = 10, [3] = 30};   // 指定下标初始化，其余为 0

    printf(" a  b  c  e\n");
    for (int i = 0; i < 5; i++) {
        printf("%2d %2d %2d %2d\n", a[i], b[i], c[i], e[i]);
    }
    printf("d 有 %zu 个元素\n", sizeof d / sizeof d[0]);
    return 0;
}
```

- `sizeof d` 是整个数组占的字节数（3 × 4 = 12），`sizeof d[0]` 是一个元素的字节数（4），相除就是**元素个数**。这个技巧只在**定义数组的那个函数里**有效（8.6 节会看到原因）。
- **没有初始化的局部数组里是垃圾值**，和未初始化的变量一样（第 2 课）。写 `int a[100] = {0};` 是个好习惯。
- 数组长度最好用一个**常量**表示，改长度时只需改一处：

```c
#include <stdio.h>

#define N 5                        // 预处理器把下面所有的 N 替换成 5

int main(void) {
    int squares[N];
    for (int i = 0; i < N; i++) {
        squares[i] = i * i;
    }
    for (int i = 0; i < N; i++) {
        printf("%d ", squares[i]);
    }
    printf("\n");
    return 0;
}
```

`#define N 5` 是**宏**：编译之前，预处理器把代码里的 `N` 都换成 `5`。习惯上宏名全用大写。（C23 也可以写 `constexpr int N = 5;`，但较老的编译器不支持。）

## 8.3 C 不检查越界！

这是 C 和 Python 最重要的区别之一。下面的循环多走了一步，访问了不存在的 `a[5]`：

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};
    for (int i = 0; i <= 5; i++) {     // 错误：i 会取到 5
        printf("%d ", a[i]);
    }
    printf("\n");
    return 0;
}
```

Python 会立刻报 `IndexError`，C 却**一声不吭**：它直接去读数组后面那块内存，打印出一个莫名其妙的数（每次运行可能还不一样）。这叫**未定义行为**（undefined behavior）——什么都可能发生：

- 读到垃圾值，程序「看起来正常」地给出错误结果；
- **写**到数组外面，悄悄改掉了别的变量；程序可能报 `*** stack smashing detected ***` 后被终止；
- 访问到不属于程序的内存，崩溃并报 `Segmentation fault`（段错误）。

很多著名的安全漏洞（「缓冲区溢出」）就是这样来的。所以：

- 循环写 `i < n`，不要写 `i <= n`；
- 来自输入的下标，用之前先检查：`if (k >= 0 && k < N) { ... }`；
- 用**内存检查**工具把越界揪出来。

### 打开内存检查

GCC 和 Clang 自带 **AddressSanitizer**（地址消毒器）。在终端里编译时加上 `-fsanitize=address,undefined -g`：

```bash
gcc -fsanitize=address,undefined -g main.c -o main
./main
```

在本教程的桌面程序里，打开菜单 **工具 → 设置…**，勾选「**C/C++ 内存检查（AddressSanitizer）**」即可（Windows 上的 MinGW 不支持这个功能，程序会自动跳过）。再运行上面的例子，程序会在越界的那一刻停下来，并指出是哪一行：

```text
main.c:6:24: runtime error: index 5 out of bounds for type 'int [5]'
```

> [!TIP]
> 写 C 程序时，建议**一直开着内存检查**。它会让程序慢一些，但能把「偶尔出错、很难重现」的内存错误变成「当场报错、指出行号」。第 10、11 课学指针和动态内存时，它会更有用。

## 8.4 读入一组数据

很多题目的输入格式是「先给个数 n，再给 n 个数」。最简单的做法是开一个**足够大**的数组：

```c input="5\n3 8 1 9 4"
#include <stdio.h>

#define MAX_N 1000                     // 最多支持 1000 个数

int main(void) {
    int a[MAX_N];
    int n = 0;
    scanf("%d", &n);
    if (n < 1 || n > MAX_N) {          // 防止越界：先检查 n
        printf("n 必须在 1 到 %d 之间\n", MAX_N);
        return 1;
    }
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);            // a[i] 就是一个 int 变量，同样要加 &
    }

    long long sum = 0;
    int max_i = 0;                     // 记住最大值的「下标」，而不只是值
    for (int i = 0; i < n; i++) {
        sum += a[i];
        if (a[i] > a[max_i]) {
            max_i = i;
        }
    }
    printf("总和 %lld，平均 %.2f\n", sum, (double)sum / n);
    printf("最大值是第 %d 个数：%d\n", max_i + 1, a[max_i]);
    return 0;
}
```

注意：数组本身的大小 `MAX_N` 是固定的，`n` 只是「用了前 n 格」。

> [!NOTE]
> 你可能会看到读入 n 之后直接写 `int a[n];` 的代码，这叫**变长数组**（VLA）。GCC 和 Clang 支持它，但它是 C 标准里的「可选功能」，微软的编译器不支持；n 太大时还会把栈撑爆（第 7 课）。本课程用「固定的最大长度」，到第 11 课再学真正按需分配内存的 `malloc`。

## 8.5 常用数组算法

```c
#include <stdio.h>

#define N 6

int main(void) {
    int a[N] = {4, 8, 15, 16, 23, 42};

    // 线性查找：从头到尾一个一个比较
    int target = 16, pos = -1;         // -1 表示「没找到」
    for (int i = 0; i < N; i++) {
        if (a[i] == target) {
            pos = i;
            break;                     // 找到第一个就停
        }
    }
    printf("%d 的下标：%d\n", target, pos);

    // 原地反转：交换 a[i] 和 a[N-1-i]，只需走一半
    for (int i = 0; i < N / 2; i++) {
        int t = a[i];
        a[i] = a[N - 1 - i];
        a[N - 1 - i] = t;
    }

    // 复制：只能一个一个元素地复制
    int b[N];
    for (int i = 0; i < N; i++) {
        b[i] = a[i];
    }

    for (int i = 0; i < N; i++) {
        printf("%d ", b[i]);
    }
    printf("\n");
    return 0;
}
```

- 反转时如果循环走完整个数组（`i < N`），每对元素会被交换两次，结果又变回原样。
- 数组**不能整体赋值**：写 `b = a;` 会报错 `assignment to expression with array type`；也不能用 `==` 比较两个数组的内容。复制要用循环，或者 `<string.h>` 里的 `memcpy(b, a, sizeof a);`。

## 8.6 数组和函数

把数组传给函数时，要**同时传长度**：

```c
#include <stdio.h>

void print_array(const int a[], int n) {   // const：保证不修改数组
    for (int i = 0; i < n; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void double_all(int a[], int n) {          // 没有 const：要修改数组
    for (int i = 0; i < n; i++) {
        a[i] *= 2;
    }
}

int sum_array(const int a[], int n) {
    int s = 0;
    for (int i = 0; i < n; i++) {
        s += a[i];
    }
    return s;
}

int main(void) {
    int nums[] = {1, 2, 3, 4};
    int n = sizeof nums / sizeof nums[0];  // 只有在这里才能这样算长度
    print_array(nums, n);
    double_all(nums, n);
    print_array(nums, n);                  // 2 4 6 8：main 里的数组真的被改了！
    printf("和：%d\n", sum_array(nums, n));
    return 0;
}
```

这里有两个和第 7 课「值传递」不一样的地方：

1. **函数能修改调用者的数组**。传数组时，C 并不复制整个数组（那可能很大），而是只传「数组在哪里」——第一个元素的**地址**。所以 `double_all` 改的就是 `main` 里的 `nums`。不想让函数修改，就在参数前加 `const`，万一不小心改了，编译器会报错。
2. **函数里不知道数组有多长**。参数 `int a[]` 其实只是一个地址（第 10 课叫它「指针」），`sizeof a` 得到的是地址的大小（通常 8），而不是数组的大小。GCC 会警告：

```text
warning: 'sizeof' on array function parameter 'a' will return size of 'int *'
```

所以长度一定要作为参数单独传进去。另外，函数**不能返回一个数组**；要「产出」一组数据，就让调用者准备好数组，传进来让函数填写（就像 `double_all`）。

## 8.7 排序：冒泡排序

**排序**是最经典的算法问题。**冒泡排序**的思路：从左到右比较相邻的两个数，前面的比后面的大就交换。走完一轮，最大的数就像气泡一样「浮」到了最右边；再对剩下的数重复，直到某一轮一次都没交换：

```c
#include <stdio.h>

void print_array(const int a[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void bubble_sort(int a[], int n) {
    for (int pass = 0; pass < n - 1; pass++) {
        bool swapped = false;
        for (int i = 0; i < n - 1 - pass; i++) {   // 最后 pass 个已经排好了
            if (a[i] > a[i + 1]) {
                int t = a[i];
                a[i] = a[i + 1];
                a[i + 1] = t;
                swapped = true;
            }
        }
        printf("第 %d 轮后：", pass + 1);
        print_array(a, n);
        if (!swapped) {
            break;                                 // 一次都没交换：已经有序
        }
    }
}

int main(void) {
    int a[] = {5, 1, 4, 2, 8};
    bubble_sort(a, 5);
    return 0;
}
```

- 内层循环到 `n - 1 - pass` 为止：因为要访问 `a[i + 1]`，`i` 最大只能到倒数第二个，否则就越界了；而最后 `pass` 个数已经在正确的位置上。
- 冒泡排序要比较大约 n²/2 次：1000 个数约 50 万次，还很快；100 万个数就要约 5000 亿次，太慢了。实际编程中用标准库的 `qsort`，它快得多——学完指针（第 10 课）后我们就会用它。

## 8.8 二维数组

表格、棋盘、图像、矩阵都是「行 × 列」的数据，用**二维数组**表示：

```c
#include <stdio.h>

#define ROWS 3
#define COLS 4

int main(void) {
    int m[ROWS][COLS] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12},
    };
    m[1][2] = 70;                      // 第 1 行第 2 列（都从 0 数起）

    for (int r = 0; r < ROWS; r++) {
        int row_sum = 0;
        for (int c = 0; c < COLS; c++) {
            printf("%4d", m[r][c]);
            row_sum += m[r][c];
        }
        printf("  | 和 %d\n", row_sum);
    }
    return 0;
}
```

- `m[r][c]`：先行后列。`int m[3][4]` 可以看成「3 个元素的数组，每个元素又是一个 4 个 int 的数组」。
- 在内存里，二维数组是**一行接一行**连续存放的：`m[0][0] m[0][1] m[0][2] m[0][3] m[1][0] …`。
- 遍历用两层循环，外层走行、内层走列（第 6 课的嵌套循环）。

把二维数组传给函数时，**列数**必须告诉编译器（它要靠列数算出 `m[r][c]` 在哪里）。可以把行数、列数作为参数，并写在数组参数前面：

```c
#include <stdio.h>

void print_matrix(int rows, int cols, const int m[rows][cols]) {
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            printf("%3d", m[r][c]);
        }
        printf("\n");
    }
}

int main(void) {
    int table[3][3] = {0};
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            table[r][c] = (r + 1) * (c + 1);   // 小九九的一角
        }
    }
    print_matrix(3, 3, table);
    return 0;
}
```

（`const int m[rows][cols]` 这种用参数当长度的写法需要 GCC 或 Clang；也可以写成固定列数 `const int m[][3]`。）

## 8.9 小结

- 数组：`类型 名字[长度]`，下标 **0 到 n-1**；`int a[5] = {0};` 全部清零，`int d[] = {...}` 让编译器数长度。
- `sizeof a / sizeof a[0]` 求元素个数，只在定义数组的函数里有效；长度用 `#define N 5` 这样的常量。
- **C 不检查越界**：越界是未定义行为，可能得到垃圾值、悄悄改坏数据或崩溃。循环用 `i < n`，并打开内存检查（`-fsanitize=address,undefined`）。
- 「先读 n 再读 n 个数」：开一个固定的最大长度，并检查 n。
- 常用算法：求和、最值（记下标）、线性查找（找不到返回 -1）、反转（走一半）、复制（逐个元素）、冒泡排序。
- 数组不能用 `=` 整体赋值，也不能用 `==` 比较。
- 传数组给函数：同时传长度；函数**能修改**原数组，不想修改就加 `const`；函数里 `sizeof` 得不到数组长度。
- 二维数组 `m[行][列]` 按行连续存放，用两层循环遍历；传给函数时要告诉它列数。

## 8.10 练习

<!-- exercise: reverse-array -->
### 练习 1：倒过来输出

输入一个正整数 `n`（不超过 1000），再输入 `n` 个整数，把它们倒序输出，用空格隔开。

```text
样例输入：
5
1 2 3 4 5
样例输出：5 4 3 2 1
```

<details>
<summary>提示</summary>

先全部读进数组，再 `for (int i = n - 1; i >= 0; i--)` 输出。注意第一个数前面不要有空格：可以在 `i < n - 1` 时先输出一个空格。
</details>

<!-- exercise: array-stats -->
### 练习 2：最大、最小和平均

输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数，输出最大值、最小值和平均值（保留 2 位小数），用空格隔开。

```text
样例输入：
4
3 -2 10 5
样例输出：10 -2 4.00
```

<details>
<summary>提示</summary>

最大值和最小值的初始值都设为 `a[0]`，而不是 0——否则全是负数（或全是正数）时会出错。平均值用 `(double)sum / n`。
</details>

<!-- exercise: above-average -->
### 练习 3：高于平均

输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数，输出**严格大于**平均值的数有几个。

```text
样例输入：
5
1 2 3 4 10
样例输出：1
```

<details>
<summary>提示</summary>

要先求出平均值，才能逐个比较——所以必须把数都存进数组。为了避开小数，可以把 `a[i] > sum / n` 改写成 `a[i] * n > sum`（用 `long long`）。
</details>

<!-- exercise: sort-numbers -->
### 练习 4：排序

输入 `n`（1 ≤ n ≤ 1000）和 `n` 个整数，把它们从小到大排序后输出，用空格隔开。

```text
样例输入：
6
5 3 9 1 3 7
样例输出：1 3 3 5 7 9
```

<details>
<summary>提示</summary>

照着 8.7 节写 `bubble_sort`（去掉每轮打印的部分）。
</details>

<!-- exercise: transpose -->
### 练习 5：矩阵转置

输入行数 `r` 和列数 `c`（都在 1 到 10 之间），再输入 `r` 行、每行 `c` 个整数。输出它的**转置**：原来的第 i 行变成第 i 列，所以输出有 `c` 行、每行 `r` 个数，用空格隔开。

```text
样例输入：
2 3
1 2 3
4 5 6
样例输出：
1 4
2 5
3 6
```

<details>
<summary>提示</summary>

定义 `int m[10][10];` 读入。输出时外层循环走原来的列 `j`，内层循环走原来的行 `i`，打印 `m[i][j]`。
</details>

<!-- exercise: array-predict -->
### 练习 6：读代码，写输出

```c
#include <stdio.h>

void change(int x, int a[]) {
    x = 100;
    a[0] = 100;
}

int main(void) {
    int x = 1;
    int a[3] = {1, 2};
    change(x, a);
    printf("%d %d %d %d\n", x, a[0], a[1], a[2]);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`x` 是值传递，函数改的是副本，外面的 `x` 还是 1；数组传的是地址，`a[0]` 真的被改成了 100；`a[2]` 没有给初始值，自动为 0。输出 `1 100 2 0`。
</details>

<!-- exercise: array-index -->
### 练习 7：合法的下标

`int a[10];` 的合法下标是哪些？

- 1 到 10
- 0 到 9
- 0 到 10
- 任意整数都可以，C 会自动检查

<details>
<summary>答案与解析</summary>

长度为 10 的数组，下标从 0 到 9。C **不会**检查越界，访问 `a[10]` 是未定义行为。
</details>

## 8.11 延伸阅读

- [cppreference：数组声明](https://zh.cppreference.com/w/c/language/array)、[数组初始化](https://zh.cppreference.com/w/c/language/array_initialization)
- [AddressSanitizer 文档（英文）](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- [VisuAlgo：排序算法动画](https://visualgo.net/zh/sorting)——看冒泡排序一步一步是怎么交换的
- 下一课：[字符串](../09-strings/zh-CN.md)
