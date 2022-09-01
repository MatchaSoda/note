# STL 四种智能指针
### 前言
智能指针的实现原理就是在一个类的内部封装了类对象的指针，然后在析构函数里对我们的类对象指针进行释放，因为类的析构是在类对象生命期结束时自动调用的，这样我们就**不需要手动释放内存**了，避免忘记手动释放导致的内存泄漏。
### auto_ptr
auto_ptr 是最初的智能指针，为了避免 有多个指针同时指向一个对象的情况下 在删除时多次删除一个对象，auto_ptr 通过**所有权**的概念来保证同时只有一个指针会指向一个对象，在复制或者赋值时将转交所有权，旧指针置空。
```cpp
//定义于头文件 <memory>
template <class T> class auto_ptr; 
//初始化
auto_ptr<int> test_int(new int(33));
//复制或者赋值时将转交所有权：1置空，2负责管理对象
auto_ptr<string> test_string1(new string("create"));
auto_ptr<string> test_string2(test_string1);
auto_ptr<int> p1(new int(1024));
auto_ptr<int> p2(new int(2048));
p1 = p2;
```
auto_ptr 存在很多问题，所以**在 C++11 中被 unique_ptr 取代了**。
1. 奇怪的复制语义  
   复制后原指针失效，容易**引发潜在的内存问题导致程序崩溃**，也因为这个原因 auto_ptr 不能作为容器对象，因为 STL 容器中的元素经常要支持拷贝、赋值等操作。
    ```cpp
    auto_ptr<Test> a;
    auto_ptr<Test> b = a;
    b->doSomething();
    a->doSomething();   //崩溃    
    ```
2. 作为参数  
   按值传递时，函数调用过程中会产生一个临时对象来接收传入的 auto_ptr (拷贝构造)，这样，传入的实参 auto_ptr 就失去了其对原对象的所有权，而该对象会在函数退出时被局部 auto_ptr 删除。
    ```cpp
    void func(auto_ptr<int> ap) { cout << *ap; }
    auto_ptr<int> ap1(new int(0));
    func(ap1);
    cout << *ap1;  //错误，经过func(ap1)函数调用，ap1已经不再拥有任何对象了。
    ```
    可以使用 const reference 来传递参数。  
3. 可能造成混乱
    ```cpp
    int* p = new int(0);
    auto_ptr<int> ap1(p);
    auto_ptr<int> ap2(p);
    ```
    没有调用复制构造或者赋值，他们不知道 p 也被其他指针持有，会多次删除。
### unique_ptr
unique_ptr 由 C++11 引入，旨在替代存在很多问题的 auto_ptr 。
```cpp
template <class T, class Deleter = std::default_delete<T>>
class unique_ptr;
template <class T, class Deleter>
class unique_ptr<T[], Deleter>;
```
相对于 auto_ptr 的提升：
* unique_ptr 有两个版本，可以管理 单个对象（例如以 `new` 分配） 和 动态分配的对象数组 （例如以 `new[]` 分配）。
* unique_ptr 禁止了拷贝语义，提供了移动语义。
  ```cpp
  unique_ptr<string> upt(new string("lvlv"));
  unique_ptr<string> upt1(upt);  //编译出错，已禁止拷贝
  unique_ptr<string> upt1 = upt;  //编译出错，已禁止拷贝
  unique_ptr<string> upt1 = std::move(upt);  //控制权限转移

  auto_ptr<string> apt(new string("lvlv"));
  auto_ptr<string> apt1(apt);  //编译通过
  auto_ptr<string> apt1 = apt;  //编译通过
  ```
  unique_ptr 类满足*可移动构造 (MoveConstructible)* 和*可移动赋值 (MoveAssignable)* 的要求，但不满足*可复制构造 (CopyConstructible)* 或*可复制赋值 (CopyAssignable)* 的要求。
* 可以自定义资源删除操作（Deleter）。

前边说了 auto_ptr 不能作为容器对象，是因为奇怪的拷贝行为。而现在 unique_ptr 禁止了拷贝，借助移动语义可以在stl容器中有着不错的发挥。但是我们在使用时也要注意不要使用拷贝构造。
```cpp
typedef std::unique_ptr<int> unique_t;
typedef std::vector< unique_t > vector_t;

vector_t vec1;                           // fine
vector_t vec2(5, unique_t(new Foo));     // Error (Copy)
vector_t vec3(vec1.begin(), vec1.end()); // Error (Copy)
vector_t vec3(make_move_iterator(vec1.begin()), make_move_iterator(vec1.end()));
    // Courtesy of sehe

std::sort(vec1.begin(), vec1.end()); // fine, because using Move Assignment Operator

std::copy(vec1.begin(), vec1.end(), std::back_inserter(vec2)); // Error (copy)
```
### shared_ptr
```cpp
template< class T > class shared_ptr;
```
shared_ptr 是一个**引用计数型智能指针**，shared_ptr 对象除了包括一个所拥有对象的指针外，还包括一个引用计数代理对象的指针。这可以使多个 shared_ptr 对象占有同一对象，当最后剩下的占有对象的 shared_ptr 被销毁或赋值为另一指针时（即引用计数为0时），对象才会被销毁并释放内存。
#### 实现
在典型的实现中， std::shared_ptr 只保有二个指针：
* get() 所返回的指针
* 指向*控制块*的指针

控制块是一个动态分配的对象，其中包含：
* 指向被管理对象的指针或被管理对象本身
* 删除器（类型擦除）
* 分配器（类型擦除）
* 占有被管理对象的 shared_ptr 的数量（即引用计数）
* 涉及被管理对象的 weak_ptr 的数量
#### 线程安全
多个线程能在 shared_ptr 的不同实例上调用所有成员函数（包含复制构造函数与复制赋值）而不附加同步，即使这些实例是副本，且共享同一对象的所有权。  
若多个执行线程访问同一 shared_ptr 而不同步，且任一线程使用 shared_ptr 的非 const 成员函数，则将出现数据竞争；原子函数的 shared_ptr 特化能用于避免数据竞争。  
没完全看懂，还有问题！  
https://zh.cppreference.com/w/cpp/memory/shared_ptr  
https://www.zhihu.com/question/56836057
#### 循环引用
在循环引用的场景下，shared_ptr无法正确释放内存。
```cpp
#include <iostream>
#include <memory>
using namespace std;

class Son;

class Father {
 public:
  shared_ptr<Son> son_;
  Father() { cout << __FUNCTION__ << endl; }
  ~Father() { cout << __FUNCTION__ << endl; }
};

class Son {
 public:
  shared_ptr<Father> father_;
  Son() { cout << __FUNCTION__ << endl; }
  ~Son() { cout << __FUNCTION__ << endl; }
};

int main() {
  auto son = make_shared<Son>();
  auto father = make_shared<Father>();
  son->father_ = father;
  father->son_ = son;
  cout << "son: " << son.use_count() << endl;
  cout << "father: " << father.use_count() << endl;
  return 0;
}
```
如上程序， son 类中有 father ， father 类中有 son 。两个对象在析构时都会等待另一个对象析构完毕，造成循环引用，内存泄漏。解决办法就是使用 weak_ptr 。
### weak_ptr
weak_ptr 对被 shared_ptr 管理的对象存在非拥有性（“弱”）引用。**在访问所引用的对象前必须先转换为 shared_ptr**。

weak_ptr 用来表达**临时所有权**的概念：当某个对象只有存在时才需要被访问，而且随时可能被他人删除时，可以使用 weak_ptr 来跟踪该对象。需要获得临时所有权时，使用成员函数 `lock()` 则将其转换为 shared_ptr，此时如果原来的 shared_ptr 被销毁，则该对象的生命期将被延长至这个临时的 shared_ptr 同样被销毁为止。

shared_ptr 被赋值给weak_ptr 时，shared_ptr的*引用计数*不变。因为在 shared_ptr 的实现中， shared_ptr 和 weak_ptr 是分别计数的，我们平常说的*引用计数*指的是 shared_ptr 的数量。

在上边 son 和 father 的例子中将 shared_ptr 替换为 weak_ptr 即可解决循环引用问题。

### 题目
下面代码有什么问题吗？
```cpp
int main() {
  std::unique_ptr<int> up(new int(1));
  std::shared_ptr<int> sp1(up.release());
  std::shared_ptr<int> sp2(up.get());
  up.reset();
  return 0;
}
```
**没有问题。**  
`up.release()`把管理对象交给了`sp1`，`up.get()`无管理对象时会返回一个`nullptr`，`sp2`以`nullptr`进行初始化。  
`up.reset()`原型为
```cpp
void reset( pointer ptr = pointer() ) noexcept;
```
`reset()`主要干三件事：
* 保存原指针的副本`old_ptr = current_ptr`
* 替换当前指针为新指针`current_ptr = ptr`
* 若旧指针非空则删除之前的管理对象`if(old_ptr != nullptr) get_deleter()(old_ptr)`

所以这里reset使用也没有问题。