# 14. 進階特性

> 🎯 **本課目標**：掌握讓 Python 程式碼更簡潔、更「地道」（Pythonic）的一批工具：`enumerate`、`zip`、`any`、`all` 等內建函式；生成器表示式和 **`yield` 生成器**（省記憶體的「懶」計算）；`*args`、`**kwargs` 和星號解包；`match` 結構化模式匹配；**裝飾器**和 `@cache`；海象運算子 `:=`；現代型別標註寫法。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：第 8～13 課

這一課的內容不是「必須會才能寫程式」的，但會大量出現在別人的程式碼、開源專案和麵試題裡。學會它們，你的程式碼會更短、更清楚，也能讀懂更多別人寫的 Python。

## 14.1 好用的內建函式

```python
names = ["小明", "小紅", "小剛"]
scores = [88, 95, 72]

for i, name in enumerate(names, start=1):        # 同時拿到序號和元素（第 8 課見過）
    print(i, name)

for name, score in zip(names, scores):           # 把幾個列表「拉鍊式」地配對
    print(f"{name}：{score}")

print(dict(zip(names, scores)))                  # 兩個列表直接變成字典

nums = [3, -1, 4, 0, 5]
print(any(n < 0 for n in nums))                  # 有沒有任何一個負數？True
print(all(n >= 0 for n in nums))                 # 是不是全都非負？False
print(sum(n for n in nums if n > 0))             # 正數之和：12
print(max(names, key=len), min(scores))          # 最長的名字、最低的分數
print(list(reversed(nums)), sorted(nums, reverse=True))
```

| 函式 | 作用 |
|------|------|
| `enumerate(seq, start=0)` | 得到 (序號, 元素) |
| `zip(a, b, ...)` | 把多個序列一一配對，按最短的結束 |
| `any(...)` / `all(...)` | 有任意一個為真 / 全部為真 |
| `sum`、`min`、`max` | 求和、最小、最大（`min`/`max` 可以用 `key=`） |
| `sorted`、`reversed` | 排序後的新列表、倒序遍歷 |
| `map(f, seq)`、`filter(f, seq)` | 對每個元素用 f、篩選（通常用推導式代替） |

## 14.2 生成器表示式：不一次性造出整個列表

把列表推導式的方括號換成圓括號，就得到**生成器表示式**：它不會馬上算出所有元素，而是**用到一個才算一個**：

```python
import sys

squares_list = [n * n for n in range(1_000_000)]     # 立刻生成 100 萬個數，全部存起來
squares_gen = (n * n for n in range(1_000_000))      # 只是一個「隨時可以算」的東西

print(sys.getsizeof(squares_list), "位元組")           # 好幾 MB
print(sys.getsizeof(squares_gen), "位元組")            # 一兩百位元組
print(sum(squares_gen))                               # 邊算邊加，從不需要存 100 萬個數
```

直接作為函式引數時，可以省掉一層括號：`sum(n * n for n in range(10))`。上一節的 `any(n < 0 for n in nums)` 也是生成器表示式。

> [!NOTE]
> 生成器只能**遍歷一次**。遍歷完再用，就什麼都沒有了。需要多次使用時，用列表。

## 14.3 `yield`：自己寫生成器

函式里只要出現 `yield`，它就變成**生成器函式**：呼叫時不會馬上執行，而是返回一個生成器；每次向它要下一個值，函式就執行到下一個 `yield`，交出一個值，然後**暫停**在那裡，等著下次繼續。

```python
def countdown(n):
    while n > 0:
        yield n           # 交出 n，然後在這裡暫停
        n -= 1
    print("發射！")

for x in countdown(3):    # for 迴圈會自動不斷地向生成器要下一個值
    print(x)
```

生成器很適合「一個接一個地產生資料」的場景：逐行讀大檔案、無窮數列、分批處理資料……

```python
def fibonacci():                     # 無窮的斐波那契數列：不會把記憶體撐爆，因為每次只算一個
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

for i, f in enumerate(fibonacci()):
    if i >= 10:
        break
    print(f, end=" ")
print()

def read_numbers(lines):             # 一條「流水線」：逐個清洗資料
    for line in lines:
        line = line.strip()
        if line and not line.startswith("#"):
            yield int(line)

data = ["10", "# 註釋", "", "20", " 30 "]
print(sum(read_numbers(data)))       # 60
```

## 14.4 星號：`*args`、`**kwargs` 和解包

```python
def average(*nums):                  # *nums 把任意多個位置引數收整合一個元組
    return sum(nums) / len(nums)

print(average(1, 2, 3), average(10, 20))

def profile(name, **info):           # **info 把任意多個關鍵字引數收整合一個字典
    print(name, info)

profile("小明", age=12, city="北京")

scores = [90, 85, 77]
print(average(*scores))              # 呼叫時 * 把列表「拆開」成多個引數
options = {"sep": " | ", "end": "!\n"}
print("a", "b", "c", **options)      # ** 把字典拆成關鍵字引數

first, *rest = [1, 2, 3, 4]          # 解包時 * 收集「剩下的」
print(first, rest)
*init, last = "abcd"
print(init, last)
merged = [*scores, 100, *[1, 2]]     # 合併列表
both = {**{"a": 1}, **{"b": 2}}      # 合併字典（也可以寫 d1 | d2）
print(merged, both)
```

## 14.5 `match`：結構化模式匹配

Python 3.10 加入了 `match` 語句。它不只是「多個 `if`」的簡寫，還能**按資料的結構**拆解和匹配：

```python
def handle(command: str) -> str:
    match command.split():
        case ["quit"]:
            return "再見"
        case ["say", *words]:                 # 第一個詞是 say，剩下的都收進 words
            return " ".join(words)
        case ["move", x, y]:                  # 恰好三個詞
            return f"移動到 ({int(x)}, {int(y)})"
        case ["move", *_]:
            return "move 需要兩個座標"
        case []:
            return "（空命令）"
        case _:                               # _ 匹配一切：相當於 else
            return f"不認識的命令：{command}"

for cmd in ["say 你好 世界", "move 3 4", "move 1", "", "quit", "jump"]:
    print(f"{cmd!r:12} → {handle(cmd)}")
```

`case` 還可以匹配常量、字典的結構、類的屬性，並且用 `if` 加條件（`case [x, y] if x == y:`）。處理命令、解析 JSON 資料時，`match` 能讓程式碼清楚很多。

## 14.6 裝飾器：給函式「加功能」

第 13 課見過的 `@dataclass`、`@property` 都是**裝飾器**（decorator）：寫在函式或類定義上方的 `@名字`，它會把下面的函式「包裝」一層，加上額外的功能。

函式是值（第 10 課），所以可以寫一個「接收函式、返回新函式」的函式：

```python
import time
from functools import wraps

def timer(func):                             # 接收一個函式
    @wraps(func)                             # 保留原函式的名字和文件
    def wrapper(*args, **kwargs):            # 新函式：接收任意引數
        start = time.perf_counter()
        result = func(*args, **kwargs)       # 呼叫原來的函式
        print(f"{func.__name__} 用了 {time.perf_counter() - start:.4f} 秒")
        return result
    return wrapper                           # 返回新函式

@timer                                       # 相當於 slow_sum = timer(slow_sum)
def slow_sum(n):
    return sum(range(n))

print(slow_sum(1_000_000))
```

`@timer` 寫在 `def` 上面，等價於定義完之後執行 `slow_sum = timer(slow_sum)`。之後呼叫 `slow_sum` 實際上呼叫的是 `wrapper`，它在原函式前後加上了計時。

### 實用的內建裝飾器：`@cache`

上一課的遞迴斐波那契，`n` 稍大就會慢得驚人，因為同樣的值被重複算了無數次。`functools.cache` 會**記住**每次呼叫的結果，下次同樣的引數直接返回：

```python
from functools import cache

@cache
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))      # 瞬間算完；不加 @cache 的話，算到宇宙毀滅也算不完
```

這種「記住算過的結果」的技巧叫**記憶化**（memoization），是動態規劃的基礎。

## 14.7 海象運算子 `:=`

`:=` 可以在**表示式中間**給變數賦值（長得像海象的眼睛和牙齒，所以叫海象運算子）：

```python
import re

text = "訂單號 A1234，金額 56 元"
if (m := re.search(r"\d+", text)) is not None:     # 先賦值給 m，再判斷
    print("找到第一個數字：", m.group())

data = [3, 8, 1, 9, 4]
big = [y for x in data if (y := x * 10) > 40]       # 推導式裡也能用
print(big)
```

它能省掉一些重複計算和臨時變數，但別用得太多，以免難讀。

## 14.8 現代型別標註

Python 3.9 起，可以直接用內建型別寫容器的型別；3.10 起，可以用 `|` 表示「或」：

```python
def word_lengths(words: list[str]) -> dict[str, int]:
    return {w: len(w) for w in words}

def find(names: list[str], target: str) -> int | None:     # 找到返回下標，找不到返回 None
    return names.index(target) if target in names else None

print(word_lengths(["hi", "hello"]))
print(find(["a", "b"], "b"), find(["a", "b"], "z"))
```

| 寫法 | 含義 |
|------|------|
| `list[int]` | 整數列表 |
| `dict[str, float]` | 鍵是字串、值是小數的字典 |
| `tuple[int, str]` | 第一個是整數、第二個是字串的元組 |
| `int \| None` | 整數或 `None` |
| `Callable[[int], str]`（來自 `collections.abc`） | 接收一個整數、返回字串的函式 |

## 14.9 小結

- `enumerate`、`zip`、`any`、`all`、`sum/min/max(key=)`、`sorted`、`reversed` 讓迴圈更簡潔。
- 生成器表示式 `(x for x in ...)` 和 `yield` 生成器「用到才算」，省記憶體，可以表示無窮序列；只能遍歷一次。
- `*args` 收集位置引數、`**kwargs` 收集關鍵字引數；呼叫時 `*`、`**` 拆開列表和字典；`first, *rest = ...` 解包。
- `match / case` 按結構匹配資料，`_` 匹配一切。
- 裝飾器 `@deco` 等價於 `f = deco(f)`；`@cache` 記住函式結果。
- `:=` 在表示式中賦值；型別標註可以寫 `list[int]`、`dict[str, int]`、`int | None`。

## 14.10 練習

<!-- exercise: evens-gen -->
### 練習 1：寫一個生成器

補全生成器函式 `evens(n)`：依次產生 0 到 `n`（包括 `n`）之間的所有偶數。程式讀入 `n`，把生成的數用空格連起來輸出。

```text
樣例輸入：9
樣例輸出：0 2 4 6 8
```

<details>
<summary>提示</summary>

`for i in range(0, n + 1, 2): yield i`
</details>

<!-- exercise: star-average -->
### 練習 2：任意多個引數

補全函式 `stats(*nums)`，返回一個元組 `(最小值, 最大值, 平均值)`。程式讀入一行整數，用 `stats(*nums)` 呼叫它，輸出最小值、最大值和平均值（平均值保留 2 位小數）。

```text
樣例輸入：4 8 15 16 23 42
樣例輸出：4 42 18.00
```

<details>
<summary>提示</summary>

在函式里 `nums` 是一個元組：`return min(nums), max(nums), sum(nums) / len(nums)`。
</details>

<!-- exercise: match-commands -->
### 練習 3：用 match 處理命令

用 `match` 處理若干條命令，直到遇到 `quit`：

- `add a b`：輸出 `a + b`；
- `mul a b`：輸出 `a * b`；
- `neg a`：輸出 `-a`；
- 其他命令：輸出 `unknown`；
- `quit`：結束程式（不輸出任何東西）。

所有數都是整數。

```text
樣例輸入：
add 2 3
neg 7
pow 2 3
mul 4 5
quit
樣例輸出：
5
-7
unknown
20
```

<details>
<summary>提示</summary>

`match input().split():`，然後 `case ["add", a, b]: print(int(a) + int(b))`……最後 `case _: print("unknown")`。`quit` 用 `break` 跳出迴圈。
</details>

<!-- exercise: big-fib -->
### 練習 4：記憶化

輸入 `n`（0 ≤ n ≤ 300），輸出第 `n` 個斐波那契數（`fib(0) = 0`，`fib(1) = 1`）。要求用**遞迴 + `@cache`** 實現。

```text
樣例輸入：90
樣例輸出：2880067194370816120
```

<details>
<summary>提示</summary>

`from functools import cache`，在遞迴函式上加 `@cache`。`n` 到幾百時遞迴深度也只有幾百層，在 Python 預設的上限（約 1000 層）以內。
</details>

<!-- exercise: gen-predict -->
### 練習 5：讀程式碼，寫輸出

```python
def gen():
    print("start")
    yield 1
    print("middle")
    yield 2

g = gen()
print("created")
print(next(g))
print(next(g))
```

<details>
<summary>答案與解析</summary>

呼叫 `gen()` 時函式體**還沒有開始執行**，所以先列印 `created`。第一次 `next(g)` 執行到第一個 `yield`：列印 `start`，交出 1。第二次從暫停處繼續：列印 `middle`，交出 2。輸出依次是 `created`、`start`、`1`、`middle`、`2`。
</details>

<!-- exercise: decorator-meaning -->
### 練習 6：裝飾器等價於什麼？

下面的寫法等價於哪一句？

```python norun
@timer
def work():
    ...
```

- `work = timer`
- `work = timer(work)`
- `timer = work(timer)`
- `work()` 定義完立刻執行一次

<details>
<summary>答案與解析</summary>

`@timer` 等價於在定義之後寫 `work = timer(work)`：用 `timer` 返回的新函式替換原來的 `work`。
</details>

## 14.11 延伸閱讀

- [Python 官方教程 · 9.9 生成器](https://docs.python.org/zh-cn/3/tutorial/classes.html#generators)
- [PEP 636：結構化模式匹配教程](https://peps.python.org/pep-0636/)
- [functools 文件](https://docs.python.org/zh-cn/3/library/functools.html)、[itertools 文件](https://docs.python.org/zh-cn/3/library/itertools.html)
- 下一課：[包與現代工具鏈](../15-packages-tooling/zh-CN.md)
