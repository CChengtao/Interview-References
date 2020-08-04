https://www.cnblogs.com/fnlingnzb-learner/p/9300073.html

STL中的容器按存储方式分为两类，一类是按以数组形式存储的容器（如：vector 、deque)；另一类是以不连续的节点形式存储的容器（如：list、set、map）。在使用erase方法来删除元素时，需要注意一些问题。

# 1.list,set,map容器

   在使用 list、set 或 map遍历删除某些元素时可以这样使用：

## 1.1 正确写法1

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for( itList = List.begin(); itList != List.end(); ) {
    if( WillDelete( *itList) ) {
        itList = List.erase( itList);
    }
    else
        itList++;
}
```

## 1.2 正确写法2

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for( itList = List.begin(); itList != List.end(); ) {
    if( WillDelete( *itList) ) {
        List.erase( itList++);
    }
    else
        itList++;
}
```

## 1.3 错误写法1

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for( itList = List.begin(); itList != List.end(); itList++) {
    if( WillDelete( *itList) ) {
        List.erase( itList);
    }
}
```

## 1.4 错误写法2

```cpp
std::list< int> List;
std::list< int>::iterator itList;
for( itList = List.begin(); itList != List.end(); ) {
    if( WillDelete( *itList) ) {
        itList = List.erase( ++itList);
    }
    else
        itList++;
}
```

## 1.5 分析

正确使用方法1：通过erase方法的返回值来获取下一个元素的位置

正确使用方法2：在调用erase方法之前先使用 “++”来获取下一个元素的位置

错误使用方法1：在调用erase方法之后使用“++”来获取下一个元素的位置，由于在调用erase方法以后，该元素的位置已经被删除，如果在根据这个旧的位置来获取下一个位置，则会出现异常。

错误使用方法2：同上。

# 2. vector,deque容器

在使用 vector、deque遍历删除元素时，也可以通过erase的返回值来获取下一个元素的位置：

## 2.1 正确写法

```cpp
std::vector< int> Vec;
std::vector< int>::iterator itVec;
for( itVec = Vec.begin(); itVec != Vec.end(); ) {
    if( WillDelete( *itVec) ) {
        itVec = Vec.erase( itVec);
    }
    else
        itList++;
}
```

## 2.2 注意

注意：vector、deque 不能像上面的“正确使用方法2”的办法来遍历删除。原因请参考Effective STL条款9。摘录到下面：

1) 对于关联容器(如map, set, multimap,multiset)，删除当前的iterator，仅仅会使当前的iterator失效，只要在erase时，递增当前iterator即可。这是因为map之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响。

```cpp
for (iter = cont.begin(); it != cont.end();) {
    (*iter)->doSomething();
    if (shouldDelete(*iter))
        cont.erase(iter++);
    else
        ++iter;
}
```



因为iter传给erase方法的是一个副本，iter++会指向下一个元素。

2)对于序列式容器(如vector,deque)，删除当前的iterator会使后面所有元素的iterator都失效。这是因为vetor,deque使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。还好erase方法可以返回下一个有效的iterator。

```cpp
for (iter = cont.begin(); iter != cont.end();) {
    (*it)->doSomething();
    if (shouldDelete(*iter))
        iter = cont.erase(iter); 
    else
        ++iter;
}
```

3)对于list来说，它使用了不连续分配的内存，并且它的erase方法也会返回下一个有效的iterator，因此上面两种方法都可以使用。

# 3.迭代器失效的情况

## 3.1 vector

内部数据结构：数组。

随机访问每个元素，所需要的时间为常量。

在末尾增加或删除元素所需时间与元素数目无关，在中间或开头增加或删除元素所需时间随元素数目呈线性变化。

可动态增加或减少元素，内存管理自动完成，但程序员可以使用reserve()成员函数来管理内存。

vector的迭代器在内存重新分配时将失效（它所指向的元素在该操作的前后不再相同）。当把超过capacity()-size()个元素插入vector中时，内存会重新分配，所有的迭代器都将失效；否则，指向当前元素以后的任何元素的迭代器都将失效。当删除元素时，指向被删除元素以后的任何元素的迭代器都将失效。

## 3.2 deque

内部数据结构：数组。

随机访问每个元素，所需要的时间为常量。

在开头和末尾增加元素所需时间与元素数目无关，在中间增加或删除元素所需时间随元素数目呈线性变化。

可动态增加或减少元素，内存管理自动完成，不提供用于内存管理的成员函数。

增加任何元素都将使deque的迭代器失效。在deque的中间删除元素将使迭代器失效。在deque的头或尾删除元素时，只有指向该元素的迭代器失效。

## 3.3 list

内部数据结构：双向环状链表。

不能随机访问一个元素。

可双向遍历。

在开头、末尾和中间任何地方增加或删除元素所需时间都为常量。

可动态增加或减少元素，内存管理自动完成。

增加任何元素都不会使迭代器失效。删除元素时，除了指向当前被删除元素的迭代器外，其它迭代器都不会失效。

## 3.4 slist

内部数据结构：单向链表。

不可双向遍历，只能从前到后地遍历。

其它的特性同list相似。

## 3.5 stack

适配器，它可以将任意类型的序列容器转换为一个堆栈，一般使用deque作为支持的序列容器。

元素只能后进先出（LIFO）。

不能遍历整个stack。

## 3.6 queue

适配器，它可以将任意类型的序列容器转换为一个队列，一般使用deque作为支持的序列容器。

元素只能先进先出（FIFO）。

不能遍历整个queue。

## 3.7 priority_queue

适配器，它可以将任意类型的序列容器转换为一个优先级队列，一般使用vector作为底层存储方式。

只能访问第一个元素，不能遍历整个priority_queue。

第一个元素始终是优先级最高的一个元素。

## 3.8 set

键和值相等。

键唯一。

元素默认按升序排列。

如果迭代器所指向的元素被删除，则该迭代器失效。其它任何增加、删除元素的操作都不会使迭代器失效。

## 3.9 multiset

键可以不唯一。

其它特点与set相同。

## 3.10 hash_set

与set相比较，它里面的元素不一定是经过排序的，而是按照所用的hash函数分派的，它能提供更快的搜索速度（当然跟hash函数有关）。

其它特点与set相同。

## 3.11 hash_multiset

键可以不唯一。

其它特点与hash_set相同。

## 3.12 map

键唯一。

元素默认按键的升序排列。

如果迭代器所指向的元素被删除，则该迭代器失效。其它任何增加、删除元素的操作都不会使迭代器失效。

## 3.13 multimap

键可以不唯一。

其它特点与map相同。

## 3.14 hash_map

与map相比较，它里面的元素不一定是按键值排序的，而是按照所用的hash函数分派的，它能提供更快的搜索速度（当然也跟hash函数有关）。

其它特点与map相同。

## 3.15 hash_multimap

键可以不唯一。

其它特点与hash_map相同。