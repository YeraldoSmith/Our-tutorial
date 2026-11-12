# 5. 讀懂錯誤資訊與除錯

> 🎯 **本課目標**：不再害怕紅色的報錯；學會從錯誤資訊裡找到「哪個檔案、第幾行、什麼問題」；掌握 40 個常見英文報錯詞彙；學會 print 除錯、二分排查等方法；知道怎樣搜尋、提問和正確使用 AI 助手。
>
> ⏱ **預計用時**：40 分鐘　｜　📋 **前置知識**：第 1 課

## 5.1 心態：報錯是朋友

新手看到滿屏紅字，第一反應往往是「我是不是不適合程式設計」。其實：

- **所有程式設計師每天都在報錯。** 寫程式碼就是「寫 → 報錯 → 改」的迴圈。
- 錯誤資訊是編譯器/直譯器**在幫你**：它告訴你哪裡有問題，有時還會告訴你怎麼改。
- 真正可怕的不是報錯，而是程式**不報錯但結果是錯的**——所以能報錯其實是好事。

> [!TIP]
> 把錯誤資訊當成一條**線索**，而不是一句批評。你的任務是當偵探：從線索出發，找到問題所在。

## 5.2 錯誤資訊的結構

不同語言的格式略有不同，但都包含這幾樣資訊：**在哪裡**（檔案、行號、列號）、**什麼型別**、**具體描述**。

### Python：從下往上讀

```python norun
def average(numbers):
    return sum(numbers) / len(numbers)

print(average([]))
```

```text
Traceback (most recent call last):
  File "main.py", line 4, in <module>
    print(average([]))
          ~~~~~~~^^^^
  File "main.py", line 2, in average
    return sum(numbers) / len(numbers)
           ~~~~~~~~~~~~~^~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

- **最後一行最重要**：`ZeroDivisionError: division by zero`——錯誤型別是「除以零」。
- 往上是 **Traceback（呼叫回溯）**：程式是怎麼一步步走到出錯位置的。最下面的 `File "main.py", line 2` 是真正出錯的那一行，上面的 `line 4` 是呼叫它的地方。
- `~~~^^^` 這些符號標出了出錯的具體位置。

所以讀 Python 報錯的順序是：**先看最後一行（是什麼錯），再看最後一個 File（在哪裡錯）。**

### C / C++：檔案:行:列: 級別: 描述

```text
main.c:5:20: error: expected ';' before 'return'
    5 |     printf("hi\n")
      |                   ^
      |                   ;
    6 |     return 0;
```

讀法：`main.c` 檔案第 `5` 行第 `20` 列，`error`（錯誤），描述是「在 return 前面應該有一個 `;`」。編譯器甚至在下面畫出了應該加分號的位置。

> [!IMPORTANT]
> 編譯型語言常常**一次報出很多錯誤**，但後面的錯誤往往是第一個錯誤引起的「連鎖反應」。**永遠先修第一個錯誤**，然後重新編譯。

### error 和 warning 的區別

- **error（錯誤）**：必須改，否則程式無法編譯/執行。
- **warning（警告）**：程式能執行，但編譯器覺得這裡**很可能有問題**（例如定義了變數卻沒用到）。不要忽略警告——很多 bug 就藏在警告裡。

## 5.3 常見報錯詞彙表

報錯資訊的英語很「模板化」，掌握下面這些詞，大部分報錯你都能猜出意思：

| 英文 | 中文 | 例子 |
|------|------|------|
| error | 錯誤 | `SyntaxError` 語法錯誤 |
| warning | 警告 | `warning: unused variable` |
| syntax | 語法 | `invalid syntax` 無效的語法 |
| invalid | 無效的 | `invalid character` 無效字元 |
| expected | 期望（這裡應該有） | `expected ':'` 這裡應該有冒號 |
| unexpected | 意外的（這裡不該有） | `unexpected indent` 意外的縮排 |
| missing | 缺少 | `missing return statement` 缺少 return |
| undefined / not defined | 未定義 | `name 'x' is not defined` |
| undeclared | 未宣告 | `'x' undeclared` |
| declared | 已宣告 | `declared and not used` 宣告瞭但沒用 |
| unused | 未使用的 | `unused import` 未使用的匯入 |
| type | 型別 | `TypeError` 型別錯誤 |
| mismatch / mismatched | 不匹配 | `mismatched types` 型別不匹配 |
| incompatible | 不相容 | `incompatible types` |
| cannot / can't | 不能 | `cannot find symbol` 找不到符號 |
| convert | 轉換 | `cannot convert string to int` |
| index | 下標 | `IndexError` |
| out of range / out of bounds | 越界 | `index out of range` |
| null / nil / None | 空值 | `NullPointerException` 空指標異常 |
| reference | 引用 | `undefined reference` 未定義的引用 |
| symbol | 符號（名字） | `cannot find symbol` |
| scope | 作用域 | `not declared in this scope` |
| argument | 實參（呼叫時傳的值） | `takes 1 positional argument but 2 were given` |
| parameter | 形參（定義時寫的名字） | |
| return | 返回 | `return type` 返回型別 |
| statement | 語句 | `unreachable statement` 永遠執行不到的語句 |
| expression | 表示式 | `expected expression` |
| token | 記號（一個「詞」或符號） | `unexpected token` |
| literal | 字面量（直接寫出的值） | `unterminated string literal` 沒結束的字串 |
| unterminated / never closed | 沒有結束 / 沒有關閉 | `'(' was never closed` |
| indent / indentation | 縮排 | `IndentationError` |
| attribute | 屬性 | `has no attribute 'x'` |
| module | 模組 | `No module named 'numpy'` |
| permission denied | 沒有許可權 | |
| not found | 找不到 | `file not found` |
| overflow | 溢位 | `integer overflow` |
| exception | 異常 | `Exception in thread "main"` |
| panic | （Rust/Go）程式崩潰 | `thread 'main' panicked` |
| segmentation fault | 段錯誤（非法訪問記憶體） | |
| deprecated | 已過時（不推薦使用） | `deprecated function` |
| traceback / stack trace | 呼叫回溯 / 呼叫堆疊 | |

> [!TIP]
> 學習程式遇到常見錯誤時，會在錯誤下面自動給出中文解釋。看完解釋，再回頭對照英文原文，你會越來越熟悉這些詞。

## 5.4 除錯方法

**除錯**（debug）就是找出並修正程式中的錯誤（bug，「蟲子」）。下面是最實用的幾種方法。

### 方法 1：先讀，再想，再改

不要一看到報錯就亂改程式碼。按順序做：

1. 讀錯誤資訊：**什麼錯誤？哪一行？**
2. 看那一行（以及**上一行**——很多語法錯誤其實是上一行少了符號）。
3. 想一想：我**期望**發生什麼？**實際**發生了什麼？
4. 做**一個**小修改，重新執行。

### 方法 2：print 除錯——讓程式「說出」它在幹什麼

程式結果不對但又不報錯時，在關鍵位置把變數的值列印出來：

```python
price = 100
discount = 20
final = price - price * discount / 100
print("除錯：final =", final)     # 看看算出來的到底是多少
print("應付金額：", final)
```

看到中間結果，你就能知道是哪一步算錯了。找到問題後，記得刪掉這些除錯用的 `print`。

### 方法 3：二分排查——註釋掉一半

程式很長、不知道錯在哪裡時：把後一半程式碼**註釋掉**，執行看看問題還在不在。還在，說明問題在前一半；不在了，說明在後一半。再對有問題的那一半重複這個過程。每次排除一半，很快就能定位。

### 方法 4：小黃鴨除錯法

把程式碼**一行一行地講給別人聽**——沒有人的話就講給一隻橡皮小黃鴨聽。解釋「這一行是做什麼的」的時候，你常常會突然發現：「咦，這裡不對！」這個方法真的很有效，以至於程式設計師們給它起了專門的名字（rubber duck debugging）。

### 方法 5：做一個「最小例子」

把出問題的程式碼複製出來，**刪掉一切與問題無關的部分**，直到剩下最少的幾行仍然能重現這個錯誤。這個過程本身經常就能讓你找到原因；如果找不到，這個最小例子也正好用來向別人提問。

### 方法 6：使用偵錯程式

更專業的工具是**偵錯程式**（debugger）：可以讓程式在某一行**暫停**（斷點），一步一步執行，隨時檢視每個變數的值。VS Code 等編輯器都內建了偵錯程式（點選行號左邊設定斷點，按 <kbd>F5</kbd> 開始除錯）。以後寫大程式時一定要學會它。

## 5.5 怎樣搜尋和提問

### 搜尋

1. 複製錯誤資訊中**最關鍵、最通用**的那部分，去掉你自己的檔名、變數名和路徑。例如搜尋 `ZeroDivisionError: division by zero`，而不是 `main.py line 2 average`。
2. 加上語言名：`python ZeroDivisionError division by zero`。
3. 英文搜尋結果通常更多、更新。常去的網站：**Stack Overflow**、官方文件、GitHub Issues。

### 提問

一個好問題能讓別人在一分鐘內幫到你。請包含：

1. **你想做什麼**（目標）；
2. **你寫的程式碼**（最小例子，用程式碼區塊，不要截圖）；
3. **完整的錯誤資訊**（複製文字）；
4. **你已經嘗試過什麼**；
5. 執行環境：作業系統、語言版本。

### 正確使用 AI 助手

ChatGPT、Claude、通義千問、DeepSeek 等 AI 助手對學習程式設計很有幫助，但要用對方法：

- ✅ 讓它**解釋**錯誤資訊、概念和程式碼：「這段程式碼為什麼報這個錯？請一步一步解釋。」
- ✅ 讓它**出題**、給提示，而不是直接給答案。
- ✅ 寫完程式碼後讓它**評審**：「有什麼可以改進的？」
- ❌ **不要直接複製答案**交差——你會失去練習的機會，下一次遇到同樣的問題還是不會。
- ⚠️ AI 也會出錯，說得很自信但完全錯誤的情況並不少見。**一定要執行驗證**，並對照官方文件。

## 5.6 動手練習：修復程式

下面這段程式有 3 個錯誤。先執行它，根據報錯一個一個修好（練習 3 是同樣的程式，修好後可以檢查答案）：

```python norun
print("購物小票")
apple = 3
banana = 5
print("蘋果：" apple, "個")
print("香蕉：", Banana, "個")
print("合計：", apple + banana, "個"
```

## 5.7 小結

- 報錯是在幫你；最可怕的是不報錯的錯誤結果。
- Python 報錯**從下往上讀**：最後一行看錯誤型別，最後一個 `File` 看位置。
- 編譯器報錯格式是「檔案:行:列: 級別: 描述」；**先修第一個錯誤**。
- error 必須改，warning 也要重視。
- 除錯方法：讀-想-改、print 除錯、二分排查、小黃鴨、最小例子、偵錯程式。
- 搜尋時去掉個人資訊；提問時給出目標、程式碼、完整報錯和已嘗試的方法。
- AI 助手用來解釋和啟發，不要盲目複製，一定要驗證。

## 5.8 練習

<!-- exercise: read-traceback -->
### 練習 1：讀 Python 報錯

看到下面的報錯，問題出在第幾行？

```text
Traceback (most recent call last):
  File "main.py", line 7, in <module>
    total = add(1, "2")
  File "main.py", line 3, in add
    return a + b
           ~~^~~
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

- 第 1 行
- 第 3 行：數字和文字不能直接相加
- 第 7 行：函式名寫錯了
- 報錯資訊看不出來

<details>
<summary>答案與解析</summary>

最後一個 `File` 是 `line 3`，錯誤型別 `TypeError` 說明 `int`（整數）和 `str`（字串）不能用 `+` 相加。根本原因是第 7 行傳入了文字 `"2"`，但報錯的位置在第 3 行。
</details>

<!-- exercise: first-error -->
### 練習 2：先修哪個？

C 編譯器一次報出了 12 個錯誤。最好的做法是？

- 從最後一個錯誤開始修
- 先修第一個錯誤，然後重新編譯
- 一次把 12 個都改完再編譯
- 換一個編譯器

<details>
<summary>答案與解析</summary>

後面的錯誤往往是第一個錯誤引起的連鎖反應。修好第一個、重新編譯，常常其他錯誤就一起消失了。
</details>

<!-- exercise: fix-receipt -->
### 練習 3：修復購物小票

把 5.6 節的程式修好，讓它輸出：

```text
購物小票
蘋果： 3 個
香蕉： 5 個
合計： 8 個
```

<details>
<summary>提示 1</summary>

第 4 行：兩個要列印的東西之間少了逗號。
</details>

<details>
<summary>提示 2</summary>

第 5 行：變數名大小寫不一致；第 6 行：少了右括號。
</details>

<!-- exercise: word-meaning -->
### 練習 4：報錯詞彙

報錯資訊 `expected ')'` 是什麼意思？

- 多了一個右括號
- 這裡應該有一個右括號（缺少右括號）
- 右括號是中文的
- 期望的輸出是右括號

<details>
<summary>答案與解析</summary>

expected 的意思是「期望在這裡看到」，也就是這裡**缺少**一個 `)`。相反，`unexpected` 表示這裡出現了**不該出現**的東西。
</details>

## 5.9 延伸閱讀

- [Python 官方教程 · 錯誤和異常](https://docs.python.org/zh-cn/3/tutorial/errors.html)
- [如何提問的智慧（中文版）](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way)：經典文章，教你如何提出好問題。
- [小黃鴨除錯法（維基百科）](https://zh.wikipedia.org/wiki/小黃鴨除錯法)
- 下一課：[電腦如何儲存數字和文字](../06-numbers-and-text/zh-CN.md)
