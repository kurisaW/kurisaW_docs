## 环境：

* VMware Workstation 16 pro

* ubuntu 20.04

## 过程

在确认虚拟机设置正常的情况下，查看ubuntu /mnt/hgfs/文件夹下没有共享文件夹

则输入命令：

```c
vmhgfs-fuse .host:/ /mnt/hgfs/ -o allow_other -o uid=1000
```

再查看文件夹：

共享文件夹正常显示。

以上，调整命令可以改变共享文件夹在ubuntu的入口。

在 /目录下 mkdir work新建一个文件夹

输入命令：

```c
vmhgfs-fuse .host:/ /work/ -o allow_other -o uid=1000
```

可以在/work/文件夹下查看共享文件夹。