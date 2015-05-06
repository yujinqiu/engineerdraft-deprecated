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



## Git checkout 某个tag分支
### 背景  
在使用 git 的时候, 经常会使用第三方开发的程序/lib 的特定版本(兼容性问题) ,  

	git tag -l
	git checkout tags/<tag_name> 
	
## Git 重命名分支 

    git branch -m FROM TO
	
## Git checkout 远程的一个分支
### 背景
在多人协作的开发场景下, Dev B 提交了一个分支, 需要拉到本地.   
	
	git fetch origin 
	git branch -v -a 
	git checkout -b test origin/test   
	
## 删除远程的一个分支

    git push origin :NAME
    
    
## 删除一个已经提交的文件
### 背景
在开发的时候有可能会不小心将配置文件提交进去, 如何删除 repo 中的文件(保留 repo 的文件),  同时保证被其他地方 git pull 的时候不会删除库里边的问题.    

    echo "*.config">>.gitignore; 
    git rm --cached "*.config"; 
    git add .; 
    git commit -m "Ignoring and deleting config files." ; 
    git push origin;
	
## 本地分支和远程分支的 diff
### 背景
在多人协作的开发场景下, 我怎么知道我现在的分支和 upstream 的差异  

    git diff master..upstream/master
    
    
## Git stash
git stash save "YOUR MSG"  
git stash list  
git stash apply stash@{1}  
git stash drop 
	
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

## Git 删除远程 repo 文件, 保留本地文件
### 背景  
在项目开发过程中, 有一个策略文件, 由于经常需要变更不太适合进入 repo, 最开始的时候没有考虑清楚提交进去,  现在需要从远程 repo 删除, 但是需要保留本地文件, 因为项目运行需要依赖这个文件. 

### 解决方案  

    git rm --cached  FILE
    
## Tortoise Git 免密码 Push 问题
### 背景 
和 FE 同学进行合作, 为了提高协同工作的效率, 开始手把手教 FE 同学使用 Git, 其中为了避免 FE 同学在 git clone 和 push 的时候输入密码, 使用 Puttygen 生成对应的公钥和私钥对, 开始的时候一切都 ok.    
第二天 FE 同学反馈说需要输入密码. 考虑到 FE 同学进行重启, 将问题怀疑在ssh-agent 没有启动, search 之后, 常见的方案是:  

	eval `ssh-agent -s` 
	ssh-add  $<path_to_private_key>
	
发现没有效果.   继续 search 之后, 发现Puttygen 上面居然有一个 Pagent, 看名字应该是这个东西.  然后启动之后, 居然没有弹出窗体, 难道没有启动? 继续执行, 提示已经 running. WTF!!!. 无意间想到, 可能在 系统托盘(system tray)里边, 找到之后, 右键直接 add key 之后一切 ok. 

## Git tag 
### 背景
在使用git的过程中, 我们通常喜欢利用 tag 来进行标记, 进行发布. 
### How to

	git tag -a  v1.0.0 #会提示对应输入, 要求输入 annotation
	git tag -l -n3  #注意-n 和 3 之间不能有空格. 
	# three lines of message for every tag
	
	
## Git hook
Git hooks are event-based. When you run certain git commands, the software will check the **hooks directory** within the git repository to see if there is an associated script to run.   

Some scripts run prior to an action taking place(pre-hook), which can be used to ensure code compliance to standars, for sanity checking, or to set up an environment.  Other scripts run after an event(post-hook) in order to deploy code, re-establish correct permissions, and so forth.   

git hook 分为 client-side hook 和  server side hook.  
### client-side hook

1. Committing-Workflow hooks: commiting hooks are used to dictate actions that should be taken around when a commit is being made.  They are used to run sanity checks, pre-populate commit messages, and verify message details. You can also use this to provide notifications upon committing.

2. Email Workflow hooks: This category of hooks encompasses actions that are taken when working with emailed patches. Projects like the Linux kernel submit and review patches using an email method. These are in a similar vein as the commit hooks, but can be used by maintainers who are responsible for applying submitted code.

3. Other: Other client-side hooks include hooks that execute when merging, checking out code, rebasing, rewriting, and cleaning repos.

### server-side hook
钩子在 server 端运行  

1. Pre-receive and post-receive: These are executed on the server receiving a push to do things like check for project conformance and to deploy after a push.  
2. Update: This is like a pre-receive, but operates on a branch-by-branch basis to execute code prior to each branch being accepted.


## Git trick Skill
### 背景
在生产环境中, 可能我们会有一个密码文件, 不想让大家看到, 当时自己又记不住.     
假设文件名: passwd


    cat passed | git hash-object -w --stdin  #可以得到文件的 HASH
    git tag passwd HASH
    然后就可以删除 passwd 文件
    git cat-file blob passed
    
    
### git log

    git log FILE 查看某个人家的变化记录
    git log -S "LINE"  查看 LINE 这行出现的提交
    
    
    git reflog  其实有更好的替代方式:  git log -g 有更完整的信息.
    
    
### 好习惯  

    git  pull --rebase 
    如果出现冲突, 解决冲突
    git add CONFLICT-FILE
    git rebase --continue
    git  diff HEAD FILE  #head working space 和 HEAD 区域对比
    
    
## 有用比较生僻功能
### cherry-pick
假设我们并行开发两个版本, 其中有一个版本2.0 是稳定版本,  其中版本3.0 是在开发版本,  3.0 引入了一个比较高级的功能或者比较重要的 bug, 我们迫切希望在2.0 中能够进行添加, 但是我们又不能够把3.0 和 2.0 直接 merge, 因为3.0 中还有很多其他的 feature 是我们目前不需要的. 这时候就可以使用 cherry-pick.   
cherry-pick 的本质功能是: 对一个 commit 在当前的 branch 中 重放.

    git checkout BRANCH
    git cherry-pick HASH
    如果正常就 ok, 如果冲突, 需要手工解决冲突, 然后按照提示
    
    Automatic cherry-pick failed.  After resolving the conflicts,
    mark the corrected paths with 'git add <paths>' or 'git rm <paths>'
    and commit the result with: 

        git commit -c 15a2b6c61927e5aed6718de89ad9dafba939a90b



## Git clone 部分 repo  
### 背景 
如果你在 clone 整个 git repo 的时候, 发现这个 repo 特别大, 可以通过以下方式来进行优化:  

    git clone --depth N  URL  
   
其中 N 表示 clone 下来几个版本. 
    
