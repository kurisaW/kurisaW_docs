# ubuntu下安装tensorflow_v1推荐

在Ubuntu中安装Anaconda可以根据 [这篇博客](https://blog.csdn.net/weixin_39059031/article/details/82085916),写的十分详细。  
装好之后，可以通过通过命令安装版本为X.X\(如2.7，3.6）的虚拟环境：

```
conda create -n your_env_name python=X.X
```

其中\(your\_env\_name\)是你所创建的虚拟环境的名字。  
然后 使用下方命令激活你刚才建立的环境

```
source activate your_env_name
```

激活了环境之后，我们可以看到命令行之前多了一个括号，括号中包含了你的环境名字，这就说明激活成功。  
在激活的环境中使用如下命令安装CUDA：

```
conda install cudatoolkit=8.0 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
```

我装CUDA的同时，自动装上了CUDNN。  
如果你没有装上，也可以使用下方命令安装CUDNN：

```
conda install cudnn=7.0.5 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/
```

其中CUDA的版本可以自己更改，我改成了9.2.0，CUDNN应该也可以改，不过我没有尝试。  
[这篇文章](https://blog.csdn.net/omodao1/article/details/83241074)还展示了Tensorflow不同版本要求与CUDA及CUDNN版本对应关系，一般我们都是装好了tensorflow-gpu，然后需要对应版本的CUDA及CUDNN，可以让大家参考一下。  
仍然是在激活的环境中安装tensorflow，比如我的CUDA是9.2.0，我的CUDNN是7.2.1，那么对应的Tensorflow就可以安装1.5.0，如下方命令所示：

```
pip install tensorflow-gpu==1.5.0
```

以上所安装的Python、CUDA、CUDNN以及Tensorflow都是在虚拟环境中安装的，下次想要使用该环境时或者在该环境安装其他库包文件，需要重新进入虚拟环境。与之前的代码一样，如下：

```
source activate your_env_name
```

查询当前环境下的库的版本号可使用以下命令：

```
conda list cudnn
conda list cuda
```