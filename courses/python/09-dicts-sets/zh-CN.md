# 9. 字典与集合

> 🎯 **本课目标**：用**字典**按「名字」而不是按「位置」查找数据；掌握字典的增、删、改、查和遍历；学会用字典**计数**（统计词频）；理解为什么列表不能做字典的键；用**集合**去重、求交集并集；知道为什么在字典和集合里查找比在列表里快得多。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：第 7、8 课

## 9.1 为什么需要字典？

想做一个通讯录。用列表的话，要找「小红」的电话，只能从头到尾一个个比对。而现实中的通讯录、字典、词汇表都是**按名字直接查**的。

Python 的**字典**（dict）就是这样一种结构：它保存很多对「**键**（key）→ **值**（value）」，可以用键直接取出值。

```python
phone = {"小明": "138-0000-1111", "小红": "139-2222-3333", "小刚": "137-4444-5555"}
print(phone["小红"])        # 用键取值：139-2222-3333
print(len(phone))           # 3：有 3 对键值
print(phone)
```

字典用**花括号** `{ }`，每一对写成 `键: 值`，对与对之间用逗号隔开。

## 9.2 取值：`[]` 和 `get`

```python
ages = {"小明": 12, "小红": 13}
print(ages["小明"])                 # 12
print(ages.get("小红"))             # 13
print(ages.get("小刚"))             # None：键不存在时返回 None，不报错
print(ages.get("小刚", 0))          # 0：也可以指定一个默认值
print("小刚" in ages)               # False：in 检查的是「键」
```

用 `字典[键]` 取一个**不存在**的键会报错：

```python norun
ages = {"小明": 12}
print(ages["小刚"])
```

```text
KeyError: '小刚'
```

所以不确定键在不在时，用 `in` 先判断，或者用 `get`。

## 9.3 增加、修改、删除

字典是**可变的**：

```python
stock = {"苹果": 5, "香蕉": 3}
stock["橙子"] = 10          # 键不存在：添加新的一对
stock["苹果"] = 8           # 键已存在：修改它的值
stock["香蕉"] += 2          # 在原来的值上加 2
print(stock)

del stock["橙子"]           # 删除一对
n = stock.pop("香蕉")       # 删除并返回它的值
print(n, stock)
```

> [!NOTE]
> 一个字典里**键不会重复**：给已经存在的键赋值，只会覆盖原来的值。值可以重复，比如两个人可以同岁。

## 9.4 遍历字典

```python
scores = {"语文": 90, "数学": 85, "英语": 92}

for subject in scores:                      # 直接遍历：得到的是键
    print(subject)

for subject, score in scores.items():       # .items()：同时得到键和值（最常用）
    print(f"{subject}：{score} 分")

print(list(scores.keys()))                  # 所有的键
print(list(scores.values()))                # 所有的值
print("总分：", sum(scores.values()))
```

字典会**记住插入的顺序**（Python 3.7 起），遍历时按添加的先后出现。

## 9.5 用字典计数

「统计每个东西出现了几次」是编程中最常见的任务之一，字典最擅长：

```python
text = "the cat and the dog and the bird"
counts = {}
for word in text.split():
    counts[word] = counts.get(word, 0) + 1     # 没见过就从 0 开始，再加 1
print(counts)

# 按出现次数从多到少排序
for word, n in sorted(counts.items(), key=lambda kv: kv[1], reverse=True):
    print(word, n)
```

`counts.get(word, 0) + 1` 是计数的经典写法：第一次遇到这个词时 `get` 返回 0。`sorted(..., key=lambda kv: kv[1])` 表示按每一对的**第二项**（次数）排序，`lambda` 是一个临时的小函数，下一课学函数时会详细讲。

Python 标准库还有一个专门计数的工具 `Counter`：

```python
from collections import Counter

counts = Counter("the cat and the dog and the bird".split())
print(counts.most_common(2))      # 出现最多的 2 个：[('the', 3), ('and', 2)]
print(Counter("hello"))           # 也能数字母
```

## 9.6 字典推导式

和列表推导式（第 8 课）类似：

```python
squares = {n: n * n for n in range(1, 6)}
print(squares)                            # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

prices = {"苹果": 5, "西瓜": 20, "葡萄": 12}
cheap = {k: v for k, v in prices.items() if v < 15}
print(cheap)                              # 只保留便宜的
```

## 9.7 组合起来：列表里放字典，字典里放列表

真实的数据往往是嵌套的。例如一个班的学生信息：

```python
students = [
    {"name": "小明", "age": 12, "hobbies": ["足球", "画画"]},
    {"name": "小红", "age": 13, "hobbies": ["钢琴"]},
]
for s in students:
    print(f"{s['name']}（{s['age']} 岁）喜欢：{'、'.join(s['hobbies'])}")

students[1]["hobbies"].append("游泳")     # 先取出字典，再取出其中的列表，再修改
print(students[1])
```

这种「列表 + 字典」的结构和网络上最常见的数据格式 **JSON** 几乎一模一样，第 11 课会学怎样把它存进文件。

## 9.8 什么可以做键？

键必须是**不可修改**（可哈希，hashable）的值：数字、字符串、元组都可以；列表、字典这类可以修改的值不行。

```python
distance = {("北京", "上海"): 1213, ("北京", "广州"): 2120}   # 元组做键：表示「一对城市」
print(distance[("北京", "上海")])
```

```python norun
bad = {["北京", "上海"]: 1213}
```

```text
TypeError: unhashable type: 'list'
```

> [!NOTE]
> 为什么？字典靠键的「哈希值」（一个由内容算出来的数）来决定把它放在哪里。如果键的内容能改，哈希值也会变，就再也找不到它了。

## 9.9 集合：不重复的一堆东西

**集合**（set）也用花括号，但只有值、没有键。它有两个特点：**元素不重复**、**没有顺序**。

```python
tags = {"python", "教程", "python", "编程"}
print(tags)                       # 重复的 "python" 只保留一个（顺序可能和你写的不一样）
print(len(tags))                  # 3

nums = [3, 1, 3, 2, 1, 3]
print(set(nums))                  # 去重：{1, 2, 3}
print(sorted(set(nums)))          # 去重并排序：[1, 2, 3]

tags.add("入门")
tags.discard("编程")              # 删除（元素不存在也不报错）
print("教程" in tags)             # True
```

> [!WARNING]
> 空集合要写 `set()`。`{}` 表示的是**空字典**！

集合最有用的是**集合运算**：

```python
a = {"小明", "小红", "小刚"}          # 参加了数学竞赛
b = {"小红", "小刚", "小丽"}          # 参加了英语竞赛
print(a & b)        # 交集：两个都参加的
print(a | b)        # 并集：至少参加一个的
print(a - b)        # 差集：只参加数学的
print(a ^ b)        # 对称差：只参加了一个的
```

## 9.10 为什么字典和集合查找特别快？

在列表里查找 `x in lst`，Python 要从头到尾一个个比较；列表有一百万个元素，最坏要比较一百万次。字典和集合用**哈希表**存储，直接根据哈希值算出位置，几乎**一步**就能找到，不管有多少元素：

```python
import time

n = 200_000
lst = list(range(n))
st = set(lst)

start = time.perf_counter()
for _ in range(200):
    (n - 1) in lst               # 要找的数在最后面
t_list = time.perf_counter() - start

start = time.perf_counter()
for _ in range(200):
    (n - 1) in st
t_set = time.perf_counter() - start

print(f"列表：{t_list:.4f} 秒   集合：{t_set:.6f} 秒")
```

所以，**要频繁判断「在不在」时，用集合或字典，不要用列表**。

## 9.11 常见错误

```python norun
counts = {}
counts["apple"] += 1
```

```text
KeyError: 'apple'
```

键还不存在，没法在原来的值上加 1。用 `counts["apple"] = counts.get("apple", 0) + 1`。

```python norun
d = {"a": 1, "b": 2, "c": 3}
for k in d:
    if d[k] < 2:
        del d[k]
```

```text
RuntimeError: dictionary changed size during iteration
```

遍历字典的同时不能删除元素。改成遍历一份副本：`for k in list(d):`。

```python norun
s = {}
s.add(1)
```

```text
AttributeError: 'dict' object has no attribute 'add'
```

`{}` 是空字典，空集合要写 `set()`。

## 9.12 小结

- 字典 `{键: 值}` 按键查找值；`d[k]` 键不存在会 `KeyError`，`d.get(k, 默认值)` 不会。
- 增改都是 `d[k] = v`；删除用 `del d[k]` 或 `d.pop(k)`；`in` 检查键。
- 遍历用 `for k, v in d.items()`；字典保持插入顺序。
- 计数：`d[x] = d.get(x, 0) + 1`，或 `collections.Counter`；排序用 `sorted(d.items(), key=...)`。
- 键必须不可修改（数字、字符串、元组）；列表不能做键。
- 集合 `{a, b}` 元素不重复、无序；`set()` 是空集合；`& | - ^` 是交、并、差、对称差。
- 在字典和集合里查找几乎是一步完成，比列表快得多。

## 9.13 练习

<!-- exercise: word-count -->
### 练习 1：词频统计

输入一行英文单词（空格隔开），按单词**第一次出现的顺序**，每行输出一个单词和它出现的次数（空格隔开）。

```text
样例输入：the cat and the dog
样例输出：
the 2
cat 1
and 1
dog 1
```

<details>
<summary>提示</summary>

用 `counts[w] = counts.get(w, 0) + 1` 计数；字典会按插入顺序遍历。
</details>

<!-- exercise: top-word -->
### 练习 2：出现最多的词

输入一行单词，输出出现次数最多的单词和次数（空格隔开）。如果有几个单词次数一样多，输出其中**按字母顺序最小**的那个。

```text
样例输入：b a c a b
样例输出：a 2
```

<details>
<summary>提示</summary>

先计数，再用 `min(counts, key=lambda w: (-counts[w], w))`：先比次数（取负数让多的排前面），次数一样再比单词本身。
</details>

<!-- exercise: phonebook -->
### 练习 3：通讯录

第一行是整数 `n`，接下来 `n` 行每行一个名字和电话（空格隔开）。然后是整数 `m`，接下来 `m` 行每行一个要查询的名字。对每个查询，输出电话；如果查不到，输出 `not found`。

```text
样例输入：
2
Tom 123
Amy 456
3
Amy
Bob
Tom
样例输出：
456
not found
123
```

<details>
<summary>提示</summary>

先把 `n` 行读进字典：`name, number = input().split()`，`book[name] = number`；查询时用 `book.get(name, "not found")`。
</details>

<!-- exercise: common -->
### 练习 4：共同的数

输入两行整数（空格隔开），输出**两行都出现过**的数，从小到大排列、不重复，空格隔开。如果没有，输出 `none`。

```text
样例输入：
1 2 3 4 4
3 4 5 4
样例输出：3 4
```

<details>
<summary>提示</summary>

把两行分别变成集合，求交集 `&`，再 `sorted`。
</details>

<!-- exercise: dict-predict -->
### 练习 5：读代码，写输出

```python
d = {"a": 1}
d["b"] = 2
d["a"] += 10
print(d)
print(len(d), "c" in d, d.get("c", 0))
```

<details>
<summary>答案与解析</summary>

`d["b"] = 2` 添加了新键；`d["a"] += 10` 把 1 变成 11。所以先输出 `{'a': 11, 'b': 2}`，然后是 `2 False 0`：长度 2，没有键 `"c"`，`get` 返回默认值 0。
</details>

<!-- exercise: valid-key -->
### 练习 6：哪个可以做键？

下面哪个值可以作为字典的键？

- `[1, 2]`
- `(1, 2)`
- `{1: 2}`
- `{1, 2}`

<details>
<summary>答案与解析</summary>

只有元组 `(1, 2)`。列表、字典、集合都是可修改的，不能做键（会报 `TypeError: unhashable type`）。
</details>

## 9.14 延伸阅读

- [Python 官方教程 · 5.4 集合、5.5 字典](https://docs.python.org/zh-cn/3/tutorial/datastructures.html#sets)
- [collections 模块：Counter、defaultdict 等](https://docs.python.org/zh-cn/3/library/collections.html)
- 下一课：[函数](../10-functions/zh-CN.md)
