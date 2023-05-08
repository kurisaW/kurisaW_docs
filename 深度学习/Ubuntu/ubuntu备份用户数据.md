在 Ubuntu 系统中备份用户数据通常包括两个步骤：

1. 备份用户的 Home 目录
2. 备份用户相关的配置文件和数据

具体操作步骤如下：

1. 备份用户的 Home 目录

Home 目录是 Ubuntu 系统中存储用户数据的主要目录，包括桌面、文档、音乐、图片等文件夹。备份 Home 目录可以保证用户的数据能够被恢复。

通常可以使用 tar 命令来打包 Home 目录，例如：

```
sudo tar -cvpzf /backup/user_home.tar.gz /home/user
```

其中，/backup/user_home.tar.gz 是备份文件的路径和名称，/home/user 是需要备份的用户 Home 目录。

2. 备份用户相关的配置文件和数据

用户的配置文件和数据可能存储在多个目录中，具体需要备份的文件和目录视情况而定。一些常见的配置文件和数据目录如下：

- ~/.config：存储用户的配置文件，例如浏览器、终端、桌面环境等的配置文件。
- ~/.ssh：存储用户的 SSH 密钥和配置文件。
- ~/.gnupg：存储用户的 GPG 密钥和配置文件。
- ~/.local/share：存储用户的应用程序数据，例如 Firefox 和 Chromium 浏览器的数据。
- /etc/skel：存储新创建用户时默认的配置文件和数据。

可以使用 tar 命令打包需要备份的文件和目录，例如：

```
sudo tar -cvpzf /backup/user_config.tar.gz ~/.config ~/.ssh ~/.gnupg ~/.local/share /etc/skel
```

其中，/backup/user_config.tar.gz 是备份文件的路径和名称，后面跟着需要备份的目录列表。

备份完成后，可以将备份文件保存到本地磁盘或者远程服务器上。

需要注意的是，备份用户数据时需要使用管理员权限，否则可能会由于权限问题导致备份不完整或者无法备份某些目录和文件。