第一步:制作uboot镜像
(1)更改Makefile，架构选择arm，交叉编译.工具链
(2)配置，make x210_ sd_ config第一-步:制作uboot镜像
(1)更改Makefile，架构选择arm，交叉编译工具链;
(2)配置，make x210_ sd_ config;
(3)编译make -j4
(4)进入烧录的文件夹，修改sd_ fusing.sh文件， 查看mkbl1的属性信息
./sd_ fusing.sh /dev/ sdb

第二步:制作内核，支持nfs启动
(1) make x210ii_ qt_ defconfig;
(2) make menuconfig; 问题有两个，第一个，界而太小;第二个，依赖库
(3) 界而化配置，支持NFS启动

第三步: busybox
(1)配置，修改Makefile;
(2)编译，出现Error, sync.o， 在make menuconfig中修改， 重新编译;
(3)安装，得到最小根文件系统，make menuconfig中安装路径

自行搭建nfs服务器
uboot配置启动参数，启动命令
set bootcmd 'tftp 30008000 zImage; bootm 30008000'
set bootargs root=/dev/nfs nfsroot=192.168.1.141:/root/rootfs ip=192.168.1.10:192.16



uboot`内核`配置
1.进入到x210_bsp/kernel: ` make disclean`---清除所有配置

2.内核配置

`ls arch/arm/configs`

`make x210ii_qt_defconfig`

![image-20220428101602414](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220428101602414.png)

`make menuconfig`

3.网络配置（参考x210挂载根文件系统）

网络配置结束后`make -j4`

`cp arch/arm/boot/zImage /tftpboot/`---开发板下载内核所需文件

4.busybox配置

（参考x210挂载根文件系统）

5.nfs挂载目录

6.设置busybox安装路径

```
Busybox Settings --->
	Installation Options ("make install" behavior) --->
		(./)BusyBox installation prefix) //这里设置安装路径
```

![image-20220428103501462](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220428103501462.png)

7.根文件系统配置

（参考x210挂载根文件系统）