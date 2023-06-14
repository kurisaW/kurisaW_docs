1、在vi编译器中，单击o可以进入插入模式并换行

2、在当前代码的上一行另起一行：大写O

3、直接到行首插入：大写I

4、行尾插入：大写A

5、Ctrl+l:清屏

![image-20220526181457380](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526181457380.png)

![image-20220526182346204](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526182346204.png)

```c
:w File 	另存为File给出的文件名，不退出

:r File   	(Read)读入File指定的文件内容插入到当前光标处
```

![image-20220526183054983](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526183054983.png)

![image-20220526183552518](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526183552518.png)

![image-20220526184031839](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526184031839.png)

![image-20220526184441268](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526184441268.png)

![image-20220526185253847](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220526185253847.png)

```c
//利用__FILE__,__LINE__,__FUCTION__实现代码跟踪调试，也就是日志查看设置，平时写项目时可以多多使用，能方便及时的知道项目的运行情况。

#include<stdio.h>
int main()
{
	printf("%s,%s,%d\n",__FILE__,__FUCTION__,__LINE__)
}
```

