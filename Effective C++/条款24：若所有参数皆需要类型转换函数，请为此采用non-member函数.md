# 条款24：若所有参数皆需要类型转换函数，请为此采用non-member函数
### 只有参数列内的参数是 隐式类型转换 的合格参与者
```c++
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);  //non-explicit
    int numerator() const;          //分子的访问函数
    int denominator() const;        //分母的访问函数
private:
    ...
};
```
若operator* 是Rational的成员函数：
```c++
class Rational {
public:
    ...
    const Rational operator* （const Rational& rhs） const;
};
```
则
```c++
Rational oneEight(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEight;   //成功！
result = result * oneEight;             //成功！
result = oneHalf * 2;   //成功！
result = 2 * oneHalf;   //错误！
//上边两行，相当于
result = oneHalf.operator*(2);  //成功！
result = 2.operator*(oneHalf);  //错误！
```
可以看到，后边这种情况试图把2隐式转换为Rational，而它作为（地位相当于this的）隐喻参数是绝对不能转换过去的。只有参数列内的参数是隐式类型转换的合格参与者。
如果所有参数都可能有类型转换，这种情况下，不能有地位相当于this的隐喻参数出现，所以要使用non-member函数，把所有参数都放在参数列中。
```c++
class Rational {            //不包含operator*
    ...
};
const Rational operator*(const Rational& lhs, const Rational& rhs){}//non-member

Rational oneForth(1, 4);
Rational result;
result = oneForth * 2;      //成功了！
result = 2 * oneForth;      //成功了！！
```