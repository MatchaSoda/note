# unsigned int
## int 与 unsigned int 比较
int 与 unsigned int 比较时，int 会被转换为 unsigned int 。两者都是正数时没有问题，但是 int 值为负数时，就会有：
```cpp
#include <iostream>
using namespace std;
int main() {
  int x = -1;
  unsigned int y = 2;
  cout << (unsigned int)x << endl;
  if (x > y) {
    cout << x << " is bigger than " << y;
  } else {
    cout << y << " is bigger than " << x;
  }
}
```
输出为
```
4294967295
-1 is bigger than 2
```
我们一般不会像上面那样比较。但是在用 vector 的时候，vector 的`size()`返回的是 unsigned int，就可能会出现上述的比较。
```cpp
vector<int> nums;
for(i = -1; i < nums.size(); ++i){
//有时会把i初始化为负数，这时与nums.size()比较就会出问题
}
```
所以我们通常会使用一个变量先记录vector的长度。
## unsigned int 不能小于0
```cpp
int main() {
  unsigned int a = 0;
  cout << a - 1;
}
```
会输出
```
4294967295
```
虽然平时不会这么用，但这可能会在不经意间发生。
```cpp
vector<int> nums;
for(i = 0; i < nums.size() - 1; ++i){
//与nums.size()做运算返回还是unsigned int,当nums为空时nums.size() - 1就会出问题
}
```
vector 的`size()`返回的是unsigned int，所以我们通常会使用一个变量先记录vector的长度。
## 参考
https://blog.csdn.net/qingtu11/article/details/115859653