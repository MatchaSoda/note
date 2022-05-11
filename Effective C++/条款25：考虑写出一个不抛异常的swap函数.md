# 条款25：考虑写出一个不抛异常的swap函数
### swap的缺省实现
```c++
namsespace std {
    tempate<typename T>
    void swap(T& a, T& b)
    {
        T temp(a);
        a = b;
        b =temp;
    }
}
```
只要类型T支持copying，缺省的swap实现代码就会自动置换类型为T的对象。但是这个是深拷贝，对于某些类型而言是没必要的，其中最主要的即**以指针指向一个对象，内含真正数据**的类型，这种类型最常见的表现形式就是“pimpl手法”（pointer to implementation）：
```c++
class WidgetImpl {
public:
    ...
private:
    int a, b, c;      //可能有许多数据，意味着复制时间很长
    std::vector<double> v;
    ...
};

class Widget {          //该class使用pimpl手法
public:
    Widget(const Widget& rhs);     //复制Widget时，令它复制其WidgetImpl对象
    Widget& operator=(const Widget& rhs)   //operator=的实现见条款10~12
    {
        ...
        *pImpl = *(rhs.pImpl);//注意这里
        ...
    }
    ...
private:
    WidgetImpl* pImpl;    //指针，所指的对象就是内含Widget数据
};
```
这样就会把指针所指内容全拷贝一遍，而实际上只需要交换指针就可以了。
### 将std::swap针对Widget特化
所以我们可以 将std::swap针对Widget特化。
```c++
namsespace std {//不能通过编译
    tempate<>
    void swap<Widget>(Widget& a, Widget& b)
    {
        swap(a.pImpl, b.pImpl);    //置换Widget时，只需要置换它们的pImpl指针即可
    }
}
```
* tempate<>表示它是std::swap的一个全特化版本
* 函数名之后的<Widget>表示这一特化版本会针对“T是Widget”而设计的；

通常而言，**我们不能改变std命名空间内的任何东西，但是可以为标准templates（如swap）制造特化版本**，使得它专属于我们自己定义的class（例如Widget）。
但是上面的代码无法通过编译，因为pImpl指针是private。
### member函数swap
所以我们需要将swap声明为一个public member函数，然后在std::swap特化版本（non-member）中调用它。
```c++
class Widget {          //与前面相同，唯一的差别就是增加swap函数
public:
    ...
    void swap(Widget& other)
    {
        using std::swap;   //这个声明是非常必要的
        swap(pImpl, other.pImpl);   //若要置换Widget，就置换其pImpl指针
    }
    ...
};

namespace std {     //修订后的std::swap特化版本
    tempate<>
    void swap<Widget>(Widget& a, Widget& b)
    {
        a.swap(b);    //如果要置换Widgets，调用其swap成员函数
    }
}
```
可以通过编译，而且还与STL容器有一致性：
**所有STL容器也都提供有public swap成员函数和std::swap特化版本（用以调用前者）**
但是，这只适用于class，而不适用于class template。
### 偏特化
```c++
template<typename T> class WidgetImpl { ... };
template<typename T> class Widget { ... };

namespace std {
    template<typename T>
    void swap<Widget<T>> (Widget<T>& a,  Widget<T>& b){  //错误！
        a.swap(b); 
    }
}
```
class换成class template时，Widget换成了Widget<T>，全特化变成了偏特化。
**C++只允许对class template偏特化，在function template身上偏特化是不可以的。**
### 当我们打算偏特化一个function template时，通常会使用重载
```c++
namespace std {
    template<typename T>    //std::swap的一个重载版本
    void swap(Widget<T>& a, Widget<T>& b){
        a.swap(b); 
    }
}
```
一般而言，我们是可以重载function template的，但是std是一个特殊的命名空间：
**使用者可以全特化std内的template，但是不可以添加新的template（或者class、function以及其他东西）**
所以，应该这样做：
依然是声明一个non-member swap，让它调用member swap，但不再将那个non-member swap声明为std::swap的特化或重载。
把它放在另一个命名空间内。
```c++
namespace WidgetStuff {
    ...                         //模板化的WidgetImpl等等
    template<typename T>        //和前面一样，内含swap成员函数
    class Widget { ... };
    ...

    template<typename T>        //non-member swap函数
    void swap(Widget<T>& a,     //这里并不属于std命名空间
              Widget<T>& b)
    {
        a.swap(b);
    }
}   
```
虽然上面的做法对于class和class template都行得通，但我们还是应该为class特化std::swap。
所以，如果我们想让“class 专属版”的swap在尽可能多的语境下被调用，我们就应该同时在该class所在命名空间内写一个non-member版本以及一个std::swap特化版本。
### 调用
```c++
template<typename T>
void doSomething(T& obj1, T& obj2)
{
    using std::swap;    //令std::swap在此函数内可用
    ...
    swap(obj1, obj2);   //为T型对象调用最佳swap版本
    ...
}
```
c++会按照名称查找法则，和我们希望的一样，按如下顺序查找：
1. 可能存在的T专属版本而且可能存在与某个命名空间内（非std内）
2. 某个可能存在的特化版本
3. std既有的一般化版本

但是注意不要写`std::swap(obj1, obj2);`这会让编译器只使用std内的swap，而忽视其他地方的T专属版本。
### 问题
p109最后一句，为什么要使用non-member调用member的形式，而不使用friend。