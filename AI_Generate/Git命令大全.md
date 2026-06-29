# Git 命令大全教程
> **本文档版本**：v1.0  
**更新日期**：2026-06-28  
**适用人群**：初学者 / 日常开发者 / 高级用户
>

---

## 前言
### 什么是 Git？
Git 是一个**分布式版本控制系统**，由 Linus Torvalds（Linux 之父）于 2005 年创建。它可以：

+ 记录文件的历史变化（谁、什么时候、改了什么）
+ 支持多人协作开发，合并各自的修改
+ 让你随时回退到任意一个历史版本
+ 创建分支，在不影响主线的情况下开发新功能

### 本文档的使用方式
+ **想系统学习**：从第 1 章开始，按顺序阅读
+ **查命令用法**：直接翻到【第 13 章 速查表】，或 Ctrl+F 搜索命令名
+ **遇到报错**：先看【第 12 章 常见问题 FAQ】

### 阅读约定
| 符号 | 含义 |
| --- | --- |
| `$` | 命令行提示符，实际输入时不需要输入 `$` |
| `<xxx>` | 占位符，使用时替换为实际值 |
| `[xxx]` | 可选参数 |
| ` | ` |


---

## 第 1 章：安装与配置
### 1.1 安装 Git
**Windows**：

1. 访问 [https://git-scm.com/download/win](https://git-scm.com/download/win) 下载安装包
2. 安装时选择默认选项即可（推荐使用 Git Bash）
3. 验证安装：打开终端输入 `git --version`

**macOS**：

```bash
# 使用 Homebrew 安装（推荐）
$ brew install git

# 验证
$ git --version
```

**Linux（Ubuntu/Debian）**：

```bash
$ sudo apt update
$ sudo apt install git
```

### 1.2 初次配置（必须）
安装完成后，**必须**先设置用户名和邮箱，否则无法提交：

```bash
# 设置全局用户名和邮箱（仅需一次）
$ git config --global user.name "Your Name"
$ git config --global user.email "your.email@example.com"

# 设置默认文本编辑器（可选，推荐设置）
$ git config --global core.editor "code --wait"   # VS Code
# 或
$ git config --global core.editor "vim"            # Vim

# 启用颜色输出（让命令输出更易读）
$ git config --global color.ui auto
```

### 1.3 查看配置
```bash
# 查看所有配置
$ git config --list

# 查看某项具体配置
$ git config user.name
$ git config user.email

# 查看系统级/全局/仓库级配置
$ git config --system --list   # 系统级（所有用户）
$ git config --global --list   # 全局（当前用户）
$ git config --local --list    # 仓库级（当前项目）
```

### 1.4 配置 SSH Key（连接 GitHub/GitLab）
```bash
# 1. 生成 SSH Key（一路回车，使用默认路径）
$ ssh-keygen -t ed25519 -C "your.email@example.com"

# 2. 查看公钥内容并复制
$ cat ~/.ssh/id_ed25519.pub

# 3. 将公钥添加到 GitHub/GitLab：
#    GitHub：Settings → SSH and GPG keys → New SSH key
#    GitLab：Preferences → SSH Keys → Add new key

# 4. 验证连接
$ ssh -T git@github.com
```

> 💡 **提示**：如果不想配置 SSH，也可以使用 HTTPS 方式克隆仓库，但需要每次输入用户名和密码（或配置凭证存储）。
>

---

## 第 2 章：Git 核心概念
### 2.1 三个区域（核心中的核心）
Git 有三个重要的工作区域，理解它们是掌握 Git 的关键：

```plain
┌─────────────────────────────────────────────────────┐
│                   工作区 (Working Directory)          │
│              你正在编辑的文件（还未暂存）              │
└──────────────────────┬──────────────────────────────┘
                       │ git add
                       ▼
┌─────────────────────────────────────────────────────┐
│                 暂存区 (Staging Area / Index)         │
│          已暂存、即将提交到仓库的文件                 │
└──────────────────────┬──────────────────────────────┘
                       │ git commit
                       ▼
┌─────────────────────────────────────────────────────┐
│                  版本库 (Repository / HEAD)           │
│             已提交的所有历史版本记录                   │
└─────────────────────────────────────────────────────┘
```

**通俗比喻**：

+ **工作区** = 你的办公桌（正在写的文件）
+ **暂存区** = 待归档篮（挑好要归档的文件放进来）
+ **版本库** = 文件柜（永久保存的历史版本）

### 2.2 文件状态
文件在 Git 中有四种状态：

| 状态 | 含义 | 在哪里 |
| --- | --- | --- |
| 未跟踪（Untracked） | 新文件，Git 还没管理它 | 工作区 |
| 已修改（Modified） | 文件被改了，但没暂存 | 工作区 |
| 已暂存（Staged） | 文件已暂存，等待提交 | 暂存区 |
| 已提交（Committed） | 文件已安全保存在版本库 | 版本库 |


### 2.3 分布式 vs 集中式
| 特性 | Git（分布式） | SVN（集中式） |
| --- | --- | --- |
| 是否有中央服务器 | 不需要（每个克隆都是完整仓库） | 必须有中央服务器 |
| 离线工作 | ✅ 可以（本地有完整历史） | ❌ 不行（必须联网） |
| 提交速度 | 快（本地提交） | 慢（需联网提交到服务器） |
| 分支操作 | 轻量（秒级创建/切换） | 重量（复制整个目录） |


---

## 第 3 章：基础命令（日常 80% 场景）
### git init
**功能**：在当前目录初始化一个新的 Git 仓库

**语法**：

```bash
git init [目录名]
```

**示例**：

```bash
# 在当前目录初始化
$ git init

# 在指定目录初始化
$ git init my-project
```

**注意事项**：

+ 执行后会在目录下创建 `.git` 隐藏目录（仓库数据）
+ 已有 `.git` 的目录再执行 `git init` 不会覆盖，只是重新初始化

---

### git clone
**功能**：从远程仓库复制一份完整的代码到本地

**语法**：

```bash
git clone <远程仓库地址> [本地目录名]
```

**示例**：

```bash
# 克隆到当前目录下的默认文件夹（仓库名）
$ git clone https://github.com/username/repo.git

# 克隆到指定目录
$ git clone https://github.com/username/repo.git my-project

# 使用 SSH 方式克隆（推荐）
$ git clone git@github.com:username/repo.git
```

**相关命令**：`git remote`、`git fetch`

---

### git status
**功能**：查看当前仓库的状态（哪些文件被修改、哪些已暂存等）

**语法**：

```bash
git status [-s/--short]
```

**示例**：

```bash
# 详细状态（默认）
$ git status

# 简短状态（每行一个文件，用字母标记状态）
$ git status -s
# 输出示例：
#  M modified.txt    ← M 在第二列：已修改但未暂存
# M  staged.txt      ← M 在第一列：已暂存
# ?? new.txt         ← ??：未跟踪文件
```

**注意事项**：

+ 这是**最常用**的命令之一，建议每次 `add`/`commit` 前都先 `status` 看看

---

### git add
**功能**：将工作区的修改添加到暂存区（准备提交）

**语法**：

```bash
git add <文件>...
git add <目录>
git add .
```

**示例**：

```bash
# 添加单个文件
$ git add README.md

# 添加多个文件
$ git add file1.txt file2.js

# 添加当前目录下所有修改（包括新文件，不包括被删除的文件）
$ git add .

# 添加所有修改（包括删除的文件）
$ git add -A
# 或
$ git add --all

# 添加所有 .js 文件
$ git add *.js
```

**注意事项**：

+ `git add .` 不会添加被删除的文件；`git add -A` 会
+ 已经被 `add` 的文件，再次修改后需要**重新 **`add` 才能把新的修改加入暂存区

---

### git commit
**功能**：将暂存区的内容提交到版本库，生成一个永久的历史记录

**语法**：

```bash
git commit -m "提交消息"
git commit -am "提交消息"   # 跳过 add，直接提交已跟踪文件的修改
```

**示例**：

```bash
# 标准提交（先 add，再 commit）
$ git add .
$ git commit -m "feat: 添加用户登录功能"

# 跳过 add（仅对已被 Git 跟踪的文件有效）
$ git commit -am "fix: 修复登录页面样式问题"

# 修改最后一次提交的消息（未 push 前可用）
$ git commit --amend -m "新的提交消息"
```

**提交消息规范**（推荐）：

```plain
feat:     新功能
fix:      修复 bug
docs:     文档变更
style:    代码格式（不影响功能）
refactor: 重构（既不是新功能也不是修复）
test:     添加测试
chore:    构建工具、依赖变更
```

**相关命令**：`git log`、`git show`

---

### git log
**功能**：查看提交历史记录

**语法**：

```bash
git log [选项]
```

**示例**：

```bash
# 基本查看（按时间倒序，最新的在最上面）
$ git log

# 简洁查看（每行一个提交）
$ git log --oneline

# 查看最近 5 条
$ git log -5 --oneline

# 查看某作者的提交
$ git log --author="张三"

# 查看某个文件的提交历史
$ git log -- README.md

# 图形化查看分支合并历史
$ git log --oneline --graph --all

# 查看某次提交的详细内容
$ git show <commit-hash>
```

**退出 log 查看**：按 `q` 键

---

### git diff
**功能**：查看文件差异（修改了什么）

**语法**：

```bash
git diff                 # 查看工作区 vs 暂存区的差异
git diff --cached        # 查看暂存区 vs 版本库的差异
git diff HEAD            # 查看工作区 vs 版本库的差异
git diff <分支1> <分支2> # 查看两个分支的差异
```

**示例**：

```bash
# 查看所有未暂存的修改
$ git diff

# 查看某个文件的未暂存修改
$ git diff README.md

# 查看已暂存的内容（即将提交的修改）
$ git diff --cached

# 比较两个提交之间的差异
$ git diff abc123 def456

# 比较两个分支
$ git diff main feature/login
```

**输出说明**：

+ 绿色 `+` 行 = 新增的内容
+ 红色 `-` 行 = 删除的内容

---

### git push
**功能**：将本地提交推送到远程仓库

**语法**：

```bash
git push [远程名] [分支名]
git push -u origin <分支名>   # 首次推送，建立追踪关系
```

**示例**：

```bash
# 推送到 origin 远程的 main 分支
$ git push origin main

# 首次推送，同时设置上游追踪（之后可以直接 git push）
$ git push -u origin main

# 推送所有分支
$ git push --all origin

# ⚠️ 强制推送（危险！会覆盖远程历史，仅限个人分支使用）
$ git push --force origin feature/my-branch
# 更安全的强制推送（如果远程有别人推送的内容，会拒绝推送）
$ git push --force-with-lease origin feature/my-branch
```

> ⚠️ **危险警告**：`git push --force` 会覆盖远程仓库的历史，可能导致他人工作丢失。**禁止在 main/master 分支使用！**
>

**相关命令**：`git pull`、`git fetch`

---

### git pull
**功能**：从远程仓库拉取最新内容并合并到本地分支（= `git fetch` + `git merge`）

**语法**：

```bash
git pull [远程名] [分支名]
```

**示例**：

```bash
# 拉取 origin 的 main 分支并合并
$ git pull origin main

# 如果已设置追踪关系，直接：
$ git pull

# 使用 rebase 方式合并（保持历史线性，推荐）
$ git pull --rebase origin main
```

`git pull`** vs **`git fetch`：

| 命令 | 行为 | 是否自动合并 |
| --- | --- | --- |
| `git fetch` | 仅下载远程最新数据到本地，不合并 | ❌ 不合并 |
| `git pull` | 下载并自动合并到当前分支 | ✅ 自动合并 |


> 💡 **推荐做法**：先 `git fetch` 查看远程更新，确认无误后再手动 `merge` 或 `rebase`，比直接 `git pull` 更安全。
>

---

### git branch
**功能**：列出、创建或删除分支

**语法**：

```bash
git branch                  # 列出所有本地分支
git branch <分支名>          # 创建新分支
git branch -d <分支名>       # 删除已合并的分支
git branch -D <分支名>       # 强制删除分支（未合并也会删除）
git branch -a               # 列出所有分支（本地 + 远程）
```

**示例**：

```bash
# 列出本地分支（当前分支前有 * 号）
$ git branch
# * main
#   feature/login

# 创建新分支
$ git branch feature/signup

# 删除已合并的分支
$ git branch -d feature/login

# 强制删除未合并的分支
$ git branch -D feature/old-feature

# 查看所有分支（包括远程）
$ git branch -a
```

**相关命令**：`git checkout`、`git switch`、`git merge`

---

### git switch（推荐）/ git checkout（旧版）
**功能**：切换分支或创建并切换到新分支

**语法**：

```bash
git switch <分支名>              # 切换到已有分支
git switch -c <新分支名>         # 创建并切换到新分支
git checkout <分支名>            # 旧版切换分支方式
git checkout -b <新分支名>       # 旧版创建并切换
```

**示例**：

```bash
# 切换到 main 分支
$ git switch main

# 创建并切换到新分支（推荐方式）
$ git switch -c feature/login

# 旧版方式（同样有效）
$ git checkout -b feature/login

# 切换到上一个分支
$ git switch -
```

> 💡 **说明**：`git switch` 是 Git 2.23+ 新增的命令，专门用来切换分支，比 `git checkout` 更语义化（`checkout` 功能太多容易混淆）。**推荐优先使用 **`git switch`。
>

---

### git merge
**功能**：将一个分支的修改合并到当前分支

**语法**：

```bash
git merge <分支名>
```

**示例**：

```bash
# 1. 先切换到目标分支（如 main）
$ git switch main

# 2. 将 feature/login 分支合并到 main
$ git merge feature/login

# 合并时禁用 Fast-Forward（强制生成 merge commit，保留分支历史）
$ git merge --no-ff feature/login
```

**合并策略说明**：

| 策略 | 说明 |
| --- | --- |
| Fast-Forward（快进） | 如果目标分支没有新提交，直接移动指针，不产生 merge commit |
| `--no-ff` | 禁用快进，总是生成一个新的 merge commit，保留分支拓扑 |
| 三方合并 | 两个分支都有新提交时，Git 自动创建一个 merge commit |


**相关命令**：`git switch`、`git branch`、`git rebase`

---

## 第 4 章：远程仓库操作
### 4.1 远程仓库概念
+ **origin**：默认的远程仓库名称（克隆时自动设置）
+ **upstream**：上游仓库（Fork 场景下，源仓库称为 upstream）
+ 一个本地仓库可以关联**多个**远程仓库

### git remote
**功能**：管理远程仓库

**语法**：

```bash
git remote -v                                  # 查看所有远程仓库
git remote add <名称> <地址>                    # 添加远程仓库
git remote remove <名称>                        # 删除远程仓库
git remote rename <旧名> <新名>                 # 重命名远程仓库
git remote set-url <名称> <新地址>              # 修改远程仓库地址
```

**示例**：

```bash
# 查看远程仓库（clone 后默认有一个 origin）
$ git remote -v
# origin  https://github.com/username/repo.git (fetch)
# origin  https://github.com/username/repo.git (push)

# 添加第二个远程仓库（例如 upstream）
$ git remote add upstream https://github.com/original/repo.git

# 修改远程仓库地址（从 HTTPS 改为 SSH）
$ git remote set-url origin git@github.com:username/repo.git

# 删除远程仓库
$ git remote remove origin
```

---

### 4.2 多人协作典型流程
```plain
1. 首次使用：克隆仓库
   $ git clone git@github.com:team/project.git
   $ cd project

2. 日常开发：创建功能分支
   $ git switch -c feature/xxx

3. 开发完成后：提交并推送
   $ git add .
   $ git commit -m "feat: 添加 xxx 功能"
   $ git push -u origin feature/xxx

4. 提交 Pull Request（GitHub）/ Merge Request（GitLab）

5. 审核通过后：合并到 main，然后拉取最新 main
   $ git switch main
   $ git pull origin main

6. 删除已合并的本地功能分支
   $ git branch -d feature/xxx
```

---

## 第 5 章：分支管理
### 5.1 分支的本质
Git 的分支本质上是一个**指向提交（commit）的指针**，因此创建、切换分支都非常快（只需移动指针）。

```plain
main:     A ← B ← C ← D (HEAD)
                      ↑
feature:              ↗
```

### 5.2 分支命名规范（推荐）
| 前缀 | 用途 | 示例 |
| --- | --- | --- |
| `feature/` | 新功能开发 | `feature/user-login` |
| `fix/` | Bug 修复 | `fix/header-overlap` |
| `hotfix/` | 线上紧急修复 | `hotfix/crash-on-startup` |
| `docs/` | 文档变更 | `docs/update-readme` |
| `refactor/` | 重构 | `refactor/api-layer` |
| `test/` | 测试相关 | `test/add-unit-tests` |


### 5.3 merge vs rebase 对比
| 特性 | merge | rebase |
| --- | --- | --- |
| 历史记录 | 保留完整合并历史（有 merge commit） | 重写历史，使记录呈线性 |
| 安全性 | ✅ 安全（不修改已有 commit） | ⚠️ 会修改历史（不要对公共分支 rebase） |
| 适用场景 | 合并公共分支、发布分支 | 个人功能分支同步主分支最新代码 |
| 冲突处理 | 一次解决 | 每个 commit 都可能要解决冲突 |


**rebase 示例**：

```bash
# 场景：feature 分支想同步 main 分支的最新代码
$ git switch feature/login
$ git rebase main
# 解决冲突（如果有）→ git add → git rebase --continue
```

> ⚠️ **重要规则**：**永远不要对已经推送到远程的公共分支执行 rebase！** 只对个人未推送或私有分支使用 rebase。
>

---

## 第 6 章：撤销与回退（高频痛点）
> ⚠️ **本章是最容易出错的地方，操作前请务必看清当前状态！**
>

### 6.1 场景对照表
| 你想做什么 | 文件状态 | 使用命令 |
| --- | --- | --- |
| 丢弃工作区的修改（未 add） | 已修改，未暂存 | `git restore <文件>` 或 `git checkout -- <文件>` |
| 取消暂存（已 add，想撤回） | 已暂存，未提交 | `git restore --staged <文件>` |
| 修改最后一次提交消息 | 已提交，未推送 | `git commit --amend -m "新消息"` |
| 撤销最后一次提交（保留修改） | 已提交，未推送 | `git reset --soft HEAD~1` |
| 撤销最后一次提交（不保留修改） | 已提交，未推送 | `git reset --hard HEAD~1` |
| 已有推送，想撤销某次提交 | 已推送 | `git revert <commit-hash>` |


### 6.2 git restore
**功能**：恢复工作区或暂存区的文件（Git 2.23+ 推荐）

**示例**：

```bash
# 丢弃工作区的修改（危险！修改将永久丢失）
$ git restore README.md

# 丢弃所有工作区的修改
$ git restore .

# 取消暂存（把文件从暂存区移回工作区）
$ git restore --staged README.md

# 取消所有文件的暂存
$ git restore --staged .
```

### 6.3 git reset
**功能**：回退到某个提交（移动 HEAD 指针）

> ⚠️ **危险命令！** 会丢失提交历史，请谨慎使用。
>

**语法**：

```bash
git reset [--soft | --mixed | --hard] <目标>
```

**三种模式对比**：

| 模式 | 移动 HEAD | 重置暂存区 | 重置工作区 | 修改丢失？ |
| --- | --- | --- | --- | --- |
| `--soft` | ✅ | ❌ | ❌ | ❌ 不丢失（修改在暂存区） |
| `--mixed`（默认） | ✅ | ✅ | ❌ | ❌ 不丢失（修改在工作区） |
| `--hard` | ✅ | ✅ | ✅ | ✅ **修改全部丢失！** |


**示例**：

```bash
# 回退到上一次提交（保留工作区修改）
$ git reset HEAD~1

# 回退到上一次提交（修改保留在暂存区）
$ git reset --soft HEAD~1

# ⚠️ 危险：回退并丢弃所有修改
$ git reset --hard HEAD~1

# 回退到指定的提交（用 commit hash）
$ git reset --hard abc123def
```

### 6.4 git revert
**功能**：创建一个**新的提交**来"撤销"某次提交的效果（不会修改历史，安全！）

**示例**：

```bash
# 撤销某次提交（会打开编辑器让你写 revert 的提交消息）
$ git revert abc123def

# 自动完成，不打开编辑器
$ git revert --no-edit abc123def
```

> ✅ **推荐**：如果提交已经推送到远程，**优先使用 **`git revert` 而不是 `git reset`，因为 `revert` 不会修改历史，不会影响他人。
>

---

## 第 7 章：进阶命令
### git stash
**功能**：临时保存工作区的修改（不提交），以便切换分支或拉取代码

**语法**：

```bash
git stash                  # 暂存当前修改
git stash list             # 查看所有暂存
git stash pop              # 恢复最近一次暂存并删除暂存记录
git stash apply            # 恢复暂存但不删除记录
git stash drop             # 删除某条暂存记录
```

**示例**：

```bash
# 正在 feature 分支开发，突然需要切换到 main 修 bug
# 但当前工作还没做完，不想提交：
$ git stash

# 切换到 main 修 bug、提交...
$ git switch main
$ git commit -am "fix: 紧急修复"
$ git switch feature

# 恢复之前暂存的修改
$ git stash pop
```

---

### git cherry-pick
**功能**：将某个分支上的**单个提交**复制到当前分支

**示例**：

```bash
# 场景：main 分支有一个紧急修复提交 abc123，想应用到 feature 分支
$ git switch feature/login
$ git cherry-pick abc123

# cherry-pick 多个提交
$ git cherry-pick abc123 def456
```

---

### git rebase（交互模式）
**功能**：重写提交历史（合并、修改、删除、重排提交）

**示例**：

```bash
# 交互式 rebase 最近 3 次提交
$ git rebase -i HEAD~3
```

运行后会打开编辑器，显示如下内容：

```plain
pick abc123 添加登录页面
pick def456 修复样式问题
pick 789aaa 临时提交

# 常用操作：
# pick   = 保留提交（默认）
# reword = 修改提交消息
# edit   = 修改提交内容
# squash = 合并到前一个提交
# drop   = 删除提交
```

> ⚠️ **警告**：`git rebase` 会重写历史，**不要对已经推送到远程的提交执行 rebase**！
>

---

### git tag
**功能**：给某次提交打标签（通常用于标记版本号）

**语法**：

```bash
git tag                         # 列出所有标签
git tag <标签名>                 # 创建轻量标签
git tag -a <标签名> -m "说明"    # 创建附注标签（推荐）
git tag -d <标签名>              # 删除本地标签
git push origin <标签名>         # 推送标签到远程
git push origin --tags           # 推送所有标签
```

**示例**：

```bash
# 创建版本标签
$ git tag -a v1.0.0 -m "第一个正式版本"

# 给某次历史提交打标签
$ git tag -a v0.9.0 abc123 -m "beta 版本"

# 推送标签
$ git push origin v1.0.0

# 删除远程标签
$ git tag -d v0.9.0
$ git push origin :refs/tags/v0.9.0
```

---

### git blame
**功能**：查看文件中每一行代码是由谁、什么时候提交的（代码追溯）

**示例**：

```bash
# 查看某个文件的逐行作者信息
$ git blame README.md

# 查看指定行范围
$ git blame -L 10,20 README.md

# 忽略 whitespace 修改
$ git blame -w README.md
```

---

### git reflog
**功能**：查看 HEAD 的变动历史（**救命命令**，可以找回"丢失"的提交）

> ⚠️ **这是 Git 中最重要的恢复命令！** 当你误操作 `reset --hard` 后，可以用 `reflog` 找回丢失的提交。
>

**示例**：

```bash
# 查看 HEAD 变动记录
$ git reflog
# 输出示例：
# abc1234 (HEAD -> main) HEAD@{0}: reset --hard HEAD~1
# def5678 HEAD@{1}: commit: 添加新功能
# 901wxyz HEAD@{2}: commit: 修复 bug

# 找回丢失的提交（def5678 是误操作前最后一次好提交的 hash）
$ git reset --hard def5678
```

**原理**：Git 有一个 **30 天** 的垃圾回收机制，`reflog` 记录所有 HEAD 移动历史，即使 `git log` 看不到的提交，只要还在 GC 期内，就能通过 `reflog` 找回。

---

## 第 8 章：冲突解决
### 8.1 什么情况下会产生冲突？
当两个分支**修改了同一个文件的同一部分内容**，Git 无法自动合并时，就会产生冲突。

### 8.2 冲突文件的格式
```plain
<<<<<<< HEAD
这是当前分支的内容
=======
这是要合并进来的分支的内容
>>>>>>> feature/login
```

| 标记 | 含义 |
| --- | --- |
| `<<<<<<< HEAD` | 冲突开始，下面是当前分支的内容 |
| `=======` | 分隔线，上下分别是两个分支的内容 |
| `>>>>>>> feature/login` | 冲突结束，上面是要合并的分支名 |


### 8.3 解决冲突的步骤
```bash
# 1. 合并时发生冲突
$ git merge feature/login
# CONFLICT (content): Merge conflict in README.md
# Automatic merge failed; fix conflicts and then commit the result.

# 2. 打开冲突文件，手动编辑，删除 <<<<<<< ======= >>>>>>> 标记，保留正确内容

# 3. 标记冲突已解决
$ git add README.md

# 4. 完成合并提交
$ git commit -m "merge: 解决 README.md 冲突"
```

### 8.4 使用工具解决冲突
**VS Code**：

1. 打开冲突文件，VS Code 会自动识别冲突
2. 点击 **"Accept Current Change"** / **"Accept Incoming Change"** / **"Accept Both Changes"**
3. 保存文件，然后 `git add` + `git commit`

**中止合并（想放弃这次合并）**：

```bash
$ git merge --abort
```

---

## 第 9 章：.gitignore 详解
### 9.1 什么是 .gitignore？
`.gitignore` 文件告诉 Git **哪些文件/目录不需要跟踪**（不提交到仓库）。

### 9.2 语法规则
```plain
# 注释行（以 # 开头）

# 忽略单个文件
secret.txt

# 忽略整个目录（末尾加 /）
node_modules/

# 忽略所有 .log 文件
*.log

# 忽略所有 .tmp 文件，但不忽略 important.tmp
*.tmp
!important.tmp

# 忽略 build 目录下的所有内容
build/

# 忽略任意深度下的 target 目录
**/target/

# 忽略所有 .env 文件，除了 .env.example
.env
!.env.example
```

### 9.3 常见模板
**Node.js 项目**：

```plain
node_modules/
dist/
.env
.DS_Store
*.log
npm-debug.log*
```

**Python 项目**：

```plain
__pycache__/
*.py[cod]
*.so
venv/
.env
*.egg-info/
dist/
build/
```

**Java 项目**：

```plain
target/
*.class
*.jar
*.war
.idea/
*.iml
```

### 9.4 已跟踪文件如何从 Git 中移除
```bash
# 从 Git 跟踪中移除，但保留本地文件
$ git rm --cached config.env

# 从 Git 跟踪和本地都删除
$ git rm config.env
```

> 💡 **注意**：`.gitignore` 只对**未跟踪（Untracked）**的文件有效。如果文件已经被 Git 跟踪，需要先 `git rm --cached` 移除跟踪，之后再修改 `.gitignore` 才会生效。
>

---

## 第 10 章：Git 工作流实践
### 10.1 Centralized Workflow（集中式工作流）
适合小团队或个人项目：所有人直接推送到 `main` 分支。

```plain
clone → 修改 → commit → pull → push
```

### 10.2 Feature Branch Workflow（功能分支工作流）
**推荐大多数团队使用**。每个新功能在独立分支开发，通过 Pull Request 合并。

```plain
main ────────────────●───────────●────
                  ╱        ╲
feature/A ─ ●────●──────────        （开发完成后 PR → merge）
```

**流程**：

1. 从 `main` 创建功能分支：`git switch -c feature/xxx`
2. 在功能分支上开发、提交
3. 推送功能分支：`git push -u origin feature/xxx`
4. 创建 Pull Request，代码审查
5. 审查通过后合并到 `main`
6. 删除功能分支

### 10.3 Gitflow Workflow
适合有固定发布周期的项目（如软件版本发布）。

```plain
main ────●──────────●─────────  （生产环境）
           ╲         ╱
release ─────●─────●─────────  （预发布）
              ╲   ╱
develop ───────●─────────────  （开发主分支）
             ╱    ╲
feature ─ ●────●────────────  （功能分支）
```

**分支职责**：

+ `main`：生产环境对应代码，只接受来自 `release` 或 `hotfix` 的合并
+ `develop`：开发主分支，功能分支从这里拉取
+ `feature/*`：功能开发分支，从 `develop` 拉取，合并回 `develop`
+ `release/*`：发布准备分支，从 `develop` 拉取，合并到 `main` + `develop`
+ `hotfix/*`：线上紧急修复，从 `main` 拉取，合并到 `main` + `develop`

### 10.4 Forking Workflow（开源项目常用）
适用于开源项目，贡献者通过 Fork 自己的副本进行开发：

1. Fork 官方仓库到自己的账号
2. 克隆自己 Fork 的仓库到本地
3. 添加上游仓库：`git remote add upstream <官方仓库地址>`
4. 同步上游：`git pull upstream main`
5. 推送修改到自己的 Fork，然后提交 Pull Request

---

## 第 11 章：常见问题 FAQ
### Q1：提交了敏感信息（密码、私钥）怎么办？
**立即执行以下步骤**：

```bash
# 1. 从 Git 历史中彻底删除该文件
$ git rm --cached secret.env
# 确保 .gitignore 中已添加该文件

# 2. 用 git-filter-repo 工具清理历史（需要安装）
$ pip install git-filter-repo
$ git filter-repo --path secret.env --invert-paths

# 3. 强制推送（清理后的历史）
$ git push --force origin main
```

> ⚠️ **重要**：如果敏感信息已经推送，除了清理 Git 历史，还需要**立即撤销相关密码/密钥**！
>

---

### Q2：commit 消息写错了怎么改？
```bash
# 修改最后一次提交的消息（未 push）
$ git commit --amend -m "正确的提交消息"

# 如果已经 push，用 revert 新建提交说明错误，不要 amend
```

---

### Q3：想丢弃所有本地修改怎么操作？
```bash
# ⚠️ 危险！所有未提交的修改将永久丢失！
$ git reset --hard HEAD

# 同时清理未跟踪的文件和目录
$ git clean -fd
```

---

### Q4：push 被拒绝（non-fast-forward）怎么办？
```plain
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs to '...'
```

**原因**：远程仓库有你没有的提交（有人先推送了）。

**解决方法**：

```bash
# 方法 1：先拉取再推送（推荐）
$ git pull --rebase origin main
$ git push origin main

# 方法 2：强制推送（仅限个人分支，禁止在 main 使用！）
$ git push --force-with-lease origin main
```

---

### Q5：如何修改历史提交的 author 信息？
```bash
# 修改最后一次提交的作者（未 push）
$ git commit --amend --author="新名字 <new@email.com>"

# 修改多个历史提交的作者（谨慎使用）
$ git rebase -i HEAD~3
# 将需要修改的行改为 edit，然后逐个执行：
$ git commit --amend --author="新名字 <new@email.com>"
$ git rebase --continue
```

---

### Q6：如何撤销已经 push 的合并？
```bash
# 方法 1：revert 合并提交（推荐，不修改历史）
$ git revert -m 1 <merge-commit-hash>

# 方法 2：reset 后强制推送（危险，仅限个人分支）
$ git reset --hard HEAD~1
$ git push --force origin main
```

---

### Q7：git pull 时不想每次都输入用户名密码？
```bash
# 配置凭证存储（明文保存在磁盘，有效期 3600 秒）
$ git config --global credential.helper 'cache --timeout=3600'

# 永久保存凭证（Windows 推荐使用 Git Credential Manager）
$ git config --global credential.helper manager
```

---

## 第 12 章：速查表（Cheat Sheet）
### 配置相关
| 命令 | 说明 |
| --- | --- |
| `git config --global user.name "name"` | 设置用户名 |
| `git config --global user.email "email"` | 设置邮箱 |
| `git config --list` | 查看所有配置 |


### 仓库操作
| 命令 | 说明 |
| --- | --- |
| `git init` | 初始化仓库 |
| `git clone <url>` | 克隆远程仓库 |


### 日常提交
| 命令 | 说明 |
| --- | --- |
| `git status` | 查看状态 |
| `git add <file>` | 添加文件到暂存区 |
| `git add .` | 添加所有修改 |
| `git commit -m "msg"` | 提交 |
| `git commit -am "msg"` | 跳过 add 直接提交 |


### 分支操作
| 命令 | 说明 |
| --- | --- |
| `git branch` | 列出分支 |
| `git switch -c <name>` | 创建并切换分支 |
| `git switch <name>` | 切换分支 |
| `git branch -d <name>` | 删除分支 |
| `git merge <name>` | 合并分支 |


### 远程操作
| 命令 | 说明 |
| --- | --- |
| `git remote -v` | 查看远程仓库 |
| `git push origin <branch>` | 推送到远程 |
| `git pull` | 拉取并合并 |
| `git fetch` | 仅拉取不合并 |


### 撤销操作
| 命令 | 说明 |
| --- | --- |
| `git restore <file>` | 丢弃工作区修改 |
| `git restore --staged <file>` | 取消暂存 |
| `git reset --soft HEAD~1` | 撤销提交（保留修改） |
| `git reset --hard HEAD~1` | 撤销提交（丢弃修改） |
| `git revert <hash>` | 用新提交撤销（安全） |


### 查看历史
| 命令 | 说明 |
| --- | --- |
| `git log --oneline` | 简洁历史 |
| `git log --graph --all` | 图形化历史 |
| `git diff` | 查看未暂存差异 |
| `git diff --cached` | 查看已暂存差异 |
| `git blame <file>` | 查看逐行作者 |
| `git reflog` | 查看 HEAD 变动历史 |


---

## 附录
### A. Git 官方文档
+ 官方文档：[https://git-scm.com/docs](https://git-scm.com/docs)
+ Pro Git 电子书（免费）：[https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2)
+ Git 官方速查表：[https://education.github.com/git-cheat-sheet-education.pdf](https://education.github.com/git-cheat-sheet-education.pdf)

### B. 推荐学习资源
+ 廖雪峰 Git 教程：[https://www.liaoxuefeng.com/wiki/896043488029600](https://www.liaoxuefeng.com/wiki/896043488029600)
+ Visual Git Cheat Sheet：[https://ndpsoftware.com/git-cheatsheet.html](https://ndpsoftware.com/git-cheatsheet.html)
+ Learn Git Branching（交互式）：[https://learngitbranching.js.org/](https://learngitbranching.js.org/)

### C. 命令字母序索引
| 命令 | 章节 |
| --- | --- |
| `git add` | 第 3 章 |
| `git blame` | 第 7 章 |
| `git branch` | 第 3 章 |
| `git checkout` | 第 3 章 |
| `git clone` | 第 3 章 |
| `git commit` | 第 3 章 |
| `git config` | 第 1 章 |
| `git diff` | 第 3 章 |
| `git fetch` | 第 3 章 |
| `git init` | 第 3 章 |
| `git log` | 第 3 章 |
| `git merge` | 第 3 章 |
| `git pull` | 第 3 章 |
| `git push` | 第 3 章 |
| `git rebase` | 第 7 章 |
| `git reflog` | 第 7 章 |
| `git remote` | 第 4 章 |
| `git reset` | 第 6 章 |
| `git restore` | 第 6 章 |
| `git revert` | 第 6 章 |
| `git rm` | 第 9 章 |
| `git stash` | 第 7 章 |
| `git status` | 第 3 章 |
| `git switch` | 第 3 章 |
| `git tag` | 第 7 章 |


---

_本文档基于 _`progress.md`_ 分析结果编写，涵盖 Git 日常使用 95% 以上的场景。_
