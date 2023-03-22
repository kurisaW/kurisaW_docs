## 今日任务

* KMP算法详解
* 28.实现 strStr()
* 459.重复的子字符串
* 字符串总结
* 双指针回顾

## 1.KMP算法详解

由于今天的算法题涉及到KMP算法，所以这里我们提前学习一下。

#### （1）什么是KMP算法

说到KMP，先说一下KMP这个名字是怎么来的，为什么叫做KMP呢。

因为是由这三位学者发明的：Knuth，Morris和Pratt，所以取了三位学者名字的首字母。所以叫做KMP。

#### （2）KMP的作用

KMP主要体现在**字符串匹配**上。

KMP算法的主要思想是**当出现字符串不相匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头到尾再去匹配。**

因此如何记录已经匹配的文本内容，是KMP的重点，也是next数组肩负的重任。

#### （3）什么是前缀表

前缀表有什么作用呢？

**前缀表是用来回退的，它记录了模式串与主串（文本串）不匹配时，模式串应该从哪里开始重新匹配。**

其中我们会了解到next数组，**next数组其实就是一个前缀表(prefix table)**。

为了更加清楚地了解前缀表的来历，我们来举一个例子：

`在文本串：aabaabaafa中查找是否出现过一个模式串：aabaaf。`

如下面动画所示（来源：代码随想录）：

![KMP精讲1](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302251720943.gif)

我们从上面的动画可以看出，文本串中第六个字符b和模式串的第六个个字符f已经不匹配了。如果暴力匹配的话，需要从头开始匹配；但是如果我们使用前缀表的话，就不会从头匹配，而是从上次已经匹配的内容开始匹配，也就是模式串中第三个字符b继续开始匹配。

那么**前缀表时如何记录的呢？**

首先要知道前缀表的任务是当前任务匹配失败，找到之前已经匹配上的位置，再重新匹配，这也意味着再某个字符失配时，前缀表会告诉你，下一步匹配中，模式串应该跳到哪个位置。

所以前缀表的定义是：**记录下标i之前(包含i)的字符串中，有多大长度的相同前缀后缀**。

#### （4）什么是最长公共前后缀

前文中字符串的前缀是指**不包含最后一个字符的所有以第一个字符开头的连续子串**。

**后缀**是指**不包含第一个字符的所有以最后一个字符结尾的连续子串**。

![image-20230226205706410](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302262057510.png)

那么我们回到**最长公共前后缀**，更加准确的理解应该是“最长相等前后缀”，因为**前缀表的要求就是相同前后缀**。

而最长公共前后缀里面的“公共”，更像是在说前缀和后缀公共的长度。这其实并不是前缀表所需要的。

所以字符串a的最长相等前后缀为0；字符串aa的最长相等前后缀为1，字符串aaa的最长相等前后缀为2。

#### （5）如何计算前缀表

我们先来看几个例子：

![image-20230225205304564](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302252053992.png)

解说：长度为前1个字符的子串a，最长相同前后缀的长度为0.

`注意：字符串的前缀是指不包含最后一个字符的所有以第一个字符开头的连续子串；后缀是指不包含第一个字符的所有以最后一个字符结尾的连续子串。`

![image-20230225205831598](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302252058968.png)

解说：长度为前2个字符的子串aa，最长相同前后缀的长度为1.

![image-20230225210252121](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302252102489.png)

解说：长度为前3个字符的子串aab，最长相同前后缀的长度为0.

...

以此类推：长度为前4个字符的子串aaba，最长相同前后缀的长度为1；长度为前5个字符的子串aabaa，最长相同前后缀的长度为2；长度为前6个字符的子串aabaaf，相同前后缀的长度为0.

最后把求得的最长相同前后缀的长度就是对应前后缀表的元素，如下图：

![image-20230225213153188](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302252131723.png)

可以看出模式串与前缀表对应位置的数字表示的就是：**下标i之前（包括i）的字符串中，有多大长度的相同前后缀**.

我们再来看下如何利用前缀表找到:当字符不匹配的时候指针应该移动的位置。如下动画所示：

![KMP精讲2](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302252329586.gif)

当找到不匹配的位置，此时我们需要看它的前一个字符的前缀表的数值是多少。

之所以要前一个字符的前缀表的数值，是因为要找到前面字符串的最长相同的前后缀。

所以我们要看前一位的前缀表数值，动画中显示为2，所以将下标移动到下标2的位置继续匹配。直到在文本串中找到和模式串匹配的子串。

#### （5）前缀表与next数组

很多KMP算法的时间都是使用next数组做回退操作，那么next数组与前缀表有什么关系？

前面我们讲了，next数组其实就可以被认为是前缀表，但是很多实现都是把前缀表统一减一（右移一位，初始位置为-1）。

#### （6）使用next数组匹配

以下我们以前缀表统一减一之后的next数组来做演示。

注意此时的前缀表已经实现同一减一了，匹配动画如下：

![KMP精讲4](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302260850345.gif)

#### （7）时间复杂度分析

其中n为文本串长度，m为模式串长度，因为在匹配的过程中，根据前缀表不断调整匹配的位置，可以看出匹配的过程是O(n)，之前还要单独生成next数组，时间复杂度是O(m)。所以整个KMP算法的时间复杂度是O(n+m)

而暴力解法的时间复杂度明显是O(n * m)，所以可知**KMP在字符串匹配中极大地提高了搜索的效率**。

##  2.Leetcode28.实现 strStr()

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string

#### （1）题目

**给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串的第一个匹配项的下标（下标从 0 开始）。如果 needle 不是 haystack 的一部分，则返回  -1 。**

示例 1：

```cpp
输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
```

示例 2：

```cpp
输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。
```

**提示：**

* 1 <= haystack.length, needle.length <= 104
* haystack 和 needle 仅由小写英文字符组成

#### （2）思路

前提说明：学习该小结需要提前对KMP算法有一定的了解，请详细阅读第一小节。

在本题目中，haystack（文本串），needle（模式串）。

解答此题目我们需要使用到KMP算法，那么使用KMP算法，需要我们构造next数组。

###### <1>构造next数组

我们定义一个函数getNext来构建next数组，函数参数为指向next数组的指针，和一个字符串。代码如下：

```cpp
void getNext(int* next, const string& s)
```

**构造next数组其实就是计算模式串s、前缀表的过程。**主要有三步：

* 1.初始化
* 2.处理前后缀不相同的情况
* 3.处理前后缀相同的情况

下面我们来详细讲解：

**1.初始化**

定义了两个指针i和j，j指向前缀末尾位置，i指向后缀末尾位置。

然后对next数组进行初始化赋值：

```cpp
int j = -1;
next[0] = j;
```

这里之所以将j初始化为-1，是因为前面我们讲过前缀表要统一减一（当然也可以选择j不初始化为-1）

next[i]表示i（包括i）之前最长相等的前后缀长度（其实就是j）

所以初始化为next[0] = j;

**2.处理前后缀不相同的情况**

因为j初始化为-1，那么i就从1开始，并将s[i]与s[j + 1]进行比较。

所以遍历模式串s的循环下标i要从1开始，代码如下：

```cpp
for(int i = 1; i < s.size(); i++){}
```

如果s[i]与s[j + 1]不相同，也就是遇到前后缀末尾不相同的情况，就要向前回退。

这里我们再次明确一点：next[j]记录着j(包括j)之前的子串的相同前后缀的长度。

s[i]与s[j + 1]不相同，那么我们就要找一个j + 1前一个元素在next数组里的值（就是next[j]）。

所以，处理前后缀不相同的情况的代码如下所示：

```cpp
while(j >= 0 && s[i] != s[j + 1]){ //前后缀不相同的情况
    j = next[j];	// 向前回退
}
```

`注意：此处之所以写成while而不是if，是因为字符串回退并不是一步就可以的，而是一个连续回退的过程。`

**3.处理前后缀相同的情况**

如果s[i]与s[j + 1]相同，那么就同时向后移动i和j说明找到了相同的前后缀，同时还要将j（前缀的长度）赋值给next[i]，因为next[i]要记录相同前后缀的长度。如下所示：

```cpp
if(s[i] == s[j + 1]){ // 找到相同的前后缀
    j++;
}
next[i] = j;
```

最后整体构建next数组的函数代码如下所示：

```cpp
void getNext(int* next, const string& s){
    int j = -1;
    next[0] = j;
    for(int i = 1; i < s.size(); i++){	// 注意i从1开始
        while(j >= 0 && s[i] != s[j + 1]){	// 前后缀不相同的时候
            j = next[j];	// 向前回退
        }
        if(s[i] == s[j + 1]){ 	// 找到相同的前后缀
            j++;
        }
        next[i] = j;	// 将j（前缀的长度）赋值给next[i]
    }
}
```

代码构造next数组的逻辑流程动画如下(来源：代码随想录)：

![KMP精讲3](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302262029303.gif)

###### <2>使用next数组进行匹配

目标：在文本串中找是否出现过模式串t。

首先定义两个下标j指向模式串起始位置，i指向文本串起始位置。

此时j初始值依然为-1，因为next数组中记录的起始位置为-1.

i从0开始，遍历文本串，代码如下：

```cpp
for(int i = 0; i < s.size(); i++)
```

接下来就是s[i]与t[j + 1]（因为从-1开始）进行比较。

如果s[i]与t[j + 1]不相同，就要从next数组中需按照下一个匹配的位置，代码如下：

```cpp
while(j >= 0 && s[i] != t[j + 1]){
    j = next[j];
}
```

如果s[i]与t[j + 1]相同，那么i和j同时向后移动，代码如下：

```cpp
if(s[i] == t[j + 1]){
    j++;	// i的增加在for循环中定义
}
```

那么如何判断在文本串中出现了模式串t？如果j指向了模式串t的末尾，那么就说明模式串t完全匹配文本串s里的某个子串了。

模式串出现的位置：当前在文本串匹配模式串的位置i减去模式串的长度。代码如下：

```cpp
if(j == (t.size() - 1)){
	return (i - t.size() + 1);
}
```

因此使用next数组，用模式串匹配文本串的整体代码如下：

```cpp
int j = -1;	// 因为next数组里记录的起始位置为-1
for(int i = 0; i < s.size(); i++){	// 注意i从0开始
    while(j >= 0 && s[i] != t[j + 1]){	// 不匹配
    	j = next[j]; 	// j寻找之前匹配的位置
    }
    if(s[i] == t[j + 1]){	// 匹配，j和i同时向后移动
    	j++;	// i的增加在for循环中
    }
    if(j == (t.size() - 1)){	// 文本串s里出现了模式串t
    	return (i - t.size() + 1);
    }
}
```

#### （3）代码实现

```cpp
// 前缀表统一减一

class Solution {
public:
    void getNext(int* next, const string& s) {
        int j = -1;
        next[0] = j;
        for(int i = 1; i < s.size(); i++) { // 注意i从1开始
            while (j >= 0 && s[i] != s[j + 1]) { // 前后缀不相同了
                j = next[j]; // 向前回退
            }
            if (s[i] == s[j + 1]) { // 找到相同的前后缀
                j++;
            }
            next[i] = j; // 将j（前缀的长度）赋给next[i]
        }
    }
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        int next[needle.size()];
        getNext(next, needle);
        int j = -1; // // 因为next数组里记录的起始位置为-1
        for (int i = 0; i < haystack.size(); i++) { // 注意i就从0开始
            while(j >= 0 && haystack[i] != needle[j + 1]) { // 不匹配
                j = next[j]; // j 寻找之前匹配的位置
            }
            if (haystack[i] == needle[j + 1]) { // 匹配，j和i同时向后移动
                j++; // i的增加在for循环里
            }
            if (j == (needle.size() - 1) ) { // 文本串s里出现了模式串t
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};
```

```cpp
// 前缀表（不减一）

class Solution {
public:
    void getNext(int* next, const string& s) {
        int j = 0;
        next[0] = 0;
        for(int i = 1; i < s.size(); i++) {
            while (j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            if (s[i] == s[j]) {
                j++;
            }
            next[i] = j;
        }
    }
    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        int next[needle.size()];
        getNext(next, needle);
        int j = 0;
        for (int i = 0; i < haystack.size(); i++) {
            while(j > 0 && haystack[i] != needle[j]) {
                j = next[j - 1];
            }
            if (haystack[i] == needle[j]) {
                j++;
            }
            if (j == needle.size() ) {
                return (i - needle.size() + 1);
            }
        }
        return -1;
    }
};
```

## 3.Leetcode459.重复的子字符串

> 来源：力扣（LeetCode）
> 链接：https://leetcode.cn/problems/repeated-substring-pattern

#### （1）题目

**给定一个非空的字符串 s ，检查是否可以通过由它的一个子串重复多次构成。** 

示例 1:

```cpp
输入: s = "abab"
输出: true
解释: 可由子串 "ab" 重复两次构成。
```

示例 2:

```cpp
输入: s = "aba"
输出: false
```

示例 3:

```cpp
输入: s = "abcabcabcabc"
输出: true
解释: 可由子串 "abc" 重复四次构成。 (或子串 "abcabc" 重复两次构成。)
```

**提示：**

* 1 <= s.length <= 104
* s 由小写英文字母组成

#### （2）思路

对这道题我们有三种解决方法：暴力解法、移动匹配和KMP。

首先来看暴力解法，也就是一个for循环去获取子串的终止位置，再嵌套一个for循环判断子串是否能够重复构成字符串，所以时间复杂度为O(n^2)。

这里我们主要对移动匹配和KMP两种方法进行讲解。

###### <1>移动匹配

首先我们来看题目，假设字符串s为：abcabc，内部由重复子串组成，那么该字符串的结构如下图所示：

![image-20230227090301956](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302270903259.png)

那么既然前面有相同的子串，后面也有相同的子串，我们换个思路，是不是将后面的子串作为前串，前面的子串作为后串，这样一来是不是也能构成一个字符串s呢。

![image-20230227090746221](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202302270907324.png)

所以我们的思路就是：将两个s拼接起来，如果还能出现额外的一个s，那就说明该串是由重复子串构成。

这里为了避免在s+s搜索的时候搜索出原来的字符串s，这里我们需要进行**掐头去尾**（刨除s+s的首字符和尾字符），代码如下：

```cpp
class Solution{
public:
    bool repeatedSubstringPatterns(string s){
        string t = s + s;
        t.erase(t.begin());
        t.erase(t.end() - 1);	// 掐头去尾
        if(t.find(s) != std::string::npos) return true;
        return false;
    }
};
```

虽然这个解法可行，但是后面我们还需要对字符串（s+s）是否出现过s做一个判断，在这个过程是增加了时间复杂度的算法成本的，例如使用库函数find、contains，一般的库函数的实现的时间复杂度为O(m + n)。

###### <2>KMP

想到KMP，就想到了KMP算法的字符串匹配，我们要在一个串中查找是否出现另外一个串，这才是KMP算法的专长所在.

代码如下：

```cpp
class Solution {
public:
    void getNext(int* next, const string& s){
        next[0] = 0;
        int j = 0;
        for(int i = 1;i < s.size(); i++){
            while(j > 0 && s[i] != s[j]){
                j = next[j - 1];
            }
            if(s[i] == s[j]){
                j++;
            }
            next[i] = j;
        }
    }
    bool repeatedSubstringPattern(string s) {
        if(s.size() == 0){
            return false;
        }
        int next[s.size()];
        getNext(next, s);
        int len = s.size();
        if(next[len - 1] != 0 && len % (len - (next[len - 1])) == 0){
            return true;
        }
        return false;
    }
};
```

## 4.字符串总结

对于本章节，涉及到很多经典的算法，最常见的就是双指针法，以及我们头疼的KMP算法（这部分其实我本人也没有很理解，需要反复理解）。

