# 条款03：尽可能使用const
### const和指针
```c++
char test[] = "Hello";
char* p = test;                     //non-const pointer,non-const data
const char* p = test;           //non-cosnt pointer,const data
char* const p = test;           //const pointer,non-const data
const char* const p = test; //const pointer,const data
```
其中，放在星号左边 都是 指向一个常量，``const char* p``和``char const * p`` 是一样的。
### const和迭代器
```c++
const iterator iter = vec.begin();  //T* const  指针固定
const_iterator iter = vec.begin();  //const T*  数据固定
```

### const和成员函数
const和non-const函数可以重载，为了避免代码重复，通常（也必须）让non-const调用const版，这样才能保证安全，使用时要用到cast转型，先转为const再调用再去掉const。
编译器执行bitwise constness，但使用时要用logical constness（合乎情理的常量性）。有的时候需要将成员变量定义为mutable，即可在const成员函数内修改。