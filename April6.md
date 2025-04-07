## plan
今天下午回的学校，打算晚上把剩下寄到算法题写完。

## dfs & bfs

广度优先搜索（Breadth-First Search, BFS）与深度优先搜索（Depth-First Search, DFS）都是图和树这种数据结构的遍历算法。两者各有特点和适用场景。

一、广度优先搜索算法（BFS）

1. 算法思想  
广度优先搜索的核心思想，就是从起点开始，先访问完所有距离当前节点为1的节点，再访问完所有距离当前节点为2的节点，以此类推。即“逐层”拓展，先近后远，一层一层地搜索。

2. 实现方法  
BFS通常使用队列（Queue）结构来实现，采用先进先出（FIFO）规则。具体过程为：  
（1）从起点出发，把起点加入队列，标记为已访问；  
（2）从队列中取出第一个节点，访问其所有未被访问的邻接节点，并依次加入队列；  
（3）重复第（2）步，直到队列为空结束。

3. 应用场景  
广度优先搜索主要应用在以下场景：  
（1）寻找图或树中的最短路径（尤其面对无权图，BFS天生擅长求最短路径）；  
（2）分层遍历，如“一层层”地打印二叉树；  
（3）网络爬虫的网页层次爬取；  
（4）在图像处理或地图导航的搜索中。

4. 算法复杂度  
时间复杂度为O(V+E)，其中V代表图中顶点（节点）数，E代表边数。  
空间复杂度为O(V)，最坏情况要保存所有节点。

5. 特点总结：
- BFS适用于解决最短路径问题；
- 遍历过程按层次逐渐展开；
- 通常借助辅助队列实现。

示例（伪代码）：
```
Algorithm BFS(graph, start):
    create a queue Q
    mark start as visited
    Q.enqueue(start)

    while Q is not empty:
        current = Q.dequeue() //取出队首元素
        process(current) //处理（访问）当前节点

        for each neighbor node n of current in graph:
            if n is not visited:
                mark n as visited
                Q.enqueue(n)
```

二、深度优先搜索算法（DFS）

1. 算法思想  
深度优先算法沿着图的某一节点开始，一路深度向下搜索到底，然后回退至上一个节点，继续向另一条路搜索，直至所有节点都已被访问。这个过程类似于迷宫探索中的“一条路走到底，碰壁后回头再尝试其他路径”的方式。

2. 实现方法  
DFS一般使用递归或栈（Stack）结构来实现，遵循后进先出（LIFO）规则。具体过程为：  
（1）选择一个起点，标记为访问过；  
（2）对于当前访问节点，找出它的未访问过的邻接节点，执行深度优先搜索（递归）；  
（3）如果当前路径无法继续前进，则回溯至前面的节点，再进行其他路径的搜索。  
如此往复，直至遍历完整个图。

3. 应用场景  
深度优先搜索主要应用于：  
（1）图中的连通性检查；  
（2）检测环路；  
（3）拓扑排序（Topological Sort）的实现；  
（4）寻路问题（寻找一条路径）；  
（5）迷宫问题的解决；  
（6）图中求连通分量、判定二分图等图论问题。

4. 算法复杂度  
与BFS一样，时间复杂度为O(V+E)。空间复杂度在最坏情况为O(V)，需要存储访问过节点，以及递归调用栈。

5. 特点总结：
- DFS先深入，再回溯；
- 一般通过递归实现（也可用栈来辅助）；
- 适合解决路径探索和连通性问题。

示例（伪代码，递归实现）：
```
Algorithm DFS(graph, node):
    mark node as visited
    process(node)

    for each neighbor node n of node in graph:
        if n is not visited:
            DFS(graph, n)  //递归调用
```

DFS算法也可以使用栈的迭代实现：
```
Algorithm DFS_iterative(graph, start):
    create a stack S
    S.push(start)

    while S is not empty:
        node = S.pop()

        if node not visited:
            mark node as visited
            process(node)

            for neighbor in neighbors(node):
                if neighbor not visited:
                    S.push(neighbor)
```

三、BFS与DFS的对比

| 对比维度 | 广度优先搜索（BFS） | 深度优先搜索（DFS） |
|----------|--------------------|--------------------|
| 搜索行为 | 一层一层地搜索，逐层推进 | 往深处搜索到底再逐层回溯 |
| 时间复杂度 | O(V+E) | O(V+E) |
| 空间复杂度 | 一般较高，需存储队列 最坏O(V) | 一般较低，取决递归栈深度 最坏O(V) |
| 最短路径计算 | 可直接找到无权图的最短路径 | 不保证找到最短路径，需要遍历所有可能路径 |
| 数据结构 | Queue（队列） | Stack或递归调用栈 |
| 适用场景 | 查询最短路径、分层问题 | 拓扑排序、路径存在性、环检测 |

总结：

- BFS与DFS为图论中的基本遍历算法，广泛应用于各种算法问题中。
- BFS强调一层层推进，适用于求最短路径或层次分析。
- DFS强调把一条路径搜索到底，再换路径搜索，适用于查找路径、拓扑排序、检测环路等。
- 两者各有优缺点，应结合实际应用场景选择使用。
</details>
