## 今日任务

* 454.四数相加II
* 383.赎金信
* 15.三数之和
* 18.四数之和

##  1.Leetcode454.四数相加II

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/4sum-ii

#### （1）题目

**给你四个整数数组 nums1、nums2、nums3 和 nums4 ，数组长度都是 n ，请你计算有多少个元组 (i, j, k, l) 能满足：**

* 0 <= i, j, k, l < n
* nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0


示例 1：

```cpp
输入：nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]
输出：2
解释：
两个元组如下：

1. (0, 0, 0, 1) -> nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) -> nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0
```

示例 2：

```cpp
输入：nums1 = [0], nums2 = [0], nums3 = [0], nums4 = [0]
输出：1
```


  **提示：**

* n == nums1.length
* n == nums2.length
* n == nums3.length
* n == nums4.length
* 1 <= n <= 200
* -228 <= nums1[i], nums2[i], nums3[i], nums4[i] <= 228

#### （2）思路

分析题意，题目中是四个独立数组，要求我们只要找到nums1[i] + nums2[j] + nums3[k] + nums4[l] = 0，同时这四个数组长度相同，并且在本题目中并没有限制数组元素出现的次数，也就是说只要满足四数组元素相加为0都可以作为一组解。

**解题步骤：**

* 首先定义一个unordered_map，key值为a、b两数之和，value值为a、b两数之和出现的次数。
* 遍历nums1和nums2数组，统计两个数组元素之和，和出现的次数，放到map中。
* 定义int变量count，用来统计nums1 + nums2 + nums3 + nums4 = 0出现的次数。
* 在遍历nums3和nums4数组，找到如果0 - (nums3 + nums4)在map中出现过的话，就用count把map中key对应的value也就是出现次数统计出来。
* 最后再返回统计值count就可以了。

#### （3）代码演示

```cpp
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int, int> umap; // key:a+b的数值，value:a+b数值出现的次数
        for(int a : nums1){
            for(int b : nums2){
                umap[a + b]++; // 遍历nums1和nums2数组，统计两个数组元素之和，和出现的次数，放到map中
            }
        }
        int count = 0; // 统计nums1 + nums2 + nums3 + nums4 = 0出现的次数
        // 在遍历nums3和nums4数组，找到如果 0-(nums3 + nums4) 在map中出现过的话，就把map中key对应的value也就是出现次数统计出来。
        for(int c : nums3){
            for(int d : nums4){
                if(umap.find(0 - (c + d)) != umap.end()){
                    count += umap[0 - (c + d)];// 此处 umap[key]可以直接访问满足key的value值
                }
            }
        }
        return count;
    }
};
```

## 2.Leetcode383.赎金信

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/ransom-note

#### （1）题目

**给你两个字符串：ransomNote 和 magazine ，判断 ransomNote 能不能由 magazine 里面的字符构成。**

**如果可以，返回 true ；否则返回 false 。**

**magazine 中的每个字符只能在 ransomNote 中使用一次。**

示例 1：

```cpp
输入：ransomNote = "a", magazine = "b"
输出：false
```

示例 2：

```cpp
输入：ransomNote = "aa", magazine = "ab"
输出：false
```

示例 3：

```cpp
输入：ransomNote = "aa", magazine = "aab"
输出：true
```

**提示：**

* 1 <= ransomNote.length, magazine.length <= 105
* ransomNote 和 magazine 由小写英文字母组成

#### （2）思路

首先锁定提示：两个字符串均由小写英文字母组成，并且magazine 中的每个字符只能在 ransomNote 中使用一次，这就跟战争时期的加密信件差不多一个意思，密信的内容在杂志中都可以找到。

对于这道题的解法，使用暴力解法，数组、map都可以实现，我们这里主要讲解暴力解法和数组，至于为什么不使用map，根据carl大神的说法就是**这道题中使用map，空间消耗要比数组大一些，因为map需要维护红黑树或哈希表，并且还要做哈希函数，是很费时的**，所以数组和map果断选择map。

暴力解法就是简单两层for循环，只要找到两个字符串中存在相同的字符就将ransomNote中对应的字符删去，直至最后ransomNote中无元素为止。

使用哈希解法的话，前面的学习我们也已经知道，数组也是一种简单的哈希表，通过定义一个record[26]的数组(因为条件说明仅为小写字母)，首先遍历所有magazine中的元素对应record数组中的索引，出现相同的key值就将该value加一

#### （3）暴力解法

```cpp
// 时间复杂度: O(n^2)
// 空间复杂度：O(1)
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        for (int i = 0; i < magazine.length(); i++) {
            for (int j = 0; j < ransomNote.length(); j++) {
                // 在ransomNote中找到和magazine相同的字符
                if (magazine[i] == ransomNote[j]) {
                    ransomNote.erase(ransomNote.begin() + j); // ransomNote删除这个字符
                    break;
                }
            }
        }
        // 如果ransomNote为空，则说明magazine的字符可以组成ransomNote
        if (ransomNote.length() == 0) {
            return true;
        }
        return false;
    }
};
```

#### （4）哈希解法

```cpp
// 时间复杂度: O(n)
// 空间复杂度：O(1)
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        int record[26] = {0};
        //add
        if (ransomNote.size() > magazine.size()) {
            return false;
        }
        for (int i = 0; i < magazine.length(); i++) {
            // 通过recode数据记录 magazine里各个字符出现次数
            record[magazine[i]-'a'] ++;
        }
        for (int j = 0; j < ransomNote.length(); j++) {
            // 遍历ransomNote，在record里对应的字符个数做--操作
            record[ransomNote[j]-'a']--;
            // 如果小于零说明ransomNote里出现的字符，magazine没有
            if(record[ransomNote[j]-'a'] < 0) {
                return false;
            }
        }
        return true;
    }
};
```

## 3.Leetcode15.三数之和

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/3sum

#### （1）题目

**给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。**

`注意：答案中不可以包含重复的三元组。` 

示例 1：

```cpp
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
```

示例 2：

```cpp
输入：nums = [0,1,1]
输出：[]
解释：唯一可能的三元组和不为 0 。
```

示例 3：

```cpp
输入：nums = [0,0,0]
输出：[[0,0,0]]
解释：唯一可能的三元组和为 0 。
```

**提示：**

* 3 <= nums.length <= 3000
* -105 <= nums[i] <= 105

#### （2）思路

这道题和Leetcode454.四数相加II有点相似，不过在本题目中，特别限制了**答案中不可包含重复的三元组**。所以解题思路不能一概而论，同样可以使用**哈希解法**，但是现在目前最大的问题就是对三元组的去重工作，哈希解法的细节需要考虑的太多了，这里还是不建议使用，博主已经是晕了，当然大佬们可以尝试着理清关系。

那么另外一种解题思路就是使用**双指针法**。拿数组nums举例，首先将数组排序，元素i从下标0开始，同时设下一个下标 left 在 i + 1 的位置上，下标right在数组末尾，如下图所示：

![image-20230221204834894](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212048396.png)

我们的目的是在数组nums中找到a、b、c，那么对于上图也就是a = nums[i], b = nums[left], c = nums[right]。由于我们提前排好序，所以此时abc相加会出现三种结果：

* nums[i] + nums[left] + nums[right] > 0 ：此时说明三数之和大了，需要我们将right下标向左移动
* nums[i] + nums[left] + nums[right] = 0 ：返回结果
* nums[i] + nums[left] + nums[right] < 0 ：说明此时三数之和小了，需要我们将left下标向右移动

此外，我们还需要解决去重的问题：

**<1>对a去重：**

按照一贯的理解我们可能是下面这种做法：

```cpp
            if (i > 0 && nums[i] == nums[i + 1]) { //三元组元素a去重
                continue;
            }
```

但是我们看这种情况：如果我们这里选择上面的去重做法，当遍历第一个-1的时候，此时nums[i + 1]也就是-1，那么这组数据直接就被pass了，根据题意：**返回不能有重复的三元组，但是三元组内的元素是可以重复的**，如果按照上面的写法，那么我们很可能漏掉一组解。

所以应该是下面这段代码这样：

```cpp
            if (i > 0 && nums[i] == nums[i - 1]) { //三元组元素a去重
                continue;
            }
```

![image-20230221205206723](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212052779.png)

**<2>b与c的去重：**

当我们收割到符合条件的结果的时候，如果不进行去重，可能会出现多个相同的结果，所以我们left和right会造成的相同结果进行去重，去重之后将两个指针再移动到一位进行比较。

![image-20230221211026547](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212110982.png)

```cpp
                    // 去重逻辑应该放在找到一个三元组之后，对b 和 c去重
                    while (right > left && nums[right] == nums[right - 1]) right--;
                    while (right > left && nums[left] == nums[left + 1]) left++;
```

![image-20230221211127566](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212111616.png)

```cpp
                    // 找到答案时，双指针同时收缩
                    right--;
                    left++;
```

![image-20230221211223973](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212112023.png)

#### （3）哈希解法*

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        // 找出a + b + c = 0
        // a = nums[i], b = nums[j], c = -(a + b)
        for (int i = 0; i < nums.size(); i++) {
            // 排序之后如果第一个元素已经大于零，那么不可能凑成三元组
            if (nums[i] > 0) {
                break;
            }
            if (i > 0 && nums[i] == nums[i - 1]) { //三元组元素a去重
                continue;
            }
            unordered_set<int> set;
            for (int j = i + 1; j < nums.size(); j++) {
                if (j > i + 2
                        && nums[j] == nums[j-1]
                        && nums[j-1] == nums[j-2]) { // 三元组元素b去重
                    continue;
                }
                int c = 0 - (nums[i] + nums[j]);
                if (set.find(c) != set.end()) {
                    result.push_back({nums[i], nums[j], c});
                    set.erase(c);// 三元组元素c去重
                } else {
                    set.insert(nums[j]);
                }
            }
        }
        return result;
    }
};
```

#### （4）双指针法

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        // 找出a + b + c = 0
        // a = nums[i], b = nums[left], c = nums[right]
        for (int i = 0; i < nums.size(); i++) {
            // 排序之后如果第一个元素已经大于零，那么无论如何组合都不可能凑成三元组，直接返回结果就可以了
            if (nums[i] > 0) {
                return result;
            }
            // 错误去重a方法，将会漏掉-1,-1,2 这种情况
            /*
            if (nums[i] == nums[i + 1]) {
                continue;
            }
            */
            // 正确去重a方法
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            int left = i + 1;
            int right = nums.size() - 1;
            while (right > left) {
                // 去重复逻辑如果放在这里，0，0，0 的情况，可能直接导致 right<=left 了，从而漏掉了 0,0,0 这种三元组
                /*
                while (right > left && nums[right] == nums[right - 1]) right--;
                while (right > left && nums[left] == nums[left + 1]) left++;
                */
                if (nums[i] + nums[left] + nums[right] > 0) right--;
                else if (nums[i] + nums[left] + nums[right] < 0) left++;
                else {
                    result.push_back(vector<int>{nums[i], nums[left], nums[right]});
                    // 去重逻辑应该放在找到一个三元组之后，对b 和 c去重
                    while (right > left && nums[right] == nums[right - 1]) right--;
                    while (right > left && nums[left] == nums[left + 1]) left++;

                    // 找到答案时，双指针同时收缩
                    right--;
                    left++;
                }
            }

        }
        return result;
    }
};
```

## 4.Leetcode18.四数之和

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/4sum

#### （1）题目

**给你一个由 n 个整数组成的数组 nums ，和一个目标值 target 。请你找出并返回满足下述全部条件且不重复的四元组 [nums[a], nums[b], nums[c], nums[d]] （若两个四元组元素一一对应，则认为两个四元组重复）：**

* 0 <= a, b, c, d < n

* a、b、c 和 d 互不相同

* nums[a] + nums[b] + nums[c] + nums[d] == target

**你可以按 任意顺序 返回答案 。**

示例 1：

```cpp
输入：nums = [1,0,-1,0,-2,2], target = 0
输出：[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]
```

示例 2：

```cpp
输入：nums = [2,2,2,2,2], target = 8
输出：[[2,2,2,2]]
```

**提示：**

* 1 <= nums.length <= 200
* -109 <= nums[i] <= 109
* -109 <= target <= 109

#### （2）思路

这道题算的上是Leetcode15.三数之和的一个延伸，四数之和其实是在三数之和的基础上再外层再套了一层循环。

但是有些许细节需要我们认真对待：

* 在三数之和中，target已经是定值0，但是在四数之和中，target可以是任意值，所以在某些地方我们可以对数组本身做一个剪枝操作。
* 在三数之和中的双指针解法是通过一层for循环nums[i]为确定值，然后循环内设置left和right下标作为双指针；而在四数之和中，我们要做的是`nums[k] + nums[i] + nums[left] + nums[right] == target`的所有可解集合，所以我们的解决方法是两层for循环`nums[k] + nums[i]`为确定值，双指针法依然是left和right作为下标。
* 三数之和的时间复杂度是O(n^2)，四数之和的时间复杂度是O(n^3) 。

![image-20230221214611511](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302212146921.png)

#### （3）代码实现

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());
        for (int k = 0; k < nums.size(); k++) {
            // 剪枝处理
            if (nums[k] > target && nums[k] >= 0) {
            	break; // 这里使用break，统一通过最后的return返回
            }
            // 对nums[k]去重
            if (k > 0 && nums[k] == nums[k - 1]) {
                continue;
            }
            for (int i = k + 1; i < nums.size(); i++) {
                // 2级剪枝处理
                if (nums[k] + nums[i] > target && nums[k] + nums[i] >= 0) {
                    break;
                }

                // 对nums[i]去重
                if (i > k + 1 && nums[i] == nums[i - 1]) {
                    continue;
                }
                int left = i + 1;
                int right = nums.size() - 1;
                while (right > left) {
                    // nums[k] + nums[i] + nums[left] + nums[right] > target 会溢出
                    if ((long) nums[k] + nums[i] + nums[left] + nums[right] > target) {
                        right--;
                    // nums[k] + nums[i] + nums[left] + nums[right] < target 会溢出
                    } else if ((long) nums[k] + nums[i] + nums[left] + nums[right]  < target) {
                        left++;
                    } else {
                        result.push_back(vector<int>{nums[k], nums[i], nums[left], nums[right]});
                        // 对nums[left]和nums[right]去重
                        while (right > left && nums[right] == nums[right - 1]) right--;
                        while (right > left && nums[left] == nums[left + 1]) left++;

                        // 找到答案时，双指针同时收缩
                        right--;
                        left++;
                    }
                }

            }
        }
        return result;
    }
};
```
