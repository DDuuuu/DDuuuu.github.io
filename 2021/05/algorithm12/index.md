# 算法 字符串压缩


## 数据压缩

基础模型

压缩盒，能够将一个比特流B 转化为压缩后的版本C(B);

展开盒，能够将C(B) 转化回B 。

![image-20210507103835963](/algorithm/image-20210507103835963.png)

这种模型叫做无损压缩模型一保证不丢失任何信息，即压缩和展开之后的比特流必须和原始的比特流完全相同。许多种类型的文件都会用到无损压缩，例如数值数据或者可执行的代码。对于某些类型的文件（例如图像、视频和音乐），有损的压缩方法也是可以接受的，此时解码器所产生的输出只是与原输入文件近似。

### 读写二进制数据

![image-20210507145629392](/algorithm/image-20210507145629392.png)

![image-20210507145637767](/algorithm/image-20210507145637767.png)

```c#
    public  class BinaryStdIn
    {
        private static  int EOF = -1;      // end of file

        private static TextReader input;  // input stream
        private static int buffer;              // one character buffer
        private static int n;                   // number of bits left in buffer
        private static bool isInitialized;   // has BinaryStdIn been called for first time?

        // don&#39;t instantiate
        private BinaryStdIn() { }

        // fill buffer
        private static void Initialize()
        {
            input = Console.In;
            buffer = 0;
            n = 0;
            FillBuffer();
            isInitialized = true;
        }

        private static void FillBuffer()
        {
            try
            {
                buffer = input.Read();//文本读取器中的下一个字符
                n = 8;
            }
            catch (IOException e)
            {
                Console.WriteLine(&#34;EOF&#34;);
                buffer = EOF;
                n = -1;
            }
        }

        
        public static void Close()
        {
            if (!isInitialized) Initialize();
            try
            {
                input.Close();
                isInitialized = false;
            }
            catch (IOException ioe)
            {
                throw new ArgumentException(&#34;Could not close BinaryStdIn&#34;, ioe);
            }
        }

        
        public static bool IsEmpty()
        {
            if (!isInitialized) Initialize();
            return buffer == EOF;
        }

        
        public static bool ReadBoolean()
        {
            if (IsEmpty()) throw new ArgumentNullException(&#34;Reading from empty input stream&#34;);
            n--;
            bool bit = ((buffer &gt;&gt; n) &amp; 1) == 1;
            if (n == 0) FillBuffer();
            return bit;
        }

       
        public static char ReadChar()
        {
            if (IsEmpty()) throw new ArgumentNullException(&#34;Reading from empty input stream&#34;);

            // special case when aligned byte
            if (n == 8)
            {
                int y = buffer;
                FillBuffer();
                return (char)(y &amp; 0xff);
            }

            // combine last n bits of current buffer with first 8-n bits of new buffer
            int x = buffer;
            x &lt;&lt;= (8 - n);
            int oldN = n;
            FillBuffer();
            if (IsEmpty()) throw new ArgumentNullException(&#34;Reading from empty input stream&#34;);
            n = oldN;
            x |= (int)((uint)buffer &gt;&gt; n);
            return (char)(x &amp; 0xff);
            // the above code doesn&#39;t quite work for the last character if n = 8
            // because buffer will be -1, so there is a special case for aligned byte
        }

        
        public static char ReadChar(int r)
        {
            if (r &lt; 1 || r &gt; 16) throw new ArgumentNullException(&#34;Illegal value of r = &#34; &#43; r);

            // optimize r = 8 case
            if (r == 8) return ReadChar();

            byte x = 0;
            for (int i = 0; i &lt; r; i&#43;&#43;)
            {
                x &lt;&lt;= 1;
                bool bit = ReadBoolean();
                if (bit) x |= 1;
            }
            return (char)x;
        }

        public static String ReadString()
        {
            if (IsEmpty()) throw new ArgumentNullException(&#34;Reading from empty input stream&#34;);

            StringBuilder sb = new StringBuilder();
            while (!IsEmpty())
            {
                char c = ReadChar();
                sb.Append(c);
            }
            return sb.ToString();
        }

        public static short ReadShort()
        {
            short x = 0;
            for (int i = 0; i &lt; 2; i&#43;&#43;)
            {
                byte c =(byte) ReadChar();
                x &lt;&lt;= 8;
                x |= c;
            }
            return x;
        }
        public static int ReadInt()
        {
            int x = 0;
            for (int i = 0; i &lt; 4; i&#43;&#43;)
            {
                byte c =(byte) ReadChar();
                x &lt;&lt;= 8;
                x |= c;
            }
            return x;
        }

        public static int ReadInt(int r)
        {
            if (r &lt; 1 || r &gt; 32) throw new ArgumentException(&#34;Illegal value of r = &#34; &#43; r);

            // optimize r = 32 case
            if (r == 32) return ReadInt();

            int x = 0;
            for (int i = 0; i &lt; r; i&#43;&#43;)
            {
                x &lt;&lt;= 1;
                bool bit = ReadBoolean();
                if (bit) x |= 1;
            }
            return x;
        }

        public static long ReadLong()
        {
            long x = 0;
            for (int i = 0; i &lt; 8; i&#43;&#43;)
            {
                char c = ReadChar();
                x &lt;&lt;= 8;
                x |= c;
            }
            return x;
        }

        public static double ReadDouble()
        {
            return BitConverter.Int64BitsToDouble(ReadLong());
        }

        public static float readFloat()
        {
            return BitConverter.ToSingle(BitConverter.GetBytes(ReadInt()),0);
        }

        public static byte ReadByte()
        {
            char c = ReadChar();
            return (byte)(c &amp; 0xff);
        }
    }
```

```c#
    public class BinaryStdOut
    {
        private static  TextWriter output;  // output stream (standard output)
        private static int buffer;                // 8-bit buffer of bits to write
        private static int n;                     // number of bits remaining in buffer
        private static bool isInitialized;     // has BinaryStdOut been called for first time?

        // don&#39;t instantiate
        private BinaryStdOut() { }

        // initialize BinaryStdOut
        private static void Initialize()
        {
        output = Console.Out;
            buffer = 0;
            n = 0;
            isInitialized = true;
        }

       
        private static void WriteBit(bool bit)
        {
            if (!isInitialized) Initialize();

            // add bit to buffer
            buffer &lt;&lt;= 1;
            if (bit) buffer |= 1;

            // if buffer is full (8 bits), write out as a single byte
            n&#43;&#43;;
            if (n == 8) ClearBuffer();
        }

        /**
          * Writes the 8-bit byte to standard output.
          */
        private static void WriteByte(int x)
        {
            if (!isInitialized) Initialize();

            if( x &lt; 0 || x &gt; 256);
                throw new ArgumentException($&#34;{x} is out range&#34;);

            // optimized if byte-aligned
            if (n == 0)
            {
                try
                {
                    output.Write(x);
                }
                catch (IOException e)
                {
                    Console.WriteLine(e.StackTrace);
                }
                return;
            }

            // otherwise write one bit at a time
            for (int i = 0; i &lt; 8; i&#43;&#43;)
            {
                bool bit = (((int)((uint)x &gt;&gt; (8 - i - 1))) &amp; 1) == 1;
                WriteBit(bit);
            }
        }

        // write out any remaining bits in buffer to standard output, padding with 0s
        private static void ClearBuffer()
        {
            if (!isInitialized) Initialize();

            if (n == 0) return;
            if (n &gt; 0) buffer &lt;&lt;= (8 - n);
            try
            {
                output.Write(buffer);
            }
            catch (IOException e)
            {
                Console.WriteLine(e.StackTrace);
            }
            n = 0;
            buffer = 0;
        }

       
        public static void Flush()
        {
            ClearBuffer();
            try
            {
            output.Flush();
            }
            catch (IOException e)
            {
                Console.WriteLine(e.StackTrace);
            }
        }

       
        public static void Close()
        {
            Flush();
            try
            {
                output.Close();
                isInitialized = false;
            }
            catch (IOException e)
            {
                Console.WriteLine(e.StackTrace);
            }
        }

        public static void Write(bool x)
        {
            WriteBit(x);
        }

        public static void Write(byte x)
        {
            WriteByte(x &amp; 0xff);
        }

        public static void Write(int x)
        {
            WriteByte((int)((uint)x &gt;&gt; 24) &amp; 0xff);
            WriteByte((int)((uint)x &gt;&gt; 16) &amp; 0xff);
            WriteByte((int)((uint)x &gt;&gt; 8) &amp; 0xff);
            WriteByte((int)((uint)x &gt;&gt; 0) &amp; 0xff);
        }

        public static void Write(int x, int r)
        {
            if (r == 32)
            {
                Write(x);
                return;
            }
            if (r &lt; 1 || r &gt; 32) throw new ArgumentException(&#34;Illegal value for r = &#34; &#43; r);
            if (x &lt; 0 || x &gt;= (1 &lt;&lt; r)) throw new ArgumentException(&#34;Illegal &#34; &#43; r &#43; &#34;-bit char = &#34; &#43; x);
            for (int i = 0; i &lt; r; i&#43;&#43;)
            {
                bool bit = ((int)((uint)x &gt;&gt; (r - i - 1)) &amp; 1) == 1;
                WriteBit(bit);
            }
        }

        public static void Write(double x)
        {
            Write(BitConverter.DoubleToInt64Bits(x));
        }

        /**
          * Writes the 64-bit long to standard output.
          * @param x the {@code long} to write.
          */
        public static void Write(long x)
        {
            WriteByte((int)((int)((uint)x &gt;&gt; 56) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 48) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 40) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 32) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 24) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 16) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 8) &amp; 0xff));
            WriteByte((int)((int)((uint)x &gt;&gt; 0) &amp; 0xff));
        }

        /**
          * Writes the 32-bit float to standard output.
          * @param x the {@code float} to write.
          */
        public static void Write(float x)
        {
            Write(BitConverter.ToInt32(BitConverter.GetBytes(x),0));
        }

        
        public static void Write(short x)
        {
            WriteByte((int)((uint)x &gt;&gt; 8) &amp; 0xff);
            WriteByte((int)((uint)x &gt;&gt; 0) &amp; 0xff);
        }

        
        public static void Write(char x)
        {
            if (x &lt; 0 || x &gt;= 256) throw new ArgumentException(&#34;Illegal 8-bit char = &#34; &#43; x);
            WriteByte(x);
        }

        
        public static void Write(char x, int r)
        {
            if (r == 8)
            {
                Write(x);
                return;
            }
            if (r &lt; 1 || r &gt; 16) throw new ArgumentException(&#34;Illegal value for r = &#34; &#43; r);
            if (x &gt;= (1 &lt;&lt; r)) throw new ArgumentException(&#34;Illegal &#34; &#43; r &#43; &#34;-bit char = &#34; &#43; x);
            for (int i = 0; i &lt; r; i&#43;&#43;)
            {
                bool bit = ((int)((uint)x &gt;&gt; (r - i - 1)) &amp; 1) == 1;
                WriteBit(bit);
            }
        }

        public static void Write(String s)
        {
            for (int i = 0; i &lt; s.Length; i&#43;&#43;)
                Write(s[i]);
        }

       
        public static void Write(String s, int r)
        {
            for (int i = 0; i &lt; s.Length; i&#43;&#43;)
                Write(s[i], r);
        }
    }
```

测试

```c#
        [TestMethod]
        public void BinaryPumpTest()
        {
            int bitsPerLine = 8;

            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\abra.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                Console.SetIn(stream);
                int count;
                for (count = 0; !BinaryStdIn.IsEmpty(); count&#43;&#43;)
                {
                    if (count != 0 &amp;&amp; count % bitsPerLine == 0) Console.WriteLine();
                    if (BinaryStdIn.ReadBoolean()) Console.Write(1);
                    else Console.Write(0);
                }
                if (bitsPerLine != 0) Console.WriteLine();
                Console.WriteLine(count &#43; &#34; bits&#34;);
            }
            // ABRACADABRA!
            // 01000001 //65
            // 01000010 //66
            // 01010010 //52
            // 01000001 //65
            // 01000011
            // 01000001
            // 01000100
            // 01000001
            // 01000010
            // 01010010
            // 01000001
            // 00100001
            // 96 bits
        }

```

不存在能够压缩任意比特流的算法。

### 基因数据

![image-20210507151527747](/algorithm/image-20210507151527747.png)

双位编码压缩

基因的一个简单性质是，它由4 种不同的字符组成。这些字符可以用两个比特编码，

基因都是4个编码，用2个bit表示基因四个字符，byte可以存储8bit，存储4个基因。

```c#
        public static void Compress()
        {
            Alphabet DNA = Alphabet.DNA;
            String s = BinaryStdIn.ReadString();
            int n = s.Length;
            BinaryStdOut.Write(n);
            Console.WriteLine();
            // Write two-bit code for char. 
            for (int i = 0; i &lt; n; i&#43;&#43;)
            {
                short d =(short) DNA.ToIndex(s[i]);//0,1,2,3
                BinaryStdOut.Write(d, 2);//输入 2 bit
            }
            BinaryStdOut.Close();
        }
```

```c
        public static void Expand()
        {
            Alphabet DNA = Alphabet.DNA;

            // Read two bits; write char. 
            for (int i = 0; i &lt; 33; i&#43;&#43;)
            {
                char c = BinaryStdIn.ReadChar(2);//指定将2bit转化成char
                Console.Write(DNA.ToChar(c));
            }
            BinaryStdOut.Close();
        }
```

测试

```c#
        [TestMethod]
        public void GenomeCompressTest()
        {

            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\genomeTiny.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {

                Console.WriteLine(&#34;Compress&#34;);
                Console.SetIn(stream);
                Genome.Compress();
            }
            //ATAGATGCATAGCGCATAGCTAGATGTGCTAGC
            //292dÉÈîr@
        }
        [TestMethod]
        public void GenomeExpandTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\genomeTinyExpand.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {

                Console.WriteLine(&#34;Expand&#34;);
                Console.SetIn(stream);

                Genome.Expand();
            }
            //ATAGATGCATAGCGCATAGCTAGATGTGCTAGC
        }
```

### 游程编码(Run-Length Encoding)

比特流中最简单的冗余形式就是一长串重复的比特。

```c
0000000000000001111111000000011111111111
```

该字符串含有15个0，7个1，7个0，11个1，将该比特字符串编码为15，7，7，11，所有的1和0都是交替出现的，用4位表四长度以连续的0作为开头，就可以得到16位长的字符串。

```c
1111011101111011
```

压缩率为16/40=40%

需要解决几个问题：

&#43; 应该使用多少比特来记录游程的长度？

&#43; 当某个游程的长度超过了能够记录的最大长度时怎么办？ 
&#43; 当游程的长度所需的比特数小于记录长度的比特数时怎么办？

如何解决

&#43; 游程长度应该在0 到255 之间，使用8 位编码
&#43; 在需要的情况下使用长度为0 的游程来保证所有游程的长度均小于256;
&#43; 我们也会将较短的游程编码，虽然这样做有可能使输出变得更长。

```c#
    public class RunLength
    {
        private static  int R = 256;
        private static  int LG_R = 8;

        public static void Expand()
        {
            bool b = false;
            while (!BinaryStdIn.IsEmpty())
            {
                int run=BinaryStdIn.ReadInt(LG_R);
                for (int i = 0; i &lt;run; i&#43;&#43;)
                    BinaryStdOut.Write(b);
                b = !b;
            }
            BinaryStdOut.Close();
        }

        public static void Compress()
        {
            byte run = 0;
            bool old = false;
            while (!BinaryStdIn.IsEmpty())
            {
                bool b = BinaryStdIn.ReadBoolean();
                if (b != old)
                {
                    BinaryStdOut.Write(run, LG_R);
                    run = 1;
                    old = !old;
                }
                else
                {
                    if (run == R - 1)//如果超过256位了就重新计算
                    {
                        BinaryStdOut.Write(run, LG_R);
                        run = 0;
                        BinaryStdOut.Write(run, LG_R);//跳过下一个变换，就是0个1
                    }
                    run&#43;&#43;;
                }
            }
            BinaryStdOut.Write(run, LG_R);
            BinaryStdOut.Close();
        }


    }
```

测试

```c#
        [TestMethod]
        public void RunLengthCompressTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\4runs.bin&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {

                Console.WriteLine(&#34;Compress&#34;);
                Console.SetIn(stream);
                RunLength.Compress();
            }
        }
        [TestMethod]
        public void RunLengthExpandTest()
        {
            var data = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, &#34;Data\\4runsExpand.txt&#34;);
            Console.WriteLine(data);
            using (StreamReader stream = new StreamReader(data))
            {
                Console.WriteLine(&#34;Expand&#34;);
                Console.SetIn(stream);
                RunLength.Expand();
            }
        }
```

游程编码广泛用于位图的主要原因是，随着分辨率的提高它的效果也会大大的提
高。证明这一点很简单。假设将上一个例子中的分辨率提高一倍，

&#43; 总比特数变为了原来的4 倍；
&#43; 游程的数最变为约原来的2 倍;
&#43; 游程的长度变为约原来的2 倍；
&#43; 压缩后的比特数量变为约原来的2倍；
&#43; 压缩率变成了原来的一半！

### 霍夫曼压缩

主要思想是放弃文本文件的普通保存方式： 不再使用7 位或8 位二进制数表示每一个字符，而是用较少的比特表示出现频率高的字符，用较多的比特表示出现频率低的字符。

假设需要将字符串A B R A C A D A B R A ! 编码。由7 位ASCII 字符编码我们可以得到比特字符串：

![image-20210508105926556](/algorithm/image-20210508105926556.png)

要将这段比特字符串解码，只需每次读取7 位并根据ASCII 编码表将它转换为字符，在这种标准的编码下，只出现了一次的D 和出现了5 次的A 所需的比特数是一样的

霍夫曼压缩的思想是通过用较少的比特表示出现频繁的字符而用较多的比特表示偶尔出现的字符来节省空间，这样字符串所使用的总比特数就会降低。

我们可以试着将最短的比特字符串赋予最常用的字符，将A 编码为0 、B 编码为1 、R 为00 、C 为0 1 、D 为10 、！为II 。这样A BRACADABRA! 的编码就是01000010100100011 。这种表示方法只用了17 位，而7 位的ASCII 编码则用了77 位。但这种表示方法并不完整，因为它需要空格来区分字符。如果所有字符编码都不会成为其他字符编码的前缀，那么就不需要分隔符了,添加前缀A为0，B为1111，C为110，D为100，R为1110，！为101.

![image-20210508112113963](/algorithm/image-20210508112113963.png)

#### 前缀单词查找树

表示前缀码的一种简便方法就是使用单词查找树，事实上，任意含有M 个空链接的单词查找树都为M 个字符定义了一种前缀码方法：我们将空链接替换为指向叶子结点（含有两个空链接的结点）的链接，每个叶子结点都含有一个需要编码的字符。

![image-20210508112555904](/algorithm/image-20210508112555904.png)

不同的前缀是影响压缩率的，寻找最优前缀码的通用方法是D .Huffinan 在1952 年发现的（当时他还是个学生），因此被称为霍夫曼编码。

使用前缀码进行数据压缩需要经过5 个主要步骤。

&#43; 构造一棵编码单词查找树；
&#43; 将该树以字节流的形式写入输出以供展开时使用；
&#43; 使用该树将字节流编码为比特流。

在展开时需要：

&#43; 读取单词查找树（保存在比特流的开头）
&#43; 使用该树将比特流解码。

#### 单词查找树结点

```c#
public  class Node : IComparable&lt;Node&gt;
{
    public  char ch;
    public  int freq;
    public  Node left, right;

    public Node(char ch, int freq, Node left, Node right)
    {
        this.ch = ch;
        this.freq = freq;
        this.left = left;
        this.right = right;
    }

    // is the node a leaf node?
    public bool IsLeaf()
    {
        if(((left == null) &amp;&amp; (right == null)) || ((left != null) &amp;&amp; (right != null)))
        {
            return (left == null) &amp;&amp; (right == null);
        }
        else
        {
            throw new ArgumentException(&#34;is not leaf&#34;);
        }

    }

    // compare, based on frequency
    public int CompareTo(Node that)
    {
        return this.freq - that.freq;
    }
}
```

被编码的字符放在叶子结点中并在每个结点中维护了一个名为freq 的实例变最来表示以它为根结点的子树中的所有字符出现的频率。

#### 前缀码展开

![image-20210508125419538](/algorithm/image-20210508125419538.png)

#### 使用前缀码压缩

对千任意单词查找树，它都能产生一张将树中的字符和比特字符串（用由0 和1
组成的String 字符串表示）相对应的编译表。编译表就是一张将每个字符和它的比特字符串相关联的符号表： 为了提升效率，我们使用了一个由字符索引的数组st[] 而非普
通的符号表，因为字符的数最并不多。在构造该符号表时，buildCode () 递归遍历整棵树并为每个结点维护了一条从根结点到它的路径所对应的二进制字符串（ 0 表示左链接， 1 表示右链接）

```c#
       /// &lt;summary&gt;
       /// 使用单词查找树构造编译表(递归)
       /// &lt;/summary&gt;
       /// &lt;param name=&#34;st&#34;&gt;&lt;/param&gt;
       /// &lt;param name=&#34;x&#34;&gt;&lt;/param&gt;
       /// &lt;param name=&#34;s&#34;&gt;&lt;/param&gt;
        private static void BuildCode(String[] st, Node x, String s)
        {
            if (!x.IsLeaf())
            {
                BuildCode(st, x.left, s &#43; &#39;0&#39;);
                BuildCode(st, x.right, s &#43; &#39;1&#39;);
            }
            else
            {
                st[x.ch] = s;
            }
        }
```

#### 单词查找树构造

![image-20210508133445163](/algorithm/image-20210508133445163.png)

我们将需要被编码的字符放在叶子结点中并在每个结点中维护了一个名为freq 的实例变最来表示以它为根结点的子树中的所有字符出现的频率。构造的第一步是创建一片由许多只有一个结点（ 即叶子结点）的树所组成的森林。每棵树都表示输入流中的一个字符，每个结点中的freq 变戴的值都表示了它在输入流中的出现频率。在我们的例子中，输入含有8 个t, 5 个e, 11 个空格等（特别提示： 为了得到这些频率，需要读取整个输入流－**霍夫曼编码是一个两轮算法， 因为需要再次读取输入流才能压缩它**)。接下来自底向上根据频率构造这棵编码的单词查找树。在构造时将它看作一棵结点中含有频率信息的二叉树； 在构造后，我们才将它看作一棵用千编码的单词查找树。构造过程如下： **首先找到两个频率最小的结点，然后创建一个以二者为子结点的新结点**（新结点的频率值为它的两个子结点的频率值之和） 。这个操作会将森林中树的数量减一。然后不断重复这个过程，找到森林中的两棵频率最小的树并用相同的方式创建一个新的结点。用优先队列能够轻易实现这个过程，随着这个过程的继续，我们构造的单词查找树将越来越大， 而森林中的树会越来越少。最终，所有的结点会被合并为一棵单独的单词查找树。

LF换行，SP空格

![image-20210508134525734](/algorithm/image-20210508134525734.png)

![image-20210508150934532](/algorithm/image-20210508150934532.png)

#### 最优性

在树中高频率的字符比低频率的字符离根结点更近，因此编码所需的比特更少，所以这种编码的方式更好。

加权外部路径长度:它是所有叶子结点的权重（频率）和深度之积的和。

给定一个含有r 个符号的集合和它们的频率，霍夫曼算法所构造的前缀码是最优的。

#### 写入和读取单词查找树

![image-20210508151529396](/algorithm/image-20210508151529396.png)

第一位是0, 对应着根结点；下一个遇到是含有A 的叶子结点，因此下一位为1, 紧接着是01000001 , 即&#34;A&#34; 的8 位ASCII 编码。下两位均为0, 因为遇到的都是两个内部结点，

首先读取一个比特以得到当前结点的类型，如果是叶子结点（比特为1) 那么就读取字符的
编码并创建一个叶子结点；如果是内部结点（比特为0) 那么就创建一个内部结点并（递归地）继续构造它的左右子树。

#### 霍夫曼压缩实现

压缩

&#43; 读取输入
&#43; 将输入中的每个char 值的出现频率制成表格；
&#43; 根据频率构造相应的霍夫曼编码树；
&#43; 构造编译表，将输入中的每个char 值和一个比特字符串相关联；
&#43; 将单词查找树编码为比特字符串并写入输出流；
&#43; 将单词总数编码为比特字符串并写入输出流；
&#43; 使用编译表翻译每个输入字符

展开

&#43; 读取单词查找树（编码在比特流的开头）；
&#43; 读取需要解码的字符数量；
&#43; 使用单词查找树将比特流解码。



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2021/05/algorithm12/  

