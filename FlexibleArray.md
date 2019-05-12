### 柔性union成员
STL union obj
```c
enum {_ALIGN = 8};  // 小型区块的上调边界
enum {_MAX_BYTES = 128}; // 小区区块的上限
enum {_NFREELISTS = 16}; // _MAX_BYTES/_ALIGN  free-list 的个数

// free-list 的节点结构，降低维护链表 list 带来的额外负担
union _Obj {
    union _Obj* _M_free_list_link;  // 利用联合体特点
    char _M_client_data[1];    /* _M_client_data 和 _M_client_data[1]的区别注意下 */
};
static _Obj* __STL_VOLATILE _S_free_list[_NFREELISTS];  // 注意，它是数组，每个数组元素包含若干相等的小额区块
 ```
 
☆ 柔性union成员地址就是变量地址
https://blog.csdn.net/hx_lfeng/article/details/78163361

### 柔性数组成员
http://www.nowamagic.net/academy/detail/1204478
