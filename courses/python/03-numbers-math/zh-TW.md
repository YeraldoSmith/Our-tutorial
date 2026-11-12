# 3. 數字與計算

> 🎯 **本課目標**：掌握 Python 的全部算術運算子（`+ - * / // % **`）和運算優先順序；用 `%` 和 `//` 解決「換算」類問題；會用 `round`、`abs`、`max`、`min` 和 `math` 模組；用 f 字串漂亮地顯示數字。
>
> ⏱ **預計用時**：40 分鐘　｜　📋 **前置知識**：第 2 課

## 3.1 算術運算子

| 運算子 | 含義 | 例子 | 結果 |
|--------|------|------|------|
| `+` | 加 | `7 + 2` | `9` |
| `-` | 減 | `7 - 2` | `5` |
| `*` | 乘 | `7 * 2` | `14` |
| `/` | 除 | `7 / 2` | `3.5` |
| `//` | 整除（向下取整） | `7 // 2` | `3` |
| `%` | 取餘數（模） | `7 % 2` | `1` |
| `**` | 乘方 | `7 ** 2` | `49` |

```python
print(7 + 2, 7 - 2, 7 * 2)
print(7 / 2, 7 // 2, 7 % 2)
print(2 ** 10)
```

### `/` 的結果總是小數

```python
print(6 / 2)     # 3.0，不是 3
```

即使能整除，`/` 也會得到 `float`。如果需要整數結果，用 `//`。

### 整數和小數混合運算

只要有一個數是 `float`，結果就是 `float`：

```python
print(3 + 2.0)    # 5.0
print(2 * 1.5)    # 3.0
```

## 3.2 整除 `//` 和取餘 `%`

小學除法：7 ÷ 2 = 3 …… 1。在 Python 裡，**商**用 `//`，**餘數**用 `%`：

```text
7 // 2 = 3    （商）
7 %  2 = 1    （餘數）
驗證：3 × 2 + 1 = 7
```

這兩個運算子非常有用：

```python
n = 17
print(n % 2)        # 1 → 奇數（偶數除以 2 餘 0）
print(n % 10)       # 7 → 個位數
print(n // 10)      # 1 → 去掉個位數後的部分
```

### 典型應用：時間換算

把 3725 秒換算成「幾小時幾分幾秒」：

```python
total = 3725
hours = total // 3600            # 一小時 3600 秒
minutes = total % 3600 // 60     # 剩下的秒數再換成分鐘
seconds = total % 60
print(hours, "小時", minutes, "分", seconds, "秒")
```

輸出：`1 小時 2 分 5 秒`。

`divmod(a, b)` 可以一次得到商和餘數：

```python
q, r = divmod(3725, 3600)
print(q, r)      # 1 125
```

> [!NOTE]
> 負數的 `//` 是**向下取整**（往更小的方向）：`-7 // 2` 是 `-4`，不是 `-3`；相應地 `-7 % 2` 是 `1`。日常使用中一般遇不到，知道有這回事即可。其他語言（C、Java）對負數的處理和 Python 不同，學到時會提醒。

## 3.3 運算優先順序

和數學一樣，**先乘除後加減**，**括號最優先**：

```python
print(2 + 3 * 4)      # 14，先算 3 * 4
print((2 + 3) * 4)    # 20，先算括號
```

完整的優先順序（從高到低）：

```text
1. ( )                括號
2. **                 乘方
3. -x                 負號
4. *  /  //  %        乘、除、整除、取餘
5. +  -               加、減
```

同一級別從左往右算：`10 - 3 - 2` 是 `(10 - 3) - 2 = 5`。唯一的例外是 `**` 從右往左：`2 ** 3 ** 2` 是 `2 ** 9 = 512`。

> [!TIP]
> 拿不準的時候**加括號**。多餘的括號不會出錯，還能讓程式碼更清楚：`average = (a + b + c) / 3`。

一個容易出錯的地方：

```python
print(-2 ** 2)     # -4：先算 2 ** 2，再取負
print((-2) ** 2)   # 4
```

## 3.4 常用的數學函式

不需要匯入就能用的**內建函式**：

```python
print(abs(-5))             # 5：絕對值
print(max(3, 8, 1))        # 8：最大值
print(min(3, 8, 1))        # 1：最小值
print(round(3.14159, 2))   # 3.14：保留 2 位小數
print(round(7.6))          # 8：四捨五入到整數
print(pow(2, 5))           # 32：等同於 2 ** 5
```

### `round` 的一個小坑

```python
print(round(2.5))   # 2
print(round(3.5))   # 4
```

Python 的 `round` 在正好一半（.5）時，會取**最近的偶數**（叫「銀行家舍入」），這樣大量資料統計時誤差更小。如果你需要傳統的「四捨五入」，後面學 `decimal` 模組時可以精確控制。

## 3.5 math 模組

更多數學功能放在 **`math` 模組**裡。**模組**就是一個裝滿工具的「工具箱」，用 `import` 把它拿進來：

```python
import math

print(math.sqrt(16))       # 4.0：平方根
print(math.pi)             # 3.141592653589793
print(math.floor(3.7))     # 3：向下取整
print(math.ceil(3.2))      # 4：向上取整
print(math.gcd(12, 18))    # 6：最大公約數
print(math.factorial(5))   # 120：5 的階乘 5×4×3×2×1
print(math.hypot(3, 4))    # 5.0：直角三角形斜邊
```

`import math` 一般寫在檔案的最上面。使用時要寫 `math.` 加函式名。

例：計算圓的面積和周長：

```python
import math

r = 5
area = math.pi * r ** 2
circumference = 2 * math.pi * r
print("面積：", area)
print("周長：", circumference)
```

## 3.6 把數字顯示得漂亮一點

上面的結果小數位太長了。用 f 字串的**格式說明**來控制：

```python
import math

r = 5
area = math.pi * r ** 2
print(f"面積：{area:.2f}")           # 保留 2 位小數：78.54
print(f"人口：{1400000000:,}")       # 千位分隔符：1,400,000,000
print(f"佔比：{0.256:.1%}")          # 百分比：25.6%
print(f"編號：{7:03d}")              # 補零到 3 位：007
print(f"[{42:>6}]")                  # 右對齊，寬度 6：[    42]
print(f"[{42:<6}]")                  # 左對齊：[42    ]
```

| 格式 | 含義 | 例子 |
|------|------|------|
| `:.2f` | 保留 2 位小數 | `3.14159` → `3.14` |
| `:,` | 千位分隔 | `1234567` → `1,234,567` |
| `:.1%` | 百分比，1 位小數 | `0.256` → `25.6%` |
| `:03d` | 整數，寬度 3，前面補 0 | `7` → `007` |
| `:>6` / `:<6` / `:^6` | 寬度 6，右/左/居中對齊 | |

> [!TIP]
> `:.2f` 只是**顯示**時保留兩位，變數本身的值沒有改變。要真正改變值，用 `round()`。

## 3.7 一個完整的例子：BMI 計算器

```python
weight = 68      # 體重，千克
height = 1.75    # 身高，米

bmi = weight / height ** 2
print(f"你的 BMI 是 {bmi:.1f}")
```

輸出：`你的 BMI 是 22.2`。注意 `height ** 2` 會先算（乘方優先順序高於除法）。

## 3.8 常見錯誤

```python norun
print(10 / 0)
```

```text
ZeroDivisionError: division by zero
```

除數不能為 0（`//` 和 `%` 也一樣）。以後學了 `if`，可以先判斷除數是不是 0。

```python norun
print(math.sqrt(9))
```

```text
NameError: name 'math' is not defined
```

忘了寫 `import math`。

```python norun
x = 5
print(2x)
```

```text
SyntaxError: invalid decimal literal
```

數學裡 `2x` 表示 2 乘 x，但在 Python 裡乘號**必須寫出來**：`2 * x`。

## 3.9 小結

- 運算子：`+ - * /`，`//` 整除，`%` 取餘，`**` 乘方；`/` 的結果總是 `float`。
- `//` 和 `%` 常用於判斷奇偶、取各位數字、時間和單位換算；`divmod` 同時得到商和餘數。
- 優先順序：括號 > `**` > 負號 > `* / // %` > `+ -`；拿不準就加括號。
- 內建函式 `abs`、`max`、`min`、`round`、`pow`；`round` 對 .5 取最近的偶數。
- `import math` 後使用 `math.sqrt`、`math.pi`、`math.floor`、`math.ceil` 等。
- f 字串格式：`:.2f`、`:,`、`:.1%`、`:03d`、`:>6`。

## 3.10 練習

<!-- exercise: time-convert -->
### 練習 1：秒數換算

變數 `total = 7384` 表示秒數。計算它是幾小時、幾分、幾秒，用**一個** `print` 輸出這三個數字（用空格隔開）：

```text
2 3 4
```

<details>
<summary>提示 1</summary>

小時：`total // 3600`；秒：`total % 60`。
</details>

<details>
<summary>提示 2</summary>

分鐘：先用 `total % 3600` 得到「不滿一小時的秒數」，再 `// 60`。
</details>

<!-- exercise: circle -->
### 練習 2：圓的面積

半徑 `r = 3`。使用 `math.pi` 計算圓的面積，保留兩位小數輸出：

```text
28.27
```

<details>
<summary>提示</summary>

`import math`，面積是 `math.pi * r ** 2`，用 `print(f"{area:.2f}")` 輸出。
</details>

<!-- exercise: digits -->
### 練習 3：拆分數字

`n = 472`。用 `//` 和 `%` 分別求出它的百位、十位和個位，輸出：

```text
4 7 2
```

<details>
<summary>提示</summary>

個位 `n % 10`；十位 `n // 10 % 10`；百位 `n // 100`。
</details>

<!-- exercise: predict-ops -->
### 練習 4：讀程式碼，寫輸出

```python
print(17 // 5)
print(17 % 5)
print(2 + 3 * 2 ** 2)
print(10 / 4)
```

<details>
<summary>答案與解析</summary>

`17 // 5 = 3`，`17 % 5 = 2`；`2 ** 2 = 4`，`3 * 4 = 12`，`2 + 12 = 14`；`10 / 4 = 2.5`。
</details>

<!-- exercise: precedence -->
### 練習 5：平均數

想計算 `a`、`b`、`c` 三個數的平均數，哪一種寫法是正確的？

- `a + b + c / 3`
- `(a + b + c) / 3`
- `a + b + (c / 3)`
- `(a + b) + c / 3`

<details>
<summary>答案與解析</summary>

除法優先順序比加法高，所以必須先用括號把三個數加起來。只有 `(a + b + c) / 3` 是對的。
</details>

## 3.11 延伸閱讀

- [Python 官方文件 · math 模組](https://docs.python.org/zh-cn/3/library/math.html)
- [格式規格迷你語言](https://docs.python.org/zh-cn/3/library/string.html#format-specification-mini-language)：f 字串冒號後面所有寫法的完整說明。
- 下一課：[字串](../04-strings/zh-CN.md)
