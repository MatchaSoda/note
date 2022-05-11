# 条款11：在operator=中处理“自我赋值”
### 保证operator=的自我赋值安全性
`w = w;`看起来很离谱，但是是合法的。
当i和j相等时，`a[i] = a[j];`是自我赋值。
当px和py指向同一个东西，`*px = *py;`是自我赋值。
甚至当他们来自同一继承体系的时候，即使声明为不同类型也可能会有自我赋值。（base 和 derived对象之间）
下边是一个不安全的operator=版本。
```c++
class Bitmap{ ... };
class Widget{
    ...
private:
    Bitmap* pb;
};

Widget& Widget::operator=(const Widget& rhs){
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
这时如果自我赋值的话，delete就把bitmap销毁了，pb会指向一个被删除的对象。可以使用证同测试。
```c++
Widget& Widget::operator=(const Widget& rhs){
    if(this == &rhs) return *this;      //证同测试，如果地址相同，就什么都不做
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
但是上边这种不能保证异常安全性。如果在new的时候出现了异常，pb还是会指向一个被删除的对象。
### 保证operator=的异常安全性
```c++
Widget& Widget::operator=(const Widget& rhs){
    Bitmap* pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```
保证pb指向新的对象之前，不要删除pb。
还有另外一种做法：copy and swap。常见并且很好！
```c++
class Widget{
    ...
    void swap(Widget& rhs);         //交换*this和rhs的数据
    ...
};

Widget& Widget::operator=(const Widget& rhs){
    Widget temp(rhs);
    swap(temp);
    return *this;
}
```
也可以这样
```c++
Widget& Widget::operator=(Widget rhs){          //传值（自动生成副本）
    swap(rhs);
    return *this;
}
```