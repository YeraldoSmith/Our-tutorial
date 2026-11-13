# 12. 結構體、列舉和連結串列

> 🎯 **本課目標**：用 **`struct`** 把相關的資料組合成一個新型別；用 `typedef` 起一個簡短的名字；理解結構體的賦值（整體複製）和比較（不能用 `==`），把結構體傳給函式、從函式返回，用指標和 `->` 修改它；排序結構體陣列；用 **`enum`** 表示一組有名字的取值；用結構體和指標搭建**連結串列**。
>
> ⏱ **預計用時**：100 分鐘　｜　📋 **前置知識**：本課程第 1～11 課

## 12.1 結構體：把相關的資料放在一起

一個點有 x 和 y 兩個座標，一個學生有名字、年齡和成績。把它們分開存在幾個變數裡，很容易弄亂。**結構體**（structure）可以把幾個相關的資料組合成一個新的型別：

```c
#include <stdio.h>

struct point {               // 定義一個新型別：struct point
    int x;                   // 它有兩個「成員」：x 和 y
    int y;
};                           // 注意這裡的分號！

int main(void) {
    struct point a = {3, 4};               // 按順序初始化
    struct point b = {.y = 10, .x = -2};   // 按成員名初始化：順序隨意，沒寫的成員為 0
    a.x = a.x + 1;                         // 用 . 訪問成員
    printf("a = (%d, %d)\n", a.x, a.y);
    printf("b = (%d, %d)\n", b.x, b.y);
    printf("struct point 佔 %zu 位元組\n", sizeof(struct point));
    return 0;
}
```

- `struct point { ... };` 只是**定義了一種型別**（相當於一張設計圖），並沒有建立變數；`struct point a;` 才建立了一個變數。
- 定義結束的 `}` 後面**一定要有分號**。忘了的話，GCC 會報一個讓人摸不著頭腦的錯誤：`expected ';', identifier or '(' before 'int'`。
- 每個成員都像一個普通變數：`a.x` 可以賦值、參與計算、傳給 `scanf`（`scanf("%d", &a.x)`）。

> [!NOTE]
> 結構體的大小有時比各個成員加起來還大。比如 `struct { char c; int n; }` 通常佔 8 位元組而不是 5 位元組：編譯器會在成員之間插入「填充」位元組，讓 `int` 從 4 的倍數的地址開始，這樣 CPU 讀起來更快。

## 12.2 `typedef`：起一個簡短的名字

每次都寫 `struct point` 有點囉嗦。`typedef` 可以給型別起個別名：

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    double score;
} Student;                         // 以後直接寫 Student

int main(void) {
    Student s = {"Ada", 20, 95.5};
    Student t;
    strcpy(t.name, "Linus");       // 字元陣列成員要用 strcpy 賦值（第 9 課）
    t.age = 21;
    t.score = 88.0;
    printf("%-8s %3d %6.1f\n", s.name, s.age, s.score);
    printf("%-8s %3d %6.1f\n", t.name, t.age, t.score);
    return 0;
}
```

習慣上，自己定義的型別名用大寫字母開頭（`Student`、`Point`），和變數名區分開。

## 12.3 結構體的賦值、比較和函式

```c
#include <stdio.h>
#include <math.h>

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b) {             // 傳值：函式拿到的是副本
    double dx = a.x - b.x, dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
}

Point midpoint(Point a, Point b) {              // 結構體可以作為返回值
    Point m = {(a.x + b.x) / 2, (a.y + b.y) / 2};
    return m;
}

void move(Point *p, double dx, double dy) {     // 傳指標：可以修改呼叫者的結構體
    p->x += dx;                                 // p->x 就是 (*p).x
    p->y += dy;
}

int main(void) {
    Point a = {0, 0}, b = {3, 4};
    printf("距離 %.1f\n", distance(a, b));
    Point m = midpoint(a, b);
    printf("中點 (%.1f, %.1f)\n", m.x, m.y);

    Point c = b;                    // 結構體可以用 = 整體複製
    move(&c, 1, 1);
    printf("b = (%.1f, %.1f)，c = (%.1f, %.1f)\n", b.x, b.y, c.x, c.y);
    return 0;
}
```

- **賦值 `c = b` 會複製所有成員**——連結構體裡的陣列也會被複制。這一點和陣列不同（陣列不能整體賦值）。
- **結構體不能用 `==` 比較**，要逐個比較成員，最好寫成一個函式。
- 傳結構體給函式是**值傳遞**，函式拿到的是一份副本。要讓函式修改它，就傳指標。
- 透過指標訪問成員用 **`->`**：`p->x` 就是 `(*p).x`。注意 `*p.x` 是錯的：`.` 的優先順序比 `*` 高，它會被理解成 `*(p.x)`。
- 結構體很大時（比如有一個 1000 個元素的陣列成員），每次傳值都要複製一大塊記憶體。這時即使函式不修改它，也常常傳 `const Student *s`：只傳一個地址，又保證不修改。

## 12.4 結構體陣列

結構體陣列就像一張表格：每個元素是一行，每個成員是一列。用 `qsort`（第 10 課）可以按任意規則排序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int score;
} Student;

int by_score_desc(const void *a, const void *b) {
    const Student *x = a;                  // void * 可以直接賦給其他型別的指標
    const Student *y = b;
    if (x->score != y->score) {
        return (x->score < y->score) - (x->score > y->score);   // 分數高的排前面
    }
    return strcmp(x->name, y->name);       // 同分時按名字排
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

`cls[i].name`：先取第 `i` 個元素（一個 `Student`），再取它的成員 `name`。

## 12.5 列舉 `enum`

當一個變數只能取幾個固定的值時（星期幾、紅綠燈、遊戲角色的狀態），用**列舉**給每個值起個名字，比用 0、1、2 清楚得多：

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
            return "紅燈";
        case YELLOW:
            return "黃燈";
        case GREEN:
            return "綠燈";
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

- 列舉值其實就是整數常量，預設從 0 開始依次加 1；也可以自己指定，比如 `enum { LOW = 1, HIGH = 10 };`。列舉值名字習慣全用大寫。
- 用 `switch` 處理列舉時，如果漏掉了某個值，GCC 會警告 `enumeration value 'GREEN' not handled in switch`——這是列舉的另一個好處。

## 12.6 連結串列

陣列的元素在記憶體裡是連續的，要在開頭插入一個元素，就得把後面所有元素都往後挪。**連結串列**換了一種思路：每個元素（叫**結點**）單獨用 `malloc` 申請，每個結點裡除了資料，還有一個**指向下一個結點的指標**：

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
    struct Node *next;          // 指向下一個結點；最後一個結點的 next 是 NULL
} Node;

// 在連結串列頭部插入一個新結點，返回新的頭
Node *push_front(Node *head, int value) {
    Node *n = malloc(sizeof *n);
    if (n == NULL) {
        return head;            // 簡單起見：申請失敗就不插入
    }
    n->value = value;
    n->next = head;             // 新結點指向原來的第一個結點
    return n;                   // 新結點成為新的頭
}

void print_list(const Node *head) {
    for (const Node *p = head; p != NULL; p = p->next) {   // 順著 next 一個一個走
        printf("%d -> ", p->value);
    }
    printf("NULL\n");
}

void free_list(Node *head) {
    while (head != NULL) {
        Node *next = head->next;    // 先記下下一個結點，再釋放當前的
        free(head);
        head = next;
    }
}

int main(void) {
    Node *list = NULL;              // 空連結串列
    for (int i = 1; i <= 4; i++) {
        list = push_front(list, i * 10);
    }
    print_list(list);               // 40 -> 30 -> 20 -> 10 -> NULL
    free_list(list);
    return 0;
}
```

- 結構體裡要用到自己的型別時（`struct Node *next`），`typedef` 後面要寫上結構體名 `struct Node`，因為在成員列表裡，`Node` 這個別名還沒定義好。
- 遍歷連結串列的固定寫法：`for (p = head; p != NULL; p = p->next)`。
- 釋放連結串列時，**先記下 `next` 再 `free`**：`free(head)` 之後就不能再讀 `head->next` 了。
- 連結串列和陣列各有長處：連結串列在頭部插入、刪除很快，但要找第 i 個元素只能從頭一個一個數；陣列按下標訪問很快，但在中間插入要挪動元素。

## 12.7 常見錯誤

| 錯誤 | 編譯器說 | 正確做法 |
|------|----------|----------|
| 結構體定義後面忘了分號 | `expected ';', identifier or '(' before ...` | `struct point { ... };` |
| 沒有 `typedef` 卻直接寫型別名 | `unknown type name 'point'` | 寫 `struct point`，或者用 `typedef` |
| 對指標用 `.` | `'p' is a pointer; did you mean to use '->'?` | `p->x` |
| 對結構體變數用 `->` | `invalid type argument of '->'` | `a.x` |
| 用 `==` 比較結構體 | `invalid operands to binary ==` | 逐個比較成員 |
| 成員名拼錯 | `has no member named ...` | 對照結構體的定義 |
| 釋放連結串列結點後再讀 `->next` | 釋放後使用（記憶體檢查會報告） | 先儲存 `next` 再 `free` |

## 12.8 小結

- `struct 名字 { 成員... };` 定義結構體型別；`變數.成員` 訪問成員；可以按順序或用 `.成員 = 值` 初始化。
- `typedef struct { ... } Name;` 起別名；自引用時寫 `typedef struct Node { ...; struct Node *next; } Node;`。
- 結構體可以用 `=` 整體複製，可以作為引數和返回值（值傳遞）；不能用 `==` 比較。
- 指向結構體的指標用 `->`：`p->x` 就是 `(*p).x`；大結構體傳 `const T *`。
- 結構體陣列 + `qsort` 可以按任意規則排序。
- `enum` 給一組整數常量起名字，配合 `switch` 使用。
- 連結串列：每個結點 `malloc` 出來，用 `next` 指標串起來，最後一個指向 `NULL`；釋放時先儲存 `next`。

## 12.9 練習

<!-- exercise: point-dist -->
### 練習 1：兩點之間的距離

定義結構體 `Point`（成員是 `double x, y`）和函式 `double distance(Point a, Point b)`。輸入兩個點的座標 `x1 y1 x2 y2`，輸出它們之間的距離，保留 2 位小數。

```text
樣例輸入：0 0 3 4
樣例輸出：5.00
```

<details>
<summary>提示</summary>

照著 12.3 節寫。讀入時可以直接 `scanf("%lf %lf", &a.x, &a.y);`。
</details>

<!-- exercise: fraction-add -->
### 練習 2：分數相加

定義結構體 `Fraction`（分子 `num`、分母 `den`）。輸入 `a b c d`，表示 `a/b + c/d`（`b`、`d` 是正整數，`a`、`c` 可以是負數或 0），輸出化成最簡分數後的結果，格式 `p/q`（`q` 是正整數；結果是整數時 `q` 為 1，結果為 0 時輸出 `0/1`）。

```text
樣例輸入：1 2 1 3
樣例輸出：5/6
```

<details>
<summary>提示</summary>

`a/b + c/d = (a*d + c*b) / (b*d)`。再用第 7 課的輾轉相除法求分子絕對值和分母的最大公約數，把分子、分母都除以它。用 `long long` 避免溢位。
</details>

<!-- exercise: student-rank -->
### 練習 3：成績排名

第一行輸入人數 `n`（1 ≤ n ≤ 100），接下來 `n` 行，每行是一個學生的英文名（不超過 20 個字母）和成績（0～100 的整數）。按成績**從高到低**輸出，成績相同的按名字的字典序。每行輸出 `名字 成績`。

```text
樣例輸入：
4
Mia 88
Leo 95
Ava 88
Noah 72
樣例輸出：
Leo 95
Ava 88
Mia 88
Noah 72
```

<details>
<summary>提示</summary>

和 12.4 節一樣：定義 `Student` 結構體陣列，用 `scanf("%20s %d", s[i].name, &s[i].score)` 讀入，再用 `qsort` 加比較函式排序。
</details>

<!-- exercise: list-remove -->
### 練習 4：從連結串列中刪除

第一行輸入 `n`（0 ≤ n ≤ 1000），第二行輸入 `n` 個整數，按順序建立連結串列（新結點接在**末尾**）。第三行輸入一個整數 `x`。從連結串列中刪除所有值等於 `x` 的結點，然後輸出剩下的值，用空格隔開；如果連結串列空了，輸出 `empty`。

```text
樣例輸入：
6
3 1 3 4 3 5
3
樣例輸出：1 4 5
```

<details>
<summary>提示</summary>

建表時用一個 `tail` 指標記住最後一個結點，新結點接在 `tail->next`。刪除時最方便的寫法是用「指向指標的指標」：`Node **pp = &head;`，當 `(*pp)->value == x` 時，先儲存 `Node *dead = *pp;`，再讓 `*pp = dead->next;` 並 `free(dead)`；否則 `pp = &(*pp)->next;`。別忘了最後釋放整個連結串列。
</details>

<!-- exercise: struct-predict -->
### 練習 5：讀程式碼，寫輸出

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
<summary>答案與解析</summary>

`Box y = x;` 複製了所有成員（包括陣列），所以改 `y.a[0]` 不影響 `x.a[0]`。`set_n` 拿到的是副本，`x.n` 還是 3。`set_n_ptr` 透過指標修改了 `y.n`。輸出：

```text
1 100 3
99
```
</details>

<!-- exercise: arrow -->
### 練習 6：訪問成員

`p` 是一個 `Point *`（指向 `Point` 結構體的指標），下面哪個表示式能得到它指向的點的 `x` 座標？

- `p.x`
- `p->x`
- `*p.x`
- `&p->x`

<details>
<summary>答案與解析</summary>

`p->x` 等於 `(*p).x`。`p.x` 對指標用了 `.`；`*p.x` 因為 `.` 優先順序更高，會被理解成 `*(p.x)`；`&p->x` 是 `x` 成員的地址。
</details>

## 12.10 延伸閱讀

- [cppreference：結構體宣告](https://zh.cppreference.com/w/c/language/struct)、[列舉](https://zh.cppreference.com/w/c/language/enum)、[typedef](https://zh.cppreference.com/w/c/language/typedef)
- [VisuAlgo：連結串列動畫](https://visualgo.net/zh/list)
- 下一課：[檔案](../13-files/zh-CN.md)
