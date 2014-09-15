---
layout: post
category : language
tagline: "c 语言总结"
tags : [linux, clang]
---
{% include JB/setup %}

###基础篇

面试中能说出pragma  package  pragma  once  算是达到要求了。
如果能把Pragma  message   pragma  hdrstop  pragma  comment也说明下，那面试官应该就很满意了

注释：
0 对于全局数据（全局变量、常量定义等）必须要加注释。
1 数值的单位一定要注释。注释应该说明某数值的单位到底是什么意思。比如：关于长度的必须说明单位是毫米，米，还是千米等；关于时间的必须说明单位是时，分，秒，还是毫秒等。
2 函数含义标识符构成：动词(一般现时)+目标词+[状语]+[目的地]；
3 变量含义标识符构成：目标词+ 动词(的过去分词)+ [状语] + [目的地]；
4 不仅要检查输入参数的有效性，还要检查通过其它途径进入函数体内的变量的有效性，例如全局变量、文件句柄等。

####变量作用域的正确理解

    #include <stdio.h>
    FILE * open_data(void)
    {
        FILE *fp;
        char databuf[BUFSIZ];
        /* setvbuf makes this the stdio buffer */
        if ((fp = fopen("datafile", "r")) == NULL)
            return(NULL);

        if (setvbuf(fp, databuf, _IOLBF, BUFSIZ) != 0)
            return(NULL);
        return(fp);
        /* error */
    }

    你能看出这个程序的问题么?

####整型溢出，类型提升

####函数：
    atexit()的使用。
    tolower()  toupper()使用要注意。先判断后使用

####回调函数

####可变参数

    typedef  char *  va_list;
    #define _INTSIZEOF(n)  ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1) )
    #define va_start(ap,v)  ( ap = (va_list)&v + _INTSIZEOF(v) )
    #define va_arg(ap,t)    ( *(t *)((ap += _INTSIZEOF(t)) - _INTSIZEOF(t)) )
    #define va_end(ap)      ( ap = (va_list)0 )

####内存拷贝、移动memmove() memcmp()   memcpy()

####柔性数组struct

####typedef   ＃define   const

对于#define来说，仅在编译前对源代码进行了字符串替换处理；而对于typedef来说，它建立了一个新的数据类型别名。

####volatile 三种用途

第一种情况涉及到内存映射硬件(memory-mapped hardware，如图形适配器，这类设备对计算机来说就好象是内存的一部分一样)，第二种情况涉及到共享内存(shared memory，即被两个以上同时运行的程序所使用的内存)。

####内存   内存越界  内存泄漏  野指针

     申请、初始化、分配、释放、指向空。

* 指针初始化为NULL，以防不能立即传给它有效值的情况
* GCC和Clang都有-Wuninitialized参数来警告未初始化的变量
* 静态和动态分配的内存不要用同一个变量
* 调用 free 后要把指针设回为NULL，这样一来即便无意中重复释放也不会导致错误
* 测试或调试时使用assert之类的断言

####固定数组大小数组做函数参数：

* 传首地址，和数组大小    void putValues(int[], int size);
* 传数组指针的引用   void putValues(int (&arr)[10]);指定只能是a［10］
* 引入某种规则来结束一个数组   void putValues(char *strings[])

####数组动态分配 －－盲点

####多维数组做函数参数：

这里需要注意的是：C 语言中，当一维数组作为函数参数的时候，编译器总是把它解析成一个指向其首元素首地址的指针。这条规则并不是递归的，也就是说只有一维数组才是如此，当数组超过一维时，将第一维改写为指向数组首元素首地址的指针之后，后面的维再也不可改写。比如：a[3][4][5]作为参数时可以被改写为（*p）[4][5]。

####数组做函数参数：

      传值
      传地址
      传地址的引用

    void GetMemory（char * p, int num）
    {
        p = (char *)malloc(num*sizeof(char));
    }

    int main()
    {
        char *str = NULL;
        GetMemory（str，10）;
        strcpy(str,”hello”);
        free（str）；//free 并没有起作用，内存泄漏
        return 0;
    }

* return。

    char * GetMemory（char * p, int num）
    {
        p = (char *)malloc(num*sizeof(char));
        return p；
    }

    int main()
    {
        char *str = NULL;
        str = GetMemory（str，10）;
        strcpy(str,”hello”);
        free（str）；
        return 0;
    }

这个方法简单，容易理解。

* 二级指针。
    void GetMemory（char ** p, int num）
    {
        *p = (char *)malloc(num*sizeof(char));
    }
    int main()
    {
        char *str = NULL;
        GetMemory（&str，10）;
        strcpy(str,”hello”);
        free（str）；
        return 0;
    }

注意，这里的参数是&str 而非str。这样的话传递过去的是str 的地址，是一个值。
在函数内部，用钥匙（“*”）来开锁：*(&str)，其值就是str。所以malloc 分配的内存地址是真正赋值给了str 本身。

* 传地址引用

    char * GetMemory（char * &p, int num）
    {
        p = (char *)malloc(num*sizeof(char));
    }

    intmain()
    {
        char *str = NULL;
        str = GetMemory（str，10）;
        strcpy(str,”hello”);
        free（str）；
        return 0;
    }

###提高篇

熟悉系统 API，并知晓某些常用接口的陷阱，及一些 hack 用法（至少仔细通读过一遍 APUE）
分析著名开源项目核心 C 代码。如 node, python, lua 源码

值得分析的开源项目代码

1 lmdb

这是一个经过优化的代码，代码可读性不好，但是非常高效，因为是一个存储引擎，所以非常你会了解到一个存储系统的实现

2 node

这个代码库，你会了解到高性能web server 的设计

3 python lua 源码

如何正确使用基础系统API


待续.....
