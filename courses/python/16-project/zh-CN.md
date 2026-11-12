# 16. 综合项目：命令行记账本

> 🎯 **本课目标**：把这门课学到的东西串起来，从零做一个**真正有用**的小程序——命令行记账本：可以记录花费、列出记录、按类别统计、删除，数据保存在文件里，关掉程序再打开也不会丢。在这个过程中练习**拆分问题**、**设计数据**、把程序分成「数据 / 存储 / 逻辑 / 界面」几层；最后把它变成一个带测试的 uv 项目，并想想还能怎样扩展。
>
> ⏱ **预计用时**：120 分钟　｜　📋 **前置知识**：本课程第 1～15 课

恭喜你来到 Python 课程的最后一课！前面每一课都在学「零件」，这一课我们来造一台完整的「机器」。不要急，跟着一步一步来，每一部分都能在前面的课里找到出处。

## 16.1 先想清楚要做什么

写代码之前，先用自己的话把需求写下来。这比你想象的重要得多：

```text
我想要一个记账本程序：
1. 记一笔花费：金额、类别（餐饮、交通……）、备注，自动记上日期和编号
2. 列出所有花费
3. 按类别统计：每类花了多少、占多少比例，最好能画个简单的条形图
4. 删除记错的一笔
5. 数据存在文件里，下次打开还在
6. 输错了不要崩溃，要给出友好的提示
```

然后想一想程序**和用户怎样交流**。我们做一个「命令行界面」：程序显示提示符 `>`，用户输入一条命令，程序执行后再等下一条：

```text
> add 35 餐饮 午饭
✓ 已记录 #1：餐饮 35.00
> report
餐饮      35.00  100.0%  ████████████████████
合计 35.00
> quit
再见！
```

## 16.2 把程序分成几层

一个常见的好习惯：把程序按职责分成几层，每层只管一件事：

| 层 | 负责 | 用到的知识 |
|----|------|-----------|
| **数据** | 一笔花费长什么样 | `@dataclass`（第 13 课） |
| **存储** | 读写文件 | `json`、`pathlib`、异常处理（第 11、12 课） |
| **逻辑** | 增加、删除、统计 | 列表、字典、函数、排序（第 8～10 课） |
| **界面** | 读命令、显示结果 | `input`、`match`、f-string（第 5、14 课） |

这样分层的好处：逻辑层的函数**不读输入、不打印**，只接收参数、返回结果，所以很容易测试；以后想把命令行界面换成网页或图形界面，只需要改界面层。

## 16.3 数据层：一笔花费

```python norun
from dataclasses import dataclass

@dataclass
class Expense:
    id: int
    day: str            # 日期，如 "2025-03-01"
    category: str       # 类别，如 "餐饮"
    amount: float       # 金额
    note: str = ""      # 备注
```

每一笔花费有一个**编号** `id`，删除时就靠它来指定是哪一笔。

## 16.4 存储层：保存到 JSON 文件

```python norun
import json
from dataclasses import asdict
from pathlib import Path

DATA_FILE = Path("expenses.json")

def load() -> list[Expense]:
    if not DATA_FILE.exists():                 # 第一次运行：还没有文件
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):  # 文件被改坏了
        print("⚠ 数据文件损坏，已从空账本开始")
        return []

def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]       # dataclass → 字典
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")
```

- `asdict(e)` 把 dataclass 对象变成字典，就能存成 JSON；
- `Expense(**row)` 反过来：用字典的键值作为关键字参数创建对象（`**` 解包，第 14 课）；
- 读文件时考虑了两种意外：文件不存在、文件内容损坏。**程序要能应付「不完美」的情况。**

## 16.5 逻辑层：增、删、统计

```python norun
from datetime import date

def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("金额必须大于 0")
    new_id = max((e.id for e in expenses), default=0) + 1     # 现有最大编号 + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item

def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False                                              # 没找到

def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount     # 第 9 课的计数写法
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))  # 花得多的排前面
```

几个细节：

- `max(..., default=0)`：列表为空时 `max` 会报错，`default=0` 让它返回 0；
- `add` 遇到不合法的金额**抛出异常**（第 12 课），由界面层决定怎么提示用户；
- `delete` 返回 `True` / `False` 告诉调用者有没有删成功。

## 16.6 界面层：读命令、显示结果

用 `while True` 不断读命令，用 `match` 区分命令（第 14 课）。完整的程序如下——运行它，它会按顺序执行下面预先准备好的一串命令（在学习程序里运行时，你可以自己输入命令）：

```python input="add 35 餐饮 午饭\nadd 12.5 交通 地铁\nadd 20 餐饮 奶茶\nadd abc 餐饮\nlist\nreport\ndelete 2\nreport\nquit"
"""命令行记账本：记录每一笔花费，按类别统计。"""
import json
from dataclasses import dataclass, asdict
from datetime import date
from pathlib import Path

DATA_FILE = Path("expenses.json")


# ---------- 1. 数据：一笔花费 ----------
@dataclass
class Expense:
    id: int
    day: str            # 日期，如 "2025-03-01"
    category: str       # 类别，如 "餐饮"
    amount: float       # 金额
    note: str = ""      # 备注


# ---------- 2. 存储：读写 JSON 文件 ----------
def load() -> list[Expense]:
    if not DATA_FILE.exists():
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):
        print("⚠ 数据文件损坏，已从空账本开始")
        return []


def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")


# ---------- 3. 逻辑：增、删、统计（不读输入、不打印，方便测试） ----------
def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("金额必须大于 0")
    new_id = max((e.id for e in expenses), default=0) + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item


def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False


def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))


# ---------- 4. 界面：解析命令、显示结果 ----------
HELP = """命令：
  add 金额 类别 [备注]    记一笔，例如：add 35 餐饮 午饭
  list                    列出所有记录
  report                  按类别统计
  delete 编号             删除一笔
  help                    显示帮助
  quit                    退出"""


def show_list(expenses: list[Expense]) -> None:
    if not expenses:
        print("（还没有记录）")
        return
    for e in expenses:
        print(f"#{e.id:<3} {e.day}  {e.category:<4} {e.amount:>8.2f}  {e.note}")


def show_report(expenses: list[Expense]) -> None:
    rows = by_category(expenses)
    if not rows:
        print("（还没有记录）")
        return
    total = sum(amount for _, amount in rows)
    for category, amount in rows:
        bar = "█" * round(amount / rows[0][1] * 20)      # 最多的那一类画 20 格
        print(f"{category:<4} {amount:>8.2f}  {amount / total:6.1%}  {bar}")
    print(f"合计 {total:.2f}")


def main() -> None:
    expenses = load()
    print(f"记账本：已有 {len(expenses)} 笔记录。输入 help 查看命令。")
    while True:
        try:
            line = input("> ")
        except EOFError:                    # 输入结束（例如按了 Ctrl+D）
            break
        match line.split():
            case ["add", amount, category, *note]:
                try:
                    value = float(amount)
                except ValueError:
                    print(f"✗ 「{amount}」不是数字")
                    continue
                try:
                    item = add(expenses, value, category, " ".join(note))
                except ValueError as e:
                    print("✗", e)
                    continue
                save(expenses)
                print(f"✓ 已记录 #{item.id}：{item.category} {item.amount:.2f}")
            case ["list"]:
                show_list(expenses)
            case ["report"]:
                show_report(expenses)
            case ["delete", expense_id] if expense_id.isdigit():
                if delete(expenses, int(expense_id)):
                    save(expenses)
                    print(f"✓ 已删除 #{expense_id}")
                else:
                    print(f"✗ 没有编号为 {expense_id} 的记录")
            case ["help"]:
                print(HELP)
            case ["quit"]:
                break
            case []:
                continue
            case _:
                print("✗ 看不懂这个命令，输入 help 查看用法")
    print("再见！")


if __name__ == "__main__":
    main()
```

> [!NOTE]
> 每次 `add`、`delete` 之后都会马上 `save`，所以即使程序中途被关掉，数据也不会丢。在网页版里，`expenses.json` 保存在浏览器的临时文件系统中，刷新页面就没了；在学习程序或你自己的电脑上运行时，它会保存在运行目录里，下次运行还在。

## 16.7 变成一个「真正的项目」

上一课学的工具现在派上用场了。在终端里：

```bash
$ uv init ledger
$ cd ledger
$ uv add --dev pytest ruff
```

把程序存成 `ledger.py`。因为逻辑层的函数不读输入、不打印，测试写起来非常轻松：

```python norun
# 文件 test_ledger.py
from ledger import add, by_category, delete

def test_add_assigns_increasing_ids():
    items = []
    a = add(items, 10, "餐饮")
    b = add(items, 5, "交通")
    assert (a.id, b.id) == (1, 2)

def test_add_rejects_non_positive_amount():
    import pytest
    with pytest.raises(ValueError):
        add([], 0, "餐饮")

def test_delete():
    items = []
    add(items, 10, "餐饮")
    assert delete(items, 1) is True
    assert delete(items, 1) is False      # 已经删掉了
    assert items == []

def test_by_category_sorted_by_amount():
    items = []
    add(items, 10, "交通")
    add(items, 30, "餐饮")
    add(items, 5, "交通")
    assert by_category(items) == [("餐饮", 30), ("交通", 15)]
```

```bash
$ uv run pytest          # 运行测试
$ uv run ruff check .    # 检查代码
$ uv run ruff format .   # 统一格式
$ uv run ledger.py       # 运行程序
```

### 用命令行参数代替交互（可选）

很多命令行工具不是「进入程序再输入命令」，而是直接在终端里一次写完：`python ledger.py add 35 餐饮 午饭`。标准库的 **`argparse`** 专门处理这种**命令行参数**，并自动生成 `--help` 说明：

```python norun
import argparse

parser = argparse.ArgumentParser(description="命令行记账本")
sub = parser.add_subparsers(dest="command", required=True)
p_add = sub.add_parser("add", help="记一笔")
p_add.add_argument("amount", type=float)
p_add.add_argument("category")
p_add.add_argument("note", nargs="*")
sub.add_parser("list", help="列出所有记录")
sub.add_parser("report", help="按类别统计")

args = parser.parse_args()
if args.command == "add":
    ...   # 调用逻辑层的 add，然后 save
```

注意：逻辑层和存储层**一行都不用改**，只换了界面层——这就是分层的好处。

## 16.8 继续扩展：自己动手的想法

做完基本功能，试着自己加一些功能。不知道怎么做时，回到对应的课复习，或者查官方文档：

1. **按月份统计**：`report 2025-03` 只统计 3 月份的花费（提示：`e.day.startswith(...)`）。
2. **预算提醒**：设定每月预算，超过 80% 时提醒。
3. **导出 CSV**：`export` 命令把记录导出成 Excel 能打开的 CSV 文件（第 11 课）。
4. **修改一笔记录**：`edit 编号 金额`。
5. **好看的表格**：用第三方包 `rich` 画出彩色表格（`uv add rich`）。
6. **图表**：用 `matplotlib` 画出每月花费的柱状图。
7. **多个账本**：用命令行参数指定数据文件。

## 16.9 你已经学会了什么

回顾这门课，你已经掌握了：变量和类型、字符串、输入输出、条件和循环、列表、元组、字典、集合、函数、模块、文件、异常、类、生成器、装饰器、`match`、虚拟环境、uv、ruff、pytest……这些已经足够写出很多实用的程序了。

接下来可以往这些方向走：

| 方向 | 学什么 |
|------|--------|
| 数据分析 | `pandas`、`matplotlib`、Jupyter |
| 网站和接口 | FastAPI、Django |
| 自动化 | 处理文件和表格、`requests` 抓取网页、定时任务 |
| 人工智能与大模型 | 本项目的 **大模型教程**（LLM-Tutorial）：从数学基础一直讲到训练自己的 GPT |
| 学第二门语言 | 本课程里的 **C**（理解计算机底层）、**JavaScript**（网页）、**Go**（服务器）、**Rust**（安全又高效）…… |

最重要的是：**找一个你自己真正想做的小项目，动手去做。**遇到不会的就查、就问，这是每个程序员每天都在做的事。

## 16.10 练习

<!-- exercise: parse-add -->
### 练习 1：解析 add 命令

输入一行命令。如果它是 `add 金额 类别 [备注……]` 的形式（至少 3 个词，第一个词是 `add`），并且金额是**大于 0 的数**，输出 `金额 | 类别 | 备注`（金额保留 2 位小数，备注由剩下的词用空格连接，没有备注时写 `-`）；金额不是数字或不大于 0，输出 `invalid amount`；不是 add 命令或词数不够，输出 `invalid command`。

```text
样例输入：add 35 餐饮 午饭 和 咖啡
样例输出：35.00 | 餐饮 | 午饭 和 咖啡
```

<details>
<summary>提示</summary>

先 `parts = line.split()`，判断 `len(parts) < 3 or parts[0] != "add"`；再用 `try: amount = float(parts[1])` 处理金额；备注是 `" ".join(parts[3:]) or "-"`。
</details>

<!-- exercise: category-report -->
### 练习 2：按类别统计

第一行是整数 `n`，接下来 `n` 行每行一个类别和金额。按总金额**从多到少**输出每个类别和它的总金额（保留 2 位小数，金额相同时按类别名排序），最后一行输出 `total 总金额`。

```text
样例输入：
3
food 35
bus 12.5
food 20
样例输出：
food 55.00
bus 12.50
total 67.50
```

<details>
<summary>提示</summary>

用字典累加：`totals[c] = totals.get(c, 0) + amount`；排序用 `sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))`。
</details>

<!-- exercise: text-bars -->
### 练习 3：文字条形图

输入若干行，每行一个标签和一个正整数，最后一行是 `end`。为每一行输出：标签（左对齐，宽度等于最长的标签），一个空格，`#` 号组成的条（长度 = `round(数值 / 最大值 × 10)`），一个空格，数值。

```text
样例输入：
food 40
bus 20
fun 8
end
样例输出：
food ########## 40
bus  ##### 20
fun  ## 8
```

<details>
<summary>提示</summary>

先把所有行读进列表，算出 `width = max(len(label) for label, _ in rows)` 和最大值；输出时用 `f"{label:<{width}} {'#' * n} {value}"`。
</details>

<!-- exercise: why-layers -->
### 练习 4：为什么要分层？

把程序分成数据、存储、逻辑、界面几层，**最主要**的好处是？

- 让程序运行得更快
- 每层只管一件事：逻辑可以单独测试，换界面（比如改成网页）时不用改逻辑和存储
- 让文件更小
- Python 要求必须这样写

<details>
<summary>答案与解析</summary>

分层让每部分职责清楚：逻辑层不读输入、不打印，测试很容易；把命令行界面换成 argparse、网页或图形界面时，只需要改界面层。
</details>

<!-- exercise: why-file -->
### 练习 5：为什么要存进文件？

记账本为什么要在每次修改后把数据存进 `expenses.json`？

- 因为变量里的数据在程序结束后就没了，存进文件才能下次接着用
- 因为文件比变量读写更快
- 因为 Python 的列表最多只能存 100 个元素
- 因为 JSON 文件可以自动计算总金额

<details>
<summary>答案与解析</summary>

变量存在内存里，程序一结束就消失；文件存在硬盘上，下次启动时用 `load()` 读回来。每次修改后立刻保存，即使程序意外关闭也不会丢数据。
</details>

## 16.11 延伸阅读

- [argparse 教程](https://docs.python.org/zh-cn/3/howto/argparse.html)
- [Rich：在终端里输出漂亮的表格和颜色](https://rich.readthedocs.io/)
- 《Python 编程：从入门到实践》（Eric Matthes）：有完整的项目实战
- 恭喜完成 Python 课程！可以在课程列表里选择下一门课。
