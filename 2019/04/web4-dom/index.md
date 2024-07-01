# 自学 DOM


DOM(document object medol)文档对象模型。

&#43; DOM就是HTML页面的模型，将每个标签都做为一个对象，JavaScript通过调用DOM中的属性、方法就可以对网页中的文本框、层等元素进行编程控制。比如通过操作文本框的DOM对象，就可以读取文本框中的值、设置文本框中的值。
&#43; JavaScript→Dom就是C#→.Net Framwork。没有.net，C#只能for、while，连WriteLine、MessageBox都不行。Dom就是一些让JavaScript能操作HTML页面控件的类、函数。
  DOM也像WinForm一样，通过事件、属性、方法进行编程。
  CSS&#43;JavaScript&#43;DOM=DHTML
&#43; 学习阶段只考虑IE。用IE Collection安装IE所有版本，学习使用IE6（要调试必须使用本机安装的版本）。

## 事件

&#43; 事件：`&lt;body onmousedown=&#34;alert(&#39;哈哈&#39;)&#34;&gt;`当点击鼠标的时候执行onmousedown中的代码。有时间事件响应的代码太多，就放到单独的函数中：

```HTML
&lt;script type=&#34;text/javascript&#34;&gt;
function bodymousedown() {
alert(&#34;网页被点坏了，赔吧！&#34;);
alert(&#34;逗你玩的！&#34;);
}
&lt;/script&gt;
&lt;body onmousedown=&#34;bodymousedown()&#34;&gt;
```

- bodymousedown后的括号不能丢（ onmousedown=&#34;bodymousedown&#34; ），因为表示onmousedown事件发生时调用bodymousedown函数，而不是onmousedown事件的响应函数是bodymousedown。

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function setTxt() {
            t1.value = &#34;1234&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;setTxt();&#34; onunload=&#34;alert(&#39;欢迎下次光临！&#39;)&#34; onbeforeunload=&#34;window.event.returnValue=&#39;确定关闭？&#39;&#34;&gt;
    &lt;input id=&#34;t1&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```


## 动态设置事件

可以在代码中动态设置事件响应函数，就像.Net中btn.Click&#43;=一样

```HTML
function f1() {
alert(&#34;1&#34;);
}
function f2(){
alert(&#34;2&#34;);
}
```

`&lt;input type=&#34;button&#34; onclick=&#34;document.ondblclick=f1&#34; value=&#34;关联事件1&#34; /&gt;`//注意f1不要加括号。如果加上括号就变成了执行f1函数，并且将函数的返回值复制给document.ondblclick
`&lt;input type=&#34;button&#34; onclick=&#34;document.ondblclick=f2&#34; value=&#34;关联事件2&#34; /&gt;`

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function load() {
            alert(&#39;地球日&#39;); 
            
            alert(&#39;2012不远了&#39;);
        }
        function f1() {
            alert(&#34;f1&#34;);
        }
        
       
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
   &lt;input type=&#34;button&#34; value=&#34;按钮&#34; onclick=&#34;document.onclick=f1&#34; /&gt;
   &lt;input type=&#34;button&#34; value=&#34;关闭&#34; onclick=&#34;window.close()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

## window对象

&#43; window对象代表当前浏览器窗口，使用window对象的属性、方法的时候可以省略window，比如window.alert(&#39;a&#39;)可以省略成alert(&#39;aa&#39;)。
  &#43; alert方法，弹出消息对话框
  &#43; confirm方法，显示“确定”、“取消”对话框，如果按了【确定】按钮，就返回true，否则就false

```HTML
if (confirm(&#34;是否继续？&#34;)) {
alert(&#34;确定&#34;);
}
else {
alert(&#34;取消&#34;);
}
```

	&#43; 重新导航到指定的地址：navigate(&#34;http://www.rupeng.com&#34;);
	&#43; setInterval每隔一段时间执行指定的代码，第一个参数为代码的字符串，第二个参数为间隔时间（单位毫秒），返回值为定时器的标识
		`setInterval(&#34;alert(&#39;hello&#39;)&#34;, 5000);`
	&#43; clearInterval取消setInterval的定时执行，相当于Timer中的Enabled=False。因为setInterval可以设定多个定时，所以clearInterval要指定清除那个定时器的标识，即setInterval的返回值。

`var intervalId = setInterval(&#34;alert(&#39;hello&#39;)&#34;, 5000);
clearInterval(intervalId);`
	&#43; setTimeout也是定时执行，但是不像setInterval那样是重复的定时执行，只执行一次，clearTimeout也是清除定时。很好区分：Interval：间隔；timeout：超时。
`var timeoutId = setTimeout(&#34;alert(&#39;hello&#39;)&#34;, 2000);`

	&#43; showModalDialog弹出模态对话框，注意showModalDialog必须在onClick等用户手动触发的事件中才会执行，否则可能会被最新版本的浏览器当成广告弹窗而拦截。
		&#43; 第一个参数为弹出模态窗口的页面地址。
		&#43; 在弹出的页面中调用window.close()（不能省略window.close()中的window.）关闭窗口，只有在对话框中调用window.close()才会自动关闭窗口，否则浏览器会提示用户进行确认。
	&#43; 除了有特有的属性之外，当然还有通用的HTML元素的事件：onclick（单击）、ondblclick（双击）、onkeydown（按键按下）、onkeypress（点击按键）、onkeyup（按键释放）、onmousedown（鼠标按下）、onmousemove（鼠标移动）、onmouseout（鼠标离开元素范围）、onmouseover（鼠标移动到元素范围）、onmouseup（鼠标按键释放）等。

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;1234567890&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        var tid;
        function setTimeoutDemo() {
            tid = setTimeout(&#34;alert(&#39;下课了&#39;)&#34;, 3000);
        }
        function clearTimeoutDemo() {
            //判断tid是否初始化
            if (tid) {
                clearTimeout(tid);
            }
        }

        var dir = &#34;left&#34;; 
        function scroll() {
            var title = window.document.title;
            if (dir == &#34;left&#34;) {
                var first = title.charAt(0);
                var last = title.substring(1, title.length); //start 从0数  end从1数
            } else if (dir == &#34;right&#34;) {
                var last = title.charAt(title.length - 1);
                var first = title.substring(0, title.length - 1);
            }
            window.document.title = last &#43; first;
        }
        setInterval(&#34;scroll()&#34;, 500);
       
        function setDir(str) {
            dir = str;
        }
        
        

        function scrollRight() {
            var title = window.document.title;
            var last = title.charAt(title.length - 1);
            var first = title.substring(0, title.length - 1);
            title = last &#43; first;

            window.document.title = title;
        }

        window.showModalDialog(&#34;1-.htm&#34;);
        
        function showDialog() {
            window.showModalDialog(&#34;1-.htm&#34;);
        }
        function show() {
            window.showModelessDialog(&#34;1-.htm&#34;);
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input type=&#34;button&#34; value=&#34;启动&#34; onclick=&#34;setTimeoutDemo();&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;取消&#34; onclick=&#34;clearTimeoutDemo();&#34; /&gt;
    
    
    &lt;input type=&#34;button&#34; value=&#34;向左&#34; onclick=&#34;setDir(&#39;left&#39;)&#34; /&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;向右&#34; onclick=&#34;setDir(&#39;right&#39;)&#34; /&gt;
    &lt;br /&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;模式窗口&#34; onclick=&#34;showDialog()&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;非模式窗口&#34; onclick=&#34;show()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        var times = 10;
        function count() {
            var btn = document.getElementById(&#34;btn&#34;);
            if (times &gt; 0) {
                btn.value = &#34;同意(倒计时&#34; &#43; times &#43; &#34;)&#34;;
                times--;
            } else {
                btn.value = &#34;同意&#34;;
                btn.disabled = false;
                clearInterval(tid);
            }
        }
        var tid = setInterval(&#34;count()&#34;, 1000);
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;count()&#34;&gt;
    注册协议
    &lt;br /&gt;
    &lt;input id=&#34;btn&#34; type=&#34;button&#34; value=&#34;同意&#34; disabled=&#34;disabled&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

## window对象属性

&#43;  `window.location.href=&#39;http://www.itcast.cn&#39;`，重新导向新的地址，和navigate方法效果一样。window.location.reload() 刷新页面
&#43;  window.event是非常重要的属性，用来获得发生事件时的信息，事件不局限于window对象的事件，所有元素的事件都可以通过event属性取到相关信息。类似于winForm中的e(EventArg).
   &#43; altKey属性，bool类型，表示发生事件时alt键是否被按下，类似的还有ctrlKey、shiftKey属性，例子 `&lt;input type=&#34;button&#34; value=&#34;点击&#34; onclick=&#34;if(event.altKey){alert(&#39;Alt点击&#39;)}else{alert(&#39;普通点击&#39;)}&#34; /&gt; ；`
   &#43; clientX、clientY 发生事件时鼠标在客户区的坐标；screenX、screenY 发生事件时鼠标在屏幕上的坐标；offsetX、offsetY 发生事件时鼠标相对于事件源（比如点击按钮时触发onclick）的坐标。
   &#43; returnValue属性，如果将returnValue设置为false，就会取消默认事件的处理。在超链接的onclick里面禁止访问href的页面。在表单校验的时候禁止提交表单到服务器，防止错误数据提交给服务器、防止页面刷新。
   &#43; srcElement，获得事件源对象。几个事件共享一个事件响应函数用。
   &#43; keyCode，发生事件时的按键值。
   &#43; button，发生事件时鼠标按键，1为左键，2为右键，3为左右键同时按。`&lt;body  onmousedown=&#34;if(event.button==2){alert(&#39;禁止复制&#39;);}&#34;&gt;`



##  （*）screen对象，屏幕的信息

`alert(&#34;分辨率：&#34; &#43; screen.width &#43; &#34;*&#34; &#43; screen.height);`

```HTML
if (screen.width &lt; 1024 || screen.height &lt; 768){
alert(&#34;分辨率太低！&#34;);
}
```

## clipboardData对象

&#43; 对粘贴板的操作。clearData(&#34;Text&#34;)清空粘贴板；getData(&#34;Text&#34;)读取粘贴板的值，返回值为粘贴板中的内容；setData(&#34;Text&#34;,val)，设置粘贴板中的值。
  &#43; 案例：复制地址给友好。见备注。
  &#43; 当复制的时候body的oncopy方法被触发，直接return false就是禁止复制。&lt;body oncopy=&#34;alert(&#39;禁止复制！&#39;);return false;&#34;
  &#43; 很多元素也有oncopy、onpaste事件：
  &#43; 案例：禁止粘贴帐号。见备注。
&#43; 在网站中复制文章的时候，为了防止那些拷贝党不添加文章来源，自动在复制的内容后添加版权声明。

```html
function modifyClipboard() {
clipboardData.setData(&#39;Text&#39;, clipboardData.getData(&#39;Text&#39;) &#43; &#39;本文来自传智播客技术专区，转载请注明来源。&#39; &#43; location.href);
}
```

	&#43; `oncopy=&#34;setTimeout(&#39;modifyClipboard()&#39;,100)&#34;`。用户复制动作发生0.1秒以后再去改粘贴板中的内容。100ms只是一个经常取值，写1000、10、50、200……都行。不能直接在oncopy里修改粘贴板。
	&#43; 不能直接在oncopy中执行对粘贴板的操作，因此设定定时器，0.1秒以后执行，这样就不再oncopy的执行调用栈上了。

&#43; history操作历史记录
  &#43; window.history.back()后退；window.history.forward()前进。也可以用window.history.go(-1)、window.history.go(1)前进

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function getXY() {
            document.title = window.event.clientX &#43; &#34;       &#34; &#43; window.event.clientY;
        }
        function turnInto(right) {
            if (right) {
                alert(&#34;欢迎进入&#34;);
            } else {
                alert(&#34;非法入侵&#34;);
                window.event.returnValue = false;
                alert(&#34;123123&#34;);
            }
        }

        function btnClick() {
            return false;
            alert(&#34;abc&#34;);
        }

        function txtKeyDown() {
            var txt = window.event.srcElement;
            if (txt.id == &#34;txtNums&#34;) {
                if (window.event.keyCode &gt;= 48 &amp;&amp; window.event.keyCode &lt;= 57) {
                } else {
                return false;
                }
            } else if (txt.id = &#34;txt&#34;) { 
                
            }
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onmousemove=&#34;getXY()&#34; onmousedown=&#34;if(window.event.button==2){alert(&#39;禁止复制&#39;)}&#34;&gt;
    &lt;input type=&#34;button&#34; value=&#34;url&#34; onclick=&#34;alert(window.location.href);&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;转向&#34; onclick=&#34;window.location.href=&#39;2-window对象.htm&#39;&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;reload&#34; onclick=&#34;window.location.reload()&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;ctrlKey&#34; onclick=&#34;if(window.event.ctrlKey){alert(&#39;按下ctrl&#39;)}else{alert(&#39;没有按下&#39;)}&#34; /&gt;
    &lt;br /&gt;
    &lt;a href=&#34;1-.htm&#34; onclick=&#34;turnInto(0)&#34;&gt;超链接&lt;/a&gt;
    &lt;input type=&#34;button&#34; value=&#34;returnValue&#34; onclick=&#34;btnClick();&#34; /&gt;
    &lt;form action=&#34;http://www.baidu.com&#34;&gt;
        &lt;input type=&#34;submit&#34; onclick=&#34;alert(&#39;请输入用户名密码&#39;);window.event.returnValue=false&#34;/&gt;
    &lt;/form&gt;

    &lt;input id=&#34;txtNums&#34; type=&#34;text&#34; value=&#34;&#34; onkeydown=&#34;return txtKeyDown()&#34; /&gt;
    &lt;input id=&#34;txt&#34; type=&#34;text&#34; value=&#34;&#34; onkeydown=&#34;txtKeyDown()&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;screen&#34; onclick=&#34;alert(window.screen.width &#43; &#39; &#39; &#43; window.screen.height)&#34; /&gt;

    &lt;hr /&gt;
    手机号：&lt;input type=&#34;text&#34; value=&#34;&#34; oncopy=&#34;alert(&#39;禁止复制&#39;);return false&#34; /&gt;&lt;br /&gt;
    
    重复手机号：&lt;input type=&#34;text&#34; value=&#34;&#34; onpaste=&#34;alert(&#39;请输入&#39;);return false&#34; /&gt;
    &lt;hr /&gt;
    &lt;input id=&#34;tabc&#34; type=&#34;text&#34; value=&#34;213123123&#34; /&gt;&lt;input type=&#34;button&#34; value=&#34;copy&#34; onclick=&#34;window.clipboardData.setData(&#39;text&#39;,tabc.value);alert(&#39;复制成功&#39;);&#34; /&gt;&lt;br /&gt;
    &lt;input id=&#34;t123&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;&lt;input type=&#34;button&#34; value=&#34;paste&#34; onclick=&#34;t123.value=clipboardData.getData(&#39;text&#39;)&#34; /&gt;&lt;br /&gt;

&lt;/body&gt;
&lt;/html&gt;

```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function setClip() {
            var text = window.clipboardData.getData(&#34;text&#34;);
            text = text &#43; &#34;转载请注明：&#34; &#43; window.location.href;
            window.clipboardData.setData(&#34;text&#34;,text);
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;

    &lt;textarea id=&#34;t1&#34; oncopy=&#34;setTimeout(&#39;setClip()&#39;,50)&#34;&gt;
      asdfasdfasdf
        asdfasdfasdf
    
    &lt;/textarea&gt;
    &lt;a href=&#34;6-history.htm&#34;&gt;链接&lt;/a&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input type=&#34;button&#34; value=&#34;后退&#34; onclick=&#34;window.history.back()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;

```

### document属性。

是最复杂的属性之一。后面讲解详细使用。

&#43; document是window对象的一个属性，因为使用window对象成员的时候可以省略window.，所以一般直接写document

&#43; document的方法：
  &#43; write：向文档中写入内容。writeln，和write差不多，只不过最后添加一个回车

```html
&lt;input type=&#34;button&#34; value=&#34;点击&#34; onclick=&#34;document.write(&#39;&lt;font color=red&gt;你好&lt;/font&gt;&#39;)&#34; /&gt;
```

	&#43; 在onclick等事件中写的代码会冲掉页面中的内容，只有在页面加载过程中write才会与原有内容融合在一起

```html
&lt;script type=&#34;text/javascript&#34;&gt;
document.write(&#39;&lt;font color=red&gt;你好&lt;/font&gt;&#39;);
&lt;/script&gt;
```

	&#43; write经常在广告代码、整合资源代码中被使用。见备注

内容联盟、广告代码、cnzz，不需要被主页面的站长去维护内容，只要被嵌入的js内容提供商修改内容，显示的内容就变了。

&#43; getElementById方法（非常常用），根据元素的Id获得对象，网页中id不能重复。也可以直接通过元素的id来引用元素，但是有有效范围、form1.textbox1之类的问题，因此不建议直接通过id操作元素，而是通过getElementById
&#43; `（*）getElementsByName`，根据元素的name获得对象，由于页面中元素的name可以重复，比如多个RadioButton的name一样，因此getElementsByName返回值是对象数组。
&#43; `（*）getElementsByTagName`，获得指定标签名称的元素数组，比如getElementsByTagName(&#34;p&#34;)可以获得所有的&lt;p&gt;标签。
  &#43; 案例：实现checkbox的全选，反选
  &#43; 案例：点击一个按钮，被点击的按钮显示“呜呜”，其他按钮显示“哈哈”。
  &#43; 案例：十秒钟后协议文本框下的注册按钮才能点击，时钟倒数。(btn.disabled = true )

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function writeDemo() {
            document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
            document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
            document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
        }
        document.writeln(&#34;123&lt;br/&gt;&#34;);
        document.writeln(&#34;123&lt;br/&gt;&#34;);
        document.writeln(&#34;&lt;a href=&#39;123.htm&#39;&gt;123&lt;/a&gt;&#34;);
        document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
        document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
        document.write(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;);
        document.write(&#34;&lt;ul&gt;&lt;li&gt;开始&lt;/li&gt;&lt;li&gt;运行&lt;/li&gt;&lt;li&gt;结束&lt;/li&gt;&lt;/ul&gt;&#34;);

        //document.write(&#34;&lt;script type=&#39;text/javascript&#39;&gt;alert(&#39;hello world&#39;);&lt;\/script&gt;&#34;);
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div&gt;abc
    &lt;script type=&#34;text/javascript&#34;&gt;
        document.writeln(&#34;123&lt;br/&gt;&#34;);
        document.writeln(&#34;123&lt;br/&gt;&#34;);
       &lt;/script&gt;abc
    &lt;/div&gt;
    &lt;div&gt;
    sdfsd
    &lt;/div&gt;
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; onclick=&#34;writeDemo()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function setText() {
            var txt = document.getElementById(&#34;t1&#34;);
            txt.value = &#34;123&#34;;
            //t1.value = &#34;123&#34;;
        }
        function setText2() {
            var txt = document.getElementById(&#34;t2&#34;);
            txt.value = &#34;abc&#34;;
           //form1.t2.value = &#34;abc&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input id=&#34;t1&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;form id=&#34;form1&#34;&gt;
        &lt;input id=&#34;t2&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;/form&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; onclick=&#34;setText()&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;按钮2&#34; onclick=&#34;setText2()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function btnClick() {
            var chks = document.getElementsByName(&#34;hobby&#34;);
              //错误
//            for (var c in chks) {
//                alert(c);
            //            }
            for (var i = 0; i &lt; chks.length; i&#43;&#43;) {
                //alert(chks[i].value);
                chks[i].checked = &#34;checked&#34;;
            }
        }

        function checkAll() {
            var chkAll = document.getElementById(&#34;chkAll&#34;);
            
            var chks = document.getElementsByName(&#34;hobby&#34;);
            for (var i = 0; i &lt; chks.length; i&#43;&#43;) {
                chks[i].checked = chkAll.checked;
            }
        }
        function reverseCheck() {
            var chks = document.getElementsByName(&#34;hobby&#34;);
            for (var i = 0; i &lt; chks.length; i&#43;&#43;) {
                chks[i].checked = !chks[i].checked;
            }
        }
        function checkSingle() {
            var b = true;    //假设全被选中   
            var chkAll = document.getElementById(&#34;chkAll&#34;);
            var chks = document.getElementsByName(&#34;hobby&#34;);
            for (var i = 0; i &lt; chks.length; i&#43;&#43;) {
                //查找所有子的checkbox，判断是否被选中
                //如果有一个checkbox没有被选中，则退出循环，最终全选的checkbox为false
                if (!chks[i].checked) {
                    b = false;
                    break;
                }
            }
            chkAll.checked = b;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input type=&#34;checkbox&#34; value=&#34;cf&#34; onclick=&#34;checkSingle()&#34; name=&#34;hobby&#34;/&gt;吃饭&lt;br /&gt;
    &lt;input type=&#34;checkbox&#34; value=&#34;sj&#34; onclick=&#34;checkSingle()&#34; name=&#34;hobby&#34;/&gt;睡觉&lt;br /&gt;
    &lt;input type=&#34;checkbox&#34; value=&#34;ddd&#34; onclick=&#34;checkSingle()&#34; name=&#34;hobby&#34;/&gt;打豆豆&lt;br /&gt;
    &lt;br /&gt;&lt;br /&gt;
    &lt;input id=&#34;chkAll&#34; type=&#34;checkbox&#34; onclick=&#34;checkAll()&#34;/&gt;全选
    &lt;input type=&#34;button&#34; value=&#34;反选&#34; onclick=&#34;reverseCheck()&#34; /&gt;
    
    
    &lt;input type=&#34;button&#34; value=&#34;全选&#34; onclick=&#34;btnClick()&#34; /&gt;
   
    &lt;br /&gt;
    &lt;input type=&#34;checkbox&#34; value=&#34;ctl&#34; /&gt;春天里&lt;br /&gt;
    &lt;input type=&#34;checkbox&#34; value=&#34;xtl&#34; /&gt;夏天里&lt;br /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function setAllText() {
            var txts = document.getElementsByTagName(&#34;input&#34;);
            for (var i = 0; i &lt; txts.length; i&#43;&#43;) {
                if (txts[i].type == &#34;text&#34;) {
                    txts[i].value = i &#43; 1;
                }
            }
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; onclick=&#34;setAllText()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;

```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function init() {
            var btns = document.getElementsByTagName(&#34;input&#34;);
            for (var i = 0; i &lt; btns.length; i&#43;&#43;) {
                if (btns[i].type == &#34;button&#34;) { 
                    //给按钮注册单击事件(并没有调用)
                    btns[i].onclick = btnClick;
                }
            }
        }
        
        function btnClick() {
            var btns = document.getElementsByTagName(&#34;input&#34;);
            for (var i = 0; i &lt; btns.length; i&#43;&#43;) {
                if (btns[i].type == &#34;button&#34;) {
                    //判断按钮 是不是事件源
                    if (btns[i] == window.event.srcElement) {
                        btns[i].value = &#34;呜呜&#34;;
                    } else {
                        btns[i].value = &#34;哈哈&#34;;
                    }
                }
            }
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34;  /&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;哈哈&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function add() {
            var txt1 = document.getElementById(&#34;txtNum1&#34;);
            var txt2 = document.getElementById(&#34;txtNum2&#34;);
            var num1 = parseInt(txt1.value);
            var num2 = parseInt(txt2.value);
            var sum = num1 &#43; num2;

            document.getElementById(&#34;txtSum&#34;).value=sum;
        }
    
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input id=&#34;txtNum1&#34; type=&#34;text&#34; /&gt;&lt;input type=&#34;button&#34; value=&#34;&#43;&#34; /&gt;
    &lt;input id=&#34;txtNum2&#34; type=&#34;text&#34; /&gt;&lt;br /&gt;
    &lt;input type=&#34;button&#34; value=&#34;=&#34; onclick=&#34;add()&#34; /&gt;&lt;br /&gt;
    &lt;input id=&#34;txtSum&#34; type=&#34;text&#34; readonly=&#34;readonly&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

## dom动态创建

document.write只能在页面加载过程中才能动态创建。
可以调用document的createElement方法来创建具有指定标签的DOM对象，然后通过调用某个元素的appendChild方法将新创建元素添加到相应的元素下

```HTMl
function showit() {
var divMain = document.getElementById(&#34;divMain&#34;);
var btn = document.createElement(&#34;input&#34;);
btn.type = &#34;button&#34;;
btn.value = &#34;我是动态的！&#34;;
divMain.appendChild(btn);
}
&lt;div id=&#34;divMain&#34;&gt;&lt;/div&gt;
&lt;input type=&#34;button&#34; value=&#34;ok&#34; onclick=&#34;showit()&#34; /&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function createBtn() {
            var btn = document.createElement(&#34;input&#34;);
            btn.type = &#34;button&#34;;
            btn.value = &#34;新按钮&#34;;
            btn.onclick = function() {
                alert(&#34;我是新来的&#34;);
            }
            var d = document.getElementById(&#34;d1&#34;);
            d.appendChild(btn);
        }

        function createLink() {
            var link = document.createElement(&#34;a&#34;);
            link.href = &#34;http://www.baidu.com&#34;;
            link.innerText = &#34;百度&#34;;
            link.target = &#34;_blank&#34;;

            var d = document.getElementById(&#34;d1&#34;);
            d.appendChild(link);
        }
        
        function btnClick(){
            alert(&#34;我是新来的&#34;);
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input type=&#34;button&#34; value=&#34;创建按钮&#34; onclick=&#34;createBtn()&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;创建超链接&#34; onclick=&#34;createLink()&#34; /&gt;
    &lt;div id=&#34;d1&#34;&gt;abc&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
```

&#43; Value 获取表单元素
&#43; 几乎所有DOM元素都有innerText、innerHTML属性（注意大小写），分别是元素标签内内容的文本表示形式和HTML源代码，这两个属性是可读可写的。

```HTML
&lt;a href=&#34;http://www.itcast.cn&#34; id=&#34;link1&#34;&gt;传&lt;font color=&#34;Red&#34;&gt;智&lt;/font&gt;播客&lt;/a&gt;
&lt;input type=&#34;button&#34; value=&#34;inner*&#34; onclick=&#34;alert(document.getElementById(&#39;link1&#39;).innerText);alert(document.getElementById(&#39;link1&#39;).innerHTML);&#34; /&gt;
```

&#43; 用innerHTML也可以替代createElement，属于简单、粗放型、后果自负的创建

```HTML
function createlink() {
var divMain = document.getElementById(&#34;divMain&#34;);
divMain.innerHTML = &#34;&lt;a href=&#39;http://www.rupeng.com&#39;&gt;如鹏网&lt;/a&gt;&#34;;
}
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function getLinkText() {
            var link = document.getElementById(&#34;link1&#34;);
            alert(link.innerText);
            alert(link.innerHTML);
        }
        function setDiv() {
            var div = document.getElementById(&#34;d1&#34;);
            //div.innerText = &#34;&lt;font color=&#39;red&#39;&gt;123123&lt;/font&gt;&#34;;
            //div.innerHTML = &#34;&lt;font color=&#39;red&#39;&gt;123123&lt;/font&gt;&#34;;
            //div.innerHTML = &#34;&lt;ul&gt;&lt;li&gt;春天里&lt;/li&gt;&lt;li&gt;怒放&lt;/li&gt;&lt;/ul&gt;&#34;;
            //div.innerHTML = &#34;&lt;input type=&#39;text&#39; value=&#39;1234&#39; /&gt;&#34;;

            //div.innerText = div.innerText &#43; &#34;123123&#34;;

            //Node节点 Element元素的区别
            //html文档里所有的内容都是节点 标签 属性  文本
            //元素  一个完整的标签
			//var txtNode = document.createTextNode(&#34;123123&#34;);
            //            div.appendChild(txtNode);

            div.innerHTML = &#34;&lt;script type=&#39;text/javascript&#39;&gt;function test(){alert(&#39;hello&#39;);}&lt;\/script&gt;&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;a id=&#34;link1&#34; href=&#34;http://www.itcast.cn&#34;&gt;传智&lt;font color=&#34;red&#34;&gt;播客&lt;/font&gt;&lt;/a&gt;
    
    
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; onclick=&#34;getLinkText()&#34; /&gt;
    &lt;div id=&#34;d1&#34;&gt;abcd&lt;/div&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;set div&#34; onclick=&#34;setDiv() &#34; /&gt;
    
        &lt;input type=&#34;button&#34; value=&#34;test&#34; onclick=&#34;test()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function createTable() {
            var div = document.getElementById(&#34;d1&#34;);
            var dic = { &#34;baidu&#34;: &#34;http://www.baidu.com&#34;, &#34;传智播客&#34;: &#34;http://www.itcast.cn&#34;, &#34;cnbeta&#34;: &#34;http://www.cnbeta.com&#34; };
            var table = document.createElement(&#34;table&#34;);
            table.border = 1;
            for (var key in dic) {
                var tr = document.createElement(&#34;tr&#34;);
                var td0 = document.createElement(&#34;td&#34;);
                td0.innerText = key;
                //把td0加到tr中
                tr.appendChild(td0);

                var td1 = document.createElement(&#34;td&#34;);
                td1.innerHTML = &#34;&lt;a href=&#39;&#34; &#43; dic[key] &#43; &#34;&#39;&gt;&#34; &#43; dic[key] &#43; &#34;&lt;/a&gt;&#34;;
                tr.appendChild(td1);
                //把tr添加到table中
                table.appendChild(tr);
            }
            //把table添加到div中
            div.appendChild(table);
        }
    
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div id=&#34;d1&#34;&gt;&lt;/div&gt;
    &lt;input type=&#34;button&#34; value=&#34;load。。。&#34; onclick=&#34;createTable()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

## 浏览器兼容性的例子

ie6，ie7对table.appendChild(&#34;tr&#34;)的支持和IE8不一样，用insertRow、insertCell来代替或者为表格添加tbody，然后向tbody中添加tr。FF不支持InnerText。

所以动态加载网站列表的程序修改为：

```HTML
var tr = tableLinks.insertRow(-1);//FF必须加-1这个参数
var td1 = tr.insertCell(-1);
td1.innerText = key;
var td2 = tr.insertCell(-1);
td2.innerHTML = &#34;&lt;a href=&#39;&#34; &#43; value &#43; &#34;&#39;&gt;&#34; &#43; value &#43; &#34;&lt;/a&gt;&#34;;
```

或者：

```HTML
&lt;table id=&#34;tableLinks&#34;&gt;
&lt;tbody&gt;&lt;/tbody&gt;
&lt;/table&gt;，然后tableLinks. tBodies[0].appendChild(tr);
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    
    &lt;script type=&#34;text/javascript&#34;&gt;
        function add() {
            var name = document.getElementById(&#34;txtName&#34;).value;
            var content = document.getElementById(&#34;txtContent&#34;).value;

            var tr = document.createElement(&#34;tr&#34;);
            var td0 = document.createElement(&#34;td&#34;);
            td0.innerText = name;
            tr.appendChild(td0);

            var td1 = document.createElement(&#34;td&#34;);
            td1.innerText = content;
            tr.appendChild(td1);

            var table = document.getElementById(&#34;re&#34;);
            table.appendChild(tr);
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;p&gt;
        最新新闻
    &lt;/p&gt;
    
    &lt;table id=&#34;re&#34;&gt;&lt;/table&gt;
    &lt;hr /&gt;
    
    评论：
        &lt;input id=&#34;txtName&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;&lt;br /&gt;
        &lt;textarea id=&#34;txtContent&#34; cols=&#34;50&#34; rows=&#34;10&#34;&gt;&lt;/textarea&gt;
        
        &lt;br /&gt;
        
        &lt;input type=&#34;button&#34; value=&#34;评论&#34; onclick=&#34;add()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function add() {
            var name = document.getElementById(&#34;txtName&#34;).value;
            var content = document.getElementById(&#34;txtContent&#34;).value;
            
            var table = document.getElementById(&#34;re&#34;);
            var tr = table.insertRow(-1);
            
            var td = tr.insertCell(-1);
            td.innerHTML = name;

            var td1 = tr.insertCell(-1);
            td1.innerHTML = content;
            
        }
    
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;p&gt;
        最新新闻
    &lt;/p&gt;
    
    &lt;table id=&#34;re&#34;&gt;&lt;/table&gt;
    &lt;hr /&gt;
    
    评论：
        &lt;input id=&#34;txtName&#34; type=&#34;text&#34; value=&#34;&#34; /&gt;&lt;br /&gt;
        &lt;textarea id=&#34;txtContent&#34; cols=&#34;50&#34; rows=&#34;10&#34;&gt;&lt;/textarea&gt;
        
        &lt;br /&gt;
        
        &lt;input type=&#34;button&#34; value=&#34;评论&#34; onclick=&#34;add()&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

### 事件冒泡

事件冒泡：如果元素A嵌套在元素B中，那么A被点击不仅A的onclick事件会被触发，B的onclick也会被触发。触发的顺序是“由内而外” 。验证：在页面上添加一个table、table里有tr、tr里有td，td里放一个p，在p、td、tr、table中添加onclick事件响应，见备注。

```HTML
&lt;table onclick=&#34;alert(&#39;table onclick&#39;);&#34;&gt;
&lt;tr onclick=&#34;alert(&#39;tr onclick&#39;);&#34;&gt;
&lt;td onclick=&#34;alert(&#39;td onclick&#39;);&#34;&gt;&lt;p onclick=&#34;alert(&#39;p onclick&#39;);&#34;&gt;aaaa&lt;/p&gt;&lt;/td&gt;
&lt;td&gt;bbb&lt;/td&gt;
&lt;/tr&gt;
&lt;/table&gt;
```

```HTML
&lt;!DOCTYPE html PUBLIC &#34;-//W3C//DTD XHTML 1.0 Transitional//EN&#34; &#34;http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&#34;&gt;
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function btn3() {
            alert(event.srcElement.value);
        }
        //事件响应函数的调用函数
        function btn4() {
            alert(this.value);
        }

        function initBtn5() {
            var btn = document.getElementById(&#34;btn5&#34;);
            //事件响应函数
            btn.onclick = btn4;
        }   
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;initBtn5()&#34;&gt;
   &lt;table onclick=&#34;alert(&#39;table&#39;)&#34;&gt;
    &lt;tr onclick=&#34;alert(&#39;tr&#39;)&#34;&gt;
        &lt;td onclick=&#34;alert(&#39;td&#39;)&#34;&gt; &lt;div onclick=&#34;alert(&#39;div&#39;)&#34;&gt;asd&lt;/div&gt;&lt;/td&gt;
        &lt;td&gt;2&lt;/td&gt;
    &lt;/tr&gt;
   &lt;/table&gt;
   
   &lt;input type=&#34;button&#34; value=&#34;click1&#34; onclick=&#34;alert(event.srcElement.value)&#34; /&gt;&lt;br /&gt;
   &lt;!-- 事件响应函数--&gt;
   &lt;input type=&#34;button&#34; value=&#34;click2&#34; onclick=&#34;alert(this.value)&#34; /&gt;
   
   &lt;input type=&#34;button&#34; value=&#34;click3&#34; onclick=&#34;btn3()&#34; /&gt;
   &lt;input type=&#34;button&#34; value=&#34;click4&#34; onclick=&#34;btn4()&#34; /&gt;
   
   &lt;input id=&#34;btn5&#34; type=&#34;button&#34; value=&#34;click5&#34; /&gt;
   
   
&lt;/body&gt;
&lt;/html&gt;

```

## this

事件中的this。

除了可以使用event.srcElement在事件响应函数中

this表示发生事件的控件。

只有在事件响应函数才能使用this获得发生事件的控件，在事件响应函数调用的函数中不能使用，如果要使用则要将this传递给函数或者使用event.srcElement。

`(*)this`和event.srcElement的语义是不一样的，this就是表示当前监听事件的这个对象，event.srcElement是引发事件的对象：事件冒泡。


## 修改样式

易错：修改元素的样式不是设置class属性，而是className属性。案例：网页开关灯的效果。
修改元素的样式不能`this.style=&#34;background-color:Red&#34;`。
易错：单独修改样式的属性使用“style.属性名”。注意在css中属性名在JavaScript中操作的时候属性名可能不一样，主要集中在那些属性名中含有-的属性，因为JavaScript中-是不能做属性、类名的。所以CSS中背景颜色是background-color，而JavaScript则是style.backgroundColor；元素样式名是`class，在JavaScript中`是className属性；`font-size→style.fontSize；margin-top→style.marginTop `
单独修改控件的样式`&lt;input type=&#34;button&#34; value=&#34;AAA&#34; onclick=&#34;this.style.color=&#39;red&#39;&#34; /&gt;`。技巧，没有文档的情况下的值属性名，随便给一个元素设定id，然后在js中就能id.style.出来能用的属性。

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;style type=&#34;text/css&#34;&gt;
        .light{
            background-color:White;
        }
        .dark
        {
        	background-color:Black;
        }
    &lt;/style&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function open1() {
            document.body.className = &#34;light&#34;;
        }
        function close1() {
            document.body.className = &#34;dark&#34;;
        }

        function change() {
            var txt = document.getElementById(&#34;txt1&#34;);
            //错误 不能这样用。可以把style看成一个只读属性
            //txt.style = &#34;background-color:Green&#34;;
            
            txt.style.backgroundColor = &#34;Green&#34;;
            txt.style.color = &#34;red&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;input type=&#34;button&#34; value=&#34;开灯&#34; onclick=&#34;open1()&#34; /&gt;
&lt;input type=&#34;button&#34; value=&#34;关灯&#34; onclick=&#34;close1()&#34; /&gt;

&lt;input id=&#34;txt1&#34; type=&#34;text&#34; value=&#34;123&#34; /&gt;
&lt;input type=&#34;button&#34; value=&#34;click&#34; onclick=&#34; change() &#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function initTxt() {
            var txts = document.getElementsByTagName(&#34;input&#34;);
            for (var i = 0; i &lt; txts.length; i&#43;&#43;) {
                if (txts[i].type == &#34;text&#34;) {
                    //事件响应函数
                    txts[i].onblur = iBlur;
                }
            }
        }
    
        function iBlur() {
            if (this.value.length &lt;= 0) {
                this.style.backgroundColor = &#34;red&#34;;
            } else {
            this.style.backgroundColor = &#34;white&#34;;
            }
        }
        function iFocus(txt) {
            txt.value = &#34;&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;initTxt()&#34;&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34;/&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;click&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function iBlur(txt) {
            var t2 = document.getElementById(&#34;t2&#34;);
            t2.style.backgroundColor = txt.style.backgroundColor;
            t2.style.color = txt.style.color;
            t2.style.width = txt.style.width;
            t2.value = txt.value;
        }
        function iFocus(txt) {
            var t1 = document.getElementById(&#34;t1&#34;);
            txt.style.backgroundColor = t1.style.backgroundColor;
            txt.style.color = t1.style.color;
            txt.style.width = t1.style.width;
            txt.value = t1.value;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;input id=&#34;t1&#34; type=&#34;text&#34; value=&#34;&#34; style=&#34;background-color:Green; color:Red; width:300px&#34; /&gt;
    &lt;input type=&#34;text&#34; onfocus=&#34;iFocus(this)&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        
        
        function init() {
            var table = document.getElementById(&#34;rating&#34;);
            var tds = table.getElementsByTagName(&#34;td&#34;);
            for (var i = 0; i &lt; tds.length; i&#43;&#43;) {
                //事件响应函数
                tds[i].onmouseover = change;
                tds[i].onclick = stop;
                tds[i].style.cursor = &#34;pointer&#34;;
            }
        }
        //记录是否点鼠标，默认没点
        var isClick = false;
        function stop() {
            isClick = true;
        }
        function indexOf(arr,element){
            var j = -1;
            for(var i=0;i&lt;arr.length;i&#43;&#43;){
                if(arr[i] == element)
                {
                    j = i;
                    break;
                }
            }
            return j;
        }
        function change() {
            //当没点鼠标的时候去执行
            if (!isClick) {
                var table = document.getElementById(&#34;rating&#34;);
                var tds = table.getElementsByTagName(&#34;td&#34;);
                var n = indexOf(tds, this);
                for (var i = 0; i &lt;= n; i&#43;&#43;) {
                    //tds[i].style.backgroundColor = &#34;red&#34;;
                    tds[i].innerText = &#34;★&#34;;
                }

                for (var i = n &#43; 1; i &lt; tds.length; i&#43;&#43;) {
                    //tds[i].style.backgroundColor = &#34;white&#34;;
                    tds[i].innerText = &#34;☆&#34;;
                }
            }
            
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;table id=&#34;rating&#34;&gt;
        &lt;tr&gt;
            &lt;td&gt;☆&lt;/td&gt;&lt;td&gt;☆&lt;/td&gt;&lt;td&gt;☆&lt;/td&gt;&lt;td&gt;☆&lt;/td&gt;&lt;td&gt;☆&lt;/td&gt;
        &lt;/tr&gt;
    &lt;/table&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function init() {
            var d1 = document.getElementById(&#34;d1&#34;);
            var links = d1.getElementsByTagName(&#34;a&#34;);
            for (var i = 0; i &lt; links.length; i&#43;&#43;) {
                links[i].onclick = linkClick;
            }
        }
        function linkClick() {
            var d1 = document.getElementById(&#34;d1&#34;);
            var links = d1.getElementsByTagName(&#34;a&#34;);
            //当前a标签变成红色
            this.style.backgroundColor = &#34;red&#34;;
            //让其它标签变成白色
            for (var i = 0; i &lt; links.length; i&#43;&#43;) {
                if (links[i] != this) {
                    links[i].style.backgroundColor = &#34;white&#34;;
                }
            }

            //取消后续内容的执行
                window.event.returnValue = false;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;div id=&#34;d1&#34;&gt;
        &lt;a href=&#34;http://www.baidu.com&#34;&gt;百度&lt;/a&gt;
        &lt;a href=&#34;http://www.qiushibaike.com&#34;&gt;糗百&lt;/a&gt;
        &lt;a href=&#34;http://www.cnbeta.com&#34;&gt;cnbeta&lt;/a&gt;
    &lt;/div&gt;
    &lt;div id=&#34;d2&#34;&gt;
        &lt;a href=&#34;http://www.baidu.com&#34;&gt;百度&lt;/a&gt;
        &lt;a href=&#34;http://www.qiushibaike.com&#34;&gt;糗百&lt;/a&gt;
        &lt;a href=&#34;http://www.cnbeta.com&#34;&gt;cnbeta&lt;/a&gt;
    &lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function init() {
            var table = document.getElementById(&#34;students&#34;);
            var trs = table.getElementsByTagName(&#34;tr&#34;);
            for (var i = 0; i &lt; trs.length; i&#43;&#43;) {
                //鼠标经过的时候，换颜色
                trs[i].onmouseover = onOver;
                //当鼠标离开时候，还原颜色
                trs[i].onmouseout = onOut;
                //隔行换色
                if (i % 2 == 0) {
                    trs[i].style.backgroundColor = &#34;red&#34;;
                } else {
                trs[i].style.backgroundColor = &#34;yellow&#34;;
                }
            }
        }
        var defaultColor;
        function onOver() {
            defaultColor = this.style.backgroundColor;
            this.style.backgroundColor = &#34;gray&#34;;
        }
        function onOut() {
            this.style.backgroundColor = defaultColor;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;table id=&#34;students&#34; border=&#34;1&#34; width=&#34;400px&#34;&gt;
        &lt;tr&gt;
            &lt;td&gt;刘备&lt;/td&gt;
            &lt;td&gt;男&lt;/td&gt;
            &lt;td&gt;19&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;关羽&lt;/td&gt;
            &lt;td&gt;男&lt;/td&gt;
            &lt;td&gt;18&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;张飞&lt;/td&gt;
            &lt;td&gt;男&lt;/td&gt;
            &lt;td&gt;17&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;曹操&lt;/td&gt;
            &lt;td&gt;男&lt;/td&gt;
            &lt;td&gt;20&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;吕布&lt;/td&gt;
            &lt;td&gt;男&lt;/td&gt;
            &lt;td&gt;19&lt;/td&gt;
        &lt;/tr&gt;
    &lt;/table&gt;
&lt;/body&gt;
&lt;/html&gt;

```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;script type=&#34;text/javascript&#34;&gt;
        function init() {
            var txts = document.getElementsByTagName(&#34;input&#34;);
            for (var i = 0; i &lt; txts.length; i&#43;&#43;) {
                if (txts[i].type == &#34;text&#34;) {
                    txts[i].onfocus = function() { this.style.backgroundColor = &#34;red&#34;; };
                    txts[i].onblur = function() { this.style.backgroundColor = &#34;white&#34;; };
                }
            }
        }
        function reset() {
            this.style.backgroundColor = &#34;white&#34;;
        }
        function change() {
            this.style.backgroundColor = &#34;red&#34;;
//            
//            var txts = document.getElementsByTagName(&#34;input&#34;);
//            for (var i = 0; i &lt; txts.length; i&#43;&#43;) {
//                if (txts[i] != this) {
//                    txts[i].style.backgroundColor = &#34;white&#34;;
//                }
//            }
            
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    &lt;input type=&#34;text&#34; value=&#34;&#34; /&gt;
    
    &lt;input type=&#34;button&#34; value=&#34;按钮&#34; /&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
    &lt;style type=&#34;text/css&#34;&gt;
        #l li
        {
        	width:300px;
        	list-style-type:none;
        	float:left;
        	}
    &lt;/style&gt;
   &lt;script type=&#34;text/javascript&#34;&gt;
       function init() {
       
           var ul = document.getElementById(&#34;l&#34;);
           var lis = ul.getElementsByTagName(&#34;li&#34;);
           for (var i = 0; i &lt; lis.length; i&#43;&#43;) {
               lis[i].style.cursor = &#34;pointer&#34;;
               
               lis[i].onclick = function() {
                   var ul = document.getElementById(&#34;l&#34;);
                   var lis = ul.getElementsByTagName(&#34;li&#34;);
                   this.style.backgroundColor = &#34;red&#34;;
                   for (var i = 0; i &lt; lis.length; i&#43;&#43;) {
                       if (lis[i] != this) {
                           lis[i].style.backgroundColor = &#34;white&#34;;
                       }
                   }
               };
           }
       }
   &lt;/script&gt;
&lt;/head&gt;
&lt;body onload=&#34;init()&#34;&gt;
    &lt;ul id=&#34;l&#34;&gt;
        &lt;li&gt;C#&lt;/li&gt;
        &lt;li&gt;java&lt;/li&gt;
        &lt;li&gt;JavaScript&lt;/li&gt;
        &lt;li&gt;html&lt;/li&gt;
    &lt;/ul&gt;
&lt;/body&gt;
&lt;/html&gt;
```

```HTML
&lt;html xmlns=&#34;http://www.w3.org/1999/xhtml&#34; &gt;
&lt;head&gt;
    &lt;title&gt;&lt;/title&gt;
   
    &lt;script type=&#34;text/javascript&#34;&gt;
        function hide(btn) {
            var div = document.getElementById(&#34;d1&#34;);
            if (div.style.display == &#34;none&#34;) {
                div.style.display = &#34;&#34;;
                btn.value = &#34;隐藏&#34;;
            }else{
            div.style.display = &#34;none&#34;;
            btn.value = &#34;显示&#34;;
            }
        }
        function show() {
            var div = document.getElementById(&#34;d1&#34;);
            div.style.display = &#34;&#34;;
        }

        function detailsShow(chk) {
            var div = document.getElementById(&#34;d3&#34;);
            if (chk.checked) {
                div.style.display = &#34;&#34;;
            } else {
                div.style.display = &#34;none&#34;;
            }
        }

        function aOver() {
            //screenX 鼠标在屏幕中坐标
            var div = document.getElementById(&#34;d4&#34;);
            div.style.position = &#34;absolute&#34;;
            div.style.top = window.event.clientY;//鼠标在浏览器中的位置
            div.style.left = window.event.clientX;
            div.style.display = &#34;&#34;;

        }
        function aOut() {
            var div = document.getElementById(&#34;d4&#34;);
            div.style.display = &#34;none&#34;;
        }
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div id=&#34;d1&#34;&gt;
        怎么会迷上你，我在问自己。。。
    &lt;/div&gt;
    &lt;div id=&#34;d2&#34;&gt;
        春天里百花香
    &lt;/div&gt;
    &lt;input type=&#34;button&#34; value=&#34;隐藏&#34; onclick=&#34;hide(this)&#34; /&gt;
    &lt;input type=&#34;button&#34; value=&#34;显示&#34; onclick=&#34;show()&#34; /&gt;
    &lt;hr /&gt;
    
    &lt;input type=&#34;checkbox&#34; onclick=&#34;detailsShow(this)&#34; /&gt;显示详细信息
    &lt;div id=&#34;d3&#34; style=&#34; display:none&#34;&gt;
        详细信息
    &lt;/div&gt;
    
    &lt;hr /&gt;
    
    &lt;a href=&#34;http://www.baidu.com&#34; onmouseover=&#34;aOver()&#34; onmouseout=&#34;aOut()&#34;&gt;百度&lt;/a&gt;
    &lt;a href=&#34;http://www.google.com&#34; onmouseover=&#34;aOver()&#34;  onmouseout=&#34;aOut()&#34;&gt;google&lt;/a&gt;
    &lt;a href=&#34;http://www.sougou.com&#34; onmouseover=&#34;aOver()&#34;  onmouseout=&#34;aOut()&#34;&gt;sougou&lt;/a&gt;
    
    &lt;div id=&#34;d4&#34; style=&#34; display:none; border:dotted 1px red;&#34;&gt;
        &lt;img src=&#34;IMG_8910.jpg&#34; width=&#34;200px&#34; height=&#34;100px&#34; /&gt;
        &lt;br /&gt;我拍的照片
        &lt;br /&gt;赵苑
    &lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
```

## 控制层显示

&#43; 修改style.display，例子：切换层的显示

```HTML
function togglediv() {
	var div1 = document.getElementById(&#39;div1&#39;);
	if (div1.style.display == &#39;&#39;) {
	    div1.style.display = &#39;none&#39;;//不显示
	}
	else {
	    div1.style.display = &#39;&#39;;//显示
	}
}

```

案例：注册页面，点击“高级”CheckBox，则显示高级选项，否则隐藏
案例：鼠标放到一个超链接的时候，在鼠标的位置显示一个黄色背景，带图片的悬浮提示，鼠标离开就消失。提示：鼠标进入控件的事件是onmouseover，离开的事件是onmouseout。

## Body事件的范围

&#43; IE中如果在body上添加onclick、onmousemove等事件响应，那么如果页面没有满，则 “body 中最后一个元素以下（横向不限制）” 的部分是无法响应事件的，必须使用代码在document上监听那些事件，比如document.onmousemove = MovePic
&#43; FF中也差不多。

## 元素的位置、大小单位

&#43; 通过dom读取元素的top、left、width、height等取到的值不是数字，而是“10px”这样的字符串；为这些属性设值的时候IE可以是80、90这样的数字，FF必须是“80px”、“90%”等这样的字符串形式，为了兼容统一用字符串形式。
&#43; 易错：不要写成div1.style.width=80px，而是div1.style.width=&#39;80px&#39;
&#43; 如果要修改元素的大小（宽度加10），则首先要取出元素的宽度，然后用parseInt将宽度转换为数字（parseInt可以将&#34;20px&#34;这样数字开头的包含其他内容的字符串解析为20，）；然后加上一个值，再加上px赋值回去。

## 层的操作

&#43; 元素的position 样式值：static（无定位，显示在默认位置）、absolute（绝对定位）、fixed（相对于窗口的固定定位，位置不会随着浏览器的滚动而变化，IE6不支持）、relative（相对元素默认位置的定位）。如果要通过代码修改元素的坐标则一般使用absolute，然后修改元素的top（上边缘距离）、left（左边缘距离）两个样式值。left、top都是指的层的左上角的坐标
&#43; 案例：跟着鼠标飞的图片。提示：鼠标移动的事件是onmousemove（一边移动事件一边触发，而不是移动开始或者移动完成才触发），通过window.event的clientX、clientY属性获得鼠标的位置。
&#43; 案例：放三个超链接，鼠标放上时候动态生成一个层，层显示在鼠标的位置，鼠标离开的时候移除该层removeChild
&#43; 案例：点击按钮层动态变大。提示：英文字母连续单词不会在中间自动换行的陷阱

## 易错

&#43; 易错：不要写成div1.style.width=80px，而是div1.style.width=&#39;80px&#39;
&#43; 修改元素的样式不能this.style=&#34;background-color:Red&#34;，哪怕可以的话也是把以前所有样式都冲掉了。单独修改控件的样式this.style. background=&#39;red&#39;，只修改要修改的样式。技巧，没有文档的情况下的值属性名，随便给一个元素设定id，然后在js中就能id.style.出来能用的属性。
&#43; createElement的两种用法，注意innerText的问题
&#43; var input = document.createElement(&#34;&lt;input type=&#39;button&#39; value=&#39;hello&#39;/&gt;&#34;)快速创建元素，并且赋值，但是注意设置的inner部分不会被设置var link = document.createElement(&#34;&lt;a href=&#39;http://www.baidu.com&#39;&gt;百度&lt;/a&gt;&#34;)
&#43; label.setAttribute(&#34;for&#34;, &#34;username&#34;); //设定一些Dom元素属性名特殊的属性,label.for = &#34;username&#34;会有问题。label.setAttribute(&#34;xuehao&#34;,&#34;33333&#34;)

## form对象

&#43; document.getElementById(&#39;btn1&#39;).click()
&#43; form对象是表单的Dom对象。
&#43; 方法：submit()提交表单，但是不会触发onsubmit事件。
&#43; 实现autopost，也就是焦点离开控件以后页面立即提交，而不是只有提交submit按钮以后才提交，当光标离开的时候触发onblur事件，在onblur中调用form的submit方法。代码见备注。
&#43; 在点击submit后form的onsubmit事件被触发，在onsubmit中可以进行数据校验，数据数据有问题，返回false即可取消提交

```hTML
&lt;form name=&#34;form1&#34; action=&#34;a.aspx&#34; method=&#34;get&#34; onsubmit=&#34;if(document.getElementById(&#39;txtname&#39;).value.length&lt;=0){alert(&#39;姓名必填&#39;);return false;}&#34;&gt;
```

&#43; 自动提交

```HTML
&lt;form name=&#34;form1&#34; action=&#34;a.aspx&#34; method=&#34;get&#34;&gt;
&lt;input type=&#34;text&#34; onblur=&#34;form1.submit()&#34; /&gt;
&lt;input type=&#34;text&#34; /&gt;        
&lt;/form&gt;

```

## 不同浏览器差异

&#43; 面试题：说说开发项目的时候不同浏览器的不同点，你是怎么解决的？Button,appendChild,insertCell,px
&#43; 不同浏览器中对DOM支持的方法不一样
&#43; 获取网页中那个元素触发了事件：在IE里使用srcElement ；在FireFox里使用target 
&#43; 使用Dom获取和更改网页标签元素内文本：在IE里使用innerText ；在FireFox里使用textContent 
&#43; 动态为网页或元素绑定事件：在IE中绑定事件的方法是attachEvent ；在FireFox中绑定事件的方法是addEventListener 
&#43; 更多http://www.360doc.com/content/09/0319/12/16915_2855107.shtml。
&#43; 不同浏览器中对CSS的支持不一样，所以出现在IE中显示正常的网页，在FF下全部乱掉了。哀悼网页使用的CSS只有IE支持，FF都不支持。
&#43; JQuery之类的框架进行了封装，将不同浏览器的差异帮开发人员处理了，开发人员只要调用JQuery的方法，JQuery会帮助在不同浏览器中进行翻译。用JQuery就可以解决不同浏览器上Dom的不同。对于CSS的不同是美工的事，IETester、FF、Chrome。

## 弹出对话框处理

&#43; 复习，使用window.showModalDialog(&#39;dialog.htm&#39;)弹出模态对话框
&#43; 给对话框传递参数，使用showModalDialog的第二个参数传递参数，在对话框中用window.dialogArguments获得传递的参数值；对话框中给window.parent.returnValue设定返回值，这样在父窗口中就可以通过showModalDialog返回值读取设置的返回值了。例子：弹出对话框询问用户姓名，向用户问好；弹出含有“是”、“否”、“取消”三个按钮的模态窗口，点击按钮的时候窗口关闭，然后主窗口显示用户点击的按钮。
&#43; 传递多个参数，将参数包装到数组中，然后仍然是通过第二个参数传递，返回多个返回值也可以返回数组：var arr = new Array();arr[0]=30;arr[1]=&#34;tom&#34;;
&#43; 练习（面试题），弹出一个含有确定、取消、重试三个按钮的对话框，并且得知用户的选择。

&#43; 给对话框传递参数，使用showModalDialog的第二个参数传递参数，在对话框中用window.dialogArguments获得传递的参数值；对话框中给window.parent.returnValue设定返回值，这样在父窗口中就可以通过showModalDialog返回值读取设置的返回值了：
  dialog.htm：

```Html
function getData()
{
	return document.getElementById(&#39;mytext1&#39;).value;
}
&lt;body onLoad=&#34;javascript:document.getElementById(&#39;mytext1&#39;).value=window.dialogArguments;&#34;&gt;
	&lt;input type=&#34;text&#34; id=&#34;mytext1&#34;/&gt;
	&lt;input type=&#34;button&#34; value=&#34;确定&#34; onclick=&#34;javascript:window.parent.returnValue=getData();window.close();&#34;&gt;
&lt;/body&gt;
```

主页面：

```HTML
var result=window.showModalDialog(&#39;dialog2.htm&#39;,777);
alert(result);
```

&#43; 传递多个参数，将参数包装到数组中即可：var arr = new Array();arr[0]=30;arr[1]=&#34;tom&#34;;

## JS正则表达式

&#43; JavaScript中创建正则表达式类的方法：
&#43; var regex = new RegExp(&#34;\\d{5}&#34;) 或者 var regex = /\d{5}/
&#43; /表达式/是JavaScript中专门为简化正则表达式编写而提供的语法，写在//中的正则表达式就不用管转义符了。
&#43; RegExp对象的方法：

（1）test(str)判断字符串str是否匹配正则表达式，相当于IsMatch
	

```Html
        var regex = /.&#43;@.&#43;/;
        alert(regex.test(&#34;a@b.com&#34;));
        alert(regex.test(&#34;ab.com&#34;));
```

（2）exec(str)进行搜索匹配，返回值为匹配结果，没找到返回null(*)
（3）compile编译表达式，提高运行速度。 (*)

## string 的正则表达

&#43; Replace  match
&#43; String对象中提供了一些与正则表达式相关的方法，相当于对于RegExp类的包装，简化调用：
&#43; match(regexp)，相当于调用exec

```HTML
var s = &#34;aaa@163.com&#34;;
var regex = /(.&#43;)@(.&#43;)/;
var match = s.match(regex);
alert(RegExp.$1 &#43; &#34;，服务器：&#34; &#43; RegExp.$2);
```

&#43; 练习：光标离开Email地址框的时候用正则表达式校验是否是合法的Email地址，如果不是的话Email地址框变红，并且注册按钮禁用，否则Email地址框颜色为白色，启用注册按钮。

## HTML JS压缩

&#43; HTML、JavaScript的压缩和混淆。去掉空格、缩短变量名，让js、html尺寸更小，提高下载速度。
&#43; HTML、JS压缩、混淆有动态和静态两种方案。HTML压缩器，比如HTML Compress，JavaScript压缩工具：Google Closure Compiler、YUI Compressor 等。
&#43; 很多js库都提供了.min.js、compress.js的压缩版本。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2019/04/web4-dom/  

