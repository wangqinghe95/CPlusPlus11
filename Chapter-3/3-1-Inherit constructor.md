# 继承构造函数

在 C++11 中，派生类使用 using 关键字来继承基类的构造函数
解决的问题是基类拥有数量众多的不同版本的构造函数时，而派生类只有少量的成员函数和成员变量时，此时派生类的构造函数就几乎等同于基类的构造函数

```
#include<iostream>
using namespace std;
struct A
{
    A(int i){
        cout << "A::A(int i) " << i << endl;
    }
    A(double d, int i) {}
    A(float f, int i, const char* c){}

    void f(int i) {
        cout << "A::f " << i << endl;
    }
};

struct B : A {
    using A::A;         // 继承构造函数，继承父类 A 的构造函数
    void f(int i) {
        cout << "B::f " << i << endl;
    }

    void printD(){
        cout << "B::d " << d << endl;
    }

    int d{0};   // 初始化表达式设置默认值
};

int main()
{
    B b(356);   // call  A(int i);
    b.printD();

    return 0;
}
```

继承构造函数只会初始化基类的中成员变量，对于派生类的成员变量初始化，可以通过类成员初始化表达式为派生类成员变量设定一个默认值。

如果以上措施还是不能满足需求的话，就需要重新设计一个派生类的构造函数了。

## 基类构造函数的默认值

当基类的构造函数有默认值时，派生类的继承构造函数是不会被继承的。

## 继承构造函数

如果一个派生类继承了多个基类，而这几个基类中的部分构造函数可能导致派生类中继承构造函数的函数名，参数都相同，那么继承的派生类中冲突的继承构造函数将导致不合法的派生类代码

```
struct A { A(int) {}};
struct B { B(int) {}};

struct C : A, B {
    using A::A;
    using B::B;
};
```

上述代码中，A 和 B 的构造函数会导致 C 中重复定义相同类型的继承构造函数。
解决方案是：显示定义继承类的冲突的构造函数，阻止隐式生成相应的继承构造函数来解决冲突。

struct C : A, B {
    using A::A;
    using B::B;
    C(int) {};
};

## 基类构造函数权限
如果基类的构造函数被声明为私有构造函数，或者派生类从基类虚继承的，那么就不能再派生类中声明继承构造函数。

## 默认构造函数有关
如果使用了继承构造函数，那么编译器就不会在给派生类生成默认构造函数了

```
struct D {
    D(int i) {}
};

struct E : D {
    using D::D;
};

// E e; // 编译失败
```

## 构造函数冲突

如果子类和父类的构造函数有一个相同的参数列表的构造函数，则子类的构造函数会覆盖父类的那个构造函数

## 基类构造函数未被调用

当声明了继承构造函数之后，但是派生类不调用，且基类有显示构造函数未定义默认构造函数，那么就会导致编译报错