# 算法 加权有向图


## 加权有向图

找到从一个顶点到达另一个顶点的成本最小的路径。

在一幅加权有向图中，从顶点s 到顶点t 的最短路径是所有从s 到t 的路径中的权重最小者。

![image-20210417101304668](/algorithm/image-20210417101304668.png)

最短路径性质

&#43; 路径是有向的，
&#43; 权重不一定等价于距离
&#43; 并不是所有顶点都是可达的
&#43; 负权重会使问题更复杂
&#43; 最短路径不一定是唯一的。
&#43; 可能存在平行边和自环。

我们的重点是单点最短路径问题，其中给出了起点s , 计算的结果是一棵最短路径树(SPT), 它包含了顶点s 到所有可达的顶点的最短路径。如图所示。

![image-20210417104047826](/algorithm/image-20210417104047826.png)

定义：给定一幅加权有向图和一个顶点s , 以s 为起点的一棵最短路径树是图的一幅子图，它包含s和从s 可达的所有顶点。这棵有向树的根结点为s,树的每条路径都是有向图中的一条最短路径。

![image-20210417104207162](/algorithm/image-20210417104207162.png)

### 加权有向图的数据结构

![image-20210417151517836](/algorithm/image-20210417151517836.png)

![image-20210417152726920](/algorithm/image-20210417152726920.png)![image-20210417160053756](/algorithm/image-20210417160053756.png)

edgeTo[v]:最短路径树中的边，连接v和它父节点的边，

distTo[v] :到达起点的距离, 为从s 到v的已知最短路径的长度。该路径上的所有顶点均在树中且路径上的最后一条边为edgeTo[v]



```c#
using SuddenGale.AlgorithmEngine.Structure;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;

namespace SuddenGale.AlgorithmEngine.Graph
{

    public class DirectedEdge:IComparable&lt;DirectedEdge&gt;
    {
        private readonly int v;//边的起点
        private readonly int w;//边的终点
        private readonly double weight;//边的权重

        public DirectedEdge(int v, int w, double weight)
        {
            if (v &lt; 0) throw new ArgumentException(&#34;Vertex names must be nonnegative integers&#34;);
            if (w &lt; 0) throw new ArgumentException(&#34;Vertex names must be nonnegative integers&#34;);
            if (Double.IsNaN(weight)) throw new ArgumentException(&#34;Weight is NaN&#34;);
            this.v = v;
            this.w = w;
            this.weight = weight;
        }

        public int From()
        {
            return v;
        }

        public int To()
        {
            return w;
        }

        public double Weight()
        {
            return weight;
        }

        public override string ToString()
        {
            return  $&#34;{v}-&gt;{w} {weight:F2}&#34;;
        }

        public int CompareTo(DirectedEdge other)
        {
            return this.weight.CompareTo(other.Weight());
        }
    }


    public class EdgeWeightedDigraph
    {
        private static readonly String NEWLINE = System.Environment.NewLine;

        private readonly int V;                // 顶点总数
        private int E;                      // 边的总数
        private LinkedBagNet&lt;DirectedEdge&gt;[] adj;    // 顶点index的临接表
        private int[] indegree;             // vertex 的度

        
        public EdgeWeightedDigraph(int V)
        {
            if (V &lt; 0) throw new ArgumentException(&#34;Number of vertices in a Digraph must be nonnegative&#34;);
            this.V = V;
            this.E = 0;
            this.indegree = new int[V];
            adj = new LinkedBagNet&lt;DirectedEdge&gt;[V];
            for (int v = 0; v &lt; V; v&#43;&#43;)
                adj[v] = new LinkedBagNet&lt;DirectedEdge&gt;();
        }

        public EdgeWeightedDigraph(int V, int E):this(V)
        {
            if (E &lt; 0) throw new ArgumentException(&#34;Number of edges in a Digraph must be nonnegative&#34;);
            Random random = new Random();
            for (int i = 0; i &lt; E; i&#43;&#43;)
            {
                int v = random.Next(V);
                int w = random.Next(V);
                double weight = Math.Round(100 * random.NextDouble()) / 100.0;
                DirectedEdge e = new DirectedEdge(v, w, weight);
                AddEdge(e);
            }
        }

        
        public EdgeWeightedDigraph(StreamReader stream)
        {
            if (stream == null) throw new ArgumentNullException(&#34;argument is null&#34;);
            try
            {
                this.V = int.Parse(stream.ReadLine());
                if (V &lt; 0) throw new ArgumentException(&#34;number of vertices in a Digraph must be nonnegative&#34;);
                indegree = new int[V];
                adj = new LinkedBagNet&lt;DirectedEdge&gt;[V];
                for (int v = 0; v &lt; V; v&#43;&#43;)
                {
                    adj[v] = new LinkedBagNet&lt;DirectedEdge&gt;();
                }

                int E = int.Parse(stream.ReadLine());
                if (E &lt; 0) throw new ArgumentException(&#34;Number of edges must be nonnegative&#34;);
                for (int i = 0; i &lt; E; i&#43;&#43;)
                {
                    string line = stream.ReadLine().Trim();
                    if (!string.IsNullOrEmpty(line) &amp;&amp; line.Split(&#39; &#39;).Count() == 3)
                    {
                        int v = int.Parse(line.Split(&#39; &#39;)[0]);
                        int w = int.Parse(line.Split(&#39; &#39;)[1]);
                        double weight = double.Parse(line.Split(&#39; &#39;)[2]);
                        AddEdge(new DirectedEdge(v, w, weight));
                    }
                    
                }
            }
            catch (Exception e)
            {
                throw new ArgumentException(&#34;invalid input format in EdgeWeightedDigraph constructor&#34;, e);
            }
        }

        public EdgeWeightedDigraph(EdgeWeightedDigraph G): this(G.V)
        {
            this.E = G.Edge();
            for (int v = 0; v &lt; G.V; v&#43;&#43;)
                this.indegree[v] = G.Indegree(v);
            for (int v = 0; v &lt; G.V; v&#43;&#43;)
            {
                // reverse so that adjacency list is in same order as original
                Stack&lt;DirectedEdge&gt; reverse = new Stack&lt;DirectedEdge&gt;();
                foreach (DirectedEdge e in G.adj[v])
                {
                    reverse.Push(e);
                }
                foreach (DirectedEdge e in reverse)
                {
                    adj[v].Add(e);
                }
            }
        }

        public int Vertic()
        {
            return V;
        }

        
        public int Edge()
        {
            return E;
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void validateVertex(int v)
        {
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

        public void AddEdge(DirectedEdge e)
        {
            int v = e.From();
            int w = e.To();
            validateVertex(v);
            validateVertex(w);
            adj[v].Add(e);
            indegree[w]&#43;&#43;;
            E&#43;&#43;;
        }


        
        public IEnumerable&lt;DirectedEdge&gt; Adj(int v)
        {
            validateVertex(v);
            return adj[v];
        }

        public int Outdegree(int v)
        {
            validateVertex(v);
            return adj[v].Length;
        }

        
        public int Indegree(int v)
        {
            validateVertex(v);
            return indegree[v];
        }

        
        public IEnumerable&lt;DirectedEdge&gt; Edges()
        {
            LinkedBagNet&lt;DirectedEdge&gt; list = new LinkedBagNet&lt;DirectedEdge&gt;();
            for (int v = 0; v &lt; V; v&#43;&#43;)
            {
                foreach (DirectedEdge e in Adj(v))
                {
                    list.Add(e);
                }
            }
            return list;
        }


        public override string ToString()
        {
            StringBuilder s = new StringBuilder();
            s.Append(V &#43; &#34; &#34; &#43; E &#43; NEWLINE);
            for (int v = 0; v &lt; V; v&#43;&#43;)
            {
                s.Append(v &#43; &#34;: &#34;);
                foreach (DirectedEdge e in adj[v])
                {
                    s.Append(e &#43; &#34;  &#34;);
                }
                s.Append(NEWLINE);
            }
            return s.ToString();
        }

    }
}

```





边松弛

松弛( relaxation ):放松边V -&gt; W 意味着检查从s 到w 的最短路径是否是先从s 到v , 然后再由v 到w 。如果是， 则根据这个情况更新数据结构的内容。由v 到达w 的最短路径是di stTo[v] 与e.weight() 之和一一如果这个值不小于distTo[w] , 称这条边失效了并将它忽略； 如果这个值更小，就更新数据。

一种情况是边失效（左边的例子），不更新任何数据；另一种情况是v -&gt; w 就是到达w 的最短路径（右边的例子），这将会更新edgeTo[w]和distTo[w] (这可能会使另一些边失效，但也可能产生一些新的有效边）

![image-20210417232224860](/algorithm/image-20210417232224860.png)

![image-20210417233427069](/algorithm/image-20210417233427069.png)

![image-20210417233449519](/algorithm/image-20210417233449519.png)

### Dijkstra算法

寻找加权无向图中的最小生成树的Prim 算法：构造最小生成树的每一步都向这棵树中添加一条新的边。Dijkstra 算法采用了类似的方法来计算最短路径树。首先将distTo[s]初始化为0, distTo[] 中的其他元素初始化为正无穷。==然后将distTo [] 最小的非树顶点放松并加入树中，如此这般，直到所有的顶点都在树中或者所有的非树顶点的distTo[]值均为无穷大。==

![image-20210417234037942](/algorithm/image-20210417234037942.png)

![image-20210417234309144](/algorithm/image-20210417234309144.png)

Prim 算法每次添加的都是离树最近的非树顶点， Dijkstra算法每次添加的都是离起点最近的非树顶点。

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using SuddenGale.AlgorithmEngine.Structure;

namespace SuddenGale.AlgorithmEngine.Graph
{
    public class DijkstraSP
    {
        private double[] distTo;          // distTo[v] = distance  of shortest s-&gt;v path，最小树的长度
        private DirectedEdge[] edgeTo;    // edgeTo[v] = last edge on shortest s-&gt;v path,最小树中的边
        private IndexMinPQ&lt;Double&gt; pq;    //顶点优先队列，保存需要放松的下一个顶点，所有添加进最小树的边的To，每次放松最小值

        public DijkstraSP(EdgeWeightedDigraph G, int s)
        {
            foreach (DirectedEdge e in G.Edges())
            {
                if (e.Weight() &lt; 0)
                    throw new ArgumentException(&#34;edge &#34; &#43; e &#43; &#34; has negative weight&#34;);
            }

            distTo = new double[G.Vertic()];
            edgeTo = new DirectedEdge[G.Vertic()];

            ValidateVertex(s);

            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                distTo[v] = Double.PositiveInfinity;
            distTo[s] = 0.0;

            // relax vertices in order of distance from s
            pq = new IndexMinPQ&lt;Double&gt;(G.Vertic());
            pq.Insert(s, distTo[s]);
            while (!pq.IsEmpty())
            {
                int v = pq.DelMin();
                foreach (DirectedEdge e in G.Adj(v))
                    Relax(e);
            }

            // check optimality conditions
            if(!Check(G, s))
                throw new Exception(&#34;Check return false&#34;);
        }

        // relax edge e and update pq if changed
        private void Relax(DirectedEdge e)
        {
            int v = e.From(), w = e.To();
            if (distTo[w] &gt; distTo[v] &#43; e.Weight())
            {
                distTo[w] = distTo[v] &#43; e.Weight();//更新最小树的长度
                edgeTo[w] = e;//更新最小树
                if (pq.Contains(w)) pq.DecreaseKey(w, distTo[w]);
                else pq.Insert(w, distTo[w]);
            }
        }

       
        public double DistTo(int v)
        {
           ValidateVertex(v);
            return distTo[v];
        }

        
        public bool HasPathTo(int v)
        {
           ValidateVertex(v);
            return distTo[v] &lt; Double.PositiveInfinity;
        }

        
        public IEnumerable&lt;DirectedEdge&gt; PathTo(int v)
        {
            ValidateVertex(v);
            if (!HasPathTo(v)) return null;
            Stack&lt;DirectedEdge&gt; path = new Stack&lt;DirectedEdge&gt;();
            for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.From()])
            {
                path.Push(e);
            }
            return path;
        }


        private bool Check(EdgeWeightedDigraph G, int s)
        {

            // check that edge weights are nonnegative
            foreach (DirectedEdge e in G.Edges())
            {
                if (e.Weight() &lt; 0)
                {
                    Console.WriteLine(&#34;negative edge weight detected&#34;);
                    return false;
                }
            }

            // check that distTo[v] and edgeTo[v] are consistent
            if (distTo[s] != 0.0 || edgeTo[s] != null)
            {
                Console.WriteLine(&#34;distTo[s] and edgeTo[s] inconsistent&#34;);
                return false;
            }
            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
            {
                if (v == s) continue;
                if (edgeTo[v] == null &amp;&amp; distTo[v] != Double.PositiveInfinity)
                {
                   Console.WriteLine(&#34;distTo[] and edgeTo[] inconsistent&#34;);
                    return false;
                }
            }

            // check that all edges e = v-&gt;w satisfy distTo[w] &lt;= distTo[v] &#43; e.weight()
            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
            {
                foreach (DirectedEdge e in G.Adj(v))
                {
                    int w = e.To();
                    if (distTo[v] &#43; e.Weight() &lt; distTo[w])
                    {
                        Console.WriteLine(&#34;edge &#34; &#43; e &#43; &#34; not relaxed&#34;);
                        return false;
                    }
                }
            }

            // check that all edges e = v-&gt;w on SPT satisfy distTo[w] == distTo[v] &#43; e.weight()
            for (int w = 0; w &lt; G.Vertic(); w&#43;&#43;)
            {
                if (edgeTo[w] == null) continue;
                DirectedEdge e = edgeTo[w];
                int v = e.From();
                if (w != e.To()) return false;
                if (distTo[v] &#43; e.Weight() != distTo[w])
                { 
                    Console.WriteLine(&#34;edge &#34; &#43; e &#43; &#34; on shortest path not tight&#34;);
                    return false;
                }
            }
            return true;
        }

        
        private void ValidateVertex(int v)
        {
            int V = distTo.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
    }
}

```

测试

```c#
        [TestMethod()]
        public void DijkstraSPTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyEWG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.EdgeWeightedDigraph G = new AlgorithmEngine.Graph.EdgeWeightedDigraph(stream);
                int s = 0;
                DijkstraSP sp = new DijkstraSP(G, s);
                //打印最短路径
                for (int t = 0; t &lt; G.Vertic(); t&#43;&#43;)
                {
                    if (sp.HasPathTo(t))
                    {
                        Console.Write($&#34;{s} to {t} {sp.DistTo(t):f2}    &#34;);
                        foreach (DirectedEdge e in sp.PathTo(t))
                        {
                            Console.Write(e &#43; &#34; &#34;);
                        }
                        Console.WriteLine();
                    }
                    else
                    {
                        Console.WriteLine($&#34;{s} to {t}         no path&#34;);
                    }
                }
            }
            // 0 to 0 0.00    
            // 0 to 1         no path
            // 0 to 2 0.26    0-&gt;2 0.26 
            // 0 to 3 0.43    0-&gt;2 0.26 2-&gt;3 0.17 
            // 0 to 4 0.38    0-&gt;4 0.38 
            // 0 to 5 0.73    0-&gt;4 0.38 4-&gt;5 0.35 
            // 0 to 6 0.95    0-&gt;2 0.26 2-&gt;3 0.17 3-&gt;6 0.52 
            // 0 to 7 0.16    0-&gt;7 0.16 
        }
```

### 任意两点的最短路径



```c#
    public class DijkstraAllPairsSP
    {
        private DijkstraSP[] all;

        public DijkstraAllPairsSP(EdgeWeightedDigraph G)
        {
            all = new DijkstraSP[G.Vertic()];
            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                all[v] = new DijkstraSP(G, v);
        }

        
        public IEnumerable&lt;DirectedEdge&gt; Path(int s, int t)
        {
            ValidateVertex(s);
            ValidateVertex(t);
            return all[s].PathTo(t);
        }

        public bool HasPath(int s, int t)
        {
            ValidateVertex(s);
            ValidateVertex(t);
            return Dist(s, t) &lt; Double.PositiveInfinity;
        }

        public double Dist(int s, int t)
        {
            ValidateVertex(s);
            ValidateVertex(t);
            return all[s].DistTo(t);
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void ValidateVertex(int v)
        {
            int V = all.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```

测试

```c#
        [TestMethod()]
        public void DijkstraAllPairsSPTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyEWD.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.EdgeWeightedDigraph G = new AlgorithmEngine.Graph.EdgeWeightedDigraph(stream);
                DijkstraAllPairsSP spt = new DijkstraAllPairsSP(G);
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                { 
                    Console.WriteLine($&#34;{v}&#34;);
                }
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    Console.Write($&#34;{v}: &#34;);
                    for (int w = 0; w &lt; G.Vertic(); w&#43;&#43;)
                    {
                        if (spt.HasPath(v, w)) Console.Write($&#34;{spt.Dist(v, w)} &#34;);
                        else Console.Write(&#34;  Inf &#34;);
                    }
                    Console.WriteLine();
                }
                Console.WriteLine();

                // print all-pairs shortest paths
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    for (int w = 0; w &lt; G.Vertic(); w&#43;&#43;)
                    {
                        if (spt.HasPath(v, w))
                        {
                            Console.Write($&#34;{v} to {w} {spt.Dist(v, w)}  &#34;);
                            foreach (DirectedEdge e in spt.Path(v, w))
                                Console.Write(e &#43; &#34;  &#34;);
                            Console.WriteLine();
                        }
                        else
                        {
                            Console.WriteLine($&#34;{v} to {w} no path&#34;, v, w);
                        }
                    }
                }


            }
        }
    }
```

### 无环加权有向图的最短路径

该算法的特点

1. 能够在线性事件内解决单点的最短路径问题
2. 能够处理负权重的边
3. 能够解决相关的问题。例如找到最长的路径。

首先，将distTo[s] 初始化为0, 其他distTo[] 元素初始化为无穷大，然后一个一个地按照==拓扑顺序==放松所有顶点。

按照拓扑顺序放松顶点，就能在和E&#43;V 成正比的时间内解决无环加权有向图的单点最短路径问题。

![image-20210418110558441](/algorithm/image-20210418110558441.png)

![image-20210418110616119](/algorithm/image-20210418110616119.png)

在拓扑排序后，构造函数会扫描整幅图并将每条边放松一次。==在已知加权图是无环的情况下，它是找出最短路径的最好方法。==

```c#
using System;
using System.Collections.Generic;

namespace SuddenGale.AlgorithmEngine.Graph
{
    public class AcyclicSP
    {
        private double[] distTo;         // distTo[v] = distance  of shortest s-&gt;v path
        private DirectedEdge[] edgeTo;   // edgeTo[v] = last edge on shortest s-&gt;v path


        public AcyclicSP(EdgeWeightedDigraph G, int s)
        {
            distTo = new double[G.Vertic()];
            edgeTo = new DirectedEdge[G.Vertic()];

            ValidateVertex(s);

            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                distTo[v] = Double.PositiveInfinity;
            distTo[s] = 0.0;

            // 拓扑排序
            Topological topological = new Topological(G);
            if (!topological.HasOrder())
                throw new ArgumentException(&#34;Digraph is not acyclic.&#34;);
            foreach (int v in topological.Order())
            {
                foreach (DirectedEdge e in G.Adj(v))
                    Relax(e);
            }
        }

        // relax edge e
        private void Relax(DirectedEdge e)
        {
            int v = e.From(), w = e.To();
            if (distTo[w] &gt; distTo[v] &#43; e.Weight())
            {
                distTo[w] = distTo[v] &#43; e.Weight();
                edgeTo[w] = e;
            }
        }

        
        public double DistTo(int v)
        {
            ValidateVertex(v);
            return distTo[v];
        }

        
        public bool HasPathTo(int v)
        {
            ValidateVertex(v);
            return distTo[v] &lt; Double.PositiveInfinity;
        }

        public IEnumerable&lt;DirectedEdge&gt; PathTo(int v)
        {
            ValidateVertex(v);
            if (!HasPathTo(v)) return null;
            Stack&lt;DirectedEdge&gt; path = new Stack&lt;DirectedEdge&gt;();
            for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.From()])
            {
                path.Push(e);
            }
            return path;
        }

        
        private void ValidateVertex(int v)
        {
            int V = distTo.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
    }
}

```

测试

```c#
        public void AcyclicSPTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyEWDAG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.EdgeWeightedDigraph G = new AlgorithmEngine.Graph.EdgeWeightedDigraph(stream);
                int s = 5;
                AcyclicSP sp = new AcyclicSP(G, s);
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    if (sp.HasPathTo(v))
                    {
                        Console.Write($&#34;{s} to {v} ({sp.DistTo(v)})  &#34;);
                        foreach (DirectedEdge e in sp.PathTo(v))
                        {
                            Console.Write(e &#43; &#34;   &#34;);
                        }
                        Console.WriteLine();
                    }
                    else
                    {
                        Console.WriteLine($&#34;{s} to {v}         no path.&#34;);
                    }
                }

            }
            // 5 to 0 (0.73)  5-&gt;4 0.35   4-&gt;0 0.38   
            // 5 to 1 (0.32)  5-&gt;1 0.32   
            // 5 to 2 (0.62)  5-&gt;7 0.28   7-&gt;2 0.34   
            // 5 to 3 (0.61)  5-&gt;1 0.32   1-&gt;3 0.29   
            // 5 to 4 (0.35)  5-&gt;4 0.35   
            // 5 to 5 (0)  
            // 5 to 6 (1.13)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   
            // 5 to 7 (0.28)  5-&gt;7 0.28   
        }
```

基于拓扑排序的方法比Dijkstra 算法快的倍数与Dijkstra 算法中所有优先队列操作的总成本成正比。

#### 最长路径

![image-20210418143658116](/algorithm/image-20210418143658116.png)

![image-20210418143724404](/algorithm/image-20210418143724404.png)

```c#
using System;
using System.Collections.Generic;

namespace SuddenGale.AlgorithmEngine.Graph
{
    /// &lt;summary&gt;
    /// 无环权重有向图的最长路径
    /// &lt;/summary&gt;
    public class AcyclicLP
    {
        private double[] distTo;          // distTo[v] = distance  of longest s-&gt;v path
        private DirectedEdge[] edgeTo;    // edgeTo[v] = last edge on longest s-&gt;v path

        
        public AcyclicLP(EdgeWeightedDigraph G, int s)
        {
            distTo = new double[G.Vertic()];
            edgeTo = new DirectedEdge[G.Vertic()];

            ValidateVertex(s);

            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                distTo[v] = Double.NegativeInfinity;
            distTo[s] = 0.0;

            // relax vertices in topological order
            Topological topological = new Topological(G);
            if (!topological.HasOrder())
                throw new ArgumentException(&#34;Digraph is not acyclic.&#34;);
            foreach (int v in topological.Order())
            {
                foreach (DirectedEdge e in G.Adj(v))
                    Relax(e);
            }
        }

        // relax edge e, but update if you find a *longer* path
        private void Relax(DirectedEdge e)
        {
            int v = e.From(), w = e.To();
            if (distTo[w] &lt; distTo[v] &#43; e.Weight())
            {
                distTo[w] = distTo[v] &#43; e.Weight();
                edgeTo[w] = e;
            }
        }

       
        public double DistTo(int v)
        {
            ValidateVertex(v);
            return distTo[v];
        }

        public bool HasPathTo(int v)
        {
            ValidateVertex(v);
            return distTo[v] &gt; Double.NegativeInfinity;
        }

        public IEnumerable&lt;DirectedEdge&gt; PathTo(int v)
        {
            ValidateVertex(v);
            if (!HasPathTo(v)) return null;
            Stack&lt;DirectedEdge&gt; path = new Stack&lt;DirectedEdge&gt;();
            for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.From()])
            {
                path.Push(e);
            }
            return path;
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void ValidateVertex(int v)
        {
            int V = distTo.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException($&#34;vertex {v} is not between 0 and {V-1}&#34;);
        }

    }
}

```

测试

```c#
        public void AcyclicLPTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyEWDAG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.EdgeWeightedDigraph G = new AlgorithmEngine.Graph.EdgeWeightedDigraph(stream);
                int s = 5;
                AcyclicLP lp = new AcyclicLP(G, s);
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    if (lp.HasPathTo(v))
                    {
                        Console.Write($&#34;{s} to {v} ({lp.DistTo(v)})  &#34;);
                        foreach (DirectedEdge e in lp.PathTo(v))
                        {
                            Console.Write(e &#43; &#34;   &#34;);
                        }
                        Console.WriteLine();
                    }
                    else
                    {
                        Console.WriteLine($&#34;{s} to {v}         no path.&#34;);
                    }
                }

            }
            // 5 to 0 (2.44)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   6-&gt;4 0.93   4-&gt;0 0.38   
            // 5 to 1 (0.32)  5-&gt;1 0.32   
            // 5 to 2 (2.77)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   6-&gt;4 0.93   4-&gt;7 0.37   7-&gt;2 0.34   
            // 5 to 3 (0.61)  5-&gt;1 0.32   1-&gt;3 0.29   
            // 5 to 4 (2.06)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   6-&gt;4 0.93   
            // 5 to 5 (0)  
            // 5 to 6 (1.13)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   
            // 5 to 7 (2.43)  5-&gt;1 0.32   1-&gt;3 0.29   3-&gt;6 0.52   6-&gt;4 0.93   4-&gt;7 0.37  
        }
```

#### 平行任务调度

优先级限制下的并行任务调度。给定一组需要完成的特定任务，以及一组关于任务完成的先后次序的优先级限制。在满足限制条件的前提下应该如何在若干相同的处理器上（数量不限）安排任务并在最短的时间内完成所有任务？

假设任意可用的处理器都能在任务所需的时间内完成它，那么
我们的重点就是尽早安排每一个任务。

![image-20210418145516517](/algorithm/image-20210418145516517.png)

![image-20210418145704888](/algorithm/image-20210418145704888.png)

解决这个问题最主要的要解决关键路径

这个问题所需的最短时间为173.0 。这份调度方案满足了所有限制条件，没有其他调度方案能比它耗时更少，**因为任务必须按照0 ----&gt; 9 ----&gt; 6 ----&gt; 8 ----&gt; 2 的顺序完成**。这个顺序就是这个问题的==关键路径==。由优先级限制指定的每一列任务都代表了调度方案的一种可能的时间下限。如果将一系列任务的长度定义为完成所有任务的最早可能时间，那么最长的任务序列就是问题的关键路径，因为在这份任务序列中任何任务的启动延迟都会影响到整个项目的完成时间。

图4.4.16 显示的是示例任务所对应的图

图4.4 . 17 则显示的是最长路径的答案。

调度问题其实就是最长路径问题。

在图中每个任务都对应着三条边（从起点到起始顶点、从结束顶点到终点的权重为零的边，以及一条从起始顶点到结束顶点的边），每个优先级限制条件都对应着一条边。

![image-20210418150841027](/algorithm/image-20210418150841027.png)

#### 相对最后期限的并行任务调度

相对最后期限限制下的并行任务调度问题是一个加权有向图中的最短路径问题（可能存在环和负权重边） 。

![image-20210418162546258](/algorithm/image-20210418162546258.png)

![image-20210418162627964](/algorithm/image-20210418162627964.png)

### 负权重加权有向图的最短路径

![image-20210418162807165](/algorithm/image-20210418162807165.png)

也许最明显的改变就是当存在负权重的边时，权重较小的路径含有的边可能会比权重较大的路径更多。在只存在正权重的边时，我们的重点在于寻找近路；但当存在负权重的边时，我们可能会为了经过负权重的边而绕弯。这种效应使得我们要将查找“最短“路径的感觉转变为对算法本质的理解。

中间探究过程不说了，直接说结论

负权重环：加权有向图中的负权重环是一个总权重（环上的所有边的权重之和）为负的有向环。

==假设从s 到可达的某个顶点v 的路径上的某个顶点在一个负权重环上。在这种情况下，从s 到v 的最短路径是不可能存在的，因为可以用这个负权重环构造权重任意小的路径。==换句话说，**在负权重环存在的情况下，最短路径问题是没有意义的**

![image-20210418163719343](/algorithm/image-20210418163719343.png)

4，7，5，4，7，5不停兜圈子最短路径就会不停减少。所以最短路径失去了意义。

要求最短路径上的任意顶点都不存在负权重环意味着最短路径是简单的，而且与正权重边的图一样都能够得到此类顶点的最短路径树。

之前算法都是限制的。1：不允许负权重环的存在。2：不接受有向环。

#### 基于队列的Bellman-Ford 算法

**只有上一轮中的distTo[]值发生变化的顶点指出的边才能够改变其他distTo[] 元素的值**。为了记录这样的顶点，
我们使用了一条FIFO 队列首先将起点加入队列，然后按照以下步骤计算最短路径树。

![image-20210419083328469](/algorithm/image-20210419083328469.png)

Bellman-Ford 算法基于以下两种其他的数据结构：

&#43; 一条用来保存即将被放松的顶点的队列q ;

&#43; 一个由顶点索引的boolean 数组onQ[] , 用来指示顶点是否已经存在于队列中，
  顶点重复插入队列。

在某一轮中，改变了edgeTo[] 和distTo [] 的值的所有顶点都会在下一轮中处理。

#### 负权重的边

1. 放松边0→2 和0→4 并将顶点2 、4 加入队列。
2. 放松边2→7 并将顶点7 加入队列。放松边4→5 并将顶点5 加入队列。然后放松失效的边
   4→7 。
3. 放松边7→3 和5 →1 并将顶点3 和1 加入队列。放松失效的边5 → 4 和5 →7 。
4. 放松边3→6 并将顶点6 加入队列。放松失效的边1→3 。
5. 放松边6→4 并将顶点4 加入队列。这条负权重边使得到顶点4 的路径变短， 因此它的边需要被再次放松（它们在第二轮中已经被放松过） 。从起点到顶点5 和1 的距离巳经失效并会在下一轮中修正。
6. 放松边4 →5 并将顶点5 加入队列。放松失效的边4 →7 。
7. 放松边5→1 并将顶点1 加入队列。放松失效的边5 →4 和5→7 。
8. 放松无效的边1→3 。队列为空。

将所有边放松V 轮之后**当且仅当队列非空时有向图中才存在从起点可达的负权重环**。

如果是这样， edgeTo [] 数组所表示的子图中必然含有这个负权重环。因此，要实现
negativeCycl e() , 会根据edgeTo[] 中的边构造一幅加权有向图并在该图中检测环。

![image-20210419090322228](/algorithm/image-20210419090322228.png)

```c#
using System;
using System.Collections.Generic;
using System.Linq;

namespace SuddenGale.AlgorithmEngine.Graph
{
    public class BellmanFordSP
    {
        // for floating-point precision issues
        private static readonly double EPSILON = 1E-14;
        /// &lt;summary&gt;
        /// 从起点到某个顶点的路径长度,
        /// distTo[v] = distance  of shortest s-&gt;v path,
        /// &lt;/summary&gt;
        private double[] distTo;
        /// &lt;summary&gt;
        /// 从起点到某个顶点的最后一条边,
        /// edgeTo[v] = last edge on shortest s-&gt;v path
        /// &lt;/summary&gt;
        private DirectedEdge[] edgeTo;
        /// &lt;summary&gt;
        /// onQueue[v] = is v currently on the queue?
        /// 该顶点是否存在于队列中
        /// &lt;/summary&gt;
        private bool[] onQueue;
        /// &lt;summary&gt;
        /// 正在被放松的顶点，
        /// queue of vertices to relax
        /// &lt;/summary&gt;
        private Queue&lt;int&gt; queue;
        /// &lt;summary&gt;
        /// relax的调用次数,
        ///  number of calls to relax()，
        /// &lt;/summary&gt;
        private int cost;
        /// &lt;summary&gt;
        /// edgeTo[]中的是否有负权重环
        /// negative cycle (or null if no such cycle)
        /// &lt;/summary&gt;
        private IEnumerable&lt;DirectedEdge&gt; cycle;
        public BellmanFordSP(EdgeWeightedDigraph G, int s)
        {
            distTo = new double[G.Vertic()];
            edgeTo = new DirectedEdge[G.Vertic()];
            onQueue = new bool[G.Vertic()];
            for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                distTo[v] = Double.PositiveInfinity;
            distTo[s] = 0.0;
            // Bellman-Ford algorithm
            queue = new Queue&lt;int&gt;();
            queue.Enqueue(s);
            onQueue[s] = true;
            while (queue.Any() &amp;&amp; !HasNegativeCycle())
            {
                int v = queue.Dequeue();
                onQueue[v] = false;
                Relax(G, v);
            }
            if(!Check(G, s))
                throw new ArgumentException(&#34;check return false &#34;);
        }

        // relax vertex v and put other endpoints on queue if changed
        private void Relax(EdgeWeightedDigraph G, int v)
        {
            foreach (DirectedEdge e in G.Adj(v))
            {
                int w = e.To();
                if (distTo[w] &gt; distTo[v] &#43; e.Weight() &#43; EPSILON)
                {
                    distTo[w] = distTo[v] &#43; e.Weight();
                    edgeTo[w] = e;
                    if (!onQueue[w])//只有上一轮中发送变化的值才会改变，防止将顶点重复插入队列
                    {
                        queue.Enqueue(w);
                        onQueue[w] = true;
                    }
                }
                if (&#43;&#43;cost % G.Vertic() == 0)//负环会造成循环，导致放松的次数一直增加，最总会达到顶点数，这个时候在最小生成树上一定能检测出负环
                {
                    FindNegativeCycle();
                    if (HasNegativeCycle()) 
                        return;  // found a negative cycle
                }
            }
        }

        
        public bool HasNegativeCycle()
        {
            return cycle != null;
        }

        public IEnumerable&lt;DirectedEdge&gt; NegativeCycle()
        {
            return cycle;
        }

        // by finding a cycle in predecessor graph
        private void FindNegativeCycle()
        {
            int V = edgeTo.Length;
            EdgeWeightedDigraph spt = new EdgeWeightedDigraph(V);//在最短路径上肯定存在负环
            for (int v = 0; v &lt; V; v&#43;&#43;)
                if (edgeTo[v] != null)
                    spt.AddEdge(edgeTo[v]);

            EdgeWeightedDirectedCycle finder = new EdgeWeightedDirectedCycle(spt);//所以只在最短路径上寻找环，找到的环一定是负环
            cycle = finder.Cycle();
        }

        
        public double DistTo(int v)
        {
            ValidateVertex(v);
            if (HasNegativeCycle())
                throw new ArgumentException(&#34;Negative cost cycle exists&#34;);
            return distTo[v];
        }

       
        public bool HasPathTo(int v)
        {
            ValidateVertex(v);
            return distTo[v] &lt; Double.PositiveInfinity;
        }

        
        public IEnumerable&lt;DirectedEdge&gt; PathTo(int v)
        {
            ValidateVertex(v);
            if (HasNegativeCycle())
                throw new ArgumentException(&#34;Negative cost cycle exists&#34;);
            if (!HasPathTo(v)) return null;
            Stack&lt;DirectedEdge&gt; path = new Stack&lt;DirectedEdge&gt;();
            for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.From()])
            {
                path.Push(e);
            }
            return path;
        }

        // check optimality conditions: either 
        // (i) there exists a negative cycle reacheable from s
        //     or 
        // (ii)  for all edges e = v-&gt;w:            distTo[w] &lt;= distTo[v] &#43; e.weight()
        // (ii&#39;) for all edges e = v-&gt;w on the SPT: distTo[w] == distTo[v] &#43; e.weight()
        private bool Check(EdgeWeightedDigraph G, int s)
        {

            // has a negative cycle
            if (HasNegativeCycle())
            {
                double weight = 0.0;
                foreach (DirectedEdge e in NegativeCycle())
                {
                    weight &#43;= e.Weight();
                }
                if (weight &gt;= 0.0)
                {
                    Console.WriteLine(&#34;error: weight of negative cycle = &#34; &#43; weight);
                    return false;
                }
            }

            // no negative cycle reachable from source
            else
            {

                // check that distTo[v] and edgeTo[v] are consistent
                if (distTo[s] != 0.0 || edgeTo[s] != null)
                {
                   Console.WriteLine(&#34;distanceTo[s] and edgeTo[s] inconsistent&#34;);
                    return false;
                }
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    if (v == s) continue;
                    if (edgeTo[v] == null &amp;&amp; distTo[v] != Double.PositiveInfinity)
                    {
                        Console.WriteLine(&#34;distTo[] and edgeTo[] inconsistent&#34;);
                        return false;
                    }
                }

                // check that all edges e = v-&gt;w satisfy distTo[w] &lt;= distTo[v] &#43; e.weight()
                for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                {
                    foreach (DirectedEdge e in G.Adj(v))
                    {
                        int w = e.To();
                        if (distTo[v] &#43; e.Weight() &lt; distTo[w])
                        {
                            Console.WriteLine(&#34;edge &#34; &#43; e &#43; &#34; not relaxed&#34;);
                            return false;
                        }
                    }
                }

                // check that all edges e = v-&gt;w on SPT satisfy distTo[w] == distTo[v] &#43; e.weight()
                for (int w = 0; w &lt; G.Vertic(); w&#43;&#43;)
                {
                    if (edgeTo[w] == null) continue;
                    DirectedEdge e = edgeTo[w];
                    int v = e.From();
                    if (w != e.To()) return false;
                    if (distTo[v] &#43; e.Weight() != distTo[w])
                    {
                        Console.WriteLine(&#34;edge &#34; &#43; e &#43; &#34; on shortest path not tight&#34;);
                        return false;
                    }
                }
            }

            Console.WriteLine(&#34;Satisfies optimality conditions&#34;);
            return true;
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void ValidateVertex(int v)
        {
            int V = distTo.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
    }
}

```

测试

```c#
        public void BellmanFordSP()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyEWDn.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.EdgeWeightedDigraph G = new AlgorithmEngine.Graph.EdgeWeightedDigraph(stream);
                int s = 0;
                BellmanFordSP sp = new BellmanFordSP(G, s);

                if (sp.HasNegativeCycle())
                {
                    Console.WriteLine($&#34;{s} has negative cycle&#34;);
                    foreach (DirectedEdge e in sp.NegativeCycle())
                    {
                        Console.Write(e);
                    }
                }
                else
                {
                    for (int v = 0; v &lt; G.Vertic(); v&#43;&#43;)
                    {
                        if (sp.HasPathTo(v))
                        {
                            Console.Write($&#34;{s} to {v} ({sp.DistTo(v)})  &#34;);
                            foreach (DirectedEdge e in sp.PathTo(v))
                            {
                                Console.Write(e &#43; &#34;   &#34;);
                            }
                            Console.WriteLine();
                        }
                        else
                        {
                            Console.WriteLine($&#34;{s} to {v}         no path.&#34;);
                        }
                    }
                }
                
                

            }

            // 0 to 0 (0)  
            // 0 to 1 (0.93)  0-&gt;2 0.26   2-&gt;7 0.34   7-&gt;3 0.39   3-&gt;6 0.52   6-&gt;4 -1.25   4-&gt;5 0.35   5-&gt;1 0.32   
            // 0 to 2 (0.26)  0-&gt;2 0.26   
            // 0 to 3 (0.99)  0-&gt;2 0.26   2-&gt;7 0.34   7-&gt;3 0.39   
            // 0 to 4 (0.26)  0-&gt;2 0.26   2-&gt;7 0.34   7-&gt;3 0.39   3-&gt;6 0.52   6-&gt;4 -1.25   
            // 0 to 5 (0.61)  0-&gt;2 0.26   2-&gt;7 0.34   7-&gt;3 0.39   3-&gt;6 0.52   6-&gt;4 -1.25   4-&gt;5 0.35   
            // 0 to 6 (1.51)  0-&gt;2 0.26   2-&gt;7 0.34   7-&gt;3 0.39   3-&gt;6 0.52   
            // 0 to 7 (0.6)   0-&gt;2 0.26   2-&gt;7 0.34   


        }
```


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2021/04/algorithm9/  

