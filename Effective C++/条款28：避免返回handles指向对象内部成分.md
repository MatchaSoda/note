# 条款28：避免返回handles指向对象内部成分
### 封装性和const
返回指向对象内部成分的handles（号码牌，用来取得某个对象，如reference、指针、迭代器）会降低封装性。
```c++
class Point {
public:
	Point(int x, int y);
 
	void setX(int newVal);
	void setY(int newVal);
}
struct RectData {
	Point ulhc;//ulhc="upper left-hand corner"(左上角)
	Point lrhs;//lrhc="lower right-hand corner"(右下角)
};
class Rectangle {
private:
	shared_ptr<RectData> pData;
public:
	Point& upperLeft()const { return pData -> ulhc; }
	Point& lowerRight()const { return pData->lrhc; }
};
Point coord1(0, 0);
Point coord2(100, 100);
const Rectangle rec(coord1, coord2);
 
rec.upperLeft().setX(50);//此处ulhc与lrhc都被声明为private，但实际上却是public,
                         //因为仍然是在直接修改私有成员。
```
虽然ulhc和lrhs是private，但因为public函数返回了他们的引用，他们就会变成pubilc，降低了封装性。
这里我们是想让用户访问节点的，是故意放松封装的，但是我们并不想让用户修改他们。虽然函数是const，但传出的是reference，是bitwise constness的错误。
```c++
class Rectangle {
private:
	shared_ptr<RectData> pData;
public:
	const Point& upperLeft()const { return pData->ulhc; }
	const Point& lowerRight()const { return pData->lrhc; }
};
```
返回类型加上const即可。可以查看不能修改，符合预期。
### 虚吊
dangling handles(空悬的号码牌)：这种handles所指东西（的所属对象）不复存在。最常见的来源是函数返回值。
```c++
class GUIObject{};
const Rectangle boundingBox(const GUIObject& obj);          //返回一个矩形
//客户使用这个函数：
GUIObject* pgo;
const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());//取矩形左上角点
```
以by value的方式返回了一个对象，而调用的函数却以reference返回了其内部成分。语句结束后，该对象就会被销毁，其内部成分就会被析构掉，reference就会指向一个不存在的对象！