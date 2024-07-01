# 算法 排序


## 排序

比较和交换：

```c#
    public class AssistSort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        public static void Shuffle(T[] seq)
        {
            int n = seq.Length;
            var random = new Random(Guid.NewGuid().GetHashCode());
            for (int i = 0; i &lt; n; i&#43;&#43;)
            {
                // choose index uniformly in [0, i]
                int r = (int)(random.Next(n));
                T swap = seq[r];
                seq[r] = seq[i];
                seq[i] = swap;
            }
        }



        // does v == w ?
        public static bool Eq(T v, T w)
        {
            if (ReferenceEquals(v, w)) return true;    // optimization when reference equal
            return v.CompareTo(w) == 0;
        }
        public static void Exch(T[] seq, int value, int min)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (value &gt; seq.Length - 1 || value &lt; 0)
                throw new ArgumentException(&#34;value is not in seq&#34;);
            if (min &gt; seq.Length - 1 || min &lt; 0)
                throw new ArgumentException(&#34;min is not in seq&#34;);
            T t = seq[value];
            seq[value] = seq[min];
            seq[min] = t;
        }

        public static bool Less(T value, T compare)
        {
            if (value == null)
                throw new ArgumentNullException(&#34;value  is null&#34;);
            if (compare == null)
                throw new ArgumentNullException(&#34;compare is null&#34;);
            return value.CompareTo(compare) &lt; 0;
        }

        public static void Show(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                throw new ArgumentException(&#34;seq length equal 0&#34;);
            Console.WriteLine(string.Join(&#34; &#34;, seq));

        }

        public static bool IsSorted(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                throw new ArgumentException(&#34;seq length equal 0&#34;);
            for (int i = 1; i &lt; seq.Length; i&#43;&#43;)
            {
                if (Less(seq[i], seq[i - 1])) return false;
            }

            return true;
        }


        public static bool IsSorted(T[] seq, int lo, int hi)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (seq.Any() || seq.Length &lt; 2)
                throw new ArgumentException(&#34;seq length is abnormal&#34;);
            if (lo &lt; 0 || lo &gt; seq.Length - 1 || (lo &#43; 1) == seq.Length)
                throw new ArgumentException(&#34;lo is abnormal&#34;);
            if (hi &lt; 0 || hi &gt; seq.Length - 1 || hi &lt; lo)
                throw new ArgumentException(&#34;hi is abnormal&#34;);


            for (int i = lo &#43; 1; i &lt;= hi; i&#43;&#43;)
                if (Less(seq[i], seq[i - 1])) return false;
            return true;

        }
    }
```



### 选择排序

运行时间与初始状态无关

数据移动是最少的，只用了N次交换，交换次数和数组的大小是线性关系，

找到剩余元素中最小数与第一个交换

结论A：对于长度为N的数组，选择排序的需要大约N^2/2次比较和N次交换

```c#
   public class SelectionSort&lt;T&gt; where T : System.IComparable&lt;T&gt;
    {
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            var count = seq.Length;
            for (int i = 0; i &lt; count; i&#43;&#43;)
            {
                int min = i;
                //找出剩余项中最小项的位置
                for (int j = i &#43; 1; j &lt; count; j&#43;&#43;)
                {
                    //if (seq[j].CompareTo(seq[min]) &lt; 0) min = j;
                    if (AssistSort&lt;T&gt;.Less(seq[j], seq[min])) min = j;
                }

                //将最小项与min交换
                AssistSort&lt;T&gt;.Exch(seq, i, min); //调用静态方法非常消耗性能
                //T t = seq[i];
                //seq[i] = seq[min];
                //seq[min] = t;
            }
        }
    }
```

### 插入排序

像打牌一项，将每一张排插入到已经有序的牌中的适当位置。

运行时间取决于元素的初始顺序。

结论B：对于随机不重复的数组，平均情况下插入排序需要N^2/4次比较和交换。最坏情况下需要N^2/2次交换和交换，最好情况下需要N-1次比较和0次交换。

```c#
    public class InsertionSort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int count = seq.Length;
            for (int i = 0; i &lt; count; i&#43;&#43;)
            {
                for (int j = i; j &gt; 0 &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - 1]); j--)
                {
                    AssistSort&lt;T&gt;.Exch(seq, j, j - 1);
                }
            }
            // 5 4 3 1 2
            // 4 5 3 1 2
            // 4 3 5 1 2
            // 3 4 5 1 2
            // 1 3 4 5 2
            // 1 2 3 4 5

            //1 2 4 3 5
            //1 2
        }
        public static void Sort(T[] seq, int lo, int hi)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            if (hi &lt; lo || lo &lt; 0 || hi &lt; 0 || lo &gt; seq.Length || hi &gt; seq.Length)
                throw new ArgumentException(&#34;Parameter error&#34;);
            for (int i = lo &#43; 1; i &lt; hi; i&#43;&#43;)
            {
                for (int j = i; j &gt; lo &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - 1]); j--)
                {
                	//交换，每个seq[j]相当于被访问了2次，跟seq[j&#43;1]和seq[j-1]各比较一次
                    AssistSort&lt;T&gt;.Exch(seq, j, j - 1);
                }
            }
        }
    }
```

### 插入排序改进减少比较

结论C：插入排序需要的交换操作和数组中倒置的数量相同，需要比较次数大于等于倒置的数量，小于等于倒置的数量加上数组的大小再减一。

改进插入排序，只需要在内循环中将较大的元素都向**右移动而不总是交换两个元素**。这样访问数组的次数能够减半。

```c#
    public static class InsertionSortX&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int n = seq.Length;

            // put smallest element in position to serve as sentinel
            int exchanges = 0;
            for (int i = n - 1; i &gt; 0; i--)
            {
                if (AssistSort&lt;T&gt;.Less(seq[i], seq[i - 1]))
                {
                    AssistSort&lt;T&gt;.Exch(seq, i, i - 1);
                    exchanges&#43;&#43;;
                }
            }
            if (exchanges == 0) return;


            // insertion sort with half-exchanges
            for (int i = 2; i &lt; n; i&#43;&#43;)
            {
                T v = seq[i];
                int j = i;
                while (AssistSort&lt;T&gt;.Less(v, seq[j - 1]))
                {
                	//右移，seq[j]只跟当前比较的值V比较了1次
                    seq[j] = seq[j - 1];
                    j--;
                }
                seq[j] = v;
            }
        }
    }
```

插入和选择比较

![1605416095547](/algorithm/1605416095547.png)

### 插入排序改进二分快速定位

可以看到插入左边都是已近排号顺序的，可以使用二分法快速确定待插入数的位置。

```c#
    public static class BinaryInsertionSort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int n = seq.Length;

            for (int i = 1; i &lt; n; i&#43;&#43;)
            {

                // binary search to determine index j at which to insert a[i]
                T v = seq[i];
                int lo = 0, hi = i;
                while (lo &lt; hi)
                {
                    int mid = lo &#43; (hi - lo) / 2;
                    if (AssistSort&lt;T&gt;.Less(v, seq[mid])) hi = mid;
                    else lo = mid &#43; 1;
                }

                // insetion sort with &#34;half exchanges&#34;
                // (insert a[i] at index j and shift a[j], ..., a[i-1] to right)
                for (int j = i; j &gt; lo; --j)
                    seq[j] = seq[j - 1];
                seq[lo] = v;
            }
        }
    }
```

结论D：对于随机排序的无重复主键的数组，插入排序和选择排序的运行时间是平方级别的，两者之比应该是一个较小的常数。

### 希尔排序

大规模乱序的插入排序很慢，因为它只会交换相邻的元素，如果最小的元素正好在数组的尽头，那么把它移动到正确位置就需要N-1次移动，希尔排序就是为了解决这个问题，交换不相邻的元素以对数组的局部进行排序。

希尔排序的思想：**数组中任意间隔为h的元素都是有序的，这样的数组成为h有序数组。一个h有序的数组是h个相互独立的有序数组组成的一个数组。**

希尔排序高效的原因是：它权衡了子数组的规模和有序性。

![1605417719877](/algorithm/1605417719877.png)

```c#
    /// &lt;summary&gt;
    /// 希尔排序，插入排序的变种
    /// 其实就是从h项开始选定，跟之前的h项进行比较交换
    /// 然后再缩小h进行重复的插入排序，h缩小的顺序，1 4 13 40.。。。 
    /// 最后一次一定是1，就是插入排序，只不过这时序列已经大部分顺序，所以进行了少量交换操作
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public class ShellSort&lt;T&gt; where T : IComparable&lt;T&gt;
    {

        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int count = seq.Length;
            int h = 1;
            //求出最大限度间隔，
            while (h &lt; count / 3) h = 3 * h &#43; 1;//1 4 13 40 121 364 1093
            while (h &gt;= 1)
            {//将数组变成h有序
                for (int i = h; i &lt; count; i&#43;&#43;)
                {
                    //将a[i]插入a[i-h],a[i-2*h],a[i-3*h].....
                    for (int j = i; j &gt;= h &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - h]); j -= h)
                    {
                        AssistSort&lt;T&gt;.Exch(seq, j, j - h);
                    }
                }

                h = h / 3;
            }
            //0 1 2 3 4 5 6   h=4
            //4:0 5:1 6:2 
            //h=1
            //1:0 2:1 3:2
        }


        public static void SortPro(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int count = seq.Length;
            int h = 1;
            //求出最大限度间隔，
            while (h &lt; count / 2) h = 2 * h &#43; 1;//1 3 7 15 31
            while (h &gt;= 1)
            {//将数组变成h有序
                for (int i = h; i &lt; count; i&#43;&#43;)
                {

                    for (int j = i; j &gt;= h &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - h]); j -= h)
                    {
                        AssistSort&lt;T&gt;.Exch(seq, j, j - h);
                    }
                }

                h = h / 2;
            }
        }
        public static void SortPlus(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int count = seq.Length;
            int h = 1;
            //求出最大限度间隔，
            while (h &lt; count / 4) h = 4 * h &#43; 1;//1 5 21 85 
            while (h &gt;= 1)
            {//将数组变成h有序
                for (int i = h; i &lt; count; i&#43;&#43;)
                {

                    for (int j = i; j &gt;= h &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - h]); j -= h)
                    {
                        AssistSort&lt;T&gt;.Exch(seq, j, j - h);
                    }
                }

                h = h / 4;
            }
        }
    }
```

希尔排序对任意排序的数组表现都很好。

![1605419675381](/algorithm/1605419675381.png)

希尔排序能够解决一些初级排序算法无能为力的问题，通过提升速度来解决其他方式无法解决的问题是研究算法的设计和性能的主要原因。

希尔排序：当一个h有序的数组按照增幅K排序后，它仍然是h有序的

希尔排序的运行时间不到平方级别，在最后情况下是N^(3/2)

![1605419883228](/algorithm/1605419883228.png)

性质E：使用递增序列1，4，13，40，121，364……的希尔排序所需的比较次数不会超出N的若干倍乘以递增序列的长度。

如果需要解决一个排序问题，优先先使用希尔，再使用其他，因为它不需要额外的内存，其他的高效排序不会比希尔排序快2倍。

### 归并排序自顶而下

归并排序核心思想是：可以先(递归地)将整个数组分成两半分别排序，然后结果归并起来。

性能：能够将任意长度为N的数组排序所需的时间和NlogN成正比。

**分而治之**

```c#
    /// &lt;summary&gt;
    /// 归并排序,自顶而下，使用了分治的思想,NlgN
    /// 核心思想就是两个顺序数列合并成一个数列
    /// 将数列以中间index为界分为前后，然后递归分，直到子数组只有2个然后合并。
    /// 浪费了大量的空间，每次合并都需一个Auxiliary
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public class MergeSortUB&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        /// &lt;summary&gt;
        /// 将数列low-mid和ming-high合并
        /// low-mid必须顺序
        /// mid-high必须顺序
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;seq&#34;&gt;数列&lt;/param&gt;
        /// &lt;param name=&#34;aux&#34;&gt;合并需要数组&lt;/param&gt;
        /// &lt;param name=&#34;lo&#34;&gt;low所在index&lt;/param&gt;
        /// &lt;param name=&#34;mid&#34;&gt;mind所在index&lt;/param&gt;
        /// &lt;param name=&#34;hi&#34;&gt;hi所在index&lt;/param&gt;
        private static void Merge(T[] seq, T[] aux, int lo, int mid, int hi)//aux:auxiliary备用
        {
            // 合并前两个数列必须是顺序的,私有方法不需要验证
            //if(!AssistSort&lt;T&gt;.IsSorted(seq, lo, mid))
            //    throw new ArgumentException(&#34;seq is not sort in low to mid&#34;);
            //if(!AssistSort&lt;T&gt;. IsSorted(seq, mid
            //&#43;1, hi))
            //    throw new ArgumentException(&#34;seq is not sort in mid to high&#34;);
            // 备份
            for (int k = lo; k &lt;= hi; k&#43;&#43;)
            {
                aux[k] = seq[k];
            }
            // 合并算法，i左半边开始位,j右半边开始位
            int i = lo, j = mid &#43; 1;
            for (int k = lo; k &lt;= hi; k&#43;&#43;)
            {
                if (i &gt; mid) seq[k] = aux[j&#43;&#43;];//左半边用尽，取右半边元素
                else if (j &gt; hi) seq[k] = aux[i&#43;&#43;];//右半边用尽，取左半边元素
                else if (AssistSort&lt;T&gt;.Less(aux[j], aux[i])) seq[k] = aux[j&#43;&#43;];//右半边元素小于左边元素，取右半边元素
                else seq[k] = aux[i&#43;&#43;];//取左半边元素
            }
        }

        private static void Sort(T[] seq, T[] aux, int lo, int hi)
        {
            if (hi &lt;= lo) return;
            int mid = lo &#43; (hi - lo) / 2;
            Sort(seq, aux, lo, mid);
            Sort(seq, aux, mid &#43; 1, hi);
            Merge(seq, aux, lo, mid, hi);
        }

        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            T[] aux = new T[seq.Length];
            Sort(seq, aux, 0, seq.Length - 1);
        }

    }
```

![1605423950695](/algorithm/1605423950695.png)

通过一棵树来解释归并的性能，每个节点都标识通过Merge()方法并归成的子数组，这棵树正好n层，k表示在K层，K层有2^K^个节点，每个节点的长度为2^(n-k)^，那么并归最多需要2^(n-k)^次比较。那么每层需要比较次数2^K^*2^(n-k)^=2^n^，n层总共为n*2^n^=NlgN;

命题F:对于长度为N的任意数组，自顶向下的并归需要1/2NlgN至NlgN次比较。

![1605429505244](/algorithm/1605429505244.png)

命题G:对于长度为N的任意数组，自顶向下的归并排序最多需要访问数组6NlgN次。

### 归并统计交换次数

```c#
/// &lt;summary&gt;
/// 统计交换次数，自顶而下并归方法
/// &lt;/summary&gt;
/// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
public class Inversion&lt;T&gt; where T : IComparable&lt;T&gt;
{
    public static long Count(T[] seq)
    {
        T[] b = new T[seq.Length];
        T[] aux = new T[seq.Length];
        for (int i = 0; i &lt; seq.Length; i&#43;&#43;)
            b[i] = seq[i];
        long inversions = Count(seq, b, aux, 0, seq.Length - 1);
        return inversions;
    }
    private static long Count(T[] a, T[] b, T[] aux, int lo, int hi)
    {
        long inversions = 0;
        if (hi &lt;= lo) return 0;
        int mid = lo &#43; (hi - lo) / 2;
        inversions &#43;= Count(a, b, aux, lo, mid);
        inversions &#43;= Count(a, b, aux, mid &#43; 1, hi);
        inversions &#43;= Merge(b, aux, lo, mid, hi);
        if (inversions != Brute(a, lo, hi))
            throw new Exception(&#34;inversion is not right&#34;);
        return inversions;
    }
    private static long Merge(T[] a, T[] aux, int lo, int mid, int hi)
    {
        long inversions = 0;

        // copy to aux[]
        for (int k = lo; k &lt;= hi; k&#43;&#43;)
        {
            aux[k] = a[k];
        }

        // merge back to a[]
        int i = lo, j = mid &#43; 1;
        for (int k = lo; k &lt;= hi; k&#43;&#43;)
        {
            if (i &gt; mid) a[k] = aux[j&#43;&#43;];
            else if (j &gt; hi) a[k] = aux[i&#43;&#43;];
            else if (AssistSort&lt;T&gt;.Less(aux[j], aux[i])) { a[k] = aux[j&#43;&#43;]; inversions &#43;= (mid - i &#43; 1); }
            else a[k] = aux[i&#43;&#43;];
        }
        return inversions;
    }

    private static long Brute(T[] a, int lo, int hi)
    {
        long inversions = 0;
        for (int i = lo; i &lt;= hi; i&#43;&#43;)
            for (int j = i &#43; 1; j &lt;= hi; j&#43;&#43;)
                if (AssistSort&lt;T&gt;.Less(a[j], a[i])) inversions&#43;&#43;;
        return inversions;
    }
}
```

### 归并排序小数组用插值排序

使用插入排序处理小规模数组（长度小于15），一般可以将归并排序的运行时间减少10%~15%

![1605430680751](/algorithm/1605430680751.png)

### 归并排序改进a[mid]小于等于a[mid&#43;1]

这样使得任意有序的子数组算法运行时间线性

### 归并排序不复制元素到辅助数组

### 归并排序自底向上

```c#
    /// &lt;summary&gt;
    /// 并归排序,自底而上，使用了分治的思想,NlgN
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public class MergeSortBU&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        /// &lt;summary&gt;
        /// 将数列low-mid和ming-high合并
        /// low-mid必须顺序
        /// mid-high必须顺序
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;seq&#34;&gt;数列&lt;/param&gt;
        /// &lt;param name=&#34;aux&#34;&gt;合并需要数组&lt;/param&gt;
        /// &lt;param name=&#34;lo&#34;&gt;low所在index&lt;/param&gt;
        /// &lt;param name=&#34;mid&#34;&gt;mind所在index&lt;/param&gt;
        /// &lt;param name=&#34;hi&#34;&gt;hi所在index&lt;/param&gt;
        private static void Merge(T[] seq, T[] aux, int lo, int mid, int hi)
        {

            // copy to aux[]
            for (int k = lo; k &lt;= hi; k&#43;&#43;)
            {
                aux[k] = seq[k];
            }

            // merge back to a[]
            int i = lo, j = mid &#43; 1;
            for (int k = lo; k &lt;= hi; k&#43;&#43;)
            {
                if (i &gt; mid) seq[k] = aux[j&#43;&#43;];  // this copying is unneccessary
                else if (j &gt; hi) seq[k] = aux[i&#43;&#43;];
                else if (AssistSort&lt;T&gt;.Less(aux[j], aux[i])) seq[k] = aux[j&#43;&#43;];
                else seq[k] = aux[i&#43;&#43;];
            }

        }

        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            int n = seq.Length;
            T[] aux = new T[n];
            for (int len = 1; len &lt; n; len *= 2) // 1 2 4 8 16
            {
                for (int lo = 0; lo &lt; n - len; lo &#43;= len &#43; len)
                {
                    int mid = lo &#43; len - 1;
                    int hi = Math.Min(lo &#43; len &#43; len - 1, n - 1);
                    Merge(seq, aux, lo, mid, hi);
                }
            }

        }
    }
```

![1605446023669](/algorithm/1605446023669.png)

![1605446078238](/algorithm/1605446078238.png)

命题H:对于长度为N的任意数组，自底向上归并排序需要1/2NlgN至NlgN次比较，最多访问数组6NlgN次。

当数组长度为2的幂的时候，自顶向下和自底向上的归并排序所用的比较次数和数组访问次数正好相同。

自底向上的归并排序比较适合链表组织的数据。这种方法只需要重新组织链表的链接就能将链表原地排序

自顶向下：化整为零。

自下向上：循序渐进。

命题I:没有任何基于比较的算法能够保证使用少于lg(N!)~NlgN次比较将长度为N的数组排序。

![1605449200186](/algorithm/1605449200186.png)

命题J：归并排序是一种渐进最有的基于比较排序的算法

### 快速排序

应用最广泛的系统。快速排序的内循环比大多数排序算法都要短小，这意味着它无论是在理论上还是在实际中都要更快。它的主要缺点是非常脆弱，在实现时要非常小心才能避免低劣的性能。

快速排序是一种分治的排序算法。它将一个数组分成两个子数组，将两部分独立地排序。快速排序和归并排序是互补的：归并排序将数组分成两个子数组分别排序，并将有序的子数组归并以将整个数组排序；而快速排序将数组排序的方式则是当两个子数组都有序时整个数组也就自然有序了

在归并排序中， 一个数组被等分为两半；在快速排序中，切分(partition ) 的位置取决于数组的内容。

快速排序和归并排序是互补的，快速排序方式当两个子数组都有序时，整个数据就自然有序了。

**快速算法怎么将序列切分呢，哪个相当于中点的数是这样的，这个数的前面都比这个数小，这个数的后面都比这个数大，**

![1605449958418](/algorithm/1605449958418.png)

```c#
    /// &lt;summary&gt;
    /// 快速算法应该是并归算法的一个变种，并归算是找中点，一切为二
    /// 快速算法也是分治算法的一种，怎么将序列切分呢，哪个相当于中点的数是这样的，这个数的前面都比这个数小，这个数的后面都比这个数大，
    /// 这样不断切分，最后自然切分完自然序列排序好了。
    /// 如果有大量重复的数据这个实现有点问题
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public static class QuickSort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            Sort(seq, 0, seq.Length - 1);
        }
        private static void Sort(T[] seq, int lo, int hi)
        {
            if (hi &lt;= lo) return;
            int j = Partition(seq, lo, hi);
            Sort(seq, lo, j - 1);
            Sort(seq, j &#43; 1, hi);
        }
        /// &lt;summary&gt;
        /// 切分
        /// a[lo..hi] =&gt; a[lo..j-1] = a[j] = a[j&#43;1..hi]
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;seq&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;lo&#34;&gt;&lt;/param&gt;
        /// &lt;param name=&#34;hi&#34;&gt;&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        private static int Partition(T[] seq, int lo, int hi)
        {
            int i = lo;
            int j = hi &#43; 1;
            T v = seq[lo];
            //确保index k之前的数都比其小，index k之后的数都比其大
            while (true)
            {
                // 找到第一个比第一项大的数index叫largeindex
                while (AssistSort&lt;T&gt;.Less(seq[&#43;&#43;i], v))
                {
                    if (i == hi) break;
                }
                // 找到最后一个比第一项小的数index,litindex
                while (AssistSort&lt;T&gt;.Less(v, seq[--j]))
                {
                    if (j == lo) break;      // redundant since a[lo] acts as sentinel
                }
                // 检查第一个和最后一个是否交叉了
                if (i &gt;= j) break;
                //将第一个项和最后一个项交换，确保比第一项小的数再第一项前面
                AssistSort&lt;T&gt;.Exch(seq, i, j);
            }
            //这个切分点就是第一个数
            AssistSort&lt;T&gt;.Exch(seq, lo, j);
            // 现在, a[lo .. j-1] &lt;= a[j] &lt;= a[j&#43;1 .. hi]
            return j;

        }
    }
```

![1605451786963](/algorithm/1605451786963.png)

交换部分

![1605451941624](/algorithm/1605451941624.png)

可能出现的问题：

&#43; 原地切分：使用辅助数组，很容易实现切分，但切分后的数组复制回去的开销会比较多。

&#43; 别越界：

&#43; 保持随机性：

&#43; 终止循环：

&#43; 处理切分元素值的重复情况

&#43; 终止递归

性能优势：

&#43; 用一个递增的索引将数组元素和一个定值比较。很难有排序算法比这更小的内循环。
&#43; 快速排序的比较次数比较少。
&#43; 快速排序最好的情况是：每次都正好将数组对半分，

命题K:将长度为N的无重复数组排序，快速排序平均需要2NlnN次比较。

命题L:快速排序最多需要约N^2/2次比较，但随机打乱数组能够预防这种情况。每次切分找到的切分值都是最大值或最小值

### 快速排序该进熵最优

如果有大量重复的元素，将数组切成三份，小于，等于和大于

![1605533808599](/algorithm/1605533808599.png)

```c#
    /// &lt;summary&gt;
    /// 该算法主要针对序列中有大量相同的项，将序列分为三部分，小于，等于，大于
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;

    public class QuickSort3WaySort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        private static void Sort(T[] seq, int lo, int hi)
        {
            if (hi &lt;= lo) return;
            //切分的lt，gt
            int lt = lo, gt = hi;
            T v = seq[lo];
            int i = lo &#43; 1;
            //切分
            while (i &lt;= gt)
            {
                int cmp = seq[i].CompareTo(v);
                if (cmp &lt; 0) AssistSort&lt;T&gt;.Exch(seq, lt&#43;&#43;, i&#43;&#43;);
                else if (cmp &gt; 0) AssistSort&lt;T&gt;.Exch(seq, i, gt--);
                else i&#43;&#43;;
            }
            //切分完成
            // a[lo..lt-1] &lt; v = a[lt..gt] &lt; a[gt&#43;1..hi]. 
            Sort(seq, lo, lt - 1);
            Sort(seq, gt &#43; 1, hi);
        }
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            //AssistSort&lt;T&gt;.Shuffle(seq);
            Sort(seq, 0, seq.Length - 1);
        }
    }
```

![1605534619434](/algorithm/1605534619434.png)

![1605534638638](/algorithm/1605534638638.png)

### 快速排序改进三取样切分并小数组插入

取子数组的中位数为切分点，取样大小s设为3并用大小s设为3并用大小居中的元素切分的效果最好。



### ![1605533465596](/algorithm/1605533465596.png)

问题：

- 小数组时插入排序比快速排序要快
- 因为递归，快速排序sort()方法在小数组中也会调用自己。

简单操作将

```c#
if(hi &lt;= lo) return;
```

替换成

```c#
if(hi &lt;= lo&#43;M) { Insertion.Sort(a,lo,hi); return;}
```

```c#
    /// &lt;summary&gt;
    /// 快速算法的痛点是在小序列的排序性能并不好，所有该X实现在小序列的时候排序采用插入排序算法
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public class QuickSortX&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        private static readonly int INSERTION_SORT_CUTOFF = 8;
        private static void Sort(T[] seq, int lo, int hi)
        {
            if (hi &lt;= lo) return;

            // cutoff to insertion sort (Insertion.sort() uses half-open intervals)
            int n = hi - lo &#43; 1;
            if (n &lt;= INSERTION_SORT_CUTOFF)//少量数组插入排序
            {
                InsertionSort&lt;T&gt;.Sort(seq, lo, hi &#43; 1);
                return;
            }

            int j = Partition(seq, lo, hi);
            Sort(seq, lo, j - 1);
            Sort(seq, j &#43; 1, hi);

        }
        // partition the subarray a[lo..hi] so that a[lo..j-1] &lt;= a[j] &lt;= a[j&#43;1..hi]
        // and return the index j.
        private static int Partition(T[] seq, int lo, int hi)
        {
            int n = hi - lo &#43; 1;
            int m = Median3(seq, lo, lo &#43; n / 2, hi);//三取样点，lo,中点,hi
            AssistSort&lt;T&gt;.Exch(seq, m, lo);

            int i = lo;
            int j = hi &#43; 1;
            T v = seq[lo];

            // a[lo] is unique largest element
            while (AssistSort&lt;T&gt;.Less(seq[&#43;&#43;i], v))
            {
                if (i == hi) { AssistSort&lt;T&gt;.Exch(seq, lo, hi); return hi; }
            }

            // a[lo] is unique smallest element
            while (AssistSort&lt;T&gt;.Less(v, seq[--j]))
            {
                if (j == lo &#43; 1) return lo;
            }

            // the main loop
            while (i &lt; j)
            {
                AssistSort&lt;T&gt;.Exch(seq, i, j);
                while (AssistSort&lt;T&gt;.Less(seq[&#43;&#43;i], v)) ;
                while (AssistSort&lt;T&gt;.Less(v, seq[--j])) ;
            }

            // put partitioning item v at a[j]
            AssistSort&lt;T&gt;.Exch(seq, lo, j);

            // now, a[lo .. j-1] &lt;= a[j] &lt;= a[j&#43;1 .. hi]
            return j;
        }
       
        private static int Median3(T[] seq, int i, int j, int k)
        {
            return (AssistSort&lt;T&gt;.Less(seq[i], seq[j]) ?
                (AssistSort&lt;T&gt;.Less(seq[j], seq[k]) ? j : AssistSort&lt;T&gt;.Less(seq[i], seq[k]) ? k : i) :
                (AssistSort&lt;T&gt;.Less(seq[k], seq[j]) ? j : AssistSort&lt;T&gt;.Less(seq[k], seq[i]) ? k : i));
        }
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            //AssistSort&lt;T&gt;.Shuffle(seq);
            Sort(seq, 0, seq.Length - 1);

        }
    }
```

![1605534759860](/algorithm/1605534759860.png)

命题M:不存在任何基于比较的排序算法能够保证在NH-N次比较之内将N个元素排序，

命题N:三向切分的快速排序需要(2ln2)NH次比较

### 快速排序中型数组排序

```c#
    /// &lt;summary&gt;
    /// 该算法进一步升级，不仅在小序列排序时实现插入排序，在中型序列实现三分排序。
    /// &lt;/summary&gt;
    /// &lt;typeparam name=&#34;T&#34;&gt;&lt;/typeparam&gt;
    public class QuickBentleyMcIlroySort&lt;T&gt; where T : IComparable&lt;T&gt;
    {
        // cutoff to insertion sort, must be &gt;= 1
        private static readonly int INSERTION_SORT_CUTOFF = 8;

        // cutoff to median-of-3 partitioning
        private static readonly int MEDIAN_OF_3_CUTOFF = 40;
        /// &lt;summary&gt;
        /// Rearranges the array in ascending order, using the natural order.
        /// &lt;/summary&gt;
        /// &lt;param name=&#34;seq&#34;&gt;&lt;/param&gt;
        public static void Sort(T[] seq)
        {
            if (seq == null)
                throw new ArgumentNullException(&#34;seq is null&#34;);
            if (!seq.Any())
                return;
            Sort(seq, 0, seq.Length - 1);
        }

        private static void Sort(T[] seq, int lo, int hi)
        {
            int n = hi - lo &#43; 1;

            // cutoff to insertion sort
            if (n &lt;= INSERTION_SORT_CUTOFF)
            {
                InsertionSort(seq, lo, hi);
                return;
            }

            // use median-of-3 as partitioning element
            else if (n &lt;= MEDIAN_OF_3_CUTOFF)
            {
                int m = Median3(seq, lo, lo &#43; n / 2, hi);
                AssistSort&lt;T&gt;.Exch(seq, m, lo);
            }

            // use Tukey ninther as partitioning element
            else
            {//其实就是扩大切分点的寻找范围，最有希望找到中间点，达到最有的快速排序
                int eps = n / 8;
                int mid = lo &#43; n / 2;
                int m1 = Median3(seq, lo, lo &#43; eps, lo &#43; eps &#43; eps);
                int m2 = Median3(seq, mid - eps, mid, mid &#43; eps);
                int m3 = Median3(seq, hi - eps - eps, hi - eps, hi);
                int ninther = Median3(seq, m1, m2, m3);
                AssistSort&lt;T&gt;.Exch(seq, ninther, lo);
            }

            // Bentley-McIlroy 3-way partitioning
            int i = lo, j = hi &#43; 1;
            int p = lo, q = hi &#43; 1;
            T v = seq[lo];
            while (true)
            {
                while (AssistSort&lt;T&gt;.Less(seq[&#43;&#43;i], v))
                    if (i == hi) break;
                while (AssistSort&lt;T&gt;.Less(v, seq[--j]))
                    if (j == lo) break;

                // pointers cross
                if (i == j &amp;&amp; AssistSort&lt;T&gt;.Eq(seq[i], v))
                    AssistSort&lt;T&gt;.Exch(seq, &#43;&#43;p, i);
                if (i &gt;= j) break;

                AssistSort&lt;T&gt;.Exch(seq, i, j);
                if (AssistSort&lt;T&gt;.Eq(seq[i], v)) AssistSort&lt;T&gt;.Exch(seq, &#43;&#43;p, i);
                if (AssistSort&lt;T&gt;.Eq(seq[j], v)) AssistSort&lt;T&gt;.Exch(seq, --q, j);
            }


            i = j &#43; 1;
            for (int k = lo; k &lt;= p; k&#43;&#43;)
                AssistSort&lt;T&gt;.Exch(seq, k, j--);
            for (int k = hi; k &gt;= q; k--)
                AssistSort&lt;T&gt;.Exch(seq, k, i&#43;&#43;);

            Sort(seq, lo, j);
            Sort(seq, i, hi);
        }
        // sort from a[lo] to a[hi] using insertion sort
        private static void InsertionSort(T[] seq, int lo, int hi)
        {
            for (int i = lo; i &lt;= hi; i&#43;&#43;)
                for (int j = i; j &gt; lo &amp;&amp; AssistSort&lt;T&gt;.Less(seq[j], seq[j - 1]); j--)
                    AssistSort&lt;T&gt;.Exch(seq, j, j - 1);
        }


        // return the index of the median element among a[i], a[j], and a[k]
        private static int Median3(T[] seq, int i, int j, int k)
        {
            return (AssistSort&lt;T&gt;.Less(seq[i], seq[j]) ?
                (AssistSort&lt;T&gt;.Less(seq[j], seq[k]) ? j : AssistSort&lt;T&gt;.Less(seq[i], seq[k]) ? k : i) :
                (AssistSort&lt;T&gt;.Less(seq[k], seq[j]) ? j : AssistSort&lt;T&gt;.Less(seq[k], seq[i]) ? k : i));
        }

    }
```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/11/algorithm3/  

