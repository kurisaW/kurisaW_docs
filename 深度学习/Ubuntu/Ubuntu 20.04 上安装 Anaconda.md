> 原文来自：[如何在 Ubuntu 20.04 上安装 Anaconda](https://cloud.tencent.com/developer/beta/article/1649008)

## 一、 安装 Anaconda

在写这篇文章的时候，Anaconda 最新绑定版本是 2020.02。在下载安装脚本之前，浏览[下载页面](https://www.anaconda.com/products/individual)，并且检查是否有更新的Anaconda 可用。

在 Ubuntu 20.04 上完成下面的步骤，安装 Anaconda。

## 1.Anaconda Navigator 是一个基于 QT 的 GUI。如果你在一个桌面版机器上安装 Anaconda，并且你想使用 GUI 应用，安装下面的软件包。否则，跳过下面的步骤。

```javascript
sudo apt install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6
```

## 2.使用你的浏览器或者`wget`去下载 Anaconda 安装脚本：

```javascript
wget -P /tmp https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
```

下载将会花费一些时间，具体依赖于你的网速。

## 3.这一步可选的，但是我们推荐你去验证脚本的数据完整性。

使用`sha256sum`命令显示脚本的 checksum:

```javascript
sha256sum /tmp/Anaconda3-2020.02-Linux-x86_64.sh
```

输出类似于：

```javascript
2b9f088b2022edb474915d9f69a803d6449d5fdb4c303041f60ac4aefcc208bb  /tmp/Anaconda3-2020.02-Linux-x86_64.sh
```

确保上面的命令打印出来的哈希值和[Anaconda with Python 3 on 64-bit Linux page](https://docs.anaconda.com/anaconda/install/hashes/lin-3-64/)页面对应版本的 Anaconda 哈希值一样。

```javascript
https://docs.anaconda.com/anaconda/install/hashes/Anaconda3-2020.02-Linux-x86_64.sh-hash/
```

## 4.运行脚本启动安装进程：

```javascript
bash /tmp/Anaconda3-2020.02-Linux-x86_64.sh
```

你应该能看到下面的输出：

```javascript
Welcome to Anaconda3 2020.02

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
>>> 
```

按`ENTER`继续。往下滑动阅读协议，使用`ENTER`按键。一旦你看完协议，你将会被询问是否接受协议条款：

```javascript
Do you approve the license terms? [yes|no]
```

输入`yes`接受协议，并且你会被提示选择安装路径：

```javascript
Anaconda3 will now be installed into this location:
/home/linuxize/anaconda3

    - Press ENTER to confirm the location
    - Press CTRL-C to abort the installation
    - Or specify a different location below
```

默认的位置应该对大部分用户都可直接使用。按`Enter`确认位置。

安装过程将会花费一些时间，并且一旦完成，脚本将会问你是否想要运行`conda init`。输入`yes`。

```javascript
Installation finished.
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
```

这将会将命令行工具`conda`添加到系统的`PATH`环境变量中。

想要激活 Anaconda，你可以关闭并且重新打开你的 shell 或者在当前 shell 会话中输入下面的命令，来重新加载`PATH`环境变量：

```javascript
source ~/.bashrc
```

想要验证安装过程，在你的终端输入`conda`。

就这些。你已经成功地在你的 Ubuntu 机器上安装好了 Anaconda， 你可以开始使用它了。

如果你在一个桌面系统上安装 Anaconda，在终端输入`anaconda-navigator`打开Navigator GUI：

## 二、升级 Anaconda

升级 Anaconda 是一个非常直接的过程。打开你的终端，并且输入：

```javascript
conda update --all
```

如果有更新，`conda`将会显示一个列表，并且提示你确认是否更新：

```javascript
The following packages will be UPDATED:

  anaconda-navigator                          1.9.12-py37_0 --> 1.9.12-py37_1
  conda                                        4.8.2-py37_0 --> 4.8.3-py37_0
  conda-package-han~                   1.6.0-py37h7b6447c_0 --> 1.6.1-py37h7b6447c_0


Proceed ([y]/n)? 
```

定期升级 Anaconda 是一个不错的想法。

## 三、卸载 Anaconda

如果你想从你的 Ubuntu 系统中卸载 Anaconda，移除 Anaconda 安装目录以及其他在安装过程中创建的文件：

```javascript
rm -rf ~/anaconda3 ~/.condarc ~/.conda ~/.continuum
```

打开`~/.bashrc`，并且从环境变量中移除 Anaconda：

```javascript
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/linuxize/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/linuxize/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/linuxize/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/linuxize/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```