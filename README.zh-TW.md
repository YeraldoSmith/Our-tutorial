# Our Tutorial · 從零學程式設計

[简体中文](README.md) · **繁體中文** · [English](README.en.md) · [Русский](README.ru.md)

一套**完全從零開始**、多語言、可以邊學邊練的程式設計教程，外加一個跨平臺的學習程式。

- 📚 **9 門課程**：程式設計起步、Python、C、C++、Java、Rust、Go、JavaScript/TypeScript、彙編（x86-64）
- 🌏 **4 種語言的課文**：簡體中文、繁體中文、English、Русский；學習程式介面還支援日本語、한국어、Español、Français、Deutsch
- ✍️ **每課都有練習**：選擇題、「讀程式碼寫輸出」、程式設計題；程式設計題由程式**自動判題**
- 💬 **錯誤資訊「翻譯」**：編譯器和直譯器的英文報錯，會配上你的語言寫的白話解釋（例如「你用了中文括號」）
- 🧪 **C/C++ 記憶體檢查**：在「工具 → 設定…」裡開啟後，陣列越界、使用已釋放的記憶體、記憶體洩漏等錯誤會被當場發現，並指出出錯的行（基於 AddressSanitizer；Windows 上的 MinGW 不支援）
- 🖥️ **獨立學習程式**（不是網頁）：Windows、macOS、Linux 都能用；讀課文、寫程式碼、執行、做題在一個視窗裡完成
- 🆕 **內容是現代的**：Python 3.14 與 uv、C23、C++23、Java 25（LTS）、Rust 2024、Go 1.2x、Node.js 直接執行 TypeScript……

## 從哪裡開始？

| 我是…… | 建議 |
|--------|------|
| 完全沒接觸過程式設計 | 先學 **[程式設計起步](courses/start/)**，再學 **[Python](courses/python/)** |
| 想做網站 / 前端 | 程式設計起步 → **[JavaScript 與 TypeScript](courses/javascript/)** |
| 想學底層 / 計算機原理 | Python → **[C](courses/c/)** → **[彙編](courses/asm/)** |
| 想做遊戲 / 高效能軟體 | C → **[C++](courses/cpp/)** 或 **[Rust](courses/rust/)** |
| 想做企業後端 / 安卓 | Python → **[Java](courses/java/)** |
| 想做雲服務 / 命令列工具 | Python → **[Go](courses/go/)** |
| 想學人工智慧 / 大模型 | Python → 姊妹倉庫 **[LLM-Tutorial](https://github.com/YeraldoSmith/LLM-Tutorial)** |

## 兩種學習方式

### 方式一：用學習程式（推薦）

1. 開啟本倉庫的 [Releases](https://github.com/YeraldoSmith/Our-tutorial/releases) 頁面，下載對應系統的壓縮包：
   - Windows：`app-windows-x64.zip`，解壓後雙擊 `OurTutorial.exe`
   - macOS（Apple 晶片）：`app-macos-arm64.zip`，解壓後把 `OurTutorial.app` 拖進「應用程式」。第一次開啟如果提示「無法驗證開發者」，請在「系統設定 → 隱私與安全性」裡點「仍要開啟」
   - Linux：`app-linux-x64.tar.gz`，解壓後執行 `OurTutorial/OurTutorial`
2. 第一次開啟時選擇介面語言，然後從「程式設計起步」第 1 課開始。
3. 學習程式**自帶 Python**，不用安裝任何東西就能學 Python 課。學其他語言時，點「工具 → 檢查開發工具」，按提示安裝編譯器即可；沒裝之前也可以用「線上執行」。

**從原始碼執行**（需要 Python 3.11+）：

```bash
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
pip install -r app/requirements.txt
python app/run.py
```

用 `git pull` 更新倉庫，學習程式裡的課程就同步更新了；下載版可以點「工具 → 更新課程內容」。

### 方式二：直接在 GitHub 上讀

每一課都是一個資料夾，裡面有各語言的課文：`zh-CN.md`（簡體）、`zh-TW.md`（繁體）、`en.md`、`ru.md`。練習的參考答案和測試資料在同一資料夾的 `exercises.toml` 裡。

## 學習程式介面

```text
┌──────────┬─────────────────────────┬──────────────────────────┐
│ 課程目錄  │  📖 課文  ✍️ 練習        │  程式碼編輯器  [▶ 執行]     │
│ 🚀 程式設計起步│                         │                          │
│ 🐍 Python │  標題、講解、例子……      │  print("你好")            │
│ 🔧 C 語言 │  [▶ 放進編輯器執行]       │                          │
│ ...      │                         ├──────────────────────────┤
│          │  練習 1  [✏️ 在編輯器中做] │  執行結果 / 輸入框         │
│ 已完成 3/115│                       │  💬 這是什麼意思？         │
└──────────┴─────────────────────────┴──────────────────────────┘
```

- **▶ 執行**（<kbd>F5</kbd>）：執行編輯器裡的程式碼；程式需要輸入時，在下面的輸入框裡輸入並按回車。
- **✔ 檢查答案**（<kbd>Ctrl</kbd>+<kbd>Enter</kbd>）：做程式設計題時自動執行全部測試，逐個告訴你哪裡不對。
- **💡 提示 / 檢視參考答案**：卡住時一步步給提示。
- 進度、草稿自動儲存在本機。

## 倉庫結構

```text
tutorial.json                 倉庫資訊與課程順序
courses/<課程>/course.toml     課程標題與簡介
courses/<課程>/<NN-課>/        每一課：zh-CN.md、zh-TW.md、en.md、ru.md、exercises.toml
app/                          學習程式（Python + PySide6/Qt）
tools/check.py                檢查全部課文，並執行每道題的參考答案
tools/zh_tw.py                由簡體中文生成繁體中文（OpenCC + 術語表）
```

## 參與貢獻

歡迎修正錯誤、補充例子、翻譯成更多語言。請先閱讀 [CONTRIBUTING.md](CONTRIBUTING.md)。提交前執行：

```bash
python tools/check.py --examples
```

## 致謝與參考

本教程在編寫時參考了許多優秀的開源專案和官方文件，課文的「延伸閱讀」裡都給出了連結，例如：
[Python 官方教程](https://docs.python.org/zh-cn/3/tutorial/)、[Python-100-Days](https://github.com/jackfrued/Python-100-Days)、
[The Rust Programming Language](https://doc.rust-lang.org/book/)、[Rust 語言聖經](https://course.rs/)、[Comprehensive Rust](https://google.github.io/comprehensive-rust/)、
[A Tour of Go](https://go.dev/tour/)、[MDN Web Docs](https://developer.mozilla.org/)、[cppreference](https://en.cppreference.com/)、
[dev.java](https://dev.java/learn/)、[Compiler Explorer](https://godbolt.org/)、[The Algorithms](https://github.com/TheAlgorithms)、[Exercism](https://exercism.org/)。

## 許可

- 課文、練習等教學內容：[CC BY-SA 4.0](LICENSE-CONTENT.md)（可自由分享和改編，需署名並以相同方式共享）
- 學習程式與工具程式碼：[MIT](LICENSE)
