# 初始化列表

C++11 允许使用花括号列表初始化。这种方法被称为初始化列表。

如下所示的数组声明
```
int a[] = {1,3,5};
int b[] {2,4,6};
vector<int> c{1,3,5};
map<int, double> d = {{1,1.0f}, {2,2.0f}, {5,3.2f}};
```

初始化列表特性的支持附带了一些对变量进行初始化的方法。比如：
+ 使用等号加上赋值表达式，比如 `int a = 3+4`
+ 使用等号加上花括号式的初始化列表，`int a = {3+4}`
+ 圆括号式的表达式，比如 `int a(3+4)`
+ 花括号式的初始化表达式，比如 `int a{3+4}`
+ 堆中申请资源，`int *i = new int(1); double *d = new double{1.2f}`

C++11 对于标准模板库中的支持，是源自于 <initializer_list> 头文件中 initializer_list 类模板的支持。

## 构造函数使用初始化列表

```
enum Gender{boy, girl};

class People
{
public:
    People(initializer_list<pair<string,Gender>> l) {
        auto i = l.begin();
        for(; i != l.end(); ++i) {
            data.push_back(*i);
        }
    }
private:
    vector<pair<string,Gender>> data;
};
```

使用了模板类 `initializer_list<pair<string,Gender>>` 作为参数的构造函数，用来初始化成员数据。

## 函数参数列表使用初始化列表

列表初始化还可以调用已有的代码来实现。

```
void Fun(initializer_list<int> iv)
{}

Fun({1,2});
Fun({});
```

## 类和结构体成员函数可以使用初始化列表

```
class Mydata
{
public:
    Mydata& operator [] (initializer_list<int> l) {
        for(auto i = l.begin(); i != l.end(); ++i) {
            idx.push_back(*i);
        }
        return *this;
    }
    Mydata& operator = (int v) {
        if(idx.empty() != true) {
            for(auto i = idx.begin(); i != idx.end(); ++i) {
                d.resize((*i > d.size()) ? *i : d.size());
                d[*i-1] = v;
            }
            idx.clear();
        }
        return *this;
    }

    void Print() {
        for(auto i = d.begin(); i != d.end(); ++i) {
            cout << *i << " ";
        }
        cout << endl;
    }
private:
    vector<int> idx;
    vector<int> d;
};

int main()
{
    Mydata d;
    d[{2,3,5}] = 7;
    d[{1,4,5,8}] = 4;
    d.Print();
    return 0;
}
```

重载了 [] 和 = 两个操作符，前者使用了初始化列表作为参数传入。

## 函数返回初始化列表

```
vector<int> Func(){
    return {1,3};
}
```

初始化列表构造成什么类型是依据返回类型决定，

```
// 返回 deque<int>
deque<int> Func() {
    return {1,3};
}
```

如果返回类型是一个引用类型，则会返回一个临时变量的引用，比如
```
const vector<int> & Func() {
    return {3,5};
}
```

这里需要加上一个 const 修饰，该规则与返回一个字面常量一样。