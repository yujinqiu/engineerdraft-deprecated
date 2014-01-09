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
涉及文件: `/etc/login.defs`

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
