# 系统相关
## 僵尸进程和孤儿进程
### 什么是僵尸进程
当子进程比父进程先结束,而父进程又没有回收子进程,释放子进程的资源,此时子进程将成为一个`僵尸进程`.  
如果父进程先退出, 此时子进程会被init进程接管, 子进程退出后会被`init`进程回收其占用资源.   
### 查看僵尸进程
使用`ps -elf | awk '$2~/Z/{print}'`  
	0 Z root     28212 28201  0  85   0 -     0 exit   Sep25 ?        00:00:00 [mysql] <defunct>
### 
### 怎样清理僵尸进程
1. 改写父进程, 在子进程死之后需要父进程为它进行收尸. 具体做法: 接管SIGCHILD信号. 子进程死后会发送SIGCHILD信号,父进程接收到此信号之后, 执行`waitpid()`函数为子进程收尸.  
2. 把父进程杀掉, 父进程被杀掉后,僵尸进程成为`孤儿进程`被init进程接管, init 会负责清理僵尸进程.  
### 僵尸进程的的危害
当用`ps`命令观察进程的执行状态,经常看到某些进程的状态为`defunct`	. "僵尸"进程虽然是早已经死亡的进程, 当时在进程表(process table) 中仍然占用一个位置(slot).由于进程表的容量是有限的, `所以defunct进程不仅占用系统的内存资源, 影响系统的性能, 而且影响系统的性能,而且如果其中数目太多,可能导致系统瘫痪.`

## linux 如何查看某个分区的大小
如果一个分区已经mount到文件系统中, 直接采用`df -h`就能够方便进行查看, 对于已经未mount 到系统中的如何进行查看 ?  
1.`fdisk -l /dev/sda` 或者 `fdisk -l /dev/sda{1,2,3}` 具体sdax 由系统分区数目决定.  
2.`partx /dev/sda` 强烈推荐该方式,输出的效果  

	[root ~]# partx /dev/sda  
    1:      2048-  1953791 (  1951744 sectors,    999 MB)  
    2:   1953792- 41015295 ( 39061504 sectors,  19999 MB)  
    3:  41015296- 80078847 ( 39063552 sectors,  20000 MB)  
    4:  80078848-119140351 ( 39061504 sectors,  19999 MB)  
    5: 119140352-138672127 ( 19531776 sectors,  10000 MB)  
    6: 138672128--207087617 (-345759744 sectors, 2021994 MB)
    
## linux 密码失效
同事反馈,有一台机器"You are required to change your password immediately (password aged)＂, 知道密码会有时效性,google 一下学习了RHEL 密码设置.    
涉及文件:   
1. `/etc/login.defs` 定义用户密码的相关范围常量. 

	# Password aging controls:
    #
    #       PASS_MAX_DAYS   Maximum number of days a password may be used.
    #       PASS_MIN_DAYS   Minimum number of days allowed between password changes.
    #       PASS_MIN_LEN    Minimum acceptable password length.
    #       PASS_WARN_AGE   Number of days warning given before a password expires.
    #
    PASS_MAX_DAYS   99999
    PASS_MIN_DAYS   0
    PASS_MIN_LEN    5
    PASS_WARN_AGE   7  
2. `/etc/shadow` 定义用户当前的设置, 其中关于密码是时效性在第**5**列,如下所示: 
    foo:$1$yGUcqOSx$5rff5VcK3auiMS8M3WpcX/:15988:7:99999:7:::
    
涉及命令: `chage` 其中 -l ${user}可以查看当前用户设置.  -M 可以调整最大失效时间,如果需要永久不失效 -M 99999


## curl 设置代理

### 问题背景

在GFW的"庇护"下, 有时候为了访问墙外的资源, 需要利用`brew install ` 来安装对应的软件, 其中`brew install ` 第一步是下载对应的软件, 有时候由于被GFW 墙需要通过`goagent`等代理来访问. 常见的curl 代理模式如下: 

     -x, --proxy <[protocol://][user:password@]proxyhost[:port]>
              Use the specified HTTP proxy. If the port number is not specified, it is assumed at port 1080.

              This  option  overrides  existing  environment  variables that set the proxy to use. If there's an environment variable setting a proxy, you can set
              proxy to "" to override it.

              All operations that are performed over an HTTP proxy will transparently be converted to HTTP. It means that  certain  protocol  specific  operations
              might not be available. This is not the case if you can tunnel through the proxy, as one with the -p, --proxytunnel option.

              User  and  password  that  might be provided in the proxy string are URL decoded by curl. This allows you to pass in special characters such as @ by
              using %40 or pass in a colon with %3a.

              The proxy host can be specified the exact same way as the proxy environment variables, including the protocol prefix (http://) and the embedded user
              + password.


  
这种模式无法满足`brew install` 的方式, 因为我们不可能修改`brew install`的行为, 因此需要采用类似环境变量的形式.  

    export http_proxy=http://your.proxy.server:port/
    如果是 HTTPS 需要配置 HTTPS_PROXY, 注意是大写哦. 
    

## 如何删除乱码文件/目录  

背景: 使用linux 命令行中断经常会不小心创建一个乱码文件或者目录, 使用`rm -rf ${乱码名`,会找不到.   
思路: linux 底层本质上是使用 inode 来进行标记, 因此可以采用inode 来进行标记. 
操作:　

	ls -i * # get file inode num
    find . -inum ${inode} -exec rm -rf -- {} \; # 注意其中 -- 来避免shell 转义. 


## CentOS 7
### 无法重启 sshd
#### 背景
在黑色星期五时候, 花费了5$开通了 linode 账号, 随着 centos 7 发布, 随便体验的一番. 在进行 ssh 远程端口转发的时候发现遇到一个问题,通过` service restart ssh` 居然不行   

	 Failed to issue method call: Unit ssh.service failed to load: No such file or directory.
	 
#### 原因  
Cent OS 利用 `systemctl` 替代了 `service`,  第一次操作需要进行以下操作初始化   
	
	systemctl enable sshd.service
	systemctl start sshd.service 
	systemctl status sshd.service

## 故意生成 coredump  
最近在研究如何限制 linux coredump, 主要解决问题  

1. 避免多线程程序同时大量 coredump  
2. 避免大内存程序 coredump   
其中为了方便测试, 研究如何强制 coredump ,  方法其实很简单   
其中 `python` 进入交互模式, 然后 `ctrl + \` 强制进行 coredump    
### 原理  
`ctrl + \` 在 linux 平台上会生成 QUIT signal , 通常会导致改程序退出或者 coredump  
这个是 *ninx 平台的特点, 和python 没有关系, 你也可以 `sleep 30` 然后 `ctrl + \` 强制进行 coredump . 


## 疑难杂症  
### 背景   
今天同事反馈, **tail** 一个文件的时候 提示:   no space left on device   
然后第一反应是 `df` 查看磁盘空间:  

    Filesystem            Size  Used Avail Use% Mounted on
    /dev/sda1             9.9G  2.1G  7.4G  22% /
    /dev/sda3              20G 1009M   18G   6% /usr/local
    tmpfs                  32G   16K   32G   1% /dev/shm
    /dev/sda4             794G  244G  511G  33% /home
    
没有看到任何空间不足的提示, 难道是 inode 节点空间不足?   `df -i` 发现也不是.     
search 之后发现原来是 kernel 2.6.13 开始引入 Inotify 导致.  需要修改 **max_user_watches**   

    查看: sysctl fs.inotify.max_user_watches   
    修改: sysctl fs.inotify.max_user_watches = 16384
    
     /proc/sys/fs/inotify/max_user_watches 表示:  
     This specifies an upper limit on the number of watches that can be created per real user ID.
     
     
### Linux 目录权限????
#### 背景 
今天在大规模升级Agent 到 2k+ 机器过程中, 发现一个比较诡异的问题.  追查之后发现有些机器的权限很诡异的权限 

    -????????? ? ?      ?             ?            ? tcollector.log
如果删除提示

    rm: cannot remove `tcollector.log': Input/output error

查看`/var/log/message` 提示错误   

    May  9 12:11:30 fd-sec-siem00 kernel: [2486652.261872] EXT3-fs error (device sda1): ext3_lookup: deleted inode referenced: 542728
    
因此确定是文件系统故障, 对于根分区无法 umount, 因此可以通过下面方式, 然后重启, 这样系统启动之后会自动 fsck.  


    touch /forcefsck

### XFS 文件系统格式化命令 

    mkfs.xfs -f -L /data1 -d agcount=64 -l size=128m,lazy-count=1,version=2 /dev/hioa
    
#### 测试性能 dd 命令 
I
    dd if=/dev/zero of=/data1/test.dbf bs=8k count=300000 conv=fdatasync
    注意要记得fdatasync 
    
    
### Linux 日志切分logrotate  
#### 背景 
日志可以很方便帮助大家追查各种问题, 其中日志切分需要综合考虑磁盘使用率, 自动清除历史.  线上开始一直使用op 自己开发的库.    
在使用 agent 的时候, 再想应该使用系统的 logrotate.  

```
logrotate - rotates, compresses, and mails system logs  
```
经常学习之后, 采用如下配置:   


```
/home//influxdb/log/influxdb.log {
    daily
    rotate 3
    compress
    missingok
    notifempty
    copytruncate
    create 0644 influxdb influxdb
    dateext
    dateformat .%Y-%m-%d
}
```

#### 注意事项  
1: 默认情况下 log rotate 只能支持最细粒度的` daily` 级别切分, 不太满足运维上小时级别的要求     
2: 比较有意思的是 `copytruncate` 能够解决, 一些写的不够友好的模块, 不会检测文件名改变, 重新打开句柄的问题. 
3: dateext 和 dataformat 一起使用来决定压缩的文件后缀名   
4: rotate 表示保留的份数  

####  reference   
[logrotate](http://manpages.ubuntu.com/manpages/hardy/man8/logrotate.8.html)   
[每小时日志切分](https://packetcloud.net/index.php/making-logrotate-rotate-apache-logs-every-hour-or-2-or-3/)   

### 系统压力测试工具  
#### 背景   
在开发监控系统的 agent 过程中, 为了构造各种压力情况, 比如cpu idle < 50 触发报警等, 因此我们特别需要一个工具来实现这个功能. 开始找了一圈发现大家使用的都是 ab 之类的 http 压测工具.  也有同学建议在本机开启一个 http 服务, 然后利用 ab 之类工具来压测, 总感觉不够简单优雅, 直到遇见` stress` 工具, 直到后面遇见 `stress-ng`. 这两个工具同样可以用来压力测试用, 因此特地记录一下.     
##### stress 和 stress-ng  
简单说 stress-ng 是 stress 的升级版本,  在 mac 下面 brew 默认有 stress, 没有 stress-ng  
stress-ng 主要功能:   
1. CPU compute
2.  Cache thrashing  
3.  Drive stress
4.  I/O syncs  
5.  VM stress
6.  Socket stressing
7.  Context switching  
8.  Process creation and termination  
9.  It includes over 60 different stress tests, over 50 CPU specific stress tests that exercise floating point, integer, bit manipulation and control flow, over 20 virtual memory stress tests.

建议在 root 权限下执行, 避免 OOM kill 和其他权限不足的错误.    


##### stress 简单用法  
[下载地址](http://people.seas.harvard.edu/~apw/stress/)  
 
    stress -c 2 -i 1 -m 1 --vm-bytes 128M -t 10s
    
    -c 2 : Spawn two workers spinning on sqrt()
    -i 1 : Spawn one worker spinning on sync() 
    -m 1 : Spawn one worker spinning on malloc()/free()
    --vm-bytes 128M : Malloc 128MB per vm worker(default 256MB)
    -t 10s: Timeout ten seconds  
    -v: Be verbose 

总体说来比较简单, 如果我们需要压测各个 core 的性能, 需要使用 stress-ng
    
    
##### stress-ng 用法 
[下载地址](http://kernel.ubuntu.com/~cking/stress-ng/)   
编译依赖    

    yum install -y libattr-devel.x86_64

打满 cpu 单核: 

    stress-ng --cpu 1 --affinity 1  
#### 插曲  
由于线上 glibc 的版本问题, 因此不能安装最新版本, 最新版本的 perf 会导致编译无法通过. 
    
#### refer   
[stress-test-linux-unix-server-with-stress-ng](http://www.cyberciti.biz/faq/stress-test-linux-unix-server-with-stress-ng/)


### Rsync 只同步目录结构, 不同步内容  


       rsync -HavP -e ssh --include='*/' --exclude='*' SRC DEST   
       
### while 读取多列文件   
#### 背景   
有时候我们需要读取一个文件的多列, 分别赋值给不同的变量, 然后对变量进行处理    

    while read col1 col2 ; 
    do
        echo $col1 $col2 
    done < FILE
    
    
### 强制 coredump  
#### 背景  
在测试监控系统的时候, 为测试 coredump 的 case, 我们需要故意 core 一个程序, 来验证监控系统能够正常报警.   
#### 解决方法   
    
    sleep  10  
    然后 ctrl + \  就会自动 coredump  sleep 程序  
    
   
### 强制 OOM kill  
#### 背景  
在测试监控系统的时候, 为测试 OOM 的 case, 我们需要让一个程序故意被系统 OOM kill, 来验证监控系统能够正常报警.     

#### 解决方法 
1. 由于 OOM 是按照一定的算法打分的, 因此为了避免 sshd 等核心进程被无辜 kill, 我们需要故意弄一个压力较大的模块, 比如利用 stress-ng 模块   
2. stress-ng 模块起压力之后   

        echo f > /proc/sysrq-trigger   

