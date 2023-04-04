## 第一章 线程的定义及线程切换的实现

#### 1.裸机系统 & 多线程系统中变量的存放（全局变量、局部变量...）

* 裸机系统中，这些统统存放在栈中。栈：是单片机RAM里一段连续的内存空间，栈的大小一般在启动文件或者链接脚本中指定，最后由库函数__main进行初始化。
* 多线程系统中，由于每个线程都是独立互不干扰的，因此需要为每个线程都分配独立的栈空间，这个栈空间通常是一个预先定义好的全局数组，也可以是动态分配的一段内存空间，但它们都存在于RAM中。

#### 2.定义线程栈

```c
ALIGN(RT_ALIGN_SIZE)
/* 定义线程栈 */
rt_uint8_t rt_flag1_thread_stack[512];
rt_uint8_t rt_flag2_thread_stack[512];
```

* ALIGN(RT_ALIGN_SIZE)：设置变量实现字节对齐，RTT中的RT_ALIGN_SIZE默认4字节对齐

#### 3.定义线程栈

线程作为一个独立的函数，线程主体需要无限循环且不能返回。

```c
/* 软件延时 */
void delay(uint32_t cpunt)
{
    for(; count !=0; count--);
}

/* 线程1 */
void flag1_thread_entry(void *p_arg)
{
    for(; ; )
    {
        flag1 = 1;
        delay(100);
        flag1 = 0;
        delay(100);
	}
}

/* 线程2 */
void flag2_thread_entry(void *p_arg)
{
    for(; ; )
    {
        flag2 = 1;
        delay(100);
        flag2 = 0;
        delay(100);
	}
}
```

#### 4.定义线程控制块

在裸机系统中，程序的主体是CPU按照顺序执行的。而在多线程系统中，线程的执行是由系统调度的。系统为了顺利调度线程，对每个线程都设定了一个“身份证“，也就是线程控制块。在线程控制块中，存有线程的所有信息：线程的栈指针、线程名称、线程的形参等等。

```c
/* 线程控制块定义 */

struct rt_thread
{
    void *sp;					/* 线程栈指针 */
    void *entry;				/* 线程入口地址 */
    void *parameter;			/* 线程形参 */
    void *stack_addr;			/* 线程栈起始地址 */
    rt_uint32_t stack_size;		/* 线程栈大小，单位为字节 */
};

typedef struct rt_thread *rt_thread_t;
```

```c
// 为两个线程给出线程控制块定义
struct rt_thread rt_flag1_thread;
struct rt_thread rt_flag2_thread;
```

