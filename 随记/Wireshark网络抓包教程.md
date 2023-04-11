> 来源：[转自：WireShark 抓包使用教程--详细](https://blog.csdn.net/HarveyH/article/details/113731485)

Wireshark是非常流行的网络封包分析软件，可以截取各种网络数据包，并显示数据包详细信息。常用于开发测试过程各种问题定位。本文主要内容包括：

1、Wireshark软件下载和安装以及Wireshark主界面介绍。

2、WireShark简单抓包示例。通过该例子学会怎么抓包以及如何简单查看分析数据包内容。

3、Wireshark过滤器使用。通过过滤器可以筛选出想要分析的内容。包括按照协议过滤、端口和主机名过滤、数据包内容过滤。

## Wireshark软件安装

软件下载路径：[wireshark官网](https://www.wireshark.org/)。按照系统版本选择下载，下载完成后，按照软件提示一路Next安装。

如果你是Win10系统，安装完成后，选择抓包但是不显示网卡，下载win10pcap兼容性安装包。下载路径：[win10pcap兼容性安装包](http://www.win10pcap.org/download/)

## **Wireshark 开始抓包示例**

先介绍一个使用wireshark工具抓取ping命令操作的示例，让读者可以先上手操作感受一下抓包的具体过程。

1、打开wireshark 2.6.5，主界面如下：

**![](https://img-blog.csdnimg.cn/img_convert/5ed67922ae9f1002600d67dfe1431c44.png)**

2、选择菜单栏上Capture \-> Option，勾选WLAN网卡（这里需要根据各自电脑网卡使用情况选择，简单的办法可以看使用的IP对应的网卡）。点击Start。启动抓包。

![](https://img-blog.csdnimg.cn/img_convert/84dee318f475ec3ba3d9071c81a6c029.png)

3、wireshark启动后，wireshark处于抓包状态中。

![](https://img-blog.csdnimg.cn/img_convert/75e0d4f2e939643a14a0b41fad5cd882.png)

4、执行需要抓包的操作，如ping www.baidu.com。

5、操作完成后相关数据包就抓取到了。为避免其他无用的数据包影响分析，可以通过在过滤栏设置过滤条件进行数据包列表过滤，获取结果如下。说明：ip.addr == 119.75.217.26 and icmp 表示只显示ICPM协议且源主机IP或者目的主机IP为119.75.217.26的数据包。

![](https://img-blog.csdnimg.cn/img_convert/e8656894f66f85c24f0110953c38c2f6.png)

5、wireshark抓包完成，就这么简单。关于wireshark过滤条件和如何查看数据包中的详细内容在后面介绍。

## Wireshakr抓包界面

![](https://img-blog.csdnimg.cn/img_convert/fcb90712262e80033bcf44232b0c1b74.png)

说明：数据包列表区中不同的协议使用了不同的颜色区分。协议颜色标识定位在菜单栏View \--> Coloring Rules。如下所示

![](https://img-blog.csdnimg.cn/img_convert/36ec30da1670d8e49df5dc1444f285e6.png)

**WireShark 主要分为这几个界面**

1\. Display Filter\(显示过滤器\)， 用于设置过滤条件进行数据包列表过滤。菜单路径：Analyze \--> Display Filters。

![](https://img-blog.csdnimg.cn/img_convert/f442caaf4fe8c4f9c7a97e87bf309ce0.png)

2\. Packet List Pane\(数据包列表\)， 显示捕获到的数据包，每个数据包包含编号，时间戳，源地址，目标地址，协议，长度，以及数据包信息。 不同协议的数据包使用了不同的颜色区分显示。

![](https://img-blog.csdnimg.cn/img_convert/a2d54518e5c75ee2cf578f32ae8df97c.png)

3\. Packet Details Pane\(数据包详细信息\), 在数据包列表中选择指定数据包，在数据包详细信息中会显示数据包的所有详细信息内容。数据包详细信息面板是最重要的，用来查看协议中的每一个字段。各行信息分别为

（1）Frame: 物理层的数据帧概况

（2）Ethernet II: 数据链路层以太网帧头部信息

（3）Internet Protocol Version 4: 互联网层IP包头部信息

（4）Transmission Control Protocol: 传输层T的数据段头部信息，此处是TCP

（5）Hypertext Transfer Protocol: 应用层的信息，此处是HTTP协议

![](https://img-blog.csdnimg.cn/img_convert/1baa1297c4d1fc3014245f8feb2e512a.png)

**TCP包的具体内容**

从下图可以看到wireshark捕获到的TCP包中的每个字段。

![](https://img-blog.csdnimg.cn/img_convert/3b4c5189860258e07bf02581bca601ad.png)

4\. Dissector Pane\(数据包字节区\)。

## Wireshark过滤器设置

初学者使用wireshark时，将会得到大量的冗余数据包列表，以至于很难找到自己自己抓取的数据包部分。wireshar工具中自带了两种类型的过滤器，学会使用这两种过滤器会帮助我们在大量的数据中迅速找到我们需要的信息。

（1）抓包过滤器

捕获过滤器的菜单栏路径为Capture \--> Capture Filters。用于**在抓取数据包前设置。**

**![](https://img-blog.csdnimg.cn/img_convert/014ee1bcf099455fa556b54a511ce770.png)**

如何使用？可以在抓取数据包前设置如下。

![](https://img-blog.csdnimg.cn/img_convert/8c80422806e86197d5c188192c97b01e.png)

ip host 60.207.246.216 and icmp表示只捕获主机IP为60.207.246.216的ICMP数据包。获取结果如下：

![](https://img-blog.csdnimg.cn/img_convert/b9ace8e1f433501f039e34cb8344dd89.png)

（2）显示过滤器

显示过滤器是用于在抓取数据包后设置过滤条件进行过滤数据包。通常是在抓取数据包时设置条件相对宽泛，抓取的数据包内容较多时使用显示过滤器设置条件顾虑以方便分析。同样上述场景，在捕获时未设置捕获规则直接通过网卡进行抓取所有数据包，如下

![](https://img-blog.csdnimg.cn/img_convert/0e42c52596ff9dcdb07225fa4486d717.png)

执行ping www.huawei.com获取的数据包列表如下

![](https://img-blog.csdnimg.cn/img_convert/3c9329d703282867bc996701a7b76070.png)

观察上述获取的数据包列表，含有大量的无效数据。这时可以通过设置显示器过滤条件进行提取分析信息。ip.addr == 211.162.2.183 and icmp。并进行过滤。

![](https://img-blog.csdnimg.cn/img_convert/eceb2ac7136208e2cd530397f54da716.png)

上述介绍了抓包过滤器和显示过滤器的基本使用方法。**在组网不复杂或者流量不大情况下，使用显示器过滤器进行抓包后处理就可以满足我们使用。**下面介绍一下两者间的语法以及它们的区别。

**wireshark过滤器表达式的规则**

1、抓包过滤器语法和实例

抓包过滤器类型Type（host、net、port）、方向Dir（src、dst）、协议Proto（ether、ip、tcp、udp、http、icmp、ftp等）、逻辑运算符（\&\& 与、|| 或、！非）

（1）协议过滤

比较简单，直接在抓包过滤框中直接输入协议名即可。

TCP，只显示TCP协议的数据包列表

HTTP，只查看HTTP协议的数据包列表

ICMP，只显示ICMP协议的数据包列表

（2）IP过滤

host 192.168.1.104

src host 192.168.1.104

dst host 192.168.1.104

（3）端口过滤

port 80

src port 80

dst port 80

（4）逻辑运算符\&\& 与、|| 或、！非

src host 192.168.1.104 \&\& dst port 80 抓取主机地址为192.168.1.80、目的端口为80的数据包

host 192.168.1.104 || host 192.168.1.102 抓取主机为192.168.1.104或者192.168.1.102的数据包

！broadcast 不抓取广播数据包

2、显示过滤器语法和实例

（1）比较操作符

比较操作符有== 等于、！= 不等于、> 大于、\< 小于、>= 大于等于、\<=小于等于。

（2）协议过滤

比较简单，直接在Filter框中直接输入协议名即可。**注意：协议名称需要输入小写。**

tcp，只显示TCP协议的数据包列表

http，只查看HTTP协议的数据包列表

icmp，只显示ICMP协议的数据包列表

![](https://img-blog.csdnimg.cn/img_convert/73eeb45b820993816d528a0866eb4913.png)

（3） ip过滤

ip.src ==192.168.1.104 显示源地址为192.168.1.104的数据包列表

ip.dst==192.168.1.104, 显示目标地址为192.168.1.104的数据包列表

ip.addr == 192.168.1.104 显示源IP地址或目标IP地址为192.168.1.104的数据包列表

![](https://img-blog.csdnimg.cn/img_convert/e81f0e34b7aa61ee14d1948cae8edabf.png)

（4）端口过滤

tcp.port ==80, 显示源主机或者目的主机端口为80的数据包列表。

tcp.srcport == 80, 只显示TCP协议的源主机端口为80的数据包列表。

tcp.dstport == 80，只显示TCP协议的目的主机端口为80的数据包列表。

![](https://img-blog.csdnimg.cn/img_convert/a2e60793fb7dd5f5c918647b686f213d.png)

（5） Http模式过滤

http.request.method=="GET", 只显示HTTP GET方法的。

（6）逻辑运算符为 and/or/not

过滤多个条件组合时，使用and/or。比如获取IP地址为192.168.1.104的ICMP数据包表达式为ip.addr == 192.168.1.104 and icmp

![](https://img-blog.csdnimg.cn/img_convert/2b5efac726f9cbb588528d4e717a77f7.png)

（7）按照数据包内容过滤。假设我要以IMCP层中的内容进行过滤，可以单击选中界面中的码流，在下方进行选中数据。如下

![](https://img-blog.csdnimg.cn/img_convert/66c90de5bd699e53eb2f00c5522469cd.png)

右键单击选中后出现如下界面

![](https://img-blog.csdnimg.cn/img_convert/7879f76bfa990d8211d0b080130f76e5.png)

选中Select后在过滤器中显示如下

![](https://img-blog.csdnimg.cn/img_convert/daad1d916c4e9a81e14bab59da20c7b9.png)

后面条件表达式就需要自己填写。如下我想过滤出data数据包中包含"abcd"内容的数据流。**包含的关键词是contains 后面跟上内容。**

![](https://img-blog.csdnimg.cn/img_convert/8dea81f8c266f72ed84a52536e062380.png)

看到这， 基本上对wireshak有了初步了解。

## Wireshark抓包分析TCP三次握手

（1）TCP三次握手连接建立过程

Step1：客户端发送一个SYN=1，ACK=0标志的数据包给服务端，请求进行连接，这是第一次握手；

Step2：服务端收到请求并且允许连接的话，就会发送一个SYN=1，ACK=1标志的数据包给发送端，告诉它，可以通讯了，并且让客户端发送一个确认数据包，这是第二次握手；

Step3：服务端发送一个SYN=0，ACK=1的数据包给客户端端，告诉它连接已被确认，这就是第三次握手。TCP连接建立，开始通讯。

![](https://img-blog.csdnimg.cn/img_convert/7b00080eaab9ec04864f53d8a2e2d6e8.png)

（2）wireshark抓包获取访问指定服务端数据包

Step1：启动wireshark抓包，打开浏览器输入www.huawei.com。

Step2：使用ping www.huawei.com获取IP。

![](https://img-blog.csdnimg.cn/img_convert/8bde24c6d4ffdb3a1d9d51d35e9e6536.png)

Step3：输入过滤条件获取待分析数据包列表 ip.addr == 211.162.2.183

![](https://img-blog.csdnimg.cn/img_convert/ea366e5a0085cb75801c57a4cdef4f5b.png)

图中可以看到wireshark截获到了三次握手的三个数据包。第四个包才是HTTP的， 这说明HTTP的确是使用TCP建立连接的。

**第一次握手数据包**

客户端发送一个TCP，标志位为SYN，序列号为0， 代表客户端请求建立连接。 如下图。

![](https://img-blog.csdnimg.cn/img_convert/6707320072f1fe8a8563751e3ff02287.png)

数据包的关键属性如下：

SYN ：标志位，表示请求建立连接

Seq = 0 ：初始建立连接值为0，数据包的相对序列号从0开始，表示当前还没有发送数据

Ack =0：初始建立连接值为0，已经收到包的数量，表示当前没有接收到数据

**第二次握手的数据包**

服务器发回确认包, 标志位为 SYN,ACK. 将确认序号\(Acknowledgement Number\)设置为客户的I S N加1以.即0+1=1, 如下图

![](https://img-blog.csdnimg.cn/img_convert/49fb1fa43834316fb051327817fba5b5.png)

数据包的关键属性如下：

\[SYN + ACK\]: 标志位，同意建立连接，并回送SYN+ACK

Seq = 0 ：初始建立值为0，表示当前还没有发送数据

Ack = 1：表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位。（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据）

**第三次握手的数据包**

客户端再次发送确认包\(ACK\) SYN标志位为0,ACK标志位为1.并且把服务器发来ACK的序号字段+1,放在确定字段中发送给对方.并且在数据段放写ISN的+1, 如下图:

![](https://img-blog.csdnimg.cn/img_convert/858a3c327a55ae0c4372301343736208.png)

数据包的关键属性如下：

ACK ：标志位，表示已经收到记录

Seq = 1 ：表示当前已经发送1个数据

Ack = 1 : 表示当前端成功接收的数据位数，虽然服务端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据\)。

就这样通过了TCP三次握手，建立了连接。开始进行数据交互

![](https://img-blog.csdnimg.cn/img_convert/eece4698a73f8f2bf1a3c54216ccfe6b.png)

下面针对数据交互过程的数据包进行一些说明：

![](https://img-blog.csdnimg.cn/img_convert/450db090b7902c51723166707219a84a.png)

数据包的关键属性说明

Seq: 1

Ack: 1: 说明现在共收到1字节数据

![](https://img-blog.csdnimg.cn/img_convert/4a438146c4bc253be107ec140564e01f.png)

Seq: 1  
Ack: 951: 说明现在服务端共收到951字节数据

在TCP层，有个FLAGS字段，这个字段有以下几个标识：SYN, FIN, ACK, PSH, RST, URG。如下

![](https://img-blog.csdnimg.cn/img_convert/796d52aca7aca51ba9ce66d493129344.png)

其中，对于我们日常的分析有用的就是前面的五个字段。它们的含义是：SYN表示建立连接，FIN表示关闭连接，ACK表示响应，PSH表示有DATA数据传输，RST表示连接重置。

## Wireshark分析常用操作

调整数据包列表中时间戳显示格式。调整方法为View \-->Time Display Format \--> Date and Time of Day。调整后格式如下：

![](https://img-blog.csdnimg.cn/img_convert/897c9c8421dea13da994ab049cbd7279.png)