# RT-Thread 7.19 记录 深度学习 人形识别

## 1.直接烧录 编译好的人形识别的[二进制](https://so.csdn.net/so/search?q=%E4%BA%8C%E8%BF%9B%E5%88%B6&spm=1001.2101.3001.7020).bin文件

## 1.1准备

ART-Pi\_W25Q64.stldr以及rtthread.bin文件下载链接：https://pan.baidu.com/s/1tFP60TIHSvbUqHM8IPSvvw   
提取码：1111

## 1.2操作步骤

第一步：打开RT-Thread Studio，创建RT-Thread项目

![](https://img-blog.csdnimg.cn/20210720015833102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 并创建示例工程：art\_pi\_bootloader![](https://img-blog.csdnimg.cn/2021072001584153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 第二步：把ART-Pi\_W25Q64.stldr以及rtthread.bin这两个文件都拷贝在下面这个路径里：

D:\\RT-ThreadStudio\\repo\\Extract\\Debugger\_Support\_Packages\\STMicroelectronics\\ST-LINK\_Debugger\\1.6.0\\tools\\bin

![](https://img-blog.csdnimg.cn/2021072001441956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

第三步：在地址栏输入cmd

![](https://img-blog.csdnimg.cn/2021072002015439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)  
第四步：在“黑框框”里输入：

STM32\_Programmer\_CLI \-c port=SWD \--extload ./ART-Pi\_W25Q64.stldr \-d rtthread.bin 0x90000000 \-hardRst \-s

 会显示如下界面：![](https://img-blog.csdnimg.cn/20210720020229549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 第五步： 把摄像头对准人像，设备可以检测出人形，并用红色矩形框选![](https://img-blog.csdnimg.cn/20210720020953821.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

## 2.从头开始创建

## 2.1准备

下载两个压缩包：Lab3-PersonDetection.zip和art-pi-gc0328c.zip

链接：https://pan.baidu.com/s/1aG2bLpYtTrAG9314uMhWyg   
提取码：1111

## 2.2操作步骤

第一步：把art-pi-gc0328c.zip保存在该路径下并解压：

D:\\RT-ThreadStudio\\workspace

![](https://img-blog.csdnimg.cn/20210720021854468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

第二步：打开RT-Thread Studio，导入RT-Thread项目

![](https://img-blog.csdnimg.cn/2021072002201291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20210720022203838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70) ![](https://img-blog.csdnimg.cn/20210720022203832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

（注意：这个“完成\(F\)“应该是可以按的，我的不能按是因为我之前导入过该工程。）

 第三步：在Lab3-PersonDetection\\BSP路径下，找见这三个文件：

![](https://img-blog.csdnimg.cn/20210720022551694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)把这个三个文件复制粘贴在该目录下：

D:\\RT-ThreadStudio\\workspace\\art-pi-gc0328c\\applications

![](https://img-blog.csdnimg.cn/20210720022759729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

第四步：RT-AK一键部署AI

在该路径下的（D:\\rtt\\Day2\_0718\\RT-AK\\rt\_ai\_tools）地址栏输入cmd：

![](https://img-blog.csdnimg.cn/20210720023045997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 然后再“黑框框”里输入：

python aitools.py \--project="D:\\RT-ThreadStudio\\workspace\\art-pi-gc0328c" \--model="D:\\rtt\\Day3\_0719\\Lab3-PersonDetection\\model\\yolo-s.tflite" \--model\_name="person\_yolo" \--platform=stm32 \--ext\_tools="D:\\rtt\\Day2\_0718\\windows" \--clear

---

```
我的修改为:

python aitools.py --project="D:\RT-Thread\RT-ThreadStudio\workspace\art-pi-gc0328c" --model="D:\RT-Thread\ART-Pi_test_1\Lab3-PersonDetection\model\yolo-s.tflite"--model_name="person_yolo"--platform=stm32--ext_tools="D:\RT-Thread\ART-Pi_test_1\windows"--clear
```

---

显示如下界面即为成功：

![](https://img-blog.csdnimg.cn/20210720023917797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 第五步：左键单击选中ari-pi-gc0328c工程，然后依次执行：更新软件包，刷新。![](https://img-blog.csdnimg.cn/20210720024203316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

然后，applications中就会把rt\_ai\_person\_yolo\_model.c，rt\_ai\_person\_yolo\_model.h，yolo\_layer.c，yolo\_layer.h，main.c五个文件全部更新加载进工程中。

 ![](https://img-blog.csdnimg.cn/20210720024401350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 第六步：编译、下载工程

编译：

![](https://img-blog.csdnimg.cn/2021072002470280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

 打开串口：![](https://img-blog.csdnimg.cn/20210720024814298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

下载：

![](https://img-blog.csdnimg.cn/20210720024814296.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

下载成功界面：

![](https://img-blog.csdnimg.cn/20210720025553169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)

第七步：把摄像头对准人像，设备可以检测出人形，并用红色矩形框选

![](https://img-blog.csdnimg.cn/20210720025553212.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2MzI3OA==,size_16,color_FFFFFF,t_70)