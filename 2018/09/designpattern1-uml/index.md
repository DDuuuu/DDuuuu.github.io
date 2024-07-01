# 敏捷软件开发原则模式与实践 UML


所看书籍是：敏捷软件开发_原则、模式与实践_C#版（美）马丁著，这本书写的非常棒，感谢作者。该归纳总结的过程按照我读的顺序写。

## UML

&amp;emsp;在建造桥梁，零件，自动化设备之前需要建模分析可行性，软件在编写之前也需要建立模型，看看类和逻辑的设计是否合理，这样的建模过程就是UML。

### 类图

类图就是来描述一个类本身或和其他类的调用关系。

&#43; &#43;public
&#43; -private
&#43; #protected

#### 实现/泛化

&#43; 集成
&#43; 实现接口

![](/designpattern/SoWkIImgAStDuNBDBSZ9hqnDLOZJrLK8Jin9BCfCJO49SdawbPQKvESfsDJewIb0s2wPYJcfHLmEgNafGFq0.png)


#### 组合

&#43; 部分可以离开整体

![](/designpattern/SoWkIImgAStDuN9EB59GCbHIoDVLjLDGCb5I2Cz8JSrHi598J4ylIarFBCdCp-DoICrB0Ie60000.png)

#### 聚合

&#43; 部分不能离开整体

![](/designpattern/SoWkIImgAStDuNBAB4fHK39KKj3IrRLJK39IKd3BpozHi598piyhISpCA-PoICrB0Qe40000.png)


#### 关联

&#43; 持有对其他对象引用的实例变量

![](/designpattern/SoWkIImgAStDuGh8oCzBLT3LjLDGCZHLKd0gBId9p-DoICrB0Se20000.png)

#### 依赖

&#43; 局部变量/方法的参数或则静态方法的调用

![](/designpattern/SoWkIImgAStDuGejI4aiILNGqxDJS77oICqfI2tYSaZDIm7A0G00.png)

**注意**

&#43; 关系的强弱：泛化/实现&gt;组合&gt;聚合&gt;关联&gt;依赖

### 对象图

表示系统执行的某个特定时刻的一组对象和关系，可以看成是内存快照。
该图大部分是从相应的类图中推导而来没啥用。

### 顺序图

描述算法的实现，重点在于消息的顺序。比较常绘制的动态模型。

#### 例子

![](/designpattern/Bhj5Fzg.png)

&#43; 对象下面画有横线，类没有，对象名：类
&#43; 垂下来的线为生命线
&#43; 中间矩形垂下来的矩形：激活，表示一个函数的执行时间
&#43; 虚线表示返回参与者并传回返回值
&#43; 箭头：消息。返回值：消息名称（参数）
&#43; 带圆圈的箭头：消息的参数

#### 注意

&#43; 循环：框起来 [for each id in idlist]
&#43; 容易被勿用和滥用

### 协作图

描述算法的实现，重点在于对象之间的关系

### 状态图

其实就是有限状态机（FSM）。

#### 例子:

![](/designpattern/123.png)

&#43; 实心黑球：初始伪状态，从这个状态开始运转
&#43; 圆矩形：状态。上层放状态的名字，下层放一些特定动作和事件，表示进入或则退出时要做什么，
&#43; 箭头：迁移。上面有触发该迁移的事件名称和要执行的动作

**注意**

&#43; entry和exit：标准事件，不管写不写都会触发
&#43; 超状态：几个状态迁移时间相同时，可以组成一个超状态。迁移时会出发超状态的entry和exit

## 如何使用UML

在使用UML的过程中，需要先通过行为优先的方式写出状态图，先是局部状态再是整体状态，抽象出会改变的，将每一种改变的类型实例化，中间再通过各种设计模式隔离

### 行为优先

从项目的功能入手，用户的交互入手写出每一种功能，大体的类有了后，再抽象出会改变的类，通过设计模式隔离

### 检查结构

检查每一种功能实现是否合理

### 想象代码

想想出代码的样子做微调


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/09/designpattern1-uml/  

