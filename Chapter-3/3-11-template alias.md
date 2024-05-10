# 模板别名

在 C++11 中，除了能够使用 typedef 可以给变量定义别名，还能够使用 using 给模板定义别名

```
template<typename T> using MapString = std::map<T, char*>;
MapString<int> numberedString;
```
