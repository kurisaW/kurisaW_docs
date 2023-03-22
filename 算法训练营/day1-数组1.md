## 今日任务

* 数组理论基础

* 704.二分查找

* 27.移除元素

##  1.数组理论基础

**（1）数组是存放在连续内存空间上的相同类型数据的集合。**

注意：

* 数组下标都是从0开始的
* 数组内存空间的地址是连续的

**（2）正因为数组在内存空间的地址是连续的，所以我们在删除或者增添元素的时候，就难免要移动其他元素的地址。**

例如删除下标为3的元素，我们需要堆下标为3的元素后面的所有元素都要做移动操作，如图所示：

![image-20230215112419117](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302151124482.png)

**（3）数组的元素是不能删除的，只能使用覆盖的方式。**

**（4）C++中二维数组在地址空间上是连续的。**

通过编写一个程序来验证：

```cpp
#include <iostream>
using namespace std;

void test_arr(){
        int array[2][3] =
        {
                {0,1,2},
                {3,4,5}
        };

        cout << &array[0][0] << " "<< &array[0][1] <<" "<< &array[0][2] <<endl;
        cout << &array[1][0] << " "<< &array[1][1] <<" "<< &array[1][2] <<endl;
}

int main()
{
        test_arr();
}
```

![image-20230215114525706](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302151145095.png)

在C++中，一个int（整型）变量占据4个字节，所以相邻两个数组元素的地址差4个字节

## 2.Leetcode704：二分查找

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/binary-search

#### （1）题目

**给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。**


示例 1:

```cpp
输入: nums = [-1,0,3,5,9,12], target = 9
输出: 4
解释: 9 出现在 nums 中并且下标为 4
```


示例 2:

```cpp
输入: nums = [-1,0,3,5,9,12], target = 2
输出: -1
解释: 2 不存在 nums 中因此返回 -1
```


提示：

* 你可以假设 nums 中的所有元素是不重复的。
* n 将在 [1, 10000]之间。
* nums 的每个元素都将在 [-9999, 9999]之间。

#### （2）思路

首先确定关键词：

* 数组为有序数组
* 数组无重复元素

根据题目和提示，我们联想到二分法。

#### （3）二分法

简单说下二分法，就是查找出特定元素（target）的位置，如果找到的话返回该元素的下标，如果没找到的话就返回-1。

关于二分法的写法，区间的定义一般分为两种：

* 左闭右闭 [left, right]
* 左闭右开 [left, right)

根据二分法的两种写法，我们分别求解：

**<1>第一种写法，我们定义target是在一个左闭右闭，也就是[left, right]**

区间的定义这就决定了二分法的代码如何编写，因为定义target在[left, right]区间，所以有如下两点：

* while(left <= right) 要使用 <=，因为left == right 是有意义的，所以使用 <=
* if (nums[middle] > target) right要赋值为middle-1

```cpp
// 下面是一段伪代码形式来讲解

首先我们确定使用的二分法的方法为左闭右闭，所以我们应该确定四个值：
Left | Right | Middle | target

很明显 Left = 数组下标0	而Right为 NumSize(array)-1	Middle = (Left + Right) / 2
所以编写如下函数：
    
//	因为我们此处允许左闭右闭，所以可能存在[1, 1]，因此此处的left == right需要被考虑
while(left <= right)		
{
    // 此时我们考虑第一种情况，当我们的middle值是明确大于target值时，此时我们更新right值为middle-1
    if(Num(middle) > Num(target))
        Num(right) = Num(middle) - 1;
    // 此时我们考虑第二种情况，当我们的middle值是明确小于target值时，此时我们更新left值为middle+1
    else if(Num(middle) < Num(target))
        Num(left) = Num(middle) + 1;
    // 那么第三种情况也就是middle直接等于target，返回即可
    else
        return middle;
}
return -1;
```

```cpp
// 版本一
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1; // 定义target在左闭右闭的区间里，[left, right]
        while (left <= right) { // 当left==right，区间[left, right]依然有效，所以用 <=
            int middle = left + ((right - left) / 2);// 防止溢出 等同于(left + right)/2
            if (nums[middle] > target) {
                right = middle - 1; // target 在左区间，所以[left, middle - 1]
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，所以[middle + 1, right]
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
```



**<2>第二种写法，我们定义target是在一个左闭右开，也就是[left, right)**

根据左闭右开的方式，那么处理方式有如下两点：

* while(left < right),这里使用 <，因为left == right在区间 [left, right)是没有意义的
* if(Num(middle) > target) Num(right)更新为middle，因为当前的Num(middle)不等于Num(target)，去左区间继续寻找，而寻找区间是左闭右开区间，那么也就是说下一和查询区间不会去比较Num(middle)

```cpp
// 下面是一段伪代码形式来讲解

// 首先我们确定使用的二分法的方法为左闭右开，所以我们应该确定四个值：

Left | Right | Middle | target

上面的定义不变，但是函数主体需要有一些改动了
    
//	注意：我们此处允许左闭右开，而不需要考虑右区间末值，此时的right = Num(array),
while(left < right)		
{
    // 此时我们考虑第一种情况，当我们的middle值是明确大于target值时，此时我们更新right值为middle，因为此时的右区间为开区间，而此时的右区间不被考虑，所以Num(right) = Num(middle)
    if(Num(middle) > Num(target))
        Num(right) = Num(middle);
    // 此时我们考虑第二种情况，当我们的middle值是明确小于target值时，此时左区间为闭区间，我们更新left值为middle+1
    else if(Num(middle) < Num(target))
        Num(left) = Num(middle) + 1;
    // 那么第三种情况也就是middle直接等于target，返回即可
    else
        return middle;
}
return -1;
```

```cpp
// 版本二
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size(); // 定义target在左闭右开的区间里，即：[left, right)
        while (left < right) { // 因为left == right的时候，在[left, right)是无效的空间，所以使用 <
            int middle = left + ((right - left) >> 1);
            if (nums[middle] > target) {
                right = middle; // target 在左区间，在[left, middle)中
            } else if (nums[middle] < target) {
                left = middle + 1; // target 在右区间，在[middle + 1, right)中
            } else { // nums[middle] == target
                return middle; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};
```

上面对二分法的两种方式都已经做出解释，分别提供了伪代码和程序代码，其中有些知识点在下方做出解释：

`解析一：int middle = left + ((right - left) / 2);// 防止溢出 等同于(left + right)/2`

![image-20230215143931951](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302151439369.png)

**解答：对于上面这段代码做出这样修改的原因，主要就是为了防止溢出，如果在进行特别大的数值运算的时候，先进行加除操作很容易导致加法溢出最大限制，而首先进行减除操作则会大大降低风险。**

`解析二：int middle = left + ((right - left) >> 1);`

![image-20230215144334277](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302151443384.png)

**解答：`>>`是位运算的符号，`>>1`代表右移一位，这里我们记住尖号对准的方向就是位移方向。而对一个数右移一位，也就是代表除2操作。例如：11>>1，将11转成二进制为1011，而对二进制数向右移动1位则变成了0101,也就是代表5，其实也就代表除2操作。**

**此外还要补充一下，从效率上看，使用移位指令有更高的效率，因为`移位指令占2个机器周期，而乘除法指令占4个机器周期`。从硬件上看，移位对硬件更容易实现，所以会用移位，移一位就乘2,这种乘法当然考虑移位了。**

## 3.Leetcode27：移除元素

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/remove-element

#### （1）题目

**给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。**

**不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。**

**元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。**

**说明:**

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```cpp
// nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```


示例 1：

```cpp
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2]
解释：函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。你不需要考虑数组中超出新长度后面的元素。例如，函数返回的新长度为 2 ，而 nums = [2,2,3,3] 或 nums = [2,2,0,0]，也会被视作正确答案。
```

示例 2：

```cpp
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3]
解释：函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。注意这五个元素可为任意顺序。你不需要考虑数组中超出新长度后面的元素。
```

提示：

* 0 <= nums.length <= 100
* 0 <= nums[i] <= 50
* 0 <= val <= 100

#### （2）思路

**首先我们应该知道，在数组中，数组的元素在内存地址中是连续的，不能单独删除数组中的某个元素，只能覆盖。**

对此我们使用**暴力解法**

#### （3）暴力解法

解法：通过使用两层for循环，一层for循环遍历数组元素，一层for循环更新数组。

```cpp
class Solution{
public:
    int removeElement(vector<int>& nums, int val){
		int size = nums.size();
        for(int i = 0; i < size; i++){
            if(nums[i] == val){		// 发现需要移除的元素，就将数组集体向前移动一位
				for(int j = i + 1; j < size; j++){		
					nums[j - 1] = nums[j];
                }
                i--；			  // 由于下标i以后的数值都向前移动了一位，所以i也向前移动一位
                size--;			   // 相对应的数组大小-1
            }
        }
        
        return size;
    }
};
```

**说明：通过上面的程序可以看出暴力破解使用了两层for循环，也导致它的时间复杂度为O(n^2)，通过遍历的形式找出目标值，并将目标值后一位前移覆盖掉目标值的形式，从而达到移除数组元素的目的。**

#### （4）双指针法

除了暴力解法，双指针法也同样适用于此场景。

通过定义两个指针，一个slow指针和一个fast指针， **通过一个快指针和慢指针在一个for循环下完成两个for循环的工作。**

* fast指针：寻找新数组的元素，新数组就是不含有目标元素的数组
* slow指针：指向更新 新数组下标的位置

```cpp
// 时间复杂度：O(n)
// 空间复杂度：O(1)
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int slowIndex = 0;
        for (int fastIndex = 0; fastIndex < nums.size(); fastIndex++) {
            if (val != nums[fastIndex]) {		// 如果快指针指向的值不是目标值，则将快指针赋值给满指针，同时慢指针向前进一位
                nums[slowIndex++] = nums[fastIndex];
            }
            // 如果找到目标值，则快指针继续向前移动一位，而慢指针不进行移位操作，这就不等同于暴力破解的覆盖了，而是重新对下标位置进行分配
        }
        return slowIndex;
    }
};
```
