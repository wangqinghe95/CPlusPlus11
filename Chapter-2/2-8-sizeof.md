# 非静态成员的 sizeof

sizeof 是一个特殊的运算符
在 C++11 中对非静态成员的使用 sizeof 是合法的。

C++98 中，在类实例化之前无法获取类的成员大小，但是通常采用以下方法

```
sizeof(((<class_name>*)0)-><class_member>)
sizeof(((Person*)0)->hand)
```

在 C++11 中可以直接获取
```
sizeof(Person::hand)
```

