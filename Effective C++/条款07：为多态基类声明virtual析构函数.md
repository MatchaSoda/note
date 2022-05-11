# 条款07：为多态基类声明virtual析构函数
### 带有多态性质的base classes，或class带有任何virtual函数，它也应该拥有一个virtual析构函数。
当derived class对象经由一个base class指针被删除，而该base class没有使用virtual析构函数，就会引发内存泄漏，因为对象的derived部分没有被销毁。如下：
```c++
class derived : public base{};
derived* p = new derived();
base* pp;
pp = p;
delete pp;
```
需要一个abstract classes（抽象类，不能被实例化）时，声明pure virtual析构函数是个很好的选择。注意为其提供一个定义。
### 不作为base classes使用或者不具有多态性，就不该声明virtual析构函数。
**virtual析构函数也不能盲目使用，因为使用virtual内部会生成virtual table以及virtual table pointer，用来在运行期决定调用那个virtual函数。**
不作为base classes不用，有的作为base classes而不是为了实现多态，比如条款6的Uncopyable，也不用。
### 问题
复习构造析构的顺序
p43为什么纯虚函数要提供定义。