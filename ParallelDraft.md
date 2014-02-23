#Parallel(并行你的工作2)
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
或许上面的case 还不具有说服力, 那就说说Parallel 解决我最头疼的事情. 
也是中控+N 台服务的模型, 只是这次比较恶心的是在N 台服务其中部署了 1 个模块假设名称为 nginx, 为了避免资源浪费你在这N台机器上部署了M个nginx实例. 需要做的事情是然后你需要完成这样的一件间的任务: 更新所有nginx的配置文件, 然后重新reload配置文件.  
通常比较高效的做法是: 

1. 将nginx.conf 文件发送所有的机器上
2. 编写一个nginx_reload.sh 脚本, 也发送到所有的机器上.

写一个脚本 **nginx_reload.sh** 类似如下:

	1. cat HOSTS | xargs -i -P 0 rsync -HavP -e ssh nginx.conf  ${HOME}/tmp/
	2.
	for nginx in `ls -d nginx-* `
	do
		cp -v $HOME/tmp/nginx.conf $HOME/${nginx}/conf/ ; 
		$HOME/${nginx}/sbin/nginx -s reload ; 
		sleep 1;
	done
	
该方式能够以较小的代价来实现复杂的任务, 但是**问题是: 需要多发送一次nginx_reload.sh, 而且有时候为了线上的部署清晰, 通常还需要删除 nginx_reload.sh, 还需要在运行一次**

## do it in Parallel way

通过Parallel 能够十分方便完成上面的内容. 

在编写好**nginx_reload.sh** 之后利用
	
` 
parallel --sshlogin server1,server2,... --tag --basefile $HOME/tmep/nginx_reload.sh --cleanup --onall sh {} ::: $HOME/tmep/nginx_reload.sh 
` 

## Parallel 入门

### 安装

在[Parallel官网下载](http://www.gnu.org/software/parallel/), 安装里边的说明进行安装即可. 
**在安装之后, 没有想到原来Parallel 居然是perl 写的 **

	➜  bin git:(master) file parallel

	parallel: a perl script text executable

	➜  bin git:(master)

###  常用的选项

通常很多新同学在学习新命令的时候,会抱怨命令的选项太多, 其实根本就没有必要记住那么多, 记住最常用的就可以了. 剩下的通过笔记或者man 来查看就可以. 以下是我觉得笔记有用的选项: 

#### 运行的目标机器

	-S [ncpu/]sshlogin[,[ncpu/]sshlogin[,...]]
    --sshlogin [ncpu/]sshlogin[,[ncpu/]sshlogin[,...]]
                Distribute jobs to remote computers. The jobs will be run on a list of
                remote computers.  GNU parallel will determine the number of CPU cores on
                the remote computers and run the number of jobs as specified by -j.  If
                the number ncpu is given GNU parallel will use this number for number of
                CPU cores on the host. Normally ncpu will not be needed.

                An sshlogin is of the form:

                  [sshcommand [options]][username@]hostname

                The sshlogin must not require a password.

                The sshlogin ':' is special, it means 'no ssh' and will therefore run on
                the local computer.

                The sshlogin '..' is special, it read sshlogins from
                ~/.parallel/sshloginfile

                The sshlogin '-' is special, too, it read sshlogins from stdin (standard
                input).

                To specify more sshlogins separate the sshlogins by comma or repeat the
                options multiple times.

                For examples: see --sshloginfile.

                The remote host must have GNU parallel installed.

                --sshlogin is known to cause problems with -m and -X.

                --sshlogin is often used with --transfer, --return, --cleanup, and --trc.

#### 清理文件

	       --cleanup
                Remove transferred files. --cleanup will remove the transferred files on
                the remote computer after processing is done.

                  find log -name '*gz' | parallel \
                    --sshlogin server.example.com --transfer --return {.}.bz2 \
                    --cleanup "zcat {} | bzip -9 >{.}.bz2"

                With --transfer the file transferred to the remote computer will be
                removed on the remote computer.  Directories created will not be removed -
                even if they are empty.

                With --return the file transferred from the remote computer will be
                removed on the remote computer.  Directories created will not be removed -
                even if they are empty.

                --cleanup is ignored when not used with --transfer or --return.
                
#### 命令输出打tag
             --tag    
             	Tag lines with arguments. Each output line will be prepended with the
                arguments and TAB (\t). When combined with --onall or --nonall the lines
                will be prepended with the sshlogin instead.

                --tag is ignored when using -u.
#### 设置并发度
              
		--jobs N
        -j N
        --max-procs N
        -P N     Number of jobslots. Run up to N jobs in parallel.  0 means as many as
                possible. Default is 100% which will run one job per CPU core.

                If --semaphore is set default is 1 thus making a mutex.
### 分布式计算
        --transfer (alpha testing)
	Transfer files to remote computers. --transfer is used with --sshlogin when the arguments are files and should be transferred to the remote computers. The files will be transferred using rsync and will be put relative to the default login dir.
	 E.g.
  		echo foo/bar.txt | parallel \ 
  			--sshlogin server.example.com --transfer wc
	This will transfer the file foo/bar.txt to the computer server.example.com to the file $HOME/foo/bar.txt before running wc foo/bar.txt on server.example.com.
  	echo /tmp/foo/bar.txt | parallel  --sshlogin server.example.com --transfer wc
	This will transfer the file foo/bar.txt to the computer server.example.com to the file /tmp/foo/bar.txt before running wc /tmp/foo/bar.txt on server.example.com.
	
	--transfer is often used with --return and --cleanup.
	--transfer is ignored when used with --sshlogin : or when not used with --sshlogin.
	--trc filename
	Transfer, Return, Cleanup. Short hand for:
	--transfer--return filename —cleanup




通过组合上面的--cleanup 和 --return 选项我们可以很容易实现自动压缩下载[来源](http://www.rankfocus.com/automagically-compress-file-remotely-downloading-scprsync/). 

	function scp_gzip {
    	parallel -S $1 --cleanup --return {/}.gz "gzip --best {} -c > {/}.gz" ::: $2
	}
	
{/} 表示文件的dirname , 假设文件名为 foo.sh  那么 dirname 就是 foo
--return {/} 表示从server端 rsync 到本地. 
--cleanup 表示rsync 完成之后,会删除 server端的 .gz 文件. 

	::: 表示:::后面内容是前面的{}的参数. 
	parallel --tag  traceroute {} ::: iteye.com v2ex.com
	
--tag 能够针对每个输出进行标记, 能够帮助我们了解输出的来源. 

### 常见用法

#### 批量并行下载文件
假设有一批文件(file1..100)需要下载, 此时我们当然希望越快越好. 可以采用Parallel 轻松解决. 
将需要下载的文件写入 jobs

	wget file1
	wget file2
	......
	wget file100
	
` parallel -j 0 < jobs `

#### 替代两层循环

	(
	for gender in M F ; 
	do    
		for size in S M L XL XXL ; 
		do      
			echo $gender $size    
		done  
	done
	) | sort

可以通过以下命令轻松搞定, 懒人总是希望write less , do more. 

` parallel echo {} {} ::: M F ::: S M L XL XXL | sort `

#### 解决ls 等命令参数过多问题. 

	When moving a lot of files like this: mv *.log destdir you will sometimes get the error:
	
	bash: /bin/mv: Argument list too long
	
	because there are too many files. You can instead do:
	
	ls | grep -E '\.log$' | parallel mv {} destdir
	
	This will run mv for each file. It can be done faster if mv gets as many arguments that will fit on the line:
	
	ls | grep -E '\.log$' | parallel -m mv {} destdir
#### 远程执行命令
如最开始的case 讲的那样. 

		parallel -S server1,server2,...,server N --tag --cleanup --basefile $HOME/tmp/nginx_reload.sh 'sh {}' ::: $HOME/tmp/nginx_reload.sh
	
