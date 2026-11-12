# 16. Проект: учёт расходов в командной строке

> 🎯 **Цели урока**: собрать вместе всё, что было в курсе, и с нуля написать **по-настоящему полезную** программу — учёт расходов в командной строке: записывать траты, показывать список, считать суммы по категориям, удалять ошибочные записи и хранить данные в файле, чтобы ничего не терялось после закрытия программы. По ходу потренироваться **разбивать задачу на части**, **проектировать данные** и делить программу на слои «данные / хранение / логика / интерфейс»; в конце превратить её в проект uv с тестами и подумать, как её расширить.
>
> ⏱ **Время**: 120 минут　|　📋 **Что нужно знать**: уроки 1–15 этого курса

Поздравляем — это последний урок курса Python! До сих пор каждый урок давал «детали», а теперь мы соберём целую «машину». Двигайтесь шаг за шагом — каждая часть взята из одного из прошлых уроков.

## 16.1 Сначала решим, что строим

Прежде чем писать код, запишите требования своими словами. Это гораздо важнее, чем кажется:

```text
Мне нужна программа учёта расходов:
1. записать трату: сумма, категория (еда, транспорт…), заметка; дата и номер добавляются автоматически
2. показать все траты
3. суммы по категориям: сколько ушло на каждую и какая это доля, желательно с простой диаграммой
4. удалить ошибочную запись
5. хранить данные в файле, чтобы в следующий раз они остались
6. не падать от неверного ввода, а дружелюбно подсказывать
```

Затем подумайте, как программа **общается с пользователем**. Сделаем «интерфейс командной строки»: программа показывает приглашение `>`, пользователь вводит команду, программа выполняет её и ждёт следующую:

```text
> add 350 еда обед
✓ записано #1: еда 350.00
> report
еда            350.00  100.0%  ████████████████████
итого 350.00
> quit
До свидания!
```

## 16.2 Делим программу на слои

Хорошая привычка — разделить программу на слои по обязанностям, чтобы каждый делал одно дело:

| Слой | Отвечает за | Используемые знания |
|------|-------------|---------------------|
| **данные** | как выглядит одна трата | `@dataclass` (урок 13) |
| **хранение** | чтение и запись файла | `json`, `pathlib`, исключения (уроки 11, 12) |
| **логика** | добавление, удаление, подсчёт | списки, словари, функции, сортировка (уроки 8–10) |
| **интерфейс** | чтение команд, показ результатов | `input`, `match`, f-строки (уроки 5, 14) |

Польза в том, что функции слоя логики **ничего не читают и не печатают** — они принимают аргументы и возвращают результат, — поэтому их легко тестировать; а если потом заменить интерфейс командной строки веб-страницей или графическим окном, меняется только слой интерфейса.

## 16.3 Слой данных: одна трата

```python norun
from dataclasses import dataclass

@dataclass
class Expense:
    id: int
    day: str            # дата, например "2025-03-01"
    category: str       # категория, например "еда"
    amount: float       # сумма
    note: str = ""      # заметка
```

У каждой траты есть **номер** `id` — по нему указывают, какую запись удалить.

## 16.4 Слой хранения: сохраняем в файл JSON

```python norun
import json
from dataclasses import asdict
from pathlib import Path

DATA_FILE = Path("expenses.json")

def load() -> list[Expense]:
    if not DATA_FILE.exists():                 # первый запуск: файла ещё нет
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):  # файл повреждён
        print("⚠ файл данных повреждён, начинаем с пустой книги")
        return []

def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]       # dataclass → словарь
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")
```

- `asdict(e)` превращает объект dataclass в словарь, который можно сохранить как JSON;
- `Expense(**row)` делает обратное: создаёт объект, передавая пары ключ-значение словаря как именованные аргументы (распаковка `**`, урок 14);
- при загрузке учтены две неприятности: файла нет или его содержимое испорчено. **Программа должна справляться с «неидеальными» ситуациями.**

## 16.5 Слой логики: добавить, удалить, подсчитать

```python norun
from datetime import date

def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("сумма должна быть больше 0")
    new_id = max((e.id for e in expenses), default=0) + 1     # наибольший существующий номер + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item

def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False                                              # не нашли

def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount     # приём подсчёта из урока 9
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))  # самые большие траты первыми
```

Несколько деталей:

- `max(..., default=0)`: на пустой последовательности `max` выдаёт ошибку, а `default=0` заставляет его вернуть 0;
- `add` при неверной сумме **вызывает исключение** (урок 12), а как сообщить об этом пользователю, решает слой интерфейса;
- `delete` возвращает `True` / `False`, сообщая, удалось ли удалить.

## 16.6 Слой интерфейса: читаем команды, показываем результат

`while True` читает команды без конца, а `match` различает их (урок 14). Вот полная программа — запустите её, и она по порядку выполнит подготовленные ниже команды (в учебной программе можно вводить свои команды):

```python input="add 350 еда обед\nadd 60 транспорт метро\nadd 200 еда кофе\nadd abc еда\nlist\nreport\ndelete 2\nreport\nquit"
"""Учёт расходов в командной строке: записываем каждую трату и считаем суммы по категориям."""
import json
from dataclasses import dataclass, asdict
from datetime import date
from pathlib import Path

DATA_FILE = Path("expenses.json")


# ---------- 1. Данные: одна трата ----------
@dataclass
class Expense:
    id: int
    day: str            # дата, например "2025-03-01"
    category: str       # категория, например "еда"
    amount: float       # сумма
    note: str = ""      # заметка


# ---------- 2. Хранение: чтение и запись файла JSON ----------
def load() -> list[Expense]:
    if not DATA_FILE.exists():
        return []
    try:
        rows = json.loads(DATA_FILE.read_text(encoding="utf-8"))
        return [Expense(**row) for row in rows]
    except (json.JSONDecodeError, TypeError):
        print("⚠ файл данных повреждён, начинаем с пустой книги")
        return []


def save(expenses: list[Expense]) -> None:
    rows = [asdict(e) for e in expenses]
    DATA_FILE.write_text(json.dumps(rows, ensure_ascii=False, indent=2), encoding="utf-8")


# ---------- 3. Логика: добавить, удалить, подсчитать (без ввода и печати — легко тестировать) ----------
def add(expenses: list[Expense], amount: float, category: str, note: str = "") -> Expense:
    if amount <= 0:
        raise ValueError("сумма должна быть больше 0")
    new_id = max((e.id for e in expenses), default=0) + 1
    item = Expense(new_id, date.today().isoformat(), category, round(amount, 2), note)
    expenses.append(item)
    return item


def delete(expenses: list[Expense], expense_id: int) -> bool:
    for i, e in enumerate(expenses):
        if e.id == expense_id:
            del expenses[i]
            return True
    return False


def by_category(expenses: list[Expense]) -> list[tuple[str, float]]:
    totals: dict[str, float] = {}
    for e in expenses:
        totals[e.category] = totals.get(e.category, 0) + e.amount
    return sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))


# ---------- 4. Интерфейс: разбор команд, показ результатов ----------
HELP = """Команды:
  add СУММА КАТЕГОРИЯ [ЗАМЕТКА]   записать трату, например: add 350 еда обед
  list                            показать все записи
  report                          суммы по категориям
  delete НОМЕР                    удалить трату
  help                            показать справку
  quit                            выйти"""


def show_list(expenses: list[Expense]) -> None:
    if not expenses:
        print("(записей пока нет)")
        return
    for e in expenses:
        print(f"#{e.id:<3} {e.day}  {e.category:<10} {e.amount:>8.2f}  {e.note}")


def show_report(expenses: list[Expense]) -> None:
    rows = by_category(expenses)
    if not rows:
        print("(записей пока нет)")
        return
    total = sum(amount for _, amount in rows)
    for category, amount in rows:
        bar = "█" * round(amount / rows[0][1] * 20)      # самой большой категории — 20 делений
        print(f"{category:<10} {amount:>8.2f}  {amount / total:6.1%}  {bar}")
    print(f"итого {total:.2f}")


def main() -> None:
    expenses = load()
    print(f"Учёт расходов: записей уже {len(expenses)}. Введите help, чтобы увидеть команды.")
    while True:
        try:
            line = input("> ")
        except EOFError:                    # ввод закончился (например, Ctrl+D)
            break
        match line.split():
            case ["add", amount, category, *note]:
                try:
                    value = float(amount)
                except ValueError:
                    print(f"✗ «{amount}» — не число")
                    continue
                try:
                    item = add(expenses, value, category, " ".join(note))
                except ValueError as e:
                    print("✗", e)
                    continue
                save(expenses)
                print(f"✓ записано #{item.id}: {item.category} {item.amount:.2f}")
            case ["list"]:
                show_list(expenses)
            case ["report"]:
                show_report(expenses)
            case ["delete", expense_id] if expense_id.isdigit():
                if delete(expenses, int(expense_id)):
                    save(expenses)
                    print(f"✓ удалено #{expense_id}")
                else:
                    print(f"✗ записи #{expense_id} нет")
            case ["help"]:
                print(HELP)
            case ["quit"]:
                break
            case []:
                continue
            case _:
                print("✗ непонятная команда; введите help, чтобы увидеть справку")
    print("До свидания!")


if __name__ == "__main__":
    main()
```

> [!NOTE]
> После каждого `add` и `delete` сразу вызывается `save`, поэтому данные не пропадут, даже если программу закроют на полпути. На сайте `expenses.json` хранится во временной файловой системе браузера и исчезает при перезагрузке страницы; в учебной программе или на вашем компьютере он сохраняется в папке, где запускается программа, и в следующий раз будет на месте.

## 16.7 Превращаем в «настоящий проект»

Пригодятся инструменты из прошлого урока. В терминале:

```bash
$ uv init ledger
$ cd ledger
$ uv add --dev pytest ruff
```

Сохраните программу как `ledger.py`. Поскольку функции слоя логики ничего не читают и не печатают, тестировать их легко:

```python norun
# файл test_ledger.py
from ledger import add, by_category, delete

def test_add_assigns_increasing_ids():
    items = []
    a = add(items, 10, "еда")
    b = add(items, 5, "транспорт")
    assert (a.id, b.id) == (1, 2)

def test_add_rejects_non_positive_amount():
    import pytest
    with pytest.raises(ValueError):
        add([], 0, "еда")

def test_delete():
    items = []
    add(items, 10, "еда")
    assert delete(items, 1) is True
    assert delete(items, 1) is False      # уже удалено
    assert items == []

def test_by_category_sorted_by_amount():
    items = []
    add(items, 10, "транспорт")
    add(items, 30, "еда")
    add(items, 5, "транспорт")
    assert by_category(items) == [("еда", 30), ("транспорт", 15)]
```

```bash
$ uv run pytest          # запустить тесты
$ uv run ruff check .    # проверить код
$ uv run ruff format .   # единое форматирование
$ uv run ledger.py       # запустить программу
```

### Аргументы командной строки вместо приглашения (по желанию)

Многие консольные инструменты не заставляют «войти в программу и вводить команды», а принимают всё сразу в терминале: `python ledger.py add 350 еда обед`. Модуль **`argparse`** из стандартной библиотеки обрабатывает такие **аргументы командной строки** и сам создаёт справку `--help`:

```python norun
import argparse

parser = argparse.ArgumentParser(description="Учёт расходов в командной строке")
sub = parser.add_subparsers(dest="command", required=True)
p_add = sub.add_parser("add", help="записать трату")
p_add.add_argument("amount", type=float)
p_add.add_argument("category")
p_add.add_argument("note", nargs="*")
sub.add_parser("list", help="показать все записи")
sub.add_parser("report", help="суммы по категориям")

args = parser.parse_args()
if args.command == "add":
    ...   # вызвать add из слоя логики, затем save
```

Обратите внимание: слои логики и хранения **не меняются вообще** — заменён только слой интерфейса. В этом и польза слоёв.

## 16.8 Идём дальше: идеи для самостоятельной работы

Когда основное заработает, попробуйте сами добавить возможности. Если застряли — вернитесь к нужному уроку или в официальную документацию:

1. **Суммы за месяц**: `report 2025-03` считает только март (подсказка: `e.day.startswith(...)`).
2. **Предупреждение о бюджете**: задать месячный бюджет и предупреждать при 80%.
3. **Экспорт в CSV**: команда `export` записывает траты в файл CSV, который открывают табличные программы (урок 11).
4. **Правка записи**: `edit НОМЕР СУММА`.
5. **Красивые таблицы**: цветные таблицы с помощью стороннего пакета `rich` (`uv add rich`).
6. **Диаграммы**: столбчатая диаграмма расходов по месяцам с `matplotlib`.
7. **Несколько книг**: выбор файла данных аргументом командной строки.

## 16.9 Чему вы научились

Оглянитесь назад: вы освоили переменные и типы, строки, ввод и вывод, условия и циклы, списки, кортежи, словари, множества, функции, модули, файлы, исключения, классы, генераторы, декораторы, `match`, виртуальные окружения, uv, ruff, pytest… Этого уже достаточно, чтобы писать множество полезных программ.

Куда двигаться дальше:

| Направление | Что изучать |
|-------------|-------------|
| анализ данных | `pandas`, `matplotlib`, Jupyter |
| сайты и API | FastAPI, Django |
| автоматизация | обработка файлов и таблиц, загрузка страниц через `requests`, задания по расписанию |
| ИИ и большие языковые модели | **учебник по LLM** в этом проекте (LLM-Tutorial): от основ математики до обучения собственного GPT |
| второй язык | другие курсы здесь: **C** (как устроен компьютер внутри), **JavaScript** (веб), **Go** (серверы), **Rust** (безопасно и быстро)… |

И самое главное: **найдите маленький проект, который вам по-настоящему хочется сделать, и сделайте его.** Если чего-то не знаете — ищите и спрашивайте; так каждый день поступает любой программист.

## 16.10 Задания

<!-- exercise: parse-add -->
### Задание 1: Разбор команды add

Прочитайте одну команду. Если она имеет вид `add СУММА КАТЕГОРИЯ [ЗАМЕТКА…]` (не меньше 3 слов, первое — `add`) и сумма — **число больше 0**, выведите `СУММА | КАТЕГОРИЯ | ЗАМЕТКА` (сумма с 2 знаками после точки; заметка — оставшиеся слова через пробел или `-`, если заметки нет). Если сумма не число или не больше 0, выведите `invalid amount`; если это не команда add или слов слишком мало — `invalid command`.

```text
Пример ввода: add 35 еда обед и кофе
Пример вывода: 35.00 | еда | обед и кофе
```

<details>
<summary>Подсказка</summary>

Начните с `parts = line.split()` и проверки `len(parts) < 3 or parts[0] != "add"`; сумму обработайте через `try: amount = float(parts[1])`; заметка — `" ".join(parts[3:]) or "-"`.
</details>

<!-- exercise: category-report -->
### Задание 2: Суммы по категориям

В первой строке — целое `n`, затем `n` строк с категорией и суммой. Выведите каждую категорию с её суммой (2 знака после точки) **от наибольшей суммы к наименьшей** (при равенстве — по названию категории), а последней строкой — `total ОБЩАЯ_СУММА`.

```text
Пример ввода:
3
food 35
bus 12.5
food 20
Пример вывода:
food 55.00
bus 12.50
total 67.50
```

<details>
<summary>Подсказка</summary>

Накапливайте в словаре: `totals[c] = totals.get(c, 0) + amount`; сортируйте через `sorted(totals.items(), key=lambda kv: (-kv[1], kv[0]))`.
</details>

<!-- exercise: text-bars -->
### Задание 3: Текстовая диаграмма

Читайте строки с меткой и положительным целым числом до строки `end`. Для каждой строки выведите: метку (выравнивание влево по ширине самой длинной метки), пробел, полосу из символов `#` (длина = `round(значение / максимум × 10)`), пробел и само значение.

```text
Пример ввода:
food 40
bus 20
fun 8
end
Пример вывода:
food ########## 40
bus  ##### 20
fun  ## 8
```

<details>
<summary>Подсказка</summary>

Сначала прочитайте все строки в список и найдите `width = max(len(label) for label, _ in rows)` и максимальное значение; выводите через `f"{label:<{width}} {'#' * n} {value}"`.
</details>

<!-- exercise: why-layers -->
### Задание 4: Зачем слои?

В чём **главная** польза деления программы на слои данных, хранения, логики и интерфейса?

- программа работает быстрее
- каждый слой делает одно дело: логику можно тестировать отдельно, а смена интерфейса (например, на веб-страницу) не затрагивает логику и хранение
- файлы становятся меньше
- этого требует Python

<details>
<summary>Ответ и разбор</summary>

Слои дают каждой части ясную ответственность: слой логики ничего не читает и не печатает, поэтому его легко тестировать; замена интерфейса командной строки на argparse, веб-страницу или графическое окно меняет только слой интерфейса.
</details>

<!-- exercise: why-file -->
### Задание 5: Зачем сохранять в файл?

Почему программа учёта расходов сохраняет данные в `expenses.json` после каждого изменения?

- потому что данные в переменных исчезают, когда программа завершается, а файл позволяет продолжить в следующий раз
- потому что файлы читаются и пишутся быстрее переменных
- потому что в списке Python не может быть больше 100 элементов
- потому что файлы JSON сами считают итоговую сумму

<details>
<summary>Ответ и разбор</summary>

Переменные живут в памяти и исчезают при завершении программы; файлы хранятся на диске, и `load()` читает их при следующем запуске. Сохранение сразу после каждого изменения означает, что данные не пропадут, даже если программа неожиданно закроется.
</details>

## 16.11 Что почитать

- [Учебник по argparse](https://docs.python.org/3/howto/argparse.html)
- [Rich: красивые таблицы и цвета в терминале](https://rich.readthedocs.io/)
- Эрик Мэтиз, «Изучаем Python. Программирование игр, визуализация данных, веб-приложения»: с полноценными практическими проектами
- Поздравляем с окончанием курса Python! Выберите следующий курс в списке курсов.
