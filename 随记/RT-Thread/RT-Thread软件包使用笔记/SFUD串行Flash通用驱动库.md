# SFUD (Serial Flash Universal Driver) 串行 Flash 通用驱动库

> 项目地址：[https://github.com/armink/SFUD](https://github.com/armink/SFUD)

## 1.什么是SFUD

[SFUD](https://github.com/armink/SFUD) 是一款开源的串行 SPI Flash 通用驱动库。由于现有市面的串行 Flash 种类居多，各个 Flash 的规格及命令存在差异， SFUD 就是为了解决这些 Flash 的差异现状而设计，让我们的产品能够支持不同品牌及规格的 Flash，提高了涉及到 Flash 功能的软件的可重用性及可扩展性，同时也可以规避 Flash 缺货或停产给产品所带来的风险。

- 主要特点：支持 SPI/QSPI 接口、面向对象（同时支持多个 Flash 对象）、可灵活裁剪、扩展性强、支持 4 字节地址
- 资源占用
  - 标准占用：RAM:0.2KB ROM:5.5KB
  - 最小占用：RAM:0.1KB ROM:3.6KB

## 2.如何使用

首先看目前已经通过测试的平台

| 型号                                                         | 制造商     | 容量  | 最高速度 | SFDP 标准 | QSPI 模式 | 备注                                               |
| ------------------------------------------------------------ | ---------- | ----- | -------- | --------- | --------- | -------------------------------------------------- |
| [W25Q40BV](http://microchip.ua/esp8266/W25Q40BV(EOL).pdf)    | Winbond    | 4Mb   | 50Mhz    | 不支持    | 双线      | 已停产                                             |
| [W25Q80DV](http://www.winbond.com/resource-files/w25q80dv_revg_07212015.pdf) | Winbond    | 8Mb   | 104Mhz   | 支持      | 双线      |                                                    |
| [W25Q16BV](https://media.digikey.com/pdf/Data Sheets/Winbond PDFs/W25Q16BV.pdf) | Winbond    | 16Mb  | 104Mhz   | 不支持    | 双线      | by [slipperstree](https://github.com/slipperstree) |
| [W25Q16CV](http://www.winbond.com/resource-files/da00-w25q16cvf1.pdf) | Winbond    | 16Mb  | 104Mhz   | 支持      | 未测试    |                                                    |
| [W25Q16DV](http://www.winbond.com/resource-files/w25q16dv revk 05232016 doc.pdf) | Winbond    | 16Mb  | 104Mhz   | 支持      | 未测试    | by [slipperstree](https://github.com/slipperstree) |
| [W25Q32BV](http://www.winbond.com/resource-files/w25q32bv_revi_100413_wo_automotive.pdf) | Winbond    | 32Mb  | 104Mhz   | 支持      | 双线      |                                                    |
| [W25Q64CV](http://www.winbond.com/resource-files/w25q64cv_revh_052214[2].pdf) | Winbond    | 64Mb  | 80Mhz    | 支持      | 四线      |                                                    |
| [W25Q128BV](http://www.winbond.com/resource-files/w25q128bv_revh_100313_wo_automotive.pdf) | Winbond    | 128Mb | 104Mhz   | 支持      | 四线      |                                                    |
| [W25Q256FV](http://www.winbond.com/resource-files/w25q256fv revi 02262016 kms.pdf) | Winbond    | 256Mb | 104Mhz   | 支持      | 四线      |                                                    |
| [MX25L3206E](http://www.macronix.com/Lists/DataSheet/Attachments/3199/MX25L3206E, 3V, 32Mb, v1.5.pdf) | Macronix   | 32Mb  | 86MHz    | 支持      | 双线      |                                                    |
| [MX25L3233F](https://www.macronix.com/Lists/Datasheet/Attachments/8754/MX25L3233F, 3V, 32Mb, v1.7.pdf) | Macronix   | 32Mb  | 133MHz   | 支持      | 未测试    | by [JiapengLi](https://github.com/JiapengLi)       |
| [KH25L4006E](http://www.macronix.com.hk/Lists/Datasheet/Attachments/117/KH25L4006E.pdf) | Macronix   | 4Mb   | 86Mhz    | 支持      | 未测试    | by [JiapengLi](https://github.com/JiapengLi)       |
| [KH25L3206E](http://www.macronix.com.hk/Lists/Datasheet/Attachments/131/KH25L3206E.pdf) | Macronix   | 32Mb  | 86Mhz    | 支持      | 双线      |                                                    |
| [SST25VF016B](http://ww1.microchip.com/downloads/en/DeviceDoc/20005044C.pdf) | Microchip  | 16Mb  | 50MHz    | 不支持    | 不支持    | SST 已被 Microchip 收购                            |
| [M25P40](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p40.pdf) | Micron     | 4Mb   | 75Mhz    | 不支持    | 未测试    | by [redocCheng](https://github.com/redocCheng)     |
| [M25P80](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p80.pdf) | Micron     | 8Mb   | 75Mhz    | 不支持    | 未测试    | by [redocCheng](https://github.com/redocCheng)     |
| [M25P32](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p32.pdf) | Micron     | 32Mb  | 75Mhz    | 不支持    | 不支持    |                                                    |
| [EN25Q32B](http://www.kean.com.au/oshw/WR703N/teardown/EN25Q32B 32Mbit SPI Flash.pdf) | EON        | 32Mb  | 104MHz   | 不支持    | 未测试    |                                                    |
| [GD25Q16B](http://www.gigadevice.com/product/detail/5/410.html) | GigaDevice | 16Mb  | 120Mhz   | 不支持    | 未测试    | by [TanekLiang](https://github.com/TanekLiang)     |
| [GD25Q32C](http://www.gigadevice.com/product/detail/5/410.html) | GigaDevice | 32Mb  | 120Mhz   | 不支持    | 未测试    | by [gaupen1186](https://github.com/gaupen1186)     |
| [GD25Q64B](http://www.gigadevice.com/product/detail/5/364.html) | GigaDevice | 64Mb  | 120Mhz   | 不支持    | 双线      |                                                    |
| [S25FL216K](http://www.cypress.com/file/197346/download)     | Cypress    | 16Mb  | 65Mhz    | 不支持    | 双线      |                                                    |
| [S25FL032P](http://www.cypress.com/file/196861/download)     | Cypress    | 32Mb  | 104Mhz   | 不支持    | 未测试    | by [yc_911](https://gitee.com/yc_911)              |
| [S25FL164K](http://www.cypress.com/file/196886/download)     | Cypress    | 64Mb  | 108Mhz   | 支持      | 未测试    |                                                    |
| [A25L080](http://www.amictechnology.com/datasheets/A25L080.pdf) | AMIC       | 8Mb   | 100Mhz   | 不支持    | 双线      |                                                    |
| [A25LQ64](http://www.amictechnology.com/datasheets/A25LQ64.pdf) | AMIC       | 64Mb  | 104Mhz   | 支持      | 支持      |                                                    |
| [F25L004](http://www.esmt.com.tw/db/manager/upload/f25l004.pdf) | ESMT       | 4Mb   | 100Mhz   | 不支持    | 不支持    |                                                    |
| [PCT25VF016B](http://pctgroup.com.tw/attachments/files/files/248_25VF016B-P.pdf) | PCT        | 16Mb  | 80Mhz    | 不支持    | 不支持    | SST 授权许可，会被识别为 SST25VF016B               |
| [AT45DB161E](http://www.adestotech.com/wp-content/uploads/doc8782.pdf) | ADESTO     | 16Mb  | 85MHz    | 不支持    | 不支持    | ADESTO 收购 Atmel 串行闪存产品线                   |
| [NM25Q128EV](http://www.normem.com/product/278082170)        | Nor_Mem    | 128Mb | 未测试   | 不支持    | 未测试    | SFDP可能会读取到信息后识别为超过32Gb               |

> 如果SFUD不支持怎么办：如果该 Flash 不支持 SFDP 标准，SFUD 会查询配置文件 ( [`/sfud/inc/sfud_flash_def.h`](https://github.com/armink/SFUD/blob/4bee2d0417a7ce853cc7aa3639b03fe825611fd9/sfud/inc/sfud_flash_def.h#L116-L142) ) 中提供的 **Flash 参数信息表** 中是否支持该款 Flash。如果不支持，则可以在配置文件中添加该款 Flash 的参数信息（添加方法详细见 [2.5 添加库目前不支持的 Flash](https://github.com/armink/SFUD#25-添加库目前不支持的-flash)）。获取到了 Flash 的规格参数后，就可以实现对 Flash 的全部操作。

> 注：QSPI 模式中，双线表示支持双线快读，四线表示支持四线快读。而一般情况下，支持四线快读的 FLASH 也支持双线快读。

### 2.1API使用说明

#### 2.1.1初始化SFUD库

* 函数声明：

```c
sfud_err sfud_init(void)
```

* 函数功能：调用`sfud_err sfud_init(void)`会对Flash设备表中的全部设备进行初始化。

> 注:初始化完的SPI Flash默认都**已取消写保护**状态，如需开启写保护，请使用 `sfud_write_status` 函数修改 SPI Flash 状态。

#### 2.1.2 初始化指定Flash设备

* 函数声明：

```c
sfud_err sfud_device_init(sfud_flash *flash)
```

| 参数  | 描述                  |
| ----- | --------------------- |
| flash | 待初始化的 Flash 设备 |

* 函数功能：对单一指定设备进行初始化

#### 2.1.3 使能快速读模式(仅QSPI模式可用)

* 函数声明：

```c
sfud_err sfud_qspi_fast_read_enable(sfud_flash *flash, uint8_t data_line_width)
```

* 说明：

  当 SFUD 开启 QSPI 模式后，SFUD 中的 Flash 驱动支持使用 QSPI 总线进行通信。相比传统的 SPI 模式，使用 QSPI 能够加速 Flash 数据的读取，但当数据需要写入时，由于 Flash 本身的数据写入速度慢于 SPI 传输速度，所以 QSPI 模式下的数据写入速度提升并不明显。

  所以 **SFUD 对于 QSPI 模式的支持仅限于快速读命令**。通过该函数可以配置 Flash 所使用的 QSPI 总线的实际支持的数据线最大宽度，例如：1 线（默认值，即传统的 SPI 模式）、2 线、4 线。

  设置后，SFUD 会去结合当前设定的 QSPI 总线数据线宽度，去 [QSPI Flash 扩展信息表](https://github.com/armink/SFUD/blob/069d2b409ec239f84d675b2c3d37894e908829e6/sfud/inc/sfud_flash_def.h#L149-L177) 中匹配最合适的、速度最快的快速读命令，之后用户在调用 sfud_read() 时，会使用 QSPI 模式的传输函数发送该命令。

#### 2.1.4 读取Flash数据

* 函数声明：

```c
sfud_err sfud_read(const sfud_flash *flash, uint32_t addr, size_t size, uint8_t *data)
```

| 参数  | 描述                           |
| ----- | ------------------------------ |
| flash | Flash 设备对象                 |
| addr  | 起始地址                       |
| size  | 从起始地址开始读取数据的总大小 |
| data  | 读取到的数据                   |

#### 2.1.5 擦除Flash数据

* 函数声明

```c
sfud_err sfud_erase(const sfud_flash *flash, uint32_t addr, size_t size)
```

| 参数  | 描述                           |
| ----- | ------------------------------ |
| flash | Flash 设备对象                 |
| addr  | 起始地址                       |
| size  | 从起始地址开始擦除数据的总大小 |

> 注：擦除操作将会按照 Flash 芯片的擦除粒度（详见 Flash 数据手册，一般为 block 大小。初始化完成后，可以通过 `sfud_flash->chip.erase_gran` 查看）对齐，请注意保证起始地址和擦除数据大小按照 Flash 芯片的擦除粒度对齐，否则执行擦除操作后，将会导致其他数据丢失。

#### 2.1.6 擦除Flash全部数据

* 函数声明

```c
sfud_err sfud_chip_erase(const sfud_flash *flash)
```

| 参数  | 描述           |
| ----- | -------------- |
| flash | Flash 设备对象 |

#### 2.1.7 Flash写数据

* 函数声明

```c
sfud_err sfud_write(const sfud_flash *flash, uint32_t addr, size_t size, const uint8_t *data)
```

| 参数  | 描述                           |
| ----- | ------------------------------ |
| flash | Flash 设备对象                 |
| addr  | 起始地址                       |
| size  | 从起始地址开始写入数据的总大小 |
| data  | 待写入的数据                   |

#### 2.1.8 先擦除再往Flash写数据

* 函数声明

```c
sfud_err sfud_erase_write(const sfud_flash *flash, uint32_t addr, size_t size, const uint8_t *data)
```

| 参数  | 描述                           |
| ----- | ------------------------------ |
| flash | Flash 设备对象                 |
| addr  | 起始地址                       |
| size  | 从起始地址开始写入数据的总大小 |
| data  | 待写入的数据                   |

> 注：擦除操作将会按照 Flash 芯片的擦除粒度（详见 Flash 数据手册，一般为 block 大小。初始化完成后，可以通过 `sfud_flash->chip.erase_gran` 查看）对齐，请注意保证起始地址和擦除数据大小按照 Flash 芯片的擦除粒度对齐，否则执行擦除操作后，将会导致其他数据丢失。

#### 2.1.9 读取Flash状态

* 函数声明

```c
sfud_err sfud_read_status(const sfud_flash *flash, uint8_t *status)
```

| 参数   | 描述             |
| ------ | ---------------- |
| flash  | Flash 设备对象   |
| status | 当前状态寄存器值 |

#### 2.1.10 写（修改）Flash状态

* 函数声明

```c
sfud_err sfud_write_status(const sfud_flash *flash, bool is_volatile, uint8_t status)
```

| 参数        | 描述                                           |
| ----------- | ---------------------------------------------- |
| flash       | Flash 设备对象                                 |
| is_volatile | 是否为易闪失的，true: 易闪失的，及断电后会丢失 |
| status      | 当前状态寄存器值                               |

### 2.2配置方式

所有配置文件设置在 `/sfud/inc/sfud_cfg.h`下，用户可按需配置 。

#### 2.2.1 调试模式

```c
#define SFUD_DEBUG_MODE
```

打开/关闭 `SFUD_DEBUG_MODE` 宏定义

#### 2.2.2 是否使用 SFDP 参数功能

```c
#define SFUD_USING_SFDP
```

打开/关闭 `SFUD_USING_SFDP` 宏定义

> 注意：关闭后只会查询该库在 `/sfud/inc/sfud_flash_def.h` 中提供的 Flash 信息表。这样虽然会降低软件的适配性，但减少代码量。

#### 2.2.3 是否使用该库自带的 Flash 参数信息表

```c
#define SFUD_USING_FLASH_INFO_TABLE
```

打开/关闭 `SFUD_USING_FLASH_INFO_TABLE` 宏定义

> 注意：关闭后该库只驱动支持 SFDP 规范的 Flash，也会适当的降低部分代码量。另外 2.2.2 及 2.2.3 这两个宏定义至少定义一种，也可以两种方式都选择。

#### 2.2.4 既不使用 SFDP ，也不使用 Flash 参数信息表

为了进一步降低代码量，`SFUD_USING_SFDP` 与 `SFUD_USING_FLASH_INFO_TABLE` 也可以 **都不定义** 。

此时，只要在定义 Flash 设备时，指定好 Flash 参数，之后再调用 `sfud_device_init` 对该设备进行初始化。参考如下代码：

```c
sfud_flash sfud_norflash0 = {
        .name = "norflash0",
        .spi.name = "SPI1",
        .chip = { "W25Q64FV", SFUD_MF_ID_WINBOND, 0x40, 0x17, 8L * 1024L * 1024L, SFUD_WM_PAGE_256B, 4096, 0x20 } };
......
sfud_device_init(&sfud_norflash0);
......
```

#### 2.2.5 Flash 设备表

如果产品中存在多个 Flash ，可以添加 Flash 设备表。修改 `SFUD_FLASH_DEVICE_TABLE` 这个宏定义，示例如下：

```c
enum {
    SFUD_W25Q64CV_DEVICE_INDEX = 0,
    SFUD_GD25Q64B_DEVICE_INDEX = 1,
};

#define SFUD_FLASH_DEVICE_TABLE                                                \
{                                                                              \
    [SFUD_W25Q64CV_DEVICE_INDEX] = {.name = "W25Q64CV", .spi.name = "SPI1"},   \
    [SFUD_GD25Q64B_DEVICE_INDEX] = {.name = "GD25Q64B", .spi.name = "SPI3"},   \
}
```

上面定义了两个 Flash 设备（大部分产品一个足以），两个设备的名称为 `"W25Q64CV"` 及 `"GD25Q64B"` ，分别对应 `"SPI1"` 及 `"SPI3"` 这两个 SPI 设备名称（在移植 SPI 接口时会用到，位于 `/sfud/port/sfud_port.c` ）， `SFUD_W25Q16CV_DEVICE_INDEX` 与 `SFUD_GD25Q64B_DEVICE_INDEX` 这两个枚举定义了两个设备位于设备表中的索引，可以通过 `sfud_get_device_table()` 方法获取到设备表，再配合这个索引值来访问指定的设备。

#### 2.2.6 QSPI 模式

```c
#define SFUD_USING_QSPI
```

打开/关闭 `SFUD_USING_QSPI` 宏定义

开启后，SFUD 也将支持使用 QSPI 总线连接的 Flash。

## 3.LPC55S69 & W25Q28使用SFUD



----

## 参考资料

* https://github.com/armink/SFUD
* [RT-Thread中使用SPI操作FLASH(W25Q128),并在W25Q128上挂载文件系统](https://www.cnblogs.com/Raowz/p/13069061.html)