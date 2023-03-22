## 今日任务

* 344.反转字符串
* 541.反转字符串II
* 剑指Offer 05.替换空格
* 151.反转字符串里的单词
* 剑指Offer58-II.左旋转字符串

##  1.Leetcode344.反转字符串

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/reverse-string

#### （1）题目

**编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 s 的形式给出。**

**不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。**

示例 1：

```cpp
输入：s = ["h","e","l","l","o"]
输出：["o","l","l","e","h"]
```

示例 2：

```cpp
输入：s = ["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]
```

**提示：**

* 1 <= s.length <= 105
* s[i] 都是 ASCII 码表中的可打印字符

#### （2）思路

看到这道题的第一反应就是双指针法，不得不说，双指针法对这种排序问题真的YYDS，相比于我们前面在学习链表的时候所使用到的双指针法，字符串的反转其实比起链表还要简单一些。在内存中链表可以是无序的，但是字符串本质上也可以说的上是一种数组，所以元素在内存中是连续分布的。

那么对于这道题我们选择使用双指针法：分别定义指针i位于字符串下标0的位置和指针j位于字符串末尾的位置，通过互换元素的方式来完成字符串的反转。

![image-20230222111753143](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302221117397.png)

对应的部分C++代码：

```cpp
void reverseString(vector<char>& s){
	for(int i = 0, j = s.size() - 1; i < s.size() / 2; i++,j--){
		swap(s[i], s[j]);
    }
}
```

#### （3）代码演示

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        for (int i = 0, j = s.size() - 1; i < s.size()/2; i++, j--) {
            swap(s[i],s[j]);
        }
    }
};
```

## 2.Leetcode541.反转字符串II

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/reverse-string-ii

#### （1）题目

**给定一个字符串 s 和一个整数 k，从字符串开头算起，每计数至 2k 个字符，就反转这 2k 字符中的前 k 个字符。**

* 如果剩余字符少于 k 个，则将剩余字符全部反转。
* 如果剩余字符小于 2k 但大于或等于 k 个，则反转前 k 个字符，其余字符保持原样。


示例 1：

```cpp
输入：s = "abcdefg", k = 2
输出："bacdfeg"
```

示例 2：

```cpp
输入：s = "abcd", k = 2
输出："bacd"
```

**提示：**

* 1 <= s.length <= 104
* s 仅由小写英文组成
* 1 <= k <= 104

#### （2）思路

我们在遍历字符串的过程中，只要让 i += (2 * k)，i 每次移动 2 * k 就可以了，然后判断是否需要有反转的区间。

该题主要需要解决两个问题：

* 每计数至 2k 个字符，就反转这 2k 字符中的前 k 个字符
* 对于剩余字符如果不足k个则全部反转；如果在k ~ 2k之间，则反转剩余字符的前k个字符

具体详细的解题步骤请看下图及代码：

![image-20230222121250753](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302221212235.png)

#### （3）代码演示

```cpp
class Solution {
public:
    // 此处为用户设计的字符串反转，其实也就是Leetcode344题，当然我们也可以使用C++的reverse()函数
    void reverse(string& s, int start, int end) {
        for (int i = start, j = end; i < j; i++, j--) {
            swap(s[i], s[j]);
        }
    }
    string reverseStr(string s, int k) {
        for (int i = 0; i < s.size(); i += (2 * k)) {
            // 1. 每隔 2k 个字符的前 k 个字符进行反转
            // 2. 剩余字符小于 2k 但大于或等于 k 个，则反转前 k 个字符
            if (i + k <= s.size()) {
                reverse(s, i, i + k - 1);
                continue;
            }
            // 3. 剩余字符少于 k 个，则将剩余字符全部反转。
            reverse(s, i, s.size() - 1);
        }
        return s;
    }
};
```

**reverse()**

* reverse函数功能是逆序（或反转），多用于字符串、数组、容器。头文件是#include <algorithm>
* reverse函数用于反转在[first,last)范围内的顺序（包括first指向的元素，不包括last指向的元素），reverse函数无返回值

##  3.剑指Offer 05.替换空格

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/ti-huan-kong-ge-lcof

#### （1）题目

**请实现一个函数，把字符串 s 中的每个空格替换成"%20"。**

示例 1：

```cpp
输入：s = "We are happy."
输出："We%20are%20happy."
```

**限制：**

* 0 <= s 的长度 <= 10000

#### （2）思路

对这道题的求解，主要分三个步骤：

* 首先扩充数组到每个空格替换成"%20"之后的大小
* 然后从后往前替换空格，也就是双指针法，如下图动画所示（来源：代码随想录）
* i指向新长度的末尾，j指向旧长度的末尾

![替换空格](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302221307960.gif)

而这里也有一个小技巧：**遇到很多数组填充类的问题，都可以先预留给数组扩容带填充后的大小，然后再从后往前操作。**

这样做的好处：

* 不用申请新数组
* 从后往前填充元素，避免了从前往后填充元素时都要讲添加元素之后的所有元素向后移动的问题。

#### （3）代码演示

```cpp
class Solution {
public:
    string replaceSpace(string s) {
        int count = 0; // 统计空格的个数
        int sOldSize = s.size();
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == ' ') {
                count++;
            }
        }
        // 扩充字符串s的大小，也就是每个空格替换成"%20"之后的大小
        s.resize(s.size() + count * 2); // 之所以count * 2而不是 * 3，是因为之前的空格抵掉一个了
        int sNewSize = s.size();
        // 从后先前将空格替换为"%20"
        for (int i = sNewSize - 1, j = sOldSize - 1; j < i; i--, j--) {
            if (s[j] != ' ') {
                s[i] = s[j];
            } else {
                s[i] = '0';
                s[i - 1] = '2';
                s[i - 2] = '%';
                i -= 2;
            }
        }
        return s;
    }
};
```

**resize()**

* 既分配了空间，也创建了对象。
* 这里空间就是capacity（指容器在分配新的存储空间之前能存储的元素总数），对象就是容器中的元素。

## 4.Leetcode151.反转字符串里的单词

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/reverse-words-in-a-string

#### （1）题目

**给你一个字符串 s ，请你反转字符串中 单词 的顺序。**

**单词 是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的 单词 分隔开。**

**返回 单词 顺序颠倒且 单词 之间用单个空格连接的结果字符串。**

`注意：输入字符串 s中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。`

示例 1：

```cpp
输入：s = "the sky is blue"
输出："blue is sky the"
```

示例 2：

```cpp
输入：s = "  hello world  "
输出："world hello"
解释：反转后的字符串中不能存在前导空格和尾随空格。
```

示例 3：

```cpp
输入：s = "a good   example"
输出："example good a"
解释：如果两个单词间有多余的空格，反转后的字符串需要将单词间的空格减少到仅有一个。
```

**提示：**

* 1 <= s.length <= 104
* s 包含英文大小写字母、数字和空格 ' '
* s 中 至少存在一个 单词

**进阶：如果字符串在你使用的编程语言中是一种可变数据类型，请尝试使用 O(1) 额外空间复杂度的 原地 解法。**

#### （2）思路

对于这样一道题，我们**不使用辅助空间，空间复杂度要求为O(1)**

所以对此我们有这样一种解法：使用整体反转加局部反转的方式解决

* 首先移除掉多余的空格
* 将整个字符串反转
* 再将每个单词反转

演示如下：

![image-20230222154346894](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302221543326.png)

前面讲了整体的一个逻辑思维方式，那么代码怎么实现呢，首先我们看**移除多余空格**：我们的做法是**通过快慢指针的方式来去除所有空格并且在相邻单词之间添加空格**。

```cpp
void removeExtraSpaces(string& s) {//去除所有空格并在相邻单词之间添加空格, 快慢指针。
    int slow = 0;  
    for (int i = 0; i < s.size(); ++i) { //
        if (s[i] != ' ') { //遇到非空格就处理，即删除所有空格。
            if (slow != 0) s[slow++] = ' '; //手动控制空格，给单词之间添加空格。slow != 0说明不是第一个单词，需要在单词前添加空格。
            while (i < s.size() && s[i] != ' ') { //补上该单词，遇到空格说明单词结束。
                s[slow++] = s[i++];
            }
        }
    }
    s.resize(slow); //slow的大小即为去除多余空格后的大小。
}
```

此外就是字符串反转的问题，其代码实现逻辑如下：

```cpp
// 反转字符串s中左闭右闭的区间[start, end]
void reverse(string& s, int start, int end) {
    for (int i = start, j = end; i < j; i++, j--) {
        swap(s[i], s[j]);
    }
}
```

#### （3）代码演示

```cpp
class Solution {
public:
    void reverse(string& s, int start, int end){ //翻转，区间写法：左闭右闭 []
        for (int i = start, j = end; i < j; i++, j--) {
            swap(s[i], s[j]);
        }
    }

    void removeExtraSpaces(string& s) {//去除所有空格并在相邻单词之间添加空格, 快慢指针。
        int slow = 0;   //整体思想参考https://programmercarl.com/0027.移除元素.html
        for (int i = 0; i < s.size(); ++i) { //
            if (s[i] != ' ') { //遇到非空格就处理，即删除所有空格。
                if (slow != 0) s[slow++] = ' '; //手动控制空格，给单词之间添加空格。slow != 0说明不是第一个单词，需要在单词前添加空格。
                while (i < s.size() && s[i] != ' ') { //补上该单词，遇到空格说明单词结束。
                    s[slow++] = s[i++];
                }
            }
        }
        s.resize(slow); //slow的大小即为去除多余空格后的大小。
    }

    string reverseWords(string s) {
        removeExtraSpaces(s); //去除多余空格，保证单词之间之只有一个空格，且字符串首尾没空格。
        reverse(s, 0, s.size() - 1);// 反转字符串
        int start = 0; //removeExtraSpaces后保证第一个单词的开始下标一定是0。
        for (int i = 0; i <= s.size(); ++i) {
            if (i == s.size() || s[i] == ' ') { //到达空格或者串尾，说明一个单词结束。进行翻转。
                reverse(s, start, i - 1); //翻转，注意是左闭右闭 []的翻转。
                start = i + 1; //更新下一个单词的开始下标start
            }
        }
        return s;
    }
};
```

## 5.剑指Offer58-II.左旋转字符串

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof

#### （1）题目

**字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。**

示例 1：

```cpp
输入: s = "abcdefg", k = 2
输出: "cdefgab"
```

示例 2：

```cpp
输入: s = "lrloseumgh", k = 6
输出: "umghlrlose"
```

**限制：**

* 1 <= k < s.length <= 10000

#### （2）思路

在本题目中，carl老师继续升级难度：**要求不能申请额外空间，只能在本串上操作**

但是对于上面Leetcode151题，我们依旧可以有借鉴之法，具体步骤如下：

* 反转区间为前n的子串
* 反转区间为n到末尾的子串
* 反转整个字符串

![image-20230222162131830](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302221621199.png)

这样一来，整体的代码逻辑就特别简单啦！

#### （3）代码演示

```cpp
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        reverse(s.begin(), s.begin() + n);
        reverse(s.begin() + n, s.end());
        reverse(s.begin(), s.end());
        return s;
    }
};
```

没想到最后一个代码的实现这么简单哈哈哈，在经历**Leetcode151.反转字符串里的单词**这道题的洗礼后是不是有种小巫见大巫的想法。
