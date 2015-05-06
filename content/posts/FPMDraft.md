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
        
### fpm 打包版本升级
#### 背景
在打包 python-supervisor 的时候遇见这样的一个问题, 该打包策略比较复杂, 需要打包多次验证效果.    

    fpm -s dir -t rpm  -n python-supervisor  -v '3.1.3' --iteration 2 -a 'x86_64' --description='A system for controlling process state under UNIX' --after-install='/root/load.sh' -d 'python-meld3' -C supervisord  .
    
执行之后, 得到的结果是 `python-supervisor-3.1.3-1.x86_64.rpm`, 第二次打包的时候, 如果还是这样的命令得到的是相同的结果, 很多时候我们还是希望能够有一个版本的标识, 来区分各个打包的版本,  其中 `3.1.3` 是模块的版本, 在出问题的时候为了方便追溯问题, 不建议修改.  因此只能修改 `3.1.3-1`, 后面的 `1` , fpm 提供了该选项.  

    --iteration ITERATION
   

    
        
### fpm 制定 package 依赖

	fpm -s dir -t rpm -n langley_online -v 1.0.0 -d 'thrift >= 0.9.1' -d 'boost >= 1.41.0' -d 'libevent >= 2.0.21' -d 'fastbit >= 2.0.1' --prefix=/home/<user>   --after-install=langley_online/install/install_hook.sh langley_online
	
<!-- more -->
        
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
  	
  	
  	
## FPM 打包升级 python2.7
###  背景  
运维过 RHEL/CentOS 的同学基本上都知道CentOS 为了追求系统的稳定, 软件包相对来说是比较旧的. 大家抱怨比较多的是 python 的版本2.6.6 , 大家比较喜欢的是 python2.7, 因此大家迫切希望能够有python2.7 的 rpm , 一键进行升级.  其实之前升级过一次, 但是不小心把 yum 给搞坏了, 要知道 yum 其实也是 python 写的程序, 因此有一个原则需要遵守的是 python2.7 必须和 python 2.6 隔离开.  
### 安装方法  

1: 下载 python2.7.9 
2:  tar xf Python-2.7.9.tgz   
3:   

    cd Python-2.7.6.tgz
    # Python2.7编译安装后会安装到这个目录，方便打包
    export INTERMEDIATE_INSTALL_DIR=/tmp/installdir-Python-2.7.9
    # RPM包安装后Python2.7的目录
    export INSTALL_DIR=/usr/local
    LDFLAGS="-Wl,-rpath=${INSTALL_DIR}/lib ${LDFLAGS}" \
    ./configure --prefix=${INSTALL_DIR}  \
    --enable-shared --enable-ipv6
    make
    make install DESTDIR=${INTERMEDIATE_INSTALL_DIR}
    
 4: 使用 fpm 打包 
 
        fpm -s dir -t rpm -n python2.7 -v '2.7.9' \
        -a 'x86_64' \
        -d 'openssl' \
        -d 'bzip2' \
        -d 'zlib' \
        -d 'expat' \
        -d 'db4' \
        -d 'sqlite' \
        -d 'ncurses' \
        -d 'readline' \
        -d 'zlib-devel' \
        -d 'bzip2-devel' \
        -d 'openssl-devel' \
        -d 'ncurses-devel'  \
        -d 'sqlite-devel'  \
        --description='python 2.7.9 package by op' \
        --directories=${INSTALL_DIR}/lib/python2.7/ \
        --directories=${INSTALL_DIR}/include/python2.7/ \
        -C ${INTERMEDIATE_INSTALL_DIR} .
   
#### ChangeLog
取消 --enable-unicode=ucs4 编译选项, 该选项会导致 json decode 的时候出错. 
        
#### refer
[http://theo.im/blog/2014/05/16/use-fpm-to-create-python-rpm-packages/](http://theo.im/blog/2014/05/16/use-fpm-to-create-python-rpm-packages/)
[https://github.com/h2oai/h2o/wiki/Installing-python-2.7-on-centos-6.3.-Follow-this-sequence-exactly-for-centos-machine-only](https://github.com/h2oai/h2o/wiki/Installing-python-2.7-on-centos-6.3.-Follow-this-sequence-exactly-for-centos-machine-only)
[http://toomuchdata.com/2014/02/16/how-to-install-python-on-centos/](http://toomuchdata.com/2014/02/16/how-to-install-python-on-centos/)

### python 2.7.9 福利
python 2.7.9 之后就内置 pip 安装程序,  开启如下:  

    python2.7 -m ensurepip


## 如何查看 RPM 包文件列表
### 背景  
经常我们需要了解一个 rpm 包在安装完成之后, 会安装在什么路径下, 安装什么文件, 需要查看 rpm 内部的文件和路径. 
### 常规方法

    rpm -qpl  pkg.RPM
    
    rpm -qpl rubygem-fpm-1.3.3-1.noarch.rpm
    /usr/bin/fpm
    /usr/lib/ruby/gems/1.8/cache/fpm-1.3.3.gem
    /usr/lib/ruby/gems/1.8/doc
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/.require_paths
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/CHANGELIST
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/CONTRIBUTORS
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/LICENSE
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/bin/fpm
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/lib/fpm.rb
    /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/lib/fpm/command.rb

### 比较巧妙的方法

    less  pkg.RPM  #发现 less zip,tar.gz 文件也是可以的
    less rubygem-fpm-1.3.3-1.noarch.rpm
    Name        : rubygem-fpm                  Relocations: /
    Version     : 1.3.3                             Vendor: Jordan Sissel
    Release     : 1                             Build Date: Tue 03 Mar 2015 01:46:09 PM CST
    Install Date: (not installed)               Build Host: svn02.qq.diditaxi.com
    Group       : Languages/Development/Ruby    Source RPM: rubygem-fpm-1.3.3-1.src.rpm
    Size        : 702822                           License: MIT-like
    Signature   : (none)
    Packager    : <root@svn02.qq.diditaxi.com>
    URL         : https://github.com/jordansissel/fpm
    Summary     : Convert directories, rpms, python eggs, rubygems, and more to rpms, debs,     solaris packages and more. Win at package management without wasting pointless hours debugging bad rpm specs!
        Description :
    Convert directories, rpms, python eggs, rubygems, and more to rpms, debs, solaris packages and more. Win at package management without wasting pointless hours debugging bad rpm specs!
    -rwxr-xr-x    1 root    root                      364 Mar  3 13:46 /usr/bin/fpm
    -rw-r--r--    1 root    root                   114176 Mar  3 13:46 /usr/lib/ruby/gems/1.8/cache/fpm-1.3.3.gem
    drwxr-xr-x    2 root    root                        0 Mar  3 13:46 /usr/lib/ruby/gems/1.8/doc
    -rw-r--r--    1 root    root                       12 Mar  3 13:46 /usr/lib/ruby/gems/1.8/gems/fpm-1.3.3/.require_paths
    
 
## 如何解压 RPM 
### 背景
有时候我们需要定制化 RPM 包的内容, 比如说在已有的 RPM 包中增加一个配置, 或者增加一个启动脚本.  这时候就需要将rpm 包重新打开, 然后增加内容, 然后在重新打回去.  

    rpm2cpio  pkg  |  cpio -idmv


## yum 安装 pkg 的时候如何指定 repo
### 背景 
最近公司使用 yum 来进行代码发布, 使用久之后, 发现一个问题就是 如果在一个 repo 下面有多个 repo 的 server , 比如有** puppet**, 然后你在` yum install -y` 的时候, 通常需要search 全部的 repo, 在执行的效率上慢了很多, 因此我们对于确定知道 pkg 的 在哪个 repo 的地方的时候, 希望直接从这个 repo 下载.  

    1. yum repolist   获取 repo id
    2. yum --disablerepo="*" --enablerepo="${repo}" install -y  pkg
    
    
## yum 如何下载到最新的 pkg
###  背景  
通常我们在 pack 最新的 package,  在使用  
    
    createrepo --update  ./
之后, 就希望在其它机器上就可以使用` yum install -y` 进行安装, 可是经常发现找不到最新的 pkg,  开始的时候有人提出来了,  

    yum make clean   
    
然后发现过了很久居然没有人质疑这件事情?  难道一定要这样 make clean ?   
RTFM之后, 发现其实是有解决方案的.   在` yum.conf` 设置时间足够短就 ok.  


    metadata_expire Time (in seconds) 
    after which the metadata will expire. So that if the current metadata downloaded is less than this many
              seconds old then yum will not update the metadata against the repository.  If you find that yum is not downloading information on updates
              as often as you would like lower the value of this option. You can also change from the default of using seconds to using days, hours  or
              minutes  by appending a d, h or m respectively.  The default is 6 hours, to compliment yum-updatesd running once an hour.  It’s also pos-
              sible to use the word "never", meaning that the metadata will never expire. Note that when using a metalink file the metalink must always
              be newer than the metadata for the repository, due to the validation, so this timeout also applies to the metalink file.