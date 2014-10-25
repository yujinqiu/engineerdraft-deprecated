## repeat
zsh 支持一个叫` repeat` 的命令, 就像这个名字的含义, 他就是类似 shell 的 `for` 命令. 

使用方式很简单:

	repeat N  command
	
```bash
	repeat  3  ls
```
	
## 快速输入上个命令的basename  


An application of modifiers is `!:t`, which results into the basename of the last argument. Very useful when working with URLs, for example. You’ll never have to strip the path manually again:

% wget ftp://ruby-lang.org/pub/ruby/1.8/ruby-1.8.7-p330.tar.gz
% tar xzvf !:t
