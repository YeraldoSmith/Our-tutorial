# 4. 安裝開發工具

> 🎯 **本課目標**：選好程式碼編輯器；在 Windows / macOS / Linux 上安裝 Python、C/C++、Java、Rust、Go、Node.js；驗證安裝是否成功；學會處理最常見的安裝問題。
>
> ⏱ **預計用時**：每種工具 10～20 分鐘（**用到哪門語言再裝哪門**，不必一次裝完）　｜　📋 **前置知識**：第 3 課（終端）

> [!TIP]
> 本課程的學習程式**自帶 Python**，學 Python 課暫時不用安裝任何東西。其他語言可以先用學習程式的「線上執行」體驗，等確定要深入學習時再安裝。點選選單「工具 → 檢查開發工具」可以隨時檢視本機裝了什麼。

## 4.1 先認識幾個概念

- **編輯器**（editor）：寫程式碼的軟體。
- **編譯器 / 直譯器**：把程式碼變成可執行程式的工具（第 1 課）。每門語言都有自己的。
- **SDK / 工具鏈**（toolchain）：編譯器 + 常用工具 + 標準庫的「全家桶」。比如 Java 的 JDK、Rust 的 rustup。
- **包管理器**（package manager）：一鍵安裝軟體的工具。Windows 有 **winget**，macOS 有 **Homebrew**，Linux 有 **apt**（Ubuntu/Debian）、**dnf**（Fedora）、**pacman**（Arch）。用它們安裝比去網站下載安裝包更省事，也方便以後升級。

## 4.2 程式碼編輯器：推薦 VS Code

初學階段用本課程的學習程式就夠了。以後寫更大的專案，推薦 **Visual Studio Code**（簡稱 VS Code）：免費、跨平臺、支援所有主流語言。

- 下載：<https://code.visualstudio.com/>（選擇你的系統，下載後按預設選項安裝）
- Windows 也可以在終端執行：`winget install -e --id Microsoft.VisualStudioCode`
- macOS：`brew install --cask visual-studio-code`
- **中文介面**：開啟 VS Code → 左側「擴充套件」圖示 → 搜尋 `Chinese (Simplified)` → 安裝 → 重啟。
- 寫哪門語言，就裝哪門語言的擴充套件：Python、C/C++、Extension Pack for Java、rust-analyzer、Go。

其他選擇：JetBrains 系列 IDE（PyCharm、IntelliJ IDEA、CLion、RustRover、GoLand，大多有免費版或教育版）、Zed、Vim/Neovim 等。

## 4.3 Windows：先確認 winget 可用

Windows 10（較新版本）和 Windows 11 自帶 **winget**。開啟終端輸入：

```console
winget --version
```

如果顯示版本號就可以了。如果提示找不到，到微軟商店搜尋並安裝「應用安裝程式」（App Installer）。

下面每個工具都給出 **winget 命令**（最省事）和**官網下載**兩種方式，任選其一。

> [!NOTE]
> 用 winget 安裝時可能會彈出「是否允許此應用更改裝置」的視窗，點「是」。安裝完成後**關閉終端再重新開啟**，讓新的 PATH 生效。

## 4.4 macOS：先安裝 Homebrew 和命令列工具

1. 安裝 Xcode 命令列工具（包含 C/C++ 編譯器 clang、git 等）：

   ```console
   xcode-select --install
   ```

   會彈出一個視窗，點「安裝」，等待完成。

2. 安裝 Homebrew（macOS 最常用的包管理器）。開啟 <https://brew.sh/>，複製首頁上那條安裝命令到終端執行（只使用官網的命令）。安裝完成後，按螢幕最後的提示（"Next steps"）執行兩條命令把 brew 加入 PATH。

3. 驗證：

   ```console
   brew --version
   ```

## 4.5 Linux：使用發行版的包管理器

以 Ubuntu / Debian 為例，先更新軟體列表：

```console
sudo apt update
```

`sudo` 會要求輸入你的登入密碼（輸入時螢幕上**不顯示任何字元**，這是正常的，輸完按回車）。Fedora 用 `sudo dnf install ...`，Arch 用 `sudo pacman -S ...`。

## 4.6 安裝 Python

| 系統 | 命令 |
|------|------|
| Windows | `winget install -e --id Python.Python.3.13` |
| macOS | `brew install python` |
| Ubuntu/Debian | `sudo apt install python3 python3-pip python3-venv` |
| Fedora | `sudo dnf install python3` |

也可以到 <https://www.python.org/downloads/> 下載安裝包。

> [!WARNING]
> Windows 用安裝包安裝時，第一個介面**一定要勾選「Add python.exe to PATH」**，否則終端裡找不到 python。

驗證（Windows 用 `python`，macOS/Linux 用 `python3`）：

```console
python3 --version
```

> [!TIP]
> 推薦再安裝 **uv**——一個非常快的現代 Python 包與專案管理工具（Python 課第 15 課會詳細講）：Windows `winget install -e --id astral-sh.uv`，macOS `brew install uv`，Linux `curl -LsSf https://astral.sh/uv/install.sh | sh`。

## 4.7 安裝 C / C++ 編譯器

| 系統 | 做法 |
|------|------|
| Windows | `winget install -e --id BrechtSanders.WinLibs.POSIX.UCRT`（WinLibs：GCC + MinGW-w64） |
| macOS | `xcode-select --install`（4.4 節已經裝過） |
| Ubuntu/Debian | `sudo apt install build-essential gdb` |
| Fedora | `sudo dnf install gcc gcc-c++ gdb` |

Windows 的其他選擇：

- **MSYS2**（<https://www.msys2.org/>）：安裝後在「MSYS2 UCRT64」終端執行 `pacman -S mingw-w64-ucrt-x86_64-gcc`，再把 `C:\msys64\ucrt64\bin` 加入 PATH。
- **Visual Studio 2022 社群版**：微軟官方 IDE，自帶 MSVC 編譯器，適合以後做 Windows 程式。本課程的學習程式使用 GCC/Clang 風格的命令，所以入門推薦 WinLibs 或 MSYS2。

驗證：

```console
gcc --version
g++ --version
```

macOS 上 `gcc` 實際上是 Apple Clang，這完全沒問題。

## 4.8 安裝 Java（JDK 25）

Java 課程基於 **JDK 25**（2025 年 9 月釋出的長期支援版）。推薦 Eclipse Temurin 發行版：

| 系統 | 命令 |
|------|------|
| Windows | `winget install -e --id EclipseAdoptium.Temurin.25.JDK` |
| macOS | `brew install --cask temurin` |
| Ubuntu 24.04+ / Debian 13+ | `sudo apt install openjdk-25-jdk` |
| Fedora | `sudo dnf install java-25-openjdk-devel` |

也可以到 <https://adoptium.net/> 下載安裝包。驗證：

```console
java -version
```

應該看到 `25` 開頭的版本號。如果顯示的是舊版本（例如 1.8、17），說明電腦上還裝著舊 Java 並且排在 PATH 前面——可以解除安裝舊版本，或調整 PATH 順序。

## 4.9 安裝 Rust

Rust 官方用 **rustup** 管理工具鏈：

- **Windows**：`winget install -e --id Rustlang.Rustup`，或從 <https://rustup.rs/> 下載 `rustup-init.exe`。執行時如果提示需要 **Visual Studio C++ 生成工具**，選擇第 1 項讓它自動安裝（Rust 在 Windows 上需要它來連結程式）。
- **macOS / Linux**：

  ```console
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  出現選項時直接回車（預設安裝）。完成後重新開啟終端。

驗證：

```console
rustc --version
cargo --version
```

以後升級 Rust 只需要：`rustup update`。

## 4.10 安裝 Go

| 系統 | 命令 |
|------|------|
| Windows | `winget install -e --id GoLang.Go` |
| macOS | `brew install go` |
| Linux | 到 <https://go.dev/dl/> 下載 `.tar.gz`，按頁面說明解壓到 `/usr/local`；或 `sudo snap install go --classic` |

> [!NOTE]
> Ubuntu 的 `apt install golang-go` 裝出來的版本通常比較舊，推薦用官網的安裝包。

驗證：

```console
go version
```

## 4.11 安裝 Node.js（JavaScript / TypeScript）

推薦安裝 **LTS（長期支援）版本**，22 或更新：

| 系統 | 命令 |
|------|------|
| Windows | `winget install -e --id OpenJS.NodeJS.LTS` |
| macOS | `brew install node` |
| Linux | 用版本管理器 fnm：`curl -fsSL https://fnm.vercel.app/install \| bash`，重開終端後 `fnm install --lts` |

也可以到 <https://nodejs.org/> 下載。驗證：

```console
node --version
npm --version
```

Node.js 22.18 及以上版本可以**直接執行 TypeScript 檔案**（`node main.ts`），不用另外安裝編譯器。

## 4.12 彙編課程需要什麼？

彙編課程使用 **Linux x86-64** 環境（GCC 自帶的彙編器）：

- **Linux（x86-64 電腦）**：裝好 4.7 節的 `build-essential` 就行。
- **Windows**：安裝 WSL（Windows 裡的 Linux）。以管理員身份開啟終端執行 `wsl --install`，重啟後會裝好 Ubuntu；在 Ubuntu 裡執行 `sudo apt install build-essential gdb`。
- **macOS**：蘋果電腦現在用的是 ARM 晶片，指令集不同。學習程式裡可以用「線上執行」；也可以用虛擬機器（UTM、OrbStack）執行 Linux。

## 4.13 安裝後檢查清單

全部裝完後，在一個**新開啟**的終端裡依次執行，每條都應該顯示版本號：

```console
python3 --version
gcc --version
java -version
rustc --version
go version
node --version
```

也可以直接在學習程式裡點「工具 → 檢查開發工具」，一眼看清哪些裝好了（✅）、哪些還沒有（❌）。

## 4.14 常見問題

| 現象 | 原因與解決 |
|------|------------|
| `command not found` / 無法將……識別為…… | ① 終端沒重開；② 沒加入 PATH；③ Windows 上 `python3` 應改為 `python` 或 `py` |
| Windows 輸入 `python` 開啟了微軟商店 | 這是系統的「應用執行別名」。安裝真正的 Python 後，到「設定 → 應用 → 高階應用設定 → 應用執行別名」裡關閉 python.exe 和 python3.exe 兩個開關 |
| `permission denied` / 拒絕訪問 | Linux/macOS 安裝系統軟體需要 `sudo`；Windows 以管理員身份執行終端 |
| 下載很慢或失敗 | 換個時間重試；國內網路可以使用 winget、Homebrew、pip 的國內映象（在搜尋引擎搜「pip 清華映象」等） |
| 防毒軟體攔截了編譯出來的程式 | 新編譯出來的 `.exe` 有時會被誤報，可以把你的程式碼資料夾加入防毒軟體的「排除項」 |
| 版本不對（例如 Java 還是 8） | 電腦上有多個版本，PATH 裡舊的排在前面。解除安裝舊版本或調整 PATH |

## 4.15 小結

- 編輯器寫程式碼，編譯器/直譯器執行程式碼；包管理器（winget / Homebrew / apt）讓安裝變簡單。
- **用到哪門語言再裝哪門**；學習程式自帶 Python，其他語言可以先線上執行。
- 每次安裝後**重新開啟終端**，用 `--version` 驗證。
- 彙編課需要 Linux x86-64：Windows 用 WSL，Mac 用線上執行或虛擬機器。

## 4.16 練習

<!-- exercise: verify -->
### 練習 1：如何驗證？

安裝完 Rust 後，用哪條命令檢查是否安裝成功？

- `rust install`
- `rustc --version`
- `cargo delete`
- `rm rust`

<details>
<summary>答案與解析</summary>

`rustc --version` 會顯示 Rust 編譯器的版本號；能顯示就說明安裝成功並且在 PATH 中。
</details>

<!-- exercise: path-checkbox -->
### 練習 2：Windows 安裝 Python

在 Windows 上用安裝包安裝 Python 時，第一個介面最重要的一個選項是什麼？

- 「Install for all users」
- 「Add python.exe to PATH」
- 「Disable path length limit」
- 「Customize installation」

<details>
<summary>答案與解析</summary>

勾選「Add python.exe to PATH」後，終端才能找到 python 命令。忘記勾選的話，可以重新執行安裝包選擇「Modify」，或手動把路徑加入 PATH。
</details>

<!-- exercise: package-manager -->
### 練習 3：包管理器

下面哪一個是 **macOS** 上最常用的包管理器？

- winget
- apt
- Homebrew
- pacman

<details>
<summary>答案與解析</summary>

Homebrew（命令是 `brew`）。winget 屬於 Windows，apt 屬於 Ubuntu/Debian，pacman 屬於 Arch Linux。
</details>

<!-- exercise: checklist -->
### 練習 4：列印檢查清單

編寫程式，輸出下面三行檢查命令（這樣你就有了一份可以照著輸入的清單）：

```text
python3 --version
gcc --version
java -version
```

<details>
<summary>提示</summary>

三個 `print`。注意 `java -version` 只有一個短橫線——這是 Java 的老習慣。
</details>

## 4.17 延伸閱讀

- [VS Code 官方文件（中文）](https://code.visualstudio.com/docs)：介面導覽和各語言的配置指南。
- [Rust 官方安裝說明](https://www.rust-lang.org/zh-CN/tools/install)、[Go 安裝說明](https://go.dev/doc/install)、[Adoptium（Temurin JDK）](https://adoptium.net/zh-CN/)
- 下一課：[讀懂錯誤資訊與除錯](../05-errors-debugging/zh-CN.md)
