# 11. 模块与文件

> 🎯 **本课目标**：用 `import` 使用 Python 自带的「工具箱」（标准库）；把自己的代码拆成多个模块，理解 `if __name__ == "__main__":`；用 `with open(...)` 读写文本文件，并且**永远记得写 `encoding="utf-8"`**；用 `pathlib` 处理路径；读写 CSV 表格和 JSON 数据，让程序关掉之后数据也不会丢。
>
> ⏱ **预计用时**：75 分钟　｜　📋 **前置知识**：第 8～10 课

## 11.1 `import`：使用现成的工具

Python 安装好之后就自带了几百个**模块**（module），合称**标准库**。一个模块就是一个 `.py` 文件，里面有写好的函数和变量。用 `import` 把它「拿进来」：

```python
import math

print(math.sqrt(16))       # 平方根：4.0
print(math.pi)             # 圆周率
print(math.ceil(2.1))      # 向上取整：3
```

`import` 有几种写法：

```python
import math                       # 用的时候写 math.sqrt
from math import sqrt, pi         # 直接拿出 sqrt 和 pi，用的时候不用写 math.
import datetime as dt             # 起个短一点的别名

print(sqrt(2), pi)
print(dt.date.today().year >= 2024)
```

> [!TIP]
> 一般推荐 `import math` 这种写法：看到 `math.sqrt` 就知道函数是从哪儿来的。**不要**写 `from math import *`（把所有东西都拿进来），容易和自己的变量重名。

### 标准库里常用的模块

| 模块 | 用途 | 例子 |
|------|------|------|
| `math` | 数学函数 | `math.sqrt(2)`、`math.floor(3.7)` |
| `random` | 随机数 | `random.randint(1, 6)`、`random.choice(lst)` |
| `datetime` | 日期和时间 | `datetime.date.today()` |
| `statistics` | 统计 | `statistics.mean(nums)`、`statistics.median(nums)` |
| `collections` | 更多数据结构 | `Counter`、`defaultdict`、`deque` |
| `pathlib` | 文件路径 | `Path("data.txt").exists()` |
| `json`、`csv` | 读写数据格式 | 见本课后半部分 |
| `time` | 计时、暂停 | `time.sleep(1)`、`time.perf_counter()` |
| `os`、`sys` | 操作系统和解释器 | `sys.version`、`os.getcwd()` |

```python
import random
import statistics
from datetime import date, timedelta

random.seed(1)                                   # 固定随机种子，每次运行结果一样
dice = [random.randint(1, 6) for _ in range(10)]
print("掷 10 次骰子：", dice)
print("平均：", statistics.mean(dice), " 中位数：", statistics.median(dice))

d = date(2025, 1, 1)
print(d + timedelta(days=100))                   # 100 天之后是哪天
print(d.isoweekday())                            # 星期几（1 = 星期一 … 7 = 星期日）
```

想知道一个模块有哪些功能，查 [Python 官方文档](https://docs.python.org/zh-cn/3/library/index.html)（有中文版），或者在交互环境里输入 `help(math)`。

## 11.2 自己写模块

程序变长以后，可以把它拆成几个文件。假设同一个文件夹里有两个文件：

```python norun
# 文件 geometry.py
PI = 3.14159

def circle_area(r):
    return PI * r * r
```

```python norun
# 文件 main.py
import geometry

print(geometry.circle_area(2))
print(geometry.PI)
```

运行 `main.py`，`import geometry` 会找到同一个文件夹里的 `geometry.py`，把里面的函数拿来用。**模块名就是文件名去掉 `.py`**，所以文件名里不要有空格和中文，也**不要和标准库重名**（比如不要把自己的文件叫 `math.py`、`random.py`，否则 `import random` 会导入你自己的文件）。

### `if __name__ == "__main__":`

每个模块都有一个特殊变量 `__name__`：直接运行这个文件时，它是 `"__main__"`；被别的文件 `import` 时，它是模块名。

```python
def main():
    print("程序从这里开始")

print("__name__ 是：", __name__)

if __name__ == "__main__":      # 只有直接运行这个文件时才执行
    main()
```

把「直接运行时才做的事」放进 `if __name__ == "__main__":` 里，这个文件就**既能直接运行，又能被别的文件导入而不会乱执行**。这是 Python 程序的标准写法。

> [!NOTE]
> 学习程序和网页版一次只运行一个文件，所以本课的多文件例子标成了「不可运行」。要动手试，可以在电脑上新建一个文件夹，把两个文件放进去，在终端里运行 `python main.py`（第一部分第 3 课讲过终端）。

## 11.3 读写文件

变量里的数据，程序一结束就没了。要**长久保存**，就得写进**文件**。

### 写文件

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("第一行\n")              # write 不会自动换行，要自己加 \n
    f.write("第二行 Second line\n")
    print("第三行", file=f)          # print 也可以写进文件（自动换行）

print("写好了")
```

- `open(文件名, 模式, encoding=...)` 打开一个文件，得到一个**文件对象** `f`；
- 模式 `"w"`：写入（**文件已存在会被清空**）；`"a"`：追加到末尾；`"r"`：读取（默认）；
- **`with ... as f:`**：缩进的代码块结束时，文件会**自动关闭**，即使中间出错也一样。永远用 `with`。

### 读文件

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("苹果 5\n香蕉 3\n橙子 8\n")

with open("notes.txt", encoding="utf-8") as f:       # 默认模式是 "r"
    content = f.read()                                 # 一次读出全部内容（一个字符串）
print(repr(content))

with open("notes.txt", encoding="utf-8") as f:
    for line in f:                                     # 一行一行地读（最常用，省内存）
        name, count = line.split()
        print(f"{name}：{count} 个")
```

一行一行读的时候，每行末尾都带着换行符 `\n`；`line.strip()` 可以去掉它（上面用 `split()` 时它会被自动去掉）。

### 一定要写 `encoding="utf-8"`！

文件里存的是**字节**，读写时要按某种**编码**转换成文字（分词那一课讲过 UTF-8）。如果不指定，Python 会用**操作系统的默认编码**——在中文版 Windows 上常常是 GBK，在俄文版 Windows 上是 cp1251，而 macOS 和 Linux 上是 UTF-8。结果就是：同一个程序，在一台电脑上好好的，换一台电脑就乱码或者报错：

```text
UnicodeDecodeError: 'gbk' codec can't decode byte 0xa7 in position 2: illegal multibyte sequence
```

**养成习惯：每次 `open` 文本文件都写上 `encoding="utf-8"`。**（按照 PEP 686，Python 3.15 起默认使用 UTF-8；但很多电脑上装的还是旧版本，所以仍然应该写上。）

## 11.4 `pathlib`：处理路径的现代方法

不同系统的路径写法不同：Windows 是 `C:\Users\xiaoming\data.txt`，macOS/Linux 是 `/home/xiaoming/data.txt`。用 `pathlib` 就不用操心这些：

```python
from pathlib import Path

folder = Path("my_data")
folder.mkdir(exist_ok=True)                      # 新建文件夹（已存在也不报错）
file = folder / "hello.txt"                      # 用 / 拼接路径，各个系统都正确
file.write_text("你好，pathlib！\n", encoding="utf-8")   # 一行写文件
print(file.read_text(encoding="utf-8"))          # 一行读文件
print(file.exists(), file.name, file.suffix)     # True hello.txt .txt
print(sorted(p.name for p in folder.glob("*.txt")))   # 找出文件夹里所有 .txt 文件
```

> [!NOTE]
> 像 `"notes.txt"` 这样的**相对路径**，是相对于「**当前工作目录**」的，不一定是 `.py` 文件所在的文件夹。程序找不到文件时，先 `print(Path.cwd())` 看看当前目录在哪里。

## 11.5 CSV：表格数据

**CSV**（逗号分隔值）是最简单的表格格式，Excel、WPS 都能打开。每行一条记录，各列用逗号隔开：

```python
import csv

rows = [["姓名", "语文", "数学"], ["小明", 90, 85], ["小红", 95, 92]]
with open("scores.csv", "w", newline="", encoding="utf-8") as f:
    csv.writer(f).writerows(rows)

with open("scores.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):              # 每行变成一个字典，键是表头
        total = int(row["语文"]) + int(row["数学"])
        print(row["姓名"], "总分", total)
```

- 打开 CSV 文件时要加 `newline=""`，避免在 Windows 上出现多余的空行；
- 读出来的都是**字符串**，要计算的话记得用 `int()`、`float()` 转换；
- 要让 Excel 正确显示中文，写文件时可以用 `encoding="utf-8-sig"`。

## 11.6 JSON：保存结构化数据

上一课的「列表 + 字典」数据，可以原样存成 **JSON** 文件。JSON 是网络上最通用的数据格式，几乎所有编程语言都能读写：

```python
import json

data = {"name": "小明", "age": 12, "hobbies": ["足球", "画画"], "member": True}

with open("profile.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)   # 写入文件

with open("profile.json", encoding="utf-8") as f:
    print(f.read())                                     # 看看文件里是什么样子

with open("profile.json", encoding="utf-8") as f:
    loaded = json.load(f)                               # 读回来，又变成了字典
print(loaded["hobbies"][0], type(loaded))

text = json.dumps(data, ensure_ascii=False)             # dumps / loads：和字符串互相转换
print(text)
print(json.loads(text) == data)
```

- `ensure_ascii=False`：让中文、俄文等**原样保存**，否则会变成 `\u5c0f\u660e` 这样的转义符；
- `indent=2`：缩进排版，方便人阅读；
- Python 的 `True`/`False`/`None` 在 JSON 里写作 `true`/`false`/`null`。

## 11.7 常见错误

```python norun
with open("不存在的文件.txt", encoding="utf-8") as f:
    print(f.read())
```

```text
FileNotFoundError: [Errno 2] No such file or directory: '不存在的文件.txt'
```

文件名拼错了，或者文件不在当前工作目录。用 `Path("...").exists()` 先检查，或用 `print(Path.cwd())` 看看当前目录。

```python norun
import random

def random():          # 自己的函数和模块重名了
    return 4

print(random.randint(1, 6))
```

```text
AttributeError: 'function' object has no attribute 'randint'
```

`random` 这个名字被自己的函数覆盖了。自己的变量、函数、文件都不要和模块重名。

```python norun
with open("data.txt", "w") as f:
    f.write(123)
```

```text
TypeError: write() argument must be str, not int
```

`write` 只接受字符串，要写 `f.write(str(123))`（顺便也别忘了 `encoding="utf-8"`）。

## 11.8 小结

- `import 模块`、`from 模块 import 名字`、`import 模块 as 别名`；标准库有 `math`、`random`、`datetime`、`statistics`、`pathlib`、`json`、`csv` 等。
- 自己的 `.py` 文件就是模块；不要和标准库重名；用 `if __name__ == "__main__":` 区分「直接运行」和「被导入」。
- 读写文件：`with open(名字, 模式, encoding="utf-8") as f:`；`"r"` 读、`"w"` 覆盖写、`"a"` 追加；`for line in f` 逐行读。
- **总是写 `encoding="utf-8"`**，否则在不同系统上可能乱码。
- `pathlib.Path` 跨平台处理路径：`/` 拼接、`read_text`、`write_text`、`exists`、`glob`。
- CSV 用 `csv` 模块（加 `newline=""`）；JSON 用 `json.dump/load`（加 `ensure_ascii=False`）。

## 11.9 练习

<!-- exercise: stats -->
### 练习 1：用标准库做统计

输入一行若干个数，用 `statistics` 模块输出它们的**平均数**和**中位数**，各保留 2 位小数，空格隔开。

```text
样例输入：3 1 4 1 5 9 2 6
样例输出：3.88 3.50
```

<details>
<summary>提示</summary>

`import statistics`，然后 `statistics.mean(nums)` 和 `statistics.median(nums)`。
</details>

<!-- exercise: weekday -->
### 练习 2：星期几、第几天

输入一个 `YYYY-MM-DD` 格式的日期，输出它是星期几（1 表示星期一，7 表示星期日）和它是这一年的第几天，空格隔开。

```text
样例输入：2025-03-01
样例输出：6 60
```

<details>
<summary>提示</summary>

`from datetime import date`；`d = date.fromisoformat(text)`；`d.isoweekday()` 是星期几，`d.timetuple().tm_yday` 是一年中的第几天。
</details>

<!-- exercise: to-json -->
### 练习 3：变成 JSON

第一行输入名字，第二行输入年龄（整数）。把它们放进字典 `{"name": 名字, "age": 年龄}`，用 `json.dumps` 输出（中文要原样显示）。

```text
样例输入：
小明
12
样例输出：{"name": "小明", "age": 12}
```

<details>
<summary>提示</summary>

`json.dumps(data, ensure_ascii=False)`。年龄要先 `int()`，这样输出的是 `12` 而不是 `"12"`。
</details>

<!-- exercise: file-roundtrip -->
### 练习 4：写进文件，再读出来

第一行是整数 `n`，接下来 `n` 行文字。把这 `n` 行写进文件 `lines.txt`（记得 `encoding="utf-8"`），然后**重新打开文件读出来**，输出文件里有几行，以及最长的一行（有几行一样长时输出最先出现的）。

```text
样例输入：
3
苹果
西瓜和葡萄
香蕉
样例输出：
3
西瓜和葡萄
```

<details>
<summary>提示</summary>

写的时候每行加 `"\n"`；读的时候 `lines = [line.rstrip("\n") for line in f]`，最长的一行是 `max(lines, key=len)`。
</details>

<!-- exercise: why-encoding -->
### 练习 5：为什么要写 encoding？

打开文本文件时总是写 `encoding="utf-8"`，主要是为了？

- 让文件变小
- 让程序在不同的操作系统（比如中文版 Windows）上读写中文、俄文等文字时不乱码、不报错
- 让文件读得更快
- 不写就无法打开文件

<details>
<summary>答案与解析</summary>

不写的话，Python 使用系统默认编码，中文版 Windows 上常常是 GBK。用 UTF-8 写的文件拿到那里按 GBK 读，就会乱码或报 `UnicodeDecodeError`。
</details>

<!-- exercise: why-with -->
### 练习 6：`with` 的作用

`with open(...) as f:` 比直接 `f = open(...)` 好在哪里？

- 能打开更大的文件
- 代码块结束时自动关闭文件，即使中间出错也会关闭
- 会自动加上 `encoding="utf-8"`
- 会自动备份文件

<details>
<summary>答案与解析</summary>

`with` 保证文件一定会被关闭。不关闭的文件，写入的内容可能还没真正保存到磁盘，也会占用系统资源。它不会替你选择编码。
</details>

## 11.10 延伸阅读

- [Python 标准库文档](https://docs.python.org/zh-cn/3/library/index.html)（中文）
- [Python 官方教程 · 7.2 读写文件](https://docs.python.org/zh-cn/3/tutorial/inputoutput.html#reading-and-writing-files)
- [pathlib 文档](https://docs.python.org/zh-cn/3/library/pathlib.html)、[json 文档](https://docs.python.org/zh-cn/3/library/json.html)
- 下一课：[异常处理](../12-exceptions/zh-CN.md)
