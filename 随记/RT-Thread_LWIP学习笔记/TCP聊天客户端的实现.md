# 基于RTT LWIP TCP聊天客户端的实现

---

## 基础知识

#### 1.TCP与UDP的区别

TCP(Transmission Control Protocol 传输控制协议)：是一种面向连接、可靠的、基于字节流的传输层通信协议，由IETF的RFC 793定义。

UDP(User Datagram Protocol 用户数据报协议)：是OSI(Open System Interconnection 开放式系统互联)：参考模型中的一种无连接的传输层协议，提供面向事务的简单不可靠传送服务。

OSI七层模型和TCP/IP四层模型详解请看[此处](https://kurisaw.github.io/p/%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8Bosi%E4%B8%83%E5%B1%82%E6%A8%A1%E5%9E%8Btcpip%E5%9B%9B%E5%B1%82%E6%A8%A1%E5%9E%8B/)

区别：

* TCP提供的是面向连接、可靠的数据流传输；UDP提供的是非面向连接、不可靠的数据流传输。
* TCP提供可靠的服务，通过TCP连接传送的数据：无差错、不丢失、不重复、按序到达；UDP尽最大努力交付，但不保证可靠性。
* TCP面向字节流；UDP面向报文。
* TCP仅支持点对点连接；UDP支持一对一、一对多、多对多的交互通信。
* TCP最低开销20字节（首部开销）；UDP首部开销8字节，开销小。
* TCP的逻辑同性能信道是全双工的可靠信道；UDP的逻辑通信信道是不可靠信道。

#### 2.TCP编程 服务端配置过程

* `socket():`创建一个socket
* `setsockopt():`设置socket属性
* `bind():`绑定IP地址、端口等信息到socket类上
* `listen():`开启监听
* `accept():`接收来自客户端的连接
* 收发数据：`send()、recv()、read()、write()`
* 关闭网络连接
* 关闭监听

#### 3.TCP编程 客户端配置过程

* `socket():`创建一个socket
* `setsockopt():`设置socket属性，可选
* `bind():`绑定IP地址、端口等信息到socket上
* `recvfrom():`循环接收数据
* 关闭网络连接

#### 4.UDP编程 客户端配置过程

* `socket():`创建一个socket
* `setsockopt():`设置socket属性，可选
* `bind():`绑定IP地址、端口等信息到socket上
* 设置对方的IP地址和端口等属性
* `sendto():`发送数据
* 关闭网络连接

## SAL套接字抽象层

SAL（套接字抽象层）是RT-Thread官方为避免系统对单一网络协议栈的依赖，同时也为适配更多网络协议栈类型而提供的一套网络组件，该组件主要完成对不同网络协议栈或网络实现接口的抽象并对上层一共一组标准BSD Socket API，这样开发者只需关心和使用网络应用层提供的网络接口，而无需关心底层具体网络协议栈类型和实现，极大提高了系统的兼容性。

#### 1.SAL组件主要功能特点：

* 抽象、统一多种网络协议栈接口
* 提供Socket层面的TLS加密传输特性
* 支持标准 BSD Socket API
* 统一的FD管理，便于使用read/write poll/select来操作网络功能

#### 2.SAL网络框架

![image-20230411131524312](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111315461.png)

* 应用层：提供一套标准BSD Socket API。如socket、connect等函数，用于系统中大部分网络开发应用。
* SAL套接字抽象层：RT-Thread通过该层能够适配下层不同的网络协议栈，并提供给上层统一的网络编程接口，方便不同协议栈的接入。套接字抽象层为上层应用层提供接口有：accept、connect、send、recv等。
* netdev网卡层：主要作用是解决多网卡情况设备网络连接和网络管理相关问题，通过netdev网卡层，用户可以统一管理各个网卡信息和网络连接状态，并且可以使用统一的网卡调试命令接口。
* 协议栈层：该层包括几种常用的TCP/IP协议栈，如嵌入式开发中常用的轻型TCP/IP协议栈lwip以及RT-Thread自主研发的AT Socket网络功能实现等。

附：伯克利套接字（Berkeley sockets），也称BSD Socket，伯克利套接字的应用编程接口（API）是采用C语言的进程间通信的库，经常用在计算机网络间的通信。 BSD Socket的应用编程接口已经是网络套接字的**抽象标准**。大多数其他程序语言使用一种相似的编程接口。最初是由加州伯克利大学为Unix系统开发出来。

#### 3.工作原理

SAL组件工作原理的介绍主要分为如下两部分：

* 多协议栈接入与接口函数统一抽象功能
* SAL TLS加密传输功能

#### 4.多协议接入与接口函数统一抽象功能

由于不同协议栈或网络功能的实现，其网络接口的名称各有不同，已连接函数为例，lwip协议栈中接口名称为lwip_connect，而AT Socket网络实现接口为at_connect。通过SAL组件可以完成对不同协议栈或网络实现接口的抽象和统一，组件再socket创建时通过判断传入的协议簇(domain)类型来判断使用的协议栈或网络功能。

目前RT-Thread SAL组件支持的协议栈或网络实现类型有：LWIP协议栈(AT_INET)、AT Socket协议栈(AF_AT)、WIZnet硬件 TCP/IP协议栈(AT_WIZ)[^WIZnet]。

```c
int socket(int domain, int type, int protocol);
```

为了动态适配不同协议栈或网络实现的接入，SAL组件中对于每个协议栈或者网络实现提供两种协议类型匹配方式：**主协议簇类型和次协议簇类型**，在socket创建之初收i西安判断传入协议簇类型是否存在已经支持的主协议类型，如果是则使用对应协议栈或网络实现，如果不是则判断次协议簇类型是否支持。

具体而言，主协议簇类型是指一个协议簇的最基本类型，例如 IPv4 或 IPv6。次协议簇类型则是在主协议簇类型的基础上进行扩展或增强，例如 TCP 或 UDP 协议。主协议簇类型可以被多个次协议簇类型所支持，但一个次协议簇类型只能属于一个主协议簇类型。

目前系统支持协议簇类型如下：

```c
LWIP协议栈：family = AF_INET、sec_family = AF_INET
AT Socket协议栈：family = AF_AT、sec_family = AF_INET
WIZnet硬件 TCP/IP协议栈：family = AF_WIZ
```

SAL组件的主要作用是统一BSD Socket API接口，我们以官方示例对SAL组件函数进行调用方式的实现：

* connect: SAL组件对外提供的抽象的BSD Socket API，用于统一fd管理；
* sal_connect: SAL组件中connect实现函数，用于调用底层协议栈注册的operation函数；
* lwip_connect: 底层协议栈提供的connect连接函数，在网卡初始化完成时注册到SAL组件中，最终调用的操作函数

```c
/* SAL 组件为应用层提供的标准 BSD Socket API */
int connect(int s, const struct sockaddr *name, socklen_t namelen)
{
    /* 获取 SAL 套接字描述符 */
    int socket = dfs_net_getsocket(s);

    /* 通过 SAL 套接字描述符执行 sal_connect 函数 */
    return sal_connect(socket, name, namelen);
}

/* SAL 组件抽象函数接口实现 */
int sal_connect(int socket, const struct sockaddr *name, socklen_t namelen)
{
    struct sal_socket *sock;
    struct sal_proto_family *pf;
    int ret;

    /* 检查 SAL socket 结构体是否正常 */
    SAL_SOCKET_OBJ_GET(sock, socket);

    /* 检查当前 socket 网络连接状态是否正常  */
    SAL_NETDEV_IS_COMMONICABLE(sock->netdev);
    /* 检查当前 socket 对应的底层 operation 函数是否正常  */
    SAL_NETDEV_SOCKETOPS_VALID(sock->netdev, pf, connect);

    /* 执行底层注册的 connect operation 函数 */
    ret = pf->skt_ops->connect((int) sock->user_data, name, namelen);
#ifdef SAL_USING_TLS
    if (ret >= 0 && SAL_SOCKOPS_PROTO_TLS_VALID(sock, connect))
    {
        if (proto_tls->ops->connect(sock->user_data_tls) < 0)
        {
            return -1;
        }
        return ret;
    }
#endif
    return ret;
}

/* lwIP 协议栈函数底层 connect 函数实现 */
int lwip_connect(int socket, const struct sockaddr *name, socklen_t namelen)
{
    ...
}
```

 #### 5.SAL TLS加密传输功能

在TCP、UDP等协议数据传输时，由于数据包是明文的，所以很可能被拦截，甚至被解析出数据，为了保证网络传输的安全性，需要用户在应用层和传输层之间添加SSL/TLS协议。

TLS(Transport Layer Security,传输层安全协议)是建立在传输层TCP协议之上的协议，其前身是SSL(Secure Socket Layer,安全套接字层)，主要作用是将应用层的报文进行非对称加密后再由TCP协议进行传输，实现了数据的加密安全交互。[^TLS]

对于通过的加密方式，需要使用其指定的加密接口和流程进行加密，而SAL TLS功能的主要作用是**提供Socket层面的TLS加密传输特性，抽象多种TLS处理方式，提供统一的接口用于完成TLS数据交互。**

使用流程：

* 配置开启任意网络协议栈支持(如LWIP协议栈)
* 配置开启MbedTLS软件包（目前仅支持MbedTLS类型加密方式）
* 配置开启SAL_TLS功能支持

配置完成后，需要在socket创建时传入的`potocol`类型是使用**PROTOCOL_TLS**或者**PROTOCOL_DTLS**，即可使用标准BSD Socket API接口，完成TLS连接的建立和数据的收发。

示例如下，参考RT-Threda文档中心：
```c
#include <stdio.h>
#include <string.h>

#include <rtthread.h>
#include <sys/socket.h>
#include <netdb.h>

/* RT-Thread 官网，支持 TLS 功能 */
#define SAL_TLS_HOST    "www.rt-thread.org"
#define SAL_TLS_PORT    443
#define SAL_TLS_BUFSZ   1024

static const char *send_data = "GET /download/rt-thread.txt HTTP/1.1\r\n"
    "Host: www.rt-thread.org\r\n"
    "User-Agent: rtthread/4.0.1 rtt\r\n\r\n";

void sal_tls_test(void)
{
    int ret, i;
    char *recv_data;
    struct hostent *host;
    int sock = -1, bytes_received;
    struct sockaddr_in server_addr;

    /* 通过函数入口参数url获得host地址（如果是域名，会做域名解析） */
    host = gethostbyname(SAL_TLS_HOST);

    recv_data = rt_calloc(1, SAL_TLS_BUFSZ);
    if (recv_data == RT_NULL)
    {
        rt_kprintf("No memory\n");
        return;
    }

    /* 创建一个socket，类型是SOCKET_STREAM，TCP 协议, TLS 类型 */
    if ((sock = socket(AF_INET, SOCK_STREAM, PROTOCOL_TLS)) < 0)
    {
        rt_kprintf("Socket error\n");
        goto __exit;
    }

    /* 初始化预连接的服务端地址 */
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(SAL_TLS_PORT);
    server_addr.sin_addr = *((struct in_addr *)host->h_addr);
    rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

    if (connect(sock, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) < 0)
    {
        rt_kprintf("Connect fail!\n");
        goto __exit;
    }

    /* 发送数据到 socket 连接 */
    ret = send(sock, send_data, strlen(send_data), 0);
    if (ret <= 0)
    {
        rt_kprintf("send error,close the socket.\n");
        goto __exit;
    }

    /* 接收并打印响应的数据，使用加密数据传输 */
    bytes_received = recv(sock, recv_data, SAL_TLS_BUFSZ  - 1, 0);
    if (bytes_received <= 0)
    {
        rt_kprintf("received error,close the socket.\n");
        goto __exit;
    }

    rt_kprintf("recv data:\n");
    for (i = 0; i < bytes_received; i++)
    {
        rt_kprintf("%c", recv_data[i]);
    }

__exit:
    if (recv_data)
        rt_free(recv_data);

    if (sock >= 0)
        closesocket(sock);
}

#ifdef FINSH_USING_MSH
#include <finsh.h>
MSH_CMD_EXPORT(sal_tls_test, SAL TLS function test);
#endif /* FINSH_USING_MSH */
```

## BSD Socket API

#### 1.创建套接字(socket)

为通信创建一个端点并返回一个文件描述符

```c
int socket(int domain, int type, int protocol);
```

* domain：确定协议簇
* type：数据类型
* protocol：协议

```c
# domain / 协议族类型
AF_INET		#  IPv4 协议族
AF_INET6	#  IPv6 协议族
```

```c
# type / 协议类型
/* Socket protocol types (1:TCP/2:UDP/3:RAW) */
#define SOCK_STREAM     1
#define SOCK_DGRAM      2
#define SOCK_RAW        3
```

#### 2.绑定套接字(bind)

当使用socket()创造一个套接字时，只是给定了协议簇，并没有分配地址。在套接字能够接收来自其他主机的连接时，必须bind()给它绑定一个地址。

```c
int bind(int s, const struct sockaddr *name, socklen_t namelen);
```

* s：代表socket的文件描述符
* name：指向sockaddr结构体的指针，代表要绑定的地址
* namelen：是sockaddr结构体的大小

附：SAL组件依赖netdev组件，当使用bind()函数时，可通过netdev网卡名称获取网卡对象中IP地址信息，用于将创建的Socket套接字绑定到指定的网卡对象。

来自RT-Thread文档中心，完成通过传入的网卡名称绑定该网卡IP地址并和服务器进行连接的过程：

```c
#include <rtthread.h>
#include <arpa/inet.h>
#include <netdev.h>

#define SERVER_HOST   "192.168.1.123"
#define SERVER_PORT   1234

static int bing_test(int argc, char **argv)
{
    struct sockaddr_in client_addr;
    struct sockaddr_in server_addr;
    struct netdev *netdev = RT_NULL;
    int sockfd = -1;

    if (argc != 2)
    {
        rt_kprintf("bind_test [netdev_name]  --bind network interface device by name.\n");
        return -RT_ERROR;
    }

    /* 通过名称获取 netdev 网卡对象 */
    netdev = netdev_get_by_name(argv[1]);
    if (netdev == RT_NULL)
    {
        rt_kprintf("get network interface device(%s) failed.\n", argv[1]);
        return -RT_ERROR;
    }

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
    {
        rt_kprintf("Socket create failed.\n");
        return -RT_ERROR;
    }

    /* 初始化需要绑定的客户端地址 */
    client_addr.sin_family = AF_INET;
    client_addr.sin_port = htons(8080);
    /* 获取网卡对象中 IP 地址信息 */
    client_addr.sin_addr.s_addr = netdev->ip_addr.addr;
    rt_memset(&(client_addr.sin_zero), 0, sizeof(client_addr.sin_zero));

    if (bind(sockfd, (struct sockaddr *)&client_addr, sizeof(struct sockaddr)) < 0)
    {
        rt_kprintf("socket bind failed.\n");
        closesocket(sockfd);
        return -RT_ERROR;
    }
    rt_kprintf("socket bind network interface device(%s) success!\n", netdev->name);

    /* 初始化预连接的服务端地址 */
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(SERVER_PORT);
    server_addr.sin_addr.s_addr = inet_addr(SERVER_HOST);
    rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

    /* 连接到服务端 */
    if (connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) < 0)
    {
        rt_kprintf("socket connect failed!\n");
        closesocket(sockfd);
        return -RT_ERROR;
    }
    else
    {
        rt_kprintf("socket connect success!\n");
    }

    /* 关闭连接 */
    closesocket(sockfd);
    return RT_EOK;
}

#ifdef FINSH_USING_MSH
#include <finsh.h>
MSH_CMD_EXPORT(bing_test, bind network interface device test);
#endif /* FINSH_USING_MSH */
```

#### 3.监听套接字(listen)

当有一个套接字和一个地址联系之后，listen()监听到来的连接。只适用于面向连接的模式。

```c
int listen(int s, int backlog);
```

* sockfd：代表socket的文件描述符
* backlog：一个整数，表示一次能够等待的最大连接数目。

#### 4.接收连接(accept)

当应用程序监听来自其他他主机的面向数据流的连接时，通过事件通知它，必须用accept()函数初始化连接。该函数为每个连接创建新的套接字并从监听队列中移除这个连接。

```c
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

* s：监听的套接字描述符
* addr：指向sockaddr结构体的指针，服务器地址信息
* addrlen：sockaddr结构体的大小

#### 5.建立连接(connect)

该函数用于建立与指定 socket 的连接。

```c
int connect(int s, const struct sockaddr *name, socklen_t namelen);
```

* s：套接字描述符
* name：服务器地址信息
* namelen：服务器地址结构体长度

#### 6.TCP数据发送(send)

该函数常用于 TCP 连接发送数据。

```c
int send(int s, const void *dataptr, size_t size, int flags);
```

* s：套接字描述符
* dataptr：发送的数据指针
* size：发送的数据长度
* flags：标志，一般为 0

#### 7.TCP数据接收(recv)

该函数用于TCP连接接收数据。

```c
int recv(int s, void *mem, size_t len, int flags);
```

* s：套接字描述符
* mem：接收的数据指针
* len：接收的数据长度
* flags：标志，一般为0

#### 8.UDP数据发送(sendto)

该函数用于UDP连接发送数据。

```c
int sendto(int s, const void *dataptr, size_t size, int flags, const struct sockaddr *to, socklen_t tolen);
```

* S：套接字描述符
* dataptr：发送的数据指针
* size：发送的数据长度
* flags：标志，一般为0
* to：目标结构体指针
* tolen：目标地址结构体长度

#### 9.UDP数据接收(recfrom)

该函数用于UDP连接发送数据。

```c
int recvfrom(int s, void *mem, size_t len, int flags, struct sockaddr *from, socklen_t *fromlen);
```

* S：套接字描述符
* mem：接收的数据指针
* len：接收的数据长度
* flags：标志，一般为0
* from：接收地址结构体指针
* fromlen：接收地址结构体长度

## SAL网络协议栈接入方式

网络协议栈或网络功能实现的接入，主要是对协议簇结构体的初始化和注册处理，并且添加到SAL组件中协议簇列表中，协议簇结构体定义如下：

```c
/* network interface socket opreations */
struct sal_socket_ops
{
    int (*socket)     (int domain, int type, int protocol);
    int (*closesocket)(int s);
    int (*bind)       (int s, const struct sockaddr *name, socklen_t namelen);
    int (*listen)     (int s, int backlog);
    int (*connect)    (int s, const struct sockaddr *name, socklen_t namelen);
    int (*accept)     (int s, struct sockaddr *addr, socklen_t *addrlen);
    int (*sendto)     (int s, const void *data, size_t size, int flags, const struct sockaddr *to, socklen_t tolen);
    int (*recvfrom)   (int s, void *mem, size_t len, int flags, struct sockaddr *from, socklen_t *fromlen);
    int (*getsockopt) (int s, int level, int optname, void *optval, socklen_t *optlen);
    int (*setsockopt) (int s, int level, int optname, const void *optval, socklen_t optlen);
    int (*shutdown)   (int s, int how);
    int (*getpeername)(int s, struct sockaddr *name, socklen_t *namelen);
    int (*getsockname)(int s, struct sockaddr *name, socklen_t *namelen);
    int (*ioctlsocket)(int s, long cmd, void *arg);
#ifdef SAL_USING_POSIX
    int (*poll)       (struct dfs_fd *file, struct rt_pollreq *req);
#endif
};

/* sal network database name resolving */
struct sal_netdb_ops
{
    struct hostent* (*gethostbyname)  (const char *name);
    int             (*gethostbyname_r)(const char *name, struct hostent *ret, char *buf, size_t buflen, struct hostent **result, int *h_errnop);
    int             (*getaddrinfo)    (const char *nodename, const char *servname, const struct addrinfo *hints, struct addrinfo **res);
    void            (*freeaddrinfo)   (struct addrinfo *ai);
};

/* 协议簇结构体定义 */
struct sal_proto_family
{
    int family;                                  /* primary protocol families type */
    int sec_family;                              /* secondary protocol families type */
    const struct sal_socket_ops *skt_ops;        /* socket opreations */
    const struct sal_netdb_ops *netdb_ops;       /* network database opreations */
};
```

* family：每个协议栈支持的主协议簇类型，例如lwip的为AF_INET、AT Socket为AF_AT，WIZnet为AF_WIZ。
* sec_family：每个协议栈支持的次协议簇类型，用于支持单个协议栈或网络实现时，匹配软件包中其他类型的协议簇类型。
* skt_ops：定义socket相关执行函数，如connect、send、recv等，每种协议簇都有一组通过的实现方式。
* netdb_ops：定义非socket相关执行函数，如gethostbyname、getaddrinfo、freeaddrinfo等，每种协议簇都有一组不同的实现方式。

## LWIP结构体

#### 1.struct sockaddr_in

```c
#if NETDEV_IPV4
/* members are in network byte order */
struct sockaddr_in
{
    uint8_t        sin_len;
    sa_family_t    sin_family;
    in_port_t      sin_port;
    struct in_addr sin_addr;
#define SIN_ZERO_LEN 8
    char            sin_zero[SIN_ZERO_LEN];
};
#endif /* NETDEV_IPV4 */

#if NETDEV_IPV6
struct sockaddr_in6
{
  uint8_t         sin6_len;      /* length of this structure    */
  sa_family_t     sin6_family;   /* AF_INET6                    */
  in_port_t       sin6_port;     /* Transport layer port #      */
  uint32_t        sin6_flowinfo; /* IPv6 flow information       */
  struct in6_addr sin6_addr;     /* IPv6 address                */
  uint32_t        sin6_scope_id; /* Set of interfaces for scope */
};
#endif /* NETDEV_IPV6 */
```

* _len：表示该结构体的长度
* _family：表示地址簇，通常`AF_INET`表示IPV4地址，`AF_INET6`表示IPV6地址
* _port：表示端口号
* _addr：存储地址簇的结构体
* _zero：预留字段，用0填充
* sin6_flowinfo：存储IPV6流信息
* sin6_scope_id：存储一组接口表示符，用于确定IPV6地址的作用域

---

## 附录

[^TLS]: 在 TLS 协议中，使用了非对称加密和对称加密两种加密方式。其中，**非对称加密主要用于密钥协商和身份认证，而对称加密则用于数据传输的加密和解密**。在TLS握手过程中，客户端和服务器会相互发送自己的公钥，并通过对方的公钥加密生成一个随机数的方式协商出用来进行对称加密的对称密钥。这个对称密钥就是用非对称加密算法加密后的数据包。接收方拿到这个数据包后，使用自己的私钥进行解密，获取生成的对称密钥。然后，双方就开始使用协商好的对称密钥进行数据传输。接收方会利用对称密钥对收到的数据进行解密，得到明文数据。这样，在整个数据传输过程中，只有公钥被公开，密钥等关键信息都是使用非对称加密算法进行加密传输的，保证了安全性。总之，在 TLS 协议中，接收方通过使用自己的私钥解密协商出的对称密钥，从而完成对加密数据的解析。这个过程是整个 TLS 协议中非常重要的一个环节，确保了加密数据在传输过程中的安全性和可靠性。
[^WIZnet]:WIZnet的硬件TCP/IP协议栈采用了TOE（TCP/IP Core Offload Engine）技术，将TCP/IP协议栈等网络处理功能转移到专用硬件中，从而减少了CPU的负担，提高了整个系统的性能和稳定性。同时，WIZnet的硬件TCP/IP协议栈还支持多种网络协议，并提供了Socket API封装等高层次接口，方便用户进行开发和集成。

