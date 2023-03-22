【经验分享】zlib库在Ubuntu下的安装与配置
## 一、资料准备

下载好zlib压缩包（[点击此处下载](https://download.csdn.net/download/qq_56914146/85151466)）

解压zlib库
![在这里插入图片描述](https://img-blog.csdnimg.cn/463ada1fcdc94aeb85c7e7a49b3327b1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 二、配置zlib库，得到makefile：
```
./configure
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7e38d4071714e4d86feb58f6e1422c8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4a3da23c07624f75acab1d0a1a81d31a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
## 三、回到上一级，进入libpng-1.6.6目录下
输入命令：

	./configure

![在这里插入图片描述](https://img-blog.csdnimg.cn/36b1e2cc4bdb42019d127e711fcd7ca7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)
再次执行：
```
make
make install
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/659e6a1a2af3433c82ae907b91ea178e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lul5pS-Xw==,size_20,color_FFFFFF,t_70,g_se,x_16)