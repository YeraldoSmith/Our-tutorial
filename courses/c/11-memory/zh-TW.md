# 11. 動態記憶體

> 🎯 **本課目標**：明白程式記憶體的幾個區域（堆疊、堆、全域性區），知道什麼時候需要**動態記憶體**；會用 `malloc` 申請記憶體、用 `free` 歸還，並檢查 `NULL`；會用 `calloc` 申請清零的記憶體，用 `realloc` 讓陣列「長大」，寫出能裝下任意多個資料的**動態陣列**；讓函式返回動態分配的記憶體，並說清楚「誰負責釋放」；用一維陣列模擬動態二維陣列；認識記憶體洩漏、釋放後使用、重複釋放等錯誤，並用記憶體檢查詢出它們。
>
> ⏱ **預計用時**：100 分鐘　｜　📋 **前置知識**：本課程第 1～10 課，特別是第 10 課「指標」

## 11.1 為什麼需要動態記憶體

到目前為止，我們的陣列都是**固定大小**的：

```c norun
#define MAX_N 1000
int a[MAX_N];          // 只有 10 個數時浪費；有 100 萬個數時又不夠
```

還有更多問題：

- 區域性變數放在**堆疊**上，堆疊通常只有幾 MB。`int big[10000000];`（約 40 MB）寫在函式里，程式一執行就會崩潰（堆疊溢位，第 7 課）。
- 區域性變數在函式返回時就消失了，所以函式不能返回區域性陣列的地址（第 10 課）。
- 很多時候，資料有多少要到**執行時**才知道，比如「讀到輸入結束為止」。

解決辦法是在**堆**（heap）上**動態**申請記憶體：需要多少就申請多少，用完自己歸還。程式的記憶體大致是這樣分佈的：

```text
┌──────────────────────────────┐
│ 堆疊 stack                     │ 區域性變數、函式引數：函式返回就自動釋放；空間小（通常幾 MB）
│              ↓               │
│                              │
│              ↑               │
│ 堆 heap                      │ malloc 申請：自己申請、自己 free；空間大（可以用到幾 GB）
├──────────────────────────────┤
│ 全域性變數、static 變數          │ 整個程式執行期間一直存在
├──────────────────────────────┤
│ 程式程式碼、字串字面量          │ 只讀
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

    int *a = malloc(n * sizeof *a);      // 申請能放 n 個 int 的記憶體
    if (a == NULL) {                     // 記憶體不夠時返回 NULL，一定要檢查
        printf("記憶體不足\n");
        return 1;
    }

    for (int i = 0; i < n; i++) {
        scanf("%d", &a[i]);              // 用起來和陣列一模一樣
    }
    long long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
    printf("%d 個數，和為 %lld\n", n, sum);

    free(a);                             // 用完了，歸還給系統
    a = NULL;                            // 好習慣：防止之後誤用
    return 0;
}
```

- **`malloc(位元組數)`** 在堆上申請一塊記憶體，返回它的地址（型別是 `void *`，在 C 裡會自動轉換成 `int *`，不用強制轉換）。
- **`n * sizeof *a`**：`*a` 是一個 `int`，`sizeof *a` 就是一個元素的大小。這樣寫的好處是：以後把 `a` 改成 `double *`，這一行不用改。
- `malloc` 來的記憶體裡是**垃圾值**，和未初始化的區域性變數一樣。
- 記憶體不夠時 `malloc` 返回 **`NULL`**，所以拿到之後要先檢查。
- **`free(指標)`** 歸還記憶體。每一塊 `malloc` 來的記憶體都要 `free`，而且**只能 `free` 一次**。`free` 之後指標本身還在，但不能再透過它讀寫了；把它設為 `NULL` 是個好習慣（`free(NULL)` 什麼也不做，是安全的）。

## 11.3 `calloc` 和 `realloc`：會長大的陣列

| 函式 | 作用 |
|------|------|
| `malloc(位元組數)` | 申請記憶體，內容是垃圾值 |
| `calloc(個數, 每個的大小)` | 申請記憶體並**全部清零**，例如 `calloc(n, sizeof *a)` |
| `realloc(p, 新的位元組數)` | 把 `p` 指向的記憶體**改成新的大小**，原來的內容會保留；可能會搬到新的地址，所以要用它的返回值 |
| `free(p)` | 歸還記憶體 |

有了 `realloc`，就能寫出**動態陣列**：先申請一點空間，滿了就把容量**翻倍**。下面的程式能讀入任意多個數：

```c input="5 12 -3 8 0 7 21"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int capacity = 2;                         // 先準備 2 個位置
    int count = 0;                            // 已經用了幾個
    int *data = malloc(capacity * sizeof *data);
    if (data == NULL) {
        return 1;
    }

    int x;
    while (scanf("%d", &x) == 1) {
        if (count == capacity) {              // 滿了：容量翻倍
            capacity *= 2;
            int *bigger = realloc(data, capacity * sizeof *data);
            if (bigger == NULL) {             // 失敗時原來的 data 還在，要記得釋放
                free(data);
                return 1;
            }
            data = bigger;
            printf("（容量擴大到 %d）\n", capacity);
        }
        data[count++] = x;                    // 先存進 data[count]，再把 count 加 1
    }

    printf("讀到 %d 個數：", count);
    for (int i = 0; i < count; i++) {
        printf("%d ", data[i]);
    }
    printf("\n");
    free(data);
    return 0;
}
```

- **為什麼要用臨時變數 `bigger`？** 如果寫成 `data = realloc(data, ...)`，一旦失敗返回 `NULL`，原來那塊記憶體的地址就丟了，再也沒法 `free`——這就是記憶體洩漏。
- **為什麼翻倍，而不是每次加 1？** `realloc` 有時要把所有資料搬到新地方，很費時間。翻倍的話，100 萬個數只需要擴大約 20 次。Python 的列表、C++ 的 `vector` 在內部也是這麼做的。

## 11.4 函式返回動態記憶體

函式的區域性變數在返回後就消失了，但 `malloc` 來的記憶體會一直存在，直到被 `free`。所以函式可以把它返回給呼叫者：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 返回一個新的字串：把 s 重複 times 次。
// 返回的記憶體由呼叫者負責 free；失敗時返回 NULL。
char *repeat(const char *s, int times) {
    size_t len = strlen(s);
    char *result = malloc(len * times + 1);       // +1 留給 '\0'
    if (result == NULL) {
        return NULL;
    }
    for (int i = 0; i < times; i++) {
        memcpy(result + i * len, s, len);         // 把 s 的 len 個位元組複製到第 i 段
    }
    result[len * times] = '\0';
    return result;
}

// 返回一個新的陣列 {0, 1, 4, 9, ...}，共 n 個平方數。呼叫者負責 free。
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

關鍵是說清楚**誰負責釋放**（「所有權」）。習慣上在函式的註釋裡寫明「返回的記憶體由呼叫者 `free`」。`memcpy(目標, 來源, 位元組數)`（在 `<string.h>` 裡）按位元組複製一塊記憶體。順便一提，C23 的標準庫裡也有 `strdup(s)`，它返回 `s` 的一份 `malloc` 來的副本。

## 11.5 動態的二維陣列

最簡單、最通用的辦法是：申請一塊 `行數 × 列數` 的一維記憶體，自己算下標——第 `r` 行第 `c` 列在 `r * cols + c`：

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

這和第 8 課說的「二維陣列在記憶體裡一行接一行地存放」是同一個道理，只是下標由我們自己計算。（GCC 和 Clang 還支援 `int (*m)[cols] = malloc(rows * sizeof *m);`，之後可以直接寫 `m[r][c]`。）

## 11.6 記憶體錯誤，以及怎樣發現它們

動態記憶體帶來的錯誤很隱蔽：程式常常「看起來正常」，過一會兒才莫名其妙地出錯。開啟**記憶體檢查**（工具 → 設定…，第 8 課），它們大都能被當場抓住：

| 錯誤 | 例子 | 記憶體檢查報告 |
|------|------|--------------|
| **記憶體洩漏**：忘了 `free` | 函式里 `malloc` 了，返回前沒 `free` | `LeakSanitizer: detected memory leaks` |
| **釋放後使用** | `free(p);` 之後又 `p[0] = 1;` | `heap-use-after-free` |
| **重複釋放** | 同一個指標 `free` 了兩次 | `attempting double-free` |
| **越界** | `malloc(4 * sizeof *p)` 卻訪問 `p[4]` | `heap-buffer-overflow` |
| **釋放不是 `malloc` 來的記憶體** | `int x; free(&x);` | `attempting free on address which was not malloc()-ed` |

```c norun
void process(void) {
    int *tmp = malloc(1000 * sizeof *tmp);
    // ……使用 tmp……
    return;              // 忘了 free(tmp)：每呼叫一次就丟掉約 4 KB
}
```

程式結束時作業系統會收回所有記憶體，所以短小的程式洩漏一點「好像沒事」。但伺服器、遊戲這類長時間執行的程式，洩漏會越積越多，最終把記憶體耗光。從一開始就養成好習慣：

- 每寫一個 `malloc`，馬上想好在哪裡 `free`；
- `free` 之後把指標設為 `NULL`；
- `realloc` 的結果先存進臨時變數；
- 寫 C 時一直開著記憶體檢查。

## 11.7 小結

- 堆疊：區域性變數，自動釋放，空間小；堆：`malloc` 申請，手動 `free`，空間大。
- `T *p = malloc(n * sizeof *p);`，檢查 `NULL`，用完 `free(p); p = NULL;`。
- `calloc` 申請並清零；`realloc` 改變大小（結果先存臨時變數）；動態陣列滿了就把容量翻倍。
- 函式可以返回 `malloc` 來的記憶體，要寫清楚由誰 `free`。
- 動態二維陣列：一維的 `rows * cols` 塊，下標 `r * cols + c`。
- 常見錯誤：洩漏、釋放後使用、重複釋放、越界、釋放非堆記憶體——用記憶體檢查詢出它們。

## 11.8 練習

<!-- exercise: reverse-unknown -->
### 練習 1：不知道有多少個數

輸入若干個整數（至少 1 個，個數事先不知道，可能多達 100 萬個），一直讀到輸入結束，把它們倒序輸出，用空格隔開。

```text
樣例輸入：1 2 3 4 5
樣例輸出：5 4 3 2 1
```

<details>
<summary>提示</summary>

照著 11.3 節寫一個會翻倍的動態陣列，讀完之後從 `count - 1` 倒著輸出。別忘了 `free`。
</details>

<!-- exercise: merge-sorted -->
### 練習 2：合併兩個有序陣列

第一行是 `n` 和 `n` 個從小到大排好的整數，第二行是 `m` 和 `m` 個從小到大排好的整數（1 ≤ n, m ≤ 100000）。把它們合併成一個從小到大的陣列輸出，用空格隔開。三個陣列都要用 `malloc` 申請。

```text
樣例輸入：
3 1 4 9
4 2 3 10 12
樣例輸出：1 2 3 4 9 10 12
```

<details>
<summary>提示</summary>

用兩個下標 `i`、`j` 分別指向兩個陣列，每次把較小的那個放進結果陣列，並讓對應的下標前進；其中一個用完後，把另一個剩下的全部接上。
</details>

<!-- exercise: lines-reverse -->
### 練習 3：倒序輸出每一行

輸入若干行文字（每行不超過 200 個字元，行數事先不知道），一直讀到輸入結束，把這些行按**相反的順序**輸出。

```text
樣例輸入：
first line
second line
third line
樣例輸出：
third line
second line
first line
```

<details>
<summary>提示</summary>

用一個會翻倍的 `char **lines` 陣列，每個元素指向一行。每讀一行（用 `fgets` 讀進一個臨時的 `char buf[202]`，去掉 `'\n'`），就 `malloc(strlen(buf) + 1)` 個位元組，用 `strcpy` 複製進去。最後倒著輸出，並且**先 `free` 每一行，再 `free(lines)`**。
</details>

<!-- exercise: matrix-mul -->
### 練習 4：矩陣乘法

第一行輸入 `n m p`（都在 1 到 50 之間），接下來 `n` 行、每行 `m` 個整數是矩陣 A，再接下來 `m` 行、每行 `p` 個整數是矩陣 B。輸出 C = A × B（`n` 行、每行 `p` 個數，用空格隔開）。其中 C 的第 i 行第 j 列等於 A 的第 i 行和 B 的第 j 列對應元素乘積之和。

```text
樣例輸入：
2 3 2
1 2 3
4 5 6
7 8
9 10
11 12
樣例輸出：
58 64
139 154
```

<details>
<summary>提示</summary>

三個矩陣都用 11.5 節的一維寫法：A 的第 i 行第 k 列是 `a[i * m + k]`。`c[i * p + j] = Σ a[i * m + k] * b[k * p + j]`（k 從 0 到 m-1）。
</details>

<!-- exercise: mem-predict -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`calloc` 把 4 個 `int` 都清零，然後 `a[1] = 5`。`realloc` 擴大到 6 個，原來的 4 個值保留（新增的 `a[4]`、`a[5]` 是垃圾值，所以程式沒有輸出 `a[4]`），再把 `a[5]` 設為 9。輸出 `0 5 0 0 9`。
</details>

<!-- exercise: mem-safe -->
### 練習 6：哪一段沒有記憶體錯誤？

- `int *p = malloc(4 * sizeof *p); p[4] = 1; free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); free(p);`
- `int *p = malloc(4 * sizeof *p); free(p); p = NULL; free(p);`
- `int x = 0; int *p = &x; free(p);`

<details>
<summary>答案與解析</summary>

第三段是正確的：`free` 之後把 `p` 設為 `NULL`，而 `free(NULL)` 什麼也不做。第一段越界（只有 `p[0]`～`p[3]`），第二段重複釋放，第四段釋放了不是 `malloc` 來的記憶體。
</details>

## 11.9 延伸閱讀

- [cppreference：動態記憶體管理](https://zh.cppreference.com/w/c/memory)（`malloc`、`calloc`、`realloc`、`free`）
- [AddressSanitizer 文件（英文）](https://github.com/google/sanitizers/wiki/AddressSanitizer)
- 下一課：[結構體](../12-structs/zh-CN.md)
