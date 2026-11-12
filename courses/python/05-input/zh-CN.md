# 5. 输入与交互

> 🎯 **本课目标**：用 `input()` 读取用户输入；理解「输入得到的永远是字符串」，并用 `int()`、`float()` 转换；一行读取多个值；写出第一批真正能和人交互的程序；了解做题时输入是怎么提供的。
>
> ⏱ **预计用时**：35 分钟　｜　📋 **前置知识**：第 2～4 课

## 5.1 input()：让程序听你说话

到目前为止，程序里的数据都是写死的。要让程序和用户**交互**，需要读取输入。`input()` 会**暂停程序**，等用户输入一行文字并按回车，然后把这行文字交给程序：

```python
name = input("你叫什么名字？")
print(f"你好，{name}！")
```

运行后，程序会显示 `你叫什么名字？` 然后停下来等待。在学习程序里，**在下方的输入框中输入**（比如 `小明`）并按回车，程序就会继续：

```text
你叫什么名字？小明
你好，小明！
```

括号里的文字叫**提示语**（prompt），会先显示出来，告诉用户该输入什么。也可以不写提示语：`name = input()`。

> [!TIP]
> 如果你知道程序会要什么输入，也可以**在点「▶ 运行」之前**就把内容输入到输入框里，按回车；程序开始运行时会自动收到这些输入。

## 5.2 输入得到的永远是字符串

这一点**非常重要**，也是新手最常踩的坑：

```python norun
age = input("你几岁了？")
print(age + 1)
```

输入 `18`，你会得到：

```text
TypeError: can only concatenate str (not "int") to str
```

为什么？因为 `input()` 返回的**永远是字符串**——即使用户输入的是数字，得到的也是文字 `"18"`，而不是数字 `18`。要做计算，必须先**转换**：

```python
age = int(input("你几岁了？"))
print(f"明年你就 {age + 1} 岁了")
```

`int(input(...))` 是一个**嵌套调用**：先执行里面的 `input()` 得到文字，再交给外面的 `int()` 转成整数。

| 需要的类型 | 写法 |
|------------|------|
| 文字 | `s = input()` |
| 整数 | `n = int(input())` |
| 小数 | `x = float(input())` |

## 5.3 一个简单的计算器

```python
a = float(input("第一个数："))
b = float(input("第二个数："))
print(f"{a} + {b} = {a + b}")
print(f"{a} - {b} = {a - b}")
print(f"{a} × {b} = {a * b}")
print(f"{a} ÷ {b} = {a / b:.4f}")
```

试着运行并输入 `7` 和 `2`。用 `float` 而不是 `int`，这样用户输入小数也没问题。（如果第二个数输入 0，最后一行会报 `ZeroDivisionError`——下一课学了 `if` 就能处理了。）

## 5.4 一行输入多个值

有时希望用户在一行里输入好几个数，用空格隔开，比如 `3 4 5`。用 `split()` 切开，再分别转换：

```python input="3 4 5"
line = input("输入三角形的三条边（用空格隔开）：")
parts = line.split()          # "3 4 5" → ['3', '4', '5']
a = int(parts[0])
b = int(parts[1])
c = int(parts[2])
print("周长：", a + b + c)
```

更简洁的写法是用 `map`，它把一个函数应用到每一个元素上：

```python input="3 4 5"
a, b, c = map(int, input().split())
print(a + b + c)
```

`map(int, ...)` 对 `split()` 得到的每一段都调用 `int()`，再用同时赋值分给 `a`、`b`、`c`。这是做编程题时**最常用**的读入方式之一，记下来！

## 5.5 清理输入

用户经常不小心多打空格。用 `strip()` 去掉首尾空白，用 `lower()` 统一大小写，可以让程序更「宽容」：

```python
answer = input("你喜欢 Python 吗？(yes/no) ").strip().lower()
print(f"你的回答是：{answer}")
```

输入 `  YES ` 也能得到 `yes`。

> [!NOTE]
> `int()` 本身会忽略首尾空白：`int(" 42 ")` 是 `42`。但 `int("4 2")`（中间有空格）或 `int("42.0")` 都会报错。

## 5.6 做编程题时的输入

练习里的编程题，判题程序会**自动提供输入**，然后比较你的输出和期望输出是否**完全一样**。所以：

> [!IMPORTANT]
> 做编程题时，**`input()` 里不要写提示语**。提示语也会被打印出来，混进你的输出，导致「答案错误」。在练习里写 `n = int(input())`，而不是 `n = int(input("请输入："))`。

题目会这样描述输入和输出：

```text
输入：两行，每行一个整数
输出：它们的和

样例输入：        样例输出：
3                 8
5
```

样例输入的**每一行**对应程序里的一次 `input()`。

## 5.7 完整例子：温度换算

```python
c = float(input())          # 摄氏度
f = c * 9 / 5 + 32
print(f"{c:.1f}°C = {f:.1f}°F")
```

输入 `36.6`，输出 `36.6°C = 97.9°F`。

## 5.8 常见错误

### 忘记转换类型

```python norun
a = input()
b = input()
print(a + b)     # 输入 3 和 5，结果是 35！
```

不会报错，但结果不对：两个**字符串**相加是拼接。记得 `int(input())`。

### 输入的内容无法转换

```python norun
n = int(input())     # 用户输入了 "abc" 或 "3.5"
```

```text
ValueError: invalid literal for int() with base 10: 'abc'
```

第 12 课会学 `try/except` 来优雅地处理这种情况。

### 读取次数比输入的行数多

```text
EOFError: EOF when reading a line
```

EOF 意思是「输入结束了」。程序想再读一行，但已经没有输入了。检查 `input()` 的次数是否和题目给的输入行数一致，或者一行里的多个值是否应该用 `split()` 一次读入。

## 5.9 小结

- `input("提示语")` 暂停程序、读取一行输入；在学习程序里用下方输入框输入。
- `input()` **永远返回字符串**；计算前用 `int()` / `float()` 转换。
- 一行多个值：`a, b = map(int, input().split())`。
- `strip()`、`lower()` 让输入处理更宽容。
- 做编程题时 `input()` **不写提示语**；样例输入的每一行对应一次 `input()`。

## 5.10 练习

<!-- exercise: greet -->
### 练习 1：打招呼

读入一行名字，输出 `Hello, 名字!`。

```text
样例输入：Alice
样例输出：Hello, Alice!
```

<details>
<summary>提示</summary>

`name = input()`，然后 `print(f"Hello, {name}!")`。记住：不要在 `input()` 里写提示语。
</details>

<!-- exercise: sum-two -->
### 练习 2：两数之和

输入两行，每行一个整数，输出它们的和。

```text
样例输入：
3
5
样例输出：
8
```

<details>
<summary>提示</summary>

`a = int(input())`，`b = int(input())`，`print(a + b)`。
</details>

<!-- exercise: rectangle-line -->
### 练习 3：一行两个数

输入**一行**，包含长方形的长和宽（整数，空格隔开），输出面积和周长，用空格隔开。

```text
样例输入：4 7
样例输出：28 22
```

<details>
<summary>提示</summary>

`w, h = map(int, input().split())`
</details>

<!-- exercise: celsius -->
### 练习 4：温度换算

输入一个摄氏温度（可能是小数），输出对应的华氏温度，**保留 1 位小数**。公式：`F = C × 9 / 5 + 32`。

```text
样例输入：36.6
样例输出：97.9
```

<details>
<summary>提示</summary>

`c = float(input())`，用 `print(f"{f:.1f}")` 输出。
</details>

<!-- exercise: input-type -->
### 练习 5：input 的类型

用户输入 `42` 后，`x = input()` 中 `x` 的类型是什么？

- `int`
- `float`
- `str`
- 取决于用户输入了什么

<details>
<summary>答案与解析</summary>

`str`。`input()` 永远返回字符串，即使看起来是数字。需要数字时用 `int(x)` 转换。
</details>

## 5.11 延伸阅读

- [Python 官方文档 · input()](https://docs.python.org/zh-cn/3/library/functions.html#input)
- [Python 官方文档 · map()](https://docs.python.org/zh-cn/3/library/functions.html#map)
- 下一课：[条件判断](../06-conditions/zh-CN.md)
