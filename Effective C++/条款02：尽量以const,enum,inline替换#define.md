# 条款02：尽量以const,enum,inline替换#define
### 对于单纯变量，最好以const对象或者enums 替换#define
```c++
#define PAI 3.14
```
define不视为语言的一部分，可能在编译之前就会被替换，报错的时候只会出现3.14而不是PAI，找起来会很麻烦，甚至可能#define在别人写的文件里，找都找不到。
#### const
* 常量指针（constant pointers）
因为常量表达式通常放在头文件内（方便被不同源码含入），所以要将指针（不是指针所指物）声明为const。
* class专属常量
```c++
class GamePlayer{
private:
    static const int NumTurns = 5; //常量声明式
    int scores[NumTurns];
    ...
};
```
将常量作用域限制在类内，需要设为成员，保证只有一个实体，设为static。
注意这是声明式，此情况下若不取地址就不需要定义式，如果需要，则要在实现文件而非头文件内定义，并且不能赋值，因为类内声明时已经给过初值了。
#### enum
```c++
class GamePlayer{
private:
    enum { NumTurns = 5 }; //枚举类型可当成 ints 被使用
    int scores[NumTurns];
    ...
};
```
enum比较像# define 而不像const，比如enum不能取地址。如果不想被pointer或者refence指的时候就可以用这个。
### 对于形似函数的宏，最好改用inline替换#define
#define和inline都不会带来function call的额外开销，但是写在#define里边不好看并且很容易出错。
### 问题
p14 第一段 
>因为放头文件内，所以将指针声明为const。

因果关系？