#### 1.RT-Thread版本信息

```c
/* RT-Thread version information */
#define RT_VERSION                      4               /**< major version number */
#define RT_SUBVERSION                   1               /**< minor version number */
#define RT_REVISION                     1               /**< revise version number */

/* RT-Thread version */
#define RTTHREAD_VERSION                RT_VERSION_CHECK(RT_VERSION, RT_SUBVERSION, RT_REVISION)
```

使用方法：可用于bsp指定RT-Thread版本

例如：

```
#if (RTTHREAD_VERSION >= RT_VERSION_CHECK(4, 1, 0) */
#define RT_VERSION_CHECK(major, minor, revise)          ((major * 10000) + \
                                                         (minor * 100) + revise)
```

#### 2.RT-Thrad基础数据类型定义

```c
/* RT-Thread basic data type definitions */
#ifndef RT_USING_ARCH_DATA_TYPE	/* 简单来说，开启此宏定义后，BSP就会在ARCH_CPU 级别定义基本数据类型 */
#ifdef RT_USING_LIBC		   /* 用于控制是否使用标准C库函数 */
typedef int8_t                          rt_int8_t;      /**<  8bit integer type */
typedef int16_t                         rt_int16_t;     /**< 16bit integer type */
typedef int32_t                         rt_int32_t;     /**< 32bit integer type */
typedef uint8_t                         rt_uint8_t;     /**<  8bit unsigned integer type */
typedef uint16_t                        rt_uint16_t;    /**< 16bit unsigned integer type */
typedef uint32_t                        rt_uint32_t;    /**< 32bit unsigned integer type */
typedef int64_t                         rt_int64_t;     /**< 64bit integer type */
typedef uint64_t                        rt_uint64_t;    /**< 64bit unsigned integer type */
typedef size_t                          rt_size_t;      /**< Type for size number */

#else
typedef signed   char                   rt_int8_t;      /**<  8bit integer type */
typedef signed   short                  rt_int16_t;     /**< 16bit integer type */
typedef signed   int                    rt_int32_t;     /**< 32bit integer type */
typedef unsigned char                   rt_uint8_t;     /**<  8bit unsigned integer type */
typedef unsigned short                  rt_uint16_t;    /**< 16bit unsigned integer type */
typedef unsigned int                    rt_uint32_t;    /**< 32bit unsigned integer type */

#ifdef ARCH_CPU_64BIT	/* 判断当前程序运行的CPU架构是否为64位 */
typedef signed long                     rt_int64_t;     /**< 64bit integer type */
typedef unsigned long                   rt_uint64_t;    /**< 64bit unsigned integer type */
typedef unsigned long                   rt_size_t;      /**< Type for size number */
#else
typedef signed long long                rt_int64_t;     /**< 64bit integer type */
typedef unsigned long long              rt_uint64_t;    /**< 64bit unsigned integer type */
typedef unsigned int                    rt_size_t;      /**< Type for size number */
#endif /* ARCH_CPU_64BIT */
#endif /* RT_USING_LIBC */
#endif /* RT_USING_ARCH_DATA_TYPE */

typedef int                             rt_bool_t;      /**< boolean type */
typedef long                            rt_base_t;      /**< Nbit CPU related date type */
typedef unsigned long                   rt_ubase_t;     /**< Nbit unsigned CPU related data type */

typedef rt_base_t                       rt_err_t;       /**< Type for error number */
typedef rt_uint32_t                     rt_time_t;      /**< Type for time stamp */
typedef rt_uint32_t                     rt_tick_t;      /**< Type for tick count */
typedef rt_base_t                       rt_flag_t;      /**< Type for flags */
typedef rt_ubase_t                      rt_dev_t;       /**< Type for device */
typedef rt_base_t                       rt_off_t;       /**< Type for offset */

/* boolean type definitions */
#define RT_TRUE                         1               /**< boolean true  */
#define RT_FALSE                        0               /**< boolean fails */

/* null pointer definition */
#define RT_NULL                         0
```

* `rt_base_t`：为了使代码可以**在不同的CPU上移植并保持向后兼容性**。`long`类型的位数（bit数）可能因不同的CPU体系结构而有所不同，但是使用`rt_base_t`代替`long`可以隐藏这种差异，以实现代码的可移植性。（rt_ubase_t原理相同）

* `rt_err_t`：代表**错误码**的数据类型，这里使用了之前定义的`rt_base_t`作为它的别名。

* `rt_time_t`：代表**时间戳**的数据类型，这里使用了`rt_uint32_t`作为它的别名。`rt_uint32_t`是一个32位无符号整数类型，可以用来表示1970年1月1日以来的秒数。

* `rt_tick_t`：代表**系统时钟节拍计数**的数据类型，这里也使用了`rt_uint32_t`作为它的别名。在嵌入式系统中，通常会使用硬件定时器来产生一个固定频率的中断信号，并且在每次中断时对`rt_tick_t`进行递增操作，从而实现对时间的计数。

* `rt_flag_t`：代表**标志位**的数据类型，这里使用了之前定义的`rt_base_t`作为它的别名。

* `rt_dev_t`：代表**设备号**的数据类型，这里使用了`rt_ubase_t`作为它的别名。在嵌入式系统中，通常会有多个外设需要使用不同的设备号进行标识，因此需要定义一个数据类型来保存设备号。

* `rt_off_t`：代表**偏移量**的数据类型，这里也使用了之前定义的`rt_base_t`作为它的别名。在文件系统中，通常需要记录某个文件中的偏移量（即当前读写位置），因此需要定义一个数据类型来保存偏移量。

#### 3.RT-Thread基本数据类型的范围

```c
/* maximum value of base type */
#ifdef RT_USING_LIBC
#define RT_UINT8_MAX                    UINT8_MAX       /**< Maximum number of UINT8 */
#define RT_UINT16_MAX                   UINT16_MAX      /**< Maximum number of UINT16 */
#define RT_UINT32_MAX                   UINT32_MAX      /**< Maximum number of UINT32 */
#else
#define RT_UINT8_MAX                    0xff            /**< Maximum number of UINT8 */
#define RT_UINT16_MAX                   0xffff          /**< Maximum number of UINT16 */
#define RT_UINT32_MAX                   0xffffffff      /**< Maximum number of UINT32 */
#endif /* RT_USING_LIBC */
```

附：此处的`UINT8_MAX`、`UINT16_MAX`、`UINT32_MAX`为编译器预定的宏定义

#### 4.RT-Thread系统滴答时钟最大计数值

```c
#define RT_TICK_MAX                     RT_UINT32_MAX   /**< Maximum number of tick */
```

#### 5.RT-Thread IPC数据类型范围

```c
/* maximum value of ipc type */
#define RT_SEM_VALUE_MAX                RT_UINT16_MAX   /**< Maximum number of semaphore .value */
#define RT_MUTEX_VALUE_MAX              RT_UINT16_MAX   /**< Maximum number of mutex .value */
#define RT_MUTEX_HOLD_MAX               RT_UINT8_MAX    /**< Maximum number of mutex .hold */
#define RT_MB_ENTRY_MAX                 RT_UINT16_MAX   /**< Maximum number of mailbox .entry */
#define RT_MQ_ENTRY_MAX                 RT_UINT16_MAX   /**< Maximum number of message queue .entry */
```

#### 6.RT-Thread避免未使用变量警告

```c
#define RT_UNUSED(x)                   ((void)x)
```

**该宏定义表示将变量x强制转换为`void`类型，从而告诉编译器该变量未被使用，从而避免编译器发出“未使用变量”的警告。这种空操作常常用于函数参数或者结构体成员的声明中，因为有时候我们为了某些原因不得不声明一个变量，但在实际使用中却无需使用它，这时候就可以使用这个宏来标记变量未被使用。 **

下面是一个例子：假设在编写一个C语言程序时，需要使用qsort()函数进行数组排序。

该函数的第一个参数是一个void类型的指针，用于表示要排序的数组。

在实际使用中，我们可能并不需要使用这个参数。但是，由于该函数的参数列表中必须要有第一个参数，而且其类型为void*，因此我们不得不将一个无用的参数传递给函数，否则就会编译错误。

这时候，就可以使用RT_UNUSED宏来标记这个参数未被使用，代码如下：

```c
#include <stdlib.h>

int cmp(const void *a, const void *b)
{
    /* sort code */
}

int main()
{
    int arr[10] = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
    qsort(arr, 10, sizeof(int), cmp);  // 必须传递一个void*类型参数
    return 0;
}

int cmp(const void *a, const void *b)
{
    RT_UNUSED(a);  // 标记参数未使用
    RT_UNUSED(b);  // 标记参数未使用
    return 0;
}
```

这样就可以避免编译器报“未使用变量a/b”的警告了。

#### 7.编译器相关定义

```c
/* Compiler Related Definitions */
#if defined(__ARMCC_VERSION)           /* ARM Compiler */
#define RT_SECTION(x)               __attribute__((section(x)))
#define RT_USED                     __attribute__((used))
#define ALIGN(n)                    __attribute__((aligned(n)))
#define RT_WEAK                     __attribute__((weak))
#define rt_inline                   static __inline
```

- `RT_SECTION(x)`：表示**将所修饰的数据/函数放置在指定的section中**，x为section名字，通常是一个字符串。这个宏可以用于在程序中指定某些数据/函数位于特定的内存区域，比如放在Flash中或者RAM中，以满足不同的需求。该宏使用了GCC的语法扩展。
- `RT_USED`：表示**告诉编译器保留所修饰的数据/函数**，即使它没有被直接引用或调用。该宏通常用于防止删除不需要的代码和变量，以及确保所需的函数和变量在链接时能够正确地生成和调用。该宏使用了GCC的语法扩展。
- `ALIGN(n)`：表示**将所修饰的数据/函数按照n字节对齐**，即从地址0开始，每隔n个字节就对齐一次。该宏通常用于解决访问未对齐的数据导致的性能问题，以及操作系统中数据结构对齐的需求。该宏同样使用了GCC的语法扩展。
- `RT_WEAK`：表示**将所修饰的数据/函数标记为弱引用**，即该数据/函数可以被重定义。当出现多个同名的弱引用时，链接器会选择其中优先级最高的一个。该宏通常用于提供一些默认实现，但允许用户在需要时重写它们。该宏同样使用了GCC的语法扩展。
- `rt_inline`：表示**将所修饰的函数定义为静态内联函数**，即在编译时将函数的代码直接嵌入到调用处，以避免隐式调用带来的额外开销。该宏同样使用了GCC的语法扩展。

#### 8.编译器相关定义

```c
/* Compiler Related Definitions */
#if defined(__ARMCC_VERSION)           /* ARM Compiler */
#define RT_SECTION(x)               __attribute__((section(x)))
#define RT_USED                     __attribute__((used))
#define ALIGN(n)                    __attribute__((aligned(n)))
#define RT_WEAK                     __attribute__((weak))
#define rt_inline                   static __inline

/* module compiling */
#ifdef RT_USING_MODULE
#define RTT_API                     __declspec(dllimport)
#else
#define RTT_API                     __declspec(dllexport)
#endif /* RT_USING_MODULE */

#elif defined (__IAR_SYSTEMS_ICC__)     /* for IAR Compiler */
#define RT_SECTION(x)               @ x
#define RT_USED                     __root
#define PRAGMA(x)                   _Pragma(#x)
#define ALIGN(n)                    PRAGMA(data_alignment=n)
#define RT_WEAK                     __weak
#define rt_inline                   static inline
#define RTT_API
#elif defined (__GNUC__)                /* GNU GCC Compiler */

#ifndef RT_USING_LIBC
/* the version of GNU GCC must be greater than 4.x */
typedef __builtin_va_list           __gnuc_va_list;
typedef __gnuc_va_list              va_list;
#define va_start(v,l)               __builtin_va_start(v,l)
#define va_end(v)                   __builtin_va_end(v)
#define va_arg(v,l)                 __builtin_va_arg(v,l)
#endif /* RT_USING_LIBC */

#define RT_SECTION(x)               __attribute__((section(x)))
#define RT_USED                     __attribute__((used))
#define ALIGN(n)                    __attribute__((aligned(n)))
#define RT_WEAK                     __attribute__((weak))
#define rt_inline                   static __inline
#define RTT_API

#elif defined (__ADSPBLACKFIN__)        /* for VisualDSP++ Compiler */
#define RT_SECTION(x)               __attribute__((section(x)))
#define RT_USED                     __attribute__((used))
#define ALIGN(n)                    __attribute__((aligned(n)))
#define RT_WEAK                     __attribute__((weak))
#define rt_inline                   static inline
#define RTT_API

#elif defined (_MSC_VER)
#define RT_SECTION(x)
#define RT_USED
#define ALIGN(n)                    __declspec(align(n))
#define RT_WEAK
#define rt_inline                   static __inline
#define RTT_API

#elif defined (__TI_COMPILER_VERSION__)
/* The way that TI compiler set section is different from other(at least
    * GCC and MDK) compilers. See ARM Optimizing C/C++ Compiler 5.9.3 for more
    * details. */
#define RT_SECTION(x)
#define RT_USED
#define PRAGMA(x)                   _Pragma(#x)
#define ALIGN(n)
#define RT_WEAK
#define rt_inline                   static inline
#define RTT_API

#elif defined (__TASKING__)
#define RT_SECTION(x)               __attribute__((section(x)))
#define RT_USED                     __attribute__((used, protect))
#define PRAGMA(x)                   _Pragma(#x)
#define ALIGN(n)                    __attribute__((__align(n)))
#define RT_WEAK                     __attribute__((weak))
#define rt_inline                   static inline
#define RTT_API
#else
    #error not supported tool chain
#endif /* __ARMCC_VERSION */
```

1. `typedef __builtin_va_list  __gnuc_va_list`： 定义了一个新类型`__gnuc_va_list`，并使用 `__builtin_va_list` 进行初始化。`__builtin_va_list` 是GCC内建的类型，用于表示可变参数列表中的参数，并在实现中进行处理。由于可变参数的实现和操作系统和编译器等因素相关，因此需要使用 `__builtin_va_list` 类型来实现可变参数列表。
2. `typedef __gnuc_va_list va_list`： 定义了一个名为`va_list`的新类型，并将其重命名为`__gnuc_va_list`。
3. `#define va_start(v,l) __builtin_va_start(v,l)`： 将 `va_start()` 重命名为 `__builtin_va_start()`，从而能够使用 GCC 内建的函数 `__builtin_va_start()` 实现可变参数的功能。该宏的作用是对变参列表进行初始化，获取第一个参数的地址和类型，并返回可变参数队列中下一个参数的地址。
4. `#define va_end(v) __builtin_va_end(v)`： 将 `va_end()` 重命名为 `__builtin_va_end()`，从而能够使用 GCC 内建的函数 `__builtin_va_end()` 实现可变参数的功能。该宏的作用是清除可变参数列表，并将其指针置为 NULL。
5. `#define va_arg(v,l) __builtin_va_arg(v,l)`： 将 `va_arg()` 重命名为 `__builtin_va_arg()`，并使用 GCC 内建的函数 `__builtin_va_arg()` 实现可变参数的功能。该宏的作用是获取可变参数队列中的下一个参数，并将指针指向该参数的位置。
6. `#define PRAGMA(x) _Pragma(#x)`：将参数`x`转化为字符串并使用`_Pragma()`将其作为编译指令执行。`_Pragma`是C99标准引入的一个新特性，它允许程序员在说明文件中进行诸如#pragma等命令式编译指令的嵌入式编程。而`#pragma`则是一种编译指令，用于控制编译器的一些行为，比如告诉编译器去链接某个库、指定编译器选项等。

#### 9.RT-Thread错误码定义

```c
/* RT-Thread error code definitions */
#define RT_EOK                          0               /**< There is no error */
#define RT_ERROR                        1               /**< A generic error happens */
#define RT_ETIMEOUT                     2               /**< Timed out */
#define RT_EFULL                        3               /**< The resource is full */
#define RT_EEMPTY                       4               /**< The resource is empty */
#define RT_ENOMEM                       5               /**< No memory */
#define RT_ENOSYS                       6               /**< No system */
#define RT_EBUSY                        7               /**< Busy */
#define RT_EIO                          8               /**< IO error */
#define RT_EINTR                        9               /**< Interrupted system call */
#define RT_EINVAL                       10              /**< Invalid argument */
```

- `RT_EOK`：表示没有错误。
- `RT_ERROR`：表示发生了一般性的错误。
- `RT_ETIMEOUT`：表示超时错误。
- `RT_EFULL`：表示资源已满。
- `RT_EEMPTY`：表示资源为空。
- `RT_ENOMEM`：表示内存不足。
- `RT_ENOSYS`：表示没有该系统。
- `RT_EBUSY`：表示忙碌。
- `RT_EIO`：表示输入/输出错误。
- `RT_EINTR`：表示中断的系统调用。
- `RT_EINVAL`：表示无效的参数。
