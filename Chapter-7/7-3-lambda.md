# Lambda 函数

lambda 表达式被称为匿名函数或者闭包，是一种在编程中使用简洁的方式表示匿名函数，通常用于创建简单的、一次性使用的函数，无需正式定义一个完成的函数。

## C++ lambda 函数

C++11 标准支持 lambda 函数，其格式如下：

```
[capture](parameters) mutable->return-type
{
    statement
}
```

+ [capture]：捕获列表，中括号是一个 lambda 引出符，内部的捕获列表用于表示能够捕捉上下文中的变量以供 lambda 函数使用
+ (parameters)：参数列表，与普通函数的列表一致，如果不需要参数传递，可以和括号一起省略
+ mutable：mutable 修饰符，默认 lambda 是一个 const 函数， mutable 可以取消其常量性，在使用该修饰符时，参数列表不可省略，即使参数为空
+ ->return-type：返回类型，用于追踪返回类型形式，声明函数的返回类型。如果不需要返回值，可以连同 -> 一同省略。此外在返回类型确认的情况下，也可以省略该部分，让编译器自行推导
+ {statement}：函数体，内容同普通函数一样。不过除了可以使用参数以外，还可以使用所有捕获的变量

### 捕获列表

lambda 函数和普通函数最大的可见区别之一，就是 lambda 函数可以通过捕获列表访问一些上下文中的数据。

如下所示：

```
#include <iostream>
using namespace std;
int main()
{
    int boys = 4, girls = 3;
    auto totalChild = [girls, &boys]() -> int {
        return girls + boys;
    };

    cout << totalChild() << endl;
    return 0;
}
```

捕获列表由多个捕获项组成，并以逗号分隔。捕获列表由以下几种形式：
+ [var]：以值传递的形式传递变量 var
+ [=]：以值传递的形式捕获父作用域中的所有变量，包括 this
+ [&var]：以引用传递方式捕获变量 var
+ [&]：以引用传递方式捕获所有父领域的变量
+ [this]：以值传递的方式捕获当前 this 指针

以上捕获类型可以组合起来，比如
+ [=, &a,&b]：以引用的方式捕获变量 a 和 b，值传递的方式捕获其他变量
+ [&, a, this]：以值传递的形式捕获变量 a 和 this，引用方式捕获其他所有变量

捕获列表不允许重复传递，如下
+ [=, a, b]：这样的 = 已经以值传递的方式捕获了所有变量，再使用值传递的方式捕获 a 就是重复了。

## lambda 和仿函数

函数对象，也被成为仿函数，是一个重定义了成员函数的 operator() 的自定义类。该类有个特点在代码层面上像跟函数使用一样。

如下

```
#include <iostream>
using namespace std;

class Functor
{
public:
    int operator()(int x, int y) {
        return x + y;
    }
};

int main()
{
    int girls = 3;
    int boys = 4;
    Functor totalChild;
    cout << totalChild(girls, boys) << endl;
    return 0;
}
```

相比于普通函数，仿函数可以通过私有成员定义初始状态，在声明对象的时候进行初始化。由于声明一个仿函数对象可以拥有多个不同初始状态的实例，因此可以借由仿函数产生多个功能类似但是又不同的仿函数实例。

```
#include <iostream>
using namespace std;

class Functor
{
public:
    int operator()(int x, int y) {
        return x + y;
    }
};

int test1()
{
    int girls = 3;
    int boys = 4;
    Functor totalChild;
    cout << totalChild(girls, boys) << endl;
    return 0;
}

class Tax
{
private:
    float rate;
    int base;
public: 
    Tax(float r, int b) : rate(r), base(b){}
    float operator()(float money) {
        return (money - base) * rate;
    }
};

int main()
{
    Tax high(0.4, 30000);
    Tax middle(0.25, 20000);
    cout << "tax over 30k: " << high(37500) << endl;
    cout << "tax over 20k: " << middle(27500) << endl;
    return 0;
}
```

## lambda 实现

实际上，仿函数是编译器实现 lambda 的一种方式。通常编译器会把 lambda 函数转化成一个仿函数对象，因此在 C++11 中，lambda 可以被视为仿函数的一种等价形式。

比如一个 lambda 函数如下

```
[tax_rate](float price)->float {
    return price * (1- tax_rage/100);
}
```

等价于

```
/*
匿名函数名称由编译器自己决定
私有成员变量由捕获列表决定
重载的操作符内容由 lambda 内部函数体决定
*/
class Functor
{
private:
    float tax_rate;
public:
    float operator()(float price) {
        return price * (1- tax_rate/100);
    }
};
```

## lambda 基础使用

+ 相较于函数外定义的 static、inline 函数，lambda 函数并没有运行上的性能优势，当然也不会很差，因为 C++11 标准中默认 lambda 内联。
+ lambda 函数在代码的作用域上仅属于父作用域，所以 lambda 函数阅读性会好

lambda 函数最大的优势是在一些复杂的函数中，提供一些与父函数局部变量交互量比较大的私有函数，所谓的私有函数就是一些功能不能与其他代码共享，但是需要在一个函数内部重用。

示例如下：
```
int Prioritize(int)
{

}

int AllWorks(int times)
{
    int i;
    int x;
    try
    {
        for(i = 0; i < times; ++i) {
            x += Prioritize(i);
        }
    }
    catch(const std::exception& e)
    {
        x = 0;
    }

    const int y = [=]{
        int i, val;
        try
        {
            for(i = 0; i < times; ++i) {
                val += Prioritize(i);
            }            
        }
        catch(const std::exception& e)
        {
            val = 0;
        }
        return val;
        
    }();
}
```

在不使用函数的情况下，由于初始化要在运行时修改 x 的值，因此虽然 x 在初始化之后对于程序而言是一个常量，但不能被定义为 const，但是在定义 y 的时候，由于定义了 lambda 函数的调用，y 仅需要返回其值，于是常量性得到了保证。

## 关于 lambda 的一些问题和有趣实验

使用 lambda 函数的时候，捕获列表不同会导致不同的结果。
具体来讲，按值方式传递捕获列表和按引用方式传递捕获列表的效果是不一样的。
对于按值方式传递的捕获列表，其传递的值在 lambda 函数定义的时候就已经决定了，但是按引用传递的捕获列表，其传递的值等于 lambda 函数调用时的值。

具体代码如下：

```

int main()
{
    int j = 12;
    auto by_val_lambda = [=] {
        return j+1;
    };

    auto by_ref_lambda = [&] {
        return j + 1;
    };

    cout << "by val lambda : " << by_val_lambda() << endl;
    cout << "by ref lambda : " << by_ref_lambda() << endl;

    j++;
    cout << "by val lambda : " << by_val_lambda() << endl;
    cout << "by ref lambda : " << by_ref_lambda() << endl;

    return 0;
}
/**
by val lambda : 13
by ref lambda : 13
by val lambda : 13
by ref lambda : 14
*/
```

引用传递时函数使用父作用域中的值，但是值传递时传入值会被当作一个常数。
所以在使用 lambda 函数时，如果需要捕获的值成为 lambda 函数的常量，就是用值传递，而如果需要捕获的值成为 lambda 函数运行时的变量，就是用引用方式。

lambda 函数不是一个函数指针，但是 C++11 标准运行 lambda 表达是向函数指针转换，但是前提是 lambda 函数没有捕获变量，且函数指针所指的函数原型必须跟 lambda 函数有着相同的调用方式。

```
int main()
{
    int girls = 3, boys = 4;
    auto totalChild = [](int x, int y)->int{
        return x + y;
    };
    typedef int(*allchild)(int x, int y);
    typedef int(*onechild)(int x);
    allchild p;
    p = totalChild;

    onechild q;
    q = totalChild; // error，参数不一致

    decltype(totalChild) allPeople = totalChild;
    decltype(totalChild) toPeople = p;   // error，指针不能转换成 lambda

}
```

再来看一下 lambda 的 mutable 关键字

```
#include <iostream>
using namespace std;
int main()
{
    int val = 0;

    // 编译失败，在 const 的 lambda 中修改常量
    auto const_val_lambda = [=](){
        // val = 3;    // erro
    };

    // 非 const 的 lambda，可以修改常量数据
    auto mutable_val_lambda = [=]() mutable {
        val = 3;
    };

    // 依然是 const lambda，不过没有修改引用本身
    auto const_ref_lambda = [&]() mutable {
        val = 4;
    };

    // 依然是 const 的 lambda，通过参数传递 val
    auto const_param_lambda = [&](int v) {
        v = 5;
        cout << "v : " << v << endl;
    };

    return 0;
}
```

const_val_lambda() 函数编译失败的原因是与 lambda 的常量性有关，lambda 函数转换成仿函数后，会成为一个常量成员函数

```
class const_val_lambda
{
public:
    const_val_lambda(int v) : val(v){}

    void operator()() const {
        val = 3;
    }
private:
    int val;
};
```

常量成员函数内部是不能对类中的任何变量做修改的，因此编译错误就很合理了。

而使用引用方式传递变量在常量成员函数中值被更改则不会导致错误，这是因为在常量成员函数中没有修改引用本身，而是修改了引用的值。

## lambda 和 STL

lambda 对 C++11 最大的贡献是在 STL 库中，这个特性会使得 STL 算法部分更容易了。

比如一个常见的 STL 算法，for_each，原型如下：
`UnaryProc for_each(InputIterator beg, InputIterator end, UnaryProc op)`

该算法接收一个标记开始的 iterator，一个标记结束的 iterator，以及一个接受单个参数的“函数”，可以是一个函数指针，仿函数或者 lambda
一个简单版本的实现如下：

```
for_each(iterator begin, iterator end, Function fn) {
    for(iterator i = begin; i <= end; ++i) fn(*i);
}
```

借此算法，我们可以写出以下示例：

```
vector<int> nums;
vector<int> largeNums;

const int ubound = 10;

inline void LargeNumsFunc(int i) 
{
    if(i > ubound) largeNums.push_back(i);
}

void Above()
{
    // 传统的 for 循环
    for(auto itr = nums.begin(); itr != nums.end(); ++itr) {
        if(*itr >= ubound) largeNums.push_back(*itr);
    }

    // 使用函数指针
    for_each(nums.begin(), nums.end(), LargeNumsFunc);

    // 使用 lambda 函数和算法 for_each
    for_each(nums.begin(), nums.end(), [=](int i){
        if(i > ubound) largeNums.push_back(i);
    });
}
```

三种循环方式， for_each 比传统循环方式好在效率、正确性、可维护性上。
函数指针方式缺点是函数需要定义在别处，而且函数指针很可能涉及到函数调用的开销，即编译器不会对其进行 inline 优化。而且只局限于函数。



