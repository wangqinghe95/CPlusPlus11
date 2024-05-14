# decltype

在 C++ 发展的过程中，类型推导随着模板和泛型的广泛使用而引入。
在泛型的编程中，类型成了未知数，这个缺点会限制模板的使用范围和编写方式。
引入类型推导可以让编译器辅助地进行类型推导，decltype 就是因此而出。

## 基础使用

```
#include <iostream>
#include <typeinfo>

using namespace std;

int main()
{
    int i;
    decltype(i) j = 0;
    cout << typeid(j).name() << endl;   // i => int

    float a;
    double b;
    decltype(a+b) c;
    cout << typeid(c).name() << endl;   // d => double

    return 0;
}
```

## decltype 应用

1. 搭配 using 使用

decltype 典型应用就是 decltype 与 typedef/using 配合使用

比如 C++11 头文件中的一些定义
```
using size_t = decltype(sizeof(0));
using ptrdiff_T = decltype((int*)0 - (int*)0);
using nullptr_t = decltype(nullptr);
```

在一些常量，基本类型，运算符和操作符都已经被定义好的情况下，然后通过类型推导，再使用 using 取别名，达成之前类型拓展需要将拓展类型映射到基本类型的常规做法。

2. 简化代码

```
int main()
{
    vector<int> vec;

    // 定义 vector iterator
    typedef decltype(vec.begin()) vectype;

    for(vectype i = vec.begin(); i < vec.end(); i++) {
        // do something
    }

    for(decltype(vec)::iterator i = vec.begin(); i < vec.end(); ++i) {
        // do something
    }

    return 0;
}
```

当我们遇到一些复杂类型的表达式或者变量时，我们可以利用 decltype 和 typedef/using 来将其转换成简单的表达，提高代码写作时的可读性和可维护性。

3. 匿名类型重用

```
enum  {
    K1, K2, K3
}anon_e;

union {
    decltype(anon_e) key;
    char* name;
}anon_u;

struct {
    int d;
    decltype(anon_u) id;
}anon_s[100];

int main()
{
    decltype(anon_s) as;
    as[0].id.key = decltype(anon_e)::K1;

    return 0;
}
```

使用了三种匿名结构，匿名的枚举类型（强类型枚举在 C++11 中是不允许存在的），匿名的联合体，以及匿名的结构体数组。
在这三个匿名类型中，decltype 都可以推导其类型并进行重用。
不过匿名都有匿名的理由，一般匿名的类型都是不希望该类型继续被重用，decltype 只是提供一种重用的可能。

### decltype 在模板和泛型中的使用

1. 在模板中使用
```
template <typename T1, typename T2>
void Sum(T1 &t1, T2& t2, decltype(t1+t2) &s) {
    s = t1 + t2;
}

int main()
{
    int a = 3;
    long b = 5;
    float c = 1.0f, d = 2.3f;

    long e;
    float f;
    Sum(a,b,e);     // s 类型被推导为 long 
    Sum(c, d, f);   // s 类型被推导为 double

    return 0;
}
```

因为不在有返回值类型，所以该函数的适用范围还在增加。
但是该函数强要求使用者知道 s 这个结果用什么来保存会最合适。

2. 需要注意特化版本
但是由于编译器的一些限制，需要开发人员对一些特殊情况来提供其他版本，比如 a，b 是一个数组的话，t1 + t2 就不是一个合法的表达式

```
template <typename T1, typename T2>
void Sum(T1 &t1, T2& t2, decltype(t1+t2) &s) {
    s = t1 + t2;
}

void Sum(int a[], int b[], int c[]) {
    // do something
}

int main()
{
    int a[5], b[5], c[5];
    Sum(a,b,c);     // 选择数组版本

    int d,e,f;
    Sum(d, e, f);   // 选择模板的实例化版本

    return 0;
}
```

3. 实例化模板使用

```
int hash(char*);

map<char*, decltype(hash)> dict_key;    // error
map<char*, decltype(nullptr)> dict_key;  
```

实例化了 map 模板，因为 map 是为了存储字符串以及其对应哈希值。
需要注意的是 decltype 只能接收表达式做参数，函数名做参数表达式是无法通过编译的。

4. C++11 中的使用

C++11 的一些标准库实现也是依赖于 decltype 的推导

如下

```
typedef double (*func)();
int main() {
    result_of<func()>::type f;  // double
}
```

result_of 没有调用 func() 这个函数，但是底层使用了 decltype
该模板的一种实现方式为
```
template<class>
struct result_of

template<class F, class... ArgTypes>
struct result_of<F(ArgsTypes...)>
{
    typedef decltype(std::declval<F>()(std::declval<ArgTypes>()...)) type;
}
```

在这里，标准库将 decltype 作用于函数调用上，并将函数调用表达式返回的类型 typedef 为一个名为 type 的类型。这样就能将代码中的表达式推导为 double

## decltype 推导规则

当通过 decltype(e) 来推导 e 的类型时，编译器依序判断有以下 4 种规则

### 规则介绍

1. 如果 e 是一个没有带括号的标记符表达式或者类成员访问表达式，那么 decltype(e) 就是 e 所命名的实体的类型。此外如果 e 是一个被重载的函数，则会导致编译时错误
2. 否则 ，假设 e 的类型是 T，如果 e 是一个将亡值，那么 decltype(e) 是 T&&
3. 否则 ，假设 e 的类型是 T，如果 e 是一个左值，那么 decltype(e) 是一个 T&
4. 否则，假设 e 的类型是 T，decltype(e) 是 T

解释以下标记符表达式，除去所有关键字，字面量等编译器需要使用的标记之外的自定义标记都可以是标记符。而单个标记符对应的表达式就是标记符表达式。

### 代码示例

以例子来解释上述的规则

```
int main()
{
    int i = 4;
    int arr[5] = {0};
    int *ptr = arr;

    struct S { double d;}s;

    void Overloaded(int);
    void Overloaded(char);

    int&& RvalRef();

    const bool Func(int);

    // 规则 1，单个标记符表达式以及访问类成员，推导为本类型
    // 该规则还适用于指针，数组，结构体，甚至函数类型的推导
    decltype(arr) val1;         // int[5]，标记符表达式
    decltype(ptr) var2;         // int*，标记符表达式
    decltype(s.d) var3;         // double，标记符表达式
    // decltype(Overloaded) var5;  // 无法通过编译，是一个重载函数

    // 规则 2，将亡值，推导的类型为右值引用
    decltype(RvalRef()) var6 = 1;   // int&&

    // 规则 3，左值，推导的类型为左值引用
    // 一个左值规则，当参数不是一个标志表达式或者类成员访问表达式，且参数都是左值，那么推导出来的类型均为左值引用
    decltype(true ? i : i) var7 = i;   // int&，三元运算符，这里返回一个 i 的左值
    decltype((i)) var8 = i;             // int&，带圆括号的左值
    decltype(++i) var9 = i;             // int&，++i，返回 i 的左值
    decltype(arr[3]) var10 = i;           // int&，[] 操作返回左值
    decltype(*ptr) var11 = i;             // int&，* 操作返回左值
    decltype("lval") var12 = "lval";      // const char(&)[9]，字符串字面常量左值

    // 规则 4，以上都不是，推到为本类型
    decltype(1) var13;             // int，除字符串外字面常量为右值
    decltype(i++) var14;            // int，i++ 返回右值    
    decltype((Func(1))) var15;      // const bool，圆括号可以忽略

    return 0;
}
```

### 辅助判断的模板库

C++ 标准库模板类 is_lvalue_reference 可以帮助做一些推导结果的识别

```
int main()
{
    int i = 4;
    int arr[5] = {0};
    int *ptr = arr;
    int&& RvalRef();

    cout << is_rvalue_reference<decltype(RvalRef())>::value << endl;      // 1

    cout << is_lvalue_reference<decltype(true ? i : i)>::value << endl;      // 1
    cout << is_lvalue_reference<decltype((i))>::value << endl;      // 1
    cout << is_lvalue_reference<decltype(++i)>::value << endl;      // 1
    cout << is_lvalue_reference<decltype(arr[3])>::value << endl;      // 1
    cout << is_lvalue_reference<decltype(*ptr)>::value << endl;      // 1
    cout << is_lvalue_reference<decltype("lval")>::value << endl;      // 1

    cout << is_lvalue_reference<decltype(i++)>::value << endl;      // 0
    cout << is_rvalue_reference<decltype(i++)>::value << endl;      // 0

    return 0;
}
```

## cv 限制符和冗余的符号

cv 限制符指的是变量 const 和 volatile 属性，auto 类型推导不会带走这个属性，但是 decltype 可以推导出。
但是如果对象定义种有 cv 限制符时，使用 decltype 推导时，其成员是不会继承 cv 属性的。

```
int main()
{
    const int ic = 0;
    volatile int iv;
    struct S {
        int i;
    };

    const S a = {0};
    volatile S b;
    volatile S*p = &b;

    cout << is_const<decltype(ic)>::value << endl;      // 1
    cout << is_volatile<decltype(iv)>::value << endl;      // 1

    cout << is_const<decltype(a)>::value << endl;      // 1
    cout << is_volatile<decltype(b)>::value << endl;      // 1

    cout << is_const<decltype(a.i)>::value << endl;      // 0
    cout << is_volatile<decltype(p->i)>::value << endl;      // 0

    return 0;
}
```

decltype 从表达式推导出类型后，进行类型定义时，会运行一些冗余的符号，比如 cv 限制符以及引用符号，通常情况下推导出的类型已经有了这些属性，冗余符号会被忽略。

```
int main()
{
    int i = 1;
    int &j = i;
    int *p = &i;
    const int k = 1;

    decltype(i) & var1 = i;
    decltype(j) & var2 = i;     // 冗余的 & 会被忽略

    cout << is_lvalue_reference<decltype(var1)>::value << endl;     // 1， 是左值引用
    cout << is_rvalue_reference<decltype(var2)>::value << endl;      // 0， 不是右值引用    
    cout << is_lvalue_reference<decltype(var2)>::value << endl;     // 1， 是左值引用

    // decltype(p) * var3 = &i;     // error
    decltype(p) * var4 = &p;        // var4 的类型是 int**
    auto *v3 = p;                   // v3 的类型是 int*
    v3 = &i;

    const decltype(k) var4 = 1;     // 冗余的 const 被忽略

    return 0;
}
```

