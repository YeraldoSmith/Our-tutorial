# 11. 模組與檔案

> 🎯 **本課目標**：用 `import` 使用 Python 自帶的「工具箱」（標準庫）；把自己的程式碼拆成多個模組，理解 `if __name__ == "__main__":`；用 `with open(...)` 讀寫文字檔案，並且**永遠記得寫 `encoding="utf-8"`**；用 `pathlib` 處理路徑；讀寫 CSV 表格和 JSON 資料，讓程式關掉之後資料也不會丟。
>
> ⏱ **預計用時**：75 分鐘　｜　📋 **前置知識**：第 8～10 課

## 11.1 `import`：使用現成的工具

Python 安裝好之後就自帶了幾百個**模組**（module），合稱**標準庫**。一個模組就是一個 `.py` 檔案，裡面有寫好的函式和變數。用 `import` 把它「拿進來」：

```python
import math

print(math.sqrt(16))       # 平方根：4.0
print(math.pi)             # 圓周率
print(math.ceil(2.1))      # 向上取整：3
```

`import` 有幾種寫法：

```python
import math                       # 用的時候寫 math.sqrt
from math import sqrt, pi         # 直接拿出 sqrt 和 pi，用的時候不用寫 math.
import datetime as dt             # 起個短一點的別名

print(sqrt(2), pi)
print(dt.date.today().year >= 2024)
```

> [!TIP]
> 一般推薦 `import math` 這種寫法：看到 `math.sqrt` 就知道函式是從哪兒來的。**不要**寫 `from math import *`（把所有東西都拿進來），容易和自己的變數重名。

### 標準庫裡常用的模組

| 模組 | 用途 | 例子 |
|------|------|------|
| `math` | 數學函式 | `math.sqrt(2)`、`math.floor(3.7)` |
| `random` | 隨機數 | `random.randint(1, 6)`、`random.choice(lst)` |
| `datetime` | 日期和時間 | `datetime.date.today()` |
| `statistics` | 統計 | `statistics.mean(nums)`、`statistics.median(nums)` |
| `collections` | 更多資料結構 | `Counter`、`defaultdict`、`deque` |
| `pathlib` | 檔案路徑 | `Path("data.txt").exists()` |
| `json`、`csv` | 讀寫資料格式 | 見本課後半部分 |
| `time` | 計時、暫停 | `time.sleep(1)`、`time.perf_counter()` |
| `os`、`sys` | 作業系統和直譯器 | `sys.version`、`os.getcwd()` |

```python
import random
import statistics
from datetime import date, timedelta

random.seed(1)                                   # 固定隨機種子，每次執行結果一樣
dice = [random.randint(1, 6) for _ in range(10)]
print("擲 10 次骰子：", dice)
print("平均：", statistics.mean(dice), " 中位數：", statistics.median(dice))

d = date(2025, 1, 1)
print(d + timedelta(days=100))                   # 100 天之後是哪天
print(d.isoweekday())                            # 星期幾（1 = 星期一 … 7 = 星期日）
```

想知道一個模組有哪些功能，查 [Python 官方文件](https://docs.python.org/zh-cn/3/library/index.html)（有中文版），或者在互動環境裡輸入 `help(math)`。

## 11.2 自己寫模組

程式變長以後，可以把它拆成幾個檔案。假設同一個資料夾裡有兩個檔案：

```python norun
# 檔案 geometry.py
PI = 3.14159

def circle_area(r):
    return PI * r * r
```

```python norun
# 檔案 main.py
import geometry

print(geometry.circle_area(2))
print(geometry.PI)
```

執行 `main.py`，`import geometry` 會找到同一個資料夾裡的 `geometry.py`，把裡面的函式拿來用。**模組名就是檔名去掉 `.py`**，所以檔名裡不要有空格和中文，也**不要和標準庫重名**（比如不要把自己的檔案叫 `math.py`、`random.py`，否則 `import random` 會匯入你自己的檔案）。

### `if __name__ == "__main__":`

每個模組都有一個特殊變數 `__name__`：直接執行這個檔案時，它是 `"__main__"`；被別的檔案 `import` 時，它是模組名。

```python
def main():
    print("程式從這裡開始")

print("__name__ 是：", __name__)

if __name__ == "__main__":      # 只有直接執行這個檔案時才執行
    main()
```

把「直接執行時才做的事」放進 `if __name__ == "__main__":` 裡，這個檔案就**既能直接執行，又能被別的檔案匯入而不會亂執行**。這是 Python 程式的標準寫法。

> [!NOTE]
> 學習程式和網頁版一次只執行一個檔案，所以本課的多檔案例子標成了「不可執行」。要動手試，可以在電腦上新建一個資料夾，把兩個檔案放進去，在終端裡執行 `python main.py`（第一部分第 3 課講過終端）。

## 11.3 讀寫檔案

變數裡的資料，程式一結束就沒了。要**長久儲存**，就得寫進**檔案**。

### 寫檔案

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("第一行\n")              # write 不會自動換行，要自己加 \n
    f.write("第二行 Second line\n")
    print("第三行", file=f)          # print 也可以寫進檔案（自動換行）

print("寫好了")
```

- `open(檔名, 模式, encoding=...)` 開啟一個檔案，得到一個**檔案物件** `f`；
- 模式 `"w"`：寫入（**檔案已存在會被清空**）；`"a"`：追加到末尾；`"r"`：讀取（預設）；
- **`with ... as f:`**：縮排的程式碼區塊結束時，檔案會**自動關閉**，即使中間出錯也一樣。永遠用 `with`。

### 讀檔案

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("蘋果 5\n香蕉 3\n橙子 8\n")

with open("notes.txt", encoding="utf-8") as f:       # 預設模式是 "r"
    content = f.read()                                 # 一次讀出全部內容（一個字串）
print(repr(content))

with open("notes.txt", encoding="utf-8") as f:
    for line in f:                                     # 一行一行地讀（最常用，省記憶體）
        name, count = line.split()
        print(f"{name}：{count} 個")
```

一行一行讀的時候，每行末尾都帶著換行符 `\n`；`line.strip()` 可以去掉它（上面用 `split()` 時它會被自動去掉）。

### 一定要寫 `encoding="utf-8"`！

檔案裡存的是**位元組**，讀寫時要按某種**編碼**轉換成文字（分詞那一課講過 UTF-8）。如果不指定，Python 會用**作業系統的預設編碼**——在中文版 Windows 上常常是 GBK，在俄文版 Windows 上是 cp1251，而 macOS 和 Linux 上是 UTF-8。結果就是：同一個程式，在一臺電腦上好好的，換一臺電腦就亂碼或者報錯：

```text
UnicodeDecodeError: 'gbk' codec can't decode byte 0xa7 in position 2: illegal multibyte sequence
```

**養成習慣：每次 `open` 文字檔案都寫上 `encoding="utf-8"`。**（按照 PEP 686，Python 3.15 起預設使用 UTF-8；但很多電腦上裝的還是舊版本，所以仍然應該寫上。）

## 11.4 `pathlib`：處理路徑的現代方法

不同系統的路徑寫法不同：Windows 是 `C:\Users\xiaoming\data.txt`，macOS/Linux 是 `/home/xiaoming/data.txt`。用 `pathlib` 就不用操心這些：

```python
from pathlib import Path

folder = Path("my_data")
folder.mkdir(exist_ok=True)                      # 新建資料夾（已存在也不報錯）
file = folder / "hello.txt"                      # 用 / 拼接路徑，各個系統都正確
file.write_text("你好，pathlib！\n", encoding="utf-8")   # 一行寫檔案
print(file.read_text(encoding="utf-8"))          # 一行讀檔案
print(file.exists(), file.name, file.suffix)     # True hello.txt .txt
print(sorted(p.name for p in folder.glob("*.txt")))   # 找出資料夾裡所有 .txt 檔案
```

> [!NOTE]
> 像 `"notes.txt"` 這樣的**相對路徑**，是相對於「**當前工作目錄**」的，不一定是 `.py` 檔案所在的資料夾。程式找不到檔案時，先 `print(Path.cwd())` 看看當前目錄在哪裡。

## 11.5 CSV：表格資料

**CSV**（逗號分隔值）是最簡單的表格格式，Excel、WPS 都能開啟。每行一條記錄，各列用逗號隔開：

```python
import csv

rows = [["姓名", "語文", "數學"], ["小明", 90, 85], ["小紅", 95, 92]]
with open("scores.csv", "w", newline="", encoding="utf-8") as f:
    csv.writer(f).writerows(rows)

with open("scores.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):              # 每行變成一個字典，鍵是表頭
        total = int(row["語文"]) + int(row["數學"])
        print(row["姓名"], "總分", total)
```

- 開啟 CSV 檔案時要加 `newline=""`，避免在 Windows 上出現多餘的空行；
- 讀出來的都是**字串**，要計算的話記得用 `int()`、`float()` 轉換；
- 要讓 Excel 正確顯示中文，寫檔案時可以用 `encoding="utf-8-sig"`。

## 11.6 JSON：儲存結構化資料

上一課的「列表 + 字典」資料，可以原樣存成 **JSON** 檔案。JSON 是網路上最通用的資料格式，幾乎所有程式語言都能讀寫：

```python
import json

data = {"name": "小明", "age": 12, "hobbies": ["足球", "畫畫"], "member": True}

with open("profile.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)   # 寫入檔案

with open("profile.json", encoding="utf-8") as f:
    print(f.read())                                     # 看看檔案裡是什麼樣子

with open("profile.json", encoding="utf-8") as f:
    loaded = json.load(f)                               # 讀回來，又變成了字典
print(loaded["hobbies"][0], type(loaded))

text = json.dumps(data, ensure_ascii=False)             # dumps / loads：和字串互相轉換
print(text)
print(json.loads(text) == data)
```

- `ensure_ascii=False`：讓中文、俄文等**原樣儲存**，否則會變成 `\u5c0f\u660e` 這樣的轉義符；
- `indent=2`：縮排排版，方便人閱讀；
- Python 的 `True`/`False`/`None` 在 JSON 裡寫作 `true`/`false`/`null`。

## 11.7 常見錯誤

```python norun
with open("不存在的檔案.txt", encoding="utf-8") as f:
    print(f.read())
```

```text
FileNotFoundError: [Errno 2] No such file or directory: '不存在的檔案.txt'
```

檔名拼錯了，或者檔案不在當前工作目錄。用 `Path("...").exists()` 先檢查，或用 `print(Path.cwd())` 看看當前目錄。

```python norun
import random

def random():          # 自己的函式和模組重名了
    return 4

print(random.randint(1, 6))
```

```text
AttributeError: 'function' object has no attribute 'randint'
```

`random` 這個名字被自己的函式覆蓋了。自己的變數、函式、檔案都不要和模組重名。

```python norun
with open("data.txt", "w") as f:
    f.write(123)
```

```text
TypeError: write() argument must be str, not int
```

`write` 只接受字串，要寫 `f.write(str(123))`（順便也別忘了 `encoding="utf-8"`）。

## 11.8 小結

- `import 模組`、`from 模組 import 名字`、`import 模組 as 別名`；標準庫有 `math`、`random`、`datetime`、`statistics`、`pathlib`、`json`、`csv` 等。
- 自己的 `.py` 檔案就是模組；不要和標準庫重名；用 `if __name__ == "__main__":` 區分「直接執行」和「被匯入」。
- 讀寫檔案：`with open(名字, 模式, encoding="utf-8") as f:`；`"r"` 讀、`"w"` 覆蓋寫、`"a"` 追加；`for line in f` 逐行讀。
- **總是寫 `encoding="utf-8"`**，否則在不同系統上可能亂碼。
- `pathlib.Path` 跨平臺處理路徑：`/` 拼接、`read_text`、`write_text`、`exists`、`glob`。
- CSV 用 `csv` 模組（加 `newline=""`）；JSON 用 `json.dump/load`（加 `ensure_ascii=False`）。

## 11.9 練習

<!-- exercise: stats -->
### 練習 1：用標準庫做統計

輸入一行若干個數，用 `statistics` 模組輸出它們的**平均數**和**中位數**，各保留 2 位小數，空格隔開。

```text
樣例輸入：3 1 4 1 5 9 2 6
樣例輸出：3.88 3.50
```

<details>
<summary>提示</summary>

`import statistics`，然後 `statistics.mean(nums)` 和 `statistics.median(nums)`。
</details>

<!-- exercise: weekday -->
### 練習 2：星期幾、第幾天

輸入一個 `YYYY-MM-DD` 格式的日期，輸出它是星期幾（1 表示星期一，7 表示星期日）和它是這一年的第幾天，空格隔開。

```text
樣例輸入：2025-03-01
樣例輸出：6 60
```

<details>
<summary>提示</summary>

`from datetime import date`；`d = date.fromisoformat(text)`；`d.isoweekday()` 是星期幾，`d.timetuple().tm_yday` 是一年中的第幾天。
</details>

<!-- exercise: to-json -->
### 練習 3：變成 JSON

第一行輸入名字，第二行輸入年齡（整數）。把它們放進字典 `{"name": 名字, "age": 年齡}`，用 `json.dumps` 輸出（中文要原樣顯示）。

```text
樣例輸入：
小明
12
樣例輸出：{"name": "小明", "age": 12}
```

<details>
<summary>提示</summary>

`json.dumps(data, ensure_ascii=False)`。年齡要先 `int()`，這樣輸出的是 `12` 而不是 `"12"`。
</details>

<!-- exercise: file-roundtrip -->
### 練習 4：寫進檔案，再讀出來

第一行是整數 `n`，接下來 `n` 行文字。把這 `n` 行寫進檔案 `lines.txt`（記得 `encoding="utf-8"`），然後**重新開啟檔案讀出來**，輸出檔案裡有幾行，以及最長的一行（有幾行一樣長時輸出最先出現的）。

```text
樣例輸入：
3
蘋果
西瓜和葡萄
香蕉
樣例輸出：
3
西瓜和葡萄
```

<details>
<summary>提示</summary>

寫的時候每行加 `"\n"`；讀的時候 `lines = [line.rstrip("\n") for line in f]`，最長的一行是 `max(lines, key=len)`。
</details>

<!-- exercise: why-encoding -->
### 練習 5：為什麼要寫 encoding？

開啟文字檔案時總是寫 `encoding="utf-8"`，主要是為了？

- 讓檔案變小
- 讓程式在不同的作業系統（比如中文版 Windows）上讀寫中文、俄文等文字時不亂碼、不報錯
- 讓檔案讀得更快
- 不寫就無法開啟檔案

<details>
<summary>答案與解析</summary>

不寫的話，Python 使用系統預設編碼，中文版 Windows 上常常是 GBK。用 UTF-8 寫的檔案拿到那裡按 GBK 讀，就會亂碼或報 `UnicodeDecodeError`。
</details>

<!-- exercise: why-with -->
### 練習 6：`with` 的作用

`with open(...) as f:` 比直接 `f = open(...)` 好在哪裡？

- 能開啟更大的檔案
- 程式碼區塊結束時自動關閉檔案，即使中間出錯也會關閉
- 會自動加上 `encoding="utf-8"`
- 會自動備份檔案

<details>
<summary>答案與解析</summary>

`with` 保證檔案一定會被關閉。不關閉的檔案，寫入的內容可能還沒真正儲存到磁碟，也會佔用系統資源。它不會替你選擇編碼。
</details>

## 11.10 延伸閱讀

- [Python 標準庫文件](https://docs.python.org/zh-cn/3/library/index.html)（中文）
- [Python 官方教程 · 7.2 讀寫檔案](https://docs.python.org/zh-cn/3/tutorial/inputoutput.html#reading-and-writing-files)
- [pathlib 文件](https://docs.python.org/zh-cn/3/library/pathlib.html)、[json 文件](https://docs.python.org/zh-cn/3/library/json.html)
- 下一課：[異常處理](../12-exceptions/zh-CN.md)
