# 常量表达式

## 运行时常量和编译时常量

const 关键字通常用于修饰运行时常量，即在运行时数据的不可更改性。

但是如果我们需要编译时期的常量怎么办呢？

比如：

```
#include <iostream>

const int GetConst()
{
    return 1;
}

void Constless(int cond)
{
    // C++11 环境中竟然可以通过编译
    int arr[GetConst()] = {0};  // 不能通过编译
    enum {
        // e1 = GetConst(),   // 不能通过编译
        e2
    };
    switch (cond)
    {
    // case GetConst():   // 不能通过编译
        break;
    
    default:
        break;
    }

}

int main()
{
    Constless(1);
    return 0;
}
```

const 只保存了运行时常量，这和编译时常量是两回事，再看下面一个示例

```
enum BitSet {
    V0 = 1 << 0,
    V1 = 1 << 1,
    V2 = 1 << 2,
    VMAX = 1 << 3
};

const BitSet opeator|(BitSet x, BitSet y)
{
    // 使用 & (VMAX - 1) 确保 或操作 不会超出 VMAX 枚举值
    return static_cast<BitSet> (((int) x | y) & (VMAX - 1));
}

// 无法通过编译
// template <int i = V0 | V1> void LinkConst(){}
```

## constexpr 常量表达式

C++11 提供 constexpr 关键字来实现编译时期的常量

`constexpr int GetConst {return 1;}`

编译器能够在编译时期就对 GetConst() 表达式进行值计算。这样以来上述中的代码就能够通过该关键字实现目的了。

该关键字不仅能够作用函数，还能作用函数声明，类的函数构造等

### 常量表达式函数

函数返回类型前加上关键字就能使该函数变成常量表达式函数，但是有以下要求的函数才能被 constexpr 关键字修饰

1. 函数体只有单一的 return 返回语句
2. 函数体必须由返回值，不能是 void
3. 在使用前必须有定义
4. return 返回语句表达式中不能使用非常量表达式的函数、全局数据，且必须是一个常量表达式

#### 单一的 return 返回语句

```
// 多条语句写法不能通过编译
constexpr int data(){
    const int i = 1;
    return i;
}
```

但是有一些不会产生实际代码的语句能够在常量表达式函数中使用
```
constexpr int f(int x)
{
    static_assert(0 == 0 ? "assert fail.");
    return x;
}
```

其他如 using，typedef 等通常也不会出现问题

#### 常量表达式函数必须由返回值

```
constexpr void f();
```

这种函数就不能是常量表达式，因为无法获取常量

#### 使用前必须定义

对于普通函数来说，调用函数只需要有函数声明即可。
但是常量表达式函数的使用不同，需要定义，也就是实现

```
constexpr int f();
int a = f();
const int b = f();
constexpr int c = f();  // compile error
constexpr int f() {
    return 1;
}
constexpr int d = f();
```

a,b 定义中编译器会将 f() 转化成一个函数调用，而在 c 的定义中，由于是一个常量表达式值，因此会要求编译器进行编译时计算，由于此时 f 常量表达式并没有定义，所以会导致编译失败。

无需给一个函数写两个带和不带 constexpr 关键字的版本
```
// 编译报错
constexpr int f();
int f();
```

#### return 返回语句必须是一个常量表达式

常量表达式中，不能使用非常量表达式的函数

```
const int e() {
    return 1;
}
constexpr int g(){
    return e();
}
```

或者
```
int g = 3;
constexpr int h() {
    return g;
}
```

这两种定义都是无法通过编译的。因为 return 语句不能包含运行时才能确定返回值的函数，这样编译器才能在编译期进行常量表达式函数的值计算。

一些危险操作比如赋值，在表达式中都是不允许的

```
constexpr int k(int x){
    return x = 1;
}
```

### 常量表达式值

常量表达式值必须在使用前被初始化。

```
const int i = 1;
constexpr int j = 1;
```

它们有什么区别？
在绝大多数情况下是没有区别的。
但是如果 i 位于全局名字空间中，编译器会为它产生数据，而对于 j 如果没有代码显式地使用它的地址，那么编译器可以选择不为 j 产生数据，而仅当作编译器时期的值。即只有名字而不产生数据。

此外需要再提一下浮点常量，编译器对浮点数做编译时期常量这件事很敏感，因为编译时环境和运行时环境可能会有不同，这会导致编译时期的浮点常量和实际运行时的浮点常量可能存在精度上的差别。
但是在 C++11 中，编译时的浮点数常量表达式时运行的，标准要求编译时的浮点常量表达式的精度要等于或者高于运行时的浮点数常量的精度。

C++11 中，constexpr 关键字是不能修饰自定义类型的定义。
比如以下用法是有问题的
```
constexpr struct MyType {int i;}
constexpr MyType mt = {0};
```

正确的做法是自定义常量构造函数，如下

```
struct MyType {
    constexpr MyType(int x) : i(x){}
    int i;
};

constexpr MyType mt = {0};
```

常量表达式的构造函数也有使用上的约束
1. 函数体必须为空
2. 初始化列表只能由常量表达式赋值

形如以下的常量表达式构造函数是无法编译通过的

```
int f();
struct MyType{
    int i;
    constexpr MyType() : i(f()){}
}
```

虽然我们声明的是常量表达式构造函数，但是在编译时的常量性体现在类型上

```
#include <iostream>
using namespace std;

struct Date
{
    constexpr Date(int y, int m, int d) : year(y), month(m), day(d){}
    constexpr int getYear() {
        return year;
    }
    constexpr int getMonth() {
        return month;
    }
    constexpr int getDay() {
        return day;
    }
private:
    int year;
    int month;
    int day;
};

constexpr Date PRCfound {1949, 10,1};
constexpr int foundmonth = PRCfound.getMonth();

int main()
{
    cout << foundmonth << endl;
    return 0;
}
// g++ const.cpp -o const -std=c++11
```

C++11 中不允许常量表达式作用于 virtual 的成员函数，这个原因是显而易见的，virtual 的目的就是运行时的行为，这与可以在编译时进行值计算的目的冲突

常量表达式构造函数也可以用于非常量表达式的类型构造，重写了编译器也不会报错。

### 常量表达式的其他应用

常量表达式还可以用于模板函数，但是由于模板函数的类型不确定性，所以模板函数是否被实例化成一个能够满足编译时常量性的版本通常也是未知的。

C++11 规定，当声明为常量表达式的模板函数后，某个该模板函数的实例化结果不满足常量表达式的需求时，constexpr 会被忽略，转成一个普通函数

```
struct NotLiteral {
    NotLiteral() {
        i = 5;
    }
    int i;
};

NotLiteral nl;

template <typename T> constexpr T ConstExp(T t) {
    return t;
}

void g()
{
    NotLiteral nl;
    NotLiteral nl1 = ConstExp(nl);
    // constexpr NotLiteral nl2 = ConstExp(n1); // error
    constexpr int a = ConstExp(1);
}
```

常量表达式遇到函数递归时，会发生什么？
C++11 标准没有反对常量表达式的递归函数，而是在标准说明，符合 C++11 标准的编译器常量表达式函数应该至少支持 512 层递归。
所以我们可以利用该特性将编译器改造成一个计算器。

```
constexpr int Fibonacci(int n) {
    return (n == 1) ? 1 : ((n == 2) ? 1 : Fibonacci(n-1) + Fibonacci(n-2));
}

int main()
{
    int fib[] = {
        Fibonacci(11), Fibonacci(12),
        Fibonacci(13), Fibonacci(14),
        Fibonacci(15), Fibonacci(16)
    };

    for( int i : fib ) cout << i << endl;

    return 0;
}
```

利用一个常量表达式构造一个数组，听过汇编发现该数组的值已经被计算好了，实际运行的代码是没有调用 Fibonacci() 函数的。

这种基于编译时期的运算的编程方式也出现在模板中。
模板元编程中也可以在编译时期进行运算的编译方式

```
#include <iostream>
using namespace std;

template <long num>
struct Fibonacci
{
    static const long val = Fibonacci<num - 1>::val + Fibonacci<num-2>::val;
};

template <> struct Fibonacci<2> {static const long val = 1 ;};
template <> struct Fibonacci<1> {static const long val = 1 ;};
template <> struct Fibonacci<0> {static const long val = 0 ;};

int main()
{
    int fib[] = {
        Fibonacci<11>::val, Fibonacci<12>::val,
        Fibonacci<13>::val, Fibonacci<14>::val,
        Fibonacci<15>::val, Fibonacci<16>::val
    };

    for( int i : fib ) cout << i << endl;

    return 0;
}
```

在上述代码中，我们定义了一个非类型参数的模板 Fibonacci，该模板定义了一个静态变量 val，而 val 定义方式是递归的，因此模板将会递归地进行推导
此外，还通过偏特化定义了模板推导地边界条件，即斐波那契的初始值，当模板推导到边界条件的时候就会终止推导。
通过这种方式，同样能够实现在编译时进行值计算，从而生成数组。

但是需要注意的是，并不是所有使用了 constexpr 的常量表达式函数都会在编译时期被编译器计算。
标准知识定义了可以用于编译时进行值运算的常量表达式的定义，却没有强制要求编译器一定在编译时进行值计算。