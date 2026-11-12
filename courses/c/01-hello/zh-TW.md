# 1. 第一個 C 程式

> 🎯 **本課目標**：知道為什麼要學 C；寫出、編譯並執行第一個 C 程式，逐行看懂它；理解「編譯」到底做了什麼（預處理 → 編譯 → 彙編 → 連結）；用 `printf` 輸出文字和特殊字元；會寫註釋；認識最常見的幾種編譯錯誤。
>
> ⏱ **預計用時**：50 分鐘　｜　📋 **前置知識**：「程式設計起步」課程（尤其是第 3 課終端、第 4 課安裝 C 編譯器、第 5 課讀懂報錯）

## 1.1 為什麼學 C？

C 語言誕生於 1972 年，比你用過的幾乎所有軟體都老。但直到今天，它仍然是世界上最重要的語言之一：

- **作業系統**：Linux、Windows、macOS 的核心主要是用 C 寫的；
- **嵌入式裝置**：洗衣機、汽車、路由器、無人機裡的晶片程式，大多是 C；
- **其他語言的地基**：Python 直譯器（CPython）、資料庫 SQLite、Redis、Git，都是用 C 寫的；
- **理解計算機**：C 讓你直接面對**記憶體**、**地址**和**位元組**。學會 C，你就明白了程式在計算機裡到底是怎樣執行的，再學其他語言也會更紮實。

和 Python 比，C 更「囉嗦」，需要你自己管理更多細節，也更容易出錯——但正是這些細節，能讓你真正理解計算機。別擔心，我們會一步一步來。

## 1.2 Hello, World!

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    printf("你好，C 語言！\n");
    return 0;
}
```

點「執行」（或者在終端編譯執行，見 1.3 節），就會看到兩行文字。我們逐行來看：

| 程式碼 | 意思 |
|------|------|
| `#include <stdio.h>` | 把「標準輸入輸出」庫的說明（標頭檔案）包含進來，這樣才能使用 `printf`。`stdio` = **st**andar**d** **i**nput/**o**utput |
| `int main(void)` | 定義 **`main` 函式**：程式從這裡開始執行。`int` 表示它最後返回一個整數，`void` 表示它不接收引數 |
| `{ ... }` | 花括號括起來的是函式體，也就是要執行的語句 |
| `printf("...\n");` | 輸出一段文字。`\n` 表示換行。**每條語句以分號 `;` 結尾** |
| `return 0;` | `main` 結束，返回 0 給作業系統，表示「程式正常結束」 |

> [!NOTE]
> 和 Python 不同，C 用**花括號**表示程式碼區塊，用**分號**結束語句；縮排只是為了讓人看得清楚，編譯器並不在乎。但請一定保持良好的縮排習慣（4 個空格），否則程式碼很快就會變得難以閱讀。

## 1.3 編譯和執行

Python 是**解釋執行**的：直譯器一邊讀程式碼一邊執行。C 是**編譯執行**的：先用**編譯器**把原始碼翻譯成計算機能直接執行的**機器碼**，生成一個**可執行檔案**，然後再執行這個檔案。

在學習程式裡，點「執行」時它會自動幫你編譯再執行。要在自己的終端裡做，把程式碼儲存為 `hello.c`，然後：

```bash
$ gcc hello.c -o hello       # 編譯：把 hello.c 變成可執行檔案 hello
$ ./hello                    # 執行（macOS / Linux）
$ hello.exe                  # 執行（Windows）
```

- `gcc` 是編譯器（macOS 上的 `gcc` 實際是 Clang，用法相同）；
- `-o hello` 指定輸出檔案的名字，不寫的話預設叫 `a.out`（Windows 上是 `a.exe`）；
- 建議加上 `-Wall -Wextra`，讓編譯器提醒你可能的問題：`gcc -Wall -Wextra hello.c -o hello`。學習程式預設就加了這兩個選項。

### 編譯到底做了什麼？

`gcc` 一條命令背後，其實依次做了四件事：

```text
hello.c ──① 預處理──► 展開 #include，去掉註釋
        ──② 編譯────► 翻譯成組合語言（hello.s）
        ──③ 彙編────► 翻譯成機器碼（hello.o，目標檔案）
        ──④ 連結────► 和庫（printf 的實現在 C 標準庫裡）拼在一起 ──► 可執行檔案 hello
```

所以：

- **編譯時錯誤**（語法錯誤、拼錯名字）會在第 ②、④ 步被發現，程式根本生成不出來；
- 生成的可執行檔案可以**單獨執行**，不需要原始碼，也不需要編譯器——這就是很多軟體釋出時只給你一個 `.exe` 的原因；
- 編譯後的程式執行得非常快，這是 C 至今仍被大量使用的重要原因。

> [!TIP]
> 好奇的話，可以用 `gcc -S hello.c` 看看生成的彙程式設計式碼（`hello.s`）。本課程的「彙編」課會講怎麼讀懂它。

## 1.4 用 printf 輸出

`printf` 不會自動換行，需要換行的地方要自己寫 `\n`：

```c
#include <stdio.h>

int main(void) {
    printf("第一行");
    printf("還是第一行\n");
    printf("第二行\n第三行\n");
    return 0;
}
```

像 `\n` 這樣以反斜槓開頭的組合叫**轉義序列**，用來表示不方便直接寫出來的字元：

| 寫法 | 表示 |
|------|------|
| `\n` | 換行 |
| `\t` | 製表符（Tab，跳到下一個對齊位置） |
| `\\` | 一個反斜槓 `\` |
| `\"` | 一個雙引號 `"` |
| `%%` | 一個百分號 `%`（這是 `printf` 自己的規則，因為 `%` 有特殊用途，下一課會講） |

```c
#include <stdio.h>

int main(void) {
    printf("名字\t年齡\n");
    printf("小明\t12\n");
    printf("他說：\"你好\"\n");
    printf("檔案在 C:\\Users\\me\n");
    printf("完成了 100%%\n");
    return 0;
}
```

## 1.5 註釋

註釋是寫給人看的說明，編譯器會忽略它們：

```c
#include <stdio.h>

/* 這是多行註釋：
   可以寫很多行，
   從 / 和 * 開始，到 * 和 / 結束。 */

int main(void) {
    // 這是單行註釋：從 // 到這一行結尾
    printf("註釋不會影響程式\n");   // 也可以寫在程式碼後面
    return 0;
}
```

## 1.6 關於 `main` 的返回值

`return 0;` 把 0 交給作業系統，表示「一切正常」；返回其他數字（比如 1）通常表示「出錯了」。在終端裡可以檢視上一個程式的返回值（叫**退出碼**）：

```bash
$ ./hello
$ echo $?                 # macOS / Linux：顯示 0
$ echo %errorlevel%       # Windows 命令提示符
```

指令碼和其他程式會根據退出碼判斷你的程式有沒有成功。從 C99 開始，`main` 函式最後如果沒有寫 `return`，會自動當作 `return 0;`——但明確寫出來是好習慣。

## 1.7 最常見的編譯錯誤

編譯器的報錯是英文的，但很有規律。學習程式會在下面用你的語言解釋原因。

**忘了分號**：

```c norun
#include <stdio.h>

int main(void) {
    printf("Hello\n")
    return 0;
}
```

```text
hello.c: In function 'main':
hello.c:4:22: error: expected ';' before 'return'
```

`hello.c:4:22` 表示「第 4 行第 22 列」。注意：分號漏在第 4 行末尾，但編譯器常常要讀到下一個詞（第 5 行的 `return`）才發現問題。**報錯的行號附近、以及它的上一行，都要檢查。**

**拼錯函式名，或者忘了 `#include`**：

```c norun
int main(void) {
    printf("Hello\n");
    return 0;
}
```

```text
hello.c:2:5: warning: implicit declaration of function 'printf' [-Wimplicit-function-declaration]
hello.c:1:1: note: include '<stdio.h>' or provide a declaration of 'printf'
```

編譯器不認識 `printf`，因為沒有 `#include <stdio.h>`。GCC 13 及更早的版本只給出**警告**（warning），程式也許還能執行；GCC 14 起和 Clang 會直接報**錯誤**（error）。不管是哪種，都要改好——**不要忽視警告**，它們常常就是 bug 的源頭。把 `printf` 拼成 `printF`、`pritnf` 也會得到類似的提示（`note` 那一行還會好心地告訴你該包含哪個標頭檔案）。

**用了中文標點**：

```c norun
#include <stdio.h>

int main(void) {
    printf（"Hello\n"）；
    return 0;
}
```

```text
hello.c:4:5: error: 'printf\U0000ff08' undeclared (first use in this function); did you mean 'printf'?
```

`\U0000ff08` 是全形括號 `（` 的 Unicode 編號，編譯器以為你寫了一個叫「printf（」的名字。（不同的編譯器和版本說法不同，也可能是 `stray '\357' in program`，意思是「程式裡有一個不該出現的位元組」。）`（`、`）`、`；`、`“` 都是**全形**的中文標點，C 只認半形的 `(`、`)`、`;`、`"`。寫程式碼時記得切換到英文輸入法（入門課第 5 課講過）。字串**裡面**可以有中文，但程式碼本身的符號必須是英文的。

## 1.8 小結

- C 是離硬體最近的高階語言，作業系統、嵌入式裝置和許多語言的直譯器都是用它寫的。
- 每個 C 程式都從 `int main(void) { ... }` 開始執行；`#include <stdio.h>` 之後才能用 `printf`。
- 語句以 `;` 結尾，程式碼區塊用 `{ }`；縮排是給人看的。
- C 是編譯型語言：`gcc hello.c -o hello` 編譯，`./hello` 執行；編譯 = 預處理 → 編譯 → 彙編 → 連結。
- `printf` 不自動換行；轉義序列 `\n`、`\t`、`\\`、`\"`，以及 `%%`。
- 註釋：`// 單行` 和 `/* 多行 */`。`return 0` 表示程式正常結束。
- 報錯先看行號，也要看上一行；分號、`#include`、中文標點是最常見的錯誤來源。

## 1.9 練習

<!-- exercise: hello-c -->
### 練習 1：你好，C

寫一個程式，輸出一行 `Hello, C!`。

```text
樣例輸出：Hello, C!
```

<details>
<summary>提示</summary>

照著 1.2 節的程式寫：`printf("Hello, C!\n");`，別忘了 `#include <stdio.h>` 和 `return 0;`。
</details>

<!-- exercise: triangle -->
### 練習 2：小三角形

輸出下面這個由星號組成的三角形（4 行）：

```text
*
**
***
****
```

<details>
<summary>提示</summary>

可以寫 4 個 `printf`，也可以只寫一個：`printf("*\n**\n***\n****\n");`。
</details>

<!-- exercise: escapes -->
### 練習 3：跳脫字元

輸出下面三行，一個字元都不能差：

```text
"C" is fun!
Path: C:\new
100% done
```

<details>
<summary>提示</summary>

雙引號寫 `\"`；反斜槓寫 `\\`（注意 `\n` 會被當成換行！）；`printf` 裡的百分號寫 `%%`。
</details>

<!-- exercise: fix-errors -->
### 練習 4：改錯

起始程式碼裡有 3 處錯誤，導致無法編譯。找出並改正它們，讓程式輸出：

```text
I can fix errors.
Compiler is my friend.
```

<details>
<summary>提示</summary>

先執行一下，看編譯器報的第一個錯誤，改好後再執行，一次改一個。檢查：是不是少了 `#include`？分號是不是英文的？函式名有沒有拼錯？
</details>

<!-- exercise: printf-predict -->
### 練習 5：讀程式碼，寫輸出

```c
#include <stdio.h>

int main(void) {
    printf("A");
    printf("B\n");
    printf("C\nD");
    printf("\n");
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`printf` 不會自動換行：`A` 和 `B` 在同一行；`C` 後面換行，`D` 單獨一行，最後再換一次行。所以輸出是 `AB`、`C`、`D` 三行。
</details>

<!-- exercise: return-zero -->
### 練習 6：`return 0` 的意思

`main` 函式里的 `return 0;` 表示什麼？

- 程式輸出數字 0
- 程式正常結束，把退出碼 0 交給作業系統
- 程式會從頭再執行一次
- 釋放所有記憶體，否則程式會出錯

<details>
<summary>答案與解析</summary>

`main` 的返回值是程式的**退出碼**，0 表示成功。它不會在螢幕上輸出任何東西。
</details>

## 1.10 延伸閱讀

- [cppreference：C 語言參考](https://zh.cppreference.com/w/c)（有中文版，以後查語法、查函式用）
- Brian W. Kernighan、Dennis M. Ritchie：《C 程式設計語言》（K&R，C 語言作者寫的經典）
- [Compiler Explorer](https://godbolt.org/)：在瀏覽器裡看 C 程式碼編譯成的彙編
- 下一課：[變數與型別](../02-variables-types/zh-CN.md)
