
The weakref module supports weak references to objects. A normal reference increments the reference count on the object and prevents it from being garbage collected. This is not always desirable, either when a circular reference might be present or when building a cache of objects that should be deleted when memory is needed.

	#tc_weakref.py
	#! /usr/bin/env python
	import weakref

	class ExpensiveObject(object):
		def __init__(self, name):
		    self.name = name
		def __del__(self):
		    print '(Deleting %s)' % self

	def callback(reference):
    	"""Invoked when referenced object is deleted"""
    	print 'callback(', reference, ')'

	obj = ExpensiveObject('My Object')
    obj1 = obj
    r1 = weakref.ref(obj)
    r2 = weakref.ref(obj, callback)
	p = weakref.proxy(obj)

	#first
	print 'via obj:', obj.name
	print 'via obj1:', obj1.name
	print 'via ref:', r().name
	print 'via proxy:', p.name

	#second
    del obj
	print 'via obj:', obj.name
	print 'via obj1:', obj1.name
	print 'via ref:', r().name
	print 'via ref1:', r1().name
	print 'via proxy:', p.name
	
	del obj1
	print 'via obj:', obj.name
	print 'via obj1:', obj1.name
	print 'via ref:', r().name
	print 'via ref1:', r1().name
	print 'via proxy:', p.name


###Caching Objects

The ref and proxy classes are considered “low level”. While they are useful for maintaining weak references to individual objects and allowing cycles to be garbage collected, if you need to create a cache of several objects the WeakKeyDictionary and WeakValueDictionary provide a more appropriate API.

As you might expect, the WeakValueDictionary uses weak references to the values it holds, allowing them to be garbage collected when other code is not actually using them.

To illustrate the difference between memory handling with a regular dictionary and WeakValueDictionary, let’s go experiment with explicitly calling the garbage collector again:

	import gc
	from pprint import pprint
	import weakref

	gc.set_debug(gc.DEBUG_LEAK)

	class ExpensiveObject(object):
		def __init__(self, name):
		    self.name = name
		def __repr__(self):
		    return 'ExpensiveObject(%s)' % self.name
		def __del__(self):
		    print '(Deleting %s)' % self
		    
	def demo(cache_factory):
		# hold objects so any weak references 
		# are not removed immediately
		all_refs = {}
		# the cache using the factory we're given
		print 'CACHE TYPE:', cache_factory
		cache = cache_factory()
		for name in [ 'one', 'two', 'three' ]:
		    o = ExpensiveObject(name)
		    cache[name] = o
		    all_refs[name] = o
		    del o # decref

		print 'all_refs =',
		pprint(all_refs)
		print 'Before, cache contains:', cache.keys()
		for name, value in cache.items():
		    print '  %s = %s' % (name, value)
		    del value # decref
		    
		# Remove all references to our objects except the cache
		print 'Cleanup:'
		del all_refs
		gc.collect()

		print 'After, cache contains:', cache.keys()
		for name, value in cache.items():
		    print '  %s = %s' % (name, value)
		print 'demo returning'
		return

	demo(dict)
	print
	demo(weakref.WeakValueDictionary)
	


Example 
---------------------
    import weakref, time
    class A:
        def __del__(self):
            print 'class A been __del__'

    class B:
        def make_a(self):
            refa = A()
            self.a_ref = weakref.ref(refa)
            return refa
    b = B()
    print '+make_a'
    b.make_a()
    print '-make_a'
    print '+x = b.make_a()'
    x = b.make_a()
    print '-x = b.make_a()'
    print '+time.sleep(10)'
    time.sleep(10)
    print '-time.sleep(10)'

執行的結果如下：

    +make_a
    class A been __del__
    -make_a
    +x = b.make_a()
    -x = b.make_a()
    +time.sleep(10)
    -time.sleep(10)
    class A been __del__

首先，__del__ 是物件被刪掉的時候會被呼叫的函式。
接下來，雖然在 class B 裡面，有用 self.a_ref 去接了類別 A 的實例 refa，
但是，若沒有在呼叫 b.make_a() 的時候用個變數去接 refa，馬上 refa 就被刪掉了。

若是有用個變數 x 去接 refa，

    x = b.make_a()

那就可以撐到執行完了！

sys.getrefcount() 查看对象引用计数。


>>>　import　sys
>>>　import　weakref
>>>　class　Class1:
　　def　test(self):
　　　　print　"test..."
　　　　
>>>　o　=　Class1()
>>>　sys.getrefcount(o)
2
>>>　r　=　weakref.ref(o)　#　创建一个弱引用
>>>　sys.getrefcount(o)　#　引用计数并没有改变
2
>>>　r
<weakref　at　00D3B3F0;　to　'instance'　at　00D37A30>　#　弱引用所指向的对象信息
>>>　o2　=　r()　#　获取弱引用所指向的对象
>>>　o　is　o2
True
>>>　sys.getrefcount(o)
3
>>>　o　=　None
>>>　o2　=　None
>>>　r　#　当对象引用计数为零时，弱引用失效。
<weakref　at　00D3B3F0;　dead>
	
