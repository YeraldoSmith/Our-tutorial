# 5. 條件判斷

> 🎯 **本課目標**：用 `if`、`else if`、`else` 讓程式根據情況做不同的事；明白**花括號**為什麼一定要寫；用 `switch` 處理「多選一」，理解 `break` 和「貫穿」；會用三元運算子 `? :`；認識「`if` 後面多一個分號」等經典陷阱。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：本課程第 3、4 課（比較和邏輯運算、scanf）

## 5.1 if 語句

```c input="75"
#include <stdio.h>

int main(void) {
    int score = 0;
    printf("分數：");
    scanf("%d", &score);
    if (score >= 60) {
        printf("及格了！\n");
    }
    printf("程式結束\n");
    return 0;
}
```

- 條件寫在**圓括號**裡，括號必須有；
- 條件為真（非 0）時，執行花括號裡的程式碼；為假時跳過；
- 不像 Python 那樣靠縮排，而是靠 `{ }` 確定哪些程式碼屬於 `if`。

## 5.2 else 和 else if

```c input="85"
#include <stdio.h>

int main(void) {
    int score = 0;
    scanf("%d", &score);
    if (score >= 90) {
        printf("優秀\n");
    } else if (score >= 80) {
        printf("良好\n");
    } else if (score >= 60) {
        printf("及格\n");
    } else {
        printf("不及格\n");
    }
    return 0;
}
```

從上往下依次檢查，**只執行第一個成立的分支**，其餘的全部跳過。所以順序很重要：如果把 `score >= 60` 寫在最前面，90 分也會被判成「及格」。

## 5.3 一定要寫花括號

如果 `if` 後面只有一條語句，C 允許省略花括號。但這是很多 bug 的來源：

```c norun
if (balance < price)
    printf("餘額不足\n");
    return 1;                // 看起來屬於 if，其實不屬於！無論如何都會執行
```

沒有花括號時，`if` 只管**緊跟著的一條語句**。第二行雖然縮排了，但它不在 `if` 裡面——程式每次都會執行 `return 1;`。編譯器（加了 `-Wall`）會提醒：

```text
warning: this 'if' clause does not guard... [-Wmisleading-indentation]
```

2014 年，蘋果系統裡一個著名的安全漏洞（「goto fail」）就是這樣產生的：多了一行沒有花括號保護的語句，導致 SSL 證書檢查被跳過。**養成習慣：`if`、`else`、`for`、`while` 後面永遠寫花括號。**

## 5.4 經典陷阱：if 後面的分號

```c norun
if (x > 100);              // 注意這個分號！
{
    printf("x 很大\n");    // 不管 x 是多少都會執行
}
```

`if (x > 100);` 後面的分號本身是一條「空語句」，`if` 管的就是它；下面的花括號成了一個普通的程式碼區塊，總會執行。編譯器會警告 `suggest braces around empty body in an 'if' statement`。

## 5.5 巢狀和組合條件

`if` 裡面可以再寫 `if`：

```c input="2024"
#include <stdio.h>

int main(void) {
    int year = 0;
    scanf("%d", &year);
    // 閏年：能被 4 整除但不能被 100 整除，或者能被 400 整除
    if (year % 4 == 0) {
        if (year % 100 != 0 || year % 400 == 0) {
            printf("%d 是閏年\n", year);
        } else {
            printf("%d 不是閏年\n", year);
        }
    } else {
        printf("%d 不是閏年\n", year);
    }
    return 0;
}
```

巢狀太深會難以閱讀。很多時候可以用邏輯運算把條件合併成一個：

```c norun
bool leap = (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
```

## 5.6 switch：多選一

當要根據**一個整數（或字元）的值**在多種情況中選一種時，`switch` 比一長串 `else if` 更清楚：

```c input="3"
#include <stdio.h>

int main(void) {
    int day = 0;
    scanf("%d", &day);
    switch (day) {
        case 1:
            printf("星期一\n");
            break;
        case 2:
            printf("星期二\n");
            break;
        case 3:
            printf("星期三\n");
            break;
        case 6:
        case 7:                        // 6 和 7 共用同一段程式碼
            printf("週末！\n");
            break;
        default:                       // 以上都不是時執行
            printf("其他日子\n");
            break;
    }
    return 0;
}
```

- `switch (表示式)` 後面的值只能是**整數或字元**（不能是小數，也不能是字串）；
- 每個 `case` 後面是一個常量，要和表示式的值相等才會從這裡開始執行；
- **`break` 跳出 `switch`**。如果忘了寫，程式會**繼續執行下一個 `case` 的程式碼**，這叫**貫穿**（fall through）；
- 上面 `case 6:` 後面什麼都沒寫，就是故意利用貫穿，讓 6 和 7 執行同樣的程式碼。

> [!WARNING]
> 忘寫 `break` 是 `switch` 最常見的錯誤。加了 `-Wextra` 的編譯器會警告 `this statement may fall through`。如果是故意要貫穿，C23 可以寫 `[[fallthrough]];` 來說明意圖。

```c
#include <stdio.h>

int main(void) {
    char op = '*';
    int a = 6, b = 7;
    switch (op) {
        case '+': printf("%d\n", a + b); break;
        case '-': printf("%d\n", a - b); break;
        case '*': printf("%d\n", a * b); break;
        case '/':
            if (b != 0) {
                printf("%d\n", a / b);
            } else {
                printf("不能除以 0\n");
            }
            break;
        default:  printf("不認識的運算子 %c\n", op); break;
    }
    return 0;
}
```

## 5.7 三元運算子 `? :`

`條件 ? 值1 : 值2`：條件為真時得到值1，否則得到值2。適合寫簡短的「二選一」：

```c
#include <stdio.h>

int main(void) {
    int a = 7, b = 12;
    int bigger = (a > b) ? a : b;
    printf("較大的是 %d\n", bigger);

    int n = 3;
    printf("%d 是%s數\n", n, (n % 2 == 0) ? "偶" : "奇");
    int stock = 0;
    printf("%s\n", stock > 0 ? "有貨" : "缺貨");
    return 0;
}
```

不要把三元運算子巢狀得太複雜，那樣還不如寫 `if`。

## 5.8 小結

- `if (條件) { ... } else if (條件) { ... } else { ... }`：從上往下檢查，只執行第一個成立的分支。
- **永遠寫花括號**：沒有花括號時 `if` 只管一條語句；縮排對 C 沒有意義。
- `if (...);` 後面的分號會讓條件失效。
- `switch` 根據整數或字元的值多選一；每個 `case` 結尾寫 `break`，否則會貫穿；`default` 處理其他情況。
- `條件 ? a : b` 是簡短的二選一。

## 5.9 練習

<!-- exercise: sign -->
### 練習 1：正數、負數還是零

輸入一個整數，是正數輸出 `positive`，負數輸出 `negative`，零輸出 `zero`。

```text
樣例輸入：-5
樣例輸出：negative
```

<details>
<summary>提示</summary>

`if (n > 0) ... else if (n < 0) ... else ...`
</details>

<!-- exercise: grade -->
### 練習 2：等級

輸入一個 0～100 的分數，輸出等級：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。

```text
樣例輸入：85
樣例輸出：B
```

<details>
<summary>提示</summary>

從高到低依次判斷 `score >= 90`、`score >= 80`……
</details>

<!-- exercise: leap -->
### 練習 3：閏年

輸入一個年份，是閏年輸出 `leap`，否則輸出 `common`。（能被 4 整除但不能被 100 整除，或者能被 400 整除的是閏年。）

```text
樣例輸入：1900
樣例輸出：common
```

<details>
<summary>提示</summary>

`if ((y % 4 == 0 && y % 100 != 0) || y % 400 == 0)`
</details>

<!-- exercise: calc -->
### 練習 4：用 switch 做計算器

輸入一個算式，格式是 `整數 運算子 整數`（運算子是 `+`、`-`、`*`、`/` 之一，前後有空格），輸出結果（除法是整數除法）。如果除數是 0 或者運算子不認識，輸出 `error`。

```text
樣例輸入：6 * 7
樣例輸出：42
```

<details>
<summary>提示</summary>

`scanf("%d %c %d", &a, &op, &b);`——`%c` 前面的空格會跳過空白。然後 `switch (op)`。
</details>

<!-- exercise: braces-predict -->
### 練習 5：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    if (x > 10)
        printf("A\n");
        printf("B\n");
    if (x > 3) printf("C\n"); else printf("D\n");
    switch (x) {
        case 5: printf("E\n");
        case 6: printf("F\n"); break;
        default: printf("G\n");
    }
    return 0;
}
```

<details>
<summary>答案與解析</summary>

第一個 `if` 沒有花括號，只管 `printf("A\n")`；`x > 10` 為假，所以不輸出 A，但 B 不受 `if` 控制，照樣輸出。`x > 3` 為真，輸出 C。`switch` 從 `case 5` 開始，輸出 E 後沒有 `break`，貫穿到 `case 6` 輸出 F，然後 `break`。所以輸出是 B、C、E、F 四行。
</details>

<!-- exercise: semicolon -->
### 練習 6：多出來的分號

`if (x > 100); { printf("big\n"); }` 會怎樣？

- 只有 x 大於 100 時才輸出 big
- 不管 x 是多少都會輸出 big
- 編譯失敗
- 永遠不會輸出 big

<details>
<summary>答案與解析</summary>

`if (x > 100);` 的分號是一條空語句，`if` 只管它；後面的花括號是獨立的程式碼區塊，總會執行。編譯器會警告 `suggest braces around empty body`。
</details>

## 5.10 延伸閱讀

- [cppreference：if 語句](https://zh.cppreference.com/w/c/language/if)、[switch 語句](https://zh.cppreference.com/w/c/language/switch)
- 「goto fail」漏洞的故事（搜尋 Apple goto fail SSL bug）
- 下一課：[迴圈](../06-loops/zh-CN.md)
