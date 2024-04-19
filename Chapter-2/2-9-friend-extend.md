# 扩展的 friend

friend 关键字用来声明类的友元，可以无视类中的成员的属性。

C++11 声明友元时，不要加上关键字 class，也可以使用别名
这个特性可以运行在类模板中使用友元。

```
class p;
template<typename T>
class People
{
    friend T;
};

People<P> PP;
People<int> Pi;
```

对于 People 这个模板类，在使用类 P 做参数时， P 就是一个 friend 类，而在使用内置类型 int 作为模板参数时，会被实例化为一个普通的没有友元定义的类型。
通过模板实例化才确定一个模板类是否有友元，以及谁是这个模板类的友元。

