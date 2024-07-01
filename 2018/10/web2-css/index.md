# 自学 CSS


CSS（层叠样式表）是用来美化页面用的，可以对页面元素进行更精细的设置，样式主要描述元素的字体颜色、背景颜色、边框等。

CSS主要有元素内联、页面嵌入和外部引用三种使用方式。CSS是描述元素的皮肤！

&#43; 元素内联，直接将样式写入元素的style属性中，`&lt;input type=&#34;text&#34; readonly=&#34;readonly&#34; style=&#34;background-color: #FF00FF&#34; /&gt;`，**style=&#34;color:Red;background-color: #FF00FF&#34;**为元素内联，适用于样式没有可复用性的场合。 

&#43; 页面嵌入：在head中加入

```HTML
&lt;style type=&#34;text/css&#34;&gt;
input{border-color:Yellow;color:Red;}
&lt;/style&gt;
```

表示页面中所有input都是采用指定的样式。适合于样式复用，减小页面体积

```HTML
&lt;style type=&#34;text/css&#34;&gt;
	P
	{
	color:Red;
	font-weight:bold;
	}
&lt;/style&gt;
```

所有的P标签都变成红色字体，加粗。

&#43; 外部引用，将css内容写入css后缀的文件
  div{background:yellow}
  然后在页面中引用，在head中加入
    `&lt;link type=&#34;text/css&#34; rel=&#34;Stylesheet&#34; href=&#34;s1.css&#34; /&gt;` 
  适合于多个页面共享css。

&#43; 更变原则：就近原则

## 常见样式

&#43; css计量单位：css中表示宽度、距离时有多种计量单位：px（像素）、30%（百分比）、em（相对单位）等。width:20px。
&#43; background-color:Red;背景颜色；color：文本颜色
&#43; 复合样式 background border
&#43; border-style:solid;边框风格，实线solid（默认是没有），还有dotted(点)等值；border-color：边框颜色；border-width：边框宽度(默认是0)。例子：`style=&#34;border-color:Red;border-width:1px;border-style:dotted;&#34;`
&#43; display：元素是否显示，可选值none（不显示,不占地儿）、block （显示为块级元素，此元素前后会带有换行符。）、inline（显示为内联元素，元素前后没有换行符 ）等。
&#43; cursor，鼠标在元素上时显示的光标图标，可选值：cursor（默认光标）、pointer（超链接上的手）、text（输入Bean）、wait（忙沙漏）、help（帮助）等。还可以通过cursor:url(dinosau2.ani)使用ani、cur格式的自定义光标图片。
&#43; LI不显示圆点：LIST-STYLE-TYPE: none;一般设在li或者ul上
&#43; 应用：图片：不显示边框，见备注
  图片：不显示边框

```HTML
IMG {
	border:0px;BORDER-BOTTOM: medium none;
	BORDER-LEFT: medium none; 
	BORDER-TOP: medium none; 
	BORDER-RIGHT: medium none
}
```

### 例子 外部引用

```HTML
*
{
	/* 所有的元素 */
	margin:0;
	padding:0;
	color:Red;
}
body
{
	background:red url(/images/back_image.GIF);
}


span
{
	/* block 块 
	display:block;*/
	cursor:pointer;
	color:Blue;
	text-decoration:underline;
}
input
{
	color:Green;
}

li
{
	
	/* 去掉ul前面的黑点 */
	list-style-type:none;
}
```

### 样式选择器

对于非元素内联的样式需要定义样式选择器，通俗的说就是这个样式适合于哪些元素，三种：**标签选择器**、**class选择器**和**id选择器**。

&#43; 标签选择器 input{border-color:Yellow;color:Red;}，对于指定的标签采用统一的样式

&#43; class选择器，以定义一个命名的样式，然后在用到它的时候设定元素的class属性为样式的名称，还可以同时设定多个class，名称之间加空格

```css   
.d1
{
	color:Red;
	width:100px;
}
.d2
{
	color:Blue;
	width:200px;
}
.d3
{
	color:Green;
	width:300px;
}
&lt;div class=&#34;d1&#34;&gt;
	123123123
&lt;/div&gt;
&lt;div class=&#34;d2&#34;&gt;
	abc
&lt;/div&gt;
&lt;div class=&#34;d3&#34;&gt;
	啊打发
&lt;/div&gt;
```


样式名称开头加“.”

```css
.warning{background:Yellow;}
.highlight{font-size:xx-large;cursor:help;}
&lt;table&gt;
&lt;tr&gt;
&lt;td class=&#34;highlight&#34;&gt;aaa&lt;/td&gt;
&lt;td class=&#34;warning&#34;&gt;bb&lt;/td&gt;
&lt;td class=&#34;highlight warning&#34;&gt;ccc&lt;/td&gt;
&lt;/tr&gt;
&lt;/table&gt;
```

&#43; 标签&#43;类选择器:通过指定标签来确定不同的类适用范围

```css
label.name
{
	background-color:Gray;
}
input.name
{
	color:Blue;
}
&lt;label for=&#34;name&#34; class=&#34;name&#34;&gt;用户名：&lt;/label&gt;
&lt;input class=&#34;name&#34; id=&#34;name&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;
```

&#43; id选择器：与类选择器不同的地方就是.改成了#。

```css
#wrap
{
	border:solid 1px blue;
	width:300px;
	height:300px;
}
&lt;div id=&#34;wrap&#34;&gt;
main
&lt;/div&gt;
```

&#43; 包含选择器：设置div中strong标签的格式，div中所有strong

```css   
div  strong
{
	color:Red;
}
&lt;strong&gt;测试&lt;/strong&gt;
&lt;div&gt;
	&lt;strong&gt;这是左边栏&lt;/strong&gt;
	left
&lt;/div&gt;
```

&#43; 后代选择器：直接属于div的strong才会属于这个样式。

```css
div strong
{
	color:Red;
}
&lt;div&gt;
	&lt;strong&gt;这是左边栏&lt;/strong&gt;
	left
&lt;/div&gt;
```

&#43; 组合选择器 ：多个标签一个样式，组合选择器可以使用类选择器.h3,.p,.span {}

```css
h3,p,span 
{
	color:Yellow;
}
&lt;h3&gt;标题3&lt;/h3&gt;
&lt;p&gt;这是段落&lt;/p&gt;
&lt;span&gt;这是span&lt;/span&gt;
```

&#43; 伪选择器：超链接使用
  &#43; A:visited：超链接点击过的样式；A:visited {TEXT-DECORATION: none}下划线
  &#43; A:active：选中超链接时的样式；A:active {TEXT-DECORATION: none}
  &#43; A:link：超链接未被访问时的状态；A:link {TEXT-DECORATION: none}
  &#43; A:hover：鼠标移到超链接时的状态。A:hover {TEXT-DECORATION: underline}

```css	
a:visited
{
	color:Gray;
}
a:link
{
	color:Red;
	
}
a:hover
{
	color:Black;
	font-style:italic;
}
a:active
{
	color:Yellow;
}
&lt;a href=&#34;http://www.itcast.cn&#34;&gt;传智播客&lt;/a&gt;
&lt;a href=&#34;http://www.csdn.com&#34;&gt;csdn&lt;/a&gt;
&lt;a href=&#34;http://www.cnbeta.com&#34;&gt;cnbeta&lt;/a&gt;
&lt;a href=&#34;http://www.123.com&#34;&gt;123&lt;/a&gt;
```

## 布局

布局的分类：表格布局，框架布局，div&#43;css布局


&#43; 表格布局：表格套表格  ，代码多，table显示很慢，一块块的显示就比较麻烦，显示圆角就比较麻烦
&#43; 框架布局：多个页面来显示：
  &#43; Frameset  框架页里不能有body

```css
&lt;head&gt;
&lt;title&gt;&lt;/title&gt;
&lt;/head&gt;
 
&lt;frameset rows=&#34;30%,*&#34;&gt;
  &lt;frame name=&#34;top&#34; src=&#34;top.htm&#34; noresize=&#34;noresize&#34;/&gt;
  &lt;frameset cols=&#34;20%,*&#34;&gt;
    &lt;frame name=&#34;left&#34; src=&#34;left.htm&#34; noresize=&#34;noresize&#34;/&gt;
    &lt;frame name=&#34;main&#34; src=&#34;main.htm&#34; noresize=&#34;noresize&#34;/&gt;
  &lt;/frameset&gt;
&lt;/frameset&gt;

left.html 
&lt;body&gt;
&lt;ul&gt;
&lt;li&gt;&lt;a href=&#34;1-注册页面.htm&#34; target=&#34;main&#34;&gt;注册&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href=&#34;5-选择器.htm&#34; target=&#34;if&#34;&gt;登陆&lt;/a&gt;&lt;/li&gt;
&lt;/ul&gt;
&lt;/body&gt;
```

&#43; &#43; iframe 嵌入页面

```css
&lt;iframe src=&#34;iframe.htm&#34; name=&#34;0&#34; width=&#34;0&#34; height=&#34;0&#34;&gt;&lt;/iframe&gt;
&lt;body&gt;
adsfasdf
asdf
&lt;iframe src=&#34;1-注册页面.htm&#34; width=&#34;0&#34; height=&#34;0&#34;&gt;&lt;/iframe&gt;
adsf
&lt;/body&gt;
```

main.html

iframe：在一个页面中嵌入一个页面

```css 
&lt;body&gt;
&lt;iframe src=&#34;1-注册页面.htm&#34; width=&#34;500px&#34; height=&#34;200px&#34; name=&#34;if&#34;&gt;&lt;/iframe&gt;
adsf
&lt;/body&gt;
```



&#43; div&#43;css布局： 

  ![image-20240701142029154](/web/image-20240701142029154.png)

&#43; 网页布局就是“这块内容显示在左边，那两块内容并排显示，那块内容漂浮在页面上”。

&#43; 不要使用`&lt;table&gt;`进行布局，因为：table可能会在所有tr、td加载完成以后才显示，所以加载完成之前界面是一片空白；用table布局会将布局方式写在html中，违反了“语义性”原则；用table会影响搜索引擎的抓取，不利于SEO。因此Table用来表达真是表格状数据的东西，布局用Div(层)&#43;Css来做,Div用来圈定元素，CSS用来定义元素的位置。

&#43; Div&#43;CSS就是将要布局的内容用`&lt;div&gt;`切成块，然后使用css描述每个块的大小、位置等。

&#43; 布局最重要的一个属性就是float，  

```css
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
&lt;title&gt;&lt;/title&gt;
&lt;link href=&#34;css/style.css&#34; rel=&#34;stylesheet&#34; type=&#34;text/css&#34; /&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;div id=&#34;wrap&#34;&gt;
&lt;div id=&#34;head&#34;&gt;
&lt;div id=&#34;logo&#34;&gt;
&lt;img src=&#34;images/back_image.GIF&#34; width=&#34;100px&#34; height=&#34;50px&#34; /&gt;&lt;/div&gt;
&lt;div id=&#34;menu&#34;&gt;
&lt;ul&gt;
&lt;li&gt;首页&lt;/li&gt;
&lt;li&gt;播客&lt;/li&gt;
&lt;li&gt;相册&lt;/li&gt;
&lt;li&gt;关于&lt;/li&gt;
&lt;/ul&gt;
&lt;/div&gt;
&lt;/div&gt;
&lt;div id=&#34;body&#34;&gt;
&lt;div id=&#34;nav&#34;&gt;
&lt;ul&gt;
&lt;li&gt;好好学习&lt;/li&gt;
&lt;li&gt;天天向上&lt;/li&gt;
&lt;li&gt;不要睡觉&lt;/li&gt;
&lt;/ul&gt;

&lt;/div&gt;
&lt;div id=&#34;content&#34;&gt;内容&lt;/div&gt;
&lt;/div&gt;
&lt;div id=&#34;footer&#34;&gt;版权&lt;/div&gt;
&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;

*
{
	margin:0px;
	padding:0px;
}
body
{
	background-color:Gray;
	
}
#wrap
{
	width:98%;
	height:500px;
	margin:0px auto;
}

#head
{
	height:150px;
	background-color:Red;
}
#head #menu
{
	margin:80px auto 0px auto;
	padding-left:200px;
}
#head #menu ul
{
	width:400px;
}
#head #menu li
{
	float:left;
	width:100px;
	list-style-type:none;
}
#body
{
	height:800px;
	background-color:White;
}
#body #nav
{
	/* 强制英文换行 word-break:break-all; */
	/* 溢出后显示滚动条 */
	overflow:auto;
	background-color:Blue;
	width:200px;
	float:left;
}
#body #nav ul
{
	padding-top:100px;
}
#body #nav li
{
	list-style-type:none;
	height:30px;
	padding-left:30px;
	
}
#body #content
{
	background-color:Green;
}

```



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/10/web2-css/  

