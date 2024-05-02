# final/override

## final
+ 阻止派生类继续继承父类的虚函数
+ 可以和 virtual 一起在父类中修饰一个函数，但是这样的做毫无意义

## override
+ 一个被 override 修饰的虚函数，在派生类中必须被重载，否则无法编译
+ 为了更好的实现继承结构复杂的类型，也是为了帮助编译器更好的检查虚函数继承结构，防止虚函数的函数名，函数类型或者重写了父类的虚函数。