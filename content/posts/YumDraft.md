+++
date = "2015-11-23T11:17:59+08:00"
draft = false
title = "YumDraft"

+++


### yum 常用变量
在 yum 的配置中, 我们经常可以看到`name=CentOS-$releasever - Base` 这样的变量, 
那么具体定义的 value 在哪里 ?  

```
/etc/yum/vars/releasever
```
