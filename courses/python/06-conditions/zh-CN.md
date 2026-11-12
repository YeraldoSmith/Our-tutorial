# 6. 条件判断

> 🎯 **本课目标**：用比较运算符和逻辑运算符写出条件；用 `if`、`elif`、`else` 让程序根据情况做不同的事；理解**缩进**在 Python 中的意义；了解条件表达式和 `match` 语句。
>
> ⏱ **预计用时**：45 分钟　｜　📋 **前置知识**：第 2～5 课

## 6.1 程序也需要「做决定」

生活中到处是判断：**如果**下雨，就带伞；**如果**考了 60 分以上，就及格，**否则**补考。程序也一样：根据不同的情况执行不同的代码。这叫**条件判断**（也叫分支）。

```python
temperature = 35
if temperature > 30:
    print("好热！记得多喝水。")
print("祝你今天愉快")
```

输出：

```text
好热！记得多喝水。
祝你今天愉快
```

把 `temperature` 改成 `20` 再运行，第一句就不会出现了。

## 6.2 比较运算符

条件的结果是**布尔值**：`True`（成立）或 `False`（不成立）。

| 运算符 | 含义 | 例子 | 结果 |
|--------|------|------|------|
| `==` | 等于 | `3 == 3` | `True` |
| `!=` | 不等于 | `3 != 4` | `True` |
| `>` | 大于 | `5 > 7` | `False` |
| `<` | 小于 | `5 < 7` | `True` |
| `>=` | 大于等于 | `7 >= 7` | `True` |
| `<=` | 小于等于 | `8 <= 7` | `False` |

```python
age = 18
print(age >= 18)        # True
print(age == 20)        # False
print("abc" == "abc")   # True：字符串也能比较
print("Abc" == "abc")   # False：大小写不同
```

> [!WARNING]
> **`=` 是赋值，`==` 才是比较。** 写成 `if age = 18:` 会报语法错误。这是所有语言的新手都会犯的错误。

Python 还允许**连续比较**，和数学写法一样：

```python
x = 5
print(0 < x < 10)      # True：x 在 0 和 10 之间
```

## 6.3 if 语句的结构

```text
if 条件:
    条件成立时执行的代码
    （可以有很多行）
后面的代码（无论如何都会执行）
```

两个关键点：

1. `if 条件` 后面**必须有冒号** `:`。
2. 属于 `if` 的代码**必须缩进**（统一用 4 个空格）。缩进的代码组成一个**代码块**；回到原来的缩进位置，就表示代码块结束了。

```python
score = 95
if score >= 90:
    print("优秀！")       # 属于 if
    print("继续保持")     # 也属于 if
print("成绩已记录")        # 不属于 if，总会执行
```

> [!IMPORTANT]
> 在 C、Java、JavaScript 里，代码块用花括号 `{ }` 表示；在 Python 里，代码块**完全由缩进决定**。所以 Python 对缩进非常严格：同一个代码块的每一行，前面的空格数必须一样。学习程序的编辑器在你按回车时会自动缩进，按 <kbd>Tab</kbd> 插入 4 个空格。

## 6.4 else：否则

```python
age = int(input())
if age >= 18:
    print("成年人")
else:
    print("未成年人")
```

`else` 后面也要有冒号，它的代码块在条件**不成立**时执行。`if` 和 `else` 两个分支**一定会走且只走一个**。

### 判断奇偶

```python
n = int(input())
if n % 2 == 0:
    print("偶数")
else:
    print("奇数")
```

## 6.5 elif：多个条件

情况不止两种时，用 `elif`（else if 的缩写）：

```python
score = int(input())
if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 70:
    print("C")
elif score >= 60:
    print("D")
else:
    print("F")
```

Python **从上往下**逐个检查条件，**遇到第一个成立的就执行它，然后跳过剩下的全部**。所以输入 `85` 时：`85 >= 90` 不成立 → `85 >= 80` 成立 → 输出 `B`，后面的不再检查。

> [!TIP]
> 正因为是「从上往下、只走一个」，`elif score >= 80` 不需要写成 `elif 80 <= score < 90`——能走到这里，说明 `score >= 90` 已经不成立了。但**顺序很重要**：如果把 `score >= 60` 放在最前面，那么 95 分也会得到 D！

## 6.6 逻辑运算符：and、or、not

把多个条件组合起来：

| 运算符 | 含义 | 例子 |
|--------|------|------|
| `and` | 并且（两个都成立才成立） | `age >= 18 and has_ticket` |
| `or` | 或者（有一个成立就成立） | `day == "周六" or day == "周日"` |
| `not` | 不（取反） | `not is_raining` |

```python
age = 20
has_ticket = True
if age >= 18 and has_ticket:
    print("可以入场")

day = "周日"
if day == "周六" or day == "周日":
    print("周末愉快！")

is_raining = False
if not is_raining:
    print("出去散步吧")
```

### 一个经典例子：闰年

闰年的规则：能被 4 整除**并且**不能被 100 整除，**或者**能被 400 整除。

```python
year = int(input())
if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
    print("闰年")
else:
    print("平年")
```

试试 2024（闰年）、1900（平年）、2000（闰年）。括号让逻辑更清楚。

> [!WARNING]
> 新手常写 `if day == "周六" or "周日":`——这是**错的**！它会被理解为 `(day == "周六") or ("周日")`，而非空字符串 `"周日"` 永远算作 `True`，所以条件永远成立。正确写法是每一边都写完整：`day == "周六" or day == "周日"`，或者 `day in ("周六", "周日")`。

## 6.7 嵌套条件

`if` 里面还可以再写 `if`，每深一层多缩进 4 个空格：

```python
age = int(input())
member = input()      # 输入 yes 或 no
if age < 12:
    print("儿童票：免费")
else:
    if member == "yes":
        print("会员票：50 元")
    else:
        print("普通票：80 元")
```

嵌套太深会难以阅读。很多时候可以用 `elif` 或 `and` 让结构变平：

```python
age = int(input())
member = input()
if age < 12:
    print("儿童票：免费")
elif member == "yes":
    print("会员票：50 元")
else:
    print("普通票：80 元")
```

## 6.8 「真值」：不只是 True 和 False

`if` 后面不一定非要写比较，任何值都可以作为条件。Python 会把它当作 `bool(值)` 来判断（第 2 课学过）：`0`、`0.0`、空字符串 `""`、`None`、空列表 `[]` 被当作「假」，其他都当作「真」。

```python
name = input()
if name:                 # 等同于 if name != ""
    print(f"你好，{name}")
else:
    print("你什么都没输入")
```

## 6.9 条件表达式：一行的 if

简单的二选一，可以写在一行里：

```python
age = 20
status = "成年" if age >= 18 else "未成年"
print(status)
```

格式：`值1 if 条件 else 值2`——条件成立取值 1，否则取值 2。适合简单情况，复杂的还是用普通 `if`。

## 6.10 match 语句：按值分情况

Python 3.10 引入了 `match` 语句，适合「根据一个值的不同取值做不同的事」：

```python
command = input()
match command:
    case "start":
        print("开始游戏")
    case "pause" | "stop":          # | 表示「或」
        print("暂停")
    case "quit":
        print("再见！")
    case _:                          # _ 匹配其他所有情况
        print("未知命令")
```

`match` 的能力远不止这些（它能拆解列表、字典等复杂数据，叫「结构化模式匹配」），以后会再见到它。对于简单判断，用 `if/elif` 完全可以。

## 6.11 常见错误

```python norun
if x = 5:
```

```text
SyntaxError: invalid syntax. Maybe you meant '==' or ':=' instead of '='?
```

比较用 `==`。Python 的报错甚至猜出了你的意思。

```python norun
if x > 5
    print("大")
```

```text
SyntaxError: expected ':'
```

忘了冒号。

```python norun
if x > 5:
print("大")
```

```text
IndentationError: expected an indented block after 'if' statement on line 1
```

忘了缩进。

```python norun
if x > 5:
    print("大")
     print("很大")
```

```text
IndentationError: unexpected indent
```

同一个代码块缩进不一致（第二行多了一个空格）。

## 6.12 小结

- 比较运算符 `== != > < >= <=` 得到 `True`/`False`；`=` 是赋值，`==` 才是比较。
- `if 条件:` + **缩进的代码块**；`else:` 处理其他情况；`elif` 处理多种情况，**从上往下只走第一个成立的分支**。
- `and`、`or`、`not` 组合条件；`or` 两边都要写完整的条件。
- 任何值都可以当条件：`0`、`""`、`None`、空容器为假。
- `值1 if 条件 else 值2` 是一行的二选一；`match/case` 按值分情况。

## 6.13 练习

<!-- exercise: odd-even -->
### 练习 1：奇数还是偶数

输入一个整数，如果是偶数输出 `even`，否则输出 `odd`。

```text
样例输入：7
样例输出：odd
```

<details>
<summary>提示</summary>

`n % 2 == 0` 表示偶数。
</details>

<!-- exercise: grade -->
### 练习 2：成绩等级

输入一个 0～100 的整数成绩，按下表输出等级：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。

```text
样例输入：85
样例输出：B
```

<details>
<summary>提示</summary>

用 `if / elif / else`，从高分往低分判断。
</details>

<!-- exercise: leap -->
### 练习 3：闰年

输入一个年份，是闰年输出 `leap`，否则输出 `common`。规则：能被 4 整除且不能被 100 整除，或者能被 400 整除。

```text
样例输入：1900
样例输出：common
```

<details>
<summary>提示</summary>

`(y % 4 == 0 and y % 100 != 0) or y % 400 == 0`
</details>

<!-- exercise: max3 -->
### 练习 4：三个数中的最大值

输入一行三个整数（空格隔开），输出其中最大的一个。**请用 `if` 来做**，不要用 `max()`（做完以后可以再用 `max()` 验证）。

```text
样例输入：3 9 4
样例输出：9
```

<details>
<summary>提示 1</summary>

先假设第一个数最大：`biggest = a`；然后依次和 `b`、`c` 比较，谁更大就更新 `biggest`。
</details>

<details>
<summary>提示 2</summary>

```python
a, b, c = map(int, input().split())
biggest = a
if b > biggest:
    biggest = b
# 再比较 c……
```
</details>

<!-- exercise: predict-if -->
### 练习 5：读代码，写输出

```python
x = 15
if x > 10:
    print("A")
elif x > 5:
    print("B")
if x % 3 == 0 and x % 5 == 0:
    print("C")
else:
    print("D")
print("E" if x < 10 else "F")
```

<details>
<summary>答案与解析</summary>

`x > 10` 成立，输出 `A`，`elif` 被跳过；这是一个新的 `if`：15 同时能被 3 和 5 整除，输出 `C`；最后 `x < 10` 不成立，输出 `F`。
</details>

<!-- exercise: or-trap -->
### 练习 6：or 的陷阱

`day = "Monday"` 时，`if day == "Saturday" or "Sunday":` 的条件结果是什么？

- `False`，因为 day 不是周末
- `True`，因为 `"Sunday"` 本身被当作真值
- 报错
- 取决于电脑的语言设置

<details>
<summary>答案与解析</summary>

`True`。这个条件被理解为 `(day == "Saturday") or "Sunday"`，非空字符串永远为真。正确写法是 `day == "Saturday" or day == "Sunday"`。
</details>

## 6.14 延伸阅读

- [Python 官方教程 · 4.1 if 语句](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#if-statements)
- [Python 官方教程 · 4.7 match 语句](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#match-statements)
- 下一课：[循环](../07-loops/zh-CN.md)
