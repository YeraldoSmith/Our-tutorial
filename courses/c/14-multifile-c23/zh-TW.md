# 14. 多檔案程式、前處理器和 C23

> 🎯 **本課目標**：把程式拆分成多個 `.c` 原始檔和 `.h` 標頭檔案，理解「宣告放標頭檔案、定義放原始檔」；寫標頭檔案保護；用 `static` 隱藏模組內部的函式和變數，用 `extern` 共享全域性變數；瞭解編譯的幾個階段，看懂連結錯誤；用好前處理器（帶引數的宏和它的陷阱、條件編譯、`assert`）；瞭解 Makefile 和 CMake；系統地認識 **C23** 的新特性。最後看看學完 C 之後可以往哪裡走。
>
> ⏱ **預計用時**：100 分鐘　｜　📋 **前置知識**：本課程第 1～13 課

## 14.1 為什麼要拆成多個檔案

到目前為止，每個程式都只有一個檔案。真實的專案動輒幾萬、幾百萬行，全寫在一個檔案裡，沒人能看得過來，改一行也要把整個程式重新編譯。所以要把程式按功能拆成**模組**：每個模組一個 `.c` 檔案（實現）加一個 `.h` 檔案（介面）。

下面是一個三個檔案的小專案。學習程式一次只能執行一個檔案，所以這幾段程式碼請在自己電腦上新建資料夾來試（編譯命令在 14.3 節）。

```c norun
// geometry.h —— 模組的「介面」：告訴別人這個模組提供什麼
#ifndef GEOMETRY_H
#define GEOMETRY_H

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b);
double triangle_area(Point a, Point b, Point c);

#endif
```

```c norun
// geometry.c —— 模組的「實現」
#include <math.h>
#include "geometry.h"

static double cross(Point o, Point a, Point b) {    // static：只在本檔案內使用
    return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
}

double distance(Point a, Point b) {
    return hypot(a.x - b.x, a.y - b.y);
}

double triangle_area(Point a, Point b, Point c) {
    return fabs(cross(a, b, c)) / 2;
}
```

```c norun
// main.c —— 使用這個模組
#include <stdio.h>
#include "geometry.h"

int main(void) {
    Point a = {0, 0}, b = {4, 0}, c = {0, 3};
    printf("AB = %.1f\n", distance(a, b));
    printf("面積 = %.1f\n", triangle_area(a, b, c));
    return 0;
}
```

- **標頭檔案（`.h`）只放宣告**：函式原型、型別定義（`struct`、`typedef`、`enum`）、宏、`extern` 變數宣告。誰要用這個模組，就 `#include` 它的標頭檔案。
- **原始檔（`.c`）放定義**：函式體、全域性變數的定義。原始檔也要包含自己的標頭檔案，這樣編譯器能檢查宣告和定義是否一致。
- `#include "geometry.h"` 用**雙引號**：先在當前資料夾裡找；`#include <stdio.h>` 用**尖括號**：只在系統的標頭檔案目錄裡找。

## 14.2 標頭檔案保護、`static` 和 `extern`

### 標頭檔案保護

`#include` 其實就是把標頭檔案的內容**原樣複製**到這裡。如果一個 `.c` 檔案間接地把 `geometry.h` 包含了兩次，`Point` 就會被定義兩次，編譯器報錯 `redefinition of ...`。上面標頭檔案開頭和結尾的三行就是**標頭檔案保護**（include guard）：

```c norun
#ifndef GEOMETRY_H      // 如果還沒定義過 GEOMETRY_H……
#define GEOMETRY_H      // ……就定義它，並處理下面的內容
/* 標頭檔案的內容 */
#endif                  // 第二次包含時，GEOMETRY_H 已經定義過，整段被跳過
```

每個標頭檔案都要寫。宏名一般用檔名的大寫形式。很多編譯器也支援更短的 `#pragma once`，效果一樣（但它不是 C 標準的一部分）。

### `static`：模組的私有部分

在函式外面，`static` 的意思是「**只在本檔案裡可見**」。上面的 `cross` 是 `geometry.c` 內部的輔助函式，加上 `static`，別的檔案就看不到它，也不會和別的檔案裡同名的函式衝突。這和面嚮物件語言裡的「私有」很像：標頭檔案裡公開的是介面，`static` 的是實現細節。

（注意：函式**裡面**的 `static` 區域性變數是另一個意思——值在多次呼叫之間保留，第 7 課講過。）

### `extern`：共享全域性變數

如果確實需要一個被多個檔案共用的全域性變數，在**標頭檔案**裡用 `extern` 宣告它，在**某一個**原始檔裡定義它：

```c norun
// config.h
extern int verbose;           // 宣告：這個變數在別處定義

// config.c
#include "config.h"
int verbose = 0;              // 定義：只能有一處
```

如果把 `int verbose = 0;` 直接寫進標頭檔案，每個包含它的 `.c` 檔案都會有一份定義，連結時報錯 `multiple definition of 'verbose'`。不過還是那句話：全域性變數能少用就少用。

## 14.3 編譯、連結和連結錯誤

在終端裡編譯多檔案程式，把所有 `.c` 檔案都列出來：

```bash
gcc -std=c23 -Wall -Wextra -g main.c geometry.c -o app -lm
./app
```

（GCC 14 以前的版本用 `-std=c2x`。`-lm` 連結數學庫，`hypot`、`fabs` 在裡面。）這一條命令其實分好幾步完成：

```text
main.c ──預處理──▶ 展開 #include、宏 ──編譯──▶ main.o ─┐
                                                      ├──連結──▶ app（可執行檔案）
geometry.c ──預處理──▶ ……         ──編譯──▶ geometry.o ─┘   ↑ 還會連結標準庫
```

1. **預處理**：處理所有 `#` 開頭的指令——把標頭檔案內容貼進來、展開宏、處理條件編譯。
2. **編譯**：把每個 `.c` 檔案**單獨**翻譯成機器碼，得到目標檔案（`.o`）。編譯 `main.c` 時，編譯器只知道 `distance` 的原型，不知道它的實現。
3. **連結**：連結器把所有 `.o` 檔案和標準庫拼在一起，把每個函式呼叫和它的定義對上號。

所以有兩類不同的錯誤：**編譯錯誤**（語法、型別問題，會指出檔案和行號）和**連結錯誤**（編譯都透過了，但拼裝時找不到或找到多個定義）：

| 連結錯誤 | 原因 |
|----------|------|
| `undefined reference to 'distance'` | 編譯命令裡漏了 `geometry.c`；或者函式名拼錯了；或者只寫了原型沒寫函式體 |
| `multiple definition of 'verbose'` | 變數或函式的**定義**寫進了標頭檔案，被多個 `.c` 檔案包含 |
| `undefined reference to 'sqrt'`（某些系統上） | 忘了加 `-lm` |

分開編譯的好處是：只改了 `geometry.c`，就只需要重新編譯它，再重新連結——大專案能節省大量時間。下面的構建工具就是幫你自動做這件事的。

## 14.4 構建工具：Make 和 CMake

每次手敲一長串編譯命令很麻煩。**Make** 根據一個叫 `Makefile` 的檔案，只重新編譯改過的部分：

```makefile
CC = gcc
CFLAGS = -std=c2x -Wall -Wextra -g

app: main.o geometry.o
	$(CC) main.o geometry.o -o app -lm

main.o: main.c geometry.h
	$(CC) $(CFLAGS) -c main.c

geometry.o: geometry.c geometry.h
	$(CC) $(CFLAGS) -c geometry.c
```

在終端裡輸入 `make` 就會構建 `app`。每條規則是「目標: 依賴」，下一行（**必須用 Tab 縮排**）是生成目標的命令；依賴的檔案比目標新時，才會重新執行。

更大的專案一般用 **CMake**，它能為不同的系統和編譯器（包括 Windows 上的 Visual Studio）生成構建檔案：

```cmake
cmake_minimum_required(VERSION 3.20)
project(geometry C)
set(CMAKE_C_STANDARD 23)
add_executable(app main.c geometry.c)
target_link_libraries(app m)
```

然後執行 `cmake -B build` 和 `cmake --build build`。（在 Windows 上去掉最後一行，那裡的數學函式不需要單獨連結。）

## 14.5 前處理器

### 帶引數的宏和它的陷阱

`#define` 不僅能定義常量，還能定義「帶引數的宏」。但宏只是**文字替換**，不是函式，很容易出錯：

```c
#include <stdio.h>

#define SQUARE_BAD(x) x * x
#define SQUARE(x) ((x) * (x))           // 每個引數、整個結果都加上括號
#define MAX(a, b) ((a) > (b) ? (a) : (b))

static inline int square(int x) {       // 更好的選擇：inline 函式
    return x * x;
}

int main(void) {
    printf("%d\n", SQUARE_BAD(1 + 2));  // 變成 1 + 2 * 1 + 2，結果是 5！
    printf("%d\n", SQUARE(1 + 2));      // ((1 + 2) * (1 + 2)) = 9
    printf("%d\n", MAX(3, 7));          // 7
    printf("%d\n", square(1 + 2));      // 9：函式先算出引數，再計算
    return 0;
}
```

- 宏的每個引數、整個表示式都要**加括號**。
- 引數可能被計算兩次：`MAX(i++, j)` 會讓 `i` 加兩次。
- 所以能用函式就用函式，`static inline` 函式一樣快，而且有型別檢查；能用 `const` 或 `constexpr` 定義常量，就別用 `#define`。

### 條件編譯和除錯資訊

```c
#include <stdio.h>
#include <assert.h>

#define DEBUG 1

int average(const int a[], int n) {
    assert(n > 0);                       // 斷言：n 必須大於 0，否則程式立刻停下並報告
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
#if DEBUG
    printf("[除錯] %s 第 %d 行：sum = %ld\n", __func__, __LINE__, sum);
#endif
    return (int)(sum / n);
}

int main(void) {
    int data[] = {3, 5, 10};
    printf("平均：%d\n", average(data, 3));
    return 0;
}
```

- `#if` / `#ifdef` / `#else` / `#endif`：根據條件決定**編譯哪些程式碼**。常用於除錯程式碼、針對不同作業系統的程式碼（`#ifdef _WIN32`）。
- `__FILE__`、`__LINE__`、`__func__`：當前的檔名、行號和函式名，適合輸出除錯資訊。
- **`assert(條件)`**（`<assert.h>`）：條件為假時，列印出錯的檔案、行號和條件，然後中止程式。它用來檢查「絕不應該發生」的情況，是很好的自我檢查工具。正式釋出時用 `-DNDEBUG` 編譯，所有 `assert` 都會被去掉。把 `average(data, 3)` 改成 `average(data, 0)` 試試看。

## 14.6 C23：現代的 C

C 語言的標準大約每隔幾年更新一次：C89、C99、C11、C17，以及最新的 **C23**。本課程從第 1 課起就在用 C23（編譯選項 `-std=c2x`，較新的編譯器也可以寫 `-std=c23`）。把它的主要新特性彙總一下：

```c
#include <stdio.h>

constexpr int MAX_PLAYERS = 4;                   // 編譯時常量，可以用作陣列長度
enum Color : unsigned char { RED, GREEN, BLUE }; // 指定列舉的底層型別

[[nodiscard]] int checked_half(int x, bool *ok) {   // [[nodiscard]]：返回值不應被忽略
    *ok = (x % 2 == 0);
    return x / 2;
}

int main(void) {
    int scores[MAX_PLAYERS] = {};            // {}：空初始化，全部為 0
    auto total = 0;                          // auto：根據初始值推斷型別（這裡是 int）
    int *p = nullptr;                        // nullptr：真正的空指標常量
    bool ok = false;                         // bool、true、false 是關鍵字了
    int million = 1'000'000;                 // 數字分隔符
    int flags = 0b1010'0101;                 // 二進位制字面量
    typeof(million) copy = million;          // typeof：取一個表示式的型別
    static_assert(sizeof(int) >= 4);         // 編譯時斷言（C23 可以省略訊息）
    enum Color c = BLUE;

    int half = checked_half(10, &ok);
    printf("%d %d %d %d\n", scores[0], total, half, ok);
    printf("%d %d %d\n", million, flags, copy);
    printf("p %s空指標，c 佔 %zu 位元組\n", p == nullptr ? "是" : "不是", sizeof c);
    return 0;
}
```

| 特性 | 說明 |
|------|------|
| `bool`、`true`、`false` | 成為關鍵字，不再需要 `<stdbool.h>` |
| `nullptr` | 空指標常量，比 `NULL` 更安全（`NULL` 可能只是整數 0） |
| `constexpr` | 編譯時常量，比 `#define` 安全，比 `const` 用途更廣 |
| `auto`、`typeof` | 讓編譯器推斷型別、取得表示式的型別 |
| `{}` 空初始化 | `int a[10] = {};` 全部清零 |
| `0b1010`、`1'000'000` | 二進位制字面量、數字分隔符 |
| `[[nodiscard]]`、`[[maybe_unused]]`、`[[deprecated]]` | 標準的「屬性」，幫編譯器發現問題 |
| `enum E : 型別` | 指定列舉的底層整數型別 |
| `static_assert(條件)` | 編譯時檢查，訊息可以省略 |
| `<stdckdint.h>` | `ckd_add(&r, a, b)` 等：帶溢位檢查的整數運算 |
| `<stdbit.h>` | `stdc_count_ones` 等位操作函式 |
| `#embed "file.bin"` | 把一個檔案的內容直接嵌入陣列 |
| `strdup`、`memccpy` | 進入了標準庫 |

（後面三行比較新：`<stdckdint.h>` 和 `<stdbit.h>` 需要 GCC 14 以上，`#embed` 需要 GCC 15 或 Clang 19 以上。如果你的編譯器報錯說不認識 `nullptr`、`bool` 之類的寫法，說明它太舊了，請升級到 GCC 13+ 或 Clang 16+。）

## 14.7 學完 C 之後

恭喜你學完了 C 語言課程！你已經掌握了變數、控制流、函式、陣列、字串、指標、動態記憶體、結構體、檔案和多檔案程式——這正是理解計算機如何工作的基礎。接下來可以：

- **做專案**：命令列記賬本、通訊錄、文字冒險遊戲、簡單的 shell、迷宮求解器、圖片格式轉換器（BMP 格式很簡單）……做專案是鞏固知識最好的辦法。
- **學資料結構和演算法**：堆疊、佇列、雜湊表、二叉樹、圖、排序和搜尋——用 C 親手實現一遍，理解會深刻得多。
- **學本教程的其他課程**：C++ 在 C 的基礎上加了類、模板和豐富的標準庫；Rust 用編譯期檢查解決了 C 的記憶體安全問題；彙編課會帶你看到 C 程式碼最終變成的機器指令。
- **讀好書**：Jens Gustedt 的《Modern C》（作者是 C 標準委員會成員，內容覆蓋 C23，有免費的英文電子版）；Robert C. Seacord 的《Effective C》；經典的《C 程式設計語言》（K&R）。

## 14.8 小結

- 模組 = `.h`（宣告：原型、型別、宏、`extern`）+ `.c`（定義）；原始檔包含自己的標頭檔案。
- 每個標頭檔案都寫標頭檔案保護（`#ifndef X_H` / `#define X_H` / `#endif`，或 `#pragma once`）。
- 檔案作用域的 `static` 讓函式和變數只在本檔案可見；共享全域性變數：標頭檔案 `extern` 宣告，一個原始檔定義。
- 編譯分預處理、編譯、連結；`undefined reference` 和 `multiple definition` 是連結錯誤。
- 多檔案編譯：`gcc main.c geometry.c -o app`；用 Make 或 CMake 自動構建。
- 宏是文字替換：引數和結果都加括號，能用函式和 `constexpr` 就別用宏；`#if` 條件編譯，`assert` 檢查不應發生的情況。
- C23：`bool`、`nullptr`、`constexpr`、`auto`、`typeof`、`{}`、`0b`、`'` 分隔符、屬性、`static_assert` 等。

## 14.9 練習

<!-- exercise: stack-module -->
### 練習 1：堆疊模組

學習程式只能執行一個檔案，所以這道題把一個「模組」的三部分寫在同一個檔案裡：介面（原型）、實現、使用。請補全實現部分的四個函式，用 `static` 的陣列 `items` 和計數器 `top` 實現一個**堆疊**（後放進去的先拿出來）：`stack_push` 放入一個數（堆疊滿時忽略）；`stack_pop` 取出並返回最後放入的數，堆疊空時返回 -1；`stack_empty` 判斷是否為空；`stack_size` 返回元素個數。`main` 已經寫好，它讀入 `push x`、`pop`、`size` 這樣的命令。

```text
樣例輸入：push 3 push 5 pop size pop pop
樣例輸出：
5
1
3
-1
```

<details>
<summary>提示</summary>

`push`：`if (top < STACK_MAX) { items[top] = value; top++; }`。`pop`：堆疊空返回 -1，否則 `top--; return items[top];`。`top` 本身就是元素個數。
</details>

<!-- exercise: find-negative -->
### 練習 2：用 `nullptr` 表示「沒找到」

寫函式 `int *find_negative(int a[], int n)`：返回陣列裡第一個負數的地址，沒有負數時返回 `nullptr`。輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數，如果找到了，輸出它的下標和值（下標用指標相減求出），用空格隔開；否則輸出 `none`。

```text
樣例輸入：
5
4 0 -7 2 -1
樣例輸出：2 -7
```

<details>
<summary>提示</summary>

找到後：`int *p = find_negative(a, n); if (p != nullptr) printf("%td %d\n", p - a, *p);`。
</details>

<!-- exercise: to-binary -->
### 練習 3：二進位制

輸入一個整數 `x`（0 ≤ x < 2³¹），輸出它的二進位制表示（沒有前導 0；`x` 為 0 時輸出 `0`）和其中 1 的個數，用空格隔開。

```text
樣例輸入：10
樣例輸出：1010 2
```

<details>
<summary>提示</summary>

從最高位往最低位檢查：`for (int bit = 30; bit >= 0; bit--)`，用 `(x >> bit) & 1` 取出這一位；遇到第一個 1 之前的 0 不輸出。也可以先用 `x % 2`、`x / 2` 從低位往高位取，存進陣列後倒著輸出。
</details>

<!-- exercise: macro-predict -->
### 練習 4：讀程式碼，寫輸出

```c
#include <stdio.h>

#define DOUBLE_BAD(x) x + x
#define DOUBLE(x) ((x) + (x))
#define SQUARE_BAD(x) x * x

int main(void) {
    printf("%d\n", 3 * DOUBLE_BAD(2));
    printf("%d\n", 3 * DOUBLE(2));
    printf("%d\n", SQUARE_BAD(1 + 2));
    return 0;
}
```

<details>
<summary>答案與解析</summary>

宏只是文字替換。`3 * DOUBLE_BAD(2)` 變成 `3 * 2 + 2`，等於 8；`3 * DOUBLE(2)` 變成 `3 * ((2) + (2))`，等於 12；`SQUARE_BAD(1 + 2)` 變成 `1 + 2 * 1 + 2`，等於 5。輸出：

```text
8
12
5
```
</details>

<!-- exercise: link-error -->
### 練習 5：連結錯誤

`area` 函式定義在 `geometry.c` 裡，`main.c` 包含了 `geometry.h` 並呼叫 `area`。用 `gcc main.c -o app` 編譯時，報錯 `undefined reference to 'area'`。原因是什麼？

- `geometry.h` 沒有寫標頭檔案保護
- 編譯命令裡漏掉了 `geometry.c`
- `main.c` 沒有 `#include <stdio.h>`
- `area` 函式應該寫成 `static`

<details>
<summary>答案與解析</summary>

`undefined reference` 是連結錯誤：`main.c` 編譯透過了，但連結時找不到 `area` 的定義，因為 `geometry.c` 根本沒參與編譯。應該寫 `gcc main.c geometry.c -o app`。如果把 `area` 寫成 `static`，它只在 `geometry.c` 裡可見，反而會出同樣的錯。
</details>

<!-- exercise: header-guard -->
### 練習 6：標頭檔案保護

標頭檔案開頭的 `#ifndef GEOMETRY_H` / `#define GEOMETRY_H` 和結尾的 `#endif` 有什麼用？

- 讓標頭檔案編譯得更快
- 防止同一個標頭檔案在一個原始檔裡被包含多次，導致型別等被重複定義
- 允許在標頭檔案裡寫函式的定義
- 禁止其他檔案包含這個標頭檔案

<details>
<summary>答案與解析</summary>

第一次包含時 `GEOMETRY_H` 還沒定義，內容被處理，同時定義了 `GEOMETRY_H`；之後再被包含時，`#ifndef` 條件為假，整段內容被跳過，就不會出現 `redefinition` 錯誤。
</details>

## 14.10 延伸閱讀

- [cppreference：前處理器](https://zh.cppreference.com/w/c/preprocessor)、[C23 新特性一覽（英文）](https://en.cppreference.com/w/c/23)
- [Modern C（Jens Gustedt，英文，免費）](https://gustedt.gitlabpages.inria.fr/modern-c/)
- [CMake 官方教程（英文）](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)
- 返回：[C 語言課程目錄](../)
