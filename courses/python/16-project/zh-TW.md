# 16. 綜合專案：命令列記賬本

> 🎯 **本課目標**：把這門課學到的東西串起來，從零做一個**真正有用**的小程式——命令列記賬本：可以記錄花費、列出記錄、按類別統計、刪除，資料儲存在檔案裡，關掉程式再開啟也不會丟。在這個過程中練習**拆分問題**、**設計資料**、把程式分成「資料 / 儲存 / 邏輯 / 介面」幾層；最後把它變成一個帶測試的 uv 專案，並想想還能怎樣擴充套件。
>
> ⏱ **預計用時**：120 分鐘　｜　📋 **前置知識**：本課程第 1～15 課

恭喜你來到 Python 課程的最後一課！前面每一課都在學「零件」，這一課我們來造一臺完整的「機器」。不要急，跟著一步一步來，每一部分都能在前面的課裡找到出處。

## 16.1 先想清楚要做什麼

寫程式碼之前，先用自己的話把需求寫下來。這比你想象的重要得多：

```text
我想要一個記賬本程式：
1. 記一筆花費：金額、類別（餐飲、交通……）、備註，自動記上日期和編號
2. 列出所有花費
3. 按類別統計：每類花了多少、佔多少比例，最好能畫個簡單的條形圖
4. 刪除記錯的一筆
5. 資料存在檔案裡，下次開啟還在
6. 輸錯了不要崩潰，要給出友好的提示
```

然後想一想程式**和使用者怎樣交流**。我們做一個「命令列介面」：程式顯示提示符 `>`，使用者輸入一條命令，程式執行後再等下一條：

```text
> add 35 餐飲 午飯
✓ 已記錄 #1：餐飲 35.00
> report
餐飲      35.00  100.0%  ████████████████████
合計 35.00
> quit
再見！
```

## 16.2 把程式分成幾層

一個常見的好習慣：把程式按職責分成幾層，每層只管一件事：

| 層 | 負責 | 用到的知識 |
|----|------|-----------|
| **資料** | 一筆花費長什麼樣 | `@dataclass`（第 13 課） |
| **儲存** | 讀寫檔案 | `json`、`pathlib`、異常處理（第 11、12 課） |
| **邏輯** | 增加、刪除、統計 | 列表、字典、函式、排序（第 8～10 課） |
| **介面** | 讀命令、顯示結果 | `input`、`match`、f-string（第 5、14 課） |

這樣分層的好處：邏輯層的函式**不讀輸入、不列印**，只接收引數、返回結果，所以很容易測試；以後想把命令列介面換成網頁或圖形介面，只需要改介面層。

## 16.3 資料層：一筆花費

```python norun
from dataclasses import dataclass

@dataclass
class Expense:
    id: int
    day: str            # 日期，如 "2025-03-01"
    category: str       # 類別，如 "餐飲"
    amount: float       # 金額
    note: str = ""      # 備註
```

每一筆花費有一個**編號** `id`，刪除時就靠它來指定是哪一筆。

## 16.4 儲存層：儲存到 JSON 檔案

```python norun
import json
from dataclasses import asdict
from pathlib import Path

DATA_FILE = Path("expenses.json")

def load() -> list[Expense]:
    if not DATA_FILE.exists():                 # 第一次執行：還沒有檔案
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):  # 檔案被改壞了
        print("⚠ 資料檔案損壞，已從空賬本開始")
        return []

def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]       # dataclass → 字典
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")
```

- `asdict(e)` 把 dataclass 物件變成字典，就能存成 JSON；
- `Expense(**row)` 反過來：用字典的鍵值作為關鍵字引數建立物件（`**` 解包，第 14 課）；
- 讀檔案時考慮了兩種意外：檔案不存在、檔案內容損壞。**程式要能應付「不完美」的情況。**

## 16.5 邏輯層：增、刪、統計

```python norun
from datetime import date

def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("金額必須大於 0")
    new_id = max((e.id for e in expenses), default=0) + 1     # 現有最大編號 + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item

def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False                                              # 沒找到

def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount     # 第 9 課的計數寫法
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))  # 花得多的排前面
```

幾個細節：

- `max(..., default=0)`：列表為空時 `max` 會報錯，`default=0` 讓它返回 0；
- `add` 遇到不合法的金額**丟擲異常**（第 12 課），由介面層決定怎麼提示使用者；
- `delete` 返回 `True` / `False` 告訴呼叫者有沒有刪成功。

## 16.6 介面層：讀命令、顯示結果

用 `while True` 不斷讀命令，用 `match` 區分命令（第 14 課）。完整的程式如下——執行它，它會按順序執行下面預先準備好的一串命令（在學習程式裡執行時，你可以自己輸入命令）：

```python input="add 35 餐飲 午飯\nadd 12.5 交通 地鐵\nadd 20 餐飲 奶茶\nadd abc 餐飲\nlist\nreport\ndelete 2\nreport\nquit"
"""命令列記賬本：記錄每一筆花費，按類別統計。"""
import json
from dataclasses import dataclass, asdict
from datetime import date
from pathlib import Path

DATA_FILE = Path("expenses.json")


# ---------- 1. 資料：一筆花費 ----------
@dataclass
class Expense:
    id: int
    day: str            # 日期，如 "2025-03-01"
    category: str       # 類別，如 "餐飲"
    amount: float       # 金額
    note: str = ""      # 備註


# ---------- 2. 儲存：讀寫 JSON 檔案 ----------
def load() -> list[Expense]:
    if not DATA_FILE.exists():
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):
        print("⚠ 資料檔案損壞，已從空賬本開始")
        return []


def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")


# ---------- 3. 邏輯：增、刪、統計（不讀輸入、不列印，方便測試） ----------
def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("金額必須大於 0")
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


# ---------- 4. 介面：解析命令、顯示結果 ----------
HELP = """命令：
  add 金額 類別 [備註]    記一筆，例如：add 35 餐飲 午飯
  list                    列出所有記錄
  report                  按類別統計
  delete 編號             刪除一筆
  help                    顯示幫助
  quit                    退出"""


def show_list(expenses: list[Expense]) -> None:
    if not expenses:
        print("（還沒有記錄）")
        return
    for e in expenses:
        print(f"#{e.id:<3} {e.day}  {e.category:<4} {e.amount:>8.2f}  {e.note}")


def show_report(expenses: list[Expense]) -> None:
    rows = by_category(expenses)
    if not rows:
        print("（還沒有記錄）")
        return
    total = sum(amount for _, amount in rows)
    for category, amount in rows:
        bar = "█" * round(amount / rows[0][1] * 20)      # 最多的那一類畫 20 格
        print(f"{category:<4} {amount:>8.2f}  {amount / total:6.1%}  {bar}")
    print(f"合計 {total:.2f}")


def main() -> None:
    expenses = load()
    print(f"記賬本：已有 {len(expenses)} 筆記錄。輸入 help 檢視命令。")
    while True:
        try:
            line = input("> ")
        except EOFError:                    # 輸入結束（例如按了 Ctrl+D）
            break
        match line.split():
            case ["add", amount, category, *note]:
                try:
                    value = float(amount)
                except ValueError:
                    print(f"✗ 「{amount}」不是數字")
                    continue
                try:
                    item = add(expenses, value, category, " ".join(note))
                except ValueError as e:
                    print("✗", e)
                    continue
                save(expenses)
                print(f"✓ 已記錄 #{item.id}：{item.category} {item.amount:.2f}")
            case ["list"]:
                show_list(expenses)
            case ["report"]:
                show_report(expenses)
            case ["delete", expense_id] if expense_id.isdigit():
                if delete(expenses, int(expense_id)):
                    save(expenses)
                    print(f"✓ 已刪除 #{expense_id}")
                else:
                    print(f"✗ 沒有編號為 {expense_id} 的記錄")
            case ["help"]:
                print(HELP)
            case ["quit"]:
                break
            case []:
                continue
            case _:
                print("✗ 看不懂這個命令，輸入 help 檢視用法")
    print("再見！")


if __name__ == "__main__":
    main()
```

> [!NOTE]
> 每次 `add`、`delete` 之後都會馬上 `save`，所以即使程式中途被關掉，資料也不會丟。在網頁版裡，`expenses.json` 儲存在瀏覽器的臨時檔案系統中，重新整理頁面就沒了；在學習程式或你自己的電腦上執行時，它會儲存在執行目錄裡，下次執行還在。

## 16.7 變成一個「真正的專案」

上一課學的工具現在派上用場了。在終端裡：

```bash
$ uv init ledger
$ cd ledger
$ uv add --dev pytest ruff
```

把程式存成 `ledger.py`。因為邏輯層的函式不讀輸入、不列印，測試寫起來非常輕鬆：

```python norun
# 檔案 test_ledger.py
from ledger import add, by_category, delete

def test_add_assigns_increasing_ids():
    items = []
    a = add(items, 10, "餐飲")
    b = add(items, 5, "交通")
    assert (a.id, b.id) == (1, 2)

def test_add_rejects_non_positive_amount():
    import pytest
    with pytest.raises(ValueError):
        add([], 0, "餐飲")

def test_delete():
    items = []
    add(items, 10, "餐飲")
    assert delete(items, 1) is True
    assert delete(items, 1) is False      # 已經刪掉了
    assert items == []

def test_by_category_sorted_by_amount():
    items = []
    add(items, 10, "交通")
    add(items, 30, "餐飲")
    add(items, 5, "交通")
    assert by_category(items) == [("餐飲", 30), ("交通", 15)]
```

```bash
$ uv run pytest          # 執行測試
$ uv run ruff check .    # 檢查程式碼
$ uv run ruff format .   # 統一格式
$ uv run ledger.py       # 執行程式
```

### 用命令列引數代替互動（可選）

很多命令列工具不是「進入程式再輸入命令」，而是直接在終端裡一次寫完：`python ledger.py add 35 餐飲 午飯`。標準庫的 **`argparse`** 專門處理這種**命令列引數**，並自動生成 `--help` 說明：

```python norun
import argparse

parser = argparse.ArgumentParser(description="命令列記賬本")
sub = parser.add_subparsers(dest="command", required=True)
p_add = sub.add_parser("add", help="記一筆")
p_add.add_argument("amount", type=float)
p_add.add_argument("category")
p_add.add_argument("note", nargs="*")
sub.add_parser("list", help="列出所有記錄")
sub.add_parser("report", help="按類別統計")

args = parser.parse_args()
if args.command == "add":
    ...   # 呼叫邏輯層的 add，然後 save
```

注意：邏輯層和儲存層**一行都不用改**，只換了介面層——這就是分層的好處。

## 16.8 繼續擴充套件：自己動手的想法

做完基本功能，試著自己加一些功能。不知道怎麼做時，回到對應的課複習，或者查官方文件：

1. **按月份統計**：`report 2025-03` 只統計 3 月份的花費（提示：`e.day.startswith(...)`）。
2. **預算提醒**：設定每月預算，超過 80% 時提醒。
3. **匯出 CSV**：`export` 命令把記錄匯出成 Excel 能開啟的 CSV 檔案（第 11 課）。
4. **修改一筆記錄**：`edit 編號 金額`。
5. **好看的表格**：用第三方包 `rich` 畫出彩色表格（`uv add rich`）。
6. **圖表**：用 `matplotlib` 畫出每月花費的柱狀圖。
7. **多個賬本**：用命令列引數指定資料檔案。

## 16.9 你已經學會了什麼

回顧這門課，你已經掌握了：變數和型別、字串、輸入輸出、條件和迴圈、列表、元組、字典、集合、函式、模組、檔案、異常、類、生成器、裝飾器、`match`、虛擬環境、uv、ruff、pytest……這些已經足夠寫出很多實用的程式了。

接下來可以往這些方向走：

| 方向 | 學什麼 |
|------|--------|
| 資料分析 | `pandas`、`matplotlib`、Jupyter |
| 網站和介面 | FastAPI、Django |
| 自動化 | 處理檔案和表格、`requests` 抓取網頁、定時任務 |
| 人工智慧與大模型 | 本專案的 **大模型教程**（LLM-Tutorial）：從數學基礎一直講到訓練自己的 GPT |
| 學第二門語言 | 本課程裡的 **C**（理解計算機底層）、**JavaScript**（網頁）、**Go**（伺服器）、**Rust**（安全又高效）…… |

最重要的是：**找一個你自己真正想做的小專案，動手去做。**遇到不會的就查、就問，這是每個程式設計師每天都在做的事。

## 16.10 練習

<!-- exercise: parse-add -->
### 練習 1：解析 add 命令

輸入一行命令。如果它是 `add 金額 類別 [備註……]` 的形式（至少 3 個詞，第一個詞是 `add`），並且金額是**大於 0 的數**，輸出 `金額 | 類別 | 備註`（金額保留 2 位小數，備註由剩下的詞用空格連線，沒有備註時寫 `-`）；金額不是數字或不大於 0，輸出 `invalid amount`；不是 add 命令或詞數不夠，輸出 `invalid command`。

```text
樣例輸入：add 35 餐飲 午飯 和 咖啡
樣例輸出：35.00 | 餐飲 | 午飯 和 咖啡
```

<details>
<summary>提示</summary>

先 `parts = line.split()`，判斷 `len(parts) < 3 or parts[0] != "add"`；再用 `try: amount = float(parts[1])` 處理金額；備註是 `" ".join(parts[3:]) or "-"`。
</details>

<!-- exercise: category-report -->
### 練習 2：按類別統計

第一行是整數 `n`，接下來 `n` 行每行一個類別和金額。按總金額**從多到少**輸出每個類別和它的總金額（保留 2 位小數，金額相同時按類別名排序），最後一行輸出 `total 總金額`。

```text
樣例輸入：
3
food 35
bus 12.5
food 20
樣例輸出：
food 55.00
bus 12.50
total 67.50
```

<details>
<summary>提示</summary>

用字典累加：`totals[c] = totals.get(c, 0) + amount`；排序用 `sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))`。
</details>

<!-- exercise: text-bars -->
### 練習 3：文字條形圖

輸入若干行，每行一個標籤和一個正整數，最後一行是 `end`。為每一行輸出：標籤（左對齊，寬度等於最長的標籤），一個空格，`#` 號組成的條（長度 = `round(數值 / 最大值 × 10)`），一個空格，數值。

```text
樣例輸入：
food 40
bus 20
fun 8
end
樣例輸出：
food ########## 40
bus  ##### 20
fun  ## 8
```

<details>
<summary>提示</summary>

先把所有行讀進列表，算出 `width = max(len(label) for label, _ in rows)` 和最大值；輸出時用 `f"{label:<{width}} {'#' * n} {value}"`。
</details>

<!-- exercise: why-layers -->
### 練習 4：為什麼要分層？

把程式分成資料、儲存、邏輯、介面幾層，**最主要**的好處是？

- 讓程式執行得更快
- 每層只管一件事：邏輯可以單獨測試，換介面（比如改成網頁）時不用改邏輯和儲存
- 讓檔案更小
- Python 要求必須這樣寫

<details>
<summary>答案與解析</summary>

分層讓每部分職責清楚：邏輯層不讀輸入、不列印，測試很容易；把命令列介面換成 argparse、網頁或圖形介面時，只需要改介面層。
</details>

<!-- exercise: why-file -->
### 練習 5：為什麼要存進檔案？

記賬本為什麼要在每次修改後把資料存進 `expenses.json`？

- 因為變數裡的資料在程式結束後就沒了，存進檔案才能下次接著用
- 因為檔案比變數讀寫更快
- 因為 Python 的列表最多隻能存 100 個元素
- 因為 JSON 檔案可以自動計算總金額

<details>
<summary>答案與解析</summary>

變數存在記憶體裡，程式一結束就消失；檔案存在硬碟上，下次啟動時用 `load()` 讀回來。每次修改後立刻儲存，即使程式意外關閉也不會丟資料。
</details>

## 16.11 延伸閱讀

- [argparse 教程](https://docs.python.org/zh-cn/3/howto/argparse.html)
- [Rich：在終端裡輸出漂亮的表格和顏色](https://rich.readthedocs.io/)
- 《Python 程式設計：從入門到實踐》（Eric Matthes）：有完整的專案實戰
- 恭喜完成 Python 課程！可以在課程列表裡選擇下一門課。
