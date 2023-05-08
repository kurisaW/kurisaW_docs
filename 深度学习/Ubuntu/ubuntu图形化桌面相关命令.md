## 安装ubuntu桌面图形化显示
```c
sudo apt-get install ubuntu-desktop
```

## 安装unity桌面系统
```c
sudo apt-get install unity
```

## 安装lightdm显示管理器

```c
sudo apt-get install lightdm
```

## 选择 lightdm或者gdm3都可以
```c
sudo service gdm3 start
sudo service lightdm start
```

## 设置ubuntu开机的默认开启方式为图形化界面显示
```c
sudo systemctl set-default graphical.target
```

## 输入命令startx 启动图形化桌面

```bash
startx
```


