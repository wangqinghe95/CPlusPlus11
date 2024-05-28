# 指针空值

## 指针空值：从 0 到 NULL，再到 nullptr

将一个变量及时的初始化是一个良好的编程习惯。尤其是对于指针类型来说，因为悬空的指针很容易产生一些问题。

指针一般会初始化为 0。地址 0 在大多数参数系统中属于内核系统的地址，所以如果使用指向地址 0 的指针做解引用时会被报错，这样可以很容易的找到问题所在。

C 中，使用了宏定义 NULL 来用作空指针的初始化值。NULL 在 C 中的定义如下
```
#undef NULL
#if defined(_cplusplus)
#define NULL 0
#else
#define NULL ((void*)0)
#endif
```

NULL 可能是字面常量 0，也可能是无类型指针(void*)常量。
这样的设定会带来一些问题，比如：

```
#include <stdio.h>

void f(char* c)
{
    printf("invoke f(char*)\n");
}

void f(int i )
{
    printf("invoke f(int)\n");
}

int main()
{
    f(0);
    // f(NULL);    // complile error
    f((char*) 0);
    return 0;
}
```

`f(NULL)` 编译失败的原因是 NULL 的二义性。字面量 0 的类型既可以是一个整型，又可以是一个无类型指针。如果想要调用 `f(char*c)` 版本，就需要将字面常量 0 强转成对应的指针类型，否则编译器就会优先将 0 当前整型常量。

就上述的问题，C++11 给了一个解决方案，就是引入一个新的常量，即 nullptr，表示指针空值类型。定义如下
`typedef decltype(nullptr) nullptr_t`

nullptr 是有类型的，且仅可以被隐式转化成指针类型，这样以来，上述的代码就能被修改为

```
int main()
{
    f(nullptr);     // 调用指针类型参数版本
    f(0);           // 调用常量类型版本
}
```

## nullptr 与 nullptr_t

nullptr 是指针空值常量，nullptr_t 是 指针空值类型，也就是说，指针空值类型并非仅有 nullptr 一个实例，也可以通过 nullptr_t 声明一个指针空值类型的变量。

除去 nullptr 和 nullptr_t 以外， C++ 还有其他各种内置类型，C++11 标准严格规定了数据间的关系，大概的规则如下：

1. 所有定义为 nullptr_t 类型的数据都是等价的，行为也是一致的；
2. nullptr_t 类型数据可以隐式转化成任一一个指针类型
3. nullptr_t 类型数据不能转化成非指针类型，即使是使用 reinterpret_cast<nullptr_t>() 的方式也是不可以的
4. nullptr_t 类型数据不适用于算术运算表达式
5. nullptr_t 类型数据可以用于关系运算表达式，但是仅能于 nullptr_t 类型数据或者指针类型数据进行比较，当且仅当运算符为 ==，<=, >= 等时返回 true

例子如下：

```
#include <iostream>
#include <typeinfo>

using namespace std;

int main()
{
    // nullptr 可以隐式转化成 char*
    char *cp = nullptr; //

    // 不可转化成整型，任何类型也不能转化成 nullptr_t
    // int n1 = nullptr;   //
    // int n2 = reinterpret_cast<int>(nullptr);    //

    // nullptr 和 nullptr_t 类型变量可以做比较
    // 当使用 ==，<=，>= 符号比较时可以返回 true
    nullptr_t nptr;
    if(nptr == nullptr) {
        cout << "nullptr_t nptr == nullptr" << endl;
    }
    else {
        cout << "nullptr_t nptr != nullptr" << endl;
    }

    if(nptr < nullptr) {
        cout << "nullptr_t nptr < nullptr" << endl;
    }
    else {
        cout << "nullptr_t nptr !< nullptr" << endl;
    }

    // 不能转化成整型或者 boo 型
    // if(0 == nullptr);
    // if(nullptr);

    // 不可进行算术运算
    // nullptr += 3;
    // nullptr *=5;

    sizeof(nullptr);
    typeid(nullptr);
    throw(nullptr);

    return 0;
}
```

在模板类中，模板只能将 nullptr_t 当为一个普通的类型，并不会将其视为 T* 指针

```
template <typename T>
void g(T* t) {}

template <typename T>
void h(T t) {}

int main()
{
    g(nullptr);     // error，nullptr 的类型是 nullptr_t，而不是指针
    g((float*) nullptr);    // 推导出 T = float

    h(0);       // 推导出 T = int
    h(nullptr);       // 推导出 T = nullptr_t，而不是指针
    h((float*)nullptr);     // 推导出 T = float*

    return 0;
}
```

g(nullptr) 并不会编译器智能地推导成某种类型的指针，因此要让编译器成功推导成 nullptr 类型，必须做显示的类型转换。

## 一些关于 nullptr 规则的讨论

C++11 中，nullptr 类型数据所占用的内存空间大小和 void* 相同，即
`sizeof(nullptr_t) == sizeof(void*)`

nullptr 是一个编译时期的常量，能够被编译器识别，而 (void*)0 只是一个强制转化表达式，其返回也是一个 void* 指针类型

nullptr 到任何指针的转化都是隐式的，而（void*） 则必须经过类型转化后才能使用

如下：

```
int foo()
{
    int *px = (void*)0; // error，不能隐式地将无类型指针转化成 int* 类型的指针
    int *py = nullptr;
}
```

nullptr_t 对象的地址可以被用户使用，但是 nullptr 不能被用户获取地址，因为它被定义为一个右值常量，取地址没有意义。
但是 C++ 标准没有禁止声明一个 nullptr 的右值隐隐，并打印其地址值。

```
int main()
{
    nullptr_t my_null;
    printf("%x\n", &my_null);

    // printf("%x\n", &nullptr);    // error, C++11 规定，不能打印 nullptr 地址
    printf("%d\n", my_null == nullptr);

    const nullptr_t && default_nullptr = nullptr;   // nullptr 的一个右值引用
    printf("%x\n", &default_nullptr);
}
```