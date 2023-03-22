# 《玩转ART-Pi开发板》第7章 基于ART-Pi 环境监测系统

发布于 2021-04-18 16:00:47  浏览：441

[原文地址](https://blog.bruceou.cn/2021/04/7-environmental-monitoring-system-1/842/)

**开发环境：**
RT-Thread版本：4.0.3
操作系统：Windows10
Keil版本：V5.30
RT-Thread Studio版本：2.0.1
开发板MCU：STM32H750XB

从本章开始，笔者不在就某一个单一功能讲解，而是针对某一个具体的项目作为讲解的主要内容。

## 7.1前言

第一个项目是一个环境监测系统，我相信很多朋友都做过，我这里主要从宏观层面来把握，你学会这个系统，那么你就可以将其移植到很多实际场景。

环境监测系统分为三个部分，一部分是**终端节点**，也就是采集环境信息，一般通过组网方式将各个节点汇聚到一个终端，再由该终端进行转发，常见的组网方式有ZigBee、BLE等。中间部分就是**网关**，它负责将各个终端的数据通过网络传到云端，根据实际的业务需求来购置相应的设备。最后一块就是**云服务器**了，当然现在云服务很多，免费版的也不少。环境监测架构如下图所示：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center.png)

## 7.2环境数据获取

为了简单起见，我这里使用RT-Thread Studio开发，关于如何使用RT-Thread Studio创建项目，请参看官方手册。

[RT-Thread Studio](https://www.rt-thread.org/document/site/rtthread-studio/nav/)

### 7.2.1 RT-Thread Studio创建项目

第一个项目笔者还是简单讲解下，首先打开RT-Thread Studio，新建项目。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255131.png)

接下来选择新建“RT-Thread项目”，然后点击“下一步”。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255132.png)

接下来填写工程名，选择开发板ART-Pi，其他默认即可。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255143.png)

点击“完成”，稍等片刻即可完成项目的创建。当然接下来你也可以使用MDK开发，我这里还是继续使用RT-Thread Studio开发。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255144.png)

工程创建好了，接下里就是开发工作了。

### 7.2.2 DHT11获取温湿度

RT-Thread提供了DHT11的驱动软件包，配置如下：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255145.png)

DHT11默认使用的 GPIO是 GPIOB_12。当然这是可以修改的，在 dht11_sample.c 中修改以下代码：

```
#define DHT11_DATA_PIN    GET_PIN(B, 12)
```

笔者这里直接接到默认引脚上，我就不修改了。

程序编译下载到板卡后，会在串口中每 1s 打印一次温湿度数据。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255146.png)

既然驱动有现成的，那么只需要写个相应的应用代码即可。后文需要将得到温湿度值上传至云端，应用部分的代码如下：

```
#include <rtthread.h>#include <rtdevice.h>#include "sensor.h"#include "sensor_dallas_dht11.h"#include "drv_gpio.h"/* Modify this pin according to the actual wiring situation */#define DHT11_DATA_PIN    GET_PIN(B, 12)typedef struct{    uint8_t temp;    uint8_t humi;}rt_dht11;rt_dht11 dht11;static void read_temp_entry(void *parameter){    rt_device_t dev = RT_NULL;    struct rt_sensor_data sensor_data;    rt_size_t res;    rt_uint8_t get_data_freq = 1; /* 1Hz */    dev = rt_device_find("temp_dht11");    if (dev == RT_NULL)    {        return;    }    if (rt_device_open(dev, RT_DEVICE_FLAG_RDWR) != RT_EOK)    {        rt_kprintf("open device failed!\n");        return;    }    rt_device_control(dev, RT_SENSOR_CTRL_SET_ODR, (void *)(&get_data_freq));    while (1)    {        res = rt_device_read(dev, 0, &sensor_data, 1);        if (res != 1)        {            rt_kprintf("read data failed! result is %d\n", res);            rt_device_close(dev);            return;        }        else        {            if (sensor_data.data.temp >= 0)            {                dht11.temp = (sensor_data.data.temp & 0xffff) >> 0;      // get temp                dht11.humi = (sensor_data.data.temp & 0xffff0000) >> 16; // get humi                rt_kprintf("temp:%d, humi:%d\n" ,dht11.temp, dht11.humi);            }        }        rt_thread_delay(1000);    }}static int dht11_thread(void){    rt_thread_t dht11_thread;    dht11_thread = rt_thread_create("dht_tem",                                     read_temp_entry,                                     RT_NULL,                                     1024,                                     RT_THREAD_PRIORITY_MAX / 2,                                     20);    if (dht11_thread != RT_NULL)    {        rt_thread_startup(dht11_thread);    }    return RT_EOK;}/* 导出到 msh 命令列表中 */MSH_CMD_EXPORT(dht11_thread, dht11 thread);static int rt_hw_dht11_port(void){    struct rt_sensor_config cfg;    cfg.intf.user_data = (void *)DHT11_DATA_PIN;    rt_hw_dht11_init("dht11", &cfg);    return RT_EOK;}INIT_COMPONENT_EXPORT(rt_hw_dht11_port);
```

### 7.2.3 DHT11总结

RT-Thread包含DHT11驱动，这里就没有去造轮子了，但是作为学习者，还是要去了解DHT11的原理及具体实现。这部分内容在笔者博客的STM32系列的外设篇已详细阐述，下面就DHT11驱动在RTT中的实现做个总结。

DHT11是采用**单总线通讯**的传感器，有的设备没有硬件单总线，DHT11的支持包采用 **GPIO 模拟单总线时序**。DHT11 的一次完整读时序需要 20ms，时间过长，故无法使用关中断或者关调度的方式实现独占 CPU 以保证时序完整正确。因此可能出现读取数据失败的情况。

DHT11的典型电路如下图所示。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255147.png)

DATA 用于微处理器与 DHT11之间的通讯和同步,采用单总线数据格式,一次通讯时间4ms左右,数据分小数部分和整数部分,具体格式在下面说明,当前小数部分用于以后扩展,现读出为零.操作流程如下:

**一次完整的数据传输为40bit，高位先出。**

数据格式：8bit湿度整数数据+8bit湿度小数数据+8bi温度整数数据+8bit温度小数数据+8bit校验和

数据传送正确时校验和数据等于“8bit湿度整数数据+8bit湿度小数数据+8bi温度整数数据+8bit温度小数数据” 所得结果的末8位。

用户MCU发送一次开始信号后，DHT11从低功耗模式转换到高速模式，等待主机开始信号结束后，DHT11发送响应信号，送出40bit的数据，并触发一次信号采集用户可选择读取部分数据。从模式下，DHT11接收到开始信号触发一次温湿度采集，如果没有接收到主机发送开始信号，DHT11不会主动进行温湿度采集.采集数据后转换到低速模式。通讯过程如下图所示：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255148.png)

熟悉RT-Thread系统都知道，RT-Thread将各个模块进行抽象，为上层提供统一的操作接口，依次提高上层代码的可重用性，自然也提高了应用开发效率。RT-Thread的驱动框架：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-16556858255159.png)

应用程序通过 I/O 设备管理接口获得正确的设备驱动，然后通过这个设备驱动与底层 I/O 硬件设备进行数据（或控制）交互。

I/O设备模型框架位于硬件和应用程序之间，共分成三层，从上到下分别是 **I/O 设备管理层、设备驱动框架层、设备驱动层**。

- I/O设备管理层实现了对设备驱动程序的封装。应用程序通过 I/O 设备层提供的标准接口访问底层设备，设备驱动程序的升级、更替不会对上层应用产生影响。这种方式使得设备的硬件操作相关的代码能够独立于应用程序而存在，双方只需关注各自的功能实现，从而降低了代码的耦合性、复杂性，提高了系统的可靠性。
- 设备驱动框架层是对同类硬件设备驱动的抽象，将不同厂家的同类硬件设备驱动中相同的部分抽取出来，将不同部分留出接口，由驱动程序实现。
- 设备驱动层是一组驱使硬件设备工作的程序，实现访问硬件设备的功能。它负责创建和注册 I/O 设备，对于操作逻辑简单的设备，可以不经过设备驱动框架层，直接将设备注册到 I/O 设备管理器中。

更加详细的内容请参看RT-Thread官方手册。

这里主要讲解Sensor驱动，RT-Thread将各个Sensor进行抽象，将不同厂商的Sensor合并为Sensor设备，从而提高了代码的重用性，提高开发效率。既然是总结，这里就只说重点，先看看DHT11设备驱动的时序图：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551510.png)

图10 DHT11设备驱动的时序图

传感器数据接收和发送数据的模式分为 3 种：**中断模式、轮询模式、FIFO 模式**。在使用的时候，这 3 种模式只能选其一，若传感器的打开参数 oflags 没有指定使用中断模式或者 FIFO 模式，则默认使用轮询模式。

oflags 参数支持下列参数：

```
#define RT_DEVICE_FLAG_RDONLY       0x001     /* 标准设备的只读模式，对应传感器的轮询模式 */#define RT_DEVICE_FLAG_INT_RX       0x100     /* 中断接收模式 */#define RT_DEVICE_FLAG_FIFO_RX      0x200     /* FIFO 接收模式 */
```

DHT11设备比较简单，采用的是轮询模式，其设备注册函数如下：

```
result = rt_hw_sensor_register(sensor, name, RT_DEVICE_FLAG_RDONLY, RT_NULL);
```

硬件初始化如下：

```
static int rt_hw_dht11_port(void){    struct rt_sensor_config cfg;    cfg.intf.user_data = (void *)DHT11_DATA_PIN;    rt_hw_dht11_init("dht11", &cfg);    return RT_EOK;}INIT_COMPONENT_EXPORT(rt_hw_dht11_port);
```

注意INIT_COMPONENT_EXPORT表示组件初始化，初始化顺序为4。
好了，DHT11就讲到这里了。

[自动初始化](https://blog.bruceou.cn/2021/02/5-api-pi-auto-initialization/603/)

## 7.3联网【WiFi】

ART-Pi有两种联网方式，一个是板载的WiFi模块AP6212，这个模块自带蓝牙；另一个是工业扩展板的网口，使用的芯片是LAN8720A，我没有扩展板，这里就只讲解如何使用WiFi联网。这里先看看WiFi的电路。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551511.png)

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551512.png)
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551513.png)

从上图可以看出WiFi和BT使用的是二合一芯片AP6212，WiFi的接口是SDIO，BT使用的是串口，还是很简单的。

接下来就是针对WiFi进行简单的配置。

### 7.3.1 WiFi配置

首先是配置WiFi底层驱动。
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551514.png)

然后是添加WiFi库。
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551515.png)

接下来的配置是为了优化WiFi而配置的。

**优化配置一：添加easyflash库**

默认情况下，连接WiFi后，重启板子后需要重新连接WiFi，可以将WiFi的信息记录在flash中，重新后再次连接即可，这里就需要添加easyflash包。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551616.png)

**优化配置二：BT配网**
我们在首次使用开发板连接WiFi，需要命令来连接，这就需要PC有串口驱动和终端助手，还是比较麻烦，接下来的方式是通过板载的BT软件，通过智能手机将WiFi信息传给板子，这样只需给板子上电就可以连接WiFi了，但这需要额外的手机APP或者微信小程序等，官方提供了相应的小程序，这里直接用就可以了。BT配网配置比较多。

首先需要使能BT协议栈。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551617.png)

BT使用的是串口3因此需要打开串口3。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551618.png)

还需要JSON库、RTC和文件系统。
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551619.png)

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551620.png)

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551621.png)

好了，关于WiFi的配置差不多了，接下来就是实现WiFi连网、断电重连、BT联网。

### 7.3.2 WiFi代码实现

首先增加一个模块文件夹，用于配置BT和WiFi，具体内容请看源码。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551622.png)

图21添加modules文件夹

**【bt_module.c】**

```
#include <rtthread.h>#include <fal.h>#include <rt_ota.h>#include <wifi_module.h>#define DBG_TAG "bt"#define DBG_LVL DBG_LOG#include <rtdbg.h>#define BT_FIRMWARE_PARTITION_NAME "bt_image"static const struct fal_partition *bt_partition = RT_NULL;extern void bt_stack_port_main(void);void bluetooth_thread(void *param){    bt_stack_port_main();}int bluetooth_firmware_check(void){    bt_partition = fal_partition_find(BT_FIRMWARE_PARTITION_NAME);    if (bt_partition == NULL)    {        LOG_E("%s partition is not exist, please check your configuration!", BT_FIRMWARE_PARTITION_NAME);        return -1;    }    else    { //try to update the bt_image        int result = 0;        /* verify OTA download partition */        if (rt_ota_init() >= 0 && rt_ota_part_fw_verify(fal_partition_find(RT_OTA_DL_PART_NAME)) >= 0)        {            /* do upgrade when check upgrade OK */            if (rt_ota_check_upgrade() && (result = rt_ota_upgrade()) < 0)            {                log_e("OTA upgrade failed!");                //TODO upgrade to safe image?            }            /* when upgrade success, erase download partition firmware.             * The purpose is to prevent other programs from using.             */            if (result >= 0)            {                fal_partition_erase(fal_partition_find(RT_OTA_DL_PART_NAME), 0, 4096);            }        }    }    //check firmware validity    if (rt_ota_part_fw_verify(bt_partition) < 0)    {        LOG_E("BT image was NOT found on %s partition!", BT_FIRMWARE_PARTITION_NAME);        return -1;    }    return 0;}int bluetooth_init(void){    rt_device_t wifi = NULL, bt_firmware = NULL;    //wait for wifi is ready    while (wifi == NULL)    {        wifi = rt_device_find(WIFI_DEVICE_NAME);        rt_thread_mdelay(500);    }    LOG_D("bluetooth start init");    //wait for firmware is ready    if (bluetooth_firmware_check() < 0)    {        return -1;    }    //create bt device fs    bt_firmware = fal_char_device_create(BT_FIRMWARE_PARTITION_NAME);    if (bt_firmware == NULL)    {        LOG_E("bt firmware device create failed");        return -1;    }    rt_thread_t tid = rt_thread_create("bt_thread", bluetooth_thread, NULL, 1024, 15, 5);    if (tid)    {        rt_thread_startup(tid);        return 0;    }    else    {        LOG_E("bluetooth thread create failed");        return -1;    }}
```

bt_module.c添加的是BT初始化的内容。

**【bt_module.h】**

```
#ifndef __BTMODULE_H#define __BTMODULE_Hint bluetooth_init(void);#endif  /*__BLUETOOTH_H*/
```

**【wifi_module.c】**

```
#include <rtthread.h>#include <rtdevice.h>#include <cJSON.h>#include <wlan_mgnt.h>#include <netdev_ipaddr.h>#include <netdev.h>#include <stdio.h>#include <stdlib.h>#include <wifi_module.h>#define DBG_TAG "wifi"#define DBG_LVL DBG_LOG#include <rtdbg.h>#define MAX_SSID_PASSWD_STR_LEN 50#define BT_SEND_TIMES 1#define BT_SEND_FAIL_RETRY 3extern int bt_stack_blufi_send(uint8_t *string, uint32_t length);extern int adb_socket_init(void);char wifi_status_str[100];struct _wifi{    char ssid[MAX_SSID_PASSWD_STR_LEN];    char passwd[MAX_SSID_PASSWD_STR_LEN];} wifi;#ifdef RT_WLAN_AUTO_CONNECT_ENABLE#include <wlan_mgnt.h>#include <wlan_cfg.h>#include <wlan_prot.h>#include <easyflash.h>#include <fal.h>static int read_cfg(void *buff, int len){    size_t saved_len;    ef_get_env_blob("wlan_cfg_info", buff, len, &saved_len);    if (saved_len == 0)    {        return 0;    }    return len;}static int get_len(void){    int len;    size_t saved_len;    ef_get_env_blob("wlan_cfg_len", &len, sizeof(len), &saved_len);    if (saved_len == 0)    {        return 0;    }    return len;}static int write_cfg(void *buff, int len){    /* set and store the wlan config lengths to Env */    ef_set_env_blob("wlan_cfg_len", &len, sizeof(len));    /* set and store the wlan config information to Env */    ef_set_env_blob("wlan_cfg_info", buff, len);    return len;}static const struct rt_wlan_cfg_ops ops ={    read_cfg,    get_len,    write_cfg};void wlan_autoconnect_init(void){    fal_init();    easyflash_init();    rt_wlan_cfg_set_ops(&ops);    rt_wlan_cfg_cache_refresh();    /* enable auto reconnect on WLAN device */    rt_wlan_config_autoreconnect(RT_TRUE);}#endifint wifi_connect(char *conn_str){    cJSON *conn = cJSON_Parse(conn_str);    if (conn)    {        cJSON *ssid = cJSON_GetObjectItem(conn, "ssid");        cJSON *passwd = cJSON_GetObjectItem(conn, "passwd");        rt_memset(wifi.ssid,0,sizeof(wifi.ssid));        rt_memset(wifi.passwd,0,sizeof(wifi.passwd));        if (ssid && passwd)        {            if (rt_strlen(ssid->valuestring) > MAX_SSID_PASSWD_STR_LEN ||                rt_strlen(passwd->valuestring) > MAX_SSID_PASSWD_STR_LEN)            {                LOG_E("invalid ssid or passwd length,max %d", MAX_SSID_PASSWD_STR_LEN);            }            else            {                rt_memcpy(wifi.ssid, ssid->valuestring, rt_strlen(ssid->valuestring));                rt_memcpy(wifi.passwd, passwd->valuestring, rt_strlen(passwd->valuestring));                return rt_wlan_connect(wifi.ssid, wifi.passwd);            }        }        else        {            LOG_E("cannot find ssid or password.");        }        cJSON_Delete(conn);    }    else    {        LOG_E("invalid wifi connection string.");    }    return -1;}int wifi_is_ready(void){    return rt_wlan_is_ready();}char *wifi_get_ip(void){    struct netdev *dev = netdev_get_by_name(WIFI_DEVICE_NAME);    if (dev)    {        return inet_ntoa(dev->ip_addr);    }    else    {        return NULL;    }}char *wifi_status_get(void){    rt_memset(wifi_status_str, 0, sizeof(wifi_status_str));    uint8_t wifi_status = wifi_is_ready();    char *wifi_ip = wifi_get_ip();    rt_sprintf(wifi_status_str, "{wifi:'%s', url:'%s'}", wifi_status ? "on" : "off", wifi_ip);    return wifi_status_str;}static void wifi_ready_handler(int event, struct rt_wlan_buff *buff, void *parameter){    int cnt = BT_SEND_TIMES;    //wifi status send    rt_memset(wifi_status_str, 0, sizeof(wifi_status_str));    while (cnt--)    {        char *wifi_status = wifi_status_get();        int retry_cnt = BT_SEND_FAIL_RETRY;        while (bt_stack_blufi_send((uint8_t *)wifi_status, rt_strlen(wifi_status)) < 0)        {            if (retry_cnt == 0)                break;            retry_cnt--;            rt_thread_mdelay(1000);        }        rt_thread_mdelay(5000);    }}int wifi_init(void){    rt_memset(&wifi, 0, sizeof(wifi));    rt_wlan_register_event_handler(RT_WLAN_EVT_READY, wifi_ready_handler, NULL);    return 0;}
```

wifi_module.c添加了WiFi接收BT信息的初始化和自动连接初始化的内容。

**【wifi_module.h】**

```
#ifndef __WIFI_MODULE_H__#define __WIFI_MODULE_H__#define WIFI_DEVICE_NAME "w0"#ifdef RT_WLAN_AUTO_CONNECT_ENABLEvoid wlan_autoconnect_init(void);#endifint wifi_init(void);int wifi_is_ready(void);char *wifi_get_ip(void);int wifi_connect(char *conn_str);char *wifi_status_get(void);#endif /*__WIFIMODULE_H*/
```

最后，在main.c中初始化即可。

```
//wifi initwifi_init();//bt initbluetooth_init();/* init Wi-Fi auto connect feature */wlan_autoconnect_init();
```

另外btstack-v0.0.1工具包修改了内容比较多，我这里就不在一一列出，主要要有bt的前缀名，设备路径，BT复位引脚。具体修改内容请参看完整代码。

**【关键问题】**

**1.未定义bt_stack_blufi_send**

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70.png)

这个问题很好解决，主要是bt_stack_blufi_send函数是一个静态函数，外部文件访问不到，去掉“static”即可。
![在这里插入图片描述](G:/Typora/user_potograph/20210418130651901.png)
**2.程序崩溃**

如果没有使能文件系统则会出现以下错误：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70-165568582551723.png)

### 7.3.3 WiFi联网测试

一切完成后，编译，下载，打开终端。

```
   ___  ______  _____         ______  _   ______  _____  _____  _____   / _ \ | ___ \|_   _|        | ___ \(_)  | ___ \/  _  \/  _  \|_   _| / /_\ \| |_/ /  | |   ______ | |_/ / _   | |_/ /| | | || | | |  | |   |  _  ||    /   | |  |______||  __/ | |  | ___ \| | | || | | |  | |   | | | || |\ \   | |          | |    | |  | |_/ /\ \_/ /\ \_/ /  | |   \_| |_/\_| \_|  \_/          \_|    |_|  \____/  \___/  \___/   \_/   Powered by RT-Thread. \ | /- RT -     Thread Operating System / | \     4.0.3 build Apr 14 2021 2006 - 2020 Copyright by rt-thread teamlwIP-2.0.2 initialized![I/sal.skt] Socket Abstraction Layer initialize success.[I/SFUD] Find a Winbond flash chip. Size is 16777216 bytes.[I/SFUD] norflash0 flash device is initialize success.[I/SFUD] Probe SPI flash norflash0 by SPI device spi10 success.[D/FAL] (fal_flash_init:63) Flash device |                norflash0 | addr: 0x00000000 | len: 0x01000000 | blk_size: 0x00001000 |initialized finish.[I/FAL] ==================== FAL partition table ====================[I/FAL] | name       | flash_dev |   offset   |    length  |[I/FAL] -------------------------------------------------------------[I/FAL] | wifi_image | norflash0 | 0x00000000 | 0x00080000 |[I/FAL] | bt_image   | norflash0 | 0x00080000 | 0x00080000 |[I/FAL] | download   | norflash0 | 0x00100000 | 0x00200000 |[I/FAL] | easyflash  | norflash0 | 0x00300000 | 0x00100000 |[I/FAL] | filesystem | norflash0 | 0x00400000 | 0x00c00000 |[I/FAL] =============================================================[I/FAL] RT-Thread Flash Abstraction Layer (V0.5.0) initialize success.[Flash] (../packages/EasyFlash-v4.1.0/src/ef_env.c:1818) ENV start address is 0x00000000, size is 8192 bytes.[Flash] EasyFlash V4.1.0 is initialize success.[Flash] You can get the latest version on https://github.com/armink/EasyFlash .[I/OTA] RT-Thread OTA package(V0.2.3) initialize success.[I/OTA] Verify 'wifi_image' partition(fw ver: 1.0, timestamp: 1592464902) success.msh />[I/WWD] wifi initialize done. wiced version 3.3.1[I/WLAN.dev] wlan init success[I/WLAN.lwip] eth device init ok name:w0
```

接下俩进行联网操作，可通过BT进行联网操作，关于BT联网操作参考笔者博文：

[ART-Pi联网](https://blog.bruceou.cn/2020/12/1-art-pi-development-board-boot-use/338/)

这里通过wifi命令来联网。

先对周围的无线网络进行扫描，如果你知道Wii信息可以跳过该步骤：

> wifi scan

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70-165568582551724.png)

连接网络：

> wifi join ssid passwd

![在这里插入图片描述](G:/Typora/user_potograph/20210418130759610.png)

联网成功后会显示IP，接下来可以使用下WiFi相关的的其他命令。

查看WiFi的IP信息 ：

> ifconfig
> ![在这里插入图片描述](G:/Typora/user_potograph/20210418130824514.png)

有WiFi信息不代表联网成功，接下来ping下IP。

> ping ip/域名

![在这里插入图片描述](G:/Typora/user_potograph/20210418130837228.png)

最后再来测试下重启时候回重新联网，使用reboot命令：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70-165568582551825.png)

可以看到重新联网成功。

### 7.3.4 WiFi联网总结

WiFi联网涉及的东西很多，主要有三块，WiFi的驱动与配置，BT的驱动与配置，另外还有flash的驱动与配置，后面两部分是对WiFi联网的优化。简单总结下本文所述的配网流程。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551826.png)

WiFi联网整体流程还是比较简单的，给开发板上电后，首次联网可使用BT和CMD联网，连网后悔保存WiFi信息，再次使用设备时，系统会自动连接flash中的WiFi信息，如果连接成功则结束，如果存储的WiFi失效，可重新使用BT或者CMD联网。当然啦，这里一个可以优化的地方，就是当存储的WiFi失效后，只有通过终端才能看到来联网失败，这里可以通过LED/蜂鸣器/语音等设备提示联网失败，提醒用户重新配网。

关于WiFi的底层驱动已经封装了，这部分内容也很多，主要是协议栈内容多，但是大体的流程和其他驱动一样的。

首先是硬件初始化：

```
int rt_hw_wlan_init(void){    if (init_flag == 1)    {        return RT_EOK;    }    rt_thread_t tid = RT_NULL;    tid = rt_thread_create("wifi_init", wifi_init_thread_entry, RT_NULL, WIFI_INIT_THREAD_STACK_SIZE, WIFI_INIT_THREAD_PRIORITY, 20);    if (tid)    {        rt_thread_startup(tid);    }    else    {        LOG_E("Create wifi initialization thread fail!");        return -RT_ERROR;    }    return RT_EOK;}INIT_APP_EXPORT(rt_hw_wlan_init);
```

硬件初始化后，就是设备注册：

```
rt_err_t rt_wlan_dev_register(struct rt_wlan_device *wlan, const char *name, const struct rt_wlan_dev_ops *ops, rt_uint32_t flag, void *user_data){    rt_err_t err = RT_EOK;    if ((wlan == RT_NULL) || (name == RT_NULL) || (ops == RT_NULL) ||        (flag & RT_WLAN_FLAG_STA_ONLY && flag & RT_WLAN_FLAG_AP_ONLY))    {        LOG_E("F:%s L:%d parameter Wrongful", __FUNCTION__, __LINE__);        return RT_NULL;    }    rt_memset(wlan, 0, sizeof(struct rt_wlan_device));#ifdef RT_USING_DEVICE_OPS    wlan->device.ops = &wlan_ops;#else    wlan->device.init       = _rt_wlan_dev_init;    wlan->device.open       = RT_NULL;    wlan->device.close      = RT_NULL;    wlan->device.read       = RT_NULL;    wlan->device.write      = RT_NULL;    wlan->device.control    = _rt_wlan_dev_control;#endif    wlan->device.user_data  = RT_NULL;    wlan->device.type = RT_Device_Class_NetIf;    wlan->ops = ops;    wlan->user_data  = user_data;    wlan->flags = flag;    err = rt_device_register(&wlan->device, name, RT_DEVICE_FLAG_RDWR);    LOG_D("F:%s L:%d run", __FUNCTION__, __LINE__);    return err;}
```

后面就是进行WIFI 的连接断开，扫描等操作，如果加上优化配置的部分，还需要进行flash的读写配置，BT的配置。

关于WiFi的更多信息，请参看官方手册：

[WiFi](https://www.rt-thread.org/document/site/programming-manual/device/wlan/wlan/)

## 7.4 数据上传到OneNET

### 7.4.1 OneNET简介

OneNET 平台是中国移动基于物联网产业打造的生态平台，具有高并发可用、多协议接入、丰富 API 支持、数据安全存储、快速应用孵化等特点，同时，OneNET 平台还提供全方位支撑，加速用户产品的开发速度。

OneNET平台是一个基于物联网产业特点打造的生态环境，可以适配各种网络环境和协议类型，现在**支持的协议有LWM2M（NB-IOT）、EDP、MQTT、HTTP、MODBUS、JT\T808、TCP透传、RGMP等**。用户可以根据不同的应用场景选择不同的接入协议。

OneNET 软件包是 RT-Thread 系统针对 OneNET 平台连接的适配，通过这个组件包可以让设备在 RT-Thread 上非常方便的连接 OneNet 平台，完成数据的发送、接收、设备的注册和控制等功能。
OneNET 软件包数据的上传和命令的接收是**基于 MQTT** 实现的，OneNET 的初始化其实就是 MQTT 客户端的初始化，初始化完成后，MQTT 客户端会自动连接 OneNET 平台。数据的上传其实就是往特定的 topic 发布消息。当服务器有命令或者响应需要下发时，会将消息推送给设备。

获取数据流、数据点，发布命令则是基于 HTTP Client 实现的，通过 POST 或 GET 将相应的请求发送给 OneNET 平台，OneNET 将对应的数据返回，这样，我们就能在网页上或者手机 APP 上看到设备上传的数据了。

下图是应用显示设备上传数据的流程图：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551827.png)

下图是应用下发命令给设备的流程图

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551828.png)

### 7.4.2准备工作

**在 OneNET 云上注册账号**

设备接入 OneNET 云之前，需要在平台注册用户账号，OneNET 云平台地址：
[https://open.iot.10086.cn](https://open.iot.10086.cn/)

**创建产品**
账号注册登录成功后，点击开发者中心进入开发者中心界面；
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551829.png)

点击创建产品，输入产品基本参数，如下图所示：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551830.png)

产品创建成功之后，可以在开发者中心的公开协议产品中找到刚刚创建的产品，点击产品名，可以看到产品的基础信息（如产品ID，接入协议，创建时间，产品 APIkey 等，后面有用）：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551931.png)

**接入设备**
在开发者中心左侧设备管理中点击添加设备按钮添加设备。

![在这里插入图片描述](G:/Typora/user_potograph/20210418153122271.png)

设备名称我们填入DHT11。鉴权信息是为了区分每一个不同的设备，如果创建了多个设备，要确保每个设备的鉴权信息都不一样，我们这里填入202104150903，填完之后点击接入设备。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551932.png)

**添加 APIkey**
接入设备之后，可以看到设备列表的界面多了一个设备，设备的右边有一些操作设备的按钮，点击查看详情按钮。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551933.png)

此设备的相关信息就都显示出来了，比如：设备 ID、鉴权信息、设备 APIkey，这些信息需要记下，在ENV配置时会用到。

点击按钮添加 APIkey，APIKey 的名称一般和设备相关联，我们这里填入DHT11_APIKey，关联设备默认为我们刚刚创建的设备DHT11。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551934.png)

到此，云端的操作暂时就完成了。

### 7.4.3开启OneNET软件包

打开 RT-Thread Settings，开启 onenet 软件包，勾选onenet 软件包后，进入 onenet 软件包的配置菜单按下图所示配置，里面的信息依据自己的产品和设备的实际情况填写。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582551935.png)

【配置说明】
Enable OneNET sample ：开启 OneNET 示例代码
Enable support MQTT protocol ：开启 MQTT 协议连接 OneNET 支持
Enable OneNET automatic register device ：开启 OneNET 自动注册设备功能
产品ID ：配置云端创建产品时获取的产品ID
主/生产 Apikey ：配置云端创建产品时获取的产品APIKey
![在这里插入图片描述](G:/Typora/user_potograph/20210418153423148.png)

设备ID ：配置云端创建设备时获取的设备ID
身份验证信息 ：配置云端创建产品时 用户自定义的鉴权信息 (每个产品的每个设备唯一)
API密钥 ：配置云端创建设备时获取的 APIkey
![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70-165568582551936.png)

### 7.4.4使用 OneNET

生成工程后，我们可以在工程的 onenet 目录下看到onenet_sample.c文件，该文件是 OneNET 软件包的示例展示，主要是展示用户如何使用 OneNET 软件包上传数据和接收命令。

在使用 OneNET 软件包之前必须要先调用onenet_mqtt_init这个命令进行初始化，初始化完成后设备会自动连接 OneNET 平台。

```
msh />onenet_mqtt_init[D/onenet.mqtt] Enter mqtt_connect_callback![D/[mqtt] ] ipv4 address port: 6002[D/[mqtt] ] HOST = '183.230.40.39'[I/ONENET] RT-Thread OneNET package(V1.0.0) initialize success.msh />[I/[mqtt] ] MQTT server connect success[D/ onenet.mqtt] Enter mqtt_online_callback!
```

![在这里插入图片描述](G:/Typora/user_potograph/20210418153503137.png)

#### 7.4.4.1上传数据

初始化完成后，用户可以调用onenet_upload_cycle这个命令周期性的往云平台上传数据。输入这个命令后，设备会每隔 5s 向数据流 temperature 上传一个随机值。并将上传的数据打印到 shell 窗口。

```
msh />onenet_upload_cycle[D/ onenet.sample] buffer : {"temperature":22}
```

![在这里插入图片描述](G:/Typora/user_potograph/20210418153515524.png)

我们打开 OneNET 平台，在设备列表界面选择刚添加的设备并进入数据流展示页面。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552137.png)

点击temperature数据流左边的小箭头显示数据流信息，我们就可以看到刚刚上传的数据了。
如果用户想往别的数据流发送信息，可以使用以下 API 往云平台上传数据。

```
onenet_mqtt_publish_digit onenet_mqtt_publish_string
```

命令格式如下所示：

```
onenet_mqtt_publish_digit 数据流名称 要上传的数据onenet_mqtt_publish_string 数据流名称 要上传的字符串
```

输入命令后没有返回错误信息就表示上传成功。示例如下:

```
msh />onenet_mqtt_publish_digit test 1msh />onenet_mqtt_publish_string test 1msh />onenet_mqtt_publish_digit test 2msh />onenet_mqtt_publish_string test 1
```

#### 7.4.4.2接收数据

在初始化时，命令响应回调函数默认指向了空，想要接收命令，必须设置命令响应回调函数，在 shell 中输入命令onenet_set_cmd_rsp,就把示例文件里的命令响应回调函数挂载上了，这个响应函数在接收到命令后会把命令打印出来。

```
msh />onenet_set_cmd_rsp
```

我们点击设备列表界面的下发命令按钮。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552138.png)

就可以在 shell 中看到云平台下发的命令了。

![在这里插入图片描述](G:/Typora/user_potograph/20210418153536987.png)

#### 7.4.4.3温湿度数据上传

以上是如何开启和简单使用OneNET，接下来需要将前面获取的温湿度进行上传，修改onenet_sample.c的代码：

```
#include <stdlib.h>#include <onenet.h>#include <wlan_mgnt.h>#include "dht11_thread.h"#define DBG_ENABLE#define DBG_COLOR#define DBG_SECTION_NAME    "onenet.sample"#if ONENET_DEBUG#define DBG_LEVEL           DBG_LOG#else#define DBG_LEVEL           DBG_INFO#endif /* ONENET_DEBUG */#include <rtdbg.h>#ifdef FINSH_USING_MSH#include <finsh.h>extern rt_dht11 dht11;static int onenetOk = -1;/* upload random value to temperature*/static void onenet_upload_entry(void *parameter){    while (1)    {        if(onenetOk == 1)        {            //dht11            //temperature            if (onenet_mqtt_upload_digit("temperature", dht11.temp) < 0)            {                LOG_E("upload has an error, please check the network");                break;            }            else            {                LOG_D("buffer : {\"temperature\":%d}", dht11.temp);            }            rt_thread_delay(rt_tick_from_millisecond(2 * 1000));            //humidity            if (onenet_mqtt_upload_digit("humidity", dht11.humi) < 0)            {                LOG_E("upload has an error, please check the network");                break;            }            else            {                LOG_D("buffer : {\"humidity\":%d}", dht11.humi);            }        }        rt_thread_delay(rt_tick_from_millisecond(5 * 1000));    }}int onenet_upload_cycle(void){    rt_thread_t tid;    tid = rt_thread_create("onenet_send",                           onenet_upload_entry,                           RT_NULL,                           2 * 1024,                           RT_THREAD_PRIORITY_MAX / 3 - 1,                           5);    if (tid)    {        rt_thread_startup(tid);    }    return 0;}//MSH_CMD_EXPORT(onenet_upload_cycle, send data to OneNET cloud cycle);INIT_APP_EXPORT(onenet_upload_cycle);int onenet_publish_digit(int argc, char **argv){    if (argc != 3)    {        LOG_E("onenet_publish [datastream_id]  [value]  - mqtt pulish digit data to OneNET.");        return -1;    }    if (onenet_mqtt_upload_digit(argv[1], atoi(argv[2])) < 0)    {        LOG_E("upload digit data has an error!\n");    }    return 0;}MSH_CMD_EXPORT_ALIAS(onenet_publish_digit, onenet_mqtt_publish_digit, send digit data to onenet cloud);int onenet_publish_string(int argc, char **argv){    if (argc != 3)    {        LOG_E("onenet_publish [datastream_id]  [string]  - mqtt pulish string data to OneNET.");        return -1;    }    if (onenet_mqtt_upload_string(argv[1], argv[2]) < 0)    {        LOG_E("upload string has an error!\n");    }    return 0;}MSH_CMD_EXPORT_ALIAS(onenet_publish_string, onenet_mqtt_publish_string, send string data to onenet cloud);/* onenet mqtt command response callback function */static void onenet_cmd_rsp_cb(uint8_t *recv_data, size_t recv_size, uint8_t **resp_data, size_t *resp_size){    char res_buf[] = { "cmd is received!\n" };    LOG_D("recv data is %.*s\n", recv_size, recv_data);    /* user have to malloc memory for response data */    *resp_data = (uint8_t *) ONENET_MALLOC(strlen(res_buf));    strncpy((char *)*resp_data, res_buf, strlen(res_buf));    *resp_size = strlen(res_buf);}/* set the onenet mqtt command response callback function */int onenet_set_cmd_rsp(int argc, char **argv){    onenet_set_cmd_rsp_cb(onenet_cmd_rsp_cb);    return 0;}MSH_CMD_EXPORT(onenet_set_cmd_rsp, set cmd response function);/* onenet init*/static void onenet_init_entry(void *parameter){    int result = -1;    while(1)    {        if (rt_wlan_is_connected())        {            result = onenet_mqtt_init();            if(result == 0)            {                onenetOk = 1;                break;            }        }        rt_thread_delay(5000);    }}int onenet_init(void){    rt_thread_t tid;    tid = rt_thread_create("onenet_init",                           onenet_init_entry,                           RT_NULL,                           2 * 1024,                           RT_THREAD_PRIORITY_MAX / 2 - 1,                           5);    if (tid)    {        rt_thread_startup(tid);    }    return 0;}INIT_APP_EXPORT(onenet_init);#endif /* FINSH_USING_MSH */
```

主要添加了自动初始化OneNET，修改onenet_upload_entry()的内容，添加DHT11温湿度获取的功能。

值得注意的是，联网后才进行OneNET初始化，还可以优化的地方就是**配置OneNET成自动化，在初始化中加入网络是否连接的判断**，这样就不可不使用终端来打开OneNET，当然啦，上传数据也是如此。另外在**上传每条记录后，需要有一定的延时，不然后面的信息上传不成功**。

打开设备后，现象如下：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552139.png)

当然啦，最后调试成功后关闭打印信息，云端则可看到上传的数据。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552140.png)

### 7.4.5 OneNET创建应用

点击左侧 【应用管理】 然后点击 【创建应用】。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552141.png)

然后输入应用名称并点击 【创建】。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552142.png)

在弹出的窗口中编辑应用，可以拖动左侧元件库中的元件到中间的显示区域

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552143.png)

点击要设置的元件可以在右侧的 属性和样式 窗口改变元件的属性。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552244.png)

最后的效果如下所示：

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552245.png)

## 7.5 总结

最后，我们来总结下这个项目，整个过程很简单，**数据采集，联网，数据上传**，当然数据采集和联网可以并行，当联网成功后就可以进行数据传输。

![在这里插入图片描述](G:/Typora/user_potograph/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMxNjIwMzU=,size_16,color_FFFFFF,t_70%23pic_center-165568582552246.png)

笔者简单起见，这里就在网关上接个传感器来获得环境信息，如果读者朋友有兴趣可以单独将环境监测的部分提取出来，使用单独的板子获得更多的环境信息，或者使用ZigBee/BLE等组网来监测更多的信息和区域。

网关部分，使用ART-Pi一般能满足要求，这里只是简单的传输数据，如果有其他需求，可考虑换用其他网关。

本文使用的云服务是OneNET，当然也可使用其他云服务，阿里云也不错，资料也比较多。

好了，这个项目就到这里了，有兴趣的，一起来玩玩吧。