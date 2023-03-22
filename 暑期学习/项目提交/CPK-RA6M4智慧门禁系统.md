

## 说明文档

#### 1、应用场景

在数字化社会发展的今天，智慧社区的建设也越来越受到人们的关注，而智能门禁作为社区的首道防线，更是频繁出现在大众视野中。

本项目立足智慧门禁对社区人员签到、社区温湿环境检测、人机交互设备以及云端数据上报四大需求进行开发。

项目产品落地后可运用在多元化场景，而不是单纯服务于社区门禁，包括在办公写字楼、工厂工地、校园签到等等人员流动性大的场合，都可以尝试部署该产品。同时由于嵌入式设备的可扩展性，可及时对接用户需求，加装模块，设备的更新升级也获得极大便捷性。

#### 2、整体架构

该项目主要由瑞萨CPK-RA6M4开发板作为核心，使用AHT10、ESP8266、SSD1306、RC522这四个模块构成项目整体。

具体的整体架构如下图所示：

* 硬件架构图：

![image-20220802202357091](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022023168.png)

* 软件架构图：

![image-20220802210559475](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022105544.png)

#### 3、硬件情况说明

（1）ESP8266模块

简介：

ESP8266是一款超低功耗的UART-WiFi 透传模块，拥有业内极富竞争力的封装尺寸和超低能耗技术，专为移动设备和物联网应用设计，可将用户的物理设备连接到Wi-Fi 无线网络上，进行互联网或局域 网通信，实现联网功能。

![esp8266模块 的图像结果](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022121337.jpeg)

接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|    TX    |   P100   |
|    RX    |   P101   |
|   VCC    |    5V    |
|   GND    |   GND    |



（2）AHT10温湿度模块(i2c1)

简介：

AHT10是款高精度, 完全校准,贴片封装的温湿度传感器，MEMS的制作工艺， 确保产品具有极高的可靠性与卓越的长期稳定性。传感器包括一个电容式感湿元件和一个高性能CMOS微处理器相连接。该产品具有品质卓越超快响应、抗干扰能力强性价比极高等优点。

AHT10通信方式采用标准i2C通信方式，超小的体积、极低的功耗,使其成为各类应用甚至最为苛刻的应用场合的最佳选择。

![image-20220802212315237](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022123281.png)

接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   SCL    |   P512   |
|   SDA    |   P511   |
|   VCC    |   3.3V   |
|   GND    |   GND    |



（3）SSD1306 OLED屏（i2c0）

简介：

SSD1306是一款带控制器的用于OLED点阵图形显示系统的单片CMOS OLED/PLED驱动器。它由128个SEG（列输出）和64个COM（行输出）组成。该芯片专为共阴极OLED面板设计。

SSD1306内置对比度控制器、显示RAM（GDDRAM）和振荡器，以此减少了外部元件的数量和功耗。该芯片有256级亮度控制。数据或命令由通用微控制器通过硬件选择的6800/8000系通用并行接口、I2C接口或串行外围接口发送。该芯片适用于许多小型便携式应用，如手机副显示屏、MP3播放器和计算器等。

![SSD1306OLED 的图像结果](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022137197.jpeg)

接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   SCL    |   P400   |
|   SDA    |   P401   |
|   VCC    |   3.3V   |
|   GND    |   GND    |



（4）RC522读卡模块(spi1)

简介：

MFRC522是高度集成的非接触式(13.56MHz)读写卡芯片。此发送模块利用调制和解调的原理，并将它们完全集成到各种非接触式通信方法和协议中(13.56MHz)。

![RC522模块 的图像结果](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022141624.jpeg)

读卡机制说明：

![在这里插入图片描述](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022147385.png)

接线示意：

| 引脚功能 | 引脚接线 |
| :------: | :------: |
|   MOSI   |   P411   |
|   MISO   |   P410   |
|   SCL    |   P412   |
|   SDA    |   P311   |
|   RST    |   P312   |
|   VCC    |   3.3V   |
|   GND    |   GND    |
|   IRQ    |   悬空   |



#### 4、RT-Thread使用情况说明

* RT-Thread4.1.0版本
* ESP8266模块，用于AT联网及onenet云端数据上报
* RC522读卡模块，对IC卡进行读取
* AHT10温湿度传感器，用于温湿度数值的读取
* SSD1306 OLED显示屏，用于读卡提示及温湿度显示还有时间显示



本项目共设计三个线程，并且通过互斥量的使用来保证对资源的获取。

（1）AHT10温湿度读取线程：上电之后，经过对AHT10模块的初始化，使用i2c协议进行数据的传输，并将读取到的信息及时上报到ONENET云端，且显示温湿度数据到OLED屏幕上。

（2）ONENET线程：系统通过ESP8266模块使用AT命令联网，以及AHT10模块对数据的采集，使用MQTT协议将数据上报到ONENET云平台。

（3）OLED显示线程：该线程负责对传感器信息传输的反馈，包括初始化，刷卡感应，实时时间显示以及温湿度四大显示界面。



#### 5、软件说明

开发平台：

* 瑞萨FSP

---说明：瑞萨电子灵活配置软件包 (FSP) 是一款增强型软件包，旨在为使用瑞萨电子 RA 系列 ARM 微控制器的嵌入式系统设计提供简单易用且可扩展的高质量软件。



* RT-Thread Studio

---说明：RT-Thread Studio 是一个基于 Eclipse 的开发工具软件，主要包括工程创建和管理，代码编辑，SDK管理，RT-Thread配置，构建配置，调试配置，程序下载和调试等功能。



FSP配置截图：

![image-20220802215900976](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022159278.png)

![image-20220802220001920](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022200987.png)

![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022200987.png)



RTT studio使用到的软件包:

![image-20220802220203266](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022202359.png)

需要注意的是：

onenet上云可能会遇到问题,需要使能打开进程间通信管道，问题详情请见：[[CPK-RA6M4] onenet上云报错<RT-Thread 的版本为 4.1.0 及以上>](https://github.com/RT-Thread/rt-thread/issues/6188)

![image-20220802220406527](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208022204592.png)



#### 6、项目演示

代码仓库：[ITNG_Project_code](https://github.com/kurisaW/Project_hosting/tree/main/ITNG_Project/ITNG_Project_code)

演示视频：[RT-Thread 夏令营 瑞萨RA6M4开发板作品](https://www.bilibili.com/video/BV14S4y1x7fr?spm_id_from=333.999.0.0&vd_source=7de393144f462a4eade54292bd598c34)

![image-20220803155811326](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208031558134.png)