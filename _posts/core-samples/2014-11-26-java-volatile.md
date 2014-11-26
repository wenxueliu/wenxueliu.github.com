---
layout: post
category : java
tagline: "java volatile 用法"
tags : [ java ]
---
{% include JB/setup %}

##Java Volatile

It's probably fair to say that on the whole, the volatile keyword in Java is poorly documented, poorly understood, and rarely used. To make matters worse, its formal definition actually changed as of Java 5. On this and the following pages, we will cut through this mess and look at what the Java volatile keyword does and when it is used. We will also compare it to other mechanisms available in Java which perform similar functions but under subtly different other circumstances.


##What is the Java volatile keyword?

Essentially, volatile is used to indicate that a variable's value will be modified by different threads.

Declaring a volatile Java variable means: 

* The value of this variable will never be cached thread-locally: all reads and writes will go straight to "main memory";
* Access to the variable acts as though it is enclosed in a synchronized block, synchronized on itself. 

We say "acts as though" in the second point, because to the programmer at least (and probably in most JVM implementations) there is no actual lock object involved. Here is how synchronized and volatile compare: 

Difference between synchronized and volatile

		Characteristic									Synchronized												Volatile

		Type of variable									Object												Object or primitive
		Null allowed?										No															Yes
		Can block?											Yes															No
		All cached variables synchronized on access?		Yes													From Java 5 onwards
		When synchronization happens					When you explicitly enter/exit 
															a synchronized block								Whenever a volatile 
																												variable is accessed.
																												
		Can be used to combined several operations																Pre-Java 5, no.
		into an atomic operation?							Yes													 Atomic get-set 
																												of volatiles possible 
		 																										in Java 5.
		 																																																			
In other words, the main differences between synchronized and volatile are:

*  a primitive variable may be declared volatile (whereas you can't synchronize on a primitive with synchronized);
* an access to a volatile variable never has the potential to block: we're only ever doing a simple read or write, so unlike a synchronized block we will never hold on to any lock;
* because accessing a volatile variable never holds a lock, it is not suitable for cases where we want to read-update-write as an atomic operation (unless we're prepared to "miss an update");
* a volatile variable that is an object reference may be null (because you're effectively synchronizing on the reference, not the actual object). 

Attempting to synchronize on a null object will throw a NullPointerException.

##Volatile variables in Java 5

We mentioned that in Java 5, the meaning of volatile has been tightened up. We'll come back to this issue in a moment. First, we'll look at a typical example of using volatile. Later, we'll look at topics such as: 

* the tighter definition of volatile in Java 5;
* atomic updates to volatile variables, useful in concurrent programming and possible as of Java 5;
* a summary of when to use volatile, along with some typical Java bugs involving the volatile keyword. 


##definition of volatile in Java 5;

We've so far seen that a volatile variable is essentially one whose value is always held in main memory so that it can be accessed by different threads. Java 5 adds a subtle change to this definition and adds some additional facilities for manipulating volatile variables.
Java 5 definition of volatile

As of Java 5, accessing a volatile variable creates a memory barrier: it effectively synchronizes all cached copies of variables with main memory, just as entering or exiting a synchronized block that synchronizes on a given object. Generally, this doesn't have a big impact on the programmer, although it does occasionally make volatile a good option for safe object publication. The infamous [double-checked locking] antipattern actually becomes valid in Java 5 if the reference is declared volatile. (See the page on [how to fix double-checked locking] for an example.) And volatiles memory synchronization behaviour has occasionally allowed more efficient design in some of the library classes via a technique dubbed [synchronization 'piggybacking'].
New Java 5 functionality for volatile variables

Perhaps more significant for most programmers is that as of Java 5, the VM exposes some additional functionality for accessing volatile variables:

* truly atomic get-and-set operations are permitted;
* an efficient means of accessing the nth element of a volatile array (and performing atomic get-and-set on the element) is provided. 

These facilities are tucked away inside the sun.misc.Unsafe class1, but exposed to the programmer in the form of various new concurrency classes:

* the atomic wrapper classes **AtomicInteger** and **AtomicLong** provide atomic access to a single underlying volatile int or long;
* the **AtomicIntegerArray**, **AtomicLongArray** and **AtomicReferenceArray** classes provide efficient atomic access to the nth element of an underlying volatile array;
* the **AtomicReferenceFieldUpdater** class allows you to access an arbitrary volatile variable atomically. 


##Dangers of the volatile keyword

Volatile arrays don't give volatile access to elements

If you declare arr[] is volatile, that means that the reference to the array is volatile; but individual field accesses (e.g. arr[5]) are not thread-safe. 

###Unary operations (++, --) aren't atomic

A danger of "raw" volatile variables is that they can misleadingly make a non-atomic operation look atomic. For example: 

	volatile int i;
	...
	i += 5;
	

Although it may look it, the operation i += 5 is not atomic. It is more the equivalent to: 

	// Note that we can't literally synchronize on an int primitive!
	int temp;
	synchronized (i) {
	  temp = i;
	}
	temp += 5;
	synchronized (i) {
	  i = temp;
	}
	
So another thread could sneak in the middle of the overall operation of i += 5.


###How to fix this bug

To fix this bug and provide a "true" atomic get-set operation.

Prior to Java 5, pretty much the only solution is to introduce an explicit lock object and synchronize on it around all accesses to the variable. 

From Java 5 onwards, CAS and related instructions are effectively exposed at the level of the Java programmer in the form of a series of "atomic" classes in the java.util.concurrent package: 


* Classes providing atomic access to main primitive types or to an object reference: **AtomicBoolean**, **AtomicInteger** and **AtomicLong**, **AtomicReference**;
* Classes providing atomic access to fields in arrays of the correspoding types: **AtomicIntegerArray**, **AtomicLongArray**, **AtomicReferenceArray**;
    
* Classes to atomically couple a boolean or integer with a reference field: **AtomicMarkableReference** and **AtomicStampedReference**;
    Atomic field updaters: classes to wrap up atomic access, via reflection, to volatile fields of another class (AtomicIntegerFieldUpdater, AtomicLongFieldUpdater and AtomicReferenceFieldUpdater);

* Concurrent collection classes, including significantly a highly concurrent hash map implementation,**ConcurrentHashMap**. 


###Avoiding messy syntax

The disadvantage of AtomicInteger, AtomicReference etc is that every access even if just a single read or write must go via one of the method calls. (On the other hand, this forces you to think about what's going on and may avoid bugs of the volatileVariable++ kind.) 



##When to use volatile

####Not Necessary 

* fields that are immutable (declared final); 
* variables that are accessed by only one thread (though of course you have to make a correct decision that they are only accessed by one thread!); 
* complex operations where you need to prevent access to a variable for the duration of the operation: in such cases, you should use object [synchronization] or one of Java 5's explicit [lock classes] added. 


####Necessary

* you write a variable, such as a flag, in one thread; 
* you check that variable in another thread; 
* crucially, the value to write doesn't depend on the current value... 
* ...or, you don't care about "missing an update". 


The last point above outlaws cases such as x++ or x += 7, because this actually involves reading the current value, incrementing it, then setting the new value. A common bug with volatile is to assume that x++ is atomic. For cases where you need this functionality, consider AtomicInteger or related classes. Now, under some circumstances, you could make the decision to perform concurrent x++ operations anyway and "take the risk" that occasionally you'll miss an update. 

#### A field of a class that you'll instantiate a large number of times

In general, where you need atomic access to a "one-off" variable or one created a fairly small number of times, then the Java 5 atomic classes (primarily AtomicInteger) are your answer.

But if you're creating a large number of instances of an object containing a field that needs atomic access, using a volatile field and accessing it via an **AtomicReferenceFieldUpdater** (or AtomicIntegerFieldUpdater or AtomicLongFieldUpdater for primitive fields) will generally be more efficient. 



