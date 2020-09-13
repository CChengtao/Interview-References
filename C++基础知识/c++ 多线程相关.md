#### 一、阻塞队列

阻塞队列主要用于线程和线程之间的通信。当队列为空时，从队列中获取元素的线程将会被挂起；当队列是满时，往队列里添加元素的线程将会挂起。

阻塞队列代码实现：

```cpp
#pragma once
#include <condition_variable>
#include <mutex>
#include <deque>
template<class T>
class BlockQueue
{
public:
	typedef std::unique_lock<std::mutex> TLock;
 
	//maxCapacity为-1，代表队列无最大限制
	explicit BlockQueue(const int maxCapacity = -1):m_maxCapacity(maxCapacity)
	{
 
	}
 
	size_t size()
	{
		TLock lock(m_mutex);
		return m_list.size();
	}
 
	void push_back(const T &item)
	{
		TLock lock(m_mutex);
		if (true == hasCapacity())
		{
			while (m_list.size() >= m_maxCapacity)
			{
				m_notFull.wait(lock);
			}
		}
 
		m_list.push_back(item);
		m_notEmpty.notify_all();
	}
 
	T pop()
	{
		TLock lock(m_mutex);
		while (m_list.empty())
		{
			m_notEmpty.wait(lock);
		}
 
		T temp = *m_list.begin();
		m_list.pop_front();
 
		m_notFull.notify_all();
 
		lock.unlock();
		return temp;
	}
 
	bool empty()
	{
		TLock lock(m_mutex);
		return m_list.empty();
	}
 
	bool full()
	{
		if (false == hasCapacity)
		{
			return false;
		}
 
		TLock lock(m_mutex);
		return m_list.size() >= m_maxCapacity;
	}
 
private:
	bool hasCapacity() const
	{
		return m_maxCapacity > 0;
	}
 
	typedef std::deque<T> TList;
	TList m_list;
 
	const int m_maxCapacity;
 
	std::mutex m_mutex;
	std::condition_variable m_notEmpty;
	std::condition_variable m_notFull;
};
```

测试代码：

```cpp
#include <iostream>
#include <thread>
#include <stdio.h>
#include "BlockQueue.hpp"
using namespace std;
 
typedef BlockQueue<int> TQueue;
 
void produce(TQueue &queue)
{
	const int num = 9;
	for (int i = 0; i < num; ++i)
	{
		queue.push_back(i);
	}
}
 
void consume(TQueue &queue)
{
	while (false==queue.empty())
	{
		int tmp = queue.pop();
		printf("%d\n", tmp);
		std::this_thread::sleep_for(chrono::seconds(1));
	}
}
 
int main()
{
	TQueue queue(2);
	std::thread t1(produce, std::ref(queue));
	std::thread t2(consume, std::ref(queue));
	std::thread t3(consume, std::ref(queue));
	t3.join();
	t2.join();
	t1.join();
	return 0;
}
```

#### 二、生产者-消费者模型

用条件变量实现了一个简单的生产者-消费者模型：

```cpp
#include <iostream>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
 
int counter=0;
int maxSize = 30;
std::mutex mtx;
std::queue<int> dataQuene; // 被生产者和消费者共享
 
std::condition_variable producer, consumer;  // 条件变量是一种同步机制，要和mutex以及lock一起使用
 
void func_consumer()
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));// 消费者比生产者慢
 
        std::unique_lock<std::mutex> lck(mtx);
        consumer.wait(lck, [] {return dataQuene.size() != 0; });     // 消费者阻塞等待， 直到队列中元素个数大于0
        int num=dataQuene.front();
        dataQuene.pop();
        std::cout << "consumer " << std::this_thread::get_id() << ": "<< num <<std::endl;
 
        producer.notify_all();                                       // 通知生产者当队列中元素个数小于maxSize
    }
}
 
void func_producer()
{
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(900));  // 生产者稍微比消费者快
        std::unique_lock<std::mutex> lck(mtx);
        producer.wait(lck, [] {return dataQuene.size() != maxSize; });// 生产者阻塞等待， 直到队列中元素个数小于maxSize
 
        ++counter;
        dataQuene.push(counter);
        std::cout << "producer " << std::this_thread::get_id() << ": "<< counter <<std::endl;
 
        consumer.notify_all();                                        //通知消费者当队列中的元素个数大于0
    }
}
 
int main()
{
    std::thread consumers[2], producers[2];
 
    // 两个生产者和消费者
    for (int i = 0; i < 2; ++i)
    {
        consumers[i] = std::thread(func_consumer);
        producers[i] = std::thread(func_producer);
    }
 
    for (int i = 0; i < 2; ++i)
    {
        producers[i].join();
        consumers[i].join();
    }
 
    system("pause");
    return 0;
}
```



#### 三、多线程实现交替打印两个数组

互斥锁实现：

```cpp
#include <string>
#include <thread>
#include <mutex>
#include <iostream>
using namespace std;
 
std::mutex data_mutex;
int flag = 0;
 
void printA(int* a, int size) {
	for (int i = 0; i < size; ) {
		data_mutex.lock();
		if (flag == 0) {
			cout << a[i] << endl;
			flag = 1;
			++i;
		}
		data_mutex.unlock();
	}
}
 
void printB(char* b, int size) {
	for (int i = 0; i < size; ) {
		data_mutex.lock();
		if (flag == 1) {
			cout << b[i] << endl;
			flag = 0;
			++i;
		}
		data_mutex.unlock();
	}
}
 
int main()
{
	int a[4] = { 1, 2, 3, 4 };
	char b[4] = { 'a', 'b', 'c', 'd' };
	std::thread tA(&printA, a, 4);
	std::thread tB(&printB, b, 4);
	tA.join();
	tB.join();
 
	return 0;
}
```

互斥锁+条件变量：

```cpp
#include <string>
#include <thread>
#include <mutex>
#include <iostream>
using namespace std;
 
std::mutex data_mutex;
std::condition_variable data_var;
int flag = 0;
 
void printA(int* a, int size) {
	for (int i = 0; i < size; i++) {
		
		std::unique_lock<std::mutex> lck(data_mutex);
		data_var.wait(lck, [] {return flag == 0; });
		
		cout << a[i] << endl;
		flag = 1;
		
		data_var.notify_all();
		
	}
}
 
void printB(char* b, int size) {
	for (int i = 0; i < size; ++i) {
		std::unique_lock<std::mutex> lck(data_mutex);
		data_var.wait(lck, [] {return flag == 1; });
		
		cout << b[i] << endl;
		flag = 0;
		
		data_var.notify_all();
	}
}
 
int main()
{
	int a[4] = { 1, 2, 3, 4 };
	char b[4] = { 'a', 'b', 'c', 'd' };
	std::thread tA(&printA, a, 4);
	std::thread tB(&printB, b, 4);
	tA.join();
	tB.join();
 
	return 0;
}
```

信号量，PV操作：

```cpp
#include <thread>
#include <mutex>
#include <iostream>
#include <vector>
 
using namespace std;
const int PRONUM = 10; // 生产者总个数
const int CONSUNUM = 5; // 消费者总个数
const int SUM = 30; // 需要的商品总个数
const int CAP = 20; // 缓冲区长度
vector<int> storage(CAP); // 缓冲区
int cnt = 0; // 仓库内货物计数器，判断是全满还是全空
mutex mutex_storage;// 保护缓冲区的锁
condition_variable con_emp;
condition_variable con_full;
static int has_consumed = 0; // 总共消费的
static int has_produced = 0; // 总共生产的
 
class Pointer {
private:
	int p;
public:
	Pointer() : p(0){}
	int& operator++() {
		p++;
		if (p == CAP) {
			p = 0;
		}
		return p;
	}
	int& operator++(int) {
		return ++(*this);
	}
	int& operator*() {
		return p;
	}
};
 
Pointer p_em; // 指向第一个空的位置
Pointer p_full; // 指向第一个满的位置
 
// 生产者线程
void produce() {
	do{
		unique_lock<mutex> lck(mutex_storage);
		con_emp.wait(lck, []() {return cnt != CAP; });
		storage[*p_em] = has_produced;
		cnt++;
		cout << this_thread::get_id() << "生产了" << storage[*p_em] << "号， 目前有：" << cnt << "个" << endl;
		this_thread::sleep_for(std::chrono::milliseconds(100));
		p_em++;
		has_produced++;
		con_full.notify_one();
	} while (has_produced <= SUM);
}
 
void consume() {
	do {
		unique_lock<mutex> lck(mutex_storage);
		con_full.wait(lck, []() {return cnt != 0; });
		cnt--;
		cout << this_thread::get_id() << "消费了" << storage[*p_full] << "号， 目前有：" << cnt << "个" << endl;
		this_thread::sleep_for(std::chrono::milliseconds(500));
		p_full++;
		has_consumed++;
		con_emp.notify_one();
	} while (has_consumed <= SUM);
}
 
int main()
{
	vector<thread> vec_p;
	vector<thread> vec_c;
	for (int i = 0; i < PRONUM; ++i) {
		vec_p.push_back(thread(produce));
	}
	for (int i = 0; i < CONSUNUM; ++i) {
		vec_c.push_back(thread(consume));
	}
	for (int i = 0; i < PRONUM; ++i) {
		vec_p[i].join();
	}
	for (int i = 0; i < CONSUNUM; ++i) {
		vec_c[i].join();
	}
	return 0;
}
```

