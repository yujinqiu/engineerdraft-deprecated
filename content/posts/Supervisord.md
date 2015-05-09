+++
date = "2015-03-25T17:17:50+08:00"
draft = false
title = "Supervisord"

+++


### supervisord 
#### 背景  
一个模块在运行的过程中可能遇到各种问题, 会导致它 crash, 如果 crash 之后, op 通常希望它能马上进行重启. supervisord 模块就是这个用途,  它在实时监测子进程模块的运行状态, 如果发现模块故障, 就会重新产生一个新实例.   

#### supervisord VS supervise   
两个模块做的事情类似. 采用supervisord 的好处

1. 开源广泛运用, 资料较多, 且具有源代码,可控性更好  
2. supervisord 可以管理多个自己进程, 避免 supervise 需要 copy-paste 多份二进制程序.  
3. 支持 xml-rpc, 方便远程控制     

#### 资料   
[supervisord](http://supervisord.org)    

#### 安装
已经将 supervisord pack 到 yum server 中, 安装如下: 

    yum install  -y  python-supervisord

#### 服务启动
安装完成之后, 系统默认是自动启动的. 目前我们已经将` python-supervisord` 嵌入到系统 service 内部, 可以使用大家熟悉的` service` 来操作(正常情况下不需要, 重启可能会导致子服务都重启, 谨慎操作)

    service supervisord restart   
    
#### 随机启动  
我们希望机器重启之后, `supervisord` 之后能自动启动, 最近` python-supervisord` 已经能够支持.  

#### 随机启动细节(不敢兴趣同学跳过)
主要步骤如下:  
1: 创建随机启动脚本[supervisord](https://git.xiaojukeji.com/op/supervisord-pack/blob/master/etc/init.d/supervisord)    
2: chkconfig --add supervisord  
3: chkconfig --level 2345 supervisord on   

将 步骤1, 随机启动脚本放到 `/etc/init.d/` 下面  
将 步骤2,3 放入到安装后置脚本[supervisord-after-install.sh](https://git.xiaojukeji.com/op/supervisord-pack/blob/master/supervisord-after-install.sh) 内  

[随机启动reference](http://unix.stackexchange.com/questions/20357/how-can-i-make-a-script-in-etc-init-d-start-at-boot)  
[打包具体命令](https://git.xiaojukeji.com/op/supervisord-pack/tree/master)

#### 配置文件规范  
supervisord 遵守 Unix 规范.    

     /etc/supervisord.conf
     /etc/supervisord.d/

**其中各个模块的配置文件放在 /etc/supervisord.d/, 如果单个模块配置文件较多, 建议大家在/etc/supervisord.d/创建子目录**    

agent 为例:   

     [program:odin_agent]
     command=odin_agent -addr=0.0.0.0:789 -mode=server -account="root"
     directory=/
     autostart=true
     startsecs=10
     autorestart=true
     startretries=8640
     user=root
     
     
### 打包时候权衡(重要)
在打包的时候, 我们考虑在 upgrade 的时候, 是否需要自动重启.   
考虑到如果自动重启的话, 会导致下面的子服务都全部重启, 这样会部分业务是不能接受的.   
从业务影响和 supervisord 升级的频率(基本上不会升级)考虑, 我们目前的策略是 `yum upgrade` 只会替换 bin 程序, 不进行重启.   
重启生效操作需要人工 `service supervisord restart` 来触发.     
`yum remove` 的时候, 也需要人工进行 `service supervisord stop` 来进行停止.


### FAQ 
#### 为什么 odin agent 启动不了 ?  
追查多次这个问题, 基本上大家在使用的时候直接 copy  `odin_agent.conf`, 然后**忘记修改第一行`program:odin_agent` 这个内容, 这个不能和 odin_agent 冲突, 否则会启动不了.   
#### 为什么使用 supervisorctl 发现提示 backoff,  不断重启?  
首先要明白 supervisor 的工作原理和适用场景. 工作原理:通过父进程 fork 出新子进程,  如果子进程退出重新 fork, 这样保证子进程挂掉可以重新拉起, 因此 supervisor **只能用于管理非 daemon 程序**,  daemon 程序在启动之后会从父进程 detatch 出来, 被 init 进程接管, 对于 php-fpm 正常情况下是不太适合的, 使用 `php-fpm -F` 来制定以非 daemon 形式启动

#### supervisord 如何接管 daemon 程序? 
对于 php-fpm/mysql 等启动之后就变成 daemon 程序, 如果使用 supervisord 管理 ?  

    [program:mysql]
    command=/path/to/pidproxy /path/to/pidfile /path/to/mysqld_safe
    
注意不能加上 `autostart=true, autorestart=true`, 原理如上面所讲. 那么这样做的好处就是: 和没有 supervisor 之前对比, 额外增加了可以集中管理所有的程序. 就是是 php-fpm, mysql 这类的重启也可以通过  
    
    supervisorctl restart app 来进行处理.
    
至于 php-fpm 的子进程还是由 fpm 来进行管理. 


### Supervisord reload 配置

    supervisorctl reread
    supervisorctl update

