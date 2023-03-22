## 今日任务

* 24.两两交换链表中的节点

* 19.删除链表的倒数第N个节点

* 面试题02.07.链表相交

* 142.环形链表II 

* 总结

##  1.Leetcode24：两两交换链表中的节点

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/swap-nodes-in-pairs

#### （1）题目

**给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。**

示例 1：

![image-20230218104104240](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181041283.png)

```cpp
输入：head = [1,2,3,4]
输出：[2,1,4,3]
```

示例 2：

```cpp
输入：head = []
输出：[]
```

示例 3：

```cpp
输入：head = [1]
输出：[1]
```

**提示：**

* 链表中节点的数目在范围 [0, 100] 内
* 0 <= Node.val <= 100

#### （2）思路

前面我们有了链表的相关基础知识，知道了对于链表节点的操作有两种形式：

* **1.直接使用原来的链表进行删除操作。**
* **2.设置一个虚拟头节点再进行删除操作。**

相比较第一种，第二种虚拟头节点的形式更加方便。

**初始时，cur指向虚拟头节点，然后依次进行三步：**

* `步骤1：将原链表的头节点变成节点2`
* `步骤2：将原链表的节点2变成一个临时节点tmp（tmp：指向原链表的头节点）`
* `步骤3：将原链表的节点3变成一个临时节点tmp2（tmp2：指向原链表的节点3）（ps：此处这样重复定义是为了后续循环条件的退出）`
* `ps：原链表：未加入虚拟头节点的链表，也就是初始化时的链表`

![image-20230218111454677](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181114718.png)

操作后的链表：

![image-20230218111528059](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181115102.png)

**终止条件：**

`当cur节点经过第一轮循环时，说明这个链表至少有2个节点，此时cur已经成了原链表的节点2，再进行下一次循环时，如果还有新的节点，只要满足cur节点之后还存在1个或2个节点，循环继续，否则结束循环，并返回原链表的头节点`

#### （3）代码实现

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode* dummyHead = new ListNode(0); // 设置一个虚拟头结点
        dummyHead->next = head; // 将虚拟头结点指向head，这样方面后面做删除操作
        ListNode* cur = dummyHead;
        while(cur->next != nullptr && cur->next->next != nullptr) {
            ListNode* tmp = cur->next; // 记录临时节点，原本的头节点
            ListNode* tmp1 = cur->next->next->next; // 记录临时节点，原本的节点3

            cur->next = cur->next->next;    // 步骤一
            cur->next->next = tmp;          // 步骤二
            cur->next->next->next = tmp1;   // 步骤三

            cur = cur->next->next; // cur移动两位，准备下一轮交换
        }
        return dummyHead->next;
    }
};
```

## 2.Leetcode19：删除链表的倒数第N个节点

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/remove-nth-node-from-end-of-list

#### （1）题目

**给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。**

示例 1：

![image-20230218114024717](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181140762.png)

```cpp
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

示例 2：

```cpp
输入：head = [1], n = 1
输出：[]
```

示例 3：

```cpp
输入：head = [1,2], n = 1
输出：[1]
```

**提示：**

* 链表中结点的数目为 sz
* 1 <= sz <= 30
* 0 <= Node.val <= 100
* 1 <= n <= sz

**进阶：你能尝试使用一趟扫描实现吗？**

#### （2）思路

**先抓题意，删除倒数第n个节点，我们很自然的就想到快慢指针法，通过设置一个fast指针和一个slow指针，首先让fast指针移动n步，到达目的节点后，fast指针和slow指针再同时移动，直到fast指针移至尾节点，此时slow指针也刚好指向目标节点，那么这里我们只需要让slow->next = slow->next->next即可完成对目标节点的删除。**

同样的对于链表的操作，我们还是采取虚拟头节点的方式进行设计。

**<1>首先定义fast指针和slow指针，初始值为虚拟头节点：**

![image-20230218115133608](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181151708.png)

**<2>fast走n+1步（因为加入了虚拟头节点）**

![image-20230218115254196](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181152230.png)

**<3>fast和slow同时移动，直到fast指向链表末**

![image-20230218115341005](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181153041.png)

**<4>删除slow指向的下一个节点**

![image-20230218115452233](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181154272.png)

#### （3）快慢指针法

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* dummyHead = new ListNode(0);
        dummyHead->next = head;
        ListNode* slow = dummyHead;
        ListNode* fast = dummyHead;
        while(n-- && fast != NULL) {	// 让fast指向目标节点
            fast = fast->next;
        }
        fast = fast->next; // fast再提前走一步，因为需要让slow指向删除节点的上一个节点
        while (fast != NULL) {
            fast = fast->next;
            slow = slow->next;
        }
        slow->next = slow->next->next; 
        
        // ListNode *tmp = slow->next;  C++释放内存的逻辑
        // slow->next = tmp->next;
        // delete nth;
        
        return dummyHead->next;
    }
};
```

## 3.Leetcode面试题02.07：链表相交

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci

#### （1）题目

**给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。**

图示两个链表在节点 c1 开始相交：

![image-20230218151939703](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181519759.png)

题目数据保证整个链式结构中不存在环。

注意，函数返回结果后，链表必须保持其原始结构 。

示例 1：

```cpp
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
输出：Intersected at '8'
解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。
在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
```

示例 2：

```cpp
输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Intersected at '2'
解释：相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。
在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
```

示例 3：

```cpp
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。
由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
这两个链表不相交，因此返回 null 。
```

**提示：**

* listA 中节点数目为 m
* listB 中节点数目为 n
* 0 <= m, n <= 3 * 104
* 1 <= Node.val <= 105
* 0 <= skipA <= m
* 0 <= skipB <= n
* 如果 listA 和 listB 没有交点，intersectVal 为 0
* 如果 listA 和 listB 有交点，intersectVal == listA[skipA + 1] == listB[skipB + 1]

**进阶：你能否设计一个时间复杂度 O(n) 、仅用 O(1) 内存的解决方案？**

#### （2）思路

根据题意，我们可以有这样一种思路，首先想要找到相交节点，但是可能两个链表的长度不一样，怎么对其是需要考虑的，通过上面的几个示例我们也可以看出，只要让链表1和链表二右对齐即可。

那么在算法中如何实现呢，那么只需要先**分别求出两个链表的长度，然后我们就可以得出两个链表长度的差值n，这个差值就是我们对其的关键**所在啦。

**先让长链表移动n步，然后两个链表同时向后移动，并对节点的数值进行判断是否一致，相同的话就是我们所要求解的相交节点了。**

![image-20230218161958873](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181619944.png)

#### （3）代码实现

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* curA = headA;
        ListNode* curB = headB;
        int lenA = 0, lenB = 0;
        while (curA != NULL) { // 求链表A的长度
            lenA++;
            curA = curA->next;
        }
        while (curB != NULL) { // 求链表B的长度
            lenB++;
            curB = curB->next;
        }
        curA = headA;
        curB = headB;
        // 让curA为最长链表的头，lenA为其长度
        if (lenB > lenA) {
            swap (lenA, lenB);
            swap (curA, curB);
        }
        // 求长度差
        int gap = lenA - lenB;
        // 让curA和curB在同一起点上（末尾位置对齐）
        while (gap--) {
            curA = curA->next;
        }
        // 遍历curA 和 curB，遇到相同则直接返回
        while (curA != NULL) {
            if (curA == curB) {
                return curA;
            }
            curA = curA->next;
            curB = curB->next;
        }
        return NULL;
    }
};
```

* 时间复杂度：O(n+m)
* 空间复杂度：O(1)

## 4.Leetcode142：环形链表II

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/linked-list-cycle-ii

#### （1）题目

**给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。**

**如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。**`注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。`

**不允许修改链表。**

示例 1：

![image-20230218163000655](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181630702.png)

```cpp
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

示例 2：

![image-20230218163029607](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181630652.png)

```cpp
输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```

示例 3：

![image-20230218163050685](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181630718.png)

```cpp
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```

**提示：**

* 链表中节点的数目范围在范围 [0, 104] 内
* -105 <= Node.val <= 105
* pos 的值为 -1 或者链表中的一个有效索引

**进阶：你是否可以使用 O(1) 空间解决此题？**

#### （2）思路

对于这道题的分析，就是为了让我们求解一个链表中是否存在环形链表，如果存在则返回该环形链表的头节点，无环则返回NULL。

对于这道题我们需要解决以下两点：

* 如何判断链表有环
* 如果有环，怎么找到这个环的入口

**<1>如何判断链表有环**

对于环形链表的判断，我们采取快慢指针法，分别定义fast指针和slow指针，**从头节点出发，fast指针每次移动2个节点，slow指针移动1个节点**，如果fast指针和slow指针在中途相遇，则说明存在环形链表。

由于fast指针走两步，slow指针走一步，那么理论上讲，如果存在环形链表的话是一定存在相遇机会的，动画如下，选自carl大神制作：

![141.环形链表](https://code-thinking.cdn.bcebos.com/gifs/141.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8.gif)

**<2>如果有环，怎么找到这个环的入口**

既然我们已经有了判断唤醒链表的方式，那么接下来就需要找到环形链表的入口了。

假设从头节点到环形入口的节点数为x，环形入口节点到fast指针与slow指针的相遇节点的节点数为y。

![image-20230218173510617](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181735057.png)

**当快指针和慢指针相遇时，快指针的走过的节点数不就等于慢指针走过节点数的两倍嘛，只要我们求出快慢指针走过的节点数，就可以联立成一个等式，并且等式中的x值就是我们要求的结果**，那么据此我们可以得出以下结论：

> 1.`slow指针走过的节点数 = x + y`

> 2.`fast指针走过的节点数 = x + y + n*(y+z)`	n：代表slow指针进入环形链表后，此时fast指针在环中的循环次数

> 3.得到等式：`x + y = x + y + n*(y+z)`	此处需要注意：n >= 1，因为在环中fast指针必然是会经历一次循环才有可能被slow指针追上，朋友们可以自己推算一遍

> 4.我们的目标为x，因此化简上式：`x = n (y + z) - y`

> 5.当n等于1时，我们可以得知上式结果为：`x = z`，这就意味着此时从相遇节点到环形链表的入口节点正好等于从头节点到入口节点的长度。

> 6.根据结论5的分析，我们只需要在fast指针和slow指针相遇时定义一个index指针，同时从头节点也定义一个index2指针，两个指针同时出发，当这两个指针相遇的时候正好就是环形入口的节点

动画如下：

![142.环形链表II（求入口）](https://code-thinking.cdn.bcebos.com/gifs/142.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8II%EF%BC%88%E6%B1%82%E5%85%A5%E5%8F%A3%EF%BC%89.gif)

上面分析的结论是基于n等于1的，那么当循环此处大于1该如何分析呢？

其实即便n大于1，结果也是一样的，不同的是index1指针会在环中多转(n - 1)圈，然后再遇到index2，建议可以做个示例自己试试。

#### （3）代码实现

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while(fast != NULL && fast->next != NULL) {
            slow = slow->next;
            fast = fast->next->next;
            // 快慢指针相遇，此时从head 和 相遇点，同时查找直至相遇
            if (slow == fast) {
                ListNode* index1 = fast;
                ListNode* index2 = head;
                while (index1 != index2) {
                    index1 = index1->next;
                    index2 = index2->next;
                }
                return index2; // 返回环的入口
            }
        }
        return NULL;
    }
};
```

## 5.链表总结

![image-20230218181324408](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302181813815.png)

图片来源： [代码随想录知识星球](https://programmercarl.com/other/kstar.html)成员：[海螺人](https://wx.zsxq.com/dweb2/index/footprint/844412858822412)

