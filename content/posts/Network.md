+++
date = "2015-11-17T15:59:26+08:00"
draft = false
title = "network"

+++

## ip route 删除路由策略
### 背景
docker 启动的时候, 需要删除192的 route  

```
ip route 
192.168.0.0/16 via 10.242.154.1 dev eth1
172.16.0.0/12 via 10.242.154.1 dev eth1
10.0.0.0/8 via 10.242.154.1 dev eth1
default dev tunnat  scope link
```

删除的时候直接输入` ip route` 的输出结果,即可. 
```
ip route  192.168.0.0/16 via 10.242.154.1 dev eth1
```

