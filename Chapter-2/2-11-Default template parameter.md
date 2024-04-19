# 模板函数的默认模板参数

C++11 支持模板有默认参数。
为多个默认模板参数声明指定默认值时，使用者必须遵循“从右向左”的规则进行指定

## 模板函数和模板类的默认参数对比
```
template<typename T1, typename T2 = int> class DefClass1;   // pass
template<typename T1 = int, typename T2> class DefClass1;   // failed

template<typename T, int i = 0> class DefClass3;   // pass
template<int i = 0, typename T> class DefClass1;   // failed

template<typename T1 = int, typename T2> void DefFunc1(T1 a, T2 b);   // pass
template<int i = 0, typename T>  void DefFunc1(T a);
```

定义默认类模板参数的模板类需要遵守顺序规则；对于函数模板来说，默认模板参数不需要遵守参数顺序。

## 模板函数参数使用介绍

```
template <class T, class U = double>
void f(T t = 0, U u = 0);

void g()
{
    f(1,'c');   // f<int, char>(1,'c);
    f(1);       // f<int, double>(1,0);
    f();        // failed
    f<int>();   // f<int, double>(0,0);
    f<int, char>(); // f<int, char>(0,0);
}
```

模板模板参数通常是搭配默认函数参数一起使用。
模板函数的默认形参不是模板参数推导的依据，都是通过实参类型推导过来的。