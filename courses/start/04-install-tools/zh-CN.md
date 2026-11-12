# 4. 安装开发工具

> 🎯 **本课目标**：选好代码编辑器；在 Windows / macOS / Linux 上安装 Python、C/C++、Java、Rust、Go、Node.js；验证安装是否成功；学会处理最常见的安装问题。
>
> ⏱ **预计用时**：每种工具 10～20 分钟（**用到哪门语言再装哪门**，不必一次装完）　｜　📋 **前置知识**：第 3 课（终端）

> [!TIP]
> 本课程的学习程序**自带 Python**，学 Python 课暂时不用安装任何东西。其他语言可以先用学习程序的「在线运行」体验，等确定要深入学习时再安装。点击菜单「工具 → 检查开发工具」可以随时查看本机装了什么。

## 4.1 先认识几个概念

- **编辑器**（editor）：写代码的软件。
- **编译器 / 解释器**：把代码变成可运行程序的工具（第 1 课）。每门语言都有自己的。
- **SDK / 工具链**（toolchain）：编译器 + 常用工具 + 标准库的「全家桶」。比如 Java 的 JDK、Rust 的 rustup。
- **包管理器**（package manager）：一键安装软件的工具。Windows 有 **winget**，macOS 有 **Homebrew**，Linux 有 **apt**（Ubuntu/Debian）、**dnf**（Fedora）、**pacman**（Arch）。用它们安装比去网站下载安装包更省事，也方便以后升级。

## 4.2 代码编辑器：推荐 VS Code

初学阶段用本课程的学习程序就够了。以后写更大的项目，推荐 **Visual Studio Code**（简称 VS Code）：免费、跨平台、支持所有主流语言。

- 下载：<https://code.visualstudio.com/>（选择你的系统，下载后按默认选项安装）
- Windows 也可以在终端运行：`winget install -e --id Microsoft.VisualStudioCode`
- macOS：`brew install --cask visual-studio-code`
- **中文界面**：打开 VS Code → 左侧「扩展」图标 → 搜索 `Chinese (Simplified)` → 安装 → 重启。
- 写哪门语言，就装哪门语言的扩展：Python、C/C++、Extension Pack for Java、rust-analyzer、Go。

其他选择：JetBrains 系列 IDE（PyCharm、IntelliJ IDEA、CLion、RustRover、GoLand，大多有免费版或教育版）、Zed、Vim/Neovim 等。

## 4.3 Windows：先确认 winget 可用

Windows 10（较新版本）和 Windows 11 自带 **winget**。打开终端输入：

```console
winget --version
```

如果显示版本号就可以了。如果提示找不到，到微软商店搜索并安装「应用安装程序」（App Installer）。

下面每个工具都给出 **winget 命令**（最省事）和**官网下载**两种方式，任选其一。

> [!NOTE]
> 用 winget 安装时可能会弹出「是否允许此应用更改设备」的窗口，点「是」。安装完成后**关闭终端再重新打开**，让新的 PATH 生效。

## 4.4 macOS：先安装 Homebrew 和命令行工具

1. 安装 Xcode 命令行工具（包含 C/C++ 编译器 clang、git 等）：

   ```console
   xcode-select --install
   ```

   会弹出一个窗口，点「安装」，等待完成。

2. 安装 Homebrew（macOS 最常用的包管理器）。打开 <https://brew.sh/>，复制首页上那条安装命令到终端运行（只使用官网的命令）。安装完成后，按屏幕最后的提示（"Next steps"）运行两条命令把 brew 加入 PATH。

3. 验证：

   ```console
   brew --version
   ```

## 4.5 Linux：使用发行版的包管理器

以 Ubuntu / Debian 为例，先更新软件列表：

```console
sudo apt update
```

`sudo` 会要求输入你的登录密码（输入时屏幕上**不显示任何字符**，这是正常的，输完按回车）。Fedora 用 `sudo dnf install ...`，Arch 用 `sudo pacman -S ...`。

## 4.6 安装 Python

| 系统 | 命令 |
|------|------|
| Windows | `winget install -e --id Python.Python.3.13` |
| macOS | `brew install python` |
| Ubuntu/Debian | `sudo apt install python3 python3-pip python3-venv` |
| Fedora | `sudo dnf install python3` |

也可以到 <https://www.python.org/downloads/> 下载安装包。

> [!WARNING]
> Windows 用安装包安装时，第一个界面**一定要勾选「Add python.exe to PATH」**，否则终端里找不到 python。

验证（Windows 用 `python`，macOS/Linux 用 `python3`）：

```console
python3 --version
```

> [!TIP]
> 推荐再安装 **uv**——一个非常快的现代 Python 包与项目管理工具（Python 课第 15 课会详细讲）：Windows `winget install -e --id astral-sh.uv`，macOS `brew install uv`，Linux `curl -LsSf https://astral.sh/uv/install.sh | sh`。

## 4.7 安装 C / C++ 编译器

| 系统 | 做法 |
|------|------|
| Windows | `winget install -e --id BrechtSanders.WinLibs.POSIX.UCRT`（WinLibs：GCC + MinGW-w64） |
| macOS | `xcode-select --install`（4.4 节已经装过） |
| Ubuntu/Debian | `sudo apt install build-essential gdb` |
| Fedora | `sudo dnf install gcc gcc-c++ gdb` |

Windows 的其他选择：

- **MSYS2**（<https://www.msys2.org/>）：安装后在「MSYS2 UCRT64」终端运行 `pacman -S mingw-w64-ucrt-x86_64-gcc`，再把 `C:\msys64\ucrt64\bin` 加入 PATH。
- **Visual Studio 2022 社区版**：微软官方 IDE，自带 MSVC 编译器，适合以后做 Windows 程序。本课程的学习程序使用 GCC/Clang 风格的命令，所以入门推荐 WinLibs 或 MSYS2。

验证：

```console
gcc --version
g++ --version
```

macOS 上 `gcc` 实际上是 Apple Clang，这完全没问题。

## 4.8 安装 Java（JDK 25）

Java 课程基于 **JDK 25**（2025 年 9 月发布的长期支持版）。推荐 Eclipse Temurin 发行版：

| 系统 | 命令 |
|------|------|
| Windows | `winget install -e --id EclipseAdoptium.Temurin.25.JDK` |
| macOS | `brew install --cask temurin` |
| Ubuntu 24.04+ / Debian 13+ | `sudo apt install openjdk-25-jdk` |
| Fedora | `sudo dnf install java-25-openjdk-devel` |

也可以到 <https://adoptium.net/> 下载安装包。验证：

```console
java -version
```

应该看到 `25` 开头的版本号。如果显示的是旧版本（例如 1.8、17），说明电脑上还装着旧 Java 并且排在 PATH 前面——可以卸载旧版本，或调整 PATH 顺序。

## 4.9 安装 Rust

Rust 官方用 **rustup** 管理工具链：

- **Windows**：`winget install -e --id Rustlang.Rustup`，或从 <https://rustup.rs/> 下载 `rustup-init.exe`。运行时如果提示需要 **Visual Studio C++ 生成工具**，选择第 1 项让它自动安装（Rust 在 Windows 上需要它来链接程序）。
- **macOS / Linux**：

  ```console
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  出现选项时直接回车（默认安装）。完成后重新打开终端。

验证：

```console
rustc --version
cargo --version
```

以后升级 Rust 只需要：`rustup update`。

## 4.10 安装 Go

| 系统 | 命令 |
|------|------|
| Windows | `winget install -e --id GoLang.Go` |
| macOS | `brew install go` |
| Linux | 到 <https://go.dev/dl/> 下载 `.tar.gz`，按页面说明解压到 `/usr/local`；或 `sudo snap install go --classic` |

> [!NOTE]
> Ubuntu 的 `apt install golang-go` 装出来的版本通常比较旧，推荐用官网的安装包。

验证：

```console
go version
```

## 4.11 安装 Node.js（JavaScript / TypeScript）

推荐安装 **LTS（长期支持）版本**，22 或更新：

| 系统 | 命令 |
|------|------|
| Windows | `winget install -e --id OpenJS.NodeJS.LTS` |
| macOS | `brew install node` |
| Linux | 用版本管理器 fnm：`curl -fsSL https://fnm.vercel.app/install \| bash`，重开终端后 `fnm install --lts` |

也可以到 <https://nodejs.org/> 下载。验证：

```console
node --version
npm --version
```

Node.js 22.18 及以上版本可以**直接运行 TypeScript 文件**（`node main.ts`），不用另外安装编译器。

## 4.12 汇编课程需要什么？

汇编课程使用 **Linux x86-64** 环境（GCC 自带的汇编器）：

- **Linux（x86-64 电脑）**：装好 4.7 节的 `build-essential` 就行。
- **Windows**：安装 WSL（Windows 里的 Linux）。以管理员身份打开终端运行 `wsl --install`，重启后会装好 Ubuntu；在 Ubuntu 里运行 `sudo apt install build-essential gdb`。
- **macOS**：苹果电脑现在用的是 ARM 芯片，指令集不同。学习程序里可以用「在线运行」；也可以用虚拟机（UTM、OrbStack）运行 Linux。

## 4.13 安装后检查清单

全部装完后，在一个**新打开**的终端里依次运行，每条都应该显示版本号：

```console
python3 --version
gcc --version
java -version
rustc --version
go version
node --version
```

也可以直接在学习程序里点「工具 → 检查开发工具」，一眼看清哪些装好了（✅）、哪些还没有（❌）。

## 4.14 常见问题

| 现象 | 原因与解决 |
|------|------------|
| `command not found` / 无法将……识别为…… | ① 终端没重开；② 没加入 PATH；③ Windows 上 `python3` 应改为 `python` 或 `py` |
| Windows 输入 `python` 打开了微软商店 | 这是系统的「应用执行别名」。安装真正的 Python 后，到「设置 → 应用 → 高级应用设置 → 应用执行别名」里关闭 python.exe 和 python3.exe 两个开关 |
| `permission denied` / 拒绝访问 | Linux/macOS 安装系统软件需要 `sudo`；Windows 以管理员身份运行终端 |
| 下载很慢或失败 | 换个时间重试；国内网络可以使用 winget、Homebrew、pip 的国内镜像（在搜索引擎搜「pip 清华镜像」等） |
| 杀毒软件拦截了编译出来的程序 | 新编译出来的 `.exe` 有时会被误报，可以把你的代码文件夹加入杀毒软件的「排除项」 |
| 版本不对（例如 Java 还是 8） | 电脑上有多个版本，PATH 里旧的排在前面。卸载旧版本或调整 PATH |

## 4.15 小结

- 编辑器写代码，编译器/解释器运行代码；包管理器（winget / Homebrew / apt）让安装变简单。
- **用到哪门语言再装哪门**；学习程序自带 Python，其他语言可以先在线运行。
- 每次安装后**重新打开终端**，用 `--version` 验证。
- 汇编课需要 Linux x86-64：Windows 用 WSL，Mac 用在线运行或虚拟机。

## 4.16 练习

<!-- exercise: verify -->
### 练习 1：如何验证？

安装完 Rust 后，用哪条命令检查是否安装成功？

- `rust install`
- `rustc --version`
- `cargo delete`
- `rm rust`

<details>
<summary>答案与解析</summary>

`rustc --version` 会显示 Rust 编译器的版本号；能显示就说明安装成功并且在 PATH 中。
</details>

<!-- exercise: path-checkbox -->
### 练习 2：Windows 安装 Python

在 Windows 上用安装包安装 Python 时，第一个界面最重要的一个选项是什么？

- 「Install for all users」
- 「Add python.exe to PATH」
- 「Disable path length limit」
- 「Customize installation」

<details>
<summary>答案与解析</summary>

勾选「Add python.exe to PATH」后，终端才能找到 python 命令。忘记勾选的话，可以重新运行安装包选择「Modify」，或手动把路径加入 PATH。
</details>

<!-- exercise: package-manager -->
### 练习 3：包管理器

下面哪一个是 **macOS** 上最常用的包管理器？

- winget
- apt
- Homebrew
- pacman

<details>
<summary>答案与解析</summary>

Homebrew（命令是 `brew`）。winget 属于 Windows，apt 属于 Ubuntu/Debian，pacman 属于 Arch Linux。
</details>

<!-- exercise: checklist -->
### 练习 4：打印检查清单

编写程序，输出下面三行检查命令（这样你就有了一份可以照着输入的清单）：

```text
python3 --version
gcc --version
java -version
```

<details>
<summary>提示</summary>

三个 `print`。注意 `java -version` 只有一个短横线——这是 Java 的老习惯。
</details>

## 4.17 延伸阅读

- [VS Code 官方文档（中文）](https://code.visualstudio.com/docs)：界面导览和各语言的配置指南。
- [Rust 官方安装说明](https://www.rust-lang.org/zh-CN/tools/install)、[Go 安装说明](https://go.dev/doc/install)、[Adoptium（Temurin JDK）](https://adoptium.net/zh-CN/)
- 下一课：[读懂错误信息与调试](../05-errors-debugging/zh-CN.md)
