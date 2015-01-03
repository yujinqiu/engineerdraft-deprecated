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
	
## Git checkout 远程的一个分支
### 背景
在多人协作的开发场景下, Dev B 提交了一个分支, 需要拉到本地.   
	
	git fetch origin 
	git branch -v -a 
	git checkout -b test origin/test
	
## Git 多人开发流程

### 如何发 PR

以下以 wiki-pages 为例

* 把项目 fork 到自己名下，然后 clone 到本地

        git clone git@git.xiaojukeji.com:yexiliang/wiki-pages.git

* 将原始项目加为上游

        git remote add upstream git@git.xiaojukeji.com:op/wiki-pages.git

* 在本地项目建立分支并切换到该分支

        git checkout -b dev

* 在 dev 分支上开发，提交

        touch foo.txt
        git add foo.txt
        git commit

* 切换到 master 分支并合并上游

        git checkout master
        git fetch upstream
        git merge upstream/master

* 切换到 dev 分支，合并 master，推送代码

        git checkout dev
        git rebase master
        git push

* 到 github / gitlab 选 dev 分支发 pull request


## Git conflict 如何保持一方修改
### 背景
在多人协作的情况下, 经常会出现冲突, 有时候不想 merge conflict, 直接使用双方的修改内容.     
####  保持本地修改, 放弃远程版本

    git checkout --ours filename.c
    
 
#### 使用远程版本, 放弃本地修改    

    git checkout --theirs filename.c
    
## Git 回滚到上一个版本
作为项目的负责人, 通过 code review 并不能保证其中提交内容ok, 如果出现异常, 需要仅限回滚. 

	branch1    branch2
	    \        /
	     \      /
	      \    /
	       \  /
	       new branch (hashcode)
	      
假设用户提交的 branch2  有问题, 需要回滚回 branch1

	git  revert  -m 1  hashcode
	
如何得到 `-m` 之后对应的 branch  是  1 ? 2 ? 

	/go/src/ServiceTree$ git show af24b57a530adcf24d098676809bc88939336063  
	commit af24b57a530adcf24d098676809bc88939336063  
	Merge: 609bb8e d160c3f   #注意这里
	Date:   Sat Dec 27 11:30:37 2014 +0800   

    Merge branch 'dev' of ssh://git.xiaojukeji.com:22/op/servicetree into dev

从 Merge 中可以选择 `-m` 对应的选项. 