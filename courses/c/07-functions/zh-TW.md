# 7. 函式

> 🎯 **本課目標**：把一段功能寫成**函式**，理解引數、返回值和 `void`；明白為什麼函式要「先宣告後使用」，會寫**函式原型**；理解 C 的引數是**值傳遞**（函式拿到的是副本）；分清區域性變數、全域性變數和 `static` 區域性變數；寫出遞迴函式（階乘、漢諾塔），瞭解**呼叫堆疊**和堆疊溢位。
>
> ⏱ **預計用時**：80 分鐘　｜　📋 **前置知識**：本課程第 1～6 課

## 7.1 定義和呼叫函式

我們一直在用別人寫好的函式：`printf`、`scanf`、`sqrt`……`main` 本身也是一個函式。現在來寫自己的：

```c
#include <stdio.h>

int square(int x) {          // 返回型別  函式名(引數型別 引數名)
    return x * x;            // return：把結果交回給呼叫者，並結束函式
}

void greet(void) {           // void：不返回任何值；引數裡的 void：不需要引數
    printf("你好！\n");
}

int main(void) {
    greet();
    int a = square(5);       // 呼叫 square，把 5 傳給 x，得到 25
    printf("%d\n", a);
    printf("%d\n", square(3) + square(4));   // 返回值可以直接參與計算：25
    return 0;
}
```

| 部分 | 意思 |
|------|------|
| `int` | **返回型別**：函式交回一個 `int`。不返回任何東西就寫 `void` |
| `square` | 函式名，命名規則和變數一樣 |
| `(int x)` | **引數列表**：每個引數都要寫型別，多個引數用逗號隔開，如 `(int a, double b)` |
| `return x * x;` | 返回結果。`void` 函式可以只寫 `return;` 提前結束，或者不寫 |

和 Python 一樣，函式可以避免重複程式碼、讓程式更好讀、更容易測試。和 Python 不同的是：**引數和返回值都必須寫明型別**。

## 7.2 先宣告，後使用：函式原型

C 編譯器**從上往下**讀程式碼。如果在 `main` 裡呼叫了一個寫在 `main` 後面的函式，編譯器讀到呼叫的時候還不認識它：

```c norun
#include <stdio.h>

int main(void) {
    printf("%d\n", square(4));     // 這裡還不知道 square 是什麼
    return 0;
}

int square(int x) {
    return x * x;
}
```

```text
warning: implicit declaration of function 'square' [-Wimplicit-function-declaration]
```

（新版本的編譯器會直接報錯。）解決辦法有兩個：把函式定義挪到呼叫之前；或者在前面寫一個**函式原型**（宣告）——只寫「簽名」，不寫函式體，以分號結尾：

```c
#include <stdio.h>

int square(int x);                 // 函式原型：告訴編譯器「有這樣一個函式」
double average(int a, int b);

int main(void) {
    printf("%d %.1f\n", square(4), average(3, 4));
    return 0;
}

int square(int x) {                // 函式的定義可以放在後面
    return x * x;
}

double average(int a, int b) {
    return (a + b) / 2.0;          // 除以 2.0 才是小數除法（第 3 課）
}
```

很多 C 程式都是這種結構：開頭是所有函式的原型，接著是 `main`，最後是各個函式的定義。多檔案的程式會把原型放進**標頭檔案**（`.h`）裡——`#include <stdio.h>` 其實就是把 `printf` 等函式的原型包含進來。第 14 課會講。

## 7.3 值傳遞：函式拿到的是副本

呼叫函式時，實參的**值被複制**給引數。函式里修改引數，不會影響外面的變數：

```c
#include <stdio.h>

void try_to_change(int n) {
    n = 100;                       // 只改了副本
    printf("函式里 n = %d\n", n);
}

void swap(int a, int b) {          // 想交換兩個數……
    int t = a;
    a = b;
    b = t;
}

int main(void) {
    int x = 5;
    try_to_change(x);
    printf("外面 x = %d\n", x);    // 還是 5

    int p = 1, q = 2;
    swap(p, q);
    printf("p = %d, q = %d\n", p, q);   // 還是 1 和 2：交換失敗了！
    return 0;
}
```

想讓函式修改外面的變數（比如真正的 `swap`），或者「返回多個值」，需要把變數的**地址**傳進去——這正是 `scanf("%d", &x)` 要加 `&` 的原因。第 10 課學了指標，我們就能寫出正確的 `swap`。

## 7.4 區域性變數、全域性變數和 static

```c
#include <stdio.h>

int total_calls = 0;               // 全域性變數：寫在所有函式外面，所有函式都能用

int next_id(void) {
    static int id = 0;             // static 區域性變數：只初始化一次，函式結束後值仍然保留
    id++;
    total_calls++;
    return id;
}

int main(void) {
    int a = next_id();
    int b = next_id();
    int c = next_id();
    printf("%d %d %d\n", a, b, c);          // 1 2 3
    printf("一共呼叫了 %d 次\n", total_calls);
    return 0;
}
```

| 種類 | 寫在哪裡 | 誰能用 | 活多久 |
|------|----------|--------|--------|
| 區域性變數 | 函式（或程式碼區塊）裡面 | 只有這個函式 | 函式結束就消失 |
| `static` 區域性變數 | 函式里面，加 `static` | 只有這個函式 | 整個程式執行期間，值一直保留 |
| 全域性變數 | 所有函式外面 | 所有函式 | 整個程式執行期間 |

> [!TIP]
> **儘量少用全域性變數**。任何函式都能修改它，程式一大就很難弄清楚是誰、在什麼時候改了它。能用引數和返回值傳遞的，就不要用全域性變數。

## 7.5 遞迴：函式呼叫自己

和 Python（Python 課第 10 課）一樣，C 的函式也可以呼叫自己：

```c
#include <stdio.h>

long long factorial(int n) {
    if (n <= 1) {                  // 終止條件
        return 1;
    }
    return n * factorial(n - 1);   // 把問題變成更小的同類問題
}

int main(void) {
    for (int i = 0; i <= 20; i += 5) {
        printf("%d! = %lld\n", i, factorial(i));
    }
    return 0;
}
```

遞迴最漂亮的例子之一是**漢諾塔**：把 n 個盤子從 A 柱移到 C 柱，每次移一個，大盤子不能壓在小盤子上。思路是：先把上面 n−1 個移到 B，再把最大的移到 C，最後把 n−1 個從 B 移到 C——而「移 n−1 個」又是同樣的問題：

```c
#include <stdio.h>

void hanoi(int n, char from, char via, char to) {
    if (n == 0) {
        return;
    }
    hanoi(n - 1, from, to, via);         // 上面 n-1 個：from → via
    printf("%c -> %c\n", from, to);      // 最大的一個：from → to
    hanoi(n - 1, via, from, to);         // 那 n-1 個：via → to
}

int main(void) {
    hanoi(3, 'A', 'B', 'C');             // 3 個盤子需要 7 步
    return 0;
}
```

### 呼叫堆疊

每次呼叫函式，計算機都會在記憶體的**堆疊**（stack）上開闢一塊空間，存放這次呼叫的引數、區域性變數和「執行完要回到哪裡」；函式返回時這塊空間被釋放。遞迴呼叫 `factorial(4)` 時，堆疊上會同時有 `factorial(4)`、`factorial(3)`、`factorial(2)`、`factorial(1)` 四層。

堆疊的大小是有限的（通常幾 MB）。如果遞迴**忘了終止條件**，或者層數太深，堆疊就會被用光——**堆疊溢位**（stack overflow），程式會崩潰，通常顯示 `Segmentation fault`。著名的程式設計師問答網站 Stack Overflow 就是以此命名的。

## 7.6 好函式的習慣

- **一個函式只做一件事**，名字說清楚它做什麼：`is_prime`、`read_scores`、`print_report`；
- 函式不要太長，超過一屏就考慮拆分；
- 讓函式透過引數拿資料、透過返回值交結果，少依賴全域性變數；
- 「計算」和「輸入輸出」分開：`int max3(int a, int b, int c)` 只負責計算，列印交給 `main`——這樣的函式最容易測試和複用。

## 7.7 小結

- 函式定義：`返回型別 名字(引數型別 引數, ...) { ... return 值; }`；不返回值用 `void`，沒有引數寫 `(void)`。
- C 從上往下讀程式碼：函式要**先宣告後使用**，可以在前面寫原型 `int square(int x);`。
- 引數是**值傳遞**：函式修改引數不影響外面的變數；要修改外面的變數需要傳地址（第 10 課）。
- 區域性變數只在函式里有效；`static` 區域性變數的值在多次呼叫之間保留；少用全域性變數。
- 遞迴必須有終止條件；每次呼叫佔用一塊堆疊空間，太深會堆疊溢位。

## 7.8 練習

<!-- exercise: max3-c -->
### 練習 1：三個數的最大值

補全函式 `int max3(int a, int b, int c)`，返回三個數中最大的一個。`main` 已經寫好：讀入三個整數，輸出 `max3` 的結果。

```text
樣例輸入：3 9 4
樣例輸出：9
```

<details>
<summary>提示</summary>

`int m = a; if (b > m) m = b; if (c > m) m = c; return m;`
</details>

<!-- exercise: gcd -->
### 練習 2：最大公約數和最小公倍數

寫函式 `int gcd(int a, int b)` 用**輾轉相除法**求最大公約數：當 `b` 不為 0 時，不斷把 `(a, b)` 換成 `(b, a % b)`，最後的 `a` 就是答案。輸入兩個正整數，輸出它們的最大公約數和最小公倍數，空格隔開。

```text
樣例輸入：12 18
樣例輸出：6 36
```

<details>
<summary>提示</summary>

最小公倍數 = `a / gcd(a, b) * b`（先除再乘，避免溢位）。用 `long long` 更保險。
</details>

<!-- exercise: count-primes -->
### 練習 3：區間裡的質數

寫函式 `bool is_prime(int n)`，然後用它統計閉區間 `[a, b]` 裡有多少個質數。輸入 `a b`，輸出個數。

```text
樣例輸入：10 30
樣例輸出：6
```

<details>
<summary>提示</summary>

`is_prime` 裡：小於 2 返回 `false`；`for (int d = 2; d * d <= n; d++)` 能整除就返回 `false`；最後返回 `true`。
</details>

<!-- exercise: hanoi -->
### 練習 4：漢諾塔

輸入盤子數 `n`（1 ≤ n ≤ 10），按 7.5 節的方法輸出把它們從 `A` 移到 `C`（藉助 `B`）的每一步，格式 `A -> C`。

```text
樣例輸入：2
樣例輸出：
A -> B
A -> C
B -> C
```

<details>
<summary>提示</summary>

照著 7.5 節的 `hanoi` 函式寫，`main` 裡呼叫 `hanoi(n, 'A', 'B', 'C');`。
</details>

<!-- exercise: func-predict -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`swap` 只交換了副本，`x`、`y` 不變：`1 2`。`counter` 裡的 `n` 是 `static`，每次呼叫都在上一次的基礎上加 10：前兩次呼叫後是 20，第三次返回 30。輸出 `1 2` 和 `30`。
</details>

<!-- exercise: prototype -->
### 練習 6：為什麼要寫函式原型？

- 讓函式執行得更快
- C 編譯器從上往下讀程式碼，呼叫一個後面才定義的函式之前，需要先用原型告訴編譯器它的名字、引數和返回型別
- 原型就是函式的定義，寫了就不用再寫函式體
- 只有 `void` 函式才需要原型

<details>
<summary>答案與解析</summary>

原型讓編譯器在看到呼叫時就能檢查引數個數和型別。也可以把函式定義直接放在呼叫之前，那樣就不需要原型。
</details>

## 7.9 延伸閱讀

- [cppreference：函式宣告](https://zh.cppreference.com/w/c/language/function_declaration)、[儲存期與連結（static）](https://zh.cppreference.com/w/c/language/storage_duration)
- 下一課：[陣列](../08-arrays/zh-CN.md)
