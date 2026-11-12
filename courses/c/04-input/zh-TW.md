# 4. 輸入：scanf

> 🎯 **本課目標**：用 `scanf` 讀取使用者輸入的整數、小數、字元和單詞；理解為什麼變數前要加 `&`；記住「`scanf` 讀 `double` 用 `%lf`」；檢查 `scanf` 的**返回值**，應對使用者輸錯；解決讀字元時「殘留的換行符」問題；會一直讀到輸入結束為止。
>
> ⏱ **預計用時**：50 分鐘　｜　📋 **前置知識**：本課程第 2、3 課

## 4.1 第一個 scanf

`printf` 負責輸出，`scanf` 負責輸入。它們的格式說明符幾乎一樣：

```c input="12"
#include <stdio.h>

int main(void) {
    int age = 0;
    printf("你幾歲了？");
    scanf("%d", &age);                 // 讀一個整數，存進 age
    printf("明年你就 %d 歲了！\n", age + 1);
    return 0;
}
```

執行後，程式會停下來等你輸入。輸入 `12` 並按回車，程式繼續執行。

### 為什麼要加 `&`？

`&age` 讀作「age 的**地址**」。每個變數都存放在記憶體的某個位置，這個位置有一個編號，就是地址。`scanf` 需要知道**把讀到的數存到哪裡**，所以要把地址告訴它。

```c
#include <stdio.h>

int main(void) {
    int age = 12;
    printf("age 的值是 %d\n", age);
    printf("age 的地址是 %p\n", (void *)&age);     // %p 輸出地址（每次執行可能不同）
    return 0;
}
```

`printf` 只需要知道值，所以不加 `&`；`scanf` 要修改變數，所以必須加 `&`。第 10 課學指標時會徹底弄明白這件事。**忘了 `&` 是新手最常見的錯誤之一，程式通常會直接崩潰。**

## 4.2 一次讀多個值

```c input="3 4"
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    printf("輸入兩個整數：");
    scanf("%d %d", &a, &b);
    printf("%d + %d = %d\n", a, b, a + b);
    return 0;
}
```

`%d` 會自動跳過前面的空格、Tab 和換行，所以使用者輸入 `3 4`、`3    4`，或者分兩行輸入 `3` 和 `4`，結果都一樣。

## 4.3 scanf 的格式說明符

| 型別 | `scanf` 用 | `printf` 用 |
|------|-----------|-------------|
| `int` | `%d` | `%d` |
| `long long` | `%lld` | `%lld` |
| `double` | **`%lf`** | `%f` |
| `float` | `%f` | `%f` |
| `char` | `%c`（或 `" %c"`，見 4.5） | `%c` |
| 一個單詞（字串） | `%s`（**不加 `&`**） | `%s` |

> [!WARNING]
> **`scanf` 讀 `double` 必須用 `%lf`**，用 `%f` 會讀出錯誤的值；而 `printf` 輸出 `double` 用 `%f`。這是 C 裡一個很容易弄錯的不對稱之處。

```c input="1.75 60"
#include <stdio.h>

int main(void) {
    double height = 0, weight = 0;
    printf("身高（米）和體重（千克）：");
    scanf("%lf %lf", &height, &weight);
    printf("BMI = %.1f\n", weight / (height * height));
    return 0;
}
```

## 4.4 檢查返回值：使用者輸錯了怎麼辦？

如果程式要一個整數，使用者卻輸入了 `abc`，會怎樣？`scanf` **讀不進去**，變數保持原來的值。好在 `scanf` 有一個**返回值**：成功讀到了幾個值。

```c input="abc"
#include <stdio.h>

int main(void) {
    int n = 0;
    printf("輸入一個整數：");
    if (scanf("%d", &n) != 1) {          // 應該讀到 1 個值
        printf("\n這不是整數！\n");
    } else {
        printf("你輸入的是 %d\n", n);
    }
    return 0;
}
```

（`if … else` 下一課會詳細講。）真實的程式遇到無法處理的輸入時，常常 `return 1;` 結束，用非 0 的退出碼告訴作業系統「出錯了」（第 1 課）。

| `scanf` 的返回值 | 意思 |
|------------------|------|
| 等於要讀的個數 | 全部成功 |
| 比要讀的個數少 | 只讀到了一部分（後面的格式對不上） |
| `EOF`（通常是 -1） | 輸入已經結束（沒有更多內容可讀） |

> [!TIP]
> **認真的程式都要檢查 `scanf` 的返回值。** 加了 `-Wall` 的編譯器有時會提醒「忽略了 scanf 的返回值」。本課程後面的練習為了簡潔，常常假設輸入總是正確的。

## 4.5 讀字元：小心殘留的換行符

```c input="5\nY"
#include <stdio.h>

int main(void) {
    int count = 0;
    char answer = 0;
    printf("買幾個？");
    scanf("%d", &count);
    printf("確定嗎？(Y/N) ");
    scanf(" %c", &answer);             // 注意 %c 前面有一個空格！
    printf("\n數量 %d，回答 %c\n", count, answer);
    return 0;
}
```

使用者輸入 `5` 後按了回車，回車產生的換行符 `\n` 還**留在輸入裡**。`%d` 會跳過空白，但 `%c` **不會**——它會老老實實地讀到這個換行符。在 `%c` 前加一個空格 `" %c"`，意思是「先跳過所有空白，再讀一個字元」，問題就解決了。

## 4.6 讀一個單詞

```c input="Tom"
#include <stdio.h>

int main(void) {
    char name[50];                     // 一個能放 49 個字元的「字元陣列」（第 9 課細講）
    printf("你的英文名：");
    scanf("%49s", name);               // 陣列名本身就代表地址，所以不加 &
    printf("Hello, %s!\n", name);
    return 0;
}
```

- `%s` 讀到**空白（空格、換行）為止**，所以只能讀一個單詞；讀一整行要用 `fgets`（第 9 課）。
- `%49s` 裡的 49 表示「最多讀 49 個字元」，留一個位置給字串結尾的 `\0`。**一定要寫這個限制**，否則使用者輸入太長時會寫壞記憶體——這是很多安全漏洞的來源。

## 4.7 一直讀到輸入結束

題目常常不告訴你一共有幾個數，只說「讀到輸入結束為止」。利用 `scanf` 的返回值：

```c input="3 8 1 9 4"
#include <stdio.h>

int main(void) {
    int x = 0, count = 0;
    long long sum = 0;
    while (scanf("%d", &x) == 1) {       // 只要還能讀到一個整數，就繼續
        sum += x;
        count++;
    }
    printf("一共 %d 個數，和為 %lld\n", count, sum);
    return 0;
}
```

`while` 迴圈下一課才會詳細講，這裡先記住這個寫法。在終端裡手動輸入時，按 **Ctrl+D**（macOS / Linux）或 **Ctrl+Z 再按回車**（Windows）表示「輸入結束」。

## 4.8 常見錯誤

```c norun
int age = 0;
scanf("%d", age);          // 忘了 &
```

```text
warning: format '%d' expects argument of type 'int *', but argument 2 has type 'int' [-Wformat=]
```

執行時多半會崩潰（`Segmentation fault`）。`int *` 表示「指向 int 的指標（地址）」，編譯器在提醒你：這裡要的是地址。

```c norun
double h = 0;
scanf("%f", &h);           // double 要用 %lf
```

```text
warning: format '%f' expects argument of type 'float *', but argument 2 has type 'double *' [-Wformat=]
```

> [!NOTE]
> 用微軟 Visual Studio 編譯時，可能會看到 `'scanf': This function or variable may be unsafe. Consider using scanf_s instead`（錯誤 C4996）。這是微軟自己的規定，不是 C 標準的要求。可以在檔案最開頭加一行 `#define _CRT_SECURE_NO_WARNINGS`，或者使用 GCC / Clang。

## 4.9 小結

- `scanf("%d", &x)` 讀一個整數存進 `x`；**`&` 表示地址**，`scanf` 要修改變數所以必須加（字元陣列除外）。
- `scanf` 的說明符：`%d`、`%lld`、**`%lf`（double）**、`%c`、`%s`；`%d`、`%lf`、`%s` 會自動跳過空白。
- `scanf` 返回成功讀到的個數，讀到輸入末尾返回 `EOF`；用 `if (scanf(...) != 1)` 檢查輸入是否正確。
- 讀字元用 `" %c"`（前面加空格）跳過殘留的換行；讀單詞用 `%49s` 限制長度。
- `while (scanf("%d", &x) == 1)` 一直讀到輸入結束。

## 4.10 練習

<!-- exercise: sum-two -->
### 練習 1：兩個數

輸入兩個整數，輸出它們的和、差、積，用空格隔開。

```text
樣例輸入：7 3
樣例輸出：10 4 21
```

<details>
<summary>提示</summary>

`scanf("%d %d", &a, &b);` 然後 `printf("%d %d %d\n", a + b, a - b, a * b);`
</details>

<!-- exercise: avg3 -->
### 練習 2：三個小數的平均數

輸入三個小數，輸出它們的平均數，保留 2 位小數。

```text
樣例輸入：1.5 2.5 4
樣例輸出：2.67
```

<details>
<summary>提示</summary>

用 `double` 變數和 `%lf` 讀入：`scanf("%lf %lf %lf", &a, &b, &c);`
</details>

<!-- exercise: next-year -->
### 練習 3：名字和年齡

輸入一個英文名字（一個單詞）和年齡，輸出 `名字 will be 年齡+1 next year.`。

```text
樣例輸入：Tom 12
樣例輸出：Tom will be 13 next year.
```

<details>
<summary>提示</summary>

`char name[50]; int age;`，`scanf("%49s %d", name, &age);`——注意 `name` 不加 `&`，`age` 要加。
</details>

<!-- exercise: until-eof -->
### 練習 4：讀到結束為止

輸入若干個整數（不知道有幾個），輸出它們的個數、總和以及最大值，空格隔開。（至少有一個數。）

```text
樣例輸入：3 8 1 9 4
樣例輸出：5 25 9
```

<details>
<summary>提示</summary>

用 `while (scanf("%d", &x) == 1)`；最大值可以先設為第一個讀到的數，或者初始化為 `INT_MIN`（在 `<limits.h>` 裡）。
</details>

<!-- exercise: scanf-return -->
### 練習 5：讀程式碼，寫輸出

假設執行下面的程式時，輸入是 `12 abc`：

```c
#include <stdio.h>

int main(void) {
    int a = 0, b = 0;
    int n = scanf("%d %d", &a, &b);
    printf("%d %d %d\n", n, a, b);
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`12` 成功讀進 `a`；`abc` 不是整數，讀 `b` 失敗，`b` 保持原來的 0。`scanf` 返回成功讀到的個數 1。輸出 `1 12 0`。
</details>

<!-- exercise: why-amp -->
### 練習 6：為什麼要加 `&`？

`scanf("%d", &age);` 裡為什麼要寫 `&age` 而不是 `age`？

- `&` 表示「這是一個整數」
- `scanf` 要把讀到的值存進變數，需要知道變數在記憶體裡的地址，`&age` 就是 age 的地址
- `&` 只是一種習慣，不加也沒關係
- `&` 讓輸入的數字自動加 1

<details>
<summary>答案與解析</summary>

`&` 是取地址運算子。只傳 `age` 的話，`scanf` 拿到的只是一個值（比如 0），不知道該把結果寫到哪裡，程式往往會崩潰。
</details>

## 4.11 延伸閱讀

- [cppreference：scanf](https://zh.cppreference.com/w/c/io/fscanf)
- 下一課：[條件判斷](../05-conditions/zh-CN.md)
