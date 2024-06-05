# CPlusPlus11

## Introduction

| Book | Author | Staus | Remark |
| -- | -- | -- | -- |
| 《Understanding C++11 Anlaysis and Application of New Feature》 | IBM 编译器中国开发团队 | Done | None |

## Content

### Chapter-2
+ [C++ 11 兼容 C99 特性](./Chapter-2/2-1-Compatible%20Feature%20with%20C99.md)
+ [long long 类型](./Chapter-2/2-2-long%20long.md)
+ [扩展的整型](./Chapter-2/2-3-Extended%20int.md)
+ [_cpluscplus 宏](./Chapter-2/2-4-__cpluscplus.md)
+ [静态断言](./Chapter-2/2-5-static_assert.md)
+ [noexcept 修饰符和操作符](./Chapter-2/2-6-noexcept.md)
+ [快速初始化成员变量](./Chapter-2/2-7-Initialization.md)
+ [非静态成员变量的 sizeof](./Chapter-2/2-8-sizeof.md)
+ [扩展的 friend 语法](./Chapter-2/2-9-friend-extend.md)
+ [final/override](./Chapter-2/2-10-final-override.md)
+ [函数的默认模板参数](./Chapter-2/2-11-Default%20template%20parameter.md)
+ [外部模板](./Chapter-2/2-12-External%20template.md)
+ [局部类型用作模板参数](./Chapter-2/2-13-Template%20Parameters.md)

### Chapter-3
+ [继承构造函数](./Chapter-3/3-1-Inherit%20constructor.md)
+ [委托构造函数](./Chapter-3/3-2-delegating%20constructor.md)
+ [右值引用，移动语义，完美转发](./Chapter-3/3-3-Rvalue%20reference.md)
+ [显示转换操作符](./Chapter-3/3-4-Explict%20Conversion%20opeartions.md)
+ [初始化列表](./Chapter-3/3-5-Initializer%20list.md)
+ [防止类型收窄](./Chapter-3/3-6-Preventing%20narrowing.md)
+ [POD（plain old data）](./Chapter-3/3-7-POD%20Type.md)
+ [非受限联合体](./Chapter-3/3-8-Unrestricted%20union.md)
+ [用户定义的字面量](./Chapter-3/3-9-user-defined%20literals.md)
+ [内联名字空间](./Chapter-3/3-10-Inline%20namespace.md)
+ [模板别名](./Chapter-3/3-11-template%20alias.md)
+ [一般化的 SFINAE 规则](./Chapter-3/3-12-Generalized%20SFINAE%20rules.md)

### Chapter-4
+ [Auto 类型推导](./Chapter-4/4-2-Auto.md)
+ [decltype](./Chapter-4/4-3-Decltype.md) 
+ [追踪返回类型](./Chapter-4/4-4-Trailing%20return%20type.md)
+ [基于范围的 for 语句](./Chapter-4/4-5-range-based%20for%20statement.md)

### Chapter-5
+ [强枚举类型](./Chapter-5/5-1-Enum%20class.md)
+ [内存管理](./Chapter-5/5-2-Memory%20Management.md)

### Chapter-6
+ [常量表达式](./Chapter-6/6-1-constexpr.md)
+ [变长模板](./Chapter-6/6-2-variadic%20templates.md)
+ [原子操作](./Chapter-6/6-3-atomic.md)
+ [线程局部存储](./Chapter-6/6-4-thread-local%20storage.md)
+ [快速退出](./Chapter-6/6-5-quick-exit.md)

### Chapter-7
+ [指针空值](./Chapter-7/7-1-nullptr.md)
+ [显示默认和删除函数（默认的控制）](./Chapter-7/7-2-defaulted%20and%20deleted%20functions.md)
+ [lambda 函数](./Chapter-7/7-3-lambda.md)

### Chapter-8
+ [对齐支持](./Chapter-8/8-1-alignment%20support.md)
+ [通用属性](./Chapter-8/8-2-general%20attribute.md)
+ [Unicode](./Chapter-8/8-3-Unicode.md)
+ [原生字符串字面量](./Chapter-8/8-4-raw%20string%20literals.md)


## Summary

### Chapter 2
1. 引入了 4 个 C99 预定的宏，__func__ 预定义标识符，_Pargma 操作符，变长参数定义以及宽窄字符连接等概念。
2. 更新了 _cplusplus 宏的值
3. 纳入了 C99 的 longlong 类型，将扩展整型的规则定义好，这样就可以保证各个编译器扩展内置类型遵守统一的规则。
4. 统一静态断言，做成了编译器级别的支持。
5. 抛弃 throw() 异常描述符，新增可以推导是否可以抛出异常的 noexcept 异常描述符
6. 对非静态成员初始化做了改进，允许 sizeof 直接作用于类成员
7. 对 friend 声明给予一定的拓展，以方便通过模板方式指定某个类是否是其他类或者函数的友元
8. 新增 final/override 
9. 把默认参数模板概念延伸到模板函数中，并且允许局部函数和匿名类型做模板参数

### Chapter 3

1. 新增右值引用特性，借用该特性实现移动语义，并实现移动构造函数；利用右值引用完成完美转发，其包括引用折叠，模板推导等。
2. 实现简单类型和复杂类型看，介绍 POD，划分平凡和标准布局。
3. 引入列表初始化
4. 新增继承构造函数和委派构造函数
5. 介绍避免意外的显示构造转换，以及能为类产生变长成员的非受限联合体。
6. 内联的名字空间
7. SFINAE给规则

### Chapter 4
1. 解决双右尖括号语法问题
2. 类型推导的改进，auto，decltype，以及追踪返回类型函数声明
3. 基于范围的 for 循环和 auto 的搭配

### Chapter 5
1. 新增强类型枚举
2. 智能指针的改进
3. 垃圾回收

### Chapter 6
1. 常量表达式 constexpr
2. 变长模板
3. 原子类型的定义，包括影响的内存顺序
4. 线程局部存储
5. 快速退出 quick_exit

### Chapter 7
1. nullptr
2. =default/=delete
3. lambda 表达式

### Chapter 8
1. 对齐方式
2. 通用属性
3. Unicode
4. 原生字符串字面量