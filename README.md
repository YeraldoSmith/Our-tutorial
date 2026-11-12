# Our Tutorial · 从零学编程

**简体中文** · [繁體中文](README.zh-TW.md) · [English](README.en.md) · [Русский](README.ru.md)

一套**完全从零开始**、多语言、可以边学边练的编程教程，外加一个跨平台的学习程序。

- 📚 **9 门课程**：编程起步、Python、C、C++、Java、Rust、Go、JavaScript/TypeScript、汇编（x86-64）
- 🌏 **4 种语言的课文**：简体中文、繁體中文、English、Русский；学习程序界面还支持日本語、한국어、Español、Français、Deutsch
- ✍️ **每课都有练习**：选择题、「读代码写输出」、编程题；编程题由程序**自动判题**
- 💬 **错误信息「翻译」**：编译器和解释器的英文报错，会配上你的语言写的白话解释（例如「你用了中文括号」）
- 🧪 **C/C++ 内存检查**：在「工具 → 设置…」里打开后，数组越界、使用已释放的内存、内存泄漏等错误会被当场发现，并指出出错的行（基于 AddressSanitizer；Windows 上的 MinGW 不支持）
- 🖥️ **独立学习程序**（不是网页）：Windows、macOS、Linux 都能用；读课文、写代码、运行、做题在一个窗口里完成
- 🆕 **内容是现代的**：Python 3.14 与 uv、C23、C++23、Java 25（LTS）、Rust 2024、Go 1.2x、Node.js 直接运行 TypeScript……

## 从哪里开始？

| 我是…… | 建议 |
|--------|------|
| 完全没接触过编程 | 先学 **[编程起步](courses/start/)**，再学 **[Python](courses/python/)** |
| 想做网站 / 前端 | 编程起步 → **[JavaScript 与 TypeScript](courses/javascript/)** |
| 想学底层 / 计算机原理 | Python → **[C](courses/c/)** → **[汇编](courses/asm/)** |
| 想做游戏 / 高性能软件 | C → **[C++](courses/cpp/)** 或 **[Rust](courses/rust/)** |
| 想做企业后端 / 安卓 | Python → **[Java](courses/java/)** |
| 想做云服务 / 命令行工具 | Python → **[Go](courses/go/)** |
| 想学人工智能 / 大模型 | Python → 姊妹仓库 **[LLM-Tutorial](https://github.com/YeraldoSmith/LLM-Tutorial)** |

## 两种学习方式

### 方式一：用学习程序（推荐）

1. 打开本仓库的 [Releases](https://github.com/YeraldoSmith/Our-tutorial/releases) 页面，下载对应系统的压缩包：
   - Windows：`app-windows-x64.zip`，解压后双击 `OurTutorial.exe`
   - macOS（Apple 芯片）：`app-macos-arm64.zip`，解压后把 `OurTutorial.app` 拖进「应用程序」。第一次打开如果提示「无法验证开发者」，请在「系统设置 → 隐私与安全性」里点「仍要打开」
   - Linux：`app-linux-x64.tar.gz`，解压后运行 `OurTutorial/OurTutorial`
2. 第一次打开时选择界面语言，然后从「编程起步」第 1 课开始。
3. 学习程序**自带 Python**，不用安装任何东西就能学 Python 课。学其他语言时，点「工具 → 检查开发工具」，按提示安装编译器即可；没装之前也可以用「在线运行」。

**从源码运行**（需要 Python 3.11+）：

```bash
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
pip install -r app/requirements.txt
python app/run.py
```

用 `git pull` 更新仓库，学习程序里的课程就同步更新了；下载版可以点「工具 → 更新课程内容」。

### 方式二：直接在 GitHub 上读

每一课都是一个文件夹，里面有各语言的课文：`zh-CN.md`（简体）、`zh-TW.md`（繁體）、`en.md`、`ru.md`。练习的参考答案和测试数据在同一文件夹的 `exercises.toml` 里。

## 学习程序界面

```text
┌──────────┬─────────────────────────┬──────────────────────────┐
│ 课程目录  │  📖 课文  ✍️ 练习        │  代码编辑器  [▶ 运行]     │
│ 🚀 编程起步│                         │                          │
│ 🐍 Python │  标题、讲解、例子……      │  print("你好")            │
│ 🔧 C 语言 │  [▶ 放进编辑器运行]       │                          │
│ ...      │                         ├──────────────────────────┤
│          │  练习 1  [✏️ 在编辑器中做] │  运行结果 / 输入框         │
│ 已完成 3/115│                       │  💬 这是什么意思？         │
└──────────┴─────────────────────────┴──────────────────────────┘
```

- **▶ 运行**（<kbd>F5</kbd>）：运行编辑器里的代码；程序需要输入时，在下面的输入框里输入并按回车。
- **✔ 检查答案**（<kbd>Ctrl</kbd>+<kbd>Enter</kbd>）：做编程题时自动运行全部测试，逐个告诉你哪里不对。
- **💡 提示 / 查看参考答案**：卡住时一步步给提示。
- 进度、草稿自动保存在本机。

## 仓库结构

```text
tutorial.json                 仓库信息与课程顺序
courses/<课程>/course.toml     课程标题与简介
courses/<课程>/<NN-课>/        每一课：zh-CN.md、zh-TW.md、en.md、ru.md、exercises.toml
app/                          学习程序（Python + PySide6/Qt）
tools/check.py                检查全部课文，并运行每道题的参考答案
tools/zh_tw.py                由简体中文生成繁體中文（OpenCC + 术语表）
```

## 参与贡献

欢迎修正错误、补充例子、翻译成更多语言。请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。提交前运行：

```bash
python tools/check.py --examples
```

## 致谢与参考

本教程在编写时参考了许多优秀的开源项目和官方文档，课文的「延伸阅读」里都给出了链接，例如：
[Python 官方教程](https://docs.python.org/zh-cn/3/tutorial/)、[Python-100-Days](https://github.com/jackfrued/Python-100-Days)、
[The Rust Programming Language](https://doc.rust-lang.org/book/)、[Rust 语言圣经](https://course.rs/)、[Comprehensive Rust](https://google.github.io/comprehensive-rust/)、
[A Tour of Go](https://go.dev/tour/)、[MDN Web Docs](https://developer.mozilla.org/)、[cppreference](https://en.cppreference.com/)、
[dev.java](https://dev.java/learn/)、[Compiler Explorer](https://godbolt.org/)、[The Algorithms](https://github.com/TheAlgorithms)、[Exercism](https://exercism.org/)。

## 许可

- 课文、练习等教学内容：[CC BY-SA 4.0](LICENSE-CONTENT.md)（可自由分享和改编，需署名并以相同方式共享）
- 学习程序与工具代码：[MIT](LICENSE)
