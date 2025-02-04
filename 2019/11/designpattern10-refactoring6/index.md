# 重构改善既有代码 简化函数调用


所有的数据都应该隐藏起来。

### 1、函数改名

&#43; Rename Method 
  &#43; ![1572491178470](/designpattern/1572491178470.png)
&#43; 动机
  &#43; 将复杂的处理过程分解成小函数。
&#43; 什么时候做
  &#43; 函数名称未能揭示函数的用途
&#43; 怎么做
  &#43; 修改函数名称

### 2、添加参数

&#43; Add Parameter
  &#43; ![1572491269669](/designpattern/1572491269669.png)
&#43; 动机
&#43; 什么时候做
  &#43; 某个函数需要从调用端得到更多信息
  &#43; 在添加参数外常常还有其他的选择，只要有可能，其他选择都比添加参数要好（查询），因为它们不会增加参数列的长度，过长的参数列是一个不好的味道。
&#43; 怎么做
  &#43; 为此函数添加一个对象参数，让该对象带进函数所需信息。

### 3、移除参数

&#43; Remove Parameter
  &#43; ![1572491400021](/designpattern/1572491400021.png)
&#43; 动机
  &#43; 可能经常添加参数却很少去除参数，因为多余的参数不会引起任何问题，相反以后可能还会用到它。请去除这些想法。
&#43; 什么时候做
  &#43; 函数本体不需要某个参数
&#43; 怎么做
  &#43; 将该参数去除。

### 4、将查询函数和修改函数分离

&#43; Separate Query from Modifier
  &#43; ![1572491591701](/designpattern/1572491591701.png)
&#43; 动机
  &#43; 在多线程系统中，查询和修改函数应该被声明为synchronized(已同步化)
&#43; 什么时候做
  &#43; 某个函数既返回对象状态值，又修改对象状态
  &#43; 任何有返回值的函数，都不应该又看得到的副作用。
  &#43; 常见的优化是将某个**查询结果放到某个字段或集合中**，后面如何查询，总是获得相同的结果。
&#43; 怎么做
  &#43; 建立两个不同的函数，其中一个负责查询，另一个负责修改。

### 5、令函数携带参数

&#43; Parameterize
  &#43; ![1572567550304](/designpattern/1572567550304.png)
&#43; 动机
  &#43; 去除重复代码
&#43; 什么时候做
  &#43; 若干函数做了类似的工作，但在函数本体中却饱含了不同的值
&#43; 怎么做
  &#43; 建立单一函数，以参数表达那些不同的值

### 6、以明确函数取代参数

&#43; Replace Parameter with Explicit Methods
  &#43; ![1572567656198](/designpattern/1572567656198.png)
&#43; 动机
  &#43; 避免出现条件表达式，接口更清楚，编译期间就可以检查，
  &#43; 如果在同一个函数中，参数是否合法还需要考虑
  &#43; 但是参数值不会对函数的行为有太多影响的话就不应该使用本项重构，如果需要条件判断的行为，可以考虑使用多态。
&#43; 什么时候做
  &#43; 有一个函数，其中完全取决于参数值不同而采取不同行为
&#43; 怎么做
  &#43; 针对该参数的每一个可能值，建立一个独立函数

### 7、保持对象完整

&#43; Preserve While Object
  &#43; ![1572568707669](/designpattern/1572568707669.png)
&#43; 动机
  &#43; 不适用完整对象会造成重复代码
  &#43; 事物都是有两面性，如果你传的是数值，被调用函数就只依赖于这些数值，如果传的是对象，就要依赖于整个对象。如果依赖对象会造成结构恶化。那么就不应该使用保持对象完整。
  &#43; 如果这个函数使用了另一个对象的多项数据，这可能以为着这个函数实际上应该定义在那些数据所属的对象上，应该考虑移动方法。
&#43; 什么时候做
  &#43; 从某个对象中取出若干值，将它们作为某一次函数调用时的参数
&#43; 怎么做
  &#43; 改为传递整个对象

### 8、以函数取代参数

&#43; Replace Parameter with Methods
  &#43; ![1572569273989](/designpattern/1572569273989.png)
&#43; 动机
  &#43; 尽可能缩减参数长度
&#43; 什么时候做
  &#43; 对象调用某个函数，并将所有结果作为参数传递给另一个函数，而接受该参数的函数本省也能够调用前一个函数。
&#43; 怎么做
  &#43; 让参数接受者去除该项参数，并直接调用前一个函数。

### 9、引入参数对象

&#43; Introduce Parameter Object
  &#43; ![1572569919231](/designpattern/1572569919231.png)
&#43; 动机
  &#43; 特定的一组参数总是一起被传递，可能有好几个函数都使用这一组参数，这些函数可能隶属于同一个类，也可能隶属于不同的类。这样的参数就是所谓的**数据泥团**，可以运用一个对象包装所有的这些数据，再以该对象取代它们。
&#43; 什么时候做
  &#43; 某些参数总是很自然地同时出现
&#43; 怎么做
  &#43; 以一个对象取代这些参数

### 10、移除设值函数

&#43; Remove Setting Method
&#43; 动机
  &#43; 使用了设值函数就暗示了这个字段值可以被改变。
&#43; 什么时候做
  &#43; 类中某个字段应该在对象创建时被设值，然后就不再改变。
&#43; 怎么做
  &#43; 去掉该字段的所有设值函数。

### 11、隐藏函数

&#43; Hide Method
  &#43; ![1572570692493](/designpattern/1572570692493.png)
&#43; 动机
  &#43; 面对一个过于丰富、提供了过多行为的接口时，就值得将非必要的取值函数和设置函数隐藏起来
&#43; 什么时候做
  &#43; 有一个函数，从来没有被其他任何类用到
&#43; 怎么做
  &#43; 将这个函数修改为private

### 12、以工厂函数取代构造函数

&#43; Replace Constructor with Factory Method
  &#43; ![1572570846559](/designpattern/1572570846559.png)
&#43; 动机
  &#43; 使用以工厂函数取代构造函数最显而易见的动机就是在**派生子类的过程中以工厂函数取代类型码**。
  &#43; 工厂函数也是将值替换成引用的方法。
&#43; 什么时候做
  &#43; 希望在创建对象时不仅仅是做简单的构建动作
&#43; 怎么做
  &#43; 将构造函数替换为工厂函数
  &#43; 使用工厂模式就使得超类必须知晓子类，如果想避免这个可以用操盘手模式，为工厂类提供一个会话层，提供对工厂类的集合对工厂类进行控制。

### 13、封装向下转型

&#43; Encapsulate Downcast
  &#43; ![1572572975294](/designpattern/1572572975294.png)
&#43; 动机
  &#43; 能不向下转型就不要向下转型，但如果需要向下转型就必须在该函数中向下转型。
&#43; 什么时候做
  &#43; 某个函数返回对象，需要由函数调用者执行 向下转型
&#43; 怎么做
  &#43; 将向下转型动作移到函数中

### 14、以异常取代错误码

&#43; Replace Error Code with Exception
  &#43; ![1572573309472](/designpattern/1572573309472.png)
&#43; 动机
  &#43; 代码可以理解应该是我们虔诚最求的目标。
&#43; 什么时候做
  &#43; 某个函数返回一个特定的代码，用以表示某种错误情况
&#43; 怎么做
  &#43; 改用异常
  &#43; 决定应该抛出受控(checked)异常还是非受控(unchecked)异常
    &#43; 如果调用者有责任在调用前检查必要状态，就抛出非受控异常
    &#43; 如果想抛出受控异常，可以新建一个异常类，也可以使用现有的异常类。
  &#43; 找到该函数的所有调用者，对它们进行相应调整。
    &#43; 如果函数抛出非受控异常，那么就调整调用者，使其在调用函数前做适当检查，
    &#43; 如果函数抛出受控异常，那么就调整调用者，使其在try区段中调用该函数。

### 15、以测试取代异常

&#43; Replace Exception with Test
  &#43; ![1572576674845](/designpattern/1572576674845.png)
&#43; 动机
  &#43; 在异常被滥用的时候
&#43; 什么时候做
  &#43; 面对一个调用者可以预先检查的体哦阿健，你抛出一个异常
&#43; 怎么做
  &#43; 修改调用者，使它在调用函数之前先做检查


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/11/designpattern10-refactoring6/  

