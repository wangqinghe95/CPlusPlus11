# 宏 __clpusclpu

在 C/C++ 混编的代码中，经常会有以下的声明出现
```
#ifdef __cplusplus
extern "C" {
#endif
// code
#ifdef __cplusplus
}
#endif
```

+ 这样的声明是为了实现 C/C++ 混编，既能被 C 编译器编译，又能被 C++ 编译器编译
+ `extern "C"` 可以组织 C++ 编译器对函数名、变量名等符号进行名称重整
+ 链接器可以可靠的对这两种类型的目标文件进行链接，实现 C/C++ 混用头文件
+ `__cplusplus` 是一个有值的宏定义，并且是一个比以往标准中更大的值，在 C++11 中，该值为 201402L