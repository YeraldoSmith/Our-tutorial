# 4. Установка инструментов разработки

> 🎯 **Цели урока**: выбрать редактор кода; установить Python, C/C++, Java, Rust, Go и Node.js в Windows, macOS или Linux; проверить, что установка удалась; справиться с самыми частыми проблемами.
>
> ⏱ **Время**: 10–20 минут на инструмент (**ставьте язык, когда начинаете его курс**)　|　📋 **Что нужно знать**: урок 3 (терминал)

> [!TIP]
> В программу курса **встроен Python**, поэтому для курса Python пока ничего ставить не нужно. Другие языки можно сначала попробовать через «запуск онлайн» в программе, а установить потом. «Инструменты → Проверить инструменты разработки» в любой момент покажет, что установлено.

## 4.1 Несколько терминов

- **Редактор** — программа, в которой пишут код.
- **Компилятор / интерпретатор** — превращает код в работающую программу (урок 1). У каждого языка свой.
- **SDK / набор инструментов** (toolchain) — компилятор, полезные утилиты и стандартная библиотека вместе. Например, JDK для Java, rustup для Rust.
- **Менеджер пакетов** — устанавливает программы одной командой. В Windows есть **winget**, в macOS — **Homebrew**, в Linux — **apt** (Ubuntu/Debian), **dnf** (Fedora), **pacman** (Arch). Это проще, чем скачивать установщики, и упрощает обновления.

## 4.2 Редактор кода: рекомендуем VS Code

Для учёбы хватит программы курса. Для больших проектов позже рекомендуем **Visual Studio Code** (VS Code): бесплатный, кроссплатформенный, поддерживает все основные языки.

- Скачать: <https://code.visualstudio.com/> (выберите систему и установите с параметрами по умолчанию)
- В терминале Windows: `winget install -e --id Microsoft.VisualStudioCode`
- macOS: `brew install --cask visual-studio-code`
- **Русский интерфейс**: VS Code → значок «Расширения» слева → найдите `Russian Language Pack` → установите → перезапустите.
- Для каждого языка поставьте расширение: Python, C/C++, Extension Pack for Java, rust-analyzer, Go.

Альтернативы: IDE от JetBrains (PyCharm, IntelliJ IDEA, CLion, RustRover, GoLand — у большинства есть бесплатные или учебные версии), Zed, Vim/Neovim и другие.

## 4.3 Windows: проверьте winget

В свежих Windows 10 и в Windows 11 есть **winget**. Наберите в терминале:

```console
winget --version
```

Если видите номер версии — всё в порядке. Если нет, установите «Установщик приложений» (App Installer) из Microsoft Store.

Для каждого инструмента ниже есть **команда winget** (проще всего) и **официальный сайт** — подойдёт любой вариант.

> [!NOTE]
> winget может спросить «Разрешить этому приложению вносить изменения?» — нажмите «Да». После установки **закройте терминал и откройте новый**, чтобы обновлённый PATH начал действовать.

## 4.4 macOS: инструменты командной строки и Homebrew

1. Установите инструменты командной строки Apple (компилятор C/C++ clang, git и др.):

   ```console
   xcode-select --install
   ```

   Появится окно — нажмите «Установить» и подождите.

2. Установите Homebrew — самый популярный менеджер пакетов для macOS. Откройте <https://brew.sh/>, скопируйте команду установки с главной страницы в терминал (используйте только команду с официального сайта). В конце выполните команды из раздела «Next steps», чтобы добавить brew в PATH.

3. Проверьте:

   ```console
   brew --version
   ```

## 4.5 Linux: менеджер пакетов вашего дистрибутива

В Ubuntu / Debian сначала обновите список пакетов:

```console
sudo apt update
```

`sudo` спросит пароль от вашей учётной записи (при вводе **ничего не отображается** — это нормально, в конце нажмите Enter). В Fedora — `sudo dnf install ...`, в Arch — `sudo pacman -S ...`.

## 4.6 Python

| Система | Команда |
|---------|---------|
| Windows | `winget install -e --id Python.Python.3.13` |
| macOS | `brew install python` |
| Ubuntu/Debian | `sudo apt install python3 python3-pip python3-venv` |
| Fedora | `sudo dnf install python3` |

Или скачайте установщик с <https://www.python.org/downloads/>.

> [!WARNING]
> В установщике для Windows **отметьте «Add python.exe to PATH»** на самом первом экране, иначе терминал не найдёт python.

Проверка (`python` в Windows, `python3` в macOS/Linux):

```console
python3 --version
```

> [!TIP]
> Стоит установить и **uv** — очень быстрый современный менеджер пакетов и проектов Python (о нём урок 15 курса Python): Windows `winget install -e --id astral-sh.uv`, macOS `brew install uv`, Linux `curl -LsSf https://astral.sh/uv/install.sh | sh`.

## 4.7 Компиляторы C / C++

| Система | Как |
|---------|-----|
| Windows | `winget install -e --id BrechtSanders.WinLibs.POSIX.UCRT` (WinLibs: GCC + MinGW-w64) |
| macOS | `xcode-select --install` (уже сделано в 4.4) |
| Ubuntu/Debian | `sudo apt install build-essential gdb` |
| Fedora | `sudo dnf install gcc gcc-c++ gdb` |

Другие варианты для Windows:

- **MSYS2** (<https://www.msys2.org/>): после установки выполните `pacman -S mingw-w64-ucrt-x86_64-gcc` в терминале «MSYS2 UCRT64» и добавьте `C:\msys64\ucrt64\bin` в PATH.
- **Visual Studio 2022 Community** — IDE от Microsoft с компилятором MSVC, пригодится позже для программ под Windows. Программа курса использует команды в стиле GCC/Clang, поэтому для учёбы лучше WinLibs или MSYS2.

Проверка:

```console
gcc --version
g++ --version
```

В macOS `gcc` на самом деле Apple Clang — это нормально.

## 4.8 Java (JDK 25)

Курс Java основан на **JDK 25** — выпуске с долгосрочной поддержкой (сентябрь 2025). Рекомендуем дистрибутив Eclipse Temurin:

| Система | Команда |
|---------|---------|
| Windows | `winget install -e --id EclipseAdoptium.Temurin.25.JDK` |
| macOS | `brew install --cask temurin` |
| Ubuntu 24.04+ / Debian 13+ | `sudo apt install openjdk-25-jdk` |
| Fedora | `sudo dnf install java-25-openjdk-devel` |

Или скачайте с <https://adoptium.net/>. Проверка:

```console
java -version
```

Версия должна начинаться с `25`. Если показана старая (1.8, 17…), значит, в PATH раньше стоит старая Java — удалите её или поменяйте порядок в PATH.

## 4.9 Rust

Официальный менеджер инструментов Rust — **rustup**:

- **Windows**: `winget install -e --id Rustlang.Rustup` или скачайте `rustup-init.exe` с <https://rustup.rs/>. Если программа сообщит, что нужны **Visual Studio C++ Build Tools**, выберите пункт 1 — они установятся автоматически (в Windows Rust использует их для компоновки программ).
- **macOS / Linux**:

  ```console
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  Нажмите Enter для установки по умолчанию, затем откройте новый терминал.

Проверка:

```console
rustc --version
cargo --version
```

Обновление в будущем: `rustup update`.

## 4.10 Go

| Система | Команда |
|---------|---------|
| Windows | `winget install -e --id GoLang.Go` |
| macOS | `brew install go` |
| Linux | скачайте `.tar.gz` с <https://go.dev/dl/> и распакуйте в `/usr/local`, как описано там; или `sudo snap install go --classic` |

> [!NOTE]
> `apt install golang-go` в Ubuntu часто ставит старую версию; лучше официальный архив.

Проверка:

```console
go version
```

## 4.11 Node.js (JavaScript / TypeScript)

Ставьте **LTS-версию** (с долгосрочной поддержкой), 22 или новее:

| Система | Команда |
|---------|---------|
| Windows | `winget install -e --id OpenJS.NodeJS.LTS` |
| macOS | `brew install node` |
| Linux | менеджер версий fnm: `curl -fsSL https://fnm.vercel.app/install \| bash`, новый терминал, затем `fnm install --lts` |

Или скачайте с <https://nodejs.org/>. Проверка:

```console
node --version
npm --version
```

Node.js 22.18 и новее умеет **запускать файлы TypeScript напрямую** (`node main.ts`) — отдельный компилятор не нужен.

## 4.12 Что нужно для курса ассемблера?

Курс ассемблера использует **Linux на x86-64** (ассемблер из состава GCC):

- **Linux на компьютере x86-64**: достаточно `build-essential` из 4.7.
- **Windows**: установите WSL (Linux внутри Windows). Выполните `wsl --install` в терминале администратора и перезагрузитесь — установится Ubuntu. В Ubuntu выполните `sudo apt install build-essential gdb`.
- **macOS**: современные Mac работают на процессорах ARM с другим набором команд. Используйте «запуск онлайн» в программе или виртуальную машину с Linux (UTM, OrbStack).

## 4.13 Проверка после установки

В **новом** терминале каждая команда должна вывести версию:

```console
python3 --version
gcc --version
java -version
rustc --version
go version
node --version
```

Или просто откройте в программе «Инструменты → Проверить инструменты разработки» — там видно ✅ / ❌ для каждого языка.

## 4.14 Частые проблемы

| Симптом | Причина и решение |
|---------|-------------------|
| `command not found` / «не распознано как имя…» | ① терминал не перезапущен; ② нет в PATH; ③ в Windows используйте `python` или `py` вместо `python3` |
| В Windows `python` открывает Microsoft Store | Это «псевдоним выполнения приложения». После установки настоящего Python: Параметры → Приложения → Дополнительные параметры → Псевдонимы выполнения приложений — выключите python.exe и python3.exe |
| `permission denied` / отказано в доступе | Для установки системных программ в Linux/macOS нужен `sudo`; в Windows запустите терминал от имени администратора |
| загрузка медленная или обрывается | повторите позже или используйте зеркало поближе |
| антивирус блокирует скомпилированную программу | свежесобранные `.exe` иногда помечаются по ошибке; добавьте папку с кодом в исключения антивируса |
| не та версия (например, всё ещё Java 8) | установлено несколько версий, старая стоит в PATH раньше; удалите её или исправьте PATH |

## 4.15 Итоги

- В редакторе пишут код, компилятор/интерпретатор его выполняет; менеджеры пакетов (winget / Homebrew / apt) упрощают установку.
- **Ставьте язык, когда он понадобится**; Python встроен в программу, а другие языки сначала можно запускать онлайн.
- После каждой установки **перезапускайте терминал** и проверяйте через `--version`.
- Для курса ассемблера нужен Linux x86-64: WSL в Windows, запуск онлайн или ВМ на Mac.

## 4.16 Задания

<!-- exercise: verify -->
### Задание 1: Как проверить?

Какая команда проверит, что Rust установлен?

- `rust install`
- `rustc --version`
- `cargo delete`
- `rm rust`

<details>
<summary>Ответ и разбор</summary>

`rustc --version` выводит версию компилятора Rust; если вывела — Rust установлен и находится в PATH.
</details>

<!-- exercise: path-checkbox -->
### Задание 2: Python в Windows

Какой пункт на первом экране установщика Python для Windows самый важный?

- «Install for all users»
- «Add python.exe to PATH»
- «Disable path length limit»
- «Customize installation»

<details>
<summary>Ответ и разбор</summary>

«Add python.exe to PATH» — тогда терминал найдёт команду python. Если забыли, запустите установщик снова и выберите «Modify» или добавьте папку в PATH вручную.
</details>

<!-- exercise: package-manager -->
### Задание 3: Менеджеры пакетов

Какой менеджер пакетов самый популярный в **macOS**?

- winget
- apt
- Homebrew
- pacman

<details>
<summary>Ответ и разбор</summary>

Homebrew (команда `brew`). winget — для Windows, apt — для Ubuntu/Debian, pacman — для Arch Linux.
</details>

<!-- exercise: checklist -->
### Задание 4: Напечатайте список проверки

Напишите программу, которая выводит три команды проверки (будет шпаргалка):

```text
python3 --version
gcc --version
java -version
```

<details>
<summary>Подсказка</summary>

Три вызова `print`. Обратите внимание: в `java -version` один дефис — старая привычка Java.
</details>

## 4.17 Что почитать

- [Документация VS Code](https://code.visualstudio.com/docs): обзор интерфейса и настройка для каждого языка.
- [Установка Rust](https://www.rust-lang.org/ru/tools/install), [установка Go](https://go.dev/doc/install), [Adoptium (Temurin JDK)](https://adoptium.net/)
- Следующий урок: [Как читать сообщения об ошибках и отлаживать](../05-errors-debugging/ru.md)
