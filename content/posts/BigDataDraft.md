+++
date = "2015-02-02T20:30:22+08:00"
draft = fase
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
