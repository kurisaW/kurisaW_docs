交流群号：245022761\(IT项目交流群\)

**目录**

[开发环境](#t1)

[运行环境](#t2)

[系统功能](#t3)

[运行效果](#t4)

[     服务端（电脑）](#t5)

[    客户端（开发板）](#t6)

[实现思路](#t7)

[    如何提高图片传输速率？](#t8)

[    如何实现远程控制桌面鼠标？](#t9)

[    坐标转换](#t10)

[    获取本地IP](#t11)

---

# 开发环境

            开发语言：C++

            开发工具:  Qt Creator

            [交叉编译](https://so.csdn.net/so/search?q=%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91&spm=1001.2101.3001.7020)环境：Unbuntu 16.04   （[【QT】ubuntu交叉编译Qt程序\_俊俊的博客-CSDN博客\_交叉编译qt程序](https://blog.csdn.net/qq_40602000/article/details/97287540 "【QT】ubuntu交叉编译Qt程序_俊俊的博客-CSDN博客_交叉编译qt程序")）

# 运行环境

           服务端：Windows平台

           客户端：ARM平台（gec6818   [首页](http://www.gec-lab.com/arm/show/72.html "首页")） 

# 系统功能

          1、window桌面实时在[开发板](https://so.csdn.net/so/search?q=%E5%BC%80%E5%8F%91%E6%9D%BF&spm=1001.2101.3001.7020)上显示

          2、开发板远程控制桌面鼠标点击（双击、单机、鼠标移动等）

          3、开发板文件分享到window桌面上

          4、多人聊天

# 运行效果

##      服务端（电脑）

        上方区域：左边手形按钮启动服务开始投屏，STOP按钮结束投屏\(QPushButton\)

        中间区域：显示所有聊天信息（IP+内容）\(QTextBrowser\)

        下方区域：显示文件接收的进度\(QProgressBar\)

         ![](https://img-blog.csdnimg.cn/20190805192633578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNjAyMDAw,size_16,color_FFFFFF,t_70)

##     客户端（开发板）

      左上区域：投屏显示区域（QScrollArea + QLabel）

      左下区域：编辑发送内容（QLineEdit）

      右上区域：显示发送的内容\(QTextBrowser\)

      右下区域：发送按钮、文件选择传输按钮\(QPushButton\)

      ![](https://img-blog.csdnimg.cn/20190805193515655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNjAyMDAw,size_16,color_FFFFFF,t_70)

# 实现思路

         本系统采用TCP、UDP协议进行数据传输，TCP负责大文件传输（PPT、视频），UDP负责文字、图片传输

        发送端（共享屏幕服务端\) :获取桌面图像 \-》图像压缩-》编码成PNG-》把图片序列化为byte-》UDP广播发出

        接收端：监听端口数据-》接收数据到内存数据里-》完成接收，QPixmap从内存数组里加载图像

        TCP传输文件参考：[【QT】基于TCP的文件传输系统\_俊俊的博客-CSDN博客\_qt tcp文件传输](https://blog.csdn.net/qq_40602000/article/details/97786090 "【QT】基于TCP的文件传输系统_俊俊的博客-CSDN博客_qt tcp文件传输")

        UDP通信参考：[【QT】QT中UDP通信的实现（单播、组播和广播）\_俊俊的博客-CSDN博客\_qt实现udp数据发送与接收](https://blog.csdn.net/qq_40602000/article/details/97953305 "【QT】QT中UDP通信的实现（单播、组播和广播）_俊俊的博客-CSDN博客_qt实现udp数据发送与接收")

##     **如何提高图片传输速率？**

      1、压缩图片大小，使用QT自带的压缩函数scaled\(\)

       该函数有2中缩放方式，各有优缺，”快速缩放”得到的图片质量不佳，”平滑缩放”质量很好但速度欠佳

       使用技巧是调用scaled\(\)将图片缩放至中等图片img.scaled\(800, 480, Qt::KeepAspectRatio, Qt::FastTransformation\)进行快速压缩

       然后在进行二次缩放scaled\(400, 240, Qt::KeepAspectRatio, Qt::SmoothTransformation\)进行平滑缩放，这样既保障图片的质量，又提高了压缩速度。

      2、减少图片传输次数，服务端判断每帧图片是否发送变化，如果发生变化则传输，如果没有变化则不传输

      3、关键代码

```cpp
void MainScreen::SendPicture(){    QDesktopWidget* desktop=QApplication::desktop();    QPixmap screen=QPixmap::grabWindow(desktop->winId());//截取桌面    //快速缩放(FastTransformation)、平滑缩放(SmoothTransformation)    //1920x1080    screen = screen.scaled(800, 480, Qt::KeepAspectRatio, Qt::FastTransformation).            scaled(400, 240, Qt::KeepAspectRatio, Qt::SmoothTransformation);     //screen=screen.scaled(320,240,Qt::IgnoreAspectRatio,Qt::SmoothTransformation);    newpixmap = screen;    //图片变化时则发送，否则不发送    if(oldpixmap.toImage()!=newpixmap.toImage())    {        qDebug("图片发生变化");        oldpixmap = newpixmap;        QImage image;        image=newpixmap.toImage();        QByteArray ba;                buffer(&ba);         buffer.open(QIODevice::ReadWrite);        image.save(&buffer,"PNG");        qint64 res;        QHostAddress address;        address.setAddress(ipAddress);        //QHostAddress("192.168.12.208")\QHostAddress::Broadcast        //广播        if((res=this->_Socket->writeDatagram(ba,ba.length(),QHostAddress("192.168.12.208"),ports))!=ba.length()){            //qDebug()<<res<<ba.length();            return;        }    }    else    {          qDebug("图片无变化");    }}
```

##     **如何实现远程控制桌面鼠标？**

      1、（监听事件）客户端给显示区域添加事件过滤器，监听鼠标（或者触摸）事件

        MouseButtonPress = 2,    // 鼠标按下

        MouseButtonRelease = 3,  // 鼠标释放

        MouseButtonDblClick = 4, // 鼠标双击

        MouseMove = 5,           // 鼠标移动

        Enter = 10,              // 鼠标进入显示区域

        Leave = 11,              // 鼠标离开显示区域

        Wheel = 31,              // 鼠标滚动

```cpp
bool OtherScreen::eventFilter(QObject *obj, QEvent *event){    //监听触摸屏的单击事件    if(obj==label)    {        if(event->type()==QEvent::MouseButtonPress)        {                  clicktype = "MouseButtonPress";            qDebug()<<"鼠标按下";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::MouseButtonRelease)        {            clicktype = "MouseButtonRelease";            qDebug()<<"鼠标释放";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::MouseButtonDblClick)        {            clicktype = "MouseButtonDblClick";            qDebug()<<"鼠标双击";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::Enter)        {            clicktype = "Enter";            qDebug()<<"鼠标进入";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::Leave)        {            clicktype = "Leave";            qDebug()<<"鼠标离开";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::MouseMove)        {            clicktype = "MouseMove";            qDebug()<<"鼠标移动";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::Wheel)        {            clicktype = "MouseMove";            qDebug()<<"鼠标滚动";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else if(event->type()==QEvent::HoverEnter)        {            clicktype = "MouseMove";            qDebug()<<"鼠标光标进入一个悬停小部件";            QMouseEvent* qEvent=(QMouseEvent*)event;            sendMessage(qEvent->pos());//将点击坐标点发送至服务端            return true;        }        else        {            clicktype = " ";            return false;        }    }    else    {        return OtherScreen::eventFilter(obj,event);    }}
```

           2、（模拟鼠标）服务端接收客户端发来的触发事件类型，调用windows鼠标触发事件，从而到达点击、双击、移动

          引入头文件 #include \<windows.h>

          **Windows鼠标按键事件**

             ![](https://img-blog.csdnimg.cn/20190805203537899.png)

            ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS81MDEyNjBkYzNlMGM0OTJmZDRiZmNiYWFhODBiODIzNy94bWxub3RlL0E4NThGMzk1NTE5QTQ5RTNCRENDMTMwMjgxNUMzODM1LzVDM0JFQjNEMjYyQzQ5MEVCNzhFQkI1RkY0MjhFMjgzLzMyODM0)

     **鼠标移动事件**

          ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS81MDEyNjBkYzNlMGM0OTJmZDRiZmNiYWFhODBiODIzNy94bWxub3RlL0E4NThGMzk1NTE5QTQ5RTNCRENDMTMwMjgxNUMzODM1LzZBMzRFMjk1NkJEQzQ2MTRCMzdERThEMEZERjUzN0UwLzMyODMx)

```cpp
//模拟鼠标点击事件void MainScreen::simulate_mouse(QString type, QPoint point){    //鼠标按下    if(type == "MouseButtonPress")    {        qDebug()<<"鼠标按下"<<point;        ::mouse_event(MOUSEEVENTF_LEFTDOWN|MOUSEEVENTF_LEFTUP,point.x(),point.y(),0,0);    }    //鼠标进去和离开    else if(type == "Enter")    {        //SetCursorPos(point.x(),point.y());//设置鼠标位置，跟随开发板的触摸位置        qDebug()<<"鼠标进入"<<point;    }    //鼠标移动    else if(type == "MouseMove")    {        SetCursorPos(point.x(),point.y());        ::mouse_event(MOUSEEVENTF_LEFTDOWN,0,0,0,0);        ::mouse_event(MOUSEEVENTF_MOVE,0,0,0,0);        qDebug()<<"鼠标移动"<<point;    }    else    {        SetCursorPos(point.x(),point.y());//设置鼠标位置，跟随开发板的触摸位置    }}
```

##       坐标转换

根据开发板和电脑的屏幕尺寸大小按比例换算，代码如下

```cpp
//计算获得PC端的点击坐标QPoint MainScreen::clickPoint(QPoint point, QRect size){    QRect rect;    QPoint click;    rect=QApplication::desktop()->screenGeometry();    qDebug()<<rect;    click.setX(rect.width()*((float)point.x()/(float)size.width()));    click.setY(rect.height()*((float)point.y()/(float)size.height()));    //qDebug()<<"转换："<<click;    ui->textBrowser->append("<font color=gray>坐标：x->"                            + QString::number(click.x())                            + "  y->"                            + QString::number(click.y())                            + "\r\n</font>");    return click;}
```

##        获取本地IP

```cpp
/* *获取本机Ip地址 */void MainScreen::getIp(){    QList<QHostAddress> ipAddressesList = QNetworkInterface::allAddresses();    // use the first non-localhost IPv4 address    for (int i = 0; i < ipAddressesList.size(); ++i) {        if (ipAddressesList.at(i) != QHostAddress::LocalHost &&            ipAddressesList.at(i).toIPv4Address()) {            ipAddress = ipAddressesList.at(i).toString();            break;        }    }    // if we did not find one, use IPv4 localhost    if (ipAddress.isEmpty())        ipAddress = QHostAddress(QHostAddress::LocalHost).toString();    qDebug()<<"IP为："<<ipAddress;}
```