# 2. 电脑基础：文件、文件夹与输入法

> 🎯 **本课目标**：搞清楚文件、文件夹、扩展名和路径；学会显示文件扩展名；分清全角和半角符号；认识编程时常用的键盘符号和快捷键。
>
> ⏱ **预计用时**：30 分钟　｜　📋 **前置知识**：第 1 课

很多人学编程卡住，不是因为不懂编程，而是因为不熟悉电脑本身的一些概念。这一课把这些「地基」补上。

## 2.1 文件和文件夹

电脑里的一切资料——照片、文档、音乐、程序——都保存在**文件**（file）里。为了方便管理，文件被放在**文件夹**（folder，也叫**目录** directory）里，文件夹里还可以再放文件夹，形成一棵「树」：

```text
C:\Users\xiaoming\              ← 你的「用户文件夹」（Windows）
├── Desktop\                     ← 桌面
├── Documents\                   ← 文档
│   ├── 简历.docx
│   └── code\                    ← 我们建一个放代码的文件夹
│       ├── hello.py
│       └── game\
│           └── main.py
└── Downloads\                   ← 下载
```

> [!TIP]
> 建议你现在就在「文档」里新建一个叫 `code` 的文件夹，以后练习写的代码都放在这里。**文件夹名最好用英文字母、数字、短横线**，不要有空格和中文——有些编程工具遇到中文路径或空格会出问题。

## 2.2 扩展名：文件的「类型标签」

文件名最后一个点 `.` 后面的部分叫**扩展名**（extension），它告诉电脑这是什么类型的文件：

| 扩展名 | 类型 |
|--------|------|
| `.txt` | 纯文本 |
| `.docx` | Word 文档 |
| `.jpg` `.png` | 图片 |
| `.exe` | Windows 程序 |
| `.py` | Python 源代码 |
| `.c` `.h` | C 源代码 |
| `.cpp` | C++ 源代码 |
| `.java` | Java 源代码 |
| `.rs` | Rust 源代码 |
| `.go` | Go 源代码 |
| `.js` `.ts` | JavaScript / TypeScript 源代码 |
| `.md` | Markdown 文档（本教程的课文就是 .md 文件） |

### 一定要打开「显示扩展名」

Windows 和 macOS 默认会**隐藏**扩展名。这会造成一个经典问题：你以为文件叫 `hello.py`，其实它叫 `hello.py.txt`，结果怎么都运行不了。

- **Windows 11**：打开「文件资源管理器」→ 顶部「查看」→「显示」→ 勾选「文件扩展名」。
- **Windows 10**：「文件资源管理器」→「查看」→ 勾选「文件扩展名」。
- **macOS**：打开「访达」→ 菜单栏「访达」→「设置」→「高级」→ 勾选「显示所有文件扩展名」。
- **Linux**：大多数文件管理器默认就会显示。

## 2.3 代码是「纯文本」

代码文件是**纯文本文件**：里面只有文字，没有字体、颜色、加粗这些格式。所以：

- ✅ 用**代码编辑器**写代码：本课程的学习程序、VS Code、记事本（Windows）、TextEdit（macOS 需要切换到「纯文本」模式）。
- ❌ **不要**用 Word、WPS、Pages 写代码——它们会偷偷加入格式，还会把直引号 `"` 自动换成弯引号 `“”`，代码就运行不了了。

> [!NOTE]
> 文本文件保存时还有一个「编码」设置，决定文字怎么变成 0 和 1。**永远选 UTF-8**——它能表示世界上所有文字，也是所有现代编程工具的默认值。第 6 课会讲它的原理。

## 2.4 路径：文件的「地址」

**路径**（path）就是文件在电脑里的完整地址，从最外层的文件夹一层层写到文件：

- Windows：`C:\Users\xiaoming\Documents\code\hello.py`（用**反斜杠** `\` 分隔，开头是盘符 `C:`）
- macOS：`/Users/xiaoming/Documents/code/hello.py`（用**斜杠** `/` 分隔）
- Linux：`/home/xiaoming/code/hello.py`

两种写法：

- **绝对路径**：从最顶层开始写的完整地址，如上面的例子。
- **相对路径**：相对于「当前所在文件夹」的地址。如果你现在在 `code` 文件夹里，那么 `game/main.py` 就指向 `code/game/main.py`；`..` 表示「上一层文件夹」。

下一课学终端时会大量用到路径。

## 2.5 全角和半角：中文用户最常踩的坑 ⚠️

中文输入法有两种标点状态：

- **半角**（英文标点）：`( ) , ; : " ' . ! ?`——每个符号占半个汉字的宽度。
- **全角**（中文标点）：`（ ），；：“ ” ‘ ’。！？`——每个符号占一个汉字的宽度。

**代码中的所有符号都必须是半角的。** 编程语言只认识半角符号，全角符号会直接导致语法错误。

对比一下（仔细看括号和引号的样子）：

```text
✅ print("你好")        ← 半角括号、半角引号
❌ print（"你好"）      ← 全角括号
❌ print(“你好”)        ← 全角（弯）引号
❌ print("你好")；      ← 全角分号
```

### 怎么避免？

1. **写代码时切换到英文输入状态**：大多数中文输入法按 <kbd>Shift</kbd> 键就能切换中/英文。需要在引号里输入中文时再切回来。
2. **把输入法设置成「中文状态下使用英文标点」**：搜狗、微软拼音、百度等输入法的设置里都有这一项（微软拼音：设置 → 常规 → 「中文输入时使用英文标点」）。强烈推荐打开！
3. **用等宽字体看代码**：本课程的编辑器用的就是等宽字体，全角符号明显更宽，更容易发现。
4. 运行出错时，看学习程序给出的中文解释——它能识别全角符号错误。

> [!IMPORTANT]
> 中文**可以**出现在两个地方：引号里的文字（如 `"你好"`）和注释（`# 这是注释`）。除此之外，代码里的一切——关键字、变量名、符号——都用英文半角。

## 2.6 编程常用的键盘符号

编程会用到很多平时不常用的符号。认识它们的名字，看英文资料时也会方便很多：

| 符号 | 中文名 | 英文名 | 输入方法 |
|------|--------|--------|----------|
| `( )` | 圆括号 / 小括号 | parentheses | <kbd>Shift</kbd> + <kbd>9</kbd> / <kbd>0</kbd> |
| `[ ]` | 方括号 / 中括号 | square brackets | 回车键上方附近的 <kbd>[</kbd> <kbd>]</kbd> |
| `{ }` | 花括号 / 大括号 | curly braces | <kbd>Shift</kbd> + <kbd>[</kbd> / <kbd>]</kbd> |
| `< >` | 尖括号 | angle brackets | <kbd>Shift</kbd> + <kbd>,</kbd> / <kbd>.</kbd> |
| `"` | 双引号 | double quote | <kbd>Shift</kbd> + <kbd>'</kbd> |
| `'` | 单引号 | single quote | 回车键左边的 <kbd>'</kbd> |
| `` ` `` | 反引号 | backtick | <kbd>Esc</kbd> 下方、<kbd>1</kbd> 左边 |
| `;` | 分号 | semicolon | <kbd>L</kbd> 右边 |
| `:` | 冒号 | colon | <kbd>Shift</kbd> + <kbd>;</kbd> |
| `=` | 等号 | equals sign | <kbd>Backspace</kbd> 左边 |
| `_` | 下划线 | underscore | <kbd>Shift</kbd> + <kbd>-</kbd> |
| `#` | 井号 | hash / pound | <kbd>Shift</kbd> + <kbd>3</kbd> |
| `\` | 反斜杠 | backslash | 回车键上方 |
| `/` | 斜杠 | slash | 右 <kbd>Shift</kbd> 左边 |
| `\|` | 竖线 | pipe / vertical bar | <kbd>Shift</kbd> + <kbd>\</kbd> |
| `&` | 和号 | ampersand | <kbd>Shift</kbd> + <kbd>7</kbd> |
| `*` | 星号 | asterisk | <kbd>Shift</kbd> + <kbd>8</kbd> |
| `%` | 百分号 | percent | <kbd>Shift</kbd> + <kbd>5</kbd> |
| `~` | 波浪号 | tilde | <kbd>Shift</kbd> + <kbd>`</kbd> |
| `^` | 脱字符 | caret | <kbd>Shift</kbd> + <kbd>6</kbd> |

（不同键盘布局可能略有不同。）

## 2.7 必会的快捷键

熟练使用快捷键能让写代码快很多。Windows/Linux 用 <kbd>Ctrl</kbd>，macOS 用 <kbd>⌘ Command</kbd>：

| 功能 | Windows / Linux | macOS |
|------|-----------------|-------|
| 复制 | <kbd>Ctrl</kbd>+<kbd>C</kbd> | <kbd>⌘</kbd>+<kbd>C</kbd> |
| 粘贴 | <kbd>Ctrl</kbd>+<kbd>V</kbd> | <kbd>⌘</kbd>+<kbd>V</kbd> |
| 剪切 | <kbd>Ctrl</kbd>+<kbd>X</kbd> | <kbd>⌘</kbd>+<kbd>X</kbd> |
| 撤销（后悔药） | <kbd>Ctrl</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Z</kbd> |
| 重做 | <kbd>Ctrl</kbd>+<kbd>Y</kbd> 或 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> | <kbd>⌘</kbd>+<kbd>Shift</kbd>+<kbd>Z</kbd> |
| 全选 | <kbd>Ctrl</kbd>+<kbd>A</kbd> | <kbd>⌘</kbd>+<kbd>A</kbd> |
| 保存 | <kbd>Ctrl</kbd>+<kbd>S</kbd> | <kbd>⌘</kbd>+<kbd>S</kbd> |
| 查找 | <kbd>Ctrl</kbd>+<kbd>F</kbd> | <kbd>⌘</kbd>+<kbd>F</kbd> |
| 注释/取消注释 | <kbd>Ctrl</kbd>+<kbd>/</kbd> | <kbd>⌘</kbd>+<kbd>/</kbd> |
| 增加缩进 / 减少缩进 | <kbd>Tab</kbd> / <kbd>Shift</kbd>+<kbd>Tab</kbd> | 同左 |

在本课程的学习程序里，还有：<kbd>F5</kbd> 运行，<kbd>Ctrl</kbd>+<kbd>Enter</kbd> 检查答案，<kbd>Alt</kbd>+<kbd>←</kbd>/<kbd>→</kbd> 上一课/下一课。

## 2.8 试一试

下面的程序打印出一些编程常用符号。运行它，然后试着**自己在编辑器里重新敲一遍**（不要复制粘贴），练习输入这些符号：

```python
print("圆括号 ( )  方括号 [ ]  花括号 { }")
print("引号 \" '  分号 ;  冒号 :  等号 =")
print("路径示例：C:\\Users\\code\\hello.py")
```

注意第三行：要在字符串里写一个反斜杠 `\`，需要写成两个 `\\`（第 1 课的「转义字符」）。

## 2.9 小结

- 资料存在**文件**里，文件放在**文件夹**里，形成树状结构。
- **扩展名**表示文件类型；一定要打开「显示扩展名」，避免 `hello.py.txt` 问题。
- 代码是**纯文本**，用代码编辑器写，不要用 Word；编码选 **UTF-8**。
- **路径**是文件的地址，有绝对路径和相对路径；Windows 用 `\`，macOS/Linux 用 `/`。
- 代码里的符号必须是**半角**的；中文只能出现在引号里和注释里。
- 记住常用符号的名字和快捷键，写代码会顺手很多。

## 2.10 练习

<!-- exercise: extension -->
### 练习 1：扩展名

一个 Python 源代码文件，扩展名通常是什么？

- `.exe`
- `.txt`
- `.py`
- `.docx`

<details>
<summary>答案与解析</summary>

`.py`。`.exe` 是 Windows 程序，`.txt` 是普通文本，`.docx` 是 Word 文档。
</details>

<!-- exercise: half-width -->
### 练习 2：找出能正常运行的代码

下面哪一行**全部使用了半角符号**，可以正常运行？

- `print（"OK"）`
- `print(“OK”)`
- `print("OK")`
- `print("OK")；`

<details>
<summary>答案与解析</summary>

第三个。第一个用了全角括号，第二个用了全角弯引号，第四个末尾多了一个全角分号。
</details>

<!-- exercise: symbols -->
### 练习 3：符号练习

自己动手输入（不要复制），编写程序输出下面这一行：

```text
(a + b) * [c - d] / {e}
```

<details>
<summary>提示</summary>

整行放在一对双引号里交给 `print`。检查每一个括号是不是半角的。
</details>

<!-- exercise: backslash -->
### 练习 4：打印 Windows 路径

编写程序输出：

```text
C:\code\hello.py
```

<details>
<summary>提示</summary>

字符串里的 `\` 要写成 `\\`，例如 `"C:\\code"`。
</details>

## 2.11 延伸阅读

- [微软：在 Windows 中显示文件扩展名](https://support.microsoft.com/zh-cn/windows)（在支持页面搜索「文件扩展名」）
- [Missing Semester 中文版](https://missing-semester-cn.github.io/)：MIT 的「计算机教育中缺失的一课」，讲 Shell、编辑器、Git 等程序员必备的工具使用技巧。
- 下一课：[终端（命令行）入门](../03-terminal/zh-CN.md)
