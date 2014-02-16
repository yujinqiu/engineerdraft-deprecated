#命令行控
-------------------------
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


