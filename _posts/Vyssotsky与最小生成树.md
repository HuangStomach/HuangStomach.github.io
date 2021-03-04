---
title: Vyssotsky与最小生成树
date: 2018-12-29 21:43:13
tags: 
- Java
- Algorithm
categories:
- Algorithm
---

> 开发一种不断使用环的性质的算法来计算最小生成树

## 环的性质

当构建出一副无向图的最小生成树时可以发现，在无向图中任意将一条边加入最小生成树，都会构建出一个环；因为生成树(不一定最小)实际上会达到图中的所有点，再加入任何一条边都相当于多连接了一遍两个顶点，必然会产生环。

利用这个性质，可一每次都从新产生的环中删除权重最大的边，来不停构造最小生成树。

## Vyssotsky

首先可以先随便拟定一个生成树（不一定最小）:

``` java
edges = new Queue<Edge>();
mst = new EdgeWeightedGraph(G.V());
boolean[] marked = new boolean[G.V()];
for (Edge e: G.edges()) {
    int v = e.either();
    int w = e.other(v);
    if (marked[v] && marked[w]) {
        edges.enqueue(e);
        continue;
    }
    
    marked[v] = true;
    marked[w] = true;
    mst.addEdge(e);
}
```

使用`marked`来记录点是否已经被连接过，如果两个顶点均被连接过，则将该条边放入备选`edges`中，否则加入新图，也就是生成树中。

然后不停地将备选边中的边加入到生成树中：

``` java
while (edges.size() > 0) {
    Edge e = edges.dequeue();
    mst.addEdge(e);
}
```

此时一定会产生环，使用深度优先算法对环进行检测，从而找到权重最大的边：

``` java
public boolean dfs(int start, int end, int skip) {
    for (Edge e: mst.adj(start)) {
        if (set.contains(e.weight())) continue;

        int v = e.other(start);
        if (v == skip) continue;

        if (v == end || dfs(v, end, start)) {
            max = Math.max(max, e.weight());
            return true;
        }
    }
    return false;
}
```

其中寻找的时候要把前一条边的起点忽略掉，这里就没没有再使用`marked`数组了。

## 删除边

然后就是要将权重最大的边删除掉，这里给的加权无向图`EdgeWeightedGraph`中没有提供删除边的API，也没有说需要自行扩展，这里就利用了**所有权重均不相同**的性质，将最大权重加入到集合中:

``` java
while (edges.size() > 0) {
    Edge e = edges.dequeue();
    max = e.weight();
    mst.addEdge(e);
    
    int v = e.either();
    dfs(v, e.other(v), e.other(v));
    set.add(max);
}
```

这样虽然是全部边都加入了新图中，但是在我们获取最小生成树的时候，如果遇到了边的权重在集合中，则进行跳过：

``` java
public Iterable<Edge> edges() {
    Queue<Edge> queue = new Queue<Edge>();
    for (Edge e: mst.edges()) {
        if (set.contains(e.weight())) continue;
        queue.enqueue(e);
    }
    return queue;
}
```

写了下测试，可以通过。整体代码：

``` java
import java.util.HashSet;

import edu.princeton.cs.algs4.*;

class VyssotskyMST {
    private Queue<Edge> edges;
    private EdgeWeightedGraph mst;
    private HashSet set;
    private double max;
    
    public VyssotskyMST(EdgeWeightedGraph G) {
        edges = new Queue<Edge>();
        set = new HashSet<Double>();
        mst = new EdgeWeightedGraph(G.V());
        boolean[] marked = new boolean[G.V()];
        for (Edge e: G.edges()) {
            int v = e.either();
            int w = e.other(v);
            if (marked[v] && marked[w]) {
                edges.enqueue(e);
                continue;
            }
            
            marked[v] = true;
            marked[w] = true;
            mst.addEdge(e);
        }

        while (edges.size() > 0) {
            Edge e = edges.dequeue();
            max = e.weight();
            mst.addEdge(e);
            
            int v = e.either();
            dfs(v, e.other(v), e.other(v));
            set.add(max);
        }
    }

    public boolean dfs(int start, int end, int skip) {
        for (Edge e: mst.adj(start)) {
            if (set.contains(e.weight())) continue;

            int v = e.other(start);
            if (v == skip) continue;

            if (v == end || dfs(v, end, start)) {
                max = Math.max(max, e.weight());
                return true;
            }
        }
        return false;
    }

    public Iterable<Edge> edges() {
        Queue<Edge> queue = new Queue<Edge>();
        for (Edge e: mst.edges()) {
            if (set.contains(e.weight())) continue;
            queue.enqueue(e);
        }
        return queue;
    }

    public static void main(String[] args) {
        In in = new In(args[0]);
        EdgeWeightedGraph G = new EdgeWeightedGraph(in);
        VyssotskyMST mst = new VyssotskyMST(G);
        for (Edge e: mst.edges()) {
            System.out.println(e);
        }
    }
}
```

自己描述一边算法也是对思路的一个整理，现在发现其实在第一次遍历中就可以开始环的判断了，没有必要收集起来再做遍历。

目前自己都觉得性能不太好，之后有机会还是会进行优化。
