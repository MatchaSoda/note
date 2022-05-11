### 构造或析构过程中不要调用virtual函数，因为这类函数从不下降至deriverd class
```c++
class Transaction {
public:
    Transaction();
    virtual void logTransaction() const = 0;        //virtual函数
};
Transaction::Transaction(){
    logTransaction();
}
class BuyTransaction:public Transaction {
public:
    virtual void logTransaction() const;
};
class SellTransaction:public Transaction {
public:
    virtual void logTransaction() const;
};
```
使用`BuyTransaction b;`时，BuyTransaction的构造函数会被调用，但是首先会调用基类Transaction的构造函数，然后调用Transaction版本的virtual函数logTransaction()。因为此时若调用derived class的函数，则会用到derived class的成员变量，而这些变量还未初始化，所以c++不会下降调用derived class的函数。
但是我们可以将derived class的信息上传至base class。
```c++
class Transaction {
public:
    explicit Transaction(const string& logInfo);
    void logTransaction(const string& logInfo) const;       //non-virtual函数
};
Transaction::Transaction(const string& logInfo){
    logTransaction(logInfo);
}
class BuyTransaction:public Transaction {
public:
    BuyTransaction(parameters) : Transaction(createLogString(parameters)) {...}     // 将log信息传给base class构造函数
private:
    static string createLogString(parameters);
};

```
* static成员函数与类实例（对象）无关。用来控制该函数的访问权限，使用类内的static变量。
* 没有使用成员初值列 而是 static函数createLogString。使用函数更加方便可读，声明为static是为了限制函数不能访问类实例（对象）的成员变量，也就不会发生“createLogString使用了初期未成熟之BuyTransaction对象内尚未初始化的成员变量”。