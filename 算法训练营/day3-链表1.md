## 今日任务

* 链表理论基础

* 203.移除链表元素

* 707.设计链表

* 206.反转链表

##  1.链表理论基础

#### （1）什么是链表？

**链表是一种通过指针串联在一起的线性结构，每一个节点由两部分组成，一个是数据域一个是指针域（存放指向下一个节点的指针），最后一个节点的指针域指向null（空指针的意思）。**

**链表的入口节点称为链表的头节点也就是head。**

#### （2）链表的类型

常见的链表类型有以下几种：

**<1>单链表**

单向链表是一种包含两部分的数据结构，即一个是数据部分(`数据域`)，另一个是地址部分(`指针域`)，其中包含下一个或后继节点的地址。节点中的地址部分也称为**指针**。

![image-20230217114232162](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171142211.png)

在单链表中，每一个节点除了包括自身的数值外，还包含了下一个节点的地址，在第三个节点它的地址部分包含的是NULL值，因为它不指向任何节点。此外，保存初始节点地址的指针称为**头指针**。

由于单链表的指针域只保存了下一个节点的地址，因此**在单链表中，只能向前遍历，而不能反向遍历**。

```cpp
// 单向链表中节点的表示

struct node 
{ 
int data; 
struct node *next; 
} 
```

**<2>双链表**

前面说了单链表中的指针域只能指向节点的下一个节点。而在双链表中，**每一个节点有两个指针域，一个指向下一个节点，一个指向上一个节点**。

这就意味着，双向链表**不仅支持向前查询，还可以向后查询**。

![image-20230217114149969](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171141017.png)

```cpp
// 双向链表中节点的表示

struct node 
{ 
int data; 
struct node *next; 
struct node *prev; 
} 
```

**<3>循环链表**

循环链表，是指头节点和尾节点首位相连，以此形成一个循环结构。也可以这么认为，循环链表是单链表的变体。也就是说，**循环链表没有起始节点和结束节点**，我们可以朝任意方向进行遍历（向前或者向后）。

![image-20230217114529563](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171145612.png)

```cpp
// 循环链表中节点的表示

struct node 
{ 
int data; 
struct node *next; 
} 
```

乍一看，循环链表和单链表节点的表示一样，其实他们之间唯一最本质的区别就是最后一个节点不指向单链表中的任何节点，因此单链表的链接部分包含一个NULL值；相反，循环链表的最后一个节点的链接部分保存着第一个节点的地址。

#### （3）链表的存储方式

前面在学习数组的时候我们知道，数组在内存中是连续分布的，但是**链表则是通过指针域的指针 链接在内存中的各个节点上，也就是说链表中的节点在内存中不是连续分布的，而是零散分布在内存中的某个地址上，分配机制取决于操作系统的内存管理。**

![image-20230217120000271](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171200336.png)

在上图中我们可以看出，该链表的起始节点为2，终止节点为7，各个节点分布在内存中的不同地址空间上，通过指针串联在一起。

#### （4）链表的定义

给出链表节点的定义：

```cpp
// 单链表
strcut ListNode{
	int val;	//节点上存储的元素
	ListNode *next;		//指向下一个节点的指针
	ListNode(int x): val(x),next(NULL){}	// 节点的构造函数
};
```

下面给出使用自己定义构造函数和使用默认构造函数的区别（推荐自定义构造函数）：

1、通过自己定义构造函数初始化节点：

```cpp
ListNode* head = new ListNode(5);
```

2、使用默认构造函数初始化节点：

```cpp
ListNode* head = new ListNode();
head->val = 5;
```

从上面不难看出，如果使用默认构造函数的话，在初始化时是不可以直接给变量赋值的。

#### （5）链表的操作

**<1>删除节点**

我们以下图为例，目的时删除D节点：

![image-20230217142406253](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171424698.png)

具体操作：

C节点的next指针指向的是D节点，而我们的需求是删除D节点，那么只需要**将C节点的next指针指向E节点就可以了**。

此时的D节点从链表中删除，但是它依然存放在内存中，需要我们手动释放这段内存。

**<2>添加节点**

在下图中，我们需要在C节点和D节点中添加一个F节点：

![image-20230217142827964](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171428029.png)

添加F节点，只需要**将C节点的next指针指向F节点，同时F节点的next指针指向D节点**，这样就完成了节点的添加。

#### （6）性能分析

这里我们将链表和数组做一个对比，详见下图：

![image-20230217143205888](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171432948.png)

* 数组在定义的时候，长度就是固定的，想要改动数组的长度，就需要重新定义一个新的数组。
* 链表的长度可以是不固定的，并且可以实现动态增删，适合场景：数据量不固定、增删频繁、查询需求较少

## 2.Leetcode203：移除链表元素

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/remove-linked-list-elements

#### （1）题目

**给你一个链表的头节点 head 和一个整数 val ，请你删除链表中所有满足 Node.val == val 的节点，并返回 新的头节点 。**

示例 1：

![image-20230217143757762](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171437824.png)

```cpp
输入：head = [1,2,6,3,4,5,6], val = 6
输出：[1,2,3,4,5]
```

示例 2：

```cpp
输入：head = [], val = 1
输出：[]
```

示例 3：

```cpp
输入：head = [7,7,7,7], val = 7
输出：[]
```

**提示：**

* 列表中的节点数目在范围 [0, 104] 内
* 1 <= Node.val <= 50
* 0 <= val <= 50

#### （2）思路

**案例1：**

> 链表：1->4->2->4		目的：移除元素4

![image-20230217144046231](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171440412.png)

其实这道题还是比较简单的，首先可以看出它是一个单链表，那么我们定义好节点的数据域和地址域，让节点1的next指针指向节点2，并且让节点2的next指针指向NULL，那么这道题就算完成了，最后的结果也就是下面这张图。

![image-20230217144342989](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171443047.png)

那么此外我们还需要完成节点4的内存回收工作！

**案例二：**

由于考虑到在实际应用中可能存在对头节点的删除需求，所以我们这里也额外做个分析。

对于链表的操作有两种形式：

* **1.直接使用原来的链表进行删除操作。**
* **2.设置一个虚拟头节点再进行删除操作。**

**<操作1>：直接使用原来的链表进行移除**

移除头节点和移除其他节点的擦欧总是不一样的，因为链表的其他节点都是通过前面一个节点来移除房前节点，而头节点没有前节点。

那么对于头节点的移除，需要将头节点向后移动一位就可以了，同时记得将原头节点从内存中释放。

![image-20230217145654957](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171456064.png)

对于操作一这种方法虽然可以实现，但是无疑是增加了代码的逻辑性，需要我们单独写一段逻辑处理头节点。那么这样的话不妨我们试试操作2的方法。

**<操作2>：设置一个虚拟头节点再进行删除操作**

如何设置虚拟头节点，**首先我们需要给链表添加一个虚拟头节点作为新的头节点，同时我们移除旧的头节点，也就是下图中的元素1，并且将新的头节点的next指针指向第二个节点4**。

![image-20230217150125673](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171501788.png)

具体实现我们详见代码。

#### （3）代码实现

```cpp
// 操作1实现：直接使用原来的链表进行删除操作

class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        // 删除头结点
        while (head != NULL && head->val == val) { 
        // head != NULL：这里判断头节点不为空是因为后续需要对头节点的值进行操作，如果为空就相当于操作空指针，编译会报错。
            ListNode* tmp = head;
            head = head->next;
            delete tmp;		// 此处需要对旧的头节点进行内存回收
        }

        // 删除非头结点
        ListNode* cur = head;	// 当前节点
        while (cur != NULL && cur->next!= NULL) {
        // cur->next!= NULL：这里是同样的道理，不可操作空指针
            if (cur->next->val == val) {
                ListNode* tmp = cur->next;
                cur->next = cur->next->next;
                delete tmp;
            } else {
                cur = cur->next;
            }
        }
        return head;
    }
};
```
这里需要注意几点：

* 对于可能存在节点的值为空的情况我们要避免空指针操作，否则编译会报错
* 操作1的关键代码就是下面的这两部分

![image-20230217153228463](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171532596.png)

```cpp
// 操作2实现：设置一个虚拟头节点再进行删除操作

class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        ListNode* dummyHead = new ListNode(0); // 设置一个虚拟头结点
        dummyHead->next = head; // 将虚拟头结点指向head，这样方面后面做删除操作
        ListNode* cur = dummyHead;
        while (cur->next != NULL) {
            if(cur->next->val == val) {
                ListNode* tmp = cur->next;
                cur->next = cur->next->next;
                delete tmp;
            } else {
                cur = cur->next;
            }
        }
        head = dummyHead->next;
        delete dummyHead;
        return head;
    }
};
```

## 3.Leetcode707：设计链表

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/design-linked-list

#### （1）题目

**设计链表的实现。您可以选择使用单链表或双链表。单链表中的节点应该具有两个属性：val 和 next。val 是当前节点的值，next 是指向下一个节点的指针/引用。如果要使用双向链表，则还需要一个属性 prev 以指示链表中的上一个节点。假设链表中的所有节点都是 0-index 的。**

**在链表类中实现这些功能：**

* get(index)：获取链表中第 index 个节点的值。如果索引无效，则返回-1。
* addAtHead(val)：在链表的第一个元素之前添加一个值为 val 的节点。插入后，新节点将成为链表的第一个节点。
* addAtTail(val)：将值为 val 的节点追加到链表的最后一个元素。
* addAtIndex(index,val)：在链表中的第 index 个节点之前添加值为 val  的节点。如果 index 等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。
* deleteAtIndex(index)：如果索引 index 有效，则删除链表中的第 index 个节点。


示例：

```cpp
MyLinkedList linkedList = new MyLinkedList();
linkedList.addAtHead(1);
linkedList.addAtTail(3);
linkedList.addAtIndex(1,2);   //链表变为1-> 2-> 3
linkedList.get(1);            //返回2
linkedList.deleteAtIndex(1);  //现在链表是1-> 3
linkedList.get(1);            //返回3
```

**提示：**

* 0 <= index, val <= 1000
* 请不要使用内置的 LinkedList 库。
* get, addAtHead, addAtTail, addAtIndex 和 deleteAtIndex 的操作次数不超过 2000。

#### （2）思路

分析题目给出的要求，主要是需要完成以下功能：

* 获取链表第index个节点的值
* 在链表的最前面插入一个节点
* 在链表的最后面插入一个节点
* 在链表第index个节点面前插入一个节点
* 删除链表的第index个节点

#### （3）代码实现

```cpp
class MyLinkedList {
public:
    // 定义链表节点结构体
    struct LinkedNode {
        int val;
        LinkedNode* next;
        LinkedNode(int val):val(val), next(nullptr){}
    };

    // 初始化链表
    MyLinkedList() {
        _dummyHead = new LinkedNode(0); // 这里定义的头结点 是一个虚拟头结点，而不是真正的链表头结点
        _size = 0;
    }

    // 获取到第index个节点数值，如果index是非法数值直接返回-1， 注意index是从0开始的，第0个节点就是头结点
    int get(int index) {
        if (index > (_size - 1) || index < 0) {
            return -1;
        }
        LinkedNode* cur = _dummyHead->next;
        while(index--){ // 如果--index 就会陷入死循环
            cur = cur->next;
        }
        return cur->val;
    }

    // 在链表最前面插入一个节点，插入完成后，新插入的节点为链表的新的头结点
    // 这里选择插入新的头节点采用的是操作1，详情可查看第二小节中的思路
    void addAtHead(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        newNode->next = _dummyHead->next;
        _dummyHead->next = newNode;
        _size++;
    }

    // 在链表最后面添加一个节点
    void addAtTail(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while(cur->next != nullptr){
            cur = cur->next;
        }
        cur->next = newNode;
        _size++;
    }

    // 在第index个节点之前插入一个新节点，例如index为0，那么新插入的节点为链表的新头节点。
    // 如果index 等于链表的长度，则说明是新插入的节点为链表的尾结点
    // 如果index大于链表的长度，则返回空
    // 如果index小于0，则在头部插入节点
    void addAtIndex(int index, int val) {

        if(index > _size) return;
        if(index < 0) index = 0;        
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        while(index--) {
            cur = cur->next;
        }
        newNode->next = cur->next;
        cur->next = newNode;
        _size++;
    }

    // 删除第index个节点，如果index 大于等于链表的长度，直接return，注意index是从0开始的
    void deleteAtIndex(int index) {
        if (index >= _size || index < 0) {
            return;
        }
        LinkedNode* cur = _dummyHead;
        while(index--) {
            cur = cur ->next;
        }
        LinkedNode* tmp = cur->next;
        cur->next = cur->next->next;
        delete tmp;
        _size--;
    }

    // 打印链表
    void printLinkedList() {
        LinkedNode* cur = _dummyHead;
        while (cur->next != nullptr) {
            cout << cur->next->val << " ";
            cur = cur->next;
        }
        cout << endl;
    }
private:
    int _size;
    LinkedNode* _dummyHead;

};
```

## 4.Leetcode206：反转链表

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/reverse-linked-list

#### （1）题目

**给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。**

示例 1：

![image-20230217163726826](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171637917.png)

```cpp
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

示例 2：

![image-20230217163749967](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171637025.png)

```cpp
输入：head = [1,2]
输出：[2,1]
```

示例 3：

```cpp
输入：head = []
输出：[]
```

**提示：**

* 链表中节点的数目范围是 [0, 5000]
* -5000 <= Node.val <= 5000

**进阶：链表可以选用迭代或递归方式完成反转。你能否用两种方法解决这道题？**

#### （2）思路

链表的反转，只需要改变next指针的指向即可。

![](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302171639640.png)

#### （3）双指针法

对于链表的反转问题，我们可以通过使用双指针的方式来解决这个问题。

* cur指针，指向链表的头节点
* pre指针，定义为cur指针的前一个节点，也就是让cur指针原本指向后一位的指针指向pre指针的地址

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* temp; // 作为一个临时节点，保存cur的下一个节点
        ListNode* cur = head;
        ListNode* pre = NULL;	// 之所以初始化为空，就是为了让cur节点指向pre节点，而我们的目标就是尾节点反转成目标的头节点，也就是NULL
        // 所以此处当pre节点和cur节点遍历到尾节点时，也就是cur指向NULL，这也就意味反转完成，因此while()的值设为cur
        while(cur) {
            temp = cur->next;  // 保存一下 cur的下一个节点，因为接下来要改变cur->next
            cur->next = pre; // 翻转操作
            // 更新pre 和 cur指针
            pre = cur;
            cur = temp;
        }
        return pre;	// 返回的是新链表的头节点pre
    }
};
```

#### （4）递归法

前面讲了双指针法，其实递归法与之逻辑都是大体一样的，不过对于递归，我们有**自前向后递归、以及自后向前递归**两种方法。

```cpp
// 递归法：自前向后

class Solution {
public:
    ListNode* reverse(ListNode* pre,ListNode* cur){
        if(cur == NULL) return pre;
        ListNode* temp = cur->next;
        cur->next = pre;
        // 可以和双指针法的代码进行对比，如下递归的写法，其实就是做了这两步
        // pre = cur;
        // cur = temp;
        return reverse(cur,temp);
    }
    ListNode* reverseList(ListNode* head) {
        // 和双指针法初始化是一样的逻辑
        // ListNode* cur = head;
        // ListNode* pre = NULL;
        return reverse(NULL, head);
    }

};
```

```cpp
// 递归法：自后向前

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        // 边缘条件判断
        if(head == NULL) return NULL;
        if (head->next == NULL) return head;
        
        // 递归调用，翻转第二个节点开始往后的链表
        ListNode *last = reverseList(head->next);
        // 翻转头节点与第二个节点的指向
        head->next->next = head;
        // 此时的 head 节点为尾节点，next 需要指向 NULL
        head->next = NULL;
        return last;
    }
}; 
```

