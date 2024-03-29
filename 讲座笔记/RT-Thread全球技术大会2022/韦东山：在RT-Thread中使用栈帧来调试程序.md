## 一、ARM中特殊的三个寄存器

在ARM体系中，一般分为四种寄存器：通用目的寄存器、堆栈指针(SP)、连接寄存器(LR) 以及 程序计数器(PC), 其中需要着重理解后面三种寄存器。

#### 堆栈指针R13(SP)
每一种异常模式都有其自己独立的r13，它通常指向异常模式所专用的堆栈，也就是说五种异常模式、非异常模式（用户模式和系统模式），都有各自独立的堆栈，用不同的堆栈指针来索引。
当ARM进入异常模式的时候，程序就可以把一般通用寄存器压入堆栈，返回时再出栈，保证了各种模式下程序的状态的完整性。

#### 连接寄存器R14（LR）

保存子程序返回地址。使用BL或BLX时，跳转指令自动把返回地址放入r14中；
子程序通过把r14复制到PC来实现返回，通常用下列指令：MOV PC, LR；BX LR；
当异常发生时，异常模式的R14用来保存异常返回地址，将R14如栈可以处理嵌套中断。

#### 程序计数器R15（PC）

PC是有读写限制的；
没有超过读取限制的时候，读取的值是指令的地址加上8个字节，由于ARM指令总是以字对齐的，故bit[1:0]总是00；
在CM3内部使用了指令流水线，读PC时返回的值是当前指令的地址+4.
向PC中写数据，就会引起一次程序的分支（但是不更新LR寄存器），CM3中的指令至少是半字对齐的，所以PC的LSB总是读回0。
在分支时，无论是直接写 PC 的值还是使用分支指令，都必须保证加载到 PC 的数值是奇数（即 LSB=1），用以表明这是在Thumb 状态下执行。
倘若写了 0，则视为企图转入 ARM 模式，CM3 将产生一个 fault 异常。

---

## 二、在汇编指令中，B与BL指令

#### B与BL指令的作用

实现程序跳转，也就是调用子程序

#### B与BL指令的区别

B指令：简单的程序跳转，跳转到目标标号处执行

BL指令：带链接程序跳转，也就是返回地址

---

## 三、CmBacktrace原理

![image-20220902095759327](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209020957424.png)

![image-20220902100001222](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209021000317.png)
