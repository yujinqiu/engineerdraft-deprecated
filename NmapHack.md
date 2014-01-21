本文的切入点:hacker使用Nmap的流程角度来介绍如何掌握Nmap这个强大的工具,帮助安全管理员发现自己网站的漏洞, 并逐步加固自己的安全.   
##nmap 简单介绍  

nmap(network mapper) 主要用途  
- 网络发现  
- 安全扫描 
- 端口扫描 
- 网络审查  

### nmap 命令选项  
	nmap [-s (Scan Type)] [Options] {target specification}
    
## Let's start hack

### 确定网络的主机
hacker使用Nmap来扫描整个网络。  

	nmap -sP 192.168.1.0/24  
    
nmap 做了什么事情?   

	-sP 表示ping scanning, nmap 发送ICMP echo request 到每个机器, 返回echo response的机器是存活状态. 但是有些站点会屏蔽echo request , 因此nmap 会发送一个TCP ack packet 到端口80 (by default), 如果获取到RST, 那么也可以确定是
    
#### 强制使用TCP ACK(PA) 或者 TCP SYN(PS) 来进行存活探测
有些机器或者firewall 会关闭**ICMP response**, 我们可以强制使用**TCP ACK** 或者**TCP SYN**来进行扫描.  

	use -PA<port1>[,port2][...].  The default port is 80, since this port is often not filtered out.  Note that this option now accepts multiple, comma-separated port numbers.

	nmap -PS 80,81 192.168.1.101
    nmap -PA 8888,8889192.158.1.101
    
批量扫描   

	nmap ip1 ip2 ip3
    nmap 192.168.1.*
    nmap 192.168.1.101,102,103
    nmap 191.168.1.101-201
    nmap 129.168.1.101-201 --exclude=192.168.1.150  
    
    
### 端口扫描
确定在子网内的机器ip之后就可以针对某些特定的机器进行端口扫描.
1: 使用TCP连接进行扫描  

	nmap -sT 192.168.1.2  
    
nmap 做了什么事情?    

	本机会利用connect() 和远程进行进行创建连接, 如果远程的机器端口是Listened, connect() 会成功  
    
优点: 不需要root权限就能够进行
缺点: 该扫描很容易被远程机器探测到, 远程的机器会记录创建连接和错误日志. 

2: 隐蔽扫描(stealth scanning)
使用TCP SYN扫描来解决上面的问题, 这种方式通常叫"half-open scanning".    

	nmap -sS 192.168.1.2
nmap 做了什么事情?  

	本机发送SYN, 如果端口listen那么就返回 ACK|SYN,本机收到之后返回RST来断开连接.  如果返回RST那么表示该端口没有listen. 
优点: half-open scanning 相对来说比较难被发现在扫描
缺点:执行该操作需要root 权限, 因为需要构建SYN包.

### 防火墙检查
检查目标机器是否在防火墙或者packet filter之后?　　

	nmap -sA 192.168.0.101
    
### 检测刚刚启动的host

	nmap -sP  192.168.1.*

### 操作系统识别(OS Fingerprinting)
hacker可能对某个os的漏洞很熟悉, 能够轻易进入该操作系统的机器.  

	nmap -O ip
    
### indent 扫描
hacker 会选择一台对某些进程存在漏洞的电脑, 比如一个root运行的webserver, 如果目标机器运行了identd(前提哦,identd是一个协议的实现) 通过 `-l`选项能够进行查看哪个用户拥有http守护进程.   

	nmap -sT -p 80 -l -O www.yourserver.com
    
 Apache运行在root下，是不安全的实践，你可以通过把/etc/indeed.conf中的**auth服务注销**来阻止ident请求，并重新启动ident。另外也可用使用ipchains或你的最常用的防火墙，在网络边界上执行防火墙规则来终止ident请求，这可以阻止来路不明的人探测你的网站用户拥有哪些进程。  
 
 ###快速扫描
 只进行nmap-services 文件中列出的服务进行检测.
 
 	nmap -F 192.168.1.101  
    
 ### TCP端口扫描
 
 	nmap -p -T:8888,80 192.168.1.101
 
### 常用选项  

	-p 指定端口 -p 21,22,23
    -v verbose mode 详细模式
 
### 检测host 服务的版本  
	
    nmap -sV www.baidu.com
    
### 小结：　　
使用什么样的方法来抵制一个黑客使用Nmap，这样的工具是有的，比如 Scanlogd, Courtney, and Shadow;，然而使用这样的工具并不能代替网络安全管理员。因为扫描只是攻击的前期准备，站点使用它只可以进行严密的监视。 使用Nmap监视自己的站点，系统和网络管理员能发现潜在入侵者对你的系统的探测。
