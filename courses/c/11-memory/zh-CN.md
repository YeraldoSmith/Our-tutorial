# 11. 动态内存

> 🎯 **本课目标**：明白程序内存的几个区域（栈、堆、全局区），知道什么时候需要**动态内存**；会用 `malloc` 申请内存、用 `free` 归还，并检查 `NULL`；会用 `calloc` 申请清零的内存，用 `realloc` 让数组「长大」，写出能装下任意多个数据的**动态数组**；让函数返回动态分配的内存，并说清楚「谁负责释放」；用一维数组模拟动态二维数组；认识内存泄漏、释放后使用、重复释放等错误，并用内存检查找出它们。
>
> ⏱ **预计用时**：100 分钟　｜　📋 **前置知识**：本课程第 1～10 课，特别是第 10 课「指针」

## 11.1 为什么需要动态内存

到目前为止，我们的数组都是**固定大小**的：

```c norun
#define MAX_N 1000
int a[MAX_N];          // 只有 10 个数时浪费；有 100 万个数时又不够
```

还有更多问题：

- 局部变量放在**栈**上，栈通常只有几 MB。`int big[10000000];`（约 40 MB）写在函数里，程序一运行就会崩溃（栈溢出，第 7 课）。
- 局部变量在函数返回时就消失了，所以函数不能返回局部数组的地址（第 10 课）。
- 很多时候，数据有多少要到**运行时**才知道，比如「读到输入结束为止」。

解决办法是在**堆**（heap）上**动态**申请内存：需要多少就申请多少，用完自己归还。程序的内存大致是这样分布的：

```text
┌──────────────────────────────┐
│ 栈 stack                     │ 局部变量、函数参数：函数返回就自动释放；空间小（通常几 MB）
│              ↓               │
│                              │
│              ↑               │
│ 堆 heap                      │ malloc 申请：自己申请、自己 free；空间大（可以用到几 GB）
├──────────────────────────────┤
│ 全局变量、static 变量          │ 整个程序运行期间一直存在
├──────────────────────────────┤
│ 程序代码、字符串字面量          │ 只读
└──────────────────────────────┘
```

## 11.2 `malloc` 和 `free`

```c input="5\n3 8 1 9 4"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n = 0;
    if (scanf("%d", &n) != 1 || n <= 0) {
        return 1;
    }

    int *a = malloc(n * sizeof *a);      // 申请能放 n 个 int 的内存
    if (a == NULL) {                     // 内存不够时返回 NULL，一定要检查
        printf("内存不足\n");
        return 1;
    }

    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);              // 用起来和数组一模一样
    }
    long long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
    printf("%d 个数，和为 %lld\n", n, sum);

    free(a);                             // 用完了，归还给系统
    a = NULL;                            // 好习惯：防止之后误用
    return 0;
}
```

- **`malloc(字节数)`** 在堆上申请一块内存，返回它的地址（类型是 `void *`，在 C 里会自动转换成 `int *`，不用强制转换）。
- **`n * sizeof *a`**：`*a` 是一个 `int`，`sizeof *a` 就是一个元素的大小。这样写的好处是：以后把 `a` 改成 `double *`，这一行不用改。
- `malloc` 来的内存里是**垃圾值**，和未初始化的局部变量一样。
- 内存不够时 `malloc` 返回 **`NULL`**，所以拿到之后要先检查。
- **`free(指针)`** 归还内存。每一块 `malloc` 来的内存都要 `free`，而且**只能 `free` 一次**。`free` 之后指针本身还在，但不能再通过它读写了；把它设为 `NULL` 是个好习惯（`free(NULL)` 什么也不做，是安全的）。

## 11.3 `calloc` 和 `realloc`：会长大的数组

| 函数 | 作用 |
|------|------|
| `malloc(字节数)` | 申请内存，内容是垃圾值 |
| `calloc(个数, 每个的大小)` | 申请内存并**全部清零**，例如 `calloc(n, sizeof *a)` |
| `realloc(p, 新的字节数)` | 把 `p` 指向的内存**改成新的大小**，原来的内容会保留；可能会搬到新的地址，所以要用它的返回值 |
| `free(p)` | 归还内存 |

有了 `realloc`，就能写出**动态数组**：先申请一点空间，满了就把容量**翻倍**。下面的程序能读入任意多个数：

```c input="5 12 -3 8 0 7 21"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int capacity = 2;                         // 先准备 2 个位置
    int count = 0;                            // 已经用了几个
    int *data = malloc(capacity * sizeof *data);
    if (data == NULL) {
        return 1;
    }

    int x;
    while (scanf("%d", &x) == 1) {
        if (count == capacity) {              // 满了：容量翻倍
            capacity *= 2;
            int *bigger = realloc(data, capacity * sizeof *data);
            if (bigger == NULL) {             // 失败时原来的 data 还在，要记得释放
                free(data);
                return 1;
            }
            data = bigger;
            printf("（容量扩大到 %d）\n", capacity);
        }
        data[count++] = x;                    // 先存进 data[count]，再把 count 加 1
    }

    printf("读到 %d 个数：", count);
    for (int i = 0; i < count; i++) {
        printf("%d ", data[i]);
    }
    printf("\n");
    free(data);
    return 0;
}
```

- **为什么要用临时变量 `bigger`？** 如果写成 `data = realloc(data, ...)`，一旦失败返回 `NULL`，原来那块内存的地址就丢了，再也没法 `free`——这就是内存泄漏。
- **为什么翻倍，而不是每次加 1？** `realloc` 有时要把所有数据搬到新地方，很费时间。翻倍的话，100 万个数只需要扩大约 20 次。Python 的列表、C++ 的 `vector` 在内部也是这么做的。

## 11.4 函数返回动态内存

函数的局部变量在返回后就消失了，但 `malloc` 来的内存会一直存在，直到被 `free`。所以函数可以把它返回给调用者：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 返回一个新的字符串：把 s 重复 times 次。
// 返回的内存由调用者负责 free；失败时返回 NULL。
char *repeat(const char *s, int times) {
    size_t len = strlen(s);
    char *result = malloc(len * times + 1);       // +1 留给 '\0'
    if (result == NULL) {
        return NULL;
    }
    for (int i = 0; i < times; i++) {
        memcpy(result + i * len, s, len);         // 把 s 的 len 个字节复制到第 i 段
    }
    result[len * times] = '\0';
    return result;
}

// 返回一个新的数组 {0, 1, 4, 9, ...}，共 n 个平方数。调用者负责 free。
int *squares(int n) {
    int *a = malloc(n * sizeof *a);
    if (a == NULL) {
        return NULL;
    }
    for (int i = 0; i < n; i++) {
        a[i] = i * i;
    }
    return a;
}

int main(void) {
    char *text = repeat("ab", 3);
    if (text != NULL) {
        printf("%s\n", text);          // ababab
        free(text);
    }

    int *sq = squares(6);
    if (sq != NULL) {
        for (int i = 0; i < 6; i++) {
            printf("%d ", sq[i]);
        }
        printf("\n");
        free(sq);
    }
    return 0;
}
```

关键是说清楚**谁负责释放**（「所有权」）。习惯上在函数的注释里写明「返回的内存由调用者 `free`」。`memcpy(目标, 来源, 字节数)`（在 `<string.h>` 里）按字节复制一块内存。顺便一提，C23 的标准库里也有 `strdup(s)`，它返回 `s` 的一份 `malloc` 来的副本。

## 11.5 动态的二维数组

最简单、最通用的办法是：申请一块 `行数 × 列数` 的一维内存，自己算下标——第 `r` 行第 `c` 列在 `r * cols + c`：

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int rows = 3, cols = 4;
    int *grid = malloc(rows * cols * sizeof *grid);
    if (grid == NULL) {
        return 1;
    }
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            grid[r * cols + c] = r * 10 + c;    // 第 r 行第 c 列
        }
    }
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            printf("%3d", grid[r * cols + c]);
        }
        printf("\n");
    }
    free(grid);
    return 0;
}
```

这和第 8 课说的「二维数组在内存里一行接一行地存放」是同一个道理，只是下标由我们自己计算。（GCC 和 Clang 还支持 `int (*m)[cols] = malloc(rows * sizeof *m);`，之后可以直接写 `m[r][c]`。）

## 11.6 内存错误，以及怎样发现它们

动态内存带来的错误很隐蔽：程序常常「看起来正常」，过一会儿才莫名其妙地出错。打开**内存检查**（工具 → 设置…，第 8 课），它们大都能被当场抓住：

| 错误 | 例子 | 内存检查报告 |
|------|------|--------------|
| **内存泄漏**：忘了 `free` | 函数里 `malloc` 了，返回前没 `free` | `LeakSanitizer: detected memory leaks` |
| **释放后使用** | `free(p);` 之后又 `p[0] = 1;` | `heap-use-after-free` |
| **重复释放** | 同一个指针 `free` 了两次 | `attempting double-free` |
| **越界** | `malloc(4 * sizeof *p)` 却访问 `p[4]` | `heap-buffer-overflow` |
| **释放不是 `malloc` 来的内存** | `int x; free(&x);` | `attempting free on address which was not malloc()-ed` |

```c norun
void process(void) {
    int *tmp = malloc(1000 * sizeof *tmp);
    // ……使用 tmp……
    return;              // 忘了 free(tmp)：每调用一次就丢掉约 4 KB
}
```

程序结束时操作系统会收回所有内存，所以短小的程序泄漏一点「好像没事」。但服务器、游戏这类长时间运行的程序，泄漏会越积越多，最终把内存耗光。从一开始就养成好习惯：

- 每写一个 `malloc`，马上想好在哪里 `free`；
- `free` 之后把指针设为 `NULL`；
- `realloc` 的结果先存进临时变量；
- 写 C 时一直开着内存检查。

## 11.7 小结

- 栈：局部变量，自动释放，空间小；堆：`malloc` 申请，手动 `free`，空间大。
- `T *p = malloc(n * sizeof *p);`，检查 `NULL`，用完 `free(p); p = NULL;`。
- `calloc` 申请并清零；`realloc` 改变大小（结果先存临时变量）；动态数组满了就把容量翻倍。
- 函数可以返回 `malloc` 来的内存，要写清楚由谁 `free`。
- 动态二维数组：一维的 `rows * cols` 块，下标 `r * cols + c`。
- 常见错误：泄漏、释放后使用、重复释放、越界、释放非堆内存——用内存检查找出它们。

## 11.8 练习

<!-- exercise: reverse-unknown -->
### 练习 1：不知道有多少个数

输入若干个整数（至少 1 个，个数事先不知道，可能多达 100 万个），一直读到输入结束，把它们倒序输出，用空格隔开。

```text
样例输入：1 2 3 4 5
样例输出：5 4 3 2 1
```

<details>
<summary>提示</summary>

照着 11.3 节写一个会翻倍的动态数组，读完之后从 `count - 1` 倒着输出。别忘了 `free`。
</details>

<!-- exercise: merge-sorted -->
### 练习 2：合并两个有序数组

第一行是 `n` 和 `n` 个从小到大排好的整数，第二行是 `m` 和 `m` 个从小到大排好的整数（1 ≤ n, m ≤ 100000）。把它们合并成一个从小到大的数组输出，用空格隔开。三个数组都要用 `malloc` 申请。

```text
样例输入：
3 1 4 9
4 2 3 10 12
样例输出：1 2 3 4 9 10 12
```

<details>
<summary>提示</summary>

用两个下标 `i`、`j` 分别指向两个数组，每次把较小的那个放进结果数组，并让对应的下标前进；其中一个用完后，把另一个剩下的全部接上。
</details>

<!-- exercise: lines-reverse -->
### 练习 3：倒序输出每一行

输入若干行文字（每行不超过 200 个字符，行数事先不知道），一直读到输入结束，把这些行按**相反的顺序**输出。

```text
样例输入：
first line
second line
third line
样例输出：
third line
second line
first line
```

<details>
<summary>提示</summary>

用一个会翻倍的 `char **lines` 数组，每个元素指向一行。每读一行（用 `fgets` 读进一个临时的 `char buf[202]`，去掉 `'\n'`），就 `malloc(strlen(buf) + 1)` 个字节，用 `strcpy` 复制进去。最后倒着输出，并且**先 `free` 每一行，再 `free(lines)`**。
</details>

<!-- exercise: matrix-mul -->
### 练习 4：矩阵乘法

第一行输入 `n m p`（都在 1 到 50 之间），接下来 `n` 行、每行 `m` 个整数是矩阵 A，再接下来 `m` 行、每行 `p` 个整数是矩阵 B。输出 C = A × B（`n` 行、每行 `p` 个数，用空格隔开）。其中 C 的第 i 行第 j 列等于 A 的第 i 行和 B 的第 j 列对应元素乘积之和。

```text
样例输入：
2 3 2
1 2 3
4 5 6
7 8
9 10
11 12
样例输出：
58 64
139 154
```

<details>
<summary>提示</summary>

三个矩阵都用 11.5 节的一维写法：A 的第 i 行第 k 列是 `a[i * m + k]`。`c[i * p + j] = Σ a[i * m + k] * b[k * p + j]`（k 从 0 到 m-1）。
</details>

<!-- exercise: mem-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int *a = calloc(4, sizeof *a);
    if (a == NULL) {
        return 1;
    }
    a[1] = 5;
    int *b = realloc(a, 6 * sizeof *b);
    if (b == NULL) {
        free(a);
        return 1;
    }
    a = b;
    a[5] = 9;
    printf("%d %d %d %d %d\n", a[0], a[1], a[2], a[3], a[5]);
    free(a);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`calloc` 把 4 个 `int` 都清零，然后 `a[1] = 5`。`realloc` 扩大到 6 个，原来的 4 个值保留（新增的 `a[4]`、`a[5]` 是垃圾值，所以程序没有输出 `a[4]`），再把 `a[5]` 设为 9。输出 `0 5 0 0 9`。
</details>

<!-- exercise: mem-safe -->
### 练习 6：哪一段没有内存错误？

- `int *p = malloc(4 * sizeof *p); p[4] = 1; free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); p = NULL; free(p);`
- `int x = 0; int *p = &x; free(p);`

<details>
<summary>答案与解析</summary>

第三段是正确的：`free` 之后把 `p` 设为 `NULL`，而 `free(NULL)` 什么也不做。第一段越界（只有 `p[0]`～`p[3]`），第二段重复释放，第四段释放了不是 `malloc` 来的内存。
</details>

## 11.9 延伸阅读

- [cppreference：动态内存管理](https://zh.cppreference.com/w/c/memory)（`malloc`、`calloc`、`realloc`、`free`）
- [AddressSanitizer 文档（英文）](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- 下一课：[结构体](../12-structs/zh-CN.md)
