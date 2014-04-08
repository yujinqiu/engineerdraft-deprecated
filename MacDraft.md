#Mac相关技巧
##如何快速访问Menu或者mac访问Menu的快捷键
在window的用户会有这样的习惯, 通过alt来快速访问菜单栏. 在mac 下怎么通过键盘来快速访问? 
终于在[access-the-dock-and-menu-bar-from-your-keyboard](http://lifehacker.com/321595/access-the-dock-and-menu-bar-from-your-keyboard) 找到方法.    
其实很简单就是通过 `ctrl+F2`(menubar) `ctrl+F3`(dock), 然后直接输入对应的菜单名称. 比如:`View`, 输入回车即可选择, 然后继续输入对应的名称,或者通过方向键即可选择.
打开Menu的Help的快捷键是`command + shift + /`, 可以这样记住: shift+/ = ?  快捷键就是command + ? == help
## mac 计算md5 值
在linux 下有md5sum 这个命令, 在mac 下发现居然没有. 仔细查找之后发现原来叫md5. 运行之后发现和md5sum输出的结果不一样, 其实可以利用`md5 -r`就可以得到一样的结果.
