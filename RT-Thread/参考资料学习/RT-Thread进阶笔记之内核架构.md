### 文章目录

- - [1、RT-Thread 软件架构](#1RTThread__1)
  - [2、RT-Thread 内核结构](#2RTThread__3)
  - [3、 预备知识](#3__5)
  - - [3.1链表](#31_6)
    - [3.2 节点](#32__14)
    - - [3.2.2 节点初始化](#322__15)
      - [3.2.3 在双向链表表头后面插入一个节点](#323__24)
      - [3.2.4 在双向链表表头前面插入一个节点](#324__37)
      - [3.2.5 从双向链表删除一个节点](#325__50)
      - [3.2.6 从节点中获取所在结构体的首地址](#326__62)
    - [3.3 面向对象编程思想](#33__75)
    - - [3.3.1 封装](#331__76)
      - [3.3.2 继承](#332__78)
      - [3.3.3 多态](#333__80)
  - [4 内核对象管理架构](#4__82)
  - - [4.1 内核对象数据结构](#41__87)
    - - [4.1.1 对象结构](#411__88)
      - [4.1.2 对象容器结构](#412__90)
      - [4.1.3 初始化对象容器](#413__92)
      - [4.1.4 从容器中获取指定类型的对象信息](#414__147)
      - [4.1.5 初始化对象](#415__162)
      - [4.1.6 线程结构](#416__239)
      - [4.1.7 初始化线程](#417__245)
      - [4.1.8 初始化线程栈](#418__275)

## 1、RT-Thread 软件架构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131105514907.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

## 2、RT-Thread 内核结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131105606716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

## 3、 预备知识

### 3.1链表

链表是通过节点把离散的数据链接成一个表，通过对节点的插入和删除操作从而实现对数据的存取。而数组是通过开辟一段连续的内存来存储数据，这是数组和链表最大的区别。数组的每个成员对应链表的节点，成员和节点的数据类型可以是标准的C类型或者是用户自定义的结构体。数组有起始地址和结束地址，而链表是一个圈，没有头和尾之分，但是为了方便节点的插入和删除操作会人为的规定一个根节点。  
单向链表：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020111502949.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020111322797.png)  
双向链表：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201114941841.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020111513342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

### 3.2 节点

#### 3.2.2 节点初始化

```c
rt_inline void rt_list_init(rt_list_t *l)
{
    l->next = l->prev = l;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020111451293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 3.2.3 在双向链表表头后面插入一个节点

```c
rt_inline void rt_list_insert_after(rt_list_t *l, rt_list_t *n)
{
    l->next->prev = n;
    n->next = l->next;

    l->next = n;
    n->prev = l;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201120057851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 3.2.4 在双向链表表头前面插入一个节点

```c
rt_inline void rt_list_insert_before(rt_list_t *l, rt_list_t *n)
{
    l->prev->next = n;
    n->prev = l->prev;

    l->prev = n;
    n->next = l;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201120145539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 3.2.5 从双向链表删除一个节点

```c
rt_inline void rt_list_remove(rt_list_t *n)
{
    n->next->prev = n->prev;
    n->prev->next = n->next;

    n->next = n->prev = n;
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201120247276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 3.2.6 从节点中获取所在结构体的首地址

```c
#define rt_list_entry(node, type, member) \
    rt_container_of(node, type, member)
#define rt_list_for_each(pos, head) \
    for (pos = (head)->next; pos != (head); pos = pos->next)
```

node 表示一个节点的地址， type 表示该节点所在的结构体的类型，  
member 表示该节点在该结构体中的成员名称。  
rt\_container\_of\(\)的实现算法具体见图：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200211145258366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)  
我们知道了一个节点 tlist 的地址 ptr，现在要推算出该节点所在的 type 类型的结构体的起始地址 f\_struct\_ptr。我们可以将 ptr 的值减去图中灰色部分的偏移的大小就可以得到 f\_struct\_ptr 的地址，现在的关键是如何计算出灰色部分的偏移大小。这里采取的做法是将 0 地址强制类型类型转换为 type，即\(type \*\)0，然后通过指针访问结构体成员的方式获取到偏移的大小，即\(\&\(\(type \*\)0\)->member\)， 最后即可算出 f\_struct\_ptr = ptr \-\(\&\(\(type \*\)0\)->member\)。

### 3.3 面向对象编程思想

#### 3.3.1 封装

[C/C++面向对象编程之封装](https://blog.csdn.net/sinat_31039061/article/details/104200127)

#### 3.3.2 继承

[C/C++面向对象编程之继承](https://blog.csdn.net/sinat_31039061/article/details/104200135)

#### 3.3.3 多态

[C/C++面向对象编程之多态](https://blog.csdn.net/sinat_31039061/article/details/104234427)

## 4 内核对象管理架构

RT-Thread 采用内核对象管理系统来访问 / 管理所有内核对象，内核对象包含了内核中线程，信号量，互斥量，事件，邮箱，消息队列和定时器，内存池，设备驱动等。对象容器中包含了每类内核对象的信息，包括对象类型，大小等。对象容器给每类内核对象分配了一个链表，所有的内核对象都被链接到该链表上，如图 RT-Thread 的内核对象容器及链表如下图所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131163829806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)  
下图则显示了 RT-Thread 中各类内核对象的派生和继承关系：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131163917851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

### 4.1 内核对象数据结构

#### 4.1.1 对象结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131165837779.png)

#### 4.1.2 对象容器结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131165916405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 4.1.3 初始化对象容器

容 器 是 一 个 全 部 变 量 的 数 组 ， 数 据 类 型 为 struct rt\_object\_information， 这是一个结构体类型， 包含对象的三个信息，分别为对象类型、对象列表节点头和对象的大小。

```c
static struct rt_object_information rt_object_container[RT_Object_Info_Unknown] =
{
    /* initialize object container - thread */
    {RT_Object_Class_Thread, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Thread), sizeof(struct rt_thread)},
#ifdef RT_USING_SEMAPHORE
    /* initialize object container - semaphore */
    {RT_Object_Class_Semaphore, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Semaphore), sizeof(struct rt_semaphore)},
#endif
#ifdef RT_USING_MUTEX
    /* initialize object container - mutex */
    {RT_Object_Class_Mutex, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Mutex), sizeof(struct rt_mutex)},
#endif
#ifdef RT_USING_EVENT
    /* initialize object container - event */
    {RT_Object_Class_Event, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Event), sizeof(struct rt_event)},
#endif
#ifdef RT_USING_MAILBOX
    /* initialize object container - mailbox */
    {RT_Object_Class_MailBox, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_MailBox), sizeof(struct rt_mailbox)},
#endif
#ifdef RT_USING_MESSAGEQUEUE
    /* initialize object container - message queue */
    {RT_Object_Class_MessageQueue, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_MessageQueue), sizeof(struct rt_messagequeue)},
#endif
#ifdef RT_USING_MEMHEAP
    /* initialize object container - memory heap */
    {RT_Object_Class_MemHeap, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_MemHeap), sizeof(struct rt_memheap)},
#endif
#ifdef RT_USING_MEMPOOL
    /* initialize object container - memory pool */
    {RT_Object_Class_MemPool, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_MemPool), sizeof(struct rt_mempool)},
#endif
#ifdef RT_USING_DEVICE
    /* initialize object container - device */
    {RT_Object_Class_Device, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Device), sizeof(struct rt_device)},
#endif
    /* initialize object container - timer */
    {RT_Object_Class_Timer, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Timer), sizeof(struct rt_timer)},
#ifdef RT_USING_MODULE
    /* initialize object container - module */
    {RT_Object_Class_Module, _OBJ_CONTAINER_LIST_INIT(RT_Object_Info_Module), sizeof(struct rt_dlmodule)},
#endif
};
```

\_OBJ\_CONTAINER\_LIST\_INIT\(\)是一个带参宏，用于初始化一个  
节点 list，在 object.c 中定义

```c
#define _OBJ_CONTAINER_LIST_INIT(c)     \
    {&(rt_object_container[c].object_list), &(rt_object_container[c].object_list)}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020013118165823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 4.1.4 从容器中获取指定类型的对象信息

```c
struct rt_object_information *
rt_object_get_information(enum rt_object_class_type type)
{
    int index;

    for (index = 0; index < RT_Object_Info_Unknown; index ++)
        if (rt_object_container[index].type == type) return &rt_object_container[index];

    return RT_NULL;
}
```

#### 4.1.5 初始化对象

每创建一个对象，都需要先将其初始化，主要分成两个部分的工作，首先将对象控制块里面与对象相关的成员初始化，然后将该对象插入到对象容器中。

```c
/**
 * This function will initialize an object and add it to object system
 * management.
 *
 * @param object the specified object to be initialized.
 * @param type the object type.
 * @param name the object name. In system, the object's name must be unique.
 */
void rt_object_init(struct rt_object         *object,
                    enum rt_object_class_type type,
                    const char               *name)
{
    register rt_base_t temp;
    struct rt_list_node *node = RT_NULL;
    struct rt_object_information *information;
#ifdef RT_USING_MODULE
    struct rt_dlmodule *module = dlmodule_self();
#endif

    /* get object information */
    information = rt_object_get_information(type);
    RT_ASSERT(information != RT_NULL);

    /* check object type to avoid re-initialization */

    /* enter critical */
    rt_enter_critical();
    /* try to find object */
    for (node  = information->object_list.next;
            node != &(information->object_list);
            node  = node->next)
    {
        struct rt_object *obj;

        obj = rt_list_entry(node, struct rt_object, list);
        if (obj) /* skip warning when disable debug */
        {
            RT_ASSERT(obj != object);
        }
    }
    /* leave critical */
    rt_exit_critical();

    /* initialize object's parameters */
    /* set object type to static */
    object->type = type | RT_Object_Class_Static;
    /* copy name */
    rt_strncpy(object->name, name, RT_NAME_MAX);

    RT_OBJECT_HOOK_CALL(rt_object_attach_hook, (object));

    /* lock interrupt */
    temp = rt_hw_interrupt_disable();

#ifdef RT_USING_MODULE
    if (module)
    {
        rt_list_insert_after(&(module->object_list), &(object->list));
        object->module_id = (void *)module;
    }
    else
#endif
    {
        /* insert object into information object list */
        rt_list_insert_after(&(information->object_list), &(object->list));
    }

    /* unlock interrupt */
    rt_hw_interrupt_enable(temp);
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020013118442746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

#### 4.1.6 线程结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201101333178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)  
线程状态切换：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201101716543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)  
RT-Thread 中，实际上线程并不存在运行状态，就绪状态和运行状态是等同的。

#### 4.1.7 初始化线程

在线程初始化之后，线程通过自身的 list 节点将自身挂到容器的对象列表中，对象初始化函数在线程初始化函数里面被调用。

```c
rt_err_t rt_thread_init(struct rt_thread *thread,
                        const char       *name,
                        void (*entry)(void *parameter),
                        void             *parameter,
                        void             *stack_start,
                        rt_uint32_t       stack_size,
                        rt_uint8_t        priority,
                        rt_uint32_t       tick)
{
    /* thread check */
    RT_ASSERT(thread != RT_NULL);
    RT_ASSERT(stack_start != RT_NULL);

    /* initialize thread object */
    rt_object_init((rt_object_t)thread, RT_Object_Class_Thread, name);

    return _rt_thread_init(thread,
                           name,
                           entry,
                           parameter,
                           stack_start,
                           stack_size,
                           priority,
                           tick);
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200131200834296.png)

#### 4.1.8 初始化线程栈

在动态创建线程和初始化线程的时候，会使用到内部的线程初始化函数\_rt\_thread\_init\(\)，\_rt\_thread\_init\(\) 函数会调用栈初始化函数 rt\_hw\_stack\_init\(\)，在栈初始化函数里会手动构造一个上下文内容，这个上下文内容将被作为每个线程第一次执行的初始值。上下文在栈里的排布如下图所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201163604172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzMxMDM5MDYx,size_16,color_FFFFFF,t_70)

```c
rt_uint8_t *rt_hw_stack_init(void       *tentry,
                             void       *parameter,
                             rt_uint8_t *stack_addr,
                             void       *texit)
{
    struct stack_frame *stack_frame;
    rt_uint8_t         *stk;
    unsigned long       i;

    /* 对传入的栈指针做对齐处理 */
    stk  = stack_addr + sizeof(rt_uint32_t);
    stk  = (rt_uint8_t *)RT_ALIGN_DOWN((rt_uint32_t)stk, 8);
    stk -= sizeof(struct stack_frame);

    /* 得到上下文的栈帧的指针 */
    stack_frame = (struct stack_frame *)stk;

    /* 把所有寄存器的默认值设置为 0xdeadbeef */
    for (i = 0; i < sizeof(struct stack_frame) / sizeof(rt_uint32_t); i ++)
    {
        ((rt_uint32_t *)stack_frame)[i] = 0xdeadbeef;
    }

    /* 根据 ARM  APCS 调用标准，将第一个参数保存在 r0 寄存器 */
    stack_frame->exception_stack_frame.r0  = (unsigned long)parameter;
    /* 将剩下的参数寄存器都设置为 0 */
    stack_frame->exception_stack_frame.r1  = 0;                 /* r1 寄存器 */
    stack_frame->exception_stack_frame.r2  = 0;                 /* r2 寄存器 */
    stack_frame->exception_stack_frame.r3  = 0;                 /* r3 寄存器 */
    /* 将 IP(Intra-Procedure-call scratch register.) 设置为 0 */
    stack_frame->exception_stack_frame.r12 = 0;                 /* r12 寄存器 */
    /* 将线程退出函数的地址保存在 lr 寄存器 */
    stack_frame->exception_stack_frame.lr  = (unsigned long)texit;
    /* 将线程入口函数的地址保存在 pc 寄存器 */
    stack_frame->exception_stack_frame.pc  = (unsigned long)tentry;
    /* 设置 psr 的值为 0x01000000L，表示默认切换过去是 Thumb 模式 */
    stack_frame->exception_stack_frame.psr = 0x01000000L;

    /* 返回当前线程的栈地址       */
    return stk;
}
```

扩展阅读：  
[RT-Thread代码启动过程与线程切换的实现](https://blog.csdn.net/sinat_31039061/article/details/104115803)  
[RT-Thread学习笔记之设备框架](https://blog.csdn.net/sinat_31039061/article/details/104135728)  
[RT-Thread学习笔记之FinSH组件](https://blog.csdn.net/sinat_31039061/article/details/104121466)  
[RT-Thread学习笔记之网络框架](https://blog.csdn.net/sinat_31039061/article/details/104200110)

关注公众号，后续有精彩内容会第一时间发送给您！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506150500150.jpg)