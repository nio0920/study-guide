## Dijkstra——贪心算法

贪心法在解决这个问题的策略上目光短浅，仅仅依据当前已有的信息就做出选择，并且一旦做出了选择。无论将来有什么结果，这个选择都不会改变。

**一句话：不求最优，仅仅求可行解。**



对于一个详细的问题，怎么知道是否可用贪心算法解此问题，以及是否能得到问题的最优解?

我们能够依据贪心法的2个重要的性质去证明：**贪心选择性质和最优子结构性质**。

### 1、贪心选择

什么叫贪心选择？从字义上就是贪心也就是目光短线。贪图眼前利益。在算法中就是仅仅依据当前已有的信息就做出选择，并且以后都不会改变这次选择。（这是和动态规划法的主要差别）　　  
 所以对于一个详细问题。要确定它是否具有贪心选择性质，必须证明每做一步贪心选择是否终于导致问题的总体最优解。

### 2、最优子结构

当一个问题的最优解包括其子问题的最优解时，称此问题具有最优子结构性质。



```
从一个顶点到其余顶点的最短路径
```

设`G=(V,E)`是一个带权有向图，把图中顶点集合V分成两组，第1组为已求出最短路径的顶点（用S表示，初始时S只有一个源点，以后每求得一条最短路径`v,...k`，就将k加到集合S中，直到全部顶点都加入S）。第2组为其余未确定最短路径的顶点集合（用U表示），按最短路径长度的递增次序把第2组的顶点加入S中。

    步骤：

    1. 初始时，S只包含源点，即`S={v}`，顶点v到自己的距离为0。U包含除v外的其他顶点，v到U中顶点i的距离为边上的权。
    2. 从U中选取一个顶点u，顶点v到u的距离最小，然后把顶点u加入S中。
    3. 以顶点u为新考虑的中间点，修改v到U中各个点的距离。
    4. 重复以上步骤知道S包含所有顶点。



[参考文章1](http://blog.csdn.net/wang704987562/article/details/70991590)

[参考文章2](https://www.cnblogs.com/dongkuo/p/5584610.html)

[参考文章3](http://blog.csdn.net/effective_coder/article/details/8736718)



## Floyd——动态规划

[参考文章1](http://blog.csdn.net/baidu_28312631/article/details/47418773)

[参考文章2](http://blog.csdn.net/sun897949163/article/details/52077460)

[参考文章3](http://blog.csdn.net/woshioosm/article/details/7438834)
