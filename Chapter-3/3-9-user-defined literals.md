# 用户自定义字面量

C/C++ 程序中，往往需要结构体或者类来创建新的类型，以满足实际的需要。
比如在进行科学计算时，可能会需要用到复数，包含实数或者虚数部分；对于颜色可能需要一个四元组，三原色加上Alpha。

C++11 允许自定义类型像内置类型一样向函数传递字面常量，即可以通过一个后缀标识的操作符，将该声明了该后缀标识的字面量转换成需要的类型。

```
#include<cstdlib>
#include<isotream>

typedef unsigned char uint8;
struct RGBA {
    uint8 r;
    uint8 g;
    uint8 b;
    uint8 a;
    RGBA(uint8 R, uint8 G, uint8 B, uint8 A = 0) : r(R), g(G), b(B), a(A){}
};

RGBA operator "" _C(const char* col, size_t n)
{
    const char* p = col;
    const char* end = col + n;
    const char* r, *g, *b, *a;

    r = g = b = a = nullptr;
    for(; p != end; ++p) {
        if(*p == 'r') r = p;
        else if (*p == 'g') g = p;
        else if (*p == 'b') b = p;
        else if (*p == 'a') a = p;
    }

    if((r == nullptr) || ( g == nullptr) || (b == nullptr)) {
        throw;
    }
    else if( a== nullptr) {
        return RGBA(atoi(r+1), atoi(g+1), atoi(b+1));
    }
    else {
        return RGBA(atoi(r+1), atoi(g+1), atoi(b+1), atoi(b+1));
    }
}

std::ostream & operator << (std::ostream& out, RGBA& col)
{
    return out << "r: " << (int)col.r << ", g: " <<(int)col.g
                << ", b: " << (int)col.b << ", a: " << (int)col.a << endl;
}

void blend(RGBA && col, RGBA&& col2)
{
    cout << "blend " << endl << col1 << col2 << endl;
}

int main()
{
    blend("r255,g240, b155"_C, "r15 g255 b10 a1"_C);
}
```

在这个代码中声明了一个字面量操作符函数 `RGBA operator "" _C(const char* col, size_t n)`，这个函数会解析以 _C 为结尾的字符串，并返回一个 RGBA 的临时变量。
有了这样一个用户字面量的定义后， main 函数中不需要再通过声明 “变量-传值计算的方式来传递实际意义”上的常量。通过声明一个字符串以及一个 _C 后缀，operator ""_C 函数会产生临时变量。
blend 函数能够通过右值引用获得这些临时变量并进行计算。

除去字符串外，后缀声明还能够作用于数值，比如，用户可以使用 60W，120W，标识方法来标识功率，用 50 kg 标识质量，1200 N 表示力。

```
struct Watt {unsigned int v;};

// 使用 _w 后缀表示瓦特
Watt operator "" _w(unsigned long long v) {
    return {(unsigned int)v};
}
int main()
{
    Watt capaticy = 1024_w;
}
```

C++11 中，标准要求声明字面量操作符有一定的规则，该规则和字面量的“类型”密切相关。
1. 如果字面量为整型数，那么字面量操作符只可接受 `unsigned long long` 或 `const char*` 为参数，当 `unsigned long long` 无法容纳该字面量的时候，编译器会自动将字面量转化为以 '\0' 为结束符的字符串，并调用以 `const char*` 为参数的版本进行处理
2. 当字符量为浮点型数时，那么字面量操作符只可接受 `long double` 或 `const char*` 为参数，`const char*` 版本调用调用规则同整型一样
3. 如果字面量为字符串，则那么字面量操作符只可接受 `const char*`，size_t 为参数
4. 如果字面量为字符，则字面量操作符只可接受一个 char 参数。

有一下几点需要注意：
1. 在字面量操作符函数的声明中，operator"" 和用户自定义后缀之间必须有空格。
2. 后缀建议以下划线开始。不宜使用非下划线后缀的用户自定义字符串常量，否则会被编译器警告。