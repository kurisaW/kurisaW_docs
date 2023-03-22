

```c
void rt_hw_serial_isr(struct rt_serial_device *serial, int event)
{
    RT_ASSERT(serial != RT_NULL);

    switch (event & 0xff)
    {
        /* Interrupt receive event */
        case RT_SERIAL_EVENT_RX_IND:
        case RT_SERIAL_EVENT_RX_DMADONE:
        {
            struct rt_serial_rx_fifo *rx_fifo;
            rt_size_t rx_length = 0;
            rx_length = serial->ops->getc(serial);

            rx_fifo = (struct rt_serial_rx_fifo *)serial->serial_rx;
            rx_fifo->buffer[rx_fifo->rb.write_index] = rx_length;
            rx_fifo->rb.write_index += 1;

            if (rx_length)
            {
                rt_serial_update_write_index(&(rx_fifo->rb), rx_length);
            }

            RT_ASSERT(rx_fifo != RT_NULL);

            /* If the event is RT_SERIAL_EVENT_RX_IND, rx_length is equal to 0 */
//            rx_length = (event & (~0xff)) >> 8;

            if (rx_length != 0)
            {
                rt_serial_update_write_index(&(rx_fifo->rb), rx_length);
            }

            /* Get the length of the data from the ringbuffer */
            rx_length = rt_ringbuffer_data_len(&rx_fifo->rb);
            if (rx_length == -1) break;

            if (serial->parent.open_flag & RT_SERIAL_RX_BLOCKING)
            {
                if (rx_fifo->rx_cpt_index && rx_length >= rx_fifo->rx_cpt_index )
                {
                    rx_fifo->rx_cpt_index = 0;
                    rt_completion_done(&(rx_fifo->rx_cpt));
                }
            }
            /* Trigger the receiving completion callback */
            if (serial->parent.rx_indicate != RT_NULL)
                serial->parent.rx_indicate(&(serial->parent), rx_length);
            break;
        }

        /* Interrupt transmit event */
        case RT_SERIAL_EVENT_TX_DONE:
        {
            struct rt_serial_tx_fifo *tx_fifo;
            rt_size_t tx_length = 0;
            tx_fifo = (struct rt_serial_tx_fifo *)serial->serial_tx;
            RT_ASSERT(tx_fifo != RT_NULL);

            /* Get the length of the data from the ringbuffer */
            tx_length = rt_ringbuffer_data_len(&tx_fifo->rb);
            /* If there is no data in tx_ringbuffer,
             * then the transmit completion callback is triggered*/
            if (tx_length == 0)
            {
                tx_fifo->activated = RT_FALSE;
                /* Trigger the transmit completion callback */
                if (serial->parent.tx_complete != RT_NULL)
                    serial->parent.tx_complete(&serial->parent, RT_NULL);

                if (serial->parent.open_flag & RT_SERIAL_TX_BLOCKING)
                    rt_completion_done(&(tx_fifo->tx_cpt));

                break;
            }

            /* Call the transmit interface for transmission again */
            /* Note that in interrupt mode, tx_fifo->buffer and tx_length
             * are inactive parameters */
            serial->ops->transmit(serial,
                                tx_fifo->buffer,
                                tx_length,
                                serial->parent.open_flag & ( \
                                RT_SERIAL_TX_BLOCKING | \
                                RT_SERIAL_TX_NON_BLOCKING));
            break;
        }

        case RT_SERIAL_EVENT_TX_DMADONE:
        {
            struct rt_serial_tx_fifo *tx_fifo;
            tx_fifo = (struct rt_serial_tx_fifo *)serial->serial_tx;
            RT_ASSERT(tx_fifo != RT_NULL);

            tx_fifo->activated = RT_FALSE;

            /* Trigger the transmit completion callback */
            if (serial->parent.tx_complete != RT_NULL)
                serial->parent.tx_complete(&serial->parent, RT_NULL);

            if (serial->parent.open_flag & RT_SERIAL_TX_BLOCKING)
            {
                rt_completion_done(&(tx_fifo->tx_cpt));
                break;
            }

            rt_serial_update_read_index(&tx_fifo->rb, tx_fifo->put_size);
            /* Get the length of the data from the ringbuffer.
             * If there is some data in tx_ringbuffer,
             * then call the transmit interface for transmission again */
            if (rt_ringbuffer_data_len(&tx_fifo->rb))
            {
                tx_fifo->activated = RT_TRUE;

                rt_uint8_t *put_ptr  = RT_NULL;
                /* Get the linear length buffer from rinbuffer */
                tx_fifo->put_size = rt_serial_get_linear_buffer(&(tx_fifo->rb), &put_ptr);
                /* Call the transmit interface for transmission again */
                serial->ops->transmit(serial,
                                    put_ptr,
                                    tx_fifo->put_size,
                                    RT_SERIAL_TX_NON_BLOCKING);
            }

            break;
        }

        default:
            break;
    }
}
```

```c
void rt_hw_serial_isr(struct rt_serial_device *serial, int event)
{
    RT_ASSERT(serial != RT_NULL);

    switch (event & 0xff)
    {
        /* Interrupt receive event */
        case RT_SERIAL_EVENT_RX_IND:
        case RT_SERIAL_EVENT_RX_DMADONE:
        {
            struct rt_serial_rx_fifo *rx_fifo;
            rt_size_t rx_length = 0;
            rx_fifo = (struct rt_serial_rx_fifo *)serial->serial_rx;
            RT_ASSERT(rx_fifo != RT_NULL);

            /* If the event is RT_SERIAL_EVENT_RX_IND, rx_length is equal to 0 */
            rx_length = (event & (~0xff)) >> 8;

            if (rx_length)
                rt_serial_update_write_index(&(rx_fifo->rb), rx_length);

            /* Get the length of the data from the ringbuffer */
            rx_length = rt_ringbuffer_data_len(&rx_fifo->rb);
            if (rx_length == 0) break;

            if (serial->parent.open_flag & RT_SERIAL_RX_BLOCKING)
            {
                if (rx_fifo->rx_cpt_index && rx_length >= rx_fifo->rx_cpt_index )
                {
                    rx_fifo->rx_cpt_index = 0;
                    rt_completion_done(&(rx_fifo->rx_cpt));
                }
            }
            /* Trigger the receiving completion callback */
            if (serial->parent.rx_indicate != RT_NULL)
                serial->parent.rx_indicate(&(serial->parent), rx_length);
            break;
        }

        /* Interrupt transmit event */
        case RT_SERIAL_EVENT_TX_DONE:
        {
            struct rt_serial_tx_fifo *tx_fifo;
            rt_size_t tx_length = 0;
            tx_fifo = (struct rt_serial_tx_fifo *)serial->serial_tx;
            RT_ASSERT(tx_fifo != RT_NULL);

            /* Get the length of the data from the ringbuffer */
            tx_length = rt_ringbuffer_data_len(&tx_fifo->rb);
            /* If there is no data in tx_ringbuffer,
             * then the transmit completion callback is triggered*/
            if (tx_length == 0)
            {
                tx_fifo->activated = RT_FALSE;
                /* Trigger the transmit completion callback */
                if (serial->parent.tx_complete != RT_NULL)
                    serial->parent.tx_complete(&serial->parent, RT_NULL);

                if (serial->parent.open_flag & RT_SERIAL_TX_BLOCKING)
                    rt_completion_done(&(tx_fifo->tx_cpt));

                break;
            }

            /* Call the transmit interface for transmission again */
            /* Note that in interrupt mode, tx_fifo->buffer and tx_length
             * are inactive parameters */
            serial->ops->transmit(serial,
                                tx_fifo->buffer,
                                tx_length,
                                serial->parent.open_flag & ( \
                                RT_SERIAL_TX_BLOCKING | \
                                RT_SERIAL_TX_NON_BLOCKING));
            break;
        }

        case RT_SERIAL_EVENT_TX_DMADONE:
        {
            struct rt_serial_tx_fifo *tx_fifo;
            tx_fifo = (struct rt_serial_tx_fifo *)serial->serial_tx;
            RT_ASSERT(tx_fifo != RT_NULL);

            tx_fifo->activated = RT_FALSE;

            /* Trigger the transmit completion callback */
            if (serial->parent.tx_complete != RT_NULL)
                serial->parent.tx_complete(&serial->parent, RT_NULL);

            if (serial->parent.open_flag & RT_SERIAL_TX_BLOCKING)
            {
                rt_completion_done(&(tx_fifo->tx_cpt));
                break;
            }

            rt_serial_update_read_index(&tx_fifo->rb, tx_fifo->put_size);
            /* Get the length of the data from the ringbuffer.
             * If there is some data in tx_ringbuffer,
             * then call the transmit interface for transmission again */
            if (rt_ringbuffer_data_len(&tx_fifo->rb))
            {
                tx_fifo->activated = RT_TRUE;

                rt_uint8_t *put_ptr  = RT_NULL;
                /* Get the linear length buffer from rinbuffer */
                tx_fifo->put_size = rt_serial_get_linear_buffer(&(tx_fifo->rb), &put_ptr);
                /* Call the transmit interface for transmission again */
                serial->ops->transmit(serial,
                                    put_ptr,
                                    tx_fifo->put_size,
                                    RT_SERIAL_TX_NON_BLOCKING);
            }

            break;
        }

        default:
            break;
    }
}
```

```c
// STM32 串口V1

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



















```c
struct rt_serial_device
{
    struct rt_device          parent;

    const struct rt_uart_ops *ops;
    struct serial_configure   config;

    void *serial_rx;
    void *serial_tx;
};
```

