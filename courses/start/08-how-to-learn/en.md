# 8. How to learn programming well

> 🎯 **Goals**: choose a learning path for your goal; practise effectively; know where to find resources and project ideas; set up a plan you can keep.
>
> ⏱ **Time**: 25 minutes　|　📋 **You need**: nothing

Congratulations on finishing the first seven lessons of "Start here"! The last lesson isn't about technology but **method** — with the same 100 hours, a good method can make you several times more effective.

## 8.1 Choose a path

Don't learn several languages at once. Build solid **programming thinking** in one language first, then branch out:

```text
                    ┌──► JavaScript/TypeScript ──► web front-end, full stack
                    │
Start here ──► Python ┼──► Java ──► enterprise back-ends, Android
                    │
                    ├──► Go ──► cloud services, servers, DevOps tools
                    │
                    ├──► C ──► Assembly ──► how computers work, embedded, security
                    │     └──► C++ / Rust ──► games, high-performance and systems software
                    │
                    └──► LLM-Tutorial ──► artificial intelligence
```

| Your goal | Suggested path |
|-----------|----------------|
| learn to program, direction undecided | Python (the whole course) |
| build websites or browser games | Python lessons 1–8 → JavaScript/TypeScript |
| study computer science / strong fundamentals | Python → C → Assembly → C++ |
| get a back-end developer job | Python → Java or Go |
| game engines, high-performance code | Python → C → C++ or Rust |
| AI and data analysis | Python → LLM-Tutorial |

## 8.2 The most effective method: do it yourself

### 1. Type every example yourself

Understanding is not the same as being able to write. When reading, **type each example into the editor yourself** (don't copy-paste), run it, then **change it**: another number, an extra line, a deliberate mistake — and see what happens.

### 2. Finish every exercise

The exercises are designed to show whether you really understood. When stuck:

1. reread the relevant part of the lesson;
2. open one "💡 Hint" at a time;
3. look at the solution only as a last resort — **then close it and write the solution again from scratch**.

### 3. A little, every day

30–60 minutes a day beats 8 hours once a weekend. The brain needs sleep to consolidate new knowledge. A streak of days matters more than the length of one session.

### 4. Recall actively instead of rereading

After a lesson, close it and try to **explain in your own words** what it covered and write the key code. Whatever you can't recall is what to review. Redo old exercises a few days later — this is **spaced repetition**, and it works.

### 5. Start projects early

Once you know loops and functions you can start small projects. Projects tie scattered knowledge together and are the most rewarding part:

- 🎲 a number-guessing game, rock-paper-scissors
- 🧮 a calculator, unit converter, personal expense tracker
- 📝 a to-do list saved to a file
- 🔐 a password generator, a tool to rename many files
- 🌐 fetch weather data, your own home page
- 🎮 Snake, Tetris (after learning a graphics library)

Projects that are **a bit harder than the exercises but not impossible** teach you the most.

## 8.3 Where to find resources

| Type | Recommendations |
|------|-----------------|
| official docs | each language's official tutorial is the most accurate and current (linked in every lesson's "Further reading") |
| practice sites | [Exercism](https://exercism.org/) (free, with mentors), [LeetCode](https://leetcode.com/), [Codewars](https://www.codewars.com/) |
| open-source guides | search GitHub for `awesome-<language>` for curated lists, e.g. [awesome-python](https://github.com/vinta/awesome-python) |
| algorithms | [The Algorithms](https://github.com/TheAlgorithms): classic algorithms in many languages |
| project ideas | [build-your-own-x](https://github.com/codecrafters-io/build-your-own-x): build your own database, language, game… from scratch |
| Q&A | [Stack Overflow](https://stackoverflow.com/), language forums and Discord servers |
| video | [CS50](https://cs50.harvard.edu/x/) (Harvard's intro to computer science) |

## 8.4 What if English isn't your strong point?

Programming needs much less English than you'd think:

1. There are only a few dozen **keywords**; you'll remember them by using them.
2. **Error messages** follow templates: lesson 5's word list plus the app's explanations cover most cases.
3. Use simple English words for **variable names** (`count`, `total`, `name`, `price`). Not sure of a word? Look it up — anything is better than `a1`, `b2`.
4. Use your browser's translation for English docs to get the gist, but **don't translate the code**.
5. Learn three programming words a day; in three months you'll be surprised.

## 8.5 Health and mindset

- 🧘 **Step away when stuck**: a walk or a night's sleep often makes the bug obvious.
- 🐢 **Don't compare your progress with others'**: everyone starts somewhere different. Compare with yesterday's you.
- 📓 **Keep notes** of the mistakes you made and how you fixed them — a resource only you have.
- 👀 **Look after your eyes and neck**: a 5-minute break every 45 minutes, dark mode, larger text ("View → Larger text").
- 🎯 **Allow yourself to be imperfect**: make it work first, then make it better.

## 8.6 Make a plan

An example plan (about an hour a day):

| Phase | Pace | Content |
|-------|------|---------|
| weeks 1–2 | one lesson a day | Start here (8 lessons) + Python lessons 1–4 |
| weeks 3–6 | one lesson every 2 days | Python lessons 5–12, a small project every week |
| weeks 7–8 | | Python lessons 13–16 and the final project |
| afterwards | | choose a second language from 8.1 |

## 8.7 Summary

- Master one language first (Python recommended), then branch out by goal.
- Type every example, finish every exercise; a little every day; recall actively and review with spacing.
- Start projects early — slightly harder than the exercises is ideal.
- Prefer official docs; use practice sites and great open-source repositories.
- English grows as you go; take breaks when stuck and compare yourself with yesterday.

## 8.8 Exercises

<!-- exercise: best-method -->
### Exercise 1: The best way to learn

You meet a code example while reading. Which approach teaches you the most?

- Skip it once you understand it
- Copy, paste and run it
- Type it yourself, run it, then change it and see what happens
- Memorise it

<details>
<summary>Answer and explanation</summary>

Typing it yourself strengthens memory, and changing and experimenting shows you what each part really does.
</details>

<!-- exercise: stuck -->
### Exercise 2: Stuck on an exercise

You've been stuck on an exercise for 10 minutes. What's the best next step?

- Look at the answer and copy it
- Give up on it
- Reread the relevant part of the lesson, then open one hint
- Switch to another language

<details>
<summary>Answer and explanation</summary>

Look for clues yourself first (the lesson), then get help gradually (hints). If you end up reading the solution, close it and write it again yourself.
</details>

<!-- exercise: plan -->
### Exercise 3: My learning pledge

Write a program that prints your plan (exactly these three lines):

```text
Goal: learn Python
Time: 1 hour every day
Start: today!
```

<details>
<summary>Hint</summary>

Three `print` calls. Note the space after each colon and the exclamation mark at the end.
</details>

## 8.9 Further reading

- *The Pragmatic Programmer* (Thomas & Hunt): the classic book on becoming a good programmer.
- [Teach Yourself Programming in Ten Years](https://norvig.com/21-days.html): Peter Norvig's famous essay — learning to program takes patience.
- Next step: start [Python](../../python/01-hello/en.md)!
