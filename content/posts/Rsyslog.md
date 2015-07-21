+++
date = "2015-06-22T23:18:49+08:00"
draft = false
title = "Rsyslog Draft"

+++

##Rsyslog
### Rsyslog 和 Syslog 关系  
In Red Hat Enterprise Linux 3/4/5, the default system log tool is syslogd which is provided by package sysklogd, but since Red Hat Enterprise Linux 6, the rsyslogd became the default. [refer](https://access.redhat.com/solutions/36328)  
简单的说可以理解为 Rsyslog 是 Syslog 的超集, 从RHEL 6 开始系统默认替换了.   

### Why Rsyslog
rsyslog 是负责收集 syslog 的程序，可以用来取代 syslogd 或 syslog-ng。
在这些 syslog 处理程序中，个人认为 rsyslog 是功能最为强大的。
其特性包括： 
支持输出日志到各种数据库，如 MySQL，PostgreSQL，MongoDB，ElasticSearch，等等；   
通过 **RELP + TCP 实现数据的可靠传输**（基于此结合丰富的过滤条件可以建立一种 可靠的数据传输通道供其他应用来使用）；   
精细的输出格式控制以及对消息的强大 过滤能力；  
高精度时间戳；队列操作（内存，磁盘以及混合模式等）； 支持数据的加密和压缩传输等。

#### Rsyslog 查看版本  

    rsyslogd -version

#### Rsyslog 配置解释     
Rsyslog 配置主要分为几个部分   

1. Modules  
2. GLOBAL DIRECTIVES 
3. RULES  

#### MODULES ####
模块相关的配置指定功能仅在 相应的模块被加载（$ModLoad）后才可用。  
配置行如果太长，可以在行尾使用 “\” 分割成多行。注释支持两种语法，一种以 # 开始到行尾，另一种为 C 语言格式。   

```
$ModLoad imuxsock.so    # provides support for local system logging (e.g. via logger command)
$ModLoad imklog.so      # provides kernel logging support (previously done by rklogd)


安装对应的 module.   
```

```
*.*                    @@10.1.5.241:10514    #通过tcp传
*.*                    @10.1.5.241:514       #通过udp传
```

其中 im 表示输入模块,  常见模块     
1. imfile 转换文件文件为日志  
2. imrelp - 通过 RELP 可靠的接收日志。
3. imklog - 获取内核日志（通过 dmesg 也能看到相关信息）

输出模块(om 前缀)
1. omprog - 发送日志给程序处理
2. ommail - 发送邮件
3. ommysql - MySQL  



####  GLOBAL DIRECTIVES
场景全局指令   

1. $ActionFileDefaultTemplate [templateName] # 定义文件动作的日志输出模板

    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat 输出的日志格式如下:  
    Dec 30 17:16:58 linux-64 logger[16898]: test syslog 
    
    $ActionFileDefaultTemplate RSYSLOG_FileFormat
    2013-12-30T17:19:50.926770+08:00 linux-64 logger[7757]: test syslog #显然这个格式是比较好的   
    
2. $IncludeConfig # 导入其他文件到配置，等价于 C 中的 #include
3. $MainMsgQueueSize # 主消息队列大小   
4. $DynaFileCacheSize # 控制动态文件保持打开的个数，针对每个动作
5. 重复消息控制

    $RepeatedMsgReduction on  
    $RepeatedMsgContainsOrigionalMsg on
    
6. 日志接收率控制

Interval 设置率计算的时间间隔，0 表示关闭；Burst 设置该间隔内允许的日志数。 IMUXSock 针对通过 system log socket 接收的日志；SystemLog 针对其他输入源。   
```
$SystemLogRateLimitInterval 0     
$SystemLogRateLimitBurst 0     
$IMUXSockRateLimitInterval 0     
$IMUXSockRateLimitBurst 0     
``` 
Dec 31 22:02:36 linux-64 rsyslogd-2039: imuxsock begins to drop messages from pid 6927 due to rate-limiting
Dec 31 22:02:39 linux-64 rsyslogd-2039: imuxsock lost 133250 messages from pid 6927 due to rate-limiting

#### Rules
Rules =  Filter + Action 组成  

    cron.*(filter)  /var/log/cron(action)
   
##### Filter
Filter 有以下几种   
1. 基于设施/优先级的过滤器  

    格式如下<FACILITY>.<PRIORITY>
 表示生成日志的子系统。取值范围为 auth，authpriv，cron， daemon，kern，lpr，mail，news，syslog，user，uucp，local0 到 local7， 参考 man 3 syslog。其对应的数值请参考 /usr/include/sys/syslog.h， 注意需要忽略移位操作。   
表示日志的级别，取值范围为（从低到高，数值对应 7-0） debug，info，notice，warning，err，crit，alert，emerg。 参考 man 3 syslog。   
在级别前可以增加相应的修饰符，例如加 = 表示仅选择该优先级的日志，加 ! 表示选择不等于优先级的所有日志，不加任何符号则表示选择该优先级及之上的日志。 * 可以用来表示所有的日志子系统和/或消息级别。关键字 none 表示未指定级别的日志。如果要定义多个设置/优先级，使用 , 分隔即可。 如果要定义多个过滤条件，则使用 ; 分隔。  
示例如下：   
 kern.*                 # 选择所有级别的内核日志
 mail.crit              # 选择所有级别为 crit 及之上的邮件系统相关日志
 cron.!info,!debug      # 选择所有 cron 日志信息，排除优先级为 info 和 debug 的日志
 *.=debug               # 选择所有的调试级别日志
 *.*;auth,authpriv.none # 选择所有级别的日志，**以及认证相关无级别的日志**

    
 2. 基于属性的过滤器
 3. 基于 RainerScript 
     
         if <EXPRESSION> then <ACTION>
 ##### Action 动作 
1. 保存到文件    
cron.* -/var/log/cron.log
如果文件路径前有 “-” 则表示每次输出日志时不同步（fsync）指定日志文件。 文件路径既可以是静态文件也可以是动态文件。动态文件由模板前加 ? 定义。
2. 通过网络发送日志
格式如下：
@[(<options>)]<HOST>:[<PORT>]
@ 表示使用 UDP 协议。@@ 表示使用 TCP 协议。<options> 可以为： z<NUMBER> 表示使用 zlib 压缩，NUMBER 表示压缩级别。多个选项 使用 , 分隔。
例如：
 *.* @192.168.0.1     # 使用 UDP 发送日志到 192.168.0.1
 *.* @@example.com:18 # 使用 TCP 发送到 "example.com" 的 18 端口
 *.* @(z9)[2001::1]   # 使用 UDP 发送消息到 2001::1，启用 zlib 9 级压缩
3. 丢弃日志  
4. cron.* ~
丢弃所有信息，即该配置之后的动作不会看到该日志。
随 rsyslog 版本不同，如果有如下警告信息，则将 ~ 修改为 stop。

        warning: ~ action is deprecated, consider using the 'stop' statement instead [try http://www.rsyslog.com/e/2307 ]
        
对每一个过滤条件，可以指定多个动作，每个动作一行即可。这种情况下，也可以通过 “&” 来表示上一行的过滤体检。例如：
:msg,contains,"error" liuzx
& @192.168.1.1
包含 “error” 日志被同时发送给用户 liuzx 和通过 UDP 发送到 192.168.1.1。



#### 测试方法  

    logger -p local5.info  "hello world"          # 这里必须指定local5级别
    
    
#### 检查 rsyslog 配置文件有效性   

    rsyslogd -f /etc/rsyslog.conf -N1
    -N  level
              Do a coNfig check. Do NOT run in regular mode, just check configuration file correctness.  This option is meant to verify a config file. To do so,
              run rsyslogd interactively in foreground, specifying -f <config-file> and -N level.   