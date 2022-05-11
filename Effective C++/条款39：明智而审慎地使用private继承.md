# 条款39：明智而审慎地使用private继承
### private继承规则
* 如果class之间的继承关系是private，编译器不会自动将一个derived class对象转换为一个base class对象。
```c++
class Person{...};
class Student :private Person {...};
void eat(const Person& p);
Person p;
Student s;
eat(p);//正确
eat(s);//错误
```
* 由private base class继承而来的所有成员，在derived class都会变成private属性，纵使它们在base class中原本是protected或public属性。
### private继承意义
* private继承意味着is-implemented-in-terms-of(根据某物实现出)。
* private继承意味只有实现部分被继承，接口部分应略去。
* private继承只在软件实现层面有意义，在“设计”层面没有意义。

**如果D以private形式继承B，意思是D对象根据B对象实现而得，再没有其他意涵了。**
### private继承和复合
private继承意味着is-implemented-in-terms-of(根据某物实现出)，这和复合很像。
**尽可能使用复合，必要时才使用private继承。**
必要时：
* 需要访问protected成员
* 需要重新定义一个或多个virtual函数
* 涉及空间最优化（特殊）
***
现在有个Widget class，我们想记录每个成员函数调用次数，在运行期间周期性审查这份信息。为了完成这项工作，需要用到定时器：
```c++
class Timer{
public:
    explicit Timer(int tickFrequency);
    virtual void OnTick() const;//定时器滴答一次，此函数被自动调用一次
    ……
};
```
使用**private继承**实现Widget：
```c++
class Widget: private Timer{
private:
    virtual void onTick() const;//查看Widget的数据等操作
    ……
};
```
使用**复合**实现Widget：
```c++
class Widget{
private:
    class WidgetTimer: public Timer{
    public:
        virtual void onTick() const;
        ……
    };
    WidgetTimer timer;
    ……
}；
```
这里应使用public继承+复合的方式，而不是private继承。
* Widget可能会有派生类，但是我们可能会想**阻止派生类重新定义onTick函数**。
如果是使用private继承，上面的想法就不能实现，因为derived classes可以重新定义virtual函数（即使是private函数）。
如果采用复合方案，因为这里的WidgetTimer是private，Widget的derived classes将无法采用WidgetTimer，自然也就无法继承或重新定义它的virtual函数了。
* 采用复合方案，还可以降低编译依存性。
如果Widget继承Timer，当Widget编译时Timer的定义必须可见，所以Widget所在的定义文件必须`#include Timer.h`。
复合方案可以将WidgetTimer移出Widget，而只含有一个指针即可。
***
特殊情况，在追求空间最优化的时候，使用private继承。
**class不带任何数据**。它不包含non-static变量、virtual函数，没有继承virtual base class。
这样的empty classes对象没使用任何空间，因为它没有任何数据对象要存储。但是因为技术原因，C++对象都必须有非零大小。
```c++
class Empty{}；
class HoldsAnInt{
private:
    int x;
    Empty e;//应该不需要任何内存
};
```
但是，**sizeof(HoldsAnInt)>sizeof(int)** 。
大多数编译器中，sizeof(Empty)为1。因为面对“大小为零之独立对象”，通常C++默默安插一个char到对象内。并且根据**齐位要求**，这个大小还可能被放大到一个int。
empty class并不是真的empty。它们内往往含有**typedef、enum、static或弄-virtual函数**。SLT有许多技术用途的empty classes，其中内含有的成员（通常是typedefs），包括base classes unary_function和binary_function，这些是“用户自定义之函数对象”通常会继承的classes，也不会增加derived class的大小。
### 问题
第一个例子，意思是
重新定义一个或多个virtual函数的情况，可以使用public继承+复合的方式解决吗。
那是不是protected也可以了。