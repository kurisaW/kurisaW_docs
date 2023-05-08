> 原文地址：[如何在ubuntu20.04上安装pytorch GPU版本](https://zhuanlan.zhihu.com/p/535712148)

## 预备工作

首先建议安装anaconda ubuntu版，这个网上有很多教程，安装也较简单，在这里不再赘述。

## 安装NVIDIA驱动及cuda

### 安装nvidia驱动

首先打开ubuntu的software\&updates，选择其中的additional drivers ，你会看到系统默认的是选择最后一个，那个是个开源的软件，使用它你将无法使用NVIDIA的GPU加速来训练你的模型

![](https://pic2.zhimg.com/v2-e5494dad2678026f986e072b5fcdca69_b.jpg)

一般来说选择第一个就行了，如果你对该选择那个不了解的话，建议不要乱选，选择默认的第一个。选完之后点击apply changes 之后输入你的密码，系统就会使用apt-get下载NVIDIA驱动，下载完成后，**reboot**,如果成功的话可以在seetings中的about里的graphics看到你的NVIDIA驱动信息，这一步完成后就可以进行下一步安装cuda了。

### 安装cuda

pytorch使用cuda来加速计算，尤其是autograd。由于ubuntu官方库里有cuda所以我们可以很轻松的使用apt-get 安装它，只需要执行下述命令：（**建议先更换阿里源在执行**）

```c
sudo apt install nvidia-cuda-toolkit
```

**这个命令将会下载1.5GB文件以及将近4GB的额外包，所以需要等待一定时间，在这里提一下建议在这里换一下ubuntu的软件源，把它换成阿里源，**操作步骤很简单，依旧是打开software\&updates选择ubuntu softwares

![](https://pic1.zhimg.com/v2-33e8213c7326b4178e0a7a350a876ef8_b.jpg)

可以看到download from那一栏是可选的，我这里因为已经选好了所以已经是阿里源了，点击小三角选择来自中国的服务器，建议不要测试以选择最优服务器，太费时间了，我用阿里源速度方面没有遇到过问题。

运行完毕后执行：

```c
nvcc -V
```

命令若出现：

![](https://pic3.zhimg.com/v2-993492b74761c72f9bf64409550004be_b.jpg)

说明安装成功，**要记住你的cuda版本号，接下来选pytorch gpu版本的时候要选择与cuda版本相对应的pytorch版本。**

## 安装pytorch GPU版本

注意执行这一步需要你先安装好anaconda（**默认你已经使用了anaconda建立了一个虚拟环境，并已经进入到了这个虚拟环境中，不建议在base环境中安装**），完成后执行：

```c
conda install pytorch torchvision cudatoolkit=10.1 -c pytorch
```

**注意这里的cuda版本要和你之前安装的相对应，建议自己去**

* [https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/locally/](https://link.zhihu.com/?target=https%3A//pytorch.org/get-started/locally/)

上找合适的版本安装

![](https://pic3.zhimg.com/v2-1b1f71141edcceee625085a29e02f9ba_b.jpg)

如果在首页找不到合适的cuda版本转去previous pytorch versions上面找。接下来只要等待安装完成即可，**当然如果你想要加速下载速度，建议去网上找conda以及pip换源教程，依旧是换到阿里源，会让你体会到飞一般的速度。**

## 测试安装

在安装完后，本节的步骤将会测试安装是否成功，首先进入你之前建的anaconda虚拟环境，输入：

```c
python
```

进入python终端模式，在终端中输入

```c
import torch
print(torch.rand(5, 3))
```

**如果顺利的话会输出如下随机矩阵：**

![](https://pic2.zhimg.com/v2-38facf910087f4db50549d52c9276a89_b.jpg)

接下来输入

```c
torch.cuda.is_available()
```

**如果输出的是True那么恭喜你，大功告成，可以炼丹了！！！！**

![](https://pic1.zhimg.com/v2-d176c3d036abb51eaeda5ad16a7fbd60_b.jpg)