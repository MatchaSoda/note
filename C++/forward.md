# forward
std::forward 用于**转发左值为左值或右值**，对于转发引用，std::forward将参数以**在传递给调用方函数时的值类别**转发给另一个函数。如下：
```cpp
template<class T>
void wrapper(T&& arg) 
{
    // arg 始终是左值
    foo(std::forward<T>(arg)); // 转发为左值或右值，依赖于 T
}
```
* 若对 wrapper() 的调用传递右值 std::string ，则推导 T 为 std::string （非 std::string& 或 std::string&& ，且 std::forward 确保将右值引用传递给 foo 。
* 若对 wrapper() 的调用传递 const 左值 std::string ，则推导 T 为 const std::string& ，且 std::forward 确保将 const 左值引用传递给 foo 。
* 若对 wrapper() 的调用传递非 const 左值 std::string ，则推导 T 为 std::string& ，且 std::forward 确保将非 const 左值引用传递给 foo 。
### 总结
右值引用为形参时，不论传进来的是左值还是右值，**实参都是左值**，再进行函数调用的时候将会出现预期之外的函数匹配，所以这时候可以用`std::forward`来将他们转发为实际的左值或是右值。

注意：*std::forward从**模板参数类型T**中得知是左值还是右值*

### 例子
```cpp
#include <iostream>
#include <memory>
using namespace std;
template <typename T>
void print(T& t) {
  cout << "lvalue" << endl;
}
template <typename T>
void print(T&& t) {
  cout << "rvalue" << endl;
}

template <typename T>
void TestForward(T&& v) {
  print(v);
  print(std::forward<T>(v));
  print(std::move(v));
  cout << endl;
}

int main() {
  TestForward(1);
  int x = 1;
  TestForward(x);
  TestForward(std::forward<int>(x));
  return 0;
}
```
结果为
```
lvalue
rvalue
rvalue

lvalue
lvalue
rvalue

lvalue
rvalue
rvalue
```