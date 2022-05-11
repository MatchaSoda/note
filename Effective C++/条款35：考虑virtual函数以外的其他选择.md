# 条款35：考虑virtual函数以外的其他选择

假如在设计一个游戏类的继承体系时,提供一个成员函数healthValue,返回一个整数,表示人物的健康程度.由于不同的人物可能以不同的方式计算健康程度,将healthValue声明为 virtual 似乎是再明白不过的做法:
```c++
class GameCharacter {
public:
    virtual int healthValue() const;    // 返回人物的健康程度
    ...
};
```
### non-virtual interface
使用**non-virtual interface(NVI)** 手法，那是Template Method设计模式的一种特殊形式。它**以public non-virtual成员函数调用一个private virtual函数**。
```c++
class GameCharacter {
public:
    int healthValue() const {
        ...                             // 做一些事前工作
        int retVal = doHealthValue();   // 做真正的工作
        ...                             // 做一些事后工作
        return retVal;
    }
private:
    virtual int doHealthValue() const {
        ...
    }
};
```
此处non-virtual函数称为virtual函数的**外覆器**（wrapper）。
这种写法的优点是确保在一个virtual函数被调用之前设定好适当场景，并在结束之后清理现场。
这里的private也可以是**protected**，在某些derived class的实现需要调用其base class兄弟的时候。
### 将virtual函数替换为“函数指针成员变量”

藉由Function Pointers实现Strategy模式。
```c++
class GameCharacter;        // 前置声明
// 以下函数是计算健康程度的缺省算法
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf)
    {}
    int healthValue() const 
    { return healthFunc(*this); }
    ...
private:
    HealthCalcFunc healthFunc;
};
``` 
有两个好处：
* 同一人物类型的不同实体可以有不同的健康计算函数。
例如：
```c++
class EviBadGuy: public GameCharacter {
public:
	explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc):
	GameCharater(hcf){...}
	...
};
int loseHealthQuickly(const GameCharacter&);  //健康值计算函数1
int loseHealthSlowly(const GameCharacter&);   //函数2

EvilBadGuy ebdg1(loseHealthQuickly);
EvilBadGuy ebd2(loseHealthSlowly);

```
* 某已知人物的健康程度计算函数可以在运行期变更。
例如GameCharacter可提供一个成员函数SetHeathCalculator，用来替换当前的健康值计算函数。

因为计算健康程度的函数已经不是继承体系内的成员函数，这也就有了缺点：他不能访问私有成员，如果要访问，就要通过friend或者public访问函数，降低了封装性。
### 以trl::function成员变量替换virtual函数
```c++
class GameCharacter;//前置声明
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    /*HealthCalcFunc可以是任何“可调用物”，可被调用并接受任何兼容于GameCharacter的物，
    返回任何兼容于int的东西*/
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) : healthFunc(hcf) {}
    int healthValue() const {
        return healthFunc(*this);
    }
private:
    HealthCalcFunc healthFunc;//此处声明healthFunc函数
};
```
function<int (const GameCharacter&)>中的签名式为int (const GameCharacter&）表示接受一个reference指向const GameCharacter，并返回int。
此function类型（也就是所定义的HealthCalcFunc类型）产生的对象可以持有（保存）任何与此签名式兼容的可调用物。所谓**兼容**，即是这个可调用物的参数可被**隐式转换**为const GameCharacter&，而其返回类型可被**隐式转换**为int。
### 古典的Strategy模式
```c++
class GameCharacter;
class HealthCalcFunc {
public:
	...
	virtual int calc(const GameCharacter& gc) const
	{...}
	...
};
HealthCalcFunc defaultHealthCalc;
class GameCharacter {
public:
	explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc): pHeathCalc(phcf) {}
	int healthValue() const
	{return pHealthCalc->calc(*this);}
	...
private:
	HealthCalcFunc* pHealthCalc;
};
```
GameCharacter有一个继承体系，而HealthCalcFunc有另一个继承体系。每一个GameCharacter对象都内含一个指针指向一个来自HealthCalcFunc体系内的对象。