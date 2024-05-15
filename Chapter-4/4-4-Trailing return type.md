# 追踪返回类型

C++11 引入了一个新语法，追踪返回类型，该特性运行我们把函数的返回值移至参数声明之后，如下

```
template<typename T1, typename T2>
auto Sum(T1 & t1, T2 &t2) -> decltype(t1+t2)
{
    return t1+t2;
}
```

复合符号 `->decltype(t1+t2)` 被成为追踪类型。
原本函数的返回值位置由 auto 占据，这样就可以让编译器推导 Sum 函数模板的返回类型了。
而 auto 占位符和 `->return_type` 也是构成追踪返回类型函数的两个基本元素

##  使用追踪返回类型的函数

追踪返回类型函数和普通函数最大的区别是在于返回类型的后置。在一般情况下，普通函数的声明方式会明显简单于最终返回类型。

`int func(char* a int b);`

如果使用追踪返回类型的写法是

`auto func(char* a, int b) -> int`

1. 追踪返回类型返回类型不必写明作用域

```
class OuterType
{
    struct InnerType {int i;};
    InnerType GetInner();
    InnerType it;
};

auto OuterType::GetInner() -> InnerType{
    return it;
}
```

2. 返回类型后置，可以实现模板类型推导

```
template <typename T1, typename T2>
auto Sum(const T1& t1, const T2& t2) -> decltype(t1 + t2)
{
    return t1 + t2;
}

template <typename T1, typename T2>
auto Mul(const T1& t1, const T2& t2) -> decltype(t1 + t2)
{
    return t1 * t2;
}

int main()
{
    auto a = 3;
    auto b = 4L;
    auto pi = 3.14;

    auto c = Mul(Sum(a,b),pi);

    cout << c << endl;

    return 0;
}
```

3. 简化函数定义

```
int (*(*pf())())() {
    return nullptr;
}

// auto (*)() -> int(*)()   一个返回函数指针的函数，假设 a 为函数
// auto pf1() -> auto(*)()->int(*)()    一个返回 pf1 函数的指针的函数
auto pf1()->auto(*)()->int(*)() {
    return nullptr;
}

int main()
{
    cout << is_same<decltype(pf), decltype(pf1)>::value << endl;    // 1

    return 0;
}
```

4. 应用在转发函数中

```
double foo(int a) {
    return (double) a + 0.1;
}

int foo(double b) {
    return (int) (b);
}

template<class T>
auto Forward(T t) ->decltype(foo(t)) {
    return foo(t);
}

int main()
{
    cout << Forward(2) << endl;
    cout << Forward(0.5) << endl;
    return 0;
}
```

由于使用了追踪返回类型，可以实现参数和返回类型不同时的转发。

5. 应用于函数指针中

```
// 等价
auto (*fp)() -> int;
int (*fp)();

// 等价
auto (&fr)()->int;
int (&fr)();
```

除此以上描述的函数模板，普通函数，函数指针，函数引用之外，追踪返回类型还可以应用于结构或者类的成员函数，类模板的成员函数中。


