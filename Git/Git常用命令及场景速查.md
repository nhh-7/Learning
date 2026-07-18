# Git 常用命令及团队开发场景速查手册

> **目标**：覆盖日常团队开发中 90% 的 Git 使用场景，每个命令都附带典型场景说明。

---

## 目录

1. [基础配置与仓库初始化](#1-基础配置与仓库初始化)
2. [暂存与提交](#2-暂存与提交)
3. [分支操作](#3-分支操作)
4. [同步与协作（fetch / pull / push）](#4-同步与协作fetch--pull--push)
5. [合并与变基（merge / rebase）](#5-合并与变基merge--rebase)
6. [暂存工作现场（stash）](#6-暂存工作现场stash)
7. [查看历史（log / reflog / blame）](#7-查看历史log--reflog--blame)
8. [撤销与回退（reset / revert / restore）](#8-撤销与回退reset--revert--restore)
9. [Cherry-pick：挑选提交](#9-cherry-pick挑选提交)
10. [标签管理（tag）](#10-标签管理tag)
11. [冲突解决](#11-冲突解决)
12. [常用组合场景速查](#12-常用组合场景速查)
13. [附录：Conventional Commits 规范](#13-附录conventional-commits-规范)

---

## 1. 基础配置与仓库初始化

### 1.1 配置用户信息

```bash
# 设置全局用户名和邮箱（提交时显示的身份）
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"

# 为单个仓库设置不同的身份（如公司项目 vs 个人项目）
git config user.name "Work Name"
git config user.email "work@company.com"

# 查看当前配置
git config --list
git config user.name          # 查看单项
```

> **场景**：入职第一天配置；或同一台电脑需要区分公司/个人 GitHub 身份。

### 1.2 常用配置优化

```bash
# 设置默认分支名为 main（GitHub 新标准）
git config --global init.defaultBranch main

# 设置别名，减少输入
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"

# 设置 pull 时默认用 rebase 而非 merge（保持历史线性）
git config --global pull.rebase true
```

### 1.3 克隆与初始化

```bash
# 克隆远程仓库
git clone https://github.com/team/project.git
git clone git@github.com:team/project.git          # SSH 方式（推荐，免密）

# 克隆指定分支
git clone -b develop https://github.com/team/project.git

# 在已有目录中初始化仓库并关联远程
git init
git remote add origin git@github.com:team/project.git
```

### 1.4 远程仓库管理

```bash
git remote -v                                       # 查看远程仓库地址
git remote add upstream git@github.com:upstream/project.git  # 添加上游仓库（Fork 场景）
git remote set-url origin git@github.com:new/url.git         # 修改远程地址
git remote show origin                              # 查看远程仓库详细信息
```

---

## 2. 暂存与提交

### 2.1 查看状态与差异

```bash
git status                          # 查看工作区状态（最常用，每天 N 次）
git status -s                       # 简短格式
git diff                            # 查看工作区 vs 暂存区的差异
git diff --staged                   # 查看暂存区 vs 上次提交的差异（即即将提交的内容）
git diff HEAD                       # 查看工作区 + 暂存区 vs 上次提交的差异
git diff <commit1> <commit2>        # 比较两次提交的差异
git diff main..feature/xxx          # 比较两个分支的差异
```

> **场景**：提交前自检——`git diff --staged` 确认即将提交的内容没有意外混入调试代码或敏感信息。

### 2.2 暂存文件

```bash
git add <file>                      # 暂存指定文件
git add .                           # 暂存所有变更（新增 + 修改 + 删除）
git add -A                          # 同上，暂存所有
git add -p                          # 交互式暂存，逐块确认（强烈推荐！）
git add -u                          # 只暂存已跟踪文件的修改和删除，不包含新文件

# 取消暂存（保留工作区修改）
git restore --staged <file>         # Git 2.23+ 推荐方式
git reset HEAD <file>               # 传统方式
```

> **场景**：`git add -p` 是最佳实践——当你在同一个文件里做了多个不相关的改动，用它把改动拆成多次提交，每次提交只包含一个逻辑完整的变更。

### 2.3 提交

```bash
git commit -m "feat: add user avatar upload"        # 提交并写消息
git commit -m "fix: handle null pointer in payment" # 修复类提交
git commit -am "chore: update dependencies"          # 跳过 git add，直接提交所有已跟踪文件的修改（不包括新文件）

# 修改上一次提交（最常用场景：漏了文件或提交消息写错了）
git commit --amend -m "new message"                  # 修改提交消息
git commit --amend --no-edit                         # 追加遗漏的文件，不改变消息

# ⚠️ 注意：--amend 会改写历史！只能 amend 还没推送到远程的提交
```

> **⚠️ 黄金法则**：永远不要 `--amend` 或 `rebase` 已经推送到远程共享分支的提交！

---

## 3. 分支操作

### 3.1 查看分支

```bash
git branch                          # 查看本地分支
git branch -r                       # 查看远程分支
git branch -a                       # 查看所有分支（本地 + 远程）
git branch -v                       # 查看分支及其最后一次提交
git branch --merged                 # 查看已合并到当前分支的分支（可安全删除）
git branch --no-merged              # 查看未合并到当前分支的分支
```

### 3.2 创建与切换分支

```bash
# 创建并切换到新分支（基于当前分支）
git switch -c feature/new-feature           # Git 2.23+ 推荐
git checkout -b feature/new-feature         # 传统方式

# 基于指定分支创建
git switch -c feature/new-feature develop   # 基于 develop 创建

# 基于远程分支创建本地跟踪分支
git switch -c feature/new-feature --track origin/feature/new-feature

# 切换到已有分支
git switch main                             # 推荐
git checkout main                           # 传统

# 基于远程已有的同名分支，自动创建本地跟踪分支
git switch feature/exists-on-remote         # Git 智能识别远程同名分支
```

### 3.3 分支的删除与重命名

```bash
# 删除本地分支（安全删除，已合并的才能删）
git branch -d feature/done

# 强制删除本地分支（不管是否合并）
git branch -D feature/abandoned

# 删除远程分支
git push origin --delete feature/done

# 重命名分支
git branch -m old-name new-name             # 重命名当前分支
git branch -m old-name new-name             # 重命名指定分支

# 清理本地已不存在的远程分支引用
git remote prune origin
```

### 3.4 查看分支的上游跟踪关系

```bash
git branch -vv                      # 查看每个分支跟踪的远程分支及 ahead/behind 状态
```

---

## 4. 同步与协作（fetch / pull / push）

### 4.1 核心概念

```
git fetch   = 从远程下载最新数据，但不合并到本地分支（安全，只读）
git pull    = git fetch + git merge  （两步合一）
git push    = 将本地提交推送到远程
```

**理解 `origin`**：`origin` 是远程仓库的默认别名，指向你 clone 的那个远端。`origin/main` 表示远程 `main` 分支在你本地的**远程跟踪引用**（不是本地的 `main` 分支！）。

### 4.2 git fetch —— 拉取但不合并

```bash
git fetch                           # 拉取所有远程分支的最新状态
git fetch origin                    # 同上，指定远程仓库名
git fetch origin main               # 只拉取远程 main 分支

# fetch 后查看远程有什么更新
git log origin/main                 # 查看远程 main 的提交历史
git diff main origin/main           # 比较本地 main 和远程 main 的差异
git log main..origin/main           # 查看远程 main 比本地多出的提交
```

> **场景**：你想先看看同事推送了什么，再决定是否合并。或者你想先 `fetch`，然后 `rebase` 而不是 `merge`。

### 4.3 git pull —— 拉取并合并

```bash
git pull                            # = git fetch + git merge
git pull origin develop             # 拉取远程 develop 并合并到当前分支
git pull --rebase                   # = git fetch + git rebase（保持线性历史，强烈推荐！）
git pull --rebase origin develop    # 指定分支的 rebase 方式拉取
```

> **场景选择**：
> - **`git pull`（默认 merge）**：简单直接，但会产生一个 merge commit，历史图会有分叉。
> - **`git pull --rebase`**：将你的本地提交"移到"远程最新提交之后，历史是一条直线。**团队开发推荐此方式**。

### 4.4 git push —— 推送本地提交

```bash
git push                            # 推送到当前分支跟踪的上游
git push origin feature/xxx         # 推送到指定远程分支
git push -u origin feature/xxx      # 首次推送并设置上游跟踪（之后只需 git push）
git push --force-with-lease         # 安全强制推送（推荐代替 --force）
git push --force                    # 危险！覆盖远程历史，只有明确知道后果时使用
```

> **⚠️ `--force` vs `--force-with-lease`**：
> - `--force`：无条件覆盖远程，可能覆盖同事刚推送的提交。
> - `--force-with-lease`：只在远程分支没有被他人更新时才强制推送。**始终优先用它！**
>
> **场景**：你 `amend` 或 `rebase` 了已经推送的分支（且该分支只有你一个人在用），需要用 force push 更新远程。

---

## 5. 合并与变基（merge / rebase）

### 5.1 分支合并的核心困惑

| 操作 | 效果 | 产生的历史 |
|------|------|-----------|
| `git merge` | 将两个分支的历史"汇合"，创建一个 merge commit | 有分叉，保留完整历史 |
| `git rebase` | 将当前分支的提交"移植"到目标分支最新提交之后 | 线性，历史干净整洁 |

### 5.2 git merge —— 合并分支

```bash
# 将 develop 合并到当前分支
git merge develop

# 将 feature/xxx 合并到当前分支
git merge feature/xxx

# 合并时禁止快进（总是创建 merge commit）
git merge --no-ff feature/xxx

# 取消正在进行的 merge（冲突太多不想解决了）
git merge --abort
```

> **场景**：
> - **`--no-ff`（禁止快进）**：合并 feature 到 develop 时使用，保留"这个功能是一个分支"的记录，方便追溯和回滚。
> - **Fast-forward（快进）**：当目标分支没有新提交，直接把指针移到最新提交，不产生 merge commit。

```
# 快进合并（默认，当可能时）
# Before:  A---B---C  (main)
#                     D---E  (feature)
# After:   A---B---C---D---E  (main, feature)
# → 看不出 feature 曾经是个独立分支

# --no-ff 合并
# Before:  A---B---C  (main)
#                     D---E  (feature)
# After:   A---B---C-------M  (main)    M 是 merge commit
#                  \     /
#                   D---E  (feature)
# → 可以清楚看到 feature 分支的起点和终点
```

### 5.3 git rebase —— 变基

```bash
# 将当前分支的提交 rebase 到 develop 最新提交之后
git rebase develop

# 将 feature 分支 rebase 到 develop（不管当前在哪个分支）
git rebase develop feature/xxx

# 交互式 rebase（最强大也最常用！）
git rebase -i HEAD~3           # 整理最近 3 次提交
git rebase -i develop          # 将当前分支上不在 develop 中的提交进行交互式整理
```

**交互式 rebase 可以做什么：**

| 命令 | 作用 |
|------|------|
| `pick` | 保留该提交（默认） |
| `reword` | 修改提交消息 |
| `squash` | 合并到上一个提交，保留提交消息 |
| `fixup` | 合并到上一个提交，丢弃提交消息 |
| `drop` | 删除该提交 |
| `edit` | 暂停，允许修改该提交的内容 |

> **场景**：你在 feature 分支上提交了 5 次："WIP"、"fix typo"、"fix typo again"、"finally works"、"真的好了"。推送之前用 `git rebase -i develop` 把 5 次压缩成 1 次干净的提交，然后 force push（如果已经推送过的话）。

### 5.4 Merge vs Rebase 选择指南

| 场景 | 推荐 | 原因 |
|------|------|------|
| 将 feature 合并到 develop/main | `git merge --no-ff` | 保留功能分支的完整性，方便回滚 |
| 将 develop 的最新代码同步到正在开发的 feature 分支 | `git rebase develop` | 保持 feature 分支的提交在 develop 最新提交之后，减少冲突 |
| 拉取远程更新到本地分支 | `git pull --rebase` | 避免无意义的 "Merge branch 'main' of ..." 提交 |
| 公开分支（多人共用） | **不要 rebase！** | 会改写历史，导致同事的仓库混乱 |

> **核心准则**：**只 rebase 你一个人的私有分支。永远不要 rebase 公开分支（main、develop 等）。**

---

## 6. 暂存工作现场（stash）

### 6.1 核心操作

```bash
git stash                           # 暂存所有修改（工作区 + 暂存区）
git stash save "描述信息"            # 暂存并添加描述
git stash -u                        # 暂存包括未跟踪的文件
git stash -a                        # 暂存包括未跟踪和忽略的文件

git stash list                      # 查看所有暂存
git stash pop                       # 恢复最近一次暂存，并删除该 stash
git stash pop stash@{1}             # 恢复指定的 stash
git stash apply                     # 恢复最近一次暂存，但保留 stash（不删除）
git stash apply stash@{1}           # 恢复指定的 stash，保留

git stash drop stash@{0}            # 删除某个 stash
git stash clear                     # 清空所有 stash
```

### 6.2 典型场景

**场景 A：紧急插队——手头工作做到一半，需要切分支修 bug**

```bash
# 1. 暂存当前工作
git stash -u

# 2. 切换到主分支修 bug
git switch main
git switch -c hotfix/critical-bug
# ... 修复、提交、推送、合并 ...

# 3. 回到原来的分支，恢复工作
git switch feature/my-work
git stash pop
```

**场景 B：把 stash 应用到不同分支**

```bash
# 在 feature/a 分支上 stash 了工作
git stash

# 发现应该在 feature/b 分支做这个改动
git switch feature/b
git stash pop           # stash 内容被应用到 feature/b
```

**场景 C：查看 stash 里存了什么**

```bash
git stash show -p                   # 查看最近 stash 的详细 diff
git stash show -p stash@{0}         # 查看指定 stash 的详细 diff
```

---

## 7. 查看历史（log / reflog / blame）

### 7.1 git log —— 提交历史

```bash
git log                             # 查看提交历史（详细）
git log --oneline                   # 每个提交一行（简洁）
git log --oneline --graph --all     # 可视化分支图（最常用！）
git log --oneline -10               # 最近 10 条
git log --author="张三"             # 查看某人的提交
git log --grep="fix"                # 搜索提交消息中包含 "fix" 的
git log --since="2024-01-01" --until="2024-01-31"  # 按时间范围
git log -p                          # 显示每次提交的详细 diff
git log --stat                      # 显示每次提交的文件变更统计
git log <file>                      # 查看某文件的提交历史
git log main..feature/xxx           # 查看 feature/xxx 有但 main 没有的提交

# 推荐别名（添加到 ~/.gitconfig）
# git config --global alias.lg "log --oneline --graph --all --decorate"
```

### 7.2 git reflog —— 操作历史（救命稻草！）

```bash
git reflog                          # 查看所有 Git 操作的历史（包括已删除的提交！）
git reflog <branch>                 # 查看某分支的操作历史
```

> **场景（救命！）**：你 `git reset --hard` 误删了提交，或者 `rebase` 搞砸了。只要能在 `reflog` 里找到那个提交的 hash，就能恢复。Git 默认保留 90 天的 reflog。

```bash
# 恢复被误删的提交
git reflog                          # 找到丢失提交的 hash（如 abc1234）
git checkout -b recovery abc1234    # 基于该 hash 创建新分支恢复
# 或者
git reset --hard abc1234            # 直接重置到那个提交
```

### 7.3 git blame —— 追责（定位问题来源）

```bash
git blame <file>                    # 查看文件每一行的最后修改者和提交
git blame -L 10,50 <file>           # 只看第 10-50 行
git blame -L '/function login/' <file>  # 只看 login 函数所在行
git blame <commit> -- <file>        # 查看某次提交时的文件状态
```

> **场景**：线上出 bug 了，追踪某行代码是谁、什么时候、在哪个 commit 里引入的，帮助定位上下文和修复方案。

---

## 8. 撤销与回退（reset / revert / restore）

### 8.1 三个命令的职责分工（Git 2.23+）

| 命令 | 作用域 | 用途 |
|------|--------|------|
| `git restore` | 工作区 / 暂存区 | 丢弃未提交的修改 |
| `git reset` | 提交历史 / 暂存区 | 移动 HEAD 和分支指针 |
| `git revert` | 提交历史 | 创建一个新提交来撤销某次提交（安全，不改历史） |

### 8.2 git restore —— 丢弃未提交的修改

```bash
# 丢弃工作区的修改（恢复到上次提交的状态）
git restore <file>                  # 丢弃单个文件的修改
git restore .                       # 丢弃所有修改

# 取消暂存（从暂存区移回工作区）
git restore --staged <file>         # 等价于 git reset HEAD <file>
git restore --staged .              # 取消所有暂存

# 同时取消暂存并丢弃工作区修改
git restore --staged --worktree <file>
```

### 8.3 git reset —— 重置提交历史

```bash
# --soft：只移动 HEAD，保留暂存区和工作区
git reset --soft HEAD~1             # 撤销最近一次提交，修改回到暂存区
# 场景：提交消息写错了，或者想多攒几个修改一起提交

# --mixed（默认）：移动 HEAD，重置暂存区，保留工作区
git reset HEAD~1                    # 撤销最近一次提交和暂存，修改回到工作区
git reset HEAD~3                    # 撤销最近 3 次提交
# 场景：最近几次提交太零碎，想重新组织

# --hard：移动 HEAD，重置暂存区和工作区（⚠️ 危险！）
git reset --hard HEAD~1             # 彻底丢弃最近一次提交和所有修改
# 场景：确定最近的提交完全没用，想要彻底回退
# ⚠️ 使用前一定确认：git status 和 git log 看清楚
```

```
原始状态:  A---B---C---D  (HEAD -> main)

git reset --soft HEAD~1:   A---B---C  (HEAD -> main)  [D 的修改在暂存区]
git reset --mixed HEAD~1:  A---B---C  (HEAD -> main)  [D 的修改在工作区]
git reset --hard HEAD~1:   A---B---C  (HEAD -> main)  [D 的修改彻底丢失]
```

### 8.4 git revert —— 安全的撤销（推荐用于已推送的提交）

```bash
git revert HEAD                     # 创建一个新提交，撤销最近一次提交的修改
git revert <commit-hash>            # 撤销指定的某次提交
git revert HEAD~3..HEAD             # 撤销最近 3 次提交（会创建 3 个 revert commit）
git revert --no-commit HEAD~3..HEAD # 撤销最近 3 次提交，但不自动提交（可以合并成一个）
```

> **场景**：已经推送到远程的提交出了问题。用 `revert` 而不是 `reset`，因为 `revert` 是追加一个"撤销提交"，不改变已有历史，同事拉取不会冲突。

### 8.5 选择 reset 还是 revert？

| 情况 | 推荐 |
|------|------|
| 提交还在本地，没推送 | `git reset` |
| 提交已经推送到远程共享分支 | `git revert` |
| 个人分支，只有自己在用 | `git reset` + `git push --force-with-lease` |
| 不确定 | 用 `git revert`，它总是安全的 |

---

## 9. Cherry-pick：挑选提交

### 9.1 基本用法

```bash
git cherry-pick <commit-hash>               # 将某次提交应用到当前分支
git cherry-pick <hash1> <hash2> <hash3>     # 挑选多个提交
git cherry-pick <hash1>..<hash3>            # 挑选一个范围内的提交（不包含 hash1）
git cherry-pick <hash1>^..<hash3>           # 挑选一个范围内的提交（包含 hash1）
```

### 9.2 典型场景

**场景：从 develop 分支挑选一个 bug 修复应用到 release 分支**

```bash
git switch release/v2.0
git cherry-pick abc1234            # abc1234 是那个 bug 修复在 develop 上的 commit hash
```

**场景：cherry-pick 时发生冲突**

```bash
# 解决冲突后
git add .
git cherry-pick --continue         # 继续 cherry-pick

# 放弃 cherry-pick
git cherry-pick --abort
```

> **场景**：hotfix 分支修复了一个紧急 bug，修复已经合并到 main。现在需要把这个修复也应用到正在开发的 develop 分支，但不想把整个 hotfix 分支合并过来——用 `cherry-pick` 只拿那一个修复提交。

---

## 10. 标签管理（tag）

### 10.1 基本用法

```bash
git tag                             # 列出所有标签
git tag -l "v2.*"                   # 列出匹配的标签

# 轻量标签（只是一个指向提交的指针）
git tag v1.0.0

# 附注标签（包含标签消息、日期、作者，推荐用于发布版本）
git tag -a v1.0.0 -m "Release version 1.0.0"

# 给历史提交打标签
git tag -a v0.9.0 <commit-hash> -m "Beta release"

# 推送标签到远程
git push origin v1.0.0              # 推送单个标签
git push origin --tags              # 推送所有标签

# 删除标签
git tag -d v1.0.0                   # 删除本地标签
git push origin --delete v1.0.0     # 删除远程标签
```

### 10.2 查看标签信息

```bash
git show v1.0.0                     # 查看标签详情和对应的提交
git checkout v1.0.0                 # 切换到标签对应的提交（detached HEAD 状态）
git switch -c branch-from-tag v1.0.0  # 基于标签创建新分支
```

> **场景**：每次发布生产版本时打一个附注标签（如 `v2.3.1`），方便快速回滚到任意历史版本，也方便查看每个版本包含了哪些改动。

---

## 11. 冲突解决

### 11.1 冲突是怎么产生的

当两个分支修改了同一个文件的同一行（或相邻行），Git 无法自动决定保留哪个版本，就会标记冲突。

### 11.2 解决冲突的标准流程

```bash
# 1. merge 或 rebase 时出现冲突提示
git merge develop
# CONFLICT (content): Merge conflict in src/app.ts

# 2. 查看哪些文件有冲突
git status

# 3. 打开冲突文件，找到冲突标记
# <<<<<<< HEAD
#   你的版本（当前分支的代码）
# =======
#   对方的版本（要合并进来的分支的代码）
# >>>>>>> develop

# 4. 手动编辑文件，保留正确的内容，删除冲突标记

# 5. 标记冲突已解决
git add <resolved-file>

# 6. 继续
git merge --continue        # 如果是 merge
git rebase --continue       # 如果是 rebase

# 如果解决到一半想放弃
git merge --abort           # 放弃 merge
git rebase --abort          # 放弃 rebase
```

### 11.3 冲突解决辅助工具

```bash
# 查看冲突文件的三方对比
git diff                         # 查看冲突标记
git diff --ours                  # 只看 HEAD（自己）的部分
git diff --theirs                # 只看对方的部分
git diff --base                  # 只看共同祖先的部分

# 完全采用某一方的版本（⚠️ 谨慎使用）
git checkout --ours <file>       # 采用当前分支的版本
git checkout --theirs <file>     # 采用合并进来的版本

# 使用可视化工具
git mergetool                    # 启动配置的 merge 工具（如 VSCode、Meld 等）
```

> **最佳实践**：
> 1. 频繁从 develop rebase 或 merge，小步快跑，避免积累大量冲突。
> 2. 冲突文件解决后，运行项目测试确保合并没有破坏功能。
> 3. 不确定时，找原来写这段代码的同事一起看。

---

## 12. 常用组合场景速查

### 场景 1：每日开工——同步最新代码

```bash
git switch develop
git fetch origin
git pull --rebase origin develop
```

### 场景 2：开始一个新功能

```bash
git switch develop
git pull --rebase origin develop      # 确保本地 develop 最新
git switch -c feature/my-feature      # 创建功能分支
# ... 开发、提交 ...
```

### 场景 3：功能开发中，同步 develop 的最新代码

```bash
git fetch origin
git rebase origin/develop
# 如有冲突 → 解决 → git add . → git rebase --continue
```

### 场景 4：提交前整理历史

```bash
git log --oneline                     # 看看自己提交了什么
# 假设有 5 个小提交需要合并
git rebase -i HEAD~5
# 将后 4 个改为 squash 或 fixup，只保留第一个为 pick
# 保存退出后编辑合并后的提交消息
```

### 场景 5：紧急修复线上 Bug（hotfix）

```bash
git switch main
git pull --rebase origin main
git switch -c hotfix/critical-fix
# ... 修复 bug、测试 ...
git add .
git commit -m "fix: resolve critical payment crash"
git push -u origin hotfix/critical-fix
# → 创建 PR 合并到 main
# → main 合并后打 tag
git switch main
git pull --rebase origin main
git tag -a v1.2.1 -m "Hotfix: payment crash"
git push origin v1.2.1
# → 别忘了也合并到 develop
git switch develop
git pull --rebase origin develop
git merge hotfix/critical-fix
git push origin develop
# → 清理
git branch -d hotfix/critical-fix
git push origin --delete hotfix/critical-fix
```

### 场景 6：代码审查后需要修改

```bash
# PR 已经被审查，同事提了修改建议
# 直接在同一个 feature 分支修改
git add .
git commit -m "fix: address review comments"
git push
# PR 会自动更新，不需要重新创建
# 如果想保持只有 1 个干净提交，可以用 amend：
git add .
git commit --amend --no-edit
git push --force-with-lease
```

### 场景 7：回滚已上线的功能

```bash
# 方法一：revert（推荐，安全）
git switch main
git pull --rebase origin main
git revert <有问题的那次提交的hash>
git push origin main

# 方法二：如果是一整个 feature 的合并提交
git revert -m 1 <merge-commit-hash>   # -m 1 表示保留 main 的版本
```

### 场景 8："我的代码跑哪里去了？"（reflog 救命）

```bash
git reflog
# 找到丢失前最后一次正常的提交 hash
git reset --hard <hash>
# 或者保险起见创建新分支
git switch -c recovery-branch <hash>
```

### 场景 9：把某个分支上的单个文件拿过来

```bash
git checkout feature/other-branch -- src/utils/helper.ts
git commit -m "chore: borrow helper from other branch"
```

### 场景 10：临时保存工作，切换到另一个任务

```bash
git stash -u                    # 暂存当前所有工作
git switch develop
git switch -c hotfix/urgent
# ... 修复、提交、推送、合并 ...
git switch feature/original
git stash pop                   # 恢复原来的工作
```

### 场景 11：Pull Request 依赖链（B 功能依赖 A 功能）

```bash
# 从 A 分支创建 B 分支（而不是从 develop 创建）
git switch feature/feature-A
git switch -c feature/feature-B
# B 分支包含了 A 的所有提交
# A 合并到 develop 后，B 再 rebase 到 develop：
git fetch origin
git rebase origin/develop
# Git 会自动识别 A 的提交已经被合并，只保留 B 独有的提交
```

### 场景 12：二分查找定位 Bug 引入点（git bisect）

```bash
git bisect start                    # 开始二分查找
git bisect bad HEAD                 # 标记当前版本有问题
git bisect good v1.0.0              # 标记 v1.0.0 是好的
# Git 会自动 checkout 到中间的一个提交
# 测试这个版本有没有 bug
git bisect bad                      # 如果这个版本有问题
# 或
git bisect good                     # 如果这个版本没问题
# 重复以上步骤，直到 Git 定位到引入 bug 的那次提交
git bisect reset                    # 结束二分查找，回到原来的分支
```

---

## 13. 附录：Conventional Commits 规范

团队协作中，统一提交消息格式能让 `git log` 一目了然。

### 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### type 类型

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: add user avatar upload` |
| `fix` | Bug 修复 | `fix: handle null pointer in payment flow` |
| `refactor` | 重构（不改变功能） | `refactor: extract common validation logic` |
| `docs` | 文档更新 | `docs: update API documentation` |
| `style` | 代码格式（不影响逻辑） | `style: format code with prettier` |
| `test` | 测试相关 | `test: add unit tests for auth service` |
| `chore` | 构建、依赖等杂务 | `chore: update lodash to 4.17.21` |
| `perf` | 性能优化 | `perf: optimize image loading with lazy load` |
| `ci` | CI/CD 配置 | `ci: add GitHub Actions workflow` |
| `build` | 构建系统变更 | `build: update webpack config` |

### Scope（可选）

表示影响的范围：`feat(auth): add OAuth2 login`

### Breaking Changes

在 body 或 footer 中标明：

```
feat(api): change user endpoint response format

BREAKING CHANGE: The user endpoint now returns `email` instead of `emailAddress`.
```

---

## 核心原则总结

1. **永远不要在共享分支上 force push 或 rebase。**
2. **提交小而频繁，每个提交只做一件事。**
3. **提交前用 `git diff --staged` 自检。**
4. **拉取用 `git pull --rebase`，保持历史干净。**
5. **Merge feature 用 `--no-ff`，保留分支轨迹。**
6. **遇到冲突不要慌，`git merge --abort` / `git rebase --abort` 可以重来。**
7. **搞砸了先看 `git reflog`——几乎所有操作都能找回。**
8. **推送前整理提交历史（交互式 rebase），让 PR 更易审查。**
9. **不确定时用 `revert` 而不是 `reset`，尤其是在共享分支上。**
10. **统一提交消息格式，利人利己。**

---

> 📚 **补充阅读**：同目录下的 `git工作流.md` 详细介绍了 Git Flow 分支策略（main、develop、feature、hotfix 的生命周期），建议配合阅读。
