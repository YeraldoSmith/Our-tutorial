# Our Tutorial · Learn to Code from Zero

[简体中文](README.md) · [繁體中文](README.zh-TW.md) · **English** · [Русский](README.ru.md)

A programming course that starts from **absolute zero**, in several languages, where you practise as you read — plus a cross-platform learning app.

- 📚 **9 courses**: Start here, Python, C, C++, Java, Rust, Go, JavaScript/TypeScript, Assembly (x86-64)
- 🌏 **Lessons in 4 languages**: Simplified Chinese, Traditional Chinese, English, Russian; the app's interface also speaks Japanese, Korean, Spanish, French and German
- ✍️ **Exercises in every lesson**: multiple choice, "predict the output" and coding tasks that the app **checks automatically**
- 💬 **Error messages explained**: compiler and interpreter errors get a plain-language explanation in your language (e.g. "you typed a full-width bracket")
- 🧪 **C/C++ memory check**: turn it on in "Tools → Settings…", and out-of-bounds array access, use after free, memory leaks and similar bugs are caught when they happen, with the line number (based on AddressSanitizer; not available with MinGW on Windows)
- 🖥️ **A real desktop app** (not a web page) for Windows, macOS and Linux: read, write, run and practise in one window
- 🆕 **Modern content**: Python 3.14 and uv, C23, C++23, Java 25 (LTS), Rust 2024, Go 1.2x, Node.js running TypeScript directly…

## Progress

| Course | Status |
|--------|--------|
| 🚀 Start here | ✅ 8 lessons |
| 🐍 Python | ✅ 16 lessons |
| 🔧 C | ✅ 14 lessons |
| C++, Java, Rust, Go, JavaScript/TypeScript, Assembly | 🚧 being written, arriving lesson by lesson |

## Where do I start?

| I am… | Suggested path |
|-------|----------------|
| new to programming | **[Start here](courses/start/)**, then **[Python](courses/python/)** |
| interested in websites | Start here → **[JavaScript & TypeScript](courses/javascript/)** |
| curious how computers work | Python → **[C](courses/c/)** → **[Assembly](courses/asm/)** |
| into games / high-performance software | C → **[C++](courses/cpp/)** or **[Rust](courses/rust/)** |
| aiming at enterprise back-ends / Android | Python → **[Java](courses/java/)** |
| building cloud services / CLI tools | Python → **[Go](courses/go/)** |
| interested in AI and LLMs | Python → sister repo **[LLM-Tutorial](https://github.com/YeraldoSmith/LLM-Tutorial)** |

## Two ways to learn

### 1. With the learning app (recommended)

1. Open the [Releases](https://github.com/YeraldoSmith/Our-tutorial/releases) page and download the file for your system:
   - Windows: `app-windows-x64.zip` — unzip and double-click `OurTutorial.exe`
   - macOS (Apple silicon): `app-macos-arm64.zip` — unzip and drag `OurTutorial.app` into Applications. If macOS says it can't verify the developer, open System Settings → Privacy & Security and click "Open Anyway"
   - Linux: `app-linux-x64.tar.gz` — unpack and run `OurTutorial/OurTutorial`
2. Choose your interface language and start with "Start here", lesson 1.
3. The app **includes Python**, so the Python course works without installing anything. For other languages open "Tools → Check developer tools" and follow the instructions; until then you can use "Run online".

**Run from source** (Python 3.11+):

```bash
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
pip install -r app/requirements.txt
python app/run.py
```

`git pull` updates the lessons the app shows; the downloaded app has "Tools → Update course content".

### 2. Read directly on GitHub

Each lesson is a folder with one file per language: `zh-CN.md`, `zh-TW.md`, `en.md`, `ru.md`. Reference solutions and tests are in `exercises.toml` in the same folder.

## Repository layout

```text
tutorial.json                 repository info and course order
courses/<course>/course.toml  course title and summary
courses/<course>/<NN-lesson>/ zh-CN.md, zh-TW.md, en.md, ru.md, exercises.toml
app/                          the learning app (Python + PySide6/Qt)
tools/check.py                checks all lessons and runs every reference solution
tools/zh_tw.py                generates Traditional Chinese from Simplified (OpenCC + glossary)
```

## Contributing

Fixes, more examples and new translations are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Before sending changes run:

```bash
python tools/check.py --examples
```

## Licence

- Lessons and exercises: [CC BY-SA 4.0](LICENSE-CONTENT.md)
- App and tools: [MIT](LICENSE)
