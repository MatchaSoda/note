# 条款21：必须返回对象时，别妄想返回其reference
### pass-by-value
正确写法
```c++
class Rational{
public:
    Rational(int numerator = 0, int denominator =1);
private:
    int n, d;
    friend const Rational operator* (const Rational& lhs, const Rational& rhs);
};
//返回一个新对象，这样就好
inline const Rational operator* (const Rational& lhs, const Rational& rhs){
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```
reference是个名称，指向的是既有对象。例如在上述代码中，执行`Rational c = a * b;`的时候，a * b的结果对象肯定是在operator函数内生成的，而不是已经有了的。
### pass-by-reference
**下边是三种会出问题的方法**
#### local stack
在stack空间内创造，不能避免构造函数的调用，并且有更严重的问题：函数退出前local对象就被销毁了，返回其引用就变成了未定义行为。
```c++
const Rational& operator* (const Rational& lhs, const Rational& rhs){
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
    return result;
}
```
#### heap-allocated
使用new在heap空间创造，也不能避免构造函数的调用，还易引发内存泄漏：需要delete，而在`w = x * y * z;`这种语句中无法准确delete。
```c++
const Rational& operator* (const Rational& lhs, const Rational& rhs){
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```
#### local static
使用static可以避免构造函数的调用。但是会影响多线程的安全性。
```c++
const Rational& operator* (const Rational& lhs, const Rational& rhs){
    static Rational result; 
    result = ...;
    return *result;
}
```
更致命的缺点是执行`if((a * b) == (c * d))  { ... ;}`这样的代码时，因为都是operator * 内的static Rational对象值，判断的结果会永远相等。