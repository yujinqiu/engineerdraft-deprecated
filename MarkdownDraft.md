# Markdown
-----------------
## 核心思想
-----------------
Markdown 只关注两件事情，以真正实现易读易写的目标：

1.格式化的纯文本语法；

2.John Gruber 用 Perl 开发的脚本工具将纯文本转换成格式化的 HTML. 

** 个人认为本质上就是 You'll get what you think .核心思想: [规范指导写作,程序自动转换显示效果] ** 

## 基本语法
------------------
### 标题
\# 一号标题

\## 二号标题

...

一共支持六号标题. 
### 斜体和强调
\* *斜体* \*

\*\* **强调** \*\*

\*\*\* ***斜体加强调*** \*\*\*

### 引用
\> 
>应用的效果

### 代码
1. 整段代码用 Tab 或者 4个空格分隔. 
2. 用``` `` ``` 来括起来 **一行代码**. 

### 分割线
采用 *** 或者 --- 就能够实现分隔: 效果如下

---

### 表格

|  这里是表头 | 第二列表头 | 
|------------|----------|
|第一行 | 第一行 | 
| 第二行 | 第二行 | 

还通过冒号为每个列制定对其方向：

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right

### 删除线
~~这个是效果图~~, ** 注意~和字体间不能有空格 **.

### 列表
+列表1

    前后空行+tab,实现段落的效果. 
     
+列表2

### 链接
Markdown 链接支持

1.**内敛式**

    [UX 平台链接](http://ux.etao.com)。
    
2.**引用式**
    文档其它地方定义好 链接常量，直接引用即可，同时放括号内不区分大小写，如：
	
UX 平台最新的技术博客分别是 [KISSY RichBase 使用][1]，[Mix网站低调上线][2] 和 [清空当前页面的用户体验][c]。

[1]: http://ux.etao.com/posts/613  "KISSY RichBase 使用"
[2]: http://ux.etao.com/posts/598  "Mix网站低调上线"
[C]: http://ux.etao.com/posts/580  "清空当前页面的用户体验"

## Markdown 常见问题
------------------

###Markdown tilda '`' 符号转义. 

经常在写文档的时候会遇到需要输入代码, 然后代码里边有 tilda `` ` `` 符号, 如果你的代码是多行的话, 
建议采用4个space 或者 一个 tab 来进行注释代码片段, 不要采用 ``` `` ``` 来标准代码. 
如果只有一行的的话, 可以采用如下方式在需要转义的`` ` ``进行**转义**:

	To show the previous example explanation in an inline codeblock: `` `table_name` `` 
	
输出的效果如下: 
To show the previous example explanation in an inline codeblock: `` `table_name` ``

### Markdown需要转义的字符

\\   backslash

`   backtick

\*   asterisk

_   underscore

{}  curly braces

[]  square brackets

()  parentheses

\#   hash mark

\+   plus sign

\-   minus sign (hyphen)   
.   dot

!   exclamation mark

