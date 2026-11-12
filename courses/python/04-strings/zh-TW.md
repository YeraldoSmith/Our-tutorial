# 4. 字串

> 🎯 **本課目標**：用下標和切片取出字串的任意部分；掌握 `len`、`in` 和十幾個最常用的字串方法；理解「字串不可修改」；熟練使用 f 字串格式化。
>
> ⏱ **預計用時**：45 分鐘　｜　📋 **前置知識**：第 2、3 課

文書處理是程式設計中最常見的任務之一：處理使用者名稱、分析文章、生成報表、清洗資料……這一課把字串（`str`）學透。

## 4.1 字串是一串字元

字串就是**按順序排列的字元**。每個字元都有一個位置編號，叫**下標**（index），**從 0 開始**：

```text
 字串:   P   y   t   h   o   n
 下標:     0   1   2   3   4   5
 負下標:  -6  -5  -4  -3  -2  -1
```

用方括號 `[下標]` 取出某個字元：

```python
word = "Python"
print(word[0])     # P：第一個字元
print(word[3])     # h
print(word[-1])    # n：負數從後往前數，-1 是最後一個
print(word[-2])    # o
print(len(word))   # 6：長度（字元個數）
```

> [!IMPORTANT]
> 下標**從 0 開始**，所以長度為 6 的字串，最後一個字元的下標是 5（= 長度 − 1）。這是所有主流程式語言的共同習慣，一開始容易搞錯，多練幾次就習慣了。

訪問不存在的下標會報錯：

```python norun
word = "Python"
print(word[6])
```

```text
IndexError: string index out of range
```

## 4.2 切片：取出一段

`字串[開始:結束]` 取出從「開始」到「結束」**之前**的部分（**包含開始，不包含結束**）：

```python
word = "Python"
print(word[0:2])    # Py：下標 0、1
print(word[2:5])    # tho：下標 2、3、4
print(word[:3])     # Pyt：省略開始 = 從頭開始
print(word[3:])     # hon：省略結束 = 一直到末尾
print(word[-3:])    # hon：最後三個字元
print(word[:])      # Python：整個字串
```

可以這樣記：**切片的位置是「字元之間的縫隙」**：

```text
   | P | y | t | h | o | n |
   0   1   2   3   4   5   6
word[2:5] 就是從縫隙 2 切到縫隙 5 → "tho"
```

### 步長

第三個數字是**步長**（每次跳幾個）：

```python
digits = "0123456789"
print(digits[::2])     # 02468：每隔一個取一個
print(digits[1::2])    # 13579
print(digits[::-1])    # 9876543210：步長為 -1，反過來！
```

`s[::-1]` 是反轉字串的經典寫法。

切片和下標不同：**切片越界不會報錯**，只是取到能取的部分：`"abc"[1:100]` 得到 `"bc"`。

## 4.3 字串不可修改

字串一旦建立，就**不能修改**其中的字元：

```python norun
word = "Python"
word[0] = "J"
```

```text
TypeError: 'str' object does not support item assignment
```

想「修改」，就建立一個新的字串：

```python
word = "Python"
word = "J" + word[1:]
print(word)     # Jython
```

這個特性叫**不可變**（immutable）。下面要學的所有字串方法，都是**返回一個新字串**，原來的字串不變：

```python
name = "alice"
upper_name = name.upper()
print(name)          # alice：原字串沒變
print(upper_name)    # ALICE
```

## 4.4 拼接、重複和包含

```python
first = "Hello"
second = "World"
print(first + ", " + second + "!")   # 拼接：Hello, World!
print("ha" * 3)                       # 重複：hahaha
print("-" * 20)                       # 畫一條分隔線
print("ell" in first)                 # True：判斷是否包含
print("xyz" not in first)             # True
```

## 4.5 常用字串方法

**方法**（method）是屬於某個值的函式，用 `值.方法名()` 呼叫。字串有幾十個方法，下面是最常用的：

### 大小寫

```python
s = "hello World"
print(s.upper())        # HELLO WORLD
print(s.lower())        # hello world
print(s.title())        # Hello World：每個單詞首字母大寫
print(s.capitalize())   # Hello world：只有第一個字母大寫
```

### 去掉空白

使用者輸入的內容經常帶著多餘的空格或換行：

```python
s = "   python   \n"
print("[" + s.strip() + "]")    # [python]：去掉兩邊的空白
print("[" + s.lstrip() + "]")   # 只去左邊
print("[" + s.rstrip() + "]")   # 只去右邊
```

### 查詢和替換

```python
s = "I like cats. Cats are cute."
print(s.find("cats"))           # 7：第一次出現的下標
print(s.find("dogs"))           # -1：找不到返回 -1
print(s.count("ats"))           # 2：出現了幾次
print(s.replace("cats", "dogs"))   # 替換：I like dogs. Cats are cute.
print(s.startswith("I"))        # True
print(s.endswith("."))          # True
```

注意 `replace` 和 `find` **區分大小寫**：`"Cats"` 沒有被替換。

### 分割和連線

這一對方法極其常用：

```python
sentence = "apple banana cherry"
words = sentence.split()        # 按空白分割
print(words)                    # ['apple', 'banana', 'cherry']

date = "2026-11-12"
parts = date.split("-")         # 按 "-" 分割
print(parts)                    # ['2026', '11', '12']

print("/".join(parts))          # 2026/11/12：用 "/" 連線起來
print(" + ".join(["1", "2", "3"]))   # 1 + 2 + 3
```

`split` 得到的是一個**列表**（方括號括起來的一組值，第 8 課詳細講）。`join` 反過來，把一組字串用指定的分隔符連起來。

### 判斷內容

```python
print("2026".isdigit())    # True：全是數字
print("abc".isalpha())     # True：全是字母
print("abc123".isalnum())  # True：字母或數字
print("   ".isspace())     # True：全是空白
print("Hello".islower())   # False
```

### 方法速查表

| 方法 | 作用 |
|------|------|
| `s.upper()` / `s.lower()` | 轉大寫 / 小寫 |
| `s.strip()` | 去掉兩端空白（也可以 `strip("x")` 去掉指定字元） |
| `s.replace(a, b)` | 把 a 全部替換成 b |
| `s.find(x)` | x 第一次出現的下標，沒有則 -1 |
| `s.count(x)` | x 出現的次數 |
| `s.split(sep)` | 按分隔符切成列表（不寫 sep 就按空白） |
| `sep.join(列表)` | 用 sep 把列表裡的字串連起來 |
| `s.startswith(x)` / `s.endswith(x)` | 是否以 x 開頭 / 結尾 |
| `s.isdigit()` / `s.isalpha()` | 是否全是數字 / 字母 |
| `s.center(w)` / `s.ljust(w)` / `s.rjust(w)` | 在寬度 w 內居中 / 左對齊 / 右對齊 |
| `s.zfill(w)` | 左邊補 0 到寬度 w |

方法可以**連著呼叫**：

```python
raw = "  Hello, PYTHON!  "
print(raw.strip().lower().replace("!", "?"))   # hello, python?
```

## 4.6 f 字串進階

第 2、3 課已經用過 f 字串。再補充幾個實用技巧：

```python
name = "Bob"
score = 87.456
print(f"{name:>10}|")          # 右對齊，寬度 10
print(f"{name:<10}|")          # 左對齊
print(f"{name:^10}|")          # 居中
print(f"{name:*^10}")          # 居中，用 * 填充：***Bob****
print(f"{score:8.2f}|")        # 寬度 8，兩位小數
print(f"{score=}")             # 除錯小技巧：輸出 score=87.456
```

做一個整齊的表格：

```python
print(f"{'商品':<6}{'數量':>4}{'價格':>8}")
print(f"{'蘋果':<6}{3:>4}{4.5:>8.2f}")
print(f"{'香蕉':<6}{12:>4}{0.99:>8.2f}")
```

> [!NOTE]
> 中文字元在等寬字型裡通常佔兩個英文字元的寬度，所以中英文混排的表格可能對不太齊，這是正常的。

### 轉義和原始字串

複習第 1 課的跳脫字元：`\n` 換行、`\t` 製表符、`\\` 反斜槓、`\"` 引號。如果字串裡有很多反斜槓（例如 Windows 路徑、正規表示式），可以在前面加 `r`，表示**原始字串**，反斜槓不再轉義：

```python
print("C:\\new\\table")     # 普通字串需要寫兩個反斜槓
print(r"C:\new\table")      # 原始字串：所見即所得
```

## 4.7 遍歷字串（預告）

想對字串的**每個字元**做點事情，可以用 `for` 迴圈（第 7 課詳細講）：

```python
for ch in "abc":
    print(ch.upper())
```

輸出 `A`、`B`、`C` 三行。先混個眼熟。

## 4.8 小結

- 下標從 **0** 開始，`s[-1]` 是最後一個字元；`len(s)` 是長度。
- 切片 `s[開始:結束:步長]`：包含開始、不包含結束；`s[::-1]` 反轉。
- 字串**不可修改**，所有方法都返回新字串。
- `+` 拼接，`*` 重複，`in` 判斷包含。
- 常用方法：`upper/lower`、`strip`、`replace`、`find/count`、`split/join`、`startswith/endswith`、`isdigit/isalpha`。
- f 字串：`{x:>10}` 對齊、`{x:.2f}` 小數、`{x=}` 除錯；`r"..."` 是原始字串。

## 4.9 練習

<!-- exercise: reverse -->
### 練習 1：反轉字串

變數 `s = "stressed"`。輸出它反過來的樣子：

```text
desserts
```

<details>
<summary>提示</summary>

切片的步長寫 `-1`：`s[::-1]`。
</details>

<!-- exercise: slice-predict -->
### 練習 2：讀程式碼，寫輸出

```python
s = "Hello, World"
print(s[0], s[-1])
print(s[7:12])
print(s[:5].upper())
print(len(s))
```

<details>
<summary>答案與解析</summary>

`s[0]` 是 `H`，`s[-1]` 是 `d`；下標 7 到 11 是 `World`；前 5 個字元 `Hello` 轉大寫是 `HELLO`；包括逗號和空格共 12 個字元。
</details>

<!-- exercise: clean-name -->
### 練習 3：整理使用者名稱

變數 `raw = "   aLiCe SMITH  "`。去掉兩邊的空格，並讓每個單詞首字母大寫，輸出：

```text
Alice Smith
```

<details>
<summary>提示</summary>

`raw.strip().title()`：先去空白，再轉換大小寫。
</details>

<!-- exercise: date-format -->
### 練習 4：日期格式轉換

變數 `date = "2026-11-12"`（年-月-日）。把它轉換成「日/月/年」的格式輸出：

```text
12/11/2026
```

<details>
<summary>提示 1</summary>

`parts = date.split("-")` 得到 `['2026', '11', '12']`，`parts[0]` 是年，`parts[2]` 是日。
</details>

<details>
<summary>提示 2</summary>

`print(parts[2] + "/" + parts[1] + "/" + parts[0])`，或者 `"/".join([parts[2], parts[1], parts[0]])`。
</details>

<!-- exercise: immutable -->
### 練習 5：不可變

`s = "cat"`，執行 `s.upper()` 之後再 `print(s)`，會輸出什麼？

- `CAT`
- `cat`
- 報錯
- `Cat`

<details>
<summary>答案與解析</summary>

`cat`。`upper()` 返回了一個新字串 `"CAT"`，但我們沒有把它儲存下來，`s` 本身沒有變。要改變 `s`，得寫 `s = s.upper()`。
</details>

<!-- exercise: count-vowels -->
### 練習 6：數母音

變數 `text = "programming is fun"`。分別統計字母 `a`、`i`、`o`、`u` 出現的次數，輸出它們的**總和**：

```text
5
```

<details>
<summary>提示</summary>

`text.count("a") + text.count("i") + ...`
</details>

## 4.10 延伸閱讀

- [Python 官方文件 · 字串方法](https://docs.python.org/zh-cn/3/library/stdtypes.html#string-methods)：所有方法的完整列表。
- [Python 官方教程 · 3.1.2 字串](https://docs.python.org/zh-cn/3/tutorial/introduction.html#text)
- 下一課：[輸入與互動](../05-input/zh-CN.md)
