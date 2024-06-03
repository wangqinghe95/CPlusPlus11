# 对齐支持

## 数据对齐

了解数据对齐之前，先看一下以下代码和打印

```
#include <iostream>

using namespace std;

struct HowManyBytes
{
    char a;
    int b;
};

int main()
{
    // 类型所占的空间
    cout << "sizeof(char) : " << sizeof(char) << endl;
    cout << "sizeof(int) : " << sizeof(int) << endl;
    cout << "sizeof(HowManyBytes) : " << sizeof(HowManyBytes) << endl;

    // 成员元素相对于类的偏移量
    cout << "offset of char a : " << offsetof(HowManyBytes, a) << endl;
    cout << "offset of char n : " << offsetof(HowManyBytes, b) << endl;

    return 0;
}

/**
sizeof(char) : 1
sizeof(int) : 4
sizeof(HowManyBytes) : 8
offset of char a : 0
offset of char n : 4
*/
```

上述代码可以看出来，有 3 字节的空间没有被使用，这是因为 C/C++ 要求的数据对齐导致的。
当平台上，C/C++ 要求 int 类型对齐到 4 字节，即 int 类型数据必须放置到一个可以被 4 整除的低智商，而 char 类型要求对齐到 1 字节上，这就造成了 a 成员后有 3 字节的空间被空出。这个三字节被成为填充数据。

这种要求数据对其的属性被称为对齐方式。对齐数据的目的是为了提高数据读写的性能。比如频繁使用的数据如果与处理器的高速缓存器大小对齐，可以提高缓存性能。

C++11 中使用修饰符 aligns 来设定对齐方式

```
#include <iostream>

using namespace std;

struct ColorVector
{
    double r;
    double g;
    double b;
    double a;
};

struct alignas(32) AlignColorVector
{
    double r;
    double g;
    double b;
    double a;   
};

int main()
{
    cout << "alignof(ColorVector): " << alignof(ColorVector) << endl;               // 8
    cout << "alignof(AlignColorVector): " << alignof(AlignColorVector) << endl; // 32
    return 0;
}
```

## C++11 alignof 和 alignas

C++11 标准为了支持对齐，引入了操作符 alignof 和 对齐描述符 alignas

### alignof

操作符 alignof 表示一个定义完整的自定义或者内置类型或者变量，返回的值是一个 std::size_t 类型的整型常量。alignof 同理，以下代码为例：

```
#include <iostream>

using namespace std;

class InComplete;

struct Complete{};

int main()
{
    int a;
    long long b;
    auto &c = b;

    char d[1024];

    // 对内置类型和完整类型使用 alignof
    cout << alignof(int) << endl        // 4
        << alignof(Complete) << endl;   // 1

    // 对变量、引用或者数组使用 alignof
    cout << alignof(a) << endl          // 4
        << alignof(b) << endl          // 8
        << alignof(c) << endl          // 8
        << alignof(d) << endl;          // 1

    // cout << alignof(InComplete) << endl;    // error，类型不完整
    
    return 0;
}
```

### alignas

alignas 是对齐描述符，既可以接收常量表达式，又可以介绍类型作为参数
`alignas(double) char c` 和 `alignas(alignof(double)) char;` 都是合法的表达，并且两者相同

使用常量表达式作为 alignas 的参数时，其结果必须是以 2 的自然数幂次作为对齐值，其对齐值越大，对齐要求越高，反之称之为对其要求越低。由于 2 的幂次的关系，能够满足严格对齐要求的对齐方式也总能满足要求低的对齐值。

## std::align

C++11 还提供了 STL 库中的 std::align 函数来动态指定内存对齐方式调整数据块的位置。

`void* align(std::size_t alignment, std::size_t size, void* &prt, std::size_t& space);`

该函数在 ptr 指向的大小为 space 的内存中对齐方式的调整，将 ptr 开始的大小的数据调整为按 alignment 对齐。如下

```
#include <iostream>
#include <memory>
using namespace std;

struct ColorVector
{
    double r;
    double g;
    double b;
    double a;
};

int main()
{
    size_t const size = 100;
    ColorVector* const vec = new ColorVector[size];

    void* p = vec;
    size_t sz = size;

    void* aligned = align(alignof(double) * 4, size, p, sz);
    if(aligned != nullptr) {
        cout << alignof(p) << endl;     // 有问题
    }
    else {
        cout << "1" << endl;
    }

    return 0;
}
```

C++11 标准库还提供了 aligned_storage 和 aligned_union

```
/*
 * Len: aligned_storage 大小
 * Align: 对齐值
 */
template<std::size_t Len, std::size_t Align = /*default-aligment*/>
struct aligned_storage;
template<std::size_t Len, class... Types>
struct aligned_union;
```

例子如下：

```
#include <iostream>
#include <type_traits>
using namespace std;

struct IntAligned
{
    int a;
    char b;
};

typedef aligned_storage<sizeof(IntAligned),alignof(long double)>::type StrictedAligned;

int main()
{
    // 通过 aligned_storage 将对齐方式变得更严格了
    StrictedAligned sa;
    IntAligned* pia = new(&sa) IntAligned;
    cout << alignof(IntAligned) << endl;      // 4
    cout << alignof(StrictedAligned) << endl;      // 16
    cout << alignof(*pia) << endl;      // 4
    cout << alignof(sa) << endl;      // 16

    return 0;
}
```

C++11 对对齐的支持是全方位的，包括查看(alignof)，设定(alignas)，STL 库函数(std::align)，STL 库模板类型(aligned_storage/aligned_union)。