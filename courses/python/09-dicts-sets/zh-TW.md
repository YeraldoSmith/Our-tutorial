# 9. 字典與集合

> 🎯 **本課目標**：用**字典**按「名字」而不是按「位置」查詢資料；掌握字典的增、刪、改、查和遍歷；學會用字典**計數**（統計詞頻）；理解為什麼列表不能做字典的鍵；用**集合**去重、求交集並集；知道為什麼在字典和集合裡查詢比在列表裡快得多。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：第 7、8 課

## 9.1 為什麼需要字典？

想做一個通訊錄。用列表的話，要找「小紅」的電話，只能從頭到尾一個個比對。而現實中的通訊錄、字典、詞彙表都是**按名字直接查**的。

Python 的**字典**（dict）就是這樣一種結構：它儲存很多對「**鍵**（key）→ **值**（value）」，可以用鍵直接取出值。

```python
phone = {"小明": "138-0000-1111", "小紅": "139-2222-3333", "小剛": "137-4444-5555"}
print(phone["小紅"])        # 用鍵取值：139-2222-3333
print(len(phone))           # 3：有 3 對鍵值
print(phone)
```

字典用**花括號** `{ }`，每一對寫成 `鍵: 值`，對與對之間用逗號隔開。

## 9.2 取值：`[]` 和 `get`

```python
ages = {"小明": 12, "小紅": 13}
print(ages["小明"])                 # 12
print(ages.get("小紅"))             # 13
print(ages.get("小剛"))             # None：鍵不存在時返回 None，不報錯
print(ages.get("小剛", 0))          # 0：也可以指定一個預設值
print("小剛" in ages)               # False：in 檢查的是「鍵」
```

用 `字典[鍵]` 取一個**不存在**的鍵會報錯：

```python norun
ages = {"小明": 12}
print(ages["小剛"])
```

```text
KeyError: '小剛'
```

所以不確定鍵在不在時，用 `in` 先判斷，或者用 `get`。

## 9.3 增加、修改、刪除

字典是**可變的**：

```python
stock = {"蘋果": 5, "香蕉": 3}
stock["橙子"] = 10          # 鍵不存在：新增新的一對
stock["蘋果"] = 8           # 鍵已存在：修改它的值
stock["香蕉"] += 2          # 在原來的值上加 2
print(stock)

del stock["橙子"]           # 刪除一對
n = stock.pop("香蕉")       # 刪除並返回它的值
print(n, stock)
```

> [!NOTE]
> 一個字典裡**鍵不會重複**：給已經存在的鍵賦值，只會覆蓋原來的值。值可以重複，比如兩個人可以同歲。

## 9.4 遍歷字典

```python
scores = {"語文": 90, "數學": 85, "英語": 92}

for subject in scores:                      # 直接遍歷：得到的是鍵
    print(subject)

for subject, score in scores.items():       # .items()：同時得到鍵和值（最常用）
    print(f"{subject}：{score} 分")

print(list(scores.keys()))                  # 所有的鍵
print(list(scores.values()))                # 所有的值
print("總分：", sum(scores.values()))
```

字典會**記住插入的順序**（Python 3.7 起），遍歷時按新增的先後出現。

## 9.5 用字典計數

「統計每個東西出現了幾次」是程式設計中最常見的任務之一，字典最擅長：

```python
text = "the cat and the dog and the bird"
counts = {}
for word in text.split():
    counts[word] = counts.get(word, 0) + 1     # 沒見過就從 0 開始，再加 1
print(counts)

# 按出現次數從多到少排序
for word, n in sorted(counts.items(), key=lambda kv: kv[1], reverse=True):
    print(word, n)
```

`counts.get(word, 0) + 1` 是計數的經典寫法：第一次遇到這個詞時 `get` 返回 0。`sorted(..., key=lambda kv: kv[1])` 表示按每一對的**第二項**（次數）排序，`lambda` 是一個臨時的小函式，下一課學函式時會詳細講。

Python 標準庫還有一個專門計數的工具 `Counter`：

```python
from collections import Counter

counts = Counter("the cat and the dog and the bird".split())
print(counts.most_common(2))      # 出現最多的 2 個：[('the', 3), ('and', 2)]
print(Counter("hello"))           # 也能數字母
```

## 9.6 字典推導式

和列表推導式（第 8 課）類似：

```python
squares = {n: n * n for n in range(1, 6)}
print(squares)                            # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

prices = {"蘋果": 5, "西瓜": 20, "葡萄": 12}
cheap = {k: v for k, v in prices.items() if v < 15}
print(cheap)                              # 只保留便宜的
```

## 9.7 組合起來：列表裡放字典，字典裡放列表

真實的資料往往是巢狀的。例如一個班的學生資訊：

```python
students = [
    {"name": "小明", "age": 12, "hobbies": ["足球", "畫畫"]},
    {"name": "小紅", "age": 13, "hobbies": ["鋼琴"]},
]
for s in students:
    print(f"{s['name']}（{s['age']} 歲）喜歡：{'、'.join(s['hobbies'])}")

students[1]["hobbies"].append("游泳")     # 先取出字典，再取出其中的列表，再修改
print(students[1])
```

這種「列表 + 字典」的結構和網路上最常見的資料格式 **JSON** 幾乎一模一樣，第 11 課會學怎樣把它存進檔案。

## 9.8 什麼可以做鍵？

鍵必須是**不可修改**（可雜湊，hashable）的值：數字、字串、元組都可以；列表、字典這類可以修改的值不行。

```python
distance = {("北京", "上海"): 1213, ("北京", "廣州"): 2120}   # 元組做鍵：表示「一對城市」
print(distance[("北京", "上海")])
```

```python norun
bad = {["北京", "上海"]: 1213}
```

```text
TypeError: unhashable type: 'list'
```

> [!NOTE]
> 為什麼？字典靠鍵的「雜湊值」（一個由內容算出來的數）來決定把它放在哪裡。如果鍵的內容能改，雜湊值也會變，就再也找不到它了。

## 9.9 集合：不重複的一堆東西

**集合**（set）也用花括號，但只有值、沒有鍵。它有兩個特點：**元素不重複**、**沒有順序**。

```python
tags = {"python", "教程", "python", "程式設計"}
print(tags)                       # 重複的 "python" 只保留一個（順序可能和你寫的不一樣）
print(len(tags))                  # 3

nums = [3, 1, 3, 2, 1, 3]
print(set(nums))                  # 去重：{1, 2, 3}
print(sorted(set(nums)))          # 去重並排序：[1, 2, 3]

tags.add("入門")
tags.discard("程式設計")              # 刪除（元素不存在也不報錯）
print("教程" in tags)             # True
```

> [!WARNING]
> 空集合要寫 `set()`。`{}` 表示的是**空字典**！

集合最有用的是**集合運算**：

```python
a = {"小明", "小紅", "小剛"}          # 參加了數學競賽
b = {"小紅", "小剛", "小麗"}          # 參加了英語競賽
print(a & b)        # 交集：兩個都參加的
print(a | b)        # 並集：至少參加一個的
print(a - b)        # 差集：只參加數學的
print(a ^ b)        # 對稱差：只參加了一個的
```

## 9.10 為什麼字典和集合查詢特別快？

在列表裡查詢 `x in lst`，Python 要從頭到尾一個個比較；列表有一百萬個元素，最壞要比較一百萬次。字典和集合用**雜湊表**儲存，直接根據雜湊值算出位置，幾乎**一步**就能找到，不管有多少元素：

```python
import time

n = 200_000
lst = list(range(n))
st = set(lst)

start = time.perf_counter()
for _ in range(200):
    (n - 1) in lst               # 要找的數在最後面
t_list = time.perf_counter() - start

start = time.perf_counter()
for _ in range(200):
    (n - 1) in st
t_set = time.perf_counter() - start

print(f"列表：{t_list:.4f} 秒   集合：{t_set:.6f} 秒")
```

所以，**要頻繁判斷「在不在」時，用集合或字典，不要用列表**。

## 9.11 常見錯誤

```python norun
counts = {}
counts["apple"] += 1
```

```text
KeyError: 'apple'
```

鍵還不存在，沒法在原來的值上加 1。用 `counts["apple"] = counts.get("apple", 0) + 1`。

```python norun
d = {"a": 1, "b": 2, "c": 3}
for k in d:
    if d[k] < 2:
        del d[k]
```

```text
RuntimeError: dictionary changed size during iteration
```

遍歷字典的同時不能刪除元素。改成遍歷一份副本：`for k in list(d):`。

```python norun
s = {}
s.add(1)
```

```text
AttributeError: 'dict' object has no attribute 'add'
```

`{}` 是空字典，空集合要寫 `set()`。

## 9.12 小結

- 字典 `{鍵: 值}` 按鍵查詢值；`d[k]` 鍵不存在會 `KeyError`，`d.get(k, 預設值)` 不會。
- 增改都是 `d[k] = v`；刪除用 `del d[k]` 或 `d.pop(k)`；`in` 檢查鍵。
- 遍歷用 `for k, v in d.items()`；字典保持插入順序。
- 計數：`d[x] = d.get(x, 0) + 1`，或 `collections.Counter`；排序用 `sorted(d.items(), key=...)`。
- 鍵必須不可修改（數字、字串、元組）；列表不能做鍵。
- 集合 `{a, b}` 元素不重複、無序；`set()` 是空集合；`& | - ^` 是交、並、差、對稱差。
- 在字典和集合裡查詢幾乎是一步完成，比列表快得多。

## 9.13 練習

<!-- exercise: word-count -->
### 練習 1：詞頻統計

輸入一行英文單詞（空格隔開），按單詞**第一次出現的順序**，每行輸出一個單詞和它出現的次數（空格隔開）。

```text
樣例輸入：the cat and the dog
樣例輸出：
the 2
cat 1
and 1
dog 1
```

<details>
<summary>提示</summary>

用 `counts[w] = counts.get(w, 0) + 1` 計數；字典會按插入順序遍歷。
</details>

<!-- exercise: top-word -->
### 練習 2：出現最多的詞

輸入一行單詞，輸出出現次數最多的單詞和次數（空格隔開）。如果有幾個單詞次數一樣多，輸出其中**按字母順序最小**的那個。

```text
樣例輸入：b a c a b
樣例輸出：a 2
```

<details>
<summary>提示</summary>

先計數，再用 `min(counts, key=lambda w: (-counts[w], w))`：先比次數（取負數讓多的排前面），次數一樣再比單詞本身。
</details>

<!-- exercise: phonebook -->
### 練習 3：通訊錄

第一行是整數 `n`，接下來 `n` 行每行一個名字和電話（空格隔開）。然後是整數 `m`，接下來 `m` 行每行一個要查詢的名字。對每個查詢，輸出電話；如果查不到，輸出 `not found`。

```text
樣例輸入：
2
Tom 123
Amy 456
3
Amy
Bob
Tom
樣例輸出：
456
not found
123
```

<details>
<summary>提示</summary>

先把 `n` 行讀進字典：`name, number = input().split()`，`book[name] = number`；查詢時用 `book.get(name, "not found")`。
</details>

<!-- exercise: common -->
### 練習 4：共同的數

輸入兩行整數（空格隔開），輸出**兩行都出現過**的數，從小到大排列、不重複，空格隔開。如果沒有，輸出 `none`。

```text
樣例輸入：
1 2 3 4 4
3 4 5 4
樣例輸出：3 4
```

<details>
<summary>提示</summary>

把兩行分別變成集合，求交集 `&`，再 `sorted`。
</details>

<!-- exercise: dict-predict -->
### 練習 5：讀程式碼，寫輸出

```python
d = {"a": 1}
d["b"] = 2
d["a"] += 10
print(d)
print(len(d), "c" in d, d.get("c", 0))
```

<details>
<summary>答案與解析</summary>

`d["b"] = 2` 新增了新鍵；`d["a"] += 10` 把 1 變成 11。所以先輸出 `{'a': 11, 'b': 2}`，然後是 `2 False 0`：長度 2，沒有鍵 `"c"`，`get` 返回預設值 0。
</details>

<!-- exercise: valid-key -->
### 練習 6：哪個可以做鍵？

下面哪個值可以作為字典的鍵？

- `[1, 2]`
- `(1, 2)`
- `{1: 2}`
- `{1, 2}`

<details>
<summary>答案與解析</summary>

只有元組 `(1, 2)`。列表、字典、集合都是可修改的，不能做鍵（會報 `TypeError: unhashable type`）。
</details>

## 9.14 延伸閱讀

- [Python 官方教程 · 5.4 集合、5.5 字典](https://docs.python.org/zh-cn/3/tutorial/datastructures.html#sets)
- [collections 模組：Counter、defaultdict 等](https://docs.python.org/zh-cn/3/library/collections.html)
- 下一課：[函式](../10-functions/zh-CN.md)
