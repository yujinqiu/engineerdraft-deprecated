+++
date = "2014-11-19T21:36:12+08:00"
draft = false
title = "CliDraft"
description = "命令行相关 trick & skill"
+++

## 命令行控

### ubuntu (命令行)

#### moc(music on console)
安装命令 `sudo apt-get -y install  moc`  
#### 特点
mod 很类似 tmux 能够attach(输入命令`mocp`) 和 detach(`q`)

特点:  
1. >  音量increase  < 音量 decrease  
2. `h` for help  
3. `Q` for quit


### MAC
#### afplay(Audio File Play)

很简单的命令, 帮助文档就一个选项`-h`

    {-t | --time} TIME	
        play for TIME seconds
    {-r | --rate} RATE
        play at playback rate
    {-q | --rQuality} QUALITY
        set the quality used for rate-scaled playback (default is 0 - low quality, 1 - high quality)

<!--more-->
   
### 简单流式计算

#### 背景

经常我们在追查线上问题的时候, 希望能够对线上的日志进行实时的统计输出?  
假设线上的日志文件名为a.log 格式为  

    NOTICE: 09-27 09:51:00: abc  
    NOTICE: 09-27 09:51:00: cde  
    NOTICE: 09-27 09:51:01: fgh  
    NOTICE: 09-27 09:51:01: ijk  
    NOTICE: 09-27 09:51:03: lmn  
    
我们希望实时统计输出每秒的日志数目. 
#### 解决命令

    tail -f a.log | cut -c 0-22 | uniq -c 
   
输出结果类似如下: 

    271 NOTICE: 09-27 09:59:54
    309 NOTICE: 09-27 09:59:55
    306 NOTICE: 09-27 09:59:56:
    
### 文件分割
#### 背景  
通常我们为了加快处理速度, 会对数据文件进行拆分, 然后同时运行多份相同的程序 load 不同的数据文件进行简单的并行处理. **比较有用的拆分是根据文件的行数进行**
#### 解决命令
    split [OPTION]... [INPUT [PREFIX]]

    split -l 100 -d -a 2 DATA  PREFIX_
    
#### 常用选项
    -l 按照行数拆分
    -d 拆分文件后缀以数字进行, 默认是字母
    -a NUM  拆分文件后缀名的长度, 默认2
    
### screen 非交互运行  
#### 背景  
有时候我们为了批量后台执行一些程序, 同时避免退出之后, 不会被杀伤, 会采用 screen, nohup等手段. 其中 screen 默认情况下是 ** attach ** 模式, 需要 `ctrl + a , d` 之后才可以 ** detach ** 出来, 其实是可以直接以 detach 模式后台运行的. 

### 解决命令

        screen -d -m -S NAME  sleep 10 
      
### 缺点  
程序执行退出之后, session 会自动消失, 建议输出写入到日志中. 
    
### autojump
> 有了 autojump 妈妈再也不用当心我手指痛了.

看了网上很多关于 autojump 的文章发现大部分都太过时了, 决定自己写一篇.
#### 背景
作为运维人员是不是经常各种 cd, 时间久了感觉手指心都觉得有点痛, 这时候你想要 autojump 来帮你解决这个问题. 
> autojump - a faster way to navigate your filesystem   

#### 应用场景  
1. 快速进入一个目录,autojump 会记录和计算各个目录权重, 通过 `j --stat` 来查看
    
        j foo
2. 快速进入当前目录下的一个子目录 

        jc images
     
3. 直接用文件管理器( mac os x Finder)打开对应目录

        jo images

4. 支持用文件管理( mac os x Finder)进入当前目录的子目录

        jco images
  
 
#### 常用选项  
1. -s, --stat 查看前100条目录的权重
2. -i, --increase VALUE 增加当前目录的权重, 可以制定自己增加权重VALUE多少.
3. -d, --decreate VALUE 减少当前目录的权重, 可以制定自己减少权重VALUE 多少. 
4. --purge 删除一些目录已经被删除但是仍然保留在数据库中.

#### 高级功能
1. ZSH 支持 Tab 补全  
    autojump ZSH 需要**compinit**模块来支持, 配置方法:把下面内容加入 ~/.zshrc  
    
            autoload -U compinit && compinit

### PIPE 命令查看出错状态
经常我们会执行一串命令, 管道连接, 可能有某个命令出错, 需要知道哪个环节出错.   

	cat remote.py  | grep close | awk '{print $1}'
	
	echo ${PIPESTATS[*]}
	0 0 0

### telnet 正常退出姿势
说实话, 不太喜欢 `telnet`, 个人还是比较希望强大的 `nc` , 发现自己居然不会退出. 万能的 google  

	ctrl + ]
	quit
	

### nc 
#### 背景
nc 和 telnet 默认是交互式, 可是我们经常需要非交互, 其实解决方案很简单   

	echo "cmd "  | nc  host  port
### date
#### 背景
我们经常需要在 timestamp 和 datetime 之间进行转换  
获取当前 timestamp

    date +%s
    root@host:~# date +%s
    1420952676
timestamp => datetime  

    date -d @ts
    date -d @1420952676
    root@host:~# date -d @1420952676
    Sun Jan 11 13:04:36 CST 2015

### crontab 
#### 新增 crontab   
经常我们需要在 crontab 中新增一个记录, 快捷的解决方案是:  

    (crontab -l ; echo '*/1 * * * * CMD') | crontab -
