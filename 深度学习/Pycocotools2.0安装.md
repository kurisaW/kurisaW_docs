# Microsoft Visual C++ 14.0 安装及Pycocotools2.0版本安装教学（防踩坑）

## 1、Microsoft Visual C++ 14.0安装

这里附上百度网盘下载链接：https://pan.baidu.com/s/1EVujXX5xCYqljxnBAh0kNw?pwd=aut0 
提取码：aut0

下载完成后双击打开

![image-20220712162104640](G:/Typora/user_potograph/image-20220712162104640-16576157745301.png)

默认下载方式即可 

---

## 2、Pycocotools2.0版本安装

#### （1）准备材料：

* [下载pycocotools安装包](https://github.com/cocodataset/cocoapi)（可直接git拉取到本地文件夹）

#### （2）源码配置

打开下载好的pycocotools，双击打开`setup.py`（文件路径：\cocoapi\PythonAPI\setup.py）

![image-20220712163123209](G:/Typora/user_potograph/image-20220712163123209-16576159645513.png)

这里`将蓝色部分删除，只保留红色部分`(切记需要执行这一步！！！)

开始界面找到所有应用并打开`Anaconda Powershell Prompt`

![image-20220712163655088](G:/Typora/user_potograph/image-20220712163655088.png)

先打开自己创建的虚拟环境，这里我的虚拟环境为python_env，可供参考。

如上图所示进入到`\cocoapi\PythonAPI`该目录下

分别执行以下两个命令：

```python
python setup.py build_ext --inplace
python setup.py build_ext install
```

![image-20220712163946593](G:/Typora/user_potograph/image-20220712163946593.png)

执行pip list查看

![image-20220712164152812](G:/Typora/user_potograph/image-20220712164152812.png)

至此所有问题解决！