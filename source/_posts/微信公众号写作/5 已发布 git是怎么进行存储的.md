---
title: git是怎么进行存储的
date: 2025-04-16 10:00:00
tags: [git]
categories: [微信公众号]

---

一朵小花，大家每天工作都需要使用git， 但是git到底是怎么存储文件的， 有些同学可能还不知道喵，今天我来给大家简单介绍一下下。
>

git 依赖./git 这个目录完成他的功能

###### ./git 目录有啥
```plain
$ ls -1 .git/
branches
COMMIT_EDITMSG
config
description
FETCH_HEAD
HEAD
hooks
index
info
logs
objects
ORIG_HEAD
packed-refs
refs
```

- [x] branches ： 旧版本git使用的目录，现在基本不用，可以忽略
- [x] COMMIT_EDITMSG - 最后一次提交的提交信息
- [x] config - Git配置信息：用户名、email、远程仓库地址、分支跟踪关系等
- [x] <font style="color:black;">Description 该git库的描述信息，如果使用了GitWeb的话，该描述信息将会被显示在该repo的页面上</font>
- [x] HEAD： 工作目录当前状态对应的commit，一般来说是当前branch的head，HEAD也可以通过git checkout 命令被直接设置到一个特定的commit上，这种情况被称之为 detached HEAD
- [x] FETCH_HEAD - 记录最近一次fetch操作获取的远程分支信息
- [x] hooks - Git钩子脚本目录，可以在特定事件时自动执行脚本
- [x] index - 暂存区文件，记录准备提交的文件状态
- [x] info - Git仓库的额外信息目录
- [x] objects     保存git对象的目录，包括三类对象commit,tag, tree和blob
- [x] 打包的引用文件，优化性能
- [x] Refs： 保存branch和tag对应的commit

###### git 的对象
<font style="color:black;">commit </font><font style="color:black;">对象存储 </font><font style="color:black;">git </font><font style="color:black;">中的提交信息</font><font style="color:black;">;</font>

<font style="color:black;">tree </font><font style="color:black;">对象存储 </font><font style="color:black;">git </font><font style="color:black;">仓库中的文件元数据信息</font><font style="color:black;">, </font><font style="color:black;">包括文件名及目录结构信息等</font><font style="color:black;">;</font>

<font style="color:black;">blob 则对应的是 git 仓库中的文件内容;</font>





###### <font style="color:black;">git add 的时候发生了什么 </font>
```plain
$ mkdir test
$ cd test
$ git init
Initialized empty Git repository in /home/user/test/.git/
$ ls
$ ls -la .git/objects/
total 16
drwxrwxr-x 4 user user 4096 6月   3 16:37 .
drwxrwxr-x 7 user user 4096 6月   3 16:37 ..
drwxrwxr-x 2 user user 4096 6月   3 16:37 info
drwxrwxr-x 2 user user 4096 6月   3 16:37 pack
$ echo '111' > a.txt
$ echo '222' > b.txt
$ ls -la .git/objects/
total 16
drwxrwxr-x 4 user user 4096 6月   3 16:37 .
drwxrwxr-x 7 user user 4096 6月   3 16:37 ..
drwxrwxr-x 2 user user 4096 6月   3 16:37 info
drwxrwxr-x 2 user user 4096 6月   3 16:37 pack
```



```plain
$ git add *.txt
$ ls -la .git/objects/
total 24
drwxrwxr-x 6 user user 4096 6月   3 16:37 .
drwxrwxr-x 7 user user 4096 6月   3 16:37 ..
drwxrwxr-x 2 user user 4096 6月   3 16:37 58
drwxrwxr-x 2 user user 4096 6月   3 16:37 c2
drwxrwxr-x 2 user user 4096 6月   3 16:37 info
drwxrwxr-x 2 user user 4096 6月   3 16:37 pack
$ find .git/objects -type f
.git/objects/c2/00906efd24ec5e783bee7f23b5d7c941b0c12c
.git/objects/58/c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
```

做的事情，Git 会对每个被 add 的文件内容生成一个 blob 对象

使用 SHA-1 算法对文件生成 40 位 sha 码 ，同时从第二位截断， 前两位当作路径，后 38 位做文件名。

我们可以用 sha 码来进行验证 （前几位就行，git 会自动补全）这个地方创建了 2 个 blob 对象。 

```plain
$ git cat-file -p c200
222
$ git cat-file -p 58c9
111
```

blob 对象和具体文件的映射关系是存在 index 文件夹下面 ，使用命令` git ls-files --stage` 可以查看

```plain
$ git ls-files --stage
100644 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c 0       a.txt
100644 c200906efd24ec5e783bee7f23b5d7c941b0c12c 0       b.txt
```



###### <font style="color:black;">git commit 做了什么 </font>
现在的` .git/objects/  `

```plain
$ tree .git/objects/
.git/objects/
├── 58
│   └── c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
├── c2
│   └── 00906efd24ec5e783bee7f23b5d7c941b0c12c
├── info
└── pack

4 directories, 2 files
```

执行 commit

```plain
$ git commit -m "first commit"
[master (root-commit) 9dd98ff] first commit
 2 files changed, 2 insertions(+)
 create mode 100644 a.txt
 create mode 100644 b.txt
$ tree .git/objects/
.git/objects/
├── 4c
│   └── aaa1a9ae0b274fba9e3675f9ef071616e5b209
├── 58
│   └── c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
├── 9d
│   └── d98ff2246a7b76ac7124d27d528771e2089357
├── c2
│   └── 00906efd24ec5e783bee7f23b5d7c941b0c12c
├── info
└── pack

6 directories, 4 files
```

这个时候发现多了两个文件， 使用`git cat-file ` 来查看文件内容和类型

```plain
$ git cat-file -t  4caaa
tree
$ git cat-file -p  4caaa
100644 blob 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c    a.txt
100644 blob c200906efd24ec5e783bee7f23b5d7c941b0c12c    b.txt
$ git  cat-file  -t  9dd98
commit
$ git  cat-file  -p  9dd98
tree 4caaa1a9ae0b274fba9e3675f9ef071616e5b209
author     User <user@example.com>           1748940927 +0800
committer  User <user@example.com> 1748940927 +0800

first commit
```

做的事情， 每次提交会读取  index 的内容 ，根据这个 index 内容 

生成 tree 对象， tree 对象包含当前的文件结构还有文件对应的 blob 哈希

然后生成 commit 对象， commit 对象包含指向的 tree 对象， author   committer    commit-message

以及   `.git/HEAD` 所指的分支指向该 commit  

结构可以简单的看成是这个样子的

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1748941787440-851793ac-7fa1-4e8d-9f35-822fad2998ec.png)



###### git 更新 commit 会发生什么？ 
```plain
$ echo "333" > a.txt
$ cat a.txt 
333
$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   a.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ tree .git/objects/
.git/objects/
├── 4c
│   └── aaa1a9ae0b274fba9e3675f9ef071616e5b209
├── 58
│   └── c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
├── 9d
│   └── d98ff2246a7b76ac7124d27d528771e2089357
├── c2
│   └── 00906efd24ec5e783bee7f23b5d7c941b0c12c
├── info
└── pack

6 directories, 4 files
```



```plain
$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   a.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add a.txt
$ tree .git/objects/
.git/objects/
├── 4c
│   └── aaa1a9ae0b274fba9e3675f9ef071616e5b209
├── 55
│   └── bd0ac4c42e46cd751eb7405e12a35e61425550
├── 58
│   └── c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
├── 9d
│   └── d98ff2246a7b76ac7124d27d528771e2089357
├── c2
│   └── 00906efd24ec5e783bee7f23b5d7c941b0c12c
├── info
└── pack

7 directories, 5 files
$ git cat-file -t 55bd0
blob
$ git cat-file -p 55bd0
333
$ git ls-files --stage
100644 55bd0ac4c42e46cd751eb7405e12a35e61425550 0       a.txt
100644 c200906efd24ec5e783bee7f23b5d7c941b0c12c 0       b.txt
```

提交 commit



```plain
$ git commit -m "2 commit"
[master 769e63a] 2 commit
 1 file changed, 1 insertion(+), 1 deletion(-)
$ tree .git/objects/
.git/objects/
├── 0f
│   └── d247c919b0faa824e03cbef3b4b375d804e481
├── 4c
│   └── aaa1a9ae0b274fba9e3675f9ef071616e5b209
├── 55
│   └── bd0ac4c42e46cd751eb7405e12a35e61425550
├── 58
│   └── c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
├── 76
│   └── 9e63aa42d80721f248229cf9523ebc57845159
├── 9d
│   └── d98ff2246a7b76ac7124d27d528771e2089357
├── c2
│   └── 00906efd24ec5e783bee7f23b5d7c941b0c12c
├── info
└── pack

9 directories, 7 files
$ git cat-file -t 0fd247
tree
$ git cat-file -p 0fd247
100644 blob 55bd0ac4c42e46cd751eb7405e12a35e61425550    a.txt
100644 blob c200906efd24ec5e783bee7f23b5d7c941b0c12c    b.txt
$ git cat-file -t 769e6
commit
$ git cat-file -p 769e6
tree 0fd247c919b0faa824e03cbef3b4b375d804e481
parent 9dd98ff2246a7b76ac7124d27d528771e2089357
author User <user@example.com> 1748942193 +0800
committer User <user@example.com> 1748942193 +0800

2 commit
```

做的事情： 

add   改变的文件产生新的 blob 对象， 更新新的索引 

commit       

1 根据暂存区 index 的内容（即最新的 blob 哈希），生成新的 **tree 对象 **

2 创建一个新的 **commit 对象**，commit 里包含：

+ **指向新 tree 的哈希**
+ **指向上一个 commit 的 parent 哈希（即旧 commit）**
+ **author 和 committer 信息**
+ **commit message**
1. 更新分支引用（HEAD）

变动的图

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1748942921857-ffc0b0a7-a9a2-472c-80b4-a0a56f7199d6.png)







## 总结
通过以上分析，我们可以看到 Git 的存储差不多是这样的。 

1. Git 使用对象存储（blob、tree、commit）来保存文件内容和版本信息
2. 通过 SHA-1 哈希值来唯一标识每个对象
3. 采用引用（refs）来管理分支和标签
4. 通过 parent 指针构建提交历史

这种设计使得 Git 具有以下优势：

+ 高效存储：相同内容只存储一次
+ 完整性保证：通过 SHA-1 确保数据不被篡改
+ 快速访问：通过引用可以快速定位到特定版本
+ 灵活的分支管理：分支本质上只是指向某个提交的指针



> 谢谢大家看我的文章。
>

