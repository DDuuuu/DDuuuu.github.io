# 自学 HTML


HTML(HyperText Markup Language)：描述网页长什么样子、有什么内容的一个文本。查看网页的描述内容（HTML）的方式：使用IE浏览器的话，在网页上点击右键，选择“查看源文件”。

## 主要构架

	&lt;html&gt;
		&lt;head&gt;
			&lt;title&gt;我的第一个网页&lt;/title&gt;
		&lt;/head&gt;
		&lt;body bgcolor=&#34;red&#34; background=&#34;bg.jpg&#34;&gt;
			Hello world
		&lt;/body&gt;
	&lt;/html&gt;

&#43; 所有内容都在`&lt;html&gt;&lt;/html&gt;`标签之内；
&#43; `&lt;head&gt;&lt;/head&gt;`内放的是头部信息，是对页面的描述，不会直接显示在页面中，`&lt;head&gt;`内的`&lt;title&gt;`中设置的是页面的标题，`&lt;title&gt;`只能放在`&lt;head&gt;`中；
&#43; `&lt;body&gt;`是页面的主体，大部分显示内容都定义在这里；

&#43; `&lt;meta&gt;`标签，`&lt;meta&gt;`有指定name和指定http-equiv两种用法，`&lt;meta name=&#34;名字&#34; content=&#34;值&#34; /&gt;、&lt;meta http-equiv=&#34;名字&#34; content=&#34;值&#34; /&gt;`两种用法
  &#43; `&lt;meta http-equiv=&#34;Content-Type&#34; content=&#34;text/html;charset=utf-8&#34; /&gt;`指定网页编码
  &#43; `&lt;meta http-equiv=&#34;Refresh&#34; content=&#34;3&#34; /&gt;` 三秒钟后刷新此网页
  &#43; `&lt;meta http-equiv=&#34;Refresh&#34; content=&#34;3;url=http://www.rupeng.com&#34; /&gt;` 三秒钟后重定向到新网页。发帖成功后提示“发帖成功，即将转向帖子查看页面”
  &#43; `&lt;meta http-equiv=&#34;Cache-Control&#34; content=&#34;no-cache&#34; /&gt;` 禁止浏览器缓存页面



## 基本语法

&#43; `&lt;h&gt;`：标签，`HTML`定义了`&lt;h1&gt;&lt;/h1&gt;`到`&lt;h6&gt;&lt;/h6&gt;`六个h标签，分别表示不同大小的字体。
&#43; `&lt;br&gt;`：回车
&#43; `&lt;p&gt;`：分段。`&lt;p&gt;`前后会有比较大的空白，而`&lt;br&gt;`则没有。
&#43; `&lt;center&gt;`这是居中`&lt;/center&gt;`居中显示
&#43; `&lt;b&gt;粗体&lt;/b&gt;`：粗体
&#43; `&lt;i&gt;斜体&lt;/i&gt;`：斜体
&#43; `&lt;font&gt;&lt;/font&gt;`：字体标签，`&lt;font color=&#34;red&#34;&gt;红色&lt;/font&gt;`  `&lt;font size=&#34;30&#34; color=&#34;red&#34;&gt;红色&lt;/font&gt; `
&#43; 特殊字符：`&amp;lt`;（小于号，less than）；`&amp;gt`;（大于号，greater than）；`&amp;nbsp`;（空格，no-break space）
&#43; `&lt;hr&gt;`： 横线 color  size  width  align=left,center,right
&#43; `&lt;pre&gt;`：预格式化  保持本色
&#43; `&lt;body bgcolor=&#34;#006699&#34;&gt;`背景颜色
&#43; `&lt;a href=&#34;http://www.rupeng.com&#34;&gt;如鹏网&lt;/a&gt;`超链接
  &#43; `&lt;a href=&#34;http://www.rupeng.com&#34;&gt;&lt;img src=&#34;http://www.rupeng.com/forum/templates/uchome/images/logo.gif&#34;/&gt;&lt;/a&gt;`镶嵌图片；
  &#43; “/”表示网站根目录，“../”表示父目录，“../../”表示父目录的父目录
  &#43; 将`&lt;a&gt;`的target属性设定为&#34;_blank&#34;就可以在新窗口中打开超链接。
  &#43; 锚记：用name属性为`&lt;a&gt;` 起名字：`&lt;a name=&#34;Last&#34;&gt;这里是最后&lt;/a&gt;`。这样可以通过`&lt;a href=&#34;#Last&#34;&gt;转到平台&lt;/a&gt;`来跳转到超链接的部分。去往评论、回到正文

&#43; `&lt;img src=&#34;a.jpg&#34;/&gt;`注意图片是链接的，不是插入的，所以如果Src指向的文件不存在了，就看不了了。
  &#43; alt： 图片无法显示时的显示文本，鼠标方式去也会有悬浮提示“点击查看大图”
  &#43; border属性指定边框，border=&#34;0&#34;不显示边框
  &#43; width、height属性指定图片的显示大小，如果不指定则是图片的原始大小
  &#43; 不要以为把bmp后缀改为jpg就是改文件格式了
  &#43; 如果网页上要显示小图（比如缩略图），不要仅仅是把大图设定一下width、height来缩小，因为仍然会下载大图，会使得加载速度很慢。

&#43; 列表：`&lt;ul&gt;&lt;li&gt;灌水区&lt;/li&gt;&lt;li&gt;版务区&lt;/li&gt;&lt;li&gt;原创贴图&lt;/li&gt;&lt;/ul&gt;`。无序列表。
  &#43; 有序的列表`&lt;ol&gt;&lt;/ol&gt;`

&#43; 表格：`&lt;table&gt;&lt;/table&gt;`为表格，在内部通过`&lt;tr&gt;`创建行，`&lt;tr&gt;`内部通过`&lt;td&gt;` 创建单元格。可以将table的border属性设为0来隐藏表格线。
  &#43; 填充、间距Cellpadding内容和表格边线之间的距离 cellspacing单元格之间的间距s
  &#43; `&lt;tr&gt;`的属性：align，水平对齐，可选值left、right、center；valign，垂直对齐，可选值top、middle、bottom。
  &#43; `&lt;td&gt;`也有align和valign。`&lt;tr align=&#34;right&#34;&gt;&lt;td&gt;tom&lt;/td&gt;&lt;td align=&#34;left&#34;&gt;20&lt;/td&gt;&lt;td&gt;男&lt;/td&gt;&lt;/tr&gt;`：子标签默认继承父标签的属性，如果自己单独指定了属性，则会覆盖父标签的属性。
  &#43; rowspan、colspan进行单元格的合并
  &#43; 表头的td可以用th代替，这样就会表头粗体、居中显示

&#43; 表单：`&lt;form&gt;`标签为表单标签。如果要把数据提交到服务器，则需要将`&lt;input&gt;`、`&lt;textarea&gt;`、`&lt;select&gt;`等表单元素放到form中

  &#43; `&lt;input&gt;`是主要的表单元素，type的可选值：submit（提交按钮）、button（普通按钮）、checkbox（复选框）、file（文件选择框）、hidden（隐藏字段）、image（图片按钮）、password（密码框）、radio（单选按钮）、reset（重置按钮）、text（文本框）。`&lt;input type=&#34;file&#34; /&gt;`
  &#43; submit：点击submit按钮表单就会被提交给服务器，中文IE下默认按钮文本为“提交查询”，可以设置value属性修改按钮的显示文本
  &#43; text：size属性为宽度，value为值，maxlength为可以输入的最大长度，readonly只读。`&lt;input type=&#34;text&#34; readonly/&gt;`（只写属性名，不写属性值）或者`&lt;input type=&#34;text&#34; readonly=&#34;readonly&#34; /&gt;`
  &#43; checkbox：checked属性表示是否被选中，`&lt;input type=&#34;checkbox&#34; checked /&gt;`或者`&lt;input type=&#34;checkbox&#34; checked=&#34;checked&#34; /&gt;`(推荐)checked、readonly等这种只有一个可选值的属性都可以省略属性值。
  &#43; radio：相同name属性的为一组，不同radio设定不同的value值，这样通过取指定name的值就可以知道谁被选中了，不用单独的判断。
  &#43; file：使用file，则form的enctype必须设置为multipart、form-data、form   method属性为POST（*），get。get：少量数据，post：大量数据。`&lt;form action=&#34;Default.aspx&#34; method=&#34;post&#34;&gt;&lt;/form&gt;` 
  &#43; image：使用src属性指定图片的地址，用来实现美化的“登录按钮”。
  &#43; Reset:重置

&#43; `&lt;select&gt;`:用来创建类似于WinForm中的ComboBox或者ListBox
  &#43; 如果size属性大于1就是ListBox（size的值为显示出来的列表数量），否则就是ComboBox。
  &#43; `&lt;select  multiple&gt;`或者`&lt;select  multiple=&#34;multiple&#34;&gt;`（推荐），那么就是可以多选的ListBox。
  &#43; select中的项是`&lt;option&gt;`，`&lt;option&gt;北京&lt;/option&gt;`还可以设定项的值`&lt;option value=&#34;1&#34;&gt;北京&lt;/option&gt;`。
  &#43; 将一个option设置为选中：`&lt;option selected&gt;333&lt;/option&gt;`或者`&lt;option selected=&#34;selected&#34;&gt;333&lt;/option&gt;`(推荐)就可以将这个项设定为选择项
  &#43; 如何实现“不选择”，添加一个`&lt;option value=&#34;-1&#34;&gt;--不选择--&lt;option&gt;`，然后编程判断select选中的值如果是-1就认为是不选择。
  &#43; select分组选项，可以使用optgroup对数据进行分组，分组本身不会被选择，无论对于下拉列表还是列表框都适用。备注

&#43; `&lt;textarea&gt;`多行文本（也是表单元素）：`&lt;textarea&gt;文本&lt;/textarea&gt;`，cols、rows属性表示行数和列数。
&#43; `&lt;label&gt;`：在`&lt;input type=&#34;text&#34;&gt;`前可以写普通的文本来修饰，但是单击修饰文本的时候input并不会得到焦点，而用label则可以，for属性指定要修饰的控件的id，`&lt;label for=&#34;txt1&#34; &gt;asdfad&lt;/label&gt;`
  &#43; 为被修饰的控件设置一个唯一的id
  &#43; `&lt;label for=&#34;ma&#34;&gt;婚否&lt;/label&gt; &lt;input id=&#34;ma&#34; type=&#34;checkbox&#34; /&gt;`
&#43; fieldset：GroupBox效果，将控件划分一个区域，看起来更规整
  &#43; `&lt;fieldset&gt;`
    `&lt;legend&gt;常用&lt;/legend&gt;`
    `&lt;input type=&#34;text&#34; /&gt;`
     `&lt;/fieldset&gt;`
&#43; 滚动文字` &lt;marquee&gt;`           scrolldelay控制速度
&#43; 播放声音、显示flash，

&gt;     &lt;object classid=&#34;clsid:D27CDB6E-AE6D-11cf-96B8-444553540000&#34; codebase=&#34;http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,29,0&#34; width=&#34;760&#34; height=&#34;700&#34;
&gt;     &lt;param name=&#34;movie&#34; value=&#34;light-bot-2205.swf&#34; /
&gt;     &lt;param name=&#34;quality&#34; value=&#34;high&#34; /
&gt;     &lt;embed src=&#34;light-bot-2205.swf&#34; quality=&#34;high&#34; pluginspage=&#34;http://www.macromedia.com/go/getflashplayer&#34; type=&#34;application/x-shockwave-flash&#34; width=&#34;760&#34; height=&#34;700&#34;&gt;&lt;/embed
&gt;     &lt;/object&gt;

&#43; 调用wmp的插件`&lt;embed src=&#34;coder.mp3&#34; loop=true autostart=true name=bgss width=&#34;460&#34; height=&#34;68&#34;&gt;`
  只能播放wav和mid格式，只支持ie&lt;bgsound src=&#34;town.mid&#34; loop=&#34;true&#34; /&gt;

&#43; div：层`&lt;div&gt;&lt;/div&gt;`将内容放到层中，就以将这些内容当成一个整体进行处理，比如整体隐藏、整体移动等。div非常强大和常用。类似于WinForm的Panel。

&#43; span:div是将内容放到一个矩形的区块中，会影响布局，而span只是把一段内容定义成一个整体进行操作，但不影响布局、显示。


## 备注

&#43; xml：描述存的什么数据
&#43; XHTML:可扩展超文本置标语言（eXtensible HyperText Markup Language，XHTML）
&#43; DHTML：动态HTML；
&#43; 格式标签：`&lt;p&gt;&lt;/p&gt;`创建段落；`&lt;br /&gt;`回车，也可以写成`&lt;br&gt;`，在HTML中有一些标签可以不关闭，`&lt;br&gt;`就是一个，这是和XML不同的地方（常考），但是为了遵循XHTML规范，推荐像XML一样严格关闭。`&lt;br/&gt;&lt;img src=&#34;1.gif&#34;/&gt;`

&#43; 属性值：HTML中属性值即可以用单引号括起来、也可以用双引号括起来、甚至不用引号都可以（不推荐），单双要配对。
&#43; 注释：HTML使用和XML一样的&lt;!--注释内容--&gt;来做注释。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/10/web1-html/  

