## 今日任务

* 977.有序数列的平方

* 209.长度最小的子数组

* 59.螺旋矩阵II

* 总结

##  1.Leetcode977：有序数列的平方

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/squares-of-a-sorted-array

#### （1）题目

**给你一个按 非递减顺序 排序的整数数组 nums，返回 每个数字的平方 组成的新数组，要求也按 非递减顺序 排序。 **

示例 1：

```cpp
输入：nums = [-4,-1,0,3,10]
输出：[0,1,9,16,100]
解释：平方后，数组变为 [16,1,0,9,100]
排序后，数组变为 [0,1,9,16,100]
```

示例 2：

```cpp
输入：nums = [-7,-3,2,3,11]
输出：[4,9,9,49,121]
```

**提示：**

* 1 <= nums.length <= 104
* -104 <= nums[i] <= 104
* nums 已按 非递减顺序 排序

**进阶：**

请你设计时间复杂度为 O(n) 的算法解决本问题

#### （2）思路

最开始的一个想法，就是首先对每个数进行平方，然后再对新数组进行排序。

#### （3）暴力排序

有了昨天的经验，我们可以直接使用暴力排序的方式进行编程：

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        for(int i = 0; i < nums.size(); i++){
            // nums[i] = pow(abs(nums[i]),2);
            nums[i] *= nums[i];
        }
        sort(nums.begin(),nums.end());
        return nums;
    }
};
```

**说明：**

* pow(a,b)：a作为目标值，b作为指数，是用作指数运算，例如pow(2，2)--->2^2=4;
* abs(n)：对n求绝对值

**解答：上面的求平方数我用了两种方式求解，但是很明显可以看出注释的那一段代码明显执行的时间复杂度更高，也就是O(nlogn+1+nlog2n)，而另外的一种方式的时间复杂度则是O(n+nlogn)**

**在这里也有大佬提出：二分法的log2就直接logn就可以，平衡二叉树 排序都直接nlogn就行 **

#### （4）双指针法

**根据数组最大值通过平方之后，不是最大值就是最小值，我们可以考虑使用双指针法，i指向起始位置，j指向终止位置。**

* 定义一个新数组result，和数组A一样的大小，让`K指向result数组终止位置`
* 如果A[i] *A[i] < A[j] * A[j]，那么result[k--] = A[j] * A[j];
* 如果A[i] *A[i] > A[j] * A[j]，那么result[k--] = A[i] * A[i];

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> result(nums.size(),0);
        int k = nums.size() - 1;
        for(int i = 0,j = nums.size() - 1; i <= j; ){
            if(nums[i] * nums[i] < nums[j] * nums[j]){
                result[k--] = nums[j] * nums[j];
                j--;
            }
            else{
                result[k--] = nums[i] * nums[i];
                i++;
            }
        }
        return result;
    }
};
```

通过双指针法求解有序数列的平方，此时的时间复杂度为O(n)，相比较暴力排序这个还是更加推荐！

## 2.Leetcode209：长度最小的子数组

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/minimum-size-subarray-sum

#### （1）题目

**给定一个含有 n 个正整数的数组和一个正整数 target 。**

**找出该数组中满足其和 ≥ target 的长度最小的 连续子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。**

示例 1：

```cpp
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

示例 2：

```cpp
输入：target = 4, nums = [1,4,4]
输出：1
```

示例 3：

```cpp
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

**提示：**

* 1 <= target <= 109
* 1 <= nums.length <= 105
* 1 <= nums[i] <= 105

**进阶：**

如果你已经实现 O(n) 时间复杂度的解法, 请尝试设计一个 O(n log(n)) 时间复杂度的解法。

#### （2）思路

首先分析题意，最明显的就是要求是`连续子数组`，然后就是要求这个子数组长度最小，遇到这个问题，我们想到的就是首先分出若干个有效子数组（要求是连续的），然后对这些子数组的长度进行筛选，留下长度最小的返回该数组长度。

#### （3）暴力排序

对这道题暴力排序的解法是通过使用两个for循环，然后不断寻找符合条件的子序列，具体判断时间复杂度是O(n^2)。

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int result = INT32_MAX; // 最终的结果
        int sum = 0; // 子序列的数值之和
        int subLength = 0; // 子序列的长度
        for (int i = 0; i < nums.size(); i++) { // 设置子序列起点为i
            sum = 0;
            for (int j = i; j < nums.size(); j++) { // 设置子序列终止位置为j
                sum += nums[j];
                if (sum >= target) { // 一旦发现子序列和超过了s，更新result
                    subLength = j - i + 1; // 取子序列的长度
                    result = result < subLength ? result : subLength;
                    break; // 因为我们是找符合条件最短的子序列，所以一旦符合条件就break
                }
            }
        }
        // 如果result没有被赋值的话，就返回0，说明没有符合条件的子序列
        return result == INT32_MAX ? 0 : result;
    }
};
```

* 时间复杂度：O(n^2)
* 空间复杂度：O(1)

**对于这部分的暴力排序其实有些还没看懂，先在这插个眼，并且根据力扣的测试，该方法已经超时，应该是不建议使用。**

#### （4）滑动窗口

**所谓滑动窗口，就是不断的调节子序列的起始位置和终止位置，从而得出我们想要的结果。**

那怎么理解滑动窗口呢，其实滑动窗口的做法也可以作为双指针法的一种，通过动态变换滑动窗口的起始和终止位置构成的滑动区域，依次遍历可能出现的子数组。

这里放上Carl大神的一张图，方便大家理解：



![209.长度最小的子数组](https://code-thinking.cdn.bcebos.com/gifs/209.长度最小的子数组.gif)

那么最重要的两点来了：

* 如何确定移动窗口的起始位置
* 如何确定移动窗口的结束位置

**解答如下：**

* 窗口的起始位置如何移动：如果当前窗口的值大于target，说明已经找到一种满足情况的子数组了，那么此时应该将窗口向前移动
* 窗口的结束位置如何移动：窗口的结束位置就是遍历数组的指针，也就是给定数组下标的最大值

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int result = INT32_MAX;
        int sum = 0; // 滑动窗口数值之和
        int i = 0; // 滑动窗口起始位置
        int subLength = 0; // 滑动窗口的长度
        for (int j = 0; j < nums.size(); j++) {
            sum += nums[j];
            // 注意这里使用while，每次更新 i（起始位置），并不断比较子序列是否符合条件
            while (sum >= target) {
                subLength = (j - i + 1); // 取子序列的长度
                result = result < subLength ? result : subLength;
                sum -= nums[i++]; // 这里体现出滑动窗口的精髓之处，不断变更i（子序列的起始位置）
            }
        }
        // 如果result没有被赋值的话，就返回0，说明没有符合条件的子序列
        return result == INT32_MAX ? 0 : result;
    }
};
```

在这里的话也才发现滑动窗口这个算法精妙所在，通过不断变更一个窗口的位置，将算法的复杂度明显优化，而且相比较暴力排序，滑动窗口也只用了一个for循环和一个while循环，从而将算法复杂度降为O(n)

## 3.Leetcode59：螺旋矩阵II

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/spiral-matrix-ii

#### （1）题目

**给你一个正整数 n ，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的 n x n 正方形矩阵 matrix 。**

示例 1：

```cpp
输入：n = 3
输出：[[1,2,3],[8,9,4],[7,6,5]]
```

示例 2：

```cpp
输入：n = 1
输出：[[1]]
```

**提示：**

* 1 <= n <= 20

#### （2）思路

在这里悉心听取Carl大神的教诲，每次遇到二分法一定要坚持**循环不变量原则**。

那么我们在模拟顺时针画矩阵时，遵循以下规则：

* 填充上行从左往右
* 填充右列从上往下
* 填充下行从右往左
* 填充左列从下往上

也就是如下图所示，好好理解一下！

![img](https://assets.leetcode.com/uploads/2020/11/13/spiraln.jpg)

回到题目，对于这种螺旋矩阵，我们首先要明确的坚持**循环不变量原则**，要么选择左闭右闭，要么选择左闭右开，选择好一种处理方式就贯彻到底，不要再做改变了。

**这里我们选择左闭右开，首先还是看到上面的螺旋矩阵图，我们分别将3X3矩阵内的所有元素切割为9个部分，解决螺旋矩阵问题，最重要就是确定外围的四个点，即图中的`1、3、5、7`，前面我们说我们遵循左闭右开规则，其实意思就是对左节点进行处理，而右节点暂不处理，而等待下一次处理时将第一次的右节点作为第二次的左节点，这样就是我们所说的左闭右开原则。**

#### （3）二分法求解

直接看代码部分：

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> res(n, vector<int>(n, 0)); // 使用vector定义一个二维数组
        int startx = 0, starty = 0; // 定义每循环一个圈的起始位置
        int loop = n / 2; // 每个圈循环几次，例如n为奇数3，那么loop = 1 只是循环一圈，矩阵中间的值需要单独处理
        int mid = n / 2; // 矩阵中间的位置，例如：n为3， 中间的位置就是(1，1)，n为5，中间位置为(2, 2)
        int count = 1; // 用来给矩阵中每一个空格赋值
        int offset = 1; // 需要控制每一条边遍历的长度，每次循环右边界收缩一位
        int i,j;
        while (loop --) {
            i = startx;
            j = starty;

            // 下面开始的四个for就是模拟转了一圈
            // 模拟填充上行从左到右(左闭右开)
            for (j = starty; j < n - offset; j++) {
                res[startx][j] = count++;
            }
            // 模拟填充右列从上到下(左闭右开)
            for (i = startx; i < n - offset; i++) {
                res[i][j] = count++;
            }
            // 模拟填充下行从右到左(左闭右开)
            for (; j > starty; j--) {
                res[i][j] = count++;
            }
            // 模拟填充左列从下到上(左闭右开)
            for (; i > startx; i--) {
                res[i][j] = count++;
            }

            // 第二圈开始的时候，起始位置要各自加1， 例如：第一圈起始位置是(0, 0)，第二圈起始位置是(1, 1)
            startx++;
            starty++;

            // offset 控制每一圈里每一条边遍历的长度
            offset += 1;
        }

        // 如果n为奇数的话，需要单独给矩阵最中间的位置赋值
        if (n % 2) {
            res[mid][mid] = count;
        }
        return res;
    }
};
```

## 4.总结

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/%E6%95%B0%E7%BB%84%E6%80%BB%E7%BB%93.png)

图片来源： [代码随想录知识星球](https://programmercarl.com/other/kstar.html)成员：[海螺人](https://wx.zsxq.com/dweb2/index/footprint/844412858822412)

