# 14. 进阶特性

> 🎯 **本课目标**：掌握让 Python 代码更简洁、更「地道」（Pythonic）的一批工具：`enumerate`、`zip`、`any`、`all` 等内置函数；生成器表达式和 **`yield` 生成器**（省内存的「懒」计算）；`*args`、`**kwargs` 和星号解包；`match` 结构化模式匹配；**装饰器**和 `@cache`；海象运算符 `:=`；现代类型标注写法。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：第 8～13 课

这一课的内容不是「必须会才能写程序」的，但会大量出现在别人的代码、开源项目和面试题里。学会它们，你的代码会更短、更清楚，也能读懂更多别人写的 Python。

## 14.1 好用的内置函数

```python
names = ["小明", "小红", "小刚"]
scores = [88, 95, 72]

for i, name in enumerate(names, start=1):        # 同时拿到序号和元素（第 8 课见过）
    print(i, name)

for name, score in zip(names, scores):           # 把几个列表「拉链式」地配对
    print(f"{name}：{score}")

print(dict(zip(names, scores)))                  # 两个列表直接变成字典

nums = [3, -1, 4, 0, 5]
print(any(n < 0 for n in nums))                  # 有没有任何一个负数？True
print(all(n >= 0 for n in nums))                 # 是不是全都非负？False
print(sum(n for n in nums if n > 0))             # 正数之和：12
print(max(names, key=len), min(scores))          # 最长的名字、最低的分数
print(list(reversed(nums)), sorted(nums, reverse=True))
```

| 函数 | 作用 |
|------|------|
| `enumerate(seq, start=0)` | 得到 (序号, 元素) |
| `zip(a, b, ...)` | 把多个序列一一配对，按最短的结束 |
| `any(...)` / `all(...)` | 有任意一个为真 / 全部为真 |
| `sum`、`min`、`max` | 求和、最小、最大（`min`/`max` 可以用 `key=`） |
| `sorted`、`reversed` | 排序后的新列表、倒序遍历 |
| `map(f, seq)`、`filter(f, seq)` | 对每个元素用 f、筛选（通常用推导式代替） |

## 14.2 生成器表达式：不一次性造出整个列表

把列表推导式的方括号换成圆括号，就得到**生成器表达式**：它不会马上算出所有元素，而是**用到一个才算一个**：

```python
import sys

squares_list = [n * n for n in range(1_000_000)]     # 立刻生成 100 万个数，全部存起来
squares_gen = (n * n for n in range(1_000_000))      # 只是一个「随时可以算」的东西

print(sys.getsizeof(squares_list), "字节")           # 好几 MB
print(sys.getsizeof(squares_gen), "字节")            # 一两百字节
print(sum(squares_gen))                               # 边算边加，从不需要存 100 万个数
```

直接作为函数参数时，可以省掉一层括号：`sum(n * n for n in range(10))`。上一节的 `any(n < 0 for n in nums)` 也是生成器表达式。

> [!NOTE]
> 生成器只能**遍历一次**。遍历完再用，就什么都没有了。需要多次使用时，用列表。

## 14.3 `yield`：自己写生成器

函数里只要出现 `yield`，它就变成**生成器函数**：调用时不会马上执行，而是返回一个生成器；每次向它要下一个值，函数就运行到下一个 `yield`，交出一个值，然后**暂停**在那里，等着下次继续。

```python
def countdown(n):
    while n > 0:
        yield n           # 交出 n，然后在这里暂停
        n -= 1
    print("发射！")

for x in countdown(3):    # for 循环会自动不断地向生成器要下一个值
    print(x)
```

生成器很适合「一个接一个地产生数据」的场景：逐行读大文件、无穷数列、分批处理数据……

```python
def fibonacci():                     # 无穷的斐波那契数列：不会把内存撑爆，因为每次只算一个
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

for i, f in enumerate(fibonacci()):
    if i >= 10:
        break
    print(f, end=" ")
print()

def read_numbers(lines):             # 一条「流水线」：逐个清洗数据
    for line in lines:
        line = line.strip()
        if line and not line.startswith("#"):
            yield int(line)

data = ["10", "# 注释", "", "20", " 30 "]
print(sum(read_numbers(data)))       # 60
```

## 14.4 星号：`*args`、`**kwargs` 和解包

```python
def average(*nums):                  # *nums 把任意多个位置参数收集成一个元组
    return sum(nums) / len(nums)

print(average(1, 2, 3), average(10, 20))

def profile(name, **info):           # **info 把任意多个关键字参数收集成一个字典
    print(name, info)

profile("小明", age=12, city="北京")

scores = [90, 85, 77]
print(average(*scores))              # 调用时 * 把列表「拆开」成多个参数
options = {"sep": " | ", "end": "!\n"}
print("a", "b", "c", **options)      # ** 把字典拆成关键字参数

first, *rest = [1, 2, 3, 4]          # 解包时 * 收集「剩下的」
print(first, rest)
*init, last = "abcd"
print(init, last)
merged = [*scores, 100, *[1, 2]]     # 合并列表
both = {**{"a": 1}, **{"b": 2}}      # 合并字典（也可以写 d1 | d2）
print(merged, both)
```

## 14.5 `match`：结构化模式匹配

Python 3.10 加入了 `match` 语句。它不只是「多个 `if`」的简写，还能**按数据的结构**拆解和匹配：

```python
def handle(command: str) -> str:
    match command.split():
        case ["quit"]:
            return "再见"
        case ["say", *words]:                 # 第一个词是 say，剩下的都收进 words
            return " ".join(words)
        case ["move", x, y]:                  # 恰好三个词
            return f"移动到 ({int(x)}, {int(y)})"
        case ["move", *_]:
            return "move 需要两个坐标"
        case []:
            return "（空命令）"
        case _:                               # _ 匹配一切：相当于 else
            return f"不认识的命令：{command}"

for cmd in ["say 你好 世界", "move 3 4", "move 1", "", "quit", "jump"]:
    print(f"{cmd!r:12} → {handle(cmd)}")
```

`case` 还可以匹配常量、字典的结构、类的属性，并且用 `if` 加条件（`case [x, y] if x == y:`）。处理命令、解析 JSON 数据时，`match` 能让代码清楚很多。

## 14.6 装饰器：给函数「加功能」

第 13 课见过的 `@dataclass`、`@property` 都是**装饰器**（decorator）：写在函数或类定义上方的 `@名字`，它会把下面的函数「包装」一层，加上额外的功能。

函数是值（第 10 课），所以可以写一个「接收函数、返回新函数」的函数：

```python
import time
from functools import wraps

def timer(func):                             # 接收一个函数
    @wraps(func)                             # 保留原函数的名字和文档
    def wrapper(*args, **kwargs):            # 新函数：接收任意参数
        start = time.perf_counter()
        result = func(*args, **kwargs)       # 调用原来的函数
        print(f"{func.__name__} 用了 {time.perf_counter() - start:.4f} 秒")
        return result
    return wrapper                           # 返回新函数

@timer                                       # 相当于 slow_sum = timer(slow_sum)
def slow_sum(n):
    return sum(range(n))

print(slow_sum(1_000_000))
```

`@timer` 写在 `def` 上面，等价于定义完之后执行 `slow_sum = timer(slow_sum)`。之后调用 `slow_sum` 实际上调用的是 `wrapper`，它在原函数前后加上了计时。

### 实用的内置装饰器：`@cache`

上一课的递归斐波那契，`n` 稍大就会慢得惊人，因为同样的值被重复算了无数次。`functools.cache` 会**记住**每次调用的结果，下次同样的参数直接返回：

```python
from functools import cache

@cache
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(100))      # 瞬间算完；不加 @cache 的话，算到宇宙毁灭也算不完
```

这种「记住算过的结果」的技巧叫**记忆化**（memoization），是动态规划的基础。

## 14.7 海象运算符 `:=`

`:=` 可以在**表达式中间**给变量赋值（长得像海象的眼睛和牙齿，所以叫海象运算符）：

```python
import re

text = "订单号 A1234，金额 56 元"
if (m := re.search(r"\d+", text)) is not None:     # 先赋值给 m，再判断
    print("找到第一个数字：", m.group())

data = [3, 8, 1, 9, 4]
big = [y for x in data if (y := x * 10) > 40]       # 推导式里也能用
print(big)
```

它能省掉一些重复计算和临时变量，但别用得太多，以免难读。

## 14.8 现代类型标注

Python 3.9 起，可以直接用内置类型写容器的类型；3.10 起，可以用 `|` 表示「或」：

```python
def word_lengths(words: list[str]) -> dict[str, int]:
    return {w: len(w) for w in words}

def find(names: list[str], target: str) -> int | None:     # 找到返回下标，找不到返回 None
    return names.index(target) if target in names else None

print(word_lengths(["hi", "hello"]))
print(find(["a", "b"], "b"), find(["a", "b"], "z"))
```

| 写法 | 含义 |
|------|------|
| `list[int]` | 整数列表 |
| `dict[str, float]` | 键是字符串、值是小数的字典 |
| `tuple[int, str]` | 第一个是整数、第二个是字符串的元组 |
| `int \| None` | 整数或 `None` |
| `Callable[[int], str]`（来自 `collections.abc`） | 接收一个整数、返回字符串的函数 |

## 14.9 小结

- `enumerate`、`zip`、`any`、`all`、`sum/min/max(key=)`、`sorted`、`reversed` 让循环更简洁。
- 生成器表达式 `(x for x in ...)` 和 `yield` 生成器「用到才算」，省内存，可以表示无穷序列；只能遍历一次。
- `*args` 收集位置参数、`**kwargs` 收集关键字参数；调用时 `*`、`**` 拆开列表和字典；`first, *rest = ...` 解包。
- `match / case` 按结构匹配数据，`_` 匹配一切。
- 装饰器 `@deco` 等价于 `f = deco(f)`；`@cache` 记住函数结果。
- `:=` 在表达式中赋值；类型标注可以写 `list[int]`、`dict[str, int]`、`int | None`。

## 14.10 练习

<!-- exercise: evens-gen -->
### 练习 1：写一个生成器

补全生成器函数 `evens(n)`：依次产生 0 到 `n`（包括 `n`）之间的所有偶数。程序读入 `n`，把生成的数用空格连起来输出。

```text
样例输入：9
样例输出：0 2 4 6 8
```

<details>
<summary>提示</summary>

`for i in range(0, n + 1, 2): yield i`
</details>

<!-- exercise: star-average -->
### 练习 2：任意多个参数

补全函数 `stats(*nums)`，返回一个元组 `(最小值, 最大值, 平均值)`。程序读入一行整数，用 `stats(*nums)` 调用它，输出最小值、最大值和平均值（平均值保留 2 位小数）。

```text
样例输入：4 8 15 16 23 42
样例输出：4 42 18.00
```

<details>
<summary>提示</summary>

在函数里 `nums` 是一个元组：`return min(nums), max(nums), sum(nums) / len(nums)`。
</details>

<!-- exercise: match-commands -->
### 练习 3：用 match 处理命令

用 `match` 处理若干条命令，直到遇到 `quit`：

- `add a b`：输出 `a + b`；
- `mul a b`：输出 `a * b`；
- `neg a`：输出 `-a`；
- 其他命令：输出 `unknown`；
- `quit`：结束程序（不输出任何东西）。

所有数都是整数。

```text
样例输入：
add 2 3
neg 7
pow 2 3
mul 4 5
quit
样例输出：
5
-7
unknown
20
```

<details>
<summary>提示</summary>

`match input().split():`，然后 `case ["add", a, b]: print(int(a) + int(b))`……最后 `case _: print("unknown")`。`quit` 用 `break` 跳出循环。
</details>

<!-- exercise: big-fib -->
### 练习 4：记忆化

输入 `n`（0 ≤ n ≤ 300），输出第 `n` 个斐波那契数（`fib(0) = 0`，`fib(1) = 1`）。要求用**递归 + `@cache`** 实现。

```text
样例输入：90
样例输出：2880067194370816120
```

<details>
<summary>提示</summary>

`from functools import cache`，在递归函数上加 `@cache`。`n` 到几百时递归深度也只有几百层，在 Python 默认的上限（约 1000 层）以内。
</details>

<!-- exercise: gen-predict -->
### 练习 5：读代码，写输出

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
<summary>答案与解析</summary>

调用 `gen()` 时函数体**还没有开始执行**，所以先打印 `created`。第一次 `next(g)` 运行到第一个 `yield`：打印 `start`，交出 1。第二次从暂停处继续：打印 `middle`，交出 2。输出依次是 `created`、`start`、`1`、`middle`、`2`。
</details>

<!-- exercise: decorator-meaning -->
### 练习 6：装饰器等价于什么？

下面的写法等价于哪一句？

```python norun
@timer
def work():
    ...
```

- `work = timer`
- `work = timer(work)`
- `timer = work(timer)`
- `work()` 定义完立刻运行一次

<details>
<summary>答案与解析</summary>

`@timer` 等价于在定义之后写 `work = timer(work)`：用 `timer` 返回的新函数替换原来的 `work`。
</details>

## 14.11 延伸阅读

- [Python 官方教程 · 9.9 生成器](https://docs.python.org/zh-cn/3/tutorial/classes.html#generators)
- [PEP 636：结构化模式匹配教程](https://peps.python.org/pep-0636/)
- [functools 文档](https://docs.python.org/zh-cn/3/library/functools.html)、[itertools 文档](https://docs.python.org/zh-cn/3/library/itertools.html)
- 下一课：[包与现代工具链](../15-packages-tooling/zh-CN.md)
