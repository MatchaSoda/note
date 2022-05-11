# 条款14：在资源管理类中小心copying行为
  ### 自己创建资源管理类
 找不到合适的资源管理类时就要自己创建。
 
```c++
void lock(Mutex* pm);       //锁定
void unlock(Mutex* pm);    //解锁
```
为确保锁住后不忘记解锁，需要使用资源管理类来管理。
```c++
class lock{
public:
    explict lock(Mutex* pm) : mutexPtr(pm){
        lock(mutePtr);
    }
    ~lock(){
        unlock(mutePtr);
    }
private:
    Mutex* mutexPtr;
}
```
copying函数怎么处理呢？
 ### 常见的copying行为
* 抑制copying 
```c++
class Lock : private Uncopyable{    //  条款6，禁止复制
    ...
}
```
* 对底层资源使用“引用计数法”
### 资源的copying行为 决定了 RAII的copying行为
因为复制RAII对象必须一并复制他所管理的资源，所以我们只要内含一个tr1::shared_ptr成员变量，RAII class便可实现出reference-counting copy，即 资源的copying行为 决定了 RAII的copying行为。
tr1::shared_ptr允许指定“删除器”，在引用次数为0时便调用它。缺省时会删除对象，我们这里需要解锁，便把unlock做为他的第二个参数。
```c++
class lock{
public:
    explict lock(Mutex* pm) : mutexPtr(pm, unlock){
        lock(mutePtr);
    }
    //不需要声明析构函数，因为析构函数会自动调用non-static对象的析构函数
    //tr1::shared_ptr对象的析构函数会在引用次数为0时调用“删除器”。
private:
    std::tr1::shared_ptr<Mutex> mutexPtr;
}
```
