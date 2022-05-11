# 条款38：通过复合塑模出has-a或“根据某物实现出”
### 复合（composition）
复合（composition）是类型之间的一种关系，**一个类型的对象包含其他类型对象**便是这种关系。
在程序中，大概可以分为两个领域（domains）。
* 程序中对象相当于你所塑造现实世界中某物，例如地址、电话号码，这样的对象属于**应用域**（application domain）。
* 还有一些是实现细节上的人工复制品，例如缓冲区（buffers）、互斥器（mutexes）、查找树（search tree）等，这些是**实现域**（implementation domain）。
### 在应用域，复合意味has-a
Person有一个名称，一个地址，一个电话号码。
```c++
class Address{ …… };
class PhoneNumber{ …… };
class Person{
public:
    ……
private:
    std::string name;
    Address address;
    PhoneNumber mobilePhone;
};
```
### 在实现域，复合意味is-implemented-in-terms-of(根据某物实现出)
Set根据一个list对象实现出来。
```c++
template<calss T>
class Set{
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
private:
    std::list<T> rep;
};
```