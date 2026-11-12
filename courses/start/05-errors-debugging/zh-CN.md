# 5. 读懂错误信息与调试

> 🎯 **本课目标**：不再害怕红色的报错；学会从错误信息里找到「哪个文件、第几行、什么问题」；掌握 40 个常见英文报错词汇；学会 print 调试、二分排查等方法；知道怎样搜索、提问和正确使用 AI 助手。
>
> ⏱ **预计用时**：40 分钟　｜　📋 **前置知识**：第 1 课

## 5.1 心态：报错是朋友

新手看到满屏红字，第一反应往往是「我是不是不适合编程」。其实：

- **所有程序员每天都在报错。** 写代码就是「写 → 报错 → 改」的循环。
- 错误信息是编译器/解释器**在帮你**：它告诉你哪里有问题，有时还会告诉你怎么改。
- 真正可怕的不是报错，而是程序**不报错但结果是错的**——所以能报错其实是好事。

> [!TIP]
> 把错误信息当成一条**线索**，而不是一句批评。你的任务是当侦探：从线索出发，找到问题所在。

## 5.2 错误信息的结构

不同语言的格式略有不同，但都包含这几样信息：**在哪里**（文件、行号、列号）、**什么类型**、**具体描述**。

### Python：从下往上读

```python norun
def average(numbers):
    return sum(numbers) / len(numbers)

print(average([]))
```

```text
Traceback (most recent call last):
  File "main.py", line 4, in <module>
    print(average([]))
          ~~~~~~~^^^^
  File "main.py", line 2, in average
    return sum(numbers) / len(numbers)
           ~~~~~~~~~~~~~^~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

- **最后一行最重要**：`ZeroDivisionError: division by zero`——错误类型是「除以零」。
- 往上是 **Traceback（调用回溯）**：程序是怎么一步步走到出错位置的。最下面的 `File "main.py", line 2` 是真正出错的那一行，上面的 `line 4` 是调用它的地方。
- `~~~^^^` 这些符号标出了出错的具体位置。

所以读 Python 报错的顺序是：**先看最后一行（是什么错），再看最后一个 File（在哪里错）。**

### C / C++：文件:行:列: 级别: 描述

```text
main.c:5:20: error: expected ';' before 'return'
    5 |     printf("hi\n")
      |                   ^
      |                   ;
    6 |     return 0;
```

读法：`main.c` 文件第 `5` 行第 `20` 列，`error`（错误），描述是「在 return 前面应该有一个 `;`」。编译器甚至在下面画出了应该加分号的位置。

> [!IMPORTANT]
> 编译型语言常常**一次报出很多错误**，但后面的错误往往是第一个错误引起的「连锁反应」。**永远先修第一个错误**，然后重新编译。

### error 和 warning 的区别

- **error（错误）**：必须改，否则程序无法编译/运行。
- **warning（警告）**：程序能运行，但编译器觉得这里**很可能有问题**（例如定义了变量却没用到）。不要忽略警告——很多 bug 就藏在警告里。

## 5.3 常见报错词汇表

报错信息的英语很「模板化」，掌握下面这些词，大部分报错你都能猜出意思：

| 英文 | 中文 | 例子 |
|------|------|------|
| error | 错误 | `SyntaxError` 语法错误 |
| warning | 警告 | `warning: unused variable` |
| syntax | 语法 | `invalid syntax` 无效的语法 |
| invalid | 无效的 | `invalid character` 无效字符 |
| expected | 期望（这里应该有） | `expected ':'` 这里应该有冒号 |
| unexpected | 意外的（这里不该有） | `unexpected indent` 意外的缩进 |
| missing | 缺少 | `missing return statement` 缺少 return |
| undefined / not defined | 未定义 | `name 'x' is not defined` |
| undeclared | 未声明 | `'x' undeclared` |
| declared | 已声明 | `declared and not used` 声明了但没用 |
| unused | 未使用的 | `unused import` 未使用的导入 |
| type | 类型 | `TypeError` 类型错误 |
| mismatch / mismatched | 不匹配 | `mismatched types` 类型不匹配 |
| incompatible | 不兼容 | `incompatible types` |
| cannot / can't | 不能 | `cannot find symbol` 找不到符号 |
| convert | 转换 | `cannot convert string to int` |
| index | 下标 | `IndexError` |
| out of range / out of bounds | 越界 | `index out of range` |
| null / nil / None | 空值 | `NullPointerException` 空指针异常 |
| reference | 引用 | `undefined reference` 未定义的引用 |
| symbol | 符号（名字） | `cannot find symbol` |
| scope | 作用域 | `not declared in this scope` |
| argument | 实参（调用时传的值） | `takes 1 positional argument but 2 were given` |
| parameter | 形参（定义时写的名字） | |
| return | 返回 | `return type` 返回类型 |
| statement | 语句 | `unreachable statement` 永远执行不到的语句 |
| expression | 表达式 | `expected expression` |
| token | 记号（一个「词」或符号） | `unexpected token` |
| literal | 字面量（直接写出的值） | `unterminated string literal` 没结束的字符串 |
| unterminated / never closed | 没有结束 / 没有关闭 | `'(' was never closed` |
| indent / indentation | 缩进 | `IndentationError` |
| attribute | 属性 | `has no attribute 'x'` |
| module | 模块 | `No module named 'numpy'` |
| permission denied | 没有权限 | |
| not found | 找不到 | `file not found` |
| overflow | 溢出 | `integer overflow` |
| exception | 异常 | `Exception in thread "main"` |
| panic | （Rust/Go）程序崩溃 | `thread 'main' panicked` |
| segmentation fault | 段错误（非法访问内存） | |
| deprecated | 已过时（不推荐使用） | `deprecated function` |
| traceback / stack trace | 调用回溯 / 调用栈 | |

> [!TIP]
> 学习程序遇到常见错误时，会在错误下面自动给出中文解释。看完解释，再回头对照英文原文，你会越来越熟悉这些词。

## 5.4 调试方法

**调试**（debug）就是找出并修正程序中的错误（bug，「虫子」）。下面是最实用的几种方法。

### 方法 1：先读，再想，再改

不要一看到报错就乱改代码。按顺序做：

1. 读错误信息：**什么错误？哪一行？**
2. 看那一行（以及**上一行**——很多语法错误其实是上一行少了符号）。
3. 想一想：我**期望**发生什么？**实际**发生了什么？
4. 做**一个**小修改，重新运行。

### 方法 2：print 调试——让程序「说出」它在干什么

程序结果不对但又不报错时，在关键位置把变量的值打印出来：

```python
price = 100
discount = 20
final = price - price * discount / 100
print("调试：final =", final)     # 看看算出来的到底是多少
print("应付金额：", final)
```

看到中间结果，你就能知道是哪一步算错了。找到问题后，记得删掉这些调试用的 `print`。

### 方法 3：二分排查——注释掉一半

程序很长、不知道错在哪里时：把后一半代码**注释掉**，运行看看问题还在不在。还在，说明问题在前一半；不在了，说明在后一半。再对有问题的那一半重复这个过程。每次排除一半，很快就能定位。

### 方法 4：小黄鸭调试法

把代码**一行一行地讲给别人听**——没有人的话就讲给一只橡皮小黄鸭听。解释「这一行是做什么的」的时候，你常常会突然发现：「咦，这里不对！」这个方法真的很有效，以至于程序员们给它起了专门的名字（rubber duck debugging）。

### 方法 5：做一个「最小例子」

把出问题的代码复制出来，**删掉一切与问题无关的部分**，直到剩下最少的几行仍然能重现这个错误。这个过程本身经常就能让你找到原因；如果找不到，这个最小例子也正好用来向别人提问。

### 方法 6：使用调试器

更专业的工具是**调试器**（debugger）：可以让程序在某一行**暂停**（断点），一步一步执行，随时查看每个变量的值。VS Code 等编辑器都内置了调试器（点击行号左边设置断点，按 <kbd>F5</kbd> 开始调试）。以后写大程序时一定要学会它。

## 5.5 怎样搜索和提问

### 搜索

1. 复制错误信息中**最关键、最通用**的那部分，去掉你自己的文件名、变量名和路径。例如搜索 `ZeroDivisionError: division by zero`，而不是 `main.py line 2 average`。
2. 加上语言名：`python ZeroDivisionError division by zero`。
3. 英文搜索结果通常更多、更新。常去的网站：**Stack Overflow**、官方文档、GitHub Issues。

### 提问

一个好问题能让别人在一分钟内帮到你。请包含：

1. **你想做什么**（目标）；
2. **你写的代码**（最小例子，用代码块，不要截图）；
3. **完整的错误信息**（复制文字）；
4. **你已经尝试过什么**；
5. 运行环境：操作系统、语言版本。

### 正确使用 AI 助手

ChatGPT、Claude、通义千问、DeepSeek 等 AI 助手对学习编程很有帮助，但要用对方法：

- ✅ 让它**解释**错误信息、概念和代码：「这段代码为什么报这个错？请一步一步解释。」
- ✅ 让它**出题**、给提示，而不是直接给答案。
- ✅ 写完代码后让它**评审**：「有什么可以改进的？」
- ❌ **不要直接复制答案**交差——你会失去练习的机会，下一次遇到同样的问题还是不会。
- ⚠️ AI 也会出错，说得很自信但完全错误的情况并不少见。**一定要运行验证**，并对照官方文档。

## 5.6 动手练习：修复程序

下面这段程序有 3 个错误。先运行它，根据报错一个一个修好（练习 3 是同样的程序，修好后可以检查答案）：

```python norun
print("购物小票")
apple = 3
banana = 5
print("苹果：" apple, "个")
print("香蕉：", Banana, "个")
print("合计：", apple + banana, "个"
```

## 5.7 小结

- 报错是在帮你；最可怕的是不报错的错误结果。
- Python 报错**从下往上读**：最后一行看错误类型，最后一个 `File` 看位置。
- 编译器报错格式是「文件:行:列: 级别: 描述」；**先修第一个错误**。
- error 必须改，warning 也要重视。
- 调试方法：读-想-改、print 调试、二分排查、小黄鸭、最小例子、调试器。
- 搜索时去掉个人信息；提问时给出目标、代码、完整报错和已尝试的方法。
- AI 助手用来解释和启发，不要盲目复制，一定要验证。

## 5.8 练习

<!-- exercise: read-traceback -->
### 练习 1：读 Python 报错

看到下面的报错，问题出在第几行？

```text
Traceback (most recent call last):
  File "main.py", line 7, in <module>
    total = add(1, "2")
  File "main.py", line 3, in add
    return a + b
           ~~^~~
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

- 第 1 行
- 第 3 行：数字和文字不能直接相加
- 第 7 行：函数名写错了
- 报错信息看不出来

<details>
<summary>答案与解析</summary>

最后一个 `File` 是 `line 3`，错误类型 `TypeError` 说明 `int`（整数）和 `str`（字符串）不能用 `+` 相加。根本原因是第 7 行传入了文字 `"2"`，但报错的位置在第 3 行。
</details>

<!-- exercise: first-error -->
### 练习 2：先修哪个？

C 编译器一次报出了 12 个错误。最好的做法是？

- 从最后一个错误开始修
- 先修第一个错误，然后重新编译
- 一次把 12 个都改完再编译
- 换一个编译器

<details>
<summary>答案与解析</summary>

后面的错误往往是第一个错误引起的连锁反应。修好第一个、重新编译，常常其他错误就一起消失了。
</details>

<!-- exercise: fix-receipt -->
### 练习 3：修复购物小票

把 5.6 节的程序修好，让它输出：

```text
购物小票
苹果： 3 个
香蕉： 5 个
合计： 8 个
```

<details>
<summary>提示 1</summary>

第 4 行：两个要打印的东西之间少了逗号。
</details>

<details>
<summary>提示 2</summary>

第 5 行：变量名大小写不一致；第 6 行：少了右括号。
</details>

<!-- exercise: word-meaning -->
### 练习 4：报错词汇

报错信息 `expected ')'` 是什么意思？

- 多了一个右括号
- 这里应该有一个右括号（缺少右括号）
- 右括号是中文的
- 期望的输出是右括号

<details>
<summary>答案与解析</summary>

expected 的意思是「期望在这里看到」，也就是这里**缺少**一个 `)`。相反，`unexpected` 表示这里出现了**不该出现**的东西。
</details>

## 5.9 延伸阅读

- [Python 官方教程 · 错误和异常](https://docs.python.org/zh-cn/3/tutorial/errors.html)
- [如何提问的智慧（中文版）](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way)：经典文章，教你如何提出好问题。
- [小黄鸭调试法（维基百科）](https://zh.wikipedia.org/wiki/小黄鸭调试法)
- 下一课：[电脑如何存储数字和文字](../06-numbers-and-text/zh-CN.md)
