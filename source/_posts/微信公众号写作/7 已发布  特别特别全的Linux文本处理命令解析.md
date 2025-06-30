---
title: 特别特别全的Linux文本处理命令解析
date: 2025-04-16 10:00:00
tags: [shell]
categories: [微信公众号]

---



# LinuxZL-文本处理工具（发布名， 特别特别全的Linux文本处理命令解析）
> **摘要**: mkdir、touch、find、cat、tail、head、sed、awk、grep、more、less、vim等命令  提供了不同场景下进行文本处理/日志分析的命令行工具喵
>

---

## 📝 写在前面
大家好！大家好，我是CloudOps一朵小花，今天我们来聊聊 Linux 下的文本处理工具。

---

## 概述
### 日志文件是什么
日志文件是系统、应用程序或服务在运行时生成的记录文件，通常包含时间戳、事件级别（如信息、警告、错误）和事件描述。它们对于系统监控、故障排查和安全审计非常重要。

日志文件通常是文本格式，以 `.txt`、`.log` 或其他文本格式结尾，但也可能采用二进制格式，如 MySQL 的二进制日志（`.bin`）。二进制日志文件通常需要特定的工具或程序来解析和查看。

> 今天我给大家讲的是文本日志喵
>

### 对日志文件的处理要求
在Linux中，我个人感觉处理文本日志文件大概有这样的要求：

+ **查找日志文件**：快速定位日志文件的位置常用命令：`find`、`grep -r`、`locate`
+ **查看日志文件内容**：快速查看日志内容，确认是否存在明显错误或变化常用命令：`cat`、`tail`、`head`、`less`
+ **创建 / 产生日志文件**：手动或自动生成日志文件常用命令：`touch` 创建空文件，`echo` / `printf` / `cat` / `tee` 重定向写入内容
+ **编辑日志文件**：直接修改日志内容，或进行格式化、清洗常用命令：`vim`、`nano`、`sed`、`awk`
+ **删除日志文件**：清理无用日志，释放磁盘空间，或定期归档和清理常用命令：`rm`、`find -exec rm`、`logrotate`（定时清理工具）
+ **分析日志内容**：从大量日志中提取关键字段、错误信息或进行数据统计  
常用命令：`grep`、`awk`、`sed`、`wc`、`diff`、`sort`、`uniq`、`jq`、`yq`、`xargs`、`cut` 等

## 基础文件操作命令
### 基础文件路径命令 | ls、pwd、cd
> **提示**: 这些是最基本的Linux命令，用于了解文件的完整路径和目录操作。
>

`ls`** 命令**: 列出目录内容，显示文件和目录信息

+ **语法**: `ls [选项] [文件或目录...]`
+ **常用参数**: `-a`(显示隐藏文件), `-l`(详细信息), `-h`(人类可读的方式), `-t`(按时间排序), `-R`(递归显示)
+ **示例**: `ls -lh` (显示详细信息并以人类可读格式显示大小)

`pwd`** 命令**: 显示当前工作目录的完整路径

+ **语法**: `pwd [选项]`
+ **常用参数**: `-L`(逻辑路径), `-P`(物理路径)
+ **示例**: `pwd` (显示当前目录), `pwd -P` (显示物理路径)

`cd`** 命令**: 更改当前工作目录

+ **语法**: `cd [选项] [目录]`
+ **特殊路径**: `~`(`home`目录), `-`(上次目录), `..`(上级目录), `.`(当前目录)
+ **示例**: `cd /var/log` (切换到指定目录), `cd ~` (`home目录`), `cd ..` (返回上级目录)

## 查找文件 | 快速定位文件位置
### `find` 命令
> **提示**: 这里我们只关注常见的功能实现，而不是枯燥的手册内容。
>

**功能**: 用于在指定目录下查找文件

**文档链接**:

+ [LinuxCool - find命令详解](https://www.linuxcool.com/find)
+ [Linux man pages - find](https://man7.org/linux/man-pages/man1/find.1.html#top_of_page)

**语法**: `find [起始路径...] [表达式]`

**适用场景**: 知道文件名但不知道完整路径时进行搜索。

**常用示例**:

```bash
# 查找特定目录下所有以 .log 结尾的文件
find /var/log -name "*.log"

# 查找特定时间范围内修改的日志文件（修改时间在1天以内）
find /var/log -name "*.log" -mtime -1

# 在/etc目录中搜索所有大于10MB的文件
find /etc -size +10M

# 在/home目录中搜索所有属于指定用户的文件
find /home -user linuxprobe

# 列出当前工作目录中的所有文件、目录以及子文件信息
find .

# 在/var/log目录下搜索所有不是以.log结尾的文件
find /var/log ! -name "*.log"
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-name` | 匹配文件名 |
| `-type` | 匹配文件类型 |
| `-size` | 匹配文件大小 |
| `-user` | 匹配文件owner |
| `-exec` | 进一步处理搜索结果（`... {} \;` 表示命令的结束） |


其他：`find` 命令的搜索速度相对较慢，我个人建议可以使用 `locate`（普遍没有预装，安装并执行 `updatedb` 建立索引），可以加快查询速度。  
类似的命令还有 `which`（查找命令路径）、`whereis`（查找命令的二进制、源文件、man 手册等）。

### `locate` 命令
> **提示**: 这里我们只关注常见的功能实现，而不是枯燥的手册内容。
>

**功能**: 快速查找文件，基于预建的数据库索引进行搜索

**文档链接**:

+ [LinuxCool - locate命令详解](https://www.linuxcool.com/locate)

**语法**: `locate [选项] 模式`

**适用场景**: 需要快速查找文件，且对实时性要求不高时使用。`locate` 比 `find` 快很多，但可能不会显示最新创建的文件。 locate的默认查找根路径是整个文件系统（从根目录 `/` 开始）。

**常用示例**:

```bash
# 查找所有以 .log 结尾的文件
locate "*.log"

# 指定目录下查找所有以 .log 结尾的文件
locate "/var/log/*.log"

# 查找包含特定关键词的文件
locate "nginx"

# 忽略大小写搜索
locate -i "nginx"

# 限制搜索结果数量
locate -n 10 "*.log"

# 只显示存在的文件
locate -e "*.log"

# 更新数据库索引（需要root权限）
sudo updatedb
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-i` | 忽略大小写 |
| `-n` | 限制显示结果数量 |
| `-e` | 只显示存在的文件 |
| `-c` | 只显示匹配文件的数量 |
| `-q` | 静默模式，不显示错误信息 |
| `-r` | 使用正则表达式 |


**工作原理**: `locate` 使用一个预建的数据库（通常是 `/var/lib/mlocate/mlocate.db`）来存储文件路径信息。数据库通过 `updatedb` 命令定期更新，因此可能不会显示最新创建的文件，同样因为权限关系，数据库可能没有更新一些高权限目录下的文件。

### `which` 命令
> **提示**: 这里我们只关注常见的功能实现，而不是枯燥的手册内容。
>

**功能**: 查找命令的完整路径

**文档链接**:

+ [LinuxCool - which命令详解](https://www.linuxcool.com/which)

**语法**: `which [选项] 命令名...`

**适用场景**: 想知道某个命令的完整路径，或者确认使用的是哪个版本的命令。

**常用示例**:

```bash
# 查找 ls 命令的路径
which ls

# 查找多个命令的路径
which ls cat grep

# 显示所有匹配的路径（如果有多个同名命令）
which -a python

# 静默模式，只返回状态值
which -q ls
echo $?  # 0为找到，1为未找到
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-a` | 显示所有匹配的路径 |
| `-q` | 静默模式，不输出任何内容 |
| `-s` | 静默模式，不显示错误信息 |


**工作原理**: `which` 命令在 `PATH` 环境变量指定的目录中查找可执行文件，返回第一个匹配的完整路径。

### `whereis` 命令
> **提示**: 这里我们只关注常见的功能实现，而不是枯燥的手册内容。
>

**功能**: 查找命令的二进制文件、源文件和man手册的位置

**文档链接**:

+ [LinuxCool - whereis命令详解](https://www.linuxcool.com/whereis)

**语法**: `whereis [选项] 命令名...`

**适用场景**: 需要查找命令的二进制文件、源代码或文档位置时使用。

**常用示例**:

```bash
# 查找 ls 命令的所有相关信息
whereis ls

# 只查找二进制文件
whereis -b ls

# 只查找man手册
whereis -m ls

# 只查找源代码
whereis -s ls

# 查找多个命令
whereis ls cat grep
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-b` | 只查找二进制文件 |
| `-m` | 只查找man手册 |
| `-s` | 只查找源代码 |
| `-u` | 查找不常见的文件类型 |
| `-B` | 指定二进制文件搜索路径 |
| `-M` | 指定man手册搜索路径 |
| `-S` | 指定源代码搜索路径 |


**工作原理**: `whereis` 在预定义的目录列表中搜索指定命令的二进制文件、源代码文件和man手册，比 `which` 提供更全面的信息。

### `grep` 命令
> **提示**: 这里我们只关注常见的功能实现，而不是枯燥的手册内容。
>

**功能**: 用于在文件中搜索包含指定模式的行

**文档链接**:

+ [LinuxCool - grep命令详解](https://www.linuxcool.com/grep)

**语法**: `grep [选项] 模式 [文件...]`

**适用场景**: 知道要搜索的关键词（文件里面的字段或者文件名的一部分）但不知道具体的文件名和具体路径时进行搜索。

**常用示例**:

```bash
# 在日志文件中搜索包含"error"的行
grep "error" /var/log/syslog

# 搜索当前工作目录中包含某个关键词内容的文件
grep -l root *

# 搜索当前工作目录中包含某个关键词内容的文件，未找到也不提示
grep -sl root *

# 不仅搜索指定目录，还搜索其内子目录内是否有关键词文件
grep -srl root /etc

# 判断指定文件中是否包含某个关键词，通过返回状态值输出结果
grep -q linuxprobe anaconda-ks.cfg
echo $?  # 0为包含，1为不包含
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-l` | 只显示包含匹配行的文件名 |
| `-s` | 静默模式，不显示错误信息 |
| `-r` | 递归搜索子目录 |
| `-i` | 忽略大小写 |
| `-v` | 反向匹配，显示不包含模式的行 |
| `-q` | 静默模式，不输出任何内容，只返回状态值 |
| `-n` | 显示行号 |
| `-E` | 使用扩展正则表达式（等同于egrep） |
| `-F` | 使用固定字符串匹配（等同于fgrep） |


其他：`grep` 命令来自英文词组"global search regular expression and print out the line"的缩写，意思是用于全面搜索的正则表达式，并将结果输出。与之容易混淆的是 `egrep` 命令（等价于 `grep -E`，支持扩展正则表达式）和 `fgrep` 命令（等价于 `grep -F`，不支持正则表达式，直接按照字符串内容进行匹配）。

## 创建 / 产生日志文件
### `touch` 命令
**功能**: 创建空文件与修改时间戳

**语法**: `touch [参数] <文件名>`

**工作原理**:

+ 如果文件不存在，则会创建一个空内容的文本文件
+ 如果文件已经存在，则会对文件的Atime（访问时间）和Ctime（修改时间）进行修改操作

**常用示例**:

```bash
# 创建一个新的空白日志文件
touch /var/log/myapp.log

# 创建多个文件
touch file1.txt file2.txt file3.txt

# 修改文件时间戳为当前时间
touch existing_file.txt
```

### `echo` 命令
**功能**: 在终端设备上输出指定字符串或变量提取后的值

**语法**: `echo [参数] 字符串或$变量名`

**工作原理**:

+ 如果文件存在，会覆盖文件内容
+ 如果文件不存在，会创建文件并写入内容
+ 配合 `>>` 可以追加文本到文件末尾

**常用示例**:

```bash
# 创建日志文件并写入内容
echo "This is a log entry" > /var/log/myapp.log

# 追加内容到日志文件
echo "New log entry" >> /var/log/myapp.log

# 输出变量值
echo $PATH

# 执行命令并输出结果
echo `uptime`
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-e` | 使反斜杠（\）具有特殊含义 |
| `-E` | 使反斜杠（\）不具有特殊含义 |
| `-n` | 不输出结尾的换行符 |
| `--help` | 显示帮助信息 |
| `--version` | 显示版本信息 |


**转义序列**:

| 转义序列 | 描述 |
| --- | --- |
| `\a` | 发出警告音 |
| `\b` | 删除前面的一个字符 |
| `\c` | 结尾不加换行符 |
| `\f` | 换行后光标仍停留在原来的位置 |
| `\n` | 换行后光标移至行首 |
| `\r` | 光标移至行首但不换行 |


### `cat` 命令
**功能**: 连接文件并打印到标准输出（concatenate files and print）

**语法**: `cat [参数] [文件...]`

**工作原理**:

+ 适合查看内容较少的纯文本文件
+ 对于大文件会快速滚屏，建议使用 `more` 或 `less` 命令
+ 可以结合重定向操作符创建或合并文件

**常用示例**:

```bash
# 合并多个文件到一个日志文件
cat file1.txt file2.txt > /var/log/combined.log

# 查看文件内容
cat /var/log/myapp.log

# 查看文件内容并显示行号
cat -n /var/log/myapp.log

# 清空文件内容
cat /dev/null > /var/log/myapp.log

# 交互式写入文件内容
cat > /var/log/myapp.log << EOF
Hello, World
Linux!
EOF
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n` | 显示行号 |
| `-A` | 显示所有字符（包括不可见字符） |
| `-b` | 只对非空行显示行号 |
| `-s` | 当遇到连续两行以上的空白行，就替换为一行空白行 |
| `-T` | 将制表符显示为^I |
| `-v` | 显示不可见字符 |


### `tree` 命令
**功能**: 以树状结构显示目录内容

**语法**: `tree [参数] [目录...]`

**工作原理**:

+ 递归显示目录结构，以树状图形展示文件和目录的层次关系
+ 默认显示当前目录，可以指定其他目录路径
+ 支持多种输出格式和过滤选项

**常用示例**:

```bash
# 显示当前目录的树状结构
tree

# 显示指定目录的树状结构
tree /var/log

# 只显示目录，不显示文件
tree -d /var/log

# 限制显示深度
tree -L 2 /var/log

# 忽略特定文件类型
tree -I "*.log" /var/log

# 显示文件大小
tree -h /var/log

# 显示文件权限信息
tree -p /var/log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-d` | 只显示目录 |
| `-L N` | 限制显示深度为N层 |
| `-I 模式` | 忽略匹配模式的文件 |
| `-h` | 显示文件大小 |
| `-p` | 显示文件权限 |
| `-a` | 显示隐藏文件 |
| `-f` | 显示完整路径 |
| `-t` | 按修改时间排序 |
| `-r` | 反向排序 |
| `-q` | 静默模式，不显示错误信息 |


## 查看日志文件
在Linux系统中，查看文本日志文件通常涉及以下几个步骤：

### `cat` 命令
**功能**: 显示整个日志文件的内容

**适用场景**: 适用于比较小行数的日志，全部显示便于定位和分析

**语法**: `cat [参数] 文件...`

**常用示例**:

```bash
# 查看整个日志文件
cat /var/log/myapp.log

# 查看文件内容并显示行号
cat -n /var/log/myapp.log

# 查看多个日志文件
cat /var/log/app1.log /var/log/app2.log

# 显示不可见字符
cat -A /var/log/myapp.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n` | 显示行号 |
| `-A` | 显示所有字符（包括不可见字符） |
| `-b` | 只对非空行显示行号 |
| `-s` | 当遇到连续两行以上的空白行，就替换为一行空白行 |
| `-T` | 将制表符显示为^I |
| `-v` | 显示不可见字符 |


### `tail` 命令
**功能**: 查看日志文件的最后几行，默认显示最后10行

**适用场景**: 适配运行中系统进行debug时，触发bug后查看最后的回显来寻找问题 一个运行中系统  进行debug的时候  触发bug然后看最后的回显来寻找bug是啥

**语法**: `tail [参数] 文件...`

**常用示例**:

```bash
# 查看文件最后10行
tail /var/log/myapp.log

# 查看文件最后20行
tail -n 20 /var/log/myapp.log

# 实时跟踪日志文件的更新
tail -f /var/log/myapp.log

# 从第100行开始显示到文件末尾
tail -n +100 /var/log/myapp.log

# 实时跟踪多个日志文件
tail -f /var/log/app1.log /var/log/app2.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n N` | 显示最后N行 |
| `-f` | 实时跟踪文件变化 |
| `-F` | 类似-f，但文件被删除后重新创建时仍跟踪 |
| `-q` | 不显示文件名 |
| `-s N` | 与-f合用，每N秒检查一次文件变化 |


### `head` 命令
**功能**: 查看日志文件的开始几行，默认显示前10行

**适用场景**: 查看系统刚开始启动就有问题时，分析启动失败的原因，或者想要知道环境变量和启动时参数的

**语法**: `head [参数] 文件...`

**常用示例**:

```bash
# 查看文件前10行
head /var/log/myapp.log

# 查看文件前20行
head -n 20 /var/log/myapp.log

# 查看文件前100个字节
head -c 100 /var/log/myapp.log

# 查看多个文件的前几行
head -n 5 /var/log/app1.log /var/log/app2.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n N` | 显示前N行 |
| `-c N` | 显示前N个字节 |
| `-q` | 不显示文件名 |
| `-v` | 总是显示文件名 |


### `grep` 命令
**功能**: 搜索包含特定文本的行

**适用场景**: 在不是很大的日志文件中定位bug地址（不超过一万行）

**语法**: `grep [参数] 模式 文件...`

**常用示例**:

```bash
# 搜索包含"error"的行
grep "error" /var/log/myapp.log

# 使用管道搜索 这个最常用
cat /var/log/myapp.log | grep "error"

# 忽略大小写搜索
grep -i "ERROR" /var/log/myapp.log

# 显示匹配行的行号
grep -n "error" /var/log/myapp.log

# 显示匹配行的上下文
grep -A 2 -B 2 "error" /var/log/myapp.log

# 反向搜索（显示不包含关键词的行）
grep -v "debug" /var/log/myapp.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-i` | 忽略大小写 |
| `-n` | 显示行号 |
| `-v` | 反向匹配 |
| `-A N` | 显示匹配行及其后N行 |
| `-B N` | 显示匹配行及其前N行 |
| `-C N` | 显示匹配行及其前后N行 |
| `-E` | 使用扩展正则表达式 |
| `-F` | 使用固定字符串匹配 |


### `more` 命令
**功能**: 分页查看日志文件，适用于查看大型文件

**适用场景**: 当终端打印不能处理行数太多的文件时使用（终端会卡死），但大部分时候有条件的话还是更喜欢vim

**语法**: `more [参数] 文件...`

**常用示例**:

```bash
# 分页查看文件
more /var/log/myapp.log

# 从第100行开始查看
more +100 /var/log/myapp.log

# 设置每页显示行数
more -10 /var/log/myapp.log
```

**操作说明**:

+ 空格键：下一页
+ 回车键：下一行
+ b键：上一页
+ q键：退出
+ /模式：搜索模式

### `less` 命令
**功能**: 分页查看日志文件，支持前后翻页

**适用场景**: 分页查看日志文件，适用于查看大型文件。less不是"更少"的意思，而是"more is less"的含义

**语法**: `less [参数] 文件...`

**常用示例**:

```bash
# 分页查看文件
less /var/log/myapp.log

# 从第100行开始查看
less +100 /var/log/myapp.log

# 实时跟踪文件变化
less +F /var/log/myapp.log

# 显示行号
less -N /var/log/myapp.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-N` | 显示行号 |
| `-F` | 如果文件小于一屏，自动退出 |
| `-R` | 显示ANSI颜色转义序列 |
| `-S` | 截断长行 |
| `+N` | 从第N行开始显示 |


**操作说明**:

+ 空格键：下一页
+ b键：上一页
+ 上下箭头：上下滚动
+ /模式：向前搜索
+ ?模式：向后搜索
+ n键：下一个匹配
+ N键：上一个匹配
+ q键：退出

### `watch` 命令
**功能**: 定期执行命令并显示输出，用于监控系统状态变化

**适用场景**: 实时监控系统资源、进程状态、日志文件变化等，特别适合调试和系统监控

**语法**: `watch [参数] 命令`

**常用示例**:

```bash
# 监控系统进程变化
watch -n 1 ps aux

# 监控磁盘使用情况
watch -n 2 df -h

# 监控内存使用情况
watch -n 1 free -h

# 监控日志文件大小变化
watch -n 1 "ls -lh /var/log/myapp.log"

# 监控特定进程
watch -n 1 "ps aux | grep nginx"

# 监控网络连接
watch -n 1 netstat -tuln

# 监控系统负载
watch -n 1 uptime

# 监控日志文件最后几行
watch -n 1 "tail -5 /var/log/myapp.log"
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n N` | 每N秒执行一次命令（默认2秒） |
| `-d` | 高亮显示变化的输出 |
| `-t` | 不显示标题栏 |
| `-b` | 当命令输出改变时发出蜂鸣声 |
| `-e` | 当命令执行失败时退出 |
| `-g` | 当命令输出改变时退出 |
| `-c` | 不显示颜色 |
| `-x` | 将命令传递给exec |


## 分析日志内容
### `sed` 命令
**功能**: 过滤日志文件内容，使用正则表达式进行内容过滤

**适用场景**: sed的使用一般比较复杂，适合进行文本处理和转换

**语法**: `sed [参数] '命令' 文件...`

**常用示例**:

```bash
# 删除空行
sed '/^$/d' /var/log/myapp.log

# 显示第10-20行
sed -n '10,20p' /var/log/myapp.log

# 替换文本
sed 's/old_text/new_text/g' /var/log/myapp.log

# 删除包含特定模式的行
sed '/error/d' /var/log/myapp.log

# 在匹配行前添加行号
sed '=' /var/log/myapp.log | sed 'N;s/\n/ /'
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n` | 不自动打印模式空间 |
| `-e` | 执行脚本 |
| `-f` | 从文件读取脚本 |
| `-i` | 直接修改文件 |
| `-r` | 使用扩展正则表达式 |


### `awk` 命令
**功能**: 处理日志文件内容，打印每行的第N个字段

**适用场景**: awk的使用一般比较复杂，适合在复杂的脚本中进行文件处理，多使用正则表达式

**语法**: `awk [参数] '程序' 文件...`

**常用示例**:

```bash
# 打印每行的最后一个字段
awk '{print $NF}' /var/log/myapp.log

# 打印第1和第3个字段
awk '{print $1, $3}' /var/log/myapp.log

# 只处理包含"error"的行
awk '/error/ {print $0}' /var/log/myapp.log

# 按字段分隔符处理
awk -F: '{print $1}' /etc/passwd

# 统计行数
awk 'END {print NR}' /var/log/myapp.log

# 条件处理
awk '$3 > 100 {print $0}' /var/log/myapp.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-F` | 指定字段分隔符 |
| `-v` | 设置变量 |
| `-f` | 从文件读取程序 |


### `vim` 命令
**功能**: 强大的文本编辑工具

**适用场景**: 需要编辑日志文件时使用

**语法**: `vim [参数] 文件...`

**常用示例**:

```bash
# 编辑文件
vim /var/log/myapp.log

# 以只读模式打开
vim -R /var/log/myapp.log

# 显示行号
vim -c "set number" /var/log/myapp.log
```

**基本操作**:

+ `i`：进入插入模式
+ `Esc`：退出插入模式
+ `:w`：保存文件
+ `:q`：退出vim
+ `:wq`：保存并退出
+ `:q!`：强制退出不保存
+ `/模式`：搜索模式
+ `n`：下一个匹配
+ `N`：上一个匹配

### `jq` 命令
**功能**: 轻量级命令行JSON处理器

**适用场景**: 处理JSON格式的日志文件、API响应数据、配置文件等

**语法**: `jq [参数] '过滤器' [文件...]`

**常用示例**:

```bash
# 格式化JSON输出
jq '.' log.json

# 提取特定字段
jq '.error' log.json

# 提取多个字段
jq '.timestamp, .level, .message' log.json

# 过滤包含error的日志
jq 'select(.level == "ERROR")' log.json

# 统计不同级别的日志数量
jq -r '.level' log.json | sort | uniq -c

# 提取数组中的元素
jq '.items[0].name' data.json

# 条件过滤
jq 'select(.response_time > 1000)' api_log.json
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-r` | 输出原始字符串（不包含引号） |
| `-c` | 紧凑输出格式 |
| `-s` | 将输入作为数组处理 |
| `-M` | 单色输出 |
| `-S` | 对对象键进行排序 |


### `yq` 命令
**功能**: 轻量级命令行YAML处理器

**适用场景**: 处理YAML格式的配置文件、Kubernetes清单、Docker Compose文件等

**语法**: `yq [参数] '表达式' [文件...]`

**常用示例**:

```bash
# 读取YAML文件
yq eval '.version' docker-compose.yml

# 提取服务名称
yq eval '.services | keys' docker-compose.yml

# 修改配置值
yq eval '.version = "3.8"' docker-compose.yml -i

# 添加新配置
yq eval '.services.nginx.ports += ["8080:80"]' docker-compose.yml -i

# 过滤特定服务
yq eval '.services | select(has("nginx"))' docker-compose.yml

# 提取环境变量
yq eval '.services.app.environment' docker-compose.yml
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `eval` | 执行表达式 |
| `-i` | 直接修改文件 |
| `-P` | 输出为YAML格式 |
| `-j` | 输出为JSON格式 |
| `-x` | 输出为XML格式 |


### `xargs` 命令
**功能**: 从标准输入构建和执行命令

**适用场景**: 批量处理文件、并行执行命令、处理find命令的输出等

**语法**: `xargs [参数] [命令]`

**常用示例**:

```bash
# 批量删除文件
find /tmp -name "*.tmp" | xargs rm

# 并行处理文件
find . -name "*.log" | xargs -P 4 -I {} gzip {}

# 批量重命名文件
ls *.txt | xargs -I {} mv {} {}.backup

# 统计多个文件的行数
find . -name "*.log" | xargs wc -l

# 搜索多个文件中的关键词
find . -name "*.conf" | xargs grep "error"

# 批量压缩文件
find . -name "*.log" -mtime +7 | xargs gzip
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n N` | 每次传递N个参数 |
| `-P N` | 并行执行，最多N个进程 |
| `-I 替换符` | 指定替换字符串 |
| `-0` | 以null字符分隔输入 |
| `-t` | 在执行前打印命令 |
| `-p` | 交互式确认 |


### `cut` 命令
**功能**: 从文件的每一行中提取指定字段

**适用场景**: 提取日志文件中的特定列、处理CSV文件、分析结构化数据等

**语法**: `cut [参数] [文件...]`

**常用示例**:

```bash
# 提取第1列（以空格分隔）
cut -d' ' -f1 log.txt

# 提取第1和第3列
cut -d' ' -f1,3 log.txt

# 提取以冒号分隔的第1列
cut -d: -f1 /etc/passwd

# 提取字符位置1-10
cut -c1-10 log.txt

# 提取第1列和第5列
cut -d',' -f1,5 data.csv

# 提取除第2列外的所有列
cut -d' ' -f1,3- log.txt

# 提取时间戳列
cut -d' ' -f1-3 log.txt
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-d 分隔符` | 指定字段分隔符 |
| `-f 字段列表` | 指定要提取的字段 |
| `-c 字符列表` | 指定要提取的字符位置 |
| `-s` | 不输出不包含分隔符的行 |
| `--complement` | 输出未指定的字段 |


### `sort` 命令
**功能**: 对文本文件的行进行排序

**适用场景**: 对日志文件按时间、级别、用户等字段排序，数据分析和统计

**语法**: `sort [参数] [文件...]`

**常用示例**:

```bash
# 按字母顺序排序
sort log.txt

# 按数字排序
sort -n numbers.txt

# 按第2列排序
sort -k2 log.txt

# 按第3列数字排序
sort -k3 -n log.txt

# 反向排序
sort -r log.txt

# 忽略大小写排序
sort -f log.txt

# 去重并排序
sort -u log.txt

# 按多个字段排序
sort -k1,1 -k3,3n log.txt
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-n` | 按数字排序 |
| `-r` | 反向排序 |
| `-f` | 忽略大小写 |
| `-u` | 去除重复行 |
| `-k 字段` | 按指定字段排序 |
| `-t 分隔符` | 指定字段分隔符 |


### `uniq` 命令
**功能**: 去除重复行或统计重复行

**适用场景**: 统计日志中重复的错误信息、分析访问模式、数据去重等

**语法**: `uniq [参数] [输入文件] [输出文件]`

**常用示例**:

```bash
# 去除连续重复行
uniq log.txt

# 统计重复行数量
uniq -c log.txt

# 只显示重复行
uniq -d log.txt

# 只显示不重复行
uniq -u log.txt

# 忽略前N个字符
uniq -s 10 log.txt

# 忽略大小写
uniq -i log.txt

# 结合sort使用
sort log.txt | uniq -c | sort -nr
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-c` | 在每行前显示重复次数 |
| `-d` | 只显示重复行 |
| `-u` | 只显示不重复行 |
| `-i` | 忽略大小写 |
| `-s N` | 忽略前N个字符 |
| `-w N` | 只比较前N个字符 |


## 现代日志处理和分析技术栈
像 `cut`、`sort`、`uniq` 这些传统命令行工具，在面对格式规整、体量不大的本地日志时，仍然非常高效：字段提取、排序、去重都能快速完成，写法也简单，适合临时排查或脚本自动化处理。

但时代变了。随着微服务、容器、云原生的广泛应用，日志体量呈指数级增长，格式也从纯文本向结构化 JSON 演进。再用 `cut -d' '` 这种按空格切分字段的方式早就不够用了——字段位置不固定、嵌套结构复杂，处理起来既繁琐又低效。

更关键的问题是 **量**。现在系统每小时产出的日志轻松以百万计，`sort` 一个大文件不仅慢，甚至可能把机器拖垮。而这些传统工具大多是 **离线批处理** 思路，不具备实时流处理能力。一旦系统出问题，靠它们根本做不到即时响应。

此外，可视化和响应速度也是问题。命令行输出对技术人员尚可，但在团队协作中显得力不从心。像 Kibana、Grafana 这类现代平台，能够将日志数据转成图表、仪表盘，趋势、分布、异常一目了然，大大提升问题定位效率。实际上，使用命令行工具排查问题时，大部分时间都花在思考如何提取相关日志上，速度不够快。且它们缺乏告警机制，无法实现自动化处理和实时响应，出现异常时只能依赖人工通知运维，效率低下，难以及时应对突发状况。

所以整体来看，传统命令行工具仍有价值，特别适用于发生了问题之后现场单机快速处理，但它们已经难以应对当前这种高并发、大规模、结构复杂、要求实时可视化的日志处理场景。要真正做好日志分析，必须构建一套完整的日志系统。

---

### ELK Stack（Elasticsearch + Logstash + Kibana）
+ **功能**：企业级日志采集、存储、搜索和分析平台
+ **适用场景**：大规模分布式系统中的日志统一管理、实时监控和故障排查
+ **组件说明**：
    - `Elasticsearch`：高性能搜索与索引引擎
    - `Logstash`：强大的日志采集与处理框架
    - `Kibana`：交互式数据可视化平台

---

### Fluentd（EFK）
+ **功能**：轻量级数据收集器，支持丰富插件
+ **适用场景**：容器化环境中的日志采集与转发，常与 Elasticsearch 和 Kibana 搭配构成 EFK 方案

---

### 其他现代日志工具
| 工具名称 | 功能简述 | 适用场景 |
| --- | --- | --- |
| **Loki** | Grafana 推出的日志聚合系统，配合 Promtail 使用 | 云原生、轻量级日志方案 |
| **Splunk** | 强大的企业级日志分析平台 | 安全审计、合规性分析、大数据日志处理 |
| **Datadog** | 云原生监控平台，日志与 APM 一体化 | 全栈可观测性 |


---

虽然在微服务和可观测性体系中，链路追踪（Tracing）和指标（Metrics）同样重要，Peter Bourgon 在他那篇广为流传的 _Metrics, Tracing, and Logging_ 中也系统性地讨论了三者的区别与联系，但今天我们暂且不展开这些内容。**本文聚焦的是"日志（Logging）"这一部分，在当前主流方案中，ELK Stack 和 Fluentd（EFK）仍然是最具代表性的技术栈。**

## 写入日志文件
### `echo` 命令追加文本
**功能**: 在脚本中添加配置或关键词到日志文件

**语法**: `echo "内容" >> 文件路径`

**常用示例**:

```bash
# 追加文本到日志文件
echo "New log entry" >> /path/to/logfile.log

# 追加带时间戳的日志
echo "$(date): New log entry" >> /path/to/logfile.log

# 追加变量内容
echo "User: $USER logged in at $(date)" >> /path/to/logfile.log

# 追加多行内容
echo -e "Line 1\nLine 2\nLine 3" >> /path/to/logfile.log
```

### `printf` 命令追加文本
**功能**: 格式化输出并追加到日志文件

**语法**: `printf "格式字符串" 参数... >> 文件路径`

**常用示例**:

```bash
# 格式化追加日志
printf "New log entry at %s\n" "$(date)" >> /path/to/logfile.log

# 格式化输出多个字段
printf "%-20s %-10s %s\n" "$(date)" "INFO" "Application started" >> /path/to/logfile.log

# 追加带编号的日志
printf "[%04d] %s: %s\n" "$(wc -l < /path/to/logfile.log)" "$(date)" "Log message" >> /path/to/logfile.log
```

### `tee` 命令
**功能**: 同时在控制台和日志文件中显示输出

**适用场景**: 常用于debug调试时，既可以在终端上实时打印输出查看运行状态，也可以等运行完成后观察日志输出

**语法**: `命令 | tee [参数] 文件路径`

**常用示例**:

```bash
# 同时输出到终端和文件
echo "New log entry" | tee -a /path/to/logfile.log

# 执行命令并记录输出
ls -la | tee -a /path/to/logfile.log

# 记录命令执行结果
./myapp.sh 2>&1 | tee -a /path/to/logfile.log

# 只记录错误输出
./myapp.sh 2>&1 | tee -a /path/to/error.log

# 同时记录到多个文件
echo "Log message" | tee -a file1.log file2.log file3.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-a` | 追加到文件（默认覆盖） |
| `-i` | 忽略中断信号 |
| `--help` | 显示帮助信息 |


## 编辑日志文件
### `vim` 命令
**功能**: 强大的文本编辑工具

**适用场景**: 需要编辑日志文件时使用

**语法**: `vim [参数] 文件路径`

**常用示例**:

```bash
# 编辑日志文件
vim /path/to/logfile.log

# 以只读模式打开
vim -R /path/to/logfile.log

# 显示行号
vim -c "set number" /path/to/logfile.log

# 跳转到特定行
vim +100 /path/to/logfile.log

# 搜索特定模式
vim -c "/error" /path/to/logfile.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-R` | 只读模式 |
| `+N` | 从第N行开始 |
| `-c 命令` | 执行vim命令 |
| `-o` | 水平分割窗口 |
| `-O` | 垂直分割窗口 |


**基本操作**:

+ `i`：进入插入模式
+ `Esc`：退出插入模式
+ `:w`：保存文件
+ `:q`：退出vim
+ `:wq`：保存并退出
+ `:q!`：强制退出不保存
+ `/模式`：搜索模式
+ `n`：下一个匹配
+ `N`：上一个匹配

### `nano` 命令
**功能**: 大部分Linux系统自带的简单文本编辑器

**适用场景**: 需要快速编辑文件时使用，比vim更容易上手

**语法**: `nano [参数] 文件路径`

**常用示例**:

```bash
# 编辑日志文件
nano /path/to/logfile.log

# 显示行号
nano -l /path/to/logfile.log

# 自动备份
nano -B /path/to/logfile.log

# 从特定行开始
nano +100 /path/to/logfile.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-l` | 显示行号 |
| `-B` | 自动备份 |
| `-T N` | 设置制表符宽度 |
| `-w` | 不自动换行 |
| `+N` | 从第N行开始 |


**基本操作**:

+ `Ctrl+O`：保存文件
+ `Ctrl+X`：退出nano
+ `Ctrl+W`：搜索
+ `Ctrl+G`：显示帮助
+ `Ctrl+K`：剪切当前行
+ `Ctrl+U`：粘贴

### `sed` 命令
**功能**: 流编辑器，用于文本处理和转换

**适用场景**: 复杂的脚本中进行文件处理，多使用正则表达式

**语法**: `sed [参数] '命令' 文件路径`

**常用示例**:

```bash
# 替换文本
sed -i 's/old_text/new_text/g' /path/to/logfile.log

# 删除包含特定模式的行
sed -i '/error/d' /path/to/logfile.log

# 在特定行前插入文本
sed -i '10i\New line content' /path/to/logfile.log

# 删除空行
sed -i '/^$/d' /path/to/logfile.log

# 只保留包含特定关键词的行
sed -i '/important/!d' /path/to/logfile.log

# 在文件开头添加时间戳
sed -i '1i\# Generated at: $(date)' /path/to/logfile.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-i` | 直接修改文件 |
| `-n` | 不自动打印模式空间 |
| `-e` | 执行脚本 |
| `-f` | 从文件读取脚本 |
| `-r` | 使用扩展正则表达式 |


### `awk` 命令
**功能**: 强大的文本处理工具

**适用场景**: 复杂的脚本中进行文件处理，多使用正则表达式

**语法**: `awk [参数] '程序' 文件路径`

**常用示例**:

```bash
# 提取特定字段
awk '{print $1}' /path/to/logfile.log > newfile.log

# 按条件过滤行
awk '$3 > 100 {print $0}' /path/to/logfile.log

# 统计特定字段的总和
awk '{sum += $3} END {print sum}' /path/to/logfile.log

# 格式化输出
awk '{printf "%-20s %-10s %s\n", $1, $2, $3}' /path/to/logfile.log

# 按字段分组统计
awk '{count[$1]++} END {for(i in count) print i, count[i]}' /path/to/logfile.log

# 处理CSV格式日志
awk -F',' '{print $1, $3}' /path/to/logfile.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-F` | 指定字段分隔符 |
| `-v` | 设置变量 |
| `-f` | 从文件读取程序 |
| `-W` | 设置警告级别 |


**内置变量**:

+ `$0`：整行内容
+ `$1, $2, ...`：第1、2、...个字段
+ `NF`：字段数量
+ `NR`：当前行号
+ `FS`：字段分隔符
+ `OFS`：输出字段分隔符

## 删除日志文件
### `rm` 命令
**功能**: 删除文件和目录

**语法**: `rm [参数] 文件或目录...`

**工作原理**:

+ 直接删除指定的文件或目录
+ 使用 `-r` 参数可以递归删除目录及其内容
+ 使用 `-f` 参数强制删除，不提示确认

**常用示例**:

```bash
# 删除单个日志文件
rm /var/log/myapp.log

# 删除多个日志文件
rm /var/log/app1.log /var/log/app2.log

# 递归删除整个日志目录
rm -rf /var/log/myapp/

# 强制删除，不提示确认
rm -f /var/log/myapp.log

# 删除所有.log文件
rm *.log
```

**常用参数**:

| 参数 | 描述 |
| --- | --- |
| `-r` 或 `-R` | 递归删除目录及其内容 |
| `-f` | 强制删除，不提示确认 |
| `-i` | 交互式删除，删除前提示确认 |
| `-v` | 显示删除过程 |
| `--help` | 显示帮助信息 |


**注意事项**:

+ `rm -rf` 命令非常危险，使用前请确认路径正确
+ 建议先用 `ls` 命令查看要删除的文件
+ 重要文件建议先备份再删除

### `find` 命令结合 `-exec` 选项
**功能**: 条件删除文件，根据特定条件批量删除

**语法**: `find 路径 [表达式] -exec 命令 {} \;`

**工作原理**:

+ 先查找符合条件的文件
+ 然后对每个找到的文件执行删除操作
+ 常用于清理旧日志文件，释放磁盘空间

**常用示例**:

```bash
# 删除7天前的所有.log文件
find /var/log -name "*.log" -mtime +7 -exec rm {} \;

# 删除30天前的特定应用日志
find /var/log -name "myapp-*.log" -mtime +30 -exec rm {} \;

# 删除大于100MB的日志文件
find /var/log -name "*.log" -size +100M -exec rm {} \;

# 删除空日志文件
find /var/log -name "*.log" -empty -exec rm {} \;

# 删除特定用户的日志文件
find /var/log -user www-data -name "*.log" -exec rm {} \;
```

**常用条件参数**:

| 参数 | 描述 |
| --- | --- |
| `-name "*.log"` | 按文件名模式匹配 |
| `-mtime +7` | 修改时间超过7天 |
| `-size +100M` | 文件大小超过100MB |
| `-empty` | 空文件 |
| `-user username` | 指定用户的文件 |
| `-group groupname` | 指定组的文件 |


**安全建议**:

+ 删除前先用 `find` 命令查看会删除哪些文件
+ 使用 `-exec rm -i {} \;` 进行交互式删除
+ 重要操作前先备份数据

### `logrotate` 工具
**功能**: 系统日志轮转工具，自动管理日志文件

**工作原理**:

+ 定期压缩、归档和删除旧日志文件
+ 防止日志文件过大占用磁盘空间
+ 通过配置文件 `/etc/logrotate.conf` 和 `/etc/logrotate.d/` 目录管理

**常用示例**:

```bash
# 手动执行日志轮转
logrotate /etc/logrotate.conf

# 强制执行轮转（即使不满足时间条件）
logrotate -f /etc/logrotate.conf

# 查看轮转配置
cat /etc/logrotate.conf

# 查看特定应用的轮转配置
cat /etc/logrotate.d/nginx
```

**配置文件示例**:

```bash
# /etc/logrotate.d/myapp
/var/log/myapp.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
    postrotate
        /bin/kill -HUP `cat /var/run/myapp.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```

**常用配置选项**:

| 选项 | 描述 |
| --- | --- |
| `daily` | 每天轮转 |
| `weekly` | 每周轮转 |
| `monthly` | 每月轮转 |
| `rotate N` | 保留N个备份文件 |
| `compress` | 压缩旧日志文件 |
| `missingok` | 如果日志文件不存在，不报错 |
| `notifempty` | 如果日志文件为空，不轮转 |


---

## 🎯 总结与思考
通过这篇文章，我们系统地梳理了 Linux 下的文本处理工具，从基础的 `cat`、`tail` 到强大的 `awk`、`sed`，再到现代的 ELK Stack 和 Fluentd。

**参考资料：**

+ [Linux man pages](https://man7.org/linux/man-pages/)
+ [LinuxCool 命令大全](https://www.linuxcool.com/)
+ [Fluentd 官方文档](https://docs.fluentd.org/)

**谢谢你看我的文章**

