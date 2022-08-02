# std::move
``std::move()``会将左值强制转换为右值，等价于到右值引用类型的 static_cast ，即 ``static_cast<T&&>`` 。  
move 并不能移动什么东西，他的功能是强制类型转换，那为什么要叫 move 呢？因为他通常用于移动语义，这样会使代码更容易阅读。  
在 move 最开始提出的时候，为了使用**移动语义**编写一个更加有效的``std::swap()``，引入了右值引用，这样交换的时候便不需要进行深拷贝，而是简单的移动，提升了时间空间效率:
```cpp
template <class T>
void swap(T& a, T& b) {
  T tmp(static_cast<T&&>(a));
  a = static_cast<T&&>(b);
  b = static_cast<T&&>(tmp);
}
```
然后引入了语法糖，让代码看起来更已读，这个函数传达的不是他做了什么（static_cast），而是他这么做的原因(需要move)：
```cpp
template <class T>
void swap(T& a, T& b) {
  T tmp(move(a));
  a = move(b);
  b = move(tmp);
}
```
正如 move 的意思一样，他之所以效率更高是因为使用了移动语义，移动后原来的对象将不再存在。但若只是 cast 而不实际的 move ，原来的对象就还在。
```cpp
#include <iostream>
#include <memory>
#include <string>
using namespace std;
int main() {
  string a = "hello";
  cout << "string a : " << a << endl;
  move(a);
  cout << "string a : " << a << endl;
  string b = move(a);
  cout << "string a : " << a << endl;
  cout << "string b : " << b << endl;
}
```
运行结果为：
```
string a : hello
string a : hello
string a :
string b : hello
```  
