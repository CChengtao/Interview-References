单例模式顾名思义，保证一个类仅可以有一个实例化对象，并且提供一个可以访问它的全局接口。实现单例模式必须注意一下几点：

- **单例类只能由一个实例化对象**。
- **单例类必须自己提供一个实例化对象**。
- **单例类必须提供一个可以访问唯一实例化对象的接口**。

单例模式分为懒汉和饿汉两种实现方式。

#### 1、懒汉单例模式

懒汉：故名思义，不到万不得已就不会去实例化类，也就是说在第一次用到类实例的时候才会去实例化一个对象。**在访问量较小，甚至可能不会去访问的情况下，采用懒汉实现，这是以时间换空间。**

##### 1.1、非线程安全的懒汉单例模式

```cpp
/*
* 关键代码：构造函数是私有的，不能通过赋值运算，拷贝构造等方式实例化对象。
*/
//懒汉式一般实现：非线程安全，getInstance返回的实例指针需要delete
class Singleton {
public:
    static Singleton* getInstance();						//静态函数
    ~Singleton(){}											//析构函数
private:
    Singleton() {}
    Singleton(const Singleton& obj) = delete;            	//明确拒绝默认构造函数
    Singleton& operator=(const Singleton& obj) = delete; 	//明确拒绝默认赋值函数
    static Singleton* m_pSingleton;							//静态成员
};

Singleton* Singleton::m_pSingleton = NULL;					//静态成员初始化

Singleton* Singleton::getInstance() {						//函数实现
    if(m_pSingleton == NULL) {
        m_pSingleton = new Singleton();
    }
    return m_pSingleton;
}
```

##### 1.2、线程安全的懒汉单例模式

```cpp
std::mutex mt;
class Singleton {
public:
    static Singleton* getInstance();					//静态函数
private:
    Singleton(){}                                    	//构造函数私有
    Singleton(const Singleton&) = delete;            	//明确拒绝构造函数
    Singleton& operator=(const Singleton&) = delete; 	//明确拒绝赋值函数
    static Singleton* m_pSingleton;						//自引用（静态变量）
};

Singleton* Singleton::m_pSingleton = NULL;				//静态成员变量初始化

Singleton* Singleton::getInstance() {					//函数实现
    if(m_pSingleton == NULL) {
        mt.lock();										//加锁判断
        if(m_pSingleton == NULL) {
            m_pSingleton = new Singleton();
        }
        mt.unlock();									//解锁
    }
    return m_pSingleton;
}
```

##### 1.3、返回一个reference指向local static对象

这种单例模式实现方式多线程可能存在不确定性：**任何一种non-const static对象，不论它是local或non-local,在多线程环境下“等待某事发生”都会有麻烦。**解决的方法：**在程序的单线程启动阶段手工调用所有reference-returning函数**。这种实现方式的好处是不需要去delete它。

```cpp
class Singleton {
public:
    static Singleton& getInstance();					//静态成员
private:
    Singleton(){}										//默认构造函数
    Singleton(const Singleton&) = delete;  				//明确拒绝构造函数
    Singleton& operator=(const Singleton&) = delete; 	//明确拒绝赋值函数
};

Singleton& Singleton::getInstance() {
    static Singleton singleton;
    return singleton;
}
```

#### 2、饿汉单例模式

饿汉：饿了肯定要饥不择食。所以在单例类定义的时候就进行实例化。**在访问量比较大，或者可能访问的线程比较多时，采用饿汉实现，可以实现更好的性能**。这是**以空间换时间**。

```cpp
//饿汉式：线程安全，注意一定要在合适的地方去delete它
class Singleton {
public:
    static Singleton* getInstance();					//静态函数
private:
    Singleton(){}                                    	//构造函数私有
    Singleton(const Singleton&) = delete;            	//明确拒绝拷贝构造函数
    Singleton& operator=(const Singleton&) = delete; 	//明确拒绝拷贝赋值函数
    static Singleton* m_pSingleton;						//静态成员
};

Singleton* Singleton::m_pSingleton = new Singleton();	//静态成员初始化

Singleton* Singleton::getInstance(){					//静态成员函数
    return m_pSingleton;
}
```

