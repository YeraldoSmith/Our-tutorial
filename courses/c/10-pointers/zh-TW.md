# 10. 指標

> 🎯 **本課目標**：理解記憶體和**地址**；會定義指標、用 `&` 取地址、用 `*` 訪問指標指向的變數；用指標引數寫出真正能交換兩個變數的 `swap`，讓函式「返回」多個結果；認識 `NULL` 和野指標；理解指標和陣列的關係（指標運算、`a[i]` 就是 `*(a + i)`）；用指標處理字串（`strchr`、`strtol`），知道字串字面量不能修改；瞭解 `void *`、函式指標，用標準庫的 `qsort` 排序。
>
> ⏱ **預計用時**：110 分鐘　｜　📋 **前置知識**：本課程第 1～9 課

指標是 C 語言最有特色、也最讓初學者緊張的部分。其實它只有一個核心概念：**地址**。慢慢來，每個例子都執行一下、改一改。

## 10.1 記憶體和地址

可以把記憶體想象成一條很長很長的街道，街上每棟房子（每個**位元組**）都有一個門牌號，這個門牌號就叫**地址**。每個變數都住在某個地址上，`int` 佔 4 棟連著的房子，`double` 佔 8 棟。

```c
#include <stdio.h>

int main(void) {
    int age = 20;
    double price = 9.5;
    printf("age 的值是 %d，地址是 %p\n", age, (void *)&age);
    printf("price 的值是 %.1f，地址是 %p\n", price, (void *)&price);
    printf("age 佔 %zu 位元組，price 佔 %zu 位元組\n", sizeof age, sizeof price);
    return 0;
}
```

- `&age` 讀作「age 的地址」，`&` 叫**取地址運算子**。你在 `scanf("%d", &age)` 裡早就見過它了。
- `%p` 以十六進位制輸出地址；按規定要先轉換成 `(void *)` 型別（10.8 節解釋 `void *`）。
- 每次執行，地址很可能都不一樣——這是作業系統為了安全故意打亂的，不用在意具體的數值。

## 10.2 指標變數

**指標**就是「存放地址的變數」。

```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int *p = &x;                  // p 是「指向 int 的指標」，裡面存著 x 的地址

    printf("x = %d\n", x);
    printf("p 裡存的地址：%p\n", (void *)p);
    printf("*p = %d\n", *p);      // *p：順著這個地址找到的那個 int，也就是 x

    *p = 25;                      // 透過指標修改 x
    printf("x = %d\n", x);        // 25

    int y = 7;
    p = &y;                       // 指標可以改為指向別的變數
    *p += 1;
    printf("y = %d\n", y);        // 8
    return 0;
}
```

```text
       x                    p
    ┌──────┐          ┌──────────┐
    │  25  │ ◀─────── │ x 的地址  │
    └──────┘          └──────────┘
```

| 寫法 | 意思 |
|------|------|
| `int *p;` | 定義一個指標 `p`，它可以存放某個 `int` 變數的地址 |
| `&x` | 取 `x` 的地址 |
| `*p` | **解引用**：`p` 指向的那個變數本身，可以讀也可以寫 |

`*` 有兩種用法，別搞混：**定義時**的 `int *p` 表示「p 是指標」；**使用時**的 `*p` 表示「p 指向的東西」。

> [!TIP]
> `int *p` 和 `int* p` 是一樣的，但推薦前一種寫法。因為 `int* a, b;` 裡只有 `a` 是指標，`b` 只是一個普通的 `int`！定義兩個指標要寫 `int *a, *b;`。

指標的型別要和它指向的變數一致：`int` 的地址存在 `int *` 裡，`double` 的地址存在 `double *` 裡。

## 10.3 指標作為函式引數

第 7 課裡的 `swap` 失敗了，因為函式拿到的是副本。現在把**地址**傳進去：

```c
#include <stdio.h>

void swap(int *a, int *b) {        // 引數是兩個地址
    int t = *a;
    *a = *b;
    *b = t;
}

void min_max(const int arr[], int n, int *min, int *max) {   // 用指標「返回」兩個結果
    *min = arr[0];
    *max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < *min) {
            *min = arr[i];
        }
        if (arr[i] > *max) {
            *max = arr[i];
        }
    }
}

int main(void) {
    int p = 1, q = 2;
    swap(&p, &q);                       // 傳的是地址
    printf("p = %d, q = %d\n", p, q);   // p = 2, q = 1：成功了！

    int data[] = {4, -2, 9, 0};
    int lo = 0, hi = 0;
    min_max(data, 4, &lo, &hi);
    printf("最小 %d，最大 %d\n", lo, hi);
    return 0;
}
```

C 仍然是值傳遞——只不過這次複製的是**地址**。副本和原來的地址指向同一個地方，所以透過 `*a` 修改的就是 `main` 裡的 `p`。

現在 `scanf("%d", &x)` 也就清楚了：`scanf` 要把讀到的數存進 `x`，所以需要 `x` 的地址。

## 10.4 `NULL` 和野指標

**沒有初始化的指標**裡是一個垃圾地址，叫**野指標**。透過它讀寫，就是往一個隨機的地方寫資料——可能崩潰，也可能悄悄改壞別的資料：

```c norun
int *p;          // 沒有初始化：野指標
*p = 5;          // 未定義行為！
```

規則很簡單：

- 指標定義時就初始化：要麼指向一個變數（`&x`），要麼設成 **`NULL`**——一個特殊的「空地址」，表示「哪兒也不指」；
- 不確定指標是否有效時，用之前先檢查 `if (p != NULL)`；
- 透過 `NULL` 讀寫會立刻崩潰（`Segmentation fault`）。開啟記憶體檢查（第 8 課）後，它會告訴你是哪一行。

很多函式用返回 `NULL` 表示「沒找到」或「失敗」：

```c
#include <stdio.h>

int *find(int arr[], int n, int target) {   // 找到就返回那個元素的地址，找不到返回 NULL
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return &arr[i];
        }
    }
    return NULL;
}

int main(void) {
    int data[] = {3, 8, 5};
    int *p = find(data, 3, 8);
    if (p != NULL) {
        *p = 80;                     // 透過指標直接修改陣列裡的那個元素
    }
    if (find(data, 3, 42) == NULL) {
        printf("沒找到 42\n");
    }
    printf("%d %d %d\n", data[0], data[1], data[2]);   // 3 80 5
    return 0;
}
```

> [!WARNING]
> 函式**不能返回自己的區域性變數的地址**：函式一結束，區域性變數就消失了，那個地址變成了「懸空指標」。上面的 `find` 返回的是呼叫者的陣列裡的元素地址，這是可以的。GCC 發現返回區域性變數地址時會警告 `function returns address of local variable`。

## 10.5 指標和陣列

陣列和指標關係非常密切：

```c
#include <stdio.h>

int main(void) {
    int a[5] = {10, 20, 30, 40, 50};
    int *p = a;                       // 陣列名自動變成首元素的地址，等於 &a[0]

    printf("%d %d\n", *p, *(p + 1));  // 10 20：p + 1 指向下一個元素
    printf("%d %d\n", a[3], *(a + 3));// a[i] 就是 *(a + i)

    for (int *q = a; q < a + 5; q++) {   // 用指標遍歷陣列
        printf("%d ", *q);
    }
    printf("\n");

    int *r = &a[4];
    printf("r 和 p 相隔 %td 個元素\n", r - p);                      // 4
    printf("sizeof a = %zu，sizeof p = %zu\n", sizeof a, sizeof p);   // 20 和 8
    return 0;
}
```

- **陣列名在大多數表示式裡會自動變成首元素的地址**。所以 `int *p = a;` 不用寫 `&`，`scanf("%s", name)` 也不用寫 `&`。
- **指標運算**：`p + 1` 不是地址加 1，而是**前進一個元素**（對 `int *` 來說前進 4 個位元組）。兩個指向同一陣列的指標相減，得到相隔的元素個數（用 `%td` 輸出）。
- `a[i]` 只是 `*(a + i)` 的簡寫。
- 函式引數 `int a[]` 和 `int *a` **完全一樣**——這就是第 8 課裡函式內 `sizeof a` 得到 8 的原因：那裡的 `a` 其實是個指標。

但陣列不等於指標：陣列是一整塊存放資料的記憶體，`sizeof a` 是整個陣列的大小，而且 `a = p;`、`a++` 都是不允許的；指標只是一個存放地址的變數（通常 8 位元組），可以隨時改為指向別處。

## 10.6 指標和字串

字串函式大量使用指標。`strchr`、`strstr` 返回的就是指向找到的位置的指標：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(void) {
    char text[] = "name=Ada;lang=C";
    char *eq = strchr(text, '=');              // 指向第一個 '='
    if (eq != NULL) {
        printf("'=' 的下標是 %td\n", eq - text);   // 指標相減：4
        printf("它後面是 %s\n", eq + 1);           // 從下一個字元開始的字串
    }

    const char *greeting = "hello";            // 指向字串字面量：只能讀，不能改
    printf("%s 的第一個字母是 %c\n", greeting, greeting[0]);

    // strtol：把字串轉成整數，並透過 end 告訴你在哪裡停下
    const char *input = "123abc";
    char *end;
    long value = strtol(input, &end, 10);      // 10 表示十進位制
    printf("讀到 %ld，剩下 \"%s\"\n", value, end);
    if (end == input) {
        printf("一個數字也沒讀到\n");
    } else if (*end != '\0') {
        printf("數字後面還有別的字元\n");
    }
    return 0;
}
```

- `strtol(字串, &end, 進位制)` 比 `atoi` 嚴謹：`end` 指向第一個沒被轉換的字元。`end == input` 說明一個數字都沒有；`*end == '\0'` 說明整個字串都是數字。還有對應小數的 `strtod`。
- **字串字面量不能修改**。`"hello"` 存放在只讀的記憶體裡，下面的程式碼會崩潰：

```c norun
char *s = "hello";
s[0] = 'J';          // 段錯誤！
```

| 寫法 | 含義 |
|------|------|
| `char s[] = "hello";` | **陣列**：把字面量**複製**一份到陣列裡，可以隨便修改 |
| `const char *s = "hello";` | **指標**：指向只讀的字面量。加上 `const`，萬一寫了 `s[0] = 'J'`，編譯器會直接報錯 |

## 10.7 `const` 和指標

函式引數裡的 `const` 是一種承諾：

- `const int *p`（或第 8 課的 `const int a[]`）：**不能透過 `p` 修改**它指向的資料。讀可以，寫會編譯報錯 `assignment of read-only location`。
- `int *const p`：`p` **自己**不能再改為指向別處，但可以透過它修改資料。這個較少用到。

經驗：函式只讀取指標指向的資料時，就給引數加上 `const`。呼叫者一看就知道「我的資料不會被改」，編譯器也會幫你檢查。

## 10.8 `void *`、函式指標和 `qsort`

### 函式指標

函式也有地址，存放函式地址的變數叫**函式指標**：

```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int mul(int a, int b) {
    return a * b;
}

int main(void) {
    int (*op)(int, int) = add;     // op：指向「接收兩個 int、返回 int」的函式
    printf("%d\n", op(3, 4));      // 7
    op = mul;                      // 換成另一個函式
    printf("%d\n", op(3, 4));      // 12
    return 0;
}
```

寫法看起來有點怪：`int (*op)(int, int)` 中，`(*op)` 說明 `op` 是指標，後面的 `(int, int)` 和前面的 `int` 說明它指向什麼樣的函式。函式名本身（不帶括號）就代表函式的地址。

### 用 `qsort` 排序

標準庫 `<stdlib.h>` 的 `qsort` 能排序**任何型別**的陣列，而且比氣泡排序快得多。因為它不知道元素是什麼型別，所以需要你提供一個**比較函式**，告訴它兩個元素誰在前：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int compare_ints(const void *a, const void *b) {
    int x = *(const int *)a;          // 先把 void * 轉成 int *，再取值
    int y = *(const int *)b;
    return (x > y) - (x < y);         // x < y 返回 -1，相等返回 0，x > y 返回 1
}

int compare_words(const void *a, const void *b) {
    return strcmp((const char *)a, (const char *)b);
}

int main(void) {
    int nums[] = {42, 7, 19, -3, 7};
    int n = sizeof nums / sizeof nums[0];
    qsort(nums, n, sizeof nums[0], compare_ints);    // 陣列、個數、每個元素的大小、比較函式
    for (int i = 0; i < n; i++) {
        printf("%d ", nums[i]);
    }
    printf("\n");

    char words[][10] = {"pear", "apple", "fig", "banana"};
    qsort(words, 4, sizeof words[0], compare_words);
    for (int i = 0; i < 4; i++) {
        printf("%s ", words[i]);
    }
    printf("\n");
    return 0;
}
```

- **`void *`** 是「通用指標」：可以存放任何型別資料的地址，但不知道那裡是什麼型別，所以**不能直接用 `*` 取值**，要先轉換成具體的指標型別，比如 `(const int *)a`。
- 比較函式的格式是固定的：`int 名字(const void *a, const void *b)`。`a` 應該排在 `b` 前面就返回負數，後面就返回正數，相等返回 0。
- 為什麼不直接寫 `return x - y;`？當 `x`、`y` 很大或很小時，相減可能**溢位**（第 3 課），得到錯誤的符號。`(x > y) - (x < y)` 永遠不會溢位。
- 想從大到小排序，把比較的結果反過來即可，比如 `return compare_ints(b, a);`。

## 10.9 常見錯誤

| 錯誤 | 後果 | 正確做法 |
|------|------|----------|
| `int *p; *p = 1;`（野指標） | 崩潰或改壞資料 | 定義時就初始化為 `&x` 或 `NULL` |
| 透過 `NULL` 讀寫 | 段錯誤 | 使用前檢查 `if (p != NULL)` |
| 返回區域性變數的地址 | 懸空指標 | 返回值本身，或讓呼叫者傳指標進來，或用 `malloc`（第 11 課） |
| 忘了 `&` 或多寫了 `*` | 警告 `makes pointer from integer without a cast` | 分清 `x`、`&x`、`p`、`*p` |
| `int* a, b;` | `b` 不是指標 | `int *a, *b;` |
| 修改字串字面量 | 段錯誤 | 要修改就用 `char s[] = "..."`；不修改就寫 `const char *` |
| 在函式里用 `sizeof` 求陣列引數的長度 | 得到指標的大小（8） | 把長度作為引數傳進去 |

## 10.10 小結

- 每個變數都有地址；`&x` 取地址，`%p` 輸出地址。
- `int *p = &x;` 定義指向 `x` 的指標；`*p` 就是 `x` 本身，可以讀寫。
- 傳指標給函式，函式就能修改呼叫者的變數（`swap`），也能「返回」多個結果。
- 指標要初始化；`NULL` 表示「不指向任何東西」，用前檢查；不要返回區域性變數的地址。
- 陣列名會變成首元素地址；`p + 1` 前進一個元素；`a[i]` 就是 `*(a + i)`；函式引數 `int a[]` 就是 `int *a`。
- `strchr`/`strstr` 返回指標；`strtol` 用 `end` 指標報告停在哪裡；字串字面量只讀，用 `const char *` 指向它。
- `const int *p` 承諾不透過 `p` 修改資料。
- `void *` 是通用指標，要先轉換型別才能取值；函式名就是函式的地址；`qsort` + 比較函式 `int cmp(const void *a, const void *b)` 可以排序任何陣列。

## 10.11 練習

<!-- exercise: swap-ptr -->
### 練習 1：真正的交換

補全函式 `void swap(int *a, int *b)`，交換兩個指標指向的整數。`main` 已經寫好：讀入兩個整數，交換後輸出。

```text
樣例輸入：3 8
樣例輸出：8 3
```

<details>
<summary>提示</summary>

和 10.3 節一樣：用一個臨時變數 `int t = *a;`。
</details>

<!-- exercise: split-time -->
### 練習 2：拆分時間

寫函式 `void split_time(int total, int *h, int *m, int *s)`，把總秒數拆成時、分、秒，透過指標「返回」。輸入總秒數（0 ≤ total < 1000000），按 `時:分:秒` 輸出，分和秒都佔兩位。

```text
樣例輸入：3661
樣例輸出：1:01:01
```

<details>
<summary>提示</summary>

`*h = total / 3600;`，`*m = total % 3600 / 60;`，`*s = total % 60;`。輸出用 `printf("%d:%02d:%02d\n", h, m, s);`。
</details>

<!-- exercise: qsort-desc -->
### 練習 3：從大到小

輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數，用 `qsort` 把它們**從大到小**排序後輸出，用空格隔開。

```text
樣例輸入：
5
3 -1 4 1 5
樣例輸出：5 4 3 1 -1
```

<details>
<summary>提示</summary>

寫一個比較函式，在 `x > y` 時返回負數（讓大的排在前面）：`return (x < y) - (x > y);`。
</details>

<!-- exercise: count-substr -->
### 練習 4：子串出現了幾次

第一行輸入一段英文，第二行輸入一個單詞。輸出這個單詞在這段文字裡出現了幾次（**允許重疊**，區分大小寫）。

```text
樣例輸入：
banana
ana
樣例輸出：2
```

<details>
<summary>提示</summary>

用 `strstr` 反覆查詢：找到一次就計數，然後從找到的位置的**下一個字元**開始繼續找：`p = strstr(p + 1, word);`，直到返回 `NULL`。兩行都用 `fgets` 讀，並去掉 `'\n'`。
</details>

<!-- exercise: reverse-ptr -->
### 練習 5：用兩個指標反轉字串

寫函式 `void reverse(char *s)`，用兩個指標（一個從頭、一個從尾往中間走）把字串原地反轉。輸入一個英文單詞，輸出反轉後的結果。

```text
樣例輸入：pointer
樣例輸出：retniop
```

<details>
<summary>提示</summary>

`char *left = s;`、`char *right = s + strlen(s) - 1;`，當 `left < right` 時交換 `*left` 和 `*right`，然後 `left++`、`right--`。注意空字串的情況。
</details>

<!-- exercise: ptr-predict -->
### 練習 6：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    int a[] = {1, 2, 3, 4};
    int *p = a + 1;
    *p = 20;
    p++;
    *p += 10;
    printf("%d %d %d %d\n", a[0], a[1], a[2], a[3]);
    printf("%td\n", p - a);

    int x = 5;
    int *q = &x;
    *q = *q * 2;
    printf("%d\n", x);
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`p` 一開始指向 `a[1]`，把它改成 20；`p++` 後指向 `a[2]`，加 10 變成 13。`p - a` 是 2。`q` 指向 `x`，`*q = *q * 2` 就是 `x = x * 2`。輸出：

```text
1 20 13 4
2
10
```
</details>

<!-- exercise: ptr-value -->
### 練習 7：哪個是 3？

```c norun
int x = 3;
int *p = &x;
```

下面哪個表示式的值是 `3`？

- `p`
- `*p`
- `&p`
- `&x`

<details>
<summary>答案與解析</summary>

`*p` 是 `p` 指向的變數，也就是 `x`，值為 3。`p` 和 `&x` 都是 `x` 的地址，`&p` 是指標變數 `p` 自己的地址。
</details>

## 10.12 延伸閱讀

- [cppreference：指標宣告](https://zh.cppreference.com/w/c/language/pointer)、[qsort](https://zh.cppreference.com/w/c/algorithm/qsort)、[strtol](https://zh.cppreference.com/w/c/string/byte/strtol)
- [Python Tutor（選 C 語言）](https://pythontutor.com/c.html)——把本課的程式碼貼進去，一步一步地看指標指向哪裡
- 下一課：[動態記憶體](../11-memory/zh-CN.md)
