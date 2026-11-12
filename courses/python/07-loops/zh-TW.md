# 7. 迴圈

> 🎯 **本課目標**：用 `while` 和 `for` 讓程式碼重複執行；熟練使用 `range()`；掌握「累加」「計數」「找最大值」等迴圈套路；用 `break`、`continue` 控制迴圈；寫出巢狀迴圈（如九九乘法表）；避開死迴圈和「差一」錯誤。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：第 6 課

## 7.1 為什麼需要迴圈？

假設要列印 1 到 5：

```python
print(1)
print(2)
print(3)
print(4)
print(5)
```

那 1 到 1000 呢？複製 1000 行顯然不現實。**迴圈**（loop）讓同一段程式碼**重複執行**很多次。電腦最擅長的，就是不知疲倦地重複。

## 7.2 while 迴圈：條件成立就一直做

```python
i = 1
while i <= 5:
    print(i)
    i += 1
print("結束")
```

執行過程：

```text
i = 1 → 1 <= 5 成立 → 列印 1，i 變成 2
i = 2 → 2 <= 5 成立 → 列印 2，i 變成 3
  ……
i = 5 → 5 <= 5 成立 → 列印 5，i 變成 6
i = 6 → 6 <= 5 不成立 → 離開迴圈，列印「結束」
```

`while` 的格式和 `if` 很像：`while 條件:` 加一個縮排的程式碼區塊。區別是：`if` 只執行一次，`while` 會**一直重複，直到條件不成立**。

### 當心死迴圈

```python norun
i = 1
while i <= 5:
    print(i)
    # 忘了寫 i += 1
```

`i` 永遠是 1，條件永遠成立，程式會**永遠**列印 1，停不下來——這叫**死迴圈**。在學習程式裡點「■ 停止」，在終端裡按 <kbd>Ctrl</kbd>+<kbd>C</kbd>。

> [!TIP]
> 寫 `while` 迴圈時，問自己三個問題：① 迴圈變數從多少開始？② 什麼條件下繼續？③ 每次迴圈裡，迴圈變數**有沒有朝著結束的方向變化**？

### while 適合「不知道要迴圈多少次」的情況

比如：一直讓使用者輸入，直到輸入正確為止：

```python input="abc\npython"
password = input()
while password != "python":
    print("密碼錯誤，請重試")
    password = input()
print("登入成功！")
```

## 7.3 for 迴圈：逐個取出

`for` 迴圈用來**依次處理一組東西中的每一個**：

```python
for ch in "你好呀":
    print(ch)
```

輸出三行：`你`、`好`、`呀`。每次迴圈，變數 `ch` 依次得到字串中的下一個字元。

### range()：生成一串數字

`for` 最常和 `range()` 搭配，用來「迴圈 n 次」或「遍歷一段數字」：

```python
for i in range(5):
    print(i)
```

輸出 `0 1 2 3 4`（每個一行）。注意：**`range(5)` 是 0 到 4，不包括 5**——和切片一樣「包含開始，不包含結束」。

`range` 的三種寫法：

| 寫法 | 生成的數字 |
|------|------------|
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(1, 6)` | 1, 2, 3, 4, 5 |
| `range(0, 10, 2)` | 0, 2, 4, 6, 8（步長 2） |
| `range(10, 0, -1)` | 10, 9, 8, …, 1（倒數） |

```python
for i in range(1, 6):
    print(f"第 {i} 次")
```

> [!NOTE]
> `for i in range(1, 6)` 和 7.2 節的 `while` 例子做的事完全一樣，但更短、更不容易出錯（不會忘記 `i += 1`）。**知道迴圈次數時，優先用 `for`。**

## 7.4 迴圈的經典套路

### 套路 1：累加

計算 1 + 2 + … + 100：

```python
total = 0                  # ① 迴圈前準備一個「裝結果的變數」，初始為 0
for i in range(1, 101):
    total += i             # ② 每次把當前的數加進去
print(total)               # ③ 迴圈結束後使用結果：5050
```

### 套路 2：累乘

計算 5 的階乘 5! = 1 × 2 × 3 × 4 × 5：

```python
result = 1                 # 乘法的初始值是 1，不是 0！
for i in range(1, 6):
    result *= i
print(result)              # 120
```

### 套路 3：計數

數一數句子裡有多少個字母 `o`：

```python
text = "hello world, good morning"
count = 0
for ch in text:
    if ch == "o":
        count += 1
print(count)               # 5
```

### 套路 4：找最大值

讀入 5 個數，找出最大的：

```python
biggest = int(input())      # 先把第一個數當作當前最大
for _ in range(4):          # 再讀剩下 4 個
    n = int(input())
    if n > biggest:
        biggest = n
print(biggest)
```

當迴圈變數用不到時，習慣上把它命名為 `_`。

### 套路 5：讀入 n 個數

很多題目先給出數量 n，再給出 n 個數：

```python
n = int(input())
total = 0
for _ in range(n):
    total += int(input())
print(total)
```

## 7.5 break 和 continue

- **`break`**：立刻**跳出整個迴圈**。
- **`continue`**：跳過本次迴圈剩下的程式碼，**直接開始下一次**。

```python
for i in range(1, 11):
    if i == 6:
        break           # 到 6 就停
    print(i, end=" ")
print()
# 輸出：1 2 3 4 5

for i in range(1, 11):
    if i % 2 == 0:
        continue        # 跳過偶數
    print(i, end=" ")
print()
# 輸出：1 3 5 7 9
```

### while True + break：「先做，滿足條件再退出」

```python input="hello\nquit"
while True:
    cmd = input()
    if cmd == "quit":
        break
    print(f"你輸入了：{cmd}")
print("程式結束")
```

`while True` 表示「一直迴圈」，靠裡面的 `break` 來結束。這是互動式程式最常用的結構。

### 找第一個滿足條件的數

```python
n = 100
while True:
    n += 1
    if n % 7 == 0:
        break
print("大於 100 的第一個 7 的倍數：", n)    # 105
```

## 7.6 巢狀迴圈

迴圈裡面套迴圈：外層每走一次，內層要完整地走一遍。

```python
for row in range(1, 4):
    for col in range(1, 5):
        print(f"({row},{col})", end=" ")
    print()          # 每行結束後換行
```

```text
(1,1) (1,2) (1,3) (1,4)
(2,1) (2,2) (2,3) (2,4)
(3,1) (3,2) (3,3) (3,4)
```

### 九九乘法表

```python
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f"{j}×{i}={i * j}", end="\t")
    print()
```

內層迴圈的範圍 `range(1, i + 1)` 依賴外層的 `i`：第 1 行 1 個，第 2 行 2 個……所以是個三角形。`\t` 是製表符，讓結果對齊。

### 列印圖形

```python
n = 4
for i in range(1, n + 1):
    print("*" * i)
```

```text
*
**
***
****
```

用字串乘法 `"*" * i`，一行就能畫出 i 個星號。

## 7.7 for…else（瞭解即可）

迴圈後面可以接 `else`：**迴圈正常結束（沒有被 `break` 打斷）時**執行。常用於「搜尋」：

```python
n = 29
for d in range(2, n):
    if n % d == 0:
        print(n, "不是質數，能被", d, "整除")
        break
else:
    print(n, "是質數")
```

## 7.8 常見錯誤

### 差一錯誤（off-by-one）

想列印 1 到 10，卻寫成：

```python
for i in range(1, 10):
    print(i)      # 只列印到 9！
```

`range` 不包括結束值，應該寫 `range(1, 11)`。這類「差一個」的錯誤在所有語言裡都很常見，叫 **off-by-one 錯誤**。檢查迴圈時，**特別留意第一次和最後一次**。

### 累加變數放錯位置

```python
for i in range(1, 4):
    total = 0          # ❌ 每次迴圈都被清零
    total += i
print(total)           # 3，而不是 6
```

初始化要放在迴圈**前面**。

### 死迴圈

`while` 條件裡的變數在迴圈中從不改變，或者改變的方向錯了（比如該 `i += 1` 卻寫成 `i -= 1`）。發現程式「卡住」不出結果時，首先懷疑它。學習程式檢查答案時，超過時間限制會提示「執行超時」。

## 7.9 小結

- `while 條件:` 條件成立就一直重複；要確保迴圈變數會讓條件最終變假，否則死迴圈。
- `for 變數 in 一組東西:` 逐個處理；`range(開始, 結束, 步長)` 不包含結束值。
- 知道次數用 `for`，不知道次數（等到某件事發生）用 `while`。
- 套路：累加（初值 0）、累乘（初值 1）、計數、找最大值、讀入 n 個數。
- `break` 跳出迴圈，`continue` 跳到下一次；`while True` + `break` 是互動程式的常用結構。
- 巢狀迴圈：外層一次，內層一整輪；小心 off-by-one。

## 7.10 練習

<!-- exercise: sum-to-n -->
### 練習 1：1 到 n 的和

輸入正整數 n，輸出 1 + 2 + … + n 的結果。

```text
樣例輸入：100
樣例輸出：5050
```

<details>
<summary>提示</summary>

`total = 0`，然後 `for i in range(1, n + 1): total += i`。
</details>

<!-- exercise: factorial -->
### 練習 2：階乘

輸入非負整數 n（0 ≤ n ≤ 20），輸出 n 的階乘 n!。規定 0! = 1。

```text
樣例輸入：5
樣例輸出：120
```

<details>
<summary>提示</summary>

`result = 1`，迴圈乘上 1 到 n。當 n = 0 時迴圈一次都不執行，結果正好是 1。
</details>

<!-- exercise: fizzbuzz -->
### 練習 3：FizzBuzz

輸入正整數 n，依次輸出 1 到 n，每個數一行。但是：3 的倍數輸出 `Fizz`，5 的倍數輸出 `Buzz`，既是 3 又是 5 的倍數輸出 `FizzBuzz`。

```text
樣例輸入：5
樣例輸出：
1
2
Fizz
4
Buzz
```

<details>
<summary>提示</summary>

先判斷「既是 3 又是 5 的倍數」（也就是 15 的倍數），再判斷 3 和 5，順序很重要。
</details>

<!-- exercise: count-digits -->
### 練習 4：數位數

輸入一個正整數，**用 while 迴圈**（不許用 `len`）計算它有幾位數。

```text
樣例輸入：12345
樣例輸出：5
```

<details>
<summary>提示</summary>

每次 `n //= 10` 去掉最後一位，同時計數加 1，直到 `n` 變成 0。
</details>

<!-- exercise: triangle -->
### 練習 5：數字三角形

輸入正整數 n，輸出 n 行，第 i 行是從 1 到 i 的數字，用空格隔開。

```text
樣例輸入：4
樣例輸出：
1
1 2
1 2 3
1 2 3 4
```

<details>
<summary>提示 1</summary>

外層迴圈 `for i in range(1, n + 1)`，內層迴圈列印 1 到 i。
</details>

<details>
<summary>提示 2</summary>

為了讓行末沒有多餘空格，可以把每行的數字轉成字串，再用 `" ".join(...)`；或者內層用 `print(j, end=" ")`——判題時會忽略行尾空格，所以也可以。
</details>

<!-- exercise: predict-loop -->
### 練習 6：讀程式碼，寫輸出

```python
total = 0
for i in range(10, 0, -3):
    total += i
    print(i, end=" ")
print()
print(total)
```

<details>
<summary>答案與解析</summary>

`range(10, 0, -3)` 生成 10、7、4、1（下一個 -2 小於結束值 0，停止）。和是 10 + 7 + 4 + 1 = 22。第一行末尾有一個空格，判題時不影響。
</details>

<!-- exercise: range-len -->
### 練習 7：range 有幾個數

`range(3, 15, 3)` 一共會生成幾個數字？

- 3 個
- 4 個
- 5 個
- 12 個

<details>
<summary>答案與解析</summary>

3、6、9、12，共 4 個。15 是結束值，不包含在內。
</details>

## 7.11 延伸閱讀

- [Python 官方教程 · 4.2 for 語句、4.3 range() 函式](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#for-statements)
- [Python Tutor](https://pythontutor.com/)：把程式碼一步步視覺化執行，非常適合觀察迴圈裡變數是怎麼變化的。
- 下一課：[列表與元組](../08-lists-tuples/zh-CN.md)
