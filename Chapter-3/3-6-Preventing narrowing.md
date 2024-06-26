# 防止类型收窄

类型收窄一般是指一些可以使得数据变化或者精度丢失的隐式类型转换。有以下几种情况
1. 浮点数隐式地转换成整型数。如 `int a = 1.2`，这里 a 实际上保存的是整数 1
2. 从高精度浮点数转换成低精度浮点数，比如从 long long 类型转换成 double，或者从 double 转换成 float。如果这些转换导致精度降低，就是类型收窄
3. 从整型转换为浮点型，整型数据大到浮点数无法精确的表示，也可以被认为是类型收窄。
4. 从整型转换成较低长度的整型，比如 `unsigned char = 1024`，1024 明显不能被一般长度为 8 bit 的 unsigned char 所容纳，所以也可以被视为类型收窄。

综上，类型收窄可以被理解为新类型无法表示原有数据的值的情况。

C++11 中，使用初始化列表数据编译器会检查其是否发生类型收窄。

```
const int x = 1024;
const int y = 10;

// 虽然会发生类型收窄，但是不会引发编译失败，只能有警告
char a = x;                 // 收窄，可以通过编译
char* b = new char(1024);   // 收窄，可以通过编译

// 使用初始化列表会因为类型收窄导致编译无法通过
char c = {x};           // 收窄，无法通过编译
char d = {y};           //  可以通过编译
unsigned char  e(-1);   // 收窄，无法通过编译

float f{7};             // 可以通过编译
int g{2.0f};            // 收窄，无法通过编译
float *h = new float{1e48}; // 收窄，无法通过编译
float i = 1.2l;         // 可以通过编译
```

C++11 中，列表初始化是唯一一种可以防止类型缩窄的初始化方式，也是列表初始化区别于其他初始化方式的地方。
编译器会在发生类型收窄的地方提示。