# linxu平台用户手册

## 一、QT4.8移植



1.1 交叉编译器的安装
将光盘中的交叉编译工具arm-2009q3.tar.bz2复制到ubuntu的任意目录并解压： cp $yourcrosscompiledir/arm-2009q3.tar.bz2 ~ tar xvf arm-2009q3.tar.bz2 -C / 这时，交叉编译工具会被安装到/usr/local/arm/目录。 

交叉编译器环境变量的设置： 编辑/etc/profile，在最末尾处增加： export PATH=/usr/local/arm/arm-2009q3/bin/:$PATH 执行如下指令让环境变量生效： source /etc/profile



1.2 安装QT4.8源码包
将光盘中的QT4.8源码包qt_x210v3_130712.tar.bz2拷贝到ubuntu的用户目录并解压： cp qt_x210v3_130712.tar.bz2 ~ tar xvf qt_x210v3_130712.tar.bz2，解压内容如下图所示：

![image-20220418122458413](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220418122458413.png)



1.3 编译uboot
注意，引导qt 的uboot和引导android的uboot有一点差异，nand flash分区做了调整，预了解详细差异，用户可对比源码。 注意：编译前，请务必确认手上的开发板是nand flash平台还是inand平台。
如果是nand flash平台，执行如下指令编译:

	./mk -un

这时，在uboot目录将会生成我们需要的映像uboot_nand.bin，同时会拷贝到映像所释放的目录，即release目录。
如果是inand平台，执行如下指令编译：
	
	./mk -ui 

这时，在uboot目录将会生成我们需要的映像uboot_inand.bin，同样也会将映像拷贝到release目录。

![image-20220418123019904](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220418123019904.png)

1.4 编译内核
执行如下指令编译内核： 

	./mk -k 

`注意：无论是inand平台还是nand flash平台，都可以用同一个zImage文件，用同一个内核。 这时，在kernel/arch/arm/boot下会生成我们需要的映像zImage，同时会将QT的内核映像zImage-qt拷贝到release目录。`

注意，这时release目录还会生成一个zImage-initrd的映像，它主要用于xboot升级映像，使用uboot时一定要烧写zImage-qt这个映像。

![image-20220418123330490](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20220418123330490.png)

1.5 编译xboot
我们精心编写了一套自主研发的bootloader，无论在代码管理，还是固件升级，批量生产，以及代码调试方面，相对uboot都有很强的优势。xboot最终生成的映像有十几M，甚至更大，而uboot只有几百K大小，这是因为xboot它本身已经将前面的zImage-qt和zImage-initrd都打包到xboot中了，甚至还可以将WINCE的eboot也打包到xboot中。详细原理，可认真琢磨xboot源码。也正因为xboot的这种机制，因此务必先编译内核，再编译xboot。而且，用户在更新内核驱动后，如果使用xboot机制，需将新的内核映像打包到xboot，再更新xboot到开发板。

 执行如下指令编译xboot：

	./mk -x

 编译完成后，xboot会被拷贝到release目录。

1.6 编译文件系统

我们使用buildroot工具配置文件系统，在QT源码根目录下执行如下指令编译文件系统：

	./mk -r 

编译完成后，我们需要的文件系统文件rootfs.tar存放在buildroot/output/images目录，编译脚本mk会自动将它复制到release目录中。有了rootfs.tar，我们就能制作我们想要的文件系统映像了。
