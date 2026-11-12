# 3. 運算子與表示式

> 🎯 **本課目標**：掌握 C 的算術運算，特別是**整數除法**和取餘 `%` 的規則；學會用**型別轉換**得到小數結果；理解運算子的優先順序；會用 `+=`、`++` 等簡寫；掌握比較運算和邏輯運算（以及「短路」）；避開 `=` 和 `==` 的經典陷阱；用 `<math.h>` 做數學計算；明白為什麼 `0.1 + 0.2 != 0.3`；初步認識位運算。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：本課程第 1、2 課

## 3.1 算術運算

```c
#include <stdio.h>

int main(void) {
    int a = 17, b = 5;
    printf("%d + %d = %d\n", a, b, a + b);
    printf("%d - %d = %d\n", a, b, a - b);
    printf("%d * %d = %d\n", a, b, a * b);
    printf("%d / %d = %d\n", a, b, a / b);      // 整數除法：3，小數部分直接丟掉
    printf("%d %% %d = %d\n", a, b, a % b);     // 取餘數：2
    return 0;
}
```

### 整數除法：最常見的「意外」

在 C 裡，**兩個整數相除，結果還是整數**，小數部分被直接截掉（向 0 取整），不會四捨五入：

| 表示式 | 結果 | 說明 |
|--------|------|------|
| `7 / 2` | `3` | 不是 3.5 |
| `-7 / 2` | `-3` | 向 0 截斷（不是 -4） |
| `7 % 3` | `1` | 餘數 |
| `-7 % 3` | `-1` | 餘數的正負號跟被除數一樣 |
| `7.0 / 2` | `3.5` | 只要有一個是小數，就做小數除法 |

這和 Python 很不一樣（Python 的 `7 / 2` 是 3.5，`-7 // 2` 是 -4）。

`%` 取餘非常有用：判斷奇偶（`n % 2 == 0`）、取個位數（`n % 10`）、迴圈計數（`i % 7` 是星期幾）……

## 3.2 型別轉換

想讓兩個整數相除得到小數，需要先把其中一個**轉換**成 `double`：

```c
#include <stdio.h>

int main(void) {
    int total = 17, count = 5;
    double wrong = total / count;               // 先做整數除法得到 3，再變成 3.0
    double right = (double)total / count;       // 先把 total 變成 17.0，再做小數除法
    printf("%.2f %.2f\n", wrong, right);        // 3.00 3.40

    double price = 9.99;
    int yuan = (int)price;                      // 小數轉整數：直接截掉小數部分（不是四捨五入）
    printf("%d\n", yuan);                       // 9
    return 0;
}
```

`(型別)值` 叫**強制型別轉換**（cast）。注意 `wrong` 那一行：等號右邊先算完 `total / count`（整數除法，結果 3），**之後**才賦值給 `double` 變數——這時小數部分早就丟了。

C 也會自動做一些轉換：`int` 和 `double` 一起運算時，`int` 會自動變成 `double`；把 `double` 賦值給 `int` 變數時會截斷小數（加了 `-Wextra` 的編譯器有時會警告）。

## 3.3 優先順序

和數學一樣，先乘除後加減，有括號先算括號：

```c
#include <stdio.h>

int main(void) {
    printf("%d\n", 2 + 3 * 4);       // 14
    printf("%d\n", (2 + 3) * 4);     // 20
    printf("%d\n", 10 - 4 - 3);      // 3：同級從左往右
    printf("%d\n", 2 * 3 % 4);       // 2：* / % 同級，從左往右：6 % 4
    return 0;
}
```

C 的運算子多達幾十種，優先順序表很長。**不確定的時候就加括號**——括號不會讓程式變慢，卻能讓程式碼更清楚。

## 3.4 賦值的簡寫和自增自減

```c
#include <stdio.h>

int main(void) {
    int score = 10;
    score += 5;      // score = score + 5    → 15
    score -= 3;      // score = score - 3    → 12
    score *= 2;      // score = score * 2    → 24
    score /= 5;      // score = score / 5    → 4（整數除法）
    score %= 3;      // score = score % 3    → 1
    printf("%d\n", score);

    int n = 5;
    n++;             // 自增：n = n + 1 → 6
    n--;             // 自減：n = n - 1 → 5
    printf("%d\n", n);

    int a = 5;
    int b = a++;     // 後置：先把 a 原來的值（5）給 b，再把 a 加 1
    int c = ++a;     // 前置：先把 a 加 1（變成 7），再把新值給 c
    printf("a=%d b=%d c=%d\n", a, b, c);   // a=7 b=5 c=7
    return 0;
}
```

> [!WARNING]
> 不要在同一個表示式裡對同一個變數做多次修改，比如 `i = i++ + ++i;`。這在 C 裡是**未定義行為**，不同編譯器的結果可能完全不同。`++` 最好單獨寫成一句，或者只用在 `for` 迴圈裡（第 6 課）。

## 3.5 比較運算

比較的結果是**真（1）**或**假（0）**：

| 運算子 | 意思 |
|--------|------|
| `==` | 等於（兩個等號！） |
| `!=` | 不等於 |
| `<`、`>` | 小於、大於 |
| `<=`、`>=` | 小於等於、大於等於 |

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    printf("%d %d %d\n", x == 5, x != 5, x > 3);    // 1 0 1
    bool adult = x >= 18;
    printf("%d\n", adult);                           // 0
    return 0;
}
```

在 C 裡，**0 表示假，任何非 0 的值都表示真**。`bool` 型別只是讓意思更清楚。

### 經典陷阱：`=` 和 `==`

```c norun
int x = 5;
if (x = 0) {           // 本想比較 x 是否等於 0，卻寫成了賦值！
    printf("x 是 0\n");
}
```

`x = 0` 是**賦值**：把 0 存進 `x`，整個表示式的值是 0（假），所以 `if` 永遠不執行，而且 `x` 被改掉了。加了 `-Wall` 的編譯器會提醒：

```text
warning: suggest parentheses around assignment used as truth value [-Wparentheses]
```

## 3.6 邏輯運算

| 運算子 | 意思 | 例子 |
|--------|------|------|
| `&&` | 並且（兩邊都真才真） | `age >= 6 && age <= 12` |
| `\|\|` | 或者（有一邊真就真） | `day == 6 \|\| day == 7` |
| `!` | 非（取反） | `!finished` |

```c
#include <stdio.h>

int main(void) {
    int age = 10, day = 6;
    printf("%d\n", age >= 6 && age <= 12);     // 1：是小學生年齡
    printf("%d\n", day == 6 || day == 7);      // 1：是週末
    printf("%d\n", !(age > 18));               // 1

    int count = 0;
    if (count != 0 && 100 / count > 5) {      // 短路：count 是 0 時，後半部分根本不會計算
        printf("平均數大於 5\n");
    }
    printf("沒有除以 0\n");
    return 0;
}
```

**短路求值**：`&&` 左邊是假，整個結果一定是假，右邊就**不再計算**；`||` 左邊是真，右邊也不再計算。上面的例子正是利用這一點，避免了除以 0。

> [!NOTE]
> 不能像數學那樣寫 `6 <= age <= 12`！C 會先算 `6 <= age`（得到 0 或 1），再拿這個 0 或 1 和 12 比較，結果幾乎總是真。要寫成 `age >= 6 && age <= 12`。

## 3.7 數學函式：`<math.h>`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    printf("%.4f\n", sqrt(2.0));        // 平方根
    printf("%.1f\n", pow(2, 10));       // 2 的 10 次方：1024.0
    printf("%.1f\n", fabs(-3.5));       // 絕對值（小數用 fabs，整數用 abs，後者在 <stdlib.h>）
    printf("%.1f %.1f %.1f\n", round(2.5), floor(-2.5), ceil(2.1));   // 四捨五入、向下取整、向上取整
    double pi = acos(-1.0);             // 標準 C 沒有 π 常量，可以這樣算出來
    printf("%.6f\n", pi);
    printf("%.4f\n", sin(pi / 6));      // 三角函式用弧度：sin(30°) = 0.5
    return 0;
}
```

> [!TIP]
> 在 Linux 上用 gcc 編譯使用了 `<math.h>` 的程式，命令最後要加 `-lm`（連結數學庫）：`gcc main.c -o main -lm`。學習程式已經自動加上了。

## 3.8 小數不精確：`0.1 + 0.2 != 0.3`

```c
#include <stdio.h>
#include <math.h>

int main(void) {
    double s = 0.1 + 0.2;
    printf("%.17f\n", s);                       // 0.30000000000000004
    printf("%d\n", s == 0.3);                   // 0：不相等！
    printf("%d\n", fabs(s - 0.3) < 1e-9);       // 1：差距足夠小，就認為相等
    return 0;
}
```

計算機用二進位制存小數，而 0.1 在二進位制裡是無限迴圈小數，只能存一個近似值（就像 1/3 在十進位制裡只能寫成 0.333…）。所以：

- **不要用 `==` 比較兩個小數**，而是看它們的差是否足夠小；
- **錢不要用小數存**：用整數存「分」，比如 12.50 元存成 1250。

## 3.9 位運算（初步）

C 可以直接操作整數的每一個**二進位制位**。這在嵌入式、網路協議、影象處理中很常用：

```c
#include <stdio.h>

int main(void) {
    int a = 13;   // 二進位制 1101
    int b = 6;    // 二進位制 0110
    printf("%d\n", a & b);    // 按位與：0100 = 4
    printf("%d\n", a | b);    // 按位或：1111 = 15
    printf("%d\n", a ^ b);    // 按位異或：1011 = 11
    printf("%d\n", 1 << 5);   // 左移 5 位：100000 = 32，相當於乘以 2⁵
    printf("%d\n", 40 >> 2);  // 右移 2 位：相當於除以 4 → 10
    printf("%d\n", (a >> 2) & 1);   // 取出 a 的第 2 位（從 0 數起）：1
    return 0;
}
```

別把 `&`（按位與）和 `&&`（邏輯與）、`|` 和 `||` 弄混。本課程的「彙編」課會更深入地講二進位制。

## 3.10 小結

- `+ - * / %`；**整數相除結果是整數**（向 0 截斷）；`%` 的符號跟被除數。
- 要小數結果：`(double)a / b`；小數轉整數會截斷。
- 優先順序：先乘除後加減，不確定就加括號。
- `+=`、`-=`、`*=`……；`++`、`--`（前置先加後用，後置先用後加）；不要在一個表示式裡多次修改同一個變數。
- 比較 `== != < > <= >=` 結果是 1 或 0；**`=` 是賦值，`==` 才是比較**；0 為假，非 0 為真。
- 邏輯 `&& || !` 會短路；區間判斷要寫 `a <= x && x <= b`。
- `<math.h>`：`sqrt`、`pow`、`fabs`、`round`、`floor`、`ceil`、三角函式（Linux 編譯加 `-lm`）。
- 小數有誤差：比較用 `fabs(a - b) < 1e-9`，錢用整數存。
- 位運算 `& | ^ ~ << >>` 直接操作二進位制位。

## 3.11 練習

<!-- exercise: divide -->
### 練習 1：整數除法與小數除法

程式裡已經有 `int a = 17, b = 5;`。輸出三個數，用空格隔開：整數商、餘數、精確的商（保留 2 位小數）。

```text
樣例輸出：3 2 3.40
```

<details>
<summary>提示</summary>

`printf("%d %d %.2f\n", a / b, a % b, (double)a / b);`
</details>

<!-- exercise: clock -->
### 練習 2：秒數換算

程式裡已經有 `int total = 3725;`（秒）。用 `/` 和 `%` 把它換算成「時:分:秒」的格式輸出，分和秒都要兩位數。

```text
樣例輸出：1:02:05
```

<details>
<summary>提示</summary>

小時 = `total / 3600`；分鐘 = `total % 3600 / 60`；秒 = `total % 60`。輸出用 `%d:%02d:%02d`。
</details>

<!-- exercise: circle -->
### 練習 3：圓

程式裡已經有 `double r = 2.5;`。輸出圓的面積和周長，保留 2 位小數，空格隔開。π 用 `acos(-1.0)` 計算。

```text
樣例輸出：19.63 15.71
```

<details>
<summary>提示</summary>

`#include <math.h>`，`double pi = acos(-1.0);`，面積 `pi * r * r`，周長 `2 * pi * r`。
</details>

<!-- exercise: bits -->
### 練習 4：位運算

輸出 `13 & 6`、`13 | 6`、`13 ^ 6`、`1 << 5` 四個值，空格隔開。

```text
樣例輸出：4 15 11 32
```

<details>
<summary>提示</summary>

直接寫在 `printf` 的引數裡就行，不需要額外的變數。
</details>

<!-- exercise: ops-predict -->
### 練習 5：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    printf("%d %d %d\n", 7 / 2, -7 / 2, 7 % -3);
    printf("%.1f %d\n", 7 / 2 * 1.0, (int)3.99);
    int a = 5;
    int b = a++;
    printf("%d %d\n", a, b);
    return 0;
}
```

<details>
<summary>答案與解析</summary>

第一行：`7 / 2` 是 3；`-7 / 2` 向 0 截斷是 -3；`7 % -3` 的符號跟被除數 7，是 1。第二行：`7 / 2` 先算出整數 3，再乘 1.0 得 3.0；`(int)3.99` 截斷為 3。第三行：`b = a++` 先把 5 給 b，再把 a 變成 6。所以輸出 `3 -3 1`、`3.0 3`、`6 5`。
</details>

<!-- exercise: assign-trap -->
### 練習 6：`=` 還是 `==`？

`int x = 5; if (x = 0) { ... }` 執行後會怎樣？

- `if` 裡的程式碼執行，`x` 仍然是 5
- `if` 裡的程式碼不執行，而且 `x` 變成了 0
- 編譯失敗
- `if` 裡的程式碼執行，`x` 變成了 0

<details>
<summary>答案與解析</summary>

`x = 0` 是賦值，`x` 被改成 0，表示式的值是 0（假），所以 `if` 裡的程式碼不執行。比較要寫 `x == 0`。
</details>

## 3.12 延伸閱讀

- [cppreference：C 運算子優先順序](https://zh.cppreference.com/w/c/language/operator_precedence)
- [cppreference：<math.h> 常用數學函式](https://zh.cppreference.com/w/c/numeric/math)
- [0.30000000000000004.com](https://0.30000000000000004.com/)：各種語言裡的小數誤差
- 下一課：[輸入：scanf](../04-input/zh-CN.md)
