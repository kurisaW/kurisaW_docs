## 1、创建虚拟环境

```
输入创建环境命令：
conda create -n rtt_env python=3.9

//rtt_env:虚拟环境名称
//python=3.9：对应python版本号

然后回车默认yes继续创建虚拟环境
```

![image-20220427103942778](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427103942778.png)

## 2.激活

```
输入虚拟环境激活命令：conda activate rtt_env(自己创建的虚拟环境名)，此时就进入到虚拟环境了,之后我们的所有操作都将在该虚拟环境中运行
```

![image-20220427104042864](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427104042864.png)

```
输入命令：
conda install package_name:安装指定包
conda install package_name=版本号：安装特定版本
```

```
conda常用命令

conda list：查看安装了哪些包。
conda deactivate rrt_env：退出虚拟环境
conda remove -n rrt_env --all：删除特定虚拟环境
conda install package_name(包名)：安装包
conda env list 或 conda info -e：查看当前存在哪些虚拟环境
conda update conda：检查更新当前conda
```



## 3.安装jupyter notebook

```
输入命令安装jupyter notebook：
conda install jupyter notebook
```

![image-20220427105350149](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427105350149.png)

```
安装成功后我们输入jupyter notebook验证，可以发现会自动跳转到一个网页，这个就是jupyter页面，jupyter默认为打开一个系统目录
```

![image-20220427105513053](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427105513053.png)

```
打开自定义目录：
miniconda ctr+c 退出
输入命令：jupyter notebook 指定目录
```

## 4.在jupyter notebook中运行虚拟环境

```
首先回到我们创建的虚拟环境终端，输入如下命令：
pip install ipykernel
python -m ipykernel install --user --name rtt_env(指定虚拟环境)
```

![image-20220427110645187](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427110645187.png)

```
这样就成功将rtt_env这个虚拟环境嵌入jupyter notebook中
```

![image-20220427111154578](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427111154578.png)

```
这样我们就可以在jupyter notebook中运行虚拟环境了
```

## 5.安装库

```
后面的训练模型会用到一些库，帮助我们更好运行，先在虚拟环境中安装
```

（1）TensorFlow

<1>背景

> [TensorFlow](https://so.csdn.net/so/search?q=TensorFlow&spm=1001.2101.3001.7020)是一个基于[数据流编程](https://baike.baidu.com/item/数据流编程/22735640)（dataflow programming）的符号数学系统，被广泛应用于各类[机器学习](https://baike.baidu.com/item/机器学习/217599)（machine learning）算法的编程实现，其前身是[谷歌](https://baike.baidu.com/item/谷歌/117920)的[神经网络](https://so.csdn.net/so/search?q=神经网络&spm=1001.2101.3001.7020)算法库DistBelief。

> Tensorflow拥有多层级结构，可部署于各类[服务器](https://baike.baidu.com/item/服务器/100571)、PC终端和[网页](https://baike.baidu.com/item/网页/99347)并支持[GPU](https://baike.baidu.com/item/GPU/105524)和[TPU](https://baike.baidu.com/item/TPU/20473545)高性能[数值计算](https://baike.baidu.com/item/数值计算/3729797)，被广泛应用于谷歌内部的产品开发和各领域的科学研究。

<2>资源包安装

```
conda install tensorflow
```

![image-20220427112358541](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220427112358541.png)

​	

（2）matplotlib

<1>背景

> **matplotlib**是[Python](https://zh.wikipedia.org/wiki/Python)语言及其数值计算库[NumPy](https://zh.wikipedia.org/wiki/NumPy)的[绘图](https://zh.wikipedia.org/w/index.php?title=绘图工具&action=edit&redlink=1)[库](https://zh.wikipedia.org/wiki/函式庫)。它提供了一个[面向对象](https://zh.wikipedia.org/wiki/面向对象程序设计)的[API](https://zh.wikipedia.org/wiki/应用程序接口)，用于使用通用[GUI工具包](https://zh.wikipedia.org/wiki/部件工具箱)（如[Tkinter](https://zh.wikipedia.org/wiki/Tkinter)、[wxPython](https://zh.wikipedia.org/wiki/WxPython)、[Qt](https://zh.wikipedia.org/wiki/Qt)或[GTK](https://zh.wikipedia.org/wiki/GTK)）将绘图嵌入到应用程序中。它还有一个基于[状态机](https://zh.wikipedia.org/wiki/有限状态机)（如[OpenGL](https://zh.wikipedia.org/wiki/OpenGL)）的[过程式编程](https://zh.wikipedia.org/wiki/过程式编程)“pylab”接口，其设计与[MATLAB](https://zh.wikipedia.org/wiki/MATLAB)非常类似，但不推荐使用。[SciPy](https://zh.wikipedia.org/wiki/SciPy)使用matplotlib进行图形绘制

<2>资源包安装

```
conda install matplotlib
```

