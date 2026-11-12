# 3. 数字与计算

> 🎯 **本课目标**：掌握 Python 的全部算术运算符（`+ - * / // % **`）和运算优先级；用 `%` 和 `//` 解决「换算」类问题；会用 `round`、`abs`、`max`、`min` 和 `math` 模块；用 f 字符串漂亮地显示数字。
>
> ⏱ **预计用时**：40 分钟　｜　📋 **前置知识**：第 2 课

## 3.1 算术运算符

| 运算符 | 含义 | 例子 | 结果 |
|--------|------|------|------|
| `+` | 加 | `7 + 2` | `9` |
| `-` | 减 | `7 - 2` | `5` |
| `*` | 乘 | `7 * 2` | `14` |
| `/` | 除 | `7 / 2` | `3.5` |
| `//` | 整除（向下取整） | `7 // 2` | `3` |
| `%` | 取余数（模） | `7 % 2` | `1` |
| `**` | 乘方 | `7 ** 2` | `49` |

```python
print(7 + 2, 7 - 2, 7 * 2)
print(7 / 2, 7 // 2, 7 % 2)
print(2 ** 10)
```

### `/` 的结果总是小数

```python
print(6 / 2)     # 3.0，不是 3
```

即使能整除，`/` 也会得到 `float`。如果需要整数结果，用 `//`。

### 整数和小数混合运算

只要有一个数是 `float`，结果就是 `float`：

```python
print(3 + 2.0)    # 5.0
print(2 * 1.5)    # 3.0
```

## 3.2 整除 `//` 和取余 `%`

小学除法：7 ÷ 2 = 3 …… 1。在 Python 里，**商**用 `//`，**余数**用 `%`：

```text
7 // 2 = 3    （商）
7 %  2 = 1    （余数）
验证：3 × 2 + 1 = 7
```

这两个运算符非常有用：

```python
n = 17
print(n % 2)        # 1 → 奇数（偶数除以 2 余 0）
print(n % 10)       # 7 → 个位数
print(n // 10)      # 1 → 去掉个位数后的部分
```

### 典型应用：时间换算

把 3725 秒换算成「几小时几分几秒」：

```python
total = 3725
hours = total // 3600            # 一小时 3600 秒
minutes = total % 3600 // 60     # 剩下的秒数再换成分钟
seconds = total % 60
print(hours, "小时", minutes, "分", seconds, "秒")
```

输出：`1 小时 2 分 5 秒`。

`divmod(a, b)` 可以一次得到商和余数：

```python
q, r = divmod(3725, 3600)
print(q, r)      # 1 125
```

> [!NOTE]
> 负数的 `//` 是**向下取整**（往更小的方向）：`-7 // 2` 是 `-4`，不是 `-3`；相应地 `-7 % 2` 是 `1`。日常使用中一般遇不到，知道有这回事即可。其他语言（C、Java）对负数的处理和 Python 不同，学到时会提醒。

## 3.3 运算优先级

和数学一样，**先乘除后加减**，**括号最优先**：

```python
print(2 + 3 * 4)      # 14，先算 3 * 4
print((2 + 3) * 4)    # 20，先算括号
```

完整的优先级（从高到低）：

```text
1. ( )                括号
2. **                 乘方
3. -x                 负号
4. *  /  //  %        乘、除、整除、取余
5. +  -               加、减
```

同一级别从左往右算：`10 - 3 - 2` 是 `(10 - 3) - 2 = 5`。唯一的例外是 `**` 从右往左：`2 ** 3 ** 2` 是 `2 ** 9 = 512`。

> [!TIP]
> 拿不准的时候**加括号**。多余的括号不会出错，还能让代码更清楚：`average = (a + b + c) / 3`。

一个容易出错的地方：

```python
print(-2 ** 2)     # -4：先算 2 ** 2，再取负
print((-2) ** 2)   # 4
```

## 3.4 常用的数学函数

不需要导入就能用的**内置函数**：

```python
print(abs(-5))             # 5：绝对值
print(max(3, 8, 1))        # 8：最大值
print(min(3, 8, 1))        # 1：最小值
print(round(3.14159, 2))   # 3.14：保留 2 位小数
print(round(7.6))          # 8：四舍五入到整数
print(pow(2, 5))           # 32：等同于 2 ** 5
```

### `round` 的一个小坑

```python
print(round(2.5))   # 2
print(round(3.5))   # 4
```

Python 的 `round` 在正好一半（.5）时，会取**最近的偶数**（叫「银行家舍入」），这样大量数据统计时误差更小。如果你需要传统的「四舍五入」，后面学 `decimal` 模块时可以精确控制。

## 3.5 math 模块

更多数学功能放在 **`math` 模块**里。**模块**就是一个装满工具的「工具箱」，用 `import` 把它拿进来：

```python
import math

print(math.sqrt(16))       # 4.0：平方根
print(math.pi)             # 3.141592653589793
print(math.floor(3.7))     # 3：向下取整
print(math.ceil(3.2))      # 4：向上取整
print(math.gcd(12, 18))    # 6：最大公约数
print(math.factorial(5))   # 120：5 的阶乘 5×4×3×2×1
print(math.hypot(3, 4))    # 5.0：直角三角形斜边
```

`import math` 一般写在文件的最上面。使用时要写 `math.` 加函数名。

例：计算圆的面积和周长：

```python
import math

r = 5
area = math.pi * r ** 2
circumference = 2 * math.pi * r
print("面积：", area)
print("周长：", circumference)
```

## 3.6 把数字显示得漂亮一点

上面的结果小数位太长了。用 f 字符串的**格式说明**来控制：

```python
import math

r = 5
area = math.pi * r ** 2
print(f"面积：{area:.2f}")           # 保留 2 位小数：78.54
print(f"人口：{1400000000:,}")       # 千位分隔符：1,400,000,000
print(f"占比：{0.256:.1%}")          # 百分比：25.6%
print(f"编号：{7:03d}")              # 补零到 3 位：007
print(f"[{42:>6}]")                  # 右对齐，宽度 6：[    42]
print(f"[{42:<6}]")                  # 左对齐：[42    ]
```

| 格式 | 含义 | 例子 |
|------|------|------|
| `:.2f` | 保留 2 位小数 | `3.14159` → `3.14` |
| `:,` | 千位分隔 | `1234567` → `1,234,567` |
| `:.1%` | 百分比，1 位小数 | `0.256` → `25.6%` |
| `:03d` | 整数，宽度 3，前面补 0 | `7` → `007` |
| `:>6` / `:<6` / `:^6` | 宽度 6，右/左/居中对齐 | |

> [!TIP]
> `:.2f` 只是**显示**时保留两位，变量本身的值没有改变。要真正改变值，用 `round()`。

## 3.7 一个完整的例子：BMI 计算器

```python
weight = 68      # 体重，千克
height = 1.75    # 身高，米

bmi = weight / height ** 2
print(f"你的 BMI 是 {bmi:.1f}")
```

输出：`你的 BMI 是 22.2`。注意 `height ** 2` 会先算（乘方优先级高于除法）。

## 3.8 常见错误

```python norun
print(10 / 0)
```

```text
ZeroDivisionError: division by zero
```

除数不能为 0（`//` 和 `%` 也一样）。以后学了 `if`，可以先判断除数是不是 0。

```python norun
print(math.sqrt(9))
```

```text
NameError: name 'math' is not defined
```

忘了写 `import math`。

```python norun
x = 5
print(2x)
```

```text
SyntaxError: invalid decimal literal
```

数学里 `2x` 表示 2 乘 x，但在 Python 里乘号**必须写出来**：`2 * x`。

## 3.9 小结

- 运算符：`+ - * /`，`//` 整除，`%` 取余，`**` 乘方；`/` 的结果总是 `float`。
- `//` 和 `%` 常用于判断奇偶、取各位数字、时间和单位换算；`divmod` 同时得到商和余数。
- 优先级：括号 > `**` > 负号 > `* / // %` > `+ -`；拿不准就加括号。
- 内置函数 `abs`、`max`、`min`、`round`、`pow`；`round` 对 .5 取最近的偶数。
- `import math` 后使用 `math.sqrt`、`math.pi`、`math.floor`、`math.ceil` 等。
- f 字符串格式：`:.2f`、`:,`、`:.1%`、`:03d`、`:>6`。

## 3.10 练习

<!-- exercise: time-convert -->
### 练习 1：秒数换算

变量 `total = 7384` 表示秒数。计算它是几小时、几分、几秒，用**一个** `print` 输出这三个数字（用空格隔开）：

```text
2 3 4
```

<details>
<summary>提示 1</summary>

小时：`total // 3600`；秒：`total % 60`。
</details>

<details>
<summary>提示 2</summary>

分钟：先用 `total % 3600` 得到「不满一小时的秒数」，再 `// 60`。
</details>

<!-- exercise: circle -->
### 练习 2：圆的面积

半径 `r = 3`。使用 `math.pi` 计算圆的面积，保留两位小数输出：

```text
28.27
```

<details>
<summary>提示</summary>

`import math`，面积是 `math.pi * r ** 2`，用 `print(f"{area:.2f}")` 输出。
</details>

<!-- exercise: digits -->
### 练习 3：拆分数字

`n = 472`。用 `//` 和 `%` 分别求出它的百位、十位和个位，输出：

```text
4 7 2
```

<details>
<summary>提示</summary>

个位 `n % 10`；十位 `n // 10 % 10`；百位 `n // 100`。
</details>

<!-- exercise: predict-ops -->
### 练习 4：读代码，写输出

```python
print(17 // 5)
print(17 % 5)
print(2 + 3 * 2 ** 2)
print(10 / 4)
```

<details>
<summary>答案与解析</summary>

`17 // 5 = 3`，`17 % 5 = 2`；`2 ** 2 = 4`，`3 * 4 = 12`，`2 + 12 = 14`；`10 / 4 = 2.5`。
</details>

<!-- exercise: precedence -->
### 练习 5：平均数

想计算 `a`、`b`、`c` 三个数的平均数，哪一种写法是正确的？

- `a + b + c / 3`
- `(a + b + c) / 3`
- `a + b + (c / 3)`
- `(a + b) + c / 3`

<details>
<summary>答案与解析</summary>

除法优先级比加法高，所以必须先用括号把三个数加起来。只有 `(a + b + c) / 3` 是对的。
</details>

## 3.11 延伸阅读

- [Python 官方文档 · math 模块](https://docs.python.org/zh-cn/3/library/math.html)
- [格式规格迷你语言](https://docs.python.org/zh-cn/3/library/string.html#format-specification-mini-language)：f 字符串冒号后面所有写法的完整说明。
- 下一课：[字符串](../04-strings/zh-CN.md)
