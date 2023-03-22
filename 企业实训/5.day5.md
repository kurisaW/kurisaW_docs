# 项目步骤3：

实现服务端简单的注册功能，只能处理一个客户端

暂时我们使用QTcpServer类实现简单的功能，等到我们做多线程并发服务器时，需要对QTcpServer进行重写。

## MsgBuilder类

客户端和服务端的MsgBuilder类要同步，事实上这个类在开发之前就应该完成，为了方便同学理解，所以才一步一步的完成MsgBuilder类。



### 数据库封装

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





# 项目步骤4：

实现服务端简单的登录功能

客户端向服务端发送 账号  密码

服务端校验 账号 密码，根据校验结果返回不同类型的数据



客户端：

点击登录按钮槽函数中，

1 获取账号和密码

2 构建json串

3 发送给服务端

4 readyRead槽函数中获得服务端的返回






