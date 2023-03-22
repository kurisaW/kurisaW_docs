自从Arm在2016年的十月发布两款Armv8-M架构的新处理器Cortex-M23和Cortex-M33以来，已经过去了3年多，而市面上基于这两款处理器的微控制器产品也刚刚才崭露头角。
很多才刚刚通过开发板熟悉Cortex-M0/M0+/M3/M4处理器的童鞋可能心中又要飘过弹幕：

> 谁TM告诉我，这个M23和M33是什么鬼？
> 从个位数一下蹦到两位数了喂！
> 前面十几位兄弟怎么了？喂！
> 别说跟M3有啥关系，这以后下第n代是不是就该叫2333333了？

该来的总会来，那么如何简单粗暴的理解这两个全新的处理器呢？以下是傻孩子独家特别提供的的无责任囫囵吞枣公式：

> Cortex-M23 =
> Cortex-M0/M0 + 硬件除法器 + 性能提升 +
> 专门的栈溢出硬件检测+

> 指令集不可忽略的小动作 +
> 安全扩展（TrustZone for Armv8-M） +
> MPU开发者模型的友好化改进

> Cortex-M33 =
> Cortex-M3/M4 + 性能提升 +
> 专门的栈溢出硬件检测+
> 指令集不可忽略的小动作 +
> 安全扩展（TrustZone for Armv8-M）+
> MPU开发者模型的友好化改进


再简单点说就是无敌增强版的“M0／M0+，M3／M4”加“安全扩展”。有人说，Armv8-M的主要功能就是为Cortex-M家族引入TrustZone，这么看来也是不无道理的。

**1.  增强版的Cortex-M0/M0+**

------

根据官方的说法，Cortex-M23实现的是Armv8-M架构的Baseline子架构，我们不妨理解为手机里面的“入门级”产品。

Cortex-M23从定位上也非常直接，就是给Cortex-M0/M0+增加个安全扩展。因此，实际上所有为Cortex-M0/M0+编译生成的二进制代码基本上都可以“无修”的在Cortex-M23/M33上执行——除非你原本的代码使用了MPU。此外Cortex-M23居然配备了硬件除法器，这无疑在原本Cortex-M0和Cortex-M0+主打的8位／16位市场上把“基本配置”又提升了一个档次。

指令集上，Cortex-M23师承Armv6-M，除了支持“安全扩展”所必须的一系列指令之外，这款入门级产品还做了一个“不可忽略的小动作”——也就是说，除了Cortex-M33以外，Cortex-M23也可以通过很小的代价支持“暗代码（eXecute Only Memory, XOM）”。

------

什么是暗代码呢？和“暗物质”只能理论上知道它存在却很难探测到类似——**“暗代码”是一类只能由处理器执行（取指令）却根本无法用任何形式读取机器码（OPCODE）内容的程序**——也就是人们常说的XO（eXecute-Only）代码。“暗代码”并不是依靠内核来实现的，但却需要编译器和内核共同努力才能支持。这是因为XOM本质上是芯片厂家在地址空间上划分出的一段特殊区域——只能由处理器取指令、用于代码的运行（Instruction Fetch），而不能进行普通的数据访问（Data Access）。这就要求“暗代码”里不能直接保存任何常数——它们必须编码到指令里面——成为指令的一部分，以指令编码中的立即数形式存在。

Armv6-M的指令集大部分都是16位的Thumb指令，16位的指令可以用于编码的立即数的二进制位长度可想而知——少得可怜。Armv7-M由于引入了32位的Thumb2指令集，从而极大增强了指令携带立即数的能力。为了将这一能力引入Armv8-M的Baseline指令集，MOVT和MOVW这两个可以分别携带32位立即数“高、低16位”的指令就被特别加入到Cortex-M23所使用的指令集中。考虑到Armv8-M 强调信息安全，“暗指令”对固件的保护有多大的分量，可想而知。

------

**结论：Cortex-M23——这个M0+不简单。**



**2.  增强版的Cortex-M3/M4**

------

相对Cortex-M3/M4来说，Cortex-M33在性能上有了提升并不是什么意料之外的事情，不提也罢。值得说明的是，从城里来的Cortex-M7在性能上仍然可以"甩其他Cortex-M土包子几条街"——6级流水线和3级流水线的差别可是"三缸夏利和六缸宝马之间的差距"所不能比拟的！（认真脸）。



**3.   ARMv8-M是个知错就改的好少年**

------

​    我不知道有多少人真正用过Armv7-M，也就是Cortex-M3/M4的MPU——简单说就是个以Region为单位来修改Memory属性的系统级外设。原本设计的时候想法很简单，一个Region，给个大小（Size）给个基地址（Base Address），再给个属性（Memory Attribute），一使能，就工作了，很简单，很Happy。然而，出于优（pi）化（gu）内（jue）核（ding）面（nao）积（dai）的原因，Region地址范围的设定被人为加入了一个限定：

> **基地址（Base Address）必须对齐（Aligned with）到它的尺寸（Size），而且尺寸必须是2的整数次方（还必须大于4次方）。**


举个例子：一个Region大小为512K，那么基地址必须是512K的整数倍……如果你还不能理解这个问题蛋疼的点在哪里，设想一个任意大小的Region该怎么设定，比如，一个234K大小的Memory该咋办？——还能咋办，用多个Region组合出来呗。正是这个蛋疼的限制，导致几乎没有什么RTOS可以很好的使用MPU，也罕有身边的项目把MPU这么骨感的现实应用的如理想般美好。

那么Armv8-M做了什么呢？他更正了这一蛋疼的设定，即：**Region的设置由“基地址+尺寸”进化为“起始地址+终止地址”**，除了这两个地址都必须是32字节的整倍数的要求外，再也没有变态的关于“基地址必须是Region大小的整倍数”这样的限定。是不是突然觉得眼前一亮，是不是突然发现了一个宝藏？MPU顿时好玩起来。

**结论：ARMv8-M的MPU是个好同志，士别三日当刮目相看**



**4.    安全扩展（Trust Zone for ARMv8-M）又如何简单的理解呢？**

------

* **TrustZone for Armv8-M** **和** **TrustZone** **是什么关系**

首先 “TrustZone for ARMv8-M” 是一个 专有名词，它和 Cortex-A 系列上引入的 “TrustZone” 具有以下的共同特点：

* 都是销售用语

- 都高举 TrustZone 大旗
- 仅在纯理论层面共享一些抽象的模型，用于理解和设计嵌入式信息安全
- 安全效果基本相同



它们至少在以下几个方面存在差异：

* 架构定义完全不同

- 技术实现完全不同
- 执行效率完全不同
- 各类成本完全不同
- 使用方法完全不同
- ……

- （其实，我**个人觉得TrustZone for ARMv8-M 比 TrustZone 要先进**。这当然不仅仅因为“我是 Cortex-M 阵营的”，更因为我觉得“用脚趾头想都知道，TrustZone for ARMv8-M 是后来者，当然有充分的理由比 TrustZone 先进啦。”）

- **功能安全****（****Safety****）****和** **信息安全****（****Security****）**

打个比方，你买了一个智能灯泡，那么对于这个产品来说：

* 用于保护灯泡不会因为电压过高、过低或者电流过大而损坏的保护电路，实现的就是**功能安全**，用英文单词 **Safety** 表示；

- 用于保护你家灯泡不被隔壁老王控制，或者保护你家灯泡上的摄像头（如果有的话）以及麦克风（如果有的话）不被隔壁老王窃听的设计，实现的就是**信息安全**，用英文单词 **Security** 表示。

- 再进一步总结来说，你可以简单粗暴的认为：

> **Safety** 保证的是系统在各种不同（通常是极端）的环境下，都拥有正常的工作逻辑；或者说所提供的功能和服务都是正常的；如果环境太极端，就进入某种保护状态，以避免为用户提供错误或者危险的服务。——**Safety 对抗的是来自环境的挑战**。
>
> **Security** 保证的是在人为破坏的情况下，系统能有效地检测到攻击行为、确保有效信息不会被泄露、系统不会被未经授权的用户所控制—— **Security 对抗的是隔壁老王以及各类隐藏在网络上的云老王**。

**NOTE：关于Safety和Security的关系，更多详细信息，请参考文章****[《大白话说嵌入式安全（1）》](http://mp.weixin.qq.com/s?__biz=MzAxMzc2ODMzNg==&mid=2656101826&idx=2&sn=a1b29ca148fc04845a78a5b85c3d13e1&chksm=8039c63db74e4f2b6a40e09042a8c5ab06f0cbab41c9a0fda7d84a4bba6e4437172e22151436&scene=21#wechat_redirect)**

实际上 Security 必然是通过硬件和软件实现的，只有它的功能、逻辑得到充分的保护，才能有效地对抗攻击者。因此，“老王”们通常利用攻击系统的Safety，也就是功能安全，来试图破构建其上的 信息安全—— **Security 是建立在 Safety 之上的，谈论 Security 的时候必然离不开Safety**——这也是大家常常混淆着两个概念的原因。同时，我们不能因为它们有单方向的依存关系（Security 依赖于 Safety，反过来却不一定），就认为我们可以不分场合的将它们混用。

- **为什么人们突然这么重视** **Security**

过去，大多数微控制器的项目，1）本地团队自己就可以完成了，2）往往不用跟第三方合作，3）也不需要大规模的连接到网络上，4）模块化的目的单纯为了快速开发，因而过去的系统在信息安全上的问题并不是非常突出，基本上就停留在克隆抄袭这个层面上。

然而，除了克隆抄袭这个永恒的原因以外，1）IoT 的到来使得更多的嵌入式设备无法孤立的存在，因此通信安全变得突出；2）生态系统和平台的概念深入人心，单一的本地团队越来越无法独立的完成整个项目，因而与第三方合作成为常态，这就必然导致第三方黑盒子模块的引入，运行时的系统信息安全变得突出；3）商业模式的建立鼓励多方合作，模块化只会帮助IP更高效的被使用而并不天然保护知识产权，更进一步说，单纯的模块化技术并不能保证厂商从最终产品的收益中获得持续稳定的收益。

基于以上原因，简单说就是：因为要与老王合作卖煎饼，我提供设备，老王提供服务，我即担心隔壁老王“你懂的”原因窥探你的财产，同时又希望老王不至于偷了我设备的图纸、把我一脚踢开自己赚钱，所以**Security 在 IoT 时代是必须的**。

- **一句话抓住安全技术的精髓**

> - **Security** 技术实现的核心是 隔离（**Isolation** ）。
> - **Isolation** 在时间上的实现就是把处理器时间按照不同的安全级别进行分配——建立所谓的安全的运行（**Secure** ）和非安全（**Non-secure**）的运行、或者是不同安全级别的运行模式。
> - **Isolation** 在空间上的实现就是各类对存储器以及外设访问的权限控制（**Access Attribution Management**）。

值得特别说明的是，对访问权限的控制是一个通用的工具（Tool），你既可以用它来实现各类资源的分配，比如操作系统中的资源管理；也可以用它来实现信息安全。这并不是说，资源管理是信息安全的一部分，也不能说信息安全就是通过资源管理来实现的——这种说法最要命的地方就是“似是而非”，因而迷惑了很多人。如果有人跟你讨（争）论这个，我的建议是：你跟他打赌吵架都没用，自己心里明白就可以了——“对对对，诸葛孔明是两个人”。

- **TrustZone for ARMv8-M** **之 程序员不得不知道的技术**

既然我们就是要简单粗暴，那么就不用扯那么多犊子。ARMv8-M Security Extension 的本质仍然是实现 Isolation。那么要达到怎样的效果呢？

* CPU在时间上被划分成了两个运行状态：**Secure state** 和 **Non-Secure state**

- 空间上，4GB 的地址空间被划分为两个阵营：**Secure Memory** 和 **Non-Secure Memory**
- 保存在Secure Memory 上的代码就是 **Secure Code**，它必须在 Secure State下运行；保存在Non-Secure Memory上的代码就是**Non-Secure Code**，它必须在Non-Secure State下运行——简单说就是“你是你、我是我”。
- Secure Code可以访问**所有**的数据。

> **Non-Secure Memory**: 你瞅啥？
>
> **Secure Code**: 瞅你咋地？
>
> **Non-Secure Memory**: 不……咋地……你……你瞅我我也不知道……

- Non-Secure Code**只能访问**Non-Secure Memory上的数据；

> **Secure Memory**: 你瞅啥？
>
> **Non-secure Code**: 瞅你咋地？
>
> **Secure Memory**: 我老大你认识不？
>
> **Secure Fault**: 是谁在我地盘上横啊？
>
> **Non-secure Code**: 哥，这……这误会阿……哥
>
> **Secure Fault**: 误会？Secure Memory是你来得地er么？你！这！就！是！搞！事！来啊，拖走！
>
> **Non-secure Code**: 哥……哥，消消气，你看啊，这地盘也不是你划分的，咱还不得找管事er的人说理不是？

- Secure Memory 和 Non-Secure Memory是由 **S****ecure Attribution Unit** 和 **I****mplementation Defined Attribution Unit** 共同决定的。
  你可以把他们理解为一对夫妻：男人 **IDAU** 主外（由芯片厂商定义），女主内（**SAU**，是 Secure Code 在运行时刻通过寄存器来配置的）。对Cortex-M23/33的每一个总线访问，SAU和IDAU都会根据目标地址对比自己所掌握的信息进行投票，仲裁的优先顺序依次是：**Non-Secure**, **Non-Secure-Callable**和**Secure**。谁更接近Secure谁说了算。

> **SAU**: 呦~ Fault哥，又抓到人啦？
>
> **Secure Fault**: 可不，Secure Memoy说这小子是Non-Secure的。
>
> **Secure Memory**: 姐，你可要帮我做主，这小子居然瞅了我一眼！
>
> **Non-secure Code**: 我哪知道……
>
> **SAU/IDAU** : 小子，说其他没用，地址多少？
>
> **Non-secure Code**: 我……来自非安全域……
>
> **SAU**: 呦，Non-Secure的人啊，你知道你偷看的地址属于Secure的么？你就瞅人家？你也不照照镜子，Secure Memory是你能瞅的么？
>
> **Non-secure Code**：我哪知道它是Secure Memory啊？
>
> **SAU/IDAU** : 我说是就是！
>
> **Non-secure Code**：哦……哦……（低头不敢看）
>
> **SAU/IDAU**：Fault 哥，去查查老祖宗立下的规矩，该汇报就汇报，该RESET就RESET，按程序办事。
>
> **Secure Fault**: 得令，走你！

- Secure Code会通过一些专用的 API 来为Non-Secure Code提供服务，这些专用的API被称为 **Secure Entry**。Secure Entry 必须放在 **Non-Secure-Callable** 属性的Memory内。Non-Secure-Callable 本身其实是Secure Memory，但是它特殊的地方就在于可以存放Secure Entry。——**你可以把NSC理解为银行的营业大厅，而那些防弹玻璃上开的一个个柜台小洞就是Secure Entry**。



- **TrustZone for ARMv8-M 所追求的是，Non-Secure Code 和 Secure Code都 以为自己独占整个系统**。对Cortex-M23来说，Non-Secure Code以为自己运行在一个Cortex-M0/M0+上；而对Cortex-M33来说，Non-Secure Code来说，它已为自己独占的是Cortex-M3/M4。
  我们知道，Non-Secure Code的独占是“错觉”，因为它并不知道Secure Code的存在，任何出格的访问（站在它角度来说，任何对未知空间的访问）都会被截获，当作Secure Fault。Secure Code的独占是货真价实的，因为它不仅知道Non-Secure的存在，也可以随时访问他们。
  为了构建这种错觉，代价是巨大的。对于一些核心的资源，比如**NVIC，SysTick，MPU**，Cortex-M23/33都货真价实的为Non-Secure Code**提供了额外的一份**；对于另外一些昂贵的核心资源，比如**流水线，通用寄存器页、Debug逻辑、浮点运算单元**，Secure Code就只能屈尊和Non-Secure Code分时复用（**共享**）了。

- **Non-Secure Code 与 Secure Code 通过Secure Entry进行信息交换。**当Non-Secure Code调用Secure Entry时，如果Secure Entry是有效的，并且存放于NSC Memory里，那么CPU就会从Non-Secure state切换到Secure State，并运行NSC里面的代码（NSC是Secure Memory，所以里面的代码是Secure Code），一般来说，Secure Entry会立即跳转到其它纯粹的Secure Memory中执行。
  **Secure Code可以借助特殊的函数指针以回调（Callback）的方式调用Non-Secure Memory中的代码**（并暂时性的切换回Non-Secure state）。更多的细节，还请自己阅读公开的各类文档。
  
- **Secure Code 和 Non-Secure Code 都可以拥有自己的异常处理程序，这里面涉及到的Secure和Non-Secure的切换都是硬件自动处理好的，程序员不用操心**。值得说明的是，复位以后，整个系统都是Secure状态，所有的异常都属于Secure Code，这个时候，只有Secure Code可以大发慈悲的分配一些中断给Non-Secure Code使用。而且，我们有一个专门的寄存器位可以让所有Non-Secure Exception的优先级比任何Secure Exception都“低人一等”。

总结来说，TrustZone for ARMv8-M 创造了两个世界，**Secure domain**和**Non-Secure domain**。SAU/IDAU共同将4G地址空间拆分为多个Secure，Non-Secure和Non-Secure-Callable的Region。无论是Secure domain还是Secure domain都可以把自己看作是一个普通的Cortex-M0/M0+或者Cortex-M3/M4处理器来开发——大家都有自己独立的NVIC，Systick甚至是独立的MPU。

Secure 可以打 Non-Secure，Non-Secure不能还手，因为**所有的资源理论上都首先属于Secure domain**。



------

- **结语**

Cortex-M23/33 所引入的安全扩展为整个ARM嵌入式系统的信息安全提供了基础（Fundation）或者说根基（The Root of the Security）。然而，单单依靠“基础”不足以构建起坚固的防御体系——芯片厂商、OEM厂商，软件IP厂商、工具链、系统软件及应用设计，任何一环都需要引入必要的信息安全技术和措施。我们可以说：

> **没有 TrustZone for Armv8-M，建立在 Cortex-M 系统之上的安全将是空中楼阁；而单单依靠 TrustZone for Armv8-M 来保护信息安全，更是掩耳盗铃。用户不仅要知道自己要保护什么，如何保护，更要知道：构建一个基础坚实的安全设计所要做的 比贴一张“TrustZone Inside”的标签纸 到自己的产品上 要多得多的多。**

---

文章来源

* [**【分享】简单粗暴Cortex-M23/M33**](https://www.amobbs.com/thread-5741564-1-1.html)