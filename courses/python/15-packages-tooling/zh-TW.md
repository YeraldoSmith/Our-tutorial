# 15. 包與現代工具鏈

> 🎯 **本課目標**：知道去哪裡找第三方**包**、怎樣安裝；理解**虛擬環境**為什麼必不可少；用現代工具 **uv** 建立專案、管理依賴和 Python 版本；讀懂 `pyproject.toml`；用 **ruff** 統一程式碼風格、發現問題；用型別檢查器提前抓 bug；用 **pytest** 寫自動化測試；學會用 `breakpoint()` 和 `logging` 除錯。這些是寫「真正的專案」時每天都要用的工具。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：入門課第 3、4、7 課（終端、安裝工具、Git）；本課程第 11、12 課

到目前為止，我們的程式都只用 Python 自帶的功能。真實的專案幾乎都會用到別人寫好的**第三方包**，也需要一套工具來保證程式碼品質。本課的大部分內容要在**終端**裡操作（入門課第 3 課），程式碼區塊裡以 `$` 開頭的行是終端命令，不是 Python 程式碼。

## 15.1 第三方包與 PyPI

**PyPI**（Python Package Index，<https://pypi.org>）是 Python 的官方「應用商店」，上面有六十多萬個免費的包。一些最常用的：

| 包 | 用途 |
|----|------|
| `requests`、`httpx` | 訪問網頁和網路介面（API） |
| `numpy`、`pandas` | 數值計算、表格資料分析 |
| `matplotlib` | 畫圖表 |
| `fastapi`、`flask`、`django` | 寫網站和網路服務 |
| `pillow` | 處理圖片 |
| `rich` | 在終端裡輸出彩色、漂亮的文字和表格 |
| `pytest` | 寫測試 |
| `torch`、`transformers` | 深度學習、大模型（見本專案的大模型教程） |

裝好之後，用法和標準庫一樣，`import` 就行：

```python norun
# 先在終端裡安裝：uv add requests（或 pip install requests）
import requests

r = requests.get("https://api.github.com")
print(r.status_code)          # 200 表示成功
print(r.json()["current_user_url"])
```

## 15.2 虛擬環境：每個專案一個獨立的「工具箱」

如果把所有包都裝進系統的 Python 裡，很快就會出問題：專案 A 需要某個包的 1.0 版，專案 B 需要 2.0 版，兩者衝突；也搞不清每個專案到底依賴哪些包。

**虛擬環境**（virtual environment）就是給每個專案一個**獨立的資料夾**（通常叫 `.venv`），這個專案的包都裝在裡面，互不干擾。

用 Python 自帶的 `venv` 和 `pip`：

```bash
$ python -m venv .venv                 # 在當前資料夾建立虛擬環境
$ source .venv/bin/activate            # 啟用（macOS / Linux）
$ .venv\Scripts\activate               # 啟用（Windows）
(.venv) $ pip install requests         # 啟用後，pip 安裝的包只進入這個環境
(.venv) $ pip freeze > requirements.txt   # 把依賴列表記下來
(.venv) $ deactivate                   # 退出虛擬環境
```

啟用後，命令列前面會出現 `(.venv)`。

> [!TIP]
> 中國大陸從 PyPI 下載可能很慢，可以使用國內映象，例如清華大學的映象：
> `pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple`

## 15.3 uv：現代的一站式工具

**uv**（由 Astral 公司開發，用 Rust 編寫）把「安裝 Python、建立虛擬環境、安裝包、鎖定版本、執行程式」都整合在一個工具裡，而且比 pip 快十幾到上百倍。它已經成為很多新專案的首選，本課程也推薦使用它（入門課第 4 課介紹了安裝方法）。

```bash
$ uv init my-app             # 建立一個新專案（包含 pyproject.toml、README 等）
$ cd my-app
$ uv add requests rich       # 新增依賴：自動建立 .venv、安裝包、寫進 pyproject.toml 和 uv.lock
$ uv add --dev pytest ruff   # 只在開發時用的工具（測試、檢查程式碼）
$ uv run main.py             # 在專案的虛擬環境裡執行程式（不需要手動啟用）
$ uv run pytest              # 執行測試
$ uv remove rich             # 刪除依賴
$ uv python install 3.13     # 安裝指定版本的 Python
$ uv sync                    # 在另一臺電腦上：按 uv.lock 裝出完全一樣的環境
```

用 uv 的好處是：**不用手動啟用虛擬環境**（`uv run` 會自動使用專案的環境），而且 `uv.lock` 記錄了每個包的**精確版本**，別人（或者一年後的你）用 `uv sync` 就能得到一模一樣的環境。

> [!TIP]
> 在中國大陸使用 uv，可以設定國內映象：設定環境變數 `UV_DEFAULT_INDEX=https://pypi.tuna.tsinghua.edu.cn/simple`，或者在 `pyproject.toml` 里加入下面幾行。

```toml
[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

### `pyproject.toml`：專案的「身份證」

`uv init` 生成的 `pyproject.toml` 是 Python 專案的標準配置檔案，大致是這樣：

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "我的第一個專案"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32",
    "rich>=14.0",
]

[dependency-groups]
dev = ["pytest>=8", "ruff>=0.12"]

[tool.ruff]
line-length = 100
```

`dependencies` 裡的 `>=2.32` 表示「2.32 或更新的版本」。版本號一般是 `主版本.次版本.修訂號`（**語義化版本**）：修訂號變了只是修 bug；次版本變了是加功能，舊程式碼仍然能用；主版本變了可能有不相容的改動。

### 一個典型的專案結構

```text
my-app/
├── pyproject.toml        專案配置和依賴
├── uv.lock               精確鎖定的版本（交給 uv 管理，不要手動改）
├── README.md             專案說明
├── .gitignore            告訴 Git 不要提交 .venv、__pycache__ 等
├── src/
│   └── my_app/
│       ├── __init__.py   有了它，這個資料夾就是一個「包」
│       └── core.py
└── tests/
    └── test_core.py      測試程式碼
```

`.gitignore` 裡至少應該寫上 `.venv/` 和 `__pycache__/`：虛擬環境可以隨時用 `uv sync` 重新生成，不需要也不應該提交到 Git（入門課第 7 課）。

## 15.4 ruff：程式碼風格與靜態檢查

Python 社群有一份程式碼風格指南 **PEP 8**：縮排 4 個空格、變數名用 `snake_case`（小寫加下劃線）、類名用 `CamelCase`、運算子兩邊加空格……統一的風格讓大家的程式碼讀起來都一樣舒服。

**ruff**（也來自 Astral）可以**自動**做兩件事：

```bash
$ uv run ruff format .       # 格式化：自動把程式碼改成統一的風格
$ uv run ruff check .        # 檢查：找出可能的錯誤和不好的寫法
$ uv run ruff check --fix .  # 能自動修的就自動修
```

例如這段程式碼：

```python norun
import os, sys
def Add(a,b):
    unused = 1
    if a == None:
        return b
    return a+b
```

`ruff check` 會指出：一行匯入了多個模組、`os` 和 `sys` 匯入了卻沒用、變數 `unused` 賦值後沒用、`== None` 應該寫成 `is None`；`ruff format` 會自動加上空格和空行。（函式名 `Add` 不符合 PEP 8 的命名規則，這一條需要在配置裡開啟 `N` 規則才會檢查。）

> [!TIP]
> 在 VS Code 裡安裝 **Ruff** 擴充套件，儲存檔案時就會自動格式化，問題會直接在編輯器裡標出來。

## 15.5 型別檢查：在執行之前抓 bug

第 10 課說過，型別標註不會被 Python 強制檢查。**型別檢查器**會讀你的程式碼和標註，**不執行程式**就找出型別錯誤：

```python norun
def total_price(prices: list[float]) -> float:
    return sum(prices)

print(total_price(["3.5", "2"]))      # 傳入的是字串列表
```

```bash
$ uv run mypy main.py
main.py:4: error: List item 0 has incompatible type "str"; expected "float"
```

常用的型別檢查器有 **mypy**、**pyright**（VS Code 的 Pylance 擴充套件內建了它），以及 Astral 新推出的 **ty**。專案越大，型別檢查越有用。

## 15.6 自動化測試：pytest

寫完一個函式，你會手動試幾個例子看看對不對。**自動化測試**就是把這些例子寫成程式碼，以後每次修改都能一鍵全部重新檢查，確保沒有把原來好的地方改壞。

**pytest** 是最流行的測試工具，寫法非常簡單：檔名以 `test_` 開頭，函式名以 `test_` 開頭，裡面用 `assert`：

```python norun
# 檔案 src/my_app/core.py
def slugify(title: str) -> str:
    """把標題變成網址裡用的形式：小寫，空格換成連字元。"""
    return "-".join(title.lower().split())
```

```python norun
# 檔案 tests/test_core.py
from my_app.core import slugify

def test_basic():
    assert slugify("Hello World") == "hello-world"

def test_extra_spaces():
    assert slugify("  Learn   Python  ") == "learn-python"

def test_empty():
    assert slugify("") == ""
```

```bash
$ uv run pytest
============================= test session starts ==============================
collected 3 items

tests/test_core.py ...                                                   [100%]

============================== 3 passed in 0.01s ===============================
```

每個點 `.` 表示一個測試透過。如果有測試失敗，pytest 會清楚地顯示「期望什麼、實際得到什麼」。

學習程式和網頁一次只執行一個檔案，沒法直接用 pytest。我們可以用標準庫自帶的 **unittest** 在一個檔案裡體驗同樣的思路：

```python
import unittest

def slugify(title: str) -> str:
    return "-".join(title.lower().split())

class TestSlugify(unittest.TestCase):
    def test_basic(self):
        self.assertEqual(slugify("Hello World"), "hello-world")

    def test_extra_spaces(self):
        self.assertEqual(slugify("  Learn   Python  "), "learn-python")

    def test_empty(self):
        self.assertEqual(slugify(""), "")

unittest.main(argv=["tests"], exit=False, verbosity=2)    # 結果輸出在錯誤輸出（stderr）裡
```

> [!NOTE]
> **先寫測試**也是一種常見的做法（測試驅動開發，TDD）：先寫出「我希望函式怎樣工作」的測試，看著它失敗，再寫程式碼讓它透過。本課程每道程式設計練習的自動判題，其實就是一組測試。

## 15.7 除錯：`breakpoint()` 與 `logging`

除了到處加 `print`，還有更好的除錯方法：

**1. 斷點除錯**：在程式碼裡寫一行 `breakpoint()`，程式執行到這裡會暫停，進入偵錯程式 `pdb`，你可以檢視變數、一步一步執行：

```python norun
def average(nums):
    total = sum(nums)
    breakpoint()            # 執行到這裡暫停；輸入 p total 檢視變數，n 執行下一行，c 繼續，q 退出
    return total / len(nums)
```

VS Code、PyCharm 等編輯器有圖形化的偵錯程式：在行號左邊點一下設定斷點，按 F5 開始除錯，更直觀。

**2. 日誌**：`logging` 模組比 `print` 更適合記錄程式執行的情況——可以分級別（除錯、資訊、警告、錯誤）、帶上時間，正式執行時只要改一個設定就能關掉除錯資訊：

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
log = logging.getLogger("shop")

def buy(item, money):
    log.debug("準備購買 %s", item)            # 級別是 INFO，所以 DEBUG 資訊不顯示
    if money < 10:
        log.warning("錢不夠買 %s：只有 %d 元", item, money)
        return False
    log.info("買到了 %s", item)
    return True

buy("書", 30)
buy("玩具", 5)
```

## 15.8 小結

- 第三方包在 PyPI 上；`uv add 包名` 或 `pip install 包名` 安裝，然後 `import`。
- **每個專案一個虛擬環境**（`.venv`），避免版本衝突；不要把 `.venv` 提交到 Git。
- **uv**：`uv init` 建專案、`uv add` 加依賴、`uv run` 執行、`uv sync` 還原環境；依賴記錄在 `pyproject.toml`，精確版本鎖在 `uv.lock`。
- **ruff format** 統一風格（PEP 8），**ruff check** 找出問題；**mypy / pyright / ty** 做型別檢查。
- **pytest**：`test_` 開頭的檔案和函式 + `assert`，`uv run pytest` 一鍵執行全部測試。
- 除錯用 `breakpoint()` 或編輯器的偵錯程式；記錄執行情況用 `logging`。

## 15.9 練習

<!-- exercise: version-compare -->
### 練習 1：比較版本號

輸入一行兩個版本號（例如 `1.10.0 1.9.3`），輸出**較新**的那個；如果一樣新，輸出 `equal`。版本號由若干個用 `.` 隔開的整陣列成，要**按數字**比較（`1.10` 比 `1.9` 新），位數不同時缺少的部分當作 0（`1.2` 和 `1.2.0` 一樣）。

```text
樣例輸入：1.10.0 1.9.3
樣例輸出：1.10.0
```

<details>
<summary>提示</summary>

把版本號變成整數列表：`[int(x) for x in v.split(".")]`，短的那個在後面補 0 到一樣長，然後直接用 `>` 比較兩個列表（Python 會逐個元素比較）。
</details>

<!-- exercise: snake-case -->
### 練習 2：改成 PEP 8 風格的名字

PEP 8 規定變數名和函式名用 `snake_case`。輸入一個駝峰寫法（camelCase）的名字，把每個大寫字母變成「下劃線 + 小寫字母」後輸出。（輸入的第一個字母一定是小寫。）

```text
樣例輸入：getUserName
樣例輸出：get_user_name
```

<details>
<summary>提示</summary>

遍歷每個字元：`if ch.isupper(): result += "_" + ch.lower()`，否則直接加上。
</details>

<!-- exercise: fix-clamp -->
### 練習 3：讓測試透過

起始程式碼裡有一個函式 `clamp(x, lo, hi)`（把 `x` 限制在 `lo` 和 `hi` 之間：小於 `lo` 就返回 `lo`，大於 `hi` 就返回 `hi`，否則返回 `x`），還有幾條用 `assert` 寫的測試。但函式寫錯了，測試會失敗。修好這個函式，讓程式輸出 `all tests passed`。**不要修改測試。**

```text
樣例輸出：all tests passed
```

<details>
<summary>提示</summary>

執行一下，看看是哪條 `assert` 失敗了（Traceback 會指出行號），想想那個輸入應該得到什麼。正確的寫法可以是 `return max(lo, min(x, hi))`。
</details>

<!-- exercise: why-venv -->
### 練習 4：為什麼要用虛擬環境？

- 讓 Python 執行得更快
- 讓每個專案有自己獨立的包和版本，互不衝突，也能清楚記錄每個專案的依賴
- 沒有虛擬環境就無法安裝包
- 為了讓程式碼更安全，防止被病毒感染

<details>
<summary>答案與解析</summary>

虛擬環境把每個專案的包隔離開：專案 A 用 1.0 版、專案 B 用 2.0 版也不會衝突；配合 `pyproject.toml` / `uv.lock`，別人能裝出完全一樣的環境。
</details>

<!-- exercise: tool-roles -->
### 練習 5：工具的分工

下面哪一項把工具和它的主要用途**全部**對應正確了？

- ruff：執行測試；pytest：格式化程式碼；uv：型別檢查
- uv：管理依賴和虛擬環境；ruff：格式化和檢查程式碼；pytest：執行測試
- uv：畫圖表；ruff：訪問網路；pytest：安裝 Python
- pytest：管理依賴；uv：檢查程式碼風格；ruff：執行程式

<details>
<summary>答案與解析</summary>

uv 管專案、依賴、虛擬環境和 Python 版本；ruff 管程式碼風格和靜態檢查；pytest 管自動化測試。
</details>

## 15.10 延伸閱讀

- [uv 官方文件](https://docs.astral.sh/uv/)、[Ruff 官方文件](https://docs.astral.sh/ruff/)
- [pytest 官方文件](https://docs.pytest.org/)
- [PEP 8 – Python 程式碼風格指南](https://peps.python.org/pep-0008/)（網上有中文翻譯）
- [Python 打包使用者指南](https://packaging.python.org/zh-cn/latest/)（有中文版）
- 下一課：[綜合專案：命令列記賬本](../16-project/zh-CN.md)
