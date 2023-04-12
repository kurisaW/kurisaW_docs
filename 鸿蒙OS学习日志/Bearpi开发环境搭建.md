## 一、BearPi-HM Micro 开发板介绍

BearPi-HM Micro开发板是一块高度集成并可运行Openharmony系统的开发板，板载高性能的工业级处理器STM32MP157芯片，搭配4.3寸LCD电容式触摸屏，并板载wifi电路及标准的E53接口，标准的E53接口可扩展智能加湿器、智能台灯、智能安防、智能烟感等案例。可折叠式屏幕设计大大提高用户开发体验，便于携带和存放，更好地满足不同用户的需求，拓展无限可能。

## 二、Linux镜像下载

下载官方提供镜像(任选一种方式下载)

- Ubuntu20.04（大小8G）下载地址（百度云）：[https://pan.baidu.com/s/1W0cgtXC5T2bv0lAya7eizA](https://gitee.com/link?target=https%3A%2F%2Fpan.baidu.com%2Fs%2F1W0cgtXC5T2bv0lAya7eizA) 提取码：1234
- Ubuntu18.04（大小4.8G）下载地址（百度云）：[https://pan.baidu.com/s/1YIdqlRWRGq_heAfrgQ7EPQ](https://gitee.com/link?target=https%3A%2F%2Fpan.baidu.com%2Fs%2F1YIdqlRWRGq_heAfrgQ7EPQ) 提取码：1234

## 三、BearPi-HM Micro编译环境配置

在完成上面的镜像下载后，我们需要对BearPi-HM Micro环境进行编译环境的配置

#### 1.首先添加如下镜像源

```vi
vi /etc/apt/source.list
```

```c
# 添加中科大源
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

#### 2.更新镜像源

```
sudo apt-get update
```

#### 3.安装依赖库及工具

```
sudo apt-get install build-essential gcc g++ make zlib* libffi-dev e2fsprogs pkg-config flex bison perl bc openssl libssl-dev libelf-dev libc6-dev-amd64 binutils binutils-dev libdwarf-dev u-boot-tools mtd-utils gcc-arm-linux-gnueabi cpio device-tree-compiler net-tools openssh-server git vim openjdk-11-jre-headless
```

#### 4.安装hb

```
# 安装hb命令
python3 -m pip install --user ohos-build==0.4.3
```

```
# 环境变量配置
sudo vim ~/.bashrc

# 在.bashrc文件最后一行添加如下代码，并保存退出
export PATH=~/.local/bin:$PATH

# 更新环境变量
source ~/.bashrc
```

#### 5.测试hb是否安装成功

```
hb -h
```

![image-20230407181805793](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071818214.png)

## 四、安装mkimage工具

首先解释这个工具的用途：**用来制作不压缩或者压缩的多种可启动映象文件。**

#### 1.新建tools目录

```
mkdir ~/tools
```

#### 2.下载mkimage.stm32工具到`~/tools`目录，并复制到/home/bearpi/tools/目录下

* [mkimage.stm32下载地址](https://pan.baidu.com/share/init?surl=T2O8luJ0-8g5ZZYdOvWfqQ) 提取码：1234

#### 3.修改mkimage.stm32文件权限

```
chmod 777 ~/tools/mkimage.stm32
```

#### 4.设置环境变量

```
vim ~/.bashrc

# 将下面的代码拷贝至.bashrc文件最后，并保存退出
export PATH=~/tools:$PATH

# 更新环境变量
source ~/.bashrc
```

## 五、bearpi镜像导入VMware

准备好前面的Linux镜像，并解压该文件，打开VMware station，选择上方导航栏：文件->打开(O)，选择我们Linux镜像中的`BearPi-HM_Micro_Ubuntu.ovf`文件，等待镜像文件的导入，开始登录

```
账户：bearpi
密码：bearpi
```

首先将网络连接模式更改为NAT模式，选择上方导航栏：虚拟机(M)->设置->网络适配器->NAT模式

此时打开一个终端，输入ifconfig查看ip

![image-20230407185206621](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071852103.png)

## 六、源码获取

```
cd /home/bearpi

mkdir project && cd project

git clone https://gitee.com/bearpi/bearpi-hm_micro_small.git
```

## 七、编译代码

首先进入到项目文件夹中

```
cd /home/bearpi/project/bearpi-hm_micro_small/
```

执行如下命令(普通用户模式终端下)：

```
hb set
```

出现`[OHOS INFO] Input code path: `提示信息后再输入`.`

![image-20230407190200859](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071902001.png)

我们选择`bearpi-hm_micro`后回车

![image-20230407190426957](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071904137.png)

输入下面的命令，等待下载程序完成

```
hb build -t notest --tee -f
```

当出现`build success`时，即代表编译成功

![image-20230407191628183](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071916323.png)

## 八、查看编译出的固件位置

当编译完后，在Windows中可以直接查看到最终编译的固件，具体路径在： `/home/bearpi/project/bearpi-hm_micro_small/out/bearpi_hm_micro/bearpi_hm_micro` 其中有以下文件是后面烧录系统需要使用的。

- OHOS_Image.stm32：系统镜像文件
- rootfs_vfat.img：根文件系统
- userfs_vfat.img：用户文件系统

![image-20230407191938678](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071919790.png)

我们将这三个文件复制到该目录下：`/home/bearpi/project/bearpi-hm_micro_small/applications/BearPi/BearPi-HM_Micro/tools/download_img/kernel/`，方便后续烧录系统使用

```
cp -r OHOS_Image.stm32 rootfs_vfat.img userfs_vfat.img /home/bearpi/project/bearpi-hm_micro_small/applications/BearPi/BearPi-HM_Micro/tools/download_img/kernel/
```

![image-20230407192926584](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304071929824.png)

## 九、固件烧录

#### 1.准备工作

* [CH340驱动](https://www.wch.cn/downloads/CH341SER_EXE.html)
* [STM32CubeProgramme(v2.4.0+)](https://www.st.com/en/development-tools/stm32cubeprog.html#get-software)

#### 2.连接开发板

首先将电脑的虚拟机和RailDriver打开，确保SFTP服务能够正常使用。（关于RailDriver配置可以查看这篇文章：[【Linux系统开发】Ubuntu配置SFTP服务](https://kurisaw.github.io/p/linux%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91ubuntu%E9%85%8D%E7%BD%AEsftp%E6%9C%8D%E5%8A%A1/)）

当计算机本地磁盘出现一个SFTP(Y:)的网络盘符出现即代表服务能正常使用。

我们将开发板的usb接口连接到电脑，此时由于虚拟机会识别到设备，我们选择连接到本机

![image-20230411183456029](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111834424.png)

#### 3.镜像烧录

* 首先将开发板的拨码开关拨至“000”模式，然后再按下Reset键。

* 打开STM32CubeProgramme，选择USB设备和正确的端口后，点击Connect连接小熊派。
* 点击STM32CubeProgrammer工具的“+”按钮，然后选择烧录配置的tvs文件(路径：`Y:\home\bearpi\project\bearpi-hm_micro_small\applications\BearPi\BearPi-HM_Micro\tools\download_img\flashlayout\bearpi-hm_micro.tsv`)。
* 点击Browse按钮，然后选择工程源码下的烧录镜像路径
* 点击下载，等待烧录成功，中间会有一次断开连接，需要再虚拟机界面再次选择将USB设备连接到主机

![image-20230411193521444](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111935608.png)

#### 4.启动系统

将开发板背面的拨码开关切换至“010”启动模式，并按一下RESET重启开发板，之后等待几秒中会看到屏幕中出现桌面及预装软件，之后就可以结合SSH进行远程终端开发了。

![3](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111940183.jpg)
