策略模式是指定义一系列的算法，把它们单独封装起来，并且使它们可以互相替换，使得算法可以独立于使用它的客户端而变化，也是说**这些算法所完成的功能类型是一样的**，**对外接口也是一样的**，**只是不同的策略为引起环境角色环境角色表现出不同的行为**。

相比于使用大量的if...else，使用策略模式可以降低复杂度，使得代码更容易维护。

缺点：可能需要定义大量的策略类，并且这些策略类都要提供给客户端。

[环境角色] 持有一个策略类的引用，最终给客户端调用。

#### 1、传统的策略模式实现

```cpp
/*
* 关键代码：实现同一个接口。
* 以下代码实例中，以游戏角色不同的攻击方式为不同的策略，游戏角色即为执行不同策略的环境角色。
*/
#include <iostream>
using namespace std;
//抽象策略类，提供一个接口
class Hurt{
public:
    virtual void blood() = 0;
};
//具体的策略实现类，具体实现接口， Adc持续普通攻击
class AdcHurt : public Hurt{
public:
    void blood() override {
        cout << "Adc hurt, Blood loss" << endl;
    }
};
//具体的策略实现类，具体实现接口， Apc技能攻击
class ApcHurt : public Hurt{
public:
    void blood() override {
        cout << "Apc Hurt, Blood loss" << endl;
    }
};

//环境角色类， 游戏角色战士，传入一个策略类指针参数。
class Soldier{
public:
    Soldier(Hurt* hurt):m_pHurt(hurt) { }	//构造函数
    //在不同的策略下，该游戏角色表现出不同的攻击
    void attack() {
        m_pHurt->blood();
    }
private:
    Hurt* m_pHurt;	//成员变量
};

//定义策略标签
typedef enum {
    Hurt_Type_Adc,
    Hurt_Type_Apc,
}HurtType;
//环境角色类， 游戏角色法师，传入一个策略标签参数。
class Mage {
public:
    Mage(HurtType type) {	//构造函数
        switch(type) {
            case Hurt_Type_Adc:
                m_pHurt = new AdcHurt();
                break;
            case Hurt_Type_Apc:
                m_pHurt = new ApcHurt();
                break;
            default:
                break;
        }
    }
    ~Mage() {
        delete m_pHurt;
        m_pHurt = nullptr;
        cout << "~Mage()" << endl;
    }
    void attack()
    {
        m_pHurt->blood();
    }
private:
    Hurt* m_pHurt;
};
//环境角色类， 游戏角色弓箭手，实现模板传递策略。
template<typename T>
class Archer {
public:
    void attack() {
        m_hurt.blood();
    }
private:
    T m_hurt;
};
int main() {
    Archer<ApcHurt>* arc = new Archer<ApcHurt>;
    arc->attack();

    delete arc;
    arc = nullptr;

    return 0;
}
```

#### 2、使用函数指针实现策略模式

```cpp
#include <iostream>
#include <functional> 

void adcHurt() {
    std::cout << "Adc Hurt" << std::endl;
}
void apcHurt() {
    std::cout << "Apc Hurt" << std::endl;
}
//环境角色类， 使用传统的函数指针
class Soldier {
public:
    typedef void (*Function)();
    Soldier(Function fun): m_fun(fun) { }
    void attack() {
        m_fun();
    }
private:
    Function m_fun;
};

//环境角色类， 使用std::function<>
class Mage{
public:
    typedef std::function<void()> Function;
    Mage(Function fun): m_fun(fun) { }
    void attack() {
        m_fun();
    }
private:
    Function m_fun;
};
int main() {
    Soldier* soldier = new Soldier(apcHurt);
    soldier->attack();
    delete soldier;
    soldier = nullptr;
    return 0;
}
```

