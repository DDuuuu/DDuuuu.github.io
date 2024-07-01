# 自学 Javascript


## 什么是JAVASCRIPT

&#43; HTML只是描述网页长相的标记语言，没有计算、判断能力，如果所有计算、判断（比如判断文本框是否为空、判断两次密码是否输入一致）都放到服务器端执行的话网页的话页面会非常慢、用起来也很难用，对服务器的压力也很大，因此要求能在浏览器中执行一些简单的运算、判断。JavaScript就是一种在浏览器端执行的语言。HTML内容，css衣服，修饰，js控制
&#43; JavaScript的Java没直接的关系，唯一的关系就是JavaScript原名LiveScript，后来吸收了Java的一些特性，升级为JavaScript。JavaScript有时被简称为JS。
&#43; JavaScript是**解释型语言**，无需编译就可以随时运行，这样哪怕语法有错误，没有语法错误的部分还是能正确运行。

## JAVASCRIPT开发环境

&#43; VS中JavaScript、JQuery的自动完成功能：在VS2010中直接有，VS2008需要安装VisualStudio 2008SP1（[http://www.microsoft.com/downloads/details.aspx?displaylang=zh-cn&amp;familyid=27673c47-b3b5-4c67-bd99-84e525b5ce61](http://www.microsoft.com/downloads/details.aspx?displaylang=zh-cn&amp;familyid=27673c47-b3b5-4c67-bd99-84e525b5ce61)）和VS90SP1-KB958502-x86（[http://code.msdn.microsoft.com/KB958502/Release/ProjectReleases.aspx?ReleaseId=1736](http://code.msdn.microsoft.com/KB958502/Release/ProjectReleases.aspx?ReleaseId=1736)）补丁会更强更好用。如果实在“.” 不出来也没关系，不影响运行。注意：先安装2008SP1，再安装VS90SP1-KB958502-x86。
&#43; JS是非常灵活的动态语言，不像C#等静态语言那样严谨，开发工具中的JS完成功能只是一个辅助、建议，“.”出来的成员调用可能不能用，“.”不出来的成员也许也能调用，因此不要因为“点儿不出来”而担心代码有问题。
&#43; VS2008的HTML编辑器中触发JavaScript自动完成：Ctrl&#43;J。

## 入门

```hTML
&lt;script type=&#34;text/javascript&#34;&gt;
alert(new Date().toLocaleDateString());
&lt;/script&gt;
```

`&lt;script language=&#34;....&gt;`已经不推荐使用。

&#43; JavaScript代码放到`&lt;script&gt;`标签中，script可以放到`&lt;head&gt;`、`&lt;body&gt;`等任意位置，而且可以有不止一个`&lt;script&gt;`标签。alert函数是弹出消息窗口，new Date()是创建一个Date类的对象，默认值就是当前时间。 **JS是大小写敏感的。**

&#43; 放到`&lt;head&gt;中的&lt;script&gt;`在body加载之前就已经运行了。写在body中的`&lt;script&gt;`是随着页面的加载而一个个执行的。

&#43; 除了可以在页面中声明JavaScript以外，还可以将JavaScript写到**单独的js文件中，然后在页面中引入**：`&lt;script src=&#34;test.js&#34; type=&#34;text/javascript&#34;&gt;&lt;/script&gt;`。声明到单独的js文件的好处是多页面也可以共享、减小网络流量。js文件的CDN(*)内容分发布网络(CDN),将别的服务器上的库

&#43; 注意：不要写成`&lt;script src=&#34;test.js&#34; type=&#34;text/javascript&#34;/&gt;`否则会有问题，这是一个比较特殊的地方。

### 事件

&#43; 在超链接的点击里执行JavaScript：`&lt;a href=&#34;javascript:alert(88)&#34;&gt;发发&lt;/a&gt;`
&#43; JavaScript中也有事件的概念，当按钮被点击的时候也可以执行JavaScript：
  &#43; `&lt;input type=&#34;button&#34; onclick=&#34;alert(99)&#34; value=&#34;久久&#34;/&gt;`
  &#43; 只有超链接的href中的JavaScript中才需要加“&#34;javascript:”，因为它不是事件，而是把“&#34;javascript:”看成像“http:”、“ftp:”、“thunder://”、“ed2k://”、&#34;mailto:&#34;**一样的网络协议**，交由js解析引擎处理。**只有href中**这是这是一个特例。

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
            var sum = 0;
            for(int i = 0;i&lt;=100;i&#43;&#43;){
                sum &#43;= i;
            }
            alert(sum);
        function test() {
            var sum = 0;
            for(int i = 0;i&lt;=100;i&#43;&#43;){
                sum &#43;= i;
            }
            alert(sum);
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;a href=&#34;javascript:test()&#34;&gt;Click me&lt;/a&gt;
    &lt;a href=&#34;win.htm&#34; onclick=&#34;alert(&#39;123&#39;)&#34;&gt;Click me&lt;/a&gt;
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; ondblclick=&#34;test()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

### 变量

&#43; JavaScript中即可以使用双引号声明字符串，也可以使用单引号声明字符串。主要是为了方便和html集成，避免转义符的麻烦。**只有一种类型var**
&#43; JavaScript中有null、undefined两种，null表示变量的值为空，undefined则表示变量还没有指向任何的对象，未初始化。两者的区别参考资料。
&#43; JavaScript是**弱类型**，声明变量的时候无法：int i=0；只能通过var i=0;声明变量，和C#中的var不一样，不是C#中那样的类型推断。
&#43; JavaScript中也可以不用var声明变量，直接用，这样的变量是“全局变量”，因此除非确实想用全局变量，否则使用的时候最好加上var。
&#43; JS是动态类型的，因此var i=0;i=&#34;abc&#34;;是合法的。并且可以把方法放到var中，

```js
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        var x = &#34;abc&#34;;
        if (!null) {
            alert(&#34;null&#34;);
        }
        if (!undefined) {
            alert(&#34;undefined&#34;);
        }
        if (x == null) {
            alert(&#34;null&#34;);
        }
        if (typeof(x) == &#34;undefined&#34;) {
            alert(&#34;undefined&#34;);
        }
        if (!x) {
            alert(&#34;未初始化   0&#34;);
        }
        if (x) {
            alert(x);
        } 
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;/body&gt;
&lt;/html&gt;
```

### 除错和调试

&#43; 如果JavaScript中的代码有语法错误，**浏览器会弹出报错信息，查看报错信息就能帮助排查错误**。
&#43; JavaScript的调试，使用VS可以很方便的进行JavaScript的调试，调试时需要注意几点：
&#43; IE6的调试选项要打开，Internet选项→高级，去掉“禁用脚本调试”前的勾选。
&#43; 以调试方式运行网页。
&#43; 设置断点、监视变量等操作和C#一样。
  案例：用循环语句的方法计算1到100之间整数的和

### 判断变量初始化

JavaScript中判断变量、参数是否初始化 的三种方法：

```js
var x;
if (x == null) {
alert(&#34;null&#34;);
}
if (typeof (x) == &#34;undefined&#34;) {
alert(&#39;undefined&#39;);
}
if (!x) {alert(&#39;不x&#39;);}
if(x){}//变量被初始化了或者变量不为空或者变量不为0.
```

推荐用最后一种方法。

### 函数声明

&#43; JavaScript中声明函数的方式：

```js
function add(i1, i2) {
return i1 &#43; i2;
}
```

int add(int i1,int i2)//C#写法

&#43; 不需要声明返回值类型、参数类型。函数定义以function开头。

```js
var r = add(1, 2);
alert(r);
r = add(&#34;你好&#34;, &#34;tom&#34;);
alert(r);
```

&#43; JavaScript中不像C#中那样要求所有路径都有返回值，没有返回值就是undefined。
&#43; 易错：自定义函数名不要和js内置、dom内置方法重名，比如selectall、focus等函数名不要用。

### 匿名函数

```js
var f1 = function(i1, i2) {
return i1 &#43; i2;
}
alert(f1(1,2));
```

&#43; 类似于C#中的匿名函数。
&#43; 这种匿名函数的用法在JQuery中的非常多
&#43; `alert(function(i1, i2) { return i1 &#43; i2; }(10,10));`//直接声明一个匿名函数，立即使用。用匿名函数省得定义一个用一次就不用的函数，而且免了命名冲突的问题，js中没有命名空间的概念，因此很容易函数名字冲突。通过例子发现一旦命名冲突以最后声明的为准
&#43; 必须`&lt;script src=&#34;my1.js&#34; type=&#34;text/javascript&#34;&gt;&lt;/script&gt;`不能：`&lt;script src=&#34;my1.js&#34; type=&#34;text/javascript&#34;/&gt;`

### 面向对象基础

JavaScript中没有类的语法，是用**函数闭包**（closure）模拟出来的，下面讲解的时候还是用C#中的类、构造函数的概念，JavaScript中String、Date等“类”都被叫做“对象”，挺怪，方便初学者理解，不严谨。JavaScript中声明类（类不是类，是对象）：

```js
function Person(name,age) {
this.name = name;
this.age =age;
this.sayHello=function(){
  alert(&#34;你好，我是&#34;&#43;this.name&#43;&#34;，我&#34;&#43;this.age&#43;&#34;岁了&#34;);
}
} 
var p1 = new Person(&#34;tom&#34;,20);
p1.sayHello();
```

&#43; 必须要声明类名，function Person(name,age)可以看做是声明构造函数，Name、Age这些属性也是使用者动态添加了。var p1 = new Person(&#34;tom&#34;, 30);//**不要丢了new，否则就变成调用函数了**，p1为undefined。new 相当于创建了函数的一个实例

### string 对象

&#43; length属性；
&#43; charAt方法；取第几个字符
&#43; indexOf    lastIndexOf
&#43; Substr(start,length)、substring(start,end)
&#43; split
&#43; match、replace(只会替换一个，替换多个要用正则表达式)、search方法，**正则表达式**相关

```js
var str = &#34;我爱北京天安门,北京天安门爱我&#34;;
var reg = /我/g;
alert(str.replace(&#34;我&#34;, &#34;你&#34;)); //replace()当第一个参数是字符串，只替换源字符串中的第一个匹配到的字符
```

如果是reg，就可以全部替换成功`alert(str.replace(reg, &#34;你&#34;)); `是正则表达式，是一个object

### ARRAY对象

JavaScript中的Array对象就是数组，首先是一个动态数组，而且是一个像C#中数组、ArrayList、Hashtable等的超强综合体。

```js
var names = new Array();
names[0] = &#34;tom&#34;;
names[1] = &#34;jerry&#34;;
names[2] = &#34;lily&#34;;
for (var i = 0; i &lt; names.length; i&#43;&#43;) {
alert(names[i]);
}
```

无需预先制定大小，动态。

### dictionary

JS中的Array是一个宝贝，不仅是一个数组，还是一个Dictionary，还是一个Stack。

```js
 var pinyins = new Array();
pinyins[&#34;人&#34;] = &#34;ren&#34;;
pinyins[&#34;口&#34;] = &#34;kou&#34;;
pinyins[&#34;手&#34;] = &#34;shou&#34;;
alert(pinyins[&#34;人&#34;]);
alert(pinyins.人);
```

像Hashtable、Dictionary那样用，而且像它们一样效率高。

### array简化

Array还可以有简化的创建方式
`var arr = [3, 5, 6, 8, 9];` 普通数组初始化
这种数组可以看做是pinyins[&#34;人&#34;] = &#34;ren&#34;;的特例，也就是key为0、1、2……
字典风格的简化创建方式：
`var arr = {&#34;tom&#34;:30,&#34;jim&#34;:20};`

### 数组，for其他

&#43; 对于数组风格的Array来说，可以使用join方法拼接为字符串
  `var arr = [&#34;tom&#34;,&#34;jim&#34;,&#34;lily&#34;];
  alert(arr.join(&#34;,&#34;));`//JS中join是array的方法，不像.Net中是string的方法
&#43; for循环可以像C#中的foreach一样用
&#43; for循环还可以获得一个对象所有的成员，类似于.Net中的反射

```js
for (var e in document) {
alert(e);
}
```


有了它没有文档也可以进行开发。

```js
var p1 = new Object();//创建一个Object对象，动态增加属性、方法s
p1.Name = &#34;tom&#34;;
p1.Age = 30;
p1.SayHello = function() { alert(&#34;hello&#34;); };
p1.SayHello();
for(var e in p1) {//对象的成员都是对象的key
alert(e);
}
```

### 扩展方法

通过类对象的**prototype**设置扩展方法，下面为String对象增加quote（两边加字符）方法

```js
String.prototype.quote = function(quotestr) {
if (!quotestr) {
quotestr = &#34;\&#34;&#34;;
}
return quotestr &#43; this &#43; quotestr;
};
alert(&#34;abc&#34;.quote());alert(&#34;abc&#34;.quote(&#34;|&#34;));
```

扩展方法的声明要在使用扩展方法之前执行。JS的函数没有专门的函数默认值的语法，但是可以不给参数传值，不传值的参数值就是undefined，自己做判断来给默认值。

&#43; 一门新的语言学：数据类型，程序接口，类库


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/03/web3-javascript/  

