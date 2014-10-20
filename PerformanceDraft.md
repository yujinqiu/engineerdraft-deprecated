# 性能相关

## Memory

内存主要用来存储应用和内核的指令,产生的数据和**文件系统cache**. 影响性能的因素主要包括以下几方面  
1. 内存和磁盘的数据交换
2. 内存的分配回收和内存拷贝和内存映射
3. 在multi-socket 架构上, 内存的locality 也会成为一个重要的因素, 主要是在NUMA的架构下, cpu 访问本地node的 延迟要比远程的node 的延迟低.

**其中用在文件系统cache的策略是:如果系统有空闲的内存,那么就尽可能使用, 被用在文件系统cache的内存可以当成未使用,在内存紧张的时候可以被kernel异步回收,不影响性能**


## 术语

**Resident Memory**: 实际物理内存(注意部分内存可能是共享内存)  
**Anonymous Memory**: memory with no file system location or path name. 包括一个进程空间的working data (heap, stacks COW pages)  # 如果系统当前内存紧张,会把anonymous memory 交换到swap分区. 有些内存的换出(dirty page)是直接刷到disk 上的文件上这种内存就不是匿名内存. 

A good example of **Anonymous Memory** is if you fork a process. All the addresses in the second process actually map back to the same bits of physical memory (the same pages). However if youre child process was then to do something different with the memory (eg. the child went off and manipulated an array in memory) the VM subsytem would copy those pages and change the mappings in the child process to point to the new pages. This new memory would be anonymous memory, and the child process would merrily make the changes to the array, unaware it now had new "physical" memory it was talking to.  
     
**Segment**: **an area of memory flagged for a particular purpose**, such as for storing executable or writeable pages.   
**Page**: a unit of memory, used by the OS and CPUS.历史原因一般大小为 4k 或者8k.现代的处理器支持multiple page size.   
**Swap**: an on-disk area for paged **anonymous data and swapped processes**, 可以是存储区(disk)的一个区域(也叫: physical swap device) 或者是文件系统中的一个文件(也叫 swap file). 

![虚拟内存](https://code.csdn.net/KnightYu/engineerdraft/file/img/vm.png)  
**注意其中在Main Memory 和 Swap Device 进行交换的是 Anonymous Page.**

### overcommit

虽然采用的是虚拟内存管理, 内存还是有一定的限制. 其中solaris-based 内核允许申请的内存大小是 sum(physical memory , swap). linux 相对来说更加灵活支持3中方式. 配置在`/proc/sys/vm/overcommit_memory`内,默认value 为 0  
1. **0** : The kernel performs heuristic memory overcommit handling by estimating the amount of memory available and failing requests that are blatantly invalid. Unfortunately, since memory is allocated using a heuristic rather than a precise algorithm, this setting can sometimes allow available memory on the system to be overloaded.
2. **1** : 不进行overcommit 检查, 总是返回成功. 
3. **2** : 最大允许分配内存 = swap + physical memory * /proc/sys/vm/overcommit_ratio (default 50) 该策略推荐当 swap 大于 物理内存的时候使用. 


### Paging

paging is the movement of pages in and out of **main memory**.  
paging 可以分为两种: 
1. file system paging
2. anonymous paging

#### File System Paging
File system paging is caused by the reading and writing of pages in **memory-mapped files**. This is normal behavior for applications that use file memory mappings(mmap()), and on file systems that use the page cache. It has been referred to as "good" paging.  

##### paging out
对于文件系统page, paging out 的时候, 如果该page 被标记为dirty, 那么写会到disk中,然后进行回收, 如果没有被标记为dirty的话,那么就直接回收, 因为在disk 中已经有一个副本. 

#### Anonymous Paging

Anonymous paging involves data that is **private to processes**:  the process heap and stacks. Anonymous page-outs require moving the data to physical swap devices or swap files. linux 对于这种paging out 叫做 `swapping`.  
Anonymous paging **hurts performance** and has therefore been referred to as "bad" paging.
When applications access memory pages that have been paged out, they **block** on the disk I/O required to read them back to main memory, 这个过程必须要同步完成, 因此会影响性能, 但是anonymous page-outs 有可能不会影响性能, 因为在一定程度内存紧张的时候 kernel 会fork pdflush 进行来进行**异步** page-out. 

## 架构

### SMP/UMA

UMA(uniform memory access)  或者 SMP(Symmetric Multi Processing)
![UMA](https://code.csdn.net/KnightYu/engineerdraft/file/img/UMA.png)   
 SMP是非常常见的一种架构。在SMP模式下，多个处理器均对称的连接在系统内存上，所有处理器都以平等的代价访问系统内存。  
**优点**:是对内存的访问是平等、一致的 
**缺点**:因为大家都是一致的，在传统的 SMP 系统中，所有处理器都共享系统总线，因此当处理器的数目增多时，系统总线的**竞争冲突迅速加大，系统总线成为了性能瓶颈**，所以目前 SMP 系统的处理器数目一般只有数十个，可扩展性受到很大限制。

### MPP (Massive Parallel Processing)
MPP则是逻辑上将整个系统划分为多个节点，每个节点的处理器只可以访问本身的本地资源，是完全无共享的架构。节点之间的数据交换需要软件实施。 
**优点**:可扩展性非常好
**缺点**:是彼此数据交换困难，需要控制软件的大量工作来实现通讯以及任务的分配、调度，对于一般的企业应用而言过于复杂，效率不高。


### NUMA

NUMA(non uniform memory access)  
![NUMA](https://code.csdn.net/KnightYu/engineerdraft/file/img/NUMA.png)  
NUMA架构则在某种意义上是综合了SMP和MPP的特点：逻辑上整个系统也是分为多个节点，每个节点可以访问本地内存资源，也可以访问远程内存资源，但访问本地内存资源远远快于远程内存资源。  
**优点**:兼顾了SMP和MPP的特点, 易于管理，可扩充性好  
**缺点**:是访问远程内存资源的所需时间非常的大  
在实际系统中使用比较广的是SMP和NUMA架构。像传统的英特尔IA架构就是SMP，而很多大型机采用了NUMA架构。


#### 相关命令

1. 查看是否支持NUMA `dmesg | grep NUMA`
2. 查看NUMA具体信息  `numastat`  

                           node0         node1
        numa_hit	     120670785322   23780308976
        numa_miss	     252644067      22217317694
        numa_foreign	 22217317694    252644067
        interleave_hit	 14717          14680
        local_node	     120670746342   23780294029
        other_node	     252683047      22217332641  
    
numa_hit 表示在该node上分配内存成功次数
numa_miss 表示在该节点上分配失败次数
numa_foreign 表示在原来在其它节点上分配内存,最后在这个节点上分配(所以在2个node的情况下numa_miss 和 numa_foreign的)
interleave_hit 表示采用interleave策略后从该节点分配的次数
locl_node 表示该节点的进程在该节点上分配内存的次数
other_node 表示其它节点的进程在该节点分配内存的次数.   

3.查看cpu和node 的映射关系 `lscpu`

    NUMA node0 CPU(s):     0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30
    NUMA node1 CPU(s):     1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31  
    
4. 节点内存大小 `numastat`

        available: 2 nodes (0-1)
        node 0 size: 32756 MB
        node 0 free: 2089 MB
        node 1 size: 32768 MB
        node 1 free: 4354 MB
        
5. NUMA 支持策略,查看当前策略 `numactl --show`

        缺省(default) - 总是在本地节点分配（分配在当前线程运行的节点上）
        绑定(bind) - 分配到指定节点上
        交织(interleave) - 在所有节点或者指定的节点上交织分配
        优先(preferred) - 在指定节点上分配，失败则在其他节点上分配
        
6. swap insanity
    内存不足无疑会SWAP，但有些时候，即便看上去内存很充裕，还可能会SWAP，这种现象被称为SWAP Insanity.
    
        shell> numactl --hardware
        available: 2 nodes (0-1)
        node 0 size: 16131 MB
        node 0 free: 100 MB
        node 1 size: 16160 MB
        node 1 free: 10 MB
        node distances:
        
        可以看到系统有两个节点（其实就是两个物理CPU），它们各自分了16G内存，其中零号节点还剩100M内存，一号节点还剩10M内存。设想启动了一个需要11M内存的进程，系统把它分给了一号节点来执行，此时虽然系统总体的可用内存大于该进程需要的内存，但因为一号节点本身剩余的可用内存不足，所以仍然可能会触发SWAP行为。
        
        
### 内存分配
![UMA](https://code.csdn.net/KnightYu/engineerdraft/file/img/free_memory.png) 

. Free List: 可以直接用来分配内存的内存list, 在NUMA架构下, 通常会有多个list, 通常是一个group一个.
. Reaping : 当内存资源紧张的时候, kernel modules 和 kernel slab allocator 会释放内存. This is also known as shrinking. 

对于Linux 的的内存回收策略:   
. Page Cache : 文件系统cache . 在linux 中有一个可以调节的参数 /proc/sys/vm/swappiness (default 60)  Swappiness is a parameter of the Linux kernel that controls the relative weight given to swapping out runtime memory, as opposed to dropping pages from the system page cache.
. Swapping : 被kswapd (page-out daemon), to find not recent used page and add to free list including memory list.





    




