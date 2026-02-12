---
title: Git 核心知识总结
published: 2026-02-11T18:13:21.381Z
tags: [Markdown, git, 博客, 演示]
pinned: false
image: "./images/a4.png"
category: 开发笔记
draft: true
---

# Git 核心知识总结

## 一、Git 简介

### 1.1 什么是 Git

-   **分布式版本控制系统**：每个人本地都拥有完整的代码仓库历史
-   **快照系统**：每次提交都是对当前文件系统的快照
-   **近乎所有操作都是本地执行**：速度快，无需网络
-   **完整性保证**：所有数据在存储前都计算校验和（SHA-1）

### 1.2 Git vs 集中式版本控制（SVN）

| 特性         | Git                 | SVN                 |
|:-------------|:-------------------:|-------------------:|
| 架构         | 分布式             | 集中式             |
| 网络依赖     | 仅推送/拉取时需要  | 几乎所有操作都需要 |
| 分支         | 轻量级，切换快     | 分支是完整拷贝，较慢 |
| 存储         | 元数据+文件快照    | 差异增量存储       |
| 离线工作     | 完整本地历史       | 部分受限           |

---

## 二、Git 核心概念

### 2.1 三大区域

-   **工作区（Working Directory）**：本地文件系统看到的实际文件
-   **暂存区（Staging Area / Index）**：下一次提交将要保存的文件清单
-   **本地仓库（Repository / .git）**：保存所有历史版本和元数据

### 2.2 文件状态生命周期

```
Untracked（未跟踪） → Staged（暂存） ← Modified（修改）
          ↓               ↓
      Unmodified（未修改）→ Committed（提交）
```

---

## 三、基础配置与初始化

### 3.1 用户配置

```bash
# 全局配置（推荐）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 项目级配置
git config --local user.name "Project Name"

# 查看配置
git config --list
git config user.name
```

### 3.2 创建仓库

```bash
# 初始化新仓库
git init

# 克隆远程仓库
git clone <repository-url>
git clone <repository-url> <directory-name>   # 指定目录
git clone --depth 1 <repo-url>                # 浅克隆（只拉取最新提交）
```

---

## 四、日常基本操作

### 4.1 添加与提交

```bash
# 查看状态
git status
git status -s   # 简短输出

# 添加文件到暂存区
git add <file>         # 添加单个文件
git add .              # 添加当前目录所有更改（包括新文件）
git add -A             # 添加所有更改（全工作区）
git add -p             # 交互式逐块添加

# 提交
git commit -m "feat: add login function"
git commit -a -m "fix: typo"   # 跳过暂存区（仅对已跟踪文件）
git commit --amend -m "new message"   # 修改最近一次提交信息

# 删除文件
git rm <file>           # 删除并暂存删除操作
git rm --cached <file>  # 从版本控制移除但保留工作区文件

# 移动/重命名
git mv <old> <new>
```

### 4.2 查看历史

```bash
# 完整日志
git log
git log --oneline       # 单行显示
git log --graph         # 图形化分支图
git log -p              # 显示具体改动
git log --stat          # 显示文件修改统计
git log --author="name" # 按作者筛选
git log --since="2 days ago"   # 时间筛选

# 查看某个文件的历史
git log -p <file>
git blame <file>        # 逐行显示最后修改者
```

### 4.3 比较差异

```bash
# 工作区 vs 暂存区
git diff

# 暂存区 vs 最新提交
git diff --staged
git diff --cached

# 工作区 vs 最新提交
git diff HEAD

# 两个提交之间的差异
git diff <commit-id1> <commit-id2>
```

### 4.4 撤销操作

```bash
# 撤销工作区修改（未 add）
git restore <file>                 # Git 2.23+
git checkout -- <file>            # 传统方式

# 撤销暂存区（回到工作区）
git restore --staged <file>       # Git 2.23+
git reset HEAD <file>            # 传统方式

# 修改最近一次提交
git commit --amend

# 撤销提交（保留修改到工作区）
git reset --soft HEAD~1

# 撤销提交（保留修改到暂存区）
git reset HEAD~1                 # 默认 --mixed

# 彻底撤销提交（丢弃修改）
git reset --hard HEAD~1

# 强制推送到远程（谨慎使用！）
git push --force origin branch

# 使用 reflog 恢复误删提交
git reflog
git reset --hard HEAD@{n}
```

---

## 五、分支管理

### 5.1 分支基本操作

```bash
# 查看分支
git branch                # 本地分支
git branch -r             # 远程分支
git branch -a             # 所有分支

# 创建分支
git branch <branch-name>               # 基于当前 HEAD
git branch <branch-name> <commit-id>   # 基于特定提交
git checkout -b <branch-name>          # 创建并切换

# 切换分支
git checkout <branch-name>
git switch <branch-name>              # Git 2.23+

# 删除分支
git branch -d <branch-name>           # 已合并的分支
git branch -D <branch-name>           # 强制删除（未合并）

# 重命名分支
git branch -m <old-name> <new-name>
```

### 5.2 合并与变基

```bash
# 合并分支（将 feature 合并到当前分支）
git merge <feature-branch>

# 合并时使用 --no-ff（保留分支历史）
git merge --no-ff <branch>

# 中止合并（遇到冲突时）
git merge --abort

# 变基（将当前分支提交移到目标分支顶端）
git rebase <base-branch>

# 交互式变基（整理提交历史）
git rebase -i HEAD~3

# 变基时跳过冲突提交
git rebase --skip
# 变基过程中解决冲突后
git add .
git rebase --continue
```

### 5.3 冲突解决

1.  合并/变基后产生冲突，Git 标记冲突文件
2.  手动编辑文件，保留最终内容
3.  `git add <file>` 标记为已解决
4.  `git commit`（合并）或 `git rebase --continue`（变基）

---

## 六、远程协作

### 6.1 远程仓库管理

```bash
# 查看远程仓库
git remote -v
git remote show origin

# 添加远程仓库
git remote add origin <url>

# 修改远程地址
git remote set-url origin <new-url>

# 删除远程仓库
git remote remove origin

# 重命名远程
git remote rename origin upstream
```

### 6.2 推送与拉取

```bash
# 推送分支
git push origin <branch>
git push -u origin <branch>   # 设置上游关联

# 删除远程分支
git push origin --delete <branch>
git push origin :<branch>     # 旧式语法

# 拉取远程更新（不自动合并）
git fetch origin

# 拉取并合并（相当于 fetch + merge）
git pull origin <branch>

# 拉取并变基
git pull --rebase origin <branch>
git config --global pull.rebase true   # 设置默认变基
```

### 6.3 标签管理

```bash
# 创建标签
git tag v1.0.0                          # 轻量标签
git tag -a v1.0.0 -m "Release version 1.0.0"   # 附注标签
git tag -a v1.0.0 <commit-id>

# 查看标签
git tag
git show v1.0.0

# 推送标签
git push origin v1.0.0
git push origin --tags        # 推送所有本地未推送标签

# 删除标签
git tag -d v1.0.0             # 本地删除
git push origin --delete v1.0.0   # 删除远程标签
```

---

## 七、高级技巧

### 7.1 储藏（Stash）

```bash
# 暂存当前工作
git stash save "wip: feature"
git stash push -m "message"

# 查看储藏列表
git stash list

# 应用储藏
git stash apply               # 应用最近储藏，保留 stash
git stash pop                 # 应用并删除最近储藏
git stash apply stash@{2}     # 指定应用

# 删除储藏
git stash drop stash@{0}
git stash clear               # 清空所有储藏
```

### 7.2 重置与回退

```bash
# 软重置：保留工作区和暂存区，只移动 HEAD
git reset --soft HEAD~1

# 混合重置（默认）：保留工作区，撤销暂存区
git reset HEAD~1
git reset --mixed HEAD~1

# 硬重置：丢弃所有修改，移动到指定提交
git reset --hard <commit-id>

# 仅重置某个文件到指定版本
git checkout <commit-id> -- <file>
```

### 7.3 挑选提交（Cherry-pick）

```bash
# 将某提交应用到当前分支
git cherry-pick <commit-id>

# 应用一系列提交
git cherry-pick <start-commit>^..<end-commit>

# 带参数
git cherry-pick -n <commit>   # 只应用到工作区，不自动提交
```

### 7.4 子模块（Submodule）

```bash
# 添加子模块
git submodule add <repository-url> <path>

# 克隆包含子模块的仓库
git clone --recursive <url>

# 更新子模块
git submodule update --init --recursive

# 子模块拉取最新
git submodule update --remote
```

### 7.5 清理与维护

```bash
# 清理未跟踪文件
git clean -n          # 预览
git clean -f          # 强制删除
git clean -fd         # 删除目录

# 优化本地仓库
git gc                # 垃圾回收
git fsck              # 完整性检查

# 重新打包
git repack -a -d --depth=250 --window=250
```

---

## 八、工作流程模型

### 8.1 Git Flow

```
master       → 稳定版本
hotfix       → 紧急修复
release      → 发布准备
develop      → 开发主干
feature      → 功能开发
```

适合大型项目，版本发布严格。

### 8.2 GitHub Flow

```
master       → 始终可部署
feature      → 新功能分支，通过 PR 合并
```

简单，适合持续部署。

### 8.3 GitLab Flow

结合环境分支（staging, production）和环境分支。

---

## 九、常用别名与配置

### 9.1 自定义别名

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

### 9.2 忽略文件（.gitignore）

**常见示例：**

```
# 编译输出
*.class
*.o
*.exe

# 依赖目录
node_modules/
vendor/

# IDE
.vscode/
.idea/

# 环境文件
.env
*.log
```

---

## 十、故障排查与恢复

### 10.1 误删分支/提交

```bash
# 使用 reflog 查找丢失的提交
git reflog
# 输出示例：
# e5a4f8c HEAD@{0}: reset: moving to HEAD~1
# 2b3e9a1 HEAD@{1}: commit: add feature

git checkout -b recovered-branch 2b3e9a1
```

### 10.2 修复错误的提交作者

```bash
# 修改最近一次提交作者
git commit --amend --author="New Author <email@example.com>"

# 批量修改历史作者（需要交互式变基）
git rebase -i -p <commit-id>
# 将需要修改的提交标记为 edit，然后：
git commit --amend --author="..." --no-edit
git rebase --continue
```

### 10.3 取消已推送的提交

```bash
# 使用 revert（推荐公共分支）
git revert HEAD~3..HEAD
git push

# 使用 reset + force（仅限私有分支）
git reset --hard HEAD~3
git push --force
```

---

## 十一、命令速查表

| 分类     | 命令                                     | 说明                   |
|:---------|:-----------------------------------------|:----------------------|
| 初始化   | `git init` `git clone`                  | 创建或克隆仓库         |
| 基础操作 | `add` `commit` `rm` `mv`                | 文件操作与提交         |
| 分支     | `branch` `checkout` `merge` `rebase`    | 分支管理               |
| 远程     | `remote` `push` `pull` `fetch`          | 远程协作               |
| 查看     | `status` `log` `diff` `show` `blame`    | 状态与历史             |
| 撤销     | `reset` `restore` `revert` `clean`      | 撤销更改               |
| 高级     | `stash` `cherry-pick` `reflog`          | 复杂操作               |

---

## 总结

Git 的核心在于**本地完整性、轻量分支、灵活的工作流**。掌握以下关键点即可应对日常开发：

1.  **理解工作区、暂存区、仓库的关系**
2.  **熟练基础操作（add/commit/push/pull）**
3.  **掌握分支创建与合并（merge/rebase）**
4.  **学会撤销和恢复（reset/revert/reflog）**
5.  **合理使用远程协作（remote/pull request）**
6.  **配置适合自己的别名和工作流程**

Git 的学习曲线较陡，但日常开发中 80% 的时间只需要 20% 的命令。**多用多查，利用 `git help` 和在线资源**，逐步深入。