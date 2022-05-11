# 条款10：令operator=返回一个reference to *this
### 令赋值操作符返回一个reference to *this
赋值的时候，经常出现
```c++
int x, y, z;
x = y = z = 10;
```
为了实现这样的连锁赋值，需要赋值操作符返回左侧对象的引用。
```c++
class Widget{
public:
    ...
    Widget& operator=(const Widget& rhs){
        ...
        return *this;
    }
};
```