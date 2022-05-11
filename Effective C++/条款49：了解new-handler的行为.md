# 条款49：了解new-handler的行为
 ### new-handler
当operator new抛出异常以反映一个未获满足的内存需求之前，他会先调用一个客户指定的错误处理函数，即new-handler。
为了指定这个“用以处理内存不足”的函数，客户必须调用set_new_handler。他是声明于<new>的一个标准程序库函数：
```c++
namespace std {
    typedef void(*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```
new_handler是一个typedef，它是一个函数指针，没有参数和返回值。
set_new_handler的参数指向当无法分配足够内存时调用的函数，返回值指向替换前的那个旧的new_handler。
用起来像这样：
```c++
//outOfMem是无法分配足够内存时，应该被调用的函数
void outOfMem(){
    std::cerr<<"Unable to satisfy  request for mempry/n"<<endl;
    std::abort();
}
int main(){
    std::set_new_handler(outOfMem);
    int* pBigDataArray=new int[1000000000];
    ...
}
```
如果operator new无法分配1000000000个int变量所需空间，不会立即抛出异常，是调用outOfMem，因为outOfMem已经被设置为默认的new-handler。
### new-handler的设计
当operator new无法满足内存申请时，它会不断的调用new-handler，直到找到足够的内存。所以一个设计良好的new-handler应该做以下事情：

1. 让更多内存可被使用。
2. 安装另一个new-handler。
3. 卸除new-handler。也就是将NULL指针传给set_new_handler，当operator new分配内存不成功时抛出异常。
4. 抛出bad_alloc(或派生自bad_alloc的)异常。这样的异常不会被operator new捕捉,因而会被传递到内存索求处。
5. 不返回。调用abort或exit。
### 类专属new-handlers
有时对于不同类，希望以不同new-handler处理operator new内存分配失败的情况。需要自己重写set_new_handler和operator new。
##### Widget::set_new_handler
```c++
class Widget{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static std::operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler currentHandler;
}
std::new_handler Widget::currentHandler=0;
std::new_handler Widget::set_new_handler(std::new_handler p) throw(){
    std::new_handler oldHandler=currentHandler;
    currentHandler=p;
    return oldHandler;
}
```
##### Widget::operator new
operator new需要做：
1. 调用**标准set_new_handler**，告知Widget的错误处理函数。这会**将Widget的new_handler安装为global new-handler。**
2. 调用**global operator new**，执行内存分配。
若分配失败会**自动调用Widget的new_handler**，因为它刚刚被安装为了global new-handler。若最终也无法分配足够的内存，则会抛出异常。使用**资源管理类**确保原来的new-handler被恢复才抛出异常。
3. 如果global operator new能够分配足够一个Widget对象所用的内存，Widget的operator new会返回一个指针，指向分配所得。**Widget的析构函数会恢复global new-handler。**

**资源管理类**
```c++
class NewHandlerHolder {
public:
    explicit NewHandlerHolder(std::new_handler nh) : handler(nh){}
    ~NewHandlerHolder() {
        std::set_new_handler(handler);//析构是把获得的new_handler安装回去
    }
private:
    std::new_handler handler;//记录下来
    NewHandlerHolder(const NewHandlerHolder&);//阻止copying
    NewHandlerHolder& operator=(const NewHandlerHolder&);
};
```
**operator new**
```c++
//对类Widget内的new操作符进行定义
void* Widget::operator new(std::size_t size) throw(std::bad_alloc) {
    NewHandlerHolder h(std::set_new_handler(currenthandler));//安装Widget的new-handler，返回值是global new-handler被暂时储存在资源管理类中
    return ::operator new(size); //调用global operator new分配内存或抛出异常
}//恢复global new-handler
```
### 模板
和非模板版本没什么区别。
```c++
template<typename T>
class NewHandlerSupport{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static std::operator new(std::size_t size) throw(std::bad_alloc);
    ...
private:
    static std::new_handler currentHandler;
}
template<typename T>
std::new_handler NewHandlerSupport<T>::currentHandler=0;
template<typename T>
std::new_handler NewHandlerSupport<T>::set_new_handler(std::new_handler p) throw(){
    std::new_handler oldHandler=currentHandler;
    currentHandler=p;
    return oldHandler;
}
template<typename T>
void* NewHandlerSupport<T>::::operator new(std::size_t size) throw(std::bad_alloc){
    NewHandlerHolder h(std::set_new_handler(currentHandler));
    return ::operator new(size);
}
```
 有了这个class template,就可以为Widget和其他类添加set_new_handler支持能力了。只要令Widget继承自NewHandlerSupport<Widget>就好：
```c++
class Widget:public NewHandlerSupport<Widget>{...}
```
### nothrow
有种传统的“分配失败便返回null”的行为，被称为nothrow。
```c++
Widget* pw1 = new Widget;//分配失败，抛出bad_alloc异常
if(pw1 == 0) ...            //一定失败
Widget* pw2 = new (std::nothrow) Widget;//分配失败，返回0
if(pw2 == 0) ...            //可能成功
```
实际上这种行为很有局限性，只适用于内存分配，后续的构造还是有可能抛出异常。
### 问题
new-handler的设计和operator new（条款51）的联系