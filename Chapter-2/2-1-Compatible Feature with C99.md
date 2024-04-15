# 第二章 保证兼容性和稳定性

## 2.1 保证和 C99 兼容

C++11 兼容了 C99 的一些宏定义

|宏名称|功能描述|
|---|---|
|__STDC_HOSTED__|编译器的目标系统环境中是否包含完整的标准C库，若是，值为1|
|__STDC__|C编译器中表示编译器的实现是否与 C 标准一致，C++11 标准表示是否定义以及定义什么值由编译器决定|
|__STDC_ISO_10646__|该宏是以yyyymmL 格式整数常量，表示C++编译器环境符合某个版本的 ISO/IEC 10646 标准 |

[C++11 和 C99 兼容的宏](./Code/macro_STDC.cpp)

### 2.1.2 __func__ 预定义标识符

功能是返回所在函数的名字

1. 允许该标识符出现在类或者结构体里
2. 不允许作为函数参数的默认值

### 2.1.3 _Pargma

`#pragma once`
预处理指令，指示编译器该头文件只会编译一次。
效果等同于
```
#ifndef
#defint
#endif
```

C++11 使用 _Pragma 操作符等效替代`#pragma`，使用方法如下：
`_Pragma (字符串字面量)`

等效 `#pragma once` 的表达式为 `_Pragma ("once")`

_Pargma 还有一个用处在于使用在宏里，如下
```
#define CONCAT(x) PRAGMA(concat on #x)
#define PRAGMA(x) _Pragma(#x)
CONCAT(..\concat.dir)
```

在这里，CONCAT(..\concat.dir) 最终会编译产生 _Pragma(concat on "..\concat.dir") 效果
而 `#pragma ` 是不能在宏中展开的

### 2.1.4 变长参数的宏定义以及 __VA_ARGS__

C99 标准中，程序员可以使用变长参数列表的宏定义来实现函数的不定参数个数。
变长参数的宏定义是指在宏定义中参数列表中的最后一个参数为省略号，而预定义宏 __VA_ARGS__ 则可以在宏定义的实现部分替换省略号所代表的字符串，比如
`#define PR(...) printf(__VA_ARGS__)`

这样就定义了一个 printf 的别名 PR

下面是一个简单的变长参数宏的应用
```
#include <stdio.h>

#define LOG(...) {\
    fprintf(stderr, "%s: Line %d:\t", __FILE__, __LINE__);  \
    fprintf(stderr, __VA_ARGS__);   \
    fprintf(stderr, "\n");  \
}

int main()
{
    int x = 3;
    LOG("x = %d", x);

}
```




