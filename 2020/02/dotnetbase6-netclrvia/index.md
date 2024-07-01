# NetCLRVia CLR




## CLR执行模型

### 1 将源码编译成托管模块

在选择framework平台开发后，就面向CLR进行开发

CLR：一个程序，代码块，平台。可以使得多种编程语言使用运行。为了执行面向CLR的代码，电脑必须要安装CLR（目前作为framework的一部分提供）

&#43; 核心功能：内存管理，程序集加载，安全性，异常处理，线程同步。

编译器：代码的语法检查器和分析器。将源码编译成托管模块。

本机编译器：生成面向特定CPU架构(比如x86,x64或ARM)的中间代码。

面向CLR的编译器：

&#43; 微软开发：C&#43;&#43;/CLI、C#、VB、F#、Iron Python、Iron Ruby、中间语言IL
&#43; 其他编译器：![1580263930036](/dotnet/1580263930036.png)

**托管模块**：标准的32位Microsoft Windows可移植执行体(PE32)文件。或者是标准64位Windows可移植执行体(PE32&#43;)文件。这些文件都需要CLR才能执行。PE:Portable Executable,可移植执行体

托管模块各个部分：PE32或PE32&#43;头、CLR头、元数据、IL中间语言代码

![1580264925156](/dotnet/1580264925156.png)

IL也称托管代码

元数据：实际上就是一个数据表集合，定义了类型及成员，还有描述引用类型及其成员，其总是和包含IL代码的文件关联（密不可分）；元数据也是老技术的超集，比如COM的类型库（type library）和接口定义语言（Interface definition language,IDL）文件

编译器总是同时生成元数据和中间代码，并将它们嵌入到生成的托管模块，这样元数据和IL永远不会失去同步。

元数据的用途：

&#43; 避免编译时对C/C&#43;&#43;头和库文件的需求，实现类型和成员的IL中已经含有引用成员类型和成员的全部信息，编译器直接从托管模块中读取元数据。
&#43; Visual studio用元数据帮助写代码。
&#43; CLR的代码验证过程使用元数据确保代码执行类型安全的操作。
&#43; 元数据允许将对象的字段序列化到内存块，将其发送给另一台机器，然后反序列化。
&#43; 元数据允许垃圾回收器跟踪对象生存期。

Microsoft的C&#43;&#43;编译器的独一无二的，只有它可以允许开发人员同时些托管代码和非托管代码，并生成到同一个模块中。

托管程序集总是利用Windows的数据执行保护(Data Execution Prevention,DEP)和地址空间布局随机化(Address Space Layout Randomization,ASLR)增强系统的安全性。

### 2 将托管模块合并成程序集

程序集：一个或多个模块/资源文件的逻辑性分组，是重用、安全性以及版本控制的最小单元。编译器可以生成单文件程序集，也可以生成多文件程序集。

CLR实际和程序集工作，在CLR中程序集相当于组件。

一些托管模块和资源(或数据)文件准备交由一个工具处理，工具生成代表文件逻辑分组的一个PE32(&#43;)文件，PE32文件包含一个清单的数据块，该清单就是元数据表的集合，表描述了构成程序集的文件，公开导出类型，以及与程序集关联的资源或数据文件。

![1580282112266](/dotnet/1580282112266.png)

编译器默认将托管模块转换成程序集。程序集可以是可执行的应用程序，也可以是DLL（其中含有一组可执行程序使用的类型）。

程序集中还包含引用程序集有关的信息（包含版本号，直接依赖的对象），这些信息为自描述的信息。所以与非托管组件相比，不需要再注册表或Active Directory Domain Services(ADDS)中保存额外的信息。

### 3 加载公共语言运行时

CLR现在是作为framework的一部分进行发布的。

检查是否安装Framework，需要检查%SystemRoot%\System32目录中的MSCorEE.dll文件，存在该文件就说明已经安装成功，检查安装哪个版本请检查一下目录

&#43; %SystemRoot%\Microsoft.NET\Framework
&#43; %SystemRoot%\Microsoft.NET\Framework64

.NET Framework SDK提供了CLRVer.exe的命令行使用程序，可以罗列出机器上所有的CLR版本。

如果程序集只包含类型安全的托管代码，再32位和64位都能运行。

如果需要在特定的CPU架构的非托管代码进行互操作。C#编译器提供了/platform命令行选项，这个选项的默认项是anycpu，表明最终生成的程序集能在任何版本的windows上运行。vs设置，程序集属性，生成，目标平台。

针对/platform选项内容，编译器编译的程序集是PE32开头（PE32文件在32或64位系统均可以运行）或PE32&#43;开头（需要在64位地址空间）。最后windows还会检查嵌入的cpu架构信息，确保当前的CPU符合要求。

![1580284150899](/dotnet/1580284150899.png)

Windows检查EXE文件头，决定创建32还是64位进程，在进程地址空间加载MSCorEE.dll的x86,x64或ARM版本。

&#43; windows x86或Arm版本，MSCorEE.dll在%SystemRoot%/System32目录
&#43; Windows x64，MSCorEE.dll在%SystemRoot%System64目录。

进程的主线程调用MSCorEE.dll中定义的一个方法， 在这个方法初始化CLR，加载EXE程序集，在调用其入口方法Main，随即托管应用程序启动并运行。

如果非托管应用程序调用LoadLibrary加载托管程序集，Windows会自动加载初始化CLR处理程序集中的代码。

### 4 执行程序集的代码

IL能够访问和操作对象类型，并提供指令来创建和初始化对象、调用对象上的虚方法以及直接操作数组元素，提供抛出和捕捉异常的指令来实现错误处理。可以将IL视为一种面向对象的机器语言。

IL可以不通过编译器生成，而是直接通过汇编语言编写，微软专门提供了ILAsm.exe的IL汇编器和名位ILDasm.exe的IL反汇编器。甚至来说，一般高级语言只公开了CLR的部分功能，IL汇编语言允许开发人员访问CLR的全部功能。

如何执行程序集代码：

1. 把方法的IL转换本机(native)指令，这部分工作由CLR的JIT(Just-in-time)编译器完成。
2. 在Main方法执行前，CLR监测Main代码中引用的所有类型，并分配一个内部数据结构来管理所有引用类型的访问。

一个例子：

![1580290801135](/dotnet/1580290801135.png)

Main方法引用Console类型，导致CLR分配一个内部结构，在这个内部结构中，Console类型定义的每个方法都有一个对应的记录项。**每个记录项都包含一个地址**，根据这个地址可找到方法的实现IL。对这个结构初始化时，CLR将每个记录项都传进CLR内部一个未编档函数（即JITCompiler）

本书将entry翻译成记录项，还有其他译法为条目，入口。

Main方法首次调用WriteLine时，**JITCompiler函数被调用**，将该方法的IL代码编译成本机CPU指令，由于IL是实时编译，所以将该组件称为JIT实时编译器。

JITCompiler执行流程：

1. JITCompiler函数被调用时，会在定义（该类型的）程序集的元数据中查找被调用的方法IL，然后验证IL代码，并将其编译成CPU指令。随后编译好的CPU指令被保存到动态分配的内存块中。
2. JITCompiler再次回到CLR为类型创建的内部数据结构，找到调用方法对应的记录，修改最初对JITCompiler的引用，**使其指向动态内存块（编译好的本机CPU指令）的地址**。
3. 最后JITCompiler函数跳转到动态内存块中的代码。代码执行完毕并返回到Main中代码，继续执行。
4. 当Main要第二次调用WriteLine时会直接执行内存块中代码，完全跳过JITCompiler函数。

![1580297894750](/dotnet/1580297894750.png)

JIT编译器的执行思路启示：

&#43; 方法调用只有在第一次时才会由性能损失
&#43; 将本机CPU指令存储到动态内存中，一旦程序终止，编译好的代码也会被丢弃。如果再次运行程序，或同时启动应用程序两个实例，JIT编译器都会再次编译IL代码。会增加内存损耗。

第一次调用的性能损失：第一次调用性能损失并不严重，方法内部时间比调用时间多的多。同时CLR的JIT编译器会对本机代码做优化。具体优化如下：

![1580298585131](/dotnet/1580298585131.png)

使用optimize-，将包含许多NOP（no operation,空操作）指令，还有许多跳转到下一行代码的指令。如果生成优化的IL代码，C#编译器会删除掉多余的NOP和分支指令。在控制流程被优化之后，就不能单步调试了。优化后的代码更小。优化后IL更易读。

指定/debug(&#43;/full/pdbonly)，编译器会生成Program Database(PDB)文件，PDB文件帮助调试器查找局部变量并将IL指令映射到源代码。

指定/debug:full，告诉编译器打算调试程序集，JIT编译器会记录每条IL指令生成的本机代码

不指定/debug:full，不记录IL与本机代码的联系，使JIT编译器运行块内存消耗少。

如果进程用VS调试器启动，会强迫JIT编译器记录IL与本机代码的联系，无论设置说明。VS在项目-调试指定/optimize&#43;和/debug:pdbonly开关。

托管程序的性能超过非托管应用程序：

1. JIT编译器能判断应用程序运行在什么类型的CPU上，针对性生成对应CPU的代码，非托管程序不会对CPU有针对性。
2. JIT编译器会针对特定类型的CPU进行代码优化。
3. IL重新编译成本机代码时，会重新组织减少不正确的分支。

.NET Framework SDK配套提供NGen.exe工具，该工具可以将所有IL代码编译成本机代码，保存在磁盘文件中，在运行加载程序集时，CLR自动判断是否存在该程序集的预编译版本，如果是就直接加载预编译代码。

可以使用System.Runtime.ProfileOptimization类，该类导致CLR检查程序运行时哪些方法被JIT编译，结果被记录到一个文件。

#### 4.1 IL和验证

IL基于栈，所有的指令都要将操作数压入栈，并从栈弹出结果。

IL指令是无类型的，IL提供了add指令将压入栈的最后两个操作数加到一起，add指令不分32还是64，add指令执行时，它判断栈中的操作数的类型，并执行恰当操作。

IL最大优势不是对底层CPU的抽象，而是应用程序的健壮性和安全性。将IL编译成本机CPU指令，**CLR执行一个名为验证的过程，验证会检查IL代码，会核实调用每个方法都有正确数量的参数和参数类型，返回值都正确**。托管模块的元数据包含验证过程需要的所有方法及其类型信息。

Windows每个进程都有自己的虚拟地址空间，获得健壮性和稳定性，一个进程干扰不到另一个进程。

通过托管代码，可以确保代码不会不正确的访问内存，不会干扰另一个应用程序的代码，可以将多个托管应用程序放到同一个windows虚拟地址空间运行。

Windows进程需要大量操作系统资源，一个进程运行多个应用程序可以增强性能。

CLR提供了一个操作系统进程中执行多个托管应用程序的能力，每个应用程序都在一个AppDomain中执行，每个托管EXE文件默认在它自己的独立地址空间中运行，这个地址空间只有一个AppDomain。CLR宿主进程（IIS或Microsoft SQL Server）可决定一个进程运行多个AppDomain。

#### 4.2 不安全的代码

Microsoft C#编译器默认生成可验证的安全代码，同时也允许开发不安全的代码，不安全的代码可以直接操作内存地址，操作这些地址处的字节。如果要编写不安全代码，C#编译器要求所有不安全代码都需要使用unsafe 关键字标记，同时还需要打开/unsafe编译器开关。

JIT编译器编译unsafe方法时，会检查程序集是否被授予System.Security.Permissions.SecurityPermission权限，而且System.Security.Permissions.SecurityPermissionFlag的SkipVerification标志是否设置，如果设置才会编译代码，允许代码执行，如果标志没有设置会抛出System.InvalidProgramException或System.Security.VerificationException异常，禁止执行方法。

&#43; 从本地计算机或网络共享加载的程序集默认被授予完全信任，能做任何事情包括不安全代码。
&#43; 从Internet执行的程序集默认不会被授予执行不安全代码的权限。

可以使用Microsoft提供的名为PEVerify.exe的实用程序，用它检查程序集的所有方法，并报告其中含有不安全方法。在引用未知程序集时可以先运行PEVerify.exe，验证是否出现问题。

&#43; PEVerify.exe验证程序集时，必须保证其能够定位并加载引用所有程序集。PEVerify.exe使用CLR来定位依赖的程序集，使用和执行其他程序集一样的绑定和探测规则来定位程序集。

IL的知识产权问题：

生成的IL可以被反汇编器逆向功能，还原应用程序源代码。

&#43; 服务端代码，知识产权完全安全
&#43; 客户端，使用混肴器，打乱程序集元数据中所有私有符号的名称。
&#43; 使用加密算法，例用CLR的互操作功能来实现应用程序的托管与非托管部分之间的通信。

### 5 本机代码生成器：NGen.exe

NGen.exe将IL编译成本机代码。编译好后就不需要JITCompiler实时编译，提升一部分性能。优势：

&#43; 提高应用程序的启动速度。
&#43; 减小应用程序的工作集。编译后的本机代码保存为单独文件。文件可以通过内存映射的方式，同时映射到多个进程地址空间中，使代码得到共享，避免每个进程都需要一份单独代码。

NGen.exe编译过程：

&#43; NGen.exe新建一个程序集文件，只包含本机代码，不含任何IL，新文件会放到%SystemRoot%\Assembly\NativeImage_v4.0.#####_64这样一个目录下的一个文件夹，目录文件除了CLR版本号，还会描述本机代码一些信息，32位还是64位。

&#43; 当CLR加载程序集文件，会检查是否存在有对应的NGen.exe生成的本机文件，找到就使用本机文件中编译好的代码。没有找到就调用JITCompiler实时编译。

NGen.exe存在的问题：

&#43; 没有知识产权保护，在运行时CLR还是要求访问程序集的元数据(用于反射和序列化功能)，这就要求发布包含IL和元数据的程序集。如果本地代码不能执行，则CLR还是会启动JITCompiler
&#43; 失去文件同步，任何不匹配的特性都可能使得本地文件不可用。
  &#43; CLR版本改变
  &#43; CPU类型改变
  &#43; Windows操作系统改变，安装补丁
  &#43; 程序集的标识模块版本
  &#43; 引用程序集版本
  &#43; 安全性改变
&#43; 较差的执行时性能：NGen不能优化使用特定的CPU指令。

最好不要使用NGen，那么怎么解决客户端启动慢的问题，小型程序不需要，大型程序可以使用Microsoft提供的Managed Profile Guided Optimization工具（MPGO.exe）。该工具分析程序执行，检查在启动时哪些需要启动。

### 6 Framework类库入门

.NET Framework包含Framework类库(Framework Class Library,FCL)，微软还有其他库Windows Azure SDK 和Direct SDK,

1. Web 服务(Web service):Microsoft的Asp.net xml web service 和windows communication foundation (WCF)技术，处理Internet发送的消息
2. 基于Html的web窗体/MVC应用程序：Asp.net查询数据库并调用Web服务，合并和筛选返回的消息。
3. Windows GUI应用程序：不用网页创建UI，用Window Store,Windows Presentation Foundation（WPF）或者Windows Forms
4. Windows控制台应用程序：控制台应用程序提供了一种快速简单的方式来生成应用程序
5. Windows服务：通过Windows服务控制管理器(Service Control Manager,SCM)控制这些服务。
6. 数据库存储过程：Microsoft的SQLserver,IBM的DB2以及Oracle的数据库服务器
7. 组件库：.NetFramework 允许生成独立程序集

![1580371231694](/dotnet/1580371231694.png)

### 7 通用类型系统

CLR一切都是围绕着类型，Microsoft定制了一套正式规范来描述类型的定义和行为，称为通用类型系统(Common Type System,CTS)

CTS规范规定，一个类型可以包含成员。

&#43; 字段Field
&#43; 方法Method
&#43; 属性Property
&#43; 事件Event

CTS规范规定类型可见性属性和访问规则。这样就定义了一个类型的可视边界。

&#43; Private:只能在同一类中访问。
&#43; Family：也叫Protected，成员只能由派生类访问。
&#43; family and assembly：成员可有派生类型访问，但是派生类型必须在同一个程序集中定义。
&#43; assembly：也叫internal，同一个程序集中任何代码访问。
&#43; family or assembly : 也叫protected internal ,成员可由任何程序集中的派生类型访问。
&#43; public: 成员可由任何程序集中的任何代码访问。

CTS还为类型集成、虚方法、对象生存期等定义了规则。学习某一类型的编程语言后，该编程语言会将自动将语法映射到IL

### 8 公共语言规范CLS

微软提供了一个最小功能集合，任何语言只要支持这个功能集，生成的类型就能兼容其他符合CLS的组件。最小的语言功能集合称为公共语言规范(Common Language Specification ,CLS)

![1580375461007](/dotnet/1580375461007.png)

```c#
#if !DEBUG
#pragma warning disable 3002, 3005
#endif
using System;

// Tell compiler to check for CLS compliance
[assembly: CLSCompliant(true)]

namespace SomeLibrary {
   // Warnings appear because the class is public
   public sealed class SomeLibraryType {

      // Warning: Return type of &#39;SomeLibrary.SomeLibraryType.Abc()&#39; 
      // is not CLS-compliant
      public UInt32 Abc() { return 0; }

      // Warning: Identifier &#39;SomeLibrary.SomeLibraryType.abc()&#39; 
      // differing only in case is not CLS-compliant
      public void abc() { }

      // No warning: Method is private
      private UInt32 ABC() { return 0; }
   }
}
```

[assembly: CLSCompliant(true)]特性应用于程序集，编译器会检查其中任何公开类型，判断是否存在任何不合适的构造阻止了其他编程语言中访问该类型。

在CLR中，类型的每个成员要么是方法（行为），要么是字段（数据）

```c#
#pragma warning disable 660, 661, 67

using System;

internal sealed class Test {
   // Constructor
   public Test() { }

   // Finalizer
   ~Test() { }

   // Operator overload
   public static Boolean operator ==(Test t1, Test t2) {
      return true;
   }
   public static Boolean operator !=(Test t1, Test t2) {
      return false;
   }

   // An operator overload
   public static Test operator &#43;(Test t1, Test t2) { return null; }

   // A property
   public String AProperty {
      get { return null; }
      set { }
   }

   // An indexer
   public String this[Int32 x] {
      get { return null; }
      set { }
   }

   // An event
#pragma warning disable 67
   public event EventHandler AnEvent;
#pragma warning restore 67
}

```

上述代码包含一个构造器，终结器，操作符，属性，索引器，事件。

.NetFrameworkSDK提供IL反汇编器工具（ILDasm.exe）检查最终生成的托管模块：

![1580376592905](/dotnet/1580376592905.png)

Test类型还有一些节点未列出，包括.class，.custom，AnEvent，AProperty
以及Item,它们标识了类型的其他元数据。这些节点不映射到字段或方法，只是提供了类型的一些额外信息，供CLR、编程语言或者工具访问。例如，工具可以检测到Test类型提供了一个名为AnEvent的事件，该事件借山两个方法(add_AnEvent和remove-AnEvent)公开。

### 9 与非托管代码的互操作性

CLR支持三种互操作情形

&#43; 托管代码能调用DLL中非托管函数：托管代码通过P/Invoke（PlatformInvoke）机制调用DLL中的函数。FCL中许多类型都要从内部调用Kernel32.dll和User32.dll导出的函数。
&#43; 托管diamagnetic可以使用现有COM组件(服务器)：许多公司实现大量非托管的COM组件，利用这些组件的可以创建一个**托管程序集**来描述COM组件。托管代码可以像访问其他任何托管类型一样访问托管程序集中的类型。参考.Net Framework SDK提供的Tlblmp.exe工具。
&#43; 非托管代码可以使用托管类型(服务器)：非托管代码要求提供COM组件来确保代码工作，使用托管代码可以简单实现这些组件，避免所有代码不得不和引用计数以及接口打交道。使用C#创建ActiveX控件或shell扩展。

## 生成、打包、部署和管理应用程序及类型

### 1 NET Framework部署目标

Windows多年来一直因为不稳定和过于复杂而口碑不佳。有以下几个原因。

&#43; 所有的应用程序都使用动态链接库(Dynamic Library,DLL)，不同的厂商更新DLL会影响其他应用程序。
&#43; 安装的复杂性，大多数应用程序在安装时都会影响系统的全部组件。
&#43; 安全性，安装时会有各种各样的文件，其中许多是由不同公司开发的。web应用程序进场会悄悄下载一些代码，这些代码能执行任何操作，包括删除文件或发送电子邮件。

微软面对这些问题在做很多的努力：

&#43; Windows安全性基于用户身份，而代码访问安全性允许宿主设置权限。控制加载的组件能做的事情。.NET Framework允许用户灵活地控制哪些东西能够安装，哪些东西能够运行，对自己及其的控制上升了一定高度。

### 2 将类型生成到模块中

例子：

```c#
public sealed class Program {
   public static void Main() {
      System.Console.WriteLine(&#34;Hi&#34;);
   }
}
```

该程序定义Program类型，有一个名为Main的Public static方法，Main中引用了另一个类型System.Console。System.Console是Microsoft实现好的类型，实现这个类型的各个方法的IL都存储在MSCorLib.dll文件中。

将上面的源代码放到一个源代码文件中，假设是Program.cs，然后执行命令行命令

`csc.exe /out:Program.exe /t:exe /r:MSCorLib.dll Program.cs`

这个命令指示编译器生成Program.exe可执行文件(/out:Program.exe)。生成的文件是Win32控制台应用程序类型(/t[arget]:exe)

编译器编译时发现引用System.Console类型的WriteLine方法，编译器首先核实该类型确实存在，确实有WriteLine方法，而且方法传递的实参和方法形参匹配。然后告诉编译器外部引用，添加了/r[eference]：MSCorLib.dll开关，

MSCorLib.dll是特殊文件，它包含核心类型包括Byte,Char,String,Int32等等，编译器其实会自动引用MSCorLib.dll程序集。简化命令

`csc.exe /out:Program.exe /t:exe Program.cs`

此外/out:Program.exe和/t:exe开关是编译器默认的，可以继续简化

csc.exe Program.cs

生成的Program.exe文件是标准的PE(可移植执行体，Portable Executable)文件。运行32或64位Windows的计算机可以加载，并执行某些动作。Windows支持三种应用程序

&#43; 控制台用户界面：Console User Interface， CUI使用 /t:exe开关
&#43; 图形用户界面:Graphical User Interface,GUI使用/t:winexe开关
&#43; Windows Store应用/t:appcontainerexe开关

**响应文件**：是包含一组编译器命令行开关的文本文件。响应文件以.rsp结尾

使用CSC.exe传递响应文件执行命令，在@符号之后指定响应文件的名称。

假设响应文件MyProject.rsp：

```
/out:MyProject.exe
/target:winexe
```

调用：

`csc.exe @MyProject.rsp CodeFile1.cs CodeFile2.cs`

C#编译器支持多个响应文件，同时还会自动查找名位CSC.rsp的文件，查找CSC.exe所有目录下全局查找CSC.rsp的文件，本地和全局响应文件中的设置冲突，将以本地设置为准，命令行上显示指定设置将覆盖本地响应文件中的设置。

.NET Framework 安装时会在%SystemRoot%\Microsoft.NET\Framework(64)\vX.X.X目录中安装默认全局CSC.rsp文件，文件如下:C:\Windows\Microsoft.NET\Framework64\v4.0.30319：

```c#
# This file contains command-line options that the C#
# command line compiler (CSC) will process as part
# of every compilation, unless the &#34;/noconfig&#34; option
# is specified. 
# Reference the common Framework libraries
/r:Accessibility.dll
/r:Microsoft.CSharp.dll
/r:System.Configuration.dll
/r:System.Configuration.Install.dll
/r:System.Core.dll
/r:System.Data.dll
/r:System.Data.DataSetExtensions.dll
/r:System.Data.Linq.dll
/r:System.Data.OracleClient.dll
/r:System.Deployment.dll
/r:System.Design.dll
/r:System.DirectoryServices.dll
/r:System.dll
/r:System.Drawing.Design.dll
/r:System.Drawing.dll
/r:System.EnterpriseServices.dll
/r:System.Management.dll
/r:System.Messaging.dll
/r:System.Runtime.Remoting.dll
/r:System.Runtime.Serialization.dll
/r:System.Runtime.Serialization.Formatters.Soap.dll
/r:System.Security.dll
/r:System.ServiceModel.dll
/r:System.ServiceModel.Web.dll
/r:System.ServiceProcess.dll
/r:System.Transactions.dll
/r:System.Web.dll
/r:System.Web.Extensions.Design.dll
/r:System.Web.Extensions.dll
/r:System.Web.Mobile.dll
/r:System.Web.RegularExpressions.dll
/r:System.Web.Services.dll
/r:System.Windows.Forms.Dll
/r:System.Workflow.Activities.dll
/r:System.Workflow.ComponentModel.dll
/r:System.Workflow.Runtime.dll
/r:System.Xml.dll
/r:System.Xml.Linq.dll
```

全局CSC.rsp可以带来极大便利，不需要再每次编译都指定。可以使用/noconfig命令行开关，编译器忽略本地和全局CSC.rsp

### 3 元数据概览

托管PE文件由4部门构成：PE32(&#43;)头，CLR头，元素据以及IL.

&#43; PE32(&#43;)头：Windows要求的标准信息

&#43; CLR头：一个小的信息块，需要CLR的模块(托管模块)特有的。包含面向CLR的major(主)和minor(次)版本号；一些标志(flag)；一个MethodDef token(稍后详述),该token指定了模块的入口方法(这个模块是CUI,GUI,或WIndowsStore执行体)；一个可选的强名称数字签名，还包含模块的一些元数据表的大小和偏移量。可以查看CorHdr.h头文件定义的IMAGE_COR20_HEADER了解CLR头的具体格式。

&#43; 元数据：几个表构成的二进制数据块。由三种表：**定义表**definition table，**引用表**reference table，**清单表**manifest table。

1. 定义表列举一些：

![1580450364815](/dotnet/1580450364815.png)

源代码的任何改变都会导致上面列出表中创建一个记录项，编译器还会检查源代码引用类型、字段、方法、属性和事件，并创建相应的元数据表记录项。

2. 引用表列举一些：

![1580453721175](/dotnet/1580453721175.png)

![1580453742252](/dotnet/1580453742252.png)

微软提供了很多工具检查托管PE文件中的元数据。推荐ILDasm.exe,IL Diasaaembler(IL反汇编器)，执行命令行：ILDasm Program.exe

![1580454364796](/dotnet/1580454364796.png)

![1580454383932](/dotnet/1580454383932.png)

![1580454408352](/dotnet/1580454408352.png)

ILDasm中选择视图、统计

![1580454584055](/dotnet/1580454584055.png)

可以看到文件大小即字节数以及文件各部分大小（字节数和百分比）

### 4 将模块合并成程序集

程序集:是一个或多个类型定义文件及资源文件的集合。 在这些文件中，有一个文件中容纳了清单(manifest)。清单是一个元数据表集合，主要包含作为程序集组成部分的那些文件名称。清单还描述了程序集的版本、语言文化、发布者、公开导出类型及构成程序集的所有文件。

CLR操作程序集，首先要加载元数据表中清单文件，根据清单来获取程序集中其他文件的名称。

程序集的清单表重要特征：

&#43; 定义了可重用的类型
&#43; 用一个版本号标记
&#43; 可以关联安全信息

程序集大多数情况下只有一个文件，有些情况下还有多个文件构成：一些是含有元数据的PE文件，另一些是.GIF或.JPG这样的资源文件。程序集视为逻辑EXE或DLL

为什么引入程序集概念，因为使用程序集可以便于重用和物理标识分开。可以将常用的类型放到一个文件中，不常用的放到另一个文件中。

为了配置应用程序载入程序集文件，可在应用程序配置文件中指定codeBase元素，codeBase定义了URL指向的未知，找到程序集的所有文件。例如：加载程序集的一个文件时，CLR获取codeBase元素的URL，检查机器的下载缓存，判断文件是否存在。如果是直接加载文件，不是，CLR去URL指向的位置将文件下载到缓存中，如果找不到抛出FileNotFoundException异常。

使用多文件程序集的理由：

&#43; 不同的类型用不同的文件，使文件能以增量方式下载。可以进行部分或分批打包部署。
&#43; 可以在程序集中添加资源或数据文件。
&#43; 程序集包含的各个类型可以用不同的编程语言来实现。

生成程序集要么选择现有PE文件作为清单的宿主，要么创建单独的PE文件。

将托管模块转换成程序集清单元数据表：

![1580459358129](/dotnet/1580459358129.png)

有了清单，程序集用户不必关心程序集的划分细节。清单也使程序集具有自描述性。

程序集文件中还有一个AssemblyRef表，程序集全部文件引用的每个程序集在这个表中都有一个记录项，工具只需打开程序集的清单，就可知道它引用的全部程序集，而不必打开程序集的其他文件。同样地，AssemblvRef的存在加强了程序集的自描述性

使用以下命令行，C#编译器会生成程序集

&#43; /t[arget]:exe生成CUI执行体
&#43; /t[arget]:winexe生成GUI执行体
&#43; /t[arget]:appcontainer生成windows store执行体
&#43; /t[arget]:library生成类库
&#43; /t[arget]:winmdobj生成WINMD库

&#43; /t[arget]:module生成不包含清单元数据表的PE文件。

VS集成开发环境不能直接创建多文件程序集，只能用命令行工具创建多文件程序集。

怎么向程序集中添加模块：

含清单的PE文件可以使用/addmodule开发。生成多文件程序集，假设两个源码文件RUT.cs含有不常用类型和FUT.cs含有常用类型

csc /t:module RUT.cs，创建含有不常用类型的托管模块。生成RUT.netmodule文件。这是一个标准DLL PE文件，但是CLR不能加载

同样的方式编译另一个模块，这个模块作为程序集清单的宿主。将程序集命名为MultiFileLibrary.dll,而不是FUT.dll

csc /out:MultiFileLIbrary.dll /t:library /addmodule:RUT.netmodule FUT.cs

C#编译器编译FUT.cs生成MultiFileLibrary.dll。/t:library：生成含有清单元数据表的DLL PE文件。/addmodule:RUT.netmodule告诉编译器RUT**.netmodule**文件是程序集的一部分，其实就是将文件添加到FileDef清单元数据表，并将RUT.netmodule的公开导出类型添加到ExportedTYpesDef清单元数据表。

![1580462045724](/dotnet/1580462045724.png)

生成MultiFileLibrary.dll程序集后，可用ILDasm.exe检查元数据的清单表。

FileDef和ExportedTypesDef元数据表的内容：

![1580462165007](/dotnet/1580462165007.png)

客户端代码必须使用/r[eference]:MultiFileLibrary.dll编译器开关生成，才能使用MultiFileLibrary.dll程序集类型。该开关指示程序集搜索外部类型时加载MultiFIleLibrary.dll程序集以及FileDef表中列出的所有文件。如果删除RUT.netmodule文件C#编译器会报告一下错误：fatal error CS0009：未能打开元数据文件“C:\MultiFileLibrary.dll”-&#34;导入程序集&#34; “C:\MultiFileLibrary.dll”的模块“RUT.netmodule”时出错-系统找不到指定文件

![1580462445296](/dotnet/1580462445296.png)

当一个方法首次调用时，CLR检测作为参数，返回值，局部变量而被方法引用的类型。如果文件存在直接执行，不存在执行内部登记并允许使用该类型。**只有方法被调用确实引用了未加载程序集时才会加载程序集。**

Vs添加引用：添加引用-引用管理器。COM选项允许从托管代码中访问非托管COM服务器。

#### 使用程序集链接器

除了C#编译器还可以使用程序集连接器的使用程序AL.exe来创建程序集

&#43; 程序集包含不同编译器生成的模块
&#43; 生成程序集时不清楚打包要求，

&#43; AL.exe生成只包含资源的程序集，也就是附属程序集。

使用AL.exe生成MultiFileLibrary.dll

csc /t:module RUT.cs

csc /t:module FUT.cs

al /out:MultiFileLibrary.dll /t:library FUT.netmodule RUT.netmodule

![1580464541310](/dotnet/1580464541310.png)

这个程序集由三个文件构成：MultiFileLibrary.dll,RUT.netmodule和FUT.netmodule,程序链接器不能将多个文件合并成一个文件。

csc /t:module /r:MultiFileLibrary.dll Program.cs

将Program.cs生成Program.netmodule文件。

al /out:Program.exe /t:exe /main:Program.Main Program.netmodule

生成包含清单元数据表的Program.exe PE文件。

由于使用了/main:Program.Main命令行开关，AL.exe生成一个小的全局函数，名为_EntryPoint。

![1580467168094](/dotnet/1580467168094.png)

```c#
@echo off
Rem %1=&#34;$(DevEnvDir)&#34;, %2=&#34;$(SolutionDir)&#34;, %3=&#34;$(OutDir)&#34;

rem Set all the VS environment variables
pushd %1
call ..\Tools\VSVars32.bat
popd

rem Change to the solution directory
cd %2

REM There are two ways to build this multi-file assembly
REM The line below picks one of those ways
Goto Way1

:Way1
csc /t:module  /debug:full /out:Ch02-3-RUT.netmodule Ch02-3-RUT.cs
csc /t:library /debug:full /out:Ch02-3-MultiFileLibrary.dll /addmodule:Ch02-3-RUT.netmodule Ch02-3-FUT.cs Ch02-3-AssemblyVersionInfo.cs
md %3
move /Y Ch02-3-RUT.netmodule %3
move /Y Ch02-3-RUT.pdb %3
move /Y Ch02-3-MultiFileLibrary.dll %3
move /Y Ch02-3-MultiFileLibrary.pdb %3
goto Exit

:Way2
csc /t:module /debug:full /out:Ch02-3-RUT.netmodule Ch02-3-RUT.cs
csc /t:module /debug:full /out:Ch02-3-FUT.netmodule Ch02-3-FUT.cs Ch02-3-AssemblyVersionInfo.cs
al  /out:Ch02-3-MultiFileLibrary.dll /t:library Ch02-3-RUT.netmodule Ch02-3-FUT.netmodule
md %3
move /Y Ch02-3-RUT.netmodule %3
move /Y Ch02-3-RUT.pdb %3
move /Y Ch02-3-FUT.netmodule %3
move /Y Ch02-3-FUT.pdb %3
move /Y Ch02-3-MultiFileLibrary.dll %3
move /Y Ch02-3-MultiFileLibrary.pdb %3
goto Exit

:Exit


```

#### 为程序集添加资源文件

AL.exe创建程序集可用/embed[resource]开关将文件作为资源添加到程序集。将文件内容嵌入最终的PE文件。清单的ManofestResourceDef表会更新反映资源存在。

AL.exe支持/link[resource]开关，同样获取包含资源的文件，但只更新ManifestResourceDef和FileDef表以反映资源存在。资源文件不会嵌入PE文件，保持独立，和其他程序集文件一起打包和部署。

CSC.exe也允许将资源合并到编译器生成的程序集中，/resource开关可以指定资源文件嵌入最终生成的程序集PE文件中，并更新ManifestResourceDef表。/linkresource开关在MainifestResourceDef和FileDef清单表中添加记录项来引用独立存在的资源文件。

程序集中也可以嵌入标准的Win32资源，只需要使用AL.exe或CSC.exe时使用/win32res开关指定.res文件路径。也可以使用/win32icon开关指定.ico文件路径。

/nowin32manifest可以不生成win32清单资源信息，默认是生成。默认清单信息：

![1580468167044](/dotnet/1580468167044.png)

### 5 程序集版本资源信息

AL.exe或CSC.exe生成PE文件程序集，还可能在PE文件中嵌入标准Win32版本资源。可以查看文件属性来检查该资源。在源代码中调用System.Diagnostics.FileVersionInfo的静态方法GetVersionInfo方法，传递程序集的路径，

![1580468295888](/dotnet/1580468295888.png)

```c#
using System.Reflection;

// FileDescription version information:
[assembly: AssemblyTitle(&#34;MultiFileLibrary.dll&#34;)]

// Comments version information:
[assembly: AssemblyDescription(&#34;This assembly contains MultiFileLibrary&#39;s types&#34;)]

// CompanyName version information:
[assembly: AssemblyCompany(&#34;Wintellect&#34;)]

// ProductName version information:
[assembly: AssemblyProduct(&#34;Wintellect (R) MultiFileLibrary&#39;s Type Library&#34;)]

// LegalCopyright version information:
[assembly: AssemblyCopyright(&#34;Copyright (c) Wintellect 2013&#34;)]

// LegalTrademarks version information:
[assembly:AssemblyTrademark(&#34;MultiFileLibrary is a registered trademark of Wintellect&#34;)]

// AssemblyVersion version information:
[assembly: AssemblyVersion(&#34;3.0.0.0&#34;)]

// FILEVERSION/FileVersion version information:
[assembly: AssemblyFileVersion(&#34;1.0.0.0&#34;)]

// PRODUCTVERSION/ProductVersion version information:
[assembly: AssemblyInformationalVersion(&#34;2.0.0.0&#34;)]

// Set the Language field (discussed later in the &#34;Culture&#34; section)
[assembly:AssemblyCulture(&#34;&#34;)]

```

![1580468373131](/dotnet/1580468373131.png)

VS中新建C#项目会在Properties文件夹中自动创建AssemblyInfo.cs文件。

#### 版本号

![1580468440439](/dotnet/1580468440439.png)

前两个编号构成公众对版本的理解。

第三个编号是程序集的build号，如果公司每天更新程序集，需要更新build号，

最后一个指出当前build修订次数。如果某天必须生成2次程序集，revision就递增。

程序集由三个版本号

1. AssemblyFileVersion

该版本号存储在Win32版本资源中，CLR即不检查也不关心这个版本号，先设置好版本的major/minor部分，然后每次生成就递增build和revision号。资源管理器中能看到这个版本号.

2. AssemblyInformanceVersion

该版本号也存储在Win32中，CLR不检查也不关心，其作用是指出该程序集的产品的版本。

3. AssemblyVersion

该版本号存储在AssemblyDef清单数据表中，CLR在绑定到强命名程序集时会用到它。这个版本号唯一标识了程序集。开始开发程序集时，应该设置号major/minor/build/revision部分。除非开发下一个版本，否则不要变动。

### 6 语言文化

程序集还将语言文化Culture作为身份标识的一部分。

![1580481469630](/dotnet/1580481469630.png)

含有代码的程序集一般不指定具体语言文化。未指定具体语言文化的程序集称为语言文化中性。

如果包含语言文化特有的资源，微软强烈建以专门创建一个程序集来包含代码和应用程序的默认资源。生成程序集时不要指定具体的语言文化，其他程序集通过引用该程序集来创建和操纵它公开的类型。

只包含语言文化特有的资源，标记语言文化的程序集称为附属程序集。

AL.exe生成附属程序集，使用其/c[ultrue]:text开关指定语言文化。其中text是语言文化字符串。en-US代表英语。部署附属程序集时，应该把它保存到专门的子目录中，子目录名称和语言文化的文本匹配。

假设应用程序的基础目录是C:\MyApp,与美国英语对应的附属程序集就应该放到C:\MyApp\en-US子目录，在运行时，使用System.Resources.ResourceManager类访问附属程序集的资源

可以使用定制特性System.Reflection.AssemblyCultureAttribute代替AL.exe的/culture开关

[assembly:AssemblyCulture(&#34;de-CH&#34;)]

一般不要引用附属程序集。程序集的AssemblyRef记录向只引用语言文化中性的程序集。

### 7 简单应用程序部署(私有部署程序集)

WindowsStore应用程序集打包有一套严格的规则，VS将应用程序所有必要的程序集打包成一个.appx文件。其中包含的所有程序集都进入一个目录，CLR从该目录加载程序集，

桌面应用，没有任何特殊要求，

其他机制的打包和安装程序集文件。使用.cab文件(从Internet下载使用，压缩文件并缩短下载事件)，MSI文件(由windows的Installer服务MSIExec.exe使用)。使用MSI文件可实现程序集的按需安装。CLR首次尝试加载一个程序集时才安装它。

VS内建的机制发布应用程序，打开项目属性页点击发布标签，利用其中的选项将VS生成的MSI文件复制到网站、FTP服务器或文件路径。MSI安装必备组件，比如.NET Framework或Microsoft SQL Server Express Edition。利用ClickOnce技术，可以自动检查更新。

私有部署程序集：应用程序基于目录或者子目录部署的程序集。程序集为念不和其他应用程序共享。这样可以带来诸多便利，移动复制文件夹就可以。

实现这种间件的部署，是因为每个程序集都用元数据注明了自己引用的程序集，不需要注册表设置。同时引用程序集限定了每个类型的作用域。应用程序总会和它生成和测试时的类型绑定。在COM中，类型是在注册表中登记的，造成机器上运行的任何应用程序都能使用那些类型。

### 8 简单管理控制(配置)

实现对应用程序控制在应用程序目录放入一个配置文件，安装程序会将配置文件安装到应用程序的基目录，CLR会解析文件内容来更改程序集文件的定位和加载策略；同时计算机管理员或最终用户能创建或修改该文件。

配置文件包含XML代码，既能和应用程序关联，也和机器关联。

例子：假设应用程序的发布者想把MultiFileLibrary程序集文件部署到不同的目录，要求目录结构如下：

![1580527730610](/dotnet/1580527730610.png)

不再同一目录下，CLR无法定位这些文件会抛出System.IO.FileNotFoundException异常。为解决这个为题需要创建XML配置文件，并将其部署到应用程序基目录，文件名必须是应用程序主程序集文件名，并加.config扩展名，也就是Program.exe.config，配置文件：

![1580527928345](/dotnet/1580527928345.png)

CLR定位策略：定位程序集时，先在应用程序基目录查找，如果没有，就查找AuxFiles子目录。可为probing元素的privatePath特性指定多个以分号分隔的路径。每个路径都要相对应用程序基目录，不能使用绝对或相对路径指定应用程序基目录外的目录。这个设计的出发点是应用程序只能控制它的目录及子目录

XML配置文件名称和位置取决于应用程序类型

&#43; exe程序，配置文件必须在应用程序基目录，必须采用exe文件全名作为文件名，再附加.config扩展名
&#43; 对于Microsoft ASP.NET Web窗体应用程序，文件必须再Web应用程序的虚拟根目录中，而且总是命名为Web.config。此外子目录可以包含自己的Web.config，配置设置会得到继承。例如http://Wintellect.com/Training的web应用程序既使用虚拟根目录Web.config设置，也会使用Training子目录。

配置应用机器，.NetFramework安装创建Machine.config，每个版本的CLR都有对应的Machine.config

Machine.config所在目录：%SystemRoot%\Microsoft.NET\Framework\version\CONFIG

CLR定位程序集会扫描几个子目录，例子：加载一个语言文化中性的程序集的目录扫描顺序（firstPrivatePath和secondPrivatePath通过配置为念privatePath特性指定）

![1580533694016](/dotnet/1580533694016.png)

如果MultiFileLibrary程序集文件部署到MultiFileLibrary子目录，就无需配置文件，因为CLR会自动扫描与目标程序名称符合的子目录。

如果上述的任何子目录都找不到目标应用程序，CLR会从头再来，用.exe扩展名替换.dll扩展名，再找不到就抛出FileNotFoundException异常。

附属程序集遵循的规则，CLR会再应用程序基目录下的一个子目录中查找，子目录名称与语言文化相符，假如AsmName.dll应用了“en-US”语言文化，会探测以下目录

![1580534744210](/dotnet/1580534744210.png)

CLR会扫描具有.exe或.dll扩展名的文件，这个是比较耗时的，所以最好在XML配置文件中指定一个或多个culture元素。限制CLR查找附属程序集的扫描。

Machine.config文件的设置是机器上运行的所有应用程序的默认设置。最好不要修改。

## 共享程序集和强命名程序集

进行私有部署，程序集放在应用程序的基目录或子目录，可以保证对程序集的命名版本和行为进行全面控制。

共享程序集：由多个应用程序共享的程序集。Microsoft .NET Framework 携带程序集就是典型的全局部署程序集。所有托管程序都需要使用Microsoft中的.Net Framework Class Library（FCL）

Windows稳定性很差，是因为共享程序集出现问题，在出现问题的时候修复BUG同时添加新功能，是完全不可能的。退而求其次，在出现问题的时候，有一种简单的方式将应用程序恢复到上一次已知的良好状态。

### 1 两种程序集、两种部署

CLR支持两种程序集：弱命名程序和强命令程序集。

弱命名和强命名程序集在结构上完全相同，真正的区别在于使用发布者的公钥/私钥进行了签名。这一对密钥允许对程序集进行唯一性的标识，保护和版本控制，并允许程序集部署到任何地方。

程序集一旦被唯一性标识，CLR可以应用一些已知的安全策略。

弱命名程序集只能私有部署，强命令程序集可以私有可以公有

![1580538551107](/dotnet/1580538551107.png)

### 2 为程序集分配强名称

两个相同名的程序集放到一个目录下就以最后安装的程序集为准。

只根据为念名来区分程序集显然不够，CLR支持对程序集进行唯一性标识机制。

强命名程序集：共同对程序集进行唯一性标识：文件名(不计扩展名)、版本号、语言文化和公钥。公钥数字很大，所以经常使用从公钥派生的最小哈希值，称为公钥标记。例如下面标识了四个不同的程序集

![1580539786038](/dotnet/1580539786038.png)

必须有一种技术能够区分相同特征的程序集。Microsoft选择了标准的公钥/私钥加密技术，没有采用唯一性标识（GUID:全球唯一标识符，URL：同一资源定位，URN：统一资源名）

使用加密技术，不仅能在程序集安装到机器时检查二进制数据的完整性，还可以允许每个发布者授予一套不同的权限。

如果公司想要唯一性标识自己的程序集，必须创建一对公钥/私钥，公钥可以和程序集关联。

辅助类System.Reflection.AssemblyName轻松构造程序集名称，并获取程序集的各个组成部分，几个公共实例属性：CultureInfo,FullName,KeyPair,Name和Version.方法：GetPublicKey,GetPublicKeyToken,SetPublicKey和SetPublicKeyToken.

创建强命名程序集：

用.NET Framewrk SDK和Microsoft Visual Studio带的Strong Name实用程序(SN.exe)获取密钥。SN.exe允许通过多个命令行开关来使用一整套功能。

SN -k MyCompany.snk

使用SN.exe创建MyCompany.snk文件，文件中包含二进制形式的公钥和私钥。

公钥数字很大，可以使用SN.exe查看实际公钥。执行两次SN.exe。使用-p开关可以创建只含公钥的文件(MyCompany.PublicKey)

SN -p MyCompany.snk MyCompany.PublicKey sha256

第二次使用-tp开关就行

SN -tp MyCompany.PublicKey

![1580541958241](/dotnet/1580541958241.png)

SN.exe未提供任何显示私钥的途径。

可以发现公钥非常大，简化开发又设计了公钥标记，公钥标记是公钥64位哈希值，SN.exe的-tp开关输出结果显示了公钥标记。

创建含有公钥/私钥对的程序集，使用/keyfile:\&lt;file\&gt;编译开关。

csc /keyfile:MyCompany.snk Program.cs

C#编译器查到/keyfile:\&lt;file\&gt;开关会指定打开文件MyCompany.snk，用私钥对程序集进行签名，并将公钥嵌入清单。注意只能**对含清单的程序集**文件进行签名，程序集其他文件不能被显示签名。

VS中创建公钥私钥，项目属性-签名-为程序集签名-选择强名称密钥文件，新建。

对文件签名的准确含义是：生成强命名程序集时，程序集的FileDef清单元数据表列出构成程序集的所有文件。每将一个文件名添加到清单，都会对文件内容进行哈希处理，哈希值和文件名一并存储到FileDef表。

如果想覆盖默认哈希算法，可使用AL.exe的/algid开关，在程序集源代码文件中使用特性，System.Reflection.AssemblyAlgorithmIdAttribute.默认使用SHA-1算法。

对PE文件的完整内容进行哈希处理。哈希值用发布者的私钥进行签名。得到RSA数字签名存储到PE文件的保留区域。PE文件的CLR进行更新。

![1580544121278](/dotnet/1580544121278.png)

发布者公钥也嵌入PE文件的AssemblyDef清单元数据表。文件名、程序集版本号、语言文化和公钥的组合保证程序集是强名称，唯一性。

托管模块的AssemblyRef元数据表，每个记录项都被指明引用程序集的名称、版本号、语言文化和公钥信息

简单类库DLL文件的AssemblyRef元数据信息

![1580544428609](/dotnet/1580544428609.png)

这个DLL引用了以下特性的程序集类型

![1580544469824](/dotnet/1580544469824.png)

检查程序集的AssemblyDef元数据表

![1580544512548](/dotnet/1580544512548.png)

等价于

![1580544572894](/dotnet/1580544572894.png)

没有公钥标记，就是弱命名程序集，

用SN.exe创建密钥文件，再用/keyfile编译器进行编译，

使用ILDasm.exe查看新程序集的元数据，AssemblyDef记录项就会在PublicKey字段之后显示相应字节。AssemblyDef记录项总是存储完整公钥，而不是公钥标记。

### 3 全局程序集缓存

程序集放到公认的目录，而且CLR在检测到对该程序集引用时，必须知道检查该目录。这个公认位置就是全局程序集缓存（Global Assembly Cache,GAC），

%SystemRoot%\Microsoft.NET\Assembly

GAC目录结构化：包含子目录，子目录名称用算法生成。永远不要将程序集文件手动复制到GAC目录，用工具完成这项任务，工具知道GAC内部结构，并知道如何生成正确的子目录名。

开发和测试GAC中安装强命名程序集工具是GACUtil.exe，直接启动会显示语法。

![1580545701689](/dotnet/1580545701689.png)

![1580545715502](/dotnet/1580545715502.png)

GACUtil.exe的/i开关将程序集安装到GAC，/u开关从GAC卸载程序集。不能将弱命名程序集放到GAC中。如果放入会报错。

GAC默认只能由Windows Administrators用户组成员操作。

.NetFramework不携带GACUtil.exe工具，应该使用Windows Installer（MSI）

程序集安装到GAC破坏了一个基本目标，简单安装，备份还原移动和卸载应用程序，程序集尽量私有。

### 4 在生成的程序集中引用强命名程序集

任何程序集都包含对其他强命名程序集的引用，System.Object在MSCorLib.dll中定义。

使用CSC.exe的/reference编译器开关指定想引用的程序集。CSC.exe会加载指定文件，根据元数据生成程序集。如果没有指定文件路径，CSC.exe会尝试以下目录查找程序集。

&#43; 工作目录
&#43; CSC.exe所在的目录，目录中还包含CLR的各种DLL文件。
&#43; 使用/lib编译器开关指定任何目录
&#43; 使用LIB环境变量指定的任何目录。

生成的程序集引用了Microsoft的System.Drawing.dll，执行CSC.exe使用/reference:System.Drawing.dll开关。依次检查上述目录，并在CSC.exe所在的目录找到System.Drawing.dll文件（运行时不会从这个目录查找程序集）。

安装.net framework时，安装Microsoft程序集文件两套拷贝。一套安装带编译器/CLR目录方便生成程序集，一套安装GAC子目录，方便运行时加载。

CSC.exe编译器之所以不再GAC中查找引用程序集。是因为代码者必须知道程序集路径，GAC也没有正式公开，还有一种方案是指定一个很长的字符串，比如“System.Drawing,Version=v4.0.0.0,Culture=neutral,PublicKeyToken=b03f5f7f11d50a3a”,两个方案都没有在用户硬盘上安装两套一样的程序集方便。

编译器/CLR目录中程序集不依赖机器，也就是说这些程序集只包含元数据。由于编译时不需要IL代码，GAC中程序集同时包含元数据和IL代码，

### 5 强命名程序集能防篡改

用私钥对程序集签名，同时将公钥和签名嵌入到程序集，CLR可验证程序集未被修改或破坏。

**程序集安装到GAC时，系统对包含清单的每个文件的内容进行哈希处理，将哈希值与PE文件中嵌入的RSA数字签名进行比较（在用公钥接触签名之后）**。如果两个值完全一致，说明文件内容没有被篡改。系统还对程序集中的其他文件进行哈希处理，并将哈希值与清单文件FileDef表中存储的哈希值进行比较，任何一处哈希值不匹配，表明至少有一个文件被篡改，程序集无法安装GAC。

应用程序需要定位程序集，CLR根据引用程序集的属性(名称，版本，语言文化和公钥)，在GAC中定位程序集，找到引用程序集就返回包含它的子目录，并加载清单所在的文件，这种方式查找程序集，可以保证运行时加载程序集和最初编译时生成的程序集来自同一个发布者。引用程序集的AssemblyRef表中公钥标记与被引用程序集的AssemblyRef表的公钥匹配。如果被引用程序集不在GAC中，CLR会查找应用程序的基目录，然后查找应用程序配置文件中标注的任何私有路径。如果应用程序是MSI安装，CLR要求MSI定位程序集，然后都找步到程序集，那么绑定失败，抛出System.IO.FileNotFoundException。

强命名程序集文件从GAC之外位置加载（应用程序基目录，配置文件codeBase元素），CLR在加载程序集后比较哈希值，也就是每次应用程序执行加载程序集，都会对文件进行哈希值，如果不匹配会抛出System.IO.FileLoadException异常。

强命名程序集安装到GAC时，系统会执行检查，确保清单的文件没有被篡改，该检查知会在安装时执行一次，同时强命名程序集完全被信息，并加载到完全信息的AppDomain中，CLR将不检查程序集是否被篡改。如果从非GAC的目录加载强命名程序集时，CLR会校验程序清单文件。

### 6 延迟签名

在准备打包自己的强命名程序集时，必须使用私钥对其进行签名，但是在开发和测试程序集时，访问私钥保护的程序集比较繁琐，所以.NetFramework提供了延迟签名技术，也成为部分签名，就是暂不使用私钥，用公钥生成程序集。使用公钥，引用程序集的其他程序集在AssemblyRef元数据表的记录中嵌入公钥值。同时还使得程序集能能够正确存储到GAC的内部结构中。

如果不用私钥对文件进行签名就无法实现防篡改，因为无法对程序集文件进行哈希处理，无法在文件中嵌入数字签名。

C#编译器可以指定/delaysign编译器开关，如果vs就打开项目属性也，在签名选项卡中勾选仅延迟签名，如果AL.exe就指定/delay[sign]命名行开关。如果想实现延迟签名，获取存储在文件中的公钥，需要将公钥文件的文件名传给生成程序集的实用程序。

编译器一旦检测到对程序集进行延迟签名，就会生成程序集的AssemblyDef清单记录想，其中包含程序集的公钥，公钥使程序集能够正确存储到GAC中。同时也不影响引用该程序集的其他程序集的生成。进行引用程序集的AssemblyRef元数据记录项中，包含（被引用程序集的）正确公钥，创建程序集使，会在生成PE文件中为RSA数字签名预留空间。文件内容不会进行哈希处理。

目前程序集没有有效签名，安装GAC会失败。在需要安装到GAC的机器上，需要禁止系统验证程序集完整性，使用SN.exe并指定-Vr命令行开关。这个开关会使得程序集的任何文件在运行时加载，并且CLR会跳过对其哈希值的检查。在内部SN的-Vr开关会将及程序及的身份添加到注册表的子项中：

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\StrongName\Verification

结束程序集开发和测试后，要正式对其进行签名，以便打包部署。再次使用SN.exe程序使用-R开关，指定包含私钥的文件。-R开关指示SN.exe对文件内容进行哈希处理，并用私钥对其签名，并将RSA数字签名嵌入文件中预留的空间。经过这一步之后，就可以部署完全签名号的程序集。同时不要忘记使用SN.exe的-Vu和-Vx命令行重新启动对程序集的验证。

延迟签名总结：

1. 开发期间，获取公钥，使用/keyfile和/delaysign编译器开关

csc /keyfile:MyCompany.PublicKey /delaysign MyAssembly.cs

2. 使用以下命令可以使CLR暂时信任程序集的内容，不对它进行哈希处理，也不对哈希值进行比较。这样程序集可以顺利安装到GAC。每台机器上都必须执行这一命令。

SN.exe -Ra MyAssembly.dll MyCompany.PrivateKey

3. 准备打包和部署程序集时，使用私钥执行以下命令

SN.exe -Ra MyAssembly.dll MyCompany.PrivateKey

4. 重新启动对程序集的验证

SN -Vu MyAssembly.dll

大公司会将自己的密钥存储到硬件设备(智能卡中)，确保密钥安全性，密钥值不能固定存储在磁盘上。&#34;加密服务提供程序(Cryptographic Service Provider ,CSP)&#34;提供对这些密钥位置进行抽象的容器。如果Microsoft使用CSP为例，一旦访问它提供的容器，就自动从一个硬件设备中获取私钥。

如果公钥/私钥在CSP容器中，必须为CSC.exe、AL.exe和SN.exe指定不同开关。

编译器（CSC.exe）指定/keycontainer开关而不是/keyfile开关

链接时Al.exe指定/keyname开关而不是/keyfile开关

强名称程序SN.exe对延迟签名的程序集重新签名指定-Rc开关而不是-R开关

在打包前，先进行混淆器程序再完全签名。

### 7 私有部署强命名程序集

GAC安装程序集有几个方面优势：

1. 能被多个应用程序共享，减少总的物理内存
2. 很容易将新版本部署到GAC，让所有程序都通过发布者策略使用新版本。
3. 实现了对程序集多个版本的并行管理

管理员才能安装到GAC，一旦安装到GAC就违反了简单部署的原则

多个应用程序共享的程序集才应部署到GAC。同时，GAC部署新的C:\Window\System32垃圾堆积场，因为新版本程序集不会相互覆盖，它们并行安装。

强命名程序集除了部署到GAC或进行私有部署，部署到私有目录，强命名程序集由三个应用程序共享，安装可创建三个目录，每个程序一个目录，再创建四个目录，专门存储共享程序集。每个应用程序安装到自己的目录时都同时安装一个XML配置文件，用codeBase元素指出共享程序集路径。运行时，CLR知道查找共享程序集。

codeBase元素实际标记了一个URL,这个URL可引用机器上的任何目录，也可引用Web地址。如果引用Web地址，CLR自动下载文件，存储到用户缓存（%UseProfile%\Local Settings\Application Data\Assembly下的子目录）。

### 8 运行时如何解析类型引用

```c#
public sealed class Program {
   public static void Main() {
      System.Console.WriteLine(&#34;Hi&#34;);
   }
}
```

编译代码生成程序集Program.exe，运行应用程序，CLR加载并初始化自身，读取程序集CLR头，查找应用程序入口Main的MethodDefToken,检索MethodDef元数据表找到方法的IL代码载文件中的偏移量，将IL代码JIT编译成本机代码(编译时会对代码进行验证确保类型安全)，执行本机代码。Main方法的IL代码，可以对程序集运行ILDasm.exe并选择视图|显示字节

![1580641925600](/dotnet/1580641925600.png)

这些代码进行JIT编译，CLR会检查所有类型和成员的引用，加载它们程序集，上诉对System.Console.WriteLine引用，IL call指令引用元数据token 0A000003，该token标识MemberRef元数据表中(表0A)的记录项3。CLR检查该MemberRef记录项，发现字段引用了TypeRef被引导至一个AssemblyRef记录项：“mscorlib，Version=4.0.0.00,Culture=neutral,PublicKeyToken=b7a5c561934e089”。这时CLR就知道了它需要的是哪个程序集。CLR必须定位并加载该程序集。

解析引用类型时，CLR可能在三个地方找到类型

1. 相同文件

编译时便能发项对相同文件中的类型的访问，这称为早期绑定，类型直接从文件中加载，执行并继续。

2. 不同文件，相同程序集

运行时确保被引用的文件在当前程序元数据的FileDef表中。检查加载程序集清单文件的目录，加载被引用的文件，检查哈希值确保文件完整性。发现类型成员执行继续

3. 不同文件，不同程序集。

引用类型在其他程序集中，运行时会加载被引用程序集的清单文件，如果需要的类型不在该文件中，就继续加载包含类型的文件，发现类型的成员，执行继续。

ModuleDef，FileDef元数据表在引用文件时使用了文件名和扩展名，但AssemblyRef元数据表只使用文件名，无扩展名。和程序集绑定时，系统通过他测目录来尝试定位文件，自动附加.dll和exe。

解析类型引用由任何错误都会抛出异常。

可以向System.AppDomain的AssemblyResolve，ReflectionOnlyAssemblyResolve和TypeResolve事件注册回调方法。

在例子中的过程：

![1580642611669](/dotnet/1580642611669.png)

对于CLR，所有程序集都根据名称，版本，语言文化和公钥来标识。但是GAC根据名称版本语言文件公钥和CPU架构来标识。

CLR提供了将类型（类，结构，枚举，接口或委托）从一个程序集移动到另一个程序集的功能。

### 9 高级管理控制(配置)

XML配置文件的其他元素的作用

![1580645985419](/dotnet/1580645985419.png)

![1580645996686](/dotnet/1580645996686.png)

参数信息：

1. probing元素：查找弱命名程序集时，检查应用程序基目录下的AuxFiles和bin\subdir子目录。对于强命名程序集，CLR检查GAC或codeBase元素指定URL.只有未指定codeBase元素时，CLR才会在应用程序私有路径中检查强命名程序集。
2. 第一个dependentAssembly,assemblyIdentity和bindingRedirect元素：查找由控制公钥标记32ab4ba45e0a69a1的组织发布是语言文化为中性的SomeClassLibrary程序集1.0.0.0版本，改为定位同一个程序集的2.0.0.0版本
3. codeBase元素：尝试查找控制着公钥标记32ab4ba45e0a69a1的组织发布的，语言文化为中性的SomeClassLibrary程序集2.0.0.0版本时，尝试在以下URL发现：www.Wintellect.com/SomeClassLibrary.dll。codeBase元素也能用于弱命名程序集，程序集版本号被据略，根本就不应该再XMLcodeBase元素中些版本号。codeBase定义的URL必须指向应用程序基目录下的子目录。
4. 第二个dependentAssembly,assemblyIdentity和bindingRedirect元素：更改定位同一个程序集
5. publicsherPolicy元素：其apply的特性设为yes，CLR会在GAC中检查新的册灰姑娘续集版本，并进行程序集发布者认为有必要的任何版本号重定向操作。

发布者控制策略：当发布者创建新的程序集时，需要创建一个配置文件，这个配置文件告诉用户旧版本不用了，更新每台机器上的XML配置文件

## 小节

这样写笔记真是效率太低了，看书也太慢了，实在受不了。以后会尝试用思维导图的笔记方式，知识介绍一个大概，大部分知识点能回想起来，并讲出来就不写下来了，以后如果有遗忘就直接看书复习一下就好。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/02/dotnetbase6-netclrvia/  

