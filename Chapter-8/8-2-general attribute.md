# 通用属性

C++ 编译器厂商为了更好的性能，会设计出一些特性来扩展 C++ 语法，这些特性并不在 C++ 的标准里。有时候这些特性也会被纳入到标准中。

而属性，也即是 attribute 就是扩展语法中最常见的一个特性。它是对语言中的实体对象，比如函数、变量、类型等附加一些额外注解信息，用来实现一些语言或者非语言层面上面的功能，或者实现优化代码等一种手段。

不同的编译器有不同的属性语法，比如对于 g++，属性是通过 GUN 关键字 __attribute__ 来声明的。只需要如下声明

`__attribute__ ((attribute-list))`

就可以为程序中的函数、变量和类型设定一些额外信息，以便编译器可以进行错误检查和性能优化等。

Windows 平台上，有另外一种关键字 __declspec，是微软用于指定存储类型的扩展属性关键字。

## C++11 通用属性

C++11 通用属性使用了左右双中括号的形式
`[[attribute-list]]`
这样做的好处是，既不会消除语言添加或者重载关键字的能力，又不会占用用户空间的关键字的名字空间。

1. C++11 通用属性可以作用于类型，变量，名称，代码块等。
2. 对于作用于声明的通用属性，即可以写在声明的起始处，也可以写在声明的标识符之后。
3. 而对于作用于整个语句的通用属性，则应该写在语句的起始处

例子1：
```
// attr1 出现在函数定义之前，attr2 出现在函数定义之后，两者均可以作用于函数 func
[[ attr1 ]] void func [[ attr2 ]]();

// 同上类似，该声明中两个属性区别是作用于数组之中
[[ attr1 ]] void array [[ attr2 ]][10];
```

例子2：
```
// 声明了类 C，及其类型的变量 c1，c2，一共 5 个不同的属性
// attr1，attr4 作用于 c1，attr1，attr5 作用于 c2
// attr2 出现在声明之后，只作用于类 C
// attr3 作用的对象和具体实现有关
[[ attr1 ]] class C [[ attr2 ]] {} [[ attr3 ]] c1 [[ attr4 ]], c2 [[attr5]];
```


例子3：
```
// attr1 只作用于 L1
[[ attr1 ]] L1;
/*
*  attr2 修饰 case1
* attr3 修饰 case2
* attr4 修饰 break
* attr5 修饰 default 表达式
*/
switch (value) {
    [[ attr2 ]] case1 : // do something...
    [[ attr3 ]] case2 : // do something...
    [[ attr4 ]] break;
    [[ attr5 ]] default : // do something...
}

// attr1 作用于 goto 语句
[[ attr6 ]] goto L1;
```

下面的 for 语句也是类似

```
// attr1 作用 for 表达式； attr2 作用 return
[[attr1]] for(int i = 0; i < top; i++>) {
    // do something
}
[[attr2]] return top;


// attr1 作用于函数 func，attr2 作用于整型参数 i，attr3 作用于整型参数 3
// attr4 作用于 return 语句
[[ attr1 ]] int func( [[ attr2 ]] int i, [[ attr3 ]] int j){
    [[attr4]] return i+j;
}
```

C++11 标准中，只预定义了两个通用属性，分别是 [[ noreturn ]] 和 [[ carries_dependency ]]

## 预定义的通用属性

如上述所说，C++11 预设了 [[ noreturn ]] 和 [[ carries_dependency ]] 两个通用属性。

### [[ noreturn ]]

[[ noreturn ]] 用于表示不会返回的函数。函数不会返回和没有返回值的函数是两回事，前者直接再次停止执行，后者会继续返回到函数调用的地方继续执行

[[ noreturn ]] 表示那么不会将控制流返回给源调用者的函数。典型的例子有：终止应用程序语句的函数，有无限循环语句的函数，有异常抛出的函数等。该属性能够帮助编译器产生警告信息，同时也可以做一些优化工作，比如死代码消除，免除为函数调用者保存一些特定寄存器等

```
#include <iostream>

using namespace std;

void dosomething1()
{
    cout << __func__ << endl;
}

void dosomething2()
{
    cout << __func__ << endl;
}

[[noreturn]] void ThrowAway()
{
    throw "expection";      // 控制流跳转到异常处理
}

void Func()
{
    dosomething1();
    ThrowAway();
    dosomething2();     // 函数不可达
}

int main()
{
    Func();
    return 0;
}
```

除了异常抛出的函数外，终止应用程序的函数和无限循环的函数也可以用该属性修饰

```
[[ noreturn ]] void abort(void) noexcept;
```

abort 会导致程序终止运行，甚至连自动变量的析构函数以及本该在 atexit() 时调用的函数都不会再去处理就直接退出了。因此使用该属性值修改该函数可以让编译器优化。

但是使用该属性值需要主要，不要对有返回值的函数错误使用 [[ noreturn ]]

如下：
```
// 编译器会报警告
#include <iostream>
using namespace std;

[[ noreturn ]] void Func(int i)
{
    if(i < 0) throw "negative";
    else if(i > 0) throw "positive";
}

int main()
{
    Func(0);
    cout << "Returned" << endl;
    return 0;
}
```

## [[ carries_dependency ]]

该属性值是在并行情况下编译器优化相关。主要是解决弱内存模型平台中 memory_order_consume 内存顺序问题