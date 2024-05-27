# 快速退出

C++ 程序中，有很多有关“终止”的函数，比如 terminate、abort、exit 等
虽然都是终止程序，但是他们还是有很多差别的，因为其对应了正常退出和异常退出两种情况。

## terminate
该函数是 C++ 异常处理的一部分。一般指的是没有被捕获的异常都会导致 terminate 函数的调用。

只要 C++ 程序中出现了非程序员预期的行为，就都会导致 terminate 的调用。
而 terminate 函数在默认情况下，会去调用 abort 函数。
程序员也可以通过 set_terminate 函数来改变 terminate 默认的行为。

## abort

abort 函数更底层，该函数不回去调用析构函数，默认情况下该函数会向合乎 POSIX 标准的系统抛出一个信号（signal）：SIGABRT

如果程序员为该信号设定一个信号处理程序的话，操作系统将默认地释放进程所有资源，从而终止程序。

abort 可能会带来一些问题，比如终止的应用进程正在与其他应用进程软件有一些交互，那么突然 abort 进程后会导致交互进程处于一些“中间态”，进而出来一些问题。

## exit

exit 属于正常退出范畴中的程序终止，会自动调用变量的析构函数，而且还会调用 atexit 注册的函数，这和 main 函数结束时清理工作的步骤是一样的。

```
#include <cstdlib>
#include <iostream>

using namespace std;

void openDevice()
{
    cout << "device is opened." << endl;
}

void resetDeviceStat()
{
    cout << "device stat is reset." << endl;
}

void closeDevice()
{
    cout << "device is close." << endl;
}

int main()
{
    atexit(closeDevice);
    atexit(resetDeviceStat);
    openDevice();
    exit(0);
}

/*
device is opened.
device stat is reset.
device is close.
*/
```

需要注意的是，注册函数被调用的顺序与注册的顺序刚好相反，这多少和析构函数执行预期声明顺序相反是一致的。

exit 和 atexit 函数的配合，可以让我们灵活的处理一些进程级的清理工作，这对一些静态、全局变量来说非常有用。

main 函数或者使用 exit 函数调用程序结束的方式也有一些并不友好的地方，比如程序中的类会有一些在堆中的零散的大量内存，main 或者 atexit 函数护调用类的析构函数一次释放这些零散的内存。但实际上这些内容完全可以将给操作系统统一回收。

另外在多线程的环境中，使用 exit 函数退出程序的话，通常需要向线程发出一个信号，并等待线程结束再执行析构函数，atexit 注册函数等。这种步骤从语法说没有问题，但是这种退出方式有时候并不能如我们预期般有效工作，比如线程中程序再等待 I/O 运行结束等，在一些更为复杂的情况下， 可能还会遭遇一些因为信号顺序而导致的死锁状态。

C++ 引入一个 quick_exit 函数，该函数并不执行析构函数，而是直接使程序终止。与 abort 函数不同，abort 的结果通常是异常退出，甚至还可能进行 coredump 等，而 quick_exit 等同于 exit，同属正常退出，此外使用 at_quick_exit注册的函数也能够在 quick_exit 时候被调用。

```
#include <iostream>
#include <cstdlib>

using namespace std;

struct A {
    ~A(){
        cout << "Destruct A" << endl;
    }
};

void closeDevice()
{
    cout << "device is close." << endl;
}

int main()
{
    A a;
    at_quick_exit(closeDevice);
    quick_exit(0);
}

// device is close.
```

只执行了注册函数，而没有执行类的析构函数

