# 3. 终端（命令行）入门

> 🎯 **本课目标**：打开终端；看懂「提示符」；学会 `pwd`、`ls`、`cd`、`mkdir` 等基本命令；用终端运行程序；理解 PATH 和「找不到命令」错误。
>
> ⏱ **预计用时**：40 分钟　｜　📋 **前置知识**：第 2 课（文件、文件夹、路径）

## 3.1 终端是什么？

平时我们用鼠标点图标来操作电脑，这叫**图形界面**（GUI）。程序员还经常用另一种方式：在一个黑色（或白色）的窗口里**打字输入命令**，电脑执行后把结果以文字显示出来。这个窗口叫**终端**（terminal），在里面解释命令的程序叫 **shell**，这种操作方式叫**命令行**（command line）。

为什么要学它？

- 很多开发工具**只有**命令行版本（编译器、Git、包管理器……）。
- 安装开发工具、运行和调试程序，教程里给的几乎都是命令。
- 熟练以后，很多操作比点鼠标快得多，还能写成脚本自动执行。

> [!NOTE]
> 本课程的学习程序可以帮你运行代码，所以**没学会终端也能继续学下一课**。但迟早要用到它，建议现在花点时间熟悉。

## 3.2 打开终端

- **Windows 11**：在开始菜单搜索「终端」（Windows Terminal），打开后默认是 **PowerShell**。Windows 10 可以搜索「PowerShell」。
- **macOS**：按 <kbd>⌘</kbd> + <kbd>空格</kbd> 打开「聚焦搜索」，输入「终端」或 Terminal，回车。默认 shell 是 **zsh**。
- **Linux**：大多数桌面按 <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>，默认 shell 一般是 **bash**。

打开后你会看到类似这样的一行：

```console
PS C:\Users\xiaoming>                 ← Windows PowerShell
xiaoming@MacBook ~ %                  ← macOS zsh
xiaoming@ubuntu:~$                    ← Linux bash
```

这叫**提示符**（prompt），意思是「我准备好了，请输入命令」。提示符里通常会显示**当前所在的文件夹**（`~` 表示你的用户文件夹）。

> [!IMPORTANT]
> 教程里的命令前面常带有 `$` 或 `>`，那是提示符，**不要输入它**。例如看到 `$ python3 --version`，你只需要输入 `python3 --version`。

## 3.3 第一批命令

终端始终「站在」某个文件夹里，这叫**当前目录**（working directory）。下面这些命令用来查看和移动位置。

| 作用 | macOS / Linux | Windows PowerShell |
|------|---------------|--------------------|
| 显示当前在哪个文件夹 | `pwd` | `pwd`（或 `Get-Location`） |
| 列出当前文件夹的内容 | `ls` | `ls`（或 `dir`） |
| 进入某个文件夹 | `cd 文件夹名` | `cd 文件夹名` |
| 回到上一层 | `cd ..` | `cd ..` |
| 回到用户文件夹 | `cd ~` 或 `cd` | `cd ~` |
| 新建文件夹 | `mkdir 名字` | `mkdir 名字` |
| 清屏 | `clear` | `cls`（或 `clear`） |
| 显示文件内容 | `cat 文件名` | `cat 文件名`（或 `type`） |
| 显示一段文字 | `echo 你好` | `echo 你好` |

好消息：PowerShell 为了方便，给很多命令起了和 macOS/Linux 一样的别名，所以上面大部分命令在三个系统上都能用。

### 跟着做一遍

在终端里依次输入下面每一行，每行输入后按回车，观察结果：

```console
pwd
ls
mkdir code
cd code
pwd
cd ..
pwd
```

你会看到：先显示用户文件夹的路径；列出里面的文件；新建 `code` 文件夹；进入它（`pwd` 显示的路径末尾多了 `code`）；再退回上一层。

> [!TIP]
> 如果 `mkdir code` 提示文件夹已经存在，说明你在第 2 课已经建好了，直接 `cd code` 就行。

## 3.4 两个超级省力的技巧

1. **<kbd>Tab</kbd> 自动补全**：输入文件夹名的前几个字母，按 <kbd>Tab</kbd>，终端会自动补全剩下的部分。比如输入 `cd Doc` 再按 <kbd>Tab</kbd>，会变成 `cd Documents`。既省时间又不会拼错。
2. **<kbd>↑</kbd> 方向键调出历史命令**：按上箭头可以找回之前输入过的命令，修改一下再回车，不用重新打。

还有一个**救命键**：程序卡住了、运行停不下来（比如写了死循环），按 <kbd>Ctrl</kbd> + <kbd>C</kbd> 强制停止。注意在终端里 <kbd>Ctrl</kbd> + <kbd>C</kbd> **不是复制**！（终端里复制一般用 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>C</kbd>，macOS 是 <kbd>⌘</kbd> + <kbd>C</kbd>。）

## 3.5 用终端运行程序

输入一个程序的名字，就能运行它。先检查一下电脑上有没有 Python：

```console
python3 --version
```

Windows 上通常用：

```console
python --version
```

或者 `py --version`。如果看到 `Python 3.13.x` 之类的版本号，说明已经安装好了。（还没安装也没关系，下一课会讲。）

### 运行一个 Python 文件

1. 用编辑器（例如学习程序的「文件 → 另存为」）把下面的代码保存为 `code` 文件夹里的 `hello.py`：

   ```python
   print("我在终端里运行了这个程序！")
   ```

2. 在终端里进入这个文件夹，运行它：

   ```console
   cd ~/Documents/code
   python3 hello.py
   ```

   Windows 上：

   ```console
   cd ~\Documents\code
   python hello.py
   ```

3. 终端会显示：`我在终端里运行了这个程序！`

命令的一般格式是：**程序名 参数1 参数2 ……**。`python3 hello.py` 的意思就是「运行 python3 这个程序，交给它的参数是 hello.py」。像 `--version` 这样以短横线开头的参数叫**选项**（option），用来调整程序的行为。

## 3.6 PATH：终端去哪里找程序？

你输入 `python3`，终端怎么知道 Python 装在哪里？答案是一个叫 **PATH** 的**环境变量**：它是一串文件夹路径，终端会按顺序在这些文件夹里找名为 `python3` 的程序。

```text
PATH = C:\Windows\System32 ; C:\Python313 ; C:\Users\xiaoming\.cargo\bin ; ...
             │                    │                     │
           找 python？没有      找到了！运行它          （后面的不再找）
```

如果所有文件夹里都找不到，就会报错：

```console
python3: command not found                                   ← macOS / Linux
python3 : 无法将“python3”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。   ← Windows
```

这类错误的意思都是「**找不到这个命令**」。常见原因：

1. **没有安装**这个程序。
2. 安装了，但**没有加入 PATH**（Windows 安装 Python 时要勾选「Add python.exe to PATH」）。
3. 刚安装完，**终端没重启**——PATH 只在终端启动时读取一次。关掉终端重新打开试试。
4. **名字打错了**，或者这个系统上的名字不一样（例如 Windows 上叫 `python` 而不是 `python3`）。

想看看 PATH 里有什么：

```console
echo $PATH          # macOS / Linux
echo $env:PATH      # Windows PowerShell
```

## 3.7 小心危险命令

终端里的命令**没有回收站**。下面这些命令要格外小心，在没完全理解之前不要运行：

- `rm`（删除文件）、`rm -r`（删除文件夹及其中所有内容）、Windows 的 `del`、`Remove-Item`
- 任何以 `sudo` 开头的命令（以管理员权限执行）
- 从网上复制来的、你看不懂的长命令

> [!WARNING]
> 永远不要运行别人发给你、你又不理解的命令，尤其是包含 `rm -rf`、`format`、`curl ... | sh` 的命令。安装正规软件时，只使用官方网站给出的命令。

## 3.8 小结

- **终端**是输入命令的窗口，**shell** 负责理解命令；提示符表示「等待输入」，教程中命令前的 `$` 不要输入。
- 基本命令：`pwd` 看位置、`ls` 看内容、`cd` 进入、`cd ..` 返回、`mkdir` 新建、`clear` 清屏。
- <kbd>Tab</kbd> 补全、<kbd>↑</kbd> 历史命令、<kbd>Ctrl</kbd>+<kbd>C</kbd> 停止程序。
- 运行程序：`程序名 参数`，例如 `python3 hello.py`。
- 「找不到命令」通常是没安装、没加入 PATH、终端没重启或名字打错。
- 删除类命令没有回收站，不懂的命令不要运行。

## 3.9 练习

<!-- exercise: cd-up -->
### 练习 1：回到上一层

你在终端里的当前目录是 `/home/xiaoming/code/game`，想回到 `/home/xiaoming/code`，应该输入哪条命令？

- `cd game`
- `cd ..`
- `mkdir ..`
- `ls ..`

<details>
<summary>答案与解析</summary>

`cd ..`：`..` 代表上一层文件夹。`ls ..` 只是列出上一层的内容，并不会移动过去。
</details>

<!-- exercise: stop -->
### 练习 2：程序停不下来

在终端里运行的程序陷入了死循环，一直停不下来。按什么键可以强制停止它？

- <kbd>Ctrl</kbd> + <kbd>S</kbd>
- <kbd>Ctrl</kbd> + <kbd>C</kbd>
- <kbd>Tab</kbd>
- <kbd>↑</kbd>

<details>
<summary>答案与解析</summary>

<kbd>Ctrl</kbd> + <kbd>C</kbd>。在终端里它的作用是「中断」，不是复制。
</details>

<!-- exercise: not-found -->
### 练习 3：找不到命令

刚刚安装完 Python，在**之前就打开着的**终端里输入 `python --version`，却提示找不到命令。最可能先尝试的做法是？

- 重装系统
- 关掉终端，重新打开一个再试
- 把命令改成 `PYTHON --VERSION`
- 删除 Python 文件夹

<details>
<summary>答案与解析</summary>

终端只在启动时读取一次 PATH。安装程序修改了 PATH 之后，需要新开一个终端才能生效。如果还不行，再检查安装时是否勾选了「加入 PATH」。
</details>

<!-- exercise: commands -->
### 练习 4：写出命令

我们用 Python 来「打印」一组终端命令。编写程序，输出下面三行——它们依次表示：新建 `project` 文件夹、进入它、显示当前位置：

```text
mkdir project
cd project
pwd
```

<details>
<summary>提示</summary>

用三个 `print`，每个打印一行文字。
</details>

## 3.10 延伸阅读

- [Missing Semester 中文版 · 课程概览与 shell](https://missing-semester-cn.github.io/2020/course-shell/)
- [微软 PowerShell 文档](https://learn.microsoft.com/zh-cn/powershell/scripting/learn/ps101/01-getting-started)
- 下一课：[安装开发工具](../04-install-tools/zh-CN.md)
