---
title: Git使用指南
date: 2023-11-15
tags: [Git, 版本控制]
categories: [微信公众号]
---
## 1. 简介

Git是目前世界上最先进的分布式版本控制系统。

## 2. Git是什么

Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何规模的项目。

## 3. 常见命令基础

* `git init`：初始化一个Git仓库
  * 初始化后会产生.git目录，默认是隐藏的，用 `ls -ah`命令可以查看
* `git status`：查看文件的状态
  * 查看某个文件的状态：`git status <文件名>`
  * 查看当前路径所有文件的状态：`git status`
  * 显示哪些文件被修改过，哪些文件已经暂存待提交，哪些文件未被Git追踪
* `git diff`：查看文件的修改内容
  * 显示的格式是Unix通用的diff格式
  * 用于查看文件具体修改了什么内容
* `git add`：将工作区的文件保存到暂存区
  * 保存某个文件到暂存区：`git add <文件名>`
  * 保存当前路径的所有文件到暂存区：`git add .`（注意，最后是一个点 . ）
* `git commit`：将暂存区的文件提交到当前分支
  * 提交某个文件到分支：`git commit -m "<注释>" <文件名>`
  * 保存当前路径的所有文件到分支：`git commit -m "<注释>"`
* `git log`：查看文件的修改日志
  * 查看某个文件的修改日志：`git log <文件名>`
  * 查看当前路径所有文件的修改日志：`git log`
  * 用一行的方式查看简单的日志信息：`git log --pretty=oneline`
  * 查看最近的N次修改：`git log -N`（N是一个整数）
* `git clone`：下载远程仓库到本地
  * 下载远程仓库到当前路径：`git clone <仓库的URL>`
  * 下载远程仓库到特定路径：`git clone <仓库的URL> <存放仓库的路径>`
* `git pull`：下载远程仓库的最新信息到本地仓库
* `git push`：将本地的仓库信息推送到远程仓库

## 4. 时光机穿梭

### 4.1 版本回退

#### 查看提交历史

使用 `git log`命令可以查看提交历史，获取需要回退的提交SHA码：

```bash
git log
```

如果输出信息太多，可以使用 `--pretty=oneline`参数简化显示：

```bash
git log --pretty=oneline
```

#### 版本表示方式

Git中使用以下方式表示版本：

- `HEAD`：表示当前版本（最新提交）
- `HEAD^`：表示上一个版本
- `HEAD^^`：表示上上个版本
- `HEAD~100`：表示往上100个版本

#### 版本回退命令

使用 `git reset`命令进行版本回退：

```bash
git reset --hard HEAD^      # 回退到上一个版本
git reset --hard 1094adb    # 回退到指定commit id的版本
```

回退参数说明：

- `--hard`：回退到上个版本的已提交状态
- `--soft`：回退到上个版本的未提交状态
- `--mixed`：回退到上个版本已添加但未提交的状态

#### 查看命令历史

如果回退后想再回到未来版本，可以使用 `git reflog`查看所有操作历史：

```bash
git reflog
```

输出示例：

```
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```

通过reflog可以找到需要"前进"到的commit id，然后使用 `git reset --hard <提交ID>`回到该版本。

### 4.2 工作区和暂存区

工作区（Working Directory）是你电脑中的目录，如果该目录已经被初始化为Git仓库，则这个目录中除了 `.git`目录外的所有内容都属于工作区。

暂存区（Stage或Index）是Git仓库中的一个区域，用于临时存放你已经修改但还未提交的文件。它位于 `.git`目录下，是一个叫做index的文件。

Git的工作流程主要涉及三个区域：

1. 工作区（你直接编辑的地方）
2. 暂存区（临时保存你的更改）
3. 版本库（最终确认的历史版本）

### 4.3 撤销修改

`git checkout -- <file>`可以丢弃工作区的修改：

`git checkout -- readme.txt`意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

- 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
- 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

命令 `git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区。

### 4.4 文件管理

这部分内容主要涉及文件的添加、修改和查看状态等操作，可以参考3.常见命令基础部分。

### 4.5 删除文件

删除文件的方法：

1. 使用 `git rm <file>`命令删除文件，然后执行 `git commit -m "<注释>"`提交更改：

```bash
git rm test.txt
git commit -m "remove test.txt"
```

2. 手动删除文件，然后使用 `git add <file>`和 `git commit`：

```bash
rm test.txt
git add test.txt
git commit -m "remove test.txt"
```

如果删除错了，可以从版本库恢复：

```bash
git checkout -- test.txt
```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以"一键还原"。

## 5. 远程仓库

### 创建新仓库并上传

```bash
# 创建新的远程仓库
echo "# 项目名" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/用户名/仓库名.git
git push -u origin main
```

### 推送现有仓库

```bash
git remote add origin https://github.com/用户名/仓库名.git
git branch -M main
git push -u origin main
```

## 6. 分支管理

### 6.1 创建与合并分支

- 查看分支：`git branch`
- 创建分支：`git branch <name>`
- 切换分支：`git checkout <name>`或者 `git switch <name>`
- 创建+切换分支：`git checkout -b <name>`或者 `git switch -c <name>`
- 合并某分支到当前分支：`git merge <name>`
- 删除分支：`git branch -d <name>`

### 6.2 解决冲突

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用 `git log --graph`命令可以看到分支合并图。

### 6.3 分支管理策略

合并分支时，加上 `--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而 `fast forward`合并就看不出来曾经做过合并。

```bash
git merge --no-ff -m "merge with no-ff" dev
```

分支策略：

- `master`分支应该是非常稳定的，仅用来发布新版本，平时不能在上面干活
- 日常开发在 `dev`分支上，`dev`分支是不稳定的
- 到某个时候（如1.0版本发布时），将 `dev`分支合并到 `master`上，在 `master`分支发布版本
- 每个人都有自己的分支，时不时地往 `dev`分支上合并

### 6.4 Bug分支

使用 `stash`功能，可以把当前工作现场"储藏"起来，等以后恢复现场后继续工作：

```bash
git stash
```

查看储藏的工作现场：

```bash
git stash list
```

恢复工作现场：

```bash
git stash apply  # 恢复后，stash内容不删除
git stash drop   # 删除stash内容
```

或者：

```bash
git stash pop    # 恢复的同时把stash内容也删了
```

复制特定提交到当前分支：

```bash
git cherry-pick <commit-id>
```

### 6.5 Feature分支

开发一个新功能，最好新建一个分支。

如果要丢弃一个没有被合并过的分支，可以通过 `git branch -D <name>`强行删除。

### 6.6 多人协作

多人协作的工作模式通常是这样：

1. 首先，可以尝试用 `git push origin <branch-name>`推送自己的修改
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用 `git pull`试图合并
3. 如果合并有冲突，则解决冲突，并在本地提交
4. 没有冲突或者解决掉冲突后，再用 `git push origin <branch-name>`推送就能成功！

如果 `git pull`提示 `no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令：

```bash
git branch --set-upstream-to=origin/<branch-name> <branch-name>
```

### 6.7 Rebase

- rebase操作可以把本地未push的分叉提交历史整理成直线
- rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比

```bash
git rebase
```

## 7. 自定义Git

### 7.1 忽略特殊文件 .gitignore

创建 `.gitignore`文件，将需要忽略的文件名填入即可。

常见的需要忽略的文件类型：

- 操作系统自动生成的文件，如缩略图
- 编译生成的中间文件、可执行文件等
- 带有敏感信息的配置文件，如包含口令的配置文件

可以在[GitHub的gitignore模板](https://github.com/github/gitignore)中下载对应语言的模板。


## 8. Git工具总结

Git常用工具：

- GUI客户端：SourceTree, GitHub Desktop, GitKraken等
- IDE集成：VS Code, IntelliJ IDEA等都有优秀的Git集成功能
- 命令行增强：oh-my-zsh的git插件等

## 常见工作流

#### 提交推送更改

```
git add <文件名>
git commit -m "<注释>"
git push 
```

#### 切换分支

```
#存在分支切换
git branch 
git checkout <分支名>
#不存在分支切换
git checkout -b <分支名>
```

#### git 解bug

```
git lab 上面提一个issue   那这个issue创建一个bug 分支

git checkout -b bug-123

git 解决bug后推送 

gitlab 发起PR 
CR通过  合并进主分支

```

#### git 开发新功能

```
git 创建一个feature 分支    

git checkout -b feature-123 

开发新功能后 推送

gitlab 发起PR 
CR通过  合并进主分支

```

#### git 合并分支

```
git merge <分支名>
```

#### git 整合分支

```
合并方法：保留合并分支的所有更改和历史记录。多次合并后，修订历史记录可能会变得复杂。
变基方法：维护一个干净的修订历史记录，因为合并的提交会附加在目标分支的末尾。与合并的方法相比，冲突可能更频繁地发生。
git rebase <分支名>
```


#### git submodule

```
git submodule add <仓库地址> <路径>
```


