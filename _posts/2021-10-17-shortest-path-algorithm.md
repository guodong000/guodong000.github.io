---
layout: post
title: 最短路径算法
mathjax: true
---

最短路径问题是图论研究中一个经典问题，旨在寻找图中两结点之间的最短路径。

最短路径问题分为以下几类：
* **确定起点的最短路径问题**：又称单源最短路问题，非负边权时适用 Dijkstra 算法，负边权时适用 Bellman-ford 或 SPFA 算法。
* **确定终点的最短路径问题**：在无向图中与确定起点的问题完全等同，在有向图中反转路径方向的确定起点的问题。
* **确定两点间的最短路径问题**：
* **全局最短路径问题**：又称多源最短路问题，求图中所有最短路径，适用 Floyd-Warshall 算法。

# 广度优先搜索

Breadth-First Search (BFS) 是一种图搜索算法，从根节点开始沿着树的宽度遍历树的节点。一般采用 open-close 表，要被检验的节点放入 open 队列中，检验过的节点放入 close 容器中，close 容器主要用于防止有环图中的节点被重复访问的问题。

**算法描述**：
1. 将根节点放入 open 队列中
2. 从队列取出首个节点，检查是否为搜索目标。
    * 如果命中，则结束搜索返回结果
    * 否则将其所有未检测过的直接子节点加入 open 队列
    * 将该节点放入 close 容器
3. 若队列为空，表示未搜索到指定目标
4. 重复步骤 2

**代码描述**：
```python
from collections import namedtuple

Node = namedtuple('Node', ['value', 'childs'])

_open = list()
_closed = set()
def bfs(graph, query):
    _open.append(graph)
    while len(_open) > 0:
        node = _open.pop(0)
        print(node.value, end=' ')
        if node.value == query:
            print('Found!')
            break
        for n in node.childs:
            if n.value in _closed:
                continue
            _open.append(n)
        _closed.add(node.value)
    
# Test Code
graph = Node(0, childs=[])
node1 = Node(1, childs=[])
node2 = Node(2, childs=[])
node3 = Node(3, childs=[])
node4 = Node(4, childs=[])
node5 = Node(5, childs=[])
graph.childs.extend([node1, node2])
node1.childs.extend([node3, node4])
node2.childs.extend([node5])
bfs(graph, 10)
```

# Dijkstra 算法

Dijkstra 原始版本仅适用于寻找图（非负边权的有向和无向图）的两个顶点之间的最短路径，后来更常见的变体适用于单源最短路径问题。

对于没有任何优化的算法复杂度为 \\(O(\|V\|^2+\|E\|)\\)，其中 \\(\|V\|\\)为顶点数 \\(\|E\|\\)为边数。

**算法描述**

假设要从顶点 s 寻找到其他任一顶点的最短路径。  
d[] 在算法结束时保存 s 到任一顶点的最短距离，路径不存在则值为无穷大。
维护两个顶点集合 S 和 Q，S 保存所有已知世纪最短路径的顶点，Q 保存其他所有顶点。S 初始状态为空。

1. 初始化操作
    * d[] 中除起始点外其他顶点对应值设置为无穷大，`d[s] = 0`
    * S 设置为空
    * Q 保存图中所有顶点
2. 如果 Q 为空，则算法结束，d[] 中为起点 s 到任一顶点的最短路径长度
3. 从 Q 中取出（删除）拥有最小 d[u] 值的顶点 u，并将其放入 S
4. 如果仅寻找 s 和 t 之间的最短路径，可在此处判断 u 是否为 t，如果相同，则算法结束，d[u] 为 s 到 t 的最短路径
4. 遍历所有顶点，对每个顶点 v 进行松弛操作，即：如果存在边 u->v ，则 `d[v] = min(d[v], w(u, v))` w(u, v) 为 u 到 v 的边的权重
5. 重复步骤 2

**代码描述**

```python
import math, heapq

# 邻接矩阵表示
graph = [
    [0, math.inf, math.inf, math.inf, math.inf, math.inf],
    [math.inf, 0, math.inf, math.inf, math.inf, math.inf],
    [math.inf, math.inf, 0, math.inf, math.inf, math.inf],
    [math.inf, math.inf, math.inf, 0, math.inf, math.inf],
    [math.inf, math.inf, math.inf, math.inf, 0, math.inf],
    [math.inf, math.inf, math.inf, math.inf, math.inf, 0],
]

# 为无向图添加一条边，也可将图看作有向图，算法可以无缝适应
def add_edge(s, t, w):
    graph[s][t] = w
    graph[t][s] = w

def dijkstra(graph, s):
    dist = [math.inf] * len(graph)
    dist[s] = 0

    visited = [False] * len(graph)
    h = [(0, s)]    # 此处采用堆优化
    heapq.heapify(h)
    
    while len(h) > 0:
        u = heapq.heappop(h)[1]
        if visited[u]:
            continue
        visited[u] = True
        for v in range(len(graph)):
            if math.isinf(graph[u][v]):
                continue
            if dist[v] > dist[u] + graph[u][v]:
                dist[v] = dist[u] + graph[u][v]
                heapq.heappush(h, (dist[v], v))

    print('Vertex distance from source:', dist)

# Test Code
add_edge(0, 1, 1)
add_edge(0, 2, 3)
add_edge(1, 2, 1)
add_edge(1, 3, 1)
add_edge(1, 4, 3)
add_edge(2, 4, 2)
add_edge(3, 4, 1)
add_edge(4, 5, 1)
dijkstra(graph, 0)
```

# Bellman-Ford 算法

相较于 Dijkstra 算法，该算法可以适用于含有负边权的图。其算法复杂度为 \\(O(\|V\|\|E\|)\\)。

如果一个环路上的总权重为负值，则称其为**负权环**。负权环可以无限降低经过路径的总权重，这意味着其会导致算法找不到正确答案。

实际操作中，算法可以在某次循环不再进行松弛（dist数组不再更新）时直接退出循环。

**代码描述**
```python
import math

def bellman(vertices, edges, src):
    # 初始化
    dist = [math.inf] * len(vertices)
    dist[src] = 0
    
    # 算法结束后保存起点到每个点的最短路径
    predecessor = [None] * len(vertices)

    # 对每条边重复操作，运行 |V|-1 次
    for _ in range(len(vertices) - 1):
        # 每次循环实际就是对相邻节点的访问
        for (u, v, w) in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                predecessor[v] = u

    # 检查是否存在负权环
    for (u, v, w) in edges:
        # 因为负权环可以无限制降低总花费
        # 如果算法完成后发现仍可以降低花销，则一定存在负权环
        if dist[u] + w < dist[v]:
            print('Negative weight cycle detected!')
            return
    
    print(predecessor)
    print(dist)

# Test Code
vertices = [0, 1, 2, 3, 4, 5]
edges = [
    [0, 1, 1], [0, 2, 3], [1, 2, 1], [1, 3, 1],
    [1, 4, 3], [2, 4, 2], [3, 4, 1], [4, 5, 1],
]
bellman(vertices, edges, 0)
```

# To be continue...