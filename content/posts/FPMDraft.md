+++
date = "2015-01-21T00:08:20+08:00"
draft = false
title = "FPMDraft"

+++

## FPM

### FPM 入门

    fpm -s <source type> -t <target type> [options]
    source types:  
        "dir"  可以是文件或者是目录

### 常见问题和解决思路
1.  Process failed: rpmbuild failed (exit code 1). 
    解决思路:  fpm 运行增加 `--verbose` 选项, 查看日志输出
    
2. 如何把我的一个程序, 比如 jq 打包为 RPM 格式文件, 期望安装之后安装 到 `/bin` 目录下 

    
        fpm -s dir -t rpm -n jq -v 1.4.0 --prefix=/bin/  jq
        
        
        http://golang-basic.blogspot.jp/2014/10/dynamic-programming-problem-maximum.html  
        
### fpm 制定 package 依赖

	fpm -s dir -t rpm -n langley_online -v 1.0.0 -d 'thrift >= 0.9.1' -d 'boost >= 1.41.0' -d 'libevent >= 2.0.21' -d 'fastbit >= 2.0.1' --prefix=/home/<user>   --after-install=langley_online/install/install_hook.sh langley_online
        
### 如何打包 C/C++ 项目
c/c++ 项目中比较讨厌的是项目的依赖, 为了提高开发效率需要将各种依赖包打包为 RPM, 最开始一直想不明白的是, 如果我 make && make install 之后就 pack 了, 那么如何pack呢 ?   直到今天多管闲事帮同事打包thrift之后才知道 how. 

#### How
答案其实很简单   

	make
	make install DESTDIR=<dir>
	
一般正常情况下, 会在`<dir>` 下面生成 usr 目录, 然后利用 fpm 直接进入 `<dir>` 进行打包, 指定`prefix=/` 即可.   
仔细思考之后, 其实 fpm 在使用 `-s dir` 的时候的本质, 就是pack 整个包, 因此可以推断在 `./configure --prefix=<dir>` 应该也是可以的.   

### 如何打包 python module   
Q: 我是 python 党, 用 pip 安装 python 的各种包好方便, but 线上没有访问外网权限, 怎么破?   
A: 为了安全线上只有部分机器可以访问外网, 因此除非你在内网搭建一个代理, 然后 pip 通过流量到 pip.  好消息是强大的 FPM 可以把python 的 pip 包打成 RPM.   python 的 thrift module 为例. 
    
     [root@svn02 ~]# fpm -s python -t rpm -a x86_64  thrift
     no value for epoch is set, defaulting to nil {:level=>:warn}
     no value for epoch is set, defaulting to nil {:level=>:warn}
     Created package {:path=>"python-thrift-0.9.2-1.x86_64.rpm"}

     
参数详解:   

    -s INPUT_TYPE                 the package type to use as input (gem, rpm, python, etc)  
    -t OUTPUT_TYPE                the type of package you want to create (deb, rpm, solaris, etc)
    -a, --architecture ARCHITECTURE The architecture name.
    
对应 python module, fpm 会自动下载对应的 package, 因此只需要指定 thrift 即可.     

### 查看安装 pkg 详情 
	rpm -q <pkg> 
eg: 

	root@10.223.21.205:~# rpm -q thrift
	thrift-0.9.2-1.x86_64


### yum 如何查看一个 rpm pkg 的详情  

	yum info  <pkg> 
	yum list  <pkg> 
	
### yum 制定安装特定版本  

	yum install <package name>-<version info>.<release info>.<architecture> 
eg:  

	sudo yum install httpd
	sudo yum install httpd-2.4.6-6
	sudo yum install httpd-2.4.6-6.fc20
	sudo yum install httpd-2.4.6-6.fc20.x84_64
	
### 下载 rpm 包, 然后手动进行安装

	yumdownloader --resolve <package>
	yum localinstall <path to rpm>  
	
### yum 取消安装操作  
#### 背景  
最近有一个glibc 安全漏洞, 安全组同学要求进行升级, 利用 yum 升级之后, 发现坑爹有问题, 需要进行回滚. 可是 glibc 依赖 package 太多, `yum downgrade pkg1.rpm pkg2.rpm ...` 会累死人.   
终于发现必杀技:  

	yum history #查看历史列表
	
	ID     | Command line             | Date and time    | Action(s)      | Altered
	-------------------------------------------------------------------------------
	120 | install glibc            | 2014-08-26 09:19 | Install        |    5   
	yum history  undo  120 


### fpm 解决 gem 依赖问题
#### 背景 
公司有同事需要在自己的公司上的一台服务器利用 fpm 进行打包, 原来其实可以通过以下命令进行安装

	yum install ruby-devel gcc
	gem install fpm  
当是在想为什么不用 fpm self-bootstrap, 利用 fpm pack 自己呢.  

开始简单以为, 就 ok 

	fpm -s gem  -t rpm  fpm 
	
结果发现自己想简单了, fpm 底层还有各种依赖. 后面通过以下方案解决: 

	mkdir /tmp/gems
	gem install --no-ri --no-rdoc --install-dir /tmp/gems fpm 
	find /tmp/gems/cache -name '*.gem' | xargs -rn1 fpm -d ruby -d rubygems --prefix /usr/lib/ruby/gems/1.8 -s gem -t rpm 
	然后将  生成的 rpm 放到 repo server 里边, 建立索引即可.  
	
	
注意`--prefix` 指的是 gem 的 `INSTALLATION DIRECTORY` 可以通过以下方式获取   

	root@10.223.21.205:~# gem env | grep INSTALL
  	- INSTALLATION DIRECTORY: /usr/lib/ruby/gems/1.8

