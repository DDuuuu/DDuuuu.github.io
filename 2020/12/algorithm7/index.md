# 算法 有向图


## 有向图

有向图：边是单向的，每条边所连接的两个顶点都是一个有序对。

![1608014665511](/algorithm/1608014665511.png)

定义：一幅有方向性的图(或有向图)是由一组顶点和一组由方向的边组成的，每条有方向的边都连接这有序的一对顶点。

**出度**：该顶点指出边的总数。

**入度**：指向该顶点边的总数。

**头**：有向边的第一个顶点

**尾**：有向边的第二个顶点

**有向路径**：一系列顶点组成，每个顶点都存在一条有向边指向序列的下一个顶点。

**有向环**：一条至少含有一条边且起点和终点相同的有向路径。

**简单有向环**：一条(除起点和终点)不含有重复的顶点和边的环。

**长度**：路径或环的长度为边的数量。

![1608018635013](/algorithm/1608018635013.png)

**有向图的API**

![1608019592841](/algorithm/1608019592841.png)

![1608019649488](/algorithm/1608019649488.png)

```c#
    public class Digraph
    {
        private static readonly String NEWLINE = System.Environment.NewLine;

        private readonly int _vertices;           // 有向图顶点的数量
        private int _edge;                 // 有向图边的数量
        private LinkedBagNet&lt;int&gt;[] _adj;    //有向图的邻接表
        private int[] _indegree;        // 有向图的度

        /// &lt;summary&gt;
        /// 顶点
        /// &lt;/summary&gt;
        public int Vertices =&gt; _vertices;
        /// &lt;summary&gt;
        /// 边
        /// &lt;/summary&gt;
        public int Edge =&gt; _edge;
        /// &lt;summary&gt;
        /// 顶点V的邻接图，也可以是顶点V射出边的数量
        /// &lt;/summary&gt;
        public LinkedBagNet&lt;int&gt;[] Adj =&gt; _adj;
        /// &lt;summary&gt;
        /// 入射到顶点V的数量
        /// &lt;/summary&gt;
        public int[] Indegree =&gt; _indegree;
        public Digraph(int V)
        {
            if (V &lt; 0) throw new ArgumentException(&#34;Number of vertices in a Digraph must be nonnegative&#34;);
            this._vertices = V;
            this._edge = 0;
            _indegree = new int[V];
            _adj = new LinkedBagNet&lt;int&gt;[V];
            for (int v = 0; v &lt; V; v&#43;&#43;)
            {
                _adj[v] = new LinkedBagNet&lt;int&gt;();
            }
        }


        public Digraph(StreamReader stream)
        {
            if (stream == null) throw new ArgumentException(&#34;argument is null&#34;);
            try
            {
                this._vertices = int.Parse(stream.ReadLine());
                if (_vertices &lt; 0) throw new ArgumentException(&#34;number of vertices in a Digraph must be nonnegative&#34;);
                _indegree = new int[_vertices];
                _adj = new LinkedBagNet&lt;int&gt;[_vertices];
                for (int v = 0; v &lt; _vertices; v&#43;&#43;)
                {
                    _adj[v] = new LinkedBagNet&lt;int&gt;();
                }
                int E = int.Parse(stream.ReadLine());
                if (E &lt; 0) throw new ArgumentException(&#34;number of edges in a Digraph must be nonnegative&#34;);
                for (int i = 0; i &lt; E; i&#43;&#43;)
                {
                    string line = stream.ReadLine();
                    if (!string.IsNullOrEmpty(line) &amp;&amp; line.Split(&#39; &#39;).Count() == 2)
                    {
                        int v = int.Parse(line.Split(&#39; &#39;)[0]);
                        int w = int.Parse(line.Split(&#39; &#39;)[1]);
                        addEdge(v, w);
                    }
                }
            }
            catch (Exception e)
            {
                throw new ArgumentException(&#34;invalid input format in Digraph constructor&#34;, e);
            }
        }


        public Digraph(Digraph G)
        {
            if (G == null) throw new ArgumentException(&#34;argument is null&#34;);

            this._vertices = G.Vertices;
            this._vertices = G.Edge;
            if (_vertices &lt; 0) throw new ArgumentException(&#34;Number of vertices in a Digraph must be nonnegative&#34;);

            // update indegrees
            _indegree = new int[_vertices];
            for (int v = 0; v &lt; _vertices; v&#43;&#43;)
                this._indegree[v] = G._indegree[v];

            // update adjacency lists
            _adj = new LinkedBagNet&lt;int&gt;[_vertices];
            for (int v = 0; v &lt; _vertices; v&#43;&#43;)
            {
                _adj[v] = new LinkedBagNet&lt;int&gt;();
            }

            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
            {
                // reverse so that adjacency list is in same order as original
                Stack&lt;int&gt; reverse = new Stack&lt;int&gt;();
                foreach (int w in G.Adj[v])
                {
                    reverse.Push(w);
                }
                foreach (int w in reverse)
                {
                    _adj[v].Add(w);
                }
            }
        }
        private void validateVertex(int v)
        {
            if (v &lt; 0 || v &gt;= _vertices)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (Vertices - 1));
        }

        /// &lt;summary&gt;
        /// Adds the directed edge v→w to this digraph.
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;v&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;w&#34;&gt;&lt;/param&gt;
        public void addEdge(int v, int w)
        {
            validateVertex(v);
            validateVertex(w);
            _adj[v].Add(w);
            _indegree[w]&#43;&#43;;
            _edge&#43;&#43;;
        }

        /// &lt;summary&gt;
        /// 返回从顶点v射出是线数量
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;v&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public int Outdegree(int v)
        {
            validateVertex(v);
            return _adj[v].Length;
        }

        /// &lt;summary&gt;
        /// 逆转图
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public Digraph Reverse()
        {
            Digraph reverse = new Digraph(_vertices);
            for (int v = 0; v &lt; _vertices; v&#43;&#43;)
            {
                foreach (int w in Adj[v])
                {
                    reverse.addEdge(w, v);
                }
            }
            return reverse;
        }

        public override String ToString()
        {
            StringBuilder s = new StringBuilder();
            s.Append(_vertices &#43; &#34; vertices, &#34; &#43; _edge &#43; &#34; edges &#34; &#43; NEWLINE);
            for (int v = 0; v &lt; _vertices; v&#43;&#43;)
            {
                s.Append(String.Format(&#34;%d: &#34;, v));
                foreach (int w in Adj[v])
                {
                    s.Append(String.Format(&#34;%d &#34;, w));
                }
                s.Append(NEWLINE);
            }
            return s.ToString();
        }
    }
```

### 符号有向图

和之前一样

```c#
   public class SymbolDigraph
    {
        private ST&lt;String, int&gt; _st;  
        private String[] _keys;           
        private Digraph _graph;         

        public SymbolDigraph(String filename, String delimiter)
        {
            _st = new ST&lt;String, int&gt;();

            // First pass builds the index by reading strings to associate
            // distinct strings with an index
            StreamReader reader = new StreamReader(filename);
            while (!reader.EndOfStream)
            {
                String[] a = reader.ReadLine().Split(delimiter.ToCharArray());
                for (int i = 0; i &lt; a.Length; i&#43;&#43;)
                {
                    if (!_st.Contains(a[i]))
                        _st.Add(a[i], _st.Count());
                }
            }

            // inverted index to get string keys in an array
            _keys = new String[_st.Count()];
            foreach (String name in _st.Keys())
            {
                _keys[_st.Get(name)] = name;
            }

            // second pass builds the digraph by connecting first vertex on each
            // line to all others
            _graph = new Digraph(_st.Count());
            reader = new StreamReader(filename);
            while (!reader.EndOfStream) {
                String[] a = reader.ReadLine().Split(delimiter.ToCharArray());
                int v = _st.Get(a[0]);
                for (int i = 1; i &lt; a.Length; i&#43;&#43;)
                {
                    int w = _st.Get(a[i]);
                    _graph.AddEdge(v, w);
                }
            }
        }

        public bool Contains(String s)
        {
            return _st.Contains(s);
        }

        
        public int Index(String s)
        {
            return _st.Get(s);
        }

        public int IndexOf(String s)
        {
            return _st.Get(s);
        }

       
        public String Name(int v)
        {
            validateVertex(v);
            return _keys[v];
        }

        public String NameOf(int v)
        {
            validateVertex(v);
            return _keys[v];
        }

        public Digraph G()
        {
            return _graph;
        }

        public Digraph Digraph()
        {
            return _graph;
        }

        private void validateVertex(int v)
        {
            int V = _graph.Vertices;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
```

测试

```c#
        [TestMethod()]
        public void SymbolDigraphTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\routes.txt&#34;);
            SymbolDigraph sg = new SymbolDigraph(data, &#34; &#34;);
            Digraph graph = sg.Digraph();
            StreamReader reader = new StreamReader(data);
            while (!reader.EndOfStream)
            {
                String source = reader.ReadLine().Split(&#39; &#39;)[0];
                if (sg.Contains(source))
                {
                    Console.Write($&#34;{source} :&#34;);
                    int s = sg.Index(source);
                    foreach (int v in graph.Adj[s])
                    {
                        Console.Write(&#34; &#34; &#43; sg.Name(v));
                    }
                }
                else
                {
                    Console.WriteLine(&#34;input not contain &#39;&#34; &#43; source &#43; &#34;&#39;&#34;);
                }
                Console.WriteLine();
            }
            //JFK: ORD ATL MCO
            //ORD : ATL PHX DFW HOU DEN
            //ORD : ATL PHX DFW HOU DEN
            //DFW : HOU PHX
            //JFK: ORD ATL MCO
            //ORD : ATL PHX DFW HOU DEN
            //ORD : ATL PHX DFW HOU DEN
            //ATL : MCO HOU
            //DEN: LAS PHX
            //PHX: LAX
            //JFK : ORD ATL MCO
            //DEN : LAS PHX
            //DFW: HOU PHX
            //ORD: ATL PHX DFW HOU DEN
            //LAS : PHX LAX
            //ATL: MCO HOU
            //HOU: MCO
            //LAS : PHX LAX
        }
```



### 查找所有节点

可达性：是否存在一条从S到达顶点V的有向路径

![1608022190584](/algorithm/1608022190584.png)

深度搜索

```c#
    public class DirectedDFS
    {
        private bool[] _marked;  
        private int _count;   //顶点数量

        public DirectedDFS(Digraph G, int s)
        {
            _marked = new bool[G.Vertices];
            validateVertex(s);
            dfs(G, s);
        }

        public DirectedDFS(Digraph G, IEnumerable&lt;int&gt; sources)
        {
            _marked = new bool[G.Vertices];
            validateVertices(sources);
            foreach (int v in sources)
            {
                if (!_marked[v]) dfs(G, v);
            }
        }
        /// &lt;summary&gt;
        /// 深度搜素
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;G&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;v&#34;&gt;&lt;/param&gt;
        private void dfs(Digraph G, int v)
        {
            _count&#43;&#43;;
            _marked[v] = true;
            foreach (int w in G.Adj[v])
            {
                if (!_marked[w]) dfs(G, w);
            }
        }

       
        public bool Marked(int v)
        {
            validateVertex(v);
            return _marked[v];
        }

        
        public int Count()
        {
            return _count;
        }

        
        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void validateVertices(IEnumerable&lt;int&gt; vertices)
        {
            if (vertices == null)
            {
                throw new ArgumentException(&#34;argument is null&#34;);
            }
            foreach (int v in vertices)
            {
                if (v == default)
                {
                    throw new ArgumentException(&#34;vertex is null&#34;);
                }

                validateVertex(v);
            }
        }

    }
```

测试

```c#
        [TestMethod()]
        public void DirectedDFSTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                DirectedDFS dfs=new DirectedDFS(G,9);
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    if (dfs.Marked(v)) Console.Write(v &#43; &#34; &#34;);
                }
            }
        }
```



![1608083256721](/algorithm/1608083256721.png)

可达性非递归

``` c#
    public class NonrecursiveDirectedDFS
    {
        private bool[] _marked;  
                                   
        public NonrecursiveDirectedDFS(Digraph G, int s)
        {
            _marked = new bool[G.Vertices];
            validateVertex(s);
                IEnumerator&lt;int&gt;[] adj =new IEnumerator&lt;int&gt;[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                adj[v] = G.Adj[v].GetEnumerator();
            Stack&lt;int&gt; stack = new Stack&lt;int&gt;();
            _marked[s] = true;
            stack.Push(s);
            while (stack.Any())
            {
                int v = stack.Peek();
                if (adj[v].MoveNext())
                {
                    int w = adj[v].Current;
                    if (!_marked[w])
                    {
                        
                        _marked[w] = true;
                        stack.Push(w);
                    }
                }
                else
                {
                    stack.Pop();
                }
            }
        }

        public bool Marked(int v)
        {
            validateVertex(v);
            return _marked[v];
        }

        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```

### 寻找所有路径

深度搜索

```c#
    public  class DepthFirstDirectedPaths
    {
        private bool[] _marked;  
        private int[] _edgeTo;      
        private readonly int s;       // 起点

        public DepthFirstDirectedPaths(Digraph G, int s)
        {
            _marked = new bool[G.Vertices];
            _edgeTo = new int[G.Vertices];
            this.s = s;
            validateVertex(s);
            dfs(G, s);
        }

        private void dfs(Digraph G, int v)
        {
            _marked[v] = true;
            foreach (int w in G.Adj[v])
            {
                if (!_marked[w])
                {
                    _edgeTo[w] = v;
                    dfs(G, w);
                }
            }
        }

        public bool HasPathTo(int v)
        {
            validateVertex(v);
            return _marked[v];
        }


        
        public IEnumerable&lt;int&gt; PathTo(int v)
        {
            validateVertex(v);
            if (!HasPathTo(v)) return null;
            Stack&lt;int&gt; path = new Stack&lt;int&gt;();
            for (int x = v; x != s; x = _edgeTo[x])
                path.Push(x);
            path.Push(s);
            return path;
        }

        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
    }
```

测试

```c#
        [TestMethod()]
        public void DepthFirstDirectedPathsTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                DepthFirstDirectedPaths dfs=new DepthFirstDirectedPaths(G,9);
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    if (dfs.HasPathTo(v))
                    {
                        Console.Write($&#34;{9} to {v}:  &#34;);
                        foreach (int x in dfs.PathTo(v))
                        {
                            if (x == 9) Console.Write(x);
                            else Console.Write(&#34;-&#34; &#43; x);
                        }
                        Console.WriteLine();
                    }
                    else
                    {
                        Console.Write($&#34;9 to {v}:  not connected\n&#34;);
                    }

                }
            }
            //9 to 0:  9 - 11 - 4 - 3 - 2 - 0
            //9 to 1:  9 - 11 - 4 - 3 - 2 - 0 - 1
            //9 to 2:  9 - 11 - 4 - 3 - 2
            //9 to 3:  9 - 11 - 4 - 3
            //9 to 4:  9 - 11 - 4
            //9 to 5:  9 - 11 - 4 - 3 - 5
            //9 to 6:  not connected
            //9 to 7:  not connected
            //9 to 8:  not connected
            //9 to 9:  9
            //9 to 10:  9 - 10
            //9 to 11:  9 - 11
            //9 to 12:  9 - 11 - 12
        }
```

广度搜索

广度搜索需要一个队列辅助，基本和无向图一样

```c#
public class BreadthFirstDirectedPaths
    {
        private static readonly int INFINITY = int.MaxValue;
        private bool[] _marked;  
        private int[] _edgeTo;      
        private int[] _distTo;      

        public BreadthFirstDirectedPaths(Digraph G, int s)
        {
            _marked = new bool[G.Vertices];
            _distTo = new int[G.Vertices];
            _edgeTo = new int[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                _distTo[v] = INFINITY;
            validateVertex(s);
            bfs(G, s);
        }

        
        public BreadthFirstDirectedPaths(Digraph G, IEnumerable&lt;int&gt; sources)
        {
            _marked = new bool[G.Vertices];
            _distTo = new int[G.Vertices];
            _edgeTo = new int[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                _distTo[v] = INFINITY;
            validateVertices(sources);
            bfs(G, sources);
        }

        // BFS from single source
        private void bfs(Digraph G, int s)
        {
            Queue&lt;int&gt; q = new Queue&lt;int&gt;();
            _marked[s] = true;
            _distTo[s] = 0;
            q.Enqueue(s);
            while (q.Any())
            {
                int v = q.Dequeue();
                foreach (int w in G.Adj[v])
                {
                    if (!_marked[w])
                    {
                        _edgeTo[w] = v;
                        _distTo[w] = _distTo[v] &#43; 1;
                        _marked[w] = true;
                        q.Enqueue(w);
                    }
                }
            }
        }

        // BFS from multiple sources
        private void bfs(Digraph G, IEnumerable&lt;int&gt; sources)
        {
            Queue&lt;int&gt; q = new Queue&lt;int&gt;();
            foreach (int s in sources)
            {
                _marked[s] = true;
                _distTo[s] = 0;
                q.Enqueue(s);
            }
            while (q.Any())
            {
                int v = q.Dequeue();
                foreach (int w in G.Adj[v])
                {
                    if (!_marked[w])
                    {
                        _edgeTo[w] = v;
                        _distTo[w] = _distTo[v] &#43; 1;
                        _marked[w] = true;
                        q.Enqueue(w);
                    }
                }
            }
        }

        public bool HasPathTo(int v)
        {
            validateVertex(v);
            return _marked[v];
        }

       
        public int DistTo(int v)
        {
            validateVertex(v);
            return _distTo[v];
        }

        public IEnumerable&lt;int&gt; PathTo(int v)
        {
            validateVertex(v);

            if (!HasPathTo(v)) return null;
            Stack&lt;int&gt; path = new Stack&lt;int&gt;();
            int x;
            for (x = v; _distTo[x] != 0; x = _edgeTo[x])
                path.Push(x);
            path.Push(x);
            return path;
        }

        
        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

        // throw an IllegalArgumentException unless {@code 0 &lt;= v &lt; V}
        private void validateVertices(IEnumerable&lt;int&gt; vertices)
        {
            if (vertices == null)
            {
                throw new ArgumentException(&#34;argument is null&#34;);
            }
            foreach (int v in vertices)
            {
                if (v == default)
                {
                    throw new ArgumentException(&#34;vertex is null&#34;);
                }
                validateVertex(v);
            }
        }

    }
```

测试

```c#
        [TestMethod()]
        public void BreadthFirstDirectedPathsTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                BreadthFirstDirectedPaths bfs = new BreadthFirstDirectedPaths(G, 9);

                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    if (bfs.HasPathTo(v))
                    {
                        Console.Write($&#34;{9} to {v}:  &#34;);
                        foreach (int x in bfs.PathTo(v))
                        {
                            if (x == 9) Console.Write(x);
                            else Console.Write(&#34;-&#34; &#43; x);
                        }
                        Console.WriteLine();
                    }
                    else
                    {
                        Console.Write($&#34;9 to {v}:  not connected\n&#34;);
                    }

                }
            }
            //9 to 0:  9 - 11 - 4 - 2 - 0
            //9 to 1:  9 - 11 - 4 - 2 - 0 - 1
            //9 to 2:  9 - 11 - 4 - 2
            //9 to 3:  9 - 11 - 4 - 3
            //9 to 4:  9 - 11 - 4
            //9 to 5:  9 - 11 - 4 - 3 - 5
            //9 to 6:  not connected
            //9 to 7:  not connected
            //9 to 8:  not connected
            //9 to 9:  9
            //9 to 10:  9 - 10
            //9 to 11:  9 - 11
            //9 to 12:  9 - 11 - 12
        }
```

广度搜索和深度搜索的路径是不一样的，广度搜索的路径是最短路径。

### 寻找有向环

![1608083320441](/algorithm/1608083320441.png)

#### 调度问题

一种广泛的模型是给定一组任务并安排它们的执行顺序，限制条件是这些任务的执行方法和起始时间。哪些任务必须在哪些任务之前完成，最重要的限制条件叫优先级限制。

![1608086114660](/algorithm/1608086114660.png) 

优先级限制下调度问题：一组关于任务完成的先后次序的优先级限制。

![1608086253669](/algorithm/1608086253669.png)

![1608086265546](/algorithm/1608086265546.png)

![1608086281170](/algorithm/1608086281170.png)

**拓扑排序**：==有向图，将所有的顶点排序，使得所有的有向边均从排在前面的元素指向排在后面的元素（或者说明无法做到这一点）。==等价于调度问题，获得拓扑排序是为了解决调度问题。

如果一个有优先级限制的问题中存在有向环，那么这个问题无解。

**有向无环图（DAG）**:不含有环的有向图。

![1608086627776](/algorithm/1608086627776.png)

深度搜索，就跟拿根绳走迷宫是一样的

```c#
   public class DirectedCycle
    {
        private bool[] _marked;        
        private int[] _edgeTo;            
        private bool[] _onStack;       //该顶点是否在栈
        private Stack&lt;int&gt; _cycle;    // 环栈

        public DirectedCycle(Digraph G)
        {
            _marked = new bool[G.Vertices];
            _onStack = new bool[G.Vertices];
            _edgeTo = new int[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                if (!_marked[v] &amp;&amp; _cycle == null) dfs(G, v);
        }
        private void dfs(Digraph G, int v)
        {
            _onStack[v] = true;
            _marked[v] = true;
            foreach (int w in G.Adj[v])
            {

                // 环已经被发现
                if (_cycle != null) return;

                // 新的顶点且环没有被发现
                else if (!_marked[w])
                {
                    _edgeTo[w] = v;
                    dfs(G, w);
                }
                else if (_onStack[w])//查找到环，压入环栈
                {
                    _cycle = new Stack&lt;int&gt;();
                    for (int x = v; x != w; x = _edgeTo[x])
                    {
                        _cycle.Push(x);
                    }
                    _cycle.Push(w);
                    _cycle.Push(v);
                   
                }
            }
            _onStack[v] = false;
        }

        
        public bool HasCycle()
        {
            return _cycle != null;
        }

         
        public IEnumerable&lt;int&gt; Cycle()
        {
            return _cycle;
        }


        
        private bool check()
        {

            if (HasCycle())
            {
                // verify cycle
                int first = -1, last = -1;
                foreach (int v in _cycle)
                {
                    if (first == -1) first = v;
                    last = v;
                }
                if (first != last)
                {
                    Console.Write(&#34;cycle begins with %d and ends with %d\n&#34;, first, last);
                    return false;
                }
            }


            return true;
        }

    }
```

测试

```c#
        [TestMethod()]
        public void DirectedCycleTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                DirectedCycle finder = new DirectedCycle(G);
                if (finder.HasCycle())
                {
                    Console.Write(&#34;Directed cycle: &#34;);
                    foreach (int v in finder.Cycle())//只能找到一个环
                    {
                        Console.Write(v &#43; &#34; &#34;);
                    }
                    Console.WriteLine();
                }
                else
                {
                   Console.WriteLine(&#34;No directed cycle&#34;);
                }
                //Directed cycle: 3 5 4 3 
            }
        }
```

![1608087377815](/algorithm/1608087377815.png)

![1608087411295](/algorithm/1608087411295.png)

广度搜索，就像一个人有影分身走不同的路径

```c#
    public  class DirectedCycleX
    {
        private Stack&lt;int&gt; _cycle;  

        public DirectedCycleX(Digraph G)
        {
            int[] indegree = new int[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
            {
                indegree[v] = G.Indegree[v];
            }

            //所有没有入射边的顶点压入队列
            Queue&lt;int&gt; queue = new Queue&lt;int&gt;();
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                if (indegree[v] == 0) queue.Enqueue(v);

            while (queue.Any())//广度搜索
            {
                int v = queue.Dequeue();
                foreach (int w in G.Adj[v])
                {
                    indegree[w]--;//去除掉从没有入射边顶点射出的边，并查看是否有没有入射边的顶点
                    if (indegree[w] == 0) queue.Enqueue(w);
                }
            }

            int[] edgeTo = new int[G.Vertices];
            int root = -1; 
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
            {
                if (indegree[v] == 0) continue;
                else root = v;//入射边大于0的顶点，有环
                foreach (int w in G.Adj[v])
                {
                    if (indegree[w] &gt; 0)
                    {
                        edgeTo[w] = v;
                    }
                }
            }

            if (root != -1)
            {
                // find any vertex on cycle
                bool[] visited = new bool[G.Vertices];
                while (!visited[root])
                {
                    visited[root] = true;
                    root = edgeTo[root];
                }

                // extract cycle
                _cycle = new Stack&lt;int&gt;();
                int v = root;
                do
                {
                    _cycle.Push(v);
                    v = edgeTo[v];
                } while (v != root);
                _cycle.Push(root);
            }
            
        }

        
        public IEnumerable&lt;int&gt; Cycle()
        {
            return _cycle;
        }

        public bool HasCycle()
        {
            return _cycle != null;
        }

        private bool check()
        {

            if (HasCycle())
            {
                int first = -1, last = -1;
                foreach (int v in _cycle)
                {
                    if (first == -1) first = v;
                    last = v;
                }
                if (first != last)
                {
                   Console.Write(&#34;cycle begins with %d and ends with %d\n&#34;, first, last);
                    return false;
                }
            }
            return true;
        }
    }
```

测试

``` c#
        [TestMethod]
        public void DirectedCycleXTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                DirectedCycleX finder = new DirectedCycleX(G);

                if (finder.HasCycle())
                {
                    Console.Write(&#34;Directed cycle: &#34;);
                    foreach (int v in finder.Cycle())
                    {
                        Console.Write(v &#43; &#34; &#34;);
                    }
                    Console.WriteLine();
                }

                else
                {
                    Console.WriteLine(&#34;No directed cycle&#34;);
                }
                //Directed cycle: 12 9 11 12  
            }
        }
```

#### 顶点排序

基于深度优先搜索的顶点排序，深度优先搜索正好只会访问每个顶点一次。

如果将dfs()的参数顶点保存在一个数据结构中，遍历这个数据结构实际上就能够访问图中的所有顶点。

要讨论的是需要在递归前保存顶点，还是递归后保存顶点。

![1608095332998](/algorithm/1608095332998.png)

```c#
    public class DepthFirstOrder
    {
        private bool[] _marked;          
        private int[] _pre;                 
        private int[] _post;                
        private Queue&lt;int&gt; _preorder;   //在递归之前将顶点保存，顺序正确
        private Queue&lt;int&gt; _postorder;  //在递归之后将顶点保存，最后顺序应该倒一下。
        private int _preCounter;           
        private int _postCounter;          

        public DepthFirstOrder(Digraph G)
        {
            _pre = new int[G.Vertices];
            _post = new int[G.Vertices];
            _postorder = new Queue&lt;int&gt;();
            _preorder = new Queue&lt;int&gt;();
            _marked = new bool[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                if (!_marked[v]) dfs(G, v);

        }

        private void dfs(Digraph G, int v)
        {
            _marked[v] = true;
            _pre[v] = _preCounter&#43;&#43;;
            _preorder.Enqueue(v);
            foreach (int w in G.Adj[v])
            {
                if (!_marked[w])
                {
                    dfs(G, w);
                }
            }
            _postorder.Enqueue(v);
            _post[v] = _postCounter&#43;&#43;;
        }

        public int Pre(int v)
        {
            validateVertex(v);
            return _pre[v];
        }

        
        public int Post(int v)
        {
            validateVertex(v);
            return _post[v];
        }

        
        public IEnumerable&lt;int&gt; Post()
        {
            return _postorder;
        }

        public IEnumerable&lt;int&gt; Pre()
        {
            return _preorder;
        }

        public IEnumerable&lt;int&gt; ReversePost()//通过栈进行反转
        {
            Stack&lt;int&gt; reverse = new Stack&lt;int&gt;();
            foreach (int v in _postorder)
                reverse.Push(v);
            return reverse;
        }

        private bool check()
        {

            // check that post(v) is consistent with post()
            int r = 0;
            foreach (int v in _post)
            {
                if (_post[v] != r)
                {
                    Console.Write(&#34;post(v) and post() inconsistent&#34;);
                    return false;
                }
                r&#43;&#43;;
            }

            // check that pre(v) is consistent with pre()
            r = 0;
            foreach (int v in _pre)
            {
                if (_pre[v] != r)
                {
                    Console.Write(&#34;pre(v) and pre() inconsistent&#34;);
                    return false;
                }
                r&#43;&#43;;
            }

            return true;
        }
        
        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```

测试

```c#
        [TestMethod()]
        public void DepthFirstOrderTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDAG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                DepthFirstOrder dfs = new DepthFirstOrder(G);

               Console.WriteLine(&#34;v  pre post&#34;);
                Console.WriteLine(&#34;--------------&#34;);
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    Console.Write($&#34;{v}   {dfs.Pre(v)}   {dfs.Post(v)}\n&#34;);
                }
                Console.WriteLine();
                Console.Write(&#34;Preorder:  &#34;);
                foreach (int v in dfs.Pre())
                {
                    Console.Write(v &#43; &#34; &#34;);
                }
                Console.WriteLine();

                Console.Write(&#34;Postorder: &#34;);
                foreach (int v in dfs.Post())
                {
                    Console.Write(v &#43; &#34; &#34;);
                }
                Console.WriteLine();

                Console.Write(&#34;Reverse postorder: &#34;);
                foreach (int v in dfs.ReversePost())
                {
                    Console.Write(v &#43; &#34; &#34;);
                }
                Console.WriteLine();
            }
            //v pre post
            //--------------
            //0   0   8
            //1   3   2
            //2   9   10
            //3   10   9
            //4   2   0
            //5   1   1
            //6   4   7
            //7   11   11
            //8   12   12
            //9   5   6
            //10   8   5
            //11   6   4
            //12   7   3

            //Preorder: 0 5 4 1 6 9 11 12 10 2 3 7 8
            //Postorder: 4 5 1 12 11 10 9 6 0 3 2 7 8
            //Reverse postorder: 8 7 2 3 0 6 9 10 11 12 1 5 4
        }
```



#### 拓扑排序

拓扑排序：给定一副有向图，将所有的顶点排序，使得所有的有向边均从排在前面的元素指向排在后面的元素

命题E：当且仅当一幅有向图是无环图时它才能进行拓扑排序

命题F:一幅有向无环图的拓扑排序即为所有顶点的逆后序排序。

命题G:使用深度优先搜索对有向无环图进行拓扑排序所需的时间和V&#43;E成正比。

![1608088729910](/algorithm/1608088729910.png)

```c#
    public class Topological
    {
        private IEnumerable&lt;int&gt; _order;  //顶点的拓扑顺序

        private int[] _rank;
        
        public Topological(Digraph G)
        {
            DirectedCycle finder = new DirectedCycle(G);//先检测一下是否有环
            if (!finder.HasCycle())
            {
                DepthFirstOrder dfs = new DepthFirstOrder(G);//顶点排序
                _order = dfs.ReversePost();//后续插入反转就是拓扑排序，又称逆后续
                _rank = new int[G.Vertices];
                int i = 0;
                foreach (int v in _order)
                    _rank[v] = i&#43;&#43;;
            }
        }

        public IEnumerable&lt;int&gt; Order()
        {
            return _order;
        }

        
        public bool HasOrder()
        {
            return _order != null;
        }

        
        public bool IsDAG()
        {
            return HasOrder();
        }

        
        public int Rank(int v)
        {
            validateVertex(v);
            if (HasOrder()) return _rank[v];
            else return -1;
        }

        private void validateVertex(int v)
        {
            int V = _rank.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```



测试

```c#
        [TestMethod()]
        public void TopologicalTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDAG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                Topological topological = new Topological(G);
                foreach (int v in topological.Order())
                {
                    Console.Write(v&#43;&#34; &#34;);
                }
            }
            //8 7 2 3 0 6 9 10 11 12 1 5 4 
        }
```

![1608096846767](/algorithm/1608096846767.png)

**逆后序就是顶点遍历完成顺序的逆，从下往上**

更流性的一种拓扑算法，其实就是广度搜索

```c#
    public class TopologicalX
    {
        private Queue&lt;int&gt; _order;     // 顶点的拓扑顺序
        private int[] _ranks;              

        public TopologicalX(Digraph G)
        {
            //入射到顶点的边的数量
            int[] indegree = new int[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
            {
                indegree[v] = G.Indegree[v];
            }
            // initialize 
            _ranks = new int[G.Vertices];
            _order = new Queue&lt;int&gt;();
            int count = 0;
            Queue&lt;int&gt; queue = new Queue&lt;int&gt;();
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                if (indegree[v] == 0) queue.Enqueue(v);//将没有入射的压入

            while (queue.Any())
            {
                int v = queue.Dequeue();
                _order.Enqueue(v);
                _ranks[v] = count&#43;&#43;;
                foreach (int w in G.Adj[v])
                {
                    indegree[w]--;
                    if (indegree[w] == 0) queue.Enqueue(w);//广度搜索
                }
            }

            if (count != G.Vertices)
            {
                _order = null;
            }
        }
        public IEnumerable&lt;int&gt; Order()
        {
            return _order;
        }

        public bool HasOrder()
        {
            return _order != null;
        }

        public int Rank(int v)
        {
            validateVertex(v);
            if (HasOrder()) return _ranks[v];
            else return -1;
        }

        private bool check(Digraph G)
        {

            // digraph is acyclic
            if (HasOrder())
            {
                // check that ranks are a permutation of 0 to V-1
                bool[] found = new bool[G.Vertices];
                for (int i = 0; i &lt; G.Vertices; i&#43;&#43;)
                {
                    found[_ranks[i]] = true;
                }
                for (int i = 0; i &lt; G.Vertices; i&#43;&#43;)
                {
                    if (!found[i])
                    {
                        Console.Write(&#34;No vertex with rank &#34; &#43; i);
                        return false;
                    }
                }
                // check that ranks provide a valid topological order
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    foreach (int w in G.Adj[v])
                    {
                        if (Rank(v) &gt; Rank(w))
                        {
                            Console.Write($&#34;{v}-{w}: rank({v}) = {Rank(v)}, rank({w}) = {Rank(w)}\n&#34;);
                            return false;
                        }
                    }
                }
                // check that order() is consistent with rank()
                int r = 0;
                foreach (int v in Order())
                {
                    if (Rank(v) != r)
                    {
                        Console.Write(&#34;order() and rank() inconsistent&#34;);
                        return false;
                    }
                    r&#43;&#43;;
                }
            }
            return true;
        }
        
        private void validateVertex(int v)
        {
            int V = _ranks.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }
    }
```

测试：

```c#
        [TestMethod()]
        public void TopologicalXTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDAG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                TopologicalX topological = new TopologicalX(G);
                foreach (int v in topological.Order())
                {
                    Console.Write(v &#43; &#34; &#34;);
                }
            }
            //2 8 0 3 7 1 5 6 9 4 11 10 12 
        }
```

解决任务调度类通常由3步：

&#43; 指明任务和优先级条件
&#43; 不断检测并去除有向图中的所有环，以确保存在可行方案的
&#43; 使用拓扑排序解决调度问题。

### 强连通性问题

强连通：如果两个顶点v和w是相互可达的，则称它们为强连通。

两个顶点是强连通的当且仅当它们都在一个普通的有向环中。

![1608104332645](/algorithm/1608104332645.png)

性质：

&#43; 自反性：任意顶点V和自己都是强连通的
&#43; 对称性：如果v和w是强连通的，那么w和v也是强连通的。
&#43; 传递性：如果v和w是强连通的且w和x也是强连通的，那么v和x也是强连通的。

强连通可以帮助进行归类，可以帮助生物学家解决生态的问题

![1608104661119](/algorithm/1608104661119.png)



![1608104677502](/algorithm/1608104677502.png)

#### Kosaraju算法

![1608104963592](/algorithm/1608104963592.png)

Kosaraju算法的步骤

&#43; 对给定的G，使用DepthFirstOrder来计算它的反向图G的逆后序排序。
&#43; 在G中进行标准的深度优先搜索，按照上一步的顺序来访问所有未被标记过的顶点。
&#43; 在构造函数中，所有在同一个递归dfs()调用中被访问到的顶点都在同一个强连通分量中，将它们按照和CC相同的方式识别出来。

```c#
    /// &lt;summary&gt;
    /// Kosaraju算法
    /// &lt;/summary&gt;
    public class KosarajuSharirSCC
    {
        private bool[] _marked; //已访问过的顶点
        private int[] _id;      //强连通分量的标识符     
        private int _count;     //强连通分量的数量    

        public KosarajuSharirSCC(Digraph G)
        {
            DepthFirstOrder dfo = new DepthFirstOrder(G.Reverse());//这边注意是G.Reverse
            _marked = new bool[G.Vertices];
            _id = new int[G.Vertices];
            foreach (int v in dfo.ReversePost())//拓扑排序的顶点
            {
                if (!_marked[v])//遍历拓扑排序的顶点
                {
                    dfs(G, v);//
                    _count&#43;&#43;;//找到一个新的连通量
                }
            }
        }

        private void dfs(Digraph G, int v)
        {
            _marked[v] = true;
            _id[v] = _count;//强连通分量的顶点对应的id都一样
            foreach (int w in G.Adj[v])
            {
                if (!_marked[w]) dfs(G, w);
            }
        }

        public int Count()
        {
            return _count;
        }

        public bool StronglyConnected(int v, int w)
        {
            validateVertex(v);
            validateVertex(w);
            return _id[v] == _id[w];
        }

        public int Id(int v)
        {
            validateVertex(v);
            return _id[v];
        }

        private void validateVertex(int v)
        {
            int V = _marked.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```

测试

```c#
        [TestMethod()]
        public void KosarajuTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                KosarajuSharirSCC scc=new KosarajuSharirSCC(G);
                // number of connected components
                int m = scc.Count();
                Console.WriteLine(m &#43; &#34; strong components&#34;);

                // compute list of vertices in each strong component
                Queue&lt;int&gt;[] components = new Queue&lt;int&gt;[m];
                for (int i = 0; i &lt; m; i&#43;&#43;)
                {
                    components[i] = new Queue&lt;int&gt;();
                }
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    components[scc.Id(v)].Enqueue(v);//将每一种连通向都
                }

                // print results
                for (int i = 0; i &lt; m; i&#43;&#43;)
                {
                    foreach (int v in components[i])
                    {
                        Console.Write(v &#43; &#34; &#34;);
                    }
                    Console.WriteLine();
                }
            }
            //5 strong components
            //1
            //0 2 3 4 5
            //9 10 11 12
            //6 8
            //7
        }
```

命题H:使用深度优先搜索查找给定有向图G的反向图GR（也就是上面的拓扑排序），根据由此得到的所有顶点的逆后序再次用深度优先搜索处理有向图G(Kosaraju算法),其构造函数中的每一次递归调用所标记的顶点都在同一个强连通分量之中。

![1608107898183](/algorithm/1608107898183.png)

![1608130145852](/algorithm/1608130145852.png)

#### 顶点对的可达性

![1608186976727](/algorithm/1608186976727.png)

有向图G的传递闭包是由相同的一组顶点组成的另一幅有向图，在传递闭包中存在一条从v指向w的边当且仅当在G中w是从v可达的。

传递闭包不如用深度优先搜索算法

```c#
    public class TransitiveClosure
    {
        private DirectedDFS[] tc;
        public TransitiveClosure(Digraph G)
        {
            tc = new DirectedDFS[G.Vertices];
            for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                tc[v] = new DirectedDFS(G, v);
        }

        public bool Reachable(int v, int w)
        {
            validateVertex(v);
            validateVertex(w);
            return tc[v].Marked(w);
        }
        private void validateVertex(int v)
        {
            int V = tc.Length;
            if (v &lt; 0 || v &gt;= V)
                throw new ArgumentException(&#34;vertex &#34; &#43; v &#43; &#34; is not between 0 and &#34; &#43; (V - 1));
        }

    }
```

测试：

```c#
        [TestMethod()]
        public void TransitiveClosureTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\tinyDG.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                AlgorithmEngine.Graph.Digraph G = new AlgorithmEngine.Graph.Digraph(stream);
                TransitiveClosure tc = new TransitiveClosure(G);
                // print header
                Console.Write(&#34;//     &#34;);
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                    Console.Write($&#34; {v:D2}&#34;);
                Console.WriteLine();
                Console.WriteLine(&#34;//--------------------------------------------&#34;);

                // print transitive closure
                for (int v = 0; v &lt; G.Vertices; v&#43;&#43;)
                {
                    Console.Write($&#34;// {v:D2}: &#34;);
                    for (int w = 0; w &lt; G.Vertices; w&#43;&#43;)
                    {
                        if (tc.Reachable(v, w)) Console.Write(&#34;  T&#34;);
                        else Console.Write(&#34;   &#34;);
                    }
                    Console.WriteLine();
                }
            }
            //      00 01 02 03 04 05 06 07 08 09 10 11 12
            //--------------------------------------------
            // 00:   T  T  T  T  T  T                     
            // 01:      T                                 
            // 02:   T  T  T  T  T  T                     
            // 03:   T  T  T  T  T  T                     
            // 04:   T  T  T  T  T  T                     
            // 05:   T  T  T  T  T  T                     
            // 06:   T  T  T  T  T  T  T     T  T  T  T  T
            // 07:   T  T  T  T  T  T  T  T  T  T  T  T  T
            // 08:   T  T  T  T  T  T  T     T  T  T  T  T
            // 09:   T  T  T  T  T  T           T  T  T  T
            // 10:   T  T  T  T  T  T           T  T  T  T
            // 11:   T  T  T  T  T  T           T  T  T  T
            // 12:   T  T  T  T  T  T           T  T  T  T
        }
```

### 总结

![1608189606513](/algorithm/1608189606513.png)



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/12/algorithm7/  

