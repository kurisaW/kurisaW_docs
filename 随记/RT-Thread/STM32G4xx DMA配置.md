## 前言

前段时间在使用RTT的STM32G474RE BSP工程的时候发现，串口DMA模式无法使用，查找具体原因，发现是DMA配置有问题，下面就作为一个开发日志进行记录。

## STM32G4xx DMA

我们查看芯片手册，注意看DMA的使用描述，这里有一句话，大致意思是DMA的使用可以分两种情况：

* DMA通道可以与外围设备的DMA请求信号相关联
* 在内存之间的传输时，DMA通道可以与存储器到存储器传输的软件触发器相关联（这部分是由软件实现）

![image-20230725083928807](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202307250839178.png)

虽然STM32G4也是Cortex-M4系列内核，但是与我之前使用的F4却并不一样，在F4的DMA描述中是以STREAM对应通道的，而G4却是统一以CHANNEL来代替。

STM32G4 DMAMUX的作用也就是将不同的外设请求信号与DMA通道进行关联，从而实现外设与DMA之间的数据传输。并且G4的DMAMUX最多支持115个DMA请求可以映射到任意一个DMA通道上，相较于F4的固定通道则有了很大的灵活性。

![image-20230725085545622](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202307250855137.png)

## RT-Thread DMA配置

言归正传，我们这时候再来看下G474 bsp的DMA配置