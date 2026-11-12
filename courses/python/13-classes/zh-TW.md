# 13. 類與物件

> 🎯 **本課目標**：理解「物件 = 資料 + 能對它做的操作」；用 `class` 定義自己的型別，寫出 `__init__`、方法和 `self`；用 `__str__` 讓物件列印得好看；理解類屬性和例項屬性；用**繼承**複用程式碼；用現代 Python 的 **`@dataclass`** 少寫很多樣板程式碼；認識 `@property` 和 `__eq__`、`__lt__`、`__add__` 等特殊方法。
>
> ⏱ **預計用時**：90 分鐘　｜　📋 **前置知識**：第 9、10、12 課

## 13.1 你其實一直在用物件

```python
name = "hello"
print(name.upper())          # 字串「物件」自帶 upper 方法
nums = [3, 1, 2]
nums.sort()                  # 列表物件自帶 sort 方法
print(nums, type(nums))
```

在 Python 裡，**一切都是物件**：字串、列表、字典、函式……每個物件都有自己的**資料**（字串裡的字元、列表裡的元素），以及可以對它做的**操作**（`upper`、`sort`、`append`），這些操作叫**方法**（method）。

`str`、`list` 這些是 Python 自帶的**型別**，也叫**類**（class）。這一課我們學習定義**自己的類**：比如一個遊戲裡的「角色」，有名字、血量，還能攻擊、受傷。

## 13.2 定義一個類

```python
class Dog:
    def __init__(self, name, age):     # 建立物件時自動呼叫：給新物件設定初始資料
        self.name = name               # self.name 是這個物件自己的「屬性」
        self.age = age

    def bark(self):                    # 方法：屬於這個類的函式，第一個引數總是 self
        print(f"{self.name}：汪汪！")

    def birthday(self):
        self.age += 1                  # 方法可以修改物件自己的資料
        print(f"{self.name} {self.age} 歲啦")

a = Dog("旺財", 3)                     # 用類建立一個物件（也叫「例項」）
b = Dog("小白", 1)
a.bark()
b.bark()
a.birthday()
print(a.age, b.age)                    # 兩隻狗的資料互不影響：4 1
```

逐個解釋：

- **`class Dog:`** 定義一個名叫 `Dog` 的類。類名習慣用**大寫字母開頭**的駝峰寫法（`BankAccount`、`GameCharacter`）。
- **類是圖紙，物件是按圖紙造出來的東西**：`Dog` 是「狗」的設計圖，`a` 和 `b` 是兩隻具體的狗。
- **`__init__`**（前後各兩個下劃線）是**初始化方法**：`Dog("旺財", 3)` 會建立一個新物件，並自動呼叫 `__init__(新物件, "旺財", 3)`。
- **`self`** 就是「這個物件自己」。`a.bark()` 實際上相當於 `Dog.bark(a)`，所以在 `bark` 裡，`self` 就是 `a`。定義方法時第一個引數必須寫 `self`，呼叫時不用傳。
- **`self.name = name`**：把引數 `name` 存進物件的屬性 `self.name`，這樣物件在別的方法裡也能用到它。

## 13.3 讓物件列印得好看：`__str__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)                     # <__main__.Point object at 0x...>：看不出內容
```

定義 `__str__` 方法，告訴 Python 列印這個物件時顯示什麼：

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
print("距離原點", p.distance_to_origin())
```

## 13.4 一個完整的例子：銀行賬戶

把資料（餘額）和規則（不能透支）放在一起，正是類最擅長的：

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("存款金額必須大於 0")
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError(f"餘額不足：只有 {self.balance} 元")
        self.balance -= amount

    def __str__(self):
        return f"{self.owner} 的賬戶，餘額 {self.balance} 元"

acc = BankAccount("小明", 100)
acc.deposit(50)
print(acc)
try:
    acc.withdraw(500)
except ValueError as e:
    print("取款失敗：", e)
acc.withdraw(30)
print(acc)
```

使用這個類的人不需要知道餘額是怎麼存的，只要呼叫 `deposit`、`withdraw`；而「不能透支」的規則寫在類裡，不會被忘記。這叫**封裝**（encapsulation）。

## 13.5 類屬性和例項屬性

- **例項屬性**：`self.xxx`，每個物件各有一份（每隻狗有自己的名字）；
- **類屬性**：直接寫在 `class` 裡面，所有物件**共享**一份（所有狗都是「犬科」）。

```python
class Dog:
    species = "犬科"              # 類屬性：所有 Dog 共享
    count = 0

    def __init__(self, name):
        self.name = name          # 例項屬性：每隻狗一份
        Dog.count += 1            # 修改類屬性要透過類名

a, b = Dog("旺財"), Dog("小白")
print(a.species, b.species)       # 犬科 犬科
print("一共建立了", Dog.count, "只狗")
```

## 13.6 繼承：在已有的類上擴充套件

很多類有共同點：狗和貓都是動物，都有名字、都會叫，只是叫聲不同。**繼承**（inheritance）讓新類自動獲得舊類的所有屬性和方法，再新增或修改不同的部分：

```python
class Animal:                              # 父類（基類）
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "……"

    def introduce(self):
        return f"我是{self.name}，我會說：{self.speak()}"

class Dog(Animal):                         # Dog 繼承 Animal
    def speak(self):                       # 重寫（override）父類的方法
        return "汪汪"

class Cat(Animal):
    def __init__(self, name, indoor):
        super().__init__(name)             # 先呼叫父類的 __init__，設定好 name
        self.indoor = indoor               # 再新增自己特有的屬性

    def speak(self):
        return "喵～"

for pet in [Dog("旺財"), Cat("咪咪", indoor=True), Animal("某種動物")]:
    print(pet.introduce())
```

- `class Dog(Animal):` 括號裡寫父類；
- 子類沒寫的方法（`introduce`）直接用父類的；子類寫了同名方法（`speak`），就用子類自己的；
- `super().__init__(...)` 呼叫父類的方法，避免重複寫程式碼；
- 同一句 `pet.introduce()`，對不同的物件做出不同的事——這叫**多型**（polymorphism）。

> [!TIP]
> 繼承很有用，但不要濫用。如果兩個東西是「A **是一種** B」（狗是一種動物），用繼承；如果是「A **有一個** B」（汽車有一個發動機），就把 B 作為 A 的一個屬性（這叫**組合**）。實踐中，組合往往比層層繼承更簡單清楚。

## 13.7 現代寫法：`@dataclass`

很多類只是用來**裝資料**的，每次都要寫 `__init__`、`__str__`、比較大小……很囉嗦。Python 3.7 起的 **`dataclasses`** 能自動生成這些：

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    score: int
    grade: int = 1                    # 可以有預設值

a = Student("小明", 88)
b = Student("小紅", 95, grade=2)
print(a)                              # 自動生成好看的列印：Student(name='小明', score=88, grade=1)
print(a == Student("小明", 88))       # 自動生成「相等」比較：True

students = [a, b, Student("小剛", 88)]
students.sort(key=lambda s: (-s.score, s.name))      # 按分數從高到低，同分按名字
print([s.name for s in students])
```

`@dataclass` 寫在類定義的上面，叫**裝飾器**（decorator）。只要寫出欄位名和型別標註，`__init__`、`__repr__`、`__eq__` 都自動有了。**需要一個「裝資料」的類時，首選 `@dataclass`。**

## 13.8 `@property`：像屬性一樣用的方法

有些值是根據其他屬性**算出來的**，或者修改時需要**檢查**。`@property` 讓方法用起來像普通屬性：

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):                  # 用的時候不加括號：r.area
        return self.width * self.height

r = Rectangle(3, 4)
print(r.area)                        # 12
r.width = 10
print(r.area)                        # 40：總是根據最新的寬和高計算
```

## 13.9 特殊方法：讓自己的類支援 `+`、`==`、`len`……

名字前後都有兩個下劃線的方法（比如 `__init__`、`__str__`）叫**特殊方法**（也叫「魔術方法」）。定義它們，就能讓自己的物件支援 Python 的各種語法：

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

    def __len__(self):                    # len(v)：這裡返回維度
        return 2

    def __repr__(self):                   # 在列表裡、互動環境裡顯示的樣子
        return f"Vector({self.x}, {self.y})"

v = Vector(1, 2) + Vector(3, 4)
print(v, v * 3, len(v))
print(Vector(1, 1) == Vector(1, 1))
print([Vector(0, 0), v])
```

| 特殊方法 | 對應的寫法 |
|----------|------------|
| `__init__` | `類名(...)` 建立物件 |
| `__str__` / `__repr__` | `print(obj)` / 互動環境、列表中顯示 |
| `__eq__`、`__lt__` | `==`、`<`（定義了 `__lt__` 就能 `sorted`） |
| `__add__`、`__sub__`、`__mul__` | `+`、`-`、`*` |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[i]` |
| `__contains__` | `x in obj` |

> [!NOTE]
> Python 不太在乎物件「是什麼型別」，只在乎它「能做什麼」：只要定義了 `__len__`，就能用 `len()`；只要能 `for` 遍歷，就能當成序列用。這叫**鴨子型別**：「如果它走起來像鴨子、叫起來像鴨子，那它就是鴨子。」

## 13.10 常見錯誤

```python norun
class Dog:
    def bark():                  # 忘了寫 self
        print("汪")

Dog().bark()
```

```text
TypeError: Dog.bark() takes 0 positional arguments but 1 was given
```

呼叫 `d.bark()` 時 Python 會自動把物件作為第一個引數傳進去，所以方法定義裡必須有 `self`。

```python norun
class Dog:
    def __init__(self, name):
        name = name              # 忘了 self.，只是給區域性變數賦值

d = Dog("旺財")
print(d.name)
```

```text
AttributeError: 'Dog' object has no attribute 'name'
```

要寫 `self.name = name`，屬性才會儲存在物件上。

```python norun
class Dog:
    def __init__(self, name):
        self.name = name

d = Dog()
```

```text
TypeError: Dog.__init__() missing 1 required positional argument: 'name'
```

建立物件時要按 `__init__` 的引數傳值：`Dog("旺財")`。

## 13.11 小結

- 物件 = 資料（屬性）+ 操作（方法）；類是圖紙，物件（例項）是按圖紙造出來的東西。
- `class 名字:` 定義類；`__init__(self, ...)` 設定初始資料；方法的第一個引數是 `self`，代表物件自己。
- `self.屬性` 是每個物件一份的例項屬性；寫在類裡的是所有物件共享的類屬性。
- `__str__` 決定 `print` 的樣子；`@property` 讓方法像屬性一樣用；`__add__`、`__eq__`、`__len__` 等特殊方法讓物件支援各種語法。
- 繼承 `class 子類(父類):`，可以重寫方法、用 `super()` 呼叫父類；「是一種」用繼承，「有一個」用組合。
- 只用來裝資料的類，用 `@dataclass` 自動生成 `__init__`、`__repr__`、`__eq__`。

## 13.12 練習

<!-- exercise: rectangle -->
### 練習 1：矩形類

補全 `Rectangle` 類的 `area`（面積）和 `perimeter`（周長）方法。程式讀入寬和高（整數），輸出面積和周長，空格隔開。

```text
樣例輸入：3 4
樣例輸出：12 14
```

<details>
<summary>提示</summary>

`return self.width * self.height`；周長是 `2 * (self.width + self.height)`。
</details>

<!-- exercise: bank -->
### 練習 2：銀行賬戶

補全 `BankAccount` 類：`deposit(amount)` 存錢；`withdraw(amount)` 取錢，餘額不夠時**不取錢**，而是返回 `False`（成功時返回 `True`）。程式讀入若干條命令，最後一行是 `end`：

- `deposit 金額`：存錢；
- `withdraw 金額`：取錢，失敗時輸出 `insufficient funds`；
- `balance`：輸出當前餘額。

```text
樣例輸入：
deposit 100
withdraw 30
withdraw 500
balance
end
樣例輸出：
insufficient funds
70
```

<details>
<summary>提示</summary>

`withdraw` 裡先判斷 `if amount > self.balance: return False`，否則扣錢並 `return True`。
</details>

<!-- exercise: shapes -->
### 練習 3：繼承與多型

已有父類 `Shape`。補全子類 `Circle`（半徑 `r`，面積 `π r²`）和 `Square`（邊長 `a`，面積 `a²`）的 `area` 方法。程式讀入若干行，每行是 `circle 半徑` 或 `square 邊長`，最後一行是 `end`；對每個形狀輸出面積，保留 2 位小數。

```text
樣例輸入：
circle 1
square 2
end
樣例輸出：
3.14
4.00
```

<details>
<summary>提示</summary>

`import math`，圓的面積 `math.pi * self.r ** 2`。主程式會根據第一個單詞建立對應的物件，然後統一呼叫 `shape.area()`。
</details>

<!-- exercise: dataclass-sort -->
### 練習 4：用 dataclass 排序

第一行是整數 `n`，接下來 `n` 行每行一個名字和分數。用 `@dataclass` 定義 `Student`，按**分數從高到低**排序，分數相同按**名字的字母順序**，每行輸出一個名字。

```text
樣例輸入：
3
Tom 80
Amy 95
Bob 80
樣例輸出：
Amy
Bob
Tom
```

<details>
<summary>提示</summary>

`students.sort(key=lambda s: (-s.score, s.name))`
</details>

<!-- exercise: counter-predict -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`count` 是例項屬性，每個物件各一份：`a` 點了 2 次，`b` 點了 1 次。`total` 是類屬性，建立了 2 個物件。輸出 `2 1 2`。
</details>

<!-- exercise: what-is-self -->
### 練習 6：`self` 是什麼？

在 `d.bark()` 這次呼叫中，`bark` 方法裡的 `self` 指的是什麼？

- `Dog` 這個類
- 物件 `d` 本身
- `bark` 這個方法
- 一個新建立的 `Dog`

<details>
<summary>答案與解析</summary>

`d.bark()` 相當於 `Dog.bark(d)`，所以 `self` 就是呼叫這個方法的物件 `d`。透過 `self`，方法才能訪問和修改這個物件自己的資料。
</details>

## 13.13 延伸閱讀

- [Python 官方教程 · 9. 類](https://docs.python.org/zh-cn/3/tutorial/classes.html)
- [dataclasses 文件](https://docs.python.org/zh-cn/3/library/dataclasses.html)
- 下一課：[進階特性](../14-advanced/zh-CN.md)
