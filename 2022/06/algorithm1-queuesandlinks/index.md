# 算法 队列与链表


## 实现队列的一些思路

数组的优劣：

1. 读取：连续的地址空间，数组全部或者部分元素被连续存在CPU缓存里面，读取速度非常快。
2. 插入/删除/扩容：插入和删除，需要修改该元素之前或之后所有元素的位置，扩容时需要重新找较大的内存块，将原数组中所有数据复制到新内存块中。这些操作都非常耗时。

链表的优劣：

1. 读取：链表的节点分散在堆空间中，无法利用CPU缓存，读写速度比较慢，是数组的33倍
2. 插入/删除/扩容：不需要改变原来元素的位置，仅仅修改节点信息即可。但是频繁的插入删除会导致堆中有大量碎片化内存。
3. 链表每个节点不仅需要保存数据还需要保存下一个节点的位置。

较好的方式是结合数组和链表的优势，用链表节点将固定大小的数组连接起来组成大的内存块，即易于扩展又在一定范围内保持良好的访问速度。

在需要构建内存池，缓存队列等应用场景中均可使用此方法进行优化。

## 队列接口

```C#
public interface IQueue&lt;TItem&gt; : IEnumerable&lt;TItem&gt;, IDisposable
{
    bool IsEmpty { get; }
    int Length { get; }
    void Enqueue(TItem item);
    TItem Dequeue();
}
```

## 基于数组的扩容队列

```C#
public class SGResizingArrayQueue&lt;TItem&gt; : IQueue&lt;TItem&gt;
{
	public SGResizingArrayQueue()
	{
		_first = 0;
		_last = 0;
		_items = new TItem[2];
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
	public int Length =&gt; _last - _first;

	private void resize(int size)
	{
		var temitems = new TItem[size];
		var temlength = Length;
		Array.Copy(_items, _first, temitems, 0, Length);
		_first = 0;
		_last = temlength;
		_items = temitems;
	}
	public void Enqueue(TItem item)
	{
		if (_last == _items.Length) resize(Length * 2);
		_items[_last&#43;&#43;] = item;
	}

	public TItem Dequeue()
	{
		if (IsEmpty) return default;
		var item = _items[_first&#43;&#43;];
		if (Length &lt; _items.Length / 4) resize(_items.Length / 2);
		return item;
	}
}
```

## 基于链表的扩容队列

```C#
public class SGLinkedQueue&lt;TItem&gt; : IQueue&lt;TItem&gt;
{
	private Node _first;
	private Node _last;
	private int _length;

	public SGLinkedQueue()
	{
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

## 结合数组和链表的扩容队列

```C#
class SGArraySegment&lt;TItem&gt;
{
   
	public TItem[] Array { get; private set; }
	
	public SGArraySegment&lt;TItem&gt; Next { get; set; }
	
	public int Offset { get; set; }
	
	public int End { get; set; } = -1; 

	public SGArraySegment(TItem[] array)
	{
		Array = array;
	}
	
	public bool IsAvailable
	{
		get { return Array.Length &gt; (End &#43; 1); }
	}
	
	public void Write(TItem value)
	{
		Array[&#43;&#43;End] = value;
	}
}


class SGPipeQueue&lt;TItem&gt; : IValueTaskSource&lt;TItem&gt;, IDisposable
{
	private const int _segmentSize = 5;
	
	private SGArraySegment&lt;TItem&gt; _first;
	
	private SGArraySegment&lt;TItem&gt; _current;
	
	private object _syncRoot = new object();
	
	private static readonly ArrayPool&lt;TItem&gt; _pool = ArrayPool&lt;TItem&gt;.Shared;
   
	private ManualResetValueTaskSourceCore&lt;TItem&gt; _taskSourceCore;
	
	private bool _waiting = false;
	
	private bool _lastReadIsWait = false;
	
	private int _length;
    
	public SGPipeQueue()
	{
		SetBufferSegment(CreateSegment());
		_taskSourceCore = new ManualResetValueTaskSourceCore&lt;TItem&gt;();
	}

	SGArraySegment&lt;TItem&gt; CreateSegment()
	{
		return new SGArraySegment&lt;TItem&gt;(_pool.Rent(_segmentSize));
	}

	void SetBufferSegment(SGArraySegment&lt;TItem&gt; segment)
	{
		if (_first == null)
			_first = segment;
	   
		var current = _current;
		if (current != null)
			current.Next = segment;
		_current = segment;
	}
	
	public int Write(TItem target)
	{
		lock (_syncRoot)
		{
			
			if (_waiting)
			{
				
				_waiting = false;
				_taskSourceCore.SetResult(target);
				return _length;
			}
			var current = _current;
			
			if (!current.IsAvailable)
			{
				current = CreateSegment();
				SetBufferSegment(current);
			}
			current.Write(target);
			_length&#43;&#43;;
			return _length;
		}
	}

	public ValueTask&lt;TItem&gt; ReadAsync()
	{
		lock (_syncRoot)
		{
			if (TryRead(out TItem value))
			{
				if (_lastReadIsWait)
				{
					_taskSourceCore.Reset();
					_lastReadIsWait = false;
				}
				_length--;
				if (_length == 0)
					OnWaitTaskStart();
				return new ValueTask&lt;TItem&gt;(value);
			}
			_waiting = true;
			_lastReadIsWait = true;
			_taskSourceCore.Reset();
			OnWaitTaskStart();
			return new ValueTask&lt;TItem&gt;(this, _taskSourceCore.Version);
		}
	}

	private bool TryRead(out TItem value)
	{
		var first = _first;
		if (first.Offset &lt; first.End)
		{
			value = first.Array[first.Offset];
			first.Array[first.Offset] = default;
			first.Offset&#43;&#43;;
			return true;
		}
		else if (first.Offset == first.End)
		{
			if (first == _current)
			{
				value = first.Array[first.Offset];
				first.Array[first.Offset] = default;
				first.Offset = 0;
				first.End = -1;
				return true;
			}
			else
			{
				value = first.Array[first.Offset];
				first.Array[first.Offset] = default;
				_first = first.Next;
				_pool.Return(first.Array);
				return true;
			}
		}
		value = default;
		return false;
	}

	protected virtual void OnWaitTaskStart()
	{

	}

	public virtual void Clear()
	{
		lock (_syncRoot)
		{
			if (_lastReadIsWait)
			{
				_taskSourceCore.Reset();
				_lastReadIsWait = false;
			}

			var first = _first;
			if (first.Offset &lt;= first.End)
			{
				while (first != _current)
				{
					for (int i = first.Offset; i &lt;= first.End; i&#43;&#43;)
					{
						first.Array[i] = default;
					}
					_first = first.Next;
					_pool.Return(first.Array);
					first = _first;
				}


				for (int i = first.Offset; i &lt;= first.End; i&#43;&#43;)
				{
					first.Array[i] = default;
				}
				first.Offset = 0;
				first.End = -1;
			}
		}
	}

	public void Dispose()
	{
		lock (_syncRoot)
		{
			var segment = _first;

			while (segment != null)
			{
				_pool.Return(segment.Array);
				segment = segment.Next;
			}

			_first = null;
			_current = null;
		}
	}

	TItem IValueTaskSource&lt;TItem&gt;.GetResult(short token)
	{
		return _taskSourceCore.GetResult(token);
	}

	ValueTaskSourceStatus IValueTaskSource&lt;TItem&gt;.GetStatus(short token)
	{
		return _taskSourceCore.GetStatus(token);
	}

	void IValueTaskSource&lt;TItem&gt;.OnCompleted(Action&lt;object?&gt; continuation, object? state, short token, ValueTaskSourceOnCompletedFlags flags)

	{
		_taskSourceCore.OnCompleted(continuation, state, token, flags);
	}
}


```

该队列额外实现2个功能，以提高队列的性能：

1. 异步出队列，当队列为空时，异步等待。
2. 当压入队列时，如果发现有异步等待对象，则不进入队列，直接给等待对象。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2022/06/algorithm1-queuesandlinks/  

