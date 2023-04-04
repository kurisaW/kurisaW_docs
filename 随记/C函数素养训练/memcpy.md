## memcpy()

#### 1、描述

C 库函数 

**int memcmp(const void \*str1, const void \*str2, size_t n))** 

作用：把存储区 **str1** 和存储区 **str2** 的前 **n** 个字节进行比较。

#### 2、声明

下面是 memcmp() 函数的声明。

```
int memcmp(const void *str1, const void *str2, size_t n)
```

#### 3、参数

- **str1** -- 指向内存块的指针。
- **str2** -- 指向内存块的指针。
- **n** -- 要被比较的字节数。

#### 4、返回值

- 如果返回值 < 0，则表示 str1 小于 str2。
- 如果返回值 > 0，则表示 str1 大于 str2。
- 如果返回值 = 0，则表示 str1 等于 str2。

#### 5、示例

```c
#include<string.h>
#include<stdio.h>

int main() {
	char s1[] = "Hello";
	char s2[] = "Hello";
	int r;
	r = memcmp(&s1,&s2,strlen(s1));
    
	if(!r)		//！r 非零返回的是 1  这个是非运算，计算机是二进制的，不是零就是一了 
		printf("s1 and s2 are identical\n");		/* s1等于s2 */
	else if(r<0)
		printf("s1 is less than s2\n");				/* s1小于s2 */
	else
		printf("s1 is greater than s2\n"); 			/* s1大于s2 */
    
	printf("%d\n",!r); //输出是一， 
	printf("%d\n",r);

    return 0;
}
```

#### 6、说明

该函数是按字节比较的。

例如：

s1,s2为字符串时候memcmp(s1,s2,1)就是比较s1和s2的第一个字节的ascII码值；

memcmp(s1,s2,n)就是比较s1和s2的前n个字节的ascII码值；

如:

```c
char *s1="abc";

char *s2="acd";

int r=memcmp(s1,s2,3);
```

**就是比较s1和s2的前3个字节，第一个字节相等，第二个字节比较中大小已经确定，不必继续比较第三字节了所以r=-1. **

#### 7、注意

`对于memcmp()，如果两个字符串相同而且count大于字符串长度的话，memcmp不会在\0处停下来，会继续比较\0后面的内存单元，直到_res不为零或者达到count次数。`

```
例子：      

  char a1[]="ABCD";   

  char a2[]="ABCD";       

对于memcmp(a1,a2,10)，memcmp在两个字符串的\0之后继续比较。所以，如果想使用memcmp比较字符串，要保证count不能超过最短字符串的长度，否则结果有可能是错误的。
```

