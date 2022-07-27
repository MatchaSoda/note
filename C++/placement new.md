# placement new
### 含义
placement new可以实现不分配内存，只调用构造函数。
```cpp
void *operator new( size_t, void *p ) throw()     { return p; }
```
placement new的执行忽略了size_t参数，只返还第二个参数。  
其结果是允许用户把一个对象放到一个特定的地方，达到调用构造函数的效果。
### 区别
* new 分配内存，调用构造函数。 
* operator new 只包含内存分配的一部分，被new调用。
* placement new 在已分配的缓冲区，调用构造函数，是operator new的一种特殊实现。
### 应用
new 会调用 operator new 来分配内存，而 placement new 是一种  operator new 的特殊实现，不需要实际的分配内存而是采用已分配的内存（当然，这需要程序员提前手动分配）。  
程序员提前分配好一个缓冲区用来存，而不是让new自己去找（需要找到一块正确大小的空间，也可能执行很多次），这样一来便可以增加时间空间效率。也可以保证不会出现内存不足而产生异常或中断。