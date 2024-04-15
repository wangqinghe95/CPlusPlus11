# 静态断言


## assert ：运行与预处理时

1. 介绍
断言是将一个返回值总是需要为真的判别式放在语句中，用于排除在设计上的逻辑上不应该产生的情况。

2. 示例
比如一个函数总是需要输入在一定范围内的参数，那么就可以使用断言，迫使在该参数发生异常的时候程序退出，从而避免程序陷入逻辑的混乱。

C++ 标准在头文件 <cassert> 或 <assert.h> 头文件提供了断言的宏，用于在运行时进行断言。
如下：

```
#include <cassert>
using namespace std;

char* ArrayAlloc(int n)
{
    assert(n > 0);
    return new char[n];
}

int main()
{
    char *a = ArrayAlloc(0);
    return 0;
}
```

运行结果：
```
a.out: Assert.cpp:7: char* ArrayAlloc(int): Assertion `n > 0' failed.
Aborted (core dumped)
```

3. 禁用 assert 宏
但是对于程序退出这种情况，在正式软件发布时是非常敏感的，而且部分错误未必会导致程序全部功能失效，所以就有比较禁用 assert 宏了。

C++ 可以使用 NDEBUG 宏发布程序尽量的避免程序退出的状况。
```
#ifdef NDEBUG
# define assert(expr) (static_cast<int> (0))
#else
#endif
```

4. 禁止头文件编译
通过预编译命令 #if、#else 的配合也可以让程序员在预处理阶段进行预言。
如下方代码
```
#ifndef _COMPLEX_H
#error "Never use <bits/cmatchcalls.h> directly; include <complex.h> instead"
#endif
```

当有程序包含了 bits/cmatchcalls.h 头文件时，#error 指令就会将后面的语句输出，提醒不要使用这个头文件，而应该包含头文件 <complex.h>。
这样就可以通过预处理时的断言，就能够避免一些头文件引用问题。


## assert 和 static_assert

1. assert 断言宏只能在程序运行时起作用，#error 在编译器预处理时起作用

2. static_assert 是在编译时期的断言。

#### 例1

```
/**
 * 按位存储属性：
 *  首先是一个枚举类型，用于列举编译器对各种特性的支持
 *  然后是一个结构体保存当前编译器的特性
 *  通过两者的比较确定编译器是否对某种特性给予支持
*/

#include <cassert>

// 每一种特性都是 “支持” 和 “不支持” 两种状态，所以我们为了节省空间
// 使用一个枚举值占据一个特定的 bit 位
enum FeatureSupports {
    C99         =   0x0001,
    ExtInt      =   0x0002,
    SAssert     =   0x0004,
    NoExcept    =   0x0008,
    SMAX        =   0x0010,
};

// 使用或匀速那检查 spp 的某位来判断编译器对特性是否支持
struct Compoler {
    const char* name;
    int spp;
};

int main()
{
    /*
    * 检查枚举值是否完备
    */
    assert((SMAX - 1) == (C99 | ExtInt | SAssert | NoExcept));

    Compoler a = {"abc", (C99 | SAssert)};
    ...
    if(a.spp & C99) {
        ...
    }

    return 0;
}
```

本例中使用断言 assert ，但是 assert 是一个运行时断言，这意味着不运行是无法得知是否有枚举重位。而单次运行代码可能并不会调用到 assert 相关的代码路径。

#### 例2
在类模板中
```
#include <cstdio>
#include <cassert>

template <typename T, typename U>
int CopyBit(T&a T&b) {
    // 需要确保 a, b 两个类型的长度一致，才能保证赋值操作不会发生越界问题
    assert(sizeof(b) == sizeof(a));
    memcpy(&a, &b, sizeof(b));
}

int main() {
    int a = 0x2468;
    double b;
    CopyBit(a, b);
    return 0;
}
```

该例中存在的问题同例1一致，都是只能在运行时才会检测出错误。

### static_assert
C++11 引入该关键字来解决该问题。

static_assert 接受两个参数，一个是断言表达式，该表达式通常需要返回一个 bool 值，一个是警告信息，它通常是一段字符串。

static_assert 是编译时期的断言，推荐写在函数体外，利于阅读，断言需要是编译时期能够计算的表达式，即常量表达式。