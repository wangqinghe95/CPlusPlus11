# 右值引用：移动语义和完美转发

## 深拷贝和浅拷贝

深拷贝和浅拷贝是两种不同的对象复制方式，主要是复制类中指针或者动态分配的内存时会发生这种不同形式的拷贝。
浅拷贝是在使用类的简单复制拷贝，会直接将类中的值复制到另一个类中；
深拷贝是在堆中重新申请内存空间对指针进行内部数据的完整拷贝；

浅拷贝存在的问题是，当存在指针时，对象拷贝的是指针指向的地址，当一个类执行完自己的析构函数后，该指针指向的地址已经被释放了，导致所有指向该地址的指针都会变成悬空指针的情况。
而深拷贝会因为自己重新申请内存空间，确保了对象之间的资源数据独立，从而解决了这个问题。

## 移动语义

### 拷贝构造函数存在的弊端

拷贝构造函数的深拷贝固然能解决指针的资源独立问题，但是也会带来一些其他问题，比如拷贝次数过多导致影响程序性能，如下所示：

```
#include <iostream>
using namespace std;

class HasPtrMem
{
public:

    HasPtrMem() : d(new int(0)){
        cout << "Constuct " << ++n_cstr << endl;
    }
    HasPtrMem(const HasPtrMem& m) :d(new int(*m.d)) {
        cout << "Copy construct " << ++n_cptr << endl;
    }
    ~HasPtrMem(){
        cout << "Destruct " << ++n_dstr << endl;
        delete d;
    }
public:
    int a;
    int *d;
    static int n_cstr;
    static int n_dstr;
    static int n_cptr;
};

int HasPtrMem::n_cstr = 0;
int HasPtrMem::n_dstr = 0;
int HasPtrMem::n_cptr = 0;

HasPtrMem getTemp()
{
    return HasPtrMem();
}

int main()
{
    HasPtrMem a = getTemp();
    return 0;
}

// compiler
g++ move.cpp -o move -fno-elide-constructors
// 运行结果
Constuct 1
Copy construct 1
Destruct 1
Copy construct 2
Destruct 2
Destruct 3
```

在上述代码中因为函数返回值的问题，导致拷贝函数调用了两次，
+ 一次是在 `getTemp():return` 时，会为返回值调用一次拷贝构造函数创建一个临时值，用于该函数的返回。
+ 二是由临时变量构造出变量 a 时调用。

如果类中包含的数据过多，并且返回次数也多的话，就会导致程序效率降低，影响性能。因为返回值创造的临时变量，会在使用后被析构，而构造 a 时又要去申请内存复制数据。这两步的操作是浪费的。

### 移动构造函数

C++11 提供了一个新的方法，将函数返回值的临时变量的资源直接给 a 使用，这样就省去了临时变量的析构和类的拷贝构造函数的过程的浪费。这个过程就被称为 “移动语义”，也就是将一个临时，即将销毁的内存资源移动给另一个数据使用。
有一个构造函数完成这个步骤，她也叫做 “移动构造函数”

如下述代码

```
#include <iostream>
using namespace std;

class HasPtrMem
{
public:

    HasPtrMem() : d(new int(0)){
        cout << "Constuct " << ++n_cstr << endl;
    }
    HasPtrMem(const HasPtrMem& m) :d(new int(*m.d)) {
        cout << "Copy construct " << ++n_cptr << endl;
    }
    HasPtrMem(HasPtrMem && h) : d(h.d) {
        h.d = nullptr;
        cout << "Move construct " << ++m_mvtr << endl;
    }
    ~HasPtrMem(){
        cout << "Destruct " << ++n_dstr << endl;
        delete d;
    }
public:
    int a;
    int *d;
    static int n_cstr;
    static int n_dstr;
    static int n_cptr;
    static int m_mvtr;
};

int HasPtrMem::n_cstr = 0;
int HasPtrMem::n_dstr = 0;
int HasPtrMem::n_cptr = 0;
int HasPtrMem::m_mvtr = 0;

HasPtrMem getTemp()
{
    HasPtrMem h;
    cout << "Resource from " << __func__ << ": " << hex << h.d << endl;
    return h;
}

int main()
{
    HasPtrMem a = getTemp();
    cout << "Resource from " << __func__ << ": " << hex << a.d << endl;

    return 0;
}

// 运行结果
Constuct 1
Resource from getTemp: 0x562dbed8ae70
Move construct 1
Destruct 1
Move construct 2
Destruct 2
Resource from main: 0x562dbed8ae70
Destruct 3
```

从结果可以看出，h.d 和 a.d 指向同一个内存地址，该堆内存在函数返回时没有被释放，后续给了变量 a。
单个看对程序的性能影响不大，但是如果涉及到 MB 为单位的堆空间，那么影响就很大了。

### 移动语义的好处

通过修改函数接口，使用指针或者引用才能达到上述的节省程序性能和效率的目的。
但是有时候绕不开这个临时变量的使用。

如下：
`Calculate(GetTemp(), SomeOther(Maybe(), Useful(Values,2)))`

上述的代码，如果使用引用，而不是用临时变量的话，就需要写成以下形式：

```
string* a;
vector<Object> b;
...
Usefule(Values, 2, a);
SomeOther(Maybe(), a, );
Calculate(GetTemp(), b);
```

这个就是临时变量的实际应用，不需要提前声明定义。

## 左值和右值

移动构造函数什么时候会被触发，这是一个需要解决的关键问题。
C++ 是如何判断出现了临时对象？如何将其用于移动构造函数？等等
这些问题的解答需要先了解值得分类。

C++11 中，所有值必属于左值或者右值的一种，其中
+ 左值：就是有可以取地址，有名字的值；
+ 右值：不能取地址，没有名字的值；右值被分为纯右值，和将亡值。纯右值有临时对象，字面量，以及运算表达式产生的临时结果，将亡值指的是即将被移动或者声明周期结束的对象

详细解释就是，左值是带有一个名字的内存空间，也就是说内存中有块地址，它在程序代码中有个名字可以访问到这块地址。一个值是左值有三个条件，内存中有保存的地址，该地址能够被访问，且有个名字能够访问到该地址。
一个值不是左值就是右值，所以右值就是左值三个条件中缺一个的值，比如，临时对象和算表达式产生的临时结果都是没有变量名的值，由编译器自动管理该临时变量的产生，使用和销毁；将亡值是那些即将被移动的对象，或者是生命周期即将结束的对象，比如 std::move(),std::forward() 函数的参数，函数返回值。

### 将亡值举例
将亡值这个概念可能优点不好理解，再举几个例子

```
// std::move() 的返回值
int a = 10;
int b = std::move(a);   // std::move(a) 的返回值就是一个将亡值

// 右值引用的的函数返回值
/*
需要注意的是，不能将函数内部的存储在栈上内部变量当作一个右值引用返回
*/
int && getObj(int& num) {
    return static_cast<int&&> num;
}

// std::forward() 的返回值
```

### 右值引用
右值引用就是对一个右值进行引用的类型，主要是为了移动语义。

右值引用绑定一个函数返回值为右值的函数时，可以减少一个对象析构和对象构造的过程

右值引用不能绑定一个左值上，所以在一个函数返回类型是右值的情况下，返回的数据需要是一个临时变量，至少不能是一个函数局部变量。

### 常量左值引用

常左值引用可以绑定一个右值，实际上他可以接受非常量左值，常量左值，右值。但是常左值引用的对象只有只读属性。

如果我们不声明移动构造函数，可以声明一个常量左值的构造函数，即将调用以常量左值引用为参数的拷贝构造函数。这是一种比较安全的涉及，移动不成，至少还可以执行拷贝。
所以通常情况下，一个类除了移动构造函数以外，还会特意生声明一个以常量左值为参数的拷贝构造函数，以保证在移动构造不成时，可以使用拷贝构造。

### C++11 引用类型及其可以引用的的值类型

![C++引用类型及其可以引用的值的类型](../Resources/C++11引用类型%20及其可以引用的值的类型.png)

在 C++11 的标准库 <type_traits> 的头文件中，有三个模板来帮助我们判断是否是引用类型，以及是左值引用还是右值引用。它们分别是 
```
is_rvalue_reference
is_lvalue_reference
is_reference
```

如，想要判断模板类成员 value 是否是一个右值引用，就可以如下所示：
`cout << is_rvalue_referenc<string &&>::value`




### std::move，强制转换为右值

C++11 中，标准库 <utility> 中提供了一个 std::move 的函数，它的作用就是将一个左值强制转换成一个右值引用，继而我们可以通过右值引用使用该值，用于移动语义。
实现的角度上来说， std::move 等同一个类型转换
`static_cast<T&&>(lvalue)`

需要额外注意的是，被转换的左值其声明周期并没有随左右值的转换而改变
这句话的意思是，如果被转换的左值在其声明周期中还存在，还能被程序使用，但是因为其内存数据已经不再安全，使用会导致不会预知的事情发生。如下所示：

```
#include <iostream>

using namespace std;

class Moveable
{
public:
    int *m_p;
public:
    Moveable() : m_p(new int(3)) {}
    ~Moveable(){
        delete m_p;
    }

    Moveable(const Moveable& m) : m_p(new int(*m.m_p)){}

    Moveable(Moveable&& m) : m_p(m.m_p) {
        m.m_p = nullptr;
    }
};

int main()
{
    Moveable a;
    Moveable c(move(a));

    cout << *a.m_p << endl;
    return 0;
}

// Resuluts:
Segmentation fault (core dumped)
```

所以 std::move 使用的前提是使用者清楚地知道需要要转换的对象的性质，想要将一个左值转换，需要确定该左值即将被会销毁回收，而且该左值涉及到动态内存分布。

下面介绍一个正确使用的样例：
```
#include <iostream>

using namespace std;

class HugeMem
{
public:
    int *c;
    int sz;
public:
    HugeMem(int size) : sz(size > 0 ? size : 1) {
        c = new int[sz];
    }
    ~HugeMem() {
        delete[] c;
    }
    HugeMem(HugeMem&& hm) : sz(hm.sz), c(hm.c) {
        hm.c = nullptr;
    }
};

class Moveable
{
public:
    int *i;
    HugeMem h;
public:
    Moveable() : i(new int(3)), h(1024) {}
    ~Moveable(){
        delete i;
    }

    // move cpoy constructor
    Moveable(Moveable&& m) : i(m.i), h(move(m.h)){
        m.i = nullptr;
    }
};

Moveable GetTemp()
{
    Moveable temp = Moveable();
    cout << hex << "Huge Mem from " << __func__ << " @" << temp.h.c << endl;
    return temp;
}

int main()
{
    Moveable a(GetTemp());
    cout << hex << "Huge Mem from " << __func__ << " @" << a.h.c << endl;
    return 0;
}
```

在 Moveable 类的移动构造函数，支持将一个动态分配的指针成员权限转移这个操作。
因为是在栈上的数据，就算跨了函数域依旧能正常使用。

在编写移动构造函数时，为了保证移动语义的传递，需要记住使用 std::move 转换拥有形如堆内存、文件句柄等资源的成员为右值。
这样如果成员支持移动构造函数，就能够实现其移动语义，而即使成员没有移动构造函数，那么接受常量左值的构造函数版本也能使用拷贝构造，不会引起大的问题。

### 移动语义存在的一些问题

1. const 导致的移动语义无法实现

移动语义一定会修改临时变量的值的，但是如果在函数中加上了 const， 就会出现一些问题，比如：

```
// 移动构造函数的参数设置成了 const
Moveable(const Moveable&&);
// 或者函数返回值被设置成了 const
const Moveable ReturnVal();
```

这两个情况都会导致临时变量常量化，成为一个常量右值，那么临时变量的引用也就无法修改，从而导致移动语义无法实现。

2. 默认的移动/拷贝构造函数

C++11 移动/拷贝构造函数有三个版本
```
T Object(T&)        //  这个是哪种构造函数，隐式的吗？
T Object(const T&); //  拷贝构造函数
T Object(T &&)      //  移动构造函数
```

默认情况下，编译器会为一个类隐式地生成一个移动构造函数，隐式指的是如果没有使用到移动构造函数，就不会生成。
如果一个类已经声明了一个拷贝构造函数，拷贝复制函数，移动复制函数，析构函数中的一个或者多个，编译器就不会隐式生成一个移动构造函数。

默认的移动构造函数等同于默认的拷贝构造函数，只能做一些按位拷贝的工作。
如果程序需要用到移动构造函数，必须为一个类自定义一个移动构造函数。
当然对于不包含任何资源的类来说，实现移动语义与否都无关紧要，因为对于这样的类型而言，移动就是拷贝，拷贝也就是移动。

在 C++11 中，拷贝构造/赋值函数，移动构造/赋值函数 必须成对提供，这样才能保证类同时具有拷贝和移动语义。

当然只有一种语义的函数在代码中很常见，比如只有拷贝语义的类。但是只有移动语义的类有额外的含义，就是说，该类所拥有的资源只能被移动，而不能被拷贝，换句话说就是资源是独占的。
例子就是 C++11 提供的智能指针 unique_ptr

3. 识别一个类型是否能移动的模板

在标准库的头文件中 <type_traits> 中，可以通过一些辅助的模板来判断一个类型是否能移动，如
```
is_move_constructible
is_trivially_move_constructible
is_nothrow_move_constrictible
```

使用方法如下：
`cout << is_move_constructible<CheckedType>::value`
这样就可以打印出 CheckedType 是否可以移动

4. 高性能置换

```
template <class T>
void swap(T& a, T& b)
{
    T tmp(move(a));
    a = move(b);
    b = move(tmp);
}
```

如果 T 是可以移动的，即有移动构造函数和移动赋值函数，那么整个交换过程中就不会有资源的释放与申请。
如果 T 只可以复制，那么拷贝语义就会被用来进行置换，就和普通的置换语句是相同的

5. 异常

在移动构造函数中，抛出异常是一个很危险的事情，很容易就会导致在移动语义还没有完成的时候，抛出一个异常导致一些指针变成了悬空状态。

所以为了保证在移动构造函数中不抛出异常，可以为其条件一个 noexcept ，可以保证移动构造函数中跑出来的异常直接调用 termianate 终止程序，而不是导致指针悬挂的情况。

在标准库中，有一个 std::move_if_noexcept 的模板函数替代 move，该函数在类的移动构造函数没有 noexcept 关键字修饰时返回一个左值引用从而使变量可以使用拷贝语义，而在类的移动构造函数有 noexcept 关键字时，返回一个右值引用，从而使变量可以使用移动语义。

```
#include <iostream>
#include <utility>

using namespace std;

struct Maythrow
{
    Maythrow(){}
    Maythrow(const Maythrow&) {
        cout << "Maythrow copy constructor" << endl;
    }

    Maythrow(Maythrow &&) {
        cout << "Maythrow move constructor" << endl;
    }
};

struct Nothrow
{
    Nothrow(){}
    Nothrow(Nothrow&&) noexcept {
        cout << "Nothrow move constructor" << endl;
    }
    Nothrow(const Nothrow&&) {
        cout << "Nothrow move constructor" << endl;
    }
};

int main()
{
    Maythrow m;
    Nothrow n;

    // Maythrow copy constructor
    Maythrow mt = move_if_noexcept(m);
    // Nothrow move constructor
    Nothrow nt = move_if_noexcept(n);

    return 0;
}
```

move_if_noexcept 是一个牺牲性能保证安全的做法。

6. 编译器优化

编译器编译参数 `-fno-elide-constructors` 是一个禁止优化函数返回值的参数，如果编译时不加入该参数，那么从函数返回值的临时变量拷贝移动等操作都会被编译器优化掉了。

但是有些构造还是无法省略，还有一些即使是编译器优化后，也不能达到最好的结果。
而移动语义能够结局编译器无法解决的问题，所以终归是更有用的。

## 完美转发

完美转发指的是在函数模板中，完全依照模板的参数类型，将参数传递给函数模板中调用另一个函数，比如现有以下需求：

+ 有一个执行函数，根据参数类型的不同，分为左值引用和右值引用两个版本。
+ 现在有多个实例类型会调用该执行函数。

所以这两条需求就被归纳为：一个模板类的转发函数，根据参数的类型调用不同的执行函数。
假如，执行函数版本如下：


```
void IrunCodeActually(int &t)
{
    cout << "left value " << t << endl;
}

void IrunCodeActually(int &&t) {
    cout << "right value " << t << endl;
}

```

### 版本1：直接使用基本类型

```
tempalte <typename T>
void IamFrowarding (T t) {
    IrunCodeActually(t);
}

int main()
{
    int x = 10;
    IamForwardingWithRValueReference(5);   // 参数是右值
    
    IamForwardingWithRValueReference(x);   // 参数是左值  
}
```

该版本存在两个问题：
+ 在转发时会多一次对象的拷贝
+ 转发函数无法识别左值和右值，执行函数统一调用了左值引用版本

### 版本2：直接使用左值引用
```
template <typename T>
void IamFrowarding (T& t) {
    IrunCodeActually(t);
}

void Solution()
{
    int x = 10;
    IamFrowarding(5);   // 参数是右值
    
    IamFrowarding(x);   // 参数是左值  
}
```

该版本不能接受右值传入

### 版本3：使用右值引用

```
template <typename T>
void IamFrowarding (T&& t) {
    IrunCodeActually(t);
}

void Solution()
{
    int x = 10;
    IamFrowarding(5);   // 参数是右值
    
    IamFrowarding(x);   // 参数是左值  
}
```

存在的问题是，无法调用执行函数的右值引用版本

### 版本4：增加一个常左值引用的转发函数

能达到根据参数类型调用不同版本的转发函数和执行函数

```
template <typename T>
void IamFrowarding (T& t) {
    IrunCodeActually(t);
}

template <typename T>
void IamFrowarding (const T& t) {
    IrunCodeActually(t);
}


void Solution()
{
    int x = 10;
    IamFrowarding(5);   // 参数是右值
    
    IamFrowarding(x);   // 参数是左值  
} 
```

但是仍然有两个问题，
+ 一是如果和模板搭配，最后实现的代码会增加一倍；
+ 二是常左值会限制传入参数的修改，使用场景会变少

### 解决方案，转发函数参数使用右值引用 + std::forward

既能够节省空间，不会发生数据拷贝，又可以将转发函数的参数性质传递给执行函数中，这种转发就叫做完美转发。

完美转发有以下两个前提：
1. C++11 引入一种特殊的右值引用，成为转发引用，转发引用是模板函数中类型推到的一部分，它可以绑定到右值和左值。
2. 引用折叠，即将复杂的未知表达式折叠为已知的简单的表达式。具体描述是当定义中出现左值引用，引用折叠会优先将其折叠为左值引用。所以当转发模板函数的形参是右值引用，遇到左值引用时，会折叠成左值，遇到右值引用，或者左值时，折叠成右值引用。

根据以上描述，我们的转发函数可以写成以下格式：
```
template <typename T>
void IamForwording(T && t) {
    IrunCodeActually(static_cast<T &&> t);
} 
```

#### 传递左值

如果我们传入了一个左值引用，那么转发函数将被实例化为
```
void IamForwording(T& && t) {
    IrunCodeActually(static_cast<T& &&> t);
}
```

引用折叠之后
```
void IamForwording(T& t) {
    IrunCodeActually(static_cast<T& > t);
}
```

在实际调用过程中，如果 IrunCodeActually() 如果接受的是左值引用，那么就可以直接调用转发函数。
这样左值传递就通过了。

#### 传递右值

比如，我们在调用转发函数传入了一个 X 类型的右值引用时，转发函数就会被实例化为 
```
void IamForwording(T&& &&t) {
    IrunCodeActually(static_cast<T&& &&> t);
}
```

引用折叠之后就成了 
```
void IamForwording(T&& t) {
    IrunCodeActually(static_cast<T&& > t);
}
```

对于一个右值而言，当它使用右值引用表达式引用的时候，该右值引用其实就是一个左值，如果我们想在函数调用时继续传递右值，就需要使用 std::move() 进行左右值转换，而 std::move() 本质是一个 static_cast() 

如此，右值引用版本就完成了。

#### std::forward

std::forwad 的作用类同 std::move，它们之间的内部实现也很相似。就是将一个左值转化成一个右值。
使用 std::forward 可以将转发函数写成如下所示：

```
template <typename T>
void IamForwording(T && t) {
    IrunCodeActually(forward(t));
}
```

### 完美转发的样例

1. 测试 4 种类型的值

```
#include <iostream>
using namespace std;

void RunCode(int && m) {
    cout << "Rvalue ref" << endl;
}

void RunCode(int & m) {
    cout << "Lvalue ref" << endl;
}

void RunCode(const int & m) {
    cout << "Const Rvalue ref" << endl;
}

void RunCode(const int && m) {
    cout << "Const Lvalue ref" << endl;
}

template <typename T>
void perfectForward(T&& t)
{
    RunCode(forward<T>(t));
}

int main()
{
    int a;
    int b;
    const int c = 1;
    const int d = 0;

    perfectForward(a);              // lvalue
    perfectForward(move(b));        // rvalue
    perfectForward(c);              // const lvalue ref
    perfectForward(move(d));        // const rvalue ref

    return 0;
}
```

2. 记录单参数函数的参数传递状况

```
#include <iostream>

using namespace std;

template <typename T, typename U>
void perfectForward(T&& t, U& func)
{
    cout << t << " " << "forwarding..." << endl;
    func(forward<T>(t));
}

void RunCode(double&& m) {

}

void RunHome(double&& h) {

}

void RunComp(double&& c) {

}

int main()
{
    perfectForward(1.5, RunComp);
    perfectForward(8, RunCode);
    perfectForward(1.5, RunHome);

    return 0;
}

```

C++11 标准库种有很多地方都使用到了 forward，比如 make_pair, make_unique 等等。减少了函数版本的重复，并充分利用了移动语义。
