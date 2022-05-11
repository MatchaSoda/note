# 条款16：成对使用new和delete时要采取相同形式
### 注意new和delete的一致性
new中用[]，delete时也要用[]。new时不用，delete时也一定不要用。
```c++
string* s1 = new string;
string* s2 = new string[100];
delete s1;
delete [] s2;
```
### 在使用typedef时要标好使用new创建时该如何delete
```c++
typedef string AddressLine{4};              //用AddressLine来替换string[4]
string* pa1 = new AddressLine;            //new AddressLine返回一个string*对象，就像string[4]一样
delete pa1;     //错误
delete [] pa1;  //正确
```
所以尽量不要对数组使用typedef。可以用vector和string替代。本例的AddressLine则可以用vector<string>替代。