## 什么是Matter？

Matter 协议是 [CSA（Connectivity Standards Alliance）连接标准联盟](https://csa-iot.org/) 于 2021 年推出的应用层连接协议。目前，亚马逊、谷歌、苹果等都已承诺 Alexa、Google Assistant 和 HomeKit 生态将兼容支持 Matter 协议的设备。CSA 联盟于 2022 年 10 月正式发布 Matter 1.0 版本。

Matter 协议是一种利用现有技术如 Wi-Fi、蓝牙和以太网的通信协议。

![Matter 认证](https://images.tuyacn.com/content-platform/hestia/1668761824fd0bc1f5393.png)

Matter 协议允许用户的所有智能家居设备在本地相互通信，旨在通过构建统一 “语言”，消除不同设备生态间兼容性顾虑，解决智能家居领域碎片化问题。

## Matter 认证产品分类

CSA 联盟对可认证的产品进行了分类：

| 产品分类            | CSA 联盟名称                     | 说明                                                         |
| :------------------ | :------------------------------- | :----------------------------------------------------------- |
| Matter 硬件设备认证 | Hardware Device certification    | Matter 硬件设备是一个可以提供 Matter 设备类型能力的的硬件，成品和完整功能的模组均可作为成品申请认证。例如传感器、开关、冰箱、空调、灯。 |
| Matter 软件组件认证 | Software Component certification | Matter 软件组件是在 CSA 联盟所支持的操作系统（例如电视、手机、树莓派等）下运行的一个软件。例如 App、OTA Provider。 |

说明：一个 Matter 设备同时可以支持集成以上两种设备类型，该设备可以同时申请硬件和软件两种认证。例如电视和中控。

## Matter 认证流程 - 涂鸦IOT

1. [注册 CSA 联盟会员](https://csa-iot.org/)。

2. 申请制造商 ID。

3. 采用涂鸦 Matter 方案进行 [产品设计开发](https://iot.tuya.com/pmg/solution)。

4. 确定传输协议。

   核实供应商的无线方案是否有对应联盟的证书，例如 WFA、BQB、Thread 联盟认证，网口需要准备物理层自测报告。详情请参考下文 [Matter 认证的成品有什么前提条件](https://developer.tuya.com/cn/docs/iot/matter-certification?id=Kc3uieyul71gh#prerequisite)。

5. 联系 CSA 联盟授权以及涂鸦合作的 [资质测试实验室](https://developer.tuya.com/cn/docs/iot/matter-certification?id=Kc3uieyul71gh#lab) 进行测试，测试实验室将直接向 CSA 联盟提交测试结果。

6. 向联盟提交产品认证申请，同时需要准备 DOC、PICS、传输协议符合性的自声明、网络安全自查表。

7. CSA 联盟审核申请材料，以及发证。获得证书后，才能打印 Matter 徽标出货。

8. CSA 联盟将已认证产品注册到 DCL（Distributed Compliance Ledger）系统。已认证的产品可以在 [CSA 联盟官网](https://csa-iot.org/csa-iot_products/) 查询。



> 来源：[Matter认证-涂鸦IOT开发平台](https://developer.tuya.com/cn/docs/iot/matter-certification?id=Kc3uieyul71gh)

## Matter解决方案 - 乐鑫

作为 CSA 的活跃成员，乐鑫是 Matter 协议开发的首批参与者和 Matter 发展的积极推动者。我们通过将无线通信 SoC、软件和完整的解决方案相组合，使客户能够轻松地构建各类支持 Matter 的智能家居互联设备。乐鑫提供全面的 Matter 解决方案，包括 Matter over Wi-Fi 终端设备、Matter over Thread 终端设备、Thread 边界路由器和 Matter 网关等参考设计。

![img](https://www.espressif.com/sites/all/themes/espressif/images/esp-matter/overview-zh.png?v=1.0)

#### Matter硬件平台

1. Wi-Fi 终端设备（[ESP32](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_cn.pdf)、[ESP8684](https://www.espressif.com/sites/default/files/documentation/esp8684_datasheet_cn.pdf)、[ESP32-C3](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_cn.pdf)、[ESP32-C6](https://www.espressif.com/sites/default/files/documentation/esp32-c6_datasheet_cn.pdf)、[ESP32-S3](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)）

乐鑫支持 Wi-Fi 的 ESP32、ESP32-C 和 ESP32-S 等系列 SoC 和模组，均可用于开发 Matter over Wi-Fi 终端设备。



2. Thread 终端设备（[ESP32-H2](https://www.espressif.com/zh-hans/products/socs/esp32-h2)）

乐鑫集成 IEEE 802.15.4 (Thread/Zigbee) 的 ESP32-H 系列 SoC 和模组，可用于开发 Matter over Thread 终端设备。



3. Ethernet 终端设备（[ESP32](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_cn.pdf)、[ESP32-S3](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)）

乐鑫 ESP32 和 ESP32-S 系列芯片可用于开发以太网终端设备。

* ESP32-S 系列设备需要使用外部以太网控制器来提供以太网连接。



4. Thread 边界路由器（[ESP32-H2](https://www.espressif.com/zh-hans/products/socs/esp32-h2)、[ESP32-S3](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)）

将 ESP32-H 系列 SoC 与乐鑫 Wi-Fi SoC 相组合，可搭建 Thread 边界路由器，连通 Thread 与 Wi-Fi 网络。



5. Matter 网桥（[ESP32-H2](https://www.espressif.com/zh-hans/products/socs/esp32-h2)、[ESP32-S3](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_cn.pdf)）

将 ESP32-H 系列 SoC 与乐鑫 Wi-Fi SoC 相组合，可搭建 Matter-Zigbee 桥接设备，连通 Matter 与非 Matter 网络。

![matter-overview-zh](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305311111050.png)

#### Matter Demo 视频

<video src="./matter-zh.mp4"></video>

Thread End Device（Thread终端设备）

Threda Boarder Router（边界路由器）

WI-FI End Device（WI-FI终端设备）

Contorller（基于Matter的控制器）

Matter Bridge、Zigbee End Device（Matter桥接设备）

