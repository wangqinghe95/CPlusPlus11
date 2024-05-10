# 内联名字空间

C++11 标准引入了名字空间，为了分隔全局共享的变量名和函数名。
通过命名空间，可以通过 **空间名:函数/变量名** 形式引用特定的版本，解决了 C 中名字冲突的问题。

在不同的命名空间中，可以有相同名称的变量，函数名，
```
// Jim_Namespace.hpp
#include <iostream>

using namespace std;

namespace Jim
{
    namespace Basic 
    {
        struct Knife {
            Knife(){
                cout << "Knife in Basic." << endl;
            }
        };
        class CorkScrew{};
    }
    namespace Toolkit {
        template<typename T> class SwissArmyKnife{};
    }

    namespace Other{
        // Knife b;
        struct Knife
        {
            Knife() {
                cout << "Knife in Other." << endl;
            }
        };
        Knife c;
        Basic::Knife k;
    }
}

// main.cpp
#include <iostream>
#include "namespace.hpp"
using namespace Jim;

int main()
{
    Toolkit::SwissArmyKnife<Basic::Knife> sknife;

    return 0;
}
```

但是如果一个名称空间中有多个子名称空间，会有一些问题存在。如上，当我声明一个 sknife 时, 非常不想写那么长的类型声明。

那么就可以在 Jim 这个命令空间的尾部打开其中需要用到的子命名空间
```
using namespace Basic;
using namesapce Toolkit;

// main.cpp
SwissArmyKnife<Knife> sknife;
```

## 命名空间中模板的特化

如果想要将命名空间中的模板函数特化，那么在旧版本中会报编译失败错误，但是 C++11 引入了一个新特性，叫做内联的名字空间，即通过关键字 “inline namespace” 声明一个内联的名字空间，内联的名字空间允许程序员在父名字空间定义和特化子名称空间的模板。

```
namespace Jim
{
    inline namespace Basic 
    {
        struct Knife {
            Knife(){
                cout << "Knife in Basic." << endl;
            }
        };
        class CorkScrew{};
    }
    inline namespace Toolkit {
        template<typename T> class SwissArmyKnife{};
    }
    namespace Other{
        Knife b;
        struct Knife
        {
            Knife() {
                cout << "Knife in Other." << endl;
            }
        };
        Knife c;
        Basic::Knife k;
    }
}

// main.cpp
#include <iostream>
#include "namespace.hpp"

namespace Jim {
    template<> class SwissArmyKnife<Knife>{};
}
using namespace Jim;

int main()
{
    // Toolkit::SwissArmyKnife<Basic::Knife> sknife;

    SwissArmyKnife<Knife> sknife;
    return 0;
}
```

加上关键字 `inline` 就可以在使用空间处特化别处的命名空间中模板函数的特化了。

但是注意，当特化功能实现后，子命名空间的 other 中，`Knife b;` 的定义也可以通过编译了。而且其是 Basic::Knife 的类型。换句话说，即在子名称空间的 Basic 中的名字现在看起来像和跟父名称空间 Jim 一样了。这样一来，名字空间的分隔属性被破坏了。
如果想要做到这一点直接把需要用到的类型或者变量放到全局名字空间即可，不需要 inline 这样复杂。

所以 inline 最重要的使用方式并不在此。

如下所示：

```
namespace Jim {
#if _cplusplus == 201103L
    inline
#endif
    namespace cpp11 {
        struct Knife {
            Knife(){
            cout << "Knife in C++11 << endl;
        }};
    }

#if _cplusplus < 201103L
    inline
#endif
    namespace oldcpp {
        struct oldcpp {
            Knife(){
            cout << "Knife in old c++ << endl;
        }};
    }
}
```

在 Jim 这个命名空间中包含了两个名字空间，cpp11 和 oldcpp，并且使用了 cplusplus 的条件编译。如果当前 C++ 版本等于这个常数时，就将名字空间 cpp11 内联到 Jim 中，小于这个常数就将 oldcpp 内联到 Jim 中。编译器就可以根据当前 C++11 版本，选择合适的实现版本，而且有需要的话，我们也可以使用名字空间来访问相应名字空间的类型、数据、函数等。

此外，匿名的名字空间也可以将包含的名字导入到父名字空间中，但是无法允许在父名字空间的模板特化。这也是 C++11 中为什么要引入新的内联名字空间的一个根本原因。
