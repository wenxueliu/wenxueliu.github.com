---
layout: post
category : cpp 
comments : false
tags : [stl container pointer, destructor]
---
{% include JB/setup %}

最近看了下STL，用的过程中有一些体会需要记一下。

容器的空间申请和基本函数操作，以及 algorithm 等都比较好理解，用起来也很方便，比较关键的是容器元素包含指针时，
空间的申请和释放问题，这个觉得STL做得挺乱的。总结了几点注意的。

自己 new 的空间，在释放的时候必须先 delete，然后再释放容器。例如 list\<char\*\> MS，链表中存的是自己的动态字符串，
如果字符串是自己动态申请的，则在释放该链表的时候，需要先delete
\[\]\(\*curIter\)，然后再 MS.erase\(curIter\)，其中
curIter 是当前迭代器。

list的成员函数 erase、remove 和 clear 都会自动调用元素各自的析构函数，所以如果元素是自己定义的类，并且有完善的
析构函数，则直接删除即可。这类链式存储，一个元素一个元素递增空间的结构，这些函数可以真正地改变 list 占用的内存大小。

vector等的空间释放方式。需要注意两点：

理解erase-remove惯用法，包括 erase 和 remove 函数的返回值等，[这篇文章](http://blog.csdn.net/peteryxk/article/details/1804696)中讲的比较清楚，下面是一个代码示例：
　　

	/*删除vector对象vec中等于10的元素，第一种方法*/
	for ( begin = vec.begin() ; begin != vec.end() ; /* ++begin */ )
	{
	 	if ( 10 == *begin )
	 	{
	 		begin = vec.erase( begin ) ;
	 	}
	 	else
	 	{
			++begin ;
		}
	}
	/*remove-erase方法*/
	vec.erase(remove (vec.begin() ,vec.end(),10) , vec.end()) ;

需要搞明白remove和erase在这种顺序结构中的工作原理。

由于 vector 的空间是阶梯递增式管理的，而且基本只增不减，也就是，虽然调用 remove、erase 或者 clear 等方法（他们会调用所存元素对象的析构函数，如果元素是用户定义的类，并且有完善析构函数释放掉该类的对象动态申请的空间，则 remove 和 erase 确实会释放掉一些内存，但是容器本身的空间还是难以被利用），容器之前分配的空间仍不会被回收，大小不变，仍旧不能被其他程序使用，这是由 STL 的内存管理机制决定的，目的是为了提高效率。那么应该怎么回收 vector 的内存呢？这里就用了一个小的技巧，swap方法，代码如下：

 	{ 
     	std::vector<int> tmp;   
     	vec.swap(tmp); 
  	}//使用一个局部变量空的容器temp，与vec交换，退出temp作用域后，temp会释放自己的空间，而此时vec已经是空的容器    

list由于是链式存储的，跟vector机制不一样，他的成员函数 remove、erase、pop_front、pop_back 和 clear 等都会回收被删除元素的内存空间，不需要 swap 协助。

　　对于vector，作如下实验就可以看到，它的空间其实一直还是保留着的，除非swap掉：


     vector<int> vec;
 
     for(int i=1;i<=10;i++)
     {
         vec.push_back(i);
         cout<<vec.size()<<"  "<<vec.capacity()<<"  "<<vec.max_size()<<endl;
     }
     cout<<endl;
     for(int i=1;i<=10;i++)
     {
         vec.erase(remove(vec.begin(),vec.end(),i),vec.end());
         cout<<vec.size()<<"  "<<vec.capacity()<<endl;
     }
     cout<<endl;
     if(true)
     {
         vector<int> temp;
         vec.swap(temp);
     }
     cout<<vec.size()<<"  "<<vec.capacity()<<endl;


运行结果如下：

	1  1  4611686018427387903
	2  2  4611686018427387903
	3  4  4611686018427387903
	4  4  4611686018427387903
	5  8  4611686018427387903
	6  8  4611686018427387903
	7  8  4611686018427387903
	8  8  4611686018427387903
	9  16  4611686018427387903
	10  16  4611686018427387903

	9  16
	8  16
	7  16
	6  16
	5  16
	4  16
	3  16
	2  16
	1  16
	0  16

	0  0
	



下面给出 erase（删除迭代器区间的内容）和 remove 函数（一般用来删除容器中等于特定值的内容）的返回值定义，以方便理解3）A.

　　List::erase() Return value:

A bidirectional iterator pointing to the new location of the element that followed the last element erased by the function call, which is the list end if the operation erased the last element in the sequence.

　　List::remove() Return value: None

　　Vector::erase() Return value:

A random access iterator pointing to the new location of the element that followed the last element erased by the function call, which is the vector end if the operation erased the last element in the sequence.

　　非成员函数，算法库中的remove() Return value：

A forward iterator pointing to the new end of the range that includes all the elements with a value that does not compare equal to value.

关于STL的内存管理，有一篇[文章](http://philoscience.iteye.com/blog/1456509)中好像说得挺清楚的，有空再看看。
