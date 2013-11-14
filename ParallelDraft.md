Parallel(并行你的工作)
--------------------
今天看到看到一篇介绍Parallel 的文章[Use multiple CPU Cores with your Linux commands — awk, sed, bzip2, grep, wc, etc.](http://www.rankfocus.com/use-cpu-cores-linux-commands/), 感觉和实际工作的没有太大的帮助, 因为平时其实很少去压缩和解压缩超大的文件. 因此下面主要结合个人的工作经验,总结几条Parallel的工作.
下面看一下Parallel的图片,感觉有点眼花,很有心理学的图片效果, 相信我Parallel 是一个值得你学习的命令, 所谓事半功倍说的就是它. 

![](http://www.gnu.org/software/parallel/logo-gray+black300.png)

## 典型场景1
在日常运维工作中,为了管理大量的机器(1k,1w台机器)我们通常搭建一台中心控制机器, 该机器和线上所有的机器有[信任关系](http://todo), 因此常见的工作就是在中心控制机器上顺序登录各台机器执行命令. 举一个简单的例子, 获取机器的启动时间 `uptime` . 考虑到机器的规模,假设是1万台, 我们基本上跑一下要消耗3-5min. 

为了解决上面的问题,我曾经学习了一下 `xargs` 命令, 已经能够基本上满足我的要求. 假设机器的列表在 machines 文件内, 那么通过以下命令

` cat machines | xargs -i ssh {} " uptime " `

上面的命令是逐台机器执行, 好处就是可以省略在bash 里边输入各种 for 循环. 上面的效果和下面的命令一致
>
for host in $( < machines ) ; do ssh ${host} " uptime " ;done 

如果需要并行执行应该怎么搞? 答案很简单翻一下 xargs 的man 手册你会发现有一个`-P` 选项, 表示并发度
`-P 0 ` 表示全部并发.  -P 10 表示10台并发 (**在运维工程中通常需要考虑控制并发度,特别是在日志下载的时候,因为进程会出现宽度被打满的case,要特别注意**)
因此 

` cat machines | xargs -i -P 0 ssh {} " uptime " ` 

就能实现全部并发. 上面的命令你执行之后会发现由于是并行执行, 因此输出基本上是乱序的. 如果某一台机器执行错误,那么基本上很难定位到是哪台.  所以个人认为 xargs 在进行简单的并行,比如批量重启服务,分发大量小文件还是蛮不错的. 我最常用的命令组合是,来实现快速的文件分发

`
cat machine | xargs -i -P 0 rsync -HavP -e ssh  FILE {}:PATH
`
** 对于错误问题定位xargs 还是不能解决, 因此需要Parallel 来解决. 

## 典型场景2
或许上面的case 还不具有说服力




