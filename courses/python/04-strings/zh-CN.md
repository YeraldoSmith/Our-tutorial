# 4. 字符串

> 🎯 **本课目标**：用下标和切片取出字符串的任意部分；掌握 `len`、`in` 和十几个最常用的字符串方法；理解「字符串不可修改」；熟练使用 f 字符串格式化。
>
> ⏱ **预计用时**：45 分钟　｜　📋 **前置知识**：第 2、3 课

文字处理是编程中最常见的任务之一：处理用户名、分析文章、生成报表、清洗数据……这一课把字符串（`str`）学透。

## 4.1 字符串是一串字符

字符串就是**按顺序排列的字符**。每个字符都有一个位置编号，叫**下标**（index），**从 0 开始**：

```text
 字符串:   P   y   t   h   o   n
 下标:     0   1   2   3   4   5
 负下标:  -6  -5  -4  -3  -2  -1
```

用方括号 `[下标]` 取出某个字符：

```python
word = "Python"
print(word[0])     # P：第一个字符
print(word[3])     # h
print(word[-1])    # n：负数从后往前数，-1 是最后一个
print(word[-2])    # o
print(len(word))   # 6：长度（字符个数）
```

> [!IMPORTANT]
> 下标**从 0 开始**，所以长度为 6 的字符串，最后一个字符的下标是 5（= 长度 − 1）。这是所有主流编程语言的共同习惯，一开始容易搞错，多练几次就习惯了。

访问不存在的下标会报错：

```python norun
word = "Python"
print(word[6])
```

```text
IndexError: string index out of range
```

## 4.2 切片：取出一段

`字符串[开始:结束]` 取出从「开始」到「结束」**之前**的部分（**包含开始，不包含结束**）：

```python
word = "Python"
print(word[0:2])    # Py：下标 0、1
print(word[2:5])    # tho：下标 2、3、4
print(word[:3])     # Pyt：省略开始 = 从头开始
print(word[3:])     # hon：省略结束 = 一直到末尾
print(word[-3:])    # hon：最后三个字符
print(word[:])      # Python：整个字符串
```

可以这样记：**切片的位置是「字符之间的缝隙」**：

```text
   | P | y | t | h | o | n |
   0   1   2   3   4   5   6
word[2:5] 就是从缝隙 2 切到缝隙 5 → "tho"
```

### 步长

第三个数字是**步长**（每次跳几个）：

```python
digits = "0123456789"
print(digits[::2])     # 02468：每隔一个取一个
print(digits[1::2])    # 13579
print(digits[::-1])    # 9876543210：步长为 -1，反过来！
```

`s[::-1]` 是反转字符串的经典写法。

切片和下标不同：**切片越界不会报错**，只是取到能取的部分：`"abc"[1:100]` 得到 `"bc"`。

## 4.3 字符串不可修改

字符串一旦创建，就**不能修改**其中的字符：

```python norun
word = "Python"
word[0] = "J"
```

```text
TypeError: 'str' object does not support item assignment
```

想「修改」，就创建一个新的字符串：

```python
word = "Python"
word = "J" + word[1:]
print(word)     # Jython
```

这个特性叫**不可变**（immutable）。下面要学的所有字符串方法，都是**返回一个新字符串**，原来的字符串不变：

```python
name = "alice"
upper_name = name.upper()
print(name)          # alice：原字符串没变
print(upper_name)    # ALICE
```

## 4.4 拼接、重复和包含

```python
first = "Hello"
second = "World"
print(first + ", " + second + "!")   # 拼接：Hello, World!
print("ha" * 3)                       # 重复：hahaha
print("-" * 20)                       # 画一条分隔线
print("ell" in first)                 # True：判断是否包含
print("xyz" not in first)             # True
```

## 4.5 常用字符串方法

**方法**（method）是属于某个值的函数，用 `值.方法名()` 调用。字符串有几十个方法，下面是最常用的：

### 大小写

```python
s = "hello World"
print(s.upper())        # HELLO WORLD
print(s.lower())        # hello world
print(s.title())        # Hello World：每个单词首字母大写
print(s.capitalize())   # Hello world：只有第一个字母大写
```

### 去掉空白

用户输入的内容经常带着多余的空格或换行：

```python
s = "   python   \n"
print("[" + s.strip() + "]")    # [python]：去掉两边的空白
print("[" + s.lstrip() + "]")   # 只去左边
print("[" + s.rstrip() + "]")   # 只去右边
```

### 查找和替换

```python
s = "I like cats. Cats are cute."
print(s.find("cats"))           # 7：第一次出现的下标
print(s.find("dogs"))           # -1：找不到返回 -1
print(s.count("ats"))           # 2：出现了几次
print(s.replace("cats", "dogs"))   # 替换：I like dogs. Cats are cute.
print(s.startswith("I"))        # True
print(s.endswith("."))          # True
```

注意 `replace` 和 `find` **区分大小写**：`"Cats"` 没有被替换。

### 分割和连接

这一对方法极其常用：

```python
sentence = "apple banana cherry"
words = sentence.split()        # 按空白分割
print(words)                    # ['apple', 'banana', 'cherry']

date = "2026-11-12"
parts = date.split("-")         # 按 "-" 分割
print(parts)                    # ['2026', '11', '12']

print("/".join(parts))          # 2026/11/12：用 "/" 连接起来
print(" + ".join(["1", "2", "3"]))   # 1 + 2 + 3
```

`split` 得到的是一个**列表**（方括号括起来的一组值，第 8 课详细讲）。`join` 反过来，把一组字符串用指定的分隔符连起来。

### 判断内容

```python
print("2026".isdigit())    # True：全是数字
print("abc".isalpha())     # True：全是字母
print("abc123".isalnum())  # True：字母或数字
print("   ".isspace())     # True：全是空白
print("Hello".islower())   # False
```

### 方法速查表

| 方法 | 作用 |
|------|------|
| `s.upper()` / `s.lower()` | 转大写 / 小写 |
| `s.strip()` | 去掉两端空白（也可以 `strip("x")` 去掉指定字符） |
| `s.replace(a, b)` | 把 a 全部替换成 b |
| `s.find(x)` | x 第一次出现的下标，没有则 -1 |
| `s.count(x)` | x 出现的次数 |
| `s.split(sep)` | 按分隔符切成列表（不写 sep 就按空白） |
| `sep.join(列表)` | 用 sep 把列表里的字符串连起来 |
| `s.startswith(x)` / `s.endswith(x)` | 是否以 x 开头 / 结尾 |
| `s.isdigit()` / `s.isalpha()` | 是否全是数字 / 字母 |
| `s.center(w)` / `s.ljust(w)` / `s.rjust(w)` | 在宽度 w 内居中 / 左对齐 / 右对齐 |
| `s.zfill(w)` | 左边补 0 到宽度 w |

方法可以**连着调用**：

```python
raw = "  Hello, PYTHON!  "
print(raw.strip().lower().replace("!", "?"))   # hello, python?
```

## 4.6 f 字符串进阶

第 2、3 课已经用过 f 字符串。再补充几个实用技巧：

```python
name = "Bob"
score = 87.456
print(f"{name:>10}|")          # 右对齐，宽度 10
print(f"{name:<10}|")          # 左对齐
print(f"{name:^10}|")          # 居中
print(f"{name:*^10}")          # 居中，用 * 填充：***Bob****
print(f"{score:8.2f}|")        # 宽度 8，两位小数
print(f"{score=}")             # 调试小技巧：输出 score=87.456
```

做一个整齐的表格：

```python
print(f"{'商品':<6}{'数量':>4}{'价格':>8}")
print(f"{'苹果':<6}{3:>4}{4.5:>8.2f}")
print(f"{'香蕉':<6}{12:>4}{0.99:>8.2f}")
```

> [!NOTE]
> 中文字符在等宽字体里通常占两个英文字符的宽度，所以中英文混排的表格可能对不太齐，这是正常的。

### 转义和原始字符串

复习第 1 课的转义字符：`\n` 换行、`\t` 制表符、`\\` 反斜杠、`\"` 引号。如果字符串里有很多反斜杠（例如 Windows 路径、正则表达式），可以在前面加 `r`，表示**原始字符串**，反斜杠不再转义：

```python
print("C:\\new\\table")     # 普通字符串需要写两个反斜杠
print(r"C:\new\table")      # 原始字符串：所见即所得
```

## 4.7 遍历字符串（预告）

想对字符串的**每个字符**做点事情，可以用 `for` 循环（第 7 课详细讲）：

```python
for ch in "abc":
    print(ch.upper())
```

输出 `A`、`B`、`C` 三行。先混个眼熟。

## 4.8 小结

- 下标从 **0** 开始，`s[-1]` 是最后一个字符；`len(s)` 是长度。
- 切片 `s[开始:结束:步长]`：包含开始、不包含结束；`s[::-1]` 反转。
- 字符串**不可修改**，所有方法都返回新字符串。
- `+` 拼接，`*` 重复，`in` 判断包含。
- 常用方法：`upper/lower`、`strip`、`replace`、`find/count`、`split/join`、`startswith/endswith`、`isdigit/isalpha`。
- f 字符串：`{x:>10}` 对齐、`{x:.2f}` 小数、`{x=}` 调试；`r"..."` 是原始字符串。

## 4.9 练习

<!-- exercise: reverse -->
### 练习 1：反转字符串

变量 `s = "stressed"`。输出它反过来的样子：

```text
desserts
```

<details>
<summary>提示</summary>

切片的步长写 `-1`：`s[::-1]`。
</details>

<!-- exercise: slice-predict -->
### 练习 2：读代码，写输出

```python
s = "Hello, World"
print(s[0], s[-1])
print(s[7:12])
print(s[:5].upper())
print(len(s))
```

<details>
<summary>答案与解析</summary>

`s[0]` 是 `H`，`s[-1]` 是 `d`；下标 7 到 11 是 `World`；前 5 个字符 `Hello` 转大写是 `HELLO`；包括逗号和空格共 12 个字符。
</details>

<!-- exercise: clean-name -->
### 练习 3：整理用户名

变量 `raw = "   aLiCe SMITH  "`。去掉两边的空格，并让每个单词首字母大写，输出：

```text
Alice Smith
```

<details>
<summary>提示</summary>

`raw.strip().title()`：先去空白，再转换大小写。
</details>

<!-- exercise: date-format -->
### 练习 4：日期格式转换

变量 `date = "2026-11-12"`（年-月-日）。把它转换成「日/月/年」的格式输出：

```text
12/11/2026
```

<details>
<summary>提示 1</summary>

`parts = date.split("-")` 得到 `['2026', '11', '12']`，`parts[0]` 是年，`parts[2]` 是日。
</details>

<details>
<summary>提示 2</summary>

`print(parts[2] + "/" + parts[1] + "/" + parts[0])`，或者 `"/".join([parts[2], parts[1], parts[0]])`。
</details>

<!-- exercise: immutable -->
### 练习 5：不可变

`s = "cat"`，执行 `s.upper()` 之后再 `print(s)`，会输出什么？

- `CAT`
- `cat`
- 报错
- `Cat`

<details>
<summary>答案与解析</summary>

`cat`。`upper()` 返回了一个新字符串 `"CAT"`，但我们没有把它保存下来，`s` 本身没有变。要改变 `s`，得写 `s = s.upper()`。
</details>

<!-- exercise: count-vowels -->
### 练习 6：数元音

变量 `text = "programming is fun"`。分别统计字母 `a`、`i`、`o`、`u` 出现的次数，输出它们的**总和**：

```text
5
```

<details>
<summary>提示</summary>

`text.count("a") + text.count("i") + ...`
</details>

## 4.10 延伸阅读

- [Python 官方文档 · 字符串方法](https://docs.python.org/zh-cn/3/library/stdtypes.html#string-methods)：所有方法的完整列表。
- [Python 官方教程 · 3.1.2 字符串](https://docs.python.org/zh-cn/3/tutorial/introduction.html#text)
- 下一课：[输入与交互](../05-input/zh-CN.md)
