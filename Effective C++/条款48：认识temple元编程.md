# 条款48：认识temple元编程
### 模板元编程
**Template metaprogramming(TMP，模板元编程）** 是编写template_based C++程序并执行于编译期的过程。所谓的TMP，就是以C++写成，执行于C++编译器内的程序。一旦TMP程序结束执行，其输出，也就是从template具现出来的若干C++源码，便会一如往常地被编译。
* 优点：可将工作由运行期移往编译期，得以实现早期错误侦测和更高的执行效率。
* 缺点：编译时间变长了。

条款47中有例子，应采用traits-based TMP解法，而不是typeid-based解法。
* typeid-based会导致编译期问题。**编译器必须保证所有代码可行，即使是永远不会执行起来的代码。**
* traits-based TMP针对不同的类型进行的代码被拆分成了不同的函数，每个函数所使用的操作都可施行于该函数所对付的类型。
### TMP示范
```c++
template<unsigned n>
struct Factorial{
    enum{value=n*Factorial<n-1>::value};
};
template<>
struct Factorial<0>{
    enum{value=1};
};
//你可以这样使用Factorial
int main()
{
    std::cout<<Factorial<5>::value;//印出120
    std::cout<<Factorial<10>::value;//印出3628800
}
```
### TMP目标
* 确保量度单位正确。使用TMP可以确保在编译期程序中所有量度单位的组合都正确，不论计算多么复杂，这也是为什么TMP可以被用来进行早期错误侦测。
* 优化矩阵运算。考虑以下代码：
```c++
typedef SquareMatrix<double,10000> BigMatrix;
BigMatrix m1,m2,m3,m4,m5;
...
BigMatrix result=m1*m2*m3*m4*m5;
```
以“正常的函数”调用动作来计算result，会创建4个暂时性矩阵，每一个用来存储operator* 的调用结果。如果使用与TMP相关的template技术，就有可能消除那些临时对象并合并循环，这一切都无需改变客户端的用法
* 可以生成客户定制的设计模式实现品。TMP可被用来生成“基于政策选择组合”的客户定制代码，也可用来避免生成某些特殊类型并不适合的代码。
