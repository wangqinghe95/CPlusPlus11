# 显示转换操作符

C++ 有个特性叫隐式类型转化，但是它的自动性，出现一些问题。

```
#include <iostream>

using namespace std;

struct Rational_1 {
public:
    int num;
    int den;
public:
    Rational_1(int n = 0, int d = 1) : num(n), den(d) {
        cout << __func__ << "(" << num << "/" << den << ")" << endl;
    }

};

struct Rational_2 {
public:
    int num;
    int den;
public:
    explicit Rational_2(int n = 0, int d = 1) : num(n), den(d) {
        cout << __func__ << "(" << num << "/" << den << ")" << endl;
    }
};

void Display_1(Rational_1 ra) {
    cout << "Numberator: " << ra.num << " Denomoator: " << ra.den << endl;
}

void Display_2(Rational_2 ra) {
    cout << "Numberator: " << ra.num << " Denomoator: " << ra.den << endl;
}

int main()
{
    Rational_1 r1_1 = 11;
    Rational_1 r1_2(12);

    // Rational_2 r2_1 = 21;    // 无法通过编译
    Rational_2 r2_2(22);

    Display_1(1);
    // Display_2(2);              // 无法通过编译
    Display_2(Rational_2(2));

    return 0;
}
```

上述代码中有两个类，唯一的区别就是构造函数的不同，后者构造函数被 explicit 修饰，这意味着该构造函数不能被隐式调用。
这也是前者可以直接被一个字面量 11 就能构造出，而后面却不能被字面量 21 构造原因。因为它被 explicit 修饰，禁止隐式构造，因此会导致构造失败。

Display_2(2) 失败的原因也是基于此。

但是在自定义类型转换符上，允许出现一个逆向的过程，从自定义类型转化成一个逆向的过程。

```
#include <iostream>
using namespace std;

template <typename T>
class Ptr
{
public:
    Ptr(T* p) : _p(p) {}
    operator bool() const {
        if(_p != 0) {
            return true;
        }
        else {
            return false;
        }
    }
private:
    T* _p;
};

int main()
{
    int a;
    Ptr<int> p(&a);

    if(p) {
        cout << "valid pointer" << endl;
    }
    else {
        cout << "invalid pointer" << endl;
    }

    Ptr<double> pd(0);
    cout << p+pd << endl;   // no meaningful

    return 0;
}
```

自定义了一个指针模板类型 Ptr。重载了 bool() 类型函数，这样就可以直接判断 if(p) 是否有效了。

C++11 的 explicit 将使用范围扩展到了自定义的类型转化操作符上，以支持 “显示类型转化”。 explicit 关键字作用于类型转换操作符上，意味着只有在直接构造目标类型或者显示类型转换的时候可以使用该类型。

如下所示：

```
class ConvertTo{};

class Convertable
{
public:
    explicit operator ConvertTo() const {
        return ConvertTo();
    }
};

void Func(ConvertTo ct) {}

void test(){
    Convertable c;
    ConvertTo ct(c);        // 直接初始化，通过
    // ConvertTo ct2 = c;      // 拷贝构造初始化编译失败
    ConvertTo ct3 = static_cast<ConvertTo>(c);     // 强制转换，通过

    // Func(c);    // 拷贝构造初始化，编译失败
}
```

显示类型转换并没有完全禁止从源类型到目标类型的转换，不过由于此时拷贝构造函数和非显示类型转换不被允许，那么我们通常就不通过赋值表达式或者函数参数的方式产生一个目标类型。
通常通过赋值表达式和和函数参选进行的转换并非是有意进行的，所以使用显示类型转换就会出现这种漏洞。
因此需要一个显示转换符就非常有意义。