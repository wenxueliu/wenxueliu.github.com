---
layout: post
category : cpp 
comments : true 
tags : [list pointer deconstructor]
---
{% include JB/setup %}

如果用容器存副本，则容器销毁的时候，副本也会自动被删除。

如果用容器存指针，则容器销毁的时候，不会删除这些指针所指向的对象，因此必须先手工删除完毕之后，再销毁容器。

    list<Foo*> foo_list;

###Method 1 

	static bool deleteAll( Foo * theElement ) { delete theElement; return true; }
	foo_list . remove_if ( deleteAll );
	

###Method 2 

	template< typename T >
	struct delete_ptr : public std::unary_function<bool,T>
	{
	   bool operator()(T*pT) const { delete pT; return true; }
	};

	std::for_each( foo_list.begin(), foo_list.end(), delete_ptr<Foo>() );
	

###Method 3 
    
	for(list<Foo*>::const_iterator it = foo_list.begin(); it != foo_list.end(); it++)
	{
		delete *it;
	} 
	foo_list.clear();

	for(auto &it:foo_list) delete it; foo_list.clear();  #c++11


###Method 4

	while(!foo.empty()) delete foo.front(), foo.pop_front();  #std::list
	template<typename Container>
	void delete_them(Container& c) 
	{ 
		while(!c.empty()) delete c.back(), c.pop_back();
	}
	
	while(!bar.empty()) delete bar.back(), bar.pop_back();    #std::vector


###Method 5

At least for a list, iterating and deleting, then calling clear at the end is a bit inneficient 
since it involves traversing the list twice, when you really only have to do it once. 

Here is a little better way:

	for (list<Foo*>::iterator i = foo_list.begin(), e = foo_list.end(); i != e; )
	{
		list<Foo*>::iterator tmp(i++);
		delete *tmp;
		foo_list.erase(tmp);
	} 
	
###MORE 0

	template <typename T>
	class Deleter {
	public:
	  Deleter(T* pointer) : pointer_(pointer) { }
	  Deleter(const Deleter& deleter) {
		Deleter* d = const_cast<Deleter*>(&deleter);
		pointer_ = d->pointer_;
		d->pointer_ = 0;
	  }
	  ~Deleter() { delete pointer_; }
	  T* pointer_;
	};

Example

	std::list<Deleter<Foo> > foo_list;
	foo_list.push_back(new Foo());
	foo_list.clear();
	

 

###MORE 1

The request was to "safely clean up" the container. But the safety of that code relies on several
 methods not throwing exceptions: begin(), end(), iterator::operator!=, iterator::operator\*, 
 iterator::operator++. Surpirsingly, such reliance is unsafe
 
 One of the pitfalls is that, perversely, the STL allows several important iterator operations to 
 throw exceptions. This makes many "obvious" approaches using iteration through a container unsafe. 

(boost pointer container library)[http://www.boost.org/doc/libs/1_37_0/libs/ptr_container/doc/ptr_container.html]

###MORE2

Actually, I believe the STD library provides a direct method of managing memory in the form of the allocator class

You can extend the basic allocator's deallocate() method to automatically delete the members of any container.

I /think/ this is the type of thing it's intended for.


Reference [ Here ](http://stackoverflow.com/questions/307082/cleaning-up-an-stl-list-vector-of-pointers)
