一个linux   下面实际有多个盘    怎么找到并查看其他盘的文件





<font style="color:rgb(44, 44, 54);">Linux 系统中，多个物理硬盘或分区可以被挂载到不同的目录下，从而让它们看起来像是系统的一部分。要找到并查看其他盘的文件，你需要按照以下步骤进行：</font>

1. **<font style="color:rgb(44, 44, 54);">列出所有可用的磁盘和分区</font>**<font style="color:rgb(44, 44, 54);">: 使用 </font><font style="color:rgb(44, 44, 54);">lsblk</font><font style="color:rgb(44, 44, 54);"> 命令可以列出所有已知的磁盘和分区信息，包括它们的状态和挂载点：</font>

```plain
Bash
1lsblk
```

2. **<font style="color:rgb(44, 44, 54);">查看分区详情</font>**<font style="color:rgb(44, 44, 54);">: 使用 </font><font style="color:rgb(44, 44, 54);">fdisk -l</font><font style="color:rgb(44, 44, 54);"> 命令可以查看所有磁盘的分区表详情：</font>

```plain
Bash
1fdisk -l
```

3. **<font style="color:rgb(44, 44, 54);">识别未挂载的磁盘或分区</font>**<font style="color:rgb(44, 44, 54);">: 从上述命令的输出中，你可以看到哪些磁盘或分区尚未被挂载。未挂载的磁盘或分区通常在 </font><font style="color:rgb(44, 44, 54);">lsblk</font><font style="color:rgb(44, 44, 54);"> 的输出中没有挂载点（</font><font style="color:rgb(44, 44, 54);">MOUNTPOINT</font><font style="color:rgb(44, 44, 54);"> 列为空）。</font>
4. **<font style="color:rgb(44, 44, 54);">挂载磁盘或分区</font>**<font style="color:rgb(44, 44, 54);">: 要访问一个未挂载的磁盘或分区，首先你需要创建一个挂载点目录，然后使用 </font><font style="color:rgb(44, 44, 54);">mount</font><font style="color:rgb(44, 44, 54);"> 命令将其挂载。例如，假设你想要挂载 </font><font style="color:rgb(44, 44, 54);">/dev/sdb1</font><font style="color:rgb(44, 44, 54);"> 分区，你可以这样做：</font>

```plain
Bash
1sudo mkdir /mnt/mydisk
2sudo mount /dev/sdb1 /mnt/mydisk
```

5. **<font style="color:rgb(44, 44, 54);">访问文件</font>**<font style="color:rgb(44, 44, 54);">: 一旦磁盘或分区被挂载，你就可以像访问任何其他目录一样访问挂载点。例如，你可以使用 </font><font style="color:rgb(44, 44, 54);">ls</font><font style="color:rgb(44, 44, 54);"> 命令查看 </font><font style="color:rgb(44, 44, 54);">/mnt/mydisk</font><font style="color:rgb(44, 44, 54);"> 下的文件：</font>

```plain
Bash
1ls /mnt/mydisk
```

6. **<font style="color:rgb(44, 44, 54);">卸载磁盘或分区</font>**<font style="color:rgb(44, 44, 54);">: 当你完成访问后，记得使用 </font><font style="color:rgb(44, 44, 54);">umount</font><font style="color:rgb(44, 44, 54);"> 命令卸载分区，以避免潜在的数据损坏：</font>

```plain
Bash
1sudo umount /mnt/mydisk
```

7. **<font style="color:rgb(44, 44, 54);">检查挂载状态</font>**<font style="color:rgb(44, 44, 54);">: 使用 </font><font style="color:rgb(44, 44, 54);">df -h</font><font style="color:rgb(44, 44, 54);"> 命令可以查看所有已挂载的文件系统及其使用情况：</font>

```plain
Bash
1df -h
```

<font style="color:rgb(44, 44, 54);">请确保在挂载和卸载分区时具有足够的权限，通常需要使用 sudo 或以 root 用户身份执行这些操作。此外，对于正在被系统使用的分区，直接挂载可能会导致错误，因此确保在尝试挂载前，该分区没有被任何进程使用。</font>

<font style="color:rgb(44, 44, 54);"></font>

```plain
mkfs.ext4 /dev/sda2
```

1. **<font style="color:rgb(44, 44, 54);">创建挂载点</font>**<font style="color:rgb(44, 44, 54);">：在系统中创建一个目录作为新的挂载点。例如：</font>

```plain
mkdir /mnt/new_disk
```

2. **<font style="color:rgb(44, 44, 54);">挂载磁盘</font>**<font style="color:rgb(44, 44, 54);">：将</font><font style="color:rgb(44, 44, 54);">sda2</font><font style="color:rgb(44, 44, 54);">挂载到新创建的目录上：</font>

```plain
mount /dev/sda /mnt/new_disk
```

![](https://cdn.nlark.com/yuque/0/2024/png/39116304/1720682025047-a24cd3b7-5556-41f4-be45-ecbdf1a45c2e.png)

