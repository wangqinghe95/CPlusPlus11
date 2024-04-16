# noexcept 修饰符和操作符

异常用于处理逻辑上可能发生的错误。

noexcept 表示修饰的的函数不会抛出异常。如果 noexcept 修饰的函数抛出了异常，那么编译器会直接选择调用 std::terminate() 终止程序的运行

异常机制会带来一些额外开销，比如函数抛出异常，会导致函数栈被依次展开，并以次调用栈帧中已构造的自动变量的析构函数

## 修饰符
noexcept 有两种形式修饰函数:

```
void expec_func() noexcept; // 修饰函数
void expec_func() noexcept(常量表达式)  // 接受一个常量表达式作为参数
```

C++11 中使用 noexcept 可以有效地阻止异常的传播与扩散。
```
#include <iostream>

using namespace std;

void Throw() {
    cout << "Throw 1" << endl;
    throw 1;
}

void NoBlockThrow() {
    cout << "NoBlockThrow" << endl;
    Throw();
}

void BlockThrow() noexcept{
    cout << "BlockThrow" << endl;

    Throw();
}

int main()
{
    try{
        Throw();
    }
    catch(...){
        cout << "Found Throw." << endl;
    }

    cout << "--- 1 ---" << endl;
    try{

        NoBlockThrow();

    }
    catch(...){
        cout << "Throw is not blocked." << endl;
    }
    
    cout << "--- 2 ---" << endl;

    try{
        cout << "--- 3 ---" << endl;
        BlockThrow();
        cout << "--- 4 ---" << endl;
    
    }
    catch(...){
        cout << "Throw is  blocked." << endl;
    }
    
    return 0;
}

// 运行结果
/*
Throw 1
Found Throw.
--- 1 ---
NoBlockThrow
Throw 1
Throw is not blocked.
--- 2 ---
--- 3 ---
BlockThrow
Throw 1
terminate called after throwing an instance of 'int'
Aborted (core dumped)
*/
```

从运行结果中可以看到，在 BlockThrow() 函数中，调用了中断函数，直接中断了该函数的运行。

## 操作符

noexcept 作为操作符可以用作模板

```
template <class T>
void fun() noexcept(noexcept(T())){}
```

该函数模板会根据 T() 表达式是否会抛出异常决定是否是 noexcept 版本。这也是 C++11 为了更好的支持泛型编程引入的特性。

## noexcept 优缺点

noexcept 修饰的函数会通过 std::terminate 调用，结束现成的执行。
这种方式会带来一些问题，比如，无法保证对象的析构函数能够正确被调用；无法保证栈的自动释放；
但是很多时候暴力的终止整个程序确实是有必要的，事实上，noexcept 被广泛地应用于 C++11 标准库中，一是用于提高标准库的性能，二是满足一些阻止异常扩散的需求

如，在 C++98 中，一些使用 throw() 的使用场景
```
// 使用 throw() 来声明不抛出异常
template<class T>
calss A
{
public:
    static constexpr T min() throw() { return T(); }
    static constexpr T max() throw() { return T(); }
    static constexpr T lowest() throw() { return T(); }
}

// 使用 throw() 抛出一些 std::bad_alloc
void* operator new(std::size_t) throw(std::bad_alloc);
void* operator new[](std::size_t) throw(std::bad_alloc);
```

而在 C++11 中，都已经使用 noexcept 代替 throw()

```
template<class T>
calss A
{
public:
    static constexpr T min() noexcept() { return T(); }
    static constexpr T max() noexcept() { return T(); }
    static constexpr T lowest() noexcept() { return T(); }
}

void* operator new(std::size_t) noexcept(std::bad_alloc);
void* operator new[](std::size_t) noexcept(std::bad_alloc);
```

noexcept 最大的作用是保证应用程序的安全。
比如类的析构函数不应该抛出异常，那么对于常被析构函数调用的 delete 函数来说，C++11 会将 delete 函数设置为 noexcept，可以提供应用程序的安全

```
void* operator delete(std::size_t) noexcept(std::bad_alloc);
void* operator delete[](std::size_t) noexcept(std::bad_alloc);
```

通常的原因，C++ 标准也会让类的析构函数默认是 noexcept(true)，当然程序员可以显式地指定析构函数或者类的基类或者成员是 noexcept(false)，这样析构函数就不会保持默认值了。

```
#include <iostream>
using namespace std;

struct A
{
    ~A(){ throw 1;}
};

struct B
{
    ~B() noexcept(false)
    { throw 1;}
};

struct C
{
    B b;
};

int funcA() {
    A a;
}

int funcB() {
    B b;
}

int funcC() {
    C c;
}

int main()
{
    try {
        funcB();
    }
    catch(...){
        cout << "caught funcB" << endl;
    }

    try {
        funcC();
    }
    catch(...){
        cout << "caught funcC" << endl;
    }

    try {
        funcA();
    }
    catch(...){
        cout << "caught funcA" << endl;
    }

    return 0;
}

/**
caught funcB
caught funcC
terminate called after throwing an instance of 'int'
Aborted (core dumped)
*/
// B C 都抛出了异常，而 A 默认不抛出，直接调用 terminate 结束进程，阻止异常扩散
```