## Leetcode

## 源码

>https://github.com/isfull/longshort/tree/master/leetcode

## 重点解析

<hr style=" height:1px;border:none;border-top:2px #185598;" />

#### 136. Single Number

>1 取巧的方式，使用异或运算过滤重复

>2 使用vector的data方法，直接操作数组，比使用迭代器快很多

```
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int rs = 0;
        vector<int>::const_iterator i = nums.begin();
        for (;i!=nums.end();i++){
            rs ^= *i;
        }
        return rs;
    }
};

16ms !

class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int rs = 0;
        int* p = nums.data();
        int i = 0;
        int size = nums.size();
        for (;i<size;p++,i++){
            rs ^= *p;
        }
        return rs;
    }
};

12ms !

```

<hr style=" height:1px;border:none;border-top:2px #185598;" />