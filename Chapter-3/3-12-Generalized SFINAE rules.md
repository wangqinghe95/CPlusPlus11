# 一般化的 SFINAE 规则

SFINEA，即 Substitution failure is not an error，即匹配失败不是错误

这条规则主要是针对重载模板的参数进行展开的时候设置的，如果在这个过程中导致了一些类型不匹配，编译器不会报错。

```
#include <iostream>
using namespace std;
struct Test
{
    typedef int foo;
};

template <typename T>
void f(typename T::foo) {
    cout << __func__ << " " << __LINE__ << endl;
}

template <typename T>
void f(T) {
    cout << __func__ << " " << __LINE__ << endl;
}

int main()
{
    /**
     * 实例化 Test::foo 类型
    */
    f<Test>(10);

    /*
    * 虽然不存在 int::foo 类型，但是编译器不会报错
    * 编译器会找到第二个模板定义并对其进行实例化
    */
    f<int>(10);

    return 0;
}
```

通过 SFINAE，程序可以使得模板匹配更加精准，即使一些模板函数、模板类在实例化的时候是使用特殊的模板版本，而另外一些则使用通用的版本，这些就可以增加模板设计使用上的灵活性。