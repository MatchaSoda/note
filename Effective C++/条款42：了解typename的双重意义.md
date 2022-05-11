# 条款42：了解typename的双重意义
### 声明类模板
以下template声明式中，class和typename**没有不同**。
```c++
template<class T> class Widget;
template<typename T> class Widget;
```
### 标识嵌套从属类型名称
一份不标准的代码：
```c++
template <typename C>
void print2nd(const C& container)
{
    if (container.size() >= 2)
    {
        C::const_iterator iter(container.begin());
        ++iter;
        int value = *iter;
        std::cout << value;
    }
}
```
iter的类型是C::const_iterator 实际上是什么必须取决于template参数C。
* template内出现的名称如果相依于某个template参数，称之为**从属名称**（dependent names）。
* 如果从属名称在class内呈嵌套状，称之为**嵌套从属名称**（nested dependent name）。
* `C::const_iterator`这样的是**嵌套从属类型名称**（nested dependent type name）。
* `value`的类型是int。不依赖任何template参数的名称。称为**非从属名称**（non-dependent name）。

嵌套从属名称可能导致解析的困难：
```c++
template <typename C>
void print2nd(const C& container){
    C::const_iterator* x;
}
```
`C::const_iterator* x;`有歧义，可能表示是个指向一个C::const_iterator指针，也有可能表示const_iterator和x相乘（如果const_iterator是c内的static成员变量，x是个global变量名称）。 
**在我们知道C以前，没有任何办法可以知道C::const_iterator 是否为一个类型。**
c++有个规则可以解析此一歧义状态：缺省情况下从属名称不是类型。如果他是个类型我们必须告诉c++，在前边加上typename即可。
上述代码应改成这样：
```c++
template <typename C>
void print2nd(const C& container)
{
    if (container.size() >= 2)
    { 
        typename C::const_iterator iter(container.begin());//这里
        ++iter;
        int value = *iter;
        std::cout << value;
    }
}
```
注意：**必须对嵌套从属类型名称使用，其他名称不行。**
#### 例外
**不能**在**base classes list（基类列）** 或**member initialization list（成员初值列）** 内以它作为base class修饰符。
```c++
template <typename T>
class Derived: public Base<T>::Nested{//base class list中不能用
public:
    explicit Derived(int x) : Base<T>::Nested(x)//mem.init.list不能用
    {
       typename Base<T>::Nested temp;//需要
    }
};
```
### 问题
最后一个例子中Base<T>::Nested是什么。如果是基类中的一个类型名称的话，为什么在基类列和成员初值列要使用。base class修饰符是什么？