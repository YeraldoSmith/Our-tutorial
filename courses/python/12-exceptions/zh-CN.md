# 12. 异常处理

> 🎯 **本课目标**：分清「语法错误」和运行时的「异常」；用 `try / except` 接住异常，让程序在用户输错时不会崩溃；会用 `else`、`finally` 和 `as e`；在自己的函数里用 `raise` 报告错误；认识常见的异常类型和它们的层级；知道什么时候该用异常、什么时候该用 `if`；不写「什么都接」的 `except:`。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：入门课第 5 课（读懂报错），本课程第 10、11 课

## 12.1 错误分两种

入门课第 5 课里，我们学会了读懂报错信息。Python 的错误大致分两种：

1. **语法错误**（`SyntaxError`、`IndentationError`）：代码写得不符合 Python 的语法，**程序根本不会开始运行**。只能改代码。
2. **异常**（exception）：语法没问题，但**运行到某一行时出了状况**——除以 0、把 `"abc"` 转成整数、打开不存在的文件……

```python norun
age = int(input("你几岁？"))     # 用户输入了「十二」
print(age + 1)
```

```text
Traceback (most recent call last):
  File "main.py", line 1, in <module>
    age = int(input("你几岁？"))
ValueError: invalid literal for int() with base 10: '十二'
```

这里的 `ValueError` 就是一种异常。默认情况下，异常会让程序**立刻停止**并打印出 Traceback。但很多异常是可以预料的（用户总会输错），我们希望程序能**自己处理**，而不是崩溃。

## 12.2 `try / except`：接住异常

```python input="十二"
text = input("你几岁？")
try:
    age = int(text)                      # 可能出错的代码放在 try 里
    print("明年你就", age + 1, "岁了")
except ValueError:                       # 如果出现 ValueError，就执行这里
    print(f"「{text}」不是一个整数，请用阿拉伯数字输入，比如 12")
print("程序继续运行")
```

执行过程：

1. 先执行 `try` 里的代码；
2. 如果**没有**出错，跳过 `except`；
3. 如果出了 `ValueError`，`try` 里剩下的代码**不再执行**，直接跳到 `except ValueError:` 里；
4. 无论哪种情况，之后的代码都会继续运行。

一个常见的用法：**一直问，直到输入正确为止**：

```python input="abc\n-5\n12"
while True:
    try:
        age = int(input("请输入年龄："))
        if age < 0:
            print("年龄不能是负数")
            continue
        break                                # 输入正确，跳出循环
    except ValueError:
        print("请输入整数")
print("你的年龄是", age)
```

## 12.3 接住不同的异常

一段代码可能出现好几种异常，可以分别处理：

```python
def divide(a_text, b_text):
    try:
        a, b = float(a_text), float(b_text)
        return a / b
    except ValueError:
        return "请输入数字"
    except ZeroDivisionError:
        return "除数不能是 0"

print(divide("10", "4"))
print(divide("10", "abc"))
print(divide("10", "0"))
```

几种异常想用同样的方式处理，可以写在一个元组里：`except (ValueError, ZeroDivisionError):`。

### 拿到异常的详细信息：`as e`

```python
try:
    nums = [1, 2, 3]
    print(nums[10])
except IndexError as e:
    print("出错了：", e)               # e 是异常对象，打印出来就是错误说明
    print("异常类型：", type(e).__name__)
```

## 12.4 `else` 和 `finally`

完整的结构是这样的：

```python
def read_number(text):
    try:
        n = int(text)
    except ValueError:
        print(f"  「{text}」：不是整数")
    else:                               # try 里没有出错时才执行
        print(f"  「{text}」：读到了 {n}")
    finally:                            # 无论有没有出错，最后都一定会执行
        print("  —— 处理完毕")

read_number("42")
read_number("4.2")
```

- `else`：只在**没有异常**时执行。把「出错就不该做的事」放在这里，而不是塞进 `try`，可以让 `try` 只包住真正可能出错的那一两行。
- `finally`：**一定会执行**，即使 `try` 里有 `return`、即使出了没被接住的异常。常用来做「善后」工作，比如关闭文件、断开网络连接。（文件用 `with` 就会自动关闭，所以用得比较少。）

## 12.5 自己抛出异常：`raise`

写函数时，如果别人传进来的参数不合理，最好的办法往往不是 `print` 一句话、返回一个奇怪的值，而是**抛出异常**，让调用者知道出了问题：

```python
def set_age(age: int) -> int:
    if not 0 <= age <= 150:
        raise ValueError(f"年龄必须在 0 到 150 之间，收到的是 {age}")
    return age

for a in (12, -3, 200):
    try:
        print("设置成功：", set_age(a))
    except ValueError as e:
        print("设置失败：", e)
```

`raise 异常类型("说明")` 会立刻停止当前函数，把异常交给调用它的地方；如果一直没有人接住，程序就会停下来并打印 Traceback。

### 自定义异常

可以定义自己的异常类型，让错误的含义更清楚（`class` 下一课会详细讲，这里照着写就行）：

```python
class InsufficientFunds(Exception):     # 继承自 Exception
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFunds(f"余额 {balance} 元，不够取 {amount} 元")
    return balance - amount

try:
    print(withdraw(100, 30))
    print(withdraw(100, 300))
except InsufficientFunds as e:
    print("取款失败：", e)
```

## 12.6 常见的异常类型

| 异常 | 什么时候出现 | 例子 |
|------|--------------|------|
| `ValueError` | 类型对，但值不合适 | `int("abc")` |
| `TypeError` | 类型不对 | `"1" + 1` |
| `ZeroDivisionError` | 除以 0 | `5 / 0` |
| `IndexError` | 下标超出范围 | `[1, 2][5]` |
| `KeyError` | 字典里没有这个键 | `{}["a"]` |
| `AttributeError` | 对象没有这个属性或方法 | `"abc".push(1)` |
| `NameError` | 用了没定义的名字 | `print(x)`（x 没定义） |
| `FileNotFoundError` | 文件不存在 | `open("没有.txt")` |
| `KeyboardInterrupt` | 用户按了 Ctrl+C | 正在运行的程序被中断 |

这些异常组成了一棵「家族树」：几乎所有异常都是 `Exception` 的「后代」，`FileNotFoundError` 又是 `OSError` 的后代……`except OSError:` 会同时接住 `FileNotFoundError`、`PermissionError` 等。

## 12.7 不要写「什么都接」的 `except`

```python norun
try:
    result = calculate(data)
except:                     # 危险！什么异常都接住，包括拼错变量名、按 Ctrl+C
    print("出错了")
```

这样写，**真正的 bug 也会被悄悄吞掉**：你把变量名拼错了（`NameError`），程序却只说「出错了」，根本不知道哪里错；用户想按 Ctrl+C 结束程序，也被它拦住了。

原则：

1. **只接住你预料到、并且知道怎么处理的异常**，比如 `except ValueError:`；
2. 实在需要兜底（比如服务器不能因为一个请求出错就整个退出），至少写 `except Exception as e:`，并把错误**记录下来**，不要默默忽略；
3. 永远不要写 `except: pass`。

## 12.8 用异常还是用 `if`？

两种风格都很常见：

```python
d = {"a": 1}

# 先检查再做（Look Before You Leap）
if "b" in d:
    print(d["b"])
else:
    print("没有 b")

# 先做，出错了再处理（Easier to Ask Forgiveness than Permission）
try:
    print(d["b"])
except KeyError:
    print("没有 b")
```

经验法则：**能轻松提前检查的、经常发生的情况，用 `if`**；**难以提前检查的（比如文件会不会在打开前一刻被删掉、网络会不会断）或者很少发生的，用 `try`**。Python 社区更偏爱后一种风格，但两种都不算错。

## 12.9 `assert`：检查「绝不应该发生」的情况

`assert 条件, "说明"`：如果条件不成立，就抛出 `AssertionError`。它用来检查**程序自己的假设**，发现 bug：

```python
def average(nums):
    assert len(nums) > 0, "average() 需要至少一个数"
    return sum(nums) / len(nums)

print(average([1, 2, 3]))
try:
    average([])
except AssertionError as e:
    print("AssertionError:", e)
```

> [!WARNING]
> `assert` 用来抓 bug，**不要**用它检查用户输入：Python 用 `-O` 参数运行时会跳过所有 `assert`。检查用户输入请用 `if` + `raise ValueError(...)`。`assert` 最常用的地方是**测试**（第 15 课的 pytest）。

## 12.10 小结

- 语法错误让程序无法开始运行；**异常**是运行中出的状况，不处理就会让程序停止。
- `try:` 放可能出错的代码，`except 类型:` 处理；`as e` 拿到异常对象；`else` 在没出错时执行；`finally` 一定执行。
- 用 `raise ValueError("说明")` 在自己的函数里报告错误；可以继承 `Exception` 定义自己的异常。
- 常见异常：`ValueError`、`TypeError`、`ZeroDivisionError`、`IndexError`、`KeyError`、`FileNotFoundError`……
- 只接住预料到的异常；不要写裸的 `except:`，更不要 `except: pass`。
- 容易检查的用 `if`，难以检查的用 `try`；`assert` 检查程序自身的假设，不用来检查用户输入。

## 12.11 练习

<!-- exercise: safe-int -->
### 练习 1：安全地读整数

第一行是整数 `n`，接下来 `n` 行，每行是用户输入的一个东西。如果是整数，输出它的 2 倍；否则输出 `invalid`。用 `try / except` 实现。

```text
样例输入：
3
21
abc
-4
样例输出：
42
invalid
-8
```

<details>
<summary>提示</summary>

在循环里写 `try: print(int(text) * 2)`，`except ValueError: print("invalid")`。
</details>

<!-- exercise: safe-divide -->
### 练习 2：安全的除法

第一行是整数 `n`，接下来 `n` 行，每行两个东西 `a b`。输出 `a / b`（保留 2 位小数）；如果 `b` 是 0，输出 `division by zero`；如果有一个不是数字，输出 `invalid`。

```text
样例输入：
3
10 4
1 0
x 2
样例输出：
2.50
division by zero
invalid
```

<details>
<summary>提示</summary>

用两个 `except`：`except ZeroDivisionError:` 和 `except ValueError:`。`float("x")` 会抛出 `ValueError`。
</details>

<!-- exercise: raise-age -->
### 练习 3：用 raise 报告错误

补全函数 `check_age(age)`：如果 `age` 小于 0 或大于 150，抛出 `ValueError("age must be between 0 and 150")`；否则返回 `age`。主程序已经写好：读入若干个整数，合法的输出 `ok`，不合法的输出异常的说明。

```text
样例输入：12 -1 151 150
样例输出：
ok
age must be between 0 and 150
age must be between 0 and 150
ok
```

<details>
<summary>提示</summary>

`if age < 0 or age > 150: raise ValueError("age must be between 0 and 150")`。说明文字要和题目完全一致。
</details>

<!-- exercise: finally-order -->
### 练习 4：读代码，写输出

```python
def f(x):
    try:
        print("start")
        return 10 / x
    except ZeroDivisionError:
        print("oops")
        return None
    finally:
        print("done")

print(f(2))
print(f(0))
```

<details>
<summary>答案与解析</summary>

`f(2)`：打印 `start`，准备返回 5.0，但返回之前先执行 `finally` 打印 `done`，然后外面打印 `5.0`。`f(0)`：打印 `start`，除以 0 跳到 `except` 打印 `oops`，再执行 `finally` 打印 `done`，最后外面打印 `None`。
</details>

<!-- exercise: bare-except -->
### 练习 5：为什么不要写 `except:`？

- 因为它运行得很慢
- 因为它会接住所有异常，包括拼错名字造成的 bug 和 Ctrl+C，让问题被悄悄隐藏
- 因为 Python 不允许这样写
- 因为它只能接住 `ValueError`

<details>
<summary>答案与解析</summary>

裸的 `except:` 什么都接，真正的错误（比如 `NameError`）会被当成「普通情况」处理掉，很难发现；连用户按 Ctrl+C 产生的 `KeyboardInterrupt` 也会被拦住。应该只接住预料到的具体异常。
</details>

<!-- exercise: which-exception -->
### 练习 6：哪种异常？

`int("3.14")` 会抛出哪种异常？

- `TypeError`
- `ValueError`
- `SyntaxError`
- 不会出错，结果是 3

<details>
<summary>答案与解析</summary>

`ValueError`：参数是字符串（类型没问题），但 `"3.14"` 不是整数的写法。想得到 3，可以写 `int(float("3.14"))`。
</details>

## 12.12 延伸阅读

- [Python 官方教程 · 8. 错误和异常](https://docs.python.org/zh-cn/3/tutorial/errors.html)
- [内置异常的层级结构](https://docs.python.org/zh-cn/3/library/exceptions.html#exception-hierarchy)
- 下一课：[类与对象](../13-classes/zh-CN.md)
