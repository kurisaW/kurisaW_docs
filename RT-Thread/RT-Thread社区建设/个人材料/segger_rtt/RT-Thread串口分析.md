## 一、串口设备的整体使用框架：

![image-20220914150238742](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141502976.png)

在RT-Thread中，串口的设备驱动框架包含UART设备驱动层、UART设备驱动框架层、IO设备管理（也就是Device框架层）以及最后的应用层。

首先需要使用到Device框架

![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141538684.png)

其次是UART设备驱动框架

![image-20220914154233322](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141542375.png)

然后是串口驱动层

![image-20220914154812116](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141548152.png)

```c
rt_err_t rt_hw_serial_register(struct rt_serial_device *serial,
                               const char              *name,
                               rt_uint32_t              flag,
                               void                    *data)
{
    rt_err_t ret;
    struct rt_device *device;
    RT_ASSERT(serial != RT_NULL);

    device = &(serial->parent);

    device->type        = RT_Device_Class_Char;
    device->rx_indicate = RT_NULL;
    device->tx_complete = RT_NULL;

#ifdef RT_USING_DEVICE_OPS
    device->ops         = &serial_ops;
#else
    device->init        = rt_serial_init;
    device->open        = rt_serial_open;
    device->close       = rt_serial_close;
    device->read        = rt_serial_read;
    device->write       = rt_serial_write;
    device->control     = rt_serial_control;
#endif
    device->user_data   = data;

    /* register a character device */
    ret = rt_device_register(device, name, flag);

#if defined(RT_USING_POSIX)
    /* set fops */
    device->fops        = &_serial_fops;
#endif

    return ret;
}
```

![image.png](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141608274.png)

## 二、串口的三种工作模式

串口目前适配了三种硬件工作模式：串口轮询（POLL）、串口中断（INT）、串口DMA。

#### 1.轮询模式（POLL）

轮询模式即调用串口接收/发送的API后将一直占用CPU资源知到数据手法完成后才返回，使用时需要用户主动调用接收/发送的API接口才会执行相应的操作。

**轮询接收**

应用程序调用rt_device_read()接收数据，轮询接收模式下会调用到底层串口驱动提供的`getc接口`每次接收一个字节，如果接收不到数据将一直占用CPU资源

```c
rt_size_t rt_device_read(rt_device_t dev,
                         rt_off_t    pos,
                         void       *buffer,
                         rt_size_t   size)
{
    RT_ASSERT(dev != RT_NULL);
    RT_ASSERT(rt_object_get_type(&dev->parent) == RT_Object_Class_Device);

    if (dev->ref_count == 0)
    {
        rt_set_errno(-RT_ERROR);
        return 0;
    }

    /* call device_read interface */
    if (device_read != RT_NULL)
    {
        return device_read(dev, pos, buffer, size);
    }

    /* set error code */
    rt_set_errno(-RT_ENOSYS);

    return 0;
}
RTM_EXPORT(rt_device_read);
```

```c
rt_inline int _serial_poll_rx(struct rt_serial_device *serial, rt_uint8_t *data, int length)
{
    int ch;
    int size;

    RT_ASSERT(serial != RT_NULL);
    size = length;

    while (length)
    {
        ch = serial->ops->getc(serial);
        if (ch == -1) break;

        *data = ch;
        data ++; length --;

        if(serial->parent.open_flag & RT_DEVICE_FLAG_STREAM)
        {
            if (ch == '\n') break;
        }
    }

    return size - length;
}
```



**轮询发送**

应用程序调用rt_device_write()，轮询发送模式下最终对调用到底层串口的`putc接口`，每次接收一个字节，如果接收不到数据将一直占用CPU资源。

```c
rt_size_t rt_device_write(rt_device_t dev,
                          rt_off_t    pos,
                          const void *buffer,
                          rt_size_t   size)
{
    RT_ASSERT(dev != RT_NULL);
    RT_ASSERT(rt_object_get_type(&dev->parent) == RT_Object_Class_Device);

    if (dev->ref_count == 0)
    {
        rt_set_errno(-RT_ERROR);
        return 0;
    }

    /* call device_write interface */
    if (device_write != RT_NULL)
    {
        return device_write(dev, pos, buffer, size);
    }

    /* set error code */
    rt_set_errno(-RT_ENOSYS);

    return 0;
}
RTM_EXPORT(rt_device_write);
```

```c
rt_inline int _serial_poll_tx(struct rt_serial_device *serial, const rt_uint8_t *data, int length)
{
    int size;
    RT_ASSERT(serial != RT_NULL);

    size = length;
    while (length)
    {
        /*
         * to be polite with serial console add a line feed
         * to the carriage return character
         */
        if (*data == '\n' && (serial->parent.open_flag & RT_DEVICE_FLAG_STREAM))
        {
            serial->ops->putc(serial, '\r');
        }

        serial->ops->putc(serial, *data);

        ++ data;
        -- length;
    }

    return size - length;
}
```



![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141632363.png)

#### 2.中断模式

中断模式在下发数据时，将数据的收发过程放在中断中进行，不再持续占用CPU资源，应用程序只负责将数据写入串口数据寄存器，然后线程就会床资源，空出了程序等待串口硬件收发数据的时间。

`注意：中断接收和发送时，每次在中断中操作的还是单个字节`

**中断接收**

_serial_int_rx代码部分，通过该函数接收数据

```c
rt_inline int _serial_int_rx(struct rt_serial_device *serial, rt_uint8_t *data, int length)
{
    int size;
    struct rt_serial_rx_fifo* rx_fifo;

    RT_ASSERT(serial != RT_NULL);
    size = length;

    rx_fifo = (struct rt_serial_rx_fifo*) serial->serial_rx;
    RT_ASSERT(rx_fifo != RT_NULL);

    /* read from software FIFO */
    while (length)
    {
        int ch;
        rt_base_t level;

        /* disable interrupt */
        level = rt_hw_interrupt_disable();

        /* there's no data: */
        if ((rx_fifo->get_index == rx_fifo->put_index) && (rx_fifo->is_full == RT_FALSE))
        {
            /* no data, enable interrupt and break out */
            rt_hw_interrupt_enable(level);
            break;
        }

        /* otherwise there's the data: */
        ch = rx_fifo->buffer[rx_fifo->get_index];
        rx_fifo->get_index += 1;
        if (rx_fifo->get_index >= serial->config.bufsz) rx_fifo->get_index = 0;

        if (rx_fifo->is_full == RT_TRUE)
        {
            rx_fifo->is_full = RT_FALSE;
        }

        /* enable interrupt */
        rt_hw_interrupt_enable(level);

        *data = ch & 0xff;
        data ++; length --;
    }

    return size - length;
}
```

串口硬件接收到一个字节的数据后会触发中断并调用串口驱动框架的rt_hw_serial_isr()函数，此函数会将这一字节的数据写入环形缓冲区，用户设置的回调函数也会被调用，应用程序读取函数实际上是从环形缓冲区读取。

```c
void rt_hw_serial_isr(struct rt_serial_device *serial, int event)
{
    switch (event & 0xff)
    {
        case RT_SERIAL_EVENT_RX_IND:
        {
            int ch = -1;
            rt_base_t level;
            struct rt_serial_rx_fifo* rx_fifo;

            /* interrupt mode receive */
            rx_fifo = (struct rt_serial_rx_fifo*)serial->serial_rx;
            RT_ASSERT(rx_fifo != RT_NULL);

            while (1)
            {
                ch = serial->ops->getc(serial);
                if (ch == -1) break;


                /* disable interrupt */
                level = rt_hw_interrupt_disable();

                rx_fifo->buffer[rx_fifo->put_index] = ch;
                rx_fifo->put_index += 1;
                if (rx_fifo->put_index >= serial->config.bufsz) rx_fifo->put_index = 0;

                /* if the next position is read index, discard this 'read char' */
                if (rx_fifo->put_index == rx_fifo->get_index)
                {
                    rx_fifo->get_index += 1;
                    rx_fifo->is_full = RT_TRUE;
                    if (rx_fifo->get_index >= serial->config.bufsz) rx_fifo->get_index = 0;

                    _serial_check_buffer_size();
                }

                /* enable interrupt */
                rt_hw_interrupt_enable(level);
            }

            /* invoke callback */
            if (serial->parent.rx_indicate != RT_NULL)
            {
                rt_size_t rx_length;

                /* get rx length */
                level = rt_hw_interrupt_disable();
                rx_length = (rx_fifo->put_index >= rx_fifo->get_index)? (rx_fifo->put_index - rx_fifo->get_index):
                    (serial->config.bufsz - (rx_fifo->get_index - rx_fifo->put_index));
                rt_hw_interrupt_enable(level);

                if (rx_length)
                {
                    serial->parent.rx_indicate(&serial->parent, rx_length);
                }
            }
            break;
        }
        case RT_SERIAL_EVENT_TX_DONE:
        {
            struct rt_serial_tx_fifo* tx_fifo;

            tx_fifo = (struct rt_serial_tx_fifo*)serial->serial_tx;
            rt_completion_done(&(tx_fifo->completion));
            break;
        }
#ifdef RT_SERIAL_USING_DMA
        case RT_SERIAL_EVENT_TX_DMADONE:
        {
            const void *data_ptr;
            rt_size_t data_size;
            const void *last_data_ptr;
            struct rt_serial_tx_dma *tx_dma;

            tx_dma = (struct rt_serial_tx_dma*) serial->serial_tx;

            rt_data_queue_pop(&(tx_dma->data_queue), &last_data_ptr, &data_size, 0);
            if (rt_data_queue_peek(&(tx_dma->data_queue), &data_ptr, &data_size) == RT_EOK)
            {
                /* transmit next data node */
                tx_dma->activated = RT_TRUE;
                serial->ops->dma_transmit(serial, (rt_uint8_t *)data_ptr, data_size, RT_SERIAL_DMA_TX);
            }
            else
            {
                tx_dma->activated = RT_FALSE;
            }

            /* invoke callback */
            if (serial->parent.tx_complete != RT_NULL)
            {
                serial->parent.tx_complete(&serial->parent, (void*)last_data_ptr);
            }
            break;
        }
        case RT_SERIAL_EVENT_RX_DMADONE:
        {
            int length;
            rt_base_t level;

            /* get DMA rx length */
            length = (event & (~0xff)) >> 8;

            if (serial->config.bufsz == 0)
            {
                struct rt_serial_rx_dma* rx_dma;

                rx_dma = (struct rt_serial_rx_dma*) serial->serial_rx;
                RT_ASSERT(rx_dma != RT_NULL);

                RT_ASSERT(serial->parent.rx_indicate != RT_NULL);
                serial->parent.rx_indicate(&(serial->parent), length);
                rx_dma->activated = RT_FALSE;
            }
            else
            {
                /* disable interrupt */
                level = rt_hw_interrupt_disable();
                /* update fifo put index */
                rt_dma_recv_update_put_index(serial, length);
                /* calculate received total length */
                length = rt_dma_calc_recved_len(serial);
                /* enable interrupt */
                rt_hw_interrupt_enable(level);
                /* invoke callback */
                if (serial->parent.rx_indicate != RT_NULL)
                {
                    serial->parent.rx_indicate(&(serial->parent), length);
                }
            }
            break;
        }
#endif /* RT_SERIAL_USING_DMA */
    }
}
```

![image-20220914172048397](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141720696.png)

`_HAL_UART_GET_FLAG(UART_FLAG_RXNE)->标志位：UART read data register not empty`

**中断发送**

串口框架负责调用底层`putc接口`，接口向串口寄存器写入待发送的数据，然后等待此数据发送完成的信号，此时当前的线程会被挂起。上一个数据发送完成后会调用串口驱动框架的`rt_wh_serial_isr()`函数，此函数会发送完成信号唤醒发送数据线程，并重置完成信号状态为未完成，线程运行后会向串口寄存器写入下一个数据，然后等待完成信号重复上一个流程，直到缓冲区数据发送完成。

```c
rt_inline int _serial_int_tx(struct rt_serial_device *serial, const rt_uint8_t *data, int length)
{
    int size;
    struct rt_serial_tx_fifo *tx;

    RT_ASSERT(serial != RT_NULL);

    size = length;
    tx = (struct rt_serial_tx_fifo*) serial->serial_tx;
    RT_ASSERT(tx != RT_NULL);

    while (length)
    {
        /*
         * to be polite with serial console add a line feed
         * to the carriage return character
         */
        if (*data == '\n' && (serial->parent.open_flag & RT_DEVICE_FLAG_STREAM))
        {
            if (serial->ops->putc(serial, '\r') == -1)
            {
                rt_completion_wait(&(tx->completion), RT_WAITING_FOREVER);
                continue;
            }
        }

        if (serial->ops->putc(serial, *(char*)data) == -1)
        {
            rt_completion_wait(&(tx->completion), RT_WAITING_FOREVER);
            continue;
        }

        data ++; length --;
    }

    return size - length;
}
```

#### 3.DMA模式

DMA模式下收发数据与中断模式很相像，可以粗略的认为，两种模式唯一不同的就是，中断模式每次收发数据为单字节，而DMA模式则是多个字节进行收发的。

**DMA接收**

使用DMA接收模式时，首先应用程序打开串口设备会指定标志为RT_DEVICE_FLAG_DMA_RX，此时串口驱动框架会创建环形缓冲区并调用串口驱动层`control接口`使能DMA接收完成中断。

```c
rt_inline int _serial_dma_rx(struct rt_serial_device *serial, rt_uint8_t *data, int length)
{
    rt_base_t level;

    RT_ASSERT((serial != RT_NULL) && (data != RT_NULL));

    level = rt_hw_interrupt_disable();

    if (serial->config.bufsz == 0)
    {
        int result = RT_EOK;
        struct rt_serial_rx_dma *rx_dma;

        rx_dma = (struct rt_serial_rx_dma*)serial->serial_rx;
        RT_ASSERT(rx_dma != RT_NULL);

        if (rx_dma->activated != RT_TRUE)
        {
            rx_dma->activated = RT_TRUE;
            RT_ASSERT(serial->ops->dma_transmit != RT_NULL);
            serial->ops->dma_transmit(serial, data, length, RT_SERIAL_DMA_RX);
        }
        else result = -RT_EBUSY;
        rt_hw_interrupt_enable(level);

        if (result == RT_EOK) return length;

        rt_set_errno(result);
        return 0;
    }
    else
    {
        struct rt_serial_rx_fifo *rx_fifo = (struct rt_serial_rx_fifo *) serial->serial_rx;
        rt_size_t recv_len = 0, fifo_recved_len = rt_dma_calc_recved_len(serial);

        RT_ASSERT(rx_fifo != RT_NULL);

        if (length < (int)fifo_recved_len)
            recv_len = length;
        else
            recv_len = fifo_recved_len;

        if (rx_fifo->get_index + recv_len < serial->config.bufsz)
            rt_memcpy(data, rx_fifo->buffer + rx_fifo->get_index, recv_len);
        else
        {
            rt_memcpy(data, rx_fifo->buffer + rx_fifo->get_index,
                    serial->config.bufsz - rx_fifo->get_index);
            rt_memcpy(data + serial->config.bufsz - rx_fifo->get_index, rx_fifo->buffer,
                    recv_len + rx_fifo->get_index - serial->config.bufsz);
        }
        rt_dma_recv_update_get_index(serial, recv_len);
        rt_hw_interrupt_enable(level);
        return recv_len;
    }
}
```

![image-20220914192826423](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209141928507.png)

在DMA中断处理函数中，有三种DMA接收中断，分别是`DMA空闲中断(IDLE)`、`DMA半满中断(HFIT)`和`DMA满中断(TCIT)`，这三个中断相辅相成，最后统一交由串口框架中断处理函数`rt_hw_serial_isr(RT_EVENT_RX_DMA_DONE)`中做处理。

其中DMA空闲中断时在uart_isr中触发的，另外两个是在DMA接收回调中触发的，这两个接收回调在DMA接收使能启动时，由HAL库注册的，用户无需关心这两个DMA接收回调的注册逻辑，只需知道，`用户如果不想使用这两个DMA接收回调时，只要把函数里面的执行代码注释掉即可`。如下所示：

```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    struct stm32_uart *uart;
    RT_ASSERT(huart != NULL);
    // uart = (struct stm32_uart *)huart;
    // dma_isr(&uart->serial);
}
void HAL_UART_RxHalfCpltCallback(UART_HandleTypeDef *huart)
{
    struct stm32_uart *uart;
    RT_ASSERT(huart != NULL);
    // uart = (struct stm32_uart *)huart;
    // dma_isr(&uart->serial);
}
```

**DMA发送**

应用程序发送数据流程如下图所示，应用程序调用`rt_device_write()`发送数据，最终待数据的地址和大小会被放入数据队列，如果此时DMA空闲，就会开始发送数据。当DMA发送完成触发中断时，`rt_hw_serial_isr()`函数会判断数据队列是否持有待传输的数据并开始下次的DMA传输，用户设置的回调函数也会被调用。

![image-20220914201135126](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209142011196.png?token=AXQGQBBAU4MG4UDIOZ3GZ7LDEHCTM)

由上图我们可以看出，在使用DMA发送的时候，串口框架并未使用环形缓冲区，然而我们又知道，DMA发送时，肯定是需要缓冲区来存放数据的，那么DMA发送时候的数据块，存放在哪里呢？这里就需要注意了，DMA发送时候，数据缓冲区是由用户应用层定义的，在传给串口框架的时候，是将应用层定义的数据缓冲区的指针传递过来。`如果在发送过程中，该缓冲区的内容被以外修改了，那么将会导致DMA发送的数据出现错误。`结合下图说明：

![image.png](https://oss-club.rt-thread.org/uploads/20210719/cd59e4c2ccde8c9a6a6e6f10ed6233d2.png)

## 三、串口的应用场景---Finsh组件

