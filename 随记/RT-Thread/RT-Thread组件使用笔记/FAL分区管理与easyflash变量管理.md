> 参考资料：
>
> * [IOT-OS之RT-Thread（十一）--- FAL分区管理与easyflash变量管理](https://blog.csdn.net/m0_37621078/article/details/102689903)
> * [RT-Thread文档中心：FAL组件](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/fal/fal)
>
> 

## 1.FAL组件

### 1.1什么是FAL

FAL (Flash Abstraction Layer) Flash 抽象层，是对 Flash 及基于 Flash 的分区进行管理、操作的抽象层，对上层统一了 Flash 及 分区操作的 API (框架图如下所示)，并具有以下特性：

- 支持静态可配置的分区表，并可关联多个 Flash 设备；
- 分区表支持 **自动装载** 。避免在多固件项目，分区表被多次定义的问题；
- 代码精简，对操作系统 **无依赖** ，可运行于裸机平台，比如对资源有一定要求的 Bootloader；
- 统一的操作接口。保证了文件系统、OTA、NVM（例如：[EasyFlash](https://github.com/armink-rtt-pkgs/EasyFlash)） 等对 Flash 有一定依赖的组件，底层 Flash 驱动的可重用性；
- 自带基于 Finsh/MSH 的测试命令，可以通过 Shell 按字节寻址的方式操作（读写擦） Flash 或分区，方便开发者进行调试、测试；

![image-20230423162047252](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231620423.png)

通过上图我们也可以清晰明了看到，FAL抽象层向下可以通过Flash硬件进行统一管理，当然也可以使用SFUD框架（串行Flash通用驱动库，这部分RT-Thread官方已完成框架的移植同时提供多个应用历程），而对上也可以使用如DFS、NVM提供的Flash硬件统一访问接口，方便用户更加直接方便对底层flash硬件的访问操作。

注：非易失性存储器 (NVM)：在芯片电源关闭期间保存存储在其中的数据。 因此，它被用于没有磁盘的便携式设备中的内存，以及用于可移动存储卡等用途。 主要类型有：非易失性半导体存储器 (Non-volatile semiconductor memory, NVSM) 将数据存储在浮栅存储单元中，每个单元都由一个浮栅(floating-gate) MOSFET 组成。

关于存储，可以用一张图来解释：

![image-20230423164134689](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231641751.png)

> 来源：[ROM、RAM、FLASH、NVM……一文搞定](https://blog.csdn.net/lianyunyouyou/article/details/118277207)

### 1.2 使用ENV配置FAL

在RT-Thread v4.1.0之前，FAL是作为软件包形式对用户开放使用的，而v4.1.0之后，FAL被RT-Thread官方重新定义为RTT组件的一部分，这样也能更加方便用户的开发。

我们下面正式讲解FAL组件的使用：

首先打开ENV工具，根据以下路径打开FAL使能`RT-Thread Components->[*]FAL: flash abstraction layer`，由于我们后面会用到SFUD，所以这里把`FAL uses SFUD drivers`一并使能，并修改FAL设备名称为`W25Q128`.

![image-20230423164700491](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231647583.png)

完成上述操作后保存退出，并使用`scons --target=mdk5`重新生成MDK5文件并打开

### 1.3 FAL SFUD 移植

为了提供示例，我们选用`W25Q128 spi flash`作为测试模块，并且使用SFUD框架对spi flash设备进行管理和驱动。

由于目前RT-Thread的SFUD已经对`W25Q128 `完成支持，根据官方的使用手册，我们仅需编写`fal_cfg.h`文件完成对`FAL_FLASH_DEV_TABLE`及`FAL_PART_TABLE`的定义即可。文件存放路径：`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\board\ports\fal_cfg.h`

```c
// fal.cfg.h

/*
 * Copyright (c) 2006-2023, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2023-04-21     Wangyuqiang  the first version
 */
#ifndef _FAL_CFG_H_
#define _FAL_CFG_H_

#include <rtthread.h>
#include <board.h>

#ifndef FAL_USING_NOR_FLASH_DEV_NAME
#define NOR_FLASH_DEV_NAME             "norflash0"
#else
#define NOR_FLASH_DEV_NAME              FAL_USING_NOR_FLASH_DEV_NAME
#endif

/* Flash device Configuration */

extern struct fal_flash_dev nor_flash0;

/* flash device table */

#define FAL_FLASH_DEV_TABLE                                          \
{                                                                    \
    &nor_flash0,                                                     \
}

/* Partition Configuration */

#ifdef FAL_PART_HAS_TABLE_CFG

/* partition table */

#define FAL_PART_TABLE                                                                                                  \
{                                                                                                                       \
    {FAL_PART_MAGIC_WROD,  "easyflash", NOR_FLASH_DEV_NAME,                                    0,       512 * 1024, 0}, \
    {FAL_PART_MAGIC_WROD,   "download", NOR_FLASH_DEV_NAME,                           512 * 1024,      1024 * 1024, 0}, \
    {FAL_PART_MAGIC_WROD, "wifi_image", NOR_FLASH_DEV_NAME,                  (512 + 1024) * 1024,       512 * 1024, 0}, \
    {FAL_PART_MAGIC_WROD,       "font", NOR_FLASH_DEV_NAME,            (512 + 1024 + 512) * 1024,  7 * 1024 * 1024, 0}, \
    {FAL_PART_MAGIC_WROD, "filesystem", NOR_FLASH_DEV_NAME, (512 + 1024 + 512 + 7 * 1024) * 1024,  7 * 1024 * 1024, 0}, \
}
#endif /* FAL_PART_HAS_TABLE_CFG */

#endif /* _FAL_CFG_H_ */
```

此时编译的话是找不到该头文件的，需要我们在Keil中设置：

![image-20230423174802203](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231748300.png)

在RTT FAL组件中的SFUD提供的`fal_flash_dev`对象默认的`nor_flash0`参数中，flash大小默认为8M，而`W25Q128`最大最16M，我们可以选择在`.\rt-thread\components\fal\samples\porting\fal_flash_sfud_port.c`文件中对`struct fal_flash_dev nor_flash0`进行修改：

```c
struct fal_flash_dev nor_flash0 =
{
    .name       = FAL_USING_NOR_FLASH_DEV_NAME,
    .addr       = 0,
    .len        = 16 * 1024 * 1024,
    .blk_size   = 4096,
    .ops        = {init, read, write, erase},
    .write_gran = 1
};
```

当然也可以选择不进行修改，根据大佬的原话就是**因为在调用初始化接口函数init后，会从flash设备读取正确的参数更新到nor_flash0表项中，我们在使用FAL组件前都需要调用FAL初始化函数fal_init，其内调用flash设备初始化函数fal_flash_init，最后会调用注册到fal_flash_dev设备表项中的初始化函数device_table[i]->ops.init，所以nor_flash0表项参数会在FAL初始化时被更新。**

同时我们需要开启SFUD框架支持，打开ENV工具，由于SFUD的使用需要指定一个spi设备，这里我选择使用最近移植好的软件spi，路径`Hardware Drivers Config->On-chip Peripheral Drivers->[*] Enable soft SPI BUS-> [*] Enable soft SPI1 BUS (software simulation)`，这里我的测试开发板是恩智浦的LPC55S69-EVK，并且这款bsp的软件模拟spi由我本人对接，关于这部分的软件spi引脚定义可以选用默认即可，当然也可以使用自定义引脚，记住不要与其他引脚产生冲突。

![image-20230423171229953](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231712081.png)

此时我们回到ENV主界面，进入`RT-Thread Components->Device Drivers->Using Serial Flash Universal Driver`，此时我们才可以看到SFUD选项出现（如果没有使能spi是没法看到的），使能后保持默认即可

![image-20230423171646352](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231716493.png)

到这里，ENV的配置暂时告一段落！

### 1.4 FAL SFUD 测试用例

为了验证`W25Q128`及软件模拟spi在SFUD框架上是否能够成功运行，我们在`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\board\ports\`下新建一个`soft_spi_flash_init.c`文件，代码如下

```c
/*
 * Copyright (c) 2006-2023, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2023-04-21     Wangyuqiang  the first version
 */

#include <rtthread.h>
#include "spi_flash.h"
#include "spi_flash_sfud.h"
#include "drv_soft_spi.h"
#include "drv_pin.h"
#include "rtconfig.h"

#define cs_pin	GET_PINS(1,9)

static int rt_soft_spi_flash_init(void)
{
	int result = -1;

    result = rt_hw_softspi_device_attach("sspi1", "sspi10", cs_pin);
	rt_kprintf("value is %d\n",result);
	
	if(result == RT_EOK)
	{
		rt_kprintf("rt_hw_softspi_device_attach successful!\n");
	}

    if (RT_NULL == rt_sfud_flash_probe("W25Q128", "sspi10"))
    {
        return -RT_ERROR;
    }

    return RT_EOK;
}
INIT_COMPONENT_EXPORT(rt_soft_spi_flash_init);
```

这里我们需要指定一个片选引脚，我暂时使用了`sspi2`的SCK引脚作为片选，这里注意不要同时打开`sspi1`和`sspi2`，后续我会专门上传一个通用GPIO作为片选引脚，到时候就不会产生问题了。然后软件spi设备的挂载使用的是`sspi1 bus`及`sspi10 device`，并且挂载flash设备到`sspi10`。

另外我们在`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\board\ports\`下新建`fal_sample.c`文件，并编写测试代码：

```c
//fal_sample.c

/*
 * Copyright (c) 2006-2023, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2023-04-21     Wangyuqiang  the first version
 */
 
#include "rtthread.h"
#include "rtdevice.h"
#include "board.h"
#include "fal.h"

#define BUF_SIZE 1024

static int fal_test(const char *partiton_name)
{
    int ret;
    int i, j, len;
    uint8_t buf[BUF_SIZE];
    const struct fal_flash_dev *flash_dev = RT_NULL;
    const struct fal_partition *partition = RT_NULL;

    if (!partiton_name)
    {
        rt_kprintf("Input param partition name is null!\n");
        return -1;
    }

    partition = fal_partition_find(partiton_name);
    if (partition == RT_NULL)
    {
        rt_kprintf("Find partition (%s) failed!\n", partiton_name);
        ret = -1;
        return ret;
    }

    flash_dev = fal_flash_device_find(partition->flash_name);
    if (flash_dev == RT_NULL)
    {
        rt_kprintf("Find flash device (%s) failed!\n", partition->flash_name);
        ret = -1;
        return ret;
    }

    rt_kprintf("Flash device : %s   "
               "Flash size : %dK   \n"
               "Partition : %s   "
               "Partition size: %dK\n", 
                partition->flash_name, 
                flash_dev->len/1024,
                partition->name,
                partition->len/1024);

    /* erase all partition */
    ret = fal_partition_erase_all(partition);
    if (ret < 0)
    {
        rt_kprintf("Partition (%s) erase failed!\n", partition->name);
        ret = -1;
        return ret;
    }
    rt_kprintf("Erase (%s) partition finish!\n", partiton_name);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0xFF)
            {
                rt_kprintf("The erase operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }
        i += len;
    }

    /* write 0x00 to the specified partition */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_write(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) write failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        i += len;
    }
    rt_kprintf("Write (%s) partition finish! Write size %d(%dK).\n", partiton_name, i, i/1024);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0xFF, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0x00)
            {
                rt_kprintf("The write operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }

        i += len;
    }

    ret = 0;
    return ret;
}

static void fal_sample(void)
{
    /* 1- init */
    fal_init();

    if (fal_test("font") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "font");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "font");
    }

    if (fal_test("download") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "download");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "download");
    }
}
MSH_CMD_EXPORT(fal_sample, fal sample);
```

### 1.5 测试结果

到这里就可以进行编译下载了，成功后的截图如下：

![image-20230423172831146](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231728293.png)

## 2.DFS文件系统

### 2.1 什么是DFS

DFS 是 RT-Thread 提供的虚拟文件系统组件，全称为 Device File System，即设备虚拟文件系统，文件系统的名称使用类似 UNIX 文件、文件夹的风格，目录结构如下图所示： 

![image-20230423173347702](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231733906.png)

在 RT-Thread DFS 中，文件系统有统一的根目录，使用 `/` 来表示。而在根目录下的 f1.bin 文件则使用 `/f1.bin` 来表示，2018 目录下的 `f1.bin` 目录则使用 `/data/2018/f1.bin` 来表示。即目录的分割符号是 `/`，这与 UNIX/Linux 完全相同，与 Windows 则不相同（Windows 操作系统上使用 `\` 来作为目录的分割符）。

### 2.2 DFS架构

RT-Thread DFS 组件的主要功能特点有：

- 为应用程序提供统一的 POSIX 文件和目录操作接口：read、write、poll/select 等。
- 支持多种类型的文件系统，如 FatFS、RomFS、DevFS 等，并提供普通文件、设备文件、网络文件描述符的管理。
- 支持多种类型的存储设备，如 SD Card、SPI Flash、Nand Flash 等。

DFS 的层次架构如下图所示，主要分为 POSIX 接口层、虚拟文件系统层和设备抽象层。

![image-20230423173515014](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231735074.png)

### 2.3 使用ENV配置DFS

打开ENV，进入路径`RT-Thread Components → DFS: device virtual file system`，使能`[*] DFS: device virtual file system`

![image-20230423174113310](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231741428.png)

由于DFS使用的是POSIX接口，而dfs_posix.h已经在新版本中被移除了，如果想要兼容老版本，可以在menuconfig中使能`RT-Thread Components->[*] Support legacy version for compatibility`

![image-20230423180859035](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231808158.png)

由于elmfat文件系统默认最大扇区大小为512，但我们使用的flash模块`W25Q128`的Flash扇区大小为4096，为了将elmfat文件系统挂载到W25Q128上，这里的`Maximum sector size`需要和W25Q128扇区大小保持一致，修改为4096，路径：`RT-Thread Components → DFS: device virtual file system → [*] Enable elm-chan fatfs / elm-chan's FatFs, Generic FAT Filesystem Module`

![image-20230423181825139](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231818347.png)

保存退出后使用`scons --target=mdk5`生成MDK5工程。

### 2.4 DFS挂载到FAL分区测试

这里增加FAL flash抽象层，我们将elmfat文件系统挂载到W25Q128 flash设备的filesystem分区上，由于FAL管理的filesystem分区不是块设备，需要先使用FAL分区转BLK设备接口函数将filesystem分区转换为块设备，然后再将DFS elmfat文件系统挂载到filesystem块设备上。

我们接着修改`fal_sample.c`文件，修改后代码：

```c
/*
 * Copyright (c) 2006-2023, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2023-04-21     Wangyuqiang  the first version
 */
 
#include "rtthread.h"
#include "rtdevice.h"
#include "board.h"
#include "fal.h"

#include <dfs_posix.h>

#define FS_PARTITION_NAME  "filesystem"

#define BUF_SIZE 1024

static int fal_test(const char *partiton_name)
{
    int ret;
    int i, j, len;
    uint8_t buf[BUF_SIZE];
    const struct fal_flash_dev *flash_dev = RT_NULL;
    const struct fal_partition *partition = RT_NULL;

    if (!partiton_name)
    {
        rt_kprintf("Input param partition name is null!\n");
        return -1;
    }

    partition = fal_partition_find(partiton_name);
    if (partition == RT_NULL)
    {
        rt_kprintf("Find partition (%s) failed!\n", partiton_name);
        ret = -1;
        return ret;
    }

    flash_dev = fal_flash_device_find(partition->flash_name);
    if (flash_dev == RT_NULL)
    {
        rt_kprintf("Find flash device (%s) failed!\n", partition->flash_name);
        ret = -1;
        return ret;
    }

    rt_kprintf("Flash device : %s   "
               "Flash size : %dK   \n"
               "Partition : %s   "
               "Partition size: %dK\n", 
                partition->flash_name, 
                flash_dev->len/1024,
                partition->name,
                partition->len/1024);

    /* erase all partition */
    ret = fal_partition_erase_all(partition);
    if (ret < 0)
    {
        rt_kprintf("Partition (%s) erase failed!\n", partition->name);
        ret = -1;
        return ret;
    }
    rt_kprintf("Erase (%s) partition finish!\n", partiton_name);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0xFF)
            {
                rt_kprintf("The erase operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }
        i += len;
    }

    /* write 0x00 to the specified partition */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_write(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) write failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        i += len;
    }
    rt_kprintf("Write (%s) partition finish! Write size %d(%dK).\n", partiton_name, i, i/1024);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0xFF, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0x00)
            {
                rt_kprintf("The write operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }

        i += len;
    }

    ret = 0;
    return ret;
}

static void fal_sample(void)
{
    /* 1- init */
    fal_init();

    if (fal_test("font") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "font");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "font");
    }

    if (fal_test("download") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "download");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "download");
    }
}
MSH_CMD_EXPORT(fal_sample, fal sample);

static void fal_elmfat_sample(void)
{
    int fd, size;
    struct statfs elm_stat;
    struct fal_blk_device *blk_dev;
    char str[] = "elmfat mount to W25Q flash.", buf[80];

    /* fal init */
    fal_init();

    /* create block device */
    blk_dev = (struct fal_blk_device *)fal_blk_device_create(FS_PARTITION_NAME);
    if(blk_dev == RT_NULL)
        rt_kprintf("Can't create a block device on '%s' partition.\n", FS_PARTITION_NAME);
    else
        rt_kprintf("Create a block device on the %s partition of flash successful.\n", FS_PARTITION_NAME);

    /* make a elmfat format filesystem */
    if(dfs_mkfs("elm", FS_PARTITION_NAME) == 0)
        rt_kprintf("make elmfat filesystem success.\n");

    /* mount elmfat file system to FS_PARTITION_NAME */
    if(dfs_mount(FS_PARTITION_NAME, "/", "elm", 0, 0) == 0)
        rt_kprintf("elmfat filesystem mount success.\n");

    /* Get elmfat file system statistics */
    if(statfs("/", &elm_stat) == 0)
        rt_kprintf("elmfat filesystem block size: %d, total blocks: %d, free blocks: %d.\n", 
                    elm_stat.f_bsize, elm_stat.f_blocks, elm_stat.f_bfree);

    if(mkdir("/user", 0x777) == 0)
        rt_kprintf("make a directory: '/user'.\n");

    rt_kprintf("Write string '%s' to /user/test.txt.\n", str);

    /* Open the file in create and read-write mode, create the file if it does not exist*/
    fd = open("/user/test.txt", O_WRONLY | O_CREAT);
    if (fd >= 0)
    {
        if(write(fd, str, sizeof(str)) == sizeof(str))
            rt_kprintf("Write data done.\n");

        close(fd);   
    }

    /* Open file in read-only mode */
    fd = open("/user/test.txt", O_RDONLY);
    if (fd >= 0)
    {
        size = read(fd, buf, sizeof(buf));

        close(fd);

        if(size == sizeof(str))
            rt_kprintf("Read data from file test.txt(size: %d): %s \n", size, buf);
    }
}
MSH_CMD_EXPORT_ALIAS(fal_elmfat_sample, fal_elmfat,fal elmfat sample);
```

### 2.5 测试结果

测试结果如下：

![image-20230423182204922](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231822040.png)

## 3.Easyflash移植到FAL分区

### 3.1 简述EasyFlash

关于EasyFlash的来源我们已经讲过了，此处不再赘述。[EasyFlash](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Farmink%2FEasyFlash)是一款开源的轻量级嵌入式Flash存储器库，方便开发者更加轻松的实现基于Flash存储器的常见应用开发。非常适合智能家居、可穿戴、工控、医疗、物联网等需要断电存储功能的产品，资源占用极低，支持各种 MCU 片上存储器。

EasyFlash不仅能够实现对产品的 **设定参数** 或 **运行日志** 等信息的掉电保存功能，还封装了简洁的 **增加、删除、修改及查询** 方法， 降低了开发者对产品参数的处理难度，也保证了产品在后期升级时拥有更好的扩展性。让Flash变为NoSQL（非关系型数据库）模型的小型键值（Key-Value）存储数据库。

### 3.2EasyFlash软件包使用

打开ENV进入路径：`RT-Thread online packages → tools packages → EasyFlash: Lightweight embedded flash memory library.`，选择软件包版本为最新版。

![image-20230423183612019](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231836146.png)

配置后退出ENV，同时使用`pkgs --update`下载软件包，然后再使用`scons --target=mdk5`重新生成MDK5文件

### 3.3 移植easyflash

下载完easyflash软件包后，我们复制`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\packages\EasyFlash-latest\ports\ef_fal_port.c`到目录`.\rt-thread\bsp\lpc55sxx\lpc55s69_nxp_evk\board\ports\easyflash\ef_fal_port.c`，双击打开该文件，完成以下修改：

* ```c
  // 修改 FAL_EF_PART_NAME 为 easyflash
  #define FAL_EF_PART_NAME               "easyflash"
  ```

* ```c
  // 修改环境变量内容为 {"boot_times", "0"}，这里我们先只设置一个开机次数
  static const ef_env default_env_set[] = {
          {"boot_times", "0"},
  };
  ```

### 3.4 编写Easyflash测试用例

```c
/*
 * Copyright (c) 2006-2023, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2023-04-21     Wangyuqiang  the first version
 */
 
#include "rtthread.h"
#include "rtdevice.h"
#include "board.h"
#include "fal.h"

#include <dfs_posix.h>

#include "easyflash.h"
#include <stdlib.h>

#define FS_PARTITION_NAME  "filesystem"

#define BUF_SIZE 1024

static int fal_test(const char *partiton_name)
{
    int ret;
    int i, j, len;
    uint8_t buf[BUF_SIZE];
    const struct fal_flash_dev *flash_dev = RT_NULL;
    const struct fal_partition *partition = RT_NULL;

    if (!partiton_name)
    {
        rt_kprintf("Input param partition name is null!\n");
        return -1;
    }

    partition = fal_partition_find(partiton_name);
    if (partition == RT_NULL)
    {
        rt_kprintf("Find partition (%s) failed!\n", partiton_name);
        ret = -1;
        return ret;
    }

    flash_dev = fal_flash_device_find(partition->flash_name);
    if (flash_dev == RT_NULL)
    {
        rt_kprintf("Find flash device (%s) failed!\n", partition->flash_name);
        ret = -1;
        return ret;
    }

    rt_kprintf("Flash device : %s   "
               "Flash size : %dK   \n"
               "Partition : %s   "
               "Partition size: %dK\n", 
                partition->flash_name, 
                flash_dev->len/1024,
                partition->name,
                partition->len/1024);

    /* erase all partition */
    ret = fal_partition_erase_all(partition);
    if (ret < 0)
    {
        rt_kprintf("Partition (%s) erase failed!\n", partition->name);
        ret = -1;
        return ret;
    }
    rt_kprintf("Erase (%s) partition finish!\n", partiton_name);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0xFF)
            {
                rt_kprintf("The erase operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }
        i += len;
    }

    /* write 0x00 to the specified partition */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0x00, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_write(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) write failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        i += len;
    }
    rt_kprintf("Write (%s) partition finish! Write size %d(%dK).\n", partiton_name, i, i/1024);

    /* read the specified partition and check data */
    for (i = 0; i < partition->len;)
    {
        rt_memset(buf, 0xFF, BUF_SIZE);

        len = (partition->len - i) > BUF_SIZE ? BUF_SIZE : (partition->len - i);

        ret = fal_partition_read(partition, i, buf, len);
        if (ret < 0)
        {
            rt_kprintf("Partition (%s) read failed!\n", partition->name);
            ret = -1;
            return ret;
        }

        for(j = 0; j < len; j++)
        {
            if (buf[j] != 0x00)
            {
                rt_kprintf("The write operation did not really succeed!\n");
                ret = -1;
                return ret;
            }
        }

        i += len;
    }

    ret = 0;
    return ret;
}

static void fal_sample(void)
{
    /* 1- init */
    fal_init();

    if (fal_test("font") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "font");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "font");
    }

    if (fal_test("download") == 0)
    {
        rt_kprintf("Fal partition (%s) test success!\n", "download");
    }
    else
    {
        rt_kprintf("Fal partition (%s) test failed!\n", "download");
    }
}
MSH_CMD_EXPORT(fal_sample, fal sample);

static void fal_elmfat_sample(void)
{
    int fd, size;
    struct statfs elm_stat;
    struct fal_blk_device *blk_dev;
    char str[] = "elmfat mount to W25Q flash.", buf[80];

    /* fal init */
    fal_init();

    /* create block device */
    blk_dev = (struct fal_blk_device *)fal_blk_device_create(FS_PARTITION_NAME);
    if(blk_dev == RT_NULL)
        rt_kprintf("Can't create a block device on '%s' partition.\n", FS_PARTITION_NAME);
    else
        rt_kprintf("Create a block device on the %s partition of flash successful.\n", FS_PARTITION_NAME);

    /* make a elmfat format filesystem */
    if(dfs_mkfs("elm", FS_PARTITION_NAME) == 0)
        rt_kprintf("make elmfat filesystem success.\n");

    /* mount elmfat file system to FS_PARTITION_NAME */
    if(dfs_mount(FS_PARTITION_NAME, "/", "elm", 0, 0) == 0)
        rt_kprintf("elmfat filesystem mount success.\n");

    /* Get elmfat file system statistics */
    if(statfs("/", &elm_stat) == 0)
        rt_kprintf("elmfat filesystem block size: %d, total blocks: %d, free blocks: %d.\n", 
                    elm_stat.f_bsize, elm_stat.f_blocks, elm_stat.f_bfree);

    if(mkdir("/user", 0x777) == 0)
        rt_kprintf("make a directory: '/user'.\n");

    rt_kprintf("Write string '%s' to /user/test.txt.\n", str);

    /* Open the file in create and read-write mode, create the file if it does not exist*/
    fd = open("/user/test.txt", O_WRONLY | O_CREAT);
    if (fd >= 0)
    {
        if(write(fd, str, sizeof(str)) == sizeof(str))
            rt_kprintf("Write data done.\n");

        close(fd);   
    }

    /* Open file in read-only mode */
    fd = open("/user/test.txt", O_RDONLY);
    if (fd >= 0)
    {
        size = read(fd, buf, sizeof(buf));

        close(fd);

        if(size == sizeof(str))
            rt_kprintf("Read data from file test.txt(size: %d): %s \n", size, buf);
    }
}
MSH_CMD_EXPORT_ALIAS(fal_elmfat_sample, fal_elmfat,fal elmfat sample);

static void easyflash_sample(void)
{
    /* fal init */
    fal_init();

    /* easyflash init */
    if(easyflash_init() == EF_NO_ERR)
    {
        uint32_t i_boot_times = NULL;
        char *c_old_boot_times, c_new_boot_times[11] = {0};

        /* get the boot count number from Env */
        c_old_boot_times = ef_get_env("boot_times");
        /* get the boot count number failed */
        if (c_old_boot_times == RT_NULL)
            c_old_boot_times[0] = '0';

        i_boot_times = atol(c_old_boot_times);
        /* boot count +1 */
        i_boot_times ++;
        rt_kprintf("===============================================\n");
        rt_kprintf("The system now boot %d times\n", i_boot_times);
        rt_kprintf("===============================================\n");
        /* interger to string */
        sprintf(c_new_boot_times, "%d", i_boot_times);
        /* set and store the boot count number to Env */
        ef_set_env("boot_times", c_new_boot_times);
        ef_save_env();
    }
}
MSH_CMD_EXPORT(easyflash_sample, easyflash sample);
```

### 3.5 测试结果

打开串口助手，输入命令：
```c
msh />easyflash_sample
```

第一次命令调用：

![image-20230423185619472](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231856640.png)

第二次RESET开发板后调用：

![image-20230423185703046](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304231857243.png)

## 4.结语

至此本博客就到此结束，经历从移植软件模拟spi框架到LPC55S69，到移植过程中遇到不断的问题，到最后解决所有问题并提供应用示例，完成开发日记、开发笔记及应用教学，这个过程确实使我受益良多，其中感受最深的就是当然也更加感谢的是一些前辈们的指点迷津和博文记录，就目前国内嵌入式这个领域，相关开发经验相比较其他计算机行业确实有些不够包容和开放，也希望未来的朋友们能够怀揣着一颗求知及授学之心，共同建设好这个领域！

## 5.联系

- [Email :yifang.wangyq@foxmail.com](mailto:yifang.wangyq@foxmail.com)
- [Github Address :https://github.com/kurisaW](https://github.com/kurisaW)
- [My Website :https://kurisaw.github.io](https://kurisaw.github.io/)