# 局部和匿名类型做模板实参

在 C++98 中不允许局部或者匿名的类和变量做模板类和模板函数，但是 C++11 支持了该项特性

```
#include<cstdio>

template <typename T> class X{};

template <typename T> void TempFunc(T t) {};

struct A{} a;
struct {int i;} b;          // b 匿名类型变量
typedef struct {int i;} B;  // B 是匿名类型

int main()
{
    struct C
    {
        int i;
    }c;     // 局部类型

    X<A> x1;        // 全局类型结构体作为模板类参数
    X<B> x2;        // 全局类型匿名结构体作为模板类参数
    X<C> x2;        // 局部类型结构体作为模板类参数

    TempFunc(a);    // 全局变量作为模板函数变量
    TempFunc(b);    // 全局匿名类型变量作为模板函数变量
    TempFunc(c);    // 局部类型变量作为模板函数变量

    // C++11 不通过
    /*
    X<struct {int a;}> t;   // 不符合 C/C++ 语法
    */

    return 0;    
}
```