

对象锁: 该对象被多个方法同时访问或修改, 同一时刻只能有一个正在操作,其他必须等待. 每个类对象之间不需要同步
方法锁: 这个方法只能有一个可以访问, 当多个线程试图访问时, 同一时刻只能由一个正在操作.
类锁: 该类的任意对象,都同时享有一个公共资源,同时只能有一个对象方法该资源.



虽然多线程编程极大地提高了效率，但是也会带来一定的隐患。比如说两个线程同时往一个数据库表中插入不重复的数据，就可能会导致
数据库中插入了相同的数据。今天我们就来一起讨论下线程安全问题，以及Java中提供了什么机制来解决线程安全问题

##什么时候会出现线程安全问题？

在单线程中不会出现线程安全问题，而在多线程编程中，有可能会出现同时访问同一个资源的情况，这种资源可以是各种类型的的资源：
一个变量、一个对象、一个文件、一个数据库表等，而当多个线程同时访问同一个资源的时候，就会存在一个问题：

由于每个线程执行的过程是不可控的，所以很可能导致最终的结果与实际上的愿望相违背或者直接导致程序出错。

举个简单的例子：现在有两个线程分别从网络上读取数据，然后插入一张数据库表中，要求不能插入重复的数据。
那么必然在插入数据的过程中存在两个操作：

* 检查数据库中是否存在该条数据；
* 如果存在，则不插入；如果不存在，则插入到数据库中。

假如两个线程分别用thread-1和thread-2表示，某一时刻，thread-1和thread-2都读取到了数据X，那么可能会发生这种情况：thread-1
去检查数据库中是否存在数据X，然后thread-2也接着去检查数据库中是否存在数据X。结果两个线程检查的结果都是数据库中不
存在数据X，那么两个线程都分别将数据X插入数据库表当中。

这个就是线程安全问题，即多个线程同时访问一个资源时，会导致程序运行结果并不是想看到的结果。

这里面，这个资源被称为：临界资源（也有称为共享资源）。

也就是说，当多个线程同时访问临界资源（一个对象，对象中的属性，一个文件，一个数据库等）时，就可能会产生线程安全问题。

不过，当多个线程执行一个方法，**方法内部的局部变量并不是临界资源，因为方法是在栈上执行的，而Java栈是线程私有的，因此不会产生线程安全问题**。

##如何解决线程安全问题？

那么一般来说，是如何解决线程安全问题的呢？

基本上所有的并发模式在解决线程安全问题时，都采用“序列化访问临界资源”的方案，即在同一时刻，只能有一个线程访问临界资源，也称作同步互斥访问。

通常来说，是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

在Java中，提供了两种方式来实现同步互斥访问：synchronized和Lock。


#synchronized

##synchronized同步方法

在了解synchronized关键字的使用方法之前，我们先来看一个概念：互斥锁，顾名思义：能到达到互斥访问目的的锁。

举个简单的例子：如果对临界资源加上互斥锁，当一个线程在访问该临界资源时，其他线程便只能等待。

在Java中，每一个对象都拥有一个锁标记（monitor），也称为监视器，多线程同时访问某个对象时，线程只有获取了该对象的锁才能访问。

在Java中，可以使用synchronized关键字来标记一个方法或者代码块，当某个线程调用该对象的synchronized方法或者访问synchronized代码块时，
这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，
其他线程才能执行这个方法或者代码块。

下面通过几个简单的例子来说明synchronized关键字的使用：

```java

//Test.java
public class Test {

    public static void main(String[] args)  {
        final InsertData insertData = new InsertData();

        new Thread() {
            public void run() {
                insertData.insert(Thread.currentThread());
            };
        }.start();

        new Thread() {
            public void run() {
                insertData.insert(Thread.currentThread());
            };
        }.start();
    }
}
```

```java
//InsertData.java
class InsertData {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();

    public void insert(Thread thread){
        for(int i=0;i<5;i++){
            System.out.println(thread.getName()+"在插入数据"+i);
            arrayList.add(i);
        }
    }
}
```

执行结果如下:

     [java] Thread-0得到了锁
     [java] Thread-1得到了锁
     [java] Thread-0在插入数据0
     [java] Thread-1在插入数据0
     [java] Thread-0在插入数据1
     [java] Thread-1在插入数据1
     [java] Thread-0在插入数据2
     [java] Thread-0在插入数据3
     [java] Thread-1在插入数据2
     [java] Thread-0在插入数据4
     [java] Thread-1在插入数据3
     [java] Thread-0释放了锁
     [java] Thread-1在插入数据4
     [java] Thread-1释放了锁

说明两个线程在同时执行insert方法。


而如果在insert方法前面加上关键字synchronized的话，运行结果为：

```java

	class InsertData {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();
		 
		public synchronized void insert(Thread thread){
		    for(int i=0;i<5;i++){
		        System.out.println(thread.getName()+"在插入数据"+i);
		        arrayList.add(i);
		    }
		}
	}
```

执行结果如下:

     [java] Thread-2得到了锁
     [java] Thread-2在插入数据0
     [java] Thread-2在插入数据1
     [java] Thread-2在插入数据2
     [java] Thread-2在插入数据3
     [java] Thread-2在插入数据4
     [java] Thread-2释放了锁
     [java] Thread-3得到了锁
     [java] Thread-3在插入数据0
     [java] Thread-3在插入数据1
     [java] Thread-3在插入数据2
     [java] Thread-3在插入数据3
     [java] Thread-3在插入数据4
     [java] Thread-3释放了锁


从上输出结果说明，Thread-3插入数据是等Thread-2插入完数据之后才进行的。说明Thread-2和Thread-3是顺序执行insert方法的。

这就是synchronized方法。

不过有几点需要注意：

* 当一个线程正在访问一个对象的 synchronized 方法，那么其他线程不能访问该对象的其他 synchronized 方法。这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他 synchronized 方法。
* 当一个线程正在访问一个对象的 synchronized 方法，那么其他线程能访问该对象的非 synchronized 方法。这个原因很简单，访问非 synchronized 方法不需要获得该对象的锁，假如一个方法没用 synchronized 关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，
* 如果一个线程 A 需要访问对象 object1 的 synchronized 方法 fun1，另外一个线程 B 需要访问对象 object2 的 synchronized 方法 fun1，即使 object1 和 object2 是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。

##synchronized代码块

synchronized代码块类似于以下这种形式：

	synchronized(synObject) {
		}


当在某个线程中执行这段代码块，该线程会获取对象synObject的锁，从而使得其他线程无法同时访问该代码块。

synObject 可以是 this，代表获取当前对象的锁，也可以是类中的一个属性，代表获取该属性的锁。

比如上面的 insert 方法可以改成以下两种形式：

```java

	class InsertData {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();

		public void insert(Thread thread){
		    synchronized (this) {
		        for(int i=0;i<100;i++){
		            System.out.println(thread.getName()+"在插入数据"+i);
		            arrayList.add(i);
		        }
		    }
		}
	}
```

```java
	class InsertData {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();
		private Object object = new Object();

		public void insert(Thread thread){
		    synchronized (object) {
		        for(int i=0;i<100;i++){
		            System.out.println(thread.getName()+"在插入数据"+i);
		            arrayList.add(i);
		        }
		    }
		}
	}
```

输出结果：

	Thread-4在插入数据0
     [java] insert3在插入数据0
     [java] insert3在插入数据1
     [java] insert3在插入数据2
     [java] Thread-4在插入数据1
     [java] Thread-4在插入数据2
     [java] Thread-4在插入数据3
     [java] Thread-4在插入数据4
	insert3在插入数据4

从上面可以看出，synchronized 代码块使用起来比 synchronized 方法要灵活得多。因为也许一个方法中只有一部分代码只需要同步，
如果此时对整个方法用 synchronized 进行同步，会影响程序执行效率。而使用 synchronized 代码块就可以避免这个问题，
synchronized 代码块可以实现只对需要同步的地方进行同步。

另外，每个类也会有一个锁，它可以用来控制对 static 数据成员的并发访问。

并且如果一个线程执行一个对象的非 static synchronized 方法，另外一个线程需要执行这个对象所属类的 static synchronized 方法，
此时不会发生互斥现象，因为访问 static synchronized 方法占用的是类锁，而访问非 static synchronized 方法占用的是对象锁，
所以不存在互斥现象。

看下面这段代码就明白了：


```java
	//Test.java
	public class Test {

		public static void main(String[] args)  {
		    final InsertData insertData = new InsertData();
		    new Thread(){
		        @Override
		        public void run() {
		            insertData.insert();
		        }
		    }.start();
		    new Thread(){
		        @Override
		        public void run() {
		            insertData.insert1();
		        }
		    }.start();
		}
	}
```

```java
	//InsertData.java
	class InsertData {
		public synchronized void insert(){
		    System.out.println("执行insert");
		    try {
		        Thread.sleep(5000);
		    } catch (InterruptedException e) {
		        e.printStackTrace();
		    }
		    System.out.println("执行insert完毕");
		}

		public synchronized static void insert1() {
		    System.out.println("执行insert1");
		    System.out.println("执行insert1完毕");
		}
	}

```

输出结果：

    待补


第一个线程里面执行的是insert方法，不会导致第二个线程执行insert1方法发生阻塞现象。

下面我们看一下synchronized关键字到底做了什么事情，我们来反编译它的字节码看一下，下面这段代码反编译后的字节码为：

```java

	public class InsertData {
		private Object object = new Object();
		 
		public void insert(Thread thread){
		    synchronized (object) {
		     
		    }
		}
		 
		public synchronized void insert1(Thread thread){
		     
		}
		 
		public void insert2(Thread thread){
		     
		}
	}
```

从反编译获得的字节码可以看出，synchronized代码块实际上多了monitorenter和monitorexit两条指令。monitorenter指令执行时会让对象的锁计数加1，而monitorexit指令执行时会让对象的锁计数减1，其实这个与操作系统里面的PV操作很像，操作系统里面的PV操作就是用来控制多个线程对临界资源的访问。对于synchronized方法，执行中的线程识别该方法的 method_info 结构是否有 ACC_SYNCHRONIZED 标记设置，然后它自动获取对象的锁，调用方法，最后释放锁。如果有异常发生，线程自动释放锁。


有一点要注意：对于 synchronized 方法或者 synchronized 代码块，当出现异常时，JVM 会自动释放当前线程占用的锁，
因此不会由于异常导致出现死锁现象。

#Lock

synchronized 是 java 中的一个关键字，也就是说是 Java 语言内置的特性。那么为什么会出现 Lock 呢？

一个代码块被 synchronized 修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，
而这里获取锁的线程释放锁只会有两种情况：

* 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
* 线程执行发生异常，此时 JVM 会让线程自动释放锁。

那么如果这个获取锁的线程由于要等待 IO 或者其他原因（比如调用 sleep 方法）被阻塞了，但是又没有释放锁，其他线程便只能干巴巴地等待，
试想一下，这多么影响程序执行效率。

因此就需要有一种机制可以不让等待的线程一直无期限地等待下去（比如只等待一定的时间或者能够响应中断），通过 Lock 就可以办到。

再举个例子：当有多个线程读写文件时，读操作和写操作会发生冲突现象，写操作和写操作会发生冲突现象，但是读操作和读操作不会发生冲突现象。

但是采用 synchronized 关键字来实现同步的话，就会导致一个问题：

如果多个线程都只是进行读操作，所以当一个线程在进行读操作时，其他线程只能等待无法进行读操作。

因此就需要一种机制来使得多个线程都只是进行读操作时，线程之间不会发生冲突，通过 Lock 就可以办到。

另外，通过 Lock 可以知道线程有没有成功获取到锁。这个是 synchronized 无法办到的。

总结一下，也就是说 Lock 提供了比 synchronized 更多的功能。但是要注意以下几点：

* Lock 不是 Java 语言内置的，synchronized 是 Java 语言的关键字，因此是内置特性。Lock 是一个类，通过这个类可以实现同步访问；
* Lock 和 synchronized 有一点非常大的不同，采用 synchronized 不需要用户去手动释放锁，当 synchronized 方法或者 synchronized 代码块执行完之后，系统会自动让线程释放对锁的占用；而 Lock 则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。

##java.util.concurrent.locks包下常用的类

下面我们就来探讨一下java.util.concurrent.locks包中常用的类和接口。

###Lock

首先要说明的就是Lock，通过查看Lock的源码可知，Lock是一个接口：


```java

	public interface Lock {
		void lock();
		void lockInterruptibly() throws InterruptedException;
		boolean tryLock();
		boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
		void unlock();
		Condition newCondition();
	}
```

下面来逐个讲述 Lock 接口中每个方法的使用，lock()、tryLock()、tryLock(long time, TimeUnit unit) 和 
lockInterruptibly() 是用来获取锁的。unLock() 方法是用来释放锁的。newCondition() 这个方法暂且不在此讲述，
会在后面的线程协作一文中讲述。

在 Lock 中声明了四个方法来获取锁，那么这四个方法有何区别呢？

首先 lock() 方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

由于在前面讲到如果采用 Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，
使用 Lock 必须在 try{}catch{} 块中进行，并且将释放锁的操作放在 finally 块中进行，以保证锁一定被
释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java

	Lock lock = ...;
	lock.lock();
	try{
		//处理任务
	}catch(Exception ex){

	}finally{
		lock.unlock();   //释放锁
	}
```

tryLock() 方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回 true，如果获取失败
（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。

tryLock(long time, TimeUnit unit) 方法和 tryLock() 方法是类似的，只不过区别在于这个方法在拿不到锁时会等待
一定的时间，在时间期限之内如果还拿不到锁，就返回 false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

所以，一般情况下通过tryLock来获取锁时是这样使用的：

```java

	Lock lock = ...;
	if(lock.tryLock()) {
		 try{
		     //处理任务
		 }catch(Exception ex){
		 }finally{
		     lock.unlock();   //释放锁
		 } 
	}else {
		//如果不能获取锁，则直接做其他事情
	}

```

lockInterruptibly() 方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应
中断，即中断线程的等待状态。也就使说，当两个线程同时通过 lock.lockInterruptibly() 想获取某个锁时，假若
此时线程 A 获取到了锁，而线程 B 只有在等待，那么对线程B调用 threadB.interrupt() 方法能够中断线程 B 的等待过程。

由于 lockInterruptibly() 的声明中抛出了异常，所以 lock.lockInterruptibly() 必须放在 try 块中或者在调用
lockInterruptibly() 的方法外声明抛出 InterruptedException。

注意，当一个线程获取了锁之后，是不会被 interrupt() 方法中断的。因为本身在前面的文章中讲过单独调用 interrupt()
方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。

因此当通过 lockInterruptibly() 方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。

而用 synchronized 修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

##ReentrantLock

ReentrantLock，意思是“可重入锁”，关于可重入锁的概念在下一节讲述。ReentrantLock 是唯一实现了 Lock 接口的类，
并且 ReentrantLock 提供了更多的方法。下面通过一些实例看具体看一下如何使用 ReentrantLock。

例: lock 的使用

```java
	public class Test {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();
		public static void main(String[] args)  {
		    final Test test = new Test();

		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();

		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();
		}

		public void insert(Thread thread) {
		    Lock lock = new ReentrantLock();    //注意这个地方
		    lock.lock();
		    try {
		        System.out.println(thread.getName()+"得到了锁");
		        for(int i=0;i<5;i++) {
		            arrayList.add(i);
		        }
		    } catch (Exception e) {
		        // TODO: handle exception
		    }finally {
		        System.out.println(thread.getName()+"释放了锁");
		        lock.unlock();
		    }
		}
```

各位朋友先想一下这段代码的输出结果是什么？

	Thread-0得到了锁
	Thread-1得到了锁
	Thread-0释放了锁
	Thread-1释放了锁

也许有朋友会问，怎么会输出这个结果？第二个线程怎么会在第一个线程释放锁之前得到了锁？原因在于，在 insert 方法中的
lock 变量是局部变量，每个线程执行该方法时都会保存一个副本，那么理所当然每个线程执行到 lock.lock() 处获取的是不同
的锁，所以就不会发生冲突。

知道了原因改起来就比较容易了，只需要将lock声明为类的属性即可。

```java

	public class Test {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();
		private Lock lock = new ReentrantLock();    //注意这个地方
		public static void main(String[] args)  {
		    final Test test = new Test();
		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();

		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();
		}

		public void insert(Thread thread) {
		    lock.lock();
		    try {
		        System.out.println(thread.getName()+"得到了锁");
		        for(int i=0;i<5;i++) {
		            arrayList.add(i);
		        }
		    } catch (Exception e) {
		        // TODO: handle exception
		    }finally {
		        System.out.println(thread.getName()+"释放了锁");
		        lock.unlock();
		    }
		}
	}
```

这样就是正确地使用Lock的方法了。

例: tryLock 使用

``` java

	public class Test {
		private ArrayList<Integer> arrayList = new ArrayList<Integer>();
		private Lock lock = new ReentrantLock();    //注意这个地方
		public static void main(String[] args)  {
		    final Test test = new Test();

		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();

		    new Thread(){
		        public void run() {
		            test.insert(Thread.currentThread());
		        };
		    }.start();
		}

		public void insert(Thread thread) {
		    if(lock.tryLock()) {
		        try {
		            System.out.println(thread.getName()+"得到了锁");
		            for(int i=0;i<5;i++) {
		                arrayList.add(i);
		            }
		        } catch (Exception e) {
		            // TODO: handle exception
		        }finally {
		            System.out.println(thread.getName()+"释放了锁");
		            lock.unlock();
		        }
		    } else {
		        System.out.println(thread.getName()+"获取锁失败");
		    }
		}
	}

```

输出结果：

	Thread-0得到了锁
	Thread-1获取锁失败
	Thread-0释放了锁

例: lockInterruptibly()响应中断的使用方法

```java

	public class Test {
		private Lock lock = new ReentrantLock();   
		public static void main(String[] args)  {
		    Test test = new Test();
		    MyThread thread1 = new MyThread(test);
		    MyThread thread2 = new MyThread(test);
		    thread1.start();
		    thread2.start();

		    try {
		        Thread.sleep(2000);
		    } catch (InterruptedException e) {
		        e.printStackTrace();
		    }
		    thread2.interrupt();
		}

		public void insert(Thread thread) throws InterruptedException{
		    lock.lockInterruptibly();   //注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
		    try {
		        System.out.println(thread.getName()+"得到了锁");
		        long startTime = System.currentTimeMillis();
		        for(    ;     ;) {
		            if(System.currentTimeMillis() - startTime >= Integer.MAX_VALUE)
		                break;
		            //插入数据
		        }
		    }
		    finally {
		        System.out.println(Thread.currentThread().getName()+"执行finally");
		        lock.unlock();
		        System.out.println(thread.getName()+"释放了锁");
		    }
		}
	}

	class MyThread extends Thread {
		private Test test = null;
		public MyThread(Test test) {
		    this.test = test;
		}
		@Override
		public void run() {
		    try {
		        test.insert(Thread.currentThread());
		    } catch (InterruptedException e) {
		        System.out.println(Thread.currentThread().getName()+"被中断");
		    }
		}
	}
```

运行之后，发现thread2能够被正确中断。

##ReadWriteLock

ReadWriteLock也是一个接口，在它里面只定义了两个方法：

```java

	public interface ReadWriteLock {
		/**
		 * Returns the lock used for reading.
		 *
		 * @return the lock used for reading.
		 */
		Lock readLock();
		/**
		 * Returns the lock used for writing.
		 *
		 * @return the lock used for writing.
		 */
		Lock writeLock();
	}
```

一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成 2 个锁来分配给线程，从而使得多个线程可以同时进行读操作。
下面的 ReentrantReadWriteLock 实现了 ReadWriteLock 接口。

##ReentrantReadWriteLock

ReentrantReadWriteLock 里面提供了很多丰富的方法，不过最主要的有两个方法：readLock() 和 writeLock() 用来获取读锁和写锁。

下面通过几个例子来看一下 ReentrantReadWriteLock 具体用法。

假如有多个线程要同时进行读操作的话，先看一下synchronized达到的效果：

```
	public class Test {
		private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
		public static void main(String[] args)  {
		    final Test test = new Test();

		    new Thread(){
		        public void run() {
		            test.get(Thread.currentThread());
		        };
		    }.start();

		    new Thread(){
		        public void run() {
		            test.get(Thread.currentThread());
		        };
		    }.start();
		}

		public synchronized void get(Thread thread) {
		    long start = System.currentTimeMillis();
			int count = 0;
		    while(count <= 5) {
		        System.out.println(thread.getName()+"正在进行读操作");
                count++;
		    }
		    System.out.println(thread.getName()+"读操作完毕");
		}
	}
```

这段程序的输出结果会是，直到thread1执行完读操作之后，才会打印thread2执行读操作的信息。

	Thread-0正在进行读操作
	Thread-0正在进行读操作
	Thread-0正在进行读操作
	Thread-0正在进行读操作
	Thread-0正在进行读操作
	Thread-0读操作完毕
	Thread-1正在进行读操作
	Thread-1正在进行读操作
	Thread-1正在进行读操作
	Thread-1正在进行读操作
	Thread-1正在进行读操作
	Thread-1读操作完毕

而改成用读写锁的话：

```java

	public class Test {
		private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

		public static void main(String[] args)  {
		    final Test test = new Test();

		    new Thread(){
		        public void run() {
		            test.get(Thread.currentThread());
		        };
		    }.start();

		    new Thread(){
		        public void run() {
		            test.get(Thread.currentThread());
		        };
		    }.start();

		}

		public void get(Thread thread) {
		    rwl.readLock().lock();
		    try {
		        long start = System.currentTimeMillis();
		        int count = 0;
		        while(count  <= 5) {
		            System.out.println(thread.getName()+"正在进行读操作");
					count++;
		        }
		        System.out.println(thread.getName()+"读操作完毕");
		    } finally {
		        rwl.readLock().unlock();
		    }
		}
	}
```

说明 thread1 和 thread2 在同时进行读操作。

这样就大大提升了读操作的效率。

不过要注意的是，如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。

如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

关于 ReentrantReadWriteLock 类中的其他方法感兴趣的朋友可以自行查阅 API 文档。

##Lock和synchronized的选择

总结来说，Lock 和 synchronized 有以下几点不同：

* Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内置的语言实现；
* synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而 Lock 在发生异常时，如果没有主动通过 unLock() 去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁；
* Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断；
* 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
* Lock 可以提高多个线程进行读操作的效率。


在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时 Lock 的性能要远远优于
synchronized。所以说，在具体使用时要根据适当情况选择。

##锁的相关概念介绍

在前面介绍了Lock的基本使用，这一节来介绍一下与锁相关的几个概念。

###可重入锁

如果锁具备可重入性，则称作为可重入锁。像 synchronized 和 ReentrantLock 都是可重入锁，可重入性在我看来实际上表明了锁的分配机制：
基于线程的分配，而不是基于方法调用的分配。举个简单的例子，当一个线程执行到某个 synchronized 方法时，比如说 method1，而在 method1
中会调用另外一个 synchronized 方法 method2，此时线程不必重新去申请锁，而是可以直接执行方法method2。

看下面这段代码就明白了：

	class MyClass {
		public synchronized void method1() {
		    method2();
		}

		public synchronized void method2() {
		}
	}

上述代码中的两个方法 method1 和 method2 都用 synchronized 修饰了，假如某一时刻，线程 A 执行到了 method1，此时线程 A
获取了这个对象的锁，而由于 method2 也是 synchronized 方法，假如 synchronized 不具备可重入性，此时线程A需要重新申请锁。
但是这就会造成一个问题，因为线程 A 已经持有了该对象的锁，而又在申请获取该对象的锁，这样就会线程 A 一直等待永远不会获取到的锁。

而由于 synchronized 和 Lock 都具备可重入性，所以不会发生上述现象。

###可中断锁

可中断锁：顾名思义，就是可以相应中断的锁。

在Java中，synchronized 就不是可中断锁，而 Lock 是可中断锁。

如果某一线程 A 正在执行锁中的代码，另一线程 B 正在等待获取该锁，可能由于等待时间过长，线程 B 不想等待了，想先处理其他事情，
我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

在前面演示 lockInterruptibly() 的用法时已经体现了 Lock 的可中断性。

###锁降级

写线程获取写入锁后可以获取读取锁，然后释放写入锁，这样就从写入锁变成了读取锁，从而实现锁降级的特性。

###锁升级

读取锁是不能直接升级为写入锁的。因为获取一个写入锁需要释放所有读取锁，所以如果有两个读取锁视图获取写入锁而都不释放读取锁时就会发
生死锁。


###公平锁

公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该，
这种就是公平锁。

公平锁 利用 AQS 的 CLH 队列，释放当前保持的锁（读锁或者写锁）时，优先为等待时间最长的那个写线程分配写入锁，当前前提是写线程的等待时间
要比所有读线程的等待时间要长。同样一个线程持有写入锁或者有一个写线程已经在等待了，那么试图获取公平锁的（非重入）所有线程（包括读写
线程）都将被阻塞，直到最先的写线程释放锁。如果读线程的等待时间比写线程的等待时间还有长，那么一旦上一个写线程释放锁，这一组读线程将
获取锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

非公平锁（默认） 这个和独占锁的非公平性一样，由于读线程之间没有锁竞争，所以读操作没有公平性和非公平性，写操作时，由于写操作可能立
即获取到锁，所以会推迟一个或多个读操作或者写操作。因此非公平锁的吞吐量要高于公平锁

在Java中，synchronized 就是非公平锁，它无法保证等待的线程获取锁的顺序。

而对于 ReentrantLock 和 ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。

我们可以在创建ReentrantLock对象时，通过以下方式来设置锁的公平性：

	ReentrantLock lock = new ReentrantLock(true);

如果参数为true表示为公平锁，为fasle为非公平锁。默认情况下，如果使用无参构造器，则是非公平锁。

另外在ReentrantLock类中定义了很多方法，比如：

* isFair()        //判断锁是否是公平锁
* isLocked()    //判断锁是否被任何线程获取了
* isHeldByCurrentThread()   //判断锁是否被当前线程获取了
* hasQueuedThreads()   //判断是否有线程在等待该锁

在ReentrantReadWriteLock中也有类似的方法，同样也可以设置为公平锁和非公平锁。不过要记住，ReentrantReadWriteLock并未实现Lock接口，
它实现的是ReadWriteLock接口。

读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。

正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。

可以通过readLock()获取读锁，通过writeLock()获取写锁。

上面已经演示过了读写锁的使用方法，在此不再赘述。

#volatile


##内存模型的相关概念

大家都知道，计算机在执行程序时，每条指令都是在CPU中执行的，而执行指令过程中，势必涉及到数据的读取和写入。由于程序运行过程中
的临时数据是存放在主存（物理内存）当中的，这时就存在一个问题，由于CPU执行速度很快，而从内存读取数据和向内存写入数据的过程跟
CPU执行指令的速度比起来要慢的多，因此如果任何时候对数据的操作都要通过和内存的交互来进行，会大大降低指令执行的速度。因此在CPU
里面就有了高速缓存。

也就是，当程序在运行过程中，会将运算需要的数据从主存复制一份到CPU的高速缓存当中，那么CPU进行计算时就可以直接从它的高速缓存读
取数据和向其中写入数据，当运算结束之后，再将高速缓存中的数据刷新到主存当中。举个简单的例子，比如下面的这段代码：

	i = i + 1;

当线程执行这个语句时，会先从主存当中读取i的值，然后复制一份到高速缓存当中，然后CPU执行指令对i进行加1操作，然后将数据写入高速
缓存，最后将高速缓存中i最新的值刷新到主存当中。

这个代码在单线程中运行是没有任何问题的，但是在多线程中运行就会有问题了。在多核CPU中，每条线程可能运行于不同的CPU中，因此每个
线程运行时有自己的高速缓存（对单核CPU来说，其实也会出现这种问题，只不过是以线程调度的形式来分别执行的）。本文我们以多核CPU为例。

比如同时有2个线程执行这段代码，假如初始时i的值为0，那么我们希望两个线程执行完之后i的值变为2。但是事实会是这样吗？

可能存在下面一种情况：初始时，两个线程分别读取i的值存入各自所在的CPU的高速缓存当中，然后线程1进行加1操作，然后把i的最新值1写入到
内存。此时线程2的高速缓存当中i的值还是0，进行加1操作之后，i的值为1，然后线程2把i的值写入内存。

最终结果i的值是1，而不是2。这就是著名的缓存一致性问题。通常称这种被多个线程访问的变量为共享变量。

也就是说，如果一个变量在多个CPU中都存在缓存（一般在多线程编程时才会出现），那么就可能存在缓存不一致的问题。

为了解决缓存不一致性问题，通常来说有以下2种解决方法：

* 通过在总线加LOCK#锁的方式
* 通过缓存一致性协议

这2种方式都是硬件层面上提供的方式。

在早期的CPU当中，是通过在总线上加LOCK#锁的形式来解决缓存不一致的问题。因为CPU和其他部件进行通信都是通过总线来进行的，
如果对总线加LOCK#锁的话，也就是说阻塞了其他CPU对其他部件访问（如内存），从而使得只能有一个CPU能使用这个变量的内存。
比如上面例子中 如果一个线程在执行 i = i +1，如果在执行这段代码的过程中，在总线上发出了LCOK#锁的信号，那么只有等待这
段代码完全执行完毕之后，其他CPU才能从变量i所在的内存读取变量，然后进行相应的操作。这样就解决了缓存不一致的问题。

但是上面的方式会有一个问题，由于在锁住总线期间，其他CPU无法访问内存，导致效率低下。

所以就出现了缓存一致性协议。最出名的就是Intel 的 MESI 协议，MESI 协议保证了每个缓存中使用的共享变量的副本是一致的。
它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他
CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就
会从内存重新读取。

![多核读取变量](multi-core-cpu.png)

##并发编程中的三个概念

在并发编程中，我们通常会遇到以下三个问题：原子性问题，可见性问题，有序性问题。我们先看具体看一下这三个概念：

###原子性

原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

一个很经典的例子就是银行账户转账问题：

	比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。

试想一下，如果这2个操作不具备原子性，会造成什么样的后果。假如从账户A减去1000元之后，操作突然中止。然后又从B取出了500元，取出500元之后，再执行 往账户B加上1000元 的操作。这样就会导致账户A虽然减去了1000元，但是账户B没有收到这个转过来的1000元。

所以这2个操作必须要具备原子性才能保证不出现一些意外的问题

同样地反映到并发编程中会出现什么结果呢？

举个最简单的例子，大家想一下假如为一个32位的变量赋值过程不具备原子性的话，会发生什么后果？

	i = 9;

假若一个线程执行到这个语句时，我暂且假设为一个32位的变量赋值包括两个过程：为低16位赋值，为高16位赋值。

那么就可能发生一种情况：当将低16位数值写入之后，突然被中断，而此时又有一个线程去读取i的值，那么读取到的就是错误的数据。

###可见性

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

举个简单的例子，看下面这段代码：

	//线程1执行的代码
	int i = 0;
	i = 10;

	//线程2执行的代码
	j = i;

假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到CPU1的高速缓存中，
然后赋值为10，那么在CPU1的高速缓存当中i的值变为10了，却没有立即写入到主存当中。

此时线程2执行 j = i，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10.

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

###有序性

有序性：即程序执行的顺序按照代码的先后顺序执行。举个简单的例子，看下面这段代码：

	int i = 0;
	boolean flag = false;
	i = 1;                //语句1
	flag = true;          //语句2

上面代码定义了一个int型变量，定义了一个boolean类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，
那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗？不一定，为什么呢？这里可能会发生指令重排序（Instruction Reorder）。

下面解释一下什么是指令重排序，一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序
同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

比如上面的代码中，语句1和语句2谁先执行对最终的程序结果并没有影响，那么就有可能在执行过程中，语句2先执行而语句1后执行。

但是要注意，虽然处理器会对指令进行重排序，但是它会保证程序最终结果会和代码顺序执行结果相同，那么它靠什么保证的呢？再看下面一个例子：


	int a = 10;    //语句1
	int r = 2;    //语句2
	a = a + 3;    //语句3
	r = a*a;     //语句4

这段代码有4个语句，那么可能的一个执行顺序是：

	语句2 --> 语句1 --> 语句3 --> 语句4

那么可不可能是这个执行顺序呢： 语句2   语句1    语句4   语句3

不可能，因为处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令Instruction 2必须用到Instruction 1的结果，那么处理器会
保证Instruction 1会在Instruction 2之前执行。

虽然重排序不会影响单个线程内程序执行的结果，但是多线程呢？下面看一个例子：

	//线程1:
	context = loadContext();   //语句1
	inited = true;             //语句2

	//线程2:
	while(!inited ){
	  sleep()
	}
	doSomethingwithconfig(context);

上面代码中，由于语句1和语句2没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程1执行过程中先执行语句2，而此是线程2会以为
初始化工作已经完成，那么就会跳出while循环，去执行doSomethingwithconfig(context)方法，而此时context并没有被初始化，就会导致程序出错。

从上面可以看出，指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。

也就是说，要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

##Java内存模型

在前面谈到了一些关于内存模型以及并发编程中可能会出现的一些问题。下面我们来看一下Java内存模型，研究一下Java内存模型为我们提供了哪些保证
以及在java中提供了哪些方法和机制来让我们在进行多线程编程时能够保证程序执行的正确性。

在Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model，JMM）来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种
平台下都能达到一致的内存访问效果。那么Java内存模型规定了哪些东西呢，它定义了程序中变量的访问规则，往大一点说是定义了程序执行的次序。注意，
为了获得较好的执行性能，Java内存模型并没有限制执行引擎使用处理器的寄存器或者高速缓存来提升指令执行速度，也没有限制编译器对指令进行重排序。
也就是说，在java内存模型中，也会存在缓存一致性问题和指令重排序的问题。

Java内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

举个简单的例子：在java中，执行下面这个语句：

执行线程必须先在自己的工作线程中对变量i所在的缓存行进行赋值操作，然后再写入主存当中。而不是直接将数值10写入主存当中。

那么Java语言 本身对 原子性、可见性以及有序性提供了哪些保证呢？

##原子性

在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。

上面一句话虽然看起来简单，但是理解起来并不是那么容易。看下面一个例子i：

请分析以下哪些操作是原子性操作：

	x = 10;         //语句1
	y = x;         //语句2
	x++;           //语句3
	x = x + 1;     //语句4

咋一看，有些朋友可能会说上面的4个语句中的操作都是原子性操作。其实只有语句1是原子性操作，其他三个语句都不是原子性操作。

语句1是直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中。

语句2实际上包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，
但是合起来就不是原子性操作了。

同样的，x++和 x = x+1包括3个操作：读取x的值，进行加1操作，写入新的值。

所以上面4个语句只有语句1的操作具备原子性。

也就是说，只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

不过这里有一点需要注意：在32位平台下，对64位数据的读取和赋值是需要通过两个操作来完成的，不能保证其原子性。但是好像在最新的JDK中，
JVM已经保证对64位数据的读取和赋值也是原子性操作了。

从上面可以看出，Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。
由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。

###可见性

对于可见性，Java提供了volatile关键字来保证可见性。

当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值.

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的
旧值，因此无法保证可见性。

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会
将对变量的修改刷新到主存当中。因此可以保证可见性。

###有序性

在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

在Java里面，可以通过volatile关键字来保证一定的“有序性”（具体原理在下一节讲述）。另外可以通过synchronized和Lock来保证有序性，很显然，
synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

另外，Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。
如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

下面就来具体介绍下happens-before原则（先行发生原则）：

* 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
* 锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作
* volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
* 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
* 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
* 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
* 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
* 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

这8条原则摘自《深入理解Java虚拟机》。这8条规则中，前4条规则是比较重要的，后4条规则都是显而易见的。

下面我们来解释一下前4条规则：

对于程序次序规则来说，我的理解就是一段程序代码的执行在单个线程中看起来是有序的。注意，虽然这条规则中提到“书写在前面的操作先行
发生于书写在后面的操作”，这个应该是程序看起来执行的顺序是按照代码顺序执行的，因为虚拟机可能会对程序代码进行指令重排序。虽然进行
重排序，但是最终执行的结果是与程序顺序执行的结果一致的，它只会对不存在数据依赖性的指令进行重排序。因此，在单个线程中，程序执行
看起来是有序执行的，这一点要注意理解。事实上，这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的
正确性。

第二条规则也比较容易理解，也就是说无论在单线程中还是多线程中，同一个锁如果出于被锁定的状态，那么必须先对锁进行了释放操作，后面
才能继续进行lock操作。

第三条规则是一条比较重要的规则，也是后文将要重点讲述的内容。直观地解释就是，如果一个线程先去写一个变量，然后一个线程去进行读取，
那么写入操作肯定会先行发生于读操作。

第四条规则实际上就是体现happens-before原则具备传递性。

##深入剖析volatile关键字

在前面讲述了很多东西，其实都是为讲述volatile关键字作铺垫，那么接下来我们就进入主题。

###volatile关键字的两层语义

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

* 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
* 禁止进行指令重排序。

先看一段代码，假如线程1先执行，线程2后执行：

```java

	//线程1
	boolean stop = false;
	while(!stop){
		doSomething();
	}

	//线程2
	stop = true;
```

这段代码是很典型的一段代码，很多人在中断线程时可能都会采用这种标记办法。但是事实上，这段代码会完全运行正确么？
即一定会将线程中断么？不一定，也许在大多数时候，这个代码能够把线程中断，但是也有可能会导致无法中断线程（虽然这个可能性很小，
但是只要一旦发生这种情况就会造成死循环了）。

下面解释一下这段代码为何有可能导致无法中断线程。在前面已经解释过，每个线程在运行过程中都有自己的工作内存，那么线程1在运行的
时候，会将 stop 变量的值拷贝一份放在自己的工作内存当中。

那么当线程2更改了stop变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1由于不知道线程2对stop变量的更改，
因此还会一直循环下去。

但是用volatile修饰之后就变得不一样了：

第一：使用volatile关键字会强制将修改的值立即写入主存；

第二：使用volatile关键字的话，当线程2进行修改时，会导致线程1的工作内存中缓存变量stop的缓存行无效（反映到硬件层的话，就是CPU的L1
或者L2缓存中对应的缓存行无效）；

第三：由于线程1的工作内存中缓存变量stop的缓存行无效，所以线程1再次读取变量stop的值时会去主存读取。

那么在线程2修改stop值时（当然这里包括2个操作，修改线程2工作内存中的值，然后将修改后的值写入内存），会使得线程1的工作内存中缓存变量
stop的缓存行无效，然后线程1读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。

那么线程1读取到的就是最新的正确的值。


###volatile保证原子性吗？

从上面知道volatile关键字保证了操作的可见性，但是volatile能保证对变量的操作是原子性吗？

下面看一个例子：

	public class Test {
		public volatile int inc = 0;

		public void increase() {
		    inc++;
		}

		public static void main(String[] args) {
		    final Test test = new Test();
		    for(int i=0;i<10;i++){
		        new Thread(){
		            public void run() {
		                for(int j=0;j<1000;j++)
		                    test.increase();
		            };
		        }.start();
		    }

		    while(Thread.activeCount() > 1)  //保证前面的线程都执行完
		        Thread.yield();
		    System.out.println(test.inc);
		}
	}

大家想一下这段程序的输出结果是多少？也许有些朋友认为是10000。但是事实上运行它会发现每次运行结果都不一致，都是一个小于10000的数字。

可能有的朋友就会有疑问，不对啊，上面是对变量inc进行自增操作，由于volatile保证了可见性，那么在每个线程中对inc自增完之后，在其他线程
中都能看到修改后的值啊，所以有10个线程分别进行了1000次操作，那么最终inc的值应该是1000*10=10000。

这里面就有一个误区了，volatile关键字能保证可见性没有错，但是上面的程序错在没能保证原子性。可见性只能保证每次读取的是最新的值，但是
volatile没办法保证对变量的操作的原子性。

在前面已经提到过，自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存。那么就是说自增操作的三个子操作可能会
分割开执行，就有可能导致下面这种情况出现：

假如某个时刻变量inc的值为10，

线程1对变量进行自增操作，线程1先读取了变量inc的原始值，然后线程1被阻塞了；

然后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，由于线程1只是对变量inc进行读取操作，而没有对变量进行修改操作，
所以不会导致线程2的工作内存中缓存变量inc的缓存行无效，所以线程2会直接去主存读取inc的值，发现inc的值时10，然后进行加1操作，
并把11写入工作内存，最后写入主存。

然后线程1接着进行加1操作，由于已经读取了inc的值，注意此时在线程1的工作内存中inc的值仍然为10，所以线程1对inc进行加1操作后inc的值为11，
然后将11写入工作内存，最后写入主存。

那么两个线程分别进行了一次自增操作后，inc只增加了1。

解释到这里，可能有朋友会有疑问，不对啊，前面不是保证一个变量在修改volatile变量时，会让缓存行无效吗？然后其他线程去读就会读到新的值，
对，这个没错。这个就是上面的happens-before规则中的volatile变量规则，但是要注意，线程1对变量进行读取操作之后，被阻塞了的话，并没有对
inc值进行修改。然后虽然volatile能保证线程2对变量inc的值读取是从内存中读取的，但是线程1没有进行修改，所以线程2根本就不会看到修改的值。


根源就在这里，自增操作不是原子性操作，而且volatile也无法保证对变量的任何操作都是原子性的。

把上面的代码改成以下任何一种都可以达到效果：


采用synchronized：

```java

	public class Test {
		public  int inc = 0;
		
		public synchronized void increase() {
		    inc++;
		}
		
		public static void main(String[] args) {
		    final Test test = new Test();
		    for(int i=0;i<10;i++){
		        new Thread(){
		            public void run() {
		                for(int j=0;j<1000;j++)
		                    test.increase();
		            };
		        }.start();
		    }
		    
		    while(Thread.activeCount()>1)  //保证前面的线程都执行完
		        Thread.yield();
		    System.out.println(test.inc);
		}
	}
```

采用Lock：

```java

	public class Test {
		public  int inc = 0;
		Lock lock = new ReentrantLock();
		
		public  void increase() {
		    lock.lock();
		    try {
		        inc++;
		    } finally{
		        lock.unlock();
		    }
		}
		
		public static void main(String[] args) {
		    final Test test = new Test();
		    for(int i=0;i<10;i++){
		        new Thread(){
		            public void run() {
		                for(int j=0;j<1000;j++)
		                    test.increase();
		            };
		        }.start();
		    }
		    
		    while(Thread.activeCount()>1)  //保证前面的线程都执行完
		        Thread.yield();
		    System.out.println(test.inc);
		}
	}
```

采用AtomicInteger：

```
public class Test {
    public  AtomicInteger inc = new AtomicInteger();
     
    public  void increase() {
        inc.getAndIncrement();
    }
    
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
        
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

在java 1.5的java.util.concurrent.atomic包下提供了一些原子操作类，即对基本数据类型的 自增（加1操作），自减（减1操作）、
以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。atomic是利用CAS来实现原子性操作的
（Compare And Swap），CAS实际上是利用处理器提供的CMPXCHG指令实现的，而处理器执行CMPXCHG指令是一个原子性操作。

###volatile能保证有序性吗？

在前面提到volatile关键字能禁止指令重排序，所以volatile能在一定程度上保证有序性。

volatile关键字禁止指令重排序有两层意思：

* 当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
* 在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行

可能上面说的比较绕，举个简单的例子：

	//x、y为非volatile变量
	//flag为volatile变量
	 
	x = 2;        //语句1
	y = 0;        //语句2
	flag = true;  //语句3
	x = 4;         //语句4
	y = -1;       //语句5

由于flag变量为volatile变量，那么在进行指令重排序的过程的时候，不会将语句3放到语句1、语句2前面，也不会讲语句3放到语句4、语句5后面。
但是要注意语句1和语句2的顺序、语句4和语句5的顺序是不作任何保证的。

并且volatile关键字能保证，执行到语句3时，语句1和语句2必定是执行完毕了的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的。

那么我们回到前面举的一个例子：

	//线程1:
	context = loadContext();   //语句1
	inited = true;             //语句2
	 
	//线程2:
	while(!inited ){
	  sleep()
	}
	doSomethingwithconfig(context);


前面举这个例子的时候，提到有可能语句2会在语句1之前执行，那么有可能导致context还没被初始化，而线程2中就使用未初始化的context
去进行操作，导致程序出错。

这里如果用volatile关键字对inited变量进行修饰，就不会出现这种问题了，因为当执行到语句2时，必定能保证context已经初始化完毕。

###volatile的原理和实现机制


前面讲述了源于volatile关键字的一些使用，下面我们来探讨一下volatile到底如何保证可见性和禁止指令重排序的。

下面这段话摘自《深入理解Java虚拟机》：

　	“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”

lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

* 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
* 它会强制将对缓存的修改操作立即写入主存；
* 如果是写操作，它会导致其他CPU中对应的缓存行无效。

###使用volatile关键字的场景

synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。通常来说，使用volatile必须具备以下2个条件：

* 对变量的写操作不依赖于当前值
* 该变量没有包含在具有其他变量的不变式中

实际上，这些条件表明，可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。

事实上，我的理解就是上面的2个条件需要保证操作是原子性操作，才能保证使用volatile关键字的程序在并发时能够正确执行。

下面列举几个Java中使用volatile的几个场景。

####状态标记量

	volatile boolean flag = false;
	 
	while(!flag){
		doSomething();
	}
	 
	public void setFlag() {
		flag = true;
	}


	volatile boolean inited = false;
	//线程1:
	context = loadContext();  
	inited = true;            
	 
	//线程2:
	while(!inited ){
	sleep()
	}
	doSomethingwithconfig(context);

####double check

	class Singleton{
		private volatile static Singleton instance = null;
		 
		private Singleton() {
		     
		}
		 
		public static Singleton getInstance() {
		    if(instance==null) {
		        synchronized (Singleton.class) {
		            if(instance==null)
		                instance = new Singleton();
		        }
		    }
		    return instance;
		}
	}




##参考
http://www.cnblogs.com/dolphin0520/p/3923167.html
http://www.cnblogs.com/dolphin0520/p/3920373.html
