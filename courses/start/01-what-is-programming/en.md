# 1. What is programming?

> 🎯 **Goals**: understand the words "program", "programming language", "compile" and "interpret"; get to know the learning app; run your first piece of code.
>
> ⏱ **Time**: 20 minutes　|　📋 **You need**: nothing

## 1.1 A program is a recipe

Imagine teaching a robot to make scrambled eggs. The robot obeys **every** instruction but has **zero** common sense. "Make breakfast" won't work; you must spell out every step:

```text
1. Open the fridge, take out 2 eggs and the butter
2. Crack the eggs into a bowl and whisk 30 times
3. Put the pan on the stove, turn the heat to medium
4. Add 1 teaspoon of butter, wait 1 minute
5. Pour in the eggs, stir until set
6. Add a pinch of salt
7. Turn off the heat and serve
```

That list is a **program**: a sequence of **precise instructions carried out in order**. The computer is the robot — it is incredibly fast (billions of instructions per second) and never gets tired, but it **never guesses what you meant**. It does exactly what you wrote; if you wrote something wrong, it does that too (or stops and reports an error).

**Programming** is the act of writing such instruction lists.

> [!NOTE]
> Everything you use every day is a program: messaging apps, browsers, games, your phone's calculator, ticket gates in the metro, the control systems in cars… All of them are lines of code written by programmers.

## 1.2 Why do we need programming languages?

The only thing a computer understands directly is **machine code** — strings of 0s and 1s like:

```text
10111000 00000001 00000000 00000000 00000000
```

That's very hard for people to read or write. So people invented **programming languages**: strict "in-between" languages that sit between human language and machine code. For example, here's Python showing a message:

```python
print("Hello!")
```

Much easier, isn't it? Text written in a programming language is called **source code**, or just **code**.

### Why are there so many languages?

Just like screwdrivers, hammers and spanners, different languages are good at different jobs:

| Language | Strengths | Typical uses |
|----------|-----------|--------------|
| **Python** | simple, readable, huge library | first language, data analysis, AI, automation scripts |
| **JavaScript / TypeScript** | the only language browsers run natively | web pages, front-ends, back-ends too |
| **Java** | stable, mature ecosystem | enterprise back-ends, Android apps |
| **C** | close to the hardware, fast | operating systems, embedded devices, microcontrollers |
| **C++** | fast and good for big programs | game engines, browsers, graphics software |
| **Rust** | fast, and the compiler prevents memory bugs | system software, CLI tools, high-performance services |
| **Go** | simple, great at concurrency | cloud services, servers, DevOps tools (Docker, Kubernetes) |
| **Assembly** | almost one line per machine instruction | understanding the CPU, optimisation, security research |

> [!TIP]
> Your first language is the hardest. After that, most ideas (variables, conditions, loops, functions) carry over — only the spelling changes. So **pick an easy first language**: this course recommends Python.

## 1.3 How does code become something the computer can run?

Source code must be "translated" into instructions the machine can execute. There are two main ways.

**Compiling** — like **translating a whole book**:

```text
source main.c ──[compiler]──► executable main.exe ──► run
              (translate everything once)   (then run it directly, fast)
```

C, C++, Rust and Go work this way. The translator is called a **compiler**. If the code has a grammar mistake, the compiler refuses to translate and tells you where the problem is.

**Interpreting** — like **live interpretation**:

```text
source main.py ──[interpreter: read a line, run a line]──► results
```

Python and JavaScript work this way. The **interpreter** reads and executes as it goes, so you can run code the moment you've written it — great for experimenting.

Java is in between: it is compiled into an intermediate format (bytecode), which the Java Virtual Machine then runs.

> [!NOTE]
> You don't need to memorise these categories. Just remember: **code must go through a compiler or an interpreter before it runs**, and when they find a problem they print an **error message**. Reading error messages is one of the most important programming skills (lesson 5 is all about it).

## 1.4 A tour of the learning app

The app that comes with this course has three columns:

```text
┌──────────────┬──────────────────────────┬─────────────────────────────┐
│ ① Courses     │ ② Lesson / Exercises     │ ③ Code editor     [▶ Run]   │
│              │                          │                             │
│ 🚀 Start here │  you are reading this    │  write code here             │
│   1. What is…│                          │                             │
│ 🐍 Python    │  code boxes have a        ├─────────────────────────────┤
│ ...          │  "▶ Try it" button        │ ④ Output                     │
│              │                          │   output, errors, explanations│
│ 0/115 done   │ [◀ Previous] [Next ▶]     │ ⑤ Input box for your program │
└──────────────┴──────────────────────────┴─────────────────────────────┘
```

1. **Courses**: click a course to open it, then click a lesson. Finished lessons get a ✓.
2. **Lesson / Exercises**: two tabs at the top. Read the lesson, then solve the "✍️ Exercises".
3. **Code editor**: write code here. Choose the language at the top left.
4. **Output**: click "▶ Run" (or press <kbd>F5</kbd>) and the program's output appears here. When something goes wrong, a "💬 What does this mean?" box explains it in plain words.
5. **Input box**: some programs wait for you to type something — type here and press Enter.

Other handy features:

- The "View" menu can **enlarge the text** (<kbd>Ctrl</kbd> + <kbd>+</kbd>), switch to **dark mode** and change the **interface language**.
- "Tools → Check developer tools" shows which programming tools are installed and how to install the missing ones.
- Your progress and your code are **saved automatically**.

## 1.5 Hands on: run your first code

Click "▶ Try it" above the box below; the code appears in the editor on the right. Then click "▶ Run":

```python
print("Hello, I've started learning to code!")
print("1 + 1 =", 1 + 1)
print("A year has", 365 * 24 * 60, "minutes")
```

You should see:

```text
Hello, I've started learning to code!
1 + 1 = 2
A year has 525600 minutes
```

Now **change something**: write your own sentence on the first line, change `365` to `366` (a leap year), and run it again.

> [!TIP]
> The best way to learn programming is **change the code, run it, look at the result**. Don't be afraid of breaking things — if it gets messy, load the example again. A computer can't be damaged by wrong code.

### Make a mistake on purpose

Delete the closing `)` at the end of the first line and run again:

```python norun
print("Hello, I've started learning to code!"
```

You'll see a red error message and an explanation below it. That's everyday programming: **write → run → read the error → fix → run again**.

## 1.6 What skills do you need?

Many people worry that they're "bad at maths" or "bad at English". Actually:

- **Maths**: at the start you only need primary-school arithmetic. Games and AI need more maths later, and you can learn it then.
- **English**: a language has only a few dozen keywords (`print`, `if`, `for`, `while`…), and you'll remember them by using them. Error messages are explained by the app, and lesson 5 has a word list.
- **What really matters**: patience, attention to detail (every symbol counts) and the habit of breaking a big problem into small steps — all of which grow with practice.

## 1.7 Summary

- A **program** is a precise list of instructions executed in order; **programming** is writing them.
- Computers only understand machine code; **programming languages** let people write programs in something closer to human language. What we write is **source code**.
- Source code is processed by a **compiler** (translate everything first) or an **interpreter** (read and run as it goes).
- The app: choose a lesson on the left, read and practise in the middle, write code on the right and click "▶ Run".
- The secret: change things, run often, read error messages carefully.

## 1.8 Exercises

<!-- exercise: what-is-program -->
### Exercise 1: What is a program?

Which sentence describes a "program" best?

- The hardware of a computer
- A sequence of precise instructions carried out in order
- An app you can chat with
- A secret code only experts understand

<details>
<summary>Answer and explanation</summary>

A program is a list of instructions, like a recipe. Hardware is the physical computer; a chat app is just one example of a program.
</details>

<!-- exercise: compile-or-interpret -->
### Exercise 2: Compile or interpret?

"First translate the whole source code into an executable file, then run that file directly." What is this called?

- Interpreting
- Compiling
- Downloading
- Copying

<details>
<summary>Answer and explanation</summary>

Compiling. C, C++, Rust and Go are compiled before running; Python and JavaScript are usually interpreted.
</details>

<!-- exercise: first-code -->
### Exercise 3: Your first code

Write a program that prints these two lines:

```text
Hello!
1 + 2 = 3
```

The `3` on the second line must be **calculated** by the computer, not typed inside the quotes.

<details>
<summary>Hint</summary>

Line one: `print("Hello!")`. Line two: `print("1 + 2 =", 1 + 2)` — the comma adds a space between the two parts.
</details>

<!-- exercise: predict-calc -->
### Exercise 4: Read the code, predict the output

What does this program print? Think first, then type your answer.

```python
print(2 * 3)
print("2 * 3")
```

<details>
<summary>Answer and explanation</summary>

The first line has no quotes, so the computer calculates `2 * 3` and prints `6`; the second line is text in quotes and is printed as it is: `2 * 3`.
</details>

## 1.9 Further reading

- [CS50x](https://cs50.harvard.edu/x/): Harvard's free introduction to computer science; lecture 0 explains "what is computer science" with lots of fun examples.
- [Scratch](https://scratch.mit.edu/): try programming by snapping blocks together, if you'd like to "play" first.
- Next lesson: [Computer basics: files, folders and typing](../02-computer-basics/en.md)
