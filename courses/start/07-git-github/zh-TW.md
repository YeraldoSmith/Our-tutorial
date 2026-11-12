# 7. Git 與 GitHub 入門

> 🎯 **本課目標**：理解「版本控制」解決什麼問題；安裝並配置 Git；學會 `init`、`status`、`add`、`commit`、`log`、`diff`；瞭解分支；用 GitHub 下載（clone）和更新（pull）本教程，並把自己的程式碼推送（push）上去。
>
> ⏱ **預計用時**：50 分鐘　｜　📋 **前置知識**：第 3 課（終端）

## 7.1 為什麼需要版本控制？

你一定見過這樣的資料夾：

```text
畢業論文.docx
畢業論文_修改版.docx
畢業論文_修改版2.docx
畢業論文_最終版.docx
畢業論文_最終版_真的最終版.docx
```

寫程式碼時這個問題更嚴重：改了一處，結果別的地方壞了，想退回去卻記不清改了什麼；多人一起寫同一個專案，互相覆蓋對方的修改……

**版本控制系統**就是用來解決這些問題的：它像一臺「時光機」，記錄下每一次修改——誰、在什麼時候、改了什麼、為什麼改。隨時可以檢視歷史、對比差異、退回任何一個版本，多人協作也不會互相覆蓋。

**Git** 是目前全世界使用最廣泛的版本控制系統，由 Linux 的作者 Linus Torvalds 在 2005 年建立。**GitHub** 是一個託管 Git 倉庫的網站（還有 GitLab、Gitee 等類似網站），本教程就放在 GitHub 上。

> [!NOTE]
> 簡單區分：**Git 是工具**（裝在你電腦上的軟體），**GitHub 是網站**（存放和分享倉庫的地方）。

## 7.2 安裝和配置 Git

| 系統 | 命令 |
|------|------|
| Windows | `winget install -e --id Git.Git`（或到 <https://git-scm.com/> 下載，安裝選項保持預設即可） |
| macOS | `xcode-select --install` 已經包含 git；也可以 `brew install git` |
| Ubuntu/Debian | `sudo apt install git` |

驗證：

```console
git --version
```

第一次使用前，告訴 Git 你是誰（這些資訊會記錄在每次提交裡）：

```console
git config --global user.name "你的名字"
git config --global user.email "你的郵箱@example.com"
git config --global init.defaultBranch main
```

## 7.3 核心概念：倉庫、暫存、提交

```text
  工作區                暫存區               倉庫（歷史記錄）
 (你正在改的檔案)  ──git add──►  (準備提交的內容) ──git commit──►  (一張張「快照」)
```

- **倉庫**（repository，簡稱 repo）：一個被 Git 管理的專案資料夾。裡面有一個隱藏的 `.git` 資料夾，儲存全部歷史。
- **提交**（commit）：專案在某一時刻的一張**快照**，附帶一句說明（提交資訊）。
- **暫存區**（staging area）：選擇「這次要提交哪些修改」的地方。可以只提交一部分修改。

## 7.4 動手：第一個倉庫

在終端裡跟著做（每行一條命令）：

```console
mkdir my-first-repo
cd my-first-repo
git init
```

`git init` 把當前資料夾變成一個 Git 倉庫。現在用編輯器在裡面新建一個檔案 `hello.py`，內容是：

```python
print("第一個版本")
```

然後檢視狀態：

```console
git status
```

Git 會告訴你：有一個**未跟蹤**（untracked）的檔案 `hello.py`。把它加入暫存區並提交：

```console
git add hello.py
git commit -m "新增 hello.py"
```

`-m` 後面是提交資訊，說明這次做了什麼。好的提交資訊要**簡短、具體**，例如「修復除以零的錯誤」「新增登入頁面」，而不是「改了點東西」。

### 再改一次

把 `hello.py` 改成：

```python
print("第二個版本")
print("學會了 Git！")
```

檢視具體改了什麼：

```console
git diff
```

輸出中以 `-` 開頭的紅色行是刪掉的內容，`+` 開頭的綠色行是新增的內容。確認無誤後提交：

```console
git add hello.py
git commit -m "更新問候語"
```

> [!TIP]
> `git add .` 可以一次把當前資料夾裡所有修改加入暫存區（`.` 表示「當前資料夾」）。

### 檢視歷史

```console
git log --oneline
```

```text
a1b2c3d (HEAD -> main) 更新問候語
9f8e7d6 新增 hello.py
```

每次提交都有一個唯一的編號（雜湊值，如 `a1b2c3d`）。想看某個舊版本的檔案內容，可以：

```console
git show 9f8e7d6:hello.py
```

想撤銷工作區裡還沒提交的修改，恢復成上次提交的樣子：

```console
git restore hello.py
```

## 7.5 .gitignore：不想被記錄的檔案

有些檔案不應該放進倉庫：編譯生成的程式、臨時檔案、密碼和金鑰等。在倉庫根目錄新建一個名為 `.gitignore` 的文字檔案，每行寫一個規則：

```text
# 編譯產物
*.exe
*.o
build/
# Python 快取
__pycache__/
# 虛擬環境
.venv/
# 金鑰——絕對不要提交！
.env
```

> [!WARNING]
> **永遠不要把密碼、API 金鑰提交到 GitHub。** 即使後來刪掉，它也會留在歷史記錄裡，而且公開倉庫會被自動掃描。萬一提交了，要立刻去對應網站把金鑰作廢、重新生成。

## 7.6 分支：平行宇宙

**分支**（branch）讓你在不影響主線的情況下嘗試新功能：

```text
main:      A ── B ── C ─────────── F (合併)
                      \           /
feature:               D ── E ───
```

```console
git switch -c new-feature     # 新建並切換到分支 new-feature
# ……修改、add、commit……
git switch main               # 回到主分支
git merge new-feature         # 把 new-feature 的修改合並進來
```

初學階段只在 `main` 分支上工作也完全可以，知道有這回事就行。

## 7.7 GitHub：下載本教程

### 方法一：下載壓縮包（不需要 Git）

開啟 <https://github.com/YeraldoSmith/Our-tutorial>，點選綠色的 **Code** 按鈕 → **Download ZIP**，解壓即可。缺點：以後更新要重新下載。

### 方法二：git clone（推薦）

```console
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
```

`clone`（克隆）會把整個倉庫連同歷史下載到本地。以後教程更新了，只需要在這個資料夾裡執行：

```console
git pull
```

就能獲取最新內容——學習程式從這個資料夾讀取課程，所以課程也會同步更新。

## 7.8 GitHub：上傳你自己的程式碼

1. 註冊 GitHub 賬號：<https://github.com/signup>。
2. 點右上角 **+** → **New repository**，填寫倉庫名（如 `my-first-repo`），點 **Create repository**。
3. 按頁面提示，在本地倉庫裡執行：

   ```console
   git remote add origin https://github.com/你的使用者名稱/my-first-repo.git
   git push -u origin main
   ```

4. 第一次推送需要登入。GitHub 不再支援用賬號密碼推送，推薦安裝 **GitHub CLI**（`winget install -e --id GitHub.cli` 或 `brew install gh`），執行 `gh auth login` 按提示在瀏覽器裡登入；Windows 上安裝 Git 時自帶的 Git Credential Manager 也會彈出瀏覽器登入視窗。

以後每次修改後：`git add .` → `git commit -m "說明"` → `git push`。

### 參與開源：Fork 和 Pull Request

想給別人的專案（比如本教程）修改錯別字？

1. 在專案頁面點 **Fork**，複製一份到你的賬號下；
2. 在你的副本里修改並提交；
3. 點 **Contribute → Open pull request**，請原作者合併你的修改。

這就是開源協作的基本流程。

## 7.9 常用命令速查

| 命令 | 作用 |
|------|------|
| `git init` | 把當前資料夾變成倉庫 |
| `git clone 網址` | 下載一個遠端倉庫 |
| `git status` | 檢視當前狀態（改了哪些檔案） |
| `git add 檔案` / `git add .` | 把修改加入暫存區 |
| `git commit -m "說明"` | 提交 |
| `git log --oneline` | 檢視歷史 |
| `git diff` | 檢視尚未暫存的修改 |
| `git restore 檔案` | 撤銷工作區的修改 |
| `git switch -c 分支名` | 新建並切換分支 |
| `git pull` | 從遠端獲取併合並最新內容 |
| `git push` | 把本地提交推送到遠端 |

## 7.10 小結

- 版本控制像時光機：記錄每次修改，可以檢視、對比、回退，支援多人協作。
- **Git** 是工具，**GitHub** 是託管倉庫的網站。
- 基本流程：修改 → `git add` → `git commit -m "..."`；用 `git status`、`git diff`、`git log` 檢視情況。
- `.gitignore` 排除不需要的檔案；**絕不提交密碼和金鑰**。
- `git clone` 下載倉庫，`git pull` 更新，`git push` 上傳。

## 7.11 練習

<!-- exercise: commit-meaning -->
### 練習 1：提交是什麼？

在 Git 中，一次 **commit（提交）** 是什麼？

- 把檔案上傳到 GitHub
- 專案在某一時刻的快照，附帶一句說明
- 刪除一個檔案
- 新建一個資料夾

<details>
<summary>答案與解析</summary>

提交是儲存在本地倉庫裡的一張快照。上傳到 GitHub 是 `git push` 的工作。
</details>

<!-- exercise: clone -->
### 練習 2：下載倉庫

想把 GitHub 上的一個倉庫連同全部歷史下載到自己電腦上，用哪個命令？

- `git init`
- `git clone`
- `git commit`
- `git log`

<details>
<summary>答案與解析</summary>

`git clone 網址`。下載完成後，用 `git pull` 獲取以後的更新。
</details>

<!-- exercise: secret -->
### 練習 3：哪些檔案不該提交？

下面哪個檔案**最不應該**提交到公開的 GitHub 倉庫？

- `README.md`
- `main.py`
- 存放 API 金鑰的 `.env`
- `.gitignore`

<details>
<summary>答案與解析</summary>

`.env` 這類存放密碼和金鑰的檔案應該寫進 `.gitignore`。一旦公開，別人就能用你的金鑰。
</details>

<!-- exercise: workflow -->
### 練習 4：日常流程

編寫程式，按順序輸出修改程式碼後上傳到 GitHub 的三條命令（提交資訊寫 `update`）：

```text
git add .
git commit -m "update"
git push
```

<details>
<summary>提示</summary>

第二行裡有雙引號。可以用單引號包住整個字串：`print('git commit -m "update"')`。
</details>

## 7.12 延伸閱讀

- [Pro Git（中文版）](https://git-scm.com/book/zh/v2)：Git 官方免費書，前三章足夠日常使用。
- [Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)：用動畫和小遊戲學習分支，強烈推薦。
- [GitHub 文件：Hello World](https://docs.github.com/zh/get-started/start-your-journey/hello-world)
- 下一課：[怎樣學好程式設計](../08-how-to-learn/zh-CN.md)
