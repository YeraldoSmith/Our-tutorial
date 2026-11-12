# 2. 變數與型別

> 🎯 **本課目標**：學會在 C 裡**宣告**變數（必須寫明型別）；認識 `int`、`long long`、`double`、`char`、`bool` 等基本型別和它們佔多少位元組；用 `printf` 的**格式說明符**（`%d`、`%f`、`%c`……）輸出各種型別，並控制寬度和小數位數；明白「字元就是小整數」；會用 `const` 定義常量；知道「未初始化的變數裡是垃圾值」；認識 `<stdint.h>` 裡固定大小的整數型別。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：本課程第 1 課

## 2.1 宣告變數：必須先說清楚型別

在 Python 裡，`age = 12` 就建立了一個變數。在 C 裡，變數**必須先宣告**，並且要寫明它的**型別**：

```c
#include <stdio.h>

int main(void) {
    int age = 12;              // 宣告一個整數變數 age，初始值是 12
    double height = 1.52;      // 小數（雙精度浮點數）
    char grade = 'A';          // 一個字元，用單引號

    printf("年齡：%d\n", age);
    printf("身高：%f 米\n", height);
    printf("等級：%c\n", grade);

    age = age + 1;             // 賦值：改變變數的值（型別不能變）
    printf("明年：%d 歲\n", age);
    return 0;
}
```

為什麼要寫型別？因為 C 要在**編譯時**就知道每個變數需要多少記憶體、該用什麼指令來計算。變數一旦宣告，型別就**不能再改變**——不能先存整數、後存字串（這叫**靜態型別**）。

可以一次宣告多個同型別的變數：`int a = 1, b = 2, c;`。

## 2.2 基本型別

| 型別 | 含義 | 典型大小 | 例子 |
|------|------|----------|------|
| `int` | 整數 | 4 位元組，約 ±21 億 | `int count = 42;` |
| `long long` | 大整數 | 8 位元組，約 ±922 億億 | `long long big = 9000000000LL;` |
| `double` | 小數（雙精度浮點數） | 8 位元組，約 15～16 位有效數字 | `double pi = 3.14159;` |
| `float` | 小數（單精度） | 4 位元組，約 6～7 位有效數字 | `float f = 2.5f;` |
| `char` | 一個字元（其實是 1 位元組的小整數） | 1 位元組 | `char c = 'x';` |
| `bool` | 真 / 假 | 1 位元組 | `bool ok = true;` |

> [!NOTE]
> `bool`、`true`、`false` 在 **C23** 標準裡是關鍵字，直接就能用。舊的標準（C99～C17）需要先 `#include <stdbool.h>`。本課程使用 C23（編譯選項 `-std=c2x`），舊編譯器不支援時加上那一行即可。

用 `sizeof` 可以檢視一個型別或變數佔多少位元組：

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    printf("char:      %zu 位元組\n", sizeof(char));
    printf("int:       %zu 位元組，範圍 %d ~ %d\n", sizeof(int), INT_MIN, INT_MAX);
    printf("long long: %zu 位元組，最大 %lld\n", sizeof(long long), LLONG_MAX);
    printf("float:     %zu 位元組，約 %d 位有效數字\n", sizeof(float), FLT_DIG);
    printf("double:    %zu 位元組，約 %d 位有效數字\n", sizeof(double), DBL_DIG);
    printf("bool:      %zu 位元組\n", sizeof(bool));
    return 0;
}
```

`<limits.h>` 和 `<float.h>` 裡定義了各種型別的極限值。`sizeof` 的結果要用 `%zu` 輸出。

> [!TIP]
> **用哪個型別？** 整數一般用 `int`；可能超過 21 億的（比如以「分」為單位的大筆金額、時間戳）用 `long long`；小數一般用 `double`（`float` 精度太低，除非是為了省記憶體）。

## 2.3 printf 的格式說明符

`printf` 的第一個引數是**格式字串**，裡面的 `%d`、`%f` 等是「佔位符」，會被後面的引數依次替換：

| 說明符 | 用於 | 例子 → 輸出 |
|--------|------|-------------|
| `%d` | `int` | `printf("%d", 42)` → `42` |
| `%lld` | `long long` | `printf("%lld", 9000000000LL)` → `9000000000` |
| `%f` | `double`（預設 6 位小數） | `printf("%f", 3.14)` → `3.140000` |
| `%.2f` | 保留 2 位小數 | `printf("%.2f", 3.14159)` → `3.14` |
| `%e` | 科學計數法 | `printf("%e", 12345.678)` → `1.234568e+04` |
| `%c` | `char` | `printf("%c", 'A')` → `A` |
| `%s` | 字串 | `printf("%s", "hi")` → `hi` |
| `%x` | 十六進位制整數 | `printf("%x", 255)` → `ff` |
| `%zu` | `sizeof` 的結果 | `printf("%zu", sizeof(int))` → `4` |
| `%%` | 百分號本身 | |

還可以控制**寬度**和**對齊**，非常適合輸出整齊的表格：

```c
#include <stdio.h>

int main(void) {
    printf("[%5d]\n", 42);       // 寬度 5，右對齊：[   42]
    printf("[%-5d]\n", 42);      // 左對齊：       [42   ]
    printf("[%05d]\n", 42);      // 用 0 補齊：    [00042]
    printf("[%8.3f]\n", 3.14159);// 寬度 8，3 位小數：[   3.142]
    printf("[%-6s|%6s]\n", "ab", "cd");

    printf("%-8s%6s%8s\n", "item", "qty", "price");
    printf("%-8s%6d%8.2f\n", "apple", 3, 4.5);
    printf("%-8s%6d%8.2f\n", "banana", 12, 0.75);
    return 0;
}
```

> [!NOTE]
> `printf` 的寬度按**位元組**計算，而一個漢字在 UTF-8 裡佔 3 個位元組、在螢幕上佔 2 個字元寬，所以含中文的列很難對齊。需要對齊的表格，表頭先用英文，或者自己計算要補多少空格。

> [!WARNING]
> **說明符必須和型別匹配**。`printf("%d", 3.14)` 把一個 `double` 當成 `int` 來讀，結果是一個莫名其妙的數（這是「未定義行為」）。好在加了 `-Wall` 的編譯器會警告：`format '%d' expects argument of type 'int', but argument 2 has type 'double'`。**看到警告一定要改。**

## 2.4 字元就是小整數

計算機裡沒有「字母」，只有數字。`char` 型別存的其實是一個 0～127（或 -128～127）的整數，每個數字按照 **ASCII 表**對應一個字元：`'A'` 是 65，`'a'` 是 97，`'0'` 是 48。

```c
#include <stdio.h>

int main(void) {
    char c = 'A';
    printf("%c 的編碼是 %d\n", c, c);       // 同一個值，用 %c 顯示成字元，用 %d 顯示成數字
    printf("%c\n", c + 2);                   // 'A' + 2 = 67 = 'C'
    printf("%c\n", 'a' - 'A' + 'G');         // 大寫變小寫：g
    char digit = '7';
    printf("字元 '7' 對應的數字：%d\n", digit - '0');   // '7' - '0' = 55 - 48 = 7
    return 0;
}
```

> [!NOTE]
> 一個 `char` 只能存一個位元組，所以只能放英文字母、數字和常見符號。漢字、俄文字母在 UTF-8 裡要佔 2～4 個位元組，不能放進一個 `char`。中文等文字要用**字串**（第 9 課）來存。

## 2.5 常量

值不會改變的量叫**常量**。有幾種寫法：

```c
#include <stdio.h>

#define MAX_STUDENTS 40              // 前處理器宏：編譯前把 MAX_STUDENTS 全部替換成 40

int main(void) {
    const double PI = 3.14159265;    // const：這個變數之後不能再修改
    double r = 2.0;
    printf("面積：%.2f\n", PI * r * r);
    printf("最多 %d 人\n", MAX_STUDENTS);
    // PI = 3;                       // 取消註釋會報錯：assignment of read-only variable 'PI'
    return 0;
}
```

- `const`：普通變數，但只讀。現代 C 更推薦這種寫法，因為它有型別，編譯器能檢查。
- `#define`：**前處理器**做的純文字替換（第 1 課的「預處理」那一步），沒有型別，名字習慣全部大寫。
- C23 還新增了 `constexpr`，表示「編譯時就能算出來的常量」，比如 `constexpr int SIZE = 10;`。

**字面量**的寫法也有講究：`10` 是 `int`，`10LL` 是 `long long`，`3.14` 是 `double`，`3.14f` 是 `float`，`'x'` 是字元，`"x"` 是字串（兩者不一樣！）。整數還可以寫成十六進位制 `0xFF`、二進位制 `0b1010`（C23）；長數字可以用單引號分隔：`1'000'000`（C23）。

## 2.6 變數命名規則

- 只能包含字母、數字和下劃線，**不能以數字開頭**：`score2` ✓，`2score` ✗；
- 區分大小寫：`count` 和 `Count` 是兩個不同的變數；
- 不能使用關鍵字：`int`、`return`、`if`、`while`……；
- 習慣：變數和函式用小寫加下劃線（`total_score`），宏常量全部大寫（`MAX_SIZE`）。

## 2.7 未初始化的變數：垃圾值

在 Python 裡，沒賦值的變數根本不能用。C 允許你宣告一個變數而**不給初始值**，但這時它裡面是**之前留在那塊記憶體裡的隨便什麼數**——叫做「垃圾值」：

```c norun
#include <stdio.h>

int main(void) {
    int x;                  // 沒有初始值
    printf("%d\n", x);      // 可能是 0，也可能是 32765，每次執行都可能不同
    return 0;
}
```

```text
warning: 'x' is used uninitialized [-Wuninitialized]
```

這是 C 程式裡最常見的 bug 之一。**養成習慣：宣告變數時總是給一個初始值。**

## 2.8 溢位：整數也有盡頭

每種整數型別都有範圍。超出範圍會發生什麼？

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    unsigned char small = 255;          // unsigned：無符號，只能存 0～255
    small = small + 1;
    printf("255 + 1 = %d\n", small);    // 0：無符號整數溢位會「繞回」0

    long long big = INT_MAX;            // 用更大的型別就不會溢位
    big = big + 1;
    printf("INT_MAX + 1 = %lld\n", big);
    return 0;
}
```

- **無符號**整數（`unsigned`）溢位時會像汽車里程錶一樣「繞回」，這是規定好的行為；
- **有符號**整數（普通的 `int`）溢位是**未定義行為**（undefined behavior）：C 標準不規定結果，程式可能得到奇怪的數，也可能出現更離奇的錯誤。**要避免它**：估計一下資料會有多大，必要時用 `long long`。

1996 年，歐洲的阿麗亞娜 5 號火箭發射 40 秒後爆炸，原因就是把一個 64 位浮點數轉換成 16 位整數時發生了溢位。

## 2.9 固定大小的整數：`<stdint.h>`

`int` 的大小在不同的平臺上可能不同（絕大多數電腦上是 4 位元組，但一些微控制器上只有 2 位元組）。需要**確切大小**時（比如處理檔案格式、網路資料、硬體暫存器），使用 `<stdint.h>`：

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    int8_t   a = -100;          // 正好 8 位（1 位元組）有符號
    uint8_t  b = 200;           // 正好 8 位無符號：0～255
    int32_t  c = 2000000000;    // 正好 32 位
    uint64_t d = 18000000000000000000u;   // 正好 64 位無符號
    printf("%d %d %" PRId32 " %" PRIu64 "\n", a, b, c, d);
    printf("%zu %zu %zu %zu\n", sizeof a, sizeof b, sizeof c, sizeof d);
    return 0;
}
```

`PRId32`、`PRIu64` 是 `<inttypes.h>` 提供的格式說明符，保證在任何平臺上都正確。

## 2.10 小結

- C 的變數必須**先宣告型別**：`int`、`long long`、`double`、`float`、`char`、`bool`；型別宣告後不能改變。
- `sizeof` 檢視位元組數（用 `%zu` 輸出）；`<limits.h>`、`<float.h>` 裡有範圍。
- `printf` 格式說明符：`%d`、`%lld`、`%f`、`%.2f`、`%c`、`%s`、`%x`、`%zu`；`%5d`、`%-5d`、`%05d` 控制寬度和對齊；**說明符必須和型別匹配**。
- `char` 就是小整數（ASCII）：`'A'` 是 65，`'7' - '0'` 是 7。
- 常量：`const`（推薦）、`#define`、C23 的 `constexpr`。
- **宣告時就初始化**，否則是垃圾值；有符號整數溢位是未定義行為；需要確切大小時用 `<stdint.h>`。

## 2.11 練習

<!-- exercise: profile -->
### 練習 1：個人資訊

在程式裡宣告三個變數：`int age = 12;`、`double height = 1.523;`、`char grade = 'B';`，然後**用格式說明符**輸出（身高保留 2 位小數）：

```text
樣例輸出：Age: 12, Height: 1.52 m, Grade: B
```

<details>
<summary>提示</summary>

`printf("Age: %d, Height: %.2f m, Grade: %c\n", age, height, grade);`
</details>

<!-- exercise: sizes -->
### 練習 2：型別的大小

用 `sizeof` 輸出 `char`、`int`、`double`、`long long` 四種型別的位元組數，用空格隔開。

```text
樣例輸出：1 4 8 8
```

<details>
<summary>提示</summary>

`printf("%zu %zu %zu %zu\n", sizeof(char), sizeof(int), sizeof(double), sizeof(long long));`
</details>

<!-- exercise: ascii -->
### 練習 3：字元和編碼

輸出兩樣東西，用空格隔開：字元 `'Z'` 的 ASCII 編碼，以及編碼 `97` 對應的字元。

```text
樣例輸出：90 a
```

<details>
<summary>提示</summary>

同一個值用 `%d` 輸出是數字，用 `%c` 輸出是字元：`printf("%d %c\n", 'Z', 97);`
</details>

<!-- exercise: table -->
### 練習 4：對齊的表格

用寬度控制輸出下面的表格（名字左對齊佔 6 格，數量右對齊佔 4 格，價格右對齊佔 8 格、保留 2 位小數）：

```text
apple    3    4.50
kiwi    12    0.75
```

<details>
<summary>提示</summary>

`printf("%-6s%4d%8.2f\n", "apple", 3, 4.5);`
</details>

<!-- exercise: format-predict -->
### 練習 5：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    int a = 7;
    double b = 2.5;
    printf("%d %.1f|%5.2f|%-4d|\n", a, b, b * 2, a);
    printf("%c%c\n", 'a' + 1, 'A' + 25);
    return 0;
}
```

<details>
<summary>答案與解析</summary>

第一行：`%d` 輸出 `7`；`%.1f` 輸出 `2.5`；`%5.2f` 輸出的 `5.00` 只有 4 個字元，寬度是 5，所以前面補 1 個空格；`%-4d` 讓 `7` 左對齊佔 4 格，後面補 3 個空格。所以是 `7 2.5| 5.00|7   |`。第二行：`'a' + 1` 是 `b`，`'A' + 25` 是 `Z`，輸出 `bZ`。
</details>

<!-- exercise: uninit -->
### 練習 6：沒有初始值的變數

`int x;` 之後直接 `printf("%d", x);`，會輸出什麼？

- 一定是 0
- 一個不可預測的「垃圾值」，每次可能不同
- 編譯器會拒絕編譯
- 程式一定會崩潰

<details>
<summary>答案與解析</summary>

C 不會自動初始化區域性變數，裡面是那塊記憶體原來留下的任意值。編譯器（加了 `-Wall`）通常只給出警告。所以宣告時一定要初始化。
</details>

## 2.12 延伸閱讀

- [cppreference：算術型別](https://zh.cppreference.com/w/c/language/arithmetic_types)
- [cppreference：printf 格式說明](https://zh.cppreference.com/w/c/io/fprintf)
- [ASCII 碼錶](https://zh.cppreference.com/w/c/language/ascii)
- 下一課：[運算子與表示式](../03-operators/zh-CN.md)
