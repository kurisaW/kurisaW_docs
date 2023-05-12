# Linux 环境下v2ray的使用

---

> v2ray官方文档：[https://v2raya.org/](https://v2raya.org/)

## curl安装

```
apt-get purge libcurl4
apt-get install curl
```

## v2ray镜像脚本安装

```c
curl -Ls https://mirrors.v2raya.org/go.sh | sudo bash
```

![image-20230504104013149](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041040498.png)

出现该提示信息则表示安装成功：`info: V2Ray v5.4.1 is installed.`

接着关掉服务，因为 v2rayA 不依赖于该 systemd 服务，如果是 Xray内核，则需要把后面的v2ray替换xray

```c
sudo systemctl disable v2ray --now 
```

## v2ray软件安装

> 仓库release地址：[https://github.com/v2rayA/v2rayA/releases](https://github.com/v2rayA/v2rayA/releases)

选择合适自己 Linux 内核架构，可以使用`dpkg --print-architecture`查看

![image-20230504105029122](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041050162.png)

这里我选择``下载到 Linux 共享文件夹

![image-20230504105226313](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041052416.png)

将共享文件夹下的`installer_debian_amd64_2.0.5.deb`文件保存到一个文件夹下，在任务管理器中选择使用软件安装打开并进行安装

![image-20230504105440300](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041054352.png)

![image-20230504105340371](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041053522.png)

## 启动v2ray进程

```c
sudo systemctl start v2raya.service
```

## 设置开机自启动

```c
sudo systemctl enable v2raya.service
```

## v2ray使用

打开火狐浏览器，输入 http://localhost:2017/

输入你要设置的用户名和密码，任意填写自己记着就好，最后点击创建

![image-20230504105825544](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041058629.png)

导入我们的机场订阅地址

![image-20230504105925321](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041100916.png)

选择想要使用的节点后，点击 Ready 

![image-20230504110635073](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041106172.png)

## v2ray Settings

我们点击网页上右上角的Setting，进行如下修改

![image-20230504111011073](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041110129.png)

## 测试

至此所有的配置就完成了，我们打开 youtube 测试一下，没有问题，可以进行开发了

![image-20230504111116983](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041111183.png)

## 常用命令

```c
# 开机自启
systemctl enable v2raya

# 取消自启
systemctl disable v2raya

# 启动服务
systemctl start v2raya

# 停止服务
systemctl stop v2raya

# 重启服务
systemctl restart v2raya
```

