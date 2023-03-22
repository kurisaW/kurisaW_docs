## 今日任务

* 哈希表理论基础

* 242.有效的字母异位词

* 349.两个数组的交集

* 202.快乐数

* 1.两数之和

##  1.哈希表理论基础

#### （1）哈希表

哈希表（Hash table，国内也有一些书籍翻译为散列表）：是**根据关键码的值而直接访问的数据结构。**

最常见的哈希表例子就是数组。

哈希表中关键码就是数组的索引下标，然后通过下标直接访问数组中的元素，如下图所示：

![image-20230220102916613](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201029762.png)

那么哈希表一般适用于哪些场景呢？**一般哈希表都是用来快速判断一个元素是否出现在集合里。**

例如我们需要对指定商品信息进行查询，如果使用枚举的话，时间复杂度为O(n)，但是如果我们选择使用哈希表，只需要O(1)就可以做到。

我们只需要初始化时将所有的商品名称存入哈希表，在查询的时候直接通过索引就可以知道该商品是否存在了。

这里将商品列表映射到哈希表上就涉及到**哈希函数(Hash function)**。

#### （2）哈希函数

哈希函数，直接将商品的名称映射为哈希表上的索引，通过索引下标查询就可以知道该商品是否在售了。

哈希函数如下图所示，通过HashCode将名字转化为数值，一般HashCode是通过特定编码方式，可以将其他数据格式转化成不同的数值，这样就可以将商品名称映射到哈希表上的索引数字了。

![image-20230220105717329](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201057379.png)

此时我们需要额外考虑一件事，如果通过hashCode得到的数值大于哈希表的大小，该怎么办？

为了保证映射出来的索引数值都落在哈希表上，我们会再一次对数值进行一个取模操作，这样我们就保证了商品名称就一定可以映射到哈希表上了。

此时由于哈希表本质上就是一个数组，如果商品的数量大于哈希表的大小该怎么办？哈希函数就算分的再均匀，也避免不了有几个商品名称同时映射到哈希表同一索引下标的位置。

这时候就需要引入**哈希碰撞**了。

#### （3）哈希碰撞

如下图所示，商品1和商品3都映射到索引1的位置上，这个现象称之为**哈希碰撞**。

![image-20230220112851251](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201128301.png)

对于哈希碰撞一般有两种解决方法：**链地址法（拉链法）和线性探测法**

#### （4）链地址法（拉链法）

*这种方法的基本思想是将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。*

由于商品1和商品3再索引2的位置发生了冲突，并且发生冲突的元素都被存储在链表中，这样我们就可以通过索引找到商品1和商品3了

![image-20230220113841529](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201138570.png)

#### （5）线性探测法

使用线性探测法，一定要保证tableSize大于dataSize。我们需要依靠哈希表中的空位来解决碰撞问题。

例如索引1的位置已经存放了商品1的名称，那么当商品3再次进入索引1的位置就发生了冲突，当冲突发生后，就顺序查看表中的下一单元，直到找到一个空单元去存放商品3的名称。

![image-20230220114854813](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201148846.png)

此外对于哈希碰撞的常用解决方法还有**开放定址法、再哈希法、建立公共溢出区等等...**

#### （6）常见的三种哈希结构

当我们想使用哈希法来解决问题的时候，我们一般会选择如下三种数据结构：

* 数组
* set（集合）
* map（映射）

数组在前面已经简单介绍了，此处不再赘述，我们看下set（集合）：

**set（集合）**

在C++中，set和map分别提供以下三种数据结构，其底层优化以及优劣如下表所示：

| 集合               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| ------------------ | -------- | -------- | ---------------- | ------------ | -------- | -------- |
| std::set           | 红黑树   | 有序     | 否               | 否           | O(log n) | O(log n) |
| std::multiset      | 红黑树   | 有序     | 是               | 否           | O(logn)  | O(logn)  |
| std::unordered_set | 哈希表   | 无序     | 否               | 否           | O(1)     | O(1)     |

**map（映射）**

| 映射               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| ------------------ | -------- | -------- | ---------------- | ------------ | -------- | -------- |
| std::map           | 红黑树   | key有序  | key不可重复      | key不可修改  | O(logn)  | O(logn)  |
| std::multimap      | 红黑树   | key有序  | key可重复        | key不可修改  | O(log n) | O(log n) |
| std::unordered_map | 哈希表   | key无序  | key不可重复      | key不可修改  | O(1)     | O(1)     |

#### （7）总结

**当我们遇到这样一个场景：快速判断一个元素是否出现在集合里，就需要考虑哈希法。**

但是哈希法的缺点也显而易见的：**牺牲空间去换取时间**。

## 2.Leetcode242.有效的字母异位词

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/valid-anagram

#### （1）题目

**给定两个字符串 s 和 t ，编写一个函数来判断 t 是否是 s 的字母异位词。**

`注意：若 s 和 t 中每个字符出现的次数都相同，则称 s 和 t 互为字母异位词。`

示例 1:

```cpp
输入: s = "anagram", t = "nagaram"
输出: true
```

示例 2:

```cpp
输入: s = "rat", t = "car"
输出: false
```

**提示:**

* 1 <= s.length, t.length <= 5 * 104
* s 和 t 仅包含小写字母

**进阶: 如果输入字符串包含 unicode 字符怎么办？你能否调整你的解法来应对这种情况？**

#### （2）思路

前面我们讲了数组其实就是一个简单的哈希表，在本题中，我们可以定义一个数组，来记录字符串s中出现的字符次数。

由于都是字母，对应的也就是26个字符，所以这里我们设置的数组长度为26即可，并且初始化为0.

例如，我们对字符串s = "aee", t = '"eae"，我们观察动画：

![242.有效的字母异位词](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201724436.gif)

我们定义一个record的数组来记录字符串s里所有字符出现的次数。

需要将字符映射到数组也就是哈希表的下标上，字符a映射为下标0，字符z映射为下标25。

**在遍历字符串s的时候，只需要将s[i] = 'a'所在的元素作+1操作即可；同时在遍历字符串t的时候，对t中出现的字符映射哈希表索引上的数值再作-1操作；最后再检查一下，record数组如果有的元素不为0，那么就说明字符t和字符s一定不互为字母异位词，return false.**

**最后如果record数组所有元素都为0，则说明字符s和字符t是字母异位词，return true。**

**时间复杂度为O(n)，空间上因为定义的是一个常量大小的辅助数组，所以空间复杂度为O(1)**

#### （3）代码实现

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        int record[26] = {0};
        for (int i = 0; i < s.size(); i++) {
            // 并不需要记住字符a的ASCII，只要求出一个相对数值就可以了
            record[s[i] - 'a']++;
        }
        for (int i = 0; i < t.size(); i++) {
            record[t[i] - 'a']--;
        }
        for (int i = 0; i < 26; i++) {
            if (record[i] != 0) {
                // record数组如果有的元素不为零0，说明字符串s和t 一定是谁多了字符或者谁少了字符。
                return false;
            }
        }
        // record数组所有元素都为零0，说明字符串s和t是字母异位词
        return true;
    }
};
```

## 3.Leetcode349. 两个数组的交集

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/intersection-of-two-arrays

#### （1）题目

**给定两个数组 nums1 和 nums2 ，返回 它们的交集 。输出结果中的每个元素一定是 唯一 的。我们可以 不考虑输出结果的顺序 。**

示例 1：

```cpp
输入：nums1 = [1,2,2,1], nums2 = [2,2]
输出：[2]
```

示例 2：

```cpp
输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出：[9,4]
解释：[4,9] 也是可通过的
```

**提示：**

* 1 <= nums1.length, nums2.length <= 1000
* 0 <= nums1[i], nums2[i] <= 1000

#### （2）思路

在这道题目中，需要我们掌握哈希数据结构：unordered_set，如下图所示：

![image-20230220175039323](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201750667.png)

题目中特别声明：输出结果的每个元素一定是唯一的，也就是说输出的结果不用对重复出现的元素输出，同时可以不考虑输出结果的顺序。

之所以这里不使用数组，是因为题目限制了数组的大小，并且**如果哈希值比较少、特别分散、跨度大，使用数组就会造成空间的极大浪费。**

所以结合`std::unordered_set`的无序性，查询效率和增删效率都是O(1)的情况下，果断使用unordered_set

![image-20230220180154535](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302201801586.png)

#### （3）代码实现

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> result_set; // 存放结果，之所以用set是为了给结果集去重
        unordered_set<int> nums_set(nums1.begin(), nums1.end());// 定义哈希表存放结果
        for (int num : nums2) {
            // 发现nums2的元素 在nums_set里又出现过
            if (nums_set.find(num) != nums_set.end()) { // 在nums1中查找num（nums2）
                result_set.insert(num);// 如果发现与nums(nums2)的元素，向result_set插入该元素
            }
        }
        return vector<int>(result_set.begin(), result_set.end());
    }
};
```

当然这道题也可以使用数组的方式进行求解：

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> result_set; // 存放结果，之所以用set是为了给结果集去重
        int hash[1005] = {0}; // 默认数值为0
        for (int num : nums1) { // nums1中出现的字母在hash数组中做记录
            hash[num] = 1;
        }
        for (int num : nums2) { // nums2中出现话，result记录
            if (hash[num] == 1) {
                result_set.insert(num);
            }
        }
        return vector<int>(result_set.begin(), result_set.end());
    }
};
```

## 4.Leetcode202.快乐数

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/happy-number

#### （1）题目

**编写一个算法来判断一个数 n 是不是快乐数。**

**「快乐数」 **定义为：

* 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。

* 然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。

* 如果这个过程 结果为 1，那么这个数就是快乐数。

`如果 n 是 快乐数 就返回 true ；不是，则返回 false 。`

示例 1：

```cpp
输入：n = 19
输出：true
解释：
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

示例 2：

```cpp
输入：n = 2
输出：false
```

**提示：**

* 1 <= n <= 231 - 1

#### （2）思路

根据题目所给出的提示：**重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。**

简单解释下这句话，那么我们是不是可以理解为如果存在循环的数的话，那么这是不是就说明这个数不是开心数？

那么对于判断是否存在重复出现的数，我们选择使用哈希法，如果重复了的话就返回false，否则一直找到sum = 1为止。

#### （3）代码实现

```cpp
class Solution {
public:
    // 取数值各个位上的单数平方之和
    int getSum(int n) {
        int sum = 0;
        while (n) {
            sum += (n % 10) * (n % 10); // n每位数的平方和
            n /= 10;
        }
        return sum;
    }
    bool isHappy(int n) {
        unordered_set<int> set;
        while(1) {
            int sum = getSum(n);
            if (sum == 1) {
                return true;
            }
            // 如果这个sum曾经出现过，说明已经陷入了无限循环了，立刻return false
            if (set.find(sum) != set.end()) {
                return false;
            } else {
                set.insert(sum); // 记录第一次出现的数
            }
            n = sum;
        }
    }
};
```

## 5.Leetcode1.两数之和

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/two-sum

#### （1）题目

**给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。**

**你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。**

**你可以按任意顺序返回答案。**

示例 1：

```cpp
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

示例 2：

```cpp
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

示例 3：

```cpp
输入：nums = [3,3], target = 6
输出：[0,1]
```

**提示：**

* 2 <= nums.length <= 104
* -109 <= nums[i] <= 109
* -109 <= target <= 109
* 只会存在一个有效答案

**进阶：你可以想出一个时间复杂度小于 O(n2) 的算法吗？**

#### （2）思路

根据提示：只存在一个有效答案。所以我们这里可以选择**unordered_map** 

接下来我们明确两点：

* map用来做什么
* map中key和value分别表示什么

**拿target = 9举例子：map的目的是用来存取我们访问过的元素，当我们遍历数组的时候，需要我们记录之前遍历过哪些元素和对应的下标，首先先选定一个值（比如2），通过map查询是否存在与之满足条件的符合		因子（只能是7），此时如果在map中索引到该值，那么就得出我们想要的结果了；如果没有则继续选定下一个值，再去寻找与之相对应的符合因子。**

所以在**map中的存储结构为：{key：数据元素， value：数组元素对应的下标}**

![image-20230220210132750](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302202101163.png)

![image-20230220211643116](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302202116179.png)

#### （3）代码实现

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        std::unordered_map <int,int> map;
        for(int i = 0; i < nums.size(); i++) {
            // 遍历当前元素，并在map中寻找是否有匹配的key
            auto iter = map.find(target - nums[i]); 
            if(iter != map.end()) {
                return {iter->second, i};
            }
            // 如果没找到匹配对，就把访问过的元素和下标加入到map中
            map.insert(pair<int, int>(nums[i], i)); 
        }
        return {};
    }
};
```

