# 原生字符串字面量

原生字符串字面量是一种不需要添加控制字符来调整字符串格式的声明方式。

如下：

```
#include <iostream>

using namespace std;

int main()
{
    cout << R"(hello\n
                world)" << endl;
    return 0;
}
/*
hello\n
                world
*/
```

所有的控制字符和换行符等等都会如实的被打印出来。

Unicode 字符串也可以通过这种方式打印

```
int main()
{
    cout << u8R"(\u4F60, \n\u597D\u554A)" << endl;

    cout << u8R"(你好)" << endl;

    cout << sizeof(u8R"(hello)") << "\t" << u8R"(hello)" << endl;
    cout << sizeof(uR"(hello)") << "\t" << uR"(hello)" << endl;
    cout << sizeof(UR"(hello)") << "\t" << UR"(hello)" << endl;

    return 0;
}
```

原生字符串也像 C 的字符串字面量一样遵守连接规则

```
#include <iostream>

using namespace std;

int main()
{
    char u8string[] = u8R"(你好)" " = hello";
    cout << u8string << endl;
    cout << sizeof(u8string) << endl;

    return 0;
}
```