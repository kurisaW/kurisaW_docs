## 简单了解

YoMo 是一个为边缘计算领域打造的低时延流式数据处理框架，基于 QUIC 协议通讯，以 Functional Reactive Programming 为编程范式，构建可靠、安全的低时延实时计算应用，挖掘 5G 潜力，释放实时计算价值。

## 环境搭建

#### 1.先决条件

请确保你已经完成GO编译运行环境的安装，参考[安装 Go](https://golang.org/doc/install)。

#### 2.安装CLI

* 二进制安装（推荐）

```go
$ curl -fsSL "https://get.yomo.run" | sh
```
![image-20230326143034647](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202303261430084.png)

* 或者从源代码编译

```go
$ go install github.com/yomorun/cli/yomo@latest
```

验证CLI是否安装成功

```go
$ yomo -V
YoMo CLI version: v1.10.3
```

出现版本号即为安装成功，这里记录一次踩坑，报了错误：

```
root@ubuntu:/home/yifang/Desktop/YOMO# yomo -V
yomo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.33' not found (required by yomo)
yomo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.32' not found (required by yomo)
yomo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by yomo)
```

查找原因就是GLIBC版本原因，我们先查看下版本：

```
strings /lib/x86_64-linux-gnu/libc.so.6 |grep GLIBC_
```

![image-20230326143450129](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202303261434188.png)

确实最高版本为2.30，我们添加下镜像源，并更新升级为libc6

```
# 编辑源

sudo vi /etc/apt/sources.list
```

```
# 添加高版本的源

deb http://th.archive.ubuntu.com/ubuntu jammy main    #添加该行到文件
```

```
# 运行升级

sudo apt update
sudo apt install libc6
```

此时再查看版本所需的版本号就被添加进来了，我们再次输入`yomo -V`测试版本，通过！

![image-20230326143816401](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202303261438443.png)

#### 3.创建一个Serverless应用

```go
$ yomo init yomo-app-demo

$ cd yomo-app-demo
```

![image-20230326143931699](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202303261439761.png)

此时我们可以看到YOMO CLI会自动创建带有以下内容的`app.go`文件：

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"time"

	"github.com/yomorun/yomo/core/frame"
	"github.com/yomorun/yomo/rx"
)

// NoiseData represents the structure of data
type NoiseData struct {
	Noise float32 `json:"noise"` // Noise value
	Time  int64   `json:"time"`  // Timestamp (ms)
	From  string  `json:"from"`  // Source IP
}

var echo = func(_ context.Context, i interface{}) (interface{}, error) {
	value := i.(*NoiseData)
	value.Noise = value.Noise / 10
	rightNow := time.Now().UnixNano() / int64(time.Millisecond)
	fmt.Printf("[%s] %d > value: %f ⚡️=%dms\n", value.From, value.Time, value.Noise, rightNow-value.Time)
	return value.Noise, nil
}

// Handler will handle data in Rx way
func Handler(rxstream rx.Stream) rx.Stream {
	stream := rxstream.
		Unmarshal(json.Unmarshal, func() interface{} { return &NoiseData{} }).
		Debounce(50).
		Map(echo).
		StdOut().
		Marshal(json.Marshal).
		PipeBackToZipper(0x34)

	return stream
}

func DataTags() []frame.Tag {
	return []frame.Tag{0x33}
}
```

#### 4.编译运行

从终端运行yomo dev:

```go
$ yomo dev
```



















## Demo跑起来

项目需准备Linux设备，配置之前我们需要搭建Go、Rust、TensorFlow的环境。

#### a. 基础准备

测试视频路径：https://github.com/yomorun/yomo-wasmedge-tensorflow/releases/download/v0.1.0/hot-dog.mp4

模型下载地址：https://storage.googleapis.com/tfhub-lite-models/google/lite-model/aiy/vision/classifier/food_V1/1.tflite

##### (1)安装YoMo

```
$ go install github.com/yomorun/cli/yomo@latest
```

YoMo的更多信息请移步官方GitHub。

##### (2)安装相关Tools

```
$ sudo apt-get update
$ sudo apt-get install -y ffmpeg libjpeg-dev libpng-dev
```

##### (3)WasmEdge安装

此处为Linux命令，项目仅用于Linux设备。

```
$ wget https://github.com/WasmEdge/WasmEdge/releases/download/0.8.0/WasmEdge-0.8.0-manylinux2014_x86_64.tar.gz
$ tar -xzf WasmEdge-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo cp WasmEdge-0.8.0-Linux/include/wasmedge.h /usr/local/include
$ sudo cp WasmEdge-0.8.0-Linux/lib64/libwasmedge_c.so /usr/local/lib
$ sudo ldconfig
```

#### b. WasmEdge-tensorflow

##### (1)安装 tensorflow 依赖项

```
$ wget https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/0.8.0/WasmEdge-tensorflow-deps-TF-0.8.0-manylinux2014_x86_64.tar.gz
$ wget https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/0.8.0/WasmEdge-tensorflow-deps-TFLite-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo tar -C /usr/local/lib -xzf WasmEdge-tensorflow-deps-TF-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo tar -C /usr/local/lib -xzf WasmEdge-tensorflow-deps-TFLite-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo ln -sf libtensorflow.so.2.4.0 /usr/local/lib/libtensorflow.so.2
$ sudo ln -sf libtensorflow.so.2 /usr/local/lib/libtensorflow.so
$ sudo ln -sf libtensorflow_framework.so.2.4.0 /usr/local/lib/libtensorflow_framework.so.2
$ sudo ln -sf libtensorflow_framework.so.2 /usr/local/lib/libtensorflow_framework.so
$ sudo ldconfig
```

##### (2)安装 WasmEdge-tensorflow

```
$ wget https://github.com/second-state/WasmEdge-tensorflow/releases/download/0.8.0/WasmEdge-tensorflow-0.8.0-manylinux2014_x86_64.tar.gz
$ wget https://github.com/second-state/WasmEdge-tensorflow/releases/download/0.8.0/WasmEdge-tensorflowlite-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo tar -C /usr/local/ -xzf WasmEdge-tensorflow-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo tar -C /usr/local/-xzf WasmEdge-tensorflowlite-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo ldconfig
```

##### (3)安装 WasmEdge-image：

```
$ wget https://github.com/second-state/WasmEdge-image/releases/download/0.8.0/WasmEdge-image-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo tar -C /usr/local/ -xzf WasmEdge-image-0.8.0-manylinux2014_x86_64.tar.gz
$ sudo ldconfig
```

如果安装有任何问题，请参考官方文档。

#### c. 运行Demo

##### (1) Streaming Serverless

```
$ cd yomo-wasmedge-tensorflow/flow
$ go get -u github.com/second-state/WasmEdge-go/wasmedge
```

模型下载至`rust_mobilenet_foods/src`:

```
$ wget ' https://storage.googleapis.com/tfhub-lite-models/google/lite-model/aiy/vision/classifier/food_V1/1.tflite ' -O ./rust_mobilenet_food/src/lite-model_aiy_vision_classifier_food_V1_1.tflite
```

安装rustc and cargo ,并设置Rust版本为1.50.0：

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ export PATH=$PATH:$HOME/.cargo/bin
$ rustup default 1.50.0
```

安装rustwasmc：

```
$ curl https://raw.githubusercontent.com/second-state/rustwasmc/master/installer/init.sh -sSf | sh
$ cd rust_mobilenet_food
$ rustwasmc build
```

WASM文件在`pkg/rust_mobilenet_food_lib_bg.wasm`，复制该文件到flow：

```
$ cp pkg/rust_mobilenet_food_lib_bg.wasm yomo-wasmedge-tensorflow/flow
```

##### (2)启动服务

启动YoMo Orchestrator Server：

```
$ yomo serve -c ./zipper/workflow.yaml
```

运行Streaming Serverless：

```
$ cd flow
$ go run --tags "tensorflow image" app.go
```

##### (3) 演示

```
$ wget -P source  ' https://github.com/yomorun/yomo-wasmedge-tensorflow/releases/download/v0.1.0/hot-dog.mp4 ' 
$ go run ./source/main.go ./source/热狗.mp4
```

演示结果如下图所示，image传入和算法执行的时间在左侧。

![img](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202303261538464.webp)
