









 c++中引入了`右值引用`和`移动语义`，可以避免无谓的复制、提高程序性能，同时能够简明定义泛型函数

#### 概述

右值引用（rvalue reference）是C++11中引入的新特性，主要用于解决两个方面的问题：

1. 实现move语义
2. 完美转发

这里我们先引出右值引用，并给出其主要的作用，但不急于给出其标准的定义。我们先从理解什么是**左值**（lvalue）和什么是**右值**（rvalue）来逐渐剖析什么是右值引用。

#### 一、左值和右值

在C语言代码中，左值和右值通常定义如下：

- 左值就是即可以出现在赋值表达式左边，也可以出现在赋值表达式右边的表达式e；
- 右值就是只能出现在赋值表达式右边的表达式。

举例来说：

```c
int a = 42;
int b = 43;
//a and b 均为左值l-values
a = b ; //ok
b = a ; //ok
a = a*b; //ok

a*b = 42; //error, a*b is an rvalue, can only appear on the right hand side of assignment
```

在C++中，由于用户定义类型的引入，导致上述定义稍有差错。因此，引入一个更加直接明了但是不甚标准的定义：

- 左值：指向一块内存，并且允许通过 & 操作符获取该内存地址的表达式；
- 右值：非左值的即为右值。

举例来说：

```c++
int i = 42;   //i 为左值
int *p = &i;  //ok
int& foo(){}

foo() = 42; 	//ok, foo() 为左值
int *q = &foo();	//ok, foo()为左值

int foobar(){}
int j = foobar();	//ok, foobar() 为右值
int *r = &foobar(); 	//erorr, foobar()为右值
```

#### 二、move 语义

假设有类A，A中持有一个数组，设为p：

```c++
class A {
public:
    A(int e):a(e) {
        cout << "ctor" << endl;
        p = new int[5]{e, e, e, e, e};
    }
    A(A &rhs);
    A& operator=(A &rhs);
private:
    int a = 0;
    int *p = nullptr;
};
```

则copy构造函数和赋值操作符重载操作为：

```c++
A(A &rhs):a(rhs.a){
    cout << "copy ctor" << endl;
    delete [] p;
    a = rhs.a;
    p = new int[5]{rhs.a};
}
A& operator=(A &rhs){
    cout << "operator=" <<endl;
    delete [] p;
    a = rhs.a;
    p = new int[5]{rhs.a};
    return *this;
}
```

则如下代码调用关系如下：

```c++
A a1(5);		//ctor
A a2(a1);		//copy ctor
A a3 = a1;		//copy ctor
A a4(10);		//ctor
a4 = a1;		//operator =
A foo(){
    A a(20);	//ctor
	return a;	//由于返回值优化，这里在调试器默认状态下无ctor调用
}
A x;			//ctor
x = foo();		//operator =
```

在第11行代码的调用中，在copy构造函数和赋值操作符重载中，主要执行如下操作：

1. 克隆foo()返回的临时值的资源；
2. 释放x原先持有的资源，并用克隆的资源替换
3. 析构临时值并释放其资源

显而易见，当A持有的资源拷贝时需要额外消耗大量资源时，对程序性能是影响巨大的。分析上述代码，可以引出如下问题：foo（）返回的临时值在将资源克隆给x后就要析构并释放，那是否可以直接到临时值的资源交给x，临时值析构时仅析构其自身，而不销毁持有的资源，如此，可以减少克隆拷贝操作，从而提高程序性能。

换句话说哦，当赋值操作的右边是一个**右值**，我们希望copy构造函数和赋值操作符重载执行逻辑如下：

```c++
delete this->resource;
this->resource = rhs.resource;
```

这就是**Move语义**。在C++11中，这种行为通过以下重载实现：

```c++
A(A &&rhs):a(rhs.a){
    cout << "copy ctor&&" << endl;
}
A& operator=(A &&rhs){
    cout << "operator=&&" <<endl;
}
```

通过重载，左值选择常规引用，右值选择move语义。

#### 三、右值引用

显而易见，所谓右值引用，就是不同于<A&>左值引用的引用，其表现形式为<A&&>。右值引用的行为类似于左值引用，但有几点显著区别，最重要的区别是：

当函数重载决议时，左值倾向于左值引用，右值倾向于右值引用：假设有如下代码：

```c++
void foo(A &a);		//lvalue reference overload
void foo(A &&a);	//rvalue reference overload
A a;
A foobar(){};
foo(a);				//argument is lvalue: calls foo(A&)
foo(foobar());		//argument is rvalue: calls foo(A&&)
```

所以右值引用的要义是

***右值引用允许函数在编译器通过重载决议来选择调用，基于条件左值调用还是右值调用***

> Rvalue references allow a function to branch at compile time (via overload resolution) on the condition “Am I being called on an lvalue or an rvalue?”

实质上，可以将任意函数实现为这种重载形式，但是在实际运用中，这种重载一般仅引用于拷贝构造函数和赋值操作符重载，以实现move语义。

但是有一点要注意的是，如果实现

```cpp
void foo(A&);
```

而不实现

```cpp
void foo(A&&);
```

则程序行为不发生任何变化，foo只能左值调用。

如果实现

```cpp
void foo(A const&);
```

而不实现

```cpp
void foo(A&&);
```

则程序行为也不发生变化，foo能同时被左值和右值调用，但是左值右值无任何语义区别。

如果仅实现

```cpp
void foo(A &&);
```

但是不实现

```cpp
void foo(A&);
void foo(A const&);
```

则foo仅能被右值调用，如果被左值调用会触发编译错误。

#### 四、强制move语义

C++11允许程序员不仅仅在右值上使用move语义，同样，允许程序员在左值上使用move语义。以标准库函数swap为例：

```c++
template<class T>
void swap(T& a, T& b) { 
  T tmp(a);
  a = b; 
  b = tmp; 
} 
A a, b;
swap(a, b);
```

在上例代码中，由于没有任何右值，所有代码均使用非move语义，但是在以下情况时使用move语义更有优势：

当一个变量作为拷贝构造函数或赋值操作符的源对象时

1. 该变量不会再被使用；
2. 该变量仅作为赋值的目标

C++11中，标准库函数std::move可以用来实现我们的目标，该函数仅仅将它的参数转换成右值，而不做其他任何操作，在C++11中，swap函数可以实现如下：

```c++
template<class T> 
void swap(T& a, T& b) { 
  T tmp(std::move(a));
  a = std::move(b); 
  b = std::move(tmp);
}

A a, b;
swap(a, b);
```

这样所有的代码都使用了move语义，而对那个没有实现move语义的类型，swap和以前一样工作。

可以在任何需要使用std::move的地方使用它，可以成程序带来如下好处：

1. 许多标准算法和操作优先使用move语义来提高程序性能；
2. STL经常需要特定类型的可拷贝性，但是在很多情况下，可移动性更常用。

#### 五、右值引用并不都是右值

假设有类X实现了move语义：

```c++
void foo(X&& x){
  X anotherX = x;
  // ...
}
```

这里有一个问题：X的哪一个拷贝构造函数重载被调用？x是有一个右值引用变量，很显然，我们希望这里调用move语义拷贝构造函数，但实际上，它调用的是传统拷贝构造函数。原因何在？

> Things that are declared as rvalue reference can be lvalues or rvalues. The distinguishing criterion is: *if it has a name*, then it is an lvalue. Otherwise, it is an rvalue.

变量被声明为右值引用既可以为左值也可以为右值，区别标准是：它是否有名字，有名字是左值；无名字，是右值。

在下例代码中，调用的move语义拷贝构造函数：

```c++
X&& goo();
X x = goo(); // calls X(X&& rhs) because the thing on
             // the right hand side has no name
```

这种现象背后的原因的是：如果允许一个有名字的变量实现move语义，那将是危险和易错的。因为，我们已经移动的变量在后面的代码仍然可访问。move语义的要点是：我们只在它“不重要“的地方应用它，从这个意义上说，我们移动过后它就消失了。因此，如果变量有名字，它就是个左值。

#### 六、move语义与编译器优化

考虑如下代码：

```c++
X foo(){
  X x;
  // perhaps do something to x
  return x;
}
```

如果按照字面理解，可能会说，从x到返回值有一个值拷贝发生，是否可以使用move语义替代：

```c++
X foo(){
  X x;
  // perhaps do something to x
  return std::move(x); // making it worse!
}
```

但是，这会让事件变的更糟，现代编译器使用返回值优化（RVO，*return value optimization*）。换句话说，程序原来的执行顺序为：

- 构造一个局部对象x
- 将x拷贝一份，并将拷贝值返回

使用RVO后，编译器直接在返回语句处直接构建一个x返回。

#### 七、完美转发

右值引用解决的另外一个问题就是完美转发问题，考虑一下代码：

```c++
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg arg){ 
  return shared_ptr<T>(new T(arg));
}
```

显而易见，函数将参数arg转发给T的构造函数。理想情况下，程序应当向factory不存在一样，构造函数被直接调用，这就是完美转发。但是上述代码的问题在于，它引入一个额外的值拷贝，当构造函数通过引用调用其参数时，情况将变得更加糟糕。

最常见的解决方法，是以引用方式传递参数：

```c++
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg& arg){ 
  return shared_ptr<T>(new T(arg));
} 
```

这并不是一个完美的解决方案，因为factory无法以右值作为其参数：

```cpp
factory<X>(hoo()); // error if hoo returns by value
factory<X>(41); // error
12
```

使用const reference可以解决上述问题

```c++
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg const & arg){ 
  return shared_ptr<T>(new T(arg));
} 
```

但是，这又有了新问题：

1. 如果函数有多个参数，需要重载所有参数的non-const和const引用的组合；
2. 这种转发阻止了move语义的应用，所有的参数都是左值，move语义将无法使用

这些问题，可以使用右值引用来解决。

在C++11之前，引用的引用**A & &**会产生编译错误，但是在C++中，引入了以下引用展开规则（*reference collapsing rules*）：

1. A& & = A&
2. A& && = A&
3. A&& & = A&
4. A&& && = A&&

这里有一个模板函数模板参数推导规则：使用右值引用作为模板参数：

```c++
template<typename T>
void foo(T&&);
```

1. 当foo被类型A的左值调用，T被解析成A&，根据RCR，参数类型等效为 A& && = A&
2. 当foo被类型A的右值调用，T被解析成A，根据RCR，参数类型等效为A&&

则使用右值引用来解决完美转发问题的解决方案如下：

```c++
template<typename T, typename Arg>
shared_ptr<T> factory(Arg&& arg){
    return shared_ptr<T>(new T(std::forward<Arg>(arg)));
}
```

std::forward定义如下：

```c++
template<class S>
S&& forward(typename remove_reference<S>::type& a) noexcept{
    return static_cast<S&&>(a);
}
```

假设有类型X和A，X为Arg的特例，A为T的特例

```c++
X x;
factory<A>(x);
```

根据上述模板推导规则，factory的模板参数Arg被解析成X&。因此，编译器将会创建factory和std::forward的实例：

```c++
shared_ptr<A> facotry(X& &&arg){
    return shared_ptr<A>(new A(std::forward<X&>(arg)));
}

X& && forward(remove_reference<X&>::type& a) noexcept{
    return static_cast<X& &&>(a);
}
```

评估remove_reference和应用RCR后，代码转换成：

```c++
shared_ptr<A> facotry(X& arg){
    return shared_ptr<A>(new A(std::forward<X&>(arg)));
}

X& forward(X& a) noexcept{
    return static_cast<X&>(a);
}
```

这对左值来说的确是完美转发，参数arg通过两次间接传递，均是通过传统的左值引用。

现在来看右值调用：

```c++
X foo(){}；
factory<A>(foo());
```

根据模板推导规则，Arg被解析成X，编译器创建以下函数实例：

```c++
shared_ptr<A> factory(X&& arg){ 
  return shared_ptr<A>(new A(std::forward<X>(arg)));
} 
X&& forward(X& a) noexcept{
  return static_cast<X&&>(a);
}
```

这对右值来说也是完美转发：参数arg通过两次间接传递，均是通过引用。A的构造函数将其参数视为一个右值引用并且没有名字。这意味着转发保持了move语义。

在该代码中，std::forward的唯一作为就是保持move语义。如果没有std::forward，代码仍然正常工作，除了A的拷贝构造函数将其参数视为有名字的左值。换句话说，std::forward的目的是在被调用处转发包装器认为的左值的或右值。