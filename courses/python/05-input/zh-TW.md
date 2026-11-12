# 5. 輸入與互動

> 🎯 **本課目標**：用 `input()` 讀取使用者輸入；理解「輸入得到的永遠是字串」，並用 `int()`、`float()` 轉換；一行讀取多個值；寫出第一批真正能和人互動的程式；瞭解做題時輸入是怎麼提供的。
>
> ⏱ **預計用時**：35 分鐘　｜　📋 **前置知識**：第 2～4 課

## 5.1 input()：讓程式聽你說話

到目前為止，程式裡的資料都是寫死的。要讓程式和使用者**互動**，需要讀取輸入。`input()` 會**暫停程式**，等使用者輸入一行文字並按回車，然後把這行文字交給程式：

```python
name = input("你叫什麼名字？")
print(f"你好，{name}！")
```

執行後，程式會顯示 `你叫什麼名字？` 然後停下來等待。在學習程式裡，**在下方的輸入框中輸入**（比如 `小明`）並按回車，程式就會繼續：

```text
你叫什麼名字？小明
你好，小明！
```

括號裡的文字叫**提示語**（prompt），會先顯示出來，告訴使用者該輸入什麼。也可以不寫提示語：`name = input()`。

> [!TIP]
> 如果你知道程式會要什麼輸入，也可以**在點「▶ 執行」之前**就把內容輸入到輸入框裡，按回車；程式開始執行時會自動收到這些輸入。

## 5.2 輸入得到的永遠是字串

這一點**非常重要**，也是新手最常踩的坑：

```python norun
age = input("你幾歲了？")
print(age + 1)
```

輸入 `18`，你會得到：

```text
TypeError: can only concatenate str (not "int") to str
```

為什麼？因為 `input()` 返回的**永遠是字串**——即使使用者輸入的是數字，得到的也是文字 `"18"`，而不是數字 `18`。要做計算，必須先**轉換**：

```python
age = int(input("你幾歲了？"))
print(f"明年你就 {age + 1} 歲了")
```

`int(input(...))` 是一個**巢狀呼叫**：先執行裡面的 `input()` 得到文字，再交給外面的 `int()` 轉成整數。

| 需要的型別 | 寫法 |
|------------|------|
| 文字 | `s = input()` |
| 整數 | `n = int(input())` |
| 小數 | `x = float(input())` |

## 5.3 一個簡單的計算器

```python
a = float(input("第一個數："))
b = float(input("第二個數："))
print(f"{a} + {b} = {a + b}")
print(f"{a} - {b} = {a - b}")
print(f"{a} × {b} = {a * b}")
print(f"{a} ÷ {b} = {a / b:.4f}")
```

試著執行並輸入 `7` 和 `2`。用 `float` 而不是 `int`，這樣使用者輸入小數也沒問題。（如果第二個數輸入 0，最後一行會報 `ZeroDivisionError`——下一課學了 `if` 就能處理了。）

## 5.4 一行輸入多個值

有時希望使用者在一行裡輸入好幾個數，用空格隔開，比如 `3 4 5`。用 `split()` 切開，再分別轉換：

```python input="3 4 5"
line = input("輸入三角形的三條邊（用空格隔開）：")
parts = line.split()          # "3 4 5" → ['3', '4', '5']
a = int(parts[0])
b = int(parts[1])
c = int(parts[2])
print("周長：", a + b + c)
```

更簡潔的寫法是用 `map`，它把一個函式應用到每一個元素上：

```python input="3 4 5"
a, b, c = map(int, input().split())
print(a + b + c)
```

`map(int, ...)` 對 `split()` 得到的每一段都呼叫 `int()`，再用同時賦值分給 `a`、`b`、`c`。這是做程式設計題時**最常用**的讀入方式之一，記下來！

## 5.5 清理輸入

使用者經常不小心多打空格。用 `strip()` 去掉首尾空白，用 `lower()` 統一大小寫，可以讓程式更「寬容」：

```python
answer = input("你喜歡 Python 嗎？(yes/no) ").strip().lower()
print(f"你的回答是：{answer}")
```

輸入 `  YES ` 也能得到 `yes`。

> [!NOTE]
> `int()` 本身會忽略首尾空白：`int(" 42 ")` 是 `42`。但 `int("4 2")`（中間有空格）或 `int("42.0")` 都會報錯。

## 5.6 做程式設計題時的輸入

練習裡的程式設計題，判題程式會**自動提供輸入**，然後比較你的輸出和期望輸出是否**完全一樣**。所以：

> [!IMPORTANT]
> 做程式設計題時，**`input()` 裡不要寫提示語**。提示語也會被列印出來，混進你的輸出，導致「答案錯誤」。在練習裡寫 `n = int(input())`，而不是 `n = int(input("請輸入："))`。

題目會這樣描述輸入和輸出：

```text
輸入：兩行，每行一個整數
輸出：它們的和

樣例輸入：        樣例輸出：
3                 8
5
```

樣例輸入的**每一行**對應程式裡的一次 `input()`。

## 5.7 完整例子：溫度換算

```python
c = float(input())          # 攝氏度
f = c * 9 / 5 + 32
print(f"{c:.1f}°C = {f:.1f}°F")
```

輸入 `36.6`，輸出 `36.6°C = 97.9°F`。

## 5.8 常見錯誤

### 忘記轉換型別

```python norun
a = input()
b = input()
print(a + b)     # 輸入 3 和 5，結果是 35！
```

不會報錯，但結果不對：兩個**字串**相加是拼接。記得 `int(input())`。

### 輸入的內容無法轉換

```python norun
n = int(input())     # 使用者輸入了 "abc" 或 "3.5"
```

```text
ValueError: invalid literal for int() with base 10: 'abc'
```

第 12 課會學 `try/except` 來優雅地處理這種情況。

### 讀取次數比輸入的行數多

```text
EOFError: EOF when reading a line
```

EOF 意思是「輸入結束了」。程式想再讀一行，但已經沒有輸入了。檢查 `input()` 的次數是否和題目給的輸入行數一致，或者一行裡的多個值是否應該用 `split()` 一次讀入。

## 5.9 小結

- `input("提示語")` 暫停程式、讀取一行輸入；在學習程式裡用下方輸入框輸入。
- `input()` **永遠返回字串**；計算前用 `int()` / `float()` 轉換。
- 一行多個值：`a, b = map(int, input().split())`。
- `strip()`、`lower()` 讓輸入處理更寬容。
- 做程式設計題時 `input()` **不寫提示語**；樣例輸入的每一行對應一次 `input()`。

## 5.10 練習

<!-- exercise: greet -->
### 練習 1：打招呼

讀入一行名字，輸出 `Hello, 名字!`。

```text
樣例輸入：Alice
樣例輸出：Hello, Alice!
```

<details>
<summary>提示</summary>

`name = input()`，然後 `print(f"Hello, {name}!")`。記住：不要在 `input()` 裡寫提示語。
</details>

<!-- exercise: sum-two -->
### 練習 2：兩數之和

輸入兩行，每行一個整數，輸出它們的和。

```text
樣例輸入：
3
5
樣例輸出：
8
```

<details>
<summary>提示</summary>

`a = int(input())`，`b = int(input())`，`print(a + b)`。
</details>

<!-- exercise: rectangle-line -->
### 練習 3：一行兩個數

輸入**一行**，包含長方形的長和寬（整數，空格隔開），輸出面積和周長，用空格隔開。

```text
樣例輸入：4 7
樣例輸出：28 22
```

<details>
<summary>提示</summary>

`w, h = map(int, input().split())`
</details>

<!-- exercise: celsius -->
### 練習 4：溫度換算

輸入一個攝氏溫度（可能是小數），輸出對應的華氏溫度，**保留 1 位小數**。公式：`F = C × 9 / 5 + 32`。

```text
樣例輸入：36.6
樣例輸出：97.9
```

<details>
<summary>提示</summary>

`c = float(input())`，用 `print(f"{f:.1f}")` 輸出。
</details>

<!-- exercise: input-type -->
### 練習 5：input 的型別

使用者輸入 `42` 後，`x = input()` 中 `x` 的型別是什麼？

- `int`
- `float`
- `str`
- 取決於使用者輸入了什麼

<details>
<summary>答案與解析</summary>

`str`。`input()` 永遠返回字串，即使看起來是數字。需要數字時用 `int(x)` 轉換。
</details>

## 5.11 延伸閱讀

- [Python 官方文件 · input()](https://docs.python.org/zh-cn/3/library/functions.html#input)
- [Python 官方文件 · map()](https://docs.python.org/zh-cn/3/library/functions.html#map)
- 下一課：[條件判斷](../06-conditions/zh-CN.md)
