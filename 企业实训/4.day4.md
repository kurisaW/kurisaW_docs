

# 项目步骤3：

实现服务端注册功能



## 知识点1：QT的tcp服务端

### 1）获得本机ip

```C++
QString localHostName = QHostInfo::localHostName();//获得本地主机的名字
QHostInfo info = QHostInfo::fromName(localHostName);//根据主机名字获得主机信息
//QList是QT给我们提供的链表
//一般情况下，封装好的数据结构都是模板
/*
模板有什么用？
用类型作为参数，避免重复的逻辑。模板又被称为静态多态。
所以QList<QHostAddress>链表中存放的数据类型是QHostAddress类型。
*/
QList<QHostAddress> list = info.addresses();//获得主机中所有的地址ipv4和ipv6
//遍历链表，但是我这个遍历方式不好！遍历链表应该使用迭代器。但是这个遍历方式好理解。
for(int i = 0;i < list.size();i++)
{
    //list[i]获得链表中的i元素，就是QHostAddress对象，并判断地址的协议类型
    //打印ipv4协议的地址
    if(list[i].protocol()==QAbstractSocket::IPv4Protocol)
        qDebug()<<list[i].toString();
}
```



### 2）QTcpServer

核心功能，让客户端连接，然后产生一个与客户端通信的QTcpSocket对象。

```C++
QTcpSever* server = new QTcpServer(this);//创建QTcpServer对象

/*
建立一个等待，等待客户端连接
参数1：等待连接的主机ip，一台主机可能有好几个ip，QHostAddress::Any监听主机的所有ip，客户端可以连接主机			的任何ip；QHostAddress("192.168.1.12")监听主机一个指定的ip，客户端只能连接主机的
			192.168.1.12
参数2：服务端的端口号。端口号用来找主机中的服务端程序。
*/
server->listen(QHostAddress::Any, 55555);
//当server收到客户端的连接，发出newConnection信号
connect(server, SIGNAL(newConnection()), this, SLOT(newConnect()));


void MainWindow::newConnect()
{
    QTcpSocket* client;
    client = server->nextPendingConnection();//获得与客户端通信用的QTcpSocket对象
    //连接QTcpSocket的信号与槽
    connect(client, SIGNAL(disconnected()), this, SLOT(socket_disconnect()));
    connect(client, SIGNAL(readyRead()), this, SLOT(socket_read()));
}
```





## 知识点2：QT的数据库和数据库的封装

Sqlite数据库，轻量级数据库，占用资源非常少。功能和非轻量级数据库稍差，但是一般数据库功能都能实现，也支持sql。

sqlite数据库适合在嵌入式设备中使用。

QT支持所有的主流数据库。

无论使用哪一款数据库，QT提供的API接口都是统一的。

### 1）数据库API

#### 配置

QT    +=  sql

#### 开关数据库

```C++
#include <QSqlDatabase>

//QT对所有的数据库提供统一的API接口，需要我们加载不同数据库的操作来完成对不同数据库的操作。
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");//加载sqlite数据库的驱动。

db.setHostName("testdb");//数据库主机名
db.setDatabaseName("testdb.db");//数据库名称，在sqlite中数据库的名称就是文件名，在sqlite中每个数据库都是一个.db文件
db.open();
db.close();
```

#### 数据库操作——创建表

对于数据库所有增删改查的操作，都使用QSqlQuery来实现。QSqlDatabase只负责加载驱动、打开关闭。

```C++
QSqlQuery query;
//exec函数  execute 执行sql语句
query.exec("create table if not exists user(\
                  userid integer primary key autoincrement,\
                  password char(20),\
                  nickname char(20),\
                  headid integer\
                  )");
//在注册用户时，我们会将用户的密码、昵称、头像插入数据库，然后自动生成userid作为用户账号。然后再把userid读取返回给客户端。
query.exec("insert into user values(100000,'abc123','admin',0)");//先插入一个原始用户，目的是为了给userid一个起始值，这样注册的第一个用户的userid就是100001
```

#### 数据库操作——插入数据

```C++
QSqlQuery query;
//当sql语句中需要一些参数时，不能直接执行exec。
//prepare是格式化sql语句用的，其中的?表示需要格式化的数据
query.prepare("insert into user(password, nickname, headid) values(?,?,?)");
query.bindValue(0, e.password);//0代表第一个?，使用e.password替换sql语句中的第一个问号
query.bindValue(1, e.nickname);
query.bindValue(2, e.headId);
query.exec();//把值绑定好之后就可以执行sql语句
```

#### 数据库操作——查询数据

```C++
QSqlQuery query;
query.exec("select * from user");
//查询结果在QSqlQuery对象中，被存在一个线性结构（链表，带有空头的链表）中。
while(query.next())//移动到链表的下一个节点，如果移动成功返回true
{
    QSqlRecord record = query.record();//获得节点中的数据，record对象中包含了一行中所有字段的数据
    QVariant userId = record.value("userid");//获得userid字段的数据，QVariant是QT给我们提供的泛型，它可以存放任何类型数据，也可以转换成任何类型数据
    QString userIdStr = userId.toString();//将QVariant对象转换成QString对象
    
    QString name = query.record().value("nickname").toString();
    int headId = query.record().value("headid").toInt();
}
```

#### 数据库查询——条件查询

```C++
QSqlQuery query;
query.prepare("select * from user where userid = ? and password = ?");
query.bindValue(0, e.id);
query.bindValue(1, e.password);
query.exec();
/*bool exist = false;
//如果账号密码查询失败，则不会有结果，所以下面的循环不会执行，所以循环下面exist值是false
while(query.next())
{
    e.nickname = query.record().value("nickname").toString();
    exist = true;
}
cout<<exist;*/
if(query.next())
{
    //登录成功
}
else
{
    //登录失败
}
```

#### 数据库查询——查询id

```C++
QSqlQuery query;
query.exec("SELECT LAST_INSERT_ROWID()");//查询插入的最后一条数据的id，就是刚刚注册的账号
QString id;
if(query.next())
{
	id = query.record().value(0).toString();//value(0)因为我们不是按照字段查找，0代表record对象中的第一个数据
}
```



### 2）数据库封装

对于数据库的封装，一般是针对表已经封装，一个表对应一个操作类。

XXXDao    d: data	a: abstract   o: object  数据抽象对象

user表封装的操作类  UserDao

一个表中会有很多字段，如果让每个字段对应一个参数，那么操作函数的参数将会写的又多又乱套。

所以我们会将所有的字段封装到一个类中，专门用于传参数。

user表   userid  password  nickname  headid

```C++
struct UserEntity
{
	QString userId;
	QString passwd;
	QString nickName;
	int headId;
};
```

综上，对于每个表的封装需要两个类  XXXDao负责对表的操作逻辑，XXEntity负责将数据封装，便于传参数





