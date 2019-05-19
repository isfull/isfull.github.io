## 右值引用

在C++11中, 右值引用是一种新的引用类型, 但它仅能绑定在右值上, 语法是T &&, 将原来C++03/98中的引用类型T &称为左值引用. 
右值引用主要是用在临时变量的处理上，如
```
string k（x+y）// x+y会产生临时变量
string(string&& aa)
{
    data = aa.data;
    aa.data = nullptr;
}
```

其中x+y生成的临时变量对应的数据空间会在执行后释放，那通过右值引用进度构造函数时，可以直接拿走赋值给k，达到节省操作的目标。

注意, T &&不是”对引用的引用”, 就是右值引用, C++中没有”引用的引用”.
```
            lvalue   const lvalue   rvalue   const rvalue
---------------------------------------------------------              
X&          yes
const X&    yes      yes            yes      yes
X&&                                 yes
const X&&                           yes      yes          无用
```
所以, 右值引用其实就是一种引用类型, 但它仅能绑定在右值上

C++11在<utility>头文件中加入了std::move，把左值变成右值
```
unique_ptr<std::string> c(std::move(a));
```


## int & &&降格到int &的类型推导过程, 被称为collapsed
```
#include <iostream>

template<typename T>
void foo(T&& t) { // 右值引用定义
    std::cout << t << std::endl;
}

int main(void) {
    foo(23);

    foo("i love you");

    int a = 2333;

    foo(a);         // <-- 可以使用一个左值去调用foo模板函数!
    return 0;
}
```

如何强制定义只能右值引用传值
```
template<typename T>
typename std::enable_if<std::is_rvalue_reference<T&&>::value, void>::type
foo(T&& t) {
    std::cout << t << std::endl;
}
```
在模板参数的类型推导中, 有一个特殊逻辑叫collapsed, std::move的实现就与这个特性有关

