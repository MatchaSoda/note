# 条款17：以独立语句将newed对象置入智能指针
### 以独立语句将newed对象存入智能指针
```c++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
//会编译错误
processWidget(new Widget, priority());
//容易引发内存泄露
processWidget(std::tr1::shared_ptr<Widget> pw(new Widget), priority());
//正确调用
std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority);
```
* 使用`processWidget(new Widget, priority());`不能通过编译，因为tr1::shared_ptr的构造函数是explicit构造函数，不能让raw pointer隐式转换为tr1::shared_ptr。
* 若使用`processWidget(std::tr1::shared_ptr<Widget> pw(new Widget), priority());`则易引发内存泄露，因为在函数调用之前要先计算好其实参，此例中有两个，一个ptr一个函数。
而该ptr在构造时有两步，执行`new Widget`和`tr1::shared_ptr`的构造。
故计算实参共有三步，执行`new Widget`，`tr1::shared_ptr`的构造和调用`priority`.
而c++编译器不确定以什么样的顺序执行这三步，可能会把调用`priority`放在第二步（假如这样更高效），但这就会使资源的产生和储存分开（违背了RAII），就有了出错的可能性（若函数`priority`引发异常则会引发内存泄露）。
* 所以应该以独立语句将newed对象存入智能指针