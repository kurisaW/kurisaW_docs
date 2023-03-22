对于I2C总线工作，我们需要在VCC线和SDA线之间连接一个电阻，以及VCC线和SCL线之间的另一个电阻。这些称为上拉电阻。

如果我们有多个从属，那么它是如何看起来的，即多于一个传感器连接到Arduino？嗯，在这种情况下，您仍然可以将一个电阻与SDA线路和另一个用于SCL线的电阻。它并不是，如果您只有一个传感器或50个传感器连接到您的Arduino，则在SCL线上只需要一个电阻和SDA线上的一个电阻。

但为什么这么做？为什么我们根本需要任何电阻？我为什么要关心？好吧，正如您现在可能怀疑的那样，通过上拉电阻导致多次分子板引起的多个断路板引起的主要问题之一是由上拉电阻引起的。

要了解这一点，请注意，例如，如何在SCL线上生成数字信号。 SDA线路以同样的方式工作，但为了清楚起见，我只是在这里显示SCL线。

在该电路中，VCC线上的电压为5V（或3.3V，根据电源），GND线路上的电压为0V，SCL线上的电压，产生的实际信号取决于位置开关。

如果开关打开，则VCC线路的5V电位也将在SCL线上。在这种情况下，SCL线上的电压将是5V，并且信号将被解释为逻辑高。由于开关打开，因此不会流过设备。

如果开关关闭，则来自GND线的0V也将在SCL线上，并且信号将被解释为逻辑低。现在我们在电阻器上的电位差异，电流将流过开关。

打开和关闭开关产生良好的数字信号，在0V和5V之间变化。

当然，这就是理想的数字信号看起来的样子，但是让＆＃39; s看真正的信号如何看起来像。如果连接安装在备用板上的单个传感器并使用I2C总线将其连接到您的Arduino，则会有类似的东西：

伟大的，现在拍摄示波器并测量SCL线上的信号。你看到了什么？

如您所见，蓝线，SCL线上的测量信号与理想的数字信号完全不同。最大值是低于5V的位，最小值高于0V，电压需要长时间才能从0V到5V。尽管如此，这就是良好的信号看起来的看起来！

现在假装我们假装我们将不是一个突破板连接到Arduino，而是同时连接多个板。

如前所述，将多个传感器连接在I2C总线上意味着将SCL引脚从所有板连接到彼此。因此，由Arduino生成的SCL信号由所有传感器共享。这同样适用于SDA信号，以及电源（VCC）和地面（GND）。那么，SCL信号现在如何看？

新的SCL信号以红色显示，看起来比以前更像理想的信号。逻辑低电平的电压现在远高于以前，但逻辑高的电压看起来相同，现在电压从低到高电平增加得多。好吧，它和＃39不是那么糟糕，对吗？

非常错误！单一原因是逻辑低电平的新电压。要了解这是多么糟糕，让＆＃39; s回到我们的第一个图表。

在SCL引脚和GND引脚之间，之前的I＆＃39; ve示出了连接在SCL引脚和GND引脚之间的机械开关。但设备内没有机械开关。相反，通过作为开关操作的晶体管进行连接。

通过打开和关闭晶体管，可以将SCL信号更改为逻辑低且逻辑高。当晶体管关闭时，SCL和GND引脚之间的晶体管上的电阻非常高，因此实际上没有电流流过晶体管，并且因此，通过电阻器流过晶体管。 SCL线上的电压将非常接近5V，因此它将被解释为逻辑高。



当晶体管接通时，晶体管上的电阻变得非常小，但是，它不是零。小电流现在流过电阻，最重要的是，在晶体管上。 SCL线上的电压等于晶体管上的电压降。由于该电压降非常接近0V，因此SCL信号将被解释为逻辑低。

现在是大问题：如果我们减少上拉电阻的电阻会发生什么？当然，电阻器上的电流增加。但相同的电流也流过晶体管！

晶体管穿过较大电流导致在装置内消散的热量，并且过热是半导体器件故障的主要原因。知道这一点，I2C总线规范和用户手册在晶体管上设置了最多3MA。该电流被称为宿电流。

这意味着使用I2C总线指定的设备必须使用穿过晶体管的3mA的宿电流。它还意味着，当尺寸调整上拉电阻时，电路设计人员应考虑到这一限制。

我们如何知道我们的电路中的吸收电流是否高于3MA限制？嗯，增加吸收电流意味着晶体管上的电压降也会增加。晶体管上的电压降，也称为低电平输出电压，是信号处于低电平的电压电平。

I2C总线规格和用户手册还为低电平输出电压设置为0.4V，因为它表示3MA的最大宿电流在晶体管上流动。因此，每当我们测量SDA或SCL信号时，逻辑低的电压高于0.4V，我们都知道宿电流过高！

具有3mA的最大沉降电流和0.4V的最大低电平输出电压，我们可以计算上拉电阻的最小值。我们所要做的就是考虑在规范内运作时最糟糕的情况。每个上拉电阻的最小值等于电阻器上的电压降除了3mA的最大宿电流。

对于5V的电源，每个上拉电阻必须具有至少1.53kΩ，而对于3.3V的电源，每个电阻必须至少为967Ω。

当然，它并不意味着只要吸收电流超过3mA，设备将停止立即工作。但是，在超出其规格之外的设备时，您应该始终要小心，因为它可以导致通信故障，降低其终身时间，甚至永久损坏设备。

现在返回我们的问题：您可以在I2C总线上连接多少突发板？没有很多......你可能已经注意到，每个突破板都有自己的一对上拉电阻。这些电阻器的值因电路板而异，但大多数有10kΩ，4.7kΩ或2.2kΩ。

当我们将多个断路板连接在一起时，我们实际上是将这些电阻彼此平行连接，降低总电阻。即使将两个板连接到2.2kΩ的上拉电阻也会将整体电阻降低到1.1kΩ。对于3.3V的电源仍然可以很好，但VCC为5V的最小值为1.53kΩ。

在最佳情况下，使用具有10kΩ的上拉电阻的电路板，可以将10个板连接在一起，导致总上拉电阻为1kΩ，这对于3.3V的VCC可以很好。然而，对于VCC为5V但是，您可以将6个板连接有6个电阻，每个电阻为10kΩ，导致总电阻为1.67kΩ。

您还可以使用万用表测量SCL或SDA线路上的总电阻。您所要做的就是断开电源的电源，并测量SCL引脚（或SDA引脚）和VCC引脚之间的电阻。



---





I2C（内部集成电路）的建立是为传感器和微控制器（如Arduino）之间的数字信息传输提供简单的方法。  
I2C具有的有点是只需要两路信号连接到Arduino，在这两路连接上使用多路设备是相当容易的，你可以在信号已被正确接收后得到确认。缺点是数据速率比SPI慢，而且数据在同一时间只能在一个方向上传送，如果需要双向通信时，数据速率降低更多。信号电路还需要连接上拉电阻，以确保信号传输的稳定性。  

### 目录指引

- - - - - [1\. I2C总线](#1_I2C_5)
        - [2\. Arduino的I2C类库 - Wire](#2_ArduinoI2C__Wire_15)
        - - [begin\( \)](#begin__18)
          - [requestFrom\( \)](#requestFrom__26)
          - [beginTransmission\( \)](#beginTransmission__36)
          - [endTransmission\( \)](#endTransmission__41)
          - [write\( \)](#write__54)
          - [available\( \)](#available__66)
          - [read\( \)](#read__71)
          - [onReceive\( \)](#onReceive__76)
          - [onRequest\( \)](#onRequest__81)
        - [3.Wire库函数示例代码](#3Wire_86)
        - - [主机请求数据，从机发送数据](#_87)
          - [主机发送数据，从机接收打印](#_131)

---

##### 1\. I2C总线

I2C总线的两路连线是SDA和SCL,它们都可以在Arduino标准板上找到，以UNO为例，模拟引脚5复用SCL，它提供了一个时钟信号，模拟引脚4复用SDA，它用于数据传送（在MEGA2560板上，数字引脚20作为SDA，数字引脚21作为SCL，不同Arduino控制板的I2C引脚都有所不同，按原理图为准）  
在I2C总线上的任一设备，都可作为主设备。它的任务是协调总线上的其他I2C设备（从设备）之间的信息传输，在I2C总线上只能有一个主设备，而且在多数情况下，主设备都为Arduino，控制连接Arduino的其他I2C通信模块  
![在这里插入图片描述](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302280019494.png)  
目前使用到的大多数Arduino相关I2C模块上，通常都已经添加了上拉电阻，所以只需要I2C从设备连接到Arduino的I2C接口上即可。

> I2C设备的通讯需要共地，即Arduino的地线必须连接到每一个从设备的地线

从设备在I2C总线中以设备地址来确定，每个从设备在总线上的地址都是唯一的，就像门牌号一样，部分I2C模块会有自己固定的I2C地址，需要查询设备的数据手册了解发生什么指令让模块有所作用， 以及会收到什么回复，有些模块允许通过设置引脚的高低电平或发生初始化指令来修改设备地址，在Arduino上最多可接128个从设备

##### 2\. Arduino的I2C类库 - Wire

对于I2C总线的使用，Arduino IDE自带了一个第三方类库Wire，Wire库中隐藏了所有I2C的低级别的功能操作，只需要使用简单的命令就可以初始化I2C总线并与其他设备进行通信  
在I2C通信中常用的函数介绍如下：

###### begin\( \)

**功能：**初始化I2C连接，并作为主机或从机加入I2C总线  
**语法：**  
`Wire.begin（）`  
`Wire.begin（address）`  
**参数：**  
address，一个7位的I2C地址，当函数不带参数时，默认是以主机模式加入I2C总线；当填写了参数时，设备会以从机模式加入I2C总线，address可以设置成0 \~ 127中的任意地址  
**返回值：**无

###### requestFrom\( \)

**功能：**主机向从机发送数据请求信号，使用`requestFrom( )`后，从机端可以使用`onRequest( )`注册一个事件以响应主机的请求；主机可以通过`available( )`和`read( )`函数读取这些数据  
**语法：**  
`Wire.requestFrom(address,quantity)`  
`Wire.requestFrom(address,quantity,stop)`  
**参数：**  
address，需要获取数据的从设备地址  
quantity，获取的字节数  
stop，boolean型值，当其值为`true`时将发送一个停止信息，释放I2C总线；当为`false`时将发送一个重新开始信息，并保持I2C总线的有效连接  
**返回值：**无

###### beginTransmission\( \)

**功能：**（主机）传输数据到指定的从机地址，随后可以使用`write()`函数发送数据，并搭配`endTransmission()`函数结束数据传输  
**语法：**Wire.beginTransmission\(address\)  
**参数：**address，要发送数据的从机的7位地址  
**返回值：**无

###### endTransmission\( \)

**功能：**（主机）结束数据传输  
**语法：**  
`Wire.endTransmission( )`  
`Wire.endTransmission(stop)`  
**参数：**stop，boolean型值，当其值为`true`时将发送一个停止信息，释放I2C总线，当没有填写stop参数时，等效于使用`true`；当其值为`false`时，将发送一个重新开始信息，并继续保持I2C总线的有效连接  
**返回值：**byte型值，表示本次传输的状态，取值为：

- 0，（发送）成功
- 1，数据过长，超出发送缓冲区
- 2，在地址发送时接收到NACK信号
- 3，在数据发送时接收到NACK信号
- 4，其他错误

###### write\( \)

**功能：**当为主机状态时，主机将要发送的数据加入发送队列；当为从机状态时，从机发生数据至发起请求的主机  
**语法：**  
`Wire.write(value)`  
`Wire.write(string)`  
`Wire.write(data,length)`  
**参数：**  
value，以单字节发送  
string，以一系列字节发送  
data，以字节形式发送数组  
length，传输的字节数  
**返回值：**byte型值，返回输入的字节数

###### available\( \)

**功能：**返回接收到的字节数，在主机中，一般用于主机发送数据请求后；在从机中，一般用于数据接收事件中，作用类似于`Serial.available( )`函数  
**语法：**`Wire.available( )`  
**参数：**无  
**返回值：**可读字节数

###### read\( \)

**功能：**读取1B的数据，在主机中，当使用requestFrom\( \)函数发送数据请求信号后，需要使用reaf\( \)函数来获取数据；在从机中需要使用该函数读取主机发送来的数据  
**语法：**`Wire.read( )`  
**参数：**无  
**返回值：**读到的字节数据

###### onReceive\( \)

**功能：**该函数可在从机端注册一个事件，当从机收到主机发送的数据时即被触发  
**语法：**`Wire.onReceive(handle)`  
**参数：**handle，当从机接收到数据时可被触发的事件，该事件带有一个`int型`参数（从主机读到的字节数）且没有返回值，如`void myHandle(int numBytes)`  
**返回值：**无

###### onRequest\( \)

**功能：**注册一个事件，当从机接收到主机的数据请求时触发  
**语法：**`Wire.onRequest(handle)`  
**参数：**handle，可被触发的事件，该事件不带参数和返回值，如`void myHandle( )`  
**返回值：**无

##### 3.Wire库函数示例代码

###### 主机请求数据，从机发送数据

![在这里插入图片描述](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302280019677.png)  
主机部分

```cpp
#include <Wire.h>

void setup() {
  Wire.begin();        // join i2c bus (address optional for master)
  Serial.begin(9600);  // start serial for output
}

void loop() {
  Wire.requestFrom(8, 6);    // request 6 bytes from slave device #8

  while (Wire.available()) { // slave may send less than requested
    char c = Wire.read(); // receive a byte as character
    Serial.print(c);         // print the character
  }

  delay(500);
}
```

从机部分

```cpp
#include <Wire.h>

void setup() {
  Wire.begin(8);                // join i2c bus with address #8
  Wire.onRequest(requestEvent); // register event
}

void loop() {
  delay(100);
}

// function that executes whenever data is requested by master
// this function is registered as an event, see setup()
void requestEvent() {
  Wire.write("hello "); // respond with message of 6 bytes
  // as expected by master
}
```

###### 主机发送数据，从机接收打印

![在这里插入图片描述](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302280019762.png)  
主机部分

```cpp
#include <Wire.h>

void setup() {
  Wire.begin(); // join i2c bus (address optional for master)
}

byte x = 0;

void loop() {
  Wire.beginTransmission(8); // transmit to device #8
  Wire.write("x is ");        // sends five bytes
  Wire.write(x);              // sends one byte
  Wire.endTransmission();    // stop transmitting

  x++;
  delay(500);
}
```

从机部分

```cpp
#include <Wire.h>

void setup() {
  Wire.begin(8);                // join i2c bus with address #8
  Wire.onReceive(receiveEvent); // register event
  Serial.begin(9600);           // start serial for output
}

void loop() {
  delay(100);
}

// function that executes whenever data is received from master
// this function is registered as an event, see setup()
void receiveEvent(int howMany) {
  while (1 < Wire.available()) { // loop through all but the last
    char c = Wire.read(); // receive byte as a character
    Serial.print(c);         // print the character
  }
  int x = Wire.read();    // receive byte as an integer
  Serial.println(x);         // print the integer
}
```