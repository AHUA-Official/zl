---
title: Linux- Linux常用命令
date: 2024-12-15 00:00:00
tags: [Linux笔记]
---

新增主线   confluence合并    大概10万字   400篇文档

# linux基本命令
## Linux常用命令解析
+ ls [选项][目录..]：列出当前目录下的文件和目录
    - 选项
        * -a：显示所有文件及目录，包括隐藏文件（.开头的文件），以及.和..
        * -A：和-a基本相同，但不显示.和..,即更干净地显示实际内容
        * -d：只列出目录本身属性而不是其内容，可用于查看某个文件夹是否存在
            + 结合通配符*/使用可以列出当前目录下所有非隐藏子目录的名称
        * -l：以长格式显示文件和目录信息
            + total： 当前目录中文件和目录占用的总块数（block count），通常以512字节为单位。这个数字反映了目录内所有文件和目录的实际磁盘使用情况。  
            + 权限，文件或目录的链接数（目录下所含子目录的个数+2），所有者。大小，创建时间
        * -r：<font style="color:rgb(51, 51, 51);">倒序显示文件和目录。</font>
        * <font style="color:rgb(51, 51, 51);">-t：-t 将按照修改时间排序，最新的文件在最前面。</font>
        * <font style="color:rgb(51, 51, 51);">-F：在列出的文件名称后加一符号；例如可执行档则加 "*", 目录则加 "/"</font>
        * <font style="color:rgb(51, 51, 51);">-R：递递归显示目录中的所有文件和子目录。</font>
    - <font style="color:rgb(51, 51, 51);">注意：</font>
        * <font style="color:rgb(51, 51, 51);">当文件名包含空格、特殊字符或者开始字符为破折号时，可以使用反斜杠（\）进行转义，或者使用引号将文件名括起来。</font>
+ <font style="color:rgb(51, 51, 51);">chmod[-cfvR][--help][--version]mode file：控制用户对文件的权限的命令</font>
    - <font style="color:rgb(51, 51, 51);">文件三级调用权限：</font>
        * <font style="color:rgb(51, 51, 51);">文件所有者owner</font>
        * <font style="color:rgb(51, 51, 51);">用户组group</font>
        * <font style="color:rgb(51, 51, 51);">其他用户other users</font>
    - <font style="color:rgb(51, 51, 51);">cfvR</font>
        * <font style="color:rgb(51, 51, 51);">- c : 若该文件权限确实已经更改，才显示其更改动作</font>
        * <font style="color:rgb(51, 51, 51);">-f : 若该文件权限无法被更改也不要显示错误讯息</font>
        * <font style="color:rgb(51, 51, 51);">-v : 显示权限变更的详细资料</font>
        * <font style="color:rgb(51, 51, 51);">-R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)</font>
    - <font style="color:rgb(51, 51, 51);">rwx：</font>
        * <font style="color:rgb(51, 51, 51);">r：可读</font>
        * <font style="color:rgb(51, 51, 51);">w：可写</font>
        * <font style="color:rgb(51, 51, 51);">x：可执行</font>
    - <font style="color:rgb(51, 51, 51);">前三个字符表示所有者的权限，中间三个字符表示所属组的权限，后三个字符表示其他用户的权限。例如：</font>

```plain
-rw-r--r-- 1 user group 4096 Feb 21 12:00 file.txt
//表示文件名为file.txt的文件，所有者具有读写权限，所属组和其他用户只有读取权限。
```

    - mode：使用数字模式和绝对模式是指定文件权限（二择一）
        * 数字模式：即二进制转十进制（1代表拥有该权限）
        * 绝对模式（8进制模式）
            + `u`：文件所有者（owner）
            + `g`：组（group）
            + `o`：其他用户（others）
            + `a`：所有用户，相当于ugo
            + `+`：添加权限
            + `-`：移除权限
            + `=`：设置确切权限
    - 例

```plain
chmod a+r file.txt    //将file设置为所有人可读
相当于chmod 444 file.txt
chmod -R a+r s*	//将该目录下所有以s开头的文件与目录设置为任何人可读
若用 chmod 4755 filename 可使此程序具有 root 的权限。
```

+ ps[options][--help]：显示当前进程状态，类似windows的任务管理器
    - 选项：
        * -A/-e：列出所有进程
        * -w：<font style="color:rgb(51, 51, 51);">显示加宽可以显示较多的资讯</font>
        * <font style="color:rgb(51, 51, 51);">-u： 显示特定用户的进程信息。它通常后面接一个用户名或用户ID，可以用来筛选出该用户所拥有的所有进程。  </font>
        * <font style="color:rgb(51, 51, 51);">-aux：显示所有包含其他使用者的进程</font>
    - <font style="color:rgb(51, 51, 51);">常见state：</font>
        * <font style="color:rgb(51, 51, 51);">D: 无法中断的休眠状态 (通常 IO 的进程)</font>
        * <font style="color:rgb(51, 51, 51);">R: 正在执行中</font>
        * <font style="color:rgb(51, 51, 51);">S: 静止状态</font>
        * <font style="color:rgb(51, 51, 51);">T: 暂停执行</font>
        * <font style="color:rgb(51, 51, 51);">Z: 不存在但暂时无法消除</font>
        * <font style="color:rgb(51, 51, 51);">W: 没有足够的记忆体分页可分配</font>
        * <font style="color:rgb(51, 51, 51);"><: 高优先序的行程</font>
        * <font style="color:rgb(51, 51, 51);">N: 低优先序的行程</font>
        * <font style="color:rgb(51, 51, 51);">L: 有记忆体分页分配并锁在记忆体内 (实时系统或捱A I/O)</font>
    - <font style="color:rgb(51, 51, 51);"></font>

# ssh
## 简介
    - ssh是linux系统中最常用的远程管理工具。ssh客户端和服务端之间的通信是加密的，而telnet客户端和服务端之间的通信是明文的。
    - ssh使用公钥进行身份验证，默认端口为22
+ 1.1 ssh工作流程

## 修改端口号
    - 编辑（vim）/etc/ssh/sshd_config
    - 查看防火墙状态：systemctl status firewalld
        * 若状态为active（running）运行状态，若为运行状态
            + 则关闭防火墙：`systemctl stop firewall ``setenforce 0`
            + 添加防火墙允许策略：`firewallcmd- --permanent --add-port=2222/tcp ``firewallcmd --reload`
    - 添加自定义端口到服务：`<font style="color:rgb(222, 127, 62);background-color:rgb(29, 31, 33);">semanage port </font><font style="color:rgb(166, 127, 89);">-</font><font style="color:rgb(222, 127, 62);background-color:rgb(29, 31, 33);">a </font><font style="color:rgb(166, 127, 89);">-</font><font style="color:rgb(222, 127, 62);background-color:rgb(29, 31, 33);">t ssh_port_t </font><font style="color:rgb(166, 127, 89);">-</font><font style="color:rgb(222, 127, 62);background-color:rgb(29, 31, 33);">p tcp </font><font style="color:rgb(181, 189, 104);">2222</font>`
    - 修改后重启ssh服务使得更改生效：`systemctl restart sshd`

## 生成ssh密钥对
    - 在.ssh目录下输入`ssh -keygen -t rsa`默认情况下生成rsa密钥
    - 生成的密钥存放在~/.ssh/目录下，id_rsa为私钥，id_rsa.pub为公钥

## ssh免密登录
    - 通过ssh公钥实现：
        * 在客户端生成ssh密钥对
        * 上传公钥到特定服务器：`ssh-copy-id 用户名@远程服务器ip/主机名`
        * 可在远程主机查看是否有收到上传的公钥：.ssh文件中的authorized_keys
        * 重启ssh服务使其生效：systemctl restart sshd
    - 注意：需要设置远程服务器的.ssh目录权限为700，其下authorized_keys文件权限为600，否则无法生效

## 限制特定用户访问ssh
    - 编辑/etc/ssh/sshd_config文件
        * 使用AllowUsers允许特定用户访问
        * 使用DenyUsers拒绝特定用户访问
    - 修改后重启ssh服务使其生效

## 在Linux服务器中方禁止使用root登录
    - 在ssh_config文件中吧PermitRootLogin yes设置为PermitRootLogin no
    - 修改后重启ssh服务使其生效

## 防止ssh暴力破解攻击
    - 修改默认端口
    - 在linux服务器中禁止使用root登录
    - 配置MaxAuthTries限制登录尝试次数
    - 使用Fail2Ban等工具检测和阻止暴力破解攻击

## 使用ssh进行文件传输
    - scp和sftp都是基于ssh的文件传输协议，
        * scp专门用于复制文件和目录，更简单，适合快速的文件复制，单次文件或目录的传输：`scp file.txt 用户名@远程服务器ip/主机名：文件位置`
            + 一次性传输完成，不能进行进程中的文件操作
        * stcp是交互式文件传输工具，提供了更加全面的文件管理功能，在scp的基础上可以进入stcp会话，在会话内实现put，get，cd，ls等功能

```plain
sftp user@remote                # 连接到远程服务器
put report.txt /home/user/documents    # 上传本地文件
get /home/user/data.csv /home/user/downloads  # 下载远程文件
exit                            # 退出 SFTP 会话
```

## 配置ssh隧道
    - ssh隧道简介
        * 通过ssh加密通道传输数据的工具，在本地客户端和远程服务器之间建立安全的隧道。可以绕过防火墙或访问限制，以访问受限的网络资源或实现端口转发等功能
    - 分类
        * 本地端口转发
            + 将本地端口转发到目标服务器上的目标端口。常用于访问内网资源。
            + `ssh -L local_port:remote_address:remote_port user@remote-server`
        * 远程端口转发
            + 将远程服务器的端口转发到本地端口，使远程服务器可以访问本地资源
            +  `ssh -R [远程端口]:[目标地址]:[目标端口] user@remote_server ` 

## ssh登录时显示connection refused
    - ssh服务未启动或奔溃
    - 防火墙阻止连接
    - ssh配置错误，例如绑定了错误的ip地址或端口
    - 网络问题

## 禁用ssh的反向dns解析
    - 反向dns解析：客户端使用ssh连接服务端时，服务器会记下客户端的ip地址，并使用该ip地址进行反向dns查询寻找对应的主机名
        * 进行dns查询可能会增加连接延迟，<font style="color:rgb(77, 77, 77);">特别是在 DNS 解析慢的网络环境中</font>
        * 攻击者可能会伪造dns记录，是安全验证失效
    - 禁用反向dns解析：在sshd_config文件中设置UseDNS为no

##  解决ssh连接慢
    - 关闭反向dns解析
    - 关闭GSSAPIAuthenticatiojn

## ssh连接时出现<font style="color:rgb(79, 79, 79);">Host key verification failed</font>
    - 错误原因：目标服务器的主机密钥发生变化，导致客户端不信任
        * 主机密钥是服务器在ssh启动时生成的密钥对，每台服务器都有自己的主机密钥。是用于向客户端表明自己身份的
    - 解决：
        * 打开~/.ssh/known_hosts删除其中相应主机条目
        * 或者ssh-keygen -R [主机名]

## 查看版本
    - 客户端：ssh -V
    - 服务端：sshd -V

# git
![](https://cdn.nlark.com/yuque/0/2024/png/42613351/1733035317786-1f48da52-848b-4942-b785-a9a3e3f5e25e.png)

## git cherry-pick rebase
+ git merge
    - 将一个分支的更改合并到当前分支

```plain
  git checkout main
  git merge branch-name
```

+ git rebase
    - 通过变基将一个分支的更改整合到当前分支上，而不是创建一个新的合并提交,会更改历史记录，记录更简洁明了

```plain
git checkout branch-name  
git rebase main
```

    - 如果发生合并冲突，解决冲突后使用 git rebase continue
+ git cherry-pick
    - 将某个特点提交应用到当前分支上

## 怎么解决git合并冲突
+ git status查看哪些文件存在冲突
+ 手动解决冲突
+ git add标记冲突已解决
+ git commit/git rebase --continue
+ git push
+ 撤销合并操作：git merge/rebase --abort
+ 









# dhcp
+ 架构：客户端-服务器架构

# Linux（Debian和基于Debian的发行版如Ubuntu）
## 包管理工具apt
+ 更新软件包列表 sudo apt update
+ 安装软件包 sudo apt install 
+ 卸载软件包 sudo apt remove，若需保存配置文件sudo apt purge
+ 查看已经安装的软件包：dpkg -l
+ 查找包 apt search 
+ 查看包信息 apt show
+ 当下载失败
    - 第一步ping常见网址查看是否是因为网络问题
    - 检查防火墙和代理设置
    - 检查软件包是否存在
    - 更新本地缓存软件包源
    - 若源服务器本身存在问题，可更换软件包源
        * 编辑etc（系统配置文件和程序配置文件）/apt/sources.list，选择一个更近的镜像源
        * 修改后更新
    - 清理本地缓存后重新尝试
        * sudo apt clean
    - 检查依赖问题
        * sudo apt --fix-broken install

## 网络配置
### ip地址配置
#### 配置静态ip
+ 编辑etc/network/interfaces文件
    - 静态地址
    - 子网掩码
    - 默认网关
    - dns服务器
+ 保存并退出文件后重启网络服务
    - sudo systemctl restart networking
+ 查看静态ip配置是否生效
    - ip addr show eth0

#### 动态ip配置网络接口（DHCP自动获取ip地址）
+ 编辑etc/network/interfaces文件
+ 配置使用DHCP自动获取ip地址

```plain
auto eth0//接口开机时自启用
iface eth0 inet dhcp
```

#### dhcp-server配置dhcp服务器
+ 安装isc-dhcp-server
+ DHCP服务器工作原理
    - DHCP Discover客户端发起广播请求分配ip地址
    - DHCP Offer：服务端响应并提供可用的ip地址
    - DHCP Request

#### dns配置
+ 通过网络接口文件指定
+ 或在etc/resolv.conf

#### 接口和路由配置ip
+ ip命令概述
    - addr：接口ip地址包括4和6
        * 添加接口地址：ip addr add xxxx dev xxx
    - link接口：名称，状态，MAC地址，MTU等
        * 启用接口：ip link set up
        * 禁用：down
    - route
    - rule
+ ip route命令查看或修改路由表
+ 添加静态路由：ip route add
+ via xxx：下一跳网关地址；dev xxx：经过的接口
+ 删除路由：ip route del
+ 清空路由：ip route flush

### 磁盘相关
#### 磁盘挂载mount
+ 概念：列出当前挂载的文件系统
+ df：查看磁盘空间使用清况，包括挂载点信息
+ lsblk：列出所有块设备，包括挂载点信息
    - 块设备用于数据存储，以块或扇区的方式访问数据
+ blkid：查看磁盘设备的uuid、文件系统类型等
+ 创建并挂载文件系统：mkfs.文件系统类型 挂载位置

