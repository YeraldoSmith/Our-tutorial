# 2. 電腦基礎：檔案、資料夾與輸入法

> 🎯 **本課目標**：搞清楚檔案、資料夾、副檔名和路徑；學會顯示副檔名；分清全形和半形符號；認識程式設計時常用的鍵磁碟機代號號和快捷鍵。
>
> ⏱ **預計用時**：30 分鐘　｜　📋 **前置知識**：第 1 課

很多人學程式設計卡住，不是因為不懂程式設計，而是因為不熟悉電腦本身的一些概念。這一課把這些「地基」補上。

## 2.1 檔案和資料夾

電腦裡的一切資料——照片、文件、音樂、程式——都儲存在**檔案**（file）裡。為了方便管理，檔案被放在**資料夾**（folder，也叫**目錄** directory）裡，資料夾裡還可以再放資料夾，形成一棵「樹」：

```text
C:\Users\xiaoming\              ← 你的「使用者資料夾」（Windows）
├── Desktop\                     ← 桌面
├── Documents\                   ← 文件
│   ├── 簡歷.docx
│   └── code\                    ← 我們建一個放程式碼的資料夾
│       ├── hello.py
│       └── game\
│           └── main.py
└── Downloads\                   ← 下載
```

> [!TIP]
> 建議你現在就在「文件」裡新建一個叫 `code` 的資料夾，以後練習寫的程式碼都放在這裡。**資料夾名最好用英文字母、數字、短橫線**，不要有空格和中文——有些程式設計工具遇到中文路徑或空格會出問題。

## 2.2 副檔名：檔案的「型別標籤」

檔名最後一個點 `.` 後面的部分叫**副檔名**（extension），它告訴電腦這是什麼型別的檔案：

| 副檔名 | 型別 |
|--------|------|
| `.txt` | 純文字 |
| `.docx` | Word 文件 |
| `.jpg` `.png` | 圖片 |
| `.exe` | Windows 程式 |
| `.py` | Python 原始碼 |
| `.c` `.h` | C 原始碼 |
| `.cpp` | C++ 原始碼 |
| `.java` | Java 原始碼 |
| `.rs` | Rust 原始碼 |
| `.go` | Go 原始碼 |
| `.js` `.ts` | JavaScript / TypeScript 原始碼 |
| `.md` | Markdown 文件（本教程的課文就是 .md 檔案） |

### 一定要開啟「顯示副檔名」

Windows 和 macOS 預設會**隱藏**副檔名。這會造成一個經典問題：你以為檔案叫 `hello.py`，其實它叫 `hello.py.txt`，結果怎麼都執行不了。

- **Windows 11**：開啟「檔案資源管理器」→ 頂部「檢視」→「顯示」→ 勾選「副檔名」。
- **Windows 10**：「檔案資源管理器」→「檢視」→ 勾選「副檔名」。
- **macOS**：開啟「訪達」→ 選單欄「訪達」→「設定」→「高階」→ 勾選「顯示所有副檔名」。
- **Linux**：大多數檔案管理器預設就會顯示。

## 2.3 程式碼是「純文字」

程式碼檔案是**純文字檔案**：裡面只有文字，沒有字型、顏色、加粗這些格式。所以：

- ✅ 用**程式碼編輯器**寫程式碼：本課程的學習程式、VS Code、記事本（Windows）、TextEdit（macOS 需要切換到「純文字」模式）。
- ❌ **不要**用 Word、WPS、Pages 寫程式碼——它們會偷偷加入格式，還會把直引號 `"` 自動換成彎引號 `“”`，程式碼就執行不了了。

> [!NOTE]
> 文字檔案儲存時還有一個「編碼」設定，決定文字怎麼變成 0 和 1。**永遠選 UTF-8**——它能表示世界上所有文字，也是所有現代程式設計工具的預設值。第 6 課會講它的原理。

## 2.4 路徑：檔案的「地址」

**路徑**（path）就是檔案在電腦裡的完整地址，從最外層的資料夾一層層寫到檔案：

- Windows：`C:\Users\xiaoming\Documents\code\hello.py`（用**反斜槓** `\` 分隔，開頭是磁碟機代號 `C:`）
- macOS：`/Users/xiaoming/Documents/code/hello.py`（用**斜槓** `/` 分隔）
- Linux：`/home/xiaoming/code/hello.py`

兩種寫法：

- **絕對路徑**：從最頂層開始寫的完整地址，如上面的例子。
- **相對路徑**：相對於「當前所在資料夾」的地址。如果你現在在 `code` 資料夾裡，那麼 `game/main.py` 就指向 `code/game/main.py`；`..` 表示「上一層資料夾」。

下一課學終端時會大量用到路徑。

## 2.5 全形和半形：中文使用者最常踩的坑 ⚠️

中文輸入法有兩種標點狀態：

- **半形**（英文標點）：`( ) , ; : " ' . ! ?`——每個符號佔半個漢字的寬度。
- **全形**（中文標點）：`（ ），；：“ ” ‘ ’。！？`——每個符號佔一個漢字的寬度。

**程式碼中的所有符號都必須是半形的。** 程式語言只認識半形符號，全形符號會直接導致語法錯誤。

對比一下（仔細看括號和引號的樣子）：

```text
✅ print("你好")        ← 半形括號、半形引號
❌ print（"你好"）      ← 全形括號
❌ print(“你好”)        ← 全形（彎）引號
❌ print("你好")；      ← 全形分號
```

### 怎麼避免？

1. **寫程式碼時切換到英文輸入狀態**：大多數中文輸入法按 <kbd>Shift</kbd> 鍵就能切換中/英文。需要在引號裡輸入中文時再切回來。
2. **把輸入法設定成「中文狀態下使用英文標點」**：搜狗、微軟拼音、百度等輸入法的設定裡都有這一項（微軟拼音：設定 → 常規 → 「中文輸入時使用英文標點」）。強烈推薦開啟！
3. **用等寬字型看程式碼**：本課程的編輯器用的就是等寬字型，全形符號明顯更寬，更容易發現。
4. 執行出錯時，看學習程式給出的中文解釋——它能識別全形符號錯誤。

> [!IMPORTANT]
> 中文**可以**出現在兩個地方：引號裡的文字（如 `"你好"`）和註釋（`# 這是註釋`）。除此之外，程式碼裡的一切——關鍵字、變數名、符號——都用英文半形。

## 2.6 程式設計常用的鍵磁碟機代號號

程式設計會用到很多平時不常用的符號。認識它們的名字，看英文資料時也會方便很多：

| 符號 | 中文名 | 英文名 | 輸入方法 |
|------|--------|--------|----------|
| `( )` | 圓括號 / 小括號 | parentheses | <kbd>Shift</kbd> + <kbd>9</kbd> / <kbd>0</kbd> |
| `[ ]` | 方括號 / 中括號 | square brackets | 回車鍵上方附近的 <kbd>[</kbd> <kbd>]</kbd> |
| `{ }` | 花括號 / 大括號 | curly braces | <kbd>Shift</kbd> + <kbd>[</kbd> / <kbd>]</kbd> |
| `< >` | 尖括號 | angle brackets | <kbd>Shift</kbd> + <kbd>,</kbd> / <kbd>.</kbd> |
| `"` | 雙引號 | double quote | <kbd>Shift</kbd> + <kbd>'</kbd> |
| `'` | 單引號 | single quote | 回車鍵左邊的 <kbd>'</kbd> |
| `` ` `` | 反引號 | backtick | <kbd>Esc</kbd> 下方、<kbd>1</kbd> 左邊 |
| `;` | 分號 | semicolon | <kbd>L</kbd> 右邊 |
| `:` | 冒號 | colon | <kbd>Shift</kbd> + <kbd>;</kbd> |
| `=` | 等號 | equals sign | <kbd>Backspace</kbd> 左邊 |
| `_` | 下劃線 | underscore | <kbd>Shift</kbd> + <kbd>-</kbd> |
| `#` | 井號 | hash / pound | <kbd>Shift</kbd> + <kbd>3</kbd> |
| `\` | 反斜槓 | backslash | 回車鍵上方 |
| `/` | 斜槓 | slash | 右 <kbd>Shift</kbd> 左邊 |
| `\|` | 豎線 | pipe / vertical bar | <kbd>Shift</kbd> + <kbd>\</kbd> |
| `&` | 和號 | ampersand | <kbd>Shift</kbd> + <kbd>7</kbd> |
| `*` | 星號 | asterisk | <kbd>Shift</kbd> + <kbd>8</kbd> |
| `%` | 百分號 | percent | <kbd>Shift</kbd> + <kbd>5</kbd> |
| `~` | 波浪號 | tilde | <kbd>Shift</kbd> + <kbd>`</kbd> |
| `^` | 脫字元 | caret | <kbd>Shift</kbd> + <kbd>6</kbd> |

（不同鍵盤佈局可能略有不同。）

## 2.7 必會的快捷鍵

熟練使用快捷鍵能讓寫程式碼快很多。Windows/Linux 用 <kbd>Ctrl</kbd>，macOS 用 <kbd>⌘ Command</kbd>：

| 功能 | Windows / Linux | macOS |
|------|-----------------|-------|
| 複製 | <kbd>Ctrl</kbd>+<kbd>C</kbd> | <kbd>⌘</kbd>+<kbd>C</kbd> |
| 貼上 | <kbd>Ctrl</kbd>+<kbd>V</kbd> | <kbd>⌘</kbd>+<kbd>V</kbd> |
| 剪下 | <kbd>Ctrl</kbd>+<kbd>X</kbd> | <kbd>⌘</kbd>+<kbd>X</kbd> |
| 撤銷（後悔藥） | <kbd>Ctrl</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Z</kbd> |
| 重做 | <kbd>Ctrl</kbd>+<kbd>Y</kbd> 或 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> |
| 全選 | <kbd>Ctrl</kbd>+<kbd>A</kbd> | <kbd>⌘</kbd>+<kbd>A</kbd> |
| 儲存 | <kbd>Ctrl</kbd>+<kbd>S</kbd> | <kbd>⌘</kbd>+<kbd>S</kbd> |
| 查詢 | <kbd>Ctrl</kbd>+<kbd>F</kbd> | <kbd>⌘</kbd>+<kbd>F</kbd> |
| 註釋/取消註釋 | <kbd>Ctrl</kbd>+<kbd>/</kbd> | <kbd>⌘</kbd>+<kbd>/</kbd> |
| 增加縮排 / 減少縮排 | <kbd>Tab</kbd> / <kbd>Shift</kbd>+<kbd>Tab</kbd> | 同左 |

在本課程的學習程式裡，還有：<kbd>F5</kbd> 執行，<kbd>Ctrl</kbd>+<kbd>Enter</kbd> 檢查答案，<kbd>Alt</kbd>+<kbd>←</kbd>/<kbd>→</kbd> 上一課/下一課。

## 2.8 試一試

下面的程式列印出一些程式設計常用符號。執行它，然後試著**自己在編輯器裡重新敲一遍**（不要複製貼上），練習輸入這些符號：

```python
print("圓括號 ( )  方括號 [ ]  花括號 { }")
print("引號 \" '  分號 ;  冒號 :  等號 =")
print("路徑範例：C:\\Users\\code\\hello.py")
```

注意第三行：要在字串裡寫一個反斜槓 `\`，需要寫成兩個 `\\`（第 1 課的「跳脫字元」）。

## 2.9 小結

- 資料存在**檔案**裡，檔案放在**資料夾**裡，形成樹狀結構。
- **副檔名**表示檔案型別；一定要開啟「顯示副檔名」，避免 `hello.py.txt` 問題。
- 程式碼是**純文字**，用程式碼編輯器寫，不要用 Word；編碼選 **UTF-8**。
- **路徑**是檔案的地址，有絕對路徑和相對路徑；Windows 用 `\`，macOS/Linux 用 `/`。
- 程式碼裡的符號必須是**半形**的；中文只能出現在引號裡和註釋裡。
- 記住常用符號的名字和快捷鍵，寫程式碼會順手很多。

## 2.10 練習

<!-- exercise: extension -->
### 練習 1：副檔名

一個 Python 原始碼檔案，副檔名通常是什麼？

- `.exe`
- `.txt`
- `.py`
- `.docx`

<details>
<summary>答案與解析</summary>

`.py`。`.exe` 是 Windows 程式，`.txt` 是普通文字，`.docx` 是 Word 文件。
</details>

<!-- exercise: half-width -->
### 練習 2：找出能正常執行的程式碼

下面哪一行**全部使用了半形符號**，可以正常執行？

- `print（"OK"）`
- `print(“OK”)`
- `print("OK")`
- `print("OK")；`

<details>
<summary>答案與解析</summary>

第三個。第一個用了全形括號，第二個用了全形彎引號，第四個末尾多了一個全形分號。
</details>

<!-- exercise: symbols -->
### 練習 3：符號練習

自己動手輸入（不要複製），編寫程式輸出下面這一行：

```text
(a + b) * [c - d] / {e}
```

<details>
<summary>提示</summary>

整行放在一對雙引號裡交給 `print`。檢查每一個括號是不是半形的。
</details>

<!-- exercise: backslash -->
### 練習 4：列印 Windows 路徑

編寫程式輸出：

```text
C:\code\hello.py
```

<details>
<summary>提示</summary>

字串裡的 `\` 要寫成 `\\`，例如 `"C:\\code"`。
</details>

## 2.11 延伸閱讀

- [微軟：在 Windows 中顯示副檔名](https://support.microsoft.com/zh-cn/windows)（在支援頁面搜尋「副檔名」）
- [Missing Semester 中文版](https://missing-semester-cn.github.io/)：MIT 的「計算機教育中缺失的一課」，講 Shell、編輯器、Git 等程式設計師必備的工具使用技巧。
- 下一課：[終端（命令列）入門](../03-terminal/zh-CN.md)
