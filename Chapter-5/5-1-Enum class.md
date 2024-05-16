# 强枚举类型

枚举是 C/C++ 一个内置类型，处于设计上简单的目的，枚举值常常对应到整型数值的一些名字

枚举类型可以是匿名的。
枚举的类型可以在编译期得到编译器额的检查

## 枚举的缺陷

具有名字的枚举以及它的成员，能够全局可见。这很容易产生冲突。
此外由于 C 中枚举常被设计为常量熟知的别名，导致枚举的成员可以被隐式地转化为整型。

## 强枚举

C++11 提供了强枚举类型，在 enum 后加上关键字 class
```
enum class
{
    General, Light, Medium, Heavy
};
```

强枚举有以下几点优势：
1. 强作用域，强枚举成员的名称不会被输出到父作用域空间
2. 转化限制，强类型枚举成员不可以与整型隐式地相互转化
3. 可以指定底层类型。强类型枚举默认地底层类型为 int，但是可以显示地指定底层类型，具体方法是在枚举名称后面加上 ":type"，其中 type 可以是除  wchar_t 外的任何整型

除此之外，匿名的强枚举类型是没有任何用的。

### 基本用法

```
#include <iostream>
using namespace std;

enum class Type
{
    General,
    Light,
    Medium,
    Heavy
};

enum class Categoty
{
    General = 1,
    Pistol,
    MachineGun,
    Gannon
};

int main()
{
    Type t = Type::Light;
    // t = General;    // 编译失败，必须使用强类型名称
    // if(t == Categoty::General)     // 编译失败，必须使用 Type 中的定义
    {
        cout << "General Wapon" << endl;
    }

    if(t > Type :: General) {
        cout << "Not General Weapon" << endl;
    }

    // if(t > 0) // 编译失败，无法隐式转化成 int
    {
        cout << "Not General Weapon" << endl;        
    }

    if((int) t > 0) {
        cout << "Not General Weapon" << endl; 
    }

    cout << is_pod<Type>::value << endl;        // 1
    cout << is_pod<Categoty>::value << endl;    // 1
    return 0;
}
```

1. 强制制止了隐式转化
2. 定义其与数值之间的关联使之能够默认拥有一种队成员排列的机制
3. 制止成员名字输出则进一步避免名字空间冲突问题
4. 是 POD 类型，不会像 class 一样被认为是结构体

### 指定底层类型

```
// 强制指定底层的基本类型为 char
enum class C : char
{
    C1 = 1,
    C2 = 2
};

enum class D : unsigned int
{
    D1 = 1,
    D2 = 2,
    Dbig = 0xFFFFFFF0U
};

int main()
{
    cout << sizeof(C::C1) << endl;

    cout << (unsigned int)D::Dbig << endl;
    cout << sizeof(D::D1) << endl;
    cout << sizeof(D::Dbig) << endl;
    return 0;
}

/**
1
4294967280
4
4
*/
```

## C++11 对 enum 进行了拓展

C++11 对原有的 enmu 进行了拓展
1. 底层类型方面，C++11 编译器默认指定，但是也可以显式指定，同 `enum class` 用法一致
2. 作用域，C++11 中枚举的成员名字会基础输出到父类作用域中，也可以在枚举类型定义的作用域生效