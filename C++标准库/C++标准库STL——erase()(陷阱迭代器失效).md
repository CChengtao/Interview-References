#### 一、序列式容器(数组式容器)

对于序列式容器(如vector,deque)，序列式容器就是数组式容器，删除当前的iterator会使后面所有元素的iterator都失效。这是因为vetor,deque使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。所以不能使用erase(iter++)的方式，标准库erase函数可以返回下一个有效的iterator。

**正确写法：**

```cpp
std::vector<int> Vec;
std::vector<int>::iterator itVec;
for(itVec = Vec.begin(); itVec != Vec.end(); ) {
    if(WillDelete(*itVec) ) {
        itVec = Vec.erase(itVec);
    }
    else
        itList++;
}
```

#### 二、关联式容器

对于关联容器(如map, set,multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响。erase迭代器只是被删元素的迭代器失效，但是**返回值为void**，所以要采用erase(iter++)的方式删除迭代器。

**正确写法：**

```cpp
std::map<int, int> Map;
std::map<int, int>::iterator itMap;
for(itMap = Map.begin(); itMap != Map.end(); ) {
    if(WillDelete(*itMap) ) {
        List.erase(itMap++);
    }
    else
        itMap++;
}
```

#### 三、链表式容器

对于链表式容器(如list)，删除当前的iterator，仅仅会使当前的iterator失效，这是因为list之类的容器，使用了链表来实现，插入、删除一个结点不会对其他结点造成影响。只要在erase时，递增当前iterator即可，并且erase方法可以返回下一个有效的iterator。

**正确写法1：**

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for(itList = List.begin(); itList != List.end(); ) {
    if(WillDelete(*itList) ) {
        itList = List.erase(itList);
    }
    else
        itList++;
}
```

**正确写法**2

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for(itList = List.begin(); itList != List.end(); ) {
    if(WillDelete(*itList) ) {
        List.erase(itList++);
    }
    else
        itList++;
}
```

**错误写法**1

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for(itList = List.begin(); itList != List.end(); itList++) {
    if(WillDelete(*itList) ) {
        List.erase(itList);
    }
}
```

**错误写法2**

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for(itList = List.begin(); itList != List.end(); ) {
    if(WillDelete(*itList) ) {
        itList = List.erase(++itList);
    }
    else
        itList++;
}
```

#### 四.迭代器失效的情况

##### 4.1 vector

内部数据结构：数组。

随机访问每个元素，所需要的时间为常量。

在末尾增加或删除元素所需时间与元素数目无关，在中间或开头增加或删除元素所需时间随元素数目呈线性变化。

可动态增加或减少元素，内存管理自动完成，但程序员可以使用reserve()成员函数来管理内存。

vector的迭代器在内存重新分配时将失效（它所指向的元素在该操作的前后不再相同）。当把超过capacity()-size()个元素插入vector中时，内存会重新分配，所有的迭代器都将失效；否则，指向当前元素以后的任何元素的迭代器都将失效。当删除元素时，指向被删除元素以后的任何元素的迭代器都将失效。

##### 4.2 deque

内部数据结构：数组。

随机访问每个元素，所需要的时间为常量。

在开头和末尾增加元素所需时间与元素数目无关，在中间增加或删除元素所需时间随元素数目呈线性变化。

可动态增加或减少元素，内存管理自动完成，不提供用于内存管理的成员函数。

增加任何元素都将使deque的迭代器失效。在deque的中间删除元素将使迭代器失效。在deque的头或尾删除元素时，只有指向该元素的迭代器失效。

##### 4.3 list

内部数据结构：双向环状链表。

不能随机访问一个元素。

可双向遍历。

在开头、末尾和中间任何地方增加或删除元素所需时间都为常量。

可动态增加或减少元素，内存管理自动完成。

增加任何元素都不会使迭代器失效。删除元素时，除了指向当前被删除元素的迭代器外，其它迭代器都不会失效。

##### 4.4 slist

内部数据结构：单向链表。

不可双向遍历，只能从前到后地遍历。

其它的特性同list相似。

##### 4.5 stack

适配器，它可以将任意类型的序列容器转换为一个堆栈，一般使用deque作为支持的序列容器。

元素只能后进先出（LIFO）。

不能遍历整个stack。

##### 4.6 queue

适配器，它可以将任意类型的序列容器转换为一个队列，一般使用deque作为支持的序列容器。

元素只能先进先出（FIFO）。

不能遍历整个queue。

##### 4.7 priority_queue

适配器，它可以将任意类型的序列容器转换为一个优先级队列，一般使用vector作为底层存储方式。

只能访问第一个元素，不能遍历整个priority_queue。

第一个元素始终是优先级最高的一个元素。

##### 4.8 set

键和值相等。

键唯一。

元素默认按升序排列。

如果迭代器所指向的元素被删除，则该迭代器失效。其它任何增加、删除元素的操作都不会使迭代器失效。

##### 4.9 multiset

键可以不唯一。

其它特点与set相同。

##### 4.10 hash_set

与set相比较，它里面的元素不一定是经过排序的，而是按照所用的hash函数分派的，它能提供更快的搜索速度（当然跟hash函数有关）。

其它特点与set相同。

##### 4.11 hash_multiset

键可以不唯一。

其它特点与hash_set相同。

##### 4.12 map

键唯一。

元素默认按键的升序排列。

如果迭代器所指向的元素被删除，则该迭代器失效。其它任何增加、删除元素的操作都不会使迭代器失效。

##### 4.13 multimap

键可以不唯一。

其它特点与map相同。

##### 4.14 hash_map

与map相比较，它里面的元素不一定是按键值排序的，而是按照所用的hash函数分派的，它能提供更快的搜索速度（当然也跟hash函数有关）。

其它特点与map相同。

##### 4.15 hash_multimap

键可以不唯一。

其它特点与hash_map相同。