+++
date = "2015-02-02T20:30:22+08:00"
draft = false
title = "BigDataDraft"

+++

## hbase 查看master
hbase master 信息存储在zookeeper 中, 可以在 zookeeper 中查询.   

	zkCli.sh -server myzoo  get  /hbase/master

## 如何启动 hmaster/regionserver
	
	hbase-daemon.sh start master #hmaster, stop 同理
	hbase-daemon.sh start regionserver #regionserver, stop 同理
	
## start master 和 ./bin/start-hbase.sh 有啥区别? 

	hbase-daemon.sh start master 是用来启动单个 master
	./bin/start-hbase.sh 会启动整个集群, 其中会读取 hbase-0.96.2-hadoop1/conf/regionservers 用来标记 regionservers 的列表, 扩容的时候不一定需要添加进去, 因为 regionserver 的机器存储在 zookeeper.  
	
	
## 如何下线 region server 

    ./bin/graceful_stop.sh  ${nodename}


## 如何下线 datanode
### 背景
所谓无知者无畏, 自己太想当然了, 以为hadoop 的 datanode 有三份冗余, 就可以毫无顾虑就把data node stop 就 ok了.  后来发现自己太天真了.  
### 正确姿势 
1: 在namenode 的 `hdfs-site.xml` 中新增(如果不存在)  

     <property>                                                                                                                       
        <name>dfs.hosts.exclude</name>                                                                                             
        <value>${path to exclude file}</ value>                                                                                                    
      </property>
      
2: 在`${path to exclude file}` 文件中新增需要摘除的节点(机器名称)  

    机器名称(也可能是 IP)可以通过下面方式获得  
    
        ./bin/hadoop dfsadmin -report
        
    正常情况下节点的信息
        Name: 10.242.149.240:50010  (Name 这里的 name 后面的 名称就是写入 ${path to exclude file} 内)
        Decommission Status : Normal (正常状态)
        Configured Capacity: 0 (0 KB)
        DFS Used: 0 (0 KB)
        Non DFS Used: 0 (0 KB)
        DFS Remaining: 0(0 KB)
        DFS Used%: 100%
        DFS Remaining%: 0%
        Last contact: Wed May 13 15:40:10 CST 2015

3: 迁移节点数据 

    ./bin/hadoop dfsadmin -refreshNodes
    
    
## 常见问题
1. WARN org.apache.hadoop.util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable  
原因:  默认下载的是32bit 上面的系统, 因此需要自己编译. 特别注意一下: file  lib/native 里边的 so 提示是64位, 感觉不对, 反正就是需要自己在64bit 上编译一套.   

### 编译方法
整个编译过程比较长, 主要浪费在下载过程.  核心点   
1: 安装 protocol buffer

    ./configure && make && make install
1: 确认依赖都安装 
     
     yum install -y gcc autoconf automake libtool zlib-devel bzip2-devel  openssl-devel ncurses-devel cmake
     
2: 安装 maven 

3: 下载 hadoop, 并编译 

    mvn package -Pdist,native -Dskiptests -Dtar
4:  编译完成之后,把 **hadoop-dist/target/hadoop-2.5.2/lib/native/** 下面的文件, 替换原来的32bit 的文件即可. 不需要像网上各种文章说需要设置各种环境变量.  

#### 特殊说明
在安装的过程中, 有莫名的报错, 重复执行编译 `mvn package -Pdist,native -Dskiptests -Dtar` 会成功

#### 额外配置  
如果需要设置 codecs 的顺序, 可以在 `core-site.xml` 里边配置 .   

        <property>
            <name>io.compression.codecs</name>
            <value>
                    org.apache.hadoop.io.compress.BZip2Codec,
                    org.apache.hadoop.io.compress.GzipCodec,
                    org.apache.hadoop.io.compress.DefaultCodec,
                    com.hadoop.compression.lzo.LzoCodec,
                    com.hadoop.compression.lzo.LzopCodec
            </value>
        </property>

    

### 如何校验是否成功
    
    ./bin/hadoop checknative -a

    
    
