# can't run '/etc/init.d/rcS': Permission denied问题解决

Freeing init memory: 420K                                                      
init started:  BusyBox v1.1.2 (2010.03.31-15:37+0000) multi-call binary        
Starting pid 768, console /dev/console: '/etc/init.d/rcS'                      
Bummer, could not run '/etc/init.d/rcS': Permission denied                     
Bummer, could not run '/etc/init.d/rcS': Permission denied 





解决方法：

cd /usr/local/src/EduKit-IV/Mini2410/simple/6.3-busybox/root-mini/etc#

 chmod -R 777 init.d/*

 


sudo sh ramdisk-install.sh