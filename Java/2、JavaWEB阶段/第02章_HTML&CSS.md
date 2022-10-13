# 第02章_HTML&CSS

---

## 第一节 单一架构回顾

我们从现在的JavaWeb阶段到后面学习SSM框架阶段都是在学习单一架构项目开发的技术。而在JavaWeb阶段由于重点是探讨如何实现Web开发，所以必须学习一部分前端开发的技术。本节就是让大家明确我们现在要学习的内容在整个架构体系中处于什么位置。

### 1、单一架构技术体系

![image-20221008163152767](images/image-20221008163152767.png)

### 2、视图层

严格来说视图分成两层：

- 前端技术：HTML/CSS/JavaScript
- 服务器端页面模板技术：Thymeleaf

其中HTML、CSS、JavaScript都是工作在浏览器上的，所以它们都属于前端技术。而Thymeleaf是在服务器上把动态内容计算出具体数据，所以严格来说Thymeleaf是后端技术。

![image-20221008164744840](images/image-20221008164744840.png)

> 这里大家会有个疑问：为什么在**视图**这个地方已经有HTML、CSS、JavaScript这些前端技术了，能够生成用户可以操作的界面，那为什么还需要Thymeleaf这样一个后端技术呢？
>
> 简单来说原因是Thymeleaf=HTML+动态数据，而HTML不支持动态数据，这部分需要借助Thymeleaf来完成。
>
> 更进一步的细节咱们讲到那再说啦！

### 3、Web2.0

Web2.0是相对于更早的网页开发规范而提出的新规范。Web2.0规范之前的网页开发并没有明确的将HTML、CSS、JavaScript代码分开，而是互相之间纠缠在一起，导致代码维护困难，开发效率很低。

> 在开发中我们把这样彼此纠缠、互相影响的现象称为**耦合**。而把耦合在一起的东西拆解开，让他们彼此独立出来称为**解耦**。各个组成部分独立完成自己负责的功能，和其他模块无关称为『内聚』。
>
> 将来大家经常会听到一句话：软件开发提倡**高内聚，低耦合**。
>
> 一个软件项目只有做到了高内聚、低耦合才能算得上结构严谨，模块化程度高，有利于开发和维护。

所以Web2.0规范主张将网页代码分成下面三个部分：

- 结构：由HTML实现，负责管理网页的内容。将来网页上不管是静态还是动态的数据都是填写到HTML的标签里。
- 表现：由CSS实现，负责管理网页内容的表现形式。比如：颜色、尺寸、位置、层级等等。也就是给数据穿上一身漂亮的衣服。
- 行为：由JavaScript实现，负责实现网页的动态交互效果。比如：轮播图、表单验证、鼠标滑过显示下拉菜单、鼠标滑过改变背景颜色等等。



## 第二节 HTML简介
### 1、名词解释
HTML是**H**yper **T**ext **M**arkup **L**anguage的缩写。意思是**超文本标记语言**。

### 2、超文本
HTML文件本质上是文本文件，而普通的文本文件只能显示字符。但是HTML技术则通过HTML标签把其他网页、图片、音频、视频等各种多媒体资源引入到当前网页中，让网页有了非常丰富的呈现方式，这就是超文本的含义——本身是文本，但是呈现出来的最终效果超越了文本。

### 3、标记语言
说HTML是一种**标记语言**是因为它不是向Java这样的**编程语言**，因为它是由一系列**标签**组成的，没有常量、变量、流程控制、异常处理、IO等等这些功能。HTML很简单，每个标签都有它固定的含义和确定的页面显示效果。

标签是通过一组尖括号+标签名的方式来定义的：

```html
<p>HTML is a very popular fore-end technology.</p>
```


这个例子中使用了一个p标签来定义一个段落，<p>叫**开始标签**，</p>叫**结束标签**。开始标签和结束标签一起构成了一个完整的标签。开始标签和结束标签之间的部分叫**文本标签体**，也简称**标签体**。

有的时候标签里还带有**属性**：

```html
<a href="http://www.xxx.com">show detail</a>
```

href="http://www.xxx.com"就是属性，href是**属性名**，"http://www.xxx.com"是**属性值**。

还有一种标签是**单标签**：

```html
<input type="text" name="username" />
```



### 4、HelloWorld
<img src="images/image-20221008170227301.png" alt="image-20221008170227301" style="zoom:80%;" />



### 5、HTML文件结构

**①文档类型声明**
HTML文件中第一行的内容，用来告诉浏览器当前HTML文档的基本信息，其中最重要的就是当前HTML文档遵循的语法标准。这里我们只需要知道HTML有4和5这两个大的版本，

HTML4版本的文档类型声明是：

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">
```

HTML5版本的文档类型声明是：

```html
<!DOCTYPE html>
```

现在主流的技术选型都是使用HTML5，之前的版本基本不用了。


历史上HTML的各个版本：

版本名称	年份
HTML	1991
HTML+	1993
HTML2.0	1995
HTML3.2	1997
HTML4.01	1999
XHTML1.0	2000
HTML5	2012
XHTML5	2013

**②根标签**
html标签是整个文档的根标签，所有其他标签都必须放在html标签里面。上面的文档类型不能当做普通标签看待。

> 所谓『根』其实是『树根』的意思。在一个树形结构中，根节点只能有一个。

**③头部**
head标签用于定义文档的头部，其他头部元素都放在head标签里。头部元素包括title标签、script标签、style标签、link标签、meta标签等等。

**④主体**
body标签定义网页的主体内容，在浏览器窗口内显示的内容都定义到body标签内。

**⑤注释**
HTML注释的写法是：

<!-- 注释内容 -->
注释的内容不会显示到浏览器窗口内，是开发人员用来对代码内容进行解释说明。

### 6、HTML语法规则
* 根标签有且只能有一个

* 无论是双标签还是单标签都必须正确关闭

* 标签可以嵌套但不能交叉嵌套

* 注释不能嵌套

* 属性必须有值，值必须加引号，单引号或双引号均可

* 标签名不区分大小写但建议使用小写

  

## 第三节 使用HTML展示文章

以文章的组织形式展示数据是HTML最基本的功能了，网页上显示的文章在没有做任何CSS样式设定的情况下如下图所示：

<img src="images/image-20221010164221053.png" alt="image-20221010164221053" style="zoom:67%;" />

本节我们要学习的HTML标签如下表：

| 标签名称 | 功能                   |
| -------- | ---------------------- |
| h1~h6    | 1级标题~6级标题        |
| p        | 段落                   |
| a        | 超链接                 |
| ul/li    | 无序列表               |
| img      | 图片                   |
| div      | 定义一个前后有换行的块 |
| span     | 定义一个前后无换行的块 |

为了方便编写代码，我们在IDEA中创建一个静态Web工程来操作：

### 1. 标签标题

1. 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

    <h1>这是一级标题</h1>
    <h2>这是二级标题</h2>
    <h3>这是三级标题</h3>
    <h4>这是四级标题</h4>
    <h5>这是五级标题</h5>
    <h6>这是六级标题</h6>

</body>
</html>
```

2. 页面显示效果

![image-20221010164857955](images/image-20221010164857955.png)

**注意**：标题标签前后有换行。

### 2、段落标签

1. 代码

```html
<p>There is clearly a need for CSS to be taken seriously by graphic artists. The Zen Garden aims to excite, inspire, and encourage participation. To begin, view some of the existing designs in the list. Clicking on any one will load the style sheet into this very page. The code remains the same, the only thing that has changed is the external .css file. Yes, really.</p>
```

2. 页面显示效果

![image-20221010165356836](images/image-20221010165356836.png)

**注意**：段落标签前后有换行。

### 3、超链接

1. 代码

```html
<a href="page02-anchor-target.html">点我跳转到下一个页面</a>
```

2. 页面显示效果

![image-20221010165443273](images/image-20221010165443273.png)

点击后跳转到href属性指定的页面

### 4、路径

在我们整个Web开发技术体系中，**路径**是一个贯穿始终的重要概念。凡是需要获取另外一个资源的时候都需要用到路径。要想理解路径这个概念，我们首先要认识一个概念：**文件系统**

**①文件系统**

我们写代码的时候通常都是在Windows系统来操作，而一个项目开发完成后想要让所有人都能够访问到就必须**部署**到服务器上，也叫**发布**。而服务器通常是Linux系统。

Windows系统和Linux系统的文件系统有很大差别，为了让我们编写的代码不会因为从Windows系统部署到了Linux系统而出现故障，实际开发时不允许使用**物理路径**。

> 物理路径举例：
>D:\aaa\pro01-HTML\page01-article-tag.html
> D:\aaa\pro01-HTML\page02-anchor-target.html

幸运的是不管是Windows系统还是Linux系统环境下，目录结构都是**树形结构**，编写路径的规则是一样的。

![image-20221010165607885](images/image-20221010165607885.png)

所以我们**以项目的树形目录结构为依据**来编写路径就不用担心操作系统平台发生变化之后路径错误的问题了。有了这个大前提，我们具体编写路径时有两种具体写法：

- 相对路径
- 绝对路径（建议使用）

**②相对路径**

**相对路径都是以『当前位置』为基准**来编写的。假设我们现在正在浏览a页面，想在a页面内通过超链接跳转到z页面。

![image-20221010165852205](images/image-20221010165852205.png)

那么按照相对路径的规则，我们现在所在的位置是a.html所在的b目录：

![image-20221010170028340](images/image-20221010170028340.png)

z.html并不在b目录下，所以我们要从b目录出发，向上走，进入b的父目录——c目录：

![image-20221010170049305](images/image-20221010170049305.png)

c目录还是不行，继续向上走，进入c的父目录——d目录：

![image-20221010170100443](images/image-20221010170100443.png)

在从d目录向下经过两级子目录——e目录、f目录才能找到z.html：

![image-20221010170112474](images/image-20221010170112474.png)

所以整个路径的写法是：

```html
<a href="../../e/f/z.html">To z.html</a>
```

可以看到使用相对路径有可能会很繁琐，而且在后面我们结合了在服务器上运行的Java程序后，相对路径的基准是有可能发生变化的，所以**不建议使用相对路径**。

**③绝对路径**

**[1]通过IDEA服务器打开HTML文件**

测试绝对路径的前提是通过IDEA的内置服务器访问我们编写的HTML页面——这样访问地址的组成结构才能和我们以后在服务器上运行的Java程序一致。

![image-20221011101258366](images/image-20221011101258366.png)



**[2]服务器访问地址的组成**

![image-20221011101321119](images/image-20221011101321119.png)

**[3]绝对路径的写法**

绝对路径要求必须是以**『正斜线』**开头。这个开头的正斜线在整个服务器访问地址中对应的位置如下图所示：

![image-20221011101410304](images/image-20221011101410304.png)

这里标注出的这个位置代表的是**『服务器根目录』**，从这里开始我们就是在服务器的内部查找一个具体的Web应用。

所以我们编写绝对路径时就从这个位置开始，**按照目录结构找到目标文件**即可。拿前面相对路径中的例子来说，我们想在a.html页面中通过超链接访问z.html。此时路径从正斜线开始，和a.html自身所在位置没有任何关系：

![image-20221011101502688](images/image-20221011101502688.png)

```html
<a href="/d/e/f/z.html">To z.html</a>
```



**[4]具体例子**

编写超链接访问下面的页面：

![image-20221011101528701](images/image-20221011101528701.png)

```html
<a href="/aaa/pro01-HTML/animal/cat/miao.html">Cat Page</a>
```



**[5]小结**

为了和我们后面学习的内容和正确的编码方式保持一致，建议大家从现在开始就使用绝对路径。



### 5、换行

1. 代码

```html
We would like to see as much CSS1 as possible. CSS2 should be limited to widely-supported elements only. The css Zen Garden is about functional, practical CSS and not the latest bleeding-edge tricks viewable by 2% of the browsing public. <br/>The only real requirement we have is that your CSS validates.
```

*2. 页面显示效果

![image-20221011101613240](images/image-20221011101613240.png)



### 6、无序列表

1. 代码

```html
    <ul>
        <li>Apple</li>
        <li>Banana</li>
        <li>Grape</li>
    </ul>
```



2. 页面显示效果

![image-20221011101737544](images/image-20221011101737544.png)

### 7、图片

1. 准备图片文件

![image-20221011101753375](images/image-20221011101753375.png)

2. 代码

src属性用来指定图片文件的路径，这里同样按我们前面说的使用**绝对路径**。

```html
<img src="/aaa/pro01-HTML/./images/mi.jpg"/>
```

3. 页面显示效果

![image-20221011101812577](images/image-20221011101812577.png)



### 8、块

**块**并不是为了显示文章内容的，而是为了方便结合CSS对页面进行布局。块有两种，

**div是前后有换行的块，span是前后没有换行的块**。

把下面代码粘贴到HTML文件中查看他们的区别：

```html
<div style="border: 1px solid black;width: 100px;height: 100px;">This is a div block</div>
<div style="border: 1px solid black;width: 100px;height: 100px;">This is a div block</div>

<span style="border: 1px solid black;width: 100px;height: 100px;">This is a span block</span>
<span style="border: 1px solid black;width: 100px;height: 100px;">This is a span block</span>
```

页面显示效果为：

![image-20221011102507489](images/image-20221011102507489.png)


### 9、HTML实体

在HTML文件中，<、>等等这样的符号已经被赋予了特定含义，不会作为符号本身显示到页面上，此时如果我们想使用符号本身怎么办呢？那就是使用HTML实体来转义。

**注：实体名称对大小写敏感**

![image-20221011102527638](images/image-20221011102527638.png)



### 10、表格

1. 代码

```html
<table border="1px" width="600px" cellspacing="0px" cellpadding="4px">
    <tr align="center">
        <th>标题1</th>
        <th>标题2</th>
        <th>标题3</th>
    </tr>
    <tr align="center">
        <td rowspan="2">行合并</td>
        <td>单元格2</td>
        <td>单元格3</td>
    </tr>
    <tr align="center">
        <td>单元格2</td>
        <td>单元格3</td>
    </tr>
    <tr align="center">
        <td colspan="3">列合计</td>
    </tr>
</table>
```



2. 页面显示效果

![image-20221011104632881](images/image-20221011104632881.png)



## 第五节 使用HTML表单收集数据

### 1、什么是表单

在项目开发过程中，凡是需要用户填写的信息都需要用到表单

![image-20221011104903403](images/image-20221011104903403.png)

---

### 2、form标签

在HTML中使用form标签来定义一个表单，对于form标签来说有两个最重要的属性：**action**和**method**

```html
<form action="/aaa/pro01-HTML/page05-form-target.html" method="post">
</form>
```

**①action属性**

用户在表单里填写的信息需要发送到服务器端，对于Java项目来说就是交给Java代码来处理。那么在页面上我们就必须正确填写服务器端的能够接收表单数据的地址。

这个地址要写在form标签的action属性中。但是现在暂时我们还没有服务器端环境，所以先借用一个HTML页面来当作服务器端地址使用



**②method属性**

『method』这个单词的意思是『方式、方法』，在form标签中method属性用来定义提交表单的**『请求方式』**。method属性只有两个可选值：get或post，没有极特殊情况的话使用post即可。

> 什么是**『请求方式』**？
>
> 浏览器和服务器之间在互相通信时有大量的**『数据』**需要传输。但是不论是浏览器还是服务器都有很多不同厂商提供的不同产品。
>
> 常见的浏览器有：
>
> - Chrome
> - Firefox
> - Safari
> - Opera
> - Edge
>
> 常见的Java服务器有：
>
> - Tomcat
> - Weblogic
> - WebSphere
> - Glassfish
> - Jetty
>
> 这么多不同厂商各自开发的应用程序怎么能保证它们彼此之间传输的**『数据』**能够被对方正确理解呢？
>
> 很简单，我们给这些数据设定**『格式』**，发送端按照格式发送数据，接收端按照格式解析数据，这样就能够实现数据的**『跨平台传输』**了。
>
> 而这里定义的**『数据格式』**就是应用程序之间的**『通信协议』**。
>
> 在JavaSE阶段的网络编程章节我们接触过TCP/IP、UDP这样的协议，而我们现在使用的**『HTTP协议』**的底层就是TCP/IP协议。
>
> HTTP1.1中共定义了八种请求方式：
>
> - GET
> - POST
> - PUT
> - DELETE
> - HEAD
> - CONNECT
> - OPTIONS
> - TRACE
>
> 但是在HTML标签中，点击超链接是GET方式的请求，提交一个表单可以通过form标签的method属性指定GET或POST请求，其他请求方式无法通过HTML标签实现。除了GET、POST之外的其他请求方式暂时我们不需要涉及（到我们学习SpringMVC时会用到PUT和DELETE）。至于**GET请求和POST请求的区别**我们会在讲HTTP协议的时候详细介绍，现在大家可以从表面现象来观察一下。

---

### 3、name和value

在用户使用一个软件系统时，需要一次性提交很多数据是非常正常的现象。我们肯定不能要求用户一个数据一个数据的提交，而肯定是所有数据填好后一起提交。那就带来一个问题，服务器怎么从众多数据中识别出来收货人、所在地区、详细地址、手机号码……？

很简单，**给每个数据都起一个『名字』**，发送数据时用**『名字』**携带对应的数据，接收数据时通过**『名字』**获取对应的数据。

在各个具体的表单标签中，我们通过**『name属性』**来给数据起**『名字』**，通过**『value属性』**来保存要发送给服务器的**『值』**。

但是名字和值之间既有可能是**『一个名字对应一个值』**，也有可能是**『一个名字对应多个值』**。

这么看来这样的关系很像我们Java中的Map，而事实上在服务器端就是使用Map类型来接收请求参数的。具体的是类型是：**Map<String,String[]>**。

name属性就是Map的键，value属性就是Map的值。

有了上面介绍的基础知识，下面我们就可以来看具体的表单标签了

---

### 4、单行文本框

1. 代码

```html
个性签名：<input type="text" name="signal"/><br/>
```

2. 显示效果

![image-20221011113337666](images/image-20221011113337666.png)

---

### 5、密码框

1. 代码

```html
密码：<input type="password" name="secret"><br>
```

2. 显示效果

![image-20221011113733276](images/image-20221011113733276.png)

---

### 6、单选框

1. 代码

```html
你最喜欢的季节是：
<input type="radio" name="season" value="spring" />春天
<input type="radio" name="season" value="summer" checked="checked" />夏天
<input type="radio" name="season" value="autumn" />秋天
<input type="radio" name="season" value="winter" />冬天

<br/><br/>

你最喜欢的动物是：
<input type="radio" name="animal" value="tiger" />路虎
<input type="radio" name="animal" value="horse" checked="checked" />宝马
<input type="radio" name="animal" value="cheetah" />捷豹
```

2. 显示效果

![image-20221011113956185](images/image-20221011113956185.png)

3. 说明

- name属性相同的radio为一组，组内互斥
- 当用户选择了一个radio并提交表单，这个radio的name属性和value属性组成一个键值对发送给服务器
- 设置checked="checked"属性设置默认被选中的radio

---

### 7、多选框

1. 代码

```html
你最喜欢的球队是：
<input type="checkbox" name="team" value="Brazil"/>巴西
<input type="checkbox" name="team" value="German" checked="checked"/>德国
<input type="checkbox" name="team" value="France"/>法国
<input type="checkbox" name="team" value="China" checked="checked"/>中国
<input type="checkbox" name="team" value="Italian"/>意大利
```

2. 显示效果

![image-20221011130541278](images/image-20221011130541278.png)

---

### 8、下拉列表

1. 代码

```html
你喜欢的运动是：
<select name="interesting">
    <option value="swimming">游泳</option>
    <option value="running">跑步</option>
    <option value="shooting" selected="selected">射击</option>
    <option value="skating">溜冰</option>
</select>
```

2. 效果

![image-20221011134250118](images/image-20221011134250118.png)

---

### 9、按钮

1. 代码

```html
<button type="button">普通按钮</button>
<button type="reset">重置按钮</button>
<button type="submit">提交按钮</button>
```

2. 效果

![image-20221011134949286](images/image-20221011134949286.png)

3. 说明

   | 类型     | 功能                                             |
   | -------- | ------------------------------------------------ |
   | 普通按钮 | 点击后无效果，需要通过JavaScript绑定单击响应函数 |
   | 重置按钮 | 点击后将表单内的所有表单项都恢复为默认值         |
   | 提交按钮 | 点击后提交表单                                   |

---

### 10、表单隐藏域

1. 代码

```html
<input type="hidden" name="userId" value="2233"/>
```

2. 说明

通过表单隐藏域设置的表单项不会显示到页面上，用户看不到。但是提交表单时会一起被提交。用来设置一些需要和表单一起提交但是不希望用户看到的数据，例如：用户id等等。

---

### 11、多行文本框

1. 代码

```html
自我介绍：<textarea name="desc"></textarea>
```

2. 效果

![image-20221011135457872](images/image-20221011135457872.png)

3. 说明

   textarea没有value属性，如果要设置默认值需要写在开始和结束标签之间

---

## 第六节 CSS的简单应用

### 1、CSS效果示例

### 2、设置CSS样式的三种方式

1. 在HTML标签内设置

   仅对当前标签有效

   ```html
   <div style="border: 1px solid black;width: 100px; height: 100px;">&nbsp;</div>
   ```

2. 在head标签内设置

   对当前页面有效

   ```html
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
       <style type="text/css">
           .one {
               border: 1px solid black;
               width: 100px;
               height: 100px;
               background-color: lightgreen;
               margin-top: 5px;
           }
       </style>
   </head>
   <body>
   
       <div style="border: 1px solid black;width: 100px; height: 100px;">&nbsp;</div>
   
       <div class="one">&nbsp;</div>
       <div class="one">&nbsp;</div>
       <div class="one">&nbsp;</div>
   
   </body>
   ```

   ![image-20221011142304779](images/image-20221011142304779.png)

3. 引入外部CSS样式文件

   [1] 创建CSS文件

   ![image-20221011142435782](images/image-20221011142435782.png)

   [2] 编辑CSS文件

   ```css
   .two {
       border: 1px solid black;
       width: 100px;
       height: 100px;
       background-color: yellow;
       margin-top: 5px;
   }
   ```

   [3] 引入外部CSS文件

   在需要这个CSS文件的HTML页面的head标签内加入

   ```html
   <link rel="stylesheet" type="text/css" href="/aaa/pro01-HTML/style/example.css" />
   ```

   ```html
   <div class="two">&nbsp;</div>
   <div class="two">&nbsp;</div>
   <div class="two">&nbsp;</div>
   ```

   ![image-20221011142953496](images/image-20221011142953496.png)



### 3、CSS代码语法

- CSS样式由选择器和声明组成，而声明又由属性和值组成。
- 属性和值之间用冒号隔开。
- 多条声明之间用分号隔开。
- 使用/* ... */声明注释。

![image-20221011143118046](images/image-20221011143118046.png)



### 4、CSS选择器

1. 标签选择器

   HTML代码

   ```html
   <p>Hello, this is a p tag.</p>
   <p>Hello, this is a p tag.</p>
   <p>Hello, this is a p tag.</p>
   <p>Hello, this is a p tag.</p>
   <p>Hello, this is a p tag.</p>
   ```

   CSS代码

   ```css
   p {
       color: blue;
       font-weight: bold;
   }
   ```

   ![image-20221011143641782](images/image-20221011143641782.png)

2. id选择器

   HTML代码

   ```html
   <p>Hello, this is a p tag.</p>
   <p id="special">Hello, this is a p tag.</p>
   ```

   CSS代码

   ```css
   #special {
       font-size:20px;
       background-color:aquq;
   }
   ```

   ![image-20221011143740632](images/image-20221011143740632.png)

3. 类选择器

   HTML代码

   ```html
   <div class="one">&nbsp;</div>
   <div class="one">&nbsp;</div>
   <div class="one">&nbsp;</div>
   ```

   CSS代码

   ```css
   .one {
       border: 1px solid black;
       width: 100px;
       height: 100px;
       background-color: lightgreen;
       margin-top: 5px;
   }
   ```

   ![image-20221011143857285](images/image-20221011143857285.png)