# 7. Git 与 GitHub 入门

> 🎯 **本课目标**：理解「版本控制」解决什么问题；安装并配置 Git；学会 `init`、`status`、`add`、`commit`、`log`、`diff`；了解分支；用 GitHub 下载（clone）和更新（pull）本教程，并把自己的代码推送（push）上去。
>
> ⏱ **预计用时**：50 分钟　｜　📋 **前置知识**：第 3 课（终端）

## 7.1 为什么需要版本控制？

你一定见过这样的文件夹：

```text
毕业论文.docx
毕业论文_修改版.docx
毕业论文_修改版2.docx
毕业论文_最终版.docx
毕业论文_最终版_真的最终版.docx
```

写代码时这个问题更严重：改了一处，结果别的地方坏了，想退回去却记不清改了什么；多人一起写同一个项目，互相覆盖对方的修改……

**版本控制系统**就是用来解决这些问题的：它像一台「时光机」，记录下每一次修改——谁、在什么时候、改了什么、为什么改。随时可以查看历史、对比差异、退回任何一个版本，多人协作也不会互相覆盖。

**Git** 是目前全世界使用最广泛的版本控制系统，由 Linux 的作者 Linus Torvalds 在 2005 年创建。**GitHub** 是一个托管 Git 仓库的网站（还有 GitLab、Gitee 等类似网站），本教程就放在 GitHub 上。

> [!NOTE]
> 简单区分：**Git 是工具**（装在你电脑上的软件），**GitHub 是网站**（存放和分享仓库的地方）。

## 7.2 安装和配置 Git

| 系统 | 命令 |
|------|------|
| Windows | `winget install -e --id Git.Git`（或到 <https://git-scm.com/> 下载，安装选项保持默认即可） |
| macOS | `xcode-select --install` 已经包含 git；也可以 `brew install git` |
| Ubuntu/Debian | `sudo apt install git` |

验证：

```console
git --version
```

第一次使用前，告诉 Git 你是谁（这些信息会记录在每次提交里）：

```console
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
git config --global init.defaultBranch main
```

## 7.3 核心概念：仓库、暂存、提交

```text
  工作区                暂存区               仓库（历史记录）
 (你正在改的文件)  ──git add──►  (准备提交的内容) ──git commit──►  (一张张「快照」)
```

- **仓库**（repository，简称 repo）：一个被 Git 管理的项目文件夹。里面有一个隐藏的 `.git` 文件夹，保存全部历史。
- **提交**（commit）：项目在某一时刻的一张**快照**，附带一句说明（提交信息）。
- **暂存区**（staging area）：选择「这次要提交哪些修改」的地方。可以只提交一部分修改。

## 7.4 动手：第一个仓库

在终端里跟着做（每行一条命令）：

```console
mkdir my-first-repo
cd my-first-repo
git init
```

`git init` 把当前文件夹变成一个 Git 仓库。现在用编辑器在里面新建一个文件 `hello.py`，内容是：

```python
print("第一个版本")
```

然后查看状态：

```console
git status
```

Git 会告诉你：有一个**未跟踪**（untracked）的文件 `hello.py`。把它加入暂存区并提交：

```console
git add hello.py
git commit -m "添加 hello.py"
```

`-m` 后面是提交信息，说明这次做了什么。好的提交信息要**简短、具体**，例如「修复除以零的错误」「添加登录页面」，而不是「改了点东西」。

### 再改一次

把 `hello.py` 改成：

```python
print("第二个版本")
print("学会了 Git！")
```

查看具体改了什么：

```console
git diff
```

输出中以 `-` 开头的红色行是删掉的内容，`+` 开头的绿色行是新增的内容。确认无误后提交：

```console
git add hello.py
git commit -m "更新问候语"
```

> [!TIP]
> `git add .` 可以一次把当前文件夹里所有修改加入暂存区（`.` 表示「当前文件夹」）。

### 查看历史

```console
git log --oneline
```

```text
a1b2c3d (HEAD -> main) 更新问候语
9f8e7d6 添加 hello.py
```

每次提交都有一个唯一的编号（哈希值，如 `a1b2c3d`）。想看某个旧版本的文件内容，可以：

```console
git show 9f8e7d6:hello.py
```

想撤销工作区里还没提交的修改，恢复成上次提交的样子：

```console
git restore hello.py
```

## 7.5 .gitignore：不想被记录的文件

有些文件不应该放进仓库：编译生成的程序、临时文件、密码和密钥等。在仓库根目录新建一个名为 `.gitignore` 的文本文件，每行写一个规则：

```text
# 编译产物
*.exe
*.o
build/
# Python 缓存
__pycache__/
# 虚拟环境
.venv/
# 密钥——绝对不要提交！
.env
```

> [!WARNING]
> **永远不要把密码、API 密钥提交到 GitHub。** 即使后来删掉，它也会留在历史记录里，而且公开仓库会被自动扫描。万一提交了，要立刻去对应网站把密钥作废、重新生成。

## 7.6 分支：平行宇宙

**分支**（branch）让你在不影响主线的情况下尝试新功能：

```text
main:      A ── B ── C ─────────── F (合并)
                      \           /
feature:               D ── E ───
```

```console
git switch -c new-feature     # 新建并切换到分支 new-feature
# ……修改、add、commit……
git switch main               # 回到主分支
git merge new-feature         # 把 new-feature 的修改合并进来
```

初学阶段只在 `main` 分支上工作也完全可以，知道有这回事就行。

## 7.7 GitHub：下载本教程

### 方法一：下载压缩包（不需要 Git）

打开 <https://github.com/YeraldoSmith/Our-tutorial>，点击绿色的 **Code** 按钮 → **Download ZIP**，解压即可。缺点：以后更新要重新下载。

### 方法二：git clone（推荐）

```console
git clone https://github.com/YeraldoSmith/Our-tutorial.git
cd Our-tutorial
```

`clone`（克隆）会把整个仓库连同历史下载到本地。以后教程更新了，只需要在这个文件夹里运行：

```console
git pull
```

就能获取最新内容——学习程序从这个文件夹读取课程，所以课程也会同步更新。

## 7.8 GitHub：上传你自己的代码

1. 注册 GitHub 账号：<https://github.com/signup>。
2. 点右上角 **+** → **New repository**，填写仓库名（如 `my-first-repo`），点 **Create repository**。
3. 按页面提示，在本地仓库里运行：

   ```console
   git remote add origin https://github.com/你的用户名/my-first-repo.git
   git push -u origin main
   ```

4. 第一次推送需要登录。GitHub 不再支持用账号密码推送，推荐安装 **GitHub CLI**（`winget install -e --id GitHub.cli` 或 `brew install gh`），运行 `gh auth login` 按提示在浏览器里登录；Windows 上安装 Git 时自带的 Git Credential Manager 也会弹出浏览器登录窗口。

以后每次修改后：`git add .` → `git commit -m "说明"` → `git push`。

### 参与开源：Fork 和 Pull Request

想给别人的项目（比如本教程）修改错别字？

1. 在项目页面点 **Fork**，复制一份到你的账号下；
2. 在你的副本里修改并提交；
3. 点 **Contribute → Open pull request**，请原作者合并你的修改。

这就是开源协作的基本流程。

## 7.9 常用命令速查

| 命令 | 作用 |
|------|------|
| `git init` | 把当前文件夹变成仓库 |
| `git clone 网址` | 下载一个远程仓库 |
| `git status` | 查看当前状态（改了哪些文件） |
| `git add 文件` / `git add .` | 把修改加入暂存区 |
| `git commit -m "说明"` | 提交 |
| `git log --oneline` | 查看历史 |
| `git diff` | 查看尚未暂存的修改 |
| `git restore 文件` | 撤销工作区的修改 |
| `git switch -c 分支名` | 新建并切换分支 |
| `git pull` | 从远程获取并合并最新内容 |
| `git push` | 把本地提交推送到远程 |

## 7.10 小结

- 版本控制像时光机：记录每次修改，可以查看、对比、回退，支持多人协作。
- **Git** 是工具，**GitHub** 是托管仓库的网站。
- 基本流程：修改 → `git add` → `git commit -m "..."`；用 `git status`、`git diff`、`git log` 查看情况。
- `.gitignore` 排除不需要的文件；**绝不提交密码和密钥**。
- `git clone` 下载仓库，`git pull` 更新，`git push` 上传。

## 7.11 练习

<!-- exercise: commit-meaning -->
### 练习 1：提交是什么？

在 Git 中，一次 **commit（提交）** 是什么？

- 把文件上传到 GitHub
- 项目在某一时刻的快照，附带一句说明
- 删除一个文件
- 新建一个文件夹

<details>
<summary>答案与解析</summary>

提交是保存在本地仓库里的一张快照。上传到 GitHub 是 `git push` 的工作。
</details>

<!-- exercise: clone -->
### 练习 2：下载仓库

想把 GitHub 上的一个仓库连同全部历史下载到自己电脑上，用哪个命令？

- `git init`
- `git clone`
- `git commit`
- `git log`

<details>
<summary>答案与解析</summary>

`git clone 网址`。下载完成后，用 `git pull` 获取以后的更新。
</details>

<!-- exercise: secret -->
### 练习 3：哪些文件不该提交？

下面哪个文件**最不应该**提交到公开的 GitHub 仓库？

- `README.md`
- `main.py`
- 存放 API 密钥的 `.env`
- `.gitignore`

<details>
<summary>答案与解析</summary>

`.env` 这类存放密码和密钥的文件应该写进 `.gitignore`。一旦公开，别人就能用你的密钥。
</details>

<!-- exercise: workflow -->
### 练习 4：日常流程

编写程序，按顺序输出修改代码后上传到 GitHub 的三条命令（提交信息写 `update`）：

```text
git add .
git commit -m "update"
git push
```

<details>
<summary>提示</summary>

第二行里有双引号。可以用单引号包住整个字符串：`print('git commit -m "update"')`。
</details>

## 7.12 延伸阅读

- [Pro Git（中文版）](https://git-scm.com/book/zh/v2)：Git 官方免费书，前三章足够日常使用。
- [Learn Git Branching](https://learngitbranching.js.org/?locale=zh_CN)：用动画和小游戏学习分支，强烈推荐。
- [GitHub 文档：Hello World](https://docs.github.com/zh/get-started/start-your-journey/hello-world)
- 下一课：[怎样学好编程](../08-how-to-learn/zh-CN.md)
