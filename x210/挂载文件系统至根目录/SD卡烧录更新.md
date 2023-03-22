SD卡烧录更新



另外找一张SD卡连接到虚拟机，进入到/x210_bsp/uboot/sd_fusing下，执行./sd_fusing.sh /dev/sdb







make x210_sd_config

进入sd_fusing/ 烧录uboot 如果烧录错误，那么说明文件是在32位下执行的，解决这个问题只需要make clean,然后make，这时候就能够生成适合虚拟机的格式了

返回uboot/，执行make,可以看到u-boot.bin文件，进入sd_fusing/,执行vi sd_fusing.sh在下面改成u-boot.bin

![image-20220503195615800](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220503195615800.png)





进入kernel/，执行make clean 

执行make x210ii_qt_defconfig







