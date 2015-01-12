

##Thread 

在Java中，“线程”指两件不同的事情

* java.lang.Thread类的一个实例
* 线程的执行。 
    使用 java.lang.Thread 类或者 java.lang.Runnable 接口编写代码来定义、实例化和启动新线程。
    一个 Thread 类实例只是一个对象，像 Java 中的任何其他对象一样，具有变量和方法，生死于堆上。
    Java中，每个线程都有一个调用栈，即使不在程序中创建任何新的线程，线程也在后台运行着。
    一个 Java 应用总是从main()方法开始运行，mian()方法运行在一个线程内，它被称为主线程。(非常关键)
    一旦创建一个新的线程，就产生一个新的调用栈。
    线程总体分两类：用户线程和守候线程。
    当所有用户线程执行完毕的时候，JVM自动关闭。但是守候线程却不独立于JVM，守候线程一般是由操作系统或者用户自己创建的。
    线程睡眠到期自动苏醒，并返回到可运行状态，不是运行状态。
 

##ThreadLocal

###ThreadLocal的定义和用途的概述（我的理解）：

它是一个线程级别变量，在并发模式下是绝对安全的变量，也是线程封闭的一种标准用法（除了局部变量外），即使你将它定义为static，它也是线程安全的。



###ThreadLocal能做什么呢？

这个一句话不好说，我们不如来看看实际项目中遇到的一些困解：当你在项目中根据一些参数调用进入一些方法，然后方法再调用方法，进而跨对象调用方法，很多层次，这些方法可能都会用到一些相似的参数，例如，A中需要参数a、b、c，A调用B后，B中需要b、c参数，而B调用C方法需要a、b参数，此时不得不将所有的参数全部传递给B，以此类推，若有很多方法的调用，此时的参数就会越来越繁杂，另外，当程序需要增加参数的时候，此时需要对相关的方法逐个增加参数，是的，很麻烦，相信你也遇到过，这也是在C语言面向对象过来的一些常见处理手段，不过我们简单的处理方法是将它包装成对象传递进去，通过增加对象的属性就可以解决这个问题，不过对象通常是有意义的，所以有些时候简单的对象包装增加一些扩展不相关的属性会使得我们class的定义变得十分的奇怪，所以在这些情况下我们在架构这类复杂的程序的时候，我们通过使用一些类似于Scope的作用域的类来处理，名称和使用起来都会比较通用，类似web应用中会有context、session、request、page等级别的scope，而ThreadLocal也可以解决这类问题，只是他并不是很适合解决这类问题，它面对这些问题通常是初期并没有按照scope以及对象的方式传递，认为不会增加参数，当增加参数时，发现要改很多地方的地方，为了不破坏代码的结构，也有可能参数已经太多，已经使得方法的代码可读性降低，增加ThreadLocal来处理，例如，一个方法调用另一个方法时传入了8个参数，通过逐层调用到第N个方法，传入了其中一个参数，此时最后一个方法需要增加一个参数，第一个方法变成9个参数是自然的，但是这个时候，相关的方法都会受到牵连，使得代码变得臃肿不堪。

上面提及到了ThreadLocal一种亡羊补牢的用途，不过也不是特别推荐使用的方式，它还有一些类似的方式用来使用，就是在框架级别有很多动态调用，调用过程中需要满足一些协议，虽然协议我们会尽量的通用，而很多扩展的参数在定义协议时是不容易考虑完全的以及版本也是随时在升级的，但是在框架扩展时也需要满足接口的通用性和向下兼容，而一些扩展的内容我们就需要ThreadLocal来做方便简单的支持。


###如何使用ThreadLocal？

在系统中任意一个适合的位置定义个ThreadLocal变量，可以定义为public static类型（直接new出来一个ThreadLocal对象），要向里面放入数据就使用set(Object)，要获取数据就用get()操作，删除元素就用remove()，其余的方法是非public的方法，不推荐使用。

	public class ThreadLocalTest2 {
	
		public final static ThreadLocal <String>TEST_THREAD_NAME_LOCAL = new ThreadLocal<String>();

		public final static ThreadLocal <String>TEST_THREAD_VALUE_LOCAL = new ThreadLocal<String>();
	
		public static void main(String[]args) {
			for(int i = 0 ; i < 100 ; i++) {
				final String name = "线程-【" + i + "】";
				final String value =  String.valueOf(i);
				new Thread() {
					public void run() {
						try {
							TEST_THREAD_NAME_LOCAL.set(name);
							TEST_THREAD_VALUE_LOCAL.set(value);
							callA();
						}finally {
							TEST_THREAD_NAME_LOCAL.remove();
							TEST_THREAD_VALUE_LOCAL.remove();
						}
					}
				}.start();
			}
		}
	
		public static void callA() {
			callB();
		}
	
		public static void callB() {
			new ThreadLocalTest2().callC();
		}
	
		public void callC() {
			callD();
		}
	
		public void callD() {
			System.out.println(TEST_THREAD_NAME_LOCAL.get() + "\t=\t" + TEST_THREAD_VALUE_LOCAL.get());
		}
	}


相信看到这里，很多程序员都对ThreadLocal的原理深有兴趣，看看它是如何做到的，尽然参数不传递，又可以像局部变量一样使用它，的确是蛮神奇的，其实看看就知道是一种设置方式，看到名称应该是是和Thread相关，那么废话少说，去看看它的源码吧。

源码总结：ThreadLocal 中有一个ThreadLocalMap类，每个Thread对象都会有自己的ThreadLocalMap对象，Thread本身不能调用ThreadLocalMap对象的get set remove方法。只能通过ThreadLocal 这个对象去调用。那么ThreadLocal 是怎么调用的呢？通过获取当前线程，然后进而得到当前线程的ThreadLocalMap对象，然后调用Map的get set remove 方法。既然是Map，那么map的key是什么呢？map的key是ThreadLocal 这个对象。 ok 。实现了数据只能是线程的局部变量要求。
 
 
Thread里面有个属性是一个类似于HashMap一样的东西，只是它的名字叫ThreadLocalMap，这个属性是default类型的，因此同一个package下面所有的类都可以引用到，因为是Thread的局部变量，所以每个线程都有一个自己单独的Map，相互之间是不冲突的，所以即使将ThreadLocal定义为static线程之间也不会冲突。

2、ThreadLocal和Thread是在同一个package下面，可以引用到这个类，可以对他做操作，此时ThreadLocal每定义一个，用this作为Key，你传入的值作为value，而this就是你定义的ThreadLocal，所以不同的ThreadLocal变量，都使用set，相互之间的数据不会冲突，因为他们的Key是不同的，当然同一个ThreadLocal做两次set操作后，会以最后一次为准。

3、综上所述，在线程之间并行，ThreadLocal可以像局部变量一样使用，且线程安全，且不同的ThreadLocal变量之间的数据毫无冲突。











###ThreadLocal 的坑?

1. 不能放置全局变量，只能放置线程私有的对象

换个思路和说法：“根本没有必要这样做”，因为全局静态变量本身都可以直接或间接引用到，为啥要用ThreadLocal呢，或者说ThreadLocal本身一般就定义为全局静态类型，才方便大家使用线程私有参数，且很关键的是这样使用线程是安全。


大家从前面应该可以看出来，这个ThreadLocal相关的对象是被绑定到一个Map中的，而这个Map是Thread线程的中的一个属性，那么就有一个问题是，如果你不自己remove的话或者说如果你自己的程序中不知道什么时候去remove的话，那么线程不注销，这些被set进去的数据也不会被注销。

反过来说，写代码中除非你清晰的认识到这个对象应该在哪里set，哪里remove，如果是模糊的，很可能你的代码中不会走remove的位置去，或导致一些逻辑问题，另外，如果不remove的话，就要等线程注销，我们在很多应用服务器中，线程是被复用的，因为在内核分配线程还是有开销的，因此在这些应用中线程很难会被注销掉，那么向ThreadLocal写入的数据自然很不容易被注销掉，这些可能在我们使用某些开源框架的时候无意中被隐藏用到，都有可能会导致问题，最后发现OOM得时候数据竟然来自ThreadLocalMap中，还不知道这些数据是从哪里设置进去的，所以你应当注意这个坑，可能不止一个人掉进这个坑里去过。


###不使用的set进去的对象，不用时remove掉，一定是这样么？

总体上确实是这样的，细化的时候会分一些场景，如果每次set进去的数据不大，且没有set类似集合类这样的数据进去（且集合类不断叠加），问题不太大。因为set进去的值每次会被替换掉。但不知绝对的，举个例子，SimpleDateFormat的format()方法其实是非线程安全的，如果将它定义为静态属性，则可能会出现偶然性的不可预测问题，但如果每个使用到它的地方都去new一个SimpleDateFormat的话，它是具有一定的编译成本的，在压测的时候可以发现CPU的开销会有明显提高。这个时候会尝试在ThreadLocal里面来做，没有就set，有就直接用，利用线程池对线程的复用来保存。虽然会创建很多个SimpleDateFormat对象，且在线程未注销前不会注销，但可以算下WEB线程数，和对象空间其实并不大。


http://blog.csdn.net/xieyuooo/article/details/8599266
