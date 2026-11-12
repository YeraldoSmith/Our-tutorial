# 4. Installing developer tools

> 🎯 **Goals**: pick a code editor; install Python, C/C++, Java, Rust, Go and Node.js on Windows, macOS or Linux; check that each installation works; fix the most common installation problems.
>
> ⏱ **Time**: 10–20 minutes per tool (**install a language only when you start its course**)　|　📋 **You need**: lesson 3 (terminal)

> [!TIP]
> This course's app **includes Python**, so you don't need to install anything for the Python course yet. For other languages you can try "Run online" in the app first and install later. "Tools → Check developer tools" shows what's installed at any time.

## 4.1 A few terms

- **Editor**: the program you write code in.
- **Compiler / interpreter**: turns code into a running program (lesson 1). Every language has its own.
- **SDK / toolchain**: the compiler plus common tools plus the standard library, bundled. E.g. Java's JDK, Rust's rustup.
- **Package manager**: installs software with one command. Windows has **winget**, macOS has **Homebrew**, Linux has **apt** (Ubuntu/Debian), **dnf** (Fedora), **pacman** (Arch). It's easier than downloading installers and makes upgrades simple.

## 4.2 Code editor: VS Code recommended

While learning, this course's app is enough. For bigger projects later, we recommend **Visual Studio Code** (VS Code): free, cross-platform, supports every mainstream language.

- Download: <https://code.visualstudio.com/> (pick your system and install with the default options)
- Windows terminal: `winget install -e --id Microsoft.VisualStudioCode`
- macOS: `brew install --cask visual-studio-code`
- Install the extension for each language you use: Python, C/C++, Extension Pack for Java, rust-analyzer, Go.

Alternatives: JetBrains IDEs (PyCharm, IntelliJ IDEA, CLion, RustRover, GoLand — most have free or education editions), Zed, Vim/Neovim and more.

## 4.3 Windows: check that winget works

Recent Windows 10 and Windows 11 include **winget**. In a terminal type:

```console
winget --version
```

If you see a version number, you're set. If not, install "App Installer" from the Microsoft Store.

Each tool below has a **winget command** (easiest) and an **official download**; use either.

> [!NOTE]
> winget may show a "Do you want to allow this app to make changes?" prompt — click "Yes". After installing, **close the terminal and open a new one** so the updated PATH takes effect.

## 4.4 macOS: install the command-line tools and Homebrew

1. Install Apple's command-line tools (includes the clang C/C++ compiler, git and more):

   ```console
   xcode-select --install
   ```

   A window appears; click "Install" and wait.

2. Install Homebrew, the most popular macOS package manager. Open <https://brew.sh/>, copy the install command from the home page into the terminal (only use the command from the official site). At the end, follow the "Next steps" it prints to add brew to your PATH.

3. Check:

   ```console
   brew --version
   ```

## 4.5 Linux: use your distribution's package manager

On Ubuntu / Debian, refresh the package list first:

```console
sudo apt update
```

`sudo` asks for your login password (nothing appears while you type — that's normal; press Enter at the end). Fedora uses `sudo dnf install ...`, Arch `sudo pacman -S ...`.

## 4.6 Python

| System | Command |
|--------|---------|
| Windows | `winget install -e --id Python.Python.3.13` |
| macOS | `brew install python` |
| Ubuntu/Debian | `sudo apt install python3 python3-pip python3-venv` |
| Fedora | `sudo dnf install python3` |

Or download an installer from <https://www.python.org/downloads/>.

> [!WARNING]
> With the Windows installer, **tick "Add python.exe to PATH"** on the very first screen, otherwise the terminal won't find python.

Check (`python` on Windows, `python3` on macOS/Linux):

```console
python3 --version
```

> [!TIP]
> Also consider installing **uv**, a very fast modern Python package and project manager (Python lesson 15 covers it): Windows `winget install -e --id astral-sh.uv`, macOS `brew install uv`, Linux `curl -LsSf https://astral.sh/uv/install.sh | sh`.

## 4.7 C / C++ compilers

| System | How |
|--------|-----|
| Windows | `winget install -e --id BrechtSanders.WinLibs.POSIX.UCRT` (WinLibs: GCC + MinGW-w64) |
| macOS | `xcode-select --install` (already done in 4.4) |
| Ubuntu/Debian | `sudo apt install build-essential gdb` |
| Fedora | `sudo dnf install gcc gcc-c++ gdb` |

Other Windows options:

- **MSYS2** (<https://www.msys2.org/>): after installing, run `pacman -S mingw-w64-ucrt-x86_64-gcc` in the "MSYS2 UCRT64" terminal and add `C:\msys64\ucrt64\bin` to PATH.
- **Visual Studio 2022 Community**: Microsoft's IDE with the MSVC compiler, good for Windows apps later. This course's app uses GCC/Clang-style commands, so WinLibs or MSYS2 is recommended for learning.

Check:

```console
gcc --version
g++ --version
```

On macOS, `gcc` is really Apple Clang — that's perfectly fine.

## 4.8 Java (JDK 25)

The Java course is based on **JDK 25**, the long-term-support release from September 2025. We recommend the Eclipse Temurin distribution:

| System | Command |
|--------|---------|
| Windows | `winget install -e --id EclipseAdoptium.Temurin.25.JDK` |
| macOS | `brew install --cask temurin` |
| Ubuntu 24.04+ / Debian 13+ | `sudo apt install openjdk-25-jdk` |
| Fedora | `sudo dnf install java-25-openjdk-devel` |

Or download from <https://adoptium.net/>. Check:

```console
java -version
```

You should see a version starting with `25`. If it shows an old one (1.8, 17…), an older Java is earlier in your PATH — uninstall it or reorder PATH.

## 4.9 Rust

Rust's official toolchain manager is **rustup**:

- **Windows**: `winget install -e --id Rustlang.Rustup`, or download `rustup-init.exe` from <https://rustup.rs/>. If it says the **Visual Studio C++ Build Tools** are needed, choose option 1 to install them automatically (Rust uses them to link programs on Windows).
- **macOS / Linux**:

  ```console
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

  Press Enter for the default installation, then open a new terminal.

Check:

```console
rustc --version
cargo --version
```

To upgrade later: `rustup update`.

## 4.10 Go

| System | Command |
|--------|---------|
| Windows | `winget install -e --id GoLang.Go` |
| macOS | `brew install go` |
| Linux | download the `.tar.gz` from <https://go.dev/dl/> and unpack it into `/usr/local` as described there; or `sudo snap install go --classic` |

> [!NOTE]
> Ubuntu's `apt install golang-go` is often an old version; prefer the official download.

Check:

```console
go version
```

## 4.11 Node.js (JavaScript / TypeScript)

Install an **LTS (long-term support) version**, 22 or newer:

| System | Command |
|--------|---------|
| Windows | `winget install -e --id OpenJS.NodeJS.LTS` |
| macOS | `brew install node` |
| Linux | use the version manager fnm: `curl -fsSL https://fnm.vercel.app/install \| bash`, open a new terminal, then `fnm install --lts` |

Or download from <https://nodejs.org/>. Check:

```console
node --version
npm --version
```

Node.js 22.18 and later can **run TypeScript files directly** (`node main.ts`) — no separate compiler needed.

## 4.12 What does the assembly course need?

The assembly course uses **Linux on x86-64** (the assembler that comes with GCC):

- **Linux on an x86-64 PC**: `build-essential` from 4.7 is all you need.
- **Windows**: install WSL (Linux inside Windows). Run `wsl --install` in an administrator terminal and restart; Ubuntu gets installed. Inside Ubuntu run `sudo apt install build-essential gdb`.
- **macOS**: Macs now use ARM chips with a different instruction set. Use "Run online" in the app, or a virtual machine (UTM, OrbStack) running Linux.

## 4.13 After-install checklist

In a **freshly opened** terminal, each of these should print a version:

```console
python3 --version
gcc --version
java -version
rustc --version
go version
node --version
```

Or simply use "Tools → Check developer tools" in the app to see ✅ / ❌ for each language.

## 4.14 Common problems

| Symptom | Cause and fix |
|---------|---------------|
| `command not found` / "is not recognized" | ① terminal not reopened; ② not on PATH; ③ on Windows use `python` or `py` instead of `python3` |
| On Windows, `python` opens the Microsoft Store | That's an "app execution alias". After installing real Python, go to Settings → Apps → Advanced app settings → App execution aliases and turn off python.exe and python3.exe |
| `permission denied` / access denied | Installing system software needs `sudo` on Linux/macOS; on Windows run the terminal as administrator |
| download slow or failing | try again later, or use a mirror close to you |
| antivirus blocks a program you compiled | freshly built `.exe` files are sometimes flagged by mistake; add your code folder to the antivirus exclusions |
| wrong version (e.g. still Java 8) | several versions installed and the old one comes first in PATH; uninstall it or fix PATH |

## 4.15 Summary

- Editors write code, compilers/interpreters run it; package managers (winget / Homebrew / apt) make installing easy.
- **Install a language when you need it**; the app includes Python, and other languages can run online at first.
- **Reopen the terminal** after every installation and check with `--version`.
- The assembly course needs Linux x86-64: WSL on Windows, online running or a VM on a Mac.

## 4.16 Exercises

<!-- exercise: verify -->
### Exercise 1: How to check?

After installing Rust, which command checks that it worked?

- `rust install`
- `rustc --version`
- `cargo delete`
- `rm rust`

<details>
<summary>Answer and explanation</summary>

`rustc --version` prints the Rust compiler's version; if it does, Rust is installed and on PATH.
</details>

<!-- exercise: path-checkbox -->
### Exercise 2: Python on Windows

When installing Python on Windows with the installer, which option on the first screen matters most?

- "Install for all users"
- "Add python.exe to PATH"
- "Disable path length limit"
- "Customize installation"

<details>
<summary>Answer and explanation</summary>

"Add python.exe to PATH" lets the terminal find the python command. If you forgot it, run the installer again and choose "Modify", or add the folder to PATH by hand.
</details>

<!-- exercise: package-manager -->
### Exercise 3: Package managers

Which is the most widely used package manager on **macOS**?

- winget
- apt
- Homebrew
- pacman

<details>
<summary>Answer and explanation</summary>

Homebrew (the command is `brew`). winget is for Windows, apt for Ubuntu/Debian, pacman for Arch Linux.
</details>

<!-- exercise: checklist -->
### Exercise 4: Print a checklist

Write a program that prints these three check commands (so you have a list to type from):

```text
python3 --version
gcc --version
java -version
```

<details>
<summary>Hint</summary>

Three `print` calls. Note that `java -version` has a single dash — an old Java habit.
</details>

## 4.17 Further reading

- [VS Code documentation](https://code.visualstudio.com/docs): a UI tour and setup guides for each language.
- [Install Rust](https://www.rust-lang.org/tools/install), [Install Go](https://go.dev/doc/install), [Adoptium (Temurin JDK)](https://adoptium.net/)
- Next lesson: [Reading error messages and debugging](../05-errors-debugging/en.md)
