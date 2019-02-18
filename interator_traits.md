```
template<class Iterator>
struct iterator_traits
{
    typedef typename Iterator::value_type value_type;
...
};

template<class T>
struct iterator_traits<T*>
{
    typedef T value_type;
...
};

```
理解类型提取的关键： 定义完多种同名模板结构体、类、函数后，在使用时优先匹配最合适的那一个定义，这是template机制的一个特性。
