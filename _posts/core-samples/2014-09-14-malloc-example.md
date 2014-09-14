---
layout: post
category : language
tagline: "c malloc 的一些例子"
tags : [clang, linux]
---
{% include JB/setup %}

###Example

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	int main()
	{
		int len1, len2;
		char *p1, *p2, *p3;
		if((p1 = (char *)malloc(0)) == NULL)
		    puts("Got a null pointer");
		else
		   puts("Got a valid pointer");
		if((len1 = (strlen(p2 = (char *)malloc(0)))) == 0)
		    puts("Got a null pointer");
		else
		    puts("Got a valid pointer");
		if((len2 = sizeof(p3 = (char *)malloc(0))) == 4)
		    puts("Got a null pointer");
		else
		    puts("Got a valid pointer");
	   return 0;
	}

输出：

	Got a valid pointer
	Got a null pointer
	Got a null pointer

###Example

	#include <stdio.h>
	int main()
	{
		*(int *)0x0012ff7c = 0xaabb;
		printf("%x", *(int *)0x0012ff7c);    
		return 0;
	}

输出

	aabb

gcc无法修改指定地址。但是可以修改常量的地址值。



###Example

	#include <stdio.h>
	int main()
	{
		unsigned int zero = 0;
		unsigned int compzero1 = 0xFFFF; /* incorrect assignment */
		unsigned int compzero2 = ~0; /* 1 is complement of zero */
		printf("%#X \n", zero);
		printf("%#X \n", compzero1);
		printf("%#X \n", compzero2);
		return 0;
	}

问题：

	char* ptr = malloc(0*sizeof(char));
	if(NULL == ptr)
      	printf("got a NULL pointer");
	else
     	printf("got a Valid pointer");

请问：上面的程序输出为什么？

在C99的标准里面解释到，如果给 malloc 传递 0 参数，其返回值是依赖于编译器的实现，但是不管返回何值，该指针指向的对象是不可以访问的。在VC6编译环境下，输出“got a Valid pointer”g++ 也是同样的结果。

但是我试图给该指针赋值，如：*ptr = "a" ;编译器并没有给出任何错误和警告信息，接着，我再输出该值，printf("*ptr=%d/n",*ptr) ;也可以正常输出。

但是当我用free(ptr) ;释放内存的时候，出现错误，为什么呢？下面是我看了网友经过讨论以后我比较认同的看法：

当malloc分配内存时它除了分配我们指定SIZE的内存块，还会分配额外的内存来存储我们的内存块信息，用于维护该内存块。因此，malloc（0）返回一个合法的指针并指向存储内存块信息的额外内存，我们当然可以在该内存上进行读写操作，但是这样做了会破坏该内存块的维护信息，因此当我们调用free(ptr)时就会出现错误。完整程序如下：

	#include <stdio.h>
	#include <stdlib.h>
	int main()
	{
	 char *ptr ;
	 ptr = malloc(0*sizeof(char)) ;
	 
	 if (NULL == ptr)
	  printf("got a NULL pointer/n");
	 else 
	 {
	  printf("got a Valid pointer/n");
	  *ptr = "a" //此处为字符串。如果为"a"则加free(ptr)仍然没有问题。
	  printf("the value at %X is:%c/n",ptr,*ptr);
	  free(ptr) ;//if we did not add this statement ,the program can run //normnlly，or we will get  a runtime error.
	 }
	 return 0 ;
	}

既然malloc另外分配内存来维护该内存块，也就是说分配来用于维护该内存块的内存的大小也是有限的，那么到底是多少呢？这和可能也依赖于实现，在VC6下，是56BYTE,下面是测试程序：

	#include 
	#include 
	#include
	int main()
	{
	 char *ptr ;
	 ptr = (char*)malloc(0*sizeof(char)) ;
	 
	 if (NULL == ptr)
	  printf("got a NULL pointer/n");
	 else 
	 {
	  printf("got a Valid pointer/n");
	  // 有56个a，另外有一个字节用于保存''/0'
	  strcpy(ptr,"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"); 
	  //printf("the value at %X is:%c/n",ptr,*ptr);
	  printf("the string at %x is :%s/n",ptr, ptr);
	 // free(ptr);
	 }
	 return 0 ;
	}

此时我们没有把free（ptr）编译进来，同样会发生异常，程序输出很多个56个a,我暂时还不明白为什么？？？？？如果把free(ptr);编译进来，就会发生运行错误！
通过上面的讨论和程序的验证，确实证明了网友和我的想法是正确的，也就是malloc(0)还会额外分配一部分空间（在VC6下是56字节）用于维护内存块。上面的程序在g++上运行正常。

关于这个问题的个人理解：

1. ptr = malloc(0*sizeof(char)) ;
ptr是局部指针变量，存储在栈中，它的值是动态分配的一块堆中的空间的首地址
所以说这个地址是合法的，但是由于malloc的大小是0，故这个这个地址指向的堆中的存储空间的大小是0，
这个指针类似于一个野指针，可以使用的，但是是有风险的，因为不知道这个指针后面的内存空间被谁使用着，要是被核心进程使用，哪肯定会造成相应程序的崩溃
2. 关于56个a的问题，我在本地测试是不存在的，我测试的是很多个都能打印出（用gcc测试）~
3. 关于加上free后，程序会崩溃，我理解是由于在堆中并没有对应的空间分配到导致的~

其中，p中的地址是堆内的首地址？ 
--------------------------------------------------------------- 

C99标准（ISO/IEC 9899:1999 (E)）上说： 

If the size of the space requested is zero, the behavior is implementationdefined: 
either a null pointer is returned, or the behavior is as if the size were some 
nonzero value, except that the returned pointer shall not be used to access an object.
如果所请求的空间大小为0，其行为由库的实现者定义：可以返回空指针，也可以让效果跟申请某个非0大小的空间一样，所不同的是返回的指针不可以被用来访问一个对象。
为什么 new T[0] 和 malloc(0) 不返回空指针
首先需要说明的是，按照C++标准，成功的 new T[0] 是不能返回空指针的。而 malloc(0)，C 语言标准则指出，成功的时候可以返回空指针，也可以返回非空指针，多数库一般也选择了返回非空指针这种行为。

为什么这么做呢？

1. 理念：0大小的对象也是对象。
2. 实践：返回空会和分配失败混淆，尤其是大小是计算出来的时候，这时如果得到的是空指针，用户程序会误以为分配失败了。
3. 实现：反正返回的地址不能读写，此时可以返回同一个固定的地址，并没什么额外开销。而且多数编译器，C和C++一起提供，C++库的new也用C库的malloc实现，使二者保持一致也比较省事。
4. 不管返回的是不是空，都可以free的。

