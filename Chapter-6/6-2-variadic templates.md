# 变长模板

## 变长函数和变长的模板参数

C 语言中一个变长函数是通过可变参数的宏实现，如下所示：

```
#include <stdio.h>
#include <stdarg.h>


/**
 * va_list: 字符类型指针，用于声明一个指向参数列表的指针
 * va_start: 该宏用于初始化 va_list 类型的变量，使其指向参数列表的第一个可变参数
 * va_arg: 用户获取参数列表的下一个参数，并将指针指向下一个参数
 * va_end: 清理 va_list 类型的变量
*/
double SumofFloat(int count, ...)
{
    va_list ap;
    double sum = 0;
    va_start(ap, count);
    for(int i = 0; i < count; i++) {
        sum += va_arg(ap, double);
    }
    va_end(ap);
    return sum;
}

int main()
{
    printf("%f\n", SumofFloat(3, 1.2f, 3.4, 5.6));
    return 0;
}
```

虽然这些宏提供了处理可变参数的能力，但是它们并不能获取可变参数的类型和数量，这些需要程序自身去控制；
此外编译器对变长参数的原型检查不够严格， 可能影响代码质量。

## C++ 实现变长函数

C++ 使用函数模板加上 tuple 模板类来实现变长函数。

### 变长模板：模板参数包和函数参数包

先看一个 tuple，tuple 是一个变长类模板

`template <typename... Elements> class tuple;`
标识符 Elements 之前使用省略号来表示该参数是变长的。

C++11 中，Elements 被称作是一个 “模板参数包”，这是一个新的模板参数类型，通过这种模板参数类型，类模板 tuple 就可以接受多个参数作为模板参数。

如实例化的 tuple 模板类

`tuple<int, char, double>`

编译器可将多个模板参数打包成为一个 “单个的” 模板参数包 Elements，即 Elements 在进行模板推导的时候，就是一个包含 int、char、和 double 三种类型的类型集合。

模板参数包也可以是非类型的

```
template<int...A> class NonTypeVariadicTemplate{};
NonTypeVariadicTemplate<1,0,2> vtvt;

// 上述声明等同于
template<int,int,int> class NonTypeVariadicTemplate{};
NonTypeVariadicTemplate<1,0,2> vtvt;
```

一个模板参数包在模板推导时会被认为是模板的单个参数，为了使用模板参数包，我们需要将其解包，C++11 中通常会通过一个包扩展的表达式来完成，如下
`template<typename...A> class Template : private B<A...>{};`

这里的 `A...` 就是一个包扩展，直观地看，参数包会在包扩展位置上展开为多个参数
```
tempalte<typename T1, typename T2> class B{};

// 声明了一个参数包 A，使用参数包 A... 是在 Template 的私有基类 B<A...> 中
template<typename...A> class Template : private B<A...>{};

// 声明了一个基类为 B<X,Y> 的模板 Template<X,Y> 的对象 xy
Template<X,Y> xy;
```

但是上述的包扩展并没有实现变长的功能，如果我们要声明一个三个变量的参数，就会导致模板推导失败。

C++11 中，使用数学归纳法来解决这个变长功能。

通过定义递归的模板偏特化定义，使得模板参数包在实例化时层层展开，知道参数包中的参数被逐渐耗尽或者到达某个数量的边界。如下

```
template <typename... Elements> class tuple;    // 变长模板的声明

template<typename Head, typename... Tail>       // 递归的偏特化定义
class tuple<Head, Tail...> : private tuple<Taile...>
{
    Head head;
}

template<> class tuple<>{};                 // 边界调解
```

这个代码中，先声明了变长模板类 tuple，其只包含了一个 Elements 模板参数包
又偏特化了一个双参数的 tuple 版本，该版本包含了两个参数，一个是类型模板参数 Head，另一个是模板参数包 Tail。其中 Head 作为 `tuple<Head, Tail...>` 的第一个成员，而将使用了包扩展表达式的模板 `tuple<Tail...>` 作为 `tuple<Head, Tail...>` 的私有基类。
上述的设计中，如果有一个形如 `tuple<double, int, char, float>` 类型被实例化时，就会先引起基类的递归构造，这样的递归构造在 tuple 的参数包为 0 时结束，因为我们定义了边界条件，即 `tuple<>` 这样不包含参数的偏特化版本造成的。

这样一来，编译器将从 `tuple<>` 构造成 `tuple<float>`，继而造出 `tuple<char, float>`, `tuple<int, char, float>`，最后构造出 `tuple<double, int, char, float>` 类型

再看非类型模板的例子

```
#include <iostream>

using namespace std;

template<long...nums> struct Multiply;

template<long first, long... last>
struct Multiply<first, last...>
{
    static const long val = first* Multiply<last...>::val;
};

template<>
struct Multiply<>
{
    static const long val = 1;
};

int main()
{
    cout << Multiply<2,3,4,5>::val << endl;
    cout << Multiply<22,44,66,88,9>::val << endl;
    return 0;
}
```

其实这里发生了编译期计算，最终在 main 中打印的是一个常量静态成员，而非是函数执行调用。也算是模板元编程范畴。

在模板参数包和函数参数包的两个概念支持下，我们就能够实现 C 中变长函数的功能了。

如下我们实现一个 C++11 中 printf 函数的例子

```
#include <iostream>
#include <stdexcept>

using namespace std;

void Printf(const char* s)
{
    while (*s)
    {
        if(*s == '%' && *++s != '%'){
            throw runtime_error("invalid format string: missing argument");
        }
        cout << *s++;
    }
}
template <typename T, typename... Args>
void Printf(const char* s, T value, Args... args)
{
    while (*s)
    {
        if(*s == '%' && *++s != '%') {
            cout << value;
            return Printf(++s, args...);
        }
        cout << *s++;
    }
    throw runtime_error("extra arguments provided to Printf"); 
    
}

int main()
{
    Printf("hello %s\n", string("world"));
    return 0;
}
```

## 变长模板：进阶

C++11 中，标准定义了 7 种参数包可以展开的位置：
1. 表达式
2. 初始化列表
3. 基类描述列表
4. 类成员初始化列表
5. 模板参数列表
6. 通用属性列表
7. lambda 函数的捕获列表

除此之外的地方，参数包是无法展开的。
而对于包拓展而言，解包也与其声明方式息息相关。事实上还可以声明一些有趣的包扩展表达式。
比如声明了 Args 为参数包，那么我们可以使用 Args&&... 这样的包扩展表达式，其解包后等价于 Arg1&&,...,Argsn&&（1，... ，表示参数下标，并非是参数名）

如下一个有趣的包扩展表达式

```
// 下面两个类模板声明的差别
template<typename... A>
class T : private B<A>...{}

template<typename... A>
class T : private B<A...>{}
```

它们解包后是不同的，对于同样实例化  T<X,Y>
```
// 前者解包为
class T<X,Y> : private B<X>, private B<Y>{};

// 后者解包为
class T<X,Y> : private B<X,Y>{};
```

类似的情况也发生在函数声明中，如下

```
template <typename ... T> void DummyWrapper(T ... t) {}

template <typename T> T pr(T t) {
    cout << t;
    return t;
}

template <typename... A>
void VTPrint(A... a)
{
    DummyWrapper(pr(a)...);     // 包扩展解包为 pr(1), pr(", ")..., pr(", abc\n")
}

int main()
{
    VTPrint(1, ", ", 1.2, ", abc\n");
    return 0;
}

/**
 * 输入结果，输出结果顺序被打乱
 * , abc
 * 1.2, 1
*/
```

### sizeof...

该关键字用来计算参数包中的参数个数

```
#include <iostream>
#include <stdexcept>
#include <cassert>
using namespace std;

template<class...A> void Print(A...arg)
{
    assert(false);
}

void Print(int a1, int a2, int a3, int a4, int a5, int a6)
{
    cout << a1 << ", " << a2 << ", " << a3 << ", " << a4 << ", " << a5 << ", " << a6 << endl;
}

template<class...A> 
int Vaargs(A...args)
{
    int size = sizeof...(A);
    switch (size)
    {
    case 0:
        Print(99,99,99,99,99,99);
        break;
    case 1:
        Print(99,99, args... ,99,99,99);
        break;
    case 2:
        Print(99,99, args... ,99,99);
        break;
    case 3:
        Print(args... ,99,99,99);
        break;
    case 4:
        Print(99, args... ,99);
        break;
    case 5:
        Print(99, args...);
        break;
    case 6:
        Print(args...);
        break;
    default:
        Print(0,0,0,0,0,0);
        break;
    }
    return size;
}

int main()
{
    Vaargs();
    Vaargs(1);
    Vaargs(1,2);
    Vaargs(1,2,3);
    Vaargs(1,2,3,4);
    Vaargs(1,2,3,4,5);
    Vaargs(1,2,3,4,5,6);
    Vaargs(1,2,3,4,5,6,7);

    return 0;
}

/**
99, 99, 99, 99, 99, 99
99, 99, 1, 99, 99, 99
99, 99, 1, 2, 99, 99
1, 2, 3, 99, 99, 99
99, 1, 2, 3, 4, 99
99, 1, 2, 3, 4, 5
1, 2, 3, 4, 5, 6
0, 0, 0, 0, 0, 0
*/
```

下面介绍一下使用模板做参数包跟使用类型和非类型的模板参数包。

```
// 使用模板作为变长模板参数包的一个变长模板
template<typename I, template<typename> class... B> struct Container{};


// 递归的偏特化定义
// 使用 I 做参数实例化模板参数 template<typename> class A
// Container<I,B...> b 则保证编译器会继续递归地推导下去，当模板参数包为空时，边界条件就会起作用
template<typename I, template<typename> class A, template<typename> class... B>
struct Container<I, A, B>
{
    A<I> a;
    Container<I, B...> b;
} 

template<typename I> struct Container<I>{};
```

看一个变长模板参数和完美转发的例子



