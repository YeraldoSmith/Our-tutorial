# 15. 包与现代工具链

> 🎯 **本课目标**：知道去哪里找第三方**包**、怎样安装；理解**虚拟环境**为什么必不可少；用现代工具 **uv** 创建项目、管理依赖和 Python 版本；读懂 `pyproject.toml`；用 **ruff** 统一代码风格、发现问题；用类型检查器提前抓 bug；用 **pytest** 写自动化测试；学会用 `breakpoint()` 和 `logging` 调试。这些是写「真正的项目」时每天都要用的工具。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：入门课第 3、4、7 课（终端、安装工具、Git）；本课程第 11、12 课

到目前为止，我们的程序都只用 Python 自带的功能。真实的项目几乎都会用到别人写好的**第三方包**，也需要一套工具来保证代码质量。本课的大部分内容要在**终端**里操作（入门课第 3 课），代码块里以 `$` 开头的行是终端命令，不是 Python 代码。

## 15.1 第三方包与 PyPI

**PyPI**（Python Package Index，<https://pypi.org>）是 Python 的官方「应用商店」，上面有六十多万个免费的包。一些最常用的：

| 包 | 用途 |
|----|------|
| `requests`、`httpx` | 访问网页和网络接口（API） |
| `numpy`、`pandas` | 数值计算、表格数据分析 |
| `matplotlib` | 画图表 |
| `fastapi`、`flask`、`django` | 写网站和网络服务 |
| `pillow` | 处理图片 |
| `rich` | 在终端里输出彩色、漂亮的文字和表格 |
| `pytest` | 写测试 |
| `torch`、`transformers` | 深度学习、大模型（见本项目的大模型教程） |

装好之后，用法和标准库一样，`import` 就行：

```python norun
# 先在终端里安装：uv add requests（或 pip install requests）
import requests

r = requests.get("https://api.github.com")
print(r.status_code)          # 200 表示成功
print(r.json()["current_user_url"])
```

## 15.2 虚拟环境：每个项目一个独立的「工具箱」

如果把所有包都装进系统的 Python 里，很快就会出问题：项目 A 需要某个包的 1.0 版，项目 B 需要 2.0 版，两者冲突；也搞不清每个项目到底依赖哪些包。

**虚拟环境**（virtual environment）就是给每个项目一个**独立的文件夹**（通常叫 `.venv`），这个项目的包都装在里面，互不干扰。

用 Python 自带的 `venv` 和 `pip`：

```bash
$ python -m venv .venv                 # 在当前文件夹创建虚拟环境
$ source .venv/bin/activate            # 激活（macOS / Linux）
$ .venv\Scripts\activate               # 激活（Windows）
(.venv) $ pip install requests         # 激活后，pip 安装的包只进入这个环境
(.venv) $ pip freeze > requirements.txt   # 把依赖列表记下来
(.venv) $ deactivate                   # 退出虚拟环境
```

激活后，命令行前面会出现 `(.venv)`。

> [!TIP]
> 中国大陆从 PyPI 下载可能很慢，可以使用国内镜像，例如清华大学的镜像：
> `pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple`

## 15.3 uv：现代的一站式工具

**uv**（由 Astral 公司开发，用 Rust 编写）把「安装 Python、创建虚拟环境、安装包、锁定版本、运行程序」都集成在一个工具里，而且比 pip 快十几到上百倍。它已经成为很多新项目的首选，本课程也推荐使用它（入门课第 4 课介绍了安装方法）。

```bash
$ uv init my-app             # 创建一个新项目（包含 pyproject.toml、README 等）
$ cd my-app
$ uv add requests rich       # 添加依赖：自动创建 .venv、安装包、写进 pyproject.toml 和 uv.lock
$ uv add --dev pytest ruff   # 只在开发时用的工具（测试、检查代码）
$ uv run main.py             # 在项目的虚拟环境里运行程序（不需要手动激活）
$ uv run pytest              # 运行测试
$ uv remove rich             # 删除依赖
$ uv python install 3.13     # 安装指定版本的 Python
$ uv sync                    # 在另一台电脑上：按 uv.lock 装出完全一样的环境
```

用 uv 的好处是：**不用手动激活虚拟环境**（`uv run` 会自动使用项目的环境），而且 `uv.lock` 记录了每个包的**精确版本**，别人（或者一年后的你）用 `uv sync` 就能得到一模一样的环境。

> [!TIP]
> 在中国大陆使用 uv，可以设置国内镜像：设置环境变量 `UV_DEFAULT_INDEX=https://pypi.tuna.tsinghua.edu.cn/simple`，或者在 `pyproject.toml` 里加入下面几行。

```toml
[[tool.uv.index]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

### `pyproject.toml`：项目的「身份证」

`uv init` 生成的 `pyproject.toml` 是 Python 项目的标准配置文件，大致是这样：

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "我的第一个项目"
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

`dependencies` 里的 `>=2.32` 表示「2.32 或更新的版本」。版本号一般是 `主版本.次版本.修订号`（**语义化版本**）：修订号变了只是修 bug；次版本变了是加功能，旧代码仍然能用；主版本变了可能有不兼容的改动。

### 一个典型的项目结构

```text
my-app/
├── pyproject.toml        项目配置和依赖
├── uv.lock               精确锁定的版本（交给 uv 管理，不要手动改）
├── README.md             项目说明
├── .gitignore            告诉 Git 不要提交 .venv、__pycache__ 等
├── src/
│   └── my_app/
│       ├── __init__.py   有了它，这个文件夹就是一个「包」
│       └── core.py
└── tests/
    └── test_core.py      测试代码
```

`.gitignore` 里至少应该写上 `.venv/` 和 `__pycache__/`：虚拟环境可以随时用 `uv sync` 重新生成，不需要也不应该提交到 Git（入门课第 7 课）。

## 15.4 ruff：代码风格与静态检查

Python 社区有一份代码风格指南 **PEP 8**：缩进 4 个空格、变量名用 `snake_case`（小写加下划线）、类名用 `CamelCase`、运算符两边加空格……统一的风格让大家的代码读起来都一样舒服。

**ruff**（也来自 Astral）可以**自动**做两件事：

```bash
$ uv run ruff format .       # 格式化：自动把代码改成统一的风格
$ uv run ruff check .        # 检查：找出可能的错误和不好的写法
$ uv run ruff check --fix .  # 能自动修的就自动修
```

例如这段代码：

```python norun
import os, sys
def Add(a,b):
    unused = 1
    if a == None:
        return b
    return a+b
```

`ruff check` 会指出：一行导入了多个模块、`os` 和 `sys` 导入了却没用、变量 `unused` 赋值后没用、`== None` 应该写成 `is None`；`ruff format` 会自动加上空格和空行。（函数名 `Add` 不符合 PEP 8 的命名规则，这一条需要在配置里开启 `N` 规则才会检查。）

> [!TIP]
> 在 VS Code 里安装 **Ruff** 扩展，保存文件时就会自动格式化，问题会直接在编辑器里标出来。

## 15.5 类型检查：在运行之前抓 bug

第 10 课说过，类型标注不会被 Python 强制检查。**类型检查器**会读你的代码和标注，**不运行程序**就找出类型错误：

```python norun
def total_price(prices: list[float]) -> float:
    return sum(prices)

print(total_price(["3.5", "2"]))      # 传入的是字符串列表
```

```bash
$ uv run mypy main.py
main.py:4: error: List item 0 has incompatible type "str"; expected "float"
```

常用的类型检查器有 **mypy**、**pyright**（VS Code 的 Pylance 扩展内置了它），以及 Astral 新推出的 **ty**。项目越大，类型检查越有用。

## 15.6 自动化测试：pytest

写完一个函数，你会手动试几个例子看看对不对。**自动化测试**就是把这些例子写成代码，以后每次修改都能一键全部重新检查，确保没有把原来好的地方改坏。

**pytest** 是最流行的测试工具，写法非常简单：文件名以 `test_` 开头，函数名以 `test_` 开头，里面用 `assert`：

```python norun
# 文件 src/my_app/core.py
def slugify(title: str) -> str:
    """把标题变成网址里用的形式：小写，空格换成连字符。"""
    return "-".join(title.lower().split())
```

```python norun
# 文件 tests/test_core.py
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

每个点 `.` 表示一个测试通过。如果有测试失败，pytest 会清楚地显示「期望什么、实际得到什么」。

学习程序和网页一次只运行一个文件，没法直接用 pytest。我们可以用标准库自带的 **unittest** 在一个文件里体验同样的思路：

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

unittest.main(argv=["tests"], exit=False, verbosity=2)    # 结果输出在错误输出（stderr）里
```

> [!NOTE]
> **先写测试**也是一种常见的做法（测试驱动开发，TDD）：先写出「我希望函数怎样工作」的测试，看着它失败，再写代码让它通过。本课程每道编程练习的自动判题，其实就是一组测试。

## 15.7 调试：`breakpoint()` 与 `logging`

除了到处加 `print`，还有更好的调试方法：

**1. 断点调试**：在代码里写一行 `breakpoint()`，程序运行到这里会暂停，进入调试器 `pdb`，你可以查看变量、一步一步执行：

```python norun
def average(nums):
    total = sum(nums)
    breakpoint()            # 运行到这里暂停；输入 p total 查看变量，n 执行下一行，c 继续，q 退出
    return total / len(nums)
```

VS Code、PyCharm 等编辑器有图形化的调试器：在行号左边点一下设置断点，按 F5 开始调试，更直观。

**2. 日志**：`logging` 模块比 `print` 更适合记录程序运行的情况——可以分级别（调试、信息、警告、错误）、带上时间，正式运行时只要改一个设置就能关掉调试信息：

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")
log = logging.getLogger("shop")

def buy(item, money):
    log.debug("准备购买 %s", item)            # 级别是 INFO，所以 DEBUG 信息不显示
    if money < 10:
        log.warning("钱不够买 %s：只有 %d 元", item, money)
        return False
    log.info("买到了 %s", item)
    return True

buy("书", 30)
buy("玩具", 5)
```

## 15.8 小结

- 第三方包在 PyPI 上；`uv add 包名` 或 `pip install 包名` 安装，然后 `import`。
- **每个项目一个虚拟环境**（`.venv`），避免版本冲突；不要把 `.venv` 提交到 Git。
- **uv**：`uv init` 建项目、`uv add` 加依赖、`uv run` 运行、`uv sync` 还原环境；依赖记录在 `pyproject.toml`，精确版本锁在 `uv.lock`。
- **ruff format** 统一风格（PEP 8），**ruff check** 找出问题；**mypy / pyright / ty** 做类型检查。
- **pytest**：`test_` 开头的文件和函数 + `assert`，`uv run pytest` 一键运行全部测试。
- 调试用 `breakpoint()` 或编辑器的调试器；记录运行情况用 `logging`。

## 15.9 练习

<!-- exercise: version-compare -->
### 练习 1：比较版本号

输入一行两个版本号（例如 `1.10.0 1.9.3`），输出**较新**的那个；如果一样新，输出 `equal`。版本号由若干个用 `.` 隔开的整数组成，要**按数字**比较（`1.10` 比 `1.9` 新），位数不同时缺少的部分当作 0（`1.2` 和 `1.2.0` 一样）。

```text
样例输入：1.10.0 1.9.3
样例输出：1.10.0
```

<details>
<summary>提示</summary>

把版本号变成整数列表：`[int(x) for x in v.split(".")]`，短的那个在后面补 0 到一样长，然后直接用 `>` 比较两个列表（Python 会逐个元素比较）。
</details>

<!-- exercise: snake-case -->
### 练习 2：改成 PEP 8 风格的名字

PEP 8 规定变量名和函数名用 `snake_case`。输入一个驼峰写法（camelCase）的名字，把每个大写字母变成「下划线 + 小写字母」后输出。（输入的第一个字母一定是小写。）

```text
样例输入：getUserName
样例输出：get_user_name
```

<details>
<summary>提示</summary>

遍历每个字符：`if ch.isupper(): result += "_" + ch.lower()`，否则直接加上。
</details>

<!-- exercise: fix-clamp -->
### 练习 3：让测试通过

起始代码里有一个函数 `clamp(x, lo, hi)`（把 `x` 限制在 `lo` 和 `hi` 之间：小于 `lo` 就返回 `lo`，大于 `hi` 就返回 `hi`，否则返回 `x`），还有几条用 `assert` 写的测试。但函数写错了，测试会失败。修好这个函数，让程序输出 `all tests passed`。**不要修改测试。**

```text
样例输出：all tests passed
```

<details>
<summary>提示</summary>

运行一下，看看是哪条 `assert` 失败了（Traceback 会指出行号），想想那个输入应该得到什么。正确的写法可以是 `return max(lo, min(x, hi))`。
</details>

<!-- exercise: why-venv -->
### 练习 4：为什么要用虚拟环境？

- 让 Python 运行得更快
- 让每个项目有自己独立的包和版本，互不冲突，也能清楚记录每个项目的依赖
- 没有虚拟环境就无法安装包
- 为了让代码更安全，防止被病毒感染

<details>
<summary>答案与解析</summary>

虚拟环境把每个项目的包隔离开：项目 A 用 1.0 版、项目 B 用 2.0 版也不会冲突；配合 `pyproject.toml` / `uv.lock`，别人能装出完全一样的环境。
</details>

<!-- exercise: tool-roles -->
### 练习 5：工具的分工

下面哪一项把工具和它的主要用途**全部**对应正确了？

- ruff：运行测试；pytest：格式化代码；uv：类型检查
- uv：管理依赖和虚拟环境；ruff：格式化和检查代码；pytest：运行测试
- uv：画图表；ruff：访问网络；pytest：安装 Python
- pytest：管理依赖；uv：检查代码风格；ruff：运行程序

<details>
<summary>答案与解析</summary>

uv 管项目、依赖、虚拟环境和 Python 版本；ruff 管代码风格和静态检查；pytest 管自动化测试。
</details>

## 15.10 延伸阅读

- [uv 官方文档](https://docs.astral.sh/uv/)、[Ruff 官方文档](https://docs.astral.sh/ruff/)
- [pytest 官方文档](https://docs.pytest.org/)
- [PEP 8 – Python 代码风格指南](https://peps.python.org/pep-0008/)（网上有中文翻译）
- [Python 打包用户指南](https://packaging.python.org/zh-cn/latest/)（有中文版）
- 下一课：[综合项目：命令行记账本](../16-project/zh-CN.md)
