* 需求：实现勾选上一个软件包后自动选中依赖项

首先找到`./board目录下的Kconfig`，修改为以下：

```c
menu "RT-Thread Components"

    config PKG_USING_MICROPYTHON
        bool "Support legacy version for compatibility"
        depends on PKG_USING_MICROPYTHON
        select RT_USING_LEGACY
        default n
endmenu
```

![image-20230206173525794](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302061735116.png)

参考资料：

* [文档中心](https://www.rt-thread.org/document/site/#/development-tools/build-config-system/Kconfig)
* [github示例](https://github.com/RT-Thread/rt-thread/blob/master/bsp/nrf5x/nrf52840/board/Kconfig)