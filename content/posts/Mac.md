+++
date = "2014-11-25T21:36:12+08:00"
draft = false
title = "Mac os X Draft"
description = "Mac os X 手稿"
+++

#Mac os X
##Yosemite 升级之后应该进行什么操作?  
1. 从 mavericks 升级到 mac os x 之后, 会遇到一些比较诡异的问题(比如找不到 clang), 下面的命令会帮助你提前避免一些问题. 

```bash
sudo xcode-select --install  
sudo xcode-select --switch /Applications/Xcode.app   
sudo xcodebuild -license  
sudo gem update --system  
sudo gem install xcodeproj   
sudo brew update   
```
<!--more-->

2. 同时 Yosemite 会收集用户的隐私数据, 建议通过[fix-macosx](https://fix-macosx.com/)进行修复


### Mac os X 窗体最小化之后恢复  
Mac os X 在最小化窗体之后, `command + tab` 是不能够直接恢复, 具体操作如下   
command + tab 定位到对应的 Application 然后 `alt` 然后同时放开 `alt command`


### 清除 DNS cache

	sudo dscacheutil -flushcache
	

### mac os x 权限位问题
在 mac os terminal 中我们经常可以看到 带有 `+`  和  `@`  的权限问题 

	-rw-r--r--@  1 knight  staff      775 Feb 26  2014 Apple.md
	-rw-r--r--   1 knight  staff     3608 Oct 25 14:23 CliDraft.md
	-rw-r--r--@  1 knight  staff     1208 Nov 16  2013 EspeakDraft.md
	-rw-r--r--@  1 knight  staff     3470 Dec 31  2013 FPDraft.md
	
#### @ 字符
`@` 表示这个文件拥有扩展的属性( additional attribute) , 可以通过`xattr` 来查看详细信息: 
	
	xattr -l  <filename> 
	
	➜  engineerdraft git:(master) ✗ xattr -l Apple.md
	com.apple.TextEncoding: utf-8;134217984
	
扩展的属性用途是: 定义一些打开文件的程序比如 Mou 打开 markdown 文件.    

#### + 字符
\+ 表示这个文件用户 ACL(Access Control List) 相对于文件的权限能够耕细粒度的控制.  

	
	ls -le 可以查看文件的 acl 权限
	
##如何快速访问Menu或者mac访问Menu的快捷键
在window的用户会有这样的习惯, 通过alt来快速访问菜单栏. 在mac 下怎么通过键盘来快速访问? 
终于在[access-the-dock-and-menu-bar-from-your-keyboard](http://lifehacker.com/321595/access-the-dock-and-menu-bar-from-your-keyboard) 找到方法.    
其实很简单就是通过 `ctrl+F2`(menubar) `ctrl+F3`(dock), 然后直接输入对应的菜单名称. 比如:`View`, 输入回车即可选择, 然后继续输入对应的名称,或者通过方向键即可选择.
打开Menu的Help的快捷键是`command + shift + /`, 可以这样记住: shift+/ = ?  快捷键就是command + ? == help
## mac 计算md5 值
在linux 下有md5sum 这个命令, 在mac 下发现居然没有. 仔细查找之后发现原来叫md5. 运行之后发现和md5sum输出的结果不一样, 其实可以利用`md5 -r`就可以得到一样的结果.


## mac 版本管理  
mac 的 keynote, pages, numbers 和原始文本编辑器是具有版本管理的. 入口在`File-> revert to -> brow all version`, 可以进入查看和 time machine 一样的界面.

## 快速打开stikcy note 快捷键
shift+ command + y ,  退出:  command + q

## 屏幕放大
ctrl + 鼠标放大,缩小 

## 快速预览文件( quick look)
### 背景
经常我们在命令行里边,看到某个文件之后, 希望能够 quick look, 一般的做法是, open File 打开, 然后关闭, 其实我们只是希望能够简单看一下, 可以利用以下命令来实现 :

	qlmanage -p  File