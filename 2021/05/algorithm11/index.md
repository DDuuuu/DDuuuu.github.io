# 算法 字符串查找


## 字符串查找

### 单词查找树

![image-20210426091237214](/algorithm/image-20210426091237214.png)

单词查找树是由链接的结点所组成的数据结构，这些链接可能为空，也可能指向其他结点。

每个结点都只可能有一个指向它的结点，称为它的父结点（只有一个结点除外，即根结点，没有任何结点指向根结点）。

每个结点都含有R 条链接， 其中R为字母表的大小。

单词查找树一般都含有大最的空链接，因此在绘制一棵单词查找树时一般会忽略空链接。

尽管链接指向的是结点，但是也可以看作链接指向的是另一棵单词查找树，

值为空的结点在符号表中没有对应的键，它们的存在是为了简化单词查找树中的查找操作。

#### 查找

&#43; 键的尾字符所对应的结点中的值非空，这是一次命中的查找~所对应的值就是键的尾字符所对应的结点中保存的值。

&#43; 键的尾字符所对应的结点中的值为空，这是一次未命中的查找一符号表中不存在被查找的键。 

&#43; 查找结束于一条空链接这也是一次未命中的查找。 

![image-20210426092022878](/algorithm/image-20210426092022878.png)

#### 插入

&#43; 在到达键的尾字符之前就遇到了一个空链接。在这种情况下，字符查找树中不存在与键的尾字符对应的结点，因此需要为键中还未被检查的**每个字符创建一个对应的结点并将键的值保存到最后一个字符的结点中**。 

&#43; 在遇到空链接之前就到达了键的尾字符。在这种情况下，和关联性数组一样，将该结点的值设为键所对应的值 

![image-20210426094028418](/algorithm/image-20210426094028418.png)

#### 表示

将空链接考虑进来将会突出单词查找树的以下重要性质：

&#43; 每个结点都含有R 个链接，对应着每个可能出现的字符；
&#43; 字符和键均隐式地保存在数据结构中。

每个结点都含有一个值和26 个链接

![image-20210426100100612](/algorithm/image-20210426100100612.png)

```c#
public class TrieST&lt;Value&gt; 
    { 
        private static readonly int R = 256;        // extended ASCII,基数


        private Node root;      // 树的根
        private int n;          // number of keys in trie

        // R-way trie node
        private  class Node
        {
            public Object val;
            public Node[] next = new Node[R];
        }

        public TrieST()
        {
        }


       /// &lt;summary&gt;
       /// 返回给定关联的值
       /// &lt;/summary&gt;
       /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
       /// &lt;returns&gt;&lt;/returns&gt;
        public Value Get(String key)
        {
            if (key == null) throw new ArgumentException(&#34;argument to get() is null&#34;);
            Node x = Get(root, key, 0);
            if (x == null) return default(Value);
            return (Value)x.val;
        }

        /// &lt;summary&gt;
        /// 此符号表是否包含给定的键
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public bool Contains(string key)
        {
            if (key == null) throw new ArgumentException(&#34;argument to contains() is null&#34;);
            return Get(key) != null;
        }

        private Node Get(Node x, String key, int d)
        {
            if (x == null) return null;
            if (d == key.Length) return x;
            char c = key[d];
            return Get(x.next[c], key, d &#43; 1);
        }

        /// &lt;summary&gt;
        /// 将键值对插入符号表，覆盖旧值
        /// 如果键已在符号表中，则使用新值。
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;val&#34;&gt;&lt;/param&gt;
        public void Put(String key, Value val)
        {
            if (key == null) throw new ArgumentException(&#34;first argument to put() is null&#34;);
            if (val == null) Delete(key);
            else root = Put(root, key, val, 0);
        }

        private Node Put(Node x, String key, Value val, int d)
        {    //如果key存在于以x 为根结点的子单词查找树中则更新与它相关联的值
            if (x == null) x = new Node();
            if (d == key.Length)
            {
                if (x.val == null) n&#43;&#43;;
                x.val = val;
                return x;
            }
            char c = key[d];////找到笫d 个字符所对应的子单词查找树
            x.next[c] = Put(x.next[c], key, val, d &#43; 1);
            return x;
        }

        /// &lt;summary&gt;
        /// 返回此符号表中的键值对的数量。
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public int Size()
        {
            return n;
        }

        /// &lt;summary&gt;
        /// 此符号表是否为空
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public bool IsEmpty()
        {
            return Size() == 0;
        }

        /// &lt;summary&gt;
        /// 返回所有键
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public IEnumerable&lt;string&gt; Keys()
        {
            return KeysWithPrefix(&#34;&#34;);
        }

        
        public IEnumerable&lt;string&gt; KeysWithPrefix(String prefix)
        {
            Queue&lt;String&gt; results = new Queue&lt;String&gt;();
            Node x = Get(root, prefix, 0);
            Collect(x, new StringBuilder(prefix), results);
            return results;
        }

        private void Collect(Node x, StringBuilder prefix, Queue&lt;String&gt; results)
        {
            if (x == null) return;
            if (x.val != null) results.Enqueue(prefix.ToString());
            for (char c = (char)0; c &lt; R; c&#43;&#43;)
            {
                prefix.Append(c);
               Collect(x.next[c], prefix, results);
                prefix.Remove(prefix.Length - 1, 1);
            }
        }

        /// &lt;summary&gt;
        /// 返回符号表中与pattern匹配的所有键，
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;pattern&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public IEnumerable&lt;String&gt; KeysThatMatch(String pattern)
        {
            Queue&lt;String&gt; results = new Queue&lt;String&gt;();
            Collect(root, new StringBuilder(), pattern, results);
            return results;
        }

        private void Collect(Node x, StringBuilder prefix, String pattern, Queue&lt;String&gt; results)
        {
            if (x == null) return;
            int d = prefix.Length;
            if (d == pattern.Length &amp;&amp; x.val != null)
                results.Enqueue(prefix.ToString());
            if (d == pattern.Length)
                return;
            char c = pattern[d];
            if (c == &#39;.&#39;)
            {
                for (char ch = (char)0; ch &lt; R; ch&#43;&#43;)
                {
                    prefix.Append(ch);
                    Collect(x.next[ch], prefix, pattern, results);
                    prefix.Remove(prefix.Length - 1, 1);
                }
            }
            else
            {
                prefix.Append(c);
                Collect(x.next[c], prefix, pattern, results);
                prefix.Remove(prefix.Length - 1, 1);
            }
        }

        /// &lt;summary&gt;
        /// 返回符号表中的字符串，该字符串是{@code query}的最长前缀，
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;query&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public String LongestPrefixOf(String query)
        {
            if (query == null) throw new ArgumentException(&#34;argument to longestPrefixOf() is null&#34;);
            int length = LongestPrefixOf(root, query, 0, -1);
            if (length == -1) return null;
            else return query.Substring(0, length);
        }

        /// &lt;summary&gt;
        /// 返回子菜单中最长字符串键的长度
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;x&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;query&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;d&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;length&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        private int LongestPrefixOf(Node x, String query, int d, int length)
        {
            if (x == null) return length;
            if (x.val != null) length = d;
            if (d == query.Length) return length;
            char c = query[d];
            return LongestPrefixOf(x.next[c], query, d &#43; 1, length);
        }

        /// &lt;summary&gt;
        /// 如果存在key，则从集合中删除key
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;key&#34;&gt;&lt;/param&gt;
        public void Delete(String key)
        {
            if (key == null) throw new ArgumentException(&#34;argument to delete() is null&#34;);
            root = Delete(root, key, 0);
        }

        private Node Delete(Node x, String key, int d)
        {
            if (x == null) return null;
            if (d == key.Length)
            {
                if (x.val != null) n--;
                x.val = null;
            }
            else
            {
                char c = key[d];
                x.next[c] = Delete(x.next[c], key, d &#43; 1);
            }

            // remove subtrie rooted at x if it is completely empty
            if (x.val != null) return x;
            for (int c = 0; c &lt; R; c&#43;&#43;)
                if (x.next[c] != null)
                    return x;
            return null;
        }

    }
```

测试

```c#
        [TestMethod]
        public void TrieSTFixture()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\shellsST.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                List&lt;string&gt; lines = new List&lt;string&gt;();
                while (!stream.EndOfStream)
                {
                    lines.AddRange(stream.ReadLine().Trim().Split(&#39; &#39;));
                }

                TrieST&lt;int&gt; st=new TrieST&lt;int&gt;();
                for (int i=0;i&lt;lines.Count;i&#43;&#43;)
                {
                    var key = lines[i];
                    st.Put(key,i);
                }
                CodeTimer.Time(&#34;TrieST select:&#34;, 1, () =&gt;
                {
                    var value = st.LongestPrefixOf(&#34;s&#34;);
                   Console.WriteLine(value??&#34;null&#34;);
                });


                foreach (var value in st.KeysWithPrefix(&#34;s&#34;))
                {
                    Console.WriteLine(value);
                }
                

            }
```

字母表的大小为R, 在一棵由N 个随机键构造的单词查找树中，未命中查找平均所需检查的结点数量为~logN。

每个节点都有R个链接，对应可能出现的字符。造成查找浪费。

### 三向单词查找树

三向单词查找树(TST) 。在三向单词查找树中，每个结点都含有一个字符、三条链接和一个值。这三条链接分别对应着当前字母小于、等于和大于结点字母的所有键。

如果遇到了一个空链接或者当键结束时结点的值为空，那么查找未命中；如果键结束时结点的值非空则查找命中。在插入一个新键时，首先进行查找，然后和在单词查找树一样，在树中补全键末尾的所有结点。

![image-20210505214210607](/algorithm/image-20210505214210607.png)

![image-20210505214236381](/algorithm/image-20210505214236381.png)

```c#
    public class TST&lt;Value&gt;
    {
        private int n;              // 树的大小
        private Node&lt;Value&gt; root;   // 树的根节点

        public  class Node&lt;Value&gt;
        {
            public char c;                        // 字符
            public Node&lt;Value&gt; left, mid, right;  // 左中右子三向单词查找树
            public Value val;                     // 和宇符串相关联的值
        }

        /// &lt;summary&gt;
        /// 初始化一个空的字符串表
        /// Initializes an empty string symbol table.
        /// &lt;/summary&gt;
        public TST()
        {
        }

        
        public int Size()
        {
            return n;
        }

        
        public bool Contains(string key)
        {
            if (key == null)
            {
                throw new ArgumentNullException(&#34;argument to contains() is null&#34;);
            }
            return Get(key) != null;
        }

        
        public Value Get(string key)
        {
            if (key == null)
            {
                throw new ArgumentNullException(&#34;calls get() with null argument&#34;);
            }
            if (key.Length == 0) throw new ArgumentException(&#34;key must have length &gt;= 1&#34;);
            Node&lt;Value&gt; x = Get(root, key, 0);
            if (x == null) return default(Value);
            return x.val;
        }

        // return subtrie corresponding to given key
        private Node&lt;Value&gt; Get(Node&lt;Value&gt; x, String key, int d)
        {
            if (x == null) return null;
            if (key.Length == 0) throw new ArgumentException(&#34;key must have length &gt;= 1&#34;);
            char c = key[d];
            if (c &lt; x.c) return Get(x.left, key, d);
            else if (c &gt; x.c) return Get(x.right, key, d);
            else if (d &lt; key.Length - 1) return Get(x.mid, key, d &#43; 1);
            else return x;
        }

        public void Put(String key, Value val)
        {
            if (key == null)
            {
                throw new ArgumentNullException(&#34;calls put() with null key&#34;);
            }
            if (!Contains(key)) n&#43;&#43;;
            else if (val == null) n--;       // delete existing key
            root = Put(root, key, val, 0);
        }

        private Node&lt;Value&gt; Put(Node&lt;Value&gt; x, String key, Value val, int d)
        {
            char c = key[d];
            if (x == null)
            {
                x = new Node&lt;Value&gt;();
                x.c = c;
            }
            if (c &lt; x.c) x.left = Put(x.left, key, val, d);
            else if (c &gt; x.c) x.right = Put(x.right, key, val, d);
            else if (d &lt; key.Length - 1) x.mid = Put(x.mid, key, val, d &#43; 1);
            else x.val = val;
            return x;
        }

        /// &lt;summary&gt;
        /// 返回符号表中的字符串，该字符串是query的最长前缀，
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;query&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public String LongestPrefixOf(String query)
        {
            if (query == null)
            {
                throw new ArgumentNullException(&#34;calls longestPrefixOf() with null argument&#34;);
            }
            if (query.Length == 0) return null;
            int length = 0;
            Node&lt;Value&gt; x = root;
            int i = 0;
            while (x != null &amp;&amp; i &lt; query.Length)
            {
                char c = query[i];
                if (c &lt; x.c) x = x.left;
                else if (c &gt; x.c) x = x.right;
                else
                {
                    i&#43;&#43;;
                    if (x.val != null) length = i;
                    x = x.mid;
                }
            }
            return query.Substring(0, length);
        }

        
        public IEnumerable&lt;string&gt; Keys()
        {
            Queue&lt;string&gt; queue = new Queue&lt;string&gt;();
            Collect(root, new StringBuilder(), queue);
            return queue;
        }

        /// &lt;summary&gt;
        /// 返回集合中以{@code prefix}开头的所有键。
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;prefix&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public IEnumerable&lt;String&gt; KeysWithPrefix(String prefix)
        {
            if (prefix == null)
            {
                throw new ArgumentNullException(&#34;calls keysWithPrefix() with null argument&#34;);
            }
            Queue&lt;String&gt; queue = new Queue&lt;String&gt;();
            Node&lt;Value&gt; x = Get(root, prefix, 0);
            if (x == null) return queue;
            if (x.val != null) queue.Enqueue(prefix);
            Collect(x.mid, new StringBuilder(prefix), queue);
            return queue;
        }

        /// &lt;summary&gt;
        /// subtrie中所有以x为根且具有给定前缀的键
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;x&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;prefix&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;queue&#34;&gt;&lt;/param&gt;
        private void Collect(Node&lt;Value&gt; x, StringBuilder prefix, Queue&lt;String&gt; queue)
        {
            if (x == null) return;
            Collect(x.left, prefix, queue);
            if (x.val != null) queue.Enqueue(prefix.ToString() &#43; x.c);
            Collect(x.mid, prefix.Append(x.c), queue);
            prefix.Remove(prefix.Length - 1, 1);
            Collect(x.right, prefix, queue);
        }


        /// &lt;summary&gt;
        /// 返回符号表中与{@code pattern}匹配的所有键，其中。 符号被视为通配符。
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;pattern&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public IEnumerable&lt;String&gt; KeysThatMatch(String pattern)
        {
            Queue&lt;String&gt; queue = new Queue&lt;String&gt;();
            Collect(root, new StringBuilder(), 0, pattern, queue);
            return queue;
        }

        private void Collect(Node&lt;Value&gt; x, StringBuilder prefix, int i, String pattern, Queue&lt;String&gt; queue)
        {
            if (x == null) return;
            char c = pattern[i];
            if (c == &#39;.&#39; || c &lt; x.c) Collect(x.left, prefix, i, pattern, queue);
            if (c == &#39;.&#39; || c == x.c)
            {
                if (i == pattern.Length - 1 &amp;&amp; x.val != null) queue.Enqueue(prefix.ToString() &#43; x.c);
                if (i &lt; pattern.Length - 1)
                {
                    Collect(x.mid, prefix.Append(x.c), i &#43; 1, pattern, queue);
                    prefix.Remove(prefix.Length - 1,1);
                }
            }
            if (c == &#39;.&#39; || c &gt; x.c) Collect(x.right, prefix, i, pattern, queue);
        }
    }
```

测试

```c#
        [TestMethod]
        public void TSTFixture()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\shellsST.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                List&lt;string&gt; lines = new List&lt;string&gt;();
                while (!stream.EndOfStream)
                {
                    lines.AddRange(stream.ReadLine().Trim().Split(&#39; &#39;));
                }

                TST&lt;int&gt; st = new TST&lt;int&gt;();
                for (int i = 0; i &lt; lines.Count; i&#43;&#43;)
                {
                    var key = lines[i];
                    st.Put(key, i);
                }



                CodeTimer.Time(&#34;TST select:&#34;, 1, () =&gt;
                {
                    var value = st.LongestPrefixOf(&#34;s&#34;);
                    Console.WriteLine(value ?? &#34;null&#34;);
                });


                foreach (var value in st.KeysWithPrefix(&#34;s&#34;))
                {
                    Console.WriteLine(value);
                }


            }
            //TST select:s
            //	Time Elapsed:		2ms
            //	Time Elapsed (one time):2ms
            //	CPU time:		0ns
            //	CPU time (one time):	0ns
            //	Gen 0: 			0
            //	Gen 1: 			0
            //	Gen 2: 			0
        }
```

由N 个平均长度为w 的字符串构造的三向单词查找树中的链接总数在3N 到3Nw 之间。

### 字符串符号表

![image-20210505221414878](/algorithm/image-20210505221414878.png)

## 子字符串查找

子宇符串查找： 给定一段长度为N 的文本和一个长度为M 的模式
( pattern) 字符串，在文本中找到一个和该模式相符的子字符串

![image-20210506121819503](/algorithm/image-20210506121819503.png)

子字符串查找有一个简单而使用广泛的暴力算法。虽然它在最坏情况下的运行时间与MN 成正比，但是在处理许多应用程序中的字符串时（除了一些变态的情况之外），它的实际运行时间一般
与M &#43; N 成正比。另外，它很好地利用了大多数计算机系统中标准的结构特性，因此即使是更加巧妙的算法也很难超越它经过优化后的版本的性能。

在19 80 年， M.O.Rabin 和R.M.Karp 使用散列开发出了一种与暴力算法儿乎一样简单但运行时间与M&#43;N 成正比的概率极高的算法。另外，它们的算法还可以扩展到二维的模式和文本中，这使
得它比其他算法更适用于图像处理。，

### 暴力子字符串查找

在最坏情况下，暴力子字符串查找算法在长度为N 的文本中查找长度为M 的模式需要-NM 次字符比较，

![image-20210506123035902](/algorithm/image-20210506123035902.png)

![image-20210506123050174](/algorithm/image-20210506123050174.png)

```c#
        public static int Search(string pat, string txt)
        {
            int M = pat.Length;
            int N = txt.Length;
            for (int i = 0; i &lt;= N - M; i&#43;&#43;)
            {
                int j;
                for (j = 0; j &lt; M; j&#43;&#43;)
                {
                    if(txt[i&#43;j]!=pat[j]) break;
                }

                if (j == M) return i;//找到匹配
            }

            return N;//未找到匹配
        }
```

另一种显式回退暴力算法

```c#
        public static int ShowBackSearch(string pat, string txt)
        {
            int j, M = pat.Length;
            int i, N = txt.Length;
            for (i = 0, j = 0; i &lt; N &amp;&amp; j &lt; M; i&#43;&#43;)
            {
                if (txt[i] == pat[j]) j&#43;&#43;;
                else
                {
                    i -= j;//显示回退
                    j = 0;
                }
            }
            if (j == M) return j - M;//找到匹配
            else return N;//未找到匹配
        }
```

### Knuth-Morris-Pratt

算法的基本思想是当出现不匹配时，就能知晓一部分文本的内容。

假设字母表中只有两个字符，查找的模式字符串为B A A A A A A A A A 。现在，假设已经匹配了模式中的5 个字符，第6 个字符匹配失败。当发现不匹配的字符时，可以知道文本中的前6 个字符肯定是BAAAAB ( 前5 个匹配，第6 个失败），文本指针现在指向的是末尾的字符B 。你可以观察到，这里不需要回退文本指针i , 因为正文中的前4 个字符都是A,均与模式的第一个字符不匹配。KMP 算法的主要思想是提前判断如何重新开始查找，而这种判断只取决于模式本身。

![image-20210506130433532](/algorithm/image-20210506130433532.png)

使用一个数组dfa[] []来记录匹配失败时模式指针j 应该回退多远。对于每个字符c , 在比较了c 和pat.charAt(j) 之后， dfa[c] [j]表示的是应该和下个文本字符比较的模式字符的位置。在查找中， dfs\[txt.charAt(i)][j] 是在比较了txt.charAt(i) 和pat.charAt(j) 之后应该和txt.charAt(i&#43;1) 比较的模式字符位置。

![image-20210507101716290](/algorithm/image-20210507101716290.png)

![image-20210507101747968](/algorithm/image-20210507101747968.png)

![image-20210507102007240](/algorithm/image-20210507102007240.png)

要体验在DFA 中的子字符串查找操作，你可以先想象一下它所完成的两件最简单的任务。在
查找过程的开始，从文本的开头进行查找，起始状态为0 。它停留在0 状态并扫描文本，直到找到一个和模式的首字母相同的字符。这时它移动到下一个状态并开始运行。在这个过程的最后，当它找到一个匹配时，它会不断地匹配模式中的字符与文本，自动机的状态会不断前进直到状态M 。

#### 构造dfa [] [] 数组



![image-20210507102848349](/algorithm/image-20210507102848349.png)

![image-20210507103140460](/algorithm/image-20210507103140460.png)

```c#
   public class KMP
    {
        private  int R;       // the radix
        private  int m;       // length of pattern
        private int[,] dfa;       // KMP[c][j]

        
        public KMP(string pat)
        {
            this.R = 256;
            this.m = pat.Length;

            // build DFA from pattern
            //由模式宇符串构造DFA
            dfa = new int[R , m];
            dfa[pat[0],0] = 1;
            for (int x = 0, j = 1; j &lt; m; j&#43;&#43;)//j是pat的序号
            {
                //计算dfa[][j]
                for (int c = 0; c &lt; R; c&#43;&#43;)
                    dfa[c,j] = dfa[c,x];     // 复制匹配失败情况下的值 
                dfa[pat[j],j] = j &#43; 1;   // 设置匹配成功情况下的值
                x = dfa[pat[j],x];     // 更新重启状态 
            }
        }

        public KMP(char[] pattern, int R)
        {
            this.R = R;
            this.m = pattern.Length;

            // build DFA from pattern
            int m = pattern.Length;
            dfa = new int[R,m];
            dfa[pattern[0],0] = 1;
            for (int x = 0, j = 1; j &lt; m; j&#43;&#43;)
            {
                for (int c = 0; c &lt; R; c&#43;&#43;)
                    dfa[c,j] = dfa[c,x];     // Copy mismatch cases. 
                dfa[pattern[j],j] = j &#43; 1;      // Set match case. 
                x = dfa[pattern[j],x];        // Update restart state. 
            }
        }

        public int Search(string txt)
        {

            // simulate operation of DFA on text
            //模拟DFA处理文本txt 时的操作
            int n = txt.Length;
            int i, j;
            for (i = 0, j = 0; i &lt; n &amp;&amp; j &lt; m; i&#43;&#43;)
            {
                j = dfa[txt[i],j];
            }
            if (j == m) return i - m;    // 找到(到达模式字符串的结尾)
            return n;                    // 未找到(到达文本字符串的结尾)
        }

        public int Search(char[] text)
        {

            // simulate operation of DFA on text
            int n = text.Length;
            int i, j;
            for (i = 0, j = 0; i &lt; n &amp;&amp; j &lt; m; i&#43;&#43;)
            {
                j = dfa[text[i],j];
            }
            if (j == m) return i - m;    // found
            return n;                    // not found
        }
    }
```

测试

```c#
        [TestMethod]
        public void KMPTest()
        {
            string[] args =new []
            {
                &#34;abracadabra abacadabrabracabracadabrabrabracad&#34;,
                &#34;rab abacadabrabracabracadabrabrabracad&#34;,
                &#34;bcara abacadabrabracabracadabrabrabracad&#34;,
                &#34;rabrabracad abacadabrabracabracadabrabrabracad&#34;,
                &#34;abacad abacadabrabracabracadabrabrabracad&#34;,
            };

            foreach (var arg in args)
            {
                Console.WriteLine(&#34;/*********************Start*********************/&#34;);
                string pat=  arg.Split(&#39; &#39;)[0];
              string txt= arg.Split(&#39; &#39;)[1];
              char[] pattern = pat.ToArray();
              char[] text = txt.ToArray();
              KMP kmp1 = new KMP(pat);
              int offset1 = kmp1.Search(txt);

              KMP kmp2 = new KMP(pattern, 256);
              int offset2 = kmp2.Search(text);

              // print results
              Console.WriteLine(&#34;text:    &#34;);
              Console.WriteLine(txt);

              Console.WriteLine(&#34;pattern: &#34;);
              for (int i = 0; i &lt; offset1; i&#43;&#43;)
                  Console.Write(&#34; &#34;);
              Console.WriteLine(pat);

              Console.WriteLine(&#34;pattern: &#34;);
              for (int i = 0; i &lt; offset2; i&#43;&#43;)
                  Console.Write(&#34; &#34;);
              Console.WriteLine(pat);
              Console.WriteLine(&#34;/*********************End*********************/&#34;);
            }
        }
```

### Boyer-Moore

### 正则表达式


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2021/05/algorithm11/  

