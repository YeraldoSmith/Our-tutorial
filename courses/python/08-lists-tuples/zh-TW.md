# 8. 列表與元組

> 🎯 **本課目標**：用列表儲存一組資料，並進行增、刪、改、查、排序；用 `for` 和 `enumerate` 遍歷列表；初步掌握列表推導式；理解「兩個變數指向同一個列表」的陷阱；認識元組和解包。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：第 4、7 課

## 8.1 為什麼需要列表？

要儲存全班 40 個同學的成績，總不能建 40 個變數 `score1`、`score2`……**列表**（list）可以把很多個值放進**一個**變數裡，按順序排好：

```python
scores = [88, 92, 75, 64, 99]
names = ["小明", "小紅", "小剛"]
mixed = [1, "two", 3.0, True]      # 可以放不同型別（但一般不推薦）
empty = []                         # 空列表
print(scores)
print(len(scores))                 # 5：元素個數
```

列表用**方括號** `[ ]`，元素之間用**逗號**隔開。

## 8.2 下標和切片：和字串一樣

列表的下標、負下標和切片，與第 4 課的字串**完全相同**：

```python
fruits = ["蘋果", "香蕉", "橙子", "葡萄", "西瓜"]
print(fruits[0])       # 蘋果
print(fruits[-1])      # 西瓜
print(fruits[1:3])     # ['香蕉', '橙子']
print(fruits[::-1])    # 反轉
print("橙子" in fruits)  # True
```

## 8.3 列表可以修改

和字串不同，**列表是可變的**（mutable），可以直接改變其中的元素：

```python
scores = [88, 92, 75]
scores[2] = 80         # 把第 3 個元素改成 80
print(scores)          # [88, 92, 80]
```

### 新增元素

```python
nums = [1, 2, 3]
nums.append(4)         # 在末尾新增一個：[1, 2, 3, 4]
nums.insert(0, 0)      # 在下標 0 處插入：[0, 1, 2, 3, 4]
nums.extend([5, 6])    # 在末尾新增多個：[0, 1, 2, 3, 4, 5, 6]
print(nums)
```

### 刪除元素

```python
nums = [10, 20, 30, 20, 40]
nums.remove(20)        # 刪除第一個值為 20 的元素：[10, 30, 20, 40]
last = nums.pop()      # 刪除並返回最後一個：last = 40
first = nums.pop(0)    # 刪除並返回下標 0 的元素：first = 10
del nums[0]            # 按下標刪除
print(nums, last, first)
nums.clear()           # 清空
print(nums)            # []
```

### 查詢和統計

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
print(nums.index(5))   # 4：值 5 第一次出現的下標
print(nums.count(1))   # 2：值 1 出現的次數
print(min(nums), max(nums), sum(nums))   # 1 9 31
print(sum(nums) / len(nums))             # 平均值 3.875
```

### 排序

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]
nums.sort()                  # 原地排序（改變 nums 本身）
print(nums)                  # [1, 1, 2, 3, 4, 5, 6, 9]
nums.sort(reverse=True)      # 從大到小
print(nums)

data = [3, 1, 2]
new = sorted(data)           # 返回一個排好序的新列表，原列表不變
print(data, new)             # [3, 1, 2] [1, 2, 3]

words = ["banana", "Apple", "cherry"]
print(sorted(words))                  # ['Apple', 'banana', 'cherry']：大寫字母排在前面
print(sorted(words, key=str.lower))   # 忽略大小寫排序
print(sorted(words, key=len))         # 按長度排序
```

> [!IMPORTANT]
> `nums.sort()` **修改原列表**，返回值是 `None`；`sorted(nums)` **不修改原列表**，返回新列表。新手常寫 `nums = nums.sort()`，結果 `nums` 變成了 `None`！

### 方法速查表

| 操作 | 寫法 |
|------|------|
| 末尾新增 | `lst.append(x)` |
| 插入 | `lst.insert(i, x)` |
| 合併 | `lst.extend(另一個列表)` 或 `lst + 另一個列表` |
| 按值刪除 | `lst.remove(x)` |
| 按下標刪除並取出 | `lst.pop(i)`（不寫 i 是最後一個） |
| 查詢下標 / 計數 | `lst.index(x)` / `lst.count(x)` |
| 排序 | `lst.sort()` / `sorted(lst)` |
| 反轉 | `lst.reverse()` / `lst[::-1]` |
| 長度、最小、最大、求和 | `len`、`min`、`max`、`sum` |

## 8.4 遍歷列表

```python
names = ["小明", "小紅", "小剛"]
for name in names:
    print(f"你好，{name}")
```

需要**同時知道下標和值**時，用 `enumerate`：

```python
names = ["小明", "小紅", "小剛"]
for i, name in enumerate(names, start=1):
    print(f"{i}. {name}")
```

```text
1. 小明
2. 小紅
3. 小剛
```

同時遍歷兩個列表，用 `zip`：

```python
names = ["小明", "小紅", "小剛"]
scores = [88, 95, 72]
for name, score in zip(names, scores):
    print(f"{name}：{score} 分")
```

## 8.5 在迴圈中構造列表

最常見的模式：從空列表開始，迴圈中不斷 `append`：

```python
n = int(input())
numbers = []
for _ in range(n):
    numbers.append(int(input()))
print("你輸入了：", numbers)
print("最大值：", max(numbers))
```

如果 n 個數在**同一行**，用第 5 課的 `split` + `map`，再用 `list()` 轉成列表：

```python
numbers = list(map(int, input().split()))
print(numbers)
```

### 篩選

```python
scores = [88, 45, 92, 59, 75, 30]
passed = []
for s in scores:
    if s >= 60:
        passed.append(s)
print(passed)        # [88, 92, 75]
```

## 8.6 列表推導式

上面「迴圈 + 篩選 + append」的模式太常見了，Python 提供了一種簡潔的寫法，叫**列表推導式**（list comprehension）：

```python
scores = [88, 45, 92, 59, 75, 30]
passed = [s for s in scores if s >= 60]
print(passed)                          # [88, 92, 75]

squares = [x * x for x in range(1, 6)]
print(squares)                         # [1, 4, 9, 16, 25]

words = ["hello", "world"]
print([w.upper() for w in words])      # ['HELLO', 'WORLD']
```

讀法：`[對 x 做什麼  for x in 哪裡  if 什麼條件]`——「對每一個 x（滿足條件的），計算出新值，放進新列表」。

> [!TIP]
> 列表推導式適合**簡單**的變換和篩選。邏輯複雜時，用普通的 `for` 迴圈更清楚。

## 8.7 巢狀列表：表格和矩陣

列表的元素也可以是列表，用來表示二維表格：

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

## 8.8 陷阱：兩個名字指向同一個列表

```python
a = [1, 2, 3]
b = a          # b 和 a 指向「同一個」列表
b.append(4)
print(a)       # [1, 2, 3, 4]——a 也變了！
```

第 2 課說變數是「標籤」。`b = a` 並沒有複製列表，只是給**同一個列表**又貼了一張標籤。透過任何一個名字修改，另一個都能看到：

```text
a ──┐
    ├──► [1, 2, 3, 4]
b ──┘
```

想要一個獨立的副本，用 `copy()` 或切片：

```python
a = [1, 2, 3]
b = a.copy()       # 或 b = a[:] 或 b = list(a)
b.append(4)
print(a, b)        # [1, 2, 3] [1, 2, 3, 4]
```

> [!NOTE]
> 數字和字串是不可變的，所以 `y = x; y += 1` 不會影響 `x`——`y += 1` 是讓 `y` 指向一個新數字。只有可變物件（列表、後面要學的字典、集合）才會出現「改一個、兩個都變」的情況。

## 8.9 元組：不能修改的列表

**元組**（tuple）和列表很像，但用**圓括號**，並且**建立後不能修改**：

```python
point = (3, 4)
print(point[0])        # 3
colors = ("red", "green", "blue")
print(len(colors), "green" in colors)
```

```python norun
point[0] = 5           # TypeError: 'tuple' object does not support item assignment
```

什麼時候用元組？當一組資料**本來就不該變**時：座標 `(x, y)`、RGB 顏色 `(255, 128, 0)`、日期 `(2026, 11, 12)`。不可變帶來安全性：別人不會不小心改掉它。

### 打包和解包

其實你早就用過元組了：

```python
x, y = 3, 4        # 右邊 3, 4 是一個元組（打包），左邊把它拆開（解包）
x, y = y, x        # 第 2 課的交換技巧，原理就是元組
```

```python
point = (3, 4)
x, y = point
print(x, y)

first, *rest = [1, 2, 3, 4]   # * 收集剩下的所有元素
print(first, rest)            # 1 [2, 3, 4]
```

> [!NOTE]
> 只有一個元素的元組要加逗號：`(5,)`。`(5)` 只是帶括號的數字 5。

## 8.10 常見錯誤

```python norun
nums = [1, 2, 3]
print(nums[3])
```

```text
IndexError: list index out of range
```

長度為 3 的列表，合法下標是 0、1、2。

```python norun
nums = [3, 1, 2]
nums = nums.sort()
print(nums[0])
```

```text
TypeError: 'NoneType' object is not subscriptable
```

`sort()` 返回 `None`，`nums` 被覆蓋成了 `None`。直接寫 `nums.sort()`，或 `nums = sorted(nums)`。

```python norun
nums = [1, 2, 3]
nums.remove(5)
```

```text
ValueError: list.remove(x): x not in list
```

要刪除的值不存在。可以先 `if 5 in nums:` 判斷。

## 8.11 小結

- 列表 `[a, b, c]` 按順序儲存多個值；下標、切片、`in`、`len` 與字串相同。
- 列表**可變**：`append`、`insert`、`extend`、`remove`、`pop`、`del`、`sort`……
- `lst.sort()` 原地排序返回 `None`；`sorted(lst)` 返回新列表；`key=` 指定排序依據。
- 遍歷：`for x in lst`；要下標用 `enumerate`；並行遍歷用 `zip`。
- 列表推導式 `[表示式 for x in 序列 if 條件]`。
- `b = a` 不復制列表，`b = a.copy()` 才複製。
- 元組 `(a, b)` 不可修改；打包與解包讓多重賦值和交換成為可能。

## 8.12 練習

<!-- exercise: average -->
### 練習 1：平均分

第一行輸入整數 n，第二行輸入 n 個整數（空格隔開）。輸出它們的平均值，保留 2 位小數。

```text
樣例輸入：
4
90 85 77 60
樣例輸出：
78.00
```

<details>
<summary>提示</summary>

`n = int(input())`，`nums = list(map(int, input().split()))`，平均值是 `sum(nums) / n`。
</details>

<!-- exercise: sort-desc -->
### 練習 2：從大到小

輸入一行若干個整數（空格隔開），從大到小排序後輸出，空格隔開。

```text
樣例輸入：3 1 4 1 5 9 2 6
樣例輸出：9 6 5 4 3 2 1 1
```

<details>
<summary>提示</summary>

排序用 `sort(reverse=True)`；輸出時把每個數轉成字串，用 `" ".join(...)`。也可以寫 `print(*nums)`，`*` 會把列表拆開作為多個引數。
</details>

<!-- exercise: unique -->
### 練習 3：去掉重複

輸入一行若干個單詞（空格隔開），去掉重複的單詞，**保持第一次出現的順序**，輸出剩下的單詞。

```text
樣例輸入：apple banana apple cherry banana
樣例輸出：apple banana cherry
```

<details>
<summary>提示</summary>

準備一個空列表 `result`，遍歷每個單詞，如果 `word not in result` 就 `append`。
</details>

<!-- exercise: evens-squared -->
### 練習 4：列表推導式

輸入一行若干個整數，用**列表推導式**選出其中的偶數，並輸出它們的平方（空格隔開）。

```text
樣例輸入：1 2 3 4 5 6
樣例輸出：4 16 36
```

<details>
<summary>提示</summary>

`[x * x for x in nums if x % 2 == 0]`
</details>

<!-- exercise: alias-predict -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`b` 和 `a` 是同一個列表，`b.append(4)` 後 `a` 也是 `[1, 2, 3, 4]`；`c` 是副本，只有它多了 5。所以 `a` 輸出 `[1, 2, 3, 4]`，`b` 長 4，`c` 也長 4（`[1, 2, 3, 5]`）。
</details>

<!-- exercise: sort-return -->
### 練習 6：sort 的返回值

`x = [3, 1, 2].sort()` 執行之後，`x` 是什麼？

- `[1, 2, 3]`
- `[3, 1, 2]`
- `None`
- 報錯

<details>
<summary>答案與解析</summary>

`None`。`sort()` 原地排序，不返回新列表。想得到排序後的新列表請用 `sorted([3, 1, 2])`。
</details>

## 8.13 延伸閱讀

- [Python 官方教程 · 5.1 列表詳解](https://docs.python.org/zh-cn/3/tutorial/datastructures.html)
- [Python 排序指南](https://docs.python.org/zh-cn/3/howto/sorting.html)
- 下一課：[字典與集合](../09-dicts-sets/zh-CN.md)
