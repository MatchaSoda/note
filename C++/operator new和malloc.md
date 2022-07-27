# operator new和malloc
operator new 和 malloc 很像，它们都可用于申请动态内存。operator new 对应 operator delete ，malloc 对应 free 。

malloc 与 free 是 C++/C 语言的标准库函数，new/delete 是 C++ 的运算符， new或delete时候，会默认调用 std::operator new() 或 std::operator delete() 。 

虽然我们不能改变new/delete的行为，但是通过重载 operator new() 和 operator delete() 我们可以实现自己想要的内存管理方式。如下：
```cpp
class A {
 public:
  A() { cout << "A constructor" << endl; }
  void* operator new(size_t size) {
    cout << "this is A's new" << endl;
    return ::operator new(size);
  }
  void operator delete(void* ptr) {
    cout << "this is A's delete" << endl;
    return ::operator delete(ptr);
  }
  ~A() { cout << "A destructor" << endl; }
};
int _tmain(int argc, _TCHAR* argv[]) {
  A* a = new A;
  delete a;
  return 0;
}
```
operator new 的三种形式：
```cpp
//异常抛出形式：
void* operator new (std::size_t size) throw (std::bad_alloc);
//不抛出异常形式
void* operator new (std::size_t size, const std::nothrow_t& nothrow_value) throw();
//placement形式
void* operator new (std::size_t size, void* ptr) throw();
```