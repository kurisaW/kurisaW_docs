**配置RT-Thread Settings**

硬件勾选“Media-IO”，暂时不选择touch和audio功能，要注意不能选择enable LCD，这个选项是对应于LDTC驱动方式的显示屏的。这个多媒体扩展版上用的是MCU屏，自带驱动的IC的，是SPI通讯口，接在SPI2上，所以勾选了enable spi2

![image-20220611190327682](G:/Typora/user_potograph/image-20220611190327682.png)

![image-20220611190357594](G:/Typora/user_potograph/image-20220611190357594.png)

保存退出，等待生成配置。会自动在项目中添加ILI9488的驱动文件。

现在将固件下载进去开发板看看什么效果。
屏幕白屏，而且终端打印错误信息，这是什么情况？

![image.png](G:/Typora/user_potograph/2f3c3cb44b2e963028b7ac48932cd2ec.png)

**添加SPI2初始化代码段**
查看了一下stm32h7xx_hal_msp.c文件，原来没有针对SPI2的初始化代码段。把下面这个代码段补上去，重新编译下载。

![image-20220611190618355](G:/Typora/user_potograph/image-20220611190618355.png)

```c
else if(hspi->Instance==SPI2)
  {
  /* USER CODE BEGIN SPI2_MspInit 0 */

  /* USER CODE END SPI2_MspInit 0 */
  /* Peripheral clock enable */
    __HAL_RCC_SPI2_CLK_ENABLE();

    __HAL_RCC_GPIOE_CLK_ENABLE();
    /**SPI2 GPIO Configuration
    PI1     ------> SPI2_SCK
    PI2     ------> SPI2_MISO
    PI3     ------> SPI2_MOSI
    */
    GPIO_InitStruct.Pin = GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3;
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
    GPIO_InitStruct.Alternate = GPIO_AF5_SPI2;
    HAL_GPIO_Init(GPIOI, &GPIO_InitStruct);

    /* SPI2 interrpt Init */
    HAL_NVIC_SetPriority(SPI2_IRQn, 0, 0);
    HAL_NVIC_EnableIRQ(SPI2_IRQn);
    /* USER CODE BEGIN SPI4_MspInit 1 */

    /* USER CODE END SPI4_MspInit 1 */
  }
```

在终端输入测试命令，就可以看到显示了
![image.png](G:/Typora/user_potograph/36e10ace2975e06a771f9c850eddd386.png)

**添加触摸功能**
首先是配置RT-Thread Settings

![image-20220611190730715](G:/Typora/user_potograph/image-20220611190730715.png)

在软件包ft6236中包含有一个sample的目录，里面有相关的代码，我们可以把代码段拷贝到main.c中，并进行相应的修改

拷贝到main.c之后，记得要把设备名改成“i2c2”，因为硬件上就是连接在I2C2上的

最终主函数：

```c
/*
 * Copyright (c) 2006-2020, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2020-09-02     RT-Thread    first version
 */

#include <rtthread.h>
#include <rtdevice.h>
#include "drv_common.h"
#include "ft6236.h"
#include "touch.h"
#include <rtdbg.h>

#define LED_PIN GET_PIN(I, 8)

#define DBG_TAG "ft6236_example"
#define DBG_LVL DBG_LOG
#define REST_PIN GET_PIN(D, 3)

rt_thread_t ft6236_thread;
rt_device_t touch;

int main(void)
{
    rt_uint32_t count = 1;

    rt_pin_mode(LED_PIN, PIN_MODE_OUTPUT);

    while(count++)
    {
        rt_thread_mdelay(500);
        rt_pin_write(LED_PIN, PIN_HIGH);
        rt_thread_mdelay(500);
        rt_pin_write(LED_PIN, PIN_LOW);
    }
    return RT_EOK;
}

#include "stm32h7xx.h"
static int vtor_config(void)
{
    /* Vector Table Relocation in Internal QSPI_FLASH */
    SCB->VTOR = QSPI_BASE;
    return 0;
}
INIT_BOARD_EXPORT(vtor_config);


void ft6236_thread_entry(void *parameter)
{
    struct rt_touch_data *read_data;

    read_data = (struct rt_touch_data *)rt_calloc(1, sizeof(struct rt_touch_data));

    while(1)
    {
        rt_device_read(touch, 0, read_data, 1);

        if (read_data->event == RT_TOUCH_EVENT_DOWN)
        {
            rt_kprintf("down x: %03d y: %03d", read_data->x_coordinate, read_data->y_coordinate);
            rt_kprintf(" t: %d\n", read_data->timestamp);
        }
        if (read_data->event == RT_TOUCH_EVENT_MOVE)
        {
            rt_kprintf("move x: %03d y: %03d", read_data->x_coordinate, read_data->y_coordinate);
            rt_kprintf(" t: %d\n", read_data->timestamp);
        }
        if (read_data->event == RT_TOUCH_EVENT_UP)
        {
            rt_kprintf("up   x: %03d y: %03d", read_data->x_coordinate, read_data->y_coordinate);
            rt_kprintf(" t: %d\n\n", read_data->timestamp);
        }
        rt_thread_delay(10);
    }

}

int ft6236_example(void)
{
    struct rt_touch_config cfg;

    cfg.dev_name = "i2c2";
    rt_hw_ft6236_init("touch", &cfg, REST_PIN);

    touch = rt_device_find("touch");

    rt_device_open(touch, RT_DEVICE_FLAG_RDONLY);

    struct rt_touch_info info;
    rt_device_control(touch, RT_TOUCH_CTRL_GET_INFO, &info);
    LOG_I("type       :%d", info.type);
    LOG_I("vendor     :%d", info.vendor);
    LOG_I("point_num  :%d", info.point_num);
    LOG_I("range_x    :%d", info.range_x);
    LOG_I("range_y    :%d", info.range_y);

    ft6236_thread = rt_thread_create("touch", ft6236_thread_entry, RT_NULL, 800, 10, 20);
    if (ft6236_thread == RT_NULL)
    {
        LOG_D("create ft6236 thread err");

        return -RT_ENOMEM;
    }
    rt_thread_startup(ft6236_thread);

    return RT_EOK;
}
INIT_APP_EXPORT(ft6236_example);


```

![image-20220611190903758](G:/Typora/user_potograph/image-20220611190903758.png)

---







# GUI界面开发

添加littleVGL2RTT软件包

![image-20220611191136148](G:/Typora/user_potograph/image-20220611191136148.png)

右击littleVGL2RTT软件包，进行如下配置

![image-20220611191325892](G:/Typora/user_potograph/image-20220611191325892.png)

这里直接把色彩深度设置为32bit，后面在相应的文件中做修改。

主要修改的是littlevgl2rtt.c文件中的lcd_fb_flush函数，该函数负责刷新屏幕显示的内容。
修改内容如下：

![image-20220611192327809](G:/Typora/user_potograph/image-20220611192327809.png)

```c
else if (info.bits_per_pixel == 24 || info.bits_per_pixel == 32)
        {
            uint8_t *lcd_buf = (uint8_t *)info.framebuffer;

            for (y = act_y1; y <= act_y2; y++)
            {
                for (x = act_x1; x <= act_x2; x++)
                {
                    location = (x) + (y)*info.width;
                    lcd_buf[3 * location] = color_p->ch.red;
                    lcd_buf[3 * location + 1] = color_p->ch.green;
                    lcd_buf[3 * location + 2] = color_p->ch.blue;
                    color_p++;
                }

                color_p += x2 - act_x2;
            }
        }
```

然后是修改littlevgl2rtt_init函数。
修改内容如下：

![image-20220611193513612](G:/Typora/user_potograph/image-20220611193513612.png)

```c
//    fbuf = rt_malloc(info.width * 10 * sizeof(*fbuf));
//    适配ART-Pi的媒体扩展屏
    fbuf = rt_malloc(info.width * info.height * sizeof(*fbuf));
    if (!fbuf)
    {
        rt_kprintf("Error: alloc disp buf fail\n");
        return -1;
    }
```

![image-20220611193616768](G:/Typora/user_potograph/image-20220611193616768.png)

```c
//    lv_disp_buf_init(&disp_buf, fbuf, NULL, info.width * 10);
//    适配ART-Pi媒体扩展屏
    lv_disp_buf_init(&disp_buf, fbuf, NULL, info.width * info.height);
    disp_drv.buffer = &disp_buf;
    lv_disp_drv_register(&disp_drv);
```

这是竖屏显示，有没有办法横屏呢，不急不急。接着修改lcd_spi_port.h，内容如下：
增加了一个关于横屏还是纵屏的宏定义

![image-20220611194250538](G:/Typora/user_potograph/image-20220611194250538.png)

```c
#define LC_HOR_SCREEN //定义屏幕是横屏还是纵屏，注意LCD的驱动要做相应的修改，touch也要做相应的修改

#define LCD_HOR_SCREEN
#ifdef LCD_HOR_SCREEN
#define LCD_WIDTH   320
#define LCD_HEIGHT  480
#else
#define LCD_WIDTH   480
#define LCD_HEIGHT  320
#endif
```

接下来在LCD初始化代码中也要做相应的修改，修改drv_spi_ili9488.c文件，内容如下：

![image-20220611194453135](G:/Typora/user_potograph/image-20220611194453135.png)

下面来看看怎么把触摸也搞起来。前面已经调通触摸驱动了，用的是ft6236的软件包。跑了里面一个sample，确实是可以从终端打印触摸信息的。

只要在触摸动作处理里面加入这个函数就可以了。
我修改了sample，如下：

![image-20220611195959787](G:/Typora/user_potograph/image-20220611195959787.png)

```c
#ifdef LCD_HOR_SCREEN
        littlevgl2rtt_send_input_event(read_data->y_coordinate, read_data->x_coordinate?(LCD_HEIGHT - read_data ->x_coordinate):0,read_data->event);
        #else
        littlevgl2rtt_send_input_event(read_data->x_coordinate, read_data->y_coordinate, RT_TOUCH_EVENT_NONE);
        #endif
```

