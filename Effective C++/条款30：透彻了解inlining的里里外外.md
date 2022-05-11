# 条款30：透彻了解inlining的里里外外
### inline优缺点
#### 优点

* 看起来像函数
* 动作像函数
* 比宏好很多
* 可以调用它们又不需要蒙受函数调用所招致的额外开销
* 编译器最优化机制通常被设计用来浓缩那些“不含函数调用”的代码，所以当你inline某个函数，或许编译器就因此——有能力对它（函数本体）执行语境相关最优化。大部分的编译器不会对一个“outlined函数调用”动作执行如此之最优化
#### 缺点
* 可能增加目标码（object code）的大小
* inline造成的代码膨胀会导致额外的换页行为，降低指令告诉缓存装置的击中率，以及伴随这些而来的效率损失
### inline是个申请
**inline只是对编译器的一个申请，不是强制命令！**
#### 隐喻的提出申请
将函数定义于class定义式内：
（friend函数也可以定义于class内，所以它们也是被隐喻声明为 inline）
```c++
class Person  {
public:
    ...
    // 一个隐喻的inline申请：age被定义于class定义式内
    int age() const  {  return theAge;  }
    ...
private:
    int theAge;
};
```
#### 明确的提出申请
在其定义式前加上关键字inline：
```c++
template<typename T>
inline const T& std::max(const T& a,const T& b)
{  return a < b ? b : a;  }
```
### inline and template
inline函数 和 template 两者通常都被定义在头文件内，这使得某些程序员认为 function template 一定必须是inline，但其实他们俩没关系:
* Inline函数通常一定被置于头文件内，因为大多数建置环境在编译过程中进行inlining，而为了将一个“函数调用”替换为“被调用函数的本体”，编译器必须知道那个函数长什么样子。
* Template 通常也被置于头文件内，因为它一旦被使用，编译器为了将它具现化，需要知道它长什么样子。

Template的具现化与inlining无关。如果你写的template没有理由要求它所具现的每一个函数都是inlined，就应该避免将这个template声明为inline。
因为inline需要成本，在缺点中提到过代码膨胀，但还存在其他的成本。
### 于编译器角度看 inline
**一个表面上看似inline的函数是否真是inline，取决于你的建置环境，主要取决于编译器。**
幸运的一件事——大多数编译器提供了一个诊断级别：如果它们无法将你要求的函数inline化，会给一个warning。
* 大部分编译器拒绝将太过复杂（含有循环或递归）的函数inlining，而所有对virtual函数的调用也都会使inline落空。
因为virtual意味着“等待，直到运行期才确定调用哪个函数”，而inline意味着“执行前，先将调用动作替换为被调用函数的本体”。编译器根本不知道你调用哪个函数，如何替换。
* 有时候虽然编译器有意愿inlining某个函数，还是可能为该函数生成一个函数本体。
比如，如果程序要取某个inline函数的地址，编译器通常必须为此函数生成一个 outlined 函数本体。毕竟编译器无法让一个指针指向并不存在的函数。
* 编译器通常不对“通过函数指针而进行的调用”实施inlining。
这意味对inline函数的调用可能被inlined，也可能不会，取决于该调用的实施方式：
```c++
inline void f()  {  ...  }  // 假设编译器有意愿inline“对f的调用”
void (*pf)() = f;   // pf 指向 f
...
f();        // 这个调用将被inlined，因为他是一个正常调用
pf();      // 这个调用或许不被inlined，因为它通过函数指针达成
```
* 构造函数和析构函数往往是inlining的糟糕候选人。
```c++
class Base  {
public:
    ...
private:
    std::string bm1,bm2;
};
class Derived : public Base  {
public:
    Derived()  {  }
    ...
private:
    std::string dm1,dm2,dm3;
};
```
表面上构造函数为空，但是他要自动构造base class及每一个成员变量，同时要考虑其是否抛出异常，若有异常还要析构已经构造的函数，实际上有很多步骤要做。
### 问题
p135为什么编译器必须知道那个函数长什么样子就要放在头文件。