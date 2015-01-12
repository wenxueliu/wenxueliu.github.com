##ThreadLocal的定义

它是一个线程级别变量，在并发模式下是绝对安全的变量，也是线程封闭的一种标准用法（除了局部变量外），即使你将它定义为static，它也是线程安全的。

ThreadLocal 中有一个ThreadLocalMap类，每个Thread对象都会有自己的ThreadLocalMap对象，Thread本身不能调用ThreadLocalMap对象的get set remove方法。只能通过ThreadLocal 这个对象去调用。那么ThreadLocal 是怎么调用的呢？通过获取当前线程，然后进而得到当前线程的ThreadLocalMap对象，然后调用Map的get set remove 方法。既然是Map，那么map的key是什么呢？map的key是ThreadLocal 这个对象。 ok 。实现了数据只能是线程的局部变量要求。

##使用场景

一个方法调用另一个方法时传入了8个参数，通过逐层调用到第N个方法，传入了其中一个参数，此时最后一个方法需要增加一个参数，第一个方法变成9个参数是自然的，但是这个时候，相关的方法都会受到牵连，使得代码变得臃肿不堪。

在框架级别有很多动态调用，调用过程中需要满足一些协议，虽然协议我们会尽量的通用，而很多扩展的参数在定义协议时是不容易考虑完全的以及版本也是随时在升级的，但是在框架扩展时也需要满足接口的通用性和向下兼容，而一些扩展的内容我们就需要ThreadLocal来做方便简单的支持。

So what is a thread-local variable? A thread-local variable is one whose value at any one time is linked to which thread it is being accessed from. In other words, it has a separate value per thread. Each thread maintains its own, separate map of thread-local variable values. (Many operating systems actually have native support for thread-local variables, but in Sun's implementation at least, native support is not used, and thread-local variables are instead held in a specialised type of hash table attached to the Thread.)

Thread-local variables are used via the ThreadLocal class in Java. We declare an instance of ThreadLocal, which has a get() and set() method. A call to these methods will read and set the calling thread's own value.
