## segger_rtt 瑞萨RA6M4提交PR

---

学习总结：

#### day:2022/8/27

* 潘多拉STM32开发板关于segger_rtt的使用需要使用ST-LINK转J-LINK（参考文章：[ST-LINK ON-Board](https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/)）

* 瑞萨开发板使用segger_rtt软件包时需要完成以下操作（初始化工作尚未完成）

![image-20220827231711274](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202208272317326.png)

```c
#define RT_CONSOLE_DEVICE_NAME "jlinkRtt"
rt_hw_jlink_rtt_init();
rt_console_set_device(RT_CONSOLE_DEVICE_NAME);
```



文献参考：

1、[Segger_rtt(RT-Thread官方文档)](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/application-note/debug/seggerRTT/segger_rtt)

2、[segger_rtt软件包使用(Thmos))](https://github.com/supperthomas/RTTHREAD_SEGGER_TOOL)

3、[supperthomas-wiki](https://supperthomas-wiki.readthedocs.io/en/latest/DEBUG/01_SEGGER_RTT/SEGGER_RTT.html#id1)

---

#### day:2022/8/30

rt_components_init()



---

#### day:2022/9/1

![image-20220901225839310](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209012258504.png)

![image-20220901225509367](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209012255484.png)

---

#### day:2022/9/2

修改原因：

V2版本的串口框架（以及驱动）主要改动点：
取消了硬件工作模式的判断，硬件工作模式由驱动层支持，使得框架层与 硬件工作模式 无关；
统一操作接口，应用层不再关心 硬件工作模式，
统一使用 阻塞/非阻塞 操作模式，不会因为模式的变更导致应用层逻辑代码行为不一致。
增加发送缓冲区功能，保证应用层数据的完整性，从而解决丢包率；
每一路的串口都有独立的发送缓冲区和接收缓冲区的宏定义，取代之前的所有串口默认使用RT_SERIAL_RB_BUFSZ这一个宏定义。
完善工作模式，分工明确，轮询、中断、DMA都能按照正确的工作模式执行。
原文链接：https://blog.csdn.net/rtthreadiotos/article/details/119178211

```
#define RT_SERIAL_CONFIG_DEFAULT           \
{                                          \
    BAUD_RATE_115200, /* 115200 bits/s */  \
    DATA_BITS_8,      /* 8 databits */     \
    STOP_BITS_1,      /* 1 stopbit */      \
    PARITY_NONE,      /* No parity  */     \
    BIT_ORDER_LSB,    /* LSB first sent */ \
    NRZ_NORMAL,       /* Normal mode */    \
    RT_SERIAL_RB_BUFSZ, /* Buffer size */  \
    0                                      \
}
```

![image-20220902231641913](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209022316184.png)

在rt_config.h中添加代码:

```
#define RT_SERIAL_RB_BUFSZ              64
#define RT_SERIAL_RX_MINBUFSZ 64
#define RT_SERIAL_TX_MINBUFSZ 64
```



![image-20220902231751799](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209022317874.png)



分析：在serial_v2.c中修改`rt_hw_serial_register`对rtt部分的对接

rt_hw_usart_init

---

#### 2022/9/3

segger_rtt对浮点数输出的支持(设置支持浮点数后五位，可自行按需配置)

打开SEGGER_RTT_printf.c

```c
      /*******************Segger RTT support for floating point numbers*************************/
      case 'f':
      case 'F':
      {
       float fv;
       fv = (float)va_arg(*pParamList, double);    //取出输入的浮点数值

       v = (int)fv;                                //取整数部分

       _PrintInt(&BufferDesc, v, 10u, NumDigits, FieldWidth, FormatFlags); //显示整数，支持负数
      _StoreChar(&BufferDesc, '.');                                        //显示小数点

       v = abs((int)(fv * 100000));
       v = v % 100000;
      _PrintInt(&BufferDesc, v, 10u, 5, FieldWidth, FormatFlags);          //显示小数点后五位
      }
      break;
      /*******************************************/
```

![image-20220903162416198](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209031624502.png)

测试：

![image-20220904092201827](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209040922152.png)

---

#### day:2022/9/4

查找segger_rtt在RT_Thread中finSH移植

![image-20220904225204769](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209042252107.png)

资料参考：

[剖析RT-Thread中console与finsh组件实现](https://blog.csdn.net/qwe5959798/article/details/115507156?spm=1001.2014.3001.5506)

---

#### day:2022/9/6

重点知识：

![image-20220906094517169](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209060945563.png)

>FinSH的接收是接收中断模式，FinSH的发送是轮询模式。



由上文可以得出一些结论：

FinSH 用串口外设作数据流时，其发送模式为轮询模式，接收模式为中断模式。

接收靠的是信号量做数据通信，且每次进接收中断时，只能接收单字节数据。

接收端当没有接收到数据时，FinSH将通过信号量挂起自身，因此属于非阻塞接收模式。

发送端 （也包括rt_kprintf）为轮询发送，因此属于阻塞发送模式。（这里延伸一下，rt_kprintf能在中断环境下使用么？答案是可以的，因为该函数没有影响中断环境状态的变化，亦没有等待信号量或者挂起线程等操作。不过还是尽量少在中断环境下打印数据，因为中断执行时间要尽可能短，加入打印将会使得中断执行时间超过预期，从而出现不安全的意外风险）





![image-20220906165123731](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209061651778.png)

![image-20220906164649731](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209061646978.png)

[serial v2 原理剖析及漏洞分析](https://thewon.gitee.io/2022/0209/rtt-serial-v2/)

---

#### day：2022/9/9

![image-20220909120257287](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209091202623.png)



[浅谈嵌入式MCU软件开发之SEGGER实时传输(RTT)的移植和printf()重定向应用](https://www.shangyexinzhi.com/article/2838627.html)

[RT-Thread驱动篇之串口驱动框架剖析及性能提升](https://www.eet-china.com/mp/a105459.html)

[在rtt-console和shell中使用JLINK RTT来代替串口调试设备](https://club.rt-thread.org/ask/question/20590405e907434b.html)

---

#### day:2022/9/17

![image-20220917225947456](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209172259531.png)

![image-20220917231005218](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209172310355.png)

---

#### day:2022/9/19

今日进度：

![image-20220919201820008](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209192018383.png)

当键盘向串口终端输入数据时，触发`rt_hw_serial_isr`中断

---

#### 参考资料

1、[STM32 jlink RTT使用详解](https://bbs.21ic.com/icview-3118746-1-1.html)

2、[CPK-RA6M4 log打印使用segger_rtt问题](https://japan.renesasrulz.com/rulz-chinese/forums-groups/mcu-mpu/ra-mcu-fsp/f/forum/8147/cpk-ra6m4/42111#42111)

3、[HardFault 之 INVSTAE 错误定位](https://www.eet-china.com/mp/a85698.html)

4、[UsageFault On Cortex-M4](https://zhuanlan.zhihu.com/p/250853927)