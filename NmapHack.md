本文的切入点:hacker使用Nmap的流程角度来介绍如何掌握Nmap这个强大的工具,帮助安全管理员发现自己网站的漏洞, 并逐步加固自己的安全. 
##nmap 简单介绍
nmap(network mapper) 主要用途  
- 网络发现  
- 安全扫描 
- 端口扫描 
- 网络审查  
### nmap 命令选项  

	# nmap [-s (Scan Type)] [Options] {target specification}
    

## Let's start hack

### 确定网络的主机
hacker使用Nmap来扫描整个网络。  
	#nmap -sP 192.168.1.0/24  
nmap 做了什么事情?   
	-sP 表示ping scanning, nmap 发送ICMP echo request 到每个机器, 返回echo response的机器是存活状态. 但是有些站点会屏蔽echo request , 因此nmap 会发送一个TCP ack packet 到端口80 (by default), 如果获取到RST, 那么也可以确定是
    
### 端口扫描
确定在子网内的机器ip之后就可以针对某些特定的机器进行端口扫描.
1: 使用TCP连接进行扫描
	#nmap -sT 192.168.1.2  
nmap 做了什么事情?   
	本机会利用connect() 和远程进行进行创建连接, 如果远程的机器端口是Listened, connect() 会成功  
    
优点: 不需要root权限就能够进行
缺点: 该扫描很容易被远程机器探测到, 远程的机器会记录创建连接和错误日志. 

2: 隐蔽扫描(stealth scanning)
使用TCP SYN扫描来解决上面的问题, 这种方式通常叫"half-open scanning".  
	#nmap -sS 192.168.1.2
nmap 做了什么事情?
	本机发送SYN, 如果端口listen那么就返回 ACK|SYN,本机收到之后返回RST来断开连接.  如果返回RST那么表示该端口没有listen. 
优点: half-open scanning 相对来说比较难被发现在扫描
缺点:执行该操作需要root 权限, 因为需要构建SYN包. 



    
 
