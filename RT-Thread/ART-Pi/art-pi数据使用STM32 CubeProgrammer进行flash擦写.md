# art-pi数据使用STM32 CubeProgrammer进行flash擦写

---

## 准备工作

* [sdk-bsp-stm32h750-realthread-artpi](https://github.com/RT-Thread-Studio/sdk-bsp-stm32h750-realthread-artpi)
* [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html#get-software)

## 固件烧写

### 1.下载算法加载

打开`sdk-bsp-stm32h750-realthread-artpi`仓库工程，在 `.\sdk-bsp-stm32h750-realthread-artpi\debug\stldr\` 目录下找到数据 flash 的 stldr 下载算法`ART-Pi_W25Q128JV.stldr`，将其复制到`.\STM32CubeProgrammer\bin\ExternalLoader`目录下

![image-20230513164311504](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305131643864.png)

