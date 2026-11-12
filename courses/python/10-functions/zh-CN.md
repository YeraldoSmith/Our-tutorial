# 10. 函数

> 🎯 **本课目标**：用 `def` 定义自己的函数，理解**参数**和**返回值**；分清 `return` 和 `print`；会用默认参数、关键字参数和多个返回值；理解**局部变量**和**全局变量**；写出带**类型标注**和文档字符串的现代 Python 函数；知道「函数也是值」，会用 `lambda`；初步认识**递归**；避开「可变默认参数」的陷阱。
>
> ⏱ **预计用时**：75 分钟　｜　📋 **前置知识**：第 6～9 课

## 10.1 为什么需要函数？

你已经用过很多函数了：`print()`、`len()`、`input()`、`sorted()`……它们把一段做好的功能**打包起来、起个名字**，用的时候只要写名字。

自己写函数有三个好处：

1. **不重复**：同样的代码写一次，到处调用；要改也只改一处。
2. **好读**：`if is_prime(n):` 比一大段循环更容易看懂。
3. **好测试**：每个函数单独检查对不对，出了问题容易定位。

## 10.2 定义和调用

```python
def greet(name):                  # def 函数名(参数):
    print(f"你好，{name}！")       # 函数体：缩进 4 个空格

greet("小明")                      # 调用：函数名(实参)
greet("小红")
```

- `def` 是 **define**（定义）的缩写；
- `name` 是**参数**（parameter）：调用时传进来的值会放进这个变量；
- 函数体要**缩进**，和 `if`、`for` 一样；
- 定义函数时**不会执行**函数体，只有**调用**时才执行。所以必须先定义、后调用。

## 10.3 返回值：`return`

函数可以把结果「交回」给调用它的地方：

```python
def area(width, height):
    return width * height          # 把结果返回，函数到这里就结束了

a = area(3, 4)                     # 返回值可以存进变量
print(a)                           # 12
print(area(2, 5) + area(1, 1))     # 也可以直接参与计算：11
```

### `return` 和 `print` 的区别

这是初学者最容易混淆的地方：

```python
def add_print(a, b):
    print(a + b)        # 只是显示在屏幕上

def add_return(a, b):
    return a + b        # 把结果交回去

x = add_print(1, 2)     # 屏幕上显示 3，但 x 得到的是 None
y = add_return(1, 2)    # 屏幕上什么也不显示，y 得到 3
print("x =", x)
print("y =", y)
```

`print` 是给**人**看的；`return` 是给**程序**用的。一个没有 `return`（或者只写了 `return`）的函数，返回值是 `None`。

## 10.4 参数的各种写法

```python
def power(base, exp=2):                 # exp 有默认值：不传就用 2
    return base ** exp

print(power(5))                          # 25：用默认值
print(power(5, 3))                       # 125：按位置传参
print(power(exp=3, base=2))              # 8：按名字传参（关键字参数），顺序可以随意

def describe(name, age, city="北京"):
    return f"{name}，{age} 岁，住在{city}"

print(describe("小明", 12))
print(describe("小红", 13, city="上海"))
```

规则：**有默认值的参数必须放在没有默认值的参数后面**。调用时，按位置传的参数要写在按名字传的前面。

## 10.5 返回多个值

用逗号隔开多个值，返回的其实是一个**元组**，可以直接解包（第 8 课）：

```python
def min_max(nums):
    return min(nums), max(nums)

lo, hi = min_max([3, 9, 1, 7])
print(lo, hi)            # 1 9

def divide(a, b):
    return a // b, a % b         # 商和余数

q, r = divide(17, 5)
print(f"17 = 5 × {q} + {r}")
```

## 10.6 局部变量和全局变量

在函数**里面**创建的变量是**局部变量**，只在这个函数里有效，函数结束就消失了：

```python
def f():
    secret = 42           # 局部变量
    return secret

print(f())
# print(secret)           # 这里会报 NameError：外面看不到函数里的变量
```

在函数**外面**创建的变量是**全局变量**，函数里可以**读取**它。但如果在函数里**给它赋值**，Python 会认为你创建了一个同名的局部变量：

```python
count = 0                 # 全局变量

def show():
    print("count =", count)    # 只读取：没问题

show()
```

```python norun
count = 0

def increase():
    count = count + 1     # 想修改全局变量……
    return count

increase()
```

```text
UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
```

可以用 `global count` 声明要修改全局变量，但**更好的做法是：通过参数传进去，通过 `return` 传出来**：

```python
def increase(count):
    return count + 1

count = 0
count = increase(count)
count = increase(count)
print(count)              # 2
```

这样函数只依赖它的参数，更容易理解和测试。

## 10.7 现代写法：类型标注和文档字符串

现代 Python 代码通常会写上**类型标注**（type hints）和**文档字符串**（docstring）：

```python
def bmi(weight_kg: float, height_m: float) -> float:
    """计算身体质量指数 BMI = 体重 / 身高²。"""
    return weight_kg / height_m ** 2

print(f"{bmi(60, 1.70):.1f}")
print(bmi.__doc__)        # 文档字符串保存在 __doc__ 里；在交互环境中用 help(bmi) 查看更完整的说明
```

- `weight_kg: float` 表示「这个参数应该是小数」，`-> float` 表示「返回一个小数」；
- 三引号里的字符串是文档字符串，说明函数做什么。

> [!NOTE]
> 类型标注**不会**被 Python 强制检查，传错类型照样能运行。它是写给人和工具看的：编辑器能据此给出提示，`mypy`、`pyright` 等工具能提前发现类型错误（第 15 课会用到）。

## 10.8 函数也是值

函数和数字、字符串一样，可以存进变量、当作参数传给别的函数：

```python
def double(x):
    return x * 2

f = double                     # 注意没有括号：这是函数本身，不是调用它
print(f(21))                   # 42

def apply_twice(func, value):
    return func(func(value))

print(apply_twice(double, 3))  # 12
```

只用一次的小函数，可以用 **`lambda`** 写成一行：`lambda 参数: 表达式`。它最常见的用途是给 `sorted`、`max`、`min` 指定比较的依据（第 9 课已经见过）：

```python
words = ["banana", "kiwi", "apple", "fig"]
print(sorted(words, key=len))                    # 按长度排序
print(sorted(words, key=lambda w: w[-1]))        # 按最后一个字母排序

students = [("小明", 88), ("小红", 95), ("小刚", 72)]
print(max(students, key=lambda s: s[1]))         # 分数最高的人
```

## 10.9 递归：函数调用自己

一个函数可以**调用它自己**，这叫**递归**（recursion）。比如阶乘 `n! = n × (n−1)!`，并且 `0! = 1`：

```python
def factorial(n: int) -> int:
    if n == 0:                    # 终止条件：必须有，否则会无限调用下去
        return 1
    return n * factorial(n - 1)   # 把问题变成一个更小的同类问题

print(factorial(5))               # 5 × 4 × 3 × 2 × 1 = 120
```

调用过程是这样展开的：

```text
factorial(3)
= 3 * factorial(2)
= 3 * (2 * factorial(1))
= 3 * (2 * (1 * factorial(0)))
= 3 * (2 * (1 * 1))
= 6
```

递归写起来很优雅，适合处理「大问题包含同样的小问题」的情况（比如文件夹里的文件夹）。但如果忘了终止条件，或者递归太深（Python 默认大约 1000 层），就会出现 `RecursionError`。很多递归也可以改写成循环。

## 10.10 陷阱：可变的默认参数

```python
def add_item(item, box=[]):        # 危险！默认的列表只在定义函数时创建一次
    box.append(item)
    return box

print(add_item("苹果"))            # ['苹果']
print(add_item("香蕉"))            # ['苹果', '香蕉']  ← 上一次的「苹果」还在！
```

默认值只在**定义函数时**计算一次，之后每次调用都共用同一个列表。正确的写法是用 `None` 做默认值：

```python
def add_item(item, box=None):
    if box is None:
        box = []                   # 每次调用都新建一个列表
    box.append(item)
    return box

print(add_item("苹果"))
print(add_item("香蕉"))
```

**默认值不要用列表、字典、集合这类可变的东西。**

## 10.11 常见错误

```python norun
def area(w, h):
    return w * h

print(area(3))
```

```text
TypeError: area() missing 1 required positional argument: 'h'
```

参数个数不对：定义时有 2 个参数，调用时只给了 1 个。

```python norun
print(square(4))

def square(x):
    return x * x
```

```text
NameError: name 'square' is not defined
```

调用发生在定义之前。把函数定义移到调用的上面。

```python norun
def square(x):
    x * x              # 忘了 return

print(square(4) + 1)
```

```text
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

函数没有 `return`，返回了 `None`，`None + 1` 就报错了。

## 10.12 小结

- `def 名字(参数):` 定义函数，缩进的部分是函数体；先定义，后调用。
- `return` 把结果交回给调用者并结束函数；没有 `return` 就返回 `None`。`print` 只是显示。
- 默认参数 `def f(x, y=2)`；关键字参数 `f(y=3, x=1)`；返回多个值得到元组，可以解包。
- 函数里的变量是局部的；想修改外面的值，最好用参数传进、`return` 传出，而不是 `global`。
- 类型标注 `def f(x: int) -> str:` 和文档字符串让代码更清楚。
- 函数是值，可以当参数传递；`lambda` 写一行的小函数，常用于 `sorted(key=...)`。
- 递归 = 函数调用自己，必须有终止条件。
- 默认参数不要用可变对象，用 `None` 代替。

## 10.13 练习

<!-- exercise: max3 -->
### 练习 1：三个数的最大值

补全函数 `max3(a, b, c)`，返回三个数中最大的一个（**不要**用内置的 `max`，用 `if` 来比较）。程序会读入一行三个整数并输出 `max3` 的结果。

```text
样例输入：3 9 4
样例输出：9
```

<details>
<summary>提示</summary>

先假设 `biggest = a`，再分别和 `b`、`c` 比较，谁更大就更新 `biggest`，最后 `return biggest`。
</details>

<!-- exercise: is-prime -->
### 练习 2：质数判断

补全函数 `is_prime(n)`：`n` 是质数就返回 `True`，否则返回 `False`。（质数是大于 1、只能被 1 和自己整除的整数。）程序读入一个整数，是质数输出 `yes`，否则输出 `no`。

```text
样例输入：97
样例输出：yes
```

<details>
<summary>提示</summary>

小于 2 直接返回 `False`；然后用 `for d in range(2, int(n ** 0.5) + 1):` 检查，能整除就返回 `False`；循环结束都没整除，返回 `True`。
</details>

<!-- exercise: grade -->
### 练习 3：分数等级

补全函数 `grade(score)`，把分数换成等级：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。输入一行若干个分数，输出对应的等级，空格隔开。

```text
样例输入：95 82 60 59
样例输出：A B D F
```

<details>
<summary>提示</summary>

在函数里用 `if / elif / else`，从高到低判断；主程序里 `print(" ".join(grade(s) for s in scores))`。
</details>

<!-- exercise: factorial -->
### 练习 4：递归求阶乘

用**递归**补全函数 `factorial(n)`。输入一个整数 `n`（0 ≤ n ≤ 20），输出 `n!`。

```text
样例输入：5
样例输出：120
```

<details>
<summary>提示</summary>

终止条件 `if n == 0: return 1`，否则 `return n * factorial(n - 1)`。
</details>

<!-- exercise: scope-predict -->
### 练习 5：读代码，写输出

```python
x = 10

def f(x):
    x = x + 1
    return x

y = f(x)
print(x, y)

def g():
    return

print(g())
```

<details>
<summary>答案与解析</summary>

函数 `f` 里的 `x` 是参数（局部变量），修改它不影响外面的 `x`，所以输出 `10 11`；`g` 只写了 `return`，返回 `None`。
</details>

<!-- exercise: mutable-default -->
### 练习 6：可变默认参数

下面的程序输出什么？

```python
def add(item, box=[]):
    box.append(item)
    return box

add(1)
print(add(2))
```

- `[2]`
- `[1, 2]`
- 报错
- `None`

<details>
<summary>答案与解析</summary>

`[1, 2]`。默认的列表只创建一次，第一次调用放进去的 1 还在。应该用 `box=None`，在函数里再新建列表。
</details>

## 10.14 延伸阅读

- [Python 官方教程 · 4.8 定义函数](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#defining-functions)
- [typing 模块：类型标注](https://docs.python.org/zh-cn/3/library/typing.html)
- 下一课：[模块与文件](../11-modules-files/zh-CN.md)
