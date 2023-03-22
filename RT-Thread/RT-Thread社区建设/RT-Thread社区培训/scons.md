总结PR：

每提交一个PR之后需要删除掉提交的那个PR

[RT-Thread文档中心PR提交规范](https://www.rt-thread.org/document/site/#/other/pr-rule/pr-rule)

---

## scons

[scons经典示例_社区教程](https://www.rt-thread.org/document/site/#/development-tools/build-config-system/SCons?id=_5-scons-%e5%87%bd%e6%95%b0%e5%9c%a8-sconscript-%e4%b8%ad%e7%9a%84%e7%bb%8f%e5%85%b8%e7%a4%ba%e4%be%8b)

添加路径

![image-20220913204016974](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132040447.png)

添加源文件

![image-20220913204041259](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132040375.png)

桥接SConscript（与上一级SConscript桥接）：

![image-20220913204838021](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132048397.png)

Glob函数：将同一组内的所有指定类型的文件放入列表

![image-20220913205412584](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132054639.png)

```
上面这句话意思是将这一层组内所有的.c后缀文件加入到列表中
```

depend:联系下面的Kconfig文件，在rtconfig.h中生成了USER_USING_MY_FILES宏定义，通过在scons中添加depend来决定在Kconfig中的设置

![image-20220913210657414](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132106500.png)

GetDepend:

![image-20220913211644994](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132116064.png)

```
上面的意思是如果定义了这个宏，那么就把user1.c文件去掉
```




---

## Kconfig

设置为bool类型，对应menuconfig中的是否选项

![image-20220913210401822](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132104916.png)

depends on

![image-20220913211246946](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132112040.png)

![image-20220913211857164](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202209132118244.png)