# 基于范围的 for 循环

C++11 引入了基于范围的 for 循环

```
#include <iostream>

using namespace std;

int main() 
{
    int arr[5] = {1,2,3,4,5};
    for(int &e : arr) {
        e *= 2;
    }

    for(auto & e : arr) {
        cout << e << '\t';
    }

    return 0;
}
```

基于范围的 for 循环跟普通循环一样，可以用 continue 退出本次访问，break 跳出整个循环

能否使用基于范围的 for 循环条件是 for 循环的范围是可确定的