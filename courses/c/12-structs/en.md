# 12. Structs, enums and linked lists

> 🎯 **Goals**: combine related data into a new type with **`struct`**; give it a short name with `typedef`; understand struct assignment (a full copy) and comparison (not with `==`), pass structs to and return them from functions, and change them through pointers with `->`; sort arrays of structs; name a set of values with **`enum`**; build a **linked list** from structs and pointers.
>
> ⏱ **Time**: 100 minutes　|　📋 **You need**: lessons 1–11 of this course

## 12.1 Structs: keeping related data together

A point has x and y coordinates; a student has a name, an age and a mark. Keeping them in separate variables is an easy way to get them mixed up. A **struct** (structure) combines several related values into a new type:

```c
#include <stdio.h>

struct point {               // define a new type: struct point
    int x;                   // it has two "members": x and y
    int y;
};                           // note the semicolon here!

int main(void) {
    struct point a = {3, 4};               // initialise in order
    struct point b = {.y = 10, .x = -2};   // initialise by member name: any order; members left out are 0
    a.x = a.x + 1;                         // reach members with .
    printf("a = (%d, %d)\n", a.x, a.y);
    printf("b = (%d, %d)\n", b.x, b.y);
    printf("struct point takes %zu bytes\n", sizeof(struct point));
    return 0;
}
```

- `struct point { ... };` only **defines a type** (a blueprint) and creates no variable; `struct point a;` creates one.
- The `}` that ends the definition **must be followed by a semicolon**. Forget it and GCC gives a baffling error: `expected ';', identifier or '(' before 'int'`.
- Each member works like an ordinary variable: `a.x` can be assigned, used in calculations and passed to `scanf` (`scanf("%d", &a.x)`).

> [!NOTE]
> A struct is sometimes bigger than its members added up. For example, `struct { char c; int n; }` usually takes 8 bytes, not 5: the compiler inserts "padding" bytes between members so that the `int` starts at an address that's a multiple of 4, which the CPU reads faster.

## 12.2 `typedef`: a shorter name

Writing `struct point` every time gets tedious. `typedef` gives a type another name:

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    double score;
} Student;                         // from now on, just write Student

int main(void) {
    Student s = {"Ada", 20, 95.5};
    Student t;
    strcpy(t.name, "Linus");       // a char-array member is assigned with strcpy (lesson 9)
    t.age = 21;
    t.score = 88.0;
    printf("%-8s %3d %6.1f\n", s.name, s.age, s.score);
    printf("%-8s %3d %6.1f\n", t.name, t.age, t.score);
    return 0;
}
```

By convention, your own type names start with a capital letter (`Student`, `Point`), which sets them apart from variable names.

## 12.3 Assigning, comparing and passing structs

```c
#include <stdio.h>
#include <math.h>

typedef struct {
    double x;
    double y;
} Point;

double distance(Point a, Point b) {             // by value: the function gets copies
    double dx = a.x - b.x, dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
}

Point midpoint(Point a, Point b) {              // a struct can be a return value
    Point m = {(a.x + b.x) / 2, (a.y + b.y) / 2};
    return m;
}

void move(Point *p, double dx, double dy) {     // by pointer: changes the caller's struct
    p->x += dx;                                 // p->x is (*p).x
    p->y += dy;
}

int main(void) {
    Point a = {0, 0}, b = {3, 4};
    printf("distance %.1f\n", distance(a, b));
    Point m = midpoint(a, b);
    printf("midpoint (%.1f, %.1f)\n", m.x, m.y);

    Point c = b;                    // structs can be copied whole with =
    move(&c, 1, 1);
    printf("b = (%.1f, %.1f), c = (%.1f, %.1f)\n", b.x, b.y, c.x, c.y);
    return 0;
}
```

- **The assignment `c = b` copies every member** — even arrays inside the struct. That's different from arrays, which can't be assigned at all.
- **Structs can't be compared with `==`**: compare the members one by one, ideally in a function.
- Passing a struct to a function is **pass by value**: the function gets a copy. To let it change the struct, pass a pointer.
- To reach a member through a pointer, use **`->`**: `p->x` is `(*p).x`. Note that `*p.x` is wrong: `.` binds more tightly than `*`, so it means `*(p.x)`.
- When a struct is large (say, with a 1000-element array member), passing it by value copies a big block of memory every time. So even when a function doesn't change it, you'll often see `const Student *s`: only an address is passed, with a promise not to change anything.

## 12.4 Arrays of structs

An array of structs is like a table: each element is a row and each member a column. With `qsort` (lesson 10) you can sort it by any rule:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int score;
} Student;

int by_score_desc(const void *a, const void *b) {
    const Student *x = a;                  // a void * can be assigned straight to another pointer type
    const Student *y = b;
    if (x->score != y->score) {
        return (x->score < y->score) - (x->score > y->score);   // higher marks first
    }
    return strcmp(x->name, y->name);       // equal marks: by name
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

`cls[i].name`: first take element `i` (a `Student`), then its member `name`.

## 12.5 Enumerations: `enum`

When a variable can only take a few fixed values (days of the week, traffic lights, a game character's state), an **enumeration** names each value — much clearer than 0, 1, 2:

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
            return "red";
        case YELLOW:
            return "yellow";
        case GREEN:
            return "green";
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

- Enumeration values are just integer constants, starting at 0 and going up by 1 by default; you can also choose them, e.g. `enum { LOW = 1, HIGH = 10 };`. Their names are usually written in capitals.
- When a `switch` over an enum leaves a value out, GCC warns `enumeration value 'GREEN' not handled in switch` — another benefit of enums.

## 12.6 Linked lists

An array's elements sit next to each other in memory, so inserting at the front means moving every other element along. A **linked list** works differently: each element (a **node**) is allocated separately with `malloc`, and besides its data each node holds **a pointer to the next node**:

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
    struct Node *next;          // points to the next node; the last node's next is NULL
} Node;

// Insert a new node at the front of the list and return the new head
Node *push_front(Node *head, int value) {
    Node *n = malloc(sizeof *n);
    if (n == NULL) {
        return head;            // to keep it simple: if malloc fails, don't insert
    }
    n->value = value;
    n->next = head;             // the new node points to the old first node
    return n;                   // and becomes the new head
}

void print_list(const Node *head) {
    for (const Node *p = head; p != NULL; p = p->next) {   // follow next, one node at a time
        printf("%d -> ", p->value);
    }
    printf("NULL\n");
}

void free_list(Node *head) {
    while (head != NULL) {
        Node *next = head->next;    // remember the next node first, then free this one
        free(head);
        head = next;
    }
}

int main(void) {
    Node *list = NULL;              // an empty list
    for (int i = 1; i <= 4; i++) {
        list = push_front(list, i * 10);
    }
    print_list(list);               // 40 -> 30 -> 20 -> 10 -> NULL
    free_list(list);
    return 0;
}
```

- When a struct refers to its own type (`struct Node *next`), give the struct a name after `typedef struct` — `struct Node` — because inside the member list the alias `Node` doesn't exist yet.
- The standard way to walk a list: `for (p = head; p != NULL; p = p->next)`.
- When freeing a list, **save `next` before calling `free`**: after `free(head)` you can no longer read `head->next`.
- Lists and arrays each have strengths: a list inserts and deletes at the front quickly, but finding element i means counting from the start; an array reaches any index instantly, but inserting in the middle means moving elements.

## 12.7 Common mistakes

| Mistake | The compiler says | Do this instead |
|---------|-------------------|-----------------|
| no semicolon after a struct definition | `expected ';', identifier or '(' before ...` | `struct point { ... };` |
| using a bare type name without `typedef` | `unknown type name 'point'` | write `struct point`, or use `typedef` |
| `.` on a pointer | `'p' is a pointer; did you mean to use '->'?` | `p->x` |
| `->` on a struct variable | `invalid type argument of '->'` | `a.x` |
| comparing structs with `==` | `invalid operands to binary ==` | compare member by member |
| a misspelt member name | `has no member named ...` | check the struct's definition |
| reading `->next` after freeing a node | use after free (reported by the memory check) | save `next` before `free` |

## 12.8 Summary

- `struct name { members... };` defines a struct type; `variable.member` reaches a member; initialise in order or with `.member = value`.
- `typedef struct { ... } Name;` gives it a short name; for self-reference write `typedef struct Node { ...; struct Node *next; } Node;`.
- Structs can be copied whole with `=` and passed and returned by value; they can't be compared with `==`.
- Use `->` with a pointer to a struct: `p->x` is `(*p).x`; pass large structs as `const T *`.
- Arrays of structs plus `qsort` sort by any rule.
- `enum` names a set of integer constants; use it with `switch`.
- Linked list: each node comes from `malloc`, nodes are chained by `next` pointers, and the last points to `NULL`; save `next` before freeing.

## 12.9 Exercises

<!-- exercise: point-dist -->
### Exercise 1: Distance between two points

Define a struct `Point` (with members `double x, y`) and a function `double distance(Point a, Point b)`. Read the coordinates of two points, `x1 y1 x2 y2`, and print the distance between them to 2 decimal places.

```text
Sample input: 0 0 3 4
Sample output: 5.00
```

<details>
<summary>Hint</summary>

Follow section 12.3. You can read straight into the members: `scanf("%lf %lf", &a.x, &a.y);`.
</details>

<!-- exercise: fraction-add -->
### Exercise 2: Adding fractions

Define a struct `Fraction` (numerator `num`, denominator `den`). Read `a b c d`, meaning `a/b + c/d` (`b` and `d` are positive; `a` and `c` may be negative or 0), and print the result in lowest terms as `p/q` (`q` is positive; when the result is a whole number `q` is 1, and a zero result is `0/1`).

```text
Sample input: 1 2 1 3
Sample output: 5/6
```

<details>
<summary>Hint</summary>

`a/b + c/d = (a*d + c*b) / (b*d)`. Then use lesson 7's Euclid's algorithm to find the greatest common divisor of the numerator's absolute value and the denominator, and divide both by it. Use `long long` to avoid overflow.
</details>

<!-- exercise: student-rank -->
### Exercise 3: Ranking

The first line holds the number of students `n` (1 ≤ n ≤ 100); each of the next `n` lines holds a student's English name (at most 20 letters) and mark (an integer from 0 to 100). Print them by mark **from highest to lowest**, and students with equal marks in dictionary order of their names, one `name mark` per line.

```text
Sample input:
4
Mia 88
Leo 95
Ava 88
Noah 72
Sample output:
Leo 95
Ava 88
Mia 88
Noah 72
```

<details>
<summary>Hint</summary>

As in section 12.4: define an array of `Student` structs, read with `scanf("%20s %d", s[i].name, &s[i].score)`, then sort with `qsort` and a comparison function.
</details>

<!-- exercise: list-remove -->
### Exercise 4: Removing from a linked list

The first line holds `n` (0 ≤ n ≤ 1000) and the second `n` integers; build a linked list in that order (new nodes go at the **end**). The third line holds an integer `x`. Remove every node whose value is `x`, then print the remaining values separated by spaces, or `empty` if nothing is left.

```text
Sample input:
6
3 1 3 4 3 5
3
Sample output: 1 4 5
```

<details>
<summary>Hint</summary>

While building, keep a `tail` pointer to the last node and attach new nodes at `tail->next`. For removal, a "pointer to a pointer" is handiest: `Node **pp = &head;`; when `(*pp)->value == x`, save `Node *dead = *pp;`, set `*pp = dead->next;` and `free(dead)`; otherwise `pp = &(*pp)->next;`. Don't forget to free the whole list at the end.
</details>

<!-- exercise: struct-predict -->
### Exercise 5: Predict the output

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
<summary>Answer and explanation</summary>

`Box y = x;` copies every member (the array included), so changing `y.a[0]` doesn't affect `x.a[0]`. `set_n` gets a copy, so `x.n` is still 3. `set_n_ptr` changes `y.n` through the pointer. The output:

```text
1 100 3
99
```
</details>

<!-- exercise: arrow -->
### Exercise 6: Reaching a member

`p` is a `Point *` (a pointer to a `Point` struct). Which expression gives the `x` coordinate of the point it points to?

- `p.x`
- `p->x`
- `*p.x`
- `&p->x`

<details>
<summary>Answer and explanation</summary>

`p->x` is `(*p).x`. `p.x` uses `.` on a pointer; `*p.x` means `*(p.x)` because `.` binds more tightly; and `&p->x` is the address of the `x` member.
</details>

## 12.10 Further reading

- [cppreference: struct declaration](https://en.cppreference.com/w/c/language/struct), [enumerations](https://en.cppreference.com/w/c/language/enum), [typedef](https://en.cppreference.com/w/c/language/typedef)
- [VisuAlgo: linked list animations](https://visualgo.net/en/list)
- Next lesson: [Files](../13-files/en.md)
