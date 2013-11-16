# Markdown
-----------------
## 核心思想
-----------------
Markdown 只关注两件事情，以真正实现易读易写的目标：

1.格式化的纯文本语法；

2.John Gruber 用 Perl 开发的脚本工具将纯文本转换成格式化的 HTML. 

** 个人认为本质上就是 You'll get what you think .核心思想: [规范指导写作,程序自动转换显示效果] ** 

## Markdown 常见问题
-----------------

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

