KMP 理解关键

```
public static int[] getNext(String ps) {

    char[] p = ps.toCharArray();

    int[] next = new int[p.length];

    next[0] = -1;

    int j = 0;

    int k = -1;

    while (j < p.length - 1) {

       if (k == -1 || p[j] == p[k]) {

           next[++j] = ++k;

       } else {

           k = next[k];// 如果一直不一样，就会回到-1

       }

    }

    return next;

}
```

1 关键词：PMT

2 getNext其实就是一个PMT计算过程

3 A前缀，M、N任意串，x、y两个字符。AMx 和 NAy 当x!=y时，子串从y调到m的第一个字符那里
