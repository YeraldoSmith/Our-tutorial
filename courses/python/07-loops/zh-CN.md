# 7. 循环

> 🎯 **本课目标**：用 `while` 和 `for` 让代码重复执行；熟练使用 `range()`；掌握「累加」「计数」「找最大值」等循环套路；用 `break`、`continue` 控制循环；写出嵌套循环（如九九乘法表）；避开死循环和「差一」错误。
>
> ⏱ **预计用时**：60 分钟　｜　📋 **前置知识**：第 6 课

## 7.1 为什么需要循环？

假设要打印 1 到 5：

```python
print(1)
print(2)
print(3)
print(4)
print(5)
```

那 1 到 1000 呢？复制 1000 行显然不现实。**循环**（loop）让同一段代码**重复执行**很多次。电脑最擅长的，就是不知疲倦地重复。

## 7.2 while 循环：条件成立就一直做

```python
i = 1
while i <= 5:
    print(i)
    i += 1
print("结束")
```

执行过程：

```text
i = 1 → 1 <= 5 成立 → 打印 1，i 变成 2
i = 2 → 2 <= 5 成立 → 打印 2，i 变成 3
  ……
i = 5 → 5 <= 5 成立 → 打印 5，i 变成 6
i = 6 → 6 <= 5 不成立 → 离开循环，打印「结束」
```

`while` 的格式和 `if` 很像：`while 条件:` 加一个缩进的代码块。区别是：`if` 只执行一次，`while` 会**一直重复，直到条件不成立**。

### 当心死循环

```python norun
i = 1
while i <= 5:
    print(i)
    # 忘了写 i += 1
```

`i` 永远是 1，条件永远成立，程序会**永远**打印 1，停不下来——这叫**死循环**。在学习程序里点「■ 停止」，在终端里按 <kbd>Ctrl</kbd>+<kbd>C</kbd>。

> [!TIP]
> 写 `while` 循环时，问自己三个问题：① 循环变量从多少开始？② 什么条件下继续？③ 每次循环里，循环变量**有没有朝着结束的方向变化**？

### while 适合「不知道要循环多少次」的情况

比如：一直让用户输入，直到输入正确为止：

```python input="abc\npython"
password = input()
while password != "python":
    print("密码错误，请重试")
    password = input()
print("登录成功！")
```

## 7.3 for 循环：逐个取出

`for` 循环用来**依次处理一组东西中的每一个**：

```python
for ch in "你好呀":
    print(ch)
```

输出三行：`你`、`好`、`呀`。每次循环，变量 `ch` 依次得到字符串中的下一个字符。

### range()：生成一串数字

`for` 最常和 `range()` 搭配，用来「循环 n 次」或「遍历一段数字」：

```python
for i in range(5):
    print(i)
```

输出 `0 1 2 3 4`（每个一行）。注意：**`range(5)` 是 0 到 4，不包括 5**——和切片一样「包含开始，不包含结束」。

`range` 的三种写法：

| 写法 | 生成的数字 |
|------|------------|
| `range(5)` | 0, 1, 2, 3, 4 |
| `range(1, 6)` | 1, 2, 3, 4, 5 |
| `range(0, 10, 2)` | 0, 2, 4, 6, 8（步长 2） |
| `range(10, 0, -1)` | 10, 9, 8, …, 1（倒数） |

```python
for i in range(1, 6):
    print(f"第 {i} 次")
```

> [!NOTE]
> `for i in range(1, 6)` 和 7.2 节的 `while` 例子做的事完全一样，但更短、更不容易出错（不会忘记 `i += 1`）。**知道循环次数时，优先用 `for`。**

## 7.4 循环的经典套路

### 套路 1：累加

计算 1 + 2 + … + 100：

```python
total = 0                  # ① 循环前准备一个「装结果的变量」，初始为 0
for i in range(1, 101):
    total += i             # ② 每次把当前的数加进去
print(total)               # ③ 循环结束后使用结果：5050
```

### 套路 2：累乘

计算 5 的阶乘 5! = 1 × 2 × 3 × 4 × 5：

```python
result = 1                 # 乘法的初始值是 1，不是 0！
for i in range(1, 6):
    result *= i
print(result)              # 120
```

### 套路 3：计数

数一数句子里有多少个字母 `o`：

```python
text = "hello world, good morning"
count = 0
for ch in text:
    if ch == "o":
        count += 1
print(count)               # 5
```

### 套路 4：找最大值

读入 5 个数，找出最大的：

```python
biggest = int(input())      # 先把第一个数当作当前最大
for _ in range(4):          # 再读剩下 4 个
    n = int(input())
    if n > biggest:
        biggest = n
print(biggest)
```

当循环变量用不到时，习惯上把它命名为 `_`。

### 套路 5：读入 n 个数

很多题目先给出数量 n，再给出 n 个数：

```python
n = int(input())
total = 0
for _ in range(n):
    total += int(input())
print(total)
```

## 7.5 break 和 continue

- **`break`**：立刻**跳出整个循环**。
- **`continue`**：跳过本次循环剩下的代码，**直接开始下一次**。

```python
for i in range(1, 11):
    if i == 6:
        break           # 到 6 就停
    print(i, end=" ")
print()
# 输出：1 2 3 4 5

for i in range(1, 11):
    if i % 2 == 0:
        continue        # 跳过偶数
    print(i, end=" ")
print()
# 输出：1 3 5 7 9
```

### while True + break：「先做，满足条件再退出」

```python input="hello\nquit"
while True:
    cmd = input()
    if cmd == "quit":
        break
    print(f"你输入了：{cmd}")
print("程序结束")
```

`while True` 表示「一直循环」，靠里面的 `break` 来结束。这是交互式程序最常用的结构。

### 找第一个满足条件的数

```python
n = 100
while True:
    n += 1
    if n % 7 == 0:
        break
print("大于 100 的第一个 7 的倍数：", n)    # 105
```

## 7.6 嵌套循环

循环里面套循环：外层每走一次，内层要完整地走一遍。

```python
for row in range(1, 4):
    for col in range(1, 5):
        print(f"({row},{col})", end=" ")
    print()          # 每行结束后换行
```

```text
(1,1) (1,2) (1,3) (1,4)
(2,1) (2,2) (2,3) (2,4)
(3,1) (3,2) (3,3) (3,4)
```

### 九九乘法表

```python
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f"{j}×{i}={i * j}", end="\t")
    print()
```

内层循环的范围 `range(1, i + 1)` 依赖外层的 `i`：第 1 行 1 个，第 2 行 2 个……所以是个三角形。`\t` 是制表符，让结果对齐。

### 打印图形

```python
n = 4
for i in range(1, n + 1):
    print("*" * i)
```

```text
*
**
***
****
```

用字符串乘法 `"*" * i`，一行就能画出 i 个星号。

## 7.7 for…else（了解即可）

循环后面可以接 `else`：**循环正常结束（没有被 `break` 打断）时**执行。常用于「搜索」：

```python
n = 29
for d in range(2, n):
    if n % d == 0:
        print(n, "不是质数，能被", d, "整除")
        break
else:
    print(n, "是质数")
```

## 7.8 常见错误

### 差一错误（off-by-one）

想打印 1 到 10，却写成：

```python
for i in range(1, 10):
    print(i)      # 只打印到 9！
```

`range` 不包括结束值，应该写 `range(1, 11)`。这类「差一个」的错误在所有语言里都很常见，叫 **off-by-one 错误**。检查循环时，**特别留意第一次和最后一次**。

### 累加变量放错位置

```python
for i in range(1, 4):
    total = 0          # ❌ 每次循环都被清零
    total += i
print(total)           # 3，而不是 6
```

初始化要放在循环**前面**。

### 死循环

`while` 条件里的变量在循环中从不改变，或者改变的方向错了（比如该 `i += 1` 却写成 `i -= 1`）。发现程序「卡住」不出结果时，首先怀疑它。学习程序检查答案时，超过时间限制会提示「运行超时」。

## 7.9 小结

- `while 条件:` 条件成立就一直重复；要确保循环变量会让条件最终变假，否则死循环。
- `for 变量 in 一组东西:` 逐个处理；`range(开始, 结束, 步长)` 不包含结束值。
- 知道次数用 `for`，不知道次数（等到某件事发生）用 `while`。
- 套路：累加（初值 0）、累乘（初值 1）、计数、找最大值、读入 n 个数。
- `break` 跳出循环，`continue` 跳到下一次；`while True` + `break` 是交互程序的常用结构。
- 嵌套循环：外层一次，内层一整轮；小心 off-by-one。

## 7.10 练习

<!-- exercise: sum-to-n -->
### 练习 1：1 到 n 的和

输入正整数 n，输出 1 + 2 + … + n 的结果。

```text
样例输入：100
样例输出：5050
```

<details>
<summary>提示</summary>

`total = 0`，然后 `for i in range(1, n + 1): total += i`。
</details>

<!-- exercise: factorial -->
### 练习 2：阶乘

输入非负整数 n（0 ≤ n ≤ 20），输出 n 的阶乘 n!。规定 0! = 1。

```text
样例输入：5
样例输出：120
```

<details>
<summary>提示</summary>

`result = 1`，循环乘上 1 到 n。当 n = 0 时循环一次都不执行，结果正好是 1。
</details>

<!-- exercise: fizzbuzz -->
### 练习 3：FizzBuzz

输入正整数 n，依次输出 1 到 n，每个数一行。但是：3 的倍数输出 `Fizz`，5 的倍数输出 `Buzz`，既是 3 又是 5 的倍数输出 `FizzBuzz`。

```text
样例输入：5
样例输出：
1
2
Fizz
4
Buzz
```

<details>
<summary>提示</summary>

先判断「既是 3 又是 5 的倍数」（也就是 15 的倍数），再判断 3 和 5，顺序很重要。
</details>

<!-- exercise: count-digits -->
### 练习 4：数位数

输入一个正整数，**用 while 循环**（不许用 `len`）计算它有几位数。

```text
样例输入：12345
样例输出：5
```

<details>
<summary>提示</summary>

每次 `n //= 10` 去掉最后一位，同时计数加 1，直到 `n` 变成 0。
</details>

<!-- exercise: triangle -->
### 练习 5：数字三角形

输入正整数 n，输出 n 行，第 i 行是从 1 到 i 的数字，用空格隔开。

```text
样例输入：4
样例输出：
1
1 2
1 2 3
1 2 3 4
```

<details>
<summary>提示 1</summary>

外层循环 `for i in range(1, n + 1)`，内层循环打印 1 到 i。
</details>

<details>
<summary>提示 2</summary>

为了让行末没有多余空格，可以把每行的数字转成字符串，再用 `" ".join(...)`；或者内层用 `print(j, end=" ")`——判题时会忽略行尾空格，所以也可以。
</details>

<!-- exercise: predict-loop -->
### 练习 6：读代码，写输出

```python
total = 0
for i in range(10, 0, -3):
    total += i
    print(i, end=" ")
print()
print(total)
```

<details>
<summary>答案与解析</summary>

`range(10, 0, -3)` 生成 10、7、4、1（下一个 -2 小于结束值 0，停止）。和是 10 + 7 + 4 + 1 = 22。第一行末尾有一个空格，判题时不影响。
</details>

<!-- exercise: range-len -->
### 练习 7：range 有几个数

`range(3, 15, 3)` 一共会生成几个数字？

- 3 个
- 4 个
- 5 个
- 12 个

<details>
<summary>答案与解析</summary>

3、6、9、12，共 4 个。15 是结束值，不包含在内。
</details>

## 7.11 延伸阅读

- [Python 官方教程 · 4.2 for 语句、4.3 range() 函数](https://docs.python.org/zh-cn/3/tutorial/controlflow.html#for-statements)
- [Python Tutor](https://pythontutor.com/)：把代码一步步可视化执行，非常适合观察循环里变量是怎么变化的。
- 下一课：[列表与元组](../08-lists-tuples/zh-CN.md)
