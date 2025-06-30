---
title: Linux系统--文件传输共享
date: 2024-12-15 00:00:00
tags: [Linux笔记]
---
#### nfs文件挂载系统

#### SCP远程文件传输

![](https://cdn.nlark.com/yuque/0/2024/png/39116304/1710943904882-4ed66e49-06b3-4305-b8f4-01ecd801d3b4.png)

介绍

步骤

在本地主机上打开命令行终端

使用以下命令执行 SCP 传输：

**scp /path/to/local/file username@remote_host:/path/to/destination**

> /path/to/local/file 是要传输的本地文件的路径。
>
> username 是远程主机的用户名。
>
> remote_host 是远程主机的 IP 地址或域名。
>
> /path/to/destination 是文件在远程主机上的目标路径。
>
> 运行命令后，系统可能会要求你输入远程主机的用户密码，以确认身份。

eg

```plain
scp   C:\redis-6.2.6.tar.gz   root@8.137.104.90:/home
```

输入密码后，SCP 将开始将文件从本地主机传输到远程主机。传输过程中会显示进度信息。

输入密码    (是隐藏的,你输入按回车就完事)

传输完成后，SCP 将在终端中显示成功的消息。

如果需要，可以使用 -P 参数指定 SSH 连接的端口号。此外，可以通过添加 -r 参数来递归传输整个目录

#### MobaXTerm文件传输

#### Vmtools/共享文件夹   (仅限本地VMware 虚拟机)
