# 条款52：写了placement new也要写placement delete
### 正常的operator new和operator delete
```c++
void* operator new(std::size_t) throw(std::bad_alloc);
void operator delete(void* rawMemory) throw();//global 作用域中的正常签名式
void operator delete(void* rawMemory, std::size_t size) throw();//class作用域中的典型签名式。
```

### placement new和placement delete
#### placement new
如果**operator new接受的参数除了一定会有的那个size_t之外还有其他**，这个便是所谓的placement new。
众多placement new中特别有一个是“接受一个指针指向对象该被构造之处”，像这样：
```c++
void* operator new(std::size_t, void* pMemory) throw();
```
**placement new也可指这一特定版本（大多数情况下说的是这个）**。也正如其名字：一个特定位置上的new。
这个版本已经被纳入c++标准库，只要#include <new>就可以取用它。这个new的用途之一是负责在vector的未使用空间上创建对象。
#### placement delete
接受的参数的就叫placement delete。
### operator new和operator delete的对应
如果一个带额外参数的operator new没有“带相同额外参数”的对应版operator delete，那么**当new的内存分配动作需要取消并恢复时就没有任何operator delete会被调用**。
所以，**operator new和operator delete要对应**。
```c++
class Widget{
public:
    static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
    static void* operator delete(void* pMemory, std::size_t size) throw();
    static void* operator delete(void* pMemory, std::ostream& logStream) throw();
};
```
如果Widget构造函数抛出异常，对应的placement delete会被自动调用，很好：
```c++
Widget* pw = new (std::cerr) Widget;
```
而如果没有抛出异常，用户想要delete时，则会调用正常的operator delete：
```c++
delete pw;  //调用正常的operator delete
```
**placement delete只有在“伴随placement new调用而触发的构造函数”出现异常时才会被调用。对一个指针施行delete绝不会导致调用placement delete。
这意味对所有placement new我们必须同时提供一个正常的delete和一个placement版本。**
### 避免遮挡
**成员函数的名称会掩盖其外围作用域中的相同名称**，你必须小心让class专属的news掩盖客户期望的其他news（包括正常版本）。
如果你有一个base class，其中声明唯一一个placement operator new，客户会发现他们无法使用正常形式的new：
```c++
class Base{
public:
    static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);//遮掩global版本
};
Base* pb = new Base;//错误，正常形式的operator new被掩盖
Base* pb = new (std::cerr) Base;//正确，调用placement new
```
同样，derived classes中的operator new会掩盖global版本和继承而得的operator new版本：
```c++
class Derived : public Base{
public:
    static void* operator new(std::size_t size)throw(std::bad_alloc); //重新声明正常形式的new
};
Derived* pd = new (std::cerr) Derived;//错误，Base的placement new被掩盖了
Derived* pd = new Derived; //正确
```
缺省情况下c++在global作用域内提供以下形式的operator new：
```c++
void* operator new(std::size_t) throw(std::bad_alloc);            //normal new
void* operator new(std::size_t, void*) throw();                        //placement new
void* operator new(std::size_t, const std::nothrow_t&) throw();// nothrow new
```
如果你在class内声明任何operator news，它会遮掩上述这些标准形式。除非你的意思就是要阻止class的客户使用这些形式，否则确保他们在你生成的任何定制型operator new之外还可用。
一个简单的做法是，建立一个base class，内含所有正常形式的new和delete：
```c++
class StandardNewDeleteForms{
public:
    //normal new/delete    
    static void* operator new(std::size_t size)throw(std::bad_alloc)
    {return ::operator new(size);}
    static void operator delete(void* pMemory) throw()
    {::operator delete(pMemory);}
    //placement new/delete    
    static void*  operator new(std::size_t size, void* ptr) throw()
    {return ::operator new(size, ptr);}
    static void operator delete(void* pMemory, void* ptr)throw()
    {return ::operator delete(pMemory, ptr);}
    //nothrow new/delete
    static void* operator new(std::size_t size, const std::nothrow_t& nt)throw()
    {return ::operator new(size, nt);}
    static void operator delete(void* pMemory, const std::nothrow_t&) throw()
    {::operator delete(pMemory);}
};
```
凡是想自定义形式扩充标准形式的客户，可利用继承机制及using声明式取得标准形式：
```c++
class Widget: public StandardNewDeleteForms{
public:
    using StandardNewDeleteForms::operator new;
    using StandardNewDeleteForms::operator delete;
    static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
    static void operator delete(void* pMemory, std::ostream& logStream) throw();
};
```