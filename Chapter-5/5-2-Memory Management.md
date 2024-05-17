# 内存管理

## 显式内存管理

由于 C/C++ 对内存管理的自由，虽然这样会使得程序的运行效率的上限很高，但是如果技术不精的情况下，也很容易产生很多严重的问题，比如程序崩溃，内存泄漏等等。

C/C++ 内存分配和释放时产生很多问题都可以归结成以下几种情况：

1. 野指针。一些内存单元已经被释放，但是指向它的指针还在使用，这就会导致无法预测的错误。
2. 重复释放。程序试图去释放已经释放过的内存单元，或者释放已经被重新分配过的内存单元，就会导致重复释放错误。这将会导致程序崩溃。
3. 内存泄漏。当不在使用的内存单元如果没有被释放就会导致内存泄漏，如果程序不断重复进行该类操作，就会使内存占用剧增，最终导致内存耗尽，程序崩溃。

基于此， C++11 改进了只能指针，而且标准库还提供了所谓的“最小垃圾回收”的支持

## C++11 的智能指针

C++11 标准中使用 unique_ptr、shared_ptr、和 weak_ptr 三个智能指针来实现自动回收堆分配对象的功能。

1. unique_ptr：该类型的指针独占指向的内存单元，不能被复制和拷贝到其他同类型的指针中，但是可以移动
2. shared_ptr：允许多个智能指针共享一个对分配对象内存。内部通过引用计数的形式统计管理该对象内存的生存期，当统计个数为 0 时释放对象内存空间。
3. weak_ptr：是一个弱引用类型，指向一个 shared_ptr 指针所指向的内存空间，但是不拥有该内存对象。用于验证 shared_ptr 类型数据的有效性。

```
#include <iostream>
#include <memory>
using namespace std;

void uniqueptr_usage()
{
    cout << __func__ << " start" << endl;
    unique_ptr<int> up1(new int(11));
    // unique_ptr<int> up2 = up1;  // unique_ptr 类型不能拷贝
    cout << "*up1 : " << *up1 << endl;  // 11

    unique_ptr<int> up3 = move(up1);    // unique_ptr 可以移动

    cout << "*up3 : " << *up3 << endl;  // 11
    // cout << "*up1 : " << *up1 << endl;   // up1 所指内存已经被移动了，运行时错误

    up3.reset();    // unique_ptr 显式释放内存
    up1.reset();    // 不会导致运行时错误

    // cout << "*up3 : " << *up3 << endl;   // 运行时错误，因为内存已经被释放了
    cout << __func__ << " end" << endl;
}

void sharedptr_usage()
{
    cout << __func__ << " start" << endl;
    shared_ptr<int> sp1(new int(22));
    shared_ptr<int> sp2 = sp1;
    cout << "sp1 : " << *sp1 << endl;     // 22
    cout << "sp2 : " << *sp2 << endl;     // 22
    sp1.reset();
    cout << "sp2 : " << *sp2 << endl;     // 22
    cout << __func__ << " end" << endl;

}

void checkPtr(weak_ptr<int>& wp) 
{
    shared_ptr<int> sp = wp.lock();
    if(sp != nullptr) {
        cout << "still " << *sp << endl;
    }
    else {
        cout << "pointer is invalid" << endl;
    }
}

void weakptr_usage()
{
    cout << __func__ << " start" << endl;
    shared_ptr<int> sp1(new int(33));
    shared_ptr<int> sp2 = sp1;

    weak_ptr<int> wp = sp1;
    checkPtr(wp);                        // still 22

    sp1.reset();
    cout << "sp2 : " << *sp2 << endl;     // 33
    checkPtr(wp);

    sp2.reset();
    checkPtr(wp);                         // pointer is invalid
    cout << __func__ << " end" << endl;
}

int main()
{
    uniqueptr_usage();
    sharedptr_usage();
    weakptr_usage();
    return 0;
}
```

## 垃圾回收的分类

“垃圾” 指的是之前使用，现在不再使用，或者没有任何指针再指向的内存空间被成为“垃圾”
而将这些“垃圾”收集起来再次利用的机制被称为“垃圾回收”

垃圾回收的方式有很多

### 基于引用计数的垃圾回收器

使用系统记录被引用对象的次数，当对象被引用的次数变为 0 时，该对象即被视为“垃圾”而回收。
此种方法需要注意环形引用的问题，并且由于计数带来的额外开销也不小，所以实用性上也有一定的限制。

### 基于跟踪处理的垃圾回收期

1. 标记-清除

该算法将程序中正在使用的对象视为根对象，从根对象开始查找它们所引用的的堆空间，并在这些堆空间中做标记，当标记结束后，所有被标记的对象就是可达对象或者或对象，而没有被标记的对象被认为是垃圾，在第二步的清扫阶段会被回收掉

该方法的特点是活的对象不会被移动，但是该方法会导致大量的内存碎片问题。

2. 标记-整理
同 方法 1 类型，但是标记完以后不再遍历所有对象清扫，而是将活的对象向左靠齐，解决了方法 1 的内存碎片问题。
特点是会移动活的对象，相应的，程序中所有对堆内存的引用必须更新。

3. 标记-拷贝
该算法将堆空间分为两部分，From 和 to
刚开始系统只从 From 的堆空间里面分配内存，当 From 分配满的时候系统就开始垃圾回收了：从 From 堆空间中找到所有活的对象，拷贝到 To 的堆空间中，这样 From 中就是剩下垃圾了。
而对象被拷贝到 To 之后，在 To 里都是紧凑排列的，然后将 From 和 To 交换一下角色，接着继续从 From 里分配空间。

该方法存在的问题是堆的利用率只有一半，也需要移动活的对象。

## C++ 与垃圾回收

## C++11 与最小垃圾回收支持

## 垃圾回收的兼容性

