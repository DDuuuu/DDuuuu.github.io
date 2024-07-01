# 算法 堆 优先队列


## 堆

堆：**当一棵二叉树的每个结点都大于等于它的两个子结点时，它被称为堆有序。**

命题O:根结点是堆有序的二叉树中的**最大结点**

二叉堆：一组能够用堆有序的完全的二叉树排序的元素，并在数组中按照层级存储（&lt;font color=&#39;red&#39;&gt;**不使用数组的第一个位置**&lt;/font&gt;）。

![1605654615253](/algorithm/1605654615253.png)

命题P:一棵大小为N的完全二叉树的高度为lgN。

### 堆有序上浮

由下至上的堆有序（上浮）实现

```c#
private void swim(int k)
{
	while(k&gt;1&amp;&amp;less(k/2,k))
	{
		exch(k/2,k);
		k=k/2;
	}
}
```

### 堆有序下沉

由上至下的堆有序（下沉）实现。

命题R:用下沉操作由N个元素构造堆只需少于2N次比较以及小于N次交换。

```c#
private void sink(int k)
{
	while(2*k &lt;= N)
	{
		int j=2*k;
		if(j&lt;N&amp;&amp;less(j,j&#43;1)) j&#43;&#43;;//找到子结点中较大的那个
		if(!less(k,j)) break;，//根节点应该比较大的那个大，否则就需要交换。
		exch(k,j);
		k=j;//一直交换到最底层
	}
}
```

![1605655443952](/algorithm/1605655443952.png)

### 堆有序排序

![1606890175485](/algorithm/1606890175485.png)

```c#
    public class HeapSortNet&lt;TItem&gt; where TItem : IComparable&lt;TItem&gt;
    {
		///注意这边有一个Bug忽略了数组的第一个元素。下面的优先队列消除了这个Bug
        public static void Sort(TItem[] items)
        {
            int n = items.Length;
            // 构造堆
            for (int k = n / 2; k &gt;= 1; k--)
                Sink(items, k, n);//K的值需要比N的大
            // 堆排序
            while (n &gt; 1)
            {
                Exch(items, 1, n--);//将最大的放到最后
                Sink(items, 1, n);//将最大之前的那个找出来放到第一个
            }
        }
        /// &lt;summary&gt;
        /// 下沉
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;items&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;k&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;n&#34;&gt;&lt;/param&gt;
        private static void Sink(TItem[] items, int k, int n)
        {
            while (2 * k &lt;= n)
            {
                int j = 2 * k;
                if (j &lt; n &amp;&amp; Less(items, j, j &#43; 1)) j&#43;&#43;;
                if (!Less(items, k, j)) break;
                Exch(items, k, j);
                k = j;
            }
        }
        private static bool Less(TItem[] items, int i, int j)
        {
            return items[i - 1].CompareTo(items[j - 1]) &lt; 0;
        }

        private static void Exch(TItem[] items, int i, int j)
        {
            TItem swap = items[i - 1];
            items[i - 1] = items[j - 1];
            items[j - 1] = swap;
        }
    }
```

![1606890292948](/algorithm/1606890292948.png)

命题S:将N个元素排序，堆排序只需要少于（2NlgN&#43;2N）次比较（以及一半次数的交换）

![1606891568103](/algorithm/1606891568103.png)

**改进**：大多数在下沉排序期间重新插入堆的元素会被直接加入到堆底。

Floyd在1964年观察发现，可以通过免去检查元素是否到达正确位置来节省时间，

下沉中总是直接提升较大的子节点直至到达堆底，然后再使元素上浮到正确的位置，可以将比较次数减少一半

唯一能够同时最优地利用空间和时间的方法，最坏情况下也能保证~2NlgN次比较和恒定的额外空间。在嵌入式或底成本的移动设备中很流行，无法利用缓存，很少于其他元素进行比较。

## 优先队列

就是用堆的方式排列的数组。支持查找最大最小元素和插入元素

### 最大优先队列

最大优先队列：支持删除最大元素和插入元素

![1605653842039](/algorithm/1605653842039.png)

![1605653982570](/algorithm/1605653982570.png)

基于堆的优先队列：存储在数组[1……N]中，**数组[0]没有使用**。

![1605655534610](/algorithm/1605655534610.png)

命题Q:对于一个含有N个元素的基于堆的优先队列，插入元素操作只需要不超过（lgN&#43;1）次比较，删除最大元素的操作需要不超过2lgN次比较。

```c#
    /// &lt;summary&gt;
    /// 优先队列
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TItem&#34;&gt;&lt;/typeparam&gt;
    public interface IMaxHeapQueue&lt;TItem&gt; : IEnumerable&lt;TItem&gt;, IDisposable
    {
        bool IsEmpty { get; }
        int Length { get; }
        void Insert(TItem x);
        TItem DeleteMax();
        TItem Max();
    }
```

​	

```c#
    public class MaxPQNet&lt;TItem&gt; : IMaxHeapQueue&lt;TItem&gt; where TItem : IComparable&lt;TItem&gt;
    {
        private TItem[] _items;
        private int _length;
        private IComparer&lt;TItem&gt; _comparer;
        public MaxPQNet(int capacity)
        {
            _items=new TItem[capacity&#43;1];
            _length = 0;
        }

        public MaxPQNet():this(1)
        {

        }

        public MaxPQNet(int capacity, IComparer&lt;TItem&gt; comparer):this(capacity)
        {
            _comparer = comparer;
        }


        public MaxPQNet(TItem[]  items)
        {
            _length = items.Length;
            _items = new TItem[items.Length &#43; 1];
            for (int i = 0; i &lt; _length; i&#43;&#43;)
                _items[i &#43; 1] = items[i];
            for (int k = _length / 2; k &gt;= 1; k--)
                sink(k);
        }

        private void sink(int k)
        {
            while (k*2&lt;=_length)
            {
                int j = 2 * k;
                if (j &lt; _length &amp;&amp; less(j, j &#43; 1)) j&#43;&#43;;
                if(!less(k,j)) break;
                exch(k,j);
                k = j;
            }
        }
        private void swim(int k)
        {
            while (k &gt; 1 &amp;&amp; less(k / 2, k))
            {
                exch(k,k/2);
                k = k / 2;
            }
        }
        private bool less(int i, int j)
        {
            if (_comparer == null)
            {
                return _items[i].CompareTo(_items[j]) &lt; 0;
            }
            else
            {
                return _comparer.Compare(_items[i], _items[j]) &lt; 0;
            }
        }

        private void exch(int i, int j)
        {
            var item = _items[i];
            _items[i] = _items[j];
            _items[j] = item;
        }

        private void resize(int capacity)
        {
            if(capacity&lt;_length)
                throw new ConfigurationException(&#34;this capacity is error less than Length&#34;);
            var tempitems = new TItem[capacity];
            for (int i = 1; i &lt; _length&#43;1; i&#43;&#43;)
            {
                tempitems[i] = _items[i];
            }

            _items = tempitems;
        }

        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var temitem = new TItem[_length-1];
                for (int i = 0; i &lt; _length; i&#43;&#43;)
                {
                    temitem[i] = _items[i &#43; 1];
                }
                var temmaxpq=new MaxPQ&lt;TItem&gt;(temitem);

                while (!temmaxpq.IsEmpty())
                {
                    yield return temmaxpq.DelMax();
                }
            }
            

        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }
        protected  virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                if (!IsEmpty)
                {
                    for (int i = 1; i &lt; _length&#43;1; i&#43;&#43;)
                    {
                        _items[i] = default;
                    }
                }
                
            }
        }
        public void Dispose()
        {
            Dispose(true);
        }

        public bool IsEmpty =&gt; Length == 0;
        public int Length =&gt; _length;
        public void Insert(TItem item)
        {
            if(_length==_items.Length-1) resize(_length*2);
            _items[&#43;&#43;_length] = item;
            swim(_length);
        }

        public TItem DeleteMax()
        {
            if (IsEmpty) return default;
            var item = _items[1];
            exch(1,_length--);//把最小值交换到第一个重新堆排序
            _items[_length &#43; 1] = default;
            sink(1);
            if(_length&gt;0&amp;&amp;_length==(_items.Length-1)/4) resize(_items.Length/2);
            return item;
        }

        public TItem Max()
        {
            if (IsEmpty)
                return default;
            return _items[1];
        }
    }
```

堆上优先队列

![1605953474567](/algorithm/1605953474567.png)

### 最小优先队列

根结点是最小值，其他相同

```c#
    public class MinPQNet&lt;TItem&gt; : IMinHeapQueue&lt;TItem&gt; where TItem : IComparable&lt;TItem&gt;
    {
        private TItem[] _items;
        private int _length;
        private IComparer&lt;TItem&gt; _comparer;
        public MinPQNet(int capacity)
        {
            _items = new TItem[capacity &#43; 1];
            _length = 0;
        }

        public MinPQNet() : this(1)
        {

        }

        public MinPQNet(int capacity, IComparer&lt;TItem&gt; comparer) : this(capacity)
        {
            _comparer = comparer;
        }


        public MinPQNet(TItem[] items)
        {
            _length = items.Length;
            _items = new TItem[items.Length &#43; 1];
            for (int i = 0; i &lt; _length; i&#43;&#43;)
                _items[i &#43; 1] = items[i];
            for (int k = _length / 2; k &gt;= 1; k--)
                sink(k);
        }

        private void sink(int k)
        {
            while (k * 2 &lt;= _length)
            {
                int j = 2 * k;
                if (j &lt; _length &amp;&amp; greater(j, j &#43; 1)) j&#43;&#43;;
                if (!greater(k, j)) break;
                exch(k, j);
                k = j;
            }
        }
        private void swim(int k)
        {
            while (k &gt; 1 &amp;&amp; greater(k / 2, k))
            {
                exch(k, k / 2);
                k = k / 2;
            }
        }

        private bool greater(int i, int j)
        {
            if (_comparer == null)
            {
                return _items[i].CompareTo(_items[j]) &gt; 0;
            }
            else
            {
                return _comparer.Compare(_items[i], _items[j]) &gt; 0;
            }
        }

        private void exch(int i, int j)
        {
            var item = _items[i];
            _items[i] = _items[j];
            _items[j] = item;
        }

        private void resize(int capacity)
        {
            if (capacity &lt; _length)
                throw new ConfigurationException(&#34;this capacity is error less than Length&#34;);
            var tempitems = new TItem[capacity];
            for (int i = 1; i &lt; _length &#43; 1; i&#43;&#43;)
            {
                tempitems[i] = _items[i];
            }

            _items = tempitems;
        }

        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var temitem = new TItem[_length - 1];
                for (int i = 0; i &lt; _length; i&#43;&#43;)
                {
                    temitem[i] = _items[i &#43; 1];
                }
                var temmaxpq = new MinPQ&lt;TItem&gt;(temitem);

                while (!temmaxpq.IsEmpty())
                {
                    yield return temmaxpq.DelMin();
                }
            }


        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }
        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                if (!IsEmpty)
                {
                    for (int i = 1; i &lt; _length &#43; 1; i&#43;&#43;)
                    {
                        _items[i] = default;
                    }
                }

            }
        }
        public void Dispose()
        {
            Dispose(true);
        }

        public bool IsEmpty =&gt; Length == 0;
        public int Length =&gt; _length;
        public void Insert(TItem item)
        {
            if (_length == _items.Length - 1) resize(_length * 2);
            _items[&#43;&#43;_length] = item;
            swim(_length);
        }

        public TItem DeleteMin()
        {
            if (IsEmpty) return default;
            var item = _items[1];
            exch(1, _length--);//把最小值交换到第一个重新堆排序
            _items[_length &#43; 1] = default;
            sink(1);
            if (_length &gt; 0 &amp;&amp; _length == (_items.Length - 1) / 4) resize(_items.Length / 2);
            return item;
        }

        public TItem Min()
        {
            if (IsEmpty)
                return default;
            return _items[1];
        }
    }
```

### 索引优先队列

可以快速访问已经插入队列中的项。

不会改变插入队列中项的数据结构

命题Q(续):在一个大小为N的索引优先队列中，插入(insert)元素，改变优先级，删除，删除最小元素和删除最大元素操作所需要的比较次数和LogN成正比。

![1606868282293](/algorithm/1606868282293.png)

示例为最大索引优先队列，简单改写就可以变成最小索引优先队列

```c#
   /// &lt;summary&gt;
    /// 增加一个key，key对应一个索引，实际交换排序是控制key对应的索引
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TItem&#34;&gt;&lt;/typeparam&gt;
    public class KeyMaxPQNet&lt;TItem&gt; : IMaxKeyHeap&lt;TItem&gt; where TItem:IComparable&lt;TItem&gt;
    {

        private int _length;
        private int _capacity;
        /// &lt;summary&gt;
        /// key对应的索引
        /// &lt;/summary&gt;
        private int[] _keysIndex;
        /// &lt;summary&gt;
        /// 索引对应的key
        /// &lt;/summary&gt;
        private int[] _indexesKey;
        /// &lt;summary&gt;
        /// key对应的item
        /// &lt;/summary&gt;
        private TItem[] _items;
        public bool IsEmpty =&gt; _length == 0;

        public int Length =&gt; _length;

        public int Capacity =&gt; _capacity;

        public KeyMaxPQNet(TItem[] items)
        {
            if (items == null || !items.Any()) throw new ArgumentNullException(&#34;items is empty&#34;);
            _length = 0;
            _capacity = items.Length;
            _items = new TItem[_capacity &#43; 1];
            _keysIndex = new int[_capacity &#43; 1];
            _indexesKey = new int[_capacity &#43; 1];
            for (int i = 0; i &lt;= _capacity; i&#43;&#43;)
            {
                _keysIndex[i] = -1; //初始化所有key对应的index都是-1，index对应的key都是0
            }
            foreach (var item in items)
            {
                this.Insert(item);
            }
        }

        public KeyMaxPQNet(int capacity)
        {
            if(capacity&lt;0) throw new ArgumentException(&#34;this argument capacity is error and can not less zero&#34;);
            this._capacity = capacity;
            _length = 0;
            _items=new TItem[capacity&#43;1];
            _keysIndex=new int[capacity&#43;1];
            _indexesKey=new int[capacity&#43;1];
            for (int i = 0; i &lt;= capacity;i&#43;&#43;)
            {
                _keysIndex[i] = -1;//初始化所有key对应的index都是-1，index对应的key都是0
            }
        }

        public void Insert(TItem item)
        {
            var key = _length&#43;1;
            Insert(key,item);
        }

        public void Insert(int key, TItem item)
        {
           validateKey(key);
           if (!Contains(key))
            throw new ArgumentException(&#34;this index has existed&#34;);
           _length&#43;&#43;;
           _keysIndex[key] = _length;
           _indexesKey[_length] = key;
           _items[key] = item;
           swim(_length);
        }
        /// &lt;summary&gt;
        /// if Empty,throw exception
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public TItem DeleteMax()
        {
            if (IsEmpty) throw new NullReferenceException(&#34;this is empty queue&#34;);
            int max = _indexesKey[1];
            var item = _items[max];
            exch(1,_length--);
            sink(1);
            _keysIndex[max] = -1;
            _items[max] = default;
            _indexesKey[_length &#43; 1] = -1;
            return item;
        }

        public void Delete(int key)
        {
            validateKey(key);
            if(!Contains(key)) throw new ArgumentException(&#34;this index is not in this queue&#34;);
            var k = _keysIndex[key];
            exch(k,_length--);//这边已经排除最后一项
            swim(k);
            sink(k);
            _items[key] = default;
            _keysIndex[key] = -1;

        }
        public void Replace(int key, TItem item)
        {
            validateKey(key);
            if(!Contains(key)) throw new NullReferenceException(&#34;this key not in queue&#34;);
            _items[key] = item;
            swim(_keysIndex[key]);
            sink(_keysIndex[key]);
        }


        public TItem KeyOf(int key)
        {
            validateKey(key);
            if(!Contains(key)) return default;
            return _items[key];

        }

        public int MaxKey()
        {
            if(IsEmpty)
                throw new ArgumentException(&#34;this is null queue&#34;);
            return _indexesKey[1];
        }

        public TItem Max()
        {
            if(IsEmpty)
                throw new ArgumentException(&#34;this is null queue&#34;);
            return _items[_indexesKey[1]];
        }
        public bool Contains(int key)
        {
            validateKey(key);
            return _keysIndex[key] != -1;
        }

        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var queue=new KeyMaxPQNet&lt;TItem&gt;(_items);
                for (int i = 1; i &lt;= _items.Length; i&#43;&#43;)
                {
                    yield return queue.DeleteMax();
                }

            }
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
           return  GetEnumerator();
        }

        public void Dispose()
        {
            Dispose(true);
        }

        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                while (!IsEmpty)
                {
                    DeleteMax();
                }
            }
        }


        private bool less(int indexi,int indexj)
        {
            return _items[_indexesKey[indexi]].CompareTo(_items[_indexesKey[indexj]]) &lt; 0;
        }
        private void validateKey(int key)
        {
            if (key &lt; 0 || key &gt; _capacity) throw new ArgumentException(&#34;this key is invalid&#34;);
        }

        private void swim(int index)
        {
            while (index&gt;1 &amp;&amp; less(index/2,index))
            {
                exch(index, index / 2);
                index = index / 2;
            }
        }

        private void sink(int index)
        {
            while (2 * index &lt;= _length)
            {
                int indexj = 2 * index;
                if (indexj &lt; _length &amp;&amp; less(indexj, indexj &#43; 1)) indexj&#43;&#43;;
                if (!less(index, indexj)) break;
                exch(index, indexj);
                index = indexj;
            }
        }

        private void exch(int indexi, int indexj)
        {
            //不需要交换实际数组只需要交换序好对应的key，和key对应的序号
            int swap = _indexesKey[indexi];
            _indexesKey[indexi] = _indexesKey[indexj];
            _indexesKey[indexj] = swap;
            
            _keysIndex[_indexesKey[indexi]] = indexi;
            _keysIndex[_indexesKey[indexj]] = indexj;
        }

    }
}
```

```c#
/// &lt;summary&gt;
    /// 增加一个key，key对应一个索引，实际交换排序是控制key对应的索引
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;TItem&#34;&gt;&lt;/typeparam&gt;
    public class KeyMinPQNet&lt;TItem&gt; : IMinKeyHeap&lt;TItem&gt; where TItem : IComparable&lt;TItem&gt;
    {
        private int _length;
        private int _capacity;
        /// &lt;summary&gt;
        /// key对应的索引
        /// &lt;/summary&gt;
        private int[] _keysIndex;
        /// &lt;summary&gt;
        /// 索引对应的key
        /// &lt;/summary&gt;
        private int[] _indexesKey;
        /// &lt;summary&gt;
        /// key对应的item
        /// &lt;/summary&gt;
        private TItem[] _items;
        public bool IsEmpty =&gt; _length == 0;

        public int Length =&gt; _length;

        public int Capacity =&gt; _capacity;

        public KeyMinPQNet(TItem[] items)
        {
            if (items == null || !items.Any()) throw new ArgumentNullException(&#34;items is empty&#34;);
            _length = 0;
            _capacity = items.Length;
            _items = new TItem[_capacity &#43; 1];
            _keysIndex = new int[_capacity &#43; 1];
            _indexesKey = new int[_capacity &#43; 1];
            for (int i = 0; i &lt;= _capacity; i&#43;&#43;)
            {
                _keysIndex[i] = -1; //初始化所有key对应的index都是-1，index对应的key都是0
            }
            foreach (var item in items)
            {
                this.Insert(item);
            }
        }

        public KeyMinPQNet(int capacity)
        {
            if (capacity &lt; 0) throw new ArgumentException(&#34;this argument capacity is error and can not less zero&#34;);
            this._capacity = capacity;
            _length = 0;
            _items = new TItem[capacity &#43; 1];
            _keysIndex = new int[capacity &#43; 1];
            _indexesKey = new int[capacity &#43; 1];
            for (int i = 0; i &lt;= capacity; i&#43;&#43;)
            {
                _keysIndex[i] = -1;//初始化所有key对应的index都是-1，index对应的key都是0
            }
        }

        public void Insert(TItem item)
        {
            var key = _length &#43; 1;
            Insert(key, item);
        }

        public void Insert(int key, TItem item)
        {
            validateKey(key);
            if (!Contains(key))
                throw new ArgumentException(&#34;this index has existed&#34;);
            _length&#43;&#43;;
            _keysIndex[key] = _length;
            _indexesKey[_length] = key;
            _items[key] = item;
            swim(_length);
        }
        /// &lt;summary&gt;
        /// if Empty,throw exception
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public TItem DeleteMin()
        {
            if (IsEmpty) throw new NullReferenceException(&#34;this is empty queue&#34;);
            int max = _indexesKey[1];
            var item = _items[max];
            exch(1, _length--);
            sink(1);
            _keysIndex[max] = -1;
            _items[max] = default;
            _indexesKey[_length &#43; 1] = -1;
            return item;
        }

        public void Delete(int key)
        {
            validateKey(key);
            if (!Contains(key)) throw new ArgumentException(&#34;this index is not in this queue&#34;);
            var k = _keysIndex[key];
            exch(k, _length--);//这边已经排除最后一项
            swim(k);
            sink(k);
            _items[key] = default;
            _keysIndex[key] = -1;

        }
        public void Replace(int key, TItem item)
        {
            validateKey(key);
            if (!Contains(key)) throw new NullReferenceException(&#34;this key not in queue&#34;);
            _items[key] = item;
            swim(_keysIndex[key]);
            sink(_keysIndex[key]);
        }


        public TItem KeyOf(int key)
        {
            validateKey(key);
            if (!Contains(key)) return default;
            return _items[key];

        }

        public int MinKey()
        {
            if (IsEmpty)
                throw new ArgumentException(&#34;this is null queue&#34;);
            return _indexesKey[1];
        }

        public TItem Min()
        {
            if (IsEmpty)
                throw new ArgumentException(&#34;this is null queue&#34;);
            return _items[_indexesKey[1]];
        }
        public bool Contains(int key)
        {
            validateKey(key);
            return _keysIndex[key] != -1;
        }

        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var queue = new KeyMinPQNet&lt;TItem&gt;(_items);
                for (int i = 1; i &lt;= _items.Length; i&#43;&#43;)
                {
                    yield return queue.DeleteMin();
                }

            }
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        public void Dispose()
        {
            Dispose(true);
        }

        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                while (!IsEmpty)
                {
                    DeleteMin();
                }
            }
        }


        private bool greater(int indexi, int indexj)
        {
            return _items[_indexesKey[indexi]].CompareTo(_items[_indexesKey[indexj]]) &gt; 0;
        }
        private void validateKey(int key)
        {
            if (key &lt; 0 || key &gt; _capacity) throw new ArgumentException(&#34;this key is invalid&#34;);
        }

        private void swim(int index)
        {
            while (index &gt; 1 &amp;&amp; greater(index / 2, index))
            {
                exch(index, index / 2);
                index = index / 2;
            }
        }

        private void sink(int index)
        {
            while (2 * index &lt;= _length)
            {
                int indexj = 2 * index;
                if (indexj &lt; _length &amp;&amp; greater(indexj, indexj &#43; 1)) indexj&#43;&#43;;
                if (!greater(index, indexj)) break;
                exch(index, indexj);
                index = indexj;
            }
        }

        private void exch(int indexi, int indexj)
        {
            //不需要交换实际数组只需要交换序好对应的key，和key对应的序号
            int swap = _indexesKey[indexi];
            _indexesKey[indexi] = _indexesKey[indexj];
            _indexesKey[indexj] = swap;

            _keysIndex[_indexesKey[indexi]] = indexi;
            _keysIndex[_indexesKey[indexj]] = indexj;
        }
    }
```

## 排序算法应用

**多向并归**

索引优先队列用例解决多向归并问题：它将多个有序的输入流归并成一个有序的输出流。

输入可能来自于多种科学仪器的输出（按时间排序）

多个音乐网站或电影网站的信息列表（名称或艺术家名字）

**商业交易**（按时间排序）

使用优先队列，无论输入有多长都可以把它们全部读入并排序

![1606887893355](/algorithm/1606887893355.png)

交易数据按照日期排序

商业计算

信息搜索

运筹学：调度问题，负载均衡

事件驱动模拟

数值计算：浮点数来进行百万次计算，一些数值的计算使用优先队列和排序来计算精确度，曲线下区域的面积，数值积分方法就是使用一个优先队列来存储一组小间隔中每段的近似精确度。积分的过程就是山区精确度最低的间隔并将其分为两半。

组合搜索：一组状态演化成另一组状态可能的步骤和步骤的优先级。

Prim算法和Dijkstra算法:

Kruskal算法:

霍夫曼压缩：

字符串处理：

## 排序特点

![1606894371071](/algorithm/1606894371071.png)

性质T:快速排序是最快的通用排序算法。

对原始数据类型使用（三向切分）快速排序，对引用类型使用归并排序。

命题U:平均来说，基于切分的选择算法的运行时间是线性级别的。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/12/algorithm4/  

