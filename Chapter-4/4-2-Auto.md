# auto 类型推导

## 1. auto 类型推导

C++11 标准提供了一个 auto 的关键字，用来实现类型推导

```
#include <iostream>
using namespace std;

int main()
{
    auto name = "world.\n";
    cout << "hello " << name << endl;
    return 0;
}
```

auto 关键字会在编译期间对变量的类型进行自动推导。编译器根据变量初始化表达式的类型，推导 name 为 char* 类型

+ C++11 标准中将 auto 作为一个新的类型指示符来指示编译器
+ auto 声明的变量的类型必须由编译器在编译时期推导而得
+ auto 声明的变量必须被初始化，以使编译器能够从初始化表达式中推导出其类型

## 2. auto 的优势

1. 简化代码
auto 推到的优势就是拥有初始化表达式的复杂类型变量声明时简化代码。
比如想要使用一个 vector 类型的迭代器

```
std::vector<std::string> vs;
std::vector<std::string>::iterator it = vs.begin();
// 使用 auto 简化
auto it = vs.begin();
```

2. 免除类型声明的麻烦

在 C/C++ 中存在很多隐式或者用户自定义的类型转换规则。
他们的返回值不是很好记忆，这时候使用 auto 就非常方便

```
#include <iostream>
using namespace std;

class PI
{
public:
    double operator*(float v) {
        return (double)val*v;
    }
    const float val = 3.1415927f;
};

int main()
{
    float radius = 1.7e10;
    PI pi;
    auto circumference = 2 * (pi * radius);

    return 0;
}
```

一个求周长的类和操作符重载函数。
当 PI 类型的变量和一个 float 数据相乘时，其返回值时 double。
而 PI 的定义可能是在其他地方，main 函数的程序员可能不知道 PI 的为了避免数据上溢，或者精度降低而返回了 double 类型的浮点数。


当然 auto 并不能解决所有的精度问题
```
unsigned int a = 4294967295;    // unsigned int 最大值
unsigned int b = 1;

// c 仍然被推导为 unsigned int 类型，不会自动推展类型解决 a+b 溢出问题
auto c = a + b;
```

3. 自适应性支持泛型的编程

当 atuo 应用于模板的定义时，会体现出它的自适应

```
#include <iostream>
using namespace std;

class PI
{
public:
    double operator*(float v) {
        return (double)val*v;
    }
    const float val = 3.1415927f;
};

template <typename T1, typename T2>
double Sum(T1 &t1, T2& t2) 
{
    auto s = t1+t2;
    return s;
}

int main()
{
    int a = 3;
    long b = 5;
    float c = 1.0f, d = 2.3f;

    // Sum 中的 s 会被自动推导为 long
    auto e = Sum<int,long>(a,b);

    // s 会被自动推导成 float
    auto f = Sum<float,float>(c,d);

    cout << e << " " << f << endl;

    return 0;
}
```

在 define 中的应用

```
#define MAX1(a, b) ((a) > (b)) ? (a) : (b)
#define MAX2(a, b) ({   \
        auto _a = (a);  \
        auto _b = (b);  \
        (_a > _b) ? _a : _b;})

// 无论是 a 还是 b，都会做两次运算
int m1 = MAX1(1*2*3*4, 5+6+7+8);

// 先计算出来 a 和 b 的值，然后再做比较
int m2 = MAX2(1*2*3*4, 5+6+7+8);
```

## 3. auto 的使用细则

1. 基础使用
```
int x;
int *y = &x;
double foo();
int& bar();

// auto* 等同于 auto
// 以下都是指针类型
auto* p1 = &x;
auto p2 = y;
aotu *p3 = y;

// 如果想要声明引用类型，需要 auto&
auto& r1 = x;
auto & h = bar();

// 以下是两种错误示范
auto *err1 = &foo();   // 编译失败，指针不能指向一个临时对象
auto & err2 = foo();    // 编译失败，non const 不能与一个临时变量绑定

anto g = bar();     // int

// 与 cv 限定符一起作用
float *func();
const auto a = foo();       // a: const double
const auto& b = foo();      // b: const double&
const auto * c = func();     // c: volatile float*

auto d = a;                 //  d : double
auto &e = a;                //  e: const double&
auto f = c;                 //  f: float*
volatile auto& g = c;       //  g: volatile float* &
```

2. auto 声明多个变量
auto 如果需要声明多个变量，那么这几个变量需要有相同的类型，否则会报错
```
auto x = 1, y = 2;              // 推导出两个 int
const auto* m = &x, n = 1;      // m 时 const int 类型的指针， n 是一个 int 类型的变量
auto i = 1, j = 3.14f;          // error
auto o = 1, &p = o, *q = &p;
```

3. 初始化列表

```
#include <initializer_list>
auto x = 1;
auto x1{1};
auto y{1};
auto z = new autp(1);
```

4. 不能生效的推导

```
// 函数参数不能推导
void fun(auto x = 1){}

struct str {
    auto var = 10;  // 非静态成员变量不能推导
}

char x[3];
auto y = x;
auto z[3] = x;  // 数组无法推导
vector<auto> v = {1};   // 模板参数无法推导
```