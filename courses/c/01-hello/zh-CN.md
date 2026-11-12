# 1. 第一个 C 程序

> 🎯 **本课目标**：知道为什么要学 C；写出、编译并运行第一个 C 程序，逐行看懂它；理解「编译」到底做了什么（预处理 → 编译 → 汇编 → 链接）；用 `printf` 输出文字和特殊字符；会写注释；认识最常见的几种编译错误。
>
> ⏱ **预计用时**：50 分钟　｜　📋 **前置知识**：「编程起步」课程（尤其是第 3 课终端、第 4 课安装 C 编译器、第 5 课读懂报错）

## 1.1 为什么学 C？

C 语言诞生于 1972 年，比你用过的几乎所有软件都老。但直到今天，它仍然是世界上最重要的语言之一：

- **操作系统**：Linux、Windows、macOS 的内核主要是用 C 写的；
- **嵌入式设备**：洗衣机、汽车、路由器、无人机里的芯片程序，大多是 C；
- **其他语言的地基**：Python 解释器（CPython）、数据库 SQLite、Redis、Git，都是用 C 写的；
- **理解计算机**：C 让你直接面对**内存**、**地址**和**字节**。学会 C，你就明白了程序在计算机里到底是怎样运行的，再学其他语言也会更扎实。

和 Python 比，C 更「啰嗦」，需要你自己管理更多细节，也更容易出错——但正是这些细节，能让你真正理解计算机。别担心，我们会一步一步来。

## 1.2 Hello, World!

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    printf("你好，C 语言！\n");
    return 0;
}
```

点「运行」（或者在终端编译运行，见 1.3 节），就会看到两行文字。我们逐行来看：

| 代码 | 意思 |
|------|------|
| `#include <stdio.h>` | 把「标准输入输出」库的说明（头文件）包含进来，这样才能使用 `printf`。`stdio` = **st**andar**d** **i**nput/**o**utput |
| `int main(void)` | 定义 **`main` 函数**：程序从这里开始执行。`int` 表示它最后返回一个整数，`void` 表示它不接收参数 |
| `{ ... }` | 花括号括起来的是函数体，也就是要执行的语句 |
| `printf("...\n");` | 输出一段文字。`\n` 表示换行。**每条语句以分号 `;` 结尾** |
| `return 0;` | `main` 结束，返回 0 给操作系统，表示「程序正常结束」 |

> [!NOTE]
> 和 Python 不同，C 用**花括号**表示代码块，用**分号**结束语句；缩进只是为了让人看得清楚，编译器并不在乎。但请一定保持良好的缩进习惯（4 个空格），否则代码很快就会变得难以阅读。

## 1.3 编译和运行

Python 是**解释执行**的：解释器一边读代码一边执行。C 是**编译执行**的：先用**编译器**把源代码翻译成计算机能直接执行的**机器码**，生成一个**可执行文件**，然后再运行这个文件。

在学习程序里，点「运行」时它会自动帮你编译再运行。要在自己的终端里做，把代码保存为 `hello.c`，然后：

```bash
$ gcc hello.c -o hello       # 编译：把 hello.c 变成可执行文件 hello
$ ./hello                    # 运行（macOS / Linux）
$ hello.exe                  # 运行（Windows）
```

- `gcc` 是编译器（macOS 上的 `gcc` 实际是 Clang，用法相同）；
- `-o hello` 指定输出文件的名字，不写的话默认叫 `a.out`（Windows 上是 `a.exe`）；
- 建议加上 `-Wall -Wextra`，让编译器提醒你可能的问题：`gcc -Wall -Wextra hello.c -o hello`。学习程序默认就加了这两个选项。

### 编译到底做了什么？

`gcc` 一条命令背后，其实依次做了四件事：

```text
hello.c ──① 预处理──► 展开 #include，去掉注释
        ──② 编译────► 翻译成汇编语言（hello.s）
        ──③ 汇编────► 翻译成机器码（hello.o，目标文件）
        ──④ 链接────► 和库（printf 的实现在 C 标准库里）拼在一起 ──► 可执行文件 hello
```

所以：

- **编译时错误**（语法错误、拼错名字）会在第 ②、④ 步被发现，程序根本生成不出来；
- 生成的可执行文件可以**单独运行**，不需要源代码，也不需要编译器——这就是很多软件发布时只给你一个 `.exe` 的原因；
- 编译后的程序运行得非常快，这是 C 至今仍被大量使用的重要原因。

> [!TIP]
> 好奇的话，可以用 `gcc -S hello.c` 看看生成的汇编代码（`hello.s`）。本课程的「汇编」课会讲怎么读懂它。

## 1.4 用 printf 输出

`printf` 不会自动换行，需要换行的地方要自己写 `\n`：

```c
#include <stdio.h>

int main(void) {
    printf("第一行");
    printf("还是第一行\n");
    printf("第二行\n第三行\n");
    return 0;
}
```

像 `\n` 这样以反斜杠开头的组合叫**转义序列**，用来表示不方便直接写出来的字符：

| 写法 | 表示 |
|------|------|
| `\n` | 换行 |
| `\t` | 制表符（Tab，跳到下一个对齐位置） |
| `\\` | 一个反斜杠 `\` |
| `\"` | 一个双引号 `"` |
| `%%` | 一个百分号 `%`（这是 `printf` 自己的规则，因为 `%` 有特殊用途，下一课会讲） |

```c
#include <stdio.h>

int main(void) {
    printf("名字\t年龄\n");
    printf("小明\t12\n");
    printf("他说：\"你好\"\n");
    printf("文件在 C:\\Users\\me\n");
    printf("完成了 100%%\n");
    return 0;
}
```

## 1.5 注释

注释是写给人看的说明，编译器会忽略它们：

```c
#include <stdio.h>

/* 这是多行注释：
   可以写很多行，
   从 / 和 * 开始，到 * 和 / 结束。 */

int main(void) {
    // 这是单行注释：从 // 到这一行结尾
    printf("注释不会影响程序\n");   // 也可以写在代码后面
    return 0;
}
```

## 1.6 关于 `main` 的返回值

`return 0;` 把 0 交给操作系统，表示「一切正常」；返回其他数字（比如 1）通常表示「出错了」。在终端里可以查看上一个程序的返回值（叫**退出码**）：

```bash
$ ./hello
$ echo $?                 # macOS / Linux：显示 0
$ echo %errorlevel%       # Windows 命令提示符
```

脚本和其他程序会根据退出码判断你的程序有没有成功。从 C99 开始，`main` 函数最后如果没有写 `return`，会自动当作 `return 0;`——但明确写出来是好习惯。

## 1.7 最常见的编译错误

编译器的报错是英文的，但很有规律。学习程序会在下面用你的语言解释原因。

**忘了分号**：

```c norun
#include <stdio.h>

int main(void) {
    printf("Hello\n")
    return 0;
}
```

```text
hello.c: In function 'main':
hello.c:4:22: error: expected ';' before 'return'
```

`hello.c:4:22` 表示「第 4 行第 22 列」。注意：分号漏在第 4 行末尾，但编译器常常要读到下一个词（第 5 行的 `return`）才发现问题。**报错的行号附近、以及它的上一行，都要检查。**

**拼错函数名，或者忘了 `#include`**：

```c norun
int main(void) {
    printf("Hello\n");
    return 0;
}
```

```text
hello.c:2:5: warning: implicit declaration of function 'printf' [-Wimplicit-function-declaration]
hello.c:1:1: note: include '<stdio.h>' or provide a declaration of 'printf'
```

编译器不认识 `printf`，因为没有 `#include <stdio.h>`。GCC 13 及更早的版本只给出**警告**（warning），程序也许还能运行；GCC 14 起和 Clang 会直接报**错误**（error）。不管是哪种，都要改好——**不要忽视警告**，它们常常就是 bug 的源头。把 `printf` 拼成 `printF`、`pritnf` 也会得到类似的提示（`note` 那一行还会好心地告诉你该包含哪个头文件）。

**用了中文标点**：

```c norun
#include <stdio.h>

int main(void) {
    printf（"Hello\n"）；
    return 0;
}
```

```text
hello.c:4:5: error: 'printf\U0000ff08' undeclared (first use in this function); did you mean 'printf'?
```

`\U0000ff08` 是全角括号 `（` 的 Unicode 编号，编译器以为你写了一个叫「printf（」的名字。（不同的编译器和版本说法不同，也可能是 `stray '\357' in program`，意思是「程序里有一个不该出现的字节」。）`（`、`）`、`；`、`“` 都是**全角**的中文标点，C 只认半角的 `(`、`)`、`;`、`"`。写代码时记得切换到英文输入法（入门课第 5 课讲过）。字符串**里面**可以有中文，但代码本身的符号必须是英文的。

## 1.8 小结

- C 是离硬件最近的高级语言，操作系统、嵌入式设备和许多语言的解释器都是用它写的。
- 每个 C 程序都从 `int main(void) { ... }` 开始执行；`#include <stdio.h>` 之后才能用 `printf`。
- 语句以 `;` 结尾，代码块用 `{ }`；缩进是给人看的。
- C 是编译型语言：`gcc hello.c -o hello` 编译，`./hello` 运行；编译 = 预处理 → 编译 → 汇编 → 链接。
- `printf` 不自动换行；转义序列 `\n`、`\t`、`\\`、`\"`，以及 `%%`。
- 注释：`// 单行` 和 `/* 多行 */`。`return 0` 表示程序正常结束。
- 报错先看行号，也要看上一行；分号、`#include`、中文标点是最常见的错误来源。

## 1.9 练习

<!-- exercise: hello-c -->
### 练习 1：你好，C

写一个程序，输出一行 `Hello, C!`。

```text
样例输出：Hello, C!
```

<details>
<summary>提示</summary>

照着 1.2 节的程序写：`printf("Hello, C!\n");`，别忘了 `#include <stdio.h>` 和 `return 0;`。
</details>

<!-- exercise: triangle -->
### 练习 2：小三角形

输出下面这个由星号组成的三角形（4 行）：

```text
*
**
***
****
```

<details>
<summary>提示</summary>

可以写 4 个 `printf`，也可以只写一个：`printf("*\n**\n***\n****\n");`。
</details>

<!-- exercise: escapes -->
### 练习 3：转义字符

输出下面三行，一个字符都不能差：

```text
"C" is fun!
Path: C:\new
100% done
```

<details>
<summary>提示</summary>

双引号写 `\"`；反斜杠写 `\\`（注意 `\n` 会被当成换行！）；`printf` 里的百分号写 `%%`。
</details>

<!-- exercise: fix-errors -->
### 练习 4：改错

起始代码里有 3 处错误，导致无法编译。找出并改正它们，让程序输出：

```text
I can fix errors.
Compiler is my friend.
```

<details>
<summary>提示</summary>

先运行一下，看编译器报的第一个错误，改好后再运行，一次改一个。检查：是不是少了 `#include`？分号是不是英文的？函数名有没有拼错？
</details>

<!-- exercise: printf-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

int main(void) {
    printf("A");
    printf("B\n");
    printf("C\nD");
    printf("\n");
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`printf` 不会自动换行：`A` 和 `B` 在同一行；`C` 后面换行，`D` 单独一行，最后再换一次行。所以输出是 `AB`、`C`、`D` 三行。
</details>

<!-- exercise: return-zero -->
### 练习 6：`return 0` 的意思

`main` 函数里的 `return 0;` 表示什么？

- 程序输出数字 0
- 程序正常结束，把退出码 0 交给操作系统
- 程序会从头再运行一次
- 释放所有内存，否则程序会出错

<details>
<summary>答案与解析</summary>

`main` 的返回值是程序的**退出码**，0 表示成功。它不会在屏幕上输出任何东西。
</details>

## 1.10 延伸阅读

- [cppreference：C 语言参考](https://zh.cppreference.com/w/c)（有中文版，以后查语法、查函数用）
- Brian W. Kernighan、Dennis M. Ritchie：《C 程序设计语言》（K&R，C 语言作者写的经典）
- [Compiler Explorer](https://godbolt.org/)：在浏览器里看 C 代码编译成的汇编
- 下一课：[变量与类型](../02-variables-types/zh-CN.md)
