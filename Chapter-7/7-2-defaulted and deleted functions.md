# 默认函数控制

C++ 自定义的类，编译器会默认生成一些代码为显示声明的成员函数，它们被称为默认函数，有：

+ 构造函数
+ 拷贝构造函数
+ 拷贝赋值函数（operator =）
+ 移动构造函数
+ 移动拷贝构造函数
+ 析构函数

此外，C++ 编译器还会为这些自定义类型提供全局默认的操作符函数

+ operator
+ operator &
+ operator &&
+ operator *
+ operator ->
+ operator ->*
+ operator new
+ operator delete

## default

当代码中实现了这些函数的自定义版，那么编译器就不再会为该类自动生成默认版本。

如果声明了自定义版本的构造函数，那么会有可能导致定义的类型不再是 POD 类型了。如下

```
#include <iostream>
#include <type_traits>

using namespace std;

class TwoCstor
{
public:
    // 提供了带参数版本的构造函数，则必须提供不带参数版本的
    // TwoCstor 不再是 POD 类型了
    TwoCstor(){}
    TwoCstor(int i) : data(i){}
private:
    int  data;
};

int main()
{
    cout << is_pod<TwoCstor>::value << endl;    // 0
    return 0;
}
```

当一个类型不再是 POD 类型后，编译器就无法再优化这种简单的数据类型。所以我们客观上需要通过一些方式恢复 POD 的特性

方法很简单，就是使用 default 修饰无参版本

`TwoCstor() = default;`

显示缺省不仅能够在类的定义中修饰成员函数，还可以在类定义之外修饰成员函数

```
#include <iostream>

using namespace std;

class DefaultedOptr
{
public:
    DefaultedOptr() = default;
    DefaultedOptr& operator= (const DefaultedOptr&);
};

inline DefaultedOptr&
DefaultedOptr::operator= (const DefaultedOptr& ) = default;

int main()
{
    return 0;
}
```

这个特性很有用，因为我们可以对一个类定义提供多个实现版本，假设我们有以下几个文件

```
type.h : struct type {type();};
type1.cpp : type::type() = default;
type2.cpp : type::type() {} 
```

使用者就可以根据自己的需要选择性的编译需要的文件，从而轻易地提供缺省函数地实现。

## delete

有时我们需要限制一些默认函数的生成，比如禁止使用者使用拷贝构造函数。以前的做法是将拷贝构造函数声明为 private，并且步提供函数实现。这样如果有人试图调用拷贝构造函数时，编译器就会报错阻止此类行为。

C++11 提供 delete 关键字，来指示编译器不生成函数的缺省版本。

```
#include <iostream>
#include <type_traits>
using namespace std;

class NoCopyCstor
{
public:
    NoCopyCstor() = default;
    NoCopyCstor(const NoCopyCstor &) = delete;
};

int main()
{
    NoCopyCstor a;
    // NoCopyCstor b(a);    // error

    return 0;
}
```

=delete 删除拷贝构造函数的缺省版本实例，并且一旦缺省版本被删除了，重载该函数也是非法的。

除此之外，显示删除还可以避免编译器做一些不必要的隐式数据类型转化，如下：

```
class ConvType
{
public:
    ConvType(int i) {}
    ConvType(char c) = delete;  // 删除 char 版本
};


void Func(ConvType ct) {}

int main()
{
    Func(1);

    // 显示地删除掉字符类型的构造函数，隐式转化隐式不被允许的
    // Func('a');  // error

    ConvType c1(3);
    // ConvType cc('a');   // error

    return 0;
}
```

另外需要注意一下 explicit 对于 delete 的影响

```
#include <iostream>
#include <type_traits>
using namespace std;

class NoCopyCstor
{
public:
    NoCopyCstor() = default;
    NoCopyCstor(const NoCopyCstor &) = delete;
};

int test1()
{
    NoCopyCstor a;
    // NoCopyCstor b(a);    // error

    return 0;
}


class ConvType
{
public:
    ConvType(int i) {}

    explicit ConvType(char c) = delete;  // 删除 char 版本
};

void Func(ConvType ct) {}

int main()
{
    Func(1);

    // 当使用 explicit 修饰 delete 的构造函数后，隐式转化的代码就能够通过编译了
    Func('a');

    ConvType c1(3);
    // ConvType cc('a');   // error

    return 0;
}
```

使用显示删除来禁止编译器做一些不必要的类型转化，这个特点其实并不局限于缺省版本的类成员函数或者全局函数，对于一些普通函数仍然可以使用显示删除来禁止类型转换，如下：

```
void func(int i){}
void func(char c) = delete; // 显示删除 char 版本

int main()
{
    func(3);
    // func('a');  // error
    return 0;
}
```

显示删除还可以有一些其他用于，比如修饰自定义的 operator new 操作符，就能够做到避免在堆上分配该类的对象。

```
class NoHeapAlloc
{
public:
    void* operator new(std::size_t) = delete;
};

int main()
{
    NoHeapAlloc nha;
    // NoHeapAlloc* pnha = new NoHeapAlloc;    // error
    return 0;
}
```

在一些情况下，需要对象在指定内存位置进行内存分配，并且不需要析构函数来完成一些对象级别的清理，这个时候就可以通过显示删除析构函数来限制自定义类型在栈上或者静态构造

```
#include <cstddef>
#include <new>

 void* p;

class NoStackAlloc
{
public:
    ~NoStackAlloc() = delete;
};

int main()
{
    // NoStackAlloc nas;    // error

    new(p) NoStackAlloc();
    return 0;
}
```

由于 placement new 构造的对象，编译器不会对其调用析构函数，因此可以构造析构函数被删除的类。