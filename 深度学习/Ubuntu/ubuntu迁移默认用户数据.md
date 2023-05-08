## 	ubuntu迁移默认用户下的所有文件及环境到另外一个用户下

```c
sudo apt-get install rsync
```

```c
sudo rsync -av /home/default_user /home/new_user
```

## `rsync`基本用法

```c
rsync 参数 来源档案 目的档案
```

- `-v`：verbose 模式，输出比较详细的讯息。
- `-r`：递回（recursive）备份所有子目录下的目录与档案。
- `-a`：封装备份模式，相当于`-rlptgoD`，递回备份所有子目录下的目录与档案，保留连结档、档案的拥有者、群组、权限以及时间戳记。
- `-z`：启用压缩。
- `-h`：将数字以比较容易阅读的格式输出。

*  显示传输进度：如果要让 `rsync` 在传输档案时可以即时显示进度，可以加上 `--progress` 参数：

```
rsync -avzh  --progress 来源档案 目的档案
```

## 更改用户权限

在完成 rsync 命令后，您需要确保新用户对转移的文件和文件夹具有适当的权限。您可以使用以下命令更改新用户的主目录的所有者和组：

```
sudo chown -R new_user:new_user /home/new_user/
```

最后，您可以删除默认用户。在删除用户之前，您应该确保默认用户没有在运行任何程序或服务，并且没有正在使用的文件。

## 删除用户

```
sudo userdel -r old_user
```

其中，-r 选项表示将默认用户的主目录一起删除。
