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

## LWIP API

#### 1.lwip_socket()

```c
int lwip_socket(int domain, int type, int protocol);
```

* domain：确定协议簇
* type：协议类型
* protocol：协议

```c
/* 创建一个主协议簇类型为AF_INET(IPV4)，协议类型为SOCK_STREAM(TCP)，次协议簇为IPPROTO_TCP(TCP)的socket套接字，返回值-1代表创建失败 */

	int socket = -1;

    if ((sock = lwip_socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == -1)
    {
        /* 创建socket失败 */
        rt_kprintf("Socket error<br>");
        /* 释放接收缓冲 */
        rt_free(recv_data);
        return;
    }
```

#### 2.lwip_connect()

```c
int lwip_connect(int s, const struct sockaddr *name, socklen_t namelen);
```

* s：套接字描述符
* name：服务器地址信息
* namelen：服务器地址结构的长度

```c
/*  */

	struct sockaddr_in server_addr;

    /* 初始化预连接的服务端地址 */
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(port);
    ip4addr_aton(url, (ip4_addr_t *)&server_addr.sin_addr);//为给定的连接类型和回调函数分配一个新的 netconn 对象，并将其赋值给 conn 变量。如果内存不足，返回 NULL
    rt_memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));

    /* 连接到服务端 */
    if (lwip_connect(sock, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) == -1)
    {
        /* 连接失败 */
        rt_kprintf("<br> Connect fail!<br>");
        closesocket(sock);
        sock = -1;
        /*释放接收缓冲 */
        rt_free(recv_data);
        return;
    }
```



