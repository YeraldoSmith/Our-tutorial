# 8. 列表与元组

> 🎯 **本课目标**：用列表保存一组数据，并进行增、删、改、查、排序；用 `for` 和 `enumerate` 遍历列表；初步掌握列表推导式；理解「两个变量指向同一个列表」的陷阱；认识元组和解包。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：第 4、7 课

## 8.1 为什么需要列表？

要保存全班 40 个同学的成绩，总不能建 40 个变量 `score1`、`score2`……**列表**（list）可以把很多个值放进**一个**变量里，按顺序排好：

```python
scores = [88, 92, 75, 64, 99]
names = ["小明", "小红", "小刚"]
mixed = [1, "two", 3.0, True]      # 可以放不同类型（但一般不推荐）
empty = []                         # 空列表
print(scores)
print(len(scores))                 # 5：元素个数
```

列表用**方括号** `[ ]`，元素之间用**逗号**隔开。

## 8.2 下标和切片：和字符串一样

列表的下标、负下标和切片，与第 4 课的字符串**完全相同**：

```python
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜"]
print(fruits[0])       # 苹果
print(fruits[-1])      # 西瓜
print(fruits[1:3])     # ['香蕉', '橙子']
print(fruits[::-1])    # 反转
print("橙子" in fruits)  # True
```

## 8.3 列表可以修改

和字符串不同，**列表是可变的**（mutable），可以直接改变其中的元素：

```python
scores = [88, 92, 75]
scores[2] = 80         # 把第 3 个元素改成 80
print(scores)          # [88, 92, 80]
```

### 添加元素

```python
nums = [1, 2, 3]
nums.append(4)         # 在末尾添加一个：[1, 2, 3, 4]
nums.insert(0, 0)      # 在下标 0 处插入：[0, 1, 2, 3, 4]
nums.extend([5, 6])    # 在末尾添加多个：[0, 1, 2, 3, 4, 5, 6]
print(nums)
```

### 删除元素

```python
nums = [10, 20, 30, 20, 40]
nums.remove(20)        # 删除第一个值为 20 的元素：[10, 30, 20, 40]
last = nums.pop()      # 删除并返回最后一个：last = 40
first = nums.pop(0)    # 删除并返回下标 0 的元素：first = 10
del nums[0]            # 按下标删除
print(nums, last, first)
nums.clear()           # 清空
print(nums)            # []
```

### 查找和统计

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
print(nums.index(5))   # 4：值 5 第一次出现的下标
print(nums.count(1))   # 2：值 1 出现的次数
print(min(nums), max(nums), sum(nums))   # 1 9 31
print(sum(nums) / len(nums))             # 平均值 3.875
```

### 排序

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
nums.sort()                  # 原地排序（改变 nums 本身）
print(nums)                  # [1, 1, 2, 3, 4, 5, 6, 9]
nums.sort(reverse=True)      # 从大到小
print(nums)

data = [3, 1, 2]
new = sorted(data)           # 返回一个排好序的新列表，原列表不变
print(data, new)             # [3, 1, 2] [1, 2, 3]

words = ["banana", "Apple", "cherry"]
print(sorted(words))                  # ['Apple', 'banana', 'cherry']：大写字母排在前面
print(sorted(words, key=str.lower))   # 忽略大小写排序
print(sorted(words, key=len))         # 按长度排序
```

> [!IMPORTANT]
> `nums.sort()` **修改原列表**，返回值是 `None`；`sorted(nums)` **不修改原列表**，返回新列表。新手常写 `nums = nums.sort()`，结果 `nums` 变成了 `None`！

### 方法速查表

| 操作 | 写法 |
|------|------|
| 末尾添加 | `lst.append(x)` |
| 插入 | `lst.insert(i, x)` |
| 合并 | `lst.extend(另一个列表)` 或 `lst + 另一个列表` |
| 按值删除 | `lst.remove(x)` |
| 按下标删除并取出 | `lst.pop(i)`（不写 i 是最后一个） |
| 查找下标 / 计数 | `lst.index(x)` / `lst.count(x)` |
| 排序 | `lst.sort()` / `sorted(lst)` |
| 反转 | `lst.reverse()` / `lst[::-1]` |
| 长度、最小、最大、求和 | `len`、`min`、`max`、`sum` |

## 8.4 遍历列表

```python
names = ["小明", "小红", "小刚"]
for name in names:
    print(f"你好，{name}")
```

需要**同时知道下标和值**时，用 `enumerate`：

```python
names = ["小明", "小红", "小刚"]
for i, name in enumerate(names, start=1):
    print(f"{i}. {name}")
```

```text
1. 小明
2. 小红
3. 小刚
```

同时遍历两个列表，用 `zip`：

```python
names = ["小明", "小红", "小刚"]
scores = [88, 95, 72]
for name, score in zip(names, scores):
    print(f"{name}：{score} 分")
```

## 8.5 在循环中构造列表

最常见的模式：从空列表开始，循环中不断 `append`：

```python
n = int(input())
numbers = []
for _ in range(n):
    numbers.append(int(input()))
print("你输入了：", numbers)
print("最大值：", max(numbers))
```

如果 n 个数在**同一行**，用第 5 课的 `split` + `map`，再用 `list()` 转成列表：

```python
numbers = list(map(int, input().split()))
print(numbers)
```

### 筛选

```python
scores = [88, 45, 92, 59, 75, 30]
passed = []
for s in scores:
    if s >= 60:
        passed.append(s)
print(passed)        # [88, 92, 75]
```

## 8.6 列表推导式

上面「循环 + 筛选 + append」的模式太常见了，Python 提供了一种简洁的写法，叫**列表推导式**（list comprehension）：

```python
scores = [88, 45, 92, 59, 75, 30]
passed = [s for s in scores if s >= 60]
print(passed)                          # [88, 92, 75]

squares = [x * x for x in range(1, 6)]
print(squares)                         # [1, 4, 9, 16, 25]

words = ["hello", "world"]
print([w.upper() for w in words])      # ['HELLO', 'WORLD']
```

读法：`[对 x 做什么  for x in 哪里  if 什么条件]`——「对每一个 x（满足条件的），计算出新值，放进新列表」。

> [!TIP]
> 列表推导式适合**简单**的变换和筛选。逻辑复杂时，用普通的 `for` 循环更清楚。

## 8.7 嵌套列表：表格和矩阵

列表的元素也可以是列表，用来表示二维表格：

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]
print(matrix[1])        # [4, 5, 6]：第 2 行
print(matrix[1][2])     # 6：第 2 行第 3 列

for row in matrix:
    for value in row:
        print(value, end=" ")
    print()
```

## 8.8 陷阱：两个名字指向同一个列表

```python
a = [1, 2, 3]
b = a          # b 和 a 指向「同一个」列表
b.append(4)
print(a)       # [1, 2, 3, 4]——a 也变了！
```

第 2 课说变量是「标签」。`b = a` 并没有复制列表，只是给**同一个列表**又贴了一张标签。通过任何一个名字修改，另一个都能看到：

```text
a ──┐
    ├──► [1, 2, 3, 4]
b ──┘
```

想要一个独立的副本，用 `copy()` 或切片：

```python
a = [1, 2, 3]
b = a.copy()       # 或 b = a[:] 或 b = list(a)
b.append(4)
print(a, b)        # [1, 2, 3] [1, 2, 3, 4]
```

> [!NOTE]
> 数字和字符串是不可变的，所以 `y = x; y += 1` 不会影响 `x`——`y += 1` 是让 `y` 指向一个新数字。只有可变对象（列表、后面要学的字典、集合）才会出现「改一个、两个都变」的情况。

## 8.9 元组：不能修改的列表

**元组**（tuple）和列表很像，但用**圆括号**，并且**创建后不能修改**：

```python
point = (3, 4)
print(point[0])        # 3
colors = ("red", "green", "blue")
print(len(colors), "green" in colors)
```

```python norun
point[0] = 5           # TypeError: 'tuple' object does not support item assignment
```

什么时候用元组？当一组数据**本来就不该变**时：坐标 `(x, y)`、RGB 颜色 `(255, 128, 0)`、日期 `(2026, 11, 12)`。不可变带来安全性：别人不会不小心改掉它。

### 打包和解包

其实你早就用过元组了：

```python
x, y = 3, 4        # 右边 3, 4 是一个元组（打包），左边把它拆开（解包）
x, y = y, x        # 第 2 课的交换技巧，原理就是元组
```

```python
point = (3, 4)
x, y = point
print(x, y)

first, *rest = [1, 2, 3, 4]   # * 收集剩下的所有元素
print(first, rest)            # 1 [2, 3, 4]
```

> [!NOTE]
> 只有一个元素的元组要加逗号：`(5,)`。`(5)` 只是带括号的数字 5。

## 8.10 常见错误

```python norun
nums = [1, 2, 3]
print(nums[3])
```

```text
IndexError: list index out of range
```

长度为 3 的列表，合法下标是 0、1、2。

```python norun
nums = [3, 1, 2]
nums = nums.sort()
print(nums[0])
```

```text
TypeError: 'NoneType' object is not subscriptable
```

`sort()` 返回 `None`，`nums` 被覆盖成了 `None`。直接写 `nums.sort()`，或 `nums = sorted(nums)`。

```python norun
nums = [1, 2, 3]
nums.remove(5)
```

```text
ValueError: list.remove(x): x not in list
```

要删除的值不存在。可以先 `if 5 in nums:` 判断。

## 8.11 小结

- 列表 `[a, b, c]` 按顺序保存多个值；下标、切片、`in`、`len` 与字符串相同。
- 列表**可变**：`append`、`insert`、`extend`、`remove`、`pop`、`del`、`sort`……
- `lst.sort()` 原地排序返回 `None`；`sorted(lst)` 返回新列表；`key=` 指定排序依据。
- 遍历：`for x in lst`；要下标用 `enumerate`；并行遍历用 `zip`。
- 列表推导式 `[表达式 for x in 序列 if 条件]`。
- `b = a` 不复制列表，`b = a.copy()` 才复制。
- 元组 `(a, b)` 不可修改；打包与解包让多重赋值和交换成为可能。

## 8.12 练习

<!-- exercise: average -->
### 练习 1：平均分

第一行输入整数 n，第二行输入 n 个整数（空格隔开）。输出它们的平均值，保留 2 位小数。

```text
样例输入：
4
90 85 77 60
样例输出：
78.00
```

<details>
<summary>提示</summary>

`n = int(input())`，`nums = list(map(int, input().split()))`，平均值是 `sum(nums) / n`。
</details>

<!-- exercise: sort-desc -->
### 练习 2：从大到小

输入一行若干个整数（空格隔开），从大到小排序后输出，空格隔开。

```text
样例输入：3 1 4 1 5 9 2 6
样例输出：9 6 5 4 3 2 1 1
```

<details>
<summary>提示</summary>

排序用 `sort(reverse=True)`；输出时把每个数转成字符串，用 `" ".join(...)`。也可以写 `print(*nums)`，`*` 会把列表拆开作为多个参数。
</details>

<!-- exercise: unique -->
### 练习 3：去掉重复

输入一行若干个单词（空格隔开），去掉重复的单词，**保持第一次出现的顺序**，输出剩下的单词。

```text
样例输入：apple banana apple cherry banana
样例输出：apple banana cherry
```

<details>
<summary>提示</summary>

准备一个空列表 `result`，遍历每个单词，如果 `word not in result` 就 `append`。
</details>

<!-- exercise: evens-squared -->
### 练习 4：列表推导式

输入一行若干个整数，用**列表推导式**选出其中的偶数，并输出它们的平方（空格隔开）。

```text
样例输入：1 2 3 4 5 6
样例输出：4 16 36
```

<details>
<summary>提示</summary>

`[x * x for x in nums if x % 2 == 0]`
</details>

<!-- exercise: alias-predict -->
### 练习 5：读代码，写输出

```python
a = [1, 2, 3]
b = a
c = a.copy()
b.append(4)
c.append(5)
print(a)
print(len(b), len(c))
```

<details>
<summary>答案与解析</summary>

`b` 和 `a` 是同一个列表，`b.append(4)` 后 `a` 也是 `[1, 2, 3, 4]`；`c` 是副本，只有它多了 5。所以 `a` 输出 `[1, 2, 3, 4]`，`b` 长 4，`c` 也长 4（`[1, 2, 3, 5]`）。
</details>

<!-- exercise: sort-return -->
### 练习 6：sort 的返回值

`x = [3, 1, 2].sort()` 执行之后，`x` 是什么？

- `[1, 2, 3]`
- `[3, 1, 2]`
- `None`
- 报错

<details>
<summary>答案与解析</summary>

`None`。`sort()` 原地排序，不返回新列表。想得到排序后的新列表请用 `sorted([3, 1, 2])`。
</details>

## 8.13 延伸阅读

- [Python 官方教程 · 5.1 列表详解](https://docs.python.org/zh-cn/3/tutorial/datastructures.html)
- [Python 排序指南](https://docs.python.org/zh-cn/3/howto/sorting.html)
- 下一课：[字典与集合](../09-dicts-sets/zh-CN.md)
