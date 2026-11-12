# 6. 迴圈

> 🎯 **本課目標**：用 `while`、`do … while`、`for` 讓程式碼重複執行；用 `break` 提前結束迴圈、用 `continue` 跳過一次；寫出巢狀迴圈（乘法表、圖案）；掌握求和、計數、找最大值、拆分數字等常用套路；學會發現「死迴圈」和「差一錯誤」。
>
> ⏱ **預計用時**：75 分鐘　｜　📋 **前置知識**：本課程第 3～5 課

## 6.1 while 迴圈

```c
#include <stdio.h>

int main(void) {
    int i = 1;
    while (i <= 5) {             // 條件為真就執行迴圈體，然後再檢查條件
        printf("第 %d 次\n", i);
        i++;                     // 別忘了改變 i，否則條件永遠為真
    }
    printf("迴圈結束時 i = %d\n", i);
    return 0;
}
```

執行過程：檢查條件 → 為真就執行花括號裡的程式碼 → 回到開頭再檢查……直到條件為假，跳出迴圈。

**死迴圈**：如果條件永遠為真（比如忘了寫 `i++`），迴圈就永遠不會結束。在終端裡按 **Ctrl+C** 可以強制結束程式；學習程式裡點「停止」按鈕。

## 6.2 for 迴圈

「從幾數到幾」是最常見的迴圈，`for` 把三個部分寫在一行裡：

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 5; i++) {
        printf("%d ", i);
    }
    printf("\n");

    for (int i = 10; i > 0; i -= 3) {    // 倒著數，每次減 3
        printf("%d ", i);
    }
    printf("\n");
    return 0;
}
```

```text
for (初始化; 條件; 每次迴圈後做的事) {
    迴圈體
}
```

它和下面的 `while` 完全等價：

```c norun
int i = 1;            // 初始化（只做一次）
while (i <= 5) {      // 條件
    printf("%d ", i); // 迴圈體
    i++;              // 每次迴圈後做的事
}
```

在 `for` 的括號裡宣告的變數（`int i`）只在迴圈裡有效，迴圈結束就不存在了。

## 6.3 do … while：至少執行一次

`do … while` 先執行迴圈體，**再**檢查條件，所以迴圈體至少執行一次。最適合「一直問到輸入正確為止」：

```c input="-3\n150\n42"
#include <stdio.h>

int main(void) {
    int age = 0;
    do {
        printf("請輸入年齡（0～120）：");
        scanf("%d", &age);
    } while (age < 0 || age > 120);       // 注意最後的分號
    printf("\n年齡是 %d\n", age);
    return 0;
}
```

## 6.4 break 和 continue

- **`break`**：立刻跳出整個迴圈；
- **`continue`**：跳過這一次迴圈剩下的部分，直接開始下一次。

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 10; i++) {
        if (i % 3 == 0) {
            continue;              // 跳過 3 的倍數
        }
        if (i > 8) {
            break;                 // 超過 8 就結束
        }
        printf("%d ", i);
    }
    printf("\n");                  // 輸出：1 2 4 5 7 8
    return 0;
}
```

一個常見的寫法是「先無限迴圈，滿足條件時 `break`」：

```c input="5 12 -1 7"
#include <stdio.h>

int main(void) {
    int sum = 0, x = 0;
    while (true) {
        scanf("%d", &x);
        if (x < 0) {
            break;                 // 讀到負數就停止
        }
        sum += x;
    }
    printf("和是 %d\n", sum);      // 17
    return 0;
}
```

## 6.5 迴圈的常用套路

**求和與計數**：

```c
#include <stdio.h>

int main(void) {
    int sum = 0, count = 0;
    for (int i = 1; i <= 100; i++) {
        if (i % 7 == 0) {
            sum += i;              // 累加
            count++;               // 計數
        }
    }
    printf("1～100 裡 7 的倍數有 %d 個，和是 %d\n", count, sum);
    return 0;
}
```

**拆分一個整數的每一位**：`% 10` 取出最後一位，`/ 10` 去掉最後一位：

```c
#include <stdio.h>

int main(void) {
    int n = 2025;
    int digits = 0, digit_sum = 0, reversed = 0;
    while (n > 0) {
        int last = n % 10;         // 最後一位
        digit_sum += last;
        reversed = reversed * 10 + last;
        n /= 10;                   // 去掉最後一位
        digits++;
    }
    printf("%d 位，各位之和 %d，倒過來是 %d\n", digits, digit_sum, reversed);
    return 0;
}
```

**判斷質數**：

```c
#include <stdio.h>

int main(void) {
    int n = 97;
    bool prime = n >= 2;
    for (int d = 2; d * d <= n; d++) {    // 只需要檢查到 √n
        if (n % d == 0) {
            prime = false;
            break;                         // 找到一個因數就夠了
        }
    }
    printf("%d %s質數\n", n, prime ? "是" : "不是");
    return 0;
}
```

## 6.6 巢狀迴圈

迴圈裡面還可以有迴圈。外層每執行一次，內層就完整地執行一遍。九九乘法表：

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 9; i++) {
        for (int j = 1; j <= i; j++) {
            printf("%d×%d=%-3d", j, i, i * j);
        }
        printf("\n");
    }
    return 0;
}
```

用迴圈畫圖案：

```c
#include <stdio.h>

int main(void) {
    int n = 4;
    for (int row = 1; row <= n; row++) {
        for (int s = 0; s < n - row; s++) {
            printf(" ");                  // 先輸出空格
        }
        for (int k = 0; k < 2 * row - 1; k++) {
            printf("*");                  // 再輸出星號
        }
        printf("\n");
    }
    return 0;
}
```

> [!TIP]
> 畫圖案的訣竅：先找出「第 row 行有幾個空格、幾個星號」，寫成和 `row` 有關的式子（比如 `n - row` 和 `2 * row - 1`），再用迴圈輸出。

注意：`break` 只能跳出**它所在的那一層**迴圈。

## 6.7 差一錯誤

迴圈次數多一次或少一次，是最常見的 bug，叫**差一錯誤**（off-by-one error）：

```c norun
for (int i = 0; i <= 10; i++) { ... }   // 執行 11 次（0 到 10）
for (int i = 0; i < 10; i++) { ... }    // 執行 10 次（0 到 9）← C 裡最常見的寫法
for (int i = 1; i <= 10; i++) { ... }   // 執行 10 次（1 到 10）
```

拿不準的時候，用很小的數（比如 1、2、3）在紙上走一遍，或者在迴圈裡 `printf` 一下 `i` 看看。

## 6.8 小結

- `while (條件) { ... }`：先檢查後執行；`do { ... } while (條件);`：至少執行一次。
- `for (初始化; 條件; 更新) { ... }`：適合「從幾數到幾」；`for` 裡宣告的變數只在迴圈內有效。
- `break` 跳出當前迴圈，`continue` 跳到下一次；`while (true) { ... if (...) break; }` 是常用寫法。
- 常用套路：累加、計數、找最大值、`% 10` 和 `/ 10` 拆分數字、`d * d <= n` 判斷質數。
- 巢狀迴圈：外層每走一步，內層走一整遍；畫圖案先找出每行的規律。
- 小心死迴圈（Ctrl+C 結束）和差一錯誤（`<` 還是 `<=`）。

## 6.9 練習

<!-- exercise: sum-n -->
### 練習 1：1 加到 n

輸入一個正整數 `n`，輸出 `1 + 2 + … + n` 的和。（`n` 最大為 100000，注意結果可能超過 `int` 的範圍。）

```text
樣例輸入：100
樣例輸出：5050
```

<details>
<summary>提示</summary>

用 `long long sum = 0;` 累加，輸出用 `%lld`。
</details>

<!-- exercise: times-table -->
### 練習 2：乘法表的一行

輸入一個整數 `n`，輸出 `n × 1` 到 `n × 9`，每行一個，格式是 `n x i = 積`。

```text
樣例輸入：3
樣例輸出：
3 x 1 = 3
3 x 2 = 6
3 x 3 = 9
3 x 4 = 12
3 x 5 = 15
3 x 6 = 18
3 x 7 = 21
3 x 8 = 24
3 x 9 = 27
```

<details>
<summary>提示</summary>

`for (int i = 1; i <= 9; i++) printf("%d x %d = %d\n", n, i, n * i);`
</details>

<!-- exercise: digits -->
### 練習 3：數位

輸入一個非負整數，輸出它有幾位數和各位數字之和，空格隔開。（0 算 1 位。）

```text
樣例輸入：2025
樣例輸出：4 9
```

<details>
<summary>提示</summary>

用 `do { ... } while (n > 0);`，這樣 0 也會被算作 1 位。
</details>

<!-- exercise: primes -->
### 練習 4：質數列表

輸入一個整數 `n`（`n ≥ 2`），輸出 2 到 `n` 之間的所有質數，用空格隔開。

```text
樣例輸入：20
樣例輸出：2 3 5 7 11 13 17 19
```

<details>
<summary>提示</summary>

外層迴圈 `k` 從 2 到 `n`，內層用 6.5 節的方法判斷 `k` 是不是質數。輸出時注意第一個數前面不要有空格：可以用一個變數記錄「是不是第一個」。
</details>

<!-- exercise: pyramid -->
### 練習 5：金字塔

輸入 `n`，輸出一個 `n` 行的星號金字塔（第 `row` 行先輸出 `n - row` 個空格，再輸出 `2 × row - 1` 個星號）。

```text
樣例輸入：3
樣例輸出：
  *
 ***
*****
```

<details>
<summary>提示</summary>

照著 6.6 節的程式寫，把 `n` 改成用 `scanf` 讀入。
</details>

<!-- exercise: loop-predict -->
### 練習 6：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    int i = 0;
    while (i < 10) {
        i += 3;
        if (i == 6) {
            continue;
        }
        printf("%d ", i);
    }
    printf("\n");
    for (int j = 5; j > 0; j--) {
        if (j == 2) break;
        printf("%d", j);
    }
    printf("\n");
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`while`：i 依次變成 3（輸出）、6（`continue` 跳過輸出）、9（輸出）、12（輸出，然後 12 < 10 不成立，結束）。第一行是 `3 9 12 `。`for`：j 為 5、4、3 時輸出，j 為 2 時 `break`。第二行是 `543`。
</details>

<!-- exercise: off-by-one -->
### 練習 7：迴圈幾次？

`for (int i = 0; i <= 10; i++)` 的迴圈體執行幾次？

- 9 次
- 10 次
- 11 次
- 無限次

<details>
<summary>答案與解析</summary>

i 取 0, 1, 2, …, 10，一共 11 個值。想要 10 次，通常寫 `i < 10`。
</details>

## 6.10 延伸閱讀

- [cppreference：for 迴圈](https://zh.cppreference.com/w/c/language/for)、[while 迴圈](https://zh.cppreference.com/w/c/language/while)
- 下一課：[函式](../07-functions/zh-CN.md)
