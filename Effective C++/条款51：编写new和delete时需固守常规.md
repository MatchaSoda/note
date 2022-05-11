# 条款51：编写new和delete时需固守常规
### operator new
* 如果有能力供应客户申请的内存，便返回一个指针指向那块内存。内存不足时必须调用new-handler函数。实在没有能力便抛出bad_alloc异常
* operator new应该内含一个无穷循环，并在其中尝试分配内存
* 必须有对付零内存需求的准备
* 需避免不慎掩盖正常形式的new
#### non-member operator new
non-member operator new伪码：
```c++
void* operator new(std::size_t size) throw(std::bad_alloc)//可能接受额外参数
{
    using namespace std;
    if(size==0){
        size=1;
    }
    while(true)
    {
        尝试分配size bytes;
        if(分配成功）
        return（一个指针，指向分配得来的内存）；
        //分配失败，找出目前的new-handler函数
        new_handler globalHandler=set_new_handler(0);
        set_new_handler(globalHandler);
        if(globalHandler)  (*globalHandler)();
        else throw std::bad_alloc();
    }
}
```
* 即使客户要求0bytes，operator new也要返回一个合法指针。这里会把0bytes的申请量视为1bytes申请量。
* 没有任何办法直接取得new_handling函数指针，所以必须调用set_new_handler找它出来。
* operator new有个无穷循环，退出次循环的唯一办法是：内存成功分配或new_handling函数做了一件描述于**条款49**的事情：让更多内存可用、安装另一个new_handler、卸除new-handler、抛出bad_alloc异常、或者承认失败而直接return。
#### member operator new
operator new成员函数可能会被derived classes继承。如果针对class X而设计的operator new，只为大小为sizeof（X)的对象而设计。一旦被继承下去，base class的operator new被调用用以分配derived class对象：
```c++
class Base{
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
};
class Derived : public Base{ ... };
Derived* p = new Derived;        //调用base::operator new
```
这往往是不符合预期的。这时候（内存申请量错误的调用）最好的做法是改用标准operator new：
```c++
void* Base::operator new(std::size_t size) throw(std::bad_alloc)
{
    if (size != sizeof(Base))
        return ::operator new(size);
    ...
}
```
需注意c++的独立（非附属）对象必须有非零大小（条款39）。因为sizeof(Base)一定不等于0，所以这里等于零的检测就已经包含进去了。
#### operator new[]
对于operator new[]，唯一要做的一件事就是分配一块未加工内存。因为你无法对array之内迄今尚未存在的元素对象做任何事情。因为不知道每个对象多大（涉及继承），也无法计算那个array将含多少个元素对象。
### operator delete
#### non-member operator delete
**c++保证“删除null指针永远安全”**。
non-member operator delete的伪码：
```c++
void operator delete (void *rawMemory) throw(){
    if (rawMemory == 0) return;
    现在，归还rawMemory所指内存
}
```
#### member operator delete
只是多出了检测删除大小。即然大小不一样要交给::operator new来分配，那不一样的删除也要交由::operator delete来执行。
```c++
class Base{
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void operator delete(void* rawMemory, std::size_t size) throw();
};
void Base::operator delete(void* rawMemory, std::size_t size) throw()
{
    if (rawMemory == 0)return;
    if(size != sizeof(Base)){
        ::operator delete(rawMemory);
        return;
    }
    现在，归还rawMemory所指的内存；
    return;
}
```
### 问题
operator new和条款49的联系。“或者承认失败而直接return。”