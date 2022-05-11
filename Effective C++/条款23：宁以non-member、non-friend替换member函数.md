# 条款23：宁以non-member、non-friend替换member函数
### 可以增加封装性、包裹弹性（packing flexibility）和机能扩展性
```c++
class  WebBrowser{
public:
    ...
    void clearCache();
    void clearHistory();
    void clearCookies();
};
```
如果我们需要某种便利函数`void clearEverything();`，他的作用是调用一系列public member函数，这时可以选择他是否成为member函数。我们选择让他成为non-member non-friend函数。
**non-member non-friend函数比member函数更具备封装性。**
* 注意是non-member non-friend函数，也不能是friend。因为从封装角度看，friends函数和member函数相同，都可以访问private。
* 只因为在意封装性而让函数“变成class的non-member”，并不意味着它“不可以是另一个class的member。”比如它可以是某个工具类（utility class）的一个static member函数（只要不是当前类的member或friend，就不会对当前类的封装造成影响）。

类似于c++标准程序库，通常会把这些便利函数**放在不同头文件下，并且放在同一namespace内**。
```c++
//头文件webbrowser.h，class WebBrowser自身以及WebBrowser核心机能
namespace WebBrowserStuff {
    class WebBrowser { ... };
    ...      //核心机能，例如几乎所用用户都会用到的non-member函数
}
//头文件webbrowserbookmarks.h
namespace WebBrowserStuff {
    ...      //与书签相关的便利函数
}
//头文件webbrowsercookies.h
namespace WebBrowserStuff {
    ...      //与cookie相关的便利函数
}

```