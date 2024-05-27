# 原子操作

## 原子操作

原子操作指的是“最小的且不可并行化的”操作，通常在多线程环境中对共享资源需要做原子操作的限制。
原子操作一般都是通过“互斥”访问来保证，而互斥通常需要平台相关的特殊指令。

在 POSIX 标准的 pthread 的互斥锁能保证粗颗粒度的互斥。

```
// 通过加锁的方式对 total 进行独占式访问
#include <pthread.h>
#include <iostream>

using namespace std;

static long long total = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void* func(void*)
{
    long long i;
    for(int i = 0; i < 100000000LL; i++) {
        pthread_mutex_lock(&m);
        total += i;
        pthread_mutex_unlock(&m);
    }
}

int main()
{
    pthread_t thread1, thread2;
    if(pthread_create(&thread1, NULL, &func, NULL)) {
        throw;
    }
    if(pthread_create(&thread2, NULL, &func, NULL)) {
        throw;
    }
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    cout << total << endl;
    return 0;
}
```

但是在 C++11 中，可以通过声明数据类型为原子类型保证所有线程对该数据的访问是互斥的。

```
#include <atomic>
#include <thread>
#include <iostream>

using namespace std;

atomic_llong total {0};

void func(int)
{
    for(long long i = 0; i < 100000000LL; ++i) {
        total += i;
    }
}

int main()
{
    thread t1(func, 0);
    thread t2(func, 0);

    t1.join();
    t2.join();

    cout << total << endl;
    return 0;
}
```

C++11 还提供 atomic 类模板以供使用

`atomic<T> t;`

原子类型只能从模板参数类型中进行构造，不允许拷贝、移动、以及 operator= 等

```
atomic<float> af{1.2f};
atomic<float> af1{af};  // error

float f = af;       // correct
float f1 {af};      // correct
```

<!-- 
## 内存模型，顺序一致性，与 memory_order

原子类型已经提供了一些同步的保障，但是这些保障都是建立在顺序一致性的内存模型基础上。

先了解一下基础模型，如以下所示代码

```
#include <thread>
#include <atomic>
#include <iostream>

using namespace std;

atomic<int> a{0};
atomic<int> b{0};

int valueSet(int)
{
    int t = 1;
    a = t;
    b = 2;
}

int observer(int)
{
    // 因为 valueSet() 中对设置 a b 的时间可能有多种组合，所以结果也有多种
    // {0,0},{1,0},{1,2}
    cout << "(" << a << ", " << b << ")" << endl;
}

int main()
{
    thread t1(valueSet, 0);
    thread t2(observer, 0);

    t1.join();
    t2.join();

    // 会稳定的出现 {1,2}
    cout << "Got (" << a << ", " << b << ")" << endl;

    return 0; 
}
```

以上，在 observer() 中是不会出现 (0,2) 这种组合的，因为按照顺序执行理解来看，a 的赋值一直会出现在 b 之前。

但是现在有个假设，如果 a，b 的赋值顺序在如果不影响结果的前提下，不再按照顺序执行一个函数内部的代码后，会怎么样呢？比如在本例中 a 的赋值和 b 的赋值顺序不再要求按照定义顺序执行呢？

在无依赖的代码执行过程中，如果不再考虑按照顺序执行，对于最终结果没有影响，但是对于编译器的影响确实很大，因为编译器能够将指令重新排序以提高性能。
通常情况下，编译器认定 a,b 两个变量的赋值语句执行顺序对输出结果没有任何影响的话，就能够以情况将指令重排列以提高性能。而如果 a,b 赋值语句的执行顺序必须是 a 先 b 后的话，编译器则不会做此操作。

我们假定，所有原子类型的执行顺序无关紧要，那么在多线程的情况下就可能发生严重的错误。如下代码

```
#include <thread>
#include <atomic>
#include <iostream>

using namespace std;

atomic<int> a{0};
atomic<int> b{0};

int Thread1(int)
{
    int t = 1;
    a = t;
    b = 2;
}

int Thread2(int)
{
    while (b != 2)
    {
        ;
    }
    
    cout << a << endl;
}

int main()
{
    thread t1(Thread1, 0);
    thread t2(Thread2, 0);

    t1.join();
    t2.join();
    return 0; 
}
```

Thread2 

 -->