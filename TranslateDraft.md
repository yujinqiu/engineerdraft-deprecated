#c-for-everyone-else
***部分翻译删掉冗余部分同时加入个人理解,[原文](http://wizardryandfunnyhats.com/2013/11/18/c-for-everyone-else/)***

## 前言    

本文是针对各个层次的程序员, 不是一个完整的介绍但是可以让你学习如何开始并指出c的一些明显的缺陷, 本文不再包含其他语言存在或者类似的的内容.

## 开发环境

c编译器主要有 `gcc` 和 `clang` 各有千秋.   
个人喜欢clang.  
1. BSD license  
2. 静态分析(可以在bug出现之前发现 `clang --analyze`)  
3. 代码整齐简介,可以给其他程序做edit-time 语法检查,e.g.: vim or emacs  
4. 出错信息可读性更强, 这点对新手bug修复特别有帮助

特殊说明: clang 在mac , linux 上均可以使用. windows 比较折腾请见原文. 
### clang 基本用法
`clang  sourcefile.c -o out`
   

## 预处理器(the Preprocessor)
`C 语言在设计的时候就没有模块概念`  
预处理器本质上就copy-paste 代码,当然也有按照一定条件的替换, 最为重要的是 `#include header.h`  
### 头文件(Headers)
头文件通常内容是各种public的定义. 可能遇到的问题是: 一个header文件被2个文件include, 然后这些文件均被一个文件include, 编译器会报错.可以利用  

    #ifndef FOO_H
    #define FOO_H
    
    int foo(int f);
    
    #endif    
    
### 数据类型(datatypes) 

#### primitive type  

+ int (signed int and unsigned int)
+ float and double are floating point types. A general rule here is to always use **double**.
+ char is basically a byte.
+ boolean don't exist.
##### compound type  

###### struct

**structures** can contain any primitive type.  

    struct point {
        int x;
        int y;
    };
    
access an element you use the well know **dot notation**.

    struct point foo = {x:5,y:4};
    foo.x
    foo.y
    
   
为了定义新数据结构, C 提供typedef  

    typedef struct point { 
        int x;
        int y;
     }Point;
*the struct tag name can also be omitted*

    typedef struct {
        int x;
        int y;
     }Point;

##### array

int a[5] = {0,1,2,3,4};

Indexing starts at 0  the reason of that is : 

    q[4] == 4[q]


To understand the reason behind it , we have to take a look at **Pointer**.  

when you allocate an array is reserve multiple variables which sit next to each other.
** A variable which holds an array is nothing more than the address of it's first element. **

Now we can understand the array notation, its actually syntactic sugar for pointer.
a[5] == *(a+5)
5[a] == *(5+a);

##### Pointer

    int* e;
    int f = 5;
    e = &f;
    printf("%d",*e);
    
e is a point , means the memory location e doesn't hod an integer but another memory location where an integer is to be found.


#### Memory Model

Stack is a way to return from a function to the previous point of execution.  
It also holds references to objects used in that function.
C is a pass by value language, than means that when you call a function with arguments the values of them get **copied** to the stack frame of the invoke function.  
To modify variables from within another function you have to pass a pointer(reference)
as an argument. ** Now the values of the pointer(the address) gets copied and you can dereference to manipulate the variable.  

values on the stack are garbage collected. If a function returns, everything that lay in the scope of the function gets removed from the stack.

You can't return an array you created from a function. For this reason if you want to access to the variables created in a function after it returned you have to **allocate the memory on the heap.**   And because C isn't garbage collected for the head you have to manually free allocated memory. 
there are a few ways to allocate memory on the heap exposed by the c standard library.  

    void* calloc(size_t size); #rule of thumb use calloc for arrays.
    void* malloc(size_t size);
    
    

##### Strings

Strings are nothing more than an array of characters. But unlike most other pangurages c does not' keep track of the strings length explicitly. It's more like a linked list. 

** 需要额外的一个byte \0 NULL 来标识字符串的结束,否则直到下一下null byte, 所以尽可能使用接受长度的函数,来避免溢出**

历史原因: 在计算机初期, 内存容量非常有限, 当时为了表示字符串的长度,主要有两种方案  
1. Pascal string: 在开头的一个byte来存储长度.
2. 在结尾的地方使用NULL. 

方案1 有一个缺点就是字符串的长度最长为256, 所以采用了第二种方案.




