# 3. 終端（命令列）入門

> 🎯 **本課目標**：開啟終端；看懂「提示符」；學會 `pwd`、`ls`、`cd`、`mkdir` 等基本命令；用終端執行程式；理解 PATH 和「找不到命令」錯誤。
>
> ⏱ **預計用時**：40 分鐘　｜　📋 **前置知識**：第 2 課（檔案、資料夾、路徑）

## 3.1 終端是什麼？

平時我們用滑鼠點圖示來操作電腦，這叫**圖形介面**（GUI）。程式設計師還經常用另一種方式：在一個黑色（或白色）的視窗裡**打字輸入命令**，電腦執行後把結果以文字顯示出來。這個視窗叫**終端**（terminal），在裡面解釋命令的程式叫 **shell**，這種操作方式叫**命令列**（command line）。

為什麼要學它？

- 很多開發工具**只有**命令列版本（編譯器、Git、包管理器……）。
- 安裝開發工具、執行和除錯程式，教程裡給的幾乎都是命令。
- 熟練以後，很多操作比點滑鼠快得多，還能寫成指令碼自動執行。

> [!NOTE]
> 本課程的學習程式可以幫你執行程式碼，所以**沒學會終端也能繼續學下一課**。但遲早要用到它，建議現在花點時間熟悉。

## 3.2 開啟終端

- **Windows 11**：在開始選單搜尋「終端」（Windows Terminal），開啟後預設是 **PowerShell**。Windows 10 可以搜尋「PowerShell」。
- **macOS**：按 <kbd>⌘</kbd> + <kbd>空格</kbd> 開啟「聚焦搜尋」，輸入「終端」或 Terminal，回車。預設 shell 是 **zsh**。
- **Linux**：大多數桌面按 <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>，預設 shell 一般是 **bash**。

開啟後你會看到類似這樣的一行：

```console
PS C:\Users\xiaoming>                 ← Windows PowerShell
xiaoming@MacBook ~ %                  ← macOS zsh
xiaoming@ubuntu:~$                    ← Linux bash
```

這叫**提示符**（prompt），意思是「我準備好了，請輸入命令」。提示符裡通常會顯示**當前所在的資料夾**（`~` 表示你的使用者資料夾）。

> [!IMPORTANT]
> 教程裡的命令前面常帶有 `$` 或 `>`，那是提示符，**不要輸入它**。例如看到 `$ python3 --version`，你只需要輸入 `python3 --version`。

## 3.3 第一批命令

終端始終「站在」某個資料夾裡，這叫**當前目錄**（working directory）。下面這些命令用來檢視和移動位置。

| 作用 | macOS / Linux | Windows PowerShell |
|------|---------------|--------------------|
| 顯示當前在哪個資料夾 | `pwd` | `pwd`（或 `Get-Location`） |
| 列出當前資料夾的內容 | `ls` | `ls`（或 `dir`） |
| 進入某個資料夾 | `cd 資料夾名` | `cd 資料夾名` |
| 回到上一層 | `cd ..` | `cd ..` |
| 回到使用者資料夾 | `cd ~` 或 `cd` | `cd ~` |
| 新建資料夾 | `mkdir 名字` | `mkdir 名字` |
| 清屏 | `clear` | `cls`（或 `clear`） |
| 顯示檔案內容 | `cat 檔名` | `cat 檔名`（或 `type`） |
| 顯示一段文字 | `echo 你好` | `echo 你好` |

好訊息：PowerShell 為了方便，給很多命令起了和 macOS/Linux 一樣的別名，所以上面大部分命令在三個系統上都能用。

### 跟著做一遍

在終端裡依次輸入下面每一行，每行輸入後按回車，觀察結果：

```console
pwd
ls
mkdir code
cd code
pwd
cd ..
pwd
```

你會看到：先顯示使用者資料夾的路徑；列出裡面的檔案；新建 `code` 資料夾；進入它（`pwd` 顯示的路徑末尾多了 `code`）；再退回上一層。

> [!TIP]
> 如果 `mkdir code` 提示資料夾已經存在，說明你在第 2 課已經建好了，直接 `cd code` 就行。

## 3.4 兩個超級省力的技巧

1. **<kbd>Tab</kbd> 自動補全**：輸入資料夾名的前幾個字母，按 <kbd>Tab</kbd>，終端會自動補全剩下的部分。比如輸入 `cd Doc` 再按 <kbd>Tab</kbd>，會變成 `cd Documents`。既省時間又不會拼錯。
2. **<kbd>↑</kbd> 方向鍵調出歷史命令**：按上箭頭可以找回之前輸入過的命令，修改一下再回車，不用重新打。

還有一個**救命鍵**：程式卡住了、執行停不下來（比如寫了死迴圈），按 <kbd>Ctrl</kbd> + <kbd>C</kbd> 強制停止。注意在終端裡 <kbd>Ctrl</kbd> + <kbd>C</kbd> **不是複製**！（終端裡複製一般用 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>，macOS 是 <kbd>⌘</kbd> + <kbd>C</kbd>。）

## 3.5 用終端執行程式

輸入一個程式的名字，就能執行它。先檢查一下電腦上有沒有 Python：

```console
python3 --version
```

Windows 上通常用：

```console
python --version
```

或者 `py --version`。如果看到 `Python 3.13.x` 之類的版本號，說明已經安裝好了。（還沒安裝也沒關係，下一課會講。）

### 執行一個 Python 檔案

1. 用編輯器（例如學習程式的「檔案 → 另存為」）把下面的程式碼儲存為 `code` 資料夾裡的 `hello.py`：

   ```python
   print("我在終端裡執行了這個程式！")
   ```

2. 在終端裡進入這個資料夾，執行它：

   ```console
   cd ~/Documents/code
   python3 hello.py
   ```

   Windows 上：

   ```console
   cd ~\Documents\code
   python hello.py
   ```

3. 終端會顯示：`我在終端裡執行了這個程式！`

命令的一般格式是：**程式名 引數1 引數2 ……**。`python3 hello.py` 的意思就是「執行 python3 這個程式，交給它的引數是 hello.py」。像 `--version` 這樣以短橫線開頭的引數叫**選項**（option），用來調整程式的行為。

## 3.6 PATH：終端去哪裡找程式？

你輸入 `python3`，終端怎麼知道 Python 裝在哪裡？答案是一個叫 **PATH** 的**環境變數**：它是一串資料夾路徑，終端會按順序在這些資料夾裡找名為 `python3` 的程式。

```text
PATH = C:\Windows\System32 ; C:\Python313 ; C:\Users\xiaoming\.cargo\bin ; ...
             │                    │                     │
           找 python？沒有      找到了！執行它          （後面的不再找）
```

如果所有資料夾裡都找不到，就會報錯：

```console
python3: command not found                                   ← macOS / Linux
python3 : 無法將“python3”項識別為 cmdlet、函式、指令碼檔案或可執行程式的名稱。   ← Windows
```

這類錯誤的意思都是「**找不到這個命令**」。常見原因：

1. **沒有安裝**這個程式。
2. 安裝了，但**沒有加入 PATH**（Windows 安裝 Python 時要勾選「Add python.exe to PATH」）。
3. 剛安裝完，**終端沒重啟**——PATH 只在終端啟動時讀取一次。關掉終端重新開啟試試。
4. **名字打錯了**，或者這個系統上的名字不一樣（例如 Windows 上叫 `python` 而不是 `python3`）。

想看看 PATH 裡有什麼：

```console
echo $PATH          # macOS / Linux
echo $env:PATH      # Windows PowerShell
```

## 3.7 小心危險命令

終端裡的命令**沒有回收站**。下面這些命令要格外小心，在沒完全理解之前不要執行：

- `rm`（刪除檔案）、`rm -r`（刪除資料夾及其中所有內容）、Windows 的 `del`、`Remove-Item`
- 任何以 `sudo` 開頭的命令（以管理員許可權執行）
- 從網上覆制來的、你看不懂的長命令

> [!WARNING]
> 永遠不要執行別人發給你、你又不理解的命令，尤其是包含 `rm -rf`、`format`、`curl ... | sh` 的命令。安裝正規軟體時，只使用官方網站給出的命令。

## 3.8 小結

- **終端**是輸入命令的視窗，**shell** 負責理解命令；提示符表示「等待輸入」，教程中命令前的 `$` 不要輸入。
- 基本命令：`pwd` 看位置、`ls` 看內容、`cd` 進入、`cd ..` 返回、`mkdir` 新建、`clear` 清屏。
- <kbd>Tab</kbd> 補全、<kbd>↑</kbd> 歷史命令、<kbd>Ctrl</kbd>+<kbd>C</kbd> 停止程式。
- 執行程式：`程式名 引數`，例如 `python3 hello.py`。
- 「找不到命令」通常是沒安裝、沒加入 PATH、終端沒重啟或名字打錯。
- 刪除類命令沒有回收站，不懂的命令不要執行。

## 3.9 練習

<!-- exercise: cd-up -->
### 練習 1：回到上一層

你在終端裡的當前目錄是 `/home/xiaoming/code/game`，想回到 `/home/xiaoming/code`，應該輸入哪條命令？

- `cd game`
- `cd ..`
- `mkdir ..`
- `ls ..`

<details>
<summary>答案與解析</summary>

`cd ..`：`..` 代表上一層資料夾。`ls ..` 只是列出上一層的內容，並不會移動過去。
</details>

<!-- exercise: stop -->
### 練習 2：程式停不下來

在終端裡執行的程式陷入了死迴圈，一直停不下來。按什麼鍵可以強制停止它？

- <kbd>Ctrl</kbd> + <kbd>S</kbd>
- <kbd>Ctrl</kbd> + <kbd>C</kbd>
- <kbd>Tab</kbd>
- <kbd>↑</kbd>

<details>
<summary>答案與解析</summary>

<kbd>Ctrl</kbd> + <kbd>C</kbd>。在終端裡它的作用是「中斷」，不是複製。
</details>

<!-- exercise: not-found -->
### 練習 3：找不到命令

剛剛安裝完 Python，在**之前就開啟著的**終端裡輸入 `python --version`，卻提示找不到命令。最可能先嚐試的做法是？

- 重灌系統
- 關掉終端，重新開啟一個再試
- 把命令改成 `PYTHON --VERSION`
- 刪除 Python 資料夾

<details>
<summary>答案與解析</summary>

終端只在啟動時讀取一次 PATH。安裝程式修改了 PATH 之後，需要新開一個終端才能生效。如果還不行，再檢查安裝時是否勾選了「加入 PATH」。
</details>

<!-- exercise: commands -->
### 練習 4：寫出命令

我們用 Python 來「列印」一組終端命令。編寫程式，輸出下面三行——它們依次表示：新建 `project` 資料夾、進入它、顯示當前位置：

```text
mkdir project
cd project
pwd
```

<details>
<summary>提示</summary>

用三個 `print`，每個列印一行文字。
</details>

## 3.10 延伸閱讀

- [Missing Semester 中文版 · 課程概覽與 shell](https://missing-semester-cn.github.io/2020/course-shell/)
- [微軟 PowerShell 文件](https://learn.microsoft.com/zh-cn/powershell/scripting/learn/ps101/01-getting-started)
- 下一課：[安裝開發工具](../04-install-tools/zh-CN.md)
