# Ubuntu下百度Linux输入法安装方法

各系统存在差异，安装步骤会稍有不同，以ubuntu1804为例：

## 1、安装中文支持包

打开`setting－> Region & Lannguage -> InputSource下的Manage installation Language`

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305072135491.jpg)

点击`Installation/ Remove Language`

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305072136717.png)

勾选中文（简体），并点击应用

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305072136471.png)

keyboard input method system选择：fcitx

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305072136754.png)

点击应用到全局，然后重启

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305072136427.png)

 ## 2、通过命令行更新系统（如果因网络连接问题无法更新，可以选择换源）

```c
sudo apt-get update

sudo apt-get upgrade
```

## 3、通过命令行安装aptitude 

```c
sudo apt-get install aptitude
```

## 4、通过命令行利用aptitude 安装fcitx、qt

```c
sudo aptitude install fcitx-bin fcitx-table fcitx-config-gtk fcitx-config-gtk2 fcitx-frontend-all

sudo aptitude install qt5-default qtcreator qml-module-qtquick-controls2
```

## 5、如果右上角没有出现fcitx，在终端中输入im-config进行配置

 ## 6、通过命令行安装百度输入法

```c
sudo dpkg –i fcitx-baidupinyin.deb
```

## 7、重启完成即可使用（如果没有，需在右上角fcitx配置中添加百度输入法）

卸载命令：

```c
sudo dpkg --purge remove fcitx-baidupinyin:amd64
```

