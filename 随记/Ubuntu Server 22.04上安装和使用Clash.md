## 前期准备

在Ubuntu Server安装、配置Clash之前，需要准备一些文件。

### 下载Clash

我们需要在[Clash官网](https://github.com/Dreamacro/clash/releases/tag/v1.11.12)下载文件`clash-linux-amd64-v1.11.12.gz `（具体需要根据系统CPU的类型来选择）。

### 准备配置文件

不同的人会有不同的Clash配置文件来源，不论来自何处，都需要将该`yaml`文件准备好，并命名为`config.yaml`。

## 安装并配置Clash

准备好上面的两个文件后，就可以安装并配置Ubuntu Server上的Clash啦！

### 安装Clash

我们需要将之前下载的`clash-linux-amd64-v1.11.12.gz`解压，得到一个文件`clash-linux-amd64`，通过`scp`指令将文件发送到目标服务器上，博主倾向于将文件放在目录`～/clash/`下。

### 配置Clash

首先，我们需要通过如下指令执行一遍Clash：

```bash
cd clash
./clash-linux-amd64
```

在执行过程中，会提示正在下载`Country.mmdb`文件，等待下载完成，使用Ctrl+C退出Clash程序，此时可以在目录`~/.config/clash/`下发现三个文件：`cache.db`、`config.yaml`和`Country.mmdb`。我们需要将我们准备好的`config.yaml`替换生成出来的`config.yaml`，这样就替换了Clash的配置文件。

同时，我们需要将如下指令复制到文件`~/.profile`，以保证能够在开机的时候执行（其中具体的端口号需要根据实际情况进行调整）：

```bash
# set proxy
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

至此，Clash配置就完成了！

## 使用Clash

在使用Clash前，博主会习惯将启动Clash的sh指令写在文件`boot_clash.sh`中，内容如下：

```bash
cd ~/clash
nohup ./clash-linux-amd64 &
```

后续每次要使用Clash时，只需要使用`sh`指令执行该文件即可，执行后，Clash会在后台运行，不会影响正常的bash操作