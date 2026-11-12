# 8. 陣列

> 🎯 **本課目標**：用**陣列**存放一組同型別的資料；會定義、初始化陣列，用下標訪問元素，用迴圈遍歷；明白 C **不檢查越界**以及它的後果，學會用「記憶體檢查」找出越界；掌握陣列的常用演算法（求和、最值、查詢、反轉、複製、氣泡排序）；把陣列傳給函式（要同時傳長度），理解為什麼函式能修改陣列；使用二維陣列。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：本課程第 1～7 課

## 8.1 為什麼需要陣列

要存 5 個學生的成績，寫 5 個變數 `s1`、`s2`……還湊合；要是 100 個、1000 個呢？**陣列**就是「一排同型別的格子」，用一個名字加一個**下標**（編號）訪問其中每一格：

```c
#include <stdio.h>

int main(void) {
    int scores[5] = {90, 85, 77, 60, 95};   // 5 個 int 排成一排

    printf("第一個：%d\n", scores[0]);       // 下標從 0 開始
    printf("最後一個：%d\n", scores[4]);     // 長度是 5，最後一個下標是 4

    scores[2] = 80;                          // 修改一個元素
    for (int i = 0; i < 5; i++) {            // 用迴圈處理每一個元素
        printf("scores[%d] = %d\n", i, scores[i]);
    }
    return 0;
}
```

```text
下標：      0    1    2    3    4
         ┌────┬────┬────┬────┬────┐
scores   │ 90 │ 85 │ 80 │ 60 │ 95 │
         └────┴────┴────┴────┴────┘
```

- 定義：`型別 名字[長度];`，比如 `double prices[100];`、`char letters[26];`；
- 下標從 **0** 開始，長度為 n 的陣列，下標是 **0 到 n-1**；
- 每個元素 `scores[i]` 用起來就和普通變數一樣：可以賦值、參與計算、傳給 `printf`；
- 下標可以是任何整數表示式：`scores[i]`、`scores[i + 1]`、`scores[n - 1]`；
- 陣列的元素在記憶體裡**緊挨著**排列，這讓按下標訪問非常快。

`for (int i = 0; i < n; i++)` 是 C 裡最常見的迴圈（第 6 課），正是因為它恰好走遍長度為 n 的陣列的所有下標。

## 8.2 初始化陣列

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};        // 全部給出
    int b[5] = {1, 2};                 // 只給前兩個，其餘自動為 0
    int c[5] = {0};                    // 常用寫法：全部清零
    int d[] = {7, 8, 9};               // 不寫長度：編譯器數出來是 3
    int e[5] = {[1] = 10, [3] = 30};   // 指定下標初始化，其餘為 0

    printf(" a  b  c  e\n");
    for (int i = 0; i < 5; i++) {
        printf("%2d %2d %2d %2d\n", a[i], b[i], c[i], e[i]);
    }
    printf("d 有 %zu 個元素\n", sizeof d / sizeof d[0]);
    return 0;
}
```

- `sizeof d` 是整個陣列佔的位元組數（3 × 4 = 12），`sizeof d[0]` 是一個元素的位元組數（4），相除就是**元素個數**。這個技巧只在**定義陣列的那個函式里**有效（8.6 節會看到原因）。
- **沒有初始化的區域性陣列裡是垃圾值**，和未初始化的變數一樣（第 2 課）。寫 `int a[100] = {0};` 是個好習慣。
- 陣列長度最好用一個**常量**表示，改長度時只需改一處：

```c
#include <stdio.h>

#define N 5                        // 前處理器把下面所有的 N 替換成 5

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

`#define N 5` 是**宏**：編譯之前，前處理器把程式碼裡的 `N` 都換成 `5`。習慣上宏名全用大寫。（C23 也可以寫 `constexpr int N = 5;`，但較老的編譯器不支援。）

## 8.3 C 不檢查越界！

這是 C 和 Python 最重要的區別之一。下面的迴圈多走了一步，訪問了不存在的 `a[5]`：

```c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};
    for (int i = 0; i <= 5; i++) {     // 錯誤：i 會取到 5
        printf("%d ", a[i]);
    }
    printf("\n");
    return 0;
}
```

Python 會立刻報 `IndexError`，C 卻**一聲不吭**：它直接去讀陣列後面那塊記憶體，列印出一個莫名其妙的數（每次執行可能還不一樣）。這叫**未定義行為**（undefined behavior）——什麼都可能發生：

- 讀到垃圾值，程式「看起來正常」地給出錯誤結果；
- **寫**到陣列外面，悄悄改掉了別的變數；程式可能報 `*** stack smashing detected ***` 後被終止；
- 訪問到不屬於程式的記憶體，崩潰並報 `Segmentation fault`（段錯誤）。

很多著名的安全漏洞（「緩衝區溢位」）就是這樣來的。所以：

- 迴圈寫 `i < n`，不要寫 `i <= n`；
- 來自輸入的下標，用之前先檢查：`if (k >= 0 && k < N) { ... }`；
- 用**記憶體檢查**工具把越界揪出來。

### 開啟記憶體檢查

GCC 和 Clang 自帶 **AddressSanitizer**（地址消毒器）。在終端裡編譯時加上 `-fsanitize=address,undefined -g`：

```bash
gcc -fsanitize=address,undefined -g main.c -o main
./main
```

在本教程的桌面程式裡，開啟選單 **工具 → 設定…**，勾選「**C/C++ 記憶體檢查（AddressSanitizer）**」即可（Windows 上的 MinGW 不支援這個功能，程式會自動跳過）。再執行上面的例子，程式會在越界的那一刻停下來，並指出是哪一行：

```text
main.c:6:24: runtime error: index 5 out of bounds for type 'int [5]'
```

> [!TIP]
> 寫 C 程式時，建議**一直開著記憶體檢查**。它會讓程式慢一些，但能把「偶爾出錯、很難重現」的記憶體錯誤變成「當場報錯、指出行號」。第 10、11 課學指標和動態記憶體時，它會更有用。

## 8.4 讀入一組資料

很多題目的輸入格式是「先給個數 n，再給 n 個數」。最簡單的做法是開一個**足夠大**的陣列：

```c input="5\n3 8 1 9 4"
#include <stdio.h>

#define MAX_N 1000                     // 最多支援 1000 個數

int main(void) {
    int a[MAX_N];
    int n = 0;
    scanf("%d", &n);
    if (n < 1 || n > MAX_N) {          // 防止越界：先檢查 n
        printf("n 必須在 1 到 %d 之間\n", MAX_N);
        return 1;
    }
    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);            // a[i] 就是一個 int 變數，同樣要加 &
    }

    long long sum = 0;
    int max_i = 0;                     // 記住最大值的「下標」，而不只是值
    for (int i = 0; i < n; i++) {
        sum += a[i];
        if (a[i] > a[max_i]) {
            max_i = i;
        }
    }
    printf("總和 %lld，平均 %.2f\n", sum, (double)sum / n);
    printf("最大值是第 %d 個數：%d\n", max_i + 1, a[max_i]);
    return 0;
}
```

注意：陣列本身的大小 `MAX_N` 是固定的，`n` 只是「用了前 n 格」。

> [!NOTE]
> 你可能會看到讀入 n 之後直接寫 `int a[n];` 的程式碼，這叫**變長陣列**（VLA）。GCC 和 Clang 支援它，但它是 C 標準裡的「可選功能」，微軟的編譯器不支援；n 太大時還會把堆疊撐爆（第 7 課）。本課程用「固定的最大長度」，到第 11 課再學真正按需分配記憶體的 `malloc`。

## 8.5 常用陣列演算法

```c
#include <stdio.h>

#define N 6

int main(void) {
    int a[N] = {4, 8, 15, 16, 23, 42};

    // 線性查詢：從頭到尾一個一個比較
    int target = 16, pos = -1;         // -1 表示「沒找到」
    for (int i = 0; i < N; i++) {
        if (a[i] == target) {
            pos = i;
            break;                     // 找到第一個就停
        }
    }
    printf("%d 的下標：%d\n", target, pos);

    // 原地反轉：交換 a[i] 和 a[N-1-i]，只需走一半
    for (int i = 0; i < N / 2; i++) {
        int t = a[i];
        a[i] = a[N - 1 - i];
        a[N - 1 - i] = t;
    }

    // 複製：只能一個一個元素地複製
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

- 反轉時如果迴圈走完整個陣列（`i < N`），每對元素會被交換兩次，結果又變回原樣。
- 陣列**不能整體賦值**：寫 `b = a;` 會報錯 `assignment to expression with array type`；也不能用 `==` 比較兩個陣列的內容。複製要用迴圈，或者 `<string.h>` 裡的 `memcpy(b, a, sizeof a);`。

## 8.6 陣列和函式

把陣列傳給函式時，要**同時傳長度**：

```c
#include <stdio.h>

void print_array(const int a[], int n) {   // const：保證不修改陣列
    for (int i = 0; i < n; i++) {
        printf("%d ", a[i]);
    }
    printf("\n");
}

void double_all(int a[], int n) {          // 沒有 const：要修改陣列
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
    int n = sizeof nums / sizeof nums[0];  // 只有在這裡才能這樣算長度
    print_array(nums, n);
    double_all(nums, n);
    print_array(nums, n);                  // 2 4 6 8：main 裡的陣列真的被改了！
    printf("和：%d\n", sum_array(nums, n));
    return 0;
}
```

這裡有兩個和第 7 課「值傳遞」不一樣的地方：

1. **函式能修改呼叫者的陣列**。傳陣列時，C 並不複製整個陣列（那可能很大），而是隻傳「陣列在哪裡」——第一個元素的**地址**。所以 `double_all` 改的就是 `main` 裡的 `nums`。不想讓函式修改，就在引數前加 `const`，萬一不小心改了，編譯器會報錯。
2. **函式里不知道陣列有多長**。引數 `int a[]` 其實只是一個地址（第 10 課叫它「指標」），`sizeof a` 得到的是地址的大小（通常 8），而不是陣列的大小。GCC 會警告：

```text
warning: 'sizeof' on array function parameter 'a' will return size of 'int *'
```

所以長度一定要作為引數單獨傳進去。另外，函式**不能返回一個陣列**；要「產出」一組資料，就讓呼叫者準備好陣列，傳進來讓函式填寫（就像 `double_all`）。

## 8.7 排序：氣泡排序

**排序**是最經典的演算法問題。**氣泡排序**的思路：從左到右比較相鄰的兩個數，前面的比後面的大就交換。走完一輪，最大的數就像氣泡一樣「浮」到了最右邊；再對剩下的數重複，直到某一輪一次都沒交換：

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
        for (int i = 0; i < n - 1 - pass; i++) {   // 最後 pass 個已經排好了
            if (a[i] > a[i + 1]) {
                int t = a[i];
                a[i] = a[i + 1];
                a[i + 1] = t;
                swapped = true;
            }
        }
        printf("第 %d 輪後：", pass + 1);
        print_array(a, n);
        if (!swapped) {
            break;                                 // 一次都沒交換：已經有序
        }
    }
}

int main(void) {
    int a[] = {5, 1, 4, 2, 8};
    bubble_sort(a, 5);
    return 0;
}
```

- 內層迴圈到 `n - 1 - pass` 為止：因為要訪問 `a[i + 1]`，`i` 最大隻能到倒數第二個，否則就越界了；而最後 `pass` 個數已經在正確的位置上。
- 氣泡排序要比較大約 n²/2 次：1000 個數約 50 萬次，還很快；100 萬個數就要約 5000 億次，太慢了。實際程式設計中用標準庫的 `qsort`，它快得多——學完指標（第 10 課）後我們就會用它。

## 8.8 二維陣列

表格、棋盤、影象、矩陣都是「行 × 列」的資料，用**二維陣列**表示：

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
    m[1][2] = 70;                      // 第 1 行第 2 列（都從 0 數起）

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

- `m[r][c]`：先行後列。`int m[3][4]` 可以看成「3 個元素的陣列，每個元素又是一個 4 個 int 的陣列」。
- 在記憶體裡，二維陣列是**一行接一行**連續存放的：`m[0][0] m[0][1] m[0][2] m[0][3] m[1][0] …`。
- 遍歷用兩層迴圈，外層走行、內層走列（第 6 課的巢狀迴圈）。

把二維陣列傳給函式時，**列數**必須告訴編譯器（它要靠列數算出 `m[r][c]` 在哪裡）。可以把行數、列數作為引數，並寫在陣列引數前面：

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

（`const int m[rows][cols]` 這種用引數當長度的寫法需要 GCC 或 Clang；也可以寫成固定列數 `const int m[][3]`。）

## 8.9 小結

- 陣列：`型別 名字[長度]`，下標 **0 到 n-1**；`int a[5] = {0};` 全部清零，`int d[] = {...}` 讓編譯器數長度。
- `sizeof a / sizeof a[0]` 求元素個數，只在定義陣列的函式里有效；長度用 `#define N 5` 這樣的常量。
- **C 不檢查越界**：越界是未定義行為，可能得到垃圾值、悄悄改壞資料或崩潰。迴圈用 `i < n`，並開啟記憶體檢查（`-fsanitize=address,undefined`）。
- 「先讀 n 再讀 n 個數」：開一個固定的最大長度，並檢查 n。
- 常用演算法：求和、最值（記下標）、線性查詢（找不到返回 -1）、反轉（走一半）、複製（逐個元素）、氣泡排序。
- 陣列不能用 `=` 整體賦值，也不能用 `==` 比較。
- 傳陣列給函式：同時傳長度；函式**能修改**原陣列，不想修改就加 `const`；函式里 `sizeof` 得不到陣列長度。
- 二維陣列 `m[行][列]` 按行連續存放，用兩層迴圈遍歷；傳給函式時要告訴它列數。

## 8.10 練習

<!-- exercise: reverse-array -->
### 練習 1：倒過來輸出

輸入一個正整數 `n`（不超過 1000），再輸入 `n` 個整數，把它們倒序輸出，用空格隔開。

```text
樣例輸入：
5
1 2 3 4 5
樣例輸出：5 4 3 2 1
```

<details>
<summary>提示</summary>

先全部讀進陣列，再 `for (int i = n - 1; i >= 0; i--)` 輸出。注意第一個數前面不要有空格：可以在 `i < n - 1` 時先輸出一個空格。
</details>

<!-- exercise: array-stats -->
### 練習 2：最大、最小和平均

輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數，輸出最大值、最小值和平均值（保留 2 位小數），用空格隔開。

```text
樣例輸入：
4
3 -2 10 5
樣例輸出：10 -2 4.00
```

<details>
<summary>提示</summary>

最大值和最小值的初始值都設為 `a[0]`，而不是 0——否則全是負數（或全是正數）時會出錯。平均值用 `(double)sum / n`。
</details>

<!-- exercise: above-average -->
### 練習 3：高於平均

輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數，輸出**嚴格大於**平均值的數有幾個。

```text
樣例輸入：
5
1 2 3 4 10
樣例輸出：1
```

<details>
<summary>提示</summary>

要先求出平均值，才能逐個比較——所以必須把數都存進陣列。為了避開小數，可以把 `a[i] > sum / n` 改寫成 `a[i] * n > sum`（用 `long long`）。
</details>

<!-- exercise: sort-numbers -->
### 練習 4：排序

輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數，把它們從小到大排序後輸出，用空格隔開。

```text
樣例輸入：
6
5 3 9 1 3 7
樣例輸出：1 3 3 5 7 9
```

<details>
<summary>提示</summary>

照著 8.7 節寫 `bubble_sort`（去掉每輪列印的部分）。
</details>

<!-- exercise: transpose -->
### 練習 5：矩陣轉置

輸入行數 `r` 和列數 `c`（都在 1 到 10 之間），再輸入 `r` 行、每行 `c` 個整數。輸出它的**轉置**：原來的第 i 行變成第 i 列，所以輸出有 `c` 行、每行 `r` 個數，用空格隔開。

```text
樣例輸入：
2 3
1 2 3
4 5 6
樣例輸出：
1 4
2 5
3 6
```

<details>
<summary>提示</summary>

定義 `int m[10][10];` 讀入。輸出時外層迴圈走原來的列 `j`，內層迴圈走原來的行 `i`，列印 `m[i][j]`。
</details>

<!-- exercise: array-predict -->
### 練習 6：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`x` 是值傳遞，函式改的是副本，外面的 `x` 還是 1；陣列傳的是地址，`a[0]` 真的被改成了 100；`a[2]` 沒有給初始值，自動為 0。輸出 `1 100 2 0`。
</details>

<!-- exercise: array-index -->
### 練習 7：合法的下標

`int a[10];` 的合法下標是哪些？

- 1 到 10
- 0 到 9
- 0 到 10
- 任意整數都可以，C 會自動檢查

<details>
<summary>答案與解析</summary>

長度為 10 的陣列，下標從 0 到 9。C **不會**檢查越界，訪問 `a[10]` 是未定義行為。
</details>

## 8.11 延伸閱讀

- [cppreference：陣列宣告](https://zh.cppreference.com/w/c/language/array)、[陣列初始化](https://zh.cppreference.com/w/c/language/array_initialization)
- [AddressSanitizer 文件（英文）](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- [VisuAlgo：排序演算法動畫](https://visualgo.net/zh/sorting)——看氣泡排序一步一步是怎麼交換的
- 下一課：[字串](../09-strings/zh-CN.md)
