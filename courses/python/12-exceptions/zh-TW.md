# 12. 異常處理

> 🎯 **本課目標**：分清「語法錯誤」和執行時的「異常」；用 `try / except` 接住異常，讓程式在使用者輸錯時不會崩潰；會用 `else`、`finally` 和 `as e`；在自己的函式里用 `raise` 報告錯誤；認識常見的異常型別和它們的層級；知道什麼時候該用異常、什麼時候該用 `if`；不寫「什麼都接」的 `except:`。
>
> ⏱ **預計用時**：60 分鐘　｜　📋 **前置知識**：入門課第 5 課（讀懂報錯），本課程第 10、11 課

## 12.1 錯誤分兩種

入門課第 5 課裡，我們學會了讀懂報錯資訊。Python 的錯誤大致分兩種：

1. **語法錯誤**（`SyntaxError`、`IndentationError`）：程式碼寫得不符合 Python 的語法，**程式根本不會開始執行**。只能改程式碼。
2. **異常**（exception）：語法沒問題，但**執行到某一行時出了狀況**——除以 0、把 `"abc"` 轉成整數、開啟不存在的檔案……

```python norun
age = int(input("你幾歲？"))     # 使用者輸入了「十二」
print(age + 1)
```

```text
Traceback (most recent call last):
  File "main.py", line 1, in <module>
    age = int(input("你幾歲？"))
ValueError: invalid literal for int() with base 10: '十二'
```

這裡的 `ValueError` 就是一種異常。預設情況下，異常會讓程式**立刻停止**並列印出 Traceback。但很多異常是可以預料的（使用者總會輸錯），我們希望程式能**自己處理**，而不是崩潰。

## 12.2 `try / except`：接住異常

```python input="十二"
text = input("你幾歲？")
try:
    age = int(text)                      # 可能出錯的程式碼放在 try 裡
    print("明年你就", age + 1, "歲了")
except ValueError:                       # 如果出現 ValueError，就執行這裡
    print(f"「{text}」不是一個整數，請用阿拉伯數字輸入，比如 12")
print("程式繼續執行")
```

執行過程：

1. 先執行 `try` 裡的程式碼；
2. 如果**沒有**出錯，跳過 `except`；
3. 如果出了 `ValueError`，`try` 裡剩下的程式碼**不再執行**，直接跳到 `except ValueError:` 裡；
4. 無論哪種情況，之後的程式碼都會繼續執行。

一個常見的用法：**一直問，直到輸入正確為止**：

```python input="abc\n-5\n12"
while True:
    try:
        age = int(input("請輸入年齡："))
        if age < 0:
            print("年齡不能是負數")
            continue
        break                                # 輸入正確，跳出迴圈
    except ValueError:
        print("請輸入整數")
print("你的年齡是", age)
```

## 12.3 接住不同的異常

一段程式碼可能出現好幾種異常，可以分別處理：

```python
def divide(a_text, b_text):
    try:
        a, b = float(a_text), float(b_text)
        return a / b
    except ValueError:
        return "請輸入數字"
    except ZeroDivisionError:
        return "除數不能是 0"

print(divide("10", "4"))
print(divide("10", "abc"))
print(divide("10", "0"))
```

幾種異常想用同樣的方式處理，可以寫在一個元組裡：`except (ValueError, ZeroDivisionError):`。

### 拿到異常的詳細資訊：`as e`

```python
try:
    nums = [1, 2, 3]
    print(nums[10])
except IndexError as e:
    print("出錯了：", e)               # e 是異常物件，列印出來就是錯誤說明
    print("異常型別：", type(e).__name__)
```

## 12.4 `else` 和 `finally`

完整的結構是這樣的：

```python
def read_number(text):
    try:
        n = int(text)
    except ValueError:
        print(f"  「{text}」：不是整數")
    else:                               # try 裡沒有出錯時才執行
        print(f"  「{text}」：讀到了 {n}")
    finally:                            # 無論有沒有出錯，最後都一定會執行
        print("  —— 處理完畢")

read_number("42")
read_number("4.2")
```

- `else`：只在**沒有異常**時執行。把「出錯就不該做的事」放在這裡，而不是塞進 `try`，可以讓 `try` 只包住真正可能出錯的那一兩行。
- `finally`：**一定會執行**，即使 `try` 裡有 `return`、即使出了沒被接住的異常。常用來做「善後」工作，比如關閉檔案、斷開網路連線。（檔案用 `with` 就會自動關閉，所以用得比較少。）

## 12.5 自己丟擲異常：`raise`

寫函式時，如果別人傳進來的引數不合理，最好的辦法往往不是 `print` 一句話、返回一個奇怪的值，而是**丟擲異常**，讓呼叫者知道出了問題：

```python
def set_age(age: int) -> int:
    if not 0 <= age <= 150:
        raise ValueError(f"年齡必須在 0 到 150 之間，收到的是 {age}")
    return age

for a in (12, -3, 200):
    try:
        print("設定成功：", set_age(a))
    except ValueError as e:
        print("設定失敗：", e)
```

`raise 異常型別("說明")` 會立刻停止當前函式，把異常交給呼叫它的地方；如果一直沒有人接住，程式就會停下來並列印 Traceback。

### 自定義異常

可以定義自己的異常型別，讓錯誤的含義更清楚（`class` 下一課會詳細講，這裡照著寫就行）：

```python
class InsufficientFunds(Exception):     # 繼承自 Exception
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFunds(f"餘額 {balance} 元，不夠取 {amount} 元")
    return balance - amount

try:
    print(withdraw(100, 30))
    print(withdraw(100, 300))
except InsufficientFunds as e:
    print("取款失敗：", e)
```

## 12.6 常見的異常型別

| 異常 | 什麼時候出現 | 例子 |
|------|--------------|------|
| `ValueError` | 型別對，但值不合適 | `int("abc")` |
| `TypeError` | 型別不對 | `"1" + 1` |
| `ZeroDivisionError` | 除以 0 | `5 / 0` |
| `IndexError` | 下標超出範圍 | `[1, 2][5]` |
| `KeyError` | 字典裡沒有這個鍵 | `{}["a"]` |
| `AttributeError` | 物件沒有這個屬性或方法 | `"abc".push(1)` |
| `NameError` | 用了沒定義的名字 | `print(x)`（x 沒定義） |
| `FileNotFoundError` | 檔案不存在 | `open("沒有.txt")` |
| `KeyboardInterrupt` | 使用者按了 Ctrl+C | 正在執行的程式被中斷 |

這些異常組成了一棵「家族樹」：幾乎所有異常都是 `Exception` 的「後代」，`FileNotFoundError` 又是 `OSError` 的後代……`except OSError:` 會同時接住 `FileNotFoundError`、`PermissionError` 等。

## 12.7 不要寫「什麼都接」的 `except`

```python norun
try:
    result = calculate(data)
except:                     # 危險！什麼異常都接住，包括拼錯變數名、按 Ctrl+C
    print("出錯了")
```

這樣寫，**真正的 bug 也會被悄悄吞掉**：你把變數名拼錯了（`NameError`），程式卻只說「出錯了」，根本不知道哪裡錯；使用者想按 Ctrl+C 結束程式，也被它攔住了。

原則：

1. **只接住你預料到、並且知道怎麼處理的異常**，比如 `except ValueError:`；
2. 實在需要兜底（比如伺服器不能因為一個請求出錯就整個退出），至少寫 `except Exception as e:`，並把錯誤**記錄下來**，不要默默忽略；
3. 永遠不要寫 `except: pass`。

## 12.8 用異常還是用 `if`？

兩種風格都很常見：

```python
d = {"a": 1}

# 先檢查再做（Look Before You Leap）
if "b" in d:
    print(d["b"])
else:
    print("沒有 b")

# 先做，出錯了再處理（Easier to Ask Forgiveness than Permission）
try:
    print(d["b"])
except KeyError:
    print("沒有 b")
```

經驗法則：**能輕鬆提前檢查的、經常發生的情況，用 `if`**；**難以提前檢查的（比如檔案會不會在開啟前一刻被刪掉、網路會不會斷）或者很少發生的，用 `try`**。Python 社群更偏愛後一種風格，但兩種都不算錯。

## 12.9 `assert`：檢查「絕不應該發生」的情況

`assert 條件, "說明"`：如果條件不成立，就丟擲 `AssertionError`。它用來檢查**程式自己的假設**，發現 bug：

```python
def average(nums):
    assert len(nums) > 0, "average() 需要至少一個數"
    return sum(nums) / len(nums)

print(average([1, 2, 3]))
try:
    average([])
except AssertionError as e:
    print("AssertionError:", e)
```

> [!WARNING]
> `assert` 用來抓 bug，**不要**用它檢查使用者輸入：Python 用 `-O` 引數執行時會跳過所有 `assert`。檢查使用者輸入請用 `if` + `raise ValueError(...)`。`assert` 最常用的地方是**測試**（第 15 課的 pytest）。

## 12.10 小結

- 語法錯誤讓程式無法開始執行；**異常**是執行中出的狀況，不處理就會讓程式停止。
- `try:` 放可能出錯的程式碼，`except 型別:` 處理；`as e` 拿到異常物件；`else` 在沒出錯時執行；`finally` 一定執行。
- 用 `raise ValueError("說明")` 在自己的函式里報告錯誤；可以繼承 `Exception` 定義自己的異常。
- 常見異常：`ValueError`、`TypeError`、`ZeroDivisionError`、`IndexError`、`KeyError`、`FileNotFoundError`……
- 只接住預料到的異常；不要寫裸的 `except:`，更不要 `except: pass`。
- 容易檢查的用 `if`，難以檢查的用 `try`；`assert` 檢查程式自身的假設，不用來檢查使用者輸入。

## 12.11 練習

<!-- exercise: safe-int -->
### 練習 1：安全地讀整數

第一行是整數 `n`，接下來 `n` 行，每行是使用者輸入的一個東西。如果是整數，輸出它的 2 倍；否則輸出 `invalid`。用 `try / except` 實現。

```text
樣例輸入：
3
21
abc
-4
樣例輸出：
42
invalid
-8
```

<details>
<summary>提示</summary>

在迴圈裡寫 `try: print(int(text) * 2)`，`except ValueError: print("invalid")`。
</details>

<!-- exercise: safe-divide -->
### 練習 2：安全的除法

第一行是整數 `n`，接下來 `n` 行，每行兩個東西 `a b`。輸出 `a / b`（保留 2 位小數）；如果 `b` 是 0，輸出 `division by zero`；如果有一個不是數字，輸出 `invalid`。

```text
樣例輸入：
3
10 4
1 0
x 2
樣例輸出：
2.50
division by zero
invalid
```

<details>
<summary>提示</summary>

用兩個 `except`：`except ZeroDivisionError:` 和 `except ValueError:`。`float("x")` 會丟擲 `ValueError`。
</details>

<!-- exercise: raise-age -->
### 練習 3：用 raise 報告錯誤

補全函式 `check_age(age)`：如果 `age` 小於 0 或大於 150，丟擲 `ValueError("age must be between 0 and 150")`；否則返回 `age`。主程式已經寫好：讀入若干個整數，合法的輸出 `ok`，不合法的輸出異常的說明。

```text
樣例輸入：12 -1 151 150
樣例輸出：
ok
age must be between 0 and 150
age must be between 0 and 150
ok
```

<details>
<summary>提示</summary>

`if age < 0 or age > 150: raise ValueError("age must be between 0 and 150")`。說明文字要和題目完全一致。
</details>

<!-- exercise: finally-order -->
### 練習 4：讀程式碼，寫輸出

```python
def f(x):
    try:
        print("start")
        return 10 / x
    except ZeroDivisionError:
        print("oops")
        return None
    finally:
        print("done")

print(f(2))
print(f(0))
```

<details>
<summary>答案與解析</summary>

`f(2)`：列印 `start`，準備返回 5.0，但返回之前先執行 `finally` 列印 `done`，然後外面列印 `5.0`。`f(0)`：列印 `start`，除以 0 跳到 `except` 列印 `oops`，再執行 `finally` 列印 `done`，最後外面列印 `None`。
</details>

<!-- exercise: bare-except -->
### 練習 5：為什麼不要寫 `except:`？

- 因為它執行得很慢
- 因為它會接住所有異常，包括拼錯名字造成的 bug 和 Ctrl+C，讓問題被悄悄隱藏
- 因為 Python 不允許這樣寫
- 因為它只能接住 `ValueError`

<details>
<summary>答案與解析</summary>

裸的 `except:` 什麼都接，真正的錯誤（比如 `NameError`）會被當成「普通情況」處理掉，很難發現；連使用者按 Ctrl+C 產生的 `KeyboardInterrupt` 也會被攔住。應該只接住預料到的具體異常。
</details>

<!-- exercise: which-exception -->
### 練習 6：哪種異常？

`int("3.14")` 會丟擲哪種異常？

- `TypeError`
- `ValueError`
- `SyntaxError`
- 不會出錯，結果是 3

<details>
<summary>答案與解析</summary>

`ValueError`：引數是字串（型別沒問題），但 `"3.14"` 不是整數的寫法。想得到 3，可以寫 `int(float("3.14"))`。
</details>

## 12.12 延伸閱讀

- [Python 官方教程 · 8. 錯誤和異常](https://docs.python.org/zh-cn/3/tutorial/errors.html)
- [內建異常的層級結構](https://docs.python.org/zh-cn/3/library/exceptions.html#exception-hierarchy)
- 下一課：[類與物件](../13-classes/zh-CN.md)
