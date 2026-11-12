# 13. Classes and objects

> 🎯 **Goals**: understand that "an object = data + the operations you can do on it"; define your own types with `class`, writing `__init__`, methods and `self`; make objects print nicely with `__str__`; understand class attributes and instance attributes; reuse code with **inheritance**; save a lot of boilerplate with modern Python's **`@dataclass`**; meet `@property` and special methods such as `__eq__`, `__lt__` and `__add__`.
>
> ⏱ **Time**: 90 minutes　|　📋 **You need**: lessons 9, 10 and 12

## 13.1 You've been using objects all along

```python
name = "hello"
print(name.upper())          # a string "object" comes with an upper method
nums = [3, 1, 2]
nums.sort()                  # a list object comes with a sort method
print(nums, type(nums))
```

In Python **everything is an object**: strings, lists, dictionaries, functions… Each object has its own **data** (the characters in a string, the elements of a list) and **operations** you can do on it (`upper`, `sort`, `append`); these operations are called **methods**.

`str` and `list` are **types** that come with Python, also called **classes**. In this lesson we learn to define **our own classes** — say, a "character" in a game, with a name and health points, that can attack and take damage.

## 13.2 Defining a class

```python
class Dog:
    def __init__(self, name, age):     # called automatically when an object is created: sets up its data
        self.name = name               # self.name is this object's own "attribute"
        self.age = age

    def bark(self):                    # a method: a function belonging to the class; its first parameter is always self
        print(f"{self.name}: Woof!")

    def birthday(self):
        self.age += 1                  # methods can change the object's own data
        print(f"{self.name} is now {self.age}")

a = Dog("Rex", 3)                      # create an object (also called an "instance") from the class
b = Dog("Snowy", 1)
a.bark()
b.bark()
a.birthday()
print(a.age, b.age)                    # the two dogs' data are independent: 4 1
```

Piece by piece:

- **`class Dog:`** defines a class called `Dog`. Class names are conventionally written in CamelCase **starting with a capital letter** (`BankAccount`, `GameCharacter`).
- **A class is a blueprint; objects are things built from it**: `Dog` is the design of "a dog", and `a` and `b` are two particular dogs.
- **`__init__`** (two underscores on each side) is the **initializer**: `Dog("Rex", 3)` creates a new object and automatically calls `__init__(new_object, "Rex", 3)`.
- **`self`** means "this object itself". `a.bark()` is effectively `Dog.bark(a)`, so inside `bark`, `self` is `a`. Methods must be defined with `self` as their first parameter, but you don't pass it when calling.
- **`self.name = name`** stores the parameter `name` in the object's attribute `self.name`, so other methods of the object can use it.

## 13.3 Making objects print nicely: `__str__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)                     # <__main__.Point object at 0x...>: tells you nothing
```

Define a `__str__` method to tell Python what to show when the object is printed:

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
print("distance to origin", p.distance_to_origin())
```

## 13.4 A complete example: a bank account

Keeping data (the balance) together with rules (no overdrafts) is exactly what classes are best at:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("deposits must be greater than 0")
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError(f"insufficient funds: only {self.balance} left")
        self.balance -= amount

    def __str__(self):
        return f"{self.owner}'s account, balance {self.balance}"

acc = BankAccount("Tom", 100)
acc.deposit(50)
print(acc)
try:
    acc.withdraw(500)
except ValueError as e:
    print("withdrawal failed:", e)
acc.withdraw(30)
print(acc)
```

People using this class don't need to know how the balance is stored; they just call `deposit` and `withdraw`. And the "no overdrafts" rule lives inside the class, so it can't be forgotten. This is called **encapsulation**.

## 13.5 Class attributes and instance attributes

- **instance attributes**: `self.xxx`, one copy per object (each dog has its own name);
- **class attributes**: written directly inside `class`, **shared** by all objects (every dog is a "canine").

```python
class Dog:
    species = "canine"            # class attribute: shared by every Dog
    count = 0

    def __init__(self, name):
        self.name = name          # instance attribute: one per dog
        Dog.count += 1            # change a class attribute through the class name

a, b = Dog("Rex"), Dog("Snowy")
print(a.species, b.species)       # canine canine
print("dogs created:", Dog.count)
```

## 13.6 Inheritance: extending an existing class

Many classes have things in common: dogs and cats are both animals with names that make sounds — just different sounds. **Inheritance** gives a new class all the attributes and methods of an existing one, so you only add or change what's different:

```python
class Animal:                              # the parent (base) class
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

    def introduce(self):
        return f"I'm {self.name} and I say: {self.speak()}"

class Dog(Animal):                         # Dog inherits from Animal
    def speak(self):                       # override the parent's method
        return "Woof"

class Cat(Animal):
    def __init__(self, name, indoor):
        super().__init__(name)             # call the parent's __init__ first to set name
        self.indoor = indoor               # then add our own attribute

    def speak(self):
        return "Meow~"

for pet in [Dog("Rex"), Cat("Kitty", indoor=True), Animal("Some animal")]:
    print(pet.introduce())
```

- `class Dog(Animal):` puts the parent class in parentheses;
- methods the child doesn't define (`introduce`) come from the parent; a method with the same name in the child (`speak`) replaces the parent's;
- `super().__init__(...)` calls the parent's method, avoiding repeated code;
- the same call `pet.introduce()` does different things for different objects — this is **polymorphism**.

> [!TIP]
> Inheritance is useful, but don't overuse it. If "A **is a kind of** B" (a dog is a kind of animal), use inheritance; if "A **has a** B" (a car has an engine), make B an attribute of A (this is **composition**). In practice, composition is often simpler and clearer than deep inheritance chains.

## 13.7 The modern way: `@dataclass`

Many classes exist just to **hold data**, and writing `__init__`, `__str__`, comparisons and so on every time is tedious. Since Python 3.7, **`dataclasses`** generate these for you:

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    score: int
    grade: int = 1                    # defaults are allowed

a = Student("Tom", 88)
b = Student("Amy", 95, grade=2)
print(a)                              # nice printing for free: Student(name='Tom', score=88, grade=1)
print(a == Student("Tom", 88))        # "equals" comparison for free: True

students = [a, b, Student("Bob", 88)]
students.sort(key=lambda s: (-s.score, s.name))      # highest score first, ties by name
print([s.name for s in students])
```

`@dataclass`, written above the class definition, is a **decorator**. Just list the field names with type hints, and `__init__`, `__repr__` and `__eq__` appear automatically. **When you need a class that "holds data", reach for `@dataclass` first.**

## 13.8 `@property`: methods used like attributes

Some values are **computed** from other attributes, or need **checking** when changed. `@property` lets a method be used like an ordinary attribute:

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):                  # used without parentheses: r.area
        return self.width * self.height

r = Rectangle(3, 4)
print(r.area)                        # 12
r.width = 10
print(r.area)                        # 40: always computed from the current width and height
```

## 13.9 Special methods: making your classes support `+`, `==`, `len`…

Methods whose names start and end with two underscores (like `__init__` and `__str__`) are **special methods** (also called "magic methods"). Define them and your objects support Python's built-in syntax:

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

    def __len__(self):                    # len(v): here, the number of dimensions
        return 2

    def __repr__(self):                   # how it appears in lists and interactive sessions
        return f"Vector({self.x}, {self.y})"

v = Vector(1, 2) + Vector(3, 4)
print(v, v * 3, len(v))
print(Vector(1, 1) == Vector(1, 1))
print([Vector(0, 0), v])
```

| Special method | Syntax it supports |
|----------------|--------------------|
| `__init__` | creating an object with `ClassName(...)` |
| `__str__` / `__repr__` | `print(obj)` / display in interactive sessions and lists |
| `__eq__`, `__lt__` | `==`, `<` (with `__lt__` you can `sorted`) |
| `__add__`, `__sub__`, `__mul__` | `+`, `-`, `*` |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[i]` |
| `__contains__` | `x in obj` |

> [!NOTE]
> Python cares less about "what type" an object is than about "what it can do": define `__len__` and `len()` works; make it loopable with `for` and it can be used as a sequence. This is **duck typing**: "if it walks like a duck and quacks like a duck, it's a duck."

## 13.10 Common mistakes

```python norun
class Dog:
    def bark():                  # forgot self
        print("Woof")

Dog().bark()
```

```text
TypeError: Dog.bark() takes 0 positional arguments but 1 was given
```

When you call `d.bark()`, Python automatically passes the object as the first argument, so the method definition must include `self`.

```python norun
class Dog:
    def __init__(self, name):
        name = name              # forgot self. — this only assigns to a local variable

d = Dog("Rex")
print(d.name)
```

```text
AttributeError: 'Dog' object has no attribute 'name'
```

Write `self.name = name` so the attribute is stored on the object.

```python norun
class Dog:
    def __init__(self, name):
        self.name = name

d = Dog()
```

```text
TypeError: Dog.__init__() missing 1 required positional argument: 'name'
```

When creating an object, pass the arguments `__init__` expects: `Dog("Rex")`.

## 13.11 Summary

- An object = data (attributes) + operations (methods); a class is a blueprint and objects (instances) are built from it.
- `class Name:` defines a class; `__init__(self, ...)` sets up the initial data; a method's first parameter is `self`, the object itself.
- `self.attribute` is an instance attribute, one per object; attributes written in the class body are class attributes shared by all objects.
- `__str__` decides how `print` shows the object; `@property` makes a method usable like an attribute; special methods like `__add__`, `__eq__` and `__len__` let objects support built-in syntax.
- Inheritance: `class Child(Parent):` — override methods and call the parent with `super()`; use inheritance for "is a kind of", composition for "has a".
- For classes that only hold data, `@dataclass` generates `__init__`, `__repr__` and `__eq__` automatically.

## 13.12 Exercises

<!-- exercise: rectangle -->
### Exercise 1: A rectangle class

Complete the `area` and `perimeter` methods of the `Rectangle` class. The program reads a width and a height (integers) and prints the area and perimeter, separated by a space.

```text
Sample input: 3 4
Sample output: 12 14
```

<details>
<summary>Hint</summary>

`return self.width * self.height`; the perimeter is `2 * (self.width + self.height)`.
</details>

<!-- exercise: bank -->
### Exercise 2: A bank account

Complete the `BankAccount` class: `deposit(amount)` adds money; `withdraw(amount)` takes money out, but if the balance is too low it **doesn't take anything** and returns `False` instead (it returns `True` on success). The program reads commands until a line saying `end`:

- `deposit amount`: deposit money;
- `withdraw amount`: withdraw money, printing `insufficient funds` if it fails;
- `balance`: print the current balance.

```text
Sample input:
deposit 100
withdraw 30
withdraw 500
balance
end
Sample output:
insufficient funds
70
```

<details>
<summary>Hint</summary>

In `withdraw`, check `if amount > self.balance: return False` first; otherwise subtract the money and `return True`.
</details>

<!-- exercise: shapes -->
### Exercise 3: Inheritance and polymorphism

The parent class `Shape` is given. Complete the `area` methods of the subclasses `Circle` (radius `r`, area `π r²`) and `Square` (side `a`, area `a²`). The program reads lines of the form `circle radius` or `square side` until a line saying `end`, and prints each shape's area with 2 decimal places.

```text
Sample input:
circle 1
square 2
end
Sample output:
3.14
4.00
```

<details>
<summary>Hint</summary>

`import math`; a circle's area is `math.pi * self.r ** 2`. The main program creates the right object from the first word, then just calls `shape.area()` for every shape.
</details>

<!-- exercise: dataclass-sort -->
### Exercise 4: Sorting with a dataclass

The first line is an integer `n`, followed by `n` lines each containing a name and a score. Define `Student` with `@dataclass` and sort the students by **score from high to low**, breaking ties by **name in alphabetical order**; print one name per line.

```text
Sample input:
3
Tom 80
Amy 95
Bob 80
Sample output:
Amy
Bob
Tom
```

<details>
<summary>Hint</summary>

`students.sort(key=lambda s: (-s.score, s.name))`
</details>

<!-- exercise: counter-predict -->
### Exercise 5: Predict the output

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
<summary>Answer and explanation</summary>

`count` is an instance attribute, one per object: `a` was clicked twice and `b` once. `total` is a class attribute, and 2 objects were created. The output is `2 1 2`.
</details>

<!-- exercise: what-is-self -->
### Exercise 6: What is `self`?

In the call `d.bark()`, what does `self` refer to inside the `bark` method?

- the class `Dog`
- the object `d` itself
- the method `bark`
- a newly created `Dog`

<details>
<summary>Answer and explanation</summary>

`d.bark()` is the same as `Dog.bark(d)`, so `self` is the object `d` the method was called on. Through `self`, a method can read and change that object's own data.
</details>

## 13.13 Further reading

- [The Python Tutorial · 9. Classes](https://docs.python.org/3/tutorial/classes.html)
- [dataclasses documentation](https://docs.python.org/3/library/dataclasses.html)
- Next lesson: [Advanced features](../14-advanced/en.md)
