# 条款20：宁以pass-by-reference-to-const替换pass-by-value
### 通常pass-by-reference-to-const更高效，并且可避免函数切割
pass-by-value会招致很多不必要的构造和析构，传引用便可以节约这些步骤，设置为const防止值被修改。
pass-by-reference-to-const也可以避免函数切割。
```c++
class Base{};
class Derived : public Base {};
void test(Base t){
    ...
}
Derived d;
test(d);
```
这种情况就会导致函数切割，实参被传入时，只构造了一个base版本的对象，derived部分就会被切割掉。
使用`void test(const Base& t){}`可以避免切割。
### 对于内置类型、stl迭代器、函数对象不适用
### 问题
stl迭代器、函数对象作为参数传递的时候是什么样的