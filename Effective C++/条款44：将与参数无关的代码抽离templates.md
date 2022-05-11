# 条款44：将与参数无关的代码抽离templates
### 代码膨胀
Templates用来节省时间和避免代码重复，但是他可能带来**二进制码**（包括代码和数据）的重复。类似于：
* 编写几个函数，会用到相同作用的代码；这时候你往往将相同代码搬到一个新函数中，给其他几个函数调用
* 编写某个class，其中某些部分和另外几个class相同，这时候你不会重复编写这些相同部分，只需把共同部分搬到新class中去，然后使用继承或复合

template也有重复，需要将他们抽离template。
### 非类型模板参数
**往往可消除，以函数参数或class成员变量替换template参数。**
一个n* n的矩阵，元素是T类型的objects，支持逆矩阵运算。
这个template接受一个T类型参数和一个size_t非类型参数：
```c++
template<typename T, std::size_t n>//T为数据类型，n为矩阵大小
class SquareMatrix {
public:
    ……
    void invert();//求逆运算
};
SquareMatrix<double, 5> sm1;
sm1.invert();//调用SquareMatrix<double,5>::invert
SquareMatrix<double, 10> sm2;
sm2.invert();//调用SquareMatrix<double,10>::invert
```
这样在使用时，一个调用`SquareMatrix<double,5>::invert`，另一个调用`SquareMatrix<double,10>::invert`,两个函数除了一个常量以外都相同，但是会有两份invert。不如**将这个模板参数换成函数参数**：
```c++
template<typename T>
class SquareMatrixBase {
protected:
    void invert(std::size_t matrixSize);
    ……
};
template<typename T, std::size_t n>
class SquareMatrix :private SquareMatrixBase<T> {
private:
    using SquareMatrixBase<T>::invert;//避免遮掩base中的invert
public:
    ……
    void invert(){   this->invsert(n);   }//注意this->
};
```
至于SquareMatrixBase::invert操作的数据，应该存放在derived class里，然后由derived class联络base class做逆运算操作。因为除了逆运算应该还有其他函数需要，所以以函数参数传递不好（一次次地告诉base class相同的信息），可以在base class内加一个指针指向这个矩阵所在的内存。
至于基类内的储存方式，可以考虑不用动态分配内存：
```c++
template<typename T>
class SquareMatrixBase {
protected:
    SquareMatirxBase(std::size_t n, T* pMem):size(n), pData(pMem) {}
    void setDataPtr(T* ptr) { pData = ptr; }
    ……
private:
    std::size_t size;
    T* pData;
};
template<typename T, std::size_t n>
class SquareMatrix :private SquareMatrixBase<T> {
public:
    SquareMatrix():SquareMatrixBase<T>(n, data) {}
    ……
private:
    T data[n * n];
};
```
也可以用动态分配内存：
```c++
template<typename T, std::size_t n>
class SquareMatrix :private SquareMatrixBase<T> {
public:
    SquareMatrix():SquareMatrixBase<T>(n, 0),pData(new T[n * n])
    {
        this->setDataPtr(pData.get());
    }
    ……
private:
    boost::scoped_array<T> pData;
};
```
是否以函数参数或class成员变量替换template参数，是否以动态分配储存，等等，这些都各有利弊，要结合实际情况判断。
### 类型模板参数
**往往可降低，让带有完全相同二进制表述的具现类型共享实现码**
类型模板参数也可能引发代码膨胀：
* 比如，有些平台上int和long是一样的，vector<int>和vector<long>也就完全相同，但是某些连接器会把template具现化为int和long两个版本；再比如，
* 再比如，许多平台上的所有指针类型都有相同的二进制表述。stl中list<int*>，list<const int*>就也一样。

所以，应该让带有完全相同二进制表述的具现类型共享实现码。
比如让一些使用T* 的函数，调用另一个操作void* 的函数，由后者完成实际工作。