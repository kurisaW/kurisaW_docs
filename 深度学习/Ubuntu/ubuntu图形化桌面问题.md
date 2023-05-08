## 系统设置丢失

### 问题描述

设置点击无响应，或各位置的设置消失。

### 解决方法

执行以下两条命令即可（没有立即生效尝试注销用户或重启）：

```sh
sudo apt-get install unity-control-center
sudo apt-get install gnome-control-center
```

---

## 桌面异常

### 问题描述

桌面无法存入文件，桌面文件夹内的内容不在桌面显示，不能在桌面进行右击创建文件、打开终端等操作。

**猜测该问题是在解决上个问题时产生的**

### 解决方法

以防万一先查看一下 **user-dirs.dirs** 文件。输入 `vim ~/.config/user-dirs.dirs`，把 **XDG_DESKTOP_DIR** 一项改成`"$HOME/Desktop"`（若是中文改为 `"/桌面"`），如下图：

![image-20230507182920684](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305071829739.png)

若该文件没有问题，执行以下两条命令即可（没有立即生效尝试注销用户或重启）：

```sh
sudo apt install gnome-shell-extension-desktop-icons-ng
sudo apt install gnome-shell-extension-prefs
```