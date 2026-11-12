# 10. 函式

> 🎯 **本課目標**：用 `def` 定義自己的函式，理解**引數**和**返回值**；分清 `return` 和 `print`；會用預設引數、關鍵字引數和多個返回值；理解**區域性變數**和**全域性變數**；寫出帶**型別標註**和文件字串的現代 Python 函式；知道「函式也是值」，會用 `lambda`；初步認識**遞迴**；避開「可變預設引數」的陷阱。
>
> ⏱ **預計用時**：75 分鐘　｜　📋 **前置知識**：第 6～9 課

## 10.1 為什麼需要函式？

你已經用過很多函式了：`print()`、`len()`、`input()`、`sorted()`……它們把一段做好的功能**打包起來、起個名字**，用的時候只要寫名字。

自己寫函式有三個好處：

1. **不重複**：同樣的程式碼寫一次，到處呼叫；要改也只改一處。
2. **好讀**：`if is_prime(n):` 比一大段迴圈更容易看懂。
3. **好測試**：每個函式單獨檢查對不對，出了問題容易定位。

## 10.2 定義和呼叫

```python
def greet(name):                  # def 函式名(引數):
    print(f"你好，{name}！")       # 函式體：縮排 4 個空格

greet("小明")                      # 呼叫：函式名(實參)
greet("小紅")
```

- `def` 是 **define**（定義）的縮寫；
- `name` 是**引數**（parameter）：呼叫時傳進來的值會放進這個變數；
- 函式體要**縮排**，和 `if`、`for` 一樣；
- 定義函式時**不會執行**函式體，只有**呼叫**時才執行。所以必須先定義、後呼叫。

## 10.3 返回值：`return`

函式可以把結果「交回」給呼叫它的地方：

```python
def area(width, height):
    return width * height          # 把結果返回，函式到這裡就結束了

a = area(3, 4)                     # 返回值可以存進變數
print(a)                           # 12
print(area(2, 5) + area(1, 1))     # 也可以直接參與計算：11
```

### `return` 和 `print` 的區別

這是初學者最容易混淆的地方：

```python
def add_print(a, b):
    print(a + b)        # 只是顯示在螢幕上

def add_return(a, b):
    return a + b        # 把結果交回去

x = add_print(1, 2)     # 螢幕上顯示 3，但 x 得到的是 None
y = add_return(1, 2)    # 螢幕上什麼也不顯示，y 得到 3
print("x =", x)
print("y =", y)
```

`print` 是給**人**看的；`return` 是給**程式**用的。一個沒有 `return`（或者只寫了 `return`）的函式，返回值是 `None`。

## 10.4 引數的各種寫法

```python
def power(base, exp=2):                 # exp 有預設值：不傳就用 2
    return base ** exp

print(power(5))                          # 25：用預設值
print(power(5, 3))                       # 125：按位置傳參
print(power(exp=3, base=2))              # 8：按名字傳參（關鍵字引數），順序可以隨意

def describe(name, age, city="北京"):
    return f"{name}，{age} 歲，住在{city}"

print(describe("小明", 12))
print(describe("小紅", 13, city="上海"))
```

規則：**有預設值的引數必須放在沒有預設值的引數後面**。呼叫時，按位置傳的引數要寫在按名字傳的前面。

## 10.5 返回多個值

用逗號隔開多個值，返回的其實是一個**元組**，可以直接解包（第 8 課）：

```python
def min_max(nums):
    return min(nums), max(nums)

lo, hi = min_max([3, 9, 1, 7])
print(lo, hi)            # 1 9

def divide(a, b):
    return a // b, a % b         # 商和餘數

q, r = divide(17, 5)
print(f"17 = 5 × {q} + {r}")
```

## 10.6 區域性變數和全域性變數

在函式**裡面**建立的變數是**區域性變數**，只在這個函式里有效，函式結束就消失了：

```python
def f():
    secret = 42           # 區域性變數
    return secret

print(f())
# print(secret)           # 這裡會報 NameError：外面看不到函式里的變數
```

在函式**外面**建立的變數是**全域性變數**，函式里可以**讀取**它。但如果在函式里**給它賦值**，Python 會認為你建立了一個同名的區域性變數：

```python
count = 0                 # 全域性變數

def show():
    print("count =", count)    # 只讀取：沒問題

show()
```

```python norun
count = 0

def increase():
    count = count + 1     # 想修改全域性變數……
    return count

increase()
```

```text
UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
```

可以用 `global count` 宣告要修改全域性變數，但**更好的做法是：透過引數傳進去，透過 `return` 傳出來**：

```python
def increase(count):
    return count + 1

count = 0
count = increase(count)
count = increase(count)
print(count)              # 2
```

這樣函式只依賴它的引數，更容易理解和測試。

## 10.7 現代寫法：型別標註和文件字串

現代 Python 程式碼通常會寫上**型別標註**（type hints）和**文件字串**（docstring）：

```python
def bmi(weight_kg: float, height_m: float) -> float:
    """計算身體品質指數 BMI = 體重 / 身高²。"""
    return weight_kg / height_m ** 2

print(f"{bmi(60, 1.70):.1f}")
print(bmi.__doc__)        # 文件字串儲存在 __doc__ 裡；在互動環境中用 help(bmi) 檢視更完整的說明
```

- `weight_kg: float` 表示「這個引數應該是小數」，`-> float` 表示「返回一個小數」；
- 三引號裡的字串是文件字串，說明函式做什麼。

> [!NOTE]
> 型別標註**不會**被 Python 強制檢查，傳錯型別照樣能執行。它是寫給人和工具看的：編輯器能據此給出提示，`mypy`、`pyright` 等工具能提前發現型別錯誤（第 15 課會用到）。

## 10.8 函式也是值

函式和數字、字串一樣，可以存進變數、當作引數傳給別的函式：

```python
def double(x):
    return x * 2

f = double                     # 注意沒有括號：這是函式本身，不是呼叫它
print(f(21))                   # 42

def apply_twice(func, value):
    return func(func(value))

print(apply_twice(double, 3))  # 12
```

只用一次的小函式，可以用 **`lambda`** 寫成一行：`lambda 引數: 表示式`。它最常見的用途是給 `sorted`、`max`、`min` 指定比較的依據（第 9 課已經見過）：

```python
words = ["banana", "kiwi", "apple", "fig"]
print(sorted(words, key=len))                    # 按長度排序
print(sorted(words, key=lambda w: w[-1]))        # 按最後一個字母排序

students = [("小明", 88), ("小紅", 95), ("小剛", 72)]
print(max(students, key=lambda s: s[1]))         # 分數最高的人
```

## 10.9 遞迴：函式呼叫自己

一個函式可以**呼叫它自己**，這叫**遞迴**（recursion）。比如階乘 `n! = n × (n−1)!`，並且 `0! = 1`：

```python
def factorial(n: int) -> int:
    if n == 0:                    # 終止條件：必須有，否則會無限呼叫下去
        return 1
    return n * factorial(n - 1)   # 把問題變成一個更小的同類問題

print(factorial(5))               # 5 × 4 × 3 × 2 × 1 = 120
```

呼叫過程是這樣展開的：

```text
factorial(3)
= 3 * factorial(2)
= 3 * (2 * factorial(1))
= 3 * (2 * (1 * factorial(0)))
= 3 * (2 * (1 * 1))
= 6
```

遞迴寫起來很優雅，適合處理「大問題包含同樣的小問題」的情況（比如資料夾裡的資料夾）。但如果忘了終止條件，或者遞迴太深（Python 預設大約 1000 層），就會出現 `RecursionError`。很多遞迴也可以改寫成迴圈。

## 10.10 陷阱：可變的預設引數

```python
def add_item(item, box=[]):        # 危險！預設的列表只在定義函式時建立一次
    box.append(item)
    return box

print(add_item("蘋果"))            # ['蘋果']
print(add_item("香蕉"))            # ['蘋果', '香蕉']  ← 上一次的「蘋果」還在！
```

預設值只在**定義函式時**計算一次，之後每次呼叫都共用同一個列表。正確的寫法是用 `None` 做預設值：

```python
def add_item(item, box=None):
    if box is None:
        box = []                   # 每次呼叫都新建一個列表
    box.append(item)
    return box

print(add_item("蘋果"))
print(add_item("香蕉"))
```

**預設值不要用列表、字典、集合這類可變的東西。**

## 10.11 常見錯誤

```python norun
def area(w, h):
    return w * h

print(area(3))
```

```text
TypeError: area() missing 1 required positional argument: 'h'
```

引數個數不對：定義時有 2 個引數，呼叫時只給了 1 個。

```python norun
print(square(4))

def square(x):
    return x * x
```

```text
NameError: name 'square' is not defined
```

呼叫發生在定義之前。把函式定義移到呼叫的上面。

```python norun
def square(x):
    x * x              # 忘了 return

print(square(4) + 1)
```

```text
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
```

函式沒有 `return`，返回了 `None`，`None + 1` 就報錯了。

## 10.12 小結

- `def 名字(引數):` 定義函式，縮排的部分是函式體；先定義，後呼叫。
- `return` 把結果交回給呼叫者並結束函式；沒有 `return` 就返回 `None`。`print` 只是顯示。
- 預設引數 `def f(x, y=2)`；關鍵字引數 `f(y=3, x=1)`；返回多個值得到元組，可以解包。
- 函式里的變數是區域性的；想修改外面的值，最好用引數傳進、`return` 傳出，而不是 `global`。
- 型別標註 `def f(x: int) -> str:` 和文件字串讓程式碼更清楚。
- 函式是值，可以當引數傳遞；`lambda` 寫一行的小函式，常用於 `sorted(key=...)`。
- 遞迴 = 函式呼叫自己，必須有終止條件。
- 預設引數不要用可變物件，用 `None` 代替。

## 10.13 練習

<!-- exercise: max3 -->
### 練習 1：三個數的最大值

補全函式 `max3(a, b, c)`，返回三個數中最大的一個（**不要**用內建的 `max`，用 `if` 來比較）。程式會讀入一行三個整數並輸出 `max3` 的結果。

```text
樣例輸入：3 9 4
樣例輸出：9
```

<details>
<summary>提示</summary>

先假設 `biggest = a`，再分別和 `b`、`c` 比較，誰更大就更新 `biggest`，最後 `return biggest`。
</details>

<!-- exercise: is-prime -->
### 練習 2：質數判斷

補全函式 `is_prime(n)`：`n` 是質數就返回 `True`，否則返回 `False`。（質數是大於 1、只能被 1 和自己整除的整數。）程式讀入一個整數，是質數輸出 `yes`，否則輸出 `no`。

```text
樣例輸入：97
樣例輸出：yes
```

<details>
<summary>提示</summary>

小於 2 直接返回 `False`；然後用 `for d in range(2, int(n ** 0.5) + 1):` 檢查，能整除就返回 `False`；迴圈結束都沒整除，返回 `True`。
</details>

<!-- exercise: grade -->
### 練習 3：分數等級

補全函式 `grade(score)`，把分數換成等級：90 及以上 `A`，80～89 `B`，70～79 `C`，60～69 `D`，60 以下 `F`。輸入一行若干個分數，輸出對應的等級，空格隔開。

```text
樣例輸入：95 82 60 59
樣例輸出：A B D F
```

<details>
<summary>提示</summary>

在函式里用 `if / elif / else`，從高到低判斷；主程式裡 `print(" ".join(grade(s) for s in scores))`。
</details>

<!-- exercise: factorial -->
### 練習 4：遞迴求階乘

用**遞迴**補全函式 `factorial(n)`。輸入一個整數 `n`（0 ≤ n ≤ 20），輸出 `n!`。

```text
樣例輸入：5
樣例輸出：120
```

<details>
<summary>提示</summary>

終止條件 `if n == 0: return 1`，否則 `return n * factorial(n - 1)`。
</details>

<!-- exercise: scope-predict -->
### 練習 5：讀程式碼，寫輸出

```python
x = 10

def f(x):
    x = x + 1
    return x

y = f(x)
print(x, y)

def g():
    return

print(g())
```

<details>
<summary>答案與解析</summary>

函式 `f` 裡的 `x` 是引數（區域性變數），修改它不影響外面的 `x`，所以輸出 `10 11`；`g` 只寫了 `return`，返回 `None`。
</details>

<!-- exercise: mutable-default -->
### 練習 6：可變預設引數

下面的程式輸出什麼？

```python
def add(item, box=[]):
    box.append(item)
    return box

add(1)
print(add(2))
```

- `[2]`
- `[1, 2]`
- 報錯
- `None`

<details>
<summary>答案與解析</summary>

`[1, 2]`。預設的列表只建立一次，第一次呼叫放進去的 1 還在。應該用 `box=None`，在函式里再新建列表。
</details>

## 10.14 延伸閱讀

- [Python 官方教程 · 4.8 定義函式](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#defining-functions)
- [typing 模組：型別標註](https://docs.python.org/zh-cn/3/library/typing.html)
- 下一課：[模組與檔案](../11-modules-files/zh-CN.md)
