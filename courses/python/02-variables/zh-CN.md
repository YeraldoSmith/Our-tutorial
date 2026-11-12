# 2. 变量与数据类型

> 🎯 **本课目标**：用变量保存数据并修改它；掌握变量命名规则；认识整数、小数、字符串、布尔值和 `None`；学会用 `type()` 查看类型，用 `int()`、`str()` 等转换类型。
>
> ⏱ **预计用时**：40 分钟　｜　📋 **前置知识**：第 1 课

## 2.1 变量：给数据起个名字

上一课我们让 Python 直接打印数据。但真正的程序需要**记住**数据，之后再使用。比如一个游戏要记住玩家的分数、名字、血量。**变量**（variable）就是用来做这件事的：

```python
score = 100
name = "小明"
print(name, "的分数是", score)
```

输出：

```text
小明 的分数是 100
```

`score = 100` 的意思是：**创建一个叫 `score` 的名字，让它指向数据 `100`**。之后在程序里写 `score`，Python 就会用 `100` 来代替它。

可以把变量想象成一张**贴在数据上的标签**：

```text
   score ──►  100
   name  ──►  "小明"
```

> [!IMPORTANT]
> 这里的 `=` **不是数学里的「等于」**，而是「**赋值**」：把右边的值交给左边的名字。读的时候可以读作「score **得到** 100」。判断两个东西是否相等，要用两个等号 `==`（第 6 课会学）。

## 2.2 修改变量的值

变量之所以叫「变」量，是因为它的值可以改变：

```python
score = 100
print(score)
score = 120        # 标签换到新数据上
print(score)
```

输出：

```text
100
120
```

更常见的写法是**在原来的基础上**修改：

```python
score = 100
score = score + 10    # 先算右边：100 + 10 = 110，再把结果交给 score
print(score)
```

`score = score + 10` 在数学里是不成立的，但在编程里很正常：**先计算右边，再赋值给左边**。因为太常用，Python 提供了简写：

```python
score = 100
score += 10     # 等同于 score = score + 10
score -= 30     # 等同于 score = score - 30
score *= 2      # 等同于 score = score * 2
print(score)    # (100 + 10 - 30) * 2 = 160
```

### 变量之间赋值

```python
a = 5
b = a       # b 也指向 5
a = 8       # a 换成 8，b 不受影响
print(a, b)
```

输出 `8 5`。`b = a` 只是把 `a` **当时**的值交给 `b`，之后 `a` 再变，`b` 不会跟着变。

## 2.3 变量的命名规则

变量名不能随便起，必须遵守下面的**规则**（违反会报错）：

1. 只能包含**字母、数字和下划线 `_`**；
2. **不能以数字开头**：`player1` 可以，`1player` 不行；
3. **区分大小写**：`Score` 和 `score` 是两个不同的变量；
4. 不能使用 Python 的**关键字**，如 `if`、`for`、`class`、`True`、`None` 等。

还有一些**约定**（不遵守不会报错，但大家都这么写）：

- 用**小写字母加下划线**，叫「蛇形命名」（snake_case）：`player_name`、`max_speed`。
- 名字要**有意义**：`total_price` 比 `tp`、`a`、`x1` 好得多。
- 永远不变的值（常量）用**全大写**：`PI = 3.14159`、`MAX_LEVEL = 99`。

| 名字 | 能用吗？ | 说明 |
|------|----------|------|
| `age` | ✅ | |
| `user_name` | ✅ | 推荐的风格 |
| `userName` | ✅ | 能用，但 Python 习惯用下划线 |
| `_count` | ✅ | 下划线开头有特殊含义，后面会学 |
| `2nd_place` | ❌ | 不能以数字开头 |
| `my-name` | ❌ | `-` 会被当成减号 |
| `class` | ❌ | 是关键字 |
| `年龄` | ✅ | Python 3 允许中文名，但**不推荐**（不方便输入，别人也可能看不懂） |

> [!TIP]
> 起名字时问问自己：三个月后再看这段代码，能一眼看懂这个变量存的是什么吗？

## 2.4 数据类型

Python 里的每个值都有一个**类型**（type），类型决定了这个值能做什么。最基本的四种：

| 类型 | 名称 | 例子 | 用途 |
|------|------|------|------|
| `int` | 整数 | `42`、`-7`、`0` | 计数、年龄、分数 |
| `float` | 浮点数（小数） | `3.14`、`-0.5`、`2.0` | 价格、身高、温度 |
| `str` | 字符串（文字） | `"你好"`、`'abc'`、`""` | 名字、消息 |
| `bool` | 布尔值 | `True`、`False` | 是/否、开/关 |

用 `type()` 可以查看一个值的类型：

```python
print(type(42))
print(type(3.14))
print(type("hello"))
print(type(True))
```

输出：

```text
<class 'int'>
<class 'float'>
<class 'str'>
<class 'bool'>
```

### 几个要注意的地方

- `2` 是 `int`，`2.0` 是 `float`，`"2"` 是 `str`——**看起来一样，类型不同**。
- 布尔值 `True` 和 `False` **首字母大写**，写成 `true` 会报 `NameError`。
- 大数字可以用下划线分隔，更好读：`population = 1_400_000_000`（下划线会被忽略）。
- 还有一个特殊的值 **`None`**，表示「**什么都没有**」，类型是 `NoneType`。比如一个变量暂时还不知道该放什么时：`winner = None`。

### 动态类型

在 Python 里，变量本身没有固定的类型，**类型属于值**。同一个变量可以先指向整数，再指向字符串：

```python
x = 10
print(type(x))
x = "十"
print(type(x))
```

这叫**动态类型**。很方便，但也容易出错，所以好习惯是：一个变量始终存同一类东西。（Java、C、Rust 等语言是**静态类型**：变量声明时就确定类型，以后不能变。学到时你会看到区别。）

## 2.5 类型转换

不同类型之间经常需要转换。最常见的场景：**数字和文字不能直接拼在一起**。

```python norun
age = 18
print("我今年" + age + "岁")    # ❌ 报错
```

```text
TypeError: can only concatenate str (not "int") to str
```

意思是：字符串只能和字符串用 `+` 连接，不能和整数连接。解决办法是**把数字转成字符串**：

```python
age = 18
print("我今年" + str(age) + "岁")
```

常用的转换函数：

| 函数 | 作用 | 例子 | 结果 |
|------|------|------|------|
| `int(x)` | 转成整数 | `int("42")`、`int(3.9)` | `42`、`3`（直接去掉小数部分，**不是**四舍五入） |
| `float(x)` | 转成小数 | `float("3.5")`、`float(2)` | `3.5`、`2.0` |
| `str(x)` | 转成字符串 | `str(100)` | `"100"` |
| `bool(x)` | 转成布尔值 | `bool(0)`、`bool("hi")` | `False`、`True` |

```python
print(int("42") + 8)       # 50：先把文字 "42" 变成数字
print("42" + "8")          # 428：两个字符串拼接
print(int(3.9))            # 3
print(round(3.9))          # 4：四舍五入用 round
```

不是什么都能转换：`int("abc")` 和 `int("3.5")` 都会报 `ValueError`，因为它们不是整数的样子。

> [!NOTE]
> `bool()` 的规则：数字 `0`、`0.0`、空字符串 `""`、`None` 会变成 `False`，其他几乎所有值都是 `True`。第 6 课学条件判断时会用到。

## 2.6 同时给多个变量赋值

```python
x, y, z = 1, 2, 3        # 一次给三个变量赋值
print(x, y, z)

a = b = 0                # 两个变量都等于 0
print(a, b)
```

一个非常实用的技巧——**交换两个变量的值**：

```python
left = "苹果"
right = "香蕉"
left, right = right, left
print(left, right)       # 香蕉 苹果
```

在很多其他语言里，交换需要一个临时变量；Python 一行就搞定了。

## 2.7 f 字符串：把变量放进文字里

用 `+` 和 `str()` 拼接很麻烦。Python 提供了更好的方式——**f 字符串**（f-string）：在字符串前面加一个 `f`，就可以用 `{变量名}` 把变量直接放进去：

```python
name = "小明"
age = 18
height = 1.756
print(f"我叫{name}，今年{age}岁，身高{height}米")
print(f"明年我就{age + 1}岁了")            # 花括号里可以写表达式
print(f"身高约 {height:.2f} 米")           # :.2f 表示保留 2 位小数
```

输出：

```text
我叫小明，今年18岁，身高1.756米
明年我就19岁了
身高约 1.76 米
```

f 字符串是现代 Python 最常用的格式化方式，后面几乎每课都会用到。第 4 课会详细讲它的更多用法。

## 2.8 常见错误

### 使用前没有赋值

```python norun
print(total)
total = 100
```

```text
NameError: name 'total' is not defined
```

程序从上往下执行，第 1 行时 `total` 还不存在。**先赋值，后使用**。

### 拼写不一致

```python norun
user_name = "小红"
print(username)        # 少了下划线
```

同样是 `NameError`。Python 不会猜你的意思，名字必须一字不差。

### 数字和字符串相加

```python norun
count = 5
print("数量：" + count)
```

`TypeError`。用 `str(count)`，或者干脆用 f 字符串：`print(f"数量：{count}")`。

## 2.9 小结

- `变量名 = 值` 把值交给变量；`=` 是**赋值**，不是等于。
- 变量可以重新赋值；`x += 1` 是 `x = x + 1` 的简写。
- 命名规则：字母/数字/下划线，不能以数字开头，区分大小写，不能用关键字；习惯用 `snake_case`，常量全大写。
- 基本类型：`int`、`float`、`str`、`bool`，还有表示「没有」的 `None`；用 `type()` 查看。
- 类型转换：`int()`、`float()`、`str()`、`bool()`；`int()` 直接截断小数，四舍五入用 `round()`。
- `a, b = b, a` 交换变量；f 字符串 `f"…{变量}…"` 把变量放进文字。

## 2.10 练习

<!-- exercise: rectangle -->
### 练习 1：长方形面积

创建两个变量 `width = 8` 和 `height = 5`，计算面积并保存到变量 `area` 中，最后输出：

```text
40
```

<details>
<summary>提示</summary>

`area = width * height`，然后 `print(area)`。
</details>

<!-- exercise: swap -->
### 练习 2：交换

下面的起始代码中，`a` 是 `1`，`b` 是 `2`。不要修改前两行，在后面加一行代码交换它们的值，让程序输出：

```text
2 1
```

<details>
<summary>提示</summary>

Python 的同时赋值：`a, b = b, a`。
</details>

<!-- exercise: valid-name -->
### 练习 3：合法的变量名

下面哪一个是**合法**的 Python 变量名？

- `3d_model`
- `my-score`
- `max_speed`
- `for`

<details>
<summary>答案与解析</summary>

`max_speed`。`3d_model` 以数字开头；`my-score` 含有减号；`for` 是关键字。
</details>

<!-- exercise: predict-vars -->
### 练习 4：读代码，写输出

```python
x = 10
y = x
x = x + 5
y *= 2
print(x, y)
print("x" + "y")
print(int("7") + int(2.9))
```

<details>
<summary>答案与解析</summary>

`y = x` 时 `y` 得到 `10`；之后 `x` 变成 `15`，`y` 变成 `20`。`"x" + "y"` 是两个字符串拼接得到 `xy`。`int("7")` 是 `7`，`int(2.9)` 去掉小数得 `2`，合计 `9`。
</details>

<!-- exercise: fstring-card -->
### 练习 5：名片

已经有变量 `name = "Alice"`、`age = 20`、`city = "Paris"`。用 **f 字符串**输出：

```text
Name: Alice, Age: 20, City: Paris
```

<details>
<summary>提示</summary>

`print(f"Name: {name}, Age: {age}, City: {city}")`
</details>

<!-- exercise: types -->
### 练习 6：类型

`type("3.14")` 的结果是什么？

- `<class 'float'>`
- `<class 'int'>`
- `<class 'str'>`
- `<class 'bool'>`

<details>
<summary>答案与解析</summary>

有引号，所以是字符串 `str`。看起来像数字的文字，仍然是文字。
</details>

## 2.11 延伸阅读

- [Python 官方教程 · 3.1 把 Python 当计算器](https://docs.python.org/zh-cn/3/tutorial/introduction.html)
- [PEP 8 命名约定](https://peps.python.org/pep-0008/#naming-conventions)：Python 官方的代码风格指南（英文）。
- 下一课：[数字与计算](../03-numbers-math/zh-CN.md)
