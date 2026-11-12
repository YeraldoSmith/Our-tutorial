# 13. 类与对象

> 🎯 **本课目标**：理解「对象 = 数据 + 能对它做的操作」；用 `class` 定义自己的类型，写出 `__init__`、方法和 `self`；用 `__str__` 让对象打印得好看；理解类属性和实例属性；用**继承**复用代码；用现代 Python 的 **`@dataclass`** 少写很多样板代码；认识 `@property` 和 `__eq__`、`__lt__`、`__add__` 等特殊方法。
>
> ⏱ **预计用时**：90 分钟　｜　📋 **前置知识**：第 9、10、12 课

## 13.1 你其实一直在用对象

```python
name = "hello"
print(name.upper())          # 字符串「对象」自带 upper 方法
nums = [3, 1, 2]
nums.sort()                  # 列表对象自带 sort 方法
print(nums, type(nums))
```

在 Python 里，**一切都是对象**：字符串、列表、字典、函数……每个对象都有自己的**数据**（字符串里的字符、列表里的元素），以及可以对它做的**操作**（`upper`、`sort`、`append`），这些操作叫**方法**（method）。

`str`、`list` 这些是 Python 自带的**类型**，也叫**类**（class）。这一课我们学习定义**自己的类**：比如一个游戏里的「角色」，有名字、血量，还能攻击、受伤。

## 13.2 定义一个类

```python
class Dog:
    def __init__(self, name, age):     # 创建对象时自动调用：给新对象设置初始数据
        self.name = name               # self.name 是这个对象自己的「属性」
        self.age = age

    def bark(self):                    # 方法：属于这个类的函数，第一个参数总是 self
        print(f"{self.name}：汪汪！")

    def birthday(self):
        self.age += 1                  # 方法可以修改对象自己的数据
        print(f"{self.name} {self.age} 岁啦")

a = Dog("旺财", 3)                     # 用类创建一个对象（也叫「实例」）
b = Dog("小白", 1)
a.bark()
b.bark()
a.birthday()
print(a.age, b.age)                    # 两只狗的数据互不影响：4 1
```

逐个解释：

- **`class Dog:`** 定义一个名叫 `Dog` 的类。类名习惯用**大写字母开头**的驼峰写法（`BankAccount`、`GameCharacter`）。
- **类是图纸，对象是按图纸造出来的东西**：`Dog` 是「狗」的设计图，`a` 和 `b` 是两只具体的狗。
- **`__init__`**（前后各两个下划线）是**初始化方法**：`Dog("旺财", 3)` 会创建一个新对象，并自动调用 `__init__(新对象, "旺财", 3)`。
- **`self`** 就是「这个对象自己」。`a.bark()` 实际上相当于 `Dog.bark(a)`，所以在 `bark` 里，`self` 就是 `a`。定义方法时第一个参数必须写 `self`，调用时不用传。
- **`self.name = name`**：把参数 `name` 存进对象的属性 `self.name`，这样对象在别的方法里也能用到它。

## 13.3 让对象打印得好看：`__str__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)                     # <__main__.Point object at 0x...>：看不出内容
```

定义 `__str__` 方法，告诉 Python 打印这个对象时显示什么：

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"({self.x}, {self.y})"

    def distance_to_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p)                             # (3, 4)
print("距离原点", p.distance_to_origin())
```

## 13.4 一个完整的例子：银行账户

把数据（余额）和规则（不能透支）放在一起，正是类最擅长的：

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("存款金额必须大于 0")
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError(f"余额不足：只有 {self.balance} 元")
        self.balance -= amount

    def __str__(self):
        return f"{self.owner} 的账户，余额 {self.balance} 元"

acc = BankAccount("小明", 100)
acc.deposit(50)
print(acc)
try:
    acc.withdraw(500)
except ValueError as e:
    print("取款失败：", e)
acc.withdraw(30)
print(acc)
```

使用这个类的人不需要知道余额是怎么存的，只要调用 `deposit`、`withdraw`；而「不能透支」的规则写在类里，不会被忘记。这叫**封装**（encapsulation）。

## 13.5 类属性和实例属性

- **实例属性**：`self.xxx`，每个对象各有一份（每只狗有自己的名字）；
- **类属性**：直接写在 `class` 里面，所有对象**共享**一份（所有狗都是「犬科」）。

```python
class Dog:
    species = "犬科"              # 类属性：所有 Dog 共享
    count = 0

    def __init__(self, name):
        self.name = name          # 实例属性：每只狗一份
        Dog.count += 1            # 修改类属性要通过类名

a, b = Dog("旺财"), Dog("小白")
print(a.species, b.species)       # 犬科 犬科
print("一共创建了", Dog.count, "只狗")
```

## 13.6 继承：在已有的类上扩展

很多类有共同点：狗和猫都是动物，都有名字、都会叫，只是叫声不同。**继承**（inheritance）让新类自动获得旧类的所有属性和方法，再添加或修改不同的部分：

```python
class Animal:                              # 父类（基类）
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "……"

    def introduce(self):
        return f"我是{self.name}，我会说：{self.speak()}"

class Dog(Animal):                         # Dog 继承 Animal
    def speak(self):                       # 重写（override）父类的方法
        return "汪汪"

class Cat(Animal):
    def __init__(self, name, indoor):
        super().__init__(name)             # 先调用父类的 __init__，设置好 name
        self.indoor = indoor               # 再添加自己特有的属性

    def speak(self):
        return "喵～"

for pet in [Dog("旺财"), Cat("咪咪", indoor=True), Animal("某种动物")]:
    print(pet.introduce())
```

- `class Dog(Animal):` 括号里写父类；
- 子类没写的方法（`introduce`）直接用父类的；子类写了同名方法（`speak`），就用子类自己的；
- `super().__init__(...)` 调用父类的方法，避免重复写代码；
- 同一句 `pet.introduce()`，对不同的对象做出不同的事——这叫**多态**（polymorphism）。

> [!TIP]
> 继承很有用，但不要滥用。如果两个东西是「A **是一种** B」（狗是一种动物），用继承；如果是「A **有一个** B」（汽车有一个发动机），就把 B 作为 A 的一个属性（这叫**组合**）。实践中，组合往往比层层继承更简单清楚。

## 13.7 现代写法：`@dataclass`

很多类只是用来**装数据**的，每次都要写 `__init__`、`__str__`、比较大小……很啰嗦。Python 3.7 起的 **`dataclasses`** 能自动生成这些：

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    score: int
    grade: int = 1                    # 可以有默认值

a = Student("小明", 88)
b = Student("小红", 95, grade=2)
print(a)                              # 自动生成好看的打印：Student(name='小明', score=88, grade=1)
print(a == Student("小明", 88))       # 自动生成「相等」比较：True

students = [a, b, Student("小刚", 88)]
students.sort(key=lambda s: (-s.score, s.name))      # 按分数从高到低，同分按名字
print([s.name for s in students])
```

`@dataclass` 写在类定义的上面，叫**装饰器**（decorator）。只要写出字段名和类型标注，`__init__`、`__repr__`、`__eq__` 都自动有了。**需要一个「装数据」的类时，首选 `@dataclass`。**

## 13.8 `@property`：像属性一样用的方法

有些值是根据其他属性**算出来的**，或者修改时需要**检查**。`@property` 让方法用起来像普通属性：

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):                  # 用的时候不加括号：r.area
        return self.width * self.height

r = Rectangle(3, 4)
print(r.area)                        # 12
r.width = 10
print(r.area)                        # 40：总是根据最新的宽和高计算
```

## 13.9 特殊方法：让自己的类支持 `+`、`==`、`len`……

名字前后都有两个下划线的方法（比如 `__init__`、`__str__`）叫**特殊方法**（也叫「魔术方法」）。定义它们，就能让自己的对象支持 Python 的各种语法：

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other):             # v1 + v2
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, k):                 # v * 3
        return Vector(self.x * k, self.y * k)

    def __eq__(self, other):              # v1 == v2
        return self.x == other.x and self.y == other.y

    def __len__(self):                    # len(v)：这里返回维度
        return 2

    def __repr__(self):                   # 在列表里、交互环境里显示的样子
        return f"Vector({self.x}, {self.y})"

v = Vector(1, 2) + Vector(3, 4)
print(v, v * 3, len(v))
print(Vector(1, 1) == Vector(1, 1))
print([Vector(0, 0), v])
```

| 特殊方法 | 对应的写法 |
|----------|------------|
| `__init__` | `类名(...)` 创建对象 |
| `__str__` / `__repr__` | `print(obj)` / 交互环境、列表中显示 |
| `__eq__`、`__lt__` | `==`、`<`（定义了 `__lt__` 就能 `sorted`） |
| `__add__`、`__sub__`、`__mul__` | `+`、`-`、`*` |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[i]` |
| `__contains__` | `x in obj` |

> [!NOTE]
> Python 不太在乎对象「是什么类型」，只在乎它「能做什么」：只要定义了 `__len__`，就能用 `len()`；只要能 `for` 遍历，就能当成序列用。这叫**鸭子类型**：「如果它走起来像鸭子、叫起来像鸭子，那它就是鸭子。」

## 13.10 常见错误

```python norun
class Dog:
    def bark():                  # 忘了写 self
        print("汪")

Dog().bark()
```

```text
TypeError: Dog.bark() takes 0 positional arguments but 1 was given
```

调用 `d.bark()` 时 Python 会自动把对象作为第一个参数传进去，所以方法定义里必须有 `self`。

```python norun
class Dog:
    def __init__(self, name):
        name = name              # 忘了 self.，只是给局部变量赋值

d = Dog("旺财")
print(d.name)
```

```text
AttributeError: 'Dog' object has no attribute 'name'
```

要写 `self.name = name`，属性才会保存在对象上。

```python norun
class Dog:
    def __init__(self, name):
        self.name = name

d = Dog()
```

```text
TypeError: Dog.__init__() missing 1 required positional argument: 'name'
```

创建对象时要按 `__init__` 的参数传值：`Dog("旺财")`。

## 13.11 小结

- 对象 = 数据（属性）+ 操作（方法）；类是图纸，对象（实例）是按图纸造出来的东西。
- `class 名字:` 定义类；`__init__(self, ...)` 设置初始数据；方法的第一个参数是 `self`，代表对象自己。
- `self.属性` 是每个对象一份的实例属性；写在类里的是所有对象共享的类属性。
- `__str__` 决定 `print` 的样子；`@property` 让方法像属性一样用；`__add__`、`__eq__`、`__len__` 等特殊方法让对象支持各种语法。
- 继承 `class 子类(父类):`，可以重写方法、用 `super()` 调用父类；「是一种」用继承，「有一个」用组合。
- 只用来装数据的类，用 `@dataclass` 自动生成 `__init__`、`__repr__`、`__eq__`。

## 13.12 练习

<!-- exercise: rectangle -->
### 练习 1：矩形类

补全 `Rectangle` 类的 `area`（面积）和 `perimeter`（周长）方法。程序读入宽和高（整数），输出面积和周长，空格隔开。

```text
样例输入：3 4
样例输出：12 14
```

<details>
<summary>提示</summary>

`return self.width * self.height`；周长是 `2 * (self.width + self.height)`。
</details>

<!-- exercise: bank -->
### 练习 2：银行账户

补全 `BankAccount` 类：`deposit(amount)` 存钱；`withdraw(amount)` 取钱，余额不够时**不取钱**，而是返回 `False`（成功时返回 `True`）。程序读入若干条命令，最后一行是 `end`：

- `deposit 金额`：存钱；
- `withdraw 金额`：取钱，失败时输出 `insufficient funds`；
- `balance`：输出当前余额。

```text
样例输入：
deposit 100
withdraw 30
withdraw 500
balance
end
样例输出：
insufficient funds
70
```

<details>
<summary>提示</summary>

`withdraw` 里先判断 `if amount > self.balance: return False`，否则扣钱并 `return True`。
</details>

<!-- exercise: shapes -->
### 练习 3：继承与多态

已有父类 `Shape`。补全子类 `Circle`（半径 `r`，面积 `π r²`）和 `Square`（边长 `a`，面积 `a²`）的 `area` 方法。程序读入若干行，每行是 `circle 半径` 或 `square 边长`，最后一行是 `end`；对每个形状输出面积，保留 2 位小数。

```text
样例输入：
circle 1
square 2
end
样例输出：
3.14
4.00
```

<details>
<summary>提示</summary>

`import math`，圆的面积 `math.pi * self.r ** 2`。主程序会根据第一个单词创建对应的对象，然后统一调用 `shape.area()`。
</details>

<!-- exercise: dataclass-sort -->
### 练习 4：用 dataclass 排序

第一行是整数 `n`，接下来 `n` 行每行一个名字和分数。用 `@dataclass` 定义 `Student`，按**分数从高到低**排序，分数相同按**名字的字母顺序**，每行输出一个名字。

```text
样例输入：
3
Tom 80
Amy 95
Bob 80
样例输出：
Amy
Bob
Tom
```

<details>
<summary>提示</summary>

`students.sort(key=lambda s: (-s.score, s.name))`
</details>

<!-- exercise: counter-predict -->
### 练习 5：读代码，写输出

```python
class Counter:
    total = 0

    def __init__(self):
        self.count = 0
        Counter.total += 1

    def click(self):
        self.count += 1

a = Counter()
b = Counter()
a.click()
a.click()
b.click()
print(a.count, b.count, Counter.total)
```

<details>
<summary>答案与解析</summary>

`count` 是实例属性，每个对象各一份：`a` 点了 2 次，`b` 点了 1 次。`total` 是类属性，创建了 2 个对象。输出 `2 1 2`。
</details>

<!-- exercise: what-is-self -->
### 练习 6：`self` 是什么？

在 `d.bark()` 这次调用中，`bark` 方法里的 `self` 指的是什么？

- `Dog` 这个类
- 对象 `d` 本身
- `bark` 这个方法
- 一个新创建的 `Dog`

<details>
<summary>答案与解析</summary>

`d.bark()` 相当于 `Dog.bark(d)`，所以 `self` 就是调用这个方法的对象 `d`。通过 `self`，方法才能访问和修改这个对象自己的数据。
</details>

## 13.13 延伸阅读

- [Python 官方教程 · 9. 类](https://docs.python.org/zh-cn/3/tutorial/classes.html)
- [dataclasses 文档](https://docs.python.org/zh-cn/3/library/dataclasses.html)
- 下一课：[进阶特性](../14-advanced/zh-CN.md)
