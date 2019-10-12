---
layout:     post
title:      容器vector中的reserve和resize
subtitle:   reserve(),resize(),size(),capacity()
date:       2019-10-12
author:     Cory
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - c++ 容器
---
最近在做项目的时候，看到这样一个代码：vector.reserve(mesh.n_vertices())我对这个reserve产生了好奇，查阅资料后，对reserve和resize有了更深的理解。
# vector
在介绍resize()，reserve()，size()和capacity()函数之前，先简单介绍一下c++中vector的概念。

vector：顺序容器（可变大小数组）。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢。

既然vector是个容器，那么一定相关特性，如添加元素、删除元素和查询容器大小等操作。本文重点介绍vector中的resize()，reserve()，size()和capacity()函数。

# 基本概念
### capacity

指容器在分配新的存储空间之前能存储的元素总数。

### size

指当前容器所存储的元素个数

**注：capacity是容器可存储的最大总数，size是当前容器存储的个数。**



### resize
既分配了空间，也创建了对象。**默认初始化为0**

这里空间就是capacity，对象就是容器中的元素。

### reserve
reserve()表示容器预留空间，但不是真正的创建对象，需要通过insert()或push_back()等操作创建对象。

其实size()和capacity()没有更多需要介绍的地方，大家平时coding时直接调用即可。当然size()的使用频率相当高，通常进行遍历操作时，最外层的for循环的次数即为size()。

# reserve或resize的意义
我们都知道，vector是一个可变大小的容器，不管你用不用reserve或者resize，你都可以使用push_back()函数往容器里添加元素，但是stl中为什么要有reserve或者resize呢？

因为当你push_back()时，如果size将要超过capacity，那么vector就会为你动态分配内存，通过一组实验结果可以看到，**大小是capacity的一半**。
```
int nums = 20;
for (int i = 0; i < nums; ++i){
    v2.push_back(i+1);
    cout << "v2_size: " << v2.size() << "\t v2_capacity: " << v2.capacity() << endl;
}

让我们直接看结果：

v2_size: 2 ， v2_capacity: 2

v2_size: 3 ， v2_capacity: 3

v2_size: 4 ， v2_capacity: 4

v2_size: 5 ， v2_capacity: 6

v2_size: 6 ， v2_capacity: 6

v2_size: 7 ， v2_capacity: 9

v2_size: 8 ， v2_capacity: 9

v2_size: 9 ， v2_capacity: 9

v2_size: 10 ， v2_capacity: 13

v2_size: 11 ， v2_capacity: 13

v2_size: 12 ， v2_capacity: 13

v2_size: 13 ， v2_capacity: 13

v2_size: 14 ， v2_capacity: 19

v2_size: 15 ， v2_capacity: 19

v2_size: 16 ， v2_capacity: 19

v2_size: 17 ， v2_capacity: 19

v2_size: 18 ， v2_capacity: 19

v2_size: 19 ， v2_capacity: 19

v2_size: 20 ， v2_capacity: 28

v2_size: 21 ， v2_capacity: 28
```
当你把特别大的数据输入到一个vector时，这个分配内存的操作就会频繁发生，这显得非常的低效，比如我想把一个模型文件.obj输入到vector中，几十万甚至上百万点的输入就会多次分配内存，而如果一开始通过reserve指定分配内存大小，就会高效的多。
# 总结
- size()：返回vector中的元素个数
- capacity()：返回vector能存储元素的总数
- resize()操作：创建指定数量的的元素并指定vector的存储空间
- reserve()操作：指定vector的元素总数
