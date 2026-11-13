# 12. 结构体、枚举和链表

> 🎯 **本课目标**：用 **`struct`** 把相关的数据组合成一个新类型；用 `typedef` 起一个简短的名字；理解结构体的赋值（整体复制）和比较（不能用 `==`），把结构体传给函数、从函数返回，用指针和 `->` 修改它；排序结构体数组；用 **`enum`** 表示一组有名字的取值；用结构体和指针搭建**链表**。
>
> ⏱ **预计用时**：100 分钟　｜　📋 **前置知识**：本课程第 1～11 课

## 12.1 结构体：把相关的数据放在一起

一个点有 x 和 y 两个坐标，一个学生有名字、年龄和成绩。把它们分开存在几个变量里，很容易弄乱。**结构体**（structure）可以把几个相关的数据组合成一个新的类型：

```c
#include <stdio.h>

struct point {               // 定义一个新类型：struct point
    int x;                   // 它有两个「成员」：x 和 y
    int y;
};                           // 注意这里的分号！

int main(void) {
    struct point a = {3, 4};               // 按顺序初始化
    struct point b = {.y = 10, .x = -2};   // 按成员名初始化：顺序随意，没写的成员为 0
    a.x = a.x + 1;                         // 用 . 访问成员
    printf("a = (%d, %d)\n", a.x, a.y);
    printf("b = (%d, %d)\n", b.x, b.y);
    printf("struct point 占 %zu 字节\n", sizeof(struct point));
    return 0;
}
```

- `struct point { ... };` 只是**定义了一种类型**（相当于一张设计图），并没有创建变量；`struct point a;` 才创建了一个变量。
- 定义结束的 `}` 后面**一定要有分号**。忘了的话，GCC 会报一个让人摸不着头脑的错误：`expected ';', identifier or '(' before 'int'`。
- 每个成员都像一个普通变量：`a.x` 可以赋值、参与计算、传给 `scanf`（`scanf("%d", &a.x)`）。

> [!NOTE]
> 结构体的大小有时比各个成员加起来还大。比如 `struct { char c; int n; }` 通常占 8 字节而不是 5 字节：编译器会在成员之间插入「填充」字节，让 `int` 从 4 的倍数的地址开始，这样 CPU 读起来更快。

## 12.2 `typedef`：起一个简短的名字

每次都写 `struct point` 有点啰嗦。`typedef` 可以给类型起个别名：

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    double score;
} Student;                         // 以后直接写 Student

int main(void) {
    Student s = {"Ada", 20, 95.5};
    Student t;
    strcpy(t.name, "Linus");       // 字符数组成员要用 strcpy 赋值（第 9 课）
    t.age = 21;
    t.score = 88.0;
    printf("%-8s %3d %6.1f\n", s.name, s.age, s.score);
    printf("%-8s %3d %6.1f\n", t.name, t.age, t.score);
    return 0;
}
```

习惯上，自己定义的类型名用大写字母开头（`Student`、`Point`），和变量名区分开。

## 12.3 结构体的赋值、比较和函数

```c
#include <stdio.h>
#include <math.h>

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b) {             // 传值：函数拿到的是副本
    double dx = a.x - b.x, dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
}

Point midpoint(Point a, Point b) {              // 结构体可以作为返回值
    Point m = {(a.x + b.x) / 2, (a.y + b.y) / 2};
    return m;
}

void move(Point *p, double dx, double dy) {     // 传指针：可以修改调用者的结构体
    p->x += dx;                                 // p->x 就是 (*p).x
    p->y += dy;
}

int main(void) {
    Point a = {0, 0}, b = {3, 4};
    printf("距离 %.1f\n", distance(a, b));
    Point m = midpoint(a, b);
    printf("中点 (%.1f, %.1f)\n", m.x, m.y);

    Point c = b;                    // 结构体可以用 = 整体复制
    move(&c, 1, 1);
    printf("b = (%.1f, %.1f)，c = (%.1f, %.1f)\n", b.x, b.y, c.x, c.y);
    return 0;
}
```

- **赋值 `c = b` 会复制所有成员**——连结构体里的数组也会被复制。这一点和数组不同（数组不能整体赋值）。
- **结构体不能用 `==` 比较**，要逐个比较成员，最好写成一个函数。
- 传结构体给函数是**值传递**，函数拿到的是一份副本。要让函数修改它，就传指针。
- 通过指针访问成员用 **`->`**：`p->x` 就是 `(*p).x`。注意 `*p.x` 是错的：`.` 的优先级比 `*` 高，它会被理解成 `*(p.x)`。
- 结构体很大时（比如有一个 1000 个元素的数组成员），每次传值都要复制一大块内存。这时即使函数不修改它，也常常传 `const Student *s`：只传一个地址，又保证不修改。

## 12.4 结构体数组

结构体数组就像一张表格：每个元素是一行，每个成员是一列。用 `qsort`（第 10 课）可以按任意规则排序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int score;
} Student;

int by_score_desc(const void *a, const void *b) {
    const Student *x = a;                  // void * 可以直接赋给其他类型的指针
    const Student *y = b;
    if (x->score != y->score) {
        return (x->score < y->score) - (x->score > y->score);   // 分数高的排前面
    }
    return strcmp(x->name, y->name);       // 同分时按名字排
}

int main(void) {
    Student cls[] = {
        {"Mia", 88},
        {"Leo", 95},
        {"Ava", 88},
        {"Noah", 72},
    };
    int n = sizeof cls / sizeof cls[0];
    qsort(cls, n, sizeof cls[0], by_score_desc);
    for (int i = 0; i < n; i++) {
        printf("%d. %-5s %d\n", i + 1, cls[i].name, cls[i].score);
    }
    return 0;
}
```

`cls[i].name`：先取第 `i` 个元素（一个 `Student`），再取它的成员 `name`。

## 12.5 枚举 `enum`

当一个变量只能取几个固定的值时（星期几、红绿灯、游戏角色的状态），用**枚举**给每个值起个名字，比用 0、1、2 清楚得多：

```c
#include <stdio.h>

typedef enum {
    RED,             // 0
    YELLOW,          // 1
    GREEN,           // 2
} Light;

const char *light_name(Light l) {
    switch (l) {
        case RED:
            return "红灯";
        case YELLOW:
            return "黄灯";
        case GREEN:
            return "绿灯";
    }
    return "?";
}

Light next(Light l) {
    switch (l) {
        case RED:
            return GREEN;
        case GREEN:
            return YELLOW;
        case YELLOW:
            return RED;
    }
    return RED;
}

int main(void) {
    Light l = RED;
    for (int i = 0; i < 5; i++) {
        printf("%s ", light_name(l));
        l = next(l);
    }
    printf("\n");
    return 0;
}
```

- 枚举值其实就是整数常量，默认从 0 开始依次加 1；也可以自己指定，比如 `enum { LOW = 1, HIGH = 10 };`。枚举值名字习惯全用大写。
- 用 `switch` 处理枚举时，如果漏掉了某个值，GCC 会警告 `enumeration value 'GREEN' not handled in switch`——这是枚举的另一个好处。

## 12.6 链表

数组的元素在内存里是连续的，要在开头插入一个元素，就得把后面所有元素都往后挪。**链表**换了一种思路：每个元素（叫**结点**）单独用 `malloc` 申请，每个结点里除了数据，还有一个**指向下一个结点的指针**：

```text
 head
  │
  ▼
┌────┬───┐    ┌────┬───┐    ┌────┬───┐
│ 40 │ ●─┼──▶ │ 30 │ ●─┼──▶ │ 20 │ ●─┼──▶ NULL
└────┴───┘    └────┴───┘    └────┴───┘
 value next
```

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int value;
    struct Node *next;          // 指向下一个结点；最后一个结点的 next 是 NULL
} Node;

// 在链表头部插入一个新结点，返回新的头
Node *push_front(Node *head, int value) {
    Node *n = malloc(sizeof *n);
    if (n == NULL) {
        return head;            // 简单起见：申请失败就不插入
    }
    n->value = value;
    n->next = head;             // 新结点指向原来的第一个结点
    return n;                   // 新结点成为新的头
}

void print_list(const Node *head) {
    for (const Node *p = head; p != NULL; p = p->next) {   // 顺着 next 一个一个走
        printf("%d -> ", p->value);
    }
    printf("NULL\n");
}

void free_list(Node *head) {
    while (head != NULL) {
        Node *next = head->next;    // 先记下下一个结点，再释放当前的
        free(head);
        head = next;
    }
}

int main(void) {
    Node *list = NULL;              // 空链表
    for (int i = 1; i <= 4; i++) {
        list = push_front(list, i * 10);
    }
    print_list(list);               // 40 -> 30 -> 20 -> 10 -> NULL
    free_list(list);
    return 0;
}
```

- 结构体里要用到自己的类型时（`struct Node *next`），`typedef` 后面要写上结构体名 `struct Node`，因为在成员列表里，`Node` 这个别名还没定义好。
- 遍历链表的固定写法：`for (p = head; p != NULL; p = p->next)`。
- 释放链表时，**先记下 `next` 再 `free`**：`free(head)` 之后就不能再读 `head->next` 了。
- 链表和数组各有长处：链表在头部插入、删除很快，但要找第 i 个元素只能从头一个一个数；数组按下标访问很快，但在中间插入要挪动元素。

## 12.7 常见错误

| 错误 | 编译器说 | 正确做法 |
|------|----------|----------|
| 结构体定义后面忘了分号 | `expected ';', identifier or '(' before ...` | `struct point { ... };` |
| 没有 `typedef` 却直接写类型名 | `unknown type name 'point'` | 写 `struct point`，或者用 `typedef` |
| 对指针用 `.` | `'p' is a pointer; did you mean to use '->'?` | `p->x` |
| 对结构体变量用 `->` | `invalid type argument of '->'` | `a.x` |
| 用 `==` 比较结构体 | `invalid operands to binary ==` | 逐个比较成员 |
| 成员名拼错 | `has no member named ...` | 对照结构体的定义 |
| 释放链表结点后再读 `->next` | 释放后使用（内存检查会报告） | 先保存 `next` 再 `free` |

## 12.8 小结

- `struct 名字 { 成员... };` 定义结构体类型；`变量.成员` 访问成员；可以按顺序或用 `.成员 = 值` 初始化。
- `typedef struct { ... } Name;` 起别名；自引用时写 `typedef struct Node { ...; struct Node *next; } Node;`。
- 结构体可以用 `=` 整体复制，可以作为参数和返回值（值传递）；不能用 `==` 比较。
- 指向结构体的指针用 `->`：`p->x` 就是 `(*p).x`；大结构体传 `const T *`。
- 结构体数组 + `qsort` 可以按任意规则排序。
- `enum` 给一组整数常量起名字，配合 `switch` 使用。
- 链表：每个结点 `malloc` 出来，用 `next` 指针串起来，最后一个指向 `NULL`；释放时先保存 `next`。

## 12.9 练习

<!-- exercise: point-dist -->
### 练习 1：两点之间的距离

定义结构体 `Point`（成员是 `double x, y`）和函数 `double distance(Point a, Point b)`。输入两个点的坐标 `x1 y1 x2 y2`，输出它们之间的距离，保留 2 位小数。

```text
样例输入：0 0 3 4
样例输出：5.00
```

<details>
<summary>提示</summary>

照着 12.3 节写。读入时可以直接 `scanf("%lf %lf", &a.x, &a.y);`。
</details>

<!-- exercise: fraction-add -->
### 练习 2：分数相加

定义结构体 `Fraction`（分子 `num`、分母 `den`）。输入 `a b c d`，表示 `a/b + c/d`（`b`、`d` 是正整数，`a`、`c` 可以是负数或 0），输出化成最简分数后的结果，格式 `p/q`（`q` 是正整数；结果是整数时 `q` 为 1，结果为 0 时输出 `0/1`）。

```text
样例输入：1 2 1 3
样例输出：5/6
```

<details>
<summary>提示</summary>

`a/b + c/d = (a*d + c*b) / (b*d)`。再用第 7 课的辗转相除法求分子绝对值和分母的最大公约数，把分子、分母都除以它。用 `long long` 避免溢出。
</details>

<!-- exercise: student-rank -->
### 练习 3：成绩排名

第一行输入人数 `n`（1 ≤ n ≤ 100），接下来 `n` 行，每行是一个学生的英文名（不超过 20 个字母）和成绩（0～100 的整数）。按成绩**从高到低**输出，成绩相同的按名字的字典序。每行输出 `名字 成绩`。

```text
样例输入：
4
Mia 88
Leo 95
Ava 88
Noah 72
样例输出：
Leo 95
Ava 88
Mia 88
Noah 72
```

<details>
<summary>提示</summary>

和 12.4 节一样：定义 `Student` 结构体数组，用 `scanf("%20s %d", s[i].name, &s[i].score)` 读入，再用 `qsort` 加比较函数排序。
</details>

<!-- exercise: list-remove -->
### 练习 4：从链表中删除

第一行输入 `n`（0 ≤ n ≤ 1000），第二行输入 `n` 个整数，按顺序建立链表（新结点接在**末尾**）。第三行输入一个整数 `x`。从链表中删除所有值等于 `x` 的结点，然后输出剩下的值，用空格隔开；如果链表空了，输出 `empty`。

```text
样例输入：
6
3 1 3 4 3 5
3
样例输出：1 4 5
```

<details>
<summary>提示</summary>

建表时用一个 `tail` 指针记住最后一个结点，新结点接在 `tail->next`。删除时最方便的写法是用「指向指针的指针」：`Node **pp = &head;`，当 `(*pp)->value == x` 时，先保存 `Node *dead = *pp;`，再让 `*pp = dead->next;` 并 `free(dead)`；否则 `pp = &(*pp)->next;`。别忘了最后释放整个链表。
</details>

<!-- exercise: struct-predict -->
### 练习 5：读代码，写输出

```c
#include <stdio.h>

typedef struct {
    int a[3];
    int n;
} Box;

void set_n(Box b) {
    b.n = 99;
}

void set_n_ptr(Box *b) {
    b->n = 99;
}

int main(void) {
    Box x = {{1, 2, 3}, 3};
    Box y = x;
    y.a[0] = 100;
    set_n(x);
    printf("%d %d %d\n", x.a[0], y.a[0], x.n);
    set_n_ptr(&y);
    printf("%d\n", y.n);
    return 0;
}
```

<details>
<summary>答案与解析</summary>

`Box y = x;` 复制了所有成员（包括数组），所以改 `y.a[0]` 不影响 `x.a[0]`。`set_n` 拿到的是副本，`x.n` 还是 3。`set_n_ptr` 通过指针修改了 `y.n`。输出：

```text
1 100 3
99
```
</details>

<!-- exercise: arrow -->
### 练习 6：访问成员

`p` 是一个 `Point *`（指向 `Point` 结构体的指针），下面哪个表达式能得到它指向的点的 `x` 坐标？

- `p.x`
- `p->x`
- `*p.x`
- `&p->x`

<details>
<summary>答案与解析</summary>

`p->x` 等于 `(*p).x`。`p.x` 对指针用了 `.`；`*p.x` 因为 `.` 优先级更高，会被理解成 `*(p.x)`；`&p->x` 是 `x` 成员的地址。
</details>

## 12.10 延伸阅读

- [cppreference：结构体声明](https://zh.cppreference.com/w/c/language/struct)、[枚举](https://zh.cppreference.com/w/c/language/enum)、[typedef](https://zh.cppreference.com/w/c/language/typedef)
- [VisuAlgo：链表动画](https://visualgo.net/zh/list)
- 下一课：[文件](../13-files/zh-CN.md)
