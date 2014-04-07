---
layout: post
category : cpp/c 
tagline: "柔性数组"
tags : [ c++/c, linux]
---
{% include JB/setup %}

##概述
  柔性数组(flexible array)分配一段连续的的内存，减少内存的碎片化，简化内存的管理

##柔性数组用途 
* Socket通信数据包的传输
* 解析数据包

##测试

###源文件

    #include <iostream>
    #include <string.h>
    #include <memory.h>
    #include <stdlib.h>
    using namespace std;
    const int BUF_SIZE = 100;

    typedef struct ScaleArray1
    {
        int cnt;
        char *pBuf;
    }SArray1, *pSArray1;

    typedef struct ScaleArray2
    {
        int cnt;
        char pBuf[0];
    }SArray2, *pSArray2;

    typedef struct ScaleArray3
    {
        int cnt;
        char pBuf[1];
    }SArray3, *pSArray3;

    int main()
    {
        //赋值用
        const char* tmp_buf = "abcdefghijklmnopqrstuvwxyz";
        int ntmp_buf_size = strlen(tmp_buf);
        
        //<1>注意s_one 与s_two的大小的不同
        cout<< "sizeof(SArray1) = " << sizeof(SArray1) << endl; //8
        cout<< "sizeof(SArray2) = " << sizeof(SArray2) << endl; //4
        cout<< "sizeof(SArray3) = " << sizeof(SArray3) << endl;//5-->8结构体对齐
        cout<< endl;
        
        //为buf分配100个字节大小的空间
        int ntotal_stwo_len = sizeof(SArray2) + (1 + ntmp_buf_size) * sizeof(char);
        int ntotal_sthree_len = sizeof(SArray3) + ntmp_buf_size * sizeof(char);
        
        //给s_one buf赋值
        //******c-style******
        pSArray1 pOne= (pSArray1)malloc(sizeof(SArray1));
        memset(pOne,0, sizeof(SArray1));
        pOne->pBuf = (char*)malloc(1 + ntmp_buf_size);
        memset(pOne->pBuf,0, 1 + ntmp_buf_size);
        memcpy(pOne->pBuf,tmp_buf, ntmp_buf_size);
        //******cpp-style******
        //pSArray1 pOne = new SArray1[sizeof(SArray1)];
        //pOne->pBuf = new char[1+ntmp_buf_size]();
        //memcpy(pOne->pBuf,tmp_buf,ntmp_buf_size);
        
        cout<< "pOne->pBuf = " << pOne->pBuf << endl;

        if(NULL != pOne->pBuf)
        {
                //free(p_sone->s_one_buf);
                delete []pOne->pBuf;
                pOne->pBuf = NULL;
        
                if(NULL != pOne)
                {
                    //free(p_sone);
                    delete []pOne;
                    pOne= NULL;
                }
                cout<< "free(pOne) successed!" << endl;
        }
        
        //给s_two buf赋值
        //******c-style******
        pSArray2 pTwo = (pSArray2)malloc(ntotal_stwo_len);
        memset(pTwo,0, ntotal_stwo_len);
        memcpy((char*)(pTwo->pBuf),tmp_buf, ntmp_buf_size);  
        //******cpp-style******
        //pSArray2 pTwo = reinterpret_cast<pSArray2>(new char[ntotal_stwo_len]);
        //memcpy((char*)(pTwo->pBuf), tmp_buf, ntmp_buf_size);

        cout<< "pTwo->pBuf = " << pTwo->pBuf << endl;

        if(NULL != pTwo)
        {
                //free(p_stwo);
                delete [] pTwo;
                pTwo = NULL;
        
                cout<< "free(pTwo) successed!" << endl;
        }
        
        //给s_three_buf赋值 //******c-style******
        pSArray3 pThree = (pSArray3)malloc(ntotal_sthree_len);
        memset(pThree,0, ntotal_sthree_len);
        memcpy((char*)(pThree->pBuf), tmp_buf, ntmp_buf_size);
        //******cpp-style******
        //pSArray3 pThree = reinterpret_cast<pSArray3>(new char[ntotal_sthree_len]);
        //memcpy((char*)(pThree->pBuf), tmp_buf, ntmp_buf_size);
        
        cout<< "pThree->pBuf = " << pThree->pBuf << endl; //不用加偏移量，直接拷贝!
        
        if(NULL != pThree)
        {
                //free(p_sthree);
                delete [] pThree; 
                pThree = NULL;
        
                cout<< "free(p_sthree) successed!" << endl;
        }
        
        return 0;
    }

###makefile

   	main: scale_array.o
		g++ scale_array.o -o main

	scale_array.o: scale_array.cpp 
		g++ -c scale_array.cpp

###运行

1. vim scale_array.cpp 打开源文件
2. `:make` 编译
3. `:cw` 查看编译结果
4. `:!./main` 查看执行结果

###三种方式对比

* 存储大小方面：Scale_Array2的存储较Scale_Array1、ScaleArray3都要少，[0]的好处，即用指针的方式需要多开辟存储空间的。
* 数据连续存储方面：ScaleArray1明显数据域是单独开辟的空间，与前的cnt不在连续的存储区域，而ScaleArray2，ScaleArray3则在连续的存储空间下。
* 释放内存方面：显然ScaleArray1的指针的方式，需要先释放数据域部分，才能释放指向结构体的指针变量；而ScaleArray2，ScaleArray3可以直接释放。

##编译器支持
C89 不支持这种东西,C99 把它作为一种特例加入了标准。但是,C99 所支持的是 incomplete type,而不是 zero array,形同 int
item[0];这种形式是非法的,C99 支持的形式是形同 int item[];只不过有些编译器把 int item[0];作为非标准扩展来支持,而且在
C99 发布之前已经有了这种非标准扩展了,C99 发布之后,有些编译器把两者合而为一了。


