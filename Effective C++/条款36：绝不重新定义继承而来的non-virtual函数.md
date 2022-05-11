# 条款36：绝不重新定义继承而来的non-virtual函数
### 绝不重新定义继承而来的non-virtual函数
```c++
class B {
public:
	void mf();
};
class D: public {
public:
	void mf();
};
D x;
B* pB = &x;
pB->mf();     //调用B::mf
D* pD = &x;
pD->mf();    //调用D::mf
```
两个对象是一样的，只是因为指向他们的指针不同就调用了不同的函数，这种行为是不符合预期的！
造成这种行为的原因是，**non-virtual函数是静态绑定**。
由于pB被声明为一个pointer-to-B，通过pB调用的non-virtual函数永远是B所定义的版本，即使pB指向一个类型为“B派生之class”的对象。
（virtual函数是动态绑定）