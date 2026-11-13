# 13. 檔案

> 🎯 **本課目標**：用 `fopen` 開啟檔案、`fclose` 關閉檔案，分清 `"r"`、`"w"`、`"a"` 等開啟模式；用 `fprintf`、`fputs`、`fgetc`、`fgets`、`fscanf` 讀寫文字檔案；明白 `stdin`、`stdout`、`stderr` 也是「檔案」，寫出對任何 `FILE *` 都適用的函式；處理「檔案打不開」之類的錯誤（`perror`、`strerror(errno)`）；讀取 CSV 這樣的格式化資料；用 `fwrite`、`fread` 讀寫二進位制檔案；接收命令列引數 `argc`、`argv`。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：本課程第 1～12 課

程式執行時的變數都放在記憶體裡，程式一結束就沒了。想把資料**儲存下來**——遊戲存檔、成績表、配置——就要寫進**檔案**。

## 13.1 開啟、寫入、關閉、讀取

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("hello.txt", "w");       // 開啟檔案準備寫；不存在就建立
    if (f == NULL) {                         // 開啟失敗時返回 NULL
        perror("hello.txt");                 // 列印失敗的原因
        return 1;
    }
    fprintf(f, "你好，檔案！\n");             // 和 printf 一樣，只是多了第一個引數
    fprintf(f, "1 + 1 = %d\n", 1 + 1);
    fclose(f);                               // 關閉：確保資料真正寫進了檔案

    f = fopen("hello.txt", "r");             // 再以「讀」的方式開啟
    if (f == NULL) {
        perror("hello.txt");
        return 1;
    }
    char line[100];
    while (fgets(line, sizeof line, f) != NULL) {   // 一行一行地讀，讀完了返回 NULL
        printf("讀到：%s", line);
    }
    fclose(f);
    return 0;
}
```

- **`FILE *`** 是標準庫定義的「檔案指標」，代表一個開啟的檔案。`fopen` 返回它，之後的讀寫函式都要用它。
- **一定要檢查 `fopen` 是否返回 `NULL`**：檔案不存在、沒有許可權、路徑寫錯，都會開啟失敗。拿 `NULL` 去讀寫，程式會崩潰。
- **用完要 `fclose`**。寫檔案時資料會先放在記憶體的緩衝區裡，`fclose` 時才真正寫進磁碟；忘了關閉，檔案可能是空的或不完整的。
- 檔名 `"hello.txt"` 是**相對路徑**，相對於程式執行時的「當前目錄」。在本教程的學習程式裡，每次執行都在一個臨時資料夾裡進行。

| 模式 | 作用 |
|------|------|
| `"r"` | 讀。檔案必須已經存在 |
| `"w"` | 寫。檔案不存在就建立；**已經存在就清空原有內容** |
| `"a"` | 追加。寫入的內容接在檔案末尾 |
| `"r+"` / `"w+"` / `"a+"` | 既能讀又能寫（`"w+"` 同樣會清空檔案） |
| 加上 `b`，如 `"rb"`、`"wb"` | 二進位制模式（13.5 節） |
| 加上 `x`，如 `"wx"` | 檔案已經存在時**開啟失敗**，防止不小心覆蓋（C11 起） |

## 13.2 `stdin`、`stdout`、`stderr` 也是檔案

程式一啟動，就已經有三個開啟的 `FILE *`：

| 名字 | 是什麼 |
|------|--------|
| `stdin` | 標準輸入：鍵盤，或者學習程式裡的輸入框 |
| `stdout` | 標準輸出：`printf` 寫到這裡 |
| `stderr` | 標準錯誤：專門用來輸出錯誤資訊 |

`printf(...)` 其實就是 `fprintf(stdout, ...)`，`scanf(...)` 就是 `fscanf(stdin, ...)`。因此，把 `FILE *` 作為引數的函式，**既能處理檔案，也能處理鍵盤輸入**：

```c input="apple\nbanana\ncherry"
#include <stdio.h>

int count_lines(FILE *in) {              // 對任何 FILE * 都適用
    int lines = 0;
    int ch;
    while ((ch = fgetc(in)) != EOF) {    // fgetc：讀一個字元；讀完了返回 EOF
        if (ch == '\n') {
            lines++;
        }
    }
    return lines;
}

int main(void) {
    FILE *f = fopen("abc.txt", "w");
    if (f == NULL) {
        perror("abc.txt");
        return 1;
    }
    fputs("a\nb\nc\nd\n", f);            // fputs：寫一個字串
    fclose(f);

    f = fopen("abc.txt", "r");
    if (f == NULL) {
        perror("abc.txt");
        return 1;
    }
    printf("abc.txt 有 %d 行\n", count_lines(f));
    fclose(f);

    printf("標準輸入有 %d 行\n", count_lines(stdin));
    fprintf(stderr, "這句話寫到了 stderr\n");
    return 0;
}
```

錯誤資訊寫到 `stderr` 有兩個好處：即使輸出被重定向到檔案裡（`./main > out.txt`），錯誤資訊仍然會顯示在螢幕上；而且它不會混進正常的輸出——學習程式判題時也只比較 `stdout`。

常用的讀寫函式：

| 寫 | 讀 | 單位 |
|----|----|------|
| `fputc(c, f)` | `fgetc(f)`（讀完返回 `EOF`） | 一個字元 |
| `fputs(s, f)` | `fgets(buf, size, f)`（讀完返回 `NULL`） | 一行 / 一個字串 |
| `fprintf(f, 格式, ...)` | `fscanf(f, 格式, ...)`（返回成功讀到的個數） | 格式化資料 |
| `fwrite(p, size, n, f)` | `fread(p, size, n, f)`（返回讀到的個數） | 一塊位元組（二進位制） |

## 13.3 處理錯誤

開啟失敗時，全域性變數 **`errno`**（在 `<errno.h>` 裡）記錄著錯誤的編號，`strerror(errno)`（在 `<string.h>` 裡）把它變成一句說明，`perror("字首")` 則直接把「字首: 說明」輸出到 `stderr`。

很多程式在找不到配置檔案時會改用預設設定，而不是直接退出：

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void) {
    int volume = 50;                                  // 預設設定
    FILE *f = fopen("settings.txt", "r");
    if (f == NULL) {
        printf("沒有讀到 settings.txt（%s），使用預設設定\n", strerror(errno));
    } else {
        if (fscanf(f, "%d", &volume) != 1) {          // 檢查 fscanf 的返回值
            printf("settings.txt 的格式不對，使用預設設定\n");
            volume = 50;
        }
        fclose(f);
    }
    printf("音量：%d\n", volume);
    return 0;
}
```

（學習程式裡會顯示 `No such file or directory`，也就是「沒有這個檔案或目錄」。）如果檔案是必需的，就輸出錯誤資訊並結束程式：`return EXIT_FAILURE;`（`<stdlib.h>` 裡定義的「失敗」退出碼，通常是 1）。

### 讀到檔案末尾：別用 `while (!feof(f))`

```c norun
while (!feof(f)) {                  // 錯誤的寫法！
    fgets(line, sizeof line, f);
    printf("%s", line);             // 最後一行會被處理兩次
}
```

`feof(f)` 只有在**某次讀取已經失敗之後**才變成真，所以這個迴圈會多轉一圈，用上一次留下的內容再處理一遍。正確做法是**直接檢查讀取函式的返回值**：`while (fgets(...) != NULL)`、`while (fscanf(...) == 1)`、`while ((ch = fgetc(f)) != EOF)`。

## 13.4 讀取格式化資料：CSV

**CSV**（逗號分隔值）是最常見的表格資料格式，Excel 也能開啟它：

```text
name,score
Ada,95
Linus,88
Grace,91
```

最穩妥的讀法是：用 `fgets` 讀一整行，再用 `sscanf` 從這一行裡取資料。這樣某一行格式不對，也不會影響後面的行：

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("scores.csv", "w");
    if (f == NULL) {
        perror("scores.csv");
        return 1;
    }
    fputs("name,score\nAda,95\nLinus,88\nGrace,91\n", f);
    fclose(f);

    f = fopen("scores.csv", "r");
    if (f == NULL) {
        perror("scores.csv");
        return 1;
    }
    char line[200];
    if (fgets(line, sizeof line, f) == NULL) {         // 跳過第一行（表頭）
        fclose(f);
        return 0;
    }
    char name[32];
    int score = 0, count = 0;
    long total = 0;
    while (fgets(line, sizeof line, f) != NULL) {
        if (sscanf(line, "%31[^,],%d", name, &score) == 2) {   // %[^,]：讀到逗號為止
            printf("%-6s %d\n", name, score);
            total += score;
            count++;
        }
    }
    fclose(f);
    if (count > 0) {
        printf("平均 %.2f\n", (double)total / count);
    }
    return 0;
}
```

`%31[^,]` 叫**掃描集**：`[^,]` 表示「除了逗號以外的任何字元」，所以它會一直讀到逗號為止（最多 31 個字元），名字裡有空格也沒關係。類似地，`%[^\n]` 讀到行尾為止。

## 13.5 二進位制檔案

`fwrite` 把記憶體裡的位元組**原樣**寫進檔案，`fread` 再原樣讀回來。儲存一個結構體陣列只需要一行：

```c
#include <stdio.h>

typedef struct {
    char name[16];
    int level;
    double hp;
} Player;

int main(void) {
    Player team[3] = {{"Ada", 12, 98.5}, {"Bob", 7, 40.0}, {"Cy", 20, 150.25}};
    FILE *f = fopen("save.dat", "wb");
    if (f == NULL) {
        perror("save.dat");
        return 1;
    }
    size_t written = fwrite(team, sizeof team[0], 3, f);    // 3 個元素，每個 sizeof team[0] 位元組
    fclose(f);
    printf("寫入了 %zu 個存檔，每個 %zu 位元組\n", written, sizeof team[0]);

    Player loaded[3];
    f = fopen("save.dat", "rb");
    if (f == NULL) {
        perror("save.dat");
        return 1;
    }
    size_t got = fread(loaded, sizeof loaded[0], 3, f);    // 返回實際讀到的元素個數
    fclose(f);
    for (size_t i = 0; i < got; i++) {
        printf("%-4s Lv.%-3d HP %.2f\n", loaded[i].name, loaded[i].level, loaded[i].hp);
    }
    return 0;
}
```

**文字檔案和二進位制檔案怎麼選？**

- 二進位制檔案小、讀寫快，但人看不懂；而且 `int` 的大小、位元組的排列順序、結構體的填充（第 12 課）在不同的電腦和編譯器上可能不一樣，換臺機器就可能讀不出來。
- 文字檔案（CSV、JSON……）稍大一些，但人能直接看、能用別的程式處理，跨平臺也沒問題。
- 要和別人、別的程式交換資料時，優先用文字格式；只給自己的程式讀寫的臨時資料，可以用二進位制。

## 13.6 命令列引數

在終端裡執行程式時，可以在後面跟上引數，比如 `./wordcount essay.txt -v`。`main` 函式可以這樣接收它們：

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("一共 %d 個引數\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    return 0;
}
```

- `argc`（argument count）是引數的個數，`argv`（argument vector）是一個字串陣列。
- `argv[0]` 是程式自己的名字（路徑），真正的引數從 `argv[1]` 開始。執行 `./wordcount essay.txt -v` 時，`argc` 是 3。
- 學習程式執行程式碼時不帶引數，所以你只會看到 `argc` 為 1。在終端裡編譯執行時可以試試帶上引數。

很多命令列工具都這樣開頭：

```c norun
if (argc < 2) {
    fprintf(stderr, "用法：%s 檔名\n", argv[0]);
    return EXIT_FAILURE;
}
FILE *f = fopen(argv[1], "r");
```

## 13.7 常見錯誤

| 錯誤 | 後果 | 正確做法 |
|------|------|----------|
| 不檢查 `fopen` 的返回值 | 檔案打不開時程式崩潰 | `if (f == NULL) { perror(...); ... }` |
| 忘了 `fclose` | 資料沒寫完整；開啟的檔案太多 | 每個 `fopen` 都配一個 `fclose` |
| 用 `"w"` 開啟了重要的檔案 | 原有內容被清空 | 追加用 `"a"`；防止覆蓋用 `"wx"` |
| `while (!feof(f))` | 最後一項被處理兩次 | 檢查讀取函式的返回值 |
| `fopen("data.txt", 'r')` | 警告：單引號是字元，不是字串 | `"r"` |
| `fprintf("...", x)` 忘了檔案引數 | 警告：第一個引數型別不對 | `fprintf(f, "...", x)` |
| 檔案「找不到」 | 相對路徑是相對於當前目錄的 | 確認當前目錄，或者用完整路徑 |

## 13.8 小結

- `FILE *f = fopen(名字, 模式);` 檢查 `NULL`；用完 `fclose(f);`。
- 模式：`"r"` 讀、`"w"` 寫（清空）、`"a"` 追加，加 `b` 為二進位制、加 `x` 防覆蓋。
- 文字讀寫：`fputc`/`fgetc`、`fputs`/`fgets`、`fprintf`/`fscanf`；`stdin`/`stdout`/`stderr` 也是 `FILE *`，錯誤資訊寫到 `stderr`。
- 出錯時用 `perror` 或 `strerror(errno)` 說明原因。
- 讀到末尾：檢查讀取函式的返回值，不要用 `while (!feof(f))`。
- CSV：`fgets` 讀一行 + `sscanf` 取資料，`%[^,]` 讀到逗號為止。
- 二進位制：`fwrite`/`fread` 原樣讀寫位元組，快但不跨平臺。
- `int main(int argc, char *argv[])` 接收命令列引數，`argv[0]` 是程式名。

## 13.9 練習

<!-- exercise: wc-lite -->
### 練習 1：數行、單詞和位元組

寫函式 `void count(FILE *in, long *lines, long *words, long *bytes)`，統計 `in` 裡的**行數**（換行符的個數）、**單詞數**（連續的非空白字元算一個單詞）和**位元組數**（所有字元，包括換行符）。`main` 用 `count(stdin, ...)` 統計輸入，輸出 `行數 單詞數 位元組數`。（這正是 Linux 的 `wc` 命令做的事。）

```text
樣例輸入：
hello world
  C is fun
樣例輸出：2 5 23
```

<details>
<summary>提示</summary>

用 `fgetc` 一個一個字元地讀，每讀一個 `bytes++`；遇到 `'\n'` 就 `lines++`。用一個變數 `in_word` 記住「當前是否在單詞裡」：遇到非空白字元而 `in_word` 為假時，`words++` 並把 `in_word` 設為真；遇到空白（`isspace`）就設為假。
</details>

<!-- exercise: file-roundtrip -->
### 練習 2：存進檔案，再讀出來

輸入 `n`（1 ≤ n ≤ 1000）和 `n` 個整數。把它們寫進檔案 `numbers.txt`（每行一個），關閉檔案；再重新開啟它，用 `fscanf` 一直讀到讀不出為止，輸出讀到的個數、總和、最大值，用空格隔開。

```text
樣例輸入：
4
3 -1 10 2
樣例輸出：4 14 10
```

<details>
<summary>提示</summary>

寫：`fprintf(f, "%d\n", x);`。讀：`while (fscanf(f, "%d", &x) == 1) { ... }`。兩次 `fopen` 都要檢查 `NULL`，兩次都要 `fclose`。
</details>

<!-- exercise: csv-top -->
### 練習 3：CSV 成績表

從標準輸入讀入一個 CSV 表格：第一行是表頭 `name,score`，之後每行是 `名字,成績`（名字不超過 30 個字元，可能含空格；至少有 1 行資料）。輸出平均分（保留 2 位小數）和最高分的人的名字（同分取先出現的），用空格隔開。

```text
樣例輸入：
name,score
Ada,95
Linus,88
Grace,91
樣例輸出：91.33 Ada
```

<details>
<summary>提示</summary>

照著 13.4 節寫，只是把檔案換成 `stdin`：`fgets(line, sizeof line, stdin)`。最高分用嚴格大於 `>` 來更新，這樣同分時保留先出現的。
</details>

<!-- exercise: grep-lite -->
### 練習 4：查詢包含某個詞的行

第一行是要找的詞，之後的每一行（直到輸入結束）是一段文字。輸出所有包含這個詞的文字行，格式是 `行號: 內容`（行號從文字的第一行開始數，從 1 開始）。如果一行都沒有，輸出 `not found`。

```text
樣例輸入：
cat
the cat sat
a dog ran
concatenate
樣例輸出：
1: the cat sat
3: concatenate
```

<details>
<summary>提示</summary>

先用 `fgets` 讀第一行並去掉 `'\n'`；然後迴圈 `fgets` 讀其餘各行，去掉 `'\n'` 後用 `strstr(line, word) != NULL` 判斷。用一個變數記住有沒有找到過。
</details>

<!-- exercise: file-predict -->
### 練習 5：讀程式碼，寫輸出

```c
#include <stdio.h>

void show(const char *name) {
    FILE *f = fopen(name, "r");
    if (f == NULL) {
        return;
    }
    char line[50];
    while (fgets(line, sizeof line, f) != NULL) {
        printf("%s", line);
    }
    fclose(f);
}

int main(void) {
    FILE *f = fopen("demo.txt", "w");
    if (f == NULL) {
        return 1;
    }
    fputs("one\n", f);
    fclose(f);

    f = fopen("demo.txt", "a");
    if (f == NULL) {
        return 1;
    }
    fputs("two\n", f);
    fclose(f);
    show("demo.txt");
    printf("---\n");

    f = fopen("demo.txt", "w");
    if (f == NULL) {
        return 1;
    }
    fputs("three\n", f);
    fclose(f);
    show("demo.txt");
    return 0;
}
```

<details>
<summary>答案與解析</summary>

`"w"` 建立檔案並寫入 `one`；`"a"` 在末尾追加 `two`，所以第一次顯示兩行。再用 `"w"` 開啟時，原有內容被清空，只剩 `three`。輸出：

```text
one
two
---
three
```
</details>

<!-- exercise: open-w -->
### 練習 6：`"w"` 模式

用 `"w"` 模式開啟一個**已經存在、裡面有內容**的檔案，會發生什麼？

- 新內容追加在原有內容的後面
- 原有內容被清空，從頭開始寫
- `fopen` 失敗，返回 `NULL`
- 檔案以只讀方式開啟

<details>
<summary>答案與解析</summary>

`"w"` 會把已有的檔案清空（長度變成 0）。要追加用 `"a"`；不想覆蓋已有檔案，可以用 `"wx"`，檔案已存在時 `fopen` 會失敗。
</details>

## 13.10 延伸閱讀

- [cppreference：檔案輸入/輸出](https://zh.cppreference.com/w/c/io)（`fopen`、`fgets`、`fscanf`、`fread` 等）
- [RFC 4180：CSV 格式（英文）](https://www.rfc-editor.org/rfc/rfc4180)——欄位裡有逗號或引號時該怎麼寫
- 下一課：[多檔案程式和 C23](../14-multifile-c23/zh-CN.md)
