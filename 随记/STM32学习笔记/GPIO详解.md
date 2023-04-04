## GPIO输入

1、输入浮空：GPIO_Mode_IN_FLOATING 

电平进入后，不经过上下拉，在触发施密特触发器后，进入输入数据寄存器，最后由CPU读取。 浮空输入状态下，IO的电平状态是不确定的，完全由外部输入决定（输入达到条件就触发），如果在该引脚悬空的情况下，读取该端口的电平是不确定的。

![img](https://pic3.zhimg.com/80/v2-df85f8267b9820d803ea3d6059fe230e_720w.webp)

2、输入上拉：GPIO_Mode_IPU 

开启上拉时，引脚默认电压为高电平，**原本需要低电平触发，有上拉电阻存在，使得端口为高电平，达到抗干扰作用，只接受低电平！**

![img](https://pic4.zhimg.com/80/v2-7ebf2fd79bd8a994233dbcfb9f275027_720w.webp)

3、输入下拉： GPIO_Mode_IPD 

开启下拉时，引脚默认电压为低电平，**原本需要高电平触发，有下拉电阻存在，使得端口为低电平，达到抗干扰作用，只接受高电平！**

![img](https://pic4.zhimg.com/80/v2-7ccd5bed8fb93d8bedf02cc24bbc4d37_720w.webp)

4、模拟输入：GPIO_Mode_AIN 

输入没有上拉下拉电阻，且输入的是电压而非电平（电平只有高低之分，电压则是一个连续值）；该输入方式可用于AD转换接受模拟信号，也就是说电信号不经过TTL施密特触发器，而是直接输出电压

![img](https://pic4.zhimg.com/80/v2-7ccd5bed8fb93d8bedf02cc24bbc4d37_720w.webp)

---

## GPIO输出

5、开漏输出：GPIO_Mode_Out_OD 

先写入输出寄存器中，再经过输出控制电路，最终到达端口**（在此模式下，IO口也可以读取IO口电压）** 

优点： 
1、输出端相当于三极管的集电极，要得到高电平需要上拉电阻才行，适合做电流型的驱动，其吸收电流的能力较强（20ma以内） 
2、开漏是用来连接不同电平的器件，匹配电平用的，因为开漏引脚不连接外部的上拉电阻时，只能输出低电平，如果需要同时具备输出高电平的功能，则需要接上拉电阻，很好的一个优点是通过改变上拉电源的电压，便可以改变传输电平。比如加上上拉电阻就可以提供TTL/CMOS电平输出等 ，但其也带来上升沿的延时！
3、可以将多个开漏输出的Pin，连接到一条线上。通过一只上拉电阻，在不增加任何器件的情况下，形成“与逻辑”关系。这也是I2C，SMBus等总线判断总线占用状态的原理。 （补充：什么是“线与”？： 在一个结点(线)上, 连接一个上拉电阻到电源 VCC 或 VDD 和 n 个 NPN 或 NMOS 晶体管的集电极 C 或漏极 D, 这些晶体管的发射极 E 或源极 S 都接到地线上, 只要有一个晶体管饱和, 这个结点(线)就被拉到地线电平上. 因为这些晶体管的基极注入电流(NPN)或栅极加上高电平(NMOS),晶体管就会饱和, 所以这些基极或栅极对这个结点(线)的关系是或非NOR 逻辑. 如果这个结点后面加一个反相器, 就是或 OR 逻辑.）

![img](https://pic1.zhimg.com/80/v2-9074e3cf04de1d4d945bf916c75b6294_720w.webp)

6、开漏复用输出功能：GPIO_Mode_AF_OD 

复用与不是复用的区别在于---非复用输出是cpu控制，复用功能则是从复用功能输出口上接受输出信号，可以是从外设接受输入信号输出。

![img](https://pic4.zhimg.com/80/v2-e573d05c701d3888df3b534155b75707_720w.webp)

7、推挽输出：GPIO_Mode_Out_PP 

与开漏输出不同，在于其在经过输出控制电路后面的MOS管不同！ 

优点：推挽电路是两个参数相同的三极管或MOSFET,以推挽方式存在于电路中,各负责正负半周的波形放大任务,电路工作时，两只对称的功率开关管每次只有一个导通，所以导通损耗小、效率高。输出既可以向负载灌电流，也可以从负载抽取电流。推拉式输出级既提高电路的负载能力，又提高开关速度。

![img](https://pic4.zhimg.com/80/v2-dd85dd4a2fcd045a65bab3736ae97ddf_720w.webp)

8、推挽式复用输出功能：GPIO_Mode_AF_PP 

复用功能与之前类似，输出信号来源来自复用功能输出口。

---

## IO口总结：

**在STM32中选用IO模式总结：** 

（1） 浮空输入_IN_FLOATING ——浮空输入，可以做KEY识别 
（2）带上拉输入_IPU——IO内部上拉电阻输入 
（3）带下拉输入_IPD—— IO内部下拉电阻输入 
（4） 模拟输入_AIN ——应用ADC模拟输入，或者低功耗下省电 
（5）开漏输出_OUT_OD ——**IO输出0接GND，IO输出1，悬空，需要外接上拉电阻，才能实现输出高电平。当输出为1时，IO口的状态由上拉电阻拉高电平**，但由于是开漏输出模式，这样IO口也就可以由外部电路改变为低电平或不变。可以读IO输入电平变化，实现STM32的IO双向功能
（6）推挽输出_OUT_PP ——**IO输出0-接GND， IO输出1 -接VCC，读输入值是未知的** 
（7）复用功能的推挽输出_AF_PP ——片内外设功能（I2C的SCL,SDA） （8）复用功能的开漏输出_AF_OD——片内外设功能（TX1,MOSI,MISO.SCK.SS）

---

## GPIO寄存器

* GPIO端口模式寄存器：GPIO port mode register (GPIOx_MODER)

![image-20230403110027567](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031100865.png)

![image-20230403110752164](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031107238.png)

* GPIO 端口输出类型寄存器：GPIO port output type register (GPIOx_OTYPER)

![image-20230403110459569](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031104631.png)

![image-20230403110854958](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031108035.png)

* GPIO 端口输出速度寄存器：GPIO port output speed register (GPIOx_OSPEEDR)

![image-20230403110650773](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031106860.png)

![image-20230403110943029](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031109121.png)

* GPIO 端口上拉/下拉寄存器：GPIO port pull-up/pull-down register (GPIOx_PUPDR)

![image-20230403111242605](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031112801.png)

![image-20230403111320805](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031113879.png)

* GPIO 端口输入数据寄存器：GPIO port input data register (GPIOx_IDR)

![image-20230403111423662](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031114723.png)

![image-20230403111437455](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031114519.png)

* GPIO 端口输出数据寄存器：GPIO port output data register (GPIOx_ODR)

![image-20230403111506986](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031115055.png)

![image-20230403111532925](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031115091.png)

* GPIO 端口置位/复位寄存器：GPIO port bit set/reset register (GPIOx_BSRR)

![image-20230403111621926](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031116038.png)

![image-20230403111639340](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031116499.png)

* GPIO 端口配置锁定寄存器：GPIO port configuration lock register (GPIOx_LCKR)

![image-20230403111934114](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031119186.png)

![image-20230403111850060](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031118150.png)

* GPIO 复用功能低位寄存器：GPIO alternate function low register (GPIOx_AFRL)

![image-20230403112646260](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031126337.png)

![image-20230403112735986](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031127059.png)

* GPIO 复用功能高位寄存器：GPIO alternate function high register (GPIOx_AFRH)

![image-20230403112822757](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031128827.png)

![image-20230403112912825](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202304031129899.png)