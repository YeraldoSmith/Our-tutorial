# 7. 函数

> 🎯 **本课目标**：把一段功能写成**函数**，理解参数、返回值和 `void`；明白为什么函数要「先声明后使用」，会写**函数原型**；理解 C 的参数是**值传递**（函数拿到的是副本）；分清局部变量、全局变量和 `static` 局部变量；写出递归函数（阶乘、汉诺塔），了解**调用栈**和栈溢出。
>
> ⏱ **预计用时**：80 分钟　｜　📋 **前置知识**：本课程第 1～6 课

## 7.1 定义和调用函数

我们一直在用别人写好的函数：`printf`、`scanf`、`sqrt`……`main` 本身也是一个函数。现在来写自己的：

```c
#include <stdio.h>

int square(int x) {          // 返回类型  函数名(参数类型 参数名)
    return x * x;            // return：把结果交回给调用者，并结束函数
}

void greet(void) {           // void：不返回任何值；参数里的 void：不需要参数
    printf("你好！\n");
}

int main(void) {
    greet();
    int a = square(5);       // 调用 square，把 5 传给 x，得到 25
    printf("%d\n", a);
    printf("%d\n", square(3) + square(4));   // 返回值可以直接参与计算：25
    return 0;
}
```

| 部分 | 意思 |
|------|------|
| `int` | **返回类型**：函数交回一个 `int`。不返回任何东西就写 `void` |
| `square` | 函数名，命名规则和变量一样 |
| `(int x)` | **参数列表**：每个参数都要写类型，多个参数用逗号隔开，如 `(int a, double b)` |
| `return x * x;` | 返回结果。`void` 函数可以只写 `return;` 提前结束，或者不写 |

和 Python 一样，函数可以避免重复代码、让程序更好读、更容易测试。和 Python 不同的是：**参数和返回值都必须写明类型**。

## 7.2 先声明，后使用：函数原型

C 编译器**从上往下**读代码。如果在 `main` 里调用了一个写在 `main` 后面的函数，编译器读到调用的时候还不认识它：

```c norun
#include <stdio.h>

int main(void) {
    printf("%d\n", square(4));     // 这里还不知道 square 是什么
    return 0;
}

int square(int x) {
    return x * x;
}
```

```text
warning: implicit declaration of function 'square' [-Wimplicit-function-declaration]
```

（新版本的编译器会直接报错。）解决办法有两个：把函数定义挪到调用之前；或者在前面写一个**函数原型**（声明）——只写「签名」，不写函数体，以分号结尾：

```c
#include <stdio.h>

int square(int x);                 // 函数原型：告诉编译器「有这样一个函数」
double average(int a, int b);

int main(void) {
    printf("%d %.1f\n", square(4), average(3, 4));
    return 0;
}

int square(int x) {                // 函数的定义可以放在后面
    return x * x;
}

double average(int a, int b) {
    return (a + b) / 2.0;          // 除以 2.0 才是小数除法（第 3 课）
}
```

很多 C 程序都是这种结构：开头是所有函数的原型，接着是 `main`，最后是各个函数的定义。多文件的程序会把原型放进**头文件**（`.h`）里——`#include <stdio.h>` 其实就是把 `printf` 等函数的原型包含进来。第 14 课会讲。

## 7.3 值传递：函数拿到的是副本

调用函数时，实参的**值被复制**给参数。函数里修改参数，不会影响外面的变量：

```c
#include <stdio.h>

void try_to_change(int n) {
    n = 100;                       // 只改了副本
    printf("函数里 n = %d\n", n);
}

void swap(int a, int b) {          // 想交换两个数……
    int t = a;
    a = b;
    b = t;
}

int main(void) {
    int x = 5;
    try_to_change(x);
    printf("外面 x = %d\n", x);    // 还是 5

    int p = 1, q = 2;
    swap(p, q);
    printf("p = %d, q = %d\n", p, q);   // 还是 1 和 2：交换失败了！
    return 0;
}
```

想让函数修改外面的变量（比如真正的 `swap`），或者「返回多个值」，需要把变量的**地址**传进去——这正是 `scanf("%d", &x)` 要加 `&` 的原因。第 10 课学了指针，我们就能写出正确的 `swap`。

## 7.4 局部变量、全局变量和 static

```c
#include <stdio.h>

int total_calls = 0;               // 全局变量：写在所有函数外面，所有函数都能用

int next_id(void) {
    static int id = 0;             // static 局部变量：只初始化一次，函数结束后值仍然保留
    id++;
    total_calls++;
    return id;
}

int main(void) {
    int a = next_id();
    int b = next_id();
    int c = next_id();
    printf("%d %d %d\n", a, b, c);          // 1 2 3
    printf("一共调用了 %d 次\n", total_calls);
    return 0;
}
```

| 种类 | 写在哪里 | 谁能用 | 活多久 |
|------|----------|--------|--------|
| 局部变量 | 函数（或代码块）里面 | 只有这个函数 | 函数结束就消失 |
| `static` 局部变量 | 函数里面，加 `static` | 只有这个函数 | 整个程序运行期间，值一直保留 |
| 全局变量 | 所有函数外面 | 所有函数 | 整个程序运行期间 |

> [!TIP]
> **尽量少用全局变量**。任何函数都能修改它，程序一大就很难弄清楚是谁、在什么时候改了它。能用参数和返回值传递的，就不要用全局变量。

## 7.5 递归：函数调用自己

和 Python（Python 课第 10 课）一样，C 的函数也可以调用自己：

```c
#include <stdio.h>

long long factorial(int n) {
    if (n <= 1) {                  // 终止条件
        return 1;
    }
    return n * factorial(n - 1);   // 把问题变成更小的同类问题
}

int main(void) {
    for (int i = 0; i <= 20; i += 5) {
        printf("%d! = %lld\n", i, factorial(i));
    }
    return 0;
}
```

递归最漂亮的例子之一是**汉诺塔**：把 n 个盘子从 A 柱移到 C 柱，每次移一个，大盘子不能压在小盘子上。思路是：先把上面 n−1 个移到 B，再把最大的移到 C，最后把 n−1 个从 B 移到 C——而「移 n−1 个」又是同样的问题：

```c
#include <stdio.h>

void hanoi(int n, char from, char via, char to) {
    if (n == 0) {
        return;
    }
    hanoi(n - 1, from, to, via);         // 上面 n-1 个：from → via
    printf("%c -> %c\n", from, to);      // 最大的一个：from → to
    hanoi(n - 1, via, from, to);         // 那 n-1 个：via → to
}

int main(void) {
    hanoi(3, 'A', 'B', 'C');             // 3 个盘子需要 7 步
    return 0;
}
```

### 调用栈

每次调用函数，计算机都会在内存的**栈**（stack）上开辟一块空间，存放这次调用的参数、局部变量和「执行完要回到哪里」；函数返回时这块空间被释放。递归调用 `factorial(4)` 时，栈上会同时有 `factorial(4)`、`factorial(3)`、`factorial(2)`、`factorial(1)` 四层。

栈的大小是有限的（通常几 MB）。如果递归**忘了终止条件**，或者层数太深，栈就会被用光——**栈溢出**（stack overflow），程序会崩溃，通常显示 `Segmentation fault`。著名的程序员问答网站 Stack Overflow 就是以此命名的。

## 7.6 好函数的习惯

- **一个函数只做一件事**，名字说清楚它做什么：`is_prime`、`read_scores`、`print_report`；
- 函数不要太长，超过一屏就考虑拆分；
- 让函数通过参数拿数据、通过返回值交结果，少依赖全局变量；
- 「计算」和「输入输出」分开：`int max3(int a, int b, int c)` 只负责计算，打印交给 `main`——这样的函数最容易测试和复用。

## 7.7 小结

- 函数定义：`返回类型 名字(参数类型 参数, ...) { ... return 值; }`；不返回值用 `void`，没有参数写 `(void)`。
- C 从上往下读代码：函数要**先声明后使用**，可以在前面写原型 `int square(int x);`。
- 参数是**值传递**：函数修改参数不影响外面的变量；要修改外面的变量需要传地址（第 10 课）。
- 局部变量只在函数里有效；`static` 局部变量的值在多次调用之间保留；少用全局变量。
- 递归必须有终止条件；每次调用占用一块栈空间，太深会栈溢出。

## 7.8 练习

<!-- exercise: max3-c -->
### 练习 1：三个数的最大值

补全函数 `int max3(int a, int b, int c)`，返回三个数中最大的一个。`main` 已经写好：读入三个整数，输出 `max3` 的结果。

```text
样例输入：3 9 4
样例输出：9
```

<details>
<summary>提示</summary>

`int m = a; if (b > m) m = b; if (c > m) m = c; return m;`
</details>

<!-- exercise: gcd -->
### 练习 2：最大公约数和最小公倍数

写函数 `int gcd(int a, int b)` 用**辗转相除法**求最大公约数：当 `b` 不为 0 时，不断把 `(a, b)` 换成 `(b, a % b)`，最后的 `a` 就是答案。输入两个正整数，输出它们的最大公约数和最小公倍数，空格隔开。

```text
样例输入：12 18
样例输出：6 36
```

<details>
<summary>提示</summary>

最小公倍数 = `a / gcd(a, b) * b`（先除再乘，避免溢出）。用 `long long` 更保险。
</details>

<!-- exercise: count-primes -->
### 练习 3：区间里的质数

写函数 `bool is_prime(int n)`，然后用它统计闭区间 `[a, b]` 里有多少个质数。输入 `a b`，输出个数。

```text
样例输入：10 30
样例输出：6
```

<details>
<summary>提示</summary>

`is_prime` 里：小于 2 返回 `false`；`for (int d = 2; d * d <= n; d++)` 能整除就返回 `false`；最后返回 `true`。
</details>

<!-- exercise: hanoi -->
### 练习 4：汉诺塔

输入盘子数 `n`（1 ≤ n ≤ 10），按 7.5 节的方法输出把它们从 `A` 移到 `C`（借助 `B`）的每一步，格式 `A -> C`。

```text
样例输入：2
样例输出：
A -> B
A -> C
B -> C
```

<details>
<summary>提示</summary>

照着 7.5 节的 `hanoi` 函数写，`main` 里调用 `hanoi(n, 'A', 'B', 'C');`。
</details>

<!-- exercise: func-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

void swap(int a, int b) {
    int t = a;
    a = b;
    b = t;
}

int counter(void) {
    static int n = 0;
    n += 10;
    return n;
}

int main(void) {
    int x = 1, y = 2;
    swap(x, y);
    printf("%d %d\n", x, y);
    counter();
    counter();
    printf("%d\n", counter());
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`swap` 只交换了副本，`x`、`y` 不变：`1 2`。`counter` 里的 `n` 是 `static`，每次调用都在上一次的基础上加 10：前两次调用后是 20，第三次返回 30。输出 `1 2` 和 `30`。
</details>

<!-- exercise: prototype -->
### 练习 6：为什么要写函数原型？

- 让函数运行得更快
- C 编译器从上往下读代码，调用一个后面才定义的函数之前，需要先用原型告诉编译器它的名字、参数和返回类型
- 原型就是函数的定义，写了就不用再写函数体
- 只有 `void` 函数才需要原型

<details>
<summary>答案与解析</summary>

原型让编译器在看到调用时就能检查参数个数和类型。也可以把函数定义直接放在调用之前，那样就不需要原型。
</details>

## 7.9 延伸阅读

- [cppreference：函数声明](https://zh.cppreference.com/w/c/language/function_declaration)、[存储期与链接（static）](https://zh.cppreference.com/w/c/language/storage_duration)
- 下一课：[数组](../08-arrays/zh-CN.md)
