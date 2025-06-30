---
title:LinuxZL-网络管理
date: 2024-12-15 00:00:00
tags: [Linux笔记]
---
![](https://cdn.nlark.com/yuque/0/2024/png/39116304/1718256013972-17ffc2bf-d749-4291-9855-2cb7352437a0.png)

+ **add/change/replace**：添加、更改或替换网络接口上的地址。
+ **del**：删除网络接口上的地址。
+ **save/flush**：保存或清除网络接口上的地址配置。
+ **show/showdump/restore**：显示当前的地址配置、以二进制格式显示或从二进制格式恢复地址配置。

### 示例命令

+ ip address add 192.168.1.1/24 dev eth0：在eth0接口上添加IP地址192.168.1.1/24。
+ ip address delete 192.168.1.1/24 dev eth0：从eth0接口删除IP地址192.168.1.1/24。
