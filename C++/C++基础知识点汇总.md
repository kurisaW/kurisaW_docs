## 基本格式

```cpp
#include <iostream>
using namespace std;

int main(){
	std::cout << "hello world!" << endl;
    cout << "hello world!\n";
    return 0;
}
```

* 编译预处理命令: `#include <iostream>`（输入输出流）
* 命令空间 `using namespace std;`
* `cin >>` :用于输入；`cout << `:用于输出； `endl`:用于换行
* 源文件扩展名.cpp
目标代码文件（编译后）扩展名.obj
可执行文件（链接后）.exe

## 特点

* C++与C完全兼容，是C的扩展和改革
* 支持面向对象程序设计
* 生成的代码质量高
* C++在C语言基础上引入了面向对象编程（OOP）的特性，它提供了类的概念，提供了OOP（和一些非OOP）语言中常见的四个特性：**抽象、封装、继承和多态**。

## C++数据类型

主要分为三类：基本数据类型、构造数据类型、类

### 基本数据类型

* 整型
* 实型（浮点型）
* 字符型
* 布尔型
* void型

### 构造数据类型

* 数组类型
* 指针类型
* 枚举类型
* 结构体类型
* 共用体类型

### 类

* ...

## 函数重载

简单来说，函数重载就是让功能相似的函数使用同一函数名，以增加程序的可读性。

如：

```cpp
double area(double a, double b);
double area(double r);
```

* 注意：如果函数重载和形参默认值同时出现，可能会引起歧义，应该避免这种情况发生

## 类和对象*

### 1.类

类由说明部分和实现部分组成，其说明部分的形式如下：

```cpp
class 类名
{
private:
    成员表1;
    
protected:
    成员表2;
    
public:
    成员表3;
};
```

实现部分的形式如下：

```cpp
类名::成员函数名（形参表）
{
	函数体;
}
```

注意：**在类内不能对数据成员进行初始化**，同时，private\protect\public三个关键字对数据成员有不同的访问控制

* private：可以让数据成员变成私有成员，这些成员只能在类内使用，如果在类内没有写三个关键字的任意一个，则数据成员默认为私有成员；
* public：可以让全数据成员变成共有成员，全部函数都能存取共有成员的数据，其定义了类的外部接口
* protected：可以让数据成员变成保护成员，只有该类的函数，该类的派生类内的函数才能存取保护成员的数据

### 2.类的成员函数

类的成员函数的定义一般在类外完成（也可以在类内完成），其形式如下：

```cpp
类型 类名::函数成员名（参数表）
{
	函数体
}
```

其中::被称为作用域运算符，能指出函数成员是属于哪个类的

### 3.类的对象

#### 含义

如果把类看作是数据类型，则**该数据类型定义的变量就是对象**。

#### 格式

在定义类之后，就可以定义对象了，一般格式为：

```cpp
类名 对象名1,对象名2,...;
```

也可以定义一个指向对象的指针，如Clock *p；则指针p指向Clock类的一个对象

#### 对象的使用

对于一般对象（非对象指针），访问其成员的方式为：

```cpp
对象名.共有数据成员名(或共有成员函数名/参数表)
```

对于指向对象的指针，访问其成员的方式为：

```cpp
对象指针名->共有数据成员名(或共有成员函数名/参数表)
```

注意：其中`.`为点运算符；`->`为箭头运算符（类似结构体）

#### 示例

在主函数中调用Clock类中的show()函数，可写成如下形式：

```cpp
Clock P, *p = &P;//定义对象P以及指向P的指针p

P.show();	//调用对象P的show()函数成员
P->show();	//调用指针P指向的show()函数成员
(*p).show();//调用指针p指向的内容P的show()函数成员
```

## 类的访问权限

| 继承方式      | 基类的public成员  | 基类的protected成员 | 基类的private成员 | 继承引起的访问控制关系变化概括         |
| :------------ | :---------------- | :------------------ | :---------------- | :------------------------------------- |
| public继承    | 仍为public成员    | 仍为protected成员   | 不可见            | 基类的非私有成员在子类的访问属性不变   |
| protected继承 | 变为protected成员 | 变为protected成员   | 不可见            | 基类的非私有成员都为子类的保护成员     |
| private继承   | 变为private成员   | 变为private成员     | 不可见            | 基类中的非私有成员都称为子类的私有成员 |

## 构造函数与析构函数

### 1.构造函数

#### 含义

构造函数的功能是将对象初始化，**其特点是与类同名，且无返回类型**

#### 格式

```cpp
...
public:
	Clock(int newC,int newN,int newM);	//类中声明构造函数
...

Clock::Clock(int newC,int newN,int newM)
{
	c = newC;
	n = newN;
    m = newM;
}

int main()
{
	Clock p(0,0,0);	//主函数中调用构造函数来初始化对象P
    P.show();	   //对象P调用成员函数show()来完成其他目的
    return 0;
}
```

### 2.析构函数

#### 含义

类的析构函数是类的一种特殊的成员函数，它会在每次删除所创建的对象时执行。

析构函数的名称与类的名称时完全相同的，只是在前面加了一个波浪号（~）作为前缀，它不会返回任何值，也不能带有任何参数。

**只要类的对象被销毁，就会调用该类的析构函数。**

#### 作用

析构函数有利于在跳出程序（比如关闭文件、释放内存等）之前释放资源。

#### 示例

```cpp
#include <iostream>
using namespace std;

class Line
{
public:
    void setLength(double len);
    double getLength(void);
    Line();		//这是构造函数声明
    ~Line();	//这是析构函数声明

private:
    double length;
};

Line::Line(void)
{
    cout << "object is being created" << endl;
}

Line::~Line(void)
{
    cout << "object is being deleted" << endl;
}

void Line::setLength(double len)
{
    length = len;
}

double Line::getLength(void)
{
    return length;
}

int main()
{
    Line line;
    line.setLength(6.0);
    cout << "length of line :" << line.getLength() << endl;
    
    return 0;	//main函数返回前，line对象会被自动销毁
}
```

## 拷贝(复制)构造函数

### 含义

拷贝构造函数时一种特殊的构造函数，其功能是用一个已知的对象去创建另一个同类对象。

拷贝构造函数常用于：

* 通过使用另一个同类型的对象来初始化新创建的对象
* 复制对象把它作为参数传递给函数
* 复制对象，并从函数返回这个对象

### 格式

如果在类中没有定义拷贝构造函数，编译器会自行定义一个。如果类带有指针变量，并由动态内存分配，则它必须有一个拷贝构造函数。

拷贝构造函数的常见形式如下：

```cpp
classname (const classname &obj)
{
    // 构造函数的主体
}
```

### 拷贝构造函数的触发

在C++中,主要有以下几种情况会调用拷贝构造函数:

#### 1.使用一个同类型对象初始化另一个对象时

```cpp
MyClass obj1(10); 
MyClass obj2(obj1); // 调用拷贝构造函数
```

#### 2.以值传递的方式将一个对象作为参数传递给函数时

```cpp
void myFunc(MyClass obj) {
  // 函数接收到的obj是调用拷贝构造函数创建的
}

MyClass obj(10);
myFunc(obj); 
```

#### 3.返回局部对象时

```cpp
MyClass myFunc() {
  MyClass ret(10);
  return ret; // 调用拷贝构造函数后返回
}
```

#### 4.编译器优化时会让临时对象调用拷贝构造函数

```cpp
MyClass(10) + MyClass(20); // 两个临时对象会调用拷贝构造函数
```

#### 5.在容器中插入一个新元素时会调用该元素的拷贝构造函数

```cpp
std::vector<MyClass> vec; 
MyClass obj(10);
vec.push_back(obj);
```

以上主要情况会触发调用拷贝构造函数。熟悉这些情况,可以帮助诊断代码中拷贝构造的调用情况。

### 示例

```cpp
#include <iostream>
using namespace std;

class Line
{
public:
    int getLength(void);
    Line(int len);			//简单的构造函数
    Line(const Line &obj);	 //拷贝构造函数
    ~Line();			    //析构函数

private:
    int *ptr;
};

// 成员函数定义，包括构造函数
Line::Line(int len)
{
	cout << "调用构造函数" << endl;
    // 为指针分配内存
    ptr = new int;
    *ptr = len;
}

Line::Line(const Line &obj)
{
    cout << "调用拷贝构造函数并为指针ptr分配内存" << endl;
    ptr = new int;
    *ptr = *obj.ptr;	//拷贝值
}

Line::~Line(void)
{
    cout << "释放内存" << endl;
    delete ptr;
}

int Line::getLength(void)
{
    return *ptr;
}

void display(Line obj)
{
    cout << "line 大小：" << endl << obj.getLength() << endl;
}

// 程序的主函数
int main()
{
    Line line(10);
    display(line);	
    
    return 0;
}
```

## 友元函数

### 含义

类的友元函数是定义在类外部，**但有权访问类的所有私有(private)成员和保护(protected)成员。**

虽然友元函数的原型有在类的定义中出现过，但**友元函数并不是成员函数。**

友元可以是一个函数，该函数称为友元函数；友元也可以是一个类，该类称为友元类，在这种情况下，整个类机器所有成员都是友元。

### 格式

声明函数为一个类的友元，需要在类定义中该函数原型前使用关键字friend

```cpp
class Box
{
    double width;
    
public:
    double length;
    friend void printWidth(Box box);
    void setWidth(double wid);
};
```

声明类ClassTwo的所有成员函数作为类ClassOne的友元，需要在类ClassOne的定义中进行声明，声明格式如下：

```cpp
friend class ClassTwo;
```

### 使用场景

C++友元函数的主要使用场景包括:

#### 1.实现两个类之间的相互访问

如果类A需要访问类B的私有成员,可以将A声明为B的友元类,这样A就可以直接访问B的私有成员。

#### 2.实现运算符重载

重载像+、-等运算符时,需要访问类的私有成员,这时可以将运算符函数定义为类的友元。

#### 3.模板类的访问

当类模板需要访问一个类的私有成员时,可以将这个类模板定义为该类的友元。

#### 4.调试和测试类的实现

在类的实现和测试阶段,可以使用友元函数方便地访问类的私有成员,以方便调试和测试。

#### 5.避免繁琐的getter/setter方法

友元函数可以直接访问私有数据,避免定义许多getter和setter方法。

#### 6.状态检查

友元函数可以方便地访问对象的状态,用于调试等目的。

需要注意的是,友元关系不可传递,过度使用友元会影响类的封装性。所以在保证必要的功能性的情况下,要优先使用公有接口,而非友元函数。

### 示例

```cpp
#include <iostream>
using namespace std;

class Box
{
    double width;

public:
    friend void printWidth(Box box);
    void setWidth(double wid);
};

void Box::setWidth(double wid)
{
    width = wid;
}

//注意：printWidth()不是任何类的成员函数
void printWidth(Box box)
{
    /*
    因为printWidth()是Box的友元，它可以直接访问该类的任何成员
    */
    cout << "Width of box: " << box.width << endl;
}

int main()
{
    Box box;
    box.setWidth(10.0);
    printWidth(box);
    
    return 0;
}
```

## C++内联函数

### 含义

C++的内联函数通常是与类一起使用，如果一个函数是内联函数，那么在编译时，编译器会把该函数的代码副本放置在每个调用该函数的地方。

对内联函数进行任何修改，都需要重新编译函数的所有客户端，因为编译器需要重新更换一次所有的代码，否则会继续使用旧的函数。

如果想把一个函数定义为内联函数，则需要在函数名前放置inline关键字，在调用函数之前需要对函数进行定义。如果已定义的函数多于一行，编译器会忽略inline限定符。

在类定义中定义的函数都是内联函数，即使没有使用inline关键字，也就是隐式内联。

### 优缺点

* 优点: 当函数体比较小的时候, 内联该函数可以令目标代码更加高效. 对于存取函数以及其它函数体比较短, 性能关键的函数, 鼓励使用内联.

* 缺点: 滥用内联将导致程序变慢. 内联可能使目标代码量或增或减, 这取决于内联函数的大小. 内联非常短小的存取函数通常会减少代码大小, 但内联一个相当大的函数将戏剧性的增加代码大小. 现代处理器由于更好的利用了指令缓存, 小巧的代码往往执行更快。

* 结论：一个较为合理的经验准则是, 不要内联超过 10 行的函数. 谨慎对待析构函数, 析构函数往往比其表面看起来要更长, 因为有隐含的成员和基类析构函数被调用！

  另一个实用的经验准则: 内联那些包含循环或 switch 语句的函数常常是得不偿失 (除非在大多数情况下, 这些循环或 switch 语句从不被执行)。

  **有些函数即使声明为内联的也不一定会被编译器内联**， 这点很重要;比如虚函数和递归函数就不会被正常内联。 

  通常，递归函数不应该声明成内联函数。(递归调用堆栈的展开并不像循环那么简单, 比如递归层数在编译时可能是未知的, 大多数编译器都不支持内联递归函数)。

  虚函数内联的主要原因则是想把它的函数体放在类定义内, 为了图个方便, 抑或是当作文档描述其行为, 比如精短的存取函数.

### 示例

```cpp
#include <iostream>
using namespace std;

inline int Max(int x, int y)
{
    return (x > y) ? x : y; 
}

int main()
{
   cout << "Max (20,10): " << Max(20,10) << endl;
   cout << "Max (0,200): " << Max(0,200) << endl;
   cout << "Max (100,1010): " << Max(100,1010) << endl;
   return 0;
}
```

### 注意事项

* 在内联函数中不允许使用循环语句和开关语句
* 内联函数的定义必须出现在内联函数第一次调用之前
* 类结构中所在的类说明内部定义的函数是内联函数

## C++ this指针

### 含义

在C++中，this指针是一个特殊指针，它指向当前对象的实例。

**在C++中，每个对象都 能通过 this 指针来访问自己的地址。**

this 是一个隐藏的指针，可以在类的成员函数中使用，它可以用来指向调用对象。

当一个对象的成员函数被调用时，编译器会隐式地传递该对象的地址作为 this 指针。

**友元函数没有 this 指针，因为友元不是类的成员，只有成员函数才有 this 指针。**

### 实例

```cpp
#include <iostream>

class MyClass
{
private:
    int value;
    
public:
    void setValue(int value)
    {
        this->value = value;
	}
    
    void printValue()
    {
        std::cout << "Value: " << this->value << std::endl;
	}
};

int main()
{
    MyClass obj;
    obj.setValue(42);
    obj.printValue();
    
    return 0;
}
```

