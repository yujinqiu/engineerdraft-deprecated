#函数式编程(functional programming)
一直听闻函数式编程的神奇,也多次进行学习遗憾的是多次遗忘, 因此有必要进行总结一下.

##背景知识
数学就像拼图一样, 很多结论都是这样推导出来的: 先是确定一些互不冲突的**基础原理**, 已经一些操作这些**原理的规则**,然后就可以把这些原理以及规则拼凑起来形成更加复杂的规则或者定理.  数学家把这种方法成为**形式系统或者是演算(calculus)**  
阿隆佐·邱奇(Alonzo Church) 在解决尝试解决关于计算的问题, 诸如:如果拥有一台无穷计算能力的超级计算机, 可以用来解决什么问题? 它可以自动的解决这些问题吗? 是不是还有些问题解决不了? 如果有的话是为什么?  如果这样的机器采用不同的设计,他们的计算能力相同吗?  
Alonzo Church 设计了 lambda calculus,这个系统本质上就是超级计算机的编程语言, 这种语言里边, 函数的参数是函数, 返回值也是函数. 这种函数用希腊字母(lambda), alonzo 利用这种形式系统来分析前面的问题,并能够给出答案, 图灵进行类似的研究, 设计了完全不同的系统(图灵机)得到相似的答案

如果二战没有发生，这个故事到这里就应该结束了，我的这篇小文没什么好说的了，你们也可以去看看有什么其他好看的文章。可是二战还是爆发了，整个世界陷于火海之中。那时的美军空前的大量使用炮兵。为了提高轰炸的精度，军方聘请了大批数学家夜以继日的**求解各种差分方程用于计算各种火炮发射数据表**。后来他们发现单纯手工计算这些方程太耗时了，为了解决这个问题，各种各样的计算设备应运而生。IBM制造的Mark一号就是用来计算这些发射数据表的第一台机器。Mark一号重5吨，由75万个零部件构成，重一秒内可以完成3次运算。
战后，人们为提高计算能力而做出的努力并没有停止。1949年第一台电子离散变量自动计算机诞生并取得了巨大的成功。它是冯·诺伊曼设计架构的第一个实例，也是一台现实世界中**实现的图灵机**。相比他的这些同事，那个时候阿隆佐的运气就没那么好了。
到了50年代末，一个叫John McCarthy的MIT教授（他也是普林斯顿的硕士）对阿隆佐的成果产生了兴趣。1958年他发明了一种列表处理语言（Lisp），这种语言是一种阿隆佐lambda演算在现实世界的实现，而且它能在冯·诺伊曼计算机上运行！很多计算机科学家都认识到了Lisp强大的能力。1973年在MIT人工智能实验室的一些程序员研发出一种机器，并把它叫做Lisp机。于是阿隆佐的lambda演算也有自己的硬件实现了！  
##基本思想
函数式编程是Alonzo Church思想在现实世界的实现, 不过不是所有的lambda演算思想都可以运用到实际中, 因为lambda 演算在设计的时候就不是为了在现实世界中的各种限制下工作, 所以就像面向对象的编程思想意义, **函数式编程只是一系列的想法**, 而不是一套严格的规定.   
1. 函数是函数式编程中基础的元素, 可以完成几乎所有的操作, 那边是最简单的计算也是用函数完成.
2. 函数式编程中所有的"变量"是 **不能被修改的**, *部分编程语言实现上是允许*, **此时变量叫符号**