# 6. How computers store numbers and text

> 🎯 **Goals**: understand binary, bits and bytes; convert between decimal, binary and hexadecimal; know why `0.1 + 0.2` isn't `0.3`; understand how ASCII, Unicode and UTF-8 let computers show text.
>
> ⏱ **Time**: 40 minutes　|　📋 **You need**: lesson 1

This lesson is a little theoretical, but it explains many strange things you'll meet later: why numbers have limits, why decimals aren't exact, why text sometimes turns into garbage.

## 6.1 Everything is 0s and 1s

Inside a computer are billions of tiny electronic switches (transistors). Each has only two states: **off (0)** and **on (1)**. So everything in a computer — numbers, text, pictures, sound, programs — is ultimately made of 0s and 1s.

- A single 0 or 1 is a **bit**, the smallest unit of information.
- **8 bits** make a **byte**, the basic unit of storage.

```text
1 byte = 8 bits:  0 1 0 0 0 0 0 1
1 KB ≈ 1000 bytes     (a page of plain text)
1 MB ≈ 1000 KB        (a compressed song)
1 GB ≈ 1000 MB        (a film)
1 TB ≈ 1000 GB        (a hard drive)
```

> [!NOTE]
> Computers often count in 1024s instead of 1000s (1 KiB = 1024 bytes). That's why a "1 TB" drive shows up as about 931 GB — the two sides use different units.

## 6.2 Binary: counting with only 0 and 1

We normally use **decimal**: ten digits 0–9, and we carry when we reach ten. The places stand for 1, 10, 100:

```text
decimal 345 = 3×100 + 4×10 + 5×1
```

**Binary** has only 0 and 1 and carries at two. The places stand for 1, 2, 4, 8, 16… (powers of two):

```text
binary 1101 = 1×8 + 1×4 + 0×2 + 1×1 = 13 (decimal)

place values:  8   4   2   1
binary:        1   1   0   1
```

Counting from 0 to 8:

| Decimal | Binary |
|---------|--------|
| 0 | 0 |
| 1 | 1 |
| 2 | 10 |
| 3 | 11 |
| 4 | 100 |
| 5 | 101 |
| 6 | 110 |
| 7 | 111 |
| 8 | 1000 |

### Decimal to binary: keep dividing by 2

To convert 13: divide by 2 repeatedly, write down the remainders, and read them **from the bottom up**:

```text
13 ÷ 2 = 6 remainder 1   ↑
 6 ÷ 2 = 3 remainder 0   │
 3 ÷ 2 = 1 remainder 1   │ bottom to top: 1101
 1 ÷ 2 = 0 remainder 1   │
```

Check it with Python:

```python
print(bin(13))          # decimal → binary; 0b means "this is binary"
print(int("1101", 2))   # binary text → decimal
print(0b1101)           # write a binary number directly
```

## 6.3 Hexadecimal: shorthand for binary

Binary is long and hard for people to read. **Hexadecimal** (hex) has 16 digits: 0–9 and A–F (A=10, B=11 … F=15). **One hex digit is exactly four binary digits**, so hex is a handy shorthand:

```text
binary 1111 1111  →  hex F F  →  decimal 255
binary 0100 0001  →  hex 4 1  →  decimal 65
```

You'll see hex everywhere: colour codes `#FF8800` (red FF, green 88, blue 00), memory addresses `0x7ffd5a3c`, character codes `U+4E2D`.

```python
print(hex(255))         # 0xff
print(0xFF)             # 255
print(int("ff", 16))    # 255
```

## 6.4 How big can an integer be?

Computers store an integer in a **fixed number of bits**, which limits the range:

| Bits | Unsigned (≥ 0 only) | Signed (can be negative) |
|------|---------------------|--------------------------|
| 8 | 0 to 255 | −128 to 127 |
| 16 | 0 to 65,535 | −32,768 to 32,767 |
| 32 | 0 to about 4.29 billion | about −2.15 to 2.15 billion |
| 64 | 0 to about 1.8×10¹⁹ | about ±9.2×10¹⁸ |

n bits can represent 2ⁿ different values: 8 bits give 2⁸ = 256 values, hence 0–255.

If a result doesn't fit, you get an **overflow** — like an old car odometer rolling over from 999999 to 000000. Integers in C, C++, Java, Rust and Go have fixed sizes, so watch out for overflow; Python's integers simply grow, so it never happens there:

```python
print(2 ** 100)    # Python computes the exact result
```

> [!NOTE]
> Negative numbers are usually stored in a form called **two's complement**. No need to dig in now — the C and assembly courses explain it.

## 6.5 Decimals: why is 0.1 + 0.2 not 0.3?

Try this:

```python
print(0.1 + 0.2)
print(0.1 + 0.2 == 0.3)
```

Output:

```text
0.30000000000000004
False
```

This isn't a Python bug; almost every language does it. Computers store fractions in binary (**floating-point numbers**), and 0.1 is an **endlessly repeating fraction** in binary — just as 1/3 is 0.3333… in decimal. The computer stores a very close approximation, and adding approximations gives a tiny error.

What to do:

- **Round** when displaying: `round(0.1 + 0.2, 2)` gives `0.3`.
- When comparing decimals, check whether they're **close enough**, not exactly equal: `abs(a - b) < 1e-9`.
- **Don't use floats for money**: store whole cents (1999 cents = 19.99) or use a decimal type (Python's `decimal` module).

```python
print(round(0.1 + 0.2, 2))
print(abs((0.1 + 0.2) - 0.3) < 1e-9)
```

## 6.6 Text: every character has a number

Computers store text by **giving every character a number** and storing the numbers.

### ASCII: the first standard

**ASCII**, created in the 1960s, numbered 128 characters: English letters, digits, punctuation and some control characters:

| Character | Code (decimal) | Code (hex) |
|-----------|----------------|------------|
| `A` | 65 | 41 |
| `B` | 66 | 42 |
| `a` | 97 | 61 |
| `0` | 48 | 30 |
| space | 32 | 20 |

Note: the character `'0'` has code 48, **not** 0. "The number 0" and "the text 0" are different things to a computer.

```python
print(ord("A"))     # character → code
print(chr(66))      # code → character
print(ord("0"))
```

### Unicode: every writing system

ASCII only covers English. To represent Chinese, Russian, Arabic, emoji and everything else, there is **Unicode**: a unique number, the **code point**, for practically every character in the world, written `U+` plus hex. It contains more than 150,000 characters:

```text
A   → U+0041
中  → U+4E2D
я   → U+044F
😀  → U+1F600
```

```python
print(ord("я"))            # 1103
print(hex(ord("中")))      # 0x4e2d
print(chr(0x1F600))        # 😀
```

### UTF-8: storing code points as bytes

A code point is a number; to store it in a file we must decide how many bytes to use and in what order — the **encoding**. **UTF-8** is by far the most common:

- ASCII characters such as English letters: **1 byte** (fully compatible with ASCII);
- Cyrillic, Greek and similar: **2 bytes**;
- most Chinese, Japanese and Korean characters: **3 bytes**;
- emoji and some rare characters: **4 bytes**.

```python
print("A".encode("utf-8"))        # b'A'           → 1 byte
print("я".encode("utf-8"))        # b'\xd1\x8f'    → 2 bytes
print(len("café"))                 # 4 characters
print(len("café".encode("utf-8")))  # 5 bytes (é takes two)
```

### Where does garbled text come from?

If a file saved as UTF-8 is read as if it were another encoding (say an old Windows code page), the bytes are misinterpreted and you get garbage like `Ã©` instead of `é` (this is called *mojibake*). The cure is simple: **use UTF-8 everywhere**.

## 6.7 Summary

- Everything in a computer is 0s and 1s; one bit is a 0 or 1, **8 bits = 1 byte**.
- **Binary** carries at two with place values 1, 2, 4, 8…; **hex** uses 0–9 and A–F, one hex digit = four bits.
- Python: `bin()`, `hex()`, `int("…", 2)`, `int("…", 16)`.
- Integers have a fixed size and range; going past it **overflows** (except in Python).
- Floats carry tiny errors: round when showing, compare with "close enough", use integers or `decimal` for money.
- Characters have numbers: ASCII (English) → **Unicode** (everything); **UTF-8** stores each code point in 1–4 bytes. Use UTF-8 everywhere and text stays intact.

## 6.8 Exercises

<!-- exercise: binary-of-10 -->
### Exercise 1: Binary

What is decimal `10` in binary?

- `1010`
- `1100`
- `10`
- `0110`

<details>
<summary>Answer and explanation</summary>

`10 = 8 + 2`, so the 8 and 2 places are 1 and the rest 0: `1010`.
</details>

<!-- exercise: convert -->
### Exercise 2: Convert with Python

Write a program that prints decimal `200` in binary, then in hex, then the decimal value of binary `11111111`:

```text
0b11001000
0xc8
255
```

<details>
<summary>Hint</summary>

Use `bin(200)`, `hex(200)` and `int("11111111", 2)`.
</details>

<!-- exercise: float-predict -->
### Exercise 3: Decimal errors

What does this program print?

```python
print(round(0.1 + 0.2, 1))
print(0.5 + 0.25 == 0.75)
```

<details>
<summary>Answer and explanation</summary>

`0.3` and `True`. 0.5 and 0.25 are exact in binary (1/2 and 1/4), so their sum is exactly 0.75 with no error.
</details>

<!-- exercise: utf8-bytes -->
### Exercise 4: UTF-8 bytes

How many bytes does the string `"Hi你好"` take in UTF-8?

- 4
- 6
- 8
- 12

<details>
<summary>Answer and explanation</summary>

`H` and `i` take 1 byte each, `你` and `好` 3 bytes each: 1 + 1 + 3 + 3 = 8. Check with `len("Hi你好".encode("utf-8"))`.
</details>

<!-- exercise: char-codes -->
### Exercise 5: Character codes

Write a program that prints the code of the character `a` and the character with code `90`:

```text
97
Z
```

<details>
<summary>Hint</summary>

`ord("a")` and `chr(90)`.
</details>

## 6.9 Further reading

- *Code: The Hidden Language of Computer Hardware and Software* (Charles Petzold): from Morse code and flashlights to computers — perfect for beginners.
- [The Floating-Point Guide](https://floating-point-gui.de/): floating-point errors explained simply.
- [Unicode](https://home.unicode.org/)
- Next lesson: [Git and GitHub](../07-git-github/en.md)
