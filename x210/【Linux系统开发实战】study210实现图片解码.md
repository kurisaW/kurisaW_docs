【Linux系统开发实战】study210实现图片解码
## 一、准备工作
1.环境：VMware® Workstation 16 Pro
2.文件：源码包（[可在此处获得](https://download.csdn.net/download/qq_56914146/85151388?spm=1001.2014.3001.5501)）

 ## 二、libpng移植
 `注意：这里需要在虚拟机中安装zlib库，如果已经安装好了，请省略。`
* 下载[zlib-1.2.8.tar.gz](https://download.csdn.net/download/qq_56914146/85151466)库
* [zlib库配置教学](https://blog.csdn.net/qq_56914146/article/details/124211318)

1.首先在根目录下创建一个名为`decodeporting`的文件夹，需要下载好上面的源码包，并将源码包复制到该文件夹下

![image-20230424143425509](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304241434573.png)

执行解压

```
tar ‐zxvf libpng‐1.6.6.tar.gz
```
2.进入`libpng-1.6.6`文件夹,将下载好的zlib库压缩包复制到decodeporting文件下

![image-20230424143445525](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304241434634.png)


3.进入libpng-1.6.6目录下，运行
```
./configure
```
再次输入：
```
make
make install
```

## 三、参考源码包的内部资料(在libpng-1.6.6文件夹中)

1.readme
2.libpng-manual.txt
3.example.c 和 pngtest.c

## 四、开始编程
#### 1.先is_png函数使用判断一个图片文件是不是png图片