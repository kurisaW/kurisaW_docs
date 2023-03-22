secureCRT启动开发板，在

![image-20220428105405507](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220428105405507.png)

c语言是面向过程的，但是Linux是面向对象的，如何在面向对象中表示C语言，需要用到`函数指针`

* 打印调试：printk



* ubuntu内核文件

```
查看Ubuntu版本号：uname -r

cd /lib/modules/3.13.0-32-generic/build
```



![image-20220428113738058](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220428113738058.png)

在共享文件夹下创建一个driver文件夹，里面创建以下两个文件：

```c
module_test.c

#include <linux/module.h>
#include <linux/init.h>

static int __init chrdev_init(void)
{
	printk(KERN_INFO,"chrdev_init Hello World.\n");
	return 0;
}

static void __exit chrdev_exit(void)
{
	printk(KERN_INFO,"chrdev exit Hello World.\n");
}

module_init(chrdev_init);
module_exit(chrdev_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("WYQ");
MODULE_DESCRIPTION("module_test");
```

```makefile
Makefile

KERN_VER = $(shell uname -r)
KERN_DIR = /lib/modules/$(KERN_VER)/build

obj-m += module_test.o

all:
	make -C $(KERN_DIR) M=`pwd` modules
	
.PHONY:clean
clean:
	make -C $(KERN_DIR) M=`pwd` modules clean
```

> 执行`make`

会发现文件夹下原本的两个文件另外还多出了其他文件：

![image-20220429115319449](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220429115319449.png)

> 查看模型信息：
> `modinfo module_test.ko`
>`lsmod`   //查看Linux系统中装载的所有模型

![image-20220429120237032](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220429120237032.png)

> 安装module_test.ko这个模块
> 执行`insmod module_test.ko`

这里在Ubuntu中有个权限问题会无法查看
输入命令：`dmesg`
然后`lsmod`就可以查看了

卸载模型
输入命令：`rmmod module_test.ko`



## 虚拟驱动创建流程

安装模块
`lsmod module_test.ko`
创建设备文件
`mknod /dev/test c 250 0`
查看设备状态
`lsmod module_test.ko`
查看设备注册信息(分为字符设备和块设备)
`cat /proc/devices`



知识补充:
```c
#include<stdio.h>
int main(void)
{
	int i;
	static int j;
	printf(i);
	printf(j);
}
// 注意：这里如果没有指定i值，则打印出来的是随机值
// 如果定义一个静态变量而没有赋值，则打印默认为0

```









首先进入x210_bsp/kernel 

make menuconfig

![image-20220507205438583](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507205438583.png)

![image-20220507205504546](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507205504546.png)

![image-20220507204734661](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507204734661.png)

make -j4 

 cp arch/arm/boot/zImage /tftpboot/ -f

重启开发板查看开发板设备

ls /sys/devices/platform/

![image-20220507205918814](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507205918814.png)

![image-20220507210713105](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507210713105.png)

cd sys/class/leds

led_test_4编写完成后

编译不报错即可

cd /root/x210_bsp/kernel/drivers/leds/

cp /mnt/hgfs/Myshare/driver/led_test_4/leds-s5pv210.c ./



vi Makefile->

`obj-$(CONFIG_LEDS_S5PV210)              += leds-s5pv210.o`

![image-20220507224506063](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507224506063.png)



vi Kconfig更改依赖（添加以下文件）

`config LEDS_S5PV210
        tristate "LED Support for S5PV210"
        help
          This option enables support for on-chip LED drivers found on Marvell
          Semiconductor 88PM8606 PMIC.`



![image-20220507225048305](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507225048305.png)





进入到x210_bsp/kernel

执行make menuconfig

可以发现生成了新的配置（Device Drivers-> LED_Support），使能这个

![image-20220507225324648](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220507225324648.png)

执行make编译

` cp arch/arm/boot/zImage /tftpboot/ -f`



secureCRT：

cd sys/class/leds

进入LED1,执行

echo 1 > brightness   // 灯亮

echo 0 > brightness //灯灭