# 算法 包 栈 队列


## Bag（包）

背包：不支持删除元素的集合数据类型。

```C#
public interface IBag&lt;TItem&gt; : IEnumerable&lt;TItem&gt;, IDisposable
{
	bool IsEmpty { get; }
	int Length { get; }
	void Add(TItem item);
}
```

### 数组实现

```c#
public class ResizingArrayBagNet&lt;TItem&gt;:IBag&lt;TItem&gt;
{
	private TItem[] _items;
	private int _length;
	public ResizingArrayBagNet()
	{
		_items=new TItem[2];
		_length = 0;

	}
	public IEnumerator&lt;TItem&gt; GetEnumerator()
	{
		if (!IsEmpty)
		{
			for (var i = 0; i &lt; _length; i&#43;&#43;)
			{
				yield return _items[i];
			}
		}
	}

	IEnumerator IEnumerable.GetEnumerator()
	{
		return GetEnumerator();
	}

	public void Dispose()
	{
		throw new NotImplementedException();
	}

	protected virtual void Dispose(bool disposing)
	{
		if (disposing)
		{
			if (!IsEmpty)
			{
				for (var i = 0; i &lt; _length ; i&#43;&#43;)
				{
					_items[i] = default;
				}
			}
		}
	}

	public bool IsEmpty =&gt; _length == 0;
	public int Length =&gt; _length;
	public void Add(TItem item)
	{
		if(_length==_items.Length) resize(_length*2);
		_items[_length&#43;&#43;] = item;
	}

	private void resize(int capacity)
	{
		var temitems=new TItem[capacity];
		Array.Copy(_items,0,temitems,0,_length);
		_items = temitems;
	}
}
```

### 链表下压栈

```c#
public class LinkedBagNet&lt;TItem&gt; : IBag&lt;TItem&gt;
{
	private Node _first;
	private int _length;
	private class Node
	{
		public TItem Item { set; get; }
		public Node Next { set; get; }
	}

	public LinkedBagNet()
	{
		_length = 0;
		_first = default;
	}
	public IEnumerator&lt;TItem&gt; GetEnumerator()
	{
		if (!IsEmpty)
		{
			var temnode = _first;
			while (temnode != default)
			{
				var item = temnode.Item;
				temnode = temnode.Next;
				yield return item;
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
			if (!IsEmpty)
			{
				var temnode = _first;
				while (temnode != default)
				{
					temnode.Item = default;
					temnode = temnode.Next;
				}
			}
		}
	}

	public bool IsEmpty =&gt; _length == 0;
	public int Length =&gt; _length;
	public void Add(TItem item)
	{
		Node oldeNode = _first;
		_first=new Node();
		_first.Item = item;
		_first.Next = oldeNode;
		_length&#43;&#43;;
	}
}
```



## Stack（栈）

栈：后进先出。

游离：弹出的元素所在的引用必须要置空，不然没办法被垃圾回收器回收。

```c#
    public interface IStack&lt;TItem&gt; : IEnumerable&lt;TItem&gt;, IDisposable
    {
        bool IsEmpty { get; }
        int Length { get; }
        void Push(TItem item);
        TItem Pop();
    }
```

栈应用：算术表达式求值：

1. 将操作数压入操作数栈
2. 将运算符压入运算栈
3. 忽略左括号
4. 在遇到右括号时，弹出一个运算符，弹出所需要数量的操作数，并将运算符和操作数的运算结果压入操作数栈。



### 数组实现

```c#
    public class ResizingArrayStackNet&lt;TItem&gt; : IStack&lt;TItem&gt;
    {
        private TItem[] _items;
        private int _length;
        public ResizingArrayStackNet()
        {
            _items = new TItem[2];
            _length = 0;
        }

        public int Length =&gt; _length;

        public bool IsEmpty =&gt; _length == 0;


        private void resize(int capacity)
        {
            var temitems=new TItem[capacity];
            Array.Copy(_items,0,temitems,0,_items.Length);
            _items = temitems;
        }

        public void Push(TItem item)
        {
            if(_length==_items.Length) resize(2*_length);
            _items[_length&#43;&#43;] = item;
        }

        public TItem Pop()
        {
            if (IsEmpty) return default;
            TItem item = _items[--_length] ;
            _items[_length] = default;
            if(_length&gt;0 &amp;&amp; _length==_items.Length/4) resize( _items.Length / 2);
            return item;
        }
        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if(!IsEmpty)
            {
                for(var i=0;i&lt;_length;i&#43;&#43;)
                {
                    yield return _items[i];
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
                if (!IsEmpty)
                {
                    for (var i = 0; i &lt; _length; i&#43;&#43;)
                    {
                        _items[i] = default;
                    }
                }
                
            }
        }
    }
```

### 链表实现下压堆栈

```c#
    public class LinkedStackNet&lt;TItem&gt; : IStack&lt;TItem&gt;
    {
        private Node _first;
        private class Node
        {
            public TItem Item { set; get; }
            public Node Next { set; get; }
        }

        public LinkedStackNet()
        {
            _length = 0;
            _first = default;
        }


        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var temnode = _first;
                while (temnode != default)
                {
                    var item = temnode.Item;
                    temnode = temnode.Next;
                    yield return item;
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
                if (!IsEmpty)
                {
                    var temnode = _first;
                    while (temnode != default)
                    {
                         temnode.Item = default;
                        temnode = temnode.Next;
                        
                    }
                    _length=0；
                    _first=default;
                }
            }
        }

        private int _length;
        public bool IsEmpty =&gt; _length == 0;
        public int Length =&gt; _length ;
        public void Push(TItem item)
        {
            Node oldfirst = _first;
            _first=new Node();
            _first.Item = item;
            _first.Next = oldfirst;
            _length&#43;&#43;;
        }

        public TItem Pop()
        {
            if (IsEmpty) return default;
            var item = _first.Item;//不存在游离的问题，这个Node被GC回收
            _first = _first.Next;
            _length--;
            return item;
        }
    }
```

## Queue（队列）

队列：先进先出（FIFO）

```c#
    public interface IQueue&lt;TItem&gt; : IEnumerable&lt;TItem&gt;, IDisposable
    {
        bool IsEmpty { get; }
        int Length { get; }
        void Enqueue(TItem item);
        TItem Dequeue();
    }
```

### 数组实现

```c#
    public class ResizingArrayQueueNet&lt;TItem&gt; : IQueue&lt;TItem&gt;
    {
        public ResizingArrayQueueNet()
        {
            _first = 0;
            _last = 0;
            _items=new TItem[2];
        }

        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                for (var i = _first; i &lt; _last; i&#43;&#43;)
                {
                    yield return _items[i];
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
                if (!IsEmpty)
                {
                    for (var i = _first; i &lt; _last; i&#43;&#43;)
                    {
                        _items[i] = default;
                    }
                    _items = null;
                }
            }
        }

        private TItem[] _items;
        private int _first;
        private int _last;
        public bool IsEmpty =&gt; (_last - _first) == 0;
        public int Length =&gt; _last-_first;

        private void resize(int size)
        {
            var temitems=new TItem[size];
            var temlength = Length;
            Array.Copy(_items,_first,temitems,0,Length);
            _first = 0;
            _last = temlength;
            _items = temitems;
        }
        public void Enqueue(TItem item)
        {
            if(_last==_items.Length) resize(Length*2);
            _items[_last&#43;&#43;] = item;
        }

        public TItem Dequeue()
        {
            if (IsEmpty) return default;
            var item = _items[_first&#43;&#43;];
            if(Length&lt;_items.Length/4) resize(_items.Length/2);
            return item;
        }
    }
```

### 链表实现下压队列

```c#
    public class LinkedQueueNet&lt;TItem&gt; : IQueue&lt;TItem&gt;
    {
        private Node _first;
        private Node _last;
        private int _length;

        public LinkedQueueNet()
        {
            _first = null;
            _last = null;
            _length = 0;
        }
        private class Node
        {
           public TItem Item { set; get; }
           public Node Next { set; get; }
        }
        public IEnumerator&lt;TItem&gt; GetEnumerator()
        {
            if (!IsEmpty)
            {
                var temnode = _first;
                while (temnode != default)
                {
                    var item = temnode.Item;
                    temnode = temnode.Next;
                    yield return item;
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
                if (!IsEmpty)
                {
                    var temfirst = _first;
                    while (temfirst != default)
                    {
                        temfirst.Item = default;
                        temfirst = temfirst.Next;
                    }
                    _length = 0;
                }
            }
        }


        public bool IsEmpty =&gt; _length == 0;
        public int Length =&gt; _length;
        public void Enqueue(TItem item)
        {
            
            var temnode = _last;
            _last = new Node();
            _last.Item = item;
            _last.Next = null;
            if (IsEmpty) _first = _last;
            else temnode.Next = _last;
            _length&#43;&#43;;

        }

        public TItem Dequeue()
        {
            if (_length &gt; 0)
            {
                var temitem = _first.Item;
                _first = _first.Next;
                _length--;
                return temitem;
            }

            return default;
        }
    }
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/11/algorithm2/  

