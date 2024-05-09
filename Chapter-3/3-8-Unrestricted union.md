# 非受限联合体

C/C++ 中联合体是一种构造类型的数据结构，在一个联合体内，我们可以定义多种不同的数据类型，它们共享一块相同的内存空间。在一些需要复用内存的情况下，可以达到节省空间的目的。

C++11 标准中，规定任何非引用类型都可以成为联合体的数据成员，编译时需要使用指定编译参数 -std=c++11
C++11 中允许在非匿名联合体中有静态成员，但是不允许有静态成员变量的存在。
C++11 标准默认删除了一些非受限联合体的默认函数，比如非受限联合体有一个非 POD 的成员，而该非 POD 成员类型拥有非平凡的构造函数，那么非受限联合体成员的默认构造函数将被编译器删除。其他特殊成员函数，例如默认拷贝构造函数，拷贝赋值操作符以及析构函数也是如此。

```
#include <iostream>

using namespace std;

union T {
    string s;   // string 有非凡的构造函数
    int n;
};

int main()
{
    T t;        // 构造失败，因为 T 的构造函数被删除了
    return 0;
}
```

解决方案是，由程序员自己为非首先联合体定义构造函数

```
union T {
    string s;   // string 有非凡的构造函数
    int n;
public:
    T() {
        new (&s) string;    // 采用 placement new 将 s 构造在其地址 &s 上
    }
    ~T() {
        s.~string();    // 析构的函数 union 是一个 string，否则就会出问题
    }
};
```

匿名非受限联合体可以运用于类的声明中，这样的类也被称为“枚举式的类”，如下：

```
#include <iostream>
#include <cstring>

using namespace std;

struct Student {
    bool gender;
    int age;
    Student(bool g, int a) : gender(g), age(a){}
};

class Singer {
public:
    enum Type {STUDENT, NATIVE, FOREIGNER};
    Singer(bool g, int a) : s(g,a) {
        t = STUDENT;
    }
    Singer(int i) : id(i) {
        t = NATIVE;
    }
    Singer(const char* n, int s) {
        int size = (s > 9) ? 9 : s;
        memcpy(name, n, size);
        name[s] = '\0';
        t = FOREIGNER;
    }
private:
    Type t;
    union {
        Student s;
        int id;
        char name[10];
    };
};

int main()
{
    Singer(true, 13);
    Singer(310217);
    Singer("J Miachael", 9);
    return 0;
}
```

借用匿名非受限联合体成为类 Signer 的变长成员，可以给类的编写带来更大的灵活性。