# long long 整型

1. longlong 整型分为两种，一种是有符号的 long long、unsigned long long、long long int、signed long long int；一种是无符号的，unsigned long long、unsigned long long int
2. C++11 标准要求 long long 整型可以在不同平台有不同的长度，但至少得 64 bit。
3. 我们在书写常数字面量时，可以使用 LL 后缀或是 ll 表示一个 long long 类型的字面量，而 ULL（ull、Ull、ulL）表示一个 unsigned long long 类型的字面量
4. 库文件 <climits> 里面有关于查看平台 long long 类型的大小，涉及的宏有三个：LLONG_MIN、LLONG_MAX、ULLONG_MIN

```
#include <climits>
#include <cstdio>

using namespace std;

int main()
{
    long long ll_min = LLONG_MIN;
    long long ll_max = LLONG_MAX;
    unsigned long long ull_max = ULLONG_MAX;

    printf("Min of long long : %lld\n", ll_min);            // -9223372036854775808
    printf("Max of long long : %lld\n", ll_max);            // 9223372036854775807 
    printf("Max of unsigned long long : %llu\n", ull_max);  // 18446744073709551615
    
    return 0;
}
```
