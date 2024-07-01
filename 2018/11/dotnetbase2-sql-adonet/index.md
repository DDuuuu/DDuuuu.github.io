# DotNet基础 入门




## 基本信息

### CIL和JIT

&#43; CIL通用中间语言
&#43; JIT  just-in-time使得CIT代码仅在需要时才编译

### 程序集

&#43; 包含可执行文件.exe和库函数.dll和资源文件，不必把程序集集中到一个地方，全局程序缓存
&#43; 程序集完全自描述的，逻辑单元而不是物理单元
&#43; 可执行代码和库代码使用相同程序集结构，可执行文件多了一个主程序入口点。
&#43; 程序集的一个重要特征是它们包含**元数据描**述了对应代码中定义的类型和方法。

### CLR 

公共语言运行库CLR：包含实时编译器JIT,在程序运行时，JIT编译器会从IL代码生成本地代码，其他部分是垃圾回收器GC,调试器扩展和线程实用工具。垃圾回收器负责回收内存，调试扩展器允许在不同编程语言之间启动调试会话，线程实用工具负责在底层平台创建线程。

&#43; 管理着正在执行的代码包括管理内存，处理安全以及跨语言调试
&#43; 代码托管最重要的是内存回收机制
&#43; winform基于像素
&#43; wpf基于pirectX

CLR执行应用程序之前，编写好源代码都需要编译，编译分为两个阶段

&#43; 将源代码编译成microsoft中间语言IL
&#43; CLR将I编译成平台专用的的本地代码

clr还有一个类型加载器的类型系统，类型加载器负责从程序集中加载类型。 

### 编译过程

&#43; .net兼容语言编写程序，托管语言
&#43; 将代码编译为中间代码CIL，这未必是单文件，可以有多个源代码文件，再把链接带一个程序集中，称之为链接
&#43; 使用JIT编译为本机代码
&#43; 在托管的CLR环境下运行本机代码

## 基本语法

### 注释

&#43; /*             */和一行//
&#43; ///可以通过配置，将这些注释提取出来组成文档文件

### 命名空间



### 代码大纲功能

&#43; \#region Using directives
&#43; \#endregion，大纲的名字为Using directives

### 变量

&#43; 先声明后使用

变量初始化：变量是类或结构中的字段，如果没有显式初始化，则创建这些变量时，其默认值就是0.方法的局部变量必须在代码中显示初始化。

&#43; 整数
  &#43; sbyte System.SByte -128~127
  &#43; byte   System.Byte   0~255
  &#43; short  System.Int16   -32768~32767
  &#43; ushort  System.UInt16  0~65535
  &#43; int    System.Int32       -21亿~21亿
  &#43; uint   System.UInt32          42亿
  &#43; long   System.Int64            19位数字
  &#43; ulong   System.UInt64          20位数字
&#43; 浮点类型
  &#43; 位是最小数据单位，只能表示0-1
  &#43; 字节，**8个二进制位构成1个字节，是存储空间的基本计量单位**，
  &#43; 字：由若干字节构成，不同计算机有不同的字长，8位计算机一个字等于一个字节，16位一个字等于两个字节，字是计算机数据处理和运算的单位
  &#43; 1kb等于1024个字节
  &#43; 1M等于1024KB	
  &#43; 1G等于1024M
  &#43; 1T等于1024G
  &#43; float   System.Single               4字节
  &#43; double  System.Double			8个字节
  &#43; decimal System.Decimal		16个字节
&#43; 布尔和文本
  &#43; char   System.Char     Unicode     0~65535     2个字节
  &#43; bool   System.Boolean                               1个字节
  &#43; string System.String	
&#43; 转义字符
  &#43; \&#39;      单引号   0x0027
  &#43; \&#39;&#39;    双引号     0x0022
  &#43; \\    反斜杠      0x005c
  &#43; \0    空            0x0000
  &#43; \a    警告         0x0007
  &#43; \b    退格       0x0008
  &#43; \f     换页      0x000C
  &#43; \n     换行    0x0000A
  &#43; \r     回车     0x000D
  &#43; \t     水平制表符    0x0009
  &#43; \v   垂直制表符     0x000B
  &#43; @转义字符，避免大量使用\
&#43; 全局变量：Program.全局变量，在声明变量前要进行初始化

&#43; 常量：
  &#43; 常量必须初始化，指定值后不能更改。
  &#43; 常量总是隐式静态的，

&#43; 值类型和引用类型：值类型存储**在栈**中，应用类型存储在**堆**中。值类型和引用类型互换要经过装箱拆箱，在传递函数参数时，值类型会**进行复制**，而引用相当于**传递指针**，返回值相同。引用类型由垃圾回收器进行回收，值类型不需要，超出其作用域就会在内存中删除。
&#43; ref:如果是结构类型使用ref传递参数，则变成传递引用，**但如果是引用类型，即使参数在函数中变化了引用，新的应用还是会传递回去。**ref传递的值要先初始化。
&#43; out：用法基本和ref一样，但传入的值只需要定义
&#43; 可空类型：int? 和int,唯一的多开销是一个可以确定它是否为空 的布尔成员，值可以直接转换可空，int?=int;可空转换成值需要强制int=(int)int?;但如果为空会生成一个异常，最好的方法是int=int?.hasvalue?int?.value:-1；可以转换成较短短语int=int??-1
&#43; 枚举也是值类型：默认情况下，枚举是int,也可以改变成其他整数类型，强制转换将int转换成枚举，当分配给常量是不同位时，flags属性需要枚举设置，获得所有枚举，var day in Enum.GetName(typeof(Color));
&#43; 结构：值类型，不能继承，每个结构都自动由ValueType派生。

### 基本运算符

&#43; 一目运算符
&#43; 二目运算符
  &#43; =
  &#43; &#43;=
  &#43; -=
  &#43; *=
  &#43; /=
  &#43; %=
  &#43; 运算符优先级
    &#43; &#43;&#43;,--(前缀);(),&#43;,-,!,~
    &#43; *,/,%
    &#43; &#43;,-
    &#43; \&gt;&gt; ,&lt;&lt;
    &#43; &lt;,&gt;,&lt;=，&gt;=
    &#43; ==,！=
    &#43; &amp;
    &#43; ^
    &#43; |
    &#43; &amp;&amp;
    &#43; ||
    &#43; =,&#43;=,-=,*=,/=,%=
    &#43; &#43;&#43;,--后缀
&#43; 布尔运算符
  &#43; ！
  &#43; &amp;
  &#43; |            
  &#43; ^一真一否才为真

### 基本语句

&#43; goto语句

  goto `&lt;labelName&gt;`

  `&lt;labelName&gt;`:

&#43; if语句 

```C#
if(&lt;test&gt;)
{
	&lt;code executed if &lt;testis true&gt;;
}
else
{
	&lt;code executed if &lt;testis false&gt;;
}
```

&#43; switch

```C#
switch(&lt;testvar&gt;)
{
	case&lt;comparsionVall&gt;:
		&lt;code to execute if &lt;testVar&gt;&gt;==&lt;comparisonvall&gt;&gt;
		break;
........
	default:
		&lt;code to execute if &lt;testVar&gt;&gt;!=&lt;comparisonvall&gt;&gt;
		break;
}
```

&#43; do 

```C#
do
{
	&lt;code to be looped&gt;
} while(&lt;Test&gt;)
```

&#43; while

```C#
while(&lt;test&gt;)
{
	&lt;code to looped&gt;
}
```

&#43; for

```C#
for(&lt;initialization&gt;;&lt;condition&gt;;&lt;operation&gt;)
{
	&lt;code to loop&gt;
}
```

&#43; foreach

```C#
foreach(&lt;baseType&gt;&lt;name&gt;in&lt;array&gt;)
{
	//can use &lt;namefor each element
}//该函数只能只读    
```

&#43; 循环中的中断
  &#43; 循环的中断：
  &#43; break立刻终止循环
  &#43; continue立刻终止当前循环
  &#43; goto可以跳出循环
  &#43; return跳出循环,即包含循环的函数

### 基本数据结构

&#43; 转换
  &#43; 隐式转换：编译器执行转换
  &#43; 显示转换：需要额外转换
    `(&lt;destinationType&gt;)&lt;sourceVar&gt;`
  &#43; 显示转换的检查：checked和unchecked
  &#43; checked(
    `&lt;expression&gt;`)检查是否会溢出，溢出则出现错误
  &#43; unchecked(`&lt;expression&gt;`)溢出不出现错误
    可以通过设置属性，使得显示转换都默认检查checked

&#43; 枚举

```C#
enum&lt;typename&gt;:&lt;underlyingType&gt;
{
	&lt;value1&gt;,
	&lt;value2&gt;=&lt;actualValue2&gt;,
	...
}
```

&#43; 结构体：值类型

```C#
struct&lt;typename&gt;:&lt;underlyingType&gt;
{
	&lt;memberDeclarations&gt;,
	
	...
}
```

&#43; 数组
  &#43; 数组声明：`&lt;baseType&gt;[]&lt;name&gt;;`
  &#43; 二维数组声明：`&lt;baseType&gt;[,]&lt;name&gt;;`
  &#43; 数组调用：`frendName[index]`
  &#43; 锯齿数组

## 基本函数

&#43; Console
  &#43; Console.WriteLine(&#34;字符串&#34;);
  &#43; Console.WriteLine(&#34;{0}{1}{2}&#34;,n_a,n_b,n_c);
  &#43; str=Console.ReadLine();
&#43; Convert
  &#43; n_double=Convert.ToDouble();强制转换成double
  &#43; n_int=Convert.ToInt32();强制转换成int
  &#43; ToBase64CharArray	将 8 位无符号整数数组的子集转换为用 Base 64 数字编码的 Unicode 字符数组的等价子集。
  &#43; ToBase64String	将 8 位无符号整数数组的值转换为它的等效 String 表示形式（使用 base 64 数字编码）。
  &#43; ToBoolean	 将指定的值转换为等效的布尔值。
  &#43; ToByte	 将指定的值转换为 8 位无符号整数。
  &#43; ToChar	 将指定的值转换为 Unicode 字符。
  &#43; ToDateTime	 将指定的值转换为 DateTime。
  &#43; ToDecimal	 将指定值转换为 Decimal 数字。
  &#43; ToDouble	 将指定的值转换为双精度浮点数字。
  &#43; ToInt16	 将指定的值转换为 16 位有符号整数。
  &#43; ToInt32	 将指定的值转换为 32 位有符号整数。
  &#43; ToInt64	 将指定的值转换为 64 位有符号整数。
  &#43; ToSByte	 将指定的值转换为 8 位有符号整数。
  &#43; ToSingle	 将指定的值转换为单精度浮点数字。
  &#43; ToString	 将指定值转换为其等效的 String 表示形式。
  &#43; ToUInt16	 将指定的值转换为 16 位无符号整数。
  &#43; ToUInt32	将指定的值转换为 32 位无符号整数。
  &#43; ToUInt64	 将指定的值转换为 64 位无符号整数。
  &#43; Convert.ReadKey( );
&#43; enum
  &#43; `(enumerationType)Enum.Parse(typeof (enumerationType),enumerationValueString);`字符串转换成枚举
&#43; string
  &#43; .ToCharArray()这个函数可以将String编程char[]数组
  &#43; .Length字符串长度
  &#43; .ToLower字符串全部小写
  &#43; .ToUpper字符串全部大写
  &#43; .Trim字符串删除空格
  &#43; .PadLeft()和.PadRight()字符串左边或者右边添加空格
  &#43; .IndexOf(&#39;,&#39;);定位到，的下标
  &#43; .Split(&#39; &#39;);切片
&#43; object
  &#43; 类的函数基本是object函数
  &#43; .ToString()输出类名；
&#43; 函数格式

```C#
static &lt;returnType&lt;FunctionName&gt;(&lt;paramType&gt;&lt;paraName&gt;,...)
{
	...
	renturn &lt;returnValue&gt;;
}//普通函数定义
```

可以直接定义表达式函数`public bool IsSquare(Reactangle rect)=&gt;rect.height==rect.width;`

&#43; 参数函数
  &#43; 参数数组，C#允许函数指定一个特殊参数，该参数必须是函数的最后一个，称为参数素组，

```C#
static &lt;returnType&lt;FunctionName&gt;(&lt;p1Type&gt;&lt;p1Name&gt;,...Params&lt;type&gt;[]&lt;name&gt;)
{
	...
	renturn &lt;returnValue&gt;;
}
```

```c#
class Prigram
{
static int SunVals(params int [] vals)
{
int sum=0;
foreach(int val in vals)
{
sum&#43;=val;
}
return sum;
}
static void Main(string [] args)
{
int sum=SumVals(1,5,2,9,8);
console.writeline(&#34;summed values={0}&#34;,sum);
}
}
```

&#43; 可选参数

```C#
public void TestMethod(int i,int j=12)
{
}
```

&#43; 引用参数
  &#43; 引用参数，需要用关键字ref
  &#43; 调用引用函数`&lt;FunctionName&gt;(ref &lt;paraName&gt;)`

```c#
static &lt;returnType&lt;FunctionName&gt;(ref &lt;paramType&gt;&lt;paraName&gt;,...)
{
	...
	renturn &lt;returnValue&gt;;
}
```

&#43; 扩展方法
  `public static int GetWordCount(this string s)=&gt;s.split().length;`
  调用`int wordcount=fox.GetWordCount();`   

&#43; 函数重载
  &#43; 特征标不一样就行
  &#43; 引用和非引用也属于重载
&#43; 委托
  &#43; 委托，和重载差不多，只不过重载特征表不同，而委托特征标相同而函数名不同，可以根据需求NEW出不同的函数，委托的利用在于可以将其当成参数传递给函数，让函数因需求执行不同的操作，类似于多态
  &#43; 委托是一种存储函数引用的类型
  &#43; 委托声明类似于函数，但是不带函数体，且要使用关键字delegate，委托指定了一个返回类型和一个参数列表
  &#43; 定义了委托后，就可以声明该委托类型的变量，接着把这个变量初始化为与委托具有相同返回类型和参数列表的函数引用，之后就可以使用委托变量调用这个函数
  &#43; 例子，可以把委托变量作为参数传递给一个函数，这样，该函数就可以使用委托调用它引用的任何函数

```c#
class Program
{
    delegate double ProcessDelegate(double param1, double param2);
    static double Multiply(double param1, double param2)
    {
        return param1 * param2;
    }
    static double Divide(double param1, double param2)
    {
        return param1 / param2;
    }
    static void Main(string[] args)
    {
        ProcessDelegate process;
        Console.WriteLine(&#34;imput two intege：&#34;);
        string input = Console.ReadLine();
        string [] commaPos = input.Split(&#39;,&#39;);
        double param1 = Convert.ToDouble(commaPos[0]);
        double param2 = Convert.ToDouble(commaPos[1]);
        Console.WriteLine(&#34;Enter M to Multiply or D to divide:&#34;);
        input = Console.ReadLine();
        if (input == &#34;M&#34;)
            process = new ProcessDelegate(Multiply);//process=Multiply;
        else
            process = new ProcessDelegate(Divide);//process=Divide;
        Console.WriteLine(&#34;result :{0}&#34;, process(param1, param2));
        Console.ReadKey();
    }
} 
```

## 调试

### 断点

&#43; 可以查看输出窗口，输出/调试，
&#43; 语句Debug.WriteLine()将调试信息显示到调试窗口
&#43; Trace.WriteLine()，用法相同，可以用于发布
&#43; Debug.WriteLine(&#34;Add 1 to i&#34;,&#34;MyFunc&#34;);结果为：
&#43; MyFunc:Add 1 to i;
&#43; using System.Diagnostics;
&#43; Debug.WriteLineIf()增加一个必选项，当是真的时候进行输出

### 跟踪点

&#43; 有点类似于Debug.WriteLine()

### Trace

&#43; `Trace.Assert(myVar&lt;0,&#34;variable out of bounds&#34;,&#34;please contact vendor with the error code KCW001.&#34;)`



![image-20240701151156966](/dotnet/image-20240701151156966.png)

### 异常

&#43; try抛出异常，catch抛出异常时执行的代码，finally包含最终执行的代码
&#43; 即使try中有return， finally还是会执行，在finally中还是会改变return中的值

```C#
try
{...}
catch(&lt;exceptionType&gt;e)
{...}
finally
{...}
```

&#43; 抛出异常包括内容
  `throw(new ArgumentOutOfRangeException(&#34;MyIntProp&#34;,value,&#34;MyIntProp must be assigned a value between 0 and 10&#34;));`
&#43; catch中捕获异常显示

```C#
catch (Exception e)
{
   Console.WriteLine(&#34;Exception {0} thrown.&#34;, e.GetType().FullName);
   Console.WriteLine(&#34;Message:\n\&#34;{0}\&#34;&#34;, e.Message);
}
```

## 面向对象编程

类包括：字段/属性/方法/常量/构造函数/索引器/运算符/事件/类型/析构函数

字段：不应该被设置为public

属性：自动实现的属性也就是没有声明私有字段，就可以使用初始化器来初始化；`public int Age{set;get;}=42;` set和get可以被访问修饰符修饰，但get和set中必须有一个具有属性的访问级别，属性其实并不怎么消耗资源，因为在JIT中，被变成内联函数，

readonly:readonly只能在构造函数中赋值，然后就不好修改。

### 构造函数

&#43; 静态构造函数
  &#43; 创建包含静态构造函数的类实例
  &#43; 访问包含静态构造函数的类的静态成员时，（无论创建多少个实例，静态构造函数只调用一次。）
&#43; 构造函数和析构函数，和C&#43;&#43;一样
&#43; 静态类不能实例化对象，只能包含静态成员，
&#43; 如果没有构造函数，编译器会自动添加一个默认构造函数


### 派生类构造函数的调用

构造函数总是按照层次结构顺序调用的：先调用object的构造函数，然后从上到下一次调用，直到达到编译器要实例化的类为止

如果要调用基类的非默认构造函数就需要使用构造函数初始化器。



### 访问修饰符

public/ 所有类型成员/任何代码均可以访问该项
protected/类型和内嵌类型的所有成员/只有派生类型能够访问该类
internal/所有类型和成员/只能包含他的程序集中能够访问该类
private/类型和内嵌类型的所有成员/只能在它所属的类型中访问该项
protected internal/类型和内嵌类型的所有成员/只能在包含他的程序集和派生类型的任何代码中访问该项

new/static /virtual/abstract/override/sealed/extern 


### 字段和属性

&#43; 统一建模语言Unified Modeling Language(UML)

### 接口

&#43; 接口可以看成是类的引用，可以引用任何实现该接口的类
&#43; 一般情况下接口只能包含：方法，属性，索引器，事件的声明
&#43; 把公共实例(非静态)方法和属性组合起来，以封装特定功能的一个集合,接口不能实例化
&#43; IDisposable接口的对象必须实现其Dispose()方法，当不再需要某个对象时就调用这个方法，释放重要资源，
&#43; C#简化了这种方法，using关键字可以在代码块中初始化使用重要资源的对象，这个代码块的结尾会自动调用Dispose()方法

```C#
&lt;ClassName&gt;&lt;VariableName&gt;=new &lt;ClassName&gt;();
...
using(&lt;VariableName&gt;)
{
...
}
或
using(&lt;ClassName&gt;&lt;VariableName&gt;=new &lt;ClassName&gt;())
{...}
```

&#43; 接口可以彼此集成，和类的集成差不多

### 继承

&#43; C#类可以派生自另一个类和多个接口，只有虚方法和抽象方法才能使用override重写，不然是不能重写的，如果不重写虚方法或抽象方法，需要使用关键字new，那么调用的时候调用子类方法就可以使用base.方法
&#43; 纯虚基类，重写方法，也可以不重写，虚方法可以实现，也可以实现（同属性）
&#43; 基类成员的访问性 protected\privated\public
&#43; 抽象基类不能实例化，抽象基类必须被继承，成员没有实现代码，在派生类中实现他们
&#43; 抽象基类在UML中为斜体

### is and  as

`IBankAccount account= o as IBankAccount;`
`if(o is IBankAccount){IBankAccount account = (IBankAccount)o;}`

### 多态

&#43; 通过实例化不同的派生类调用派生类的重写方法
&#43; 将派生类赋给基类调用基类重写方法，实际上调用了基类的重写方法
&#43; 接口的多态性，尽管不能实例化接口，可以建立接口类型的变量，然后就可以在支持该接口的对象上，使用这个变量来访问接口提供的方法和属性
  &#43; 例如不适用基类提供的方法，而是把该方法放在接口上，实例化也支持该接口，唯一的区别是方法的实现代码不一样（接口不包含方法的实现），**通过接口可以实现访问多个对象的方法而不依赖于一个公共的基类**	

```C#
Cow myCow=new Cow();
Chicken myChicken=new Chicken();
IConsume consumeInterface;
consumeInterface = myCow;
consumeInterface.EatFood();
consumeInterface = myChicken;
consumeInterface.EatFood();
```

&#43; 对象之间的关系
  &#43; **包含关系**，这个成员字段可以是公共字段，此时是继承关系一样，容器对象的用户就可以访问它的方法和属性，与继承不同的是，不能访问类的内部代码，类变成其私有变量
  &#43; 集合关系，增加了功能的数组

### 密封类和密封方法

如果不允许创建派生自某个自定义类的类，该自定义类就应该是密封的，sealed

&#43; 使用密封类可以提高性能，编译器知道该类没有派生，因此就没有虚函数表
&#43; string是密封的


### 托管和非托管

托管和非托管资源，存储在托管或本机堆中的对象，垃圾回收器释放存储在托管堆中的对象，却不会释放本机堆中的对象。

值数据类型：window使用虚拟寻址系统，可以将程序所使用的内存地址映射内存中的实际地址（编译器来做）。每个进程都会有4G的内存，该内存被称为虚拟内存，从0开始往下排。在虚拟内存中有一个地方叫做栈，栈存储的是**不是对象成员的值数据类型**，调用方法时，栈存储传递给方法的所有参数副本。

栈实际上是向下填充的，先0x800000,然后0x799999

```C#
int a=10;
double b=7999;
```

在运行到a=10时，int是四个字节，0x799999-0x799996被10占用，下一个空闲单元是0x799995,double 8个字节，就是0x799987 超出b的定义域后就会加8个字节，超出a的定义域后就会加4个字节。
ab进入栈的顺序是由编译器来决定的，编译器会查看作用域的顺序来决定哪个先进栈哪个后进栈。


引用类型的堆：堆上的内存是向上分配的

```C#
void Work()
{
Customer arabel;//1
arabel= new Customer();//2
Customer othercustomer2=new Customer();//3
}
```

首先1声明一个customer的引用arabel，在栈上给这个引用分配存储空间，该引用为4个字节，
2分配堆上的内存，以存储一个真正的Customer对象，然后把变量arabel的值设置为新分配的customer对象的内存地址，该customer可能包含32个字节，类的字节数和类的字段等成员有关

3是声明一个customer的引用，放在栈上，实例化一个customer对象放在堆上，将堆上的地址放到栈的引用上。

非托管堆：

当一个引用变量超出作用域时，它的引用会从栈中删除，但引用的对象任然在堆中，一直到程序终止，或垃圾回收器回收他们。

垃圾回收，删除堆中不再有被引用的所有对象，垃圾回收器（C&#43;&#43;垃圾回收）在引用跟表中找到所有引用对象，在引用对象树中查找，在完成删除后，堆会立即把对象分散开来，与已经释放的内存混在一起。

托管堆：

如果托管堆也是这样，其给新对象分配内存时就是一个很难处理的问题，运行库必须遍历整个堆，才能找到足够大内存来存储新的对象。但垃圾回收器不会让堆处在这种状态，它在释放完可以释放的资源后，会将其他对象回推到堆的底部，再次形成连续数据块，在移动时所用引用都要更新，这需要垃圾回收器来完成。

垃圾回收期的压缩操作就是托管堆和非托管堆的区别，托管堆虽然要压缩，但分配内存时，只需要地址就好，不需要遍历地址列表。

创建对象时，**会把对象放到托管堆上称为0代，创建新对象会被移动到这一部分，第一次回收后，保留下来的内容会被压缩移动到堆的下一部分上或世代部分，第1代对应部分，此时第0代对应的部分为空，新的对象依然被放到这个部分，遗留下来的内容让在第1代部分，第二次回收，第1代被压缩到第2代部分，第0代被压缩到第1代，第0代为空，放新的内容。如果新的对象超出第0代部分就会进行垃圾回收**

垃圾回收可以提供应用程序性能，可以在架构堆上处理较大对象的方式，

.NET较大对象有自己的堆，称为大对象堆，对象大于85000个字节就会被放在这个堆上，而不是主堆上，因为较大对象的压缩代价比较大，所以不放在主堆上压缩

第二代和大对象堆的回收即压缩过程是在后台线程中进行的


&#43; 强引用和弱引用

仍在**引用的对象的内存为强引用**，可以回收不在根表中直接或间接引用的托管内存

A引用B,B引用C,C引用A，则GC会销毁所有对象。

实例化一个类，只要有代码引用它就是强引用

```C#
var myclassVariable = new MyClass();

var myCache=new MyCache();
myCache.Add(myclassVariable);

myclassVariable=null//3
```

当第3步运行时就不能释放myclassVariable所引用的内存，因为缓存中还存在引用。在弱引用中就可以避免这种现象。

弱引用可以创建和使用对象，但是如果垃圾回收器在运行就可以回收

弱引用是由WeakReference创建的。需要使用IsAlive属性来确认是否被回收，Target属性可以返回一个强引用，不为null就可以访问

```C#
var myWeakReference = new WeakRefernece(new DataObject());

if(myWeakReference.IsActive)
{
DataObject strongReference=myWeakReference.Target as DataTarget;
if(strongReference!=null)
{}
}
else
{}
```


垃圾回收器不释放非托管资源

比如：文件句柄，网络连接，数据库连接，等

定义类时，可以有两种机制来自动释放非托管资源

&#43; 声明一个析构函数（或则终结器），作为类成员

```C#
class MyClass
{
~MyClass()
{
}
}
```

编译器在编译析构函数时，会隐式把析构函数的代码编译为等价于重写Finalize()方法的代码，从而确保父类的Finalize（）会被执行

等价

```C#
protected override void Finalize()
{
try
{}
finally
{
base.Finally();
}
}
```

C#析构函数具有不确定性，无法确定析构函数会被何时执行，同时析构函数会延迟对象从内存中删除的时间，有析构函数两次才能删除对象，第一次调用析构函数，第二次调用才真正删除，同时会被另起一个线程来调用Finally()

&#43; 实现System.IDisposable 接口
  该接口替代析构函数

```C#
class MyClass:IDisposable
{
public void Dispose()
{
}
}
```

Dispose()方法的实现显示释放由对象直接使用的所有非托管资源。Dispose为何时释放非托管资源提供了精确的控制。


using语句会自动调用Dispose()方法


上面的两种方法都实现了释放非托管资源

下面是一个双重实现的代码：


```C#
using System;
public class ResourceHolder:IDisposable
{
private bool _isDisposed=false;
public void Dispose()
{
DisPose(true);
GC.SuppressFinalize(this);
}
protected virtual void Dispose(bool disposing)
{
if(!_isDisposed)
{
if(disposing)
{
//cleanup managed objects by calling their dispose() methods 
}
//clean up unmanaged objects
}
_isDisposed = true;
}

~ResourceHolder()
{
Dispose(false);
}
public void SomeMethod()
{
//enture object not already disposed before execution of any method 
if(_isDisposed)
{
throw new ObjectDisposedException(&#34;ResourceHolder&#34;);
}
//method implementation
}
}
```

### 指针

C#也是可以使用指针的，unsafe{}，在C#高级编程5.5不安全代码。留坑

### 运算符重载

### 事件

&#43; 对象可以激活和使用事件，作为他们处理的一部分，事件非常重要，可以在代码的其他部分起作用，类似于异常，但是功能更强大
  &#43; 可以在Animal对象添加到Animal集合中，执行特性的代码，而这部分代码不是Animal类的一部分，也不是调用Add()方法的代码的一部分，为此，需要给代码添加事件处理程序，这是一种特殊类型的函数，在事件发生时调用，还需要配置这个处理程序，以监听自己感兴趣的事情	

```C#
private void Button1_Click_1(object sender, RoutedEventArgs e)
{
    ((Button)sender).Content = &#34;clicked&#34;;
    Button newButton = new Button();
    newButton.Content = &#34;NEW Button&#34;;
    newButton.Margin = new Thickness(10, 10, 200, 200);
    newButton.Click &#43;= newButton_click;
    ((Grid)((Button)sender).Parent).Children.Add(newButton);
}
private void newButton_click(object sender, RoutedEventArgs e)
{
    ((Button)sender).Content = &#34;clicked&#34;;
}
```


### 引用类型还是值类型

&#43; 在C#中，数据根据变量的类型以两种方法中的一种存储在一个变量中：
  &#43; 值类型的内存的同一个地方存储它们和它们的内容，栈
  &#43; 引用类型存储指向内存中其他某个位置的引用，实际内容存储在这个位置，堆

### 定义类

&#43; 类声明为内部，只有当前项目的代码能够访问 internal(默认)
  其他项目代码也能访问public
&#43; 类是抽象的，不能实例化，只能继承abstract，密封的sealed不能继承,如果基类是抽象的，在派生类必须全部实现
  &#43; 无或internal  只能在当前项目中访问
  &#43; public可以在任何地方访问类
  &#43; abstract或internal abstract类只能在当前项目访问，不能实例化，只能被继承
  &#43; public abstracrt类可以在任何地方访问，不能实例化，只能继承
  &#43; sealed或internal sealed类只能在当前项目访问，不能被继承，只能实例化
  &#43; public sealed类可以在任何地方访问，不能被继承只能实例化	

```C#
public sealed class MyClass
{
	//Class members
}
```

&#43; 派生类的可访问性不能高于基类
&#43; 类还可以指定接口，同时必须实现该接口的所有成员，如果不想使用给定的接口成员，就可以提供一个空的实现方法，继承类（只能有一个）再继承接口，接口可以有多个
&#43; 接口的定义,不能在接口中使用abstract或sealed，可以使用继承的方式继承接口

```C#
interface IMyInterface
{
....
}
```

&#43; 所有类都隐式继承System.Object类
  &#43; 多态性便可以使用，比较重要的基类方法GetType()ToStrign()

```C#
if(myObject.GetType()==typeof(MyComplexClass))
{
}
```

&#43; 初始化列表
  &#43; ：base(i)
  &#43; :this(5,6)调用自己的两个参数的构造函数，可以伪装成默认构造函数

### 定义类成员

&#43; public,private,internal,protected
&#43; readonly表示字段只能在执行构造函数的过程中赋值，static 静态变量，const也是静态的
&#43; 方法
  &#43; virtual方法可以重写
  &#43; abstract方法必须在非抽象的派生类中重写（只用于抽象类中）
  &#43; override方法重写了一个基类方法（如果方法被重写，就必须要使用关键字）,如果使用了override,也可以使用sealed来指定在派生类中不能对这个方法做进一步修改即这个方法不能由派生类重写
  &#43; extern方法定义在其他地方
&#43; 多态测试，使用virtual和override多态测试成功
&#43; 属性
  &#43; 定义与字段类似，get和set关键字来定义，可以控制属性的访问级别
  &#43; 右击refactor(重构)-Encapsulate Field可以快熟添加属性
  &#43; 自动属性：

```C#
public int MyIntProp 
{
	get;set;
}
```

	&#43; 按通常的方式定义属性的名称、类型和可访问性，但是没有提供get和set的实现代码，实现代码由编译器提供（私有字段的名字也由编译器提供，将set变成private set即变成只读）

```C#
private int myInt;
public int myIntProp
{
	get
	{
		return myInt;
	}
	public set
	{
		if(value&gt;0&amp;&amp;value&lt;10)
			myInt=value;
		else
			throw(new ArgumentOutOfRangeException(&#34;MyIntProp&#34;,value,&#34;MyIntProp must be assigned a value between 0 and 10&#34;));
	}
}
```

&#43; 隐藏
  &#43; 隐藏基类方法，会产生2个警告（如果要确定隐藏需要加上new）,无论基类怎样限制，只要派生类没有调用override都是影藏
  &#43; 要对派生类的用户隐藏继承的公共成员，但仍能访问其功能，要给继承的虚拟成员添加实现代码，而不是简单地用重写的新实现代码替换它，可以使用base关键字：base.DoSomething()
  &#43; this最常用的功能是将当前对象实例的引用传递给一个方法，该方法有一个参数是指向基类的，this关键字的另一个常用方法是限定局部类型成员

```C#
class Animal
{
    public void EatFood()
    {
        Console.WriteLine(&#34;Animal is adjusted&#34;);
    }
}
class Chicken : Animal
{
    public void EatFood()//new public void EatFood()
    {
        Console.WriteLine(&#34;Chicken is adjusted&#34;);
    }
}
class Cow : Animal
{
    public void EatFood()//new  public void EatFood()
    {
        Console.WriteLine(&#34;Cow is adjusted&#34;);
    }
}
```

&#43; 套嵌类型成员

```C#
public class MyClass
{
	public class MyNestedClass
		{
			public int NestedClassField;
		}
}//实例化时：
MyClass.MyNestedClass myObj=new MyClass.MyNestedClass();
```

```C#
public class ClassA
    {
        private int state = -1;
        public int State
        {
            get
            {
                return state;
            }
        }
        public class ClassB
        {
            public void SetPrivateState(ClassA target, int newState)
            {
                target.state = newState;
            }
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            ClassA myObject = new ClassA();
            Console.WriteLine(&#34;myObject.State = {0}&#34;, myObject.State);
            ClassA.ClassB myOtherObject = new ClassA.ClassB();
            myOtherObject.SetPrivateState(myObject, 999);
            Console.WriteLine(&#34;myObject.State = {0}&#34;, myObject.State);
            Console.ReadKey();
        }
    }//嵌套类修改只读变量
```

&#43; 接口
  &#43; 接口不会有访问修饰符
  &#43; 接口成员没有实现
  &#43; 接口没有字段成员(但是接口可以定义自动属性)
  &#43; 不能有关键字static/virtual/abstract/sealed来定义接口
  &#43; 但接口是可以继承的，如果要隐藏基类成员，需要加关键字new

```C#
public interface IMyInterface
{
void DoSomething();
void DoSomethingElse();
}
public class MyClass:IMyInterface
{
public void DoSomething()
{
}
public void DoSomethingElse()
{
}
}//接口类必须包含接口的所有成员包括匹配的指定签名包括get和set
public interface IMyInterface
{
	void DoSomething();
	void DoSomethingElse();
	}
	public class MyBaseClass
	{
	public void DoSomething()
	{
	}
	}
	public class MyDerivedClass:MyBaseClass,IMyInterface
	{
	public void DoSomethingElse()
	{
	}
}
public interface IMyInterface
{
	void DoSomething();
	void DoSomethingElse();
	}
	public class MyBaseClass:IMyInterface
	{
	public virtual void DoSomething()
	{
	}
	public virtual void DoSomethingElse()
	{
	}
	}
	public class MyDerivedClass:MyBaseClass
	{
	public override void DoSomethingElse()
	{
	}
	public override void DoSomethingElse()
	{
	}
}
```

	&#43; 实现接口

```C#
MyClass myObj=new MyClass();
IMyInterface myInt=myObj;
myInt.DoSomething();
public class MyClass :IMyInterface
{
void IMyInterface.DoSomething()//显示调用
{
}
public void DoSomething()//隐式调用
{
}
}
public interface IMyInterface
{
int MyIntProperty
{
get;
}
}
public class MyBaseClass:IMyInterface
{
public int MyIntProperty{get;protected set;}
}
```

	&#43; 把接口放在不同的文件中partial

```C#
//定义类时使用
public partial class MyClass
{
}
//部分方法定义
public partial class MyClass
{
partial void DoSomethingElse();
public void DoSomething()
{
console.writeline(&#34;do something started&#34;);
DoSomethingElse();
console.writeline(&#34;do something ended&#34;);
}
}
public partial class MyClass
{
partial void DoSomethingElse()
{
console.writeline(&#34;DoSomethingElse called&#34;);
}
}
//如果删除部分方法的实现部分，编译时编译器会以为调用了一个空的部分方法会直接删除部分方法的调用
```

&#43; 集合
  &#43; 可以使用集合来维护对象组
    &#43; 集合大多是通过System.Collections名称空间中的接口而获得的，集合的语法已经标准化了
    &#43; System.Collections包括几个接口：
    &#43; IEnumerable可以迭代集合中的项foreach
    &#43; ICollection继承上一个接口，可以获取集合中项的个数，并能把项复制到一个简单的数组类型中count()copyto(),add(),remove(),clear()
    &#43; IList继承上两个接口，提供了集合的项列表，允许访问这些项，并提供一些基本功能insert(),removeat(),派生于`Icollection&lt;T&gt;`接口
    &#43; ISet获取两个集合的交集
    &#43; IDictionary继承与前两个接口，可以通过键值访问项列表
    &#43; ILookup接口，键和值
    &#43; Icomparer实现compare（）
    &#43; IEqualityCompare(）接口。
    &#43; System.Array类实现了IList,ICollection,IEnumerable接口
    &#43; System.Collections名称空间中的类System.Collections,ArrayList也实现了这些接口并且更为复杂
  &#43; 定义集合通过Collections
    &#43; 直接定义比较复杂可以派生System.Collection.CollectionBase，同时该类提供了两个受保护的属性，List和InnerList

```C#
using System.Collections;
public class Cards:CillectionBase
{
}
```

		&#43; 索引符是一种特殊类型的属性

```C#
public Animal this[int animalIndex]
{
get
{
return (Animal)List[animalIndex];
}
set
{
List[animalIndex]=value;
}
}
```

		&#43; 基类为DictionaryBase,提供了一个Dictionary属性

```C#
public void Add(string newID,Animal newAnimal)
{
Directionary.Add(newID,newAnimal);
}
public void Remove(string animalID)
{
Directionary.Remove(animalID);
}
//索引符是一种特殊类型的属性
public Animal this[string animalID]
{
get
{
return (Animal)Directionary[animalID];
}
set
{
Directionary[animalID]=value;
}
}
```

	&#43; 迭代器
		&#43; 在foreach循环中，迭代一个collectionObject集合
		&#43; foreach
			&#43; 调用collectionObject.GetEnumerator(),返回一个IEnumerator引用
			&#43; 调用所返回的IEnumerator接口的MoveNext()方法
			&#43; 如果MoveNext()返回true.就使用IEnumerator接口的Current属性获取对象的引用，用于foreach循环
			&#43; 重复前两步，知道返回false
		&#43; 迭代器
			&#43; 两种可能返回类型是前面提到的接口类型IEnumerable和IEnumerator
			&#43; 如果要迭代一个类，可使用方法GetEnumerator()其返回类型IEnumerator;如果要迭代一个类成员，例如一个方法，则使用Enumerable，
			&#43; `yield return&lt;value&gt;;`
	&#43; 深度复制
		&#43; 实现ICloneable接口，该接口有一个Clone()方法，

```C#
public class Content
{
public int val;
}
public class Cloner :IConeable
{
public Content MyContent=new Content();
public Cloner(int newVal)
{
MyContent .val=newVal;
}
public object Clone()
{
Cloner clonedCloner =new Cloner (MyContent.Val);
return cloneCloner;
}
}
```

&#43; 比较，比较对象 
  &#43; if(myObj.GetType()==typeof(MyComplexClass))
  &#43; 封箱和拆箱
    &#43; 封箱就是把值类型转换为System.Object类型，
    &#43; 拆箱就是把System.Object类型转换为值类型

```C#
struct MyStruct
{
public int Val;
}
MyStruct valType=new MyStruct();
valType.Val=5;
object.refType=valType;
//这样称为封箱，这种封装只是封装的原值的副本
class MyStruct
{
public int Val;
}
MyStruct valType=new MyStruct();
valType.Val=5;
object.refType=valType;
//这样称为封箱，这种封装封装原值
interface IMyInterface
{}
struct MyStruct:IMyIterface
{
public int Val;
}
MyStruct valType=new MyStruct();
valType.Val=5;
IMyInterface refType=valType;
MyStruct ValType2=(MyStruct)refType;//解箱
```

		&#43; 封箱允许在项的类型是object的集合中使用值类型，在一个内部机制允许在值类型上调用object方法
		&#43; is运算符不是用来说明对象是某种类型，而是用来检查对象是不是给定类型，或者是否可以转换成给定类型
			&#43; `&lt;operand&gt;is &lt;type&gt;`
			&#43; 如果type是一个类类型，operand也该是该类类型
			&#43; 如果type是一个接口类型，operand也该是该类型
			&#43; 如果type是一个值类型，那么operand也是该类型
	&#43; 值比较
		&#43; 运算符重载

```C#
public static AddClass1 operator &#43;(AddClass1 op1,AddClass1 op2)
{
AddClass1 returnValue=new AddClass1();
returnVal.val=op1.val&#43;op2.val;
return returnVal;
}
```

			&#43; 可以重载的运算符&#43;，-，！，~，&#43;&#43;，--，true，false
			&#43; &#43;,-,*,%,/,&amp;,|,^,&lt;&lt;,&gt;&gt;
			&#43; ==,!=,&lt;,&gt;,&lt;=,&gt;=
		&#43; 使用IComparable和IConparer
			&#43; IComparable比较对象中类的实现，可以比较一个对象和另一个对象
			&#43; IComparer在一个单独的类中实现，可以比较任意两对象
			&#43; IComparable中有一个CompareTo()方法，这个方法接受一个对象`if(person1.CompareTo(person2)==0)`
			&#43; IComparer提供了一个Compare(),这个方法接受两个对象，该方法接受两个对象`personCompare(person1,person2);`

```C#
string firststring=&#34;first string&#34;;
string secondstring = &#34;second string&#34;;
Console.writeline(&#34;{0}{1}{2}&#34;,firststring,secondstring,Comparer.Default.Compare(firststring ,secondstring));
```

			&#43; Comparer.Default静态成员获取Comparer类的实例，然后使用Compare比较
			&#43; 注意事项：在使用conparer时，必须是可以比较的类型，如果试图比较string和int类型就会发生异常，检查传送给Comparer.Compare()的对象，看他们是否支持IComparable，支持实现该代码,允许使用NULL值，它表示小于其他任意对象
			&#43; 这两种方法需要比较某种类型，不然会抛出异常
	&#43; 对集合排序
		&#43; System.Collections名称空间中的类都没有提供排序方法，Arraylist类提供了sort()排序算法，对于自己的类使用IComparer进行比较排序

```C#
public class PersonComparerName : IComparer
{
  public static IComparer Default = new PersonComparerName();
  public int Compare(object x, object y)
  {
     if (x is Person &amp;&amp; y is Person)
     {
        return Comparer.Default.Compare(
           ((Person)x).Name, ((Person)y).Name);
     }
     else
     {
        throw new ArgumentException(
           &#34;One or both objects to compare are not Person objects.&#34;);
     }
  }
}
using System.Collections;   
class Program
{
  static void Main(string[] args)
  {
     ArrayList list = new ArrayList();
     list.Add(new Person(&#34;Jim&#34;, 30));
     list.Add(new Person(&#34;Bob&#34;, 25));
     list.Add(new Person(&#34;Bert&#34;, 27));
     list.Add(new Person(&#34;Ernie&#34;, 22));
     list.Sort();
     list.Sort(PersonComparerName.Default);
  }
}
```

&#43; 转换：定义类型转换
  &#43; 如果既没有继承，又没有公有接口的类如果转换

```C#
ConClass1 op1=new ConClass1();
ConClass2 op2=op1;//隐式转换
ConClass1 op1=new ConClass1();
ConClass2 op2=(ConClass2 )op1;//显示转换
public class ConvClass1
{
public int val;
public static implicit operator ConvClass2(ConvClass1 op1)//允许隐式转换
{
ConvClass2 returnVal=new ConvClass2();
returnVal.val=op1.val;
return returnVal;
}
}
public class ConvClass2
{
public double val;
public static explicit operator ConvClass1(ConvClass2 op1)//只能显式转换
{
ConvClass1 returnVal=new ConvClass1();
checked(returnVal.val=(int)op1.val);
return returnVal;
}
}
```

	&#43; as运算符`&lt;operand&gt;as&lt;type&gt;`,operand类型是type类型,operand可隐式转换为type类型,operand可以封箱到type类型,如果operand不能转化为type则表达式结果为null

```C#
class ClassA:IMyInterface
{}
class ClassB:ClassA
{}
ClassA ob1=new ClassA ();
ClassD ob2=ob1;//抛出异常
ClassD ob2=ob1 as ClassB;//不会抛出异常但是ob2的值为null;
```

### 泛型

泛型是以实例化的过程中提供的类型或类为基础建立的，可以毫不费力的对对象进行强类型化,使用泛型强化为各种类型。与C&#43;&#43;不同，C&#43;&#43;是在编译期间来强类型化，而C#是在运行期间强类型化

```C#
CollectionClass&lt;ItemClass&gt; items= new CollectionClass&lt;ItemClass&gt;();
items.Add(new ItemClass);
```

&#43; .net 提供的泛型，包括System.Collections.Generic名称空间
&#43; 可空类型，引用类型可以为空，值类型必须有一个值，扩展值类型让其可为空，泛型使用了System.Nullable&lt;T&gt;类型提供了值类型为空的一种方法
  &#43; `System.Nullable&lt;int&gt; nullableInt;`变量nullableInt可以包含int类型的任意值，还可以拥有null
  &#43; `nullableInt=null;等价于nullableInt=new System.Nullable&lt;int&gt;();`
  &#43; nullableInt.HasValue,该方法不适用于引用类型，引用类型本身可能为NULL
  &#43; `int？ nullableInt；int？是System.Nullable&lt;int&gt; 的缩写`
  &#43; 可控类型null&#43;任何数都为null,null在布尔运算中的大小介于FALSE和TRUE之间
&#43; ？？运算符为空结合运算符，是一个二元运算符，允许给可能等于NULL的表达式提供另一个值，op1??op2等价于op1==null?op2:op1;
&#43; `List&lt;T&gt; myCollection=new List&lt;T&gt;();`
  &#43; Item	获取或设置指定索引处的元素。
  &#43; Count	获取 List&lt;T&gt; 中实际包含的元素数。
  &#43; Add	将对象添加到 List&lt;T&gt; 的结尾处。
  &#43; AddRange	将指定集合的元素添加到 List&lt;T&gt; 的末尾。
  &#43; Clear	从 List&lt;T&gt; 中移除所有元素。
  &#43; Contains	确定某元素是否在 List&lt;T&gt; 中。
  &#43; ConvertAll&lt;TOutput&gt;	将当前 List&lt;T&gt; 中的元素转换为另一种类型，并返回包含转换后的元素的列表。
  &#43; Equals(Object)	确定指定的 Object 是否等于当前的 Object。 （继承自 Object。）
  &#43; Remove	从 List&lt;T&gt; 中移除特定对象的第一个匹配项。
  &#43; Sort()	使用默认比较器对整个 List&lt;T&gt; 中的元素进行排序。
  &#43; ToArray	将 List&lt;T&gt; 的元素复制到新数组中。
  &#43; ToString	返回表示当前对象的字符串。 （继承自 Object。）

```C#
using System.Collections.Generic;
List&lt;Animal&gt; animalCollection = new List&lt;Animal&gt;();
animalCollection.Add(new Cow(&#34;Jack&#34;));
animalCollection.Add(new Chicken(&#34;Vera&#34;));
foreach (Animal myAnimal in animalCollection)
{
myAnimal.Feed();
}
或者
public class Animals:List&lt;Animal&gt;
{}
```

&#43; 对泛型进行排序
  &#43; System.Collections.Generic名称空间包含`List&lt;T&gt;`T类型对象集合,`Dictionary&lt;K,V&gt;`与K类型的键值相关的V类型的项的集合
  &#43; `List&lt;T&gt;`
    &#43; 使用泛型接口`IComparer&lt;T&gt;和IComparable&lt;T&gt;`
      &#43; `int IComparable&lt;T&gt;.CompareTo(T otherObj)`
      &#43; `bool IComparable&lt;T&gt;.Equals(T otherObj)`
      &#43; `int IComparer&lt;T&gt;.Compare(T objectA,T objectB)`
      &#43; `bool ICompare&lt;T&gt;.Equals(T objectA,T objectB)`
      &#43; `int IComparer&lt;T&gt;.GetHashCode(T objectA)`
    &#43; 给列表排序，需要有一个方法来比较两个T类型的对象，要在列表中搜索，需要用一个方法来检查T类型的对象
    &#43; 两个泛型委托：

```C#
Comparison&lt;T&gt;
	int method(T objectA,TobjectB)
Predicate&lt;T&gt;
	bool method(T targetObject)
```

```C#
public class Vectors : List&lt;Vector&gt;
{
  public Vectors()
  {
  }
  public Vectors(IEnumerable&lt;Vector&gt; initialItems)
  {
     foreach (Vector vector in initialItems)
     {
        Add(vector);
     }
  }
  public string Sum()
  {
     StringBuilder sb = new StringBuilder();//表示可变字符字符串
     Vector currentPoint = new Vector(0.0, 0.0);
     sb.Append(&#34;origin&#34;);
     foreach (Vector vector in this)
     {
        sb.AppendFormat(&#34; &#43; {0}&#34;, vector);
        currentPoint &#43;= vector;
     }
     sb.AppendFormat(&#34; = {0}&#34;, currentPoint);
     return sb.ToString();
  }
}
public static class VectorDelegates
{
  public static int Compare(Vector x, Vector y)
  {
     if (x.R &gt; y.R)
     {
        return 1;
     }
     else if (x.R &lt; y.R)
     {
        return -1;
     }
     return 0;
  }
  public static bool TopRightQuadrant(Vector target)
  {
     if (target.Theta &gt;= 0.0 &amp;&amp; target.Theta &lt;= 90.0)
     {
        return true;
     }
     else
     {
        return false;
     }
  }
}
class Program
{
  static void Main(string[] args)
  {
     Vectors route = new Vectors();
     route.Add(new Vector(2.0, 90.0));
     route.Add(new Vector(1.0, 180.0));
     route.Add(new Vector(0.5, 45.0));
     route.Add(new Vector(2.5, 315.0));
     Console.WriteLine(route.Sum());
     Comparison&lt;Vector&gt; sorter = new Comparison&lt;Vector&gt;(VectorDelegates.Compare);//委托
     route.Sort(sorter);//这个个语句可以简化route.Sort(VectorDelegates.Compare);
     Console.WriteLine(route.Sum());
     Predicate&lt;Vector&gt; searcher = new Predicate&lt;Vector&gt;(VectorDelegates.TopRightQuadrant);//委托
     Vectors topRightQuadrantRoute = new Vectors(route.FindAll(searcher));
     Console.WriteLine(topRightQuadrantRoute.Sum());
     Console.ReadKey();
  }
}
```

	&#43; `Dictionary&lt;K,V&gt;`该类型建立键/值对应这个类需要实例化两个类型，分别用于键和值

```C#
Dictionary&lt;string ,int&gt;things=new Directionary&lt;string ,int &gt;()
things.Add(&#34;Green Things&#34;,29);
foreach(strign key in things.Values)
{
using key;
}
foreach(int value in things.Values)
{
usign value
}
foreach(KeyValuePair&lt;string,int &gt; thing in things)
{
using things.key and things.value
}
```

		&#43; 每个键都独立的，如果相同抛出异常`Dictionary&lt;string ,int&gt;things=new Directionary&lt;string ,int &gt;(stringComparer.CurrentCultureIgnoreCase)`自己的类用作键，不区分发小写比较，这样同一个类就会出现异常

&#43; 定义泛型
  &#43; 要创建泛型类

```C#
class MyGenericClass&lt;T1,T2,T3&gt;
{
}
```

	&#43; 定义泛型后就可以像使用其他数据类型一样使用它们
	&#43; default关键字,default(T1),如果T1是值类型使之默认为0，如果为引用就默认为null
	&#43; 约束

```C#
class MyGenericClass&lt;T1,T2,T3&gt; where T:constraint1,constraint2
{
}
//约束必须在继承后面
```

		&#43; struct类型必须为值类型
		&#43; class类型必须为引用类型
		&#43; baseclass类型必须为基类或继承基类
		&#43; interface类型必须有接口或实现了接口
		&#43; new()类型必须有一个公共的无参构造函数

```C#
public abstract class Animal
{
  protected string name;
  public string Name
  {
	 get
	 {
		return name;
	 }
	 set
	 {
		name = value;
	 }
  }
  public Animal()
  {
	 name = &#34;The animal with no name&#34;;
  }
  public Animal(string newName)
  {
	 name = newName;
  }
  public void Feed()
  {
	 Console.WriteLine(&#34;{0} has been fed.&#34;, name);
  }
  public abstract void MakeANoise();
}
public class Chicken : Animal
{
  public void LayEgg()
  {
	 Console.WriteLine(&#34;{0} has laid an egg.&#34;, name);
  }
  public Chicken(string newName) : base(newName)
  {
  }
  public override void MakeANoise()
  {
	 Console.WriteLine(&#34;{0} says &#39;cluck!&#39;;&#34;, name);
  }
}
public class Chicken : Animal
{
  public void LayEgg()
  {
	 Console.WriteLine(&#34;{0} has laid an egg.&#34;, name);
  }
  public Chicken(string newName) : base(newName)
  {
  }
  public override void MakeANoise()
  {
	 Console.WriteLine(&#34;{0} says &#39;cluck!&#39;;&#34;, name);
  }
}
public class Cow : Animal
{
  public void Milk()
  {
	 Console.WriteLine(&#34;{0} has been milked.&#34;, name);
  }
  public Cow(string newName) : base(newName)
  {
  }
  public override void MakeANoise()
  {
	 Console.WriteLine(&#34;{0} says &#39;moo!&#39;&#34;, name);
  }
}
public class SuperCow : Cow
{
  public void Fly()
  {
	 Console.WriteLine(&#34;{0} is flying!&#34;, name);
  }
  public SuperCow(string newName): base(newName)
  {
  }
  public override void MakeANoise()
  {
	 Console.WriteLine(
		&#34;{0} says &#39;here I come to save the day!&#39;&#34;, name);
  }
}
public class Farm&lt;T&gt; : IEnumerable&lt;T&gt;
  where T : Animal
{
  private List&lt;T&gt; animals = new List&lt;T&gt;();
  public List&lt;T&gt; Animals
  {
	 get
	 {
		return animals;
	 }
  }
  public IEnumerator&lt;T&gt; GetEnumerator()//实现IEnumerator&lt;T&gt; 接口
  {
	 return animals.GetEnumerator();//才能实现foreach
  }
  IEnumerator IEnumerable.GetEnumerator()//IEnumerator&lt;T&gt; 接口继承IEnumerable，必须实现IEnumerable接口
  {
	 return animals.GetEnumerator();
  }
  public void MakeNoises()
  {
	 foreach (T animal in animals)
	 {
		animal.MakeANoise();
	 }
  }
  public void FeedTheAnimals()
  {
	 foreach (T animal in animals)
	 {
		animal.Feed();
	 }
  }
  public Farm&lt;Cow&gt; GetCows()
  {
	 Farm&lt;Cow&gt; cowFarm = new Farm&lt;Cow&gt;();
	 foreach (T animal in animals)
	 {
		if (animal is Cow)
		{
		   cowFarm.Animals.Add(animal as Cow);
		}
	 }
	 return cowFarm;
  }
  // For generic methods section.
  //public Farm&lt;U&gt; GetSpecies&lt;U&gt;() where U : T
  //{
  //   Farm&lt;U&gt; speciesFarm = new Farm&lt;U&gt;();
  //   foreach (T animal in animals)
  //   {
  //      if (animal is U)
  //      {
  //         speciesFarm.Animals.Add(animal as U);
  //      }
  //   }
  //   return speciesFarm;
  //}
}
class Program
{
  static void Main(string[] args)
  {
	 Farm&lt;Animal&gt; farm = new Farm&lt;Animal&gt;();
	 farm.Animals.Add(new Cow(&#34;Jack&#34;));
	 farm.Animals.Add(new Chicken(&#34;Vera&#34;));
	 farm.Animals.Add(new Chicken(&#34;Sally&#34;));
	 farm.Animals.Add(new SuperCow(&#34;Kevin&#34;));
	 farm.MakeNoises();
	 Farm&lt;Cow&gt; dairyFarm = farm.GetCows();
	 dairyFarm.FeedTheAnimals();
	 foreach (Cow cow in dairyFarm)
	 {
		if (cow is SuperCow)
		{
		   (cow as SuperCow).Fly();
		}
	 }
	 Console.ReadKey();
  }
}
Jack says &#39;moo!&#39;
Vera says &#39;cluck!&#39;;
Sally says &#39;cluck!&#39;;
Kevin says &#39;here I come to save the day!&#39;
Jack has been fed.
Kevin has been fed.
Kevin is flying!
```

&#43; 泛型运算符

```C#
public static implicit operator List&lt;Animal&gt;(Farm&lt;T&gt; farm)
{
	List&lt;Animal&gt;result=new List&lt;Animal&gt;();
	foreach(T animal in farm)
	{
		result.Add(animal);
	}
	return result;
} 
pulic static Farm&lt;T&gt; operator &#43;(Farm&lt;T&gt; farm1,List&lt;T&gt; farm2)
{
	Farm&lt;T&gt; result= new Farm&lt;T&gt; ();
	foreach(T animal in farm1)
	{
		result.Animals.add(animal);
	}
	foreach(T animal in farm2)
	{
		if(!result.Animals.Contains(animal))
		{
			result.Animals.add(animal);
		}
	}
	return result;
}
public static Farm&lt;T&gt; operator &#43;(List&lt;T&gt; farm1,Farm&lt;T&gt; farm2)
{
	return farm1&#43;farm2;
}
Farm&lt;T&gt; newFarm=farm &#43; dairyFarm;
```

&#43; 泛型结构
  结构是值类型，不是引用类型
  可以用泛型来构造泛型结构

```C#
public struct Mystructs&lt;T1,T2&gt;
{
	public T1 item1;
	public T2 item2;
}
```

&#43; 泛型接口

```C#
interface MyFarmingInterface where T:Animal
{
	bool AttempToBreed(T animal1,T animal2)
	T oldestInHerd{get;}
}
```

&#43; 泛型方法

```C#
public T GetDefault&lt;T&gt;()
{
	return default&lt;T&gt;;//使用关键字default，为类型T返回默认值
}
int myDefaultInt=GetDefault&lt;int&gt;();//调用
public class Defaulter
{
	public T GetDefault&lt;T&gt;()
	{
		return default&lt;T&gt;;//通过非泛型类实现泛型
	}
}
public class Defaulter&lt;T1&gt;
{
	public T2 GetDefault&lt;T2&gt;()where T2:T1
	{
		return default&lt;T2&gt;;//通过泛型类与泛型方法不同的标识
	}
}
public farm&lt;u&gt; getspecies&lt;u&gt;() where u : t
{
	farm&lt;u&gt; speciesfarm = new farm&lt;u&gt;();
	foreach (t animal in animals)
	{
	   if (animal is u)
	   {
		  speciesfarm.animals.add(animal as u);
	   }
	}
	return speciesfarm;
}
```

&#43; 泛型委托

```C#
public delegate T1 MyDelegate&lt;T1,T2&gt;(T2 op1,T2 op2) where T2:T1;
```

&#43; 变体：是抗变和协变的统称
  需要定义编译过程

```C#
Cow myCow=new Cow(&#34;gemeroter&#34;);
Animal myAnimal=myCow;//在类中这样是可行的
IMethaneProduce&lt;Cow&gt; cowMethaneProduce=myCow;
IMethaneProduce&lt;Animal&gt; animalMethaneProduce=cowMethaneProduce;//在接口中这样是不可以的
//在泛型中所有类型都是不变的，但是可以在泛型接口和泛型委托的基础上定义变体类型参数
//IMethaneProduce&lt;T&gt;接口类型参数必须是协变的，
//抗变与协变类似但是方向相反，抗变可以把接口放在派生类型中
IGrassMuncher&lt;Cow&gt; cowGrassMuncher=myCow;
IGrassMuncher&lt;SuperCow&gt; supercowGrassMuncher=cowGrassMuncher;
```

	&#43; 协便

```C#
//协变需要使用关键字out
public interface IMethaneProduce&lt;out T&gt;//这样IEnumerable&lt;Cow&gt;对象可以放在IEnumerable&lt;Animal&gt;类型的变量中
{
	...
}
static void Main(string[] args)
{
	List&lt;Cow&gt; cows=new List&lt;Cow&gt;();
	cows.Add(new Cow(&#34;dfdf&#34;));
	cows.Add(new SuperCow(&#34;ddddd&#34;));
	ListAnimals(cows);
}
static void ListAnimals(IEnumerable&lt;Animal&gt; animals)
{
	foreach(Animal animal in animals)
	{
		Console.WriteLine(animal.Tostring());
	}
}
```

	&#43; 抗变

```C#
//抗变需要使用关键字in
public interface IGrassMuncher&lt;in T&gt;//抗变类型只能用作方法参数
{
	...
}
public class AnimalNameLengthComparer:IComparer&lt;Animal&gt;
{
	public int Compare(Animal x,Animal y)
	{
		return x.Name.Length.CompareTo(y.Name.Length)
	}
}
```

### 事件

&#43; 事件必须订阅，可以由多个订阅

```C#
using System.Timers;
class Program
   {
      static int counter = 0;

      static string displayString = &#34;This string will appear one letter at a time. &#34;;

      static void Main(string[] args)
      {
         Timer myTimer = new Timer(100);//创建定时器，毫秒
         myTimer.Elapsed &#43;= new ElapsedEventHandler(WriteChar);//Elapsed事件该事件必须匹配System.Timers.ElapsedEventHandler委托类型的返回值
								//处理程序和事件订阅起来，事件处理方法初始化为一个新的委托实例
								//可以直接写成myTimer.Elapsed &#43;=WriteChar；
         myTimer.Start();//启动定时器
         System.Threading.Thread.Sleep(200);//将当前线程阻塞指定的毫秒数
         Console.ReadKey();
      }
      static void WriteChar(object source, ElapsedEventArgs e)
      {
         Console.Write(displayString[counter&#43;&#43; % displayString.Length]);
      }
   }
System.Timers.ElapsedEventHandler委托类型是标准委托之一， 
void &lt;MethodName&gt;(object source,ElapsedEventArgs e)
```

&#43; 定义事件

```C#
public delegate void MessageHandler(string messageText);//定义委托，该委托用于定义的事件必须指明返回值的参数值
public class Connection
{
  public event MessageHandler MessageArrived;//给时间命名并指定委托的类型
  private Timer pollTimer;//定时器
  public Connection()
  {
     pollTimer = new Timer(100);//定时器时间
     pollTimer.Elapsed &#43;= new ElapsedEventHandler(CheckForMessage);//定时器的事件委托
  }
  public void Connect()
  {
     pollTimer.Start();//定时器开始
  }
  public void Disconnect()
  {
     pollTimer.Stop();//定时器结束
  }
  private static Random random = new Random();
  private void CheckForMessage(object source, ElapsedEventArgs e)//定时器的委托函数；
  {
     Console.WriteLine(&#34;Checking for new messages.&#34;);
     if ((random.Next(9) == 0) &amp;&amp; (MessageArrived != null))///生成一个0-9的数，如果是0，并且事件有订阅者，则事件实现委托就使用委托来
     {
        MessageArrived(&#34;Hello Mum!&#34;);
     }
  }
}
public class Display
{
  public void DisplayMessage(string message)
  {
     Console.WriteLine(&#34;Message arrived: {0}&#34;, message);
  }
}
class Program
{
  static void Main(string[] args)
  {
     Connection myConnection = new Connection();
     Display myDisplay = new Display();
     myConnection.MessageArrived &#43;=new MessageHandler(myDisplay.DisplayMessage);//将委托和函数绑定
     myConnection.Connect();
     System.Threading.Thread.Sleep(200);
     Console.ReadKey();
  }
}
```

&#43; 多用途事件处理程序
  &#43; Timer.Elapsed事件的委托包含了事件处理程序中常见的两类参数
  &#43; object source:引发事件对象的引用
  &#43; ElapsdEventArgs:由事件传送的参数
  &#43; 由不同对象引发的几个相同事件使用相同事件处理程序

```C#
public class MessageArrivedEventArgs : EventArgs//定义消息类
{
  private string message;
  public string Message
  {
     get
     {
        return message;
     }
  }
  public MessageArrivedEventArgs()
  {
     message = &#34;No message sent.&#34;;
  }
  public MessageArrivedEventArgs(string newMessage)
  {
     message = newMessage;
  }
}
public class Connection
{
  public event EventHandler&lt;MessageArrivedEventArgs&gt; MessageArrived;//事件，EventHandler&lt;T&gt;为委托模板，将消息类传入
  private Timer pollTimer;
  public string Name { get; set; }
  public Connection()
  {
     pollTimer = new Timer(100);
     pollTimer.Elapsed &#43;= new ElapsedEventHandler(CheckForMessage);
  }
  public void Connect()
  {
     pollTimer.Start();
  }
  public void Disconnect()
  {
     pollTimer.Stop();
  }
  private static Random random = new Random();
  private void CheckForMessage(object source, ElapsedEventArgs e)
  {
     Console.WriteLine(&#34;Checking for new messages.&#34;);
     if ((random.Next(9) == 0) &amp;&amp; (MessageArrived != null))
     {
        MessageArrived(this, new MessageArrivedEventArgs(&#34;Hello Mum!&#34;));//发送消息
     }
  }
}
public class Display//事件响应函数
{
  public void DisplayMessage(object source, MessageArrivedEventArgs e)
  {
     Console.WriteLine(&#34;Message arrived from: {0}&#34;,
                       ((Connection)source).Name);
     Console.WriteLine(&#34;Message Text: {0}&#34;, e.Message);
  }
}
class Program
{
  static void Main(string[] args)
  {
     Connection myConnection1 = new Connection();
     myConnection1.Name = &#34;First connection.&#34;;
     Connection myConnection2 = new Connection();
     myConnection2.Name = &#34;Second connection.&#34;;
     Display myDisplay = new Display();
     myConnection1.MessageArrived &#43;= myDisplay.DisplayMessage;
     myConnection2.MessageArrived &#43;= myDisplay.DisplayMessage;
     myConnection1.Connect();
     myConnection2.Connect();
     System.Threading.Thread.Sleep(200);
     Console.ReadKey();
  }
}
```

	&#43; .net提供了两个委托类型：EventHandler和`EventHandler&lt;T&gt;`
	&#43; 匿名方法，纯粹是为了用作委托目的而创建的

```C#
delegate(parameters)
{
}
parameters参数化列表
myConnection1.MessageArrived &#43;= delegate(Connection source,MessageArriveEventArgs e)
{Console.writeline(&#34;message arrived from{0}&#34;,source.Name);}
```

### 特性，元数据

可以为代码段标记一些信息，这些信息可以从外部读取

```C#
[DebuggerStepThrough]
public void DullMethod()
```

&#43; {}该特性说明，在调试的时候不进入该方法进行逐句调试，而是跳过该方法,该特性通过DebuggerStepThroughAttribute这个类来实现的，这个类位于System.Diagnostics名称空间中
&#43; 特性的参数可以自己设置[DoesInterestingThings(1000, WhatDoesItDo = &#34;voodoo&#34;)]
&#43; 读取特性需要用到反射,Type.GetCustomAttributes来实现

## 补充

&#43; 对象初始化器
  &#43; 省略构造函数的括号，自动调用无参数构造函数，之后调用初始化器
  &#43; 同时对象初始化器还可以进行嵌套

```C#
//初始化器,使用默认构造函数实现赋值
&lt;classname&gt; &lt;variableName&gt;=new &lt;claseename&gt;
{
	&lt;propertyOrField1&gt;=&lt;value1&gt;,
	&lt;propertyOrField2&gt;=&lt;value2&gt;,
	&lt;propertyOrField3&gt;=&lt;value3&gt;,
	...
}
Curry tastyCurry=new Curry
{
	MainInte=&#34;ddddd&#34;;
	Origin=new restaurant
	{
		Name=&#34;ddd&#34;;
	}
};
```

&#43; 集合初始化器
&#43; 类型推理：var ,编译器来确定类型，var关键字还可以通过数组初始化器来推断数组的类型：`var myarray=new [] {4,5,6,7};`4,5,6,7必须遵循：相同的类型，相同的引用类型或空，所有元素的类型都可以隐式转换为同一类型

&#43; 匿名对象：为了减少创建对象所消耗的时间，其理念就是让编辑器根据要存储的数据自动创建类型，而不是定义简单的数据存储类型
  &#43; 匿名类型

```C#
var curry=new
{
MainIngredient=&#34;lamb&#34;,
Style=&#34;Dhansnk&#34;,
Spiciness=5
};
```

		&#43; 使用var关键字，因为匿名类型没有可以使用的标识符，
		&#43; new后面没有指定类型的名称
		&#43; 匿名对象的属性被定义为只读，如果在存储对象中修改属性的值，就不能使用匿名对象。
		&#43; 使用动态查找功能可以处理未知的C#类型

```C#
class Program
{
  static void Main(string[] args)
  {
	 var curries = new[]
	 {
		new
		{
		   MainIngredient = &#34;Lamb&#34;,
		   Style = &#34;Dhansak&#34;,
		   Spiciness = 5
		},
		new
		{
		   MainIngredient = &#34;Lamb&#34;,
		   Style = &#34;Dhansak&#34;,
		   Spiciness = 5
		},
		new
		{
		   MainIngredient = &#34;Chicken&#34;,
		   Style = &#34;Dhansak&#34;,
		   Spiciness = 5
		}
	 };
	 Console.WriteLine(curries[0].ToString());
	 Console.WriteLine(curries[0].GetHashCode());
	 Console.WriteLine(curries[1].GetHashCode());
	 Console.WriteLine(curries[2].GetHashCode());
	 Console.WriteLine(curries[0].Equals(curries[1]));//比较状态，即每个属性的值
	 Console.WriteLine(curries[0].Equals(curries[2]));
	 Console.WriteLine(curries[0] == curries[1]);
	 Console.WriteLine(curries[0] == curries[2]);
	 Console.ReadKey();
  }
}
{ MainIngredient = Lamb, Style = Dhansak, Spiciness = 5 }
294897435
294897435
621671265
True
False
False
False
```

&#43; 动态类型：dynamic myDynamicVar;在编译期间会被object替代，
&#43; 默认参数类型
&#43; 可变参数
&#43; 命名参数：首先选定必选参数，再指定命名的可选参数

```C#
public static List&lt;string&gt; GetWords(
	 string sentence,
	 bool capitalizeWords = false,
	 bool reverseOrder = false,
	 bool reverseWords = false)
{
 List&lt;string&gt; words = new List&lt;string&gt;(sentence.Split(&#39; &#39;));
 if (capitalizeWords)
	words = CapitalizeWords(words);
 if (reverseOrder)
	words = ReverseOrder(words);
 if (reverseWords)
	words = ReverseWords(words);
 return words;
}
words = WordProcessor.GetWords(
            sentence, 
            reverseWords: true,
            capitalizeWords: true);
```

&#43; 扩展方法
  &#43; 要创建和使用扩展方法必须：
    &#43; 创建一个非泛型静态类
    &#43; 使用扩展方法的语法，为所创建的类添加扩展方法，做为静态方法
    &#43; 确保使用扩展方法的代码用using语法导入包含扩展方法类的名称空间
    &#43; 通过扩展类型的一个实例调用扩展方法，与调用扩展类型的其他方法一样。
  &#43; 扩展方法的要求：
    &#43; 方法必须是静态的
    &#43; 方法必须包含一个参数，表示调用扩展方法的类型实例
    &#43; 实例参数必须是方法定义的第一个参数，
    &#43; 除了this关键字外，实例参数不能有其他修饰符

```C#
public static class ExtensionClass
{
	public static &lt;ReturnType&gt; &lt;ExtensionMethodName&gt;(this &lt;TyprToExtend&gt; instance ,&lt;otherParamters&gt;)
	{
		...
	}
}
public static class ExtensionClass
{
	public static &lt;ReturnType&gt; &lt;ExtensionMethodName&gt;(this &lt;TyprToExtend&gt; instance ,&lt;otherParamters&gt;)
	{
		...
	}
}
&lt;TypeToExtend&gt; myVar;
//myVar is initialized by code  not shown here
myVar.&lt;ExtensionMethodName&gt;();
&lt;TypeToExtend&gt; myVar;
ExtensionClass.&lt;ExtensionMethodName&gt;(myVar);
//导入后可以通过IntelliSense查看扩展方法
//定义了一个扩展方法后还可以将其运用到派生于这个类型的子类型中
namespace ExtensionLib
{
   public static class WordProcessor
   {
      public static List&lt;string&gt; GetWords(
         this string sentence,
         bool capitalizeWords = false,
         bool reverseOrder = false,
         bool reverseWords = false)
      {
         List&lt;string&gt; words = new List&lt;string&gt;(sentence.Split(&#39; &#39;));
         if (capitalizeWords)
            words = CapitalizeWords(words);
         if (reverseOrder)
            words = ReverseOrder(words);
         if (reverseWords)
            words = ReverseWords(words);
         return words;
      }
	public static string ToStringReversed(this object inputObject)
	{
		return ReverseWord(inputObject.ToString());
	}
   }
}
using ExtensionLib;
static void Main(string[] args)
{
 Console.WriteLine(&#34;Enter a string to convert:&#34;);
 string sourceString = Console.ReadLine();
 Console.WriteLine(&#34;String with title casing: {0}&#34;,
    sourceString.GetWords(capitalizeWords: true)
       .AsSentence());
 Console.WriteLine(&#34;String backwards: {0}&#34;,
    sourceString.GetWords(reverseOrder: true,
       reverseWords: true).AsSentence());
 Console.WriteLine(&#34;String length backwards: {0}&#34;,
    sourceString.Length.ToStringReversed());
 Console.ReadKey();
}
```

&#43; lambda表达式
  &#43; 定义一个事件处理方法，其返回类型和参数匹配要订阅的事件需要委托返回类型和参数
  &#43; 声明一个委托类型的变量
  &#43; 把委托变量初始化为委托类型的实例，实例指向事件处理方法
  &#43; 把委托变量添加到时间的订阅者列表中

```C#
正常的事件
Timer myTimer =new Timer(1000);
myTimer.Elapsed&#43;=new ElapsedEvendHandler(WriteChar);
//可以直接写成
myTimer.Elapsed&#43;=WriteChar；
//使用匿名的方法
myTimer.Elapsed&#43;=
	delegate(object source,ElapsedEvendArgs e)
	{
		Console.WriteLine(&#34;Event handler called after {0} milliseconds&#34;),
		(source as Timer).Interval);
	}
//使用lambda表达式
myTimer.Elapsed&#43;=(source ,e )=&gt;Console.WriteLine(
		&#34;Event handler called after {0} milliseconds&#34;,
		(source as Timer).Interval);
//放在括号中的参数列表（未类型化）
//=&gt;运算符
//C#语法
//使用lambda表达式的例子
delegate int TwoIntegerOperationDelegate(int paramA, int paramB);
class Program
{
  static void PerformOperations(TwoIntegerOperationDelegate del)
  {
	 for (int paramAVal = 1; paramAVal &lt;= 5; paramAVal&#43;&#43;)
	 {
		for (int paramBVal = 1; paramBVal &lt;= 5; paramBVal&#43;&#43;)
		{
		   int delegateCallResult = del(paramAVal, paramBVal);
		   Console.Write(&#34;f({0},{1})={2}&#34;,
			  paramAVal, paramBVal, delegateCallResult);
		   if (paramBVal != 5)
		   {
			  Console.Write(&#34;, &#34;);
		   }
		}
		Console.WriteLine();
	 }
  }
  static void Main(string[] args)
  {
	 Console.WriteLine(&#34;f(a, b) = a &#43; b:&#34;);
	 PerformOperations((paramA, paramB) =&gt; paramA &#43; paramB);
	 Console.WriteLine();
	 Console.WriteLine(&#34;f(a, b) = a * b:&#34;);
	 PerformOperations((paramA, paramB) =&gt; paramA * paramB);
	 Console.WriteLine();
	 Console.WriteLine(&#34;f(a, b) = (a - b) % b:&#34;);
	 PerformOperations((paramA, paramB) =&gt; (paramA - paramB)
		% paramB);
	 Console.ReadKey();
  }
}
结果：
f(a, b) = a &#43; b:
f(1,1)=2, f(1,2)=3, f(1,3)=4, f(1,4)=5, f(1,5)=6
f(2,1)=3, f(2,2)=4, f(2,3)=5, f(2,4)=6, f(2,5)=7
f(3,1)=4, f(3,2)=5, f(3,3)=6, f(3,4)=7, f(3,5)=8
f(4,1)=5, f(4,2)=6, f(4,3)=7, f(4,4)=8, f(4,5)=9
f(5,1)=6, f(5,2)=7, f(5,3)=8, f(5,4)=9, f(5,5)=10
f(a, b) = a * b:
f(1,1)=1, f(1,2)=2, f(1,3)=3, f(1,4)=4, f(1,5)=5
f(2,1)=2, f(2,2)=4, f(2,3)=6, f(2,4)=8, f(2,5)=10
f(3,1)=3, f(3,2)=6, f(3,3)=9, f(3,4)=12, f(3,5)=15
f(4,1)=4, f(4,2)=8, f(4,3)=12, f(4,4)=16, f(4,5)=20
f(5,1)=5, f(5,2)=10, f(5,3)=15, f(5,4)=20, f(5,5)=25
f(a, b) = (a - b) % b:
f(1,1)=0, f(1,2)=-1, f(1,3)=-2, f(1,4)=-3, f(1,5)=-4
f(2,1)=0, f(2,2)=0, f(2,3)=-1, f(2,4)=-2, f(2,5)=-3
f(3,1)=0, f(3,2)=1, f(3,3)=0, f(3,4)=-1, f(3,5)=-2
f(4,1)=0, f(4,2)=0, f(4,3)=1, f(4,4)=0, f(4,5)=-1
f(5,1)=0, f(5,2)=1, f(5,3)=2, f(5,4)=1, f(5,5)=0
//lambda表达式的参数,参数可以指定类型，但是不能一个指定，另一个不指定
//(int paramA,int paramB)=&gt;paramA&#43;paramB
//lambda表达式的语句体
(param1,param2)=&gt;
{
	//Multiple statements ahoy
}
(param1,param2)=&gt;
{
	return returnValue;
}
```

	&#43; lambda有两种理解：
		&#43; lambda表达式是一个委托，可以将其赋给一个委托类型的变量，
		&#43; Action，表示lambda表达式不带参数,返回类型void
		&#43; Action&lt;&gt;,表示lambda表达式最多8个参数返回类型是void
		&#43; Func&lt;&gt;,表示lambda表达式最多8个参数，返回类型不是void
		&#43; lambda表达式是表达式树，
		&#43; LINQ框架包含一各泛型Expression&lt;&gt;，可以转换为封装的Lambda表达式
	&#43; lambda表达式和集合
		&#43; 学习Func&lt;&gt;委托后就可以理解，System.Linq名称空间中的为数组提供的扩展方法

```C#
public static TSource Aggregate&lt;TSource&gt;(
	this Enumerable&lt;TSource&gt; source,
	Func&lt;TSource,TSource,TSource&gt; func);
public static TAccumlate Aggregate&lt;TSource,TAccumlate&gt;(
	this Enumerable&lt;TSource&gt; source,
	TAccumlate seed,
	Func&lt;TAccumlate,TSource,TAccumlate&gt; func);
public static TResult Aggregate&lt;TSource,TAccumlate,Aggregate&lt;TSource,TAccumlate,TResult&gt;
	(TResult&gt;(this IEnumerable&lt;TSource&gt; source,
	TAccumlate seed,
	Func&lt;TAccumlate,TSource,TAccumlate&gt; func,
	Func&lt;TAccumlate,TResult&gt;resultSelectoe);
//这是一个累计器
int[] myIntArray={1,2,3,4};
int result = myIntArray.Aggregate(...);
int result = myIntArray.Aggregate&lt;int&gt;(...);
int result = myIntArray.Aggregate((paramA,paramB)=&gt;paramA&#43;paramB);
```

```C#
class Program
{
  static void Main(string[] args)
  {
	 string[] curries = { &#34;pathia&#34;, &#34;jalfrezi&#34;, &#34;korma&#34; };//6,8,5
	 Console.WriteLine(curries.Aggregate(
		(a, b) =&gt; a &#43; &#34; &#34; &#43; b));
	 Console.WriteLine(curries.Aggregate&lt;string, int&gt;(
		0,
		(a, b) =&gt; a &#43; b.Length));
	 Console.WriteLine(curries.Aggregate&lt;string, string, string&gt;(
		&#34;Some curries:&#34;,
		(a, b) =&gt; a &#43; &#34; &#34; &#43; b,
		a =&gt; a));
	 Console.WriteLine(curries.Aggregate&lt;string, string, int&gt;(
		&#34;Some curries:&#34;,
		(a, b) =&gt; a &#43; &#34; &#34; &#43; b,
		a =&gt; a.Length));
	 Console.ReadKey();
  }
}
pathia jalfrezi korma
19
Some curries: pathia jalfrezi korma
35
```

&#43; 调用方信息特征
  &#43; CallFilePath()该特性用于获取调用代码所在的文件路径
  &#43; CallerLineNumber()该特性用于获取调用代码在文件中的行号
  &#43; CallMemberName()获取调用代码所在成员的名称

```C#
class Program
{
  static void Main(string[] args)
  {
	 DisplayCallerInformation();
	 Action callerDelegate = new Action(() =&gt; DisplayCallerInformation());
	 callerDelegate();
	 CallerHelper(callerDelegate);
	 Caller();
	 DisplayCallerInformation(3, &#34;Bob&#34;, @&#34;C:\Temp\NotRealCode.cs&#34;);
	 Console.ReadKey();
  }
  static void CallerHelper(Action callToMake)
  {
	 callToMake();
  }
  static void Caller()
  {
	 DisplayCallerInformation();
  }
  static void DisplayCallerInformation(
	 [CallerLineNumber] int callerLineNumber = 0,
	 [CallerMemberName] string callerMemberName = &#34;&#34;,
	 [CallerFilePath] string callerFilePath = &#34;&#34;)
  {
	 Console.WriteLine(
		string.Format(
		   &#34;Method called from line {0} of member {1} in file {2}.&#34;,
		   callerLineNumber,
		   callerMemberName,
		   callerFilePath));
  }
}
```





























---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/11/dotnetbase2-sql-adonet/  

