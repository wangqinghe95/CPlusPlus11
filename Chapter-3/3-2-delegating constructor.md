# 委派构造函数

## 定义
C++11 引入委派构造函数，可以在一个构造函数中调用其他构造函数来初始化对象的方式，从而避免了重复的初始化逻辑。


## 使用
具体的使用方法是：
当一个构造函数使用冒号后面跟着其他构造函数时，我们称该函数为委派构造函数。被调用的函数则成为目标构造函数。

```
class Info
{
public:
    Info(/* args */) : Info(1, 'a'){};
    Info(int i) : Info(i, 'a') {}
    Info(char c) : Info(1, c){}
    ~Info() {};
private:
    Info(int i, char e) : type(i), name(e) {}
    int type;
    char name;
}
```

1. 委派构造函数不能和初始化列表同时存在。但是可以调用一个初始化列表的构造函数。
2. 在使用委派构造函数时，建议提取出最通用的行为做目标构造函数。
3. 在有多个委派构造函数时，委派的构造函数的调用可能会产生环状，这个需要注意。

## 实际应用

使用构造函数模板产生目标构造函数

```
class TDConstructed
{
    template<class T> TDConstructed(T first, T last) : l(first, last) {}
    list<int> l;
private:
    TDConstructed(vector<short> &v) : TDConstructed(v.begin(), v.end()){}
    TDConstructed(deque<int>& d) : TDConstructed(d.begin(), d.end()) {}
};
```

1. 定义了一个构造函数模板，通过两个委托构造函数的委托，构造函数模板会被实例化。
2. T 会被推导为 `vector<short>::iterator` 以及 `deque<int>::iterator` 两个类型

## 异常处理

如果在委派构造函数中使用 try 的话，那么从目标构造函数中产生异常，可以在委派构造函数中捕获到。如下所示：

```
#include <iostream>
using namespace std;

class DCExcept
{
public:
    DCExcept(double d) try : DCExcept (1, d) {
        cout << "Run the body" << endl;
    } 
    catch(...) {
        cout << "Caught exception" << endl;
    }
private:
    DCExcept(int i, double d) {
        cout << "Going to throw" << endl;
        throw 0;
    }
    int type;
    double data;
};

int main()
{
    DCExcept a(1.2);

    return 0;
}

/*
Going to throw
Caught exception
terminate called after throwing an instance of 'int'
Aborted (core dumped)
*/
```

从运行结果可以看出 `cout << "Run the body" << endl;` 没有打印出来，即委派构造函数的函数体没有被执行到。
这样的设计是合理的，因为如果函数体依赖于目标构造函数构造的结果，那么当目标构造函数发生异常的情况下，还是不要执行委派构造函数体中的代码较好。
