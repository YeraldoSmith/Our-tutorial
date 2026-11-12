# 6. 條件判斷

> 🎯 **本課目標**：用比較運算子和邏輯運算子寫出條件；用 `if`、`elif`、`else` 讓程式根據情況做不同的事；理解**縮排**在 Python 中的意義；瞭解條件表示式和 `match` 語句。
>
> ⏱ **預計用時**：45 分鐘　｜　📋 **前置知識**：第 2～5 課

## 6.1 程式也需要「做決定」

生活中到處是判斷：**如果**下雨，就帶傘；**如果**考了 60 分以上，就及格，**否則**補考。程式也一樣：根據不同的情況執行不同的程式碼。這叫**條件判斷**（也叫分支）。

```python
temperature = 35
if temperature > 30:
    print("好熱！記得多喝水。")
print("祝你今天愉快")
```

輸出：

```text
好熱！記得多喝水。
祝你今天愉快
```

把 `temperature` 改成 `20` 再執行，第一句就不會出現了。

## 6.2 比較運算子

條件的結果是**布林值**：`True`（成立）或 `False`（不成立）。

| 運算子 | 含義 | 例子 | 結果 |
|--------|------|------|------|
| `==` | 等於 | `3 == 3` | `True` |
| `!=` | 不等於 | `3 != 4` | `True` |
| `>` | 大於 | `5 > 7` | `False` |
| `<` | 小於 | `5 < 7` | `True` |
| `>=` | 大於等於 | `7 >= 7` | `True` |
| `<=` | 小於等於 | `8 <= 7` | `False` |

```python
age = 18
print(age >= 18)        # True
print(age == 20)        # False
print("abc" == "abc")   # True：字串也能比較
print("Abc" == "abc")   # False：大小寫不同
```

> [!WARNING]
> **`=` 是賦值，`==` 才是比較。** 寫成 `if age = 18:` 會報語法錯誤。這是所有語言的新手都會犯的錯誤。

Python 還允許**連續比較**，和數學寫法一樣：

```python
x = 5
print(0 < x < 10)      # True：x 在 0 和 10 之間
```

## 6.3 if 語句的結構

```text
if 條件:
    條件成立時執行的程式碼
    （可以有很多行）
後面的程式碼（無論如何都會執行）
```

兩個關鍵點：

1. `if 條件` 後面**必須有冒號** `:`。
2. 屬於 `if` 的程式碼**必須縮排**（統一用 4 個空格）。縮排的程式碼組成一個**程式碼區塊**；回到原來的縮排位置，就表示程式碼區塊結束了。

```python
score = 95
if score >= 90:
    print("優秀！")       # 屬於 if
    print("繼續保持")     # 也屬於 if
print("成績已記錄")        # 不屬於 if，總會執行
```

> [!IMPORTANT]
> 在 C、Java、JavaScript 裡，程式碼區塊用花括號 `{ }` 表示；在 Python 裡，程式碼區塊**完全由縮排決定**。所以 Python 對縮排非常嚴格：同一個程式碼區塊的每一行，前面的空格數必須一樣。學習程式的編輯器在你按回車時會自動縮排，按 <kbd>Tab</kbd> 插入 4 個空格。

## 6.4 else：否則

```python
age = int(input())
if age >= 18:
    print("成年人")
else:
    print("未成年人")
```

`else` 後面也要有冒號，它的程式碼區塊在條件**不成立**時執行。`if` 和 `else` 兩個分支**一定會走且只走一個**。

### 判斷奇偶

```python
n = int(input())
if n % 2 == 0:
    print("偶數")
else:
    print("奇數")
```

## 6.5 elif：多個條件

情況不止兩種時，用 `elif`（else if 的縮寫）：

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

Python **從上往下**逐個檢查條件，**遇到第一個成立的就執行它，然後跳過剩下的全部**。所以輸入 `85` 時：`85 >= 90` 不成立 → `85 >= 80` 成立 → 輸出 `B`，後面的不再檢查。

> [!TIP]
> 正因為是「從上往下、只走一個」，`elif score >= 80` 不需要寫成 `elif 80 <= score < 90`——能走到這裡，說明 `score >= 90` 已經不成立了。但**順序很重要**：如果把 `score >= 60` 放在最前面，那麼 95 分也會得到 D！

## 6.6 邏輯運算子：and、or、not

把多個條件組合起來：

| 運算子 | 含義 | 例子 |
|--------|------|------|
| `and` | 並且（兩個都成立才成立） | `age >= 18 and has_ticket` |
| `or` | 或者（有一個成立就成立） | `day == "週六" or day == "週日"` |
| `not` | 不（取反） | `not is_raining` |

```python
age = 20
has_ticket = True
if age >= 18 and has_ticket:
    print("可以入場")

day = "週日"
if day == "週六" or day == "週日":
    print("週末愉快！")

is_raining = False
if not is_raining:
    print("出去散步吧")
```

### 一個經典例子：閏年

閏年的規則：能被 4 整除**並且**不能被 100 整除，**或者**能被 400 整除。

```python
year = int(input())
if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
    print("閏年")
else:
    print("平年")
```

試試 2024（閏年）、1900（平年）、2000（閏年）。括號讓邏輯更清楚。

> [!WARNING]
> 新手常寫 `if day == "週六" or "週日":`——這是**錯的**！它會被理解為 `(day == "週六") or ("週日")`，而非空字串 `"週日"` 永遠算作 `True`，所以條件永遠成立。正確寫法是每一邊都寫完整：`day == "週六" or day == "週日"`，或者 `day in ("週六", "週日")`。

## 6.7 巢狀條件

`if` 裡面還可以再寫 `if`，每深一層多縮排 4 個空格：

```python
age = int(input())
member = input()      # 輸入 yes 或 no
if age < 12:
    print("兒童票：免費")
else:
    if member == "yes":
        print("會員票：50 元")
    else:
        print("普通票：80 元")
```

巢狀太深會難以閱讀。很多時候可以用 `elif` 或 `and` 讓結構變平：

```python
age = int(input())
member = input()
if age < 12:
    print("兒童票：免費")
elif member == "yes":
    print("會員票：50 元")
else:
    print("普通票：80 元")
```

## 6.8 「真值」：不只是 True 和 False

`if` 後面不一定非要寫比較，任何值都可以作為條件。Python 會把它當作 `bool(值)` 來判斷（第 2 課學過）：`0`、`0.0`、空字串 `""`、`None`、空列表 `[]` 被當作「假」，其他都當作「真」。

```python
name = input()
if name:                 # 等同於 if name != ""
    print(f"你好，{name}")
else:
    print("你什麼都沒輸入")
```

## 6.9 條件表示式：一行的 if

簡單的二選一，可以寫在一行裡：

```python
age = 20
status = "成年" if age >= 18 else "未成年"
print(status)
```

格式：`值1 if 條件 else 值2`——條件成立取值 1，否則取值 2。適合簡單情況，複雜的還是用普通 `if`。

## 6.10 match 語句：按值分情況

Python 3.10 引入了 `match` 語句，適合「根據一個值的不同取值做不同的事」：

```python
command = input()
match command:
    case "start":
        print("開始遊戲")
    case "pause" | "stop":          # | 表示「或」
        print("暫停")
    case "quit":
        print("再見！")
    case _:                          # _ 匹配其他所有情況
        print("未知命令")
```

`match` 的能力遠不止這些（它能拆解列表、字典等複雜資料，叫「結構化模式匹配」），以後會再見到它。對於簡單判斷，用 `if/elif` 完全可以。

## 6.11 常見錯誤

```python norun
if x = 5:
```

```text
SyntaxError: invalid syntax. Maybe you meant '==' or ':=' instead of '='?
```

比較用 `==`。Python 的報錯甚至猜出了你的意思。

```python norun
if x > 5
    print("大")
```

```text
SyntaxError: expected ':'
```

忘了冒號。

```python norun
if x > 5:
print("大")
```

```text
IndentationError: expected an indented block after 'if' statement on line 1
```

忘了縮排。

```python norun
if x > 5:
    print("大")
     print("很大")
```

```text
IndentationError: unexpected indent
```

同一個程式碼區塊縮排不一致（第二行多了一個空格）。

## 6.12 小結

- 比較運算子 `== != > < >= <=` 得到 `True`/`False`；`=` 是賦值，`==` 才是比較。
- `if 條件:` + **縮排的程式碼區塊**；`else:` 處理其他情況；`elif` 處理多種情況，**從上往下只走第一個成立的分支**。
- `and`、`or`、`not` 組合條件；`or` 兩邊都要寫完整的條件。
- 任何值都可以當條件：`0`、`""`、`None`、空容器為假。
- `值1 if 條件 else 值2` 是一行的二選一；`match/case` 按值分情況。

## 6.13 練習

<!-- exercise: odd-even -->
### 練習 1：奇數還是偶數

輸入一個整數，如果是偶數輸出 `even`，否則輸出 `odd`。

```text
樣例輸入：7
樣例輸出：odd
```

<details>
<summary>提示</summary>

`n % 2 == 0` 表示偶數。
</details>

<!-- exercise: grade -->
### 練習 2：成績等級

輸入一個 0～100 的整數成績，按下表輸出等級：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。

```text
樣例輸入：85
樣例輸出：B
```

<details>
<summary>提示</summary>

用 `if / elif / else`，從高分往低分判斷。
</details>

<!-- exercise: leap -->
### 練習 3：閏年

輸入一個年份，是閏年輸出 `leap`，否則輸出 `common`。規則：能被 4 整除且不能被 100 整除，或者能被 400 整除。

```text
樣例輸入：1900
樣例輸出：common
```

<details>
<summary>提示</summary>

`(y % 4 == 0 and y % 100 != 0) or y % 400 == 0`
</details>

<!-- exercise: max3 -->
### 練習 4：三個數中的最大值

輸入一行三個整數（空格隔開），輸出其中最大的一個。**請用 `if` 來做**，不要用 `max()`（做完以後可以再用 `max()` 驗證）。

```text
樣例輸入：3 9 4
樣例輸出：9
```

<details>
<summary>提示 1</summary>

先假設第一個數最大：`biggest = a`；然後依次和 `b`、`c` 比較，誰更大就更新 `biggest`。
</details>

<details>
<summary>提示 2</summary>

```python
a, b, c = map(int, input().split())
biggest = a
if b > biggest:
    biggest = b
# 再比較 c……
```
</details>

<!-- exercise: predict-if -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`x > 10` 成立，輸出 `A`，`elif` 被跳過；這是一個新的 `if`：15 同時能被 3 和 5 整除，輸出 `C`；最後 `x < 10` 不成立，輸出 `F`。
</details>

<!-- exercise: or-trap -->
### 練習 6：or 的陷阱

`day = "Monday"` 時，`if day == "Saturday" or "Sunday":` 的條件結果是什麼？

- `False`，因為 day 不是週末
- `True`，因為 `"Sunday"` 本身被當作真值
- 報錯
- 取決於電腦的語言設定

<details>
<summary>答案與解析</summary>

`True`。這個條件被理解為 `(day == "Saturday") or "Sunday"`，非空字串永遠為真。正確寫法是 `day == "Saturday" or day == "Sunday"`。
</details>

## 6.14 延伸閱讀

- [Python 官方教程 · 4.1 if 語句](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#if-statements)
- [Python 官方教程 · 4.7 match 語句](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#match-statements)
- 下一課：[迴圈](../07-loops/zh-CN.md)
