示例：

**输入：nums1 = \[1,2,2,1\], nums2 = \[2,2\] 
输出：\[2\]** 

---

## 1.API使用

unordered\_set成员方法.find\(key\)

```cpp
unordered_set.find(key)
```

查找值为key的元素，如果找到，就返回指向该元素的正向迭代器；反之，则返回尾后迭代器\(结束迭代器\)。

```cpp
unordered_set.end()
```

直接返回尾后迭代器。即返回指向容器最后一个数据单元+1的指针。

```cpp
unordered_set<int> result_set;
```

定义哈希表存放结果

```cpp
unordered_set<int> nums_set(nums1.begin(), nums1.end());
```

实现功能：\[1,1,2,3,3\]->\[1,2,3\]，即去除重复元素。

```cpp
if(nums_set.find(num) != nums_set.end())
```

这个判断语句的作用是：如果在nums2中找到了存在在nums\_set中的元素，那么.find\(\)返回的不是尾迭代器，所以进入if判断语句，然后将元素，放进result\_set中；如果没有找到也就是说nums2中存在和nums\_set不重复的数，即返回end迭代器，继续下一次循环。

## 2.API总结

1、C++运算法 / 代表除法运算，如果商含有小数，则直接弃除。例：1/10的值为0。  
2、如果题目要求是判断某一个元素是否出现在集合里的时候，就要考虑使用哈希法，`(unordered_set)`  
3、哈希法一般会使用的三种数据结构：

* 数组
* set（集合）
* map（映射）

在C++中，set 和 map 分别提供以下三种数据结构：

#### （1）set

1.  std::set
2.  std::multiset
3.  std::unordered\_set

#### （2）map

1.  std::map
2.  std::multimap
3.  std::unordered\_map

**关于unordered\_map： **
unordered\_map和map类似，都是存储的key-value的值，可以通过key快速索引到value。不同的是unordered\_map不会根据key的大小进行排序，存储时是根据key的hash值判断元素是否相同，即unordered\_map内部元素是无序的。

**关于map和set： **
map是指两个集合之间元素的相互之间的关系，一个元素对应另外的元素。 
而set是存储同以数据类型的数据类型，就是没有所谓的对应关系。

**关于C++中的iterator->second**

```cpp
std::unordered_map<int,int> map;
```

实际上是存储了一串`std::pair<const X, Y>`， 
如果定义`auto iter = map.begin();` 
就是说lter这个迭代器相当于指向了map的第一个元素的pair，然后这个时候可以访问这个pair的两个元素：

```cpp
(*iter).first//得到key值
(*iter).second//得到value值
```

等价与：

```cpp
iter->first
iter->second
```

> 函数erase：

（1）erase \( size\_t pos = 0, size\_t n = npos \);删除从pos开始的n个字符，比如erase\(0,1\)就是删除第一个字符  
（2）iterator erase \( iterator position \);删除position处的一个字符\(position是个string类型的迭代器\)  
（3）iterator erase \( iterator first, iterator last \);删除从first到last之间的字符（first和last都是迭代器）

```cpp
vector<vector<int>> C
```

相当于容器的嵌套，最里面是int型，外层是`vector<int>`类型，类似于二元数组。

> continue

跳出本次循环，开始下一次循环。

> break

直接终止循环，执行循环下面的语句。