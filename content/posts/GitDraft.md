+++
date = "2014-11-25T21:36:12+08:00"
draft = false
title = "Git Draft"
description = "Git 相关文档和技巧"
+++

# Git  
## 原子提交  

在我们修改文件的时候, 通常是希望一次提交只包含该次功能修改相关的操作.   
但是如果一个文件包含多个操作该怎么办 ?   
或者你的一个程序文件里边,为了调试写了很多的log日志输出, 你不希望这些语句ci 到server上, 该怎么办 ?   
  
`git add -p` 能够解决上面的场景(虽然已经add 过,还是需要add 到 stage 进去).   
执行之后会提示: 

    Stage this hunk [y,n,q,a,d,/,e,?]?
    
hit `s` to split whatever change into smaller chunks. This only works if there is at least one unchanged line in the "middle" of the hunk, which is where hunk will be split  
then hit `y` to stage that chunk  
or `n` to not stage that hunk  
or `e` to manually edit the chunk (useful when git can't split it automatically)  
and `d` to exit or go to the next file.  
Use `?` to get the whole list of available options.

<!--more-->


## Git checkout 某个分支
### 背景  
在使用 git 的时候, 经常会使用第三方开发的程序/lib 的特定版本(兼容性问题) ,  

	git tag -l
	git checkout tags/<tag_name> 