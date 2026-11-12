# 2. 變數與資料型別

> 🎯 **本課目標**：用變數儲存資料並修改它；掌握變數命名規則；認識整數、小數、字串、布林值和 `None`；學會用 `type()` 檢視型別，用 `int()`、`str()` 等轉換型別。
>
> ⏱ **預計用時**：40 分鐘　｜　📋 **前置知識**：第 1 課

## 2.1 變數：給資料起個名字

上一課我們讓 Python 直接列印資料。但真正的程式需要**記住**資料，之後再使用。比如一個遊戲要記住玩家的分數、名字、血量。**變數**（variable）就是用來做這件事的：

```python
score = 100
name = "小明"
print(name, "的分數是", score)
```

輸出：

```text
小明 的分數是 100
```

`score = 100` 的意思是：**建立一個叫 `score` 的名字，讓它指向資料 `100`**。之後在程式裡寫 `score`，Python 就會用 `100` 來代替它。

可以把變數想象成一張**貼在資料上的標籤**：

```text
   score ──►  100
   name  ──►  "小明"
```

> [!IMPORTANT]
> 這裡的 `=` **不是數學裡的「等於」**，而是「**賦值**」：把右邊的值交給左邊的名字。讀的時候可以讀作「score **得到** 100」。判斷兩個東西是否相等，要用兩個等號 `==`（第 6 課會學）。

## 2.2 修改變數的值

變數之所以叫「變」量，是因為它的值可以改變：

```python
score = 100
print(score)
score = 120        # 標籤換到新資料上
print(score)
```

輸出：

```text
100
120
```

更常見的寫法是**在原來的基礎上**修改：

```python
score = 100
score = score + 10    # 先算右邊：100 + 10 = 110，再把結果交給 score
print(score)
```

`score = score + 10` 在數學裡是不成立的，但在程式設計裡很正常：**先計算右邊，再賦值給左邊**。因為太常用，Python 提供了簡寫：

```python
score = 100
score += 10     # 等同於 score = score + 10
score -= 30     # 等同於 score = score - 30
score *= 2      # 等同於 score = score * 2
print(score)    # (100 + 10 - 30) * 2 = 160
```

### 變數之間賦值

```python
a = 5
b = a       # b 也指向 5
a = 8       # a 換成 8，b 不受影響
print(a, b)
```

輸出 `8 5`。`b = a` 只是把 `a` **當時**的值交給 `b`，之後 `a` 再變，`b` 不會跟著變。

## 2.3 變數的命名規則

變數名不能隨便起，必須遵守下面的**規則**（違反會報錯）：

1. 只能包含**字母、數字和下劃線 `_`**；
2. **不能以數字開頭**：`player1` 可以，`1player` 不行；
3. **區分大小寫**：`Score` 和 `score` 是兩個不同的變數；
4. 不能使用 Python 的**關鍵字**，如 `if`、`for`、`class`、`True`、`None` 等。

還有一些**約定**（不遵守不會報錯，但大家都這麼寫）：

- 用**小寫字母加下劃線**，叫「蛇形命名」（snake_case）：`player_name`、`max_speed`。
- 名字要**有意義**：`total_price` 比 `tp`、`a`、`x1` 好得多。
- 永遠不變的值（常量）用**全大寫**：`PI = 3.14159`、`MAX_LEVEL = 99`。

| 名字 | 能用嗎？ | 說明 |
|------|----------|------|
| `age` | ✅ | |
| `user_name` | ✅ | 推薦的風格 |
| `userName` | ✅ | 能用，但 Python 習慣用下劃線 |
| `_count` | ✅ | 下劃線開頭有特殊含義，後面會學 |
| `2nd_place` | ❌ | 不能以數字開頭 |
| `my-name` | ❌ | `-` 會被當成減號 |
| `class` | ❌ | 是關鍵字 |
| `年齡` | ✅ | Python 3 允許中文名，但**不推薦**（不方便輸入，別人也可能看不懂） |

> [!TIP]
> 起名字時問問自己：三個月後再看這段程式碼，能一眼看懂這個變數存的是什麼嗎？

## 2.4 資料型別

Python 裡的每個值都有一個**型別**（type），型別決定了這個值能做什麼。最基本的四種：

| 型別 | 名稱 | 例子 | 用途 |
|------|------|------|------|
| `int` | 整數 | `42`、`-7`、`0` | 計數、年齡、分數 |
| `float` | 浮點數（小數） | `3.14`、`-0.5`、`2.0` | 價格、身高、溫度 |
| `str` | 字串（文字） | `"你好"`、`'abc'`、`""` | 名字、訊息 |
| `bool` | 布林值 | `True`、`False` | 是/否、開/關 |

用 `type()` 可以檢視一個值的型別：

```python
print(type(42))
print(type(3.14))
print(type("hello"))
print(type(True))
```

輸出：

```text
<class 'int'>
<class 'float'>
<class 'str'>
<class 'bool'>
```

### 幾個要注意的地方

- `2` 是 `int`，`2.0` 是 `float`，`"2"` 是 `str`——**看起來一樣，型別不同**。
- 布林值 `True` 和 `False` **首字母大寫**，寫成 `true` 會報 `NameError`。
- 大數字可以用下劃線分隔，更好讀：`population = 1_400_000_000`（下劃線會被忽略）。
- 還有一個特殊的值 **`None`**，表示「**什麼都沒有**」，型別是 `NoneType`。比如一個變數暫時還不知道該放什麼時：`winner = None`。

### 動態型別

在 Python 裡，變數本身沒有固定的型別，**型別屬於值**。同一個變數可以先指向整數，再指向字串：

```python
x = 10
print(type(x))
x = "十"
print(type(x))
```

這叫**動態型別**。很方便，但也容易出錯，所以好習慣是：一個變數始終存同一類東西。（Java、C、Rust 等語言是**靜態型別**：變數宣告時就確定型別，以後不能變。學到時你會看到區別。）

## 2.5 型別轉換

不同型別之間經常需要轉換。最常見的場景：**數字和文字不能直接拼在一起**。

```python norun
age = 18
print("我今年" + age + "歲")    # ❌ 報錯
```

```text
TypeError: can only concatenate str (not "int") to str
```

意思是：字串只能和字串用 `+` 連線，不能和整數連線。解決辦法是**把數字轉成字串**：

```python
age = 18
print("我今年" + str(age) + "歲")
```

常用的轉換函式：

| 函式 | 作用 | 例子 | 結果 |
|------|------|------|------|
| `int(x)` | 轉成整數 | `int("42")`、`int(3.9)` | `42`、`3`（直接去掉小數部分，**不是**四捨五入） |
| `float(x)` | 轉成小數 | `float("3.5")`、`float(2)` | `3.5`、`2.0` |
| `str(x)` | 轉成字串 | `str(100)` | `"100"` |
| `bool(x)` | 轉成布林值 | `bool(0)`、`bool("hi")` | `False`、`True` |

```python
print(int("42") + 8)       # 50：先把文字 "42" 變成數字
print("42" + "8")          # 428：兩個字串拼接
print(int(3.9))            # 3
print(round(3.9))          # 4：四捨五入用 round
```

不是什麼都能轉換：`int("abc")` 和 `int("3.5")` 都會報 `ValueError`，因為它們不是整數的樣子。

> [!NOTE]
> `bool()` 的規則：數字 `0`、`0.0`、空字串 `""`、`None` 會變成 `False`，其他幾乎所有值都是 `True`。第 6 課學條件判斷時會用到。

## 2.6 同時給多個變數賦值

```python
x, y, z = 1, 2, 3        # 一次給三個變數賦值
print(x, y, z)

a = b = 0                # 兩個變數都等於 0
print(a, b)
```

一個非常實用的技巧——**交換兩個變數的值**：

```python
left = "蘋果"
right = "香蕉"
left, right = right, left
print(left, right)       # 香蕉 蘋果
```

在很多其他語言裡，交換需要一個臨時變數；Python 一行就搞定了。

## 2.7 f 字串：把變數放進文字裡

用 `+` 和 `str()` 拼接很麻煩。Python 提供了更好的方式——**f 字串**（f-string）：在字串前面加一個 `f`，就可以用 `{變數名}` 把變數直接放進去：

```python
name = "小明"
age = 18
height = 1.756
print(f"我叫{name}，今年{age}歲，身高{height}米")
print(f"明年我就{age + 1}歲了")            # 花括號裡可以寫表示式
print(f"身高約 {height:.2f} 米")           # :.2f 表示保留 2 位小數
```

輸出：

```text
我叫小明，今年18歲，身高1.756米
明年我就19歲了
身高約 1.76 米
```

f 字串是現代 Python 最常用的格式化方式，後面幾乎每課都會用到。第 4 課會詳細講它的更多用法。

## 2.8 常見錯誤

### 使用前沒有賦值

```python norun
print(total)
total = 100
```

```text
NameError: name 'total' is not defined
```

程式從上往下執行，第 1 行時 `total` 還不存在。**先賦值，後使用**。

### 拼寫不一致

```python norun
user_name = "小紅"
print(username)        # 少了下劃線
```

同樣是 `NameError`。Python 不會猜你的意思，名字必須一字不差。

### 數字和字串相加

```python norun
count = 5
print("數量：" + count)
```

`TypeError`。用 `str(count)`，或者乾脆用 f 字串：`print(f"數量：{count}")`。

## 2.9 小結

- `變數名 = 值` 把值交給變數；`=` 是**賦值**，不是等於。
- 變數可以重新賦值；`x += 1` 是 `x = x + 1` 的簡寫。
- 命名規則：字母/數字/下劃線，不能以數字開頭，區分大小寫，不能用關鍵字；習慣用 `snake_case`，常量全大寫。
- 基本型別：`int`、`float`、`str`、`bool`，還有表示「沒有」的 `None`；用 `type()` 檢視。
- 型別轉換：`int()`、`float()`、`str()`、`bool()`；`int()` 直接截斷小數，四捨五入用 `round()`。
- `a, b = b, a` 交換變數；f 字串 `f"…{變數}…"` 把變數放進文字。

## 2.10 練習

<!-- exercise: rectangle -->
### 練習 1：長方形面積

建立兩個變數 `width = 8` 和 `height = 5`，計算面積並儲存到變數 `area` 中，最後輸出：

```text
40
```

<details>
<summary>提示</summary>

`area = width * height`，然後 `print(area)`。
</details>

<!-- exercise: swap -->
### 練習 2：交換

下面的起始程式碼中，`a` 是 `1`，`b` 是 `2`。不要修改前兩行，在後面加一行程式碼交換它們的值，讓程式輸出：

```text
2 1
```

<details>
<summary>提示</summary>

Python 的同時賦值：`a, b = b, a`。
</details>

<!-- exercise: valid-name -->
### 練習 3：合法的變數名

下面哪一個是**合法**的 Python 變數名？

- `3d_model`
- `my-score`
- `max_speed`
- `for`

<details>
<summary>答案與解析</summary>

`max_speed`。`3d_model` 以數字開頭；`my-score` 含有減號；`for` 是關鍵字。
</details>

<!-- exercise: predict-vars -->
### 練習 4：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`y = x` 時 `y` 得到 `10`；之後 `x` 變成 `15`，`y` 變成 `20`。`"x" + "y"` 是兩個字串拼接得到 `xy`。`int("7")` 是 `7`，`int(2.9)` 去掉小數得 `2`，合計 `9`。
</details>

<!-- exercise: fstring-card -->
### 練習 5：名片

已經有變數 `name = "Alice"`、`age = 20`、`city = "Paris"`。用 **f 字串**輸出：

```text
Name: Alice, Age: 20, City: Paris
```

<details>
<summary>提示</summary>

`print(f"Name: {name}, Age: {age}, City: {city}")`
</details>

<!-- exercise: types -->
### 練習 6：型別

`type("3.14")` 的結果是什麼？

- `<class 'float'>`
- `<class 'int'>`
- `<class 'str'>`
- `<class 'bool'>`

<details>
<summary>答案與解析</summary>

有引號，所以是字串 `str`。看起來像數字的文字，仍然是文字。
</details>

## 2.11 延伸閱讀

- [Python 官方教程 · 3.1 把 Python 當計算器](https://docs.python.org/zh-cn/3/tutorial/introduction.html)
- [PEP 8 命名約定](https://peps.python.org/pep-0008/#naming-conventions)：Python 官方的程式碼風格指南（英文）。
- 下一課：[數字與計算](../03-numbers-math/zh-CN.md)
