# 重构改善既有代码 简化条件表达式


### 1、分解条件表达式

&#43; Decompose Conditional
  &#43; ![1572486513216](/designpattern/1572486513216.png)
&#43; 动机
  &#43; 复杂的条件逻辑是最常导致复杂度上升的地点之一，
&#43; 什么时候做
  &#43; 有一个复杂的条件语句
&#43; 怎么做
  &#43; 从if，then,else三个段落中分别提炼出独立函数
  &#43; 将其分解为多个独立函数，根据每个小块代码的用途分解而得的新函数命名。
  &#43; 很多人都不愿意去提炼分支条件，因为这些条件非常短，但是提炼之后函数的可读性很强，就像一段注释一样清楚明白。

### 2、合并条件表达式

&#43; Consolidate Conditional Expression
  &#43; 其实就是用一个小型函数封装一下，小型函数的名字可以作为注释。
  &#43; ![1572486838506](/designpattern/1572486838506.png)
&#43; 动机
  &#43; 合并后的条件代码会使得检查的用意更加清晰，合并前和合并后的代码有着相同的效果。
&#43; 什么时候做
  &#43; 有一系列条件测试，都得到相同结果
&#43; 怎么做
  &#43; 将这些测试合并为一个条件表达式，并将这个条件表达式提炼成为一个独立函数。

### 3、合并重复的条件片段

&#43; Consolidate Duplicate Conditional Fragments
  &#43; ![1572487145359](/designpattern/1572487145359.png)
&#43; 动机
&#43; 什么时候做
  &#43; 在条件表达式的每个分支上都有着相同的一段代码
&#43; 怎么做
  &#43; 将这段重复代码搬移到条件表达式之外。

### 4、移除控制标记

&#43; Remove Control Flag
&#43; 动机
  &#43; 单一出口原则会迫使让妈中加入讨厌的控制标记，大大降低条件表达式的可读性，
&#43; 什么时候做
  &#43; 在一系列布尔表达式中，某个变量带有&#34;控制标记&#34;(control flag)的作用
&#43; 怎么做
  &#43; 以**break语句或return语句**取代控制标记

### 5、以卫语句取代嵌套条件表达式

&#43; Replace Nested Conditional with Guard Clauses
  &#43; ![1572487743463](/designpattern/1572487743463.png)
&#43; 动机
  &#43; 单一出口的规则其实并不是那么有用，保持代码清晰才是最关键的。
&#43; 什么时候做
  &#43; 函数中条件逻辑使人难以看清正常的执行路径
&#43; 怎么做
  &#43; 使用卫语句表现所有特殊情况。

### 6、以多态取代条件表达式

&#43; Replace Conditional with Polymorphism
  &#43; ![1572488390812](/designpattern/1572488390812.png)
&#43; 动机
  &#43; 如果需要根据对象的不同类型而采取不同的行为，多态使你不必编写明显的条件表达式。
  &#43; 同一组条件表达在程序许多地点出现，那么使用多态的收益是最大的。
&#43; 什么时候做
  &#43; 有一个条件表达式，根据对象类型的不同而选择不同的行为
&#43; 怎么做
  &#43; 将这个体哦阿健表示式的每个分支放进一个子类的覆写函数中，然后将原始函数声明为抽象函数。

### 7、引入Null对象

&#43; Introduce Null Object
  &#43; ![1572488884565](/designpattern/1572488884565.png)
&#43; 动机
  &#43; 多态的最根本好处就是不必要想对象询问你是什么类型而后根据得到的答案调用对象的某个行为，只管调用该行为就是了。
  &#43; 空对象一定是常量，它们的任何成分都不会发生变化，因此可以使用单例模式来实现它们。
&#43; 什么时候做
  &#43; 需要再三检查对象是否为Null
&#43; 怎么做
  &#43; 将null对象替换成null对象。

### 8、引入断言

&#43; Introduce Assertion
  &#43; ![1572489734260](/designpattern/1572489734260.png)
&#43; 动机
  &#43; 断言是一个条件表达式，应该总是为真，如果它失败，表示程序员犯了一个错误。因此断言的失败应该导致一个非受控异常（unchecked exception）。
  &#43; 加入断言永远不会影响程序的行为。
  &#43; 用它来检查一定必须为真的条件。
&#43; 什么时候做
  &#43; 某一段代码需要对程序状态做出某种假设
&#43; 怎么做
  &#43; 以断言明确表现这种假设


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/11/designpattern9-refactoring5/  

