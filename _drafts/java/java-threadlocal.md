##ThreadLocal的定义

它是一个线程级别变量，在并发模式下是绝对安全的变量，也是线程封闭的一种标准用法（除了局部变量外），即使你将它定义为 static，它也是线程安全的。

ThreadLocal 中有一个ThreadLocalMap类，每个Thread对象都会有自己的ThreadLocalMap对象，Thread本身不能调用ThreadLocalMap对象的get set remove方法。只能通过ThreadLocal 这个对象去调用。那么ThreadLocal 是怎么调用的呢？通过获取当前线程，然后进而得到当前线程的ThreadLocalMap对象，然后调用Map的get set remove 方法。既然是Map，那么map的key是什么呢？map的key是ThreadLocal 这个对象。 ok 。实现了数据只能是线程的局部变量要求。

##使用场景

一个方法调用另一个方法时传入了8个参数，通过逐层调用到第N个方法，传入了其中一个参数，此时最后一个方法需要增加一个参数，第一个方法变成9个参数是自然的，但是这个时候，相关的方法都会受到牵连，使得代码变得臃肿不堪。

在框架级别有很多动态调用，调用过程中需要满足一些协议，虽然协议我们会尽量的通用，而很多扩展的参数在定义协议时是不容易考虑完全的以及版本也是随时在升级的，但是在框架扩展时也需要满足接口的通用性和向下兼容，而一些扩展的内容我们就需要ThreadLocal来做方便简单的支持。

So what is a thread-local variable? A thread-local variable is one whose value at any one time is linked to which thread it is being accessed from. In other words, it has a separate value per thread. Each thread maintains its own, separate map of thread-local variable values. (Many operating systems actually have native support for thread-local variables, but in Sun's implementation at least, native support is not used, and thread-local variables are instead held in a specialised type of hash table attached to the Thread.)

Thread-local variables are used via the ThreadLocal class in Java. We declare an instance of ThreadLocal, which has a get() and set() method. A call to these methods will read and set the calling thread's own value.

##实例

So when in practice would we use ThreadLocal? A typical example of using ThreadLocal would be as an alternative to an object or resource pool, when we don't mind creating one object per thread. Let's consider the example of a pool of Calendar instances. In an application that does a lot of date manipulation, Calendar classes may be a good candidates for pooling because: 

* Creating a Calendar is non-trivial (various calculations and accesses to localisation resources need to be made each time one is created); 

* There's no actual requirement to share Calendars between threads or have fewer calendars than threads. 

One (inefficient) way to re-use Calendars would be to create a 'calendar pool' class such as this:

    package org.test.Calendar
    import java.util.Calendar
    import java.util.GregorianCalendar

    public class CalendarFactory {
      private List calendars = new ArrayList();
      private static CalendarFactory instance = new CalendarFactory();

      public static CalendarFactory getFactory() { return instance; }

      public Calendar getCalendar() {
        synchronized (calendars) {
          if (calendars.isEmpty()) {
            return new GregorianCalendar();
          } else {
            return calendars.remove(calendars.size()-1);
          }
        }
      }
      public void returnCalendar(Calendar cal) {
        synchronized (calendars) {
          calendars.add(cal);
        }
      }
      // Don't let outsiders create new factories directly
      private CalendarFactory() {}
    }

Then a client could call: 

    Calendar cal = CalendarFactory.getFactory().getCalendar();
    try {
      // perform some calculation using cal
    } finally {
      CalendarFactory.getFactory().returnCalendar(cal);
    }


This would allow us to re-use Calendar objects but it's a bit inefficient because in each case it makes two synchronized calls when we're actually not that bothered about sharing the calendars across the threads. We wouldn't care, for example, if Thread 1 created and finished with a calendar and then Thread 2 created another, even though Thread 1 had finished with its. The number of threads will typically be small-ish (maybe in the hundreds at the very most), and so having as many calendars knocking around as there are threads is an OK compromise. (This would not be the case, for example, with database connections: if possible, we generally would want Thread 2 to use the connection that Thread 1 had just finished with rather than creating another; in such cases, thread-local variables aren't the right solution.) Now let's see how we can improve CalendarFactory using ThreadLocal:

当后续的线程不依赖与前面线程完成操作，这时候可以通过　ThreadLocal　来增强并发性能。对于数据库并发写操作，只有当前面的线程已经写完，后面的线程才可以写的情况，并不适合用　ThreadLocal。


    package org.test.Calendar
    import java.util.Calendar
    import java.util.GregorianCalendar

    public class CalendarFactory {
      private ThreadLocal<Calendar> calendarRef = new ThreadLocal<Calendar>() {
        protected Calendar initialValue() {
          return new GregorianCalendar();
        }
      };
      private static CalendarFactory instance = new CalendarFactory();

      public static CalendarFactory getFactory() { return instance; }

      public Calendar getCalendar() {
        return calendarRef.get();
      }

      // Don't let outsiders create new factories directly
      private CalendarFactory() {}
    }


Note that there is still a single, static instance of CalendarFactory shared by all threads. But that single instance uses the ThreadLocal variable calendarRef, which has a per-thread value. Inside getCalendar(), the call to calendarRef.get() will always operate on our thread-private "instance" of the variable, and we don't need any synchronization.

This example uses the Java 5 generics feature: we declare ThreadLocal as 'containing' Calendar objects, so that the subsequent get() method doesn't need an explicit cast. (That is, the cast is inserted automatically by the compiler.) Another feature of this example is the initialValue() method. We actually subclass ThreadLocal and override initialValue() to provide an appropriate object each time a new one is required (i.e. when get() is called for the first time on a particular thread). We could of course have simply checked for null inside the getCalendar() method (the first time get() is called on a ThreadLocal for a particular thread, it returns null) and set the value if it was null: 

    public Calendar getCalendar() {
      Calendar cal = calendarRef.get();
        if (cal == null) {
          calendarRef.set(cal = new GregorianCalendar());
        }
        return cal;
      }

However, overriding ThreadLocal.initialValue() automatically handles this logic and makes our code a bit neater– especially if we're calling get() in multiple places.

Note that we don't have a "return to pool" method. Once created, we let the calendar hang around for as long as the thread is alive. If we really wanted to remove these instances at a particular moment, then as of Java 5, we can call calendarRef.remove(), which removes the calling thread's value; it would be set to the initial value again on the next call to calendarRef.get(). 

**Notes**

All values set on a ThreadLocal also become garbage collectable if the ThreadLocal becomes no longer reachable outside of the Thread class. In other words, a thread's map of ThreadLocal to value holds on to the ThreadLocal only via a weak reference. 

###When to use ThreadLocal?

So what are other good candidates for object re-use via ThreadLocal? Basically, objects where: 

* The objects are non-trivial to construct; 
* An instance of the object is frequently needed by a given thread; 
* The application pools threads, such as in a typical server (if every time the thread-local is used it is from a new thread, then a new object will still be created on each call!); 
* It doesn't matter that Thread A will never share an instance with Thread B; 
* It's not convenient to subclass Thread. If you can subclass Thread, you could add extra instance variables to your subclass instead of using ThreadLocal. But for example, if you are writing a servlet running in an off-the-shelf servlet runner such as Tomcat, you generally have no control over the class of created threads. Of course, even if you can subclass Thread, you may simply prefer the cleaner syntax of ThreadLocal. 

That means that typical objects to use with ThreadLocal could be:

* Random number generators (provided a per-thread sequence was acceptable); 
* Collators; 
* native ByteBuffers (which in some environments cannot be destroyed once they're created); 
* XML parsers or other cases where creating an instance involves going through slightly non-trival code to 'choose a registered service provider'; 
* Per-thread information such as profiling data which will be periodically collated. 


Note that it is generally better not to re-use objects that are trivial to construct and finalize. (By "trivial to finalize", we mean objects that don't override finalize.) This is because recent garbage collector implementations are optimised for "temporary" objects that are constructed, trivially used and then fall out of scope without needing to be added to the finalizer queue. Pooling something trivial like a StringBuffer, Integer or small byte array can actually degrade performance on modern JVMs. 