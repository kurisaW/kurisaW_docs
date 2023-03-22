EST + / :跳转到指定代码

![image-20220508211313572](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220508211313572.png)



EST+:+N  （跳转到指定行）

![image-20220508233344494](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220508233344494.png)





先排除所有错误，然后执行make distclean

然后输入

```
./configure  -prefix /opt/QT5.6.2 -opensource -confirm-license -release -shared -accessibility -c++std c++98 -xplatform linux-arm-gnueabi-g++ -qpa linuxfb -linuxfb -qreal float -pch -qt-zlib -qt-libjpeg -qt-libpng -no-sse2 -no-largefile -no-qml-debug -no-glib -no-gtkstyle -no-opengl -nomake tools -nomake examples -tslib -skip qt3d -skip qtcanvas3d -skip qtdoc -skip qtwayland -I /opt/tslib/include -L /opt/tslib/lib
```

执行make编译





Qt5.3.2安装路径：/opt/Qt5.3.2

进入Qt5.3.2创建工程

`root@ubuntu:/opt/Qt5.3.2/Tools/QtCreator/bin# ./qtcreator`







触摸板校正

export QWS_MOUSE_PROTO=Tslib:/dev/input/event1  //选择触摸板
export QWS_DISPLAY=LinuxFb:/dev/fb0  //指定设备
export QWS_DISPLAY="LinuxFb:mmWidth50:mmHeight130:0"  //设置字体显示
export QWS_SIZE=800x480  //设置屏幕大小