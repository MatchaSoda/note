# 条款47：请使用traits classes来表现类型信息
### iterator_category
STL迭代器的类型，总共有五种：
* input_iterator：只读，只能逐个前移
* output_iterator：只写，只能逐个前移
* forward_iterator：可读可写，只能逐个前移
* bidirectional_iterator：可读可写，支持逐个前移和后移
* random_access_iterator：可读可写，支持随机访问（任意步数移动）

针对这五种分类,C++标准库分别提供专属的"卷标结构"(tag struct)加以区分:
```c++
struct input_iterator_tag{};
struct output_iterator_tag{};
struct forward_iterator_tag:public input_iterator_tag{};
struct bidirectional_iterator_tag:public forward_iterator_tag{};
struct random_access_iterator_tag:public bidirectional_iterator_tag{};
```
### traits classes
我们需要根据不同的迭代器类型来使用不同的实现方法。
```c++
template<typename IteT, typename DistT>void advance(IterT& iter,DistT d){
    if(iter is a random access iterator)
        iter+=d;
    else{
        if(d>=0)    { while(d--) ++iter; }
        else{ while(d++) --iter; }
    }
}
```
要想实现`if(iter is a random access iterator)`这种判断，就是说要判断Iter的迭代器类型是否为random access。即，**要从类型中获取信息**。
>Traits并不是C++关键字或一个预先定义好的构件；它们是一种技术，也是一个C++程序员共同遵守的协议。

traits技术就是用来使STL的某些泛型算法能够在编译期取得某些类型信息的。他要求对内置类型和用户自定义类型都表现良好。而这也就意味着“不能使用类型内嵌套信息（nesting information）”，因为我们无法将信息嵌套到原始指针内。我们应该使用**template及其特化版本**。
```c++
//traits要求每一个自定义类型使用typedef将信息储存进去
//这里是将容器所对应的迭代器tag存入iterator_category
template<...>
class deque{
public:
    class iterator{
    public:
        typedef random_access_iterator_tag iterator_category;
        ...
    };
    ...
};
template<...>
class list{
public:
    class Iterator{
    public:
         typedef bidirectional_iterator_tag iterator_category;
        ...
    };
    ...
};
//traits中再用typedef复述一下
template<typename IterT>
struct iterator_traits{
    typedef typename IterT::iterator_category iterator_category;
    ...
};  
//template偏特化，针对内置指针
template<typename IterT>
struct iterator_traits<IterT*>{
    typedef random_access_iterator_tag iterator_category;
    ...
};
```
### 使用traits classes
设计完traits之后，我们可以实现之前的伪码：
```c++
template<typename IterT,typename DisT>void advance(IterT& iter,DistT d){
    if(typeid(typename::std::iterator_traits<IterT>::iterator_category
       ==typeid(typename::std::random_access_iterator_tag))
    ...
}
```
有两个问题：
①首先，这会导致编译期问题。条款48
假设对advance作以下调用：
```c++
std::list<int>::iterator iter;
advance(iter,10);
```
那么advance将被特化为以下形式:：
```c++
void advance(std::list<int>::iterator iter,int d){
 if(typeid(typename::std::iterator_traits<std::list<int>::iterator>:iterator_category)==typeid(typename::std::random_access_iterator_tag))
        iter+=d;  //错误,编译时不通过!
...
}
```
**编译器必须保证所有代码可行，即使是永远不会执行起来的代码。** 即使==不成立，if里的代码不会被执行但仍然要可行。如果类型不匹配，就很难对其做出正确的行为。
②更重要的是，IterT在编译期间获知，iterator_category在编译期就可以确定，而这个if语句是在运行期才会核定。浪费时间并且会造成代码膨胀。我们可以使用**重载**！
我们需要一个条件式来判断“编译期核定成功之类型”，使用**用重载来代替if-else**。让编译器来替你做选择，并且这是在编译期就可以完成的。
```c++
template<typename IterT,typename DistT>void doAdvance(IterT& iter,Dist d,std::random_access_iterator_tag){
    iter+=d;
}
template<typename IterT,typename DistT>void doAdvance(IterT& iter,Dist d,std::bidirectional_iterator_tag){
    if(d>=0)
        while(d--)
            ++iter;
    else
        while(d++)
            --iter;
}
template<typename IterT,typename DistT>void doAdvance(IterT& iter,Dist d,std::input_iterator_tag){
    if(d<0)
        throw out_of_range("Negative distance");
    while(d--)
        ++iter;
}
template<typename IterT,typename DistT>void doAdvance(IterT& iter,Dist d){
    doAdvance(iter,d,typename std::iterator_traits<IterT>::iterator_category());
}
```
由于之前iterator卷标结构的继承关系，doAdvance的input_iterator版本也可以被forward iterator调用。
有了这样的doAdvance，我们就可以让advance调用他，在参数中传递traits，traits会根据类型（即使是内置类型）返回对应的迭代器类型，之后编译器会自动的选择合适的doAdvance重载版本：
```c++
template<typename IterT,typename DisT>void advance(IterT& iter,Dist d){
    doAdvance( iter, d, typename std::iterator_traits<IterT>::iterator_category() );
}
```
完美。
总结一下，**trait classes的使用过程**：
* 建立一组重载函数(身份像劳工)或函数模板(例如doAdvance)，彼此之间的差异仅在于各自的traits参数。令每个函数实现码与其接受之traits相应和。
* 建立一个控制函数(身份像工头)或函数模板(例如advance)，它调用上述"劳工函数并传递traits classes所提供的信息"。