---
title: python垃圾回收机制
date: 2020-12-15 13:14:21
tags: [python,垃圾回收机制,引用计数]
---

* 本文根据内存管理机制，采用实例来验证内存管理机制，并提出了自己对于python内存管理机制的一些见解。

### 1 By code

#### 1.1 引用计数+垃圾回收机制

```python
import sys
class Person:
    pass

p1 = Person()
p2 = p1
print('p1引用计数：', sys.getrefcount(p1))           # p1引用计数： 3
# del p2
# print('p1引用计数：', sys.getrefcount(p1))         # p2引用计数： 2

print(id(p1))                                       # 1901831032904
print(id(p2))                                       # 1901831032904

if p1 is p2:
    print('yes')                                    # yes

print(sys.getsizeof(p1))                            # 56 bites
```

#### 1.2 分代回收

```python
import sys
class Person:
    pass

class Dog:
    pass

p = Person()
d = Dog()
p.pet = d
d.master = p

print('p引用计数：', sys.getrefcount(p))           # p引用计数： 3
print('d引用计数：', sys.getrefcount(d))           # d引用计数： 3

del p
print('d引用计数：', sys.getrefcount(d))           # d引用计数： 3
```

###  2 Notice

#### 2.1 引用计数

引用计数是编程语言总的一种内存管理技术，将资源的被引用次数保存起来。

* 资源：可以是对象，内存或磁盘空间等
* def语句会删除对象的一个引用，这将导致该引用指向的对象的引用计数减1
* 任何追踪或调试程序会给对象增加一个额外引用，这会推迟该对象被回收的时间
* 由于两个或以上对象互相引用时，彼此引用计数不为0，造成循环应用而无法回收

#### 2.2 标记-清除

* 标记清除分为两个阶段，首先标记对象（垃圾检测），然后清除垃圾（垃圾回收）

##### 2.2.1 标记

在此阶段，垃圾回收器会从跟对象开始遍历。

每个可以从根对象访问到的对象都会被添加标识，这个对象被标识为可到达对象，可到达对象不会被清除。

##### 2.2.2 清除

在此阶段，垃圾回收器会对堆内存从头到尾进行线性遍历

如果发现有对象没有被标识为可到达对象，那么就此对象内存回收

之后将原来标记为可到达对象的标识抹掉，以便进行下一次垃圾回收操作

##### 2.2.3 标记清除存在的问题

标记-清除算法的比较大的缺点就是垃圾收集后有可能会造成大量的内存碎片，其次由于需要遍历的内存节点较多时，每次回收都将花费时间在遍历节点过程中

##### 2.2.4 注意事项

* collector 

垃圾收集器

* mutator

指的是垃圾收集器之外的部分（比如当前的应用程序，可以直接被mutator直接访问到的对象，一般指静态---->全局变量）

* 可到达对象

所谓的可到达对象就是从根对象开始遍历，可以访问到的对象，也就是mutator(应用程序)正在使用的对象

#### 2.3 分代回收

* 由于嵌套引用无法被回收（分层，打标签）

##### 2.3.1 核心思想

分代是一种典型的以空间换时间的技术

这种思想简单点说就是：对象存在时间越长，越可能不是垃圾，应该越少去收集

分代是解决内存遍历需要太久时间而诞生的解决方案

##### 2.3.2 分代回收的好处

分代回收可以减少标记-清除机制所带来的额外操作

分代就是将回收对象分成数个代（青年代，中年代，老年代），每个代都是一个链表（集合）。当青年代满时，将触发清理所有三代，执行标记清除动作。之后，当中年代满时，将会触发清理中年代，老年代；最后，老年代触发后只会清理自己

老年代的存活时间是最长的