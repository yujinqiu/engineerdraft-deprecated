Parallel(并行你的工作)
--------------------
今天看到看到一篇介绍Parallel 的文章[Use multiple CPU Cores with your Linux commands — awk, sed, bzip2, grep, wc, etc.](http://www.rankfocus.com/use-cpu-cores-linux-commands/), 感觉和实际工作的没有太大的帮助, 因为平时其实很少去压缩和解压缩超大的文件. 因此下面主要结合个人的工作经验,总结几条Parallel的工作.
![](http://www.gnu.org/software/parallel/logo-gray+black300.png)

## 简单介绍一下场景
在日常运维工作中,为了管理大量的机器(1k,1w台机器)我们通常搭建一台中心控制机器, 该机器和线上所有的机器有[信任关系](http://todo), 因此常见的工作就是在中心控制机器上顺序登录各台机器执行命令. 举一个简单的例子, 获取机器的启动时间 `uptime` . 考虑到机器的规模,假设是1万台, 我们基本上跑一下要消耗3-5min. 

为了解决上面的问题,我曾经学习了一下 `xargs` 命令, 已经能够基本上满足我的要求. 假设机器的列表在 machines 文件内, 那么通过以下命令

` cat machines | xargs -i -P 0 ssh {} " uptime " `

