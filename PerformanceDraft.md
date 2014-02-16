# 性能相关

## Memory

内存主要用来存储应用和内核的指令,产生的数据和**文件系统cache**. 影响性能的因素主要包括以下几方面  
1. 内存和磁盘的数据交换
2. 内存的分配回收和内存拷贝和内存映射
3. 在multi-socket 架构上, 内存的locality 也会成为一个重要的因素, 主要是在NUMA的架构下, cpu 访问本地node的 延迟要比远程的node 的延迟低.

## 术语

**Resident Memory**: 实际物理内存(注意部分内存可能是共享内存)  
**Anonymous Memory**: memory with no file system location or path name. 包括一个进程空间的working data (heap)  # 如果系统当前内存紧张,会把anonymous memory 交换到swap分区. 有些内存的换出(dirty page)是直接刷到disk 上的文件上这种内存就不是匿名内存.  
     
**Segment**: **an area of memory flagged for a particular purpose**, such as for storing executable or writeable pages.   
**Page**: a unit of memory, used by the OS and CPUS.历史原因一般大小为 4k 或者8k.现代的处理器支持multiple page size.   
**Swap**: an on-disk area for paged **anonymous data and swapped processes**, 可以是存储区(disk)的一个区域(也叫: physical swap device) 或者是文件系统中的一个文件(也叫 swap file). 

![虚拟内存](https://code.csdn.net/KnightYu/engineerdraft/file/img/vm.png)  
**注意其中在Main Memory 和 Swap Device 进行交换的是 Anonymous Page.**
