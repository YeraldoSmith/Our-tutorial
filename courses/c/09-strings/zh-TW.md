# 9. 字串

> 🎯 **本課目標**：明白 C 的字串就是「以 `'\0'` 結尾的字元陣列」；會輸出、讀入字串（讀單詞用 `scanf`，讀整行用 `fgets`），解決「殘留換行符」問題；會用 `<string.h>` 裡的常用函式（`strlen`、`strcpy`、`strcat`、`strcmp`、`strstr`），知道它們的危險以及更安全的 `snprintf`；用 `<ctype.h>` 逐個處理字元；在字串和數字之間轉換；理解漢字在 UTF-8 裡佔好幾個位元組；使用字串陣列。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：本課程第 1～8 課

## 9.1 字串就是以 `'\0'` 結尾的字元陣列

C 沒有專門的「字串型別」。字串就是一個 `char` 陣列，最後放一個特殊的字元 `'\0'`（數值為 0 的字元，叫**空字元**）表示「到這裡結束」：

```c
#include <stdio.h>

int main(void) {
    char a[6] = {'H', 'e', 'l', 'l', 'o', '\0'};   // 一個一個字元地寫
    char b[] = "Hello";                            // 同樣的內容：編譯器自動在末尾加 '\0'

    printf("%s %s\n", a, b);                       // %s 輸出字串
    printf("b 佔 %zu 個位元組\n", sizeof b);         // 6：5 個字母 + 1 個 '\0'

    for (int i = 0; b[i] != '\0'; i++) {           // 一個一個字元地走，直到 '\0'
        printf("[%c]", b[i]);
    }
    printf("\n");

    b[0] = 'J';                                    // 字元陣列裡的字元可以修改
    printf("%s\n", b);                             // Jello
    b[2] = '\0';                                   // 在中間放一個 '\0'……
    printf("%s\n", b);                             // Je：字串到這裡就「結束」了
    return 0;
}
```

```text
         b[0] b[1] b[2] b[3] b[4] b[5]
        ┌────┬────┬────┬────┬────┬────┐
"Hello" │ H  │ e  │ l  │ l  │ o  │ \0 │
        └────┴────┴────┴────┴────┴────┘
```

- `"Hello"`（雙引號）是字串，`'H'`（單引號）是一個字元——它們完全不同（第 2 課）。
- `printf("%s", b)`、`strlen(b)` 等所有字串函式，都是**從頭一直讀到 `'\0'`** 為止。所以 `'\0'` 至關重要：沒有它，函式就會一直讀到陣列外面去（越界，第 8 課）。
- 因此存放 n 個字元的字串，陣列至少要 **n + 1** 個 `char`。

## 9.2 讀入字串

**讀一個單詞**用 `scanf("%Ns", 陣列)`（第 4 課）：讀到空格或換行為止；`N` 寫「陣列長度 − 1」，防止輸入太長寫出界；陣列名前**不加** `&`。

**讀一整行**（可以包含空格）用 `fgets`：

```c input="20\nAda Lovelace"
#include <stdio.h>
#include <string.h>

int main(void) {
    int age = 0;
    char name[50];

    printf("年齡：");
    scanf("%d", &age);

    int ch;                                        // 丟掉這一行剩下的部分（包括換行符）
    while ((ch = getchar()) != '\n' && ch != EOF) {
    }

    printf("\n全名：");
    if (fgets(name, sizeof name, stdin) == NULL) { // 最多讀 49 個字元；讀不到時返回 NULL
        return 1;
    }
    name[strcspn(name, "\n")] = '\0';              // 去掉末尾的換行符

    printf("\n%s，%d 歲\n", name, age);
    return 0;
}
```

這段程式碼裡有三個要點：

1. **`fgets(陣列, 大小, stdin)`** 最多讀「大小 − 1」個字元，絕不會寫出界。它會把行末的**換行符 `'\n'` 也讀進來**，所以通常緊接著用 `name[strcspn(name, "\n")] = '\0';` 把它去掉（`strcspn` 返回第一個 `'\n'` 的位置；沒有換行時返回字串長度，這一句也是安全的）。
2. **殘留的換行符**：`scanf("%d", &age)` 讀完 `20` 就停了，後面的換行符還留在輸入裡。如果不處理，`fgets` 會立刻讀到這個換行符，得到一個**空行**。那個 `while` 迴圈用 `getchar()` 一個一個字元地讀，直到把這一行剩下的東西（包括換行符）都丟掉。
3. `ch` 要定義成 `int` 而不是 `char`：`getchar()` 在輸入結束時返回 `EOF`（通常是 -1），它不是任何一個字元，需要用 `int` 才能和普通字元區分開。

> [!WARNING]
> 你可能在老書裡見過 `gets(name)`。它不檢查長度，輸入一長就寫出界，**已經從 C 語言裡刪除了**。永遠用 `fgets`。

## 9.3 `<string.h>`：字串函式

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[50] = "hello";
    printf("長度 %zu，陣列大小 %zu\n", strlen(s), sizeof s);   // 5 和 50

    char t[50];
    strcpy(t, s);                          // 複製：t = s 是不行的（陣列不能整體賦值）
    strcat(t, ", world");                  // 拼接：接到 t 的末尾
    printf("%s\n", t);                     // hello, world

    if (strcmp(s, "hello") == 0) {         // 比較內容：相等時返回 0
        printf("s 的內容是 hello\n");
    }
    if (strcmp("apple", "banana") < 0) {   // 負數：前一個按字典序排在前面
        printf("apple 排在 banana 前面\n");
    }
    if (strstr(t, "world") != NULL) {      // 找子串：找不到返回 NULL
        printf("t 裡有 world\n");
    }
    if (strchr(t, ',') != NULL) {          // 找一個字元
        printf("t 裡有逗號\n");
    }
    return 0;
}
```

| 函式 | 作用 |
|------|------|
| `strlen(s)` | 字串長度，**不算** `'\0'` |
| `strcpy(d, s)` | 把 `s` 複製到 `d`（`d` 必須足夠大！） |
| `strcat(d, s)` | 把 `s` 接到 `d` 的末尾（`d` 必須足夠大！） |
| `strcmp(a, b)` | 比較：相等返回 0；`a` 排在前面返回負數，排在後面返回正數 |
| `strncmp(a, b, n)` | 只比較前 `n` 個字元，比如判斷開頭：`strncmp(s, "http", 4) == 0` |
| `strchr(s, c)` / `strstr(s, t)` | 查詢字元 `c` / 子串 `t`；找不到返回 `NULL`（返回值其實是一個指標，第 10 課細講） |
| `snprintf(d, size, 格式, ...)` | 像 `printf` 一樣，但「列印」到字串 `d` 裡，最多寫 `size - 1` 個字元 |

### `strcpy` 和 `strcat` 很危險

它們**不知道目標陣列有多大**，只管一直寫。目標放不下時就會寫出界——這正是「緩衝區溢位」漏洞的來源。GCC 能發現一部分這樣的錯誤：

```text
warning: '__builtin_memcpy' writing 8 bytes into a region of size 4 overflows the destination
```

拼接、格式化字串時，更安全的辦法是 `snprintf`：它最多寫 `size - 1` 個字元，再補上 `'\0'`，放不下就**截斷**，絕不越界：

```c
#include <stdio.h>

int main(void) {
    char first[] = "Grace", last[] = "Hopper";
    char full[50];
    snprintf(full, sizeof full, "%s %s", first, last);   // 空間足夠
    printf("[%s]\n", full);

    char tiny[8];
    snprintf(tiny, sizeof tiny, "%s %s", first, last);   // 只放得下 7 個字元
    printf("[%s]\n", tiny);                              // [Grace H]：被安全地截斷了
    return 0;
}
```

（編譯時 GCC 會對第二個 `snprintf` 發出 `output may be truncated` 的警告——它提醒你「這裡會被截斷」，這正是我們想演示的。）

## 9.4 比較字串：不能用 `==`

```c norun
char answer[10] = "yes";
if (answer == "yes") { ... }          // 錯！
```

```text
warning: comparison with string literal results in unspecified behavior [-Waddress]
```

`answer == "yes"` 比較的是兩個字串**在記憶體裡的位置**，而不是內容，結果幾乎總是「不相等」。比較內容**一定要用 `strcmp`**：

```c norun
if (strcmp(answer, "yes") == 0) { ... }   // 對
```

同理，複製用 `strcpy`（或 `snprintf`），不能用 `=`；`strcmp` 相等時返回的是 **0**，別寫成 `if (strcmp(a, b))`——那表示「不相等」。

## 9.5 逐個處理字元：`<ctype.h>`

`<ctype.h>` 提供了判斷和轉換單個字元的函式：

```c
#include <stdio.h>
#include <ctype.h>

int main(void) {
    char s[] = "Hello, World 2026!";
    int letters = 0, digits = 0, spaces = 0, others = 0;

    for (int i = 0; s[i] != '\0'; i++) {
        unsigned char c = (unsigned char)s[i];   // ctype 函式要求傳入 unsigned char 範圍的值
        if (isalpha(c)) {
            letters++;
        } else if (isdigit(c)) {
            digits++;
        } else if (isspace(c)) {
            spaces++;
        } else {
            others++;
        }
        s[i] = (char)toupper(c);                 // 轉成大寫；不是字母的保持不變
    }
    printf("%s\n", s);
    printf("字母 %d，數字 %d，空白 %d，其他 %d\n", letters, digits, spaces, others);
    return 0;
}
```

| 函式 | 判斷 / 作用 |
|------|-------------|
| `isalpha(c)` | 是英文字母嗎 |
| `isdigit(c)` | 是數字 `0`～`9` 嗎 |
| `isalnum(c)` | 是字母或數字嗎 |
| `isspace(c)` | 是空白（空格、`\t`、`\n` 等）嗎 |
| `isupper(c)` / `islower(c)` | 是大寫 / 小寫字母嗎 |
| `toupper(c)` / `tolower(c)` | 轉成大寫 / 小寫（不是字母就原樣返回） |

> [!NOTE]
> 為什麼要先轉成 `unsigned char`？在很多平臺上 `char` 是有符號的，漢字等非英文字元的位元組會變成**負數**，而把負數傳給 `isalpha` 等函式是未定義行為。轉換一下就安全了。

字元就是小整數（第 2 課），所以可以對字元做算術。數字字元 `'0'`～`'9'` 的編碼是連續的，`c - '0'` 就是數字的值；字母也是連續的，`'a' + 2` 就是 `'c'`：

```c
#include <stdio.h>

int main(void) {
    char number[] = "9527";
    int sum = 0;
    for (int i = 0; number[i] != '\0'; i++) {
        sum += number[i] - '0';          // '9' - '0' = 9
    }
    printf("各位數字之和：%d\n", sum);    // 23

    for (char c = 'a'; c <= 'e'; c++) {
        printf("%c", c - 'a' + 'A');     // 小寫變大寫的「手工版」
    }
    printf("\n");
    return 0;
}
```

## 9.6 字串和數字互相轉換

- **字串 → 數字**：用 `sscanf`，它和 `scanf` 一樣，只是從字串裡讀。它的返回值告訴你成功讀到了幾個值。
- **數字 → 字串**：用 `snprintf`。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char text[] = "42 3.5";
    int n = 0;
    double x = 0;
    if (sscanf(text, "%d %lf", &n, &x) == 2) {        // 讀到了 2 個值才算成功
        printf("%d %.1f\n", n * 2, x * 2);
    }
    if (sscanf("abc", "%d", &n) != 1) {
        printf("\"abc\" 不是整數\n");
    }

    char date[20];
    snprintf(date, sizeof date, "%d-%02d-%02d", 2026, 3, 7);
    printf("%s，長度 %zu\n", date, strlen(date));      // 2026-03-07，長度 10
    return 0;
}
```

你也許會見到 `atoi("42")`。它用起來簡單，但遇到 `"abc"` 會悄悄返回 0，沒法知道出了錯；更嚴謹的 `strtol` 要用到指標，第 10 課再介紹。

## 9.7 漢字和 UTF-8

`strlen` 數的是**位元組數**，不是「字數」。現在通用的 **UTF-8** 編碼裡，英文字母佔 1 個位元組，大部分漢字佔 3 個位元組，表情符號佔 4 個位元組：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[] = "C語言";
    printf("strlen = %zu\n", strlen(s));               // 7 = 1 + 3 + 3

    int chars = 0;
    for (int i = 0; s[i] != '\0'; i++) {
        if (((unsigned char)s[i] & 0xC0) != 0x80) {   // 不是「後續位元組」，就是一個新字元的開始
            chars++;
        }
    }
    printf("字元數 = %d\n", chars);                     // 3
    return 0;
}
```

UTF-8 的規則：一個字元的第一個位元組之後的「後續位元組」，二進位制都以 `10` 開頭（`0x80`～`0xBF`）。`& 0xC0` 取出最高兩位（位運算，第 3 課），不等於 `0x80` 的位元組就是一個新字元的開頭。

所以處理中文時要小心：按位元組反轉字串會把漢字拆壞；`char s[4]` 只放得下 1 個漢字；`toupper` 之類的函式只對英文字母有用。

## 9.8 字串陣列

多個字串可以放進一個**二維字元陣列**，每一行是一個字串：

```c
#include <stdio.h>
#include <string.h>

#define COUNT 4
#define LEN 20

int main(void) {
    char names[COUNT][LEN] = {"Carol", "alice", "Bob", "Dave"};

    // 氣泡排序（第 8 課）：比較用 strcmp，交換用 strcpy
    for (int pass = 0; pass < COUNT - 1; pass++) {
        for (int i = 0; i < COUNT - 1 - pass; i++) {
            if (strcmp(names[i], names[i + 1]) > 0) {
                char t[LEN];
                strcpy(t, names[i]);
                strcpy(names[i], names[i + 1]);
                strcpy(names[i + 1], t);
            }
        }
    }
    for (int i = 0; i < COUNT; i++) {
        printf("%s\n", names[i]);
    }
    return 0;
}
```

結果裡 `alice` 排在了最後！因為 `strcmp` 按字元的**編碼**比較，所有大寫字母（`A`～`Z` 是 65～90）都排在小寫字母（`a`～`z` 是 97～122）前面。想忽略大小寫，可以先把兩個字串都轉成小寫再比較。

## 9.9 常見錯誤

| 錯誤 | 後果 | 正確做法 |
|------|------|----------|
| `char s[5] = "Hello";` 沒給 `'\0'` 留位置 | 字串沒有結尾，函式讀出界 | 長度至少 n + 1，或寫 `char s[] = "Hello";` |
| `s = "abc";` | 編譯錯誤：陣列不能整體賦值 | `strcpy(s, "abc");` 或 `snprintf` |
| `if (s == "abc")` | 比較的是位置，幾乎總是假 | `if (strcmp(s, "abc") == 0)` |
| `scanf("%s", s);` 不寫長度 | 輸入太長時寫出界 | `scanf("%49s", s);` |
| `fgets` 之後忘了去掉 `'\n'` | 比較、輸出時多一個換行 | `s[strcspn(s, "\n")] = '\0';` |
| `scanf("%d")` 之後直接 `fgets` | 讀到一個空行 | 先用 `getchar` 迴圈丟掉這一行剩下的部分 |
| `strcpy` / `strcat` 目標太小 | 緩衝區溢位 | 算好大小，或用 `snprintf` |

## 9.10 小結

- C 的字串是以 `'\0'` 結尾的 `char` 陣列；`"..."` 會自動加 `'\0'`；存 n 個字元要 n + 1 的空間。
- 讀單詞：`scanf("%49s", s)`；讀整行：`fgets(s, sizeof s, stdin)`，再去掉 `'\n'`；`scanf` 和 `fgets` 混用時，先丟掉殘留的換行符。
- `<string.h>`：`strlen`、`strcpy`、`strcat`、`strcmp`（相等返回 0）、`strncmp`、`strchr`、`strstr`；`strcpy`/`strcat` 不檢查大小，拼接和格式化優先用 `snprintf`。
- 比較內容用 `strcmp`，不能用 `==`；賦值用 `strcpy`，不能用 `=`。
- `<ctype.h>`：`isalpha`、`isdigit`、`isspace`、`toupper`……引數先轉 `unsigned char`；`c - '0'` 得到數字字元的值。
- 字串 ↔ 數字：`sscanf`（檢查返回值）和 `snprintf`。
- UTF-8 裡一個漢字通常佔 3 個位元組，`strlen` 數的是位元組。
- 字串陣列：`char names[N][LEN]`。

## 9.11 練習

<!-- exercise: char-classes -->
### 練習 1：字元分類統計

輸入一行英文（可能包含空格），分別統計其中的**字母**、**數字**、**空格**和**其他字元**的個數，用空格隔開輸出。（行末的換行符不算。）

```text
樣例輸入：Hello World 123!
樣例輸出：10 3 2 1
```

<details>
<summary>提示</summary>

用 `fgets` 讀一行，去掉 `'\n'`，再用 `isalpha`、`isdigit` 和 `c == ' '` 判斷每個字元。
</details>

<!-- exercise: palindrome -->
### 練習 2：迴文

輸入一個英文單詞（不超過 100 個字母），如果它正著讀和倒著讀一樣（**不區分大小寫**），輸出 `yes`，否則輸出 `no`。

```text
樣例輸入：Level
樣例輸出：yes
```

<details>
<summary>提示</summary>

用兩個下標 `i = 0` 和 `j = strlen(s) - 1` 從兩頭往中間走，比較 `tolower(s[i])` 和 `tolower(s[j])`。
</details>

<!-- exercise: caesar -->
### 練習 3：凱撒密碼

第一行輸入一個整數 `k`（0 ≤ k ≤ 25），第二行輸入一句英文。把每個字母向後移動 `k` 位（`z` 後面回到 `a`），大寫字母仍是大寫、小寫仍是小寫，其他字元不變，輸出加密後的句子。

```text
樣例輸入：
3
Hello, World!
樣例輸出：Khoor, Zruog!
```

<details>
<summary>提示</summary>

小寫字母：`'a' + (c - 'a' + k) % 26`，大寫字母同理。別忘了 `scanf` 讀完 `k` 之後要丟掉殘留的換行符，再用 `fgets` 讀句子。
</details>

<!-- exercise: word-count -->
### 練習 4：數單詞

輸入一行英文，單詞之間用一個或多個空格隔開（行首、行尾也可能有空格），輸出單詞的個數。空行輸出 `0`。

```text
樣例輸入：  the quick  brown fox 
樣例輸出：4
```

<details>
<summary>提示</summary>

數「單詞的開頭」：當前字元不是空格，並且它是第一個字元或者前一個字元是空格時，計數加 1。
</details>

<!-- exercise: title-case -->
### 練習 5：首字母大寫

輸入一行英文，把每個單詞的第一個字母變成大寫，其餘字母變成小寫，輸出結果。單詞之間用空格隔開。

```text
樣例輸入：hello wORLD from c
樣例輸出：Hello World From C
```

<details>
<summary>提示</summary>

和練習 4 一樣判斷「單詞的開頭」：開頭用 `toupper`，其他位置用 `tolower`。
</details>

<!-- exercise: str-predict -->
### 練習 6：讀程式碼，寫輸出

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char s[10] = "abc";
    printf("%zu %zu\n", strlen(s), sizeof s);
    s[1] = '\0';
    printf("%s %zu\n", s, strlen(s));
    printf("%d\n", strcmp("cat", "car") > 0);
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`strlen` 是 3（不算 `'\0'`），`sizeof` 是整個陣列 10。把 `s[1]` 改成 `'\0'` 後，字串在那裡就結束了，只剩 `a`，長度 1。`"cat"` 和 `"car"` 前兩個字元相同，第三個 `t` 比 `r` 大，所以 `strcmp` 返回正數，`> 0` 為真，輸出 1。輸出：

```text
3 10
a 1
1
```
</details>

<!-- exercise: str-compare -->
### 練習 7：比較字串

要判斷字元陣列 `a` 和 `b` 裡的字串內容是否相同，應該怎麼寫？

- `if (a == b)`
- `if (strcmp(a, b) == 0)`
- `if (strcmp(a, b))`
- `if (strlen(a) == strlen(b))`

<details>
<summary>答案與解析</summary>

`strcmp` 在兩個字串相同時返回 0。`a == b` 比較的是位置；`if (strcmp(a, b))` 恰好反了，表示「不相同」；長度相同不代表內容相同。
</details>

## 9.12 延伸閱讀

- [cppreference：空終止位元組字串](https://zh.cppreference.com/w/c/string/byte)（`<string.h>` 和 `<ctype.h>` 的所有函式）
- [UTF-8（維基百科）](https://zh.wikipedia.org/wiki/UTF-8)——看看 UTF-8 是怎樣用 1～4 個位元組表示一個字元的
- 下一課：[指標](../10-pointers/zh-CN.md)
