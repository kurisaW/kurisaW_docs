##### 一、length()函数

c++中，length()只是用来获取字符串的长度。

> 例如：string str = “asdfghjkl” 则，str.length() = 9。

##### 二、size()函数

c++中，在获取字符串长度时，size()函数与length()函数作用相同。 除此之外，size()函数还可以获取vector类型的长度。

> 例如：vector < int> num(15,2) 则，num.size() = 15。

> 例如：string str = "d1da"; 则， cout<<str.size();

##### 三、sizeof()运算符

sizeof()运算符用来求对象所占内存空间的大小。

> char c[] = "asdsds";
>
> char* cc = c;
>
> char cn[40] = "asdsds";
>
> int a[] = {1,2,3,4,5,6};
>
> int* aa = a;
>
> cout << sizeof(c) << sizeof(cc) << sizeof(*cc) << sizeof(cn);
>
> cout << sizeof(a) << sizeof(aa) << sizeof(*aa);
>
> 结果输出：
>
> sizeof(c) = 7          //c是数组，计算到'\0'位置，结果为6*1+1=7
>
> sizeof(cc) = 8         //cc为指针类型，大小为8
>
> sizeof(*cc) = 1        //*cc指向c的第一个字符，大小为1
>
> sizeof(cn) = 40        //开辟40个char空间，大小为40*1=40
>
> sizeof(a) = 24         //a是数组，但不需计算到'\0'，结果为6*4=24
>
> sizeof(aa) = 8         //aa为指针类型，大小为8
>
> sizeof(*aa) = 4        //*aa指向a的第一个数字，大小为4

需要注意的是，如果不使用Vector作为数组进行参数传递，那么在传递数组引用是需要再传递一个数组的大小，否则在函数中无法根据首地址计算出数组大小。