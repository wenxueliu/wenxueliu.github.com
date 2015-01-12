From the Netty web site:

	"Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server."

Netty is a Java library and API primarilly aimed at writing highly concurrent networked and networking applications and services. One aspect of Netty you may find different from standard Java APIs is that it is predominantly an asynchronous API. That term implies differnet things to different people and may overlap with the terms non-blocking and event-driven. Regardless, if you have never used an asynchronous API before, it takes a little bit of a mind shift to implement Netty if you are accustomed to writing linear software. Here's how I would boil it down.

You build a Netty stack and start it. Issuing requests is easy and much the same as it is in any Java API. The mind shift comes in processing responses because there are none. Almost every single method invocation of substance is asynchronous, which means that there is no return value and invocation is usually instantaneous. The results (if there are any) will be delivered back in another thread. This is the fundamental difference between a standard API and an asynchronous one. Consider a client API that supplies a method to acquire the number of widgets from the server.

###A Standard API

	public int getWidgetCount();
	
When a thread calls getWidgetCount(), some period of time will elapse and an int will be returned.

###Asynchronous API

	public WidgetCountListener myListener = new WidgetCountListener() {
		 public void onWidgetCount(int widgetCount) {
		      ...... do your thing with the widget count
	 
		 }
	};

In my fabricated asynchronous version of the same API, the getWidgetCount call does not return anything and could conceivably execute instantly. However, it accepts a response handler as an argument and that listener will be called back when the widget count has been acquired and the listener can then do whatever is usefully defined with that result.

It might seem the additional layer[s] of complexity are unwarranted, but this is a crucial aspect of high performance applications and services written with Netty and the like. Your client need not waste resources having threads stuck in waiting or timed waiting mode, waiting for the server to respond. They can be notified when the result is available. For one thread, issuing one call, this may seem overkill, but consider hundreds of threads executing this method millions of times. Moreover, a fundamental benefit of NIO is that Selectors can be used to delegate the notification of events we are interested in to the underlying operating system and at the root of it all, to the hardware we're running on. Being informed through callbacks from the OS that 16 bytes have been successfully written out across the network to a server, or that 14 bytes were just read in from the same is obviously a very low level and granular way to work, but through a chain of abstractions, a developer can implement the Java NIO and Netty APIs to enable handling things at much less granular and abstracted level.

In this introduction, I want to start with some basic concepts and name some of the core building blocks and then slide into some actual code examples.

###How Does All That Work ?

 The basic currency of Netty is a ChannelBuffer. From the Netty JavaDocs:

    A random and sequential accessible sequence of zero or more bytes (octets). This interface provides an abstract view for one or more primitive byte arrays (byte[]) and NIO buffers.
    
 That's how data is passed around in Netty. If your application only deals with byte arrays and byte buffers, you're in luck ! However, if you need to take a higher order data representation, such as a Java object,  and send it to a remote server, and then receive another object back again, those byte collections need to be converted. Or, if you need to issue a request to an HTTP server, your request might start life as a simple string in the form of a URL, but then it needs to be wrapped into a request that the HTTP server will understand and then decomposed into some raw bytes and sent off across the network.  The HTTP server needs to accept the bytes that are sent and compose them back into an HTTP request that it can interpret and fulfil. Once fulfilled, the reverse must occur where the response (perhaps the response is a JPEG or a JavaScript file) must be wrapped in an HTTP response, converted into bytes and sent back to the calling client.
 
The basic abstraction of the network through which these bytes are transported is the Netty Channel. Once more to the Netty JavaDocs:

	A nexus to a network socket or a component which is capable of I/O operations such as read, write, connect, and bind.
	
Nexus is an apt name to describe a channel:  (from dictionary.com)

* a means of connection; tie; link
* a connected series or group
* the core or center, as of a matter or situation

A channel is the externally exposed API through which you interact with Netty. A more grounded way of saying it, a Channel is an abstracted representation of a socket. But.... it's not necessarily a socket, it could be a file, or something even more abstract, so Nexus is a good way to describe it. Suffice it so say, a Channel provides the interface to connect and write to the destination represented by the Channel. No read ? you might ask ? Nope. Remember, it's like the asynchronous getWidgetCount method mentioned above. There's no return value.

All methods on a Channel fall into two categories:


* A simple attribute method that (synchronously) provides information about the channel itself.
* I/O operations like bind, disconnect, or write.

All methods in category #2 are asynchronous* and they all return a ChannelFuture which is a deferred result, meaning that it is a container for a result that is not known yet, but through which the result will be delivered when it is available. It's a bit like our contrived WidgetCountListener, except that you register your listener with the ChannelFuture instead of passing the listener with the original invocation. (Makes for a cleaner API, no ?) The interface that describes the listener you can implement is the ChannelFutureListener which has one method, called when the operation is complete: public void operationComplete(ChannelFuture future). When this method is called, the operation is complete, but it may not have succeeded, so the passed future can be interrogated to determine the outcome of the operation.

Not completely true. Netty is predominantly used for NIO, but it also has channel implementations for OIO which refers to the Old IO which is completely synchronous. OIO has some benefits and the Netty implementation is consistent with NIO implementation so components can be interchangeable and reusable.

##Creating a Connected Channel

 Channels are not created directly. They are created by a ChannelFactory. There are two variations for ChannelFactories, one for client channels and one for server channels. For each of those two variations, there are several implementations to account for the I/O type as well as the transport protocol:


* TCP NIO Channels: NioClientSocketChannelFactory and NioServerSocketChannelFactory
* UDP NIO Channels: NioDatagramChannelFactory
* TCP OIO Channels: OioClientSocketChannelFactory and OioServerSocketChannelFactory
* UDP OIO Channels: OioDatagramChannelFactory 

UDP based channel factories are the same for clients and servers as they are considered connectionless. There are two additional types:

* HTTP Client:  HttpTunnelingClientSocketChannelFactory: This is a convenience factory that generates channels equipped to tunnel to a Netty server through a special Netty Servlet.
* Local Channels: DefaultLocalClientChannelFactory and DefaultLocalServerChannelFactory are the client and server components of In-VM channels which behave just like the real networked channels but handle invocations within the same JVM. This allows for requests to be dispatched or handled through the same abstracted Channel interface but in some cases, a request can be handled or served locally.
 
I will be focusing on the TCP NIO channels in this blog, but be aware there are slight differences in creating different types of channel factories.

The constructors for the TCP NIO ChannelFactories have the same signatures and while there are a few overloads, the basics are that the factory needs two thread pools, or Executors for:


* The Boss Threads: Threads provided by this pool are boss threads. Boss threads create and connect/bind sockets and then pass them off to the worker threads. In the server, there is one boss thread allocated per listening socket. In the client, there is only one boss thread*
    
* The Worker Threads: Worker threads perform all the asynchronous I/O. They are not general purpose threads and developers should take precautions not to assign unrelated tasks to threads in this pool which may cause the threads to block, be rendered unable to perform their real work which in turn may cause deadlocks and an untold number of performance issues.

So if there is only one boss thread in the client, why does the NioClientSocketChannelFactory require an Executor ?


    The boss thread can be released when there is no work to do and is created lazily, but it may be more efficient to pool a small number of threads than create a new one when required and destroying it when idle. 
    
    It is possible that one might want to create several different channel factories and rather than giving each one their own boss pool, they can all share one.


NIO channel factories are the only type that use a boss pool since they are the only ones that can asynchronously connect to sockets or bind to server sockets. The others either have virtual connections (Local), only synchronously connect (OIO) or are connectionless (UDP). The HttpTunelingClientSocketChannelFactory is simply a wrapper for another client socket channel factory, so it may or may not be using a boss thread but it not configured with one.

Keep this in mind about ChannelFactories:  in the course of conducting business with Netty, the factories will allocate resources, including the thread pools. Once you're done with a ChannelFactory, be sure to call releaseExternalResources() on the factory. This will ensure that all its resources are released. 

In a nutshell, to send something to a listening server:

1 Create a channel
2 Connect the channel to the remote listening socket
3 Call write(Object message) on the channel.

Passing Object to the channel seems fairly flexible, no ?  So what happens if I do this ?

	channel.write(new Date());

Netty will issue this exception:

	java.lang.IllegalArgumentException: unsupported message type: class java.util.Date


So what is supported ? ChannelBuffers. That's it. However, Channels have a construct called Pipelines (or more specifically, ChannelPipelines). A pipeline is a stack of interceptors that can manipulate or transform the values that are passed to them. Then, when they're done, the pass the value on to the next interceptor. These interceptors are referred to as ChannelHandlers.  The pipeline maintains strict ordering of the ChannelHandler instances that it contains, and typically, the first channel handler will accept a raw ChannelBuffer and the last chanel handler (referred to as the Sink) will discard whatever payload has been passed to it. Somewhere in the pipeline, you want to implement a handler that does something useful. The ChannelHandler itself is only a marker interface with no methods, so handlers have a lot of flexibility, but for a handler to do anything useful, it must be able to respond and/or forward ChannelEvents.  (There's a lot of terminology here.....)

What the heck is a ChannelEvent ? For the purposes of these two paragraphs, consider a ChannelEvent to be a package containing a ChannelBuffer, which as we already know, is the singular currency of Channels. Skip the next paragraphs if you're satisfied with this simplification.

##Channel Events -For Real ?

Ok, ChannelEvents are not just packets containing ChannelBuffers.Since almost everything in Netty occurs asynchronously, the driving code patterns consist of bits of code that generate events (like Connect, Disconnect and Write) and bits of code that handle those events once they've been executed by one of the ChannelFactory's thread pools. All these events in the Netty API are instances of the interface ChannelEvent. The javadoc for ChannelEvent has a comprehensive list of the different types of events and explanations of each. For a good example of what an event handler looks like, see the javadoc for the class SimpleChannelHandler. This class implements handler methods for almost every type of event. There is even an event called IdleStateEvent which can be fired when ....... nothing. It fires when a channel has been idle, which can be useful.

In order to implement anything other than a simple ChannelBuffer sender or receiver, we need to add ChannelHandlers to the pipeline. Most of the time, ChannelHandlers in use are probably encoders and decoders, meaning:

###Encoder

Converts a non ChannelBuffer object into a ChannelBuffer, suitable for transmission to somewhere else. It might not encode directly to a ChannelBuffer, rather, it might do a partial conversion, implicitly relying on another handler[s] in the pipeline to complete the conversion. One way or the other, if you're not sending straight ChannelBuffers, one or more of the handlers in concert in the pipeline must convert the payload to a ChannelBuffer before it goes out the door. An example of a encoder is ObjectEncoder which converts regular [serializable] java objects into ChannelBuffers containing the byte representation of the written object.

###Decoder 

The reverse of an encoder where a ChannelBuffer's contents are converted into something more useful. The counterpart of the ObjectEncoder mentioned above, is the ObjectDecoder and it does exactly this.

The Netty SDK supplies various codecs (contraction of Coder/Decoder[s]) such as Google ProtoBufs, Compression, HTTP Request/Response and Base64.

Not all ChannelHandlers are Encoders/Decoders (although the majority in the core API are). A ChannelHandler can be put to work doing all sorts of useful things. For example:

* ExecutionHandler: Forwards channel events to another thread pool. Intended to push events out of the Worker (I/O processing) thread pool in order to make sure it stays lively.

* BlockingReadHandler: Allows your code to treat a Channel as though it was not asynchronous for the purposes of reading incoming data.

* LoggingHandler:Simply logs what events are passing through the handler. A highly useful debugging tool......


So how do I get these ChannelHandlers to help me with my java.util.Date problem ? The answer is that you need to add an ObjectEncoder* to the channel's pipeline so it can translate the Date to a ChannelBuffer. Here's a visual, and I'll get to the code in a minute:

[流程图]()

 ObjectEncoder is actually an optimized Netty special. So long as you have a Netty ObjectDecoder on the other end, you're good to go. Otherwise, you need to use a CompatibleObjectEncoder which uses standard Java serialization.
 
 Alright, checkpoint:  We need a Channel, which we get from a ChannelFactory, but we also need to define a ChannelPipeline. There's a few ways you can do this, but Netty provides a construct called a Bootstrap that wraps all these things together quite nicely, so here's how all this stuff fits together in order to create a Netty client that can send a Date. This example will implement the NioClientSocketChannelFactory which will create an instance of an NioClientSocketChannel. It bears mentioning, though, that for the most part, the Netty public API simply returns a SocketChannel, an abstract parent class, and you can just consider this to be a plain Channel as the implementations themselves exhibit identical behavior and exposed functionality. This code sample is a little more simplified that what you might see in other examples, but I doeth it for clarity.
 
	Executor bossPool = Executors.newCachedThreadPool();
	Executor workerPool = Executors.newCachedThreadPool();
	ChannelFactory channelFactory = new NioClientSocketChannelFactory(bossPool, workerPool);
	ChannelPipelineFactory pipelineFactory = new ChannelPipelineFactory() {
	  public ChannelPipeline getPipeline() throws Exception {
		return Channels.pipeline(
		  new ObjectEncoder()
		}
	};
	Bootstrap boostrap = new ClientBootstrap(channelFactory);
	boostrap.setPipelineFactory(pipelineFactory);
	// Phew. Ok. We built all that. Now what ?
	InetSocketAddress addressToConnectTo = new InetSocketAddress(remoteHost, remotePort);
	ChannelFuture cf = bootstrap.connect(addressToConnectTo);


That's how you get a Channel.....  wait up !  That's not a Channel. It's a ChannelFuture. What's that ? Remember that [almost] everything is asynchronous in Netty, so when you request a connection, the actual process of connecting is asynchronous. For that reason, the bootstrap returns a ChannelFuture which is a "handle" to a forthcoming completion event. The ChannelFuture provides the state of the requested operation, and assuming the connect completes successfully, will also provide the Channel. There are (as always....) several options for exactly how the calling thread can wait for the connection to complete, and the following outline of these options applies equally to all asynchronous invocations against channels (disconnects, writes etc.) since all these operations return ChannelFutures.


##Waiting on Godot (where Godot is a Channel Invocation)

Regardless of the strategy implemented to wait for a Channel operation to complete, completion itself does not necessarilly indicate success, simply that the operation concluded. The result of completion could be one of the following:

* Success !: The operation completed successfully.
* Failed: The operation failed and the reason (a Throwable) for the failure is available from ChannelFuture.getCause().
* Timed Out: Also a failure as far as we're concerned.
* Cancelled: A channel invocation can be cancelled by calling ChannelFuture.cancel().

Having said that, here's how you can wait for completion:

###Await

Await is a fancy way of saying wait, (perhaps used to avoid confusing with Object.wait()  ?) The ChannelFuture provides a number of methods that will cause the connecting thread to wait until the connect operation is complete. Note that this does not mean the thread will wait until it connects, rather, it waits until the final conclusion of the call which might be a successful connect, a failed connect, a connect timeout or the connect request could be cancelled. If there are too many options here to keep you attention, skip to the next paragraph, but the asynchronous nature of these operations requires a few different ways of awaiting, but basically stated:

* Interruptibility: 

The waiting thread may be interrupted. This is a fact of life with Threads and the core Java thread API always throws a checked exception (InterruptedException) whenever a thread is directed to wait (sleep, join etc.). However, the ChannelFuture gives you the option of supressing calls to interrupt() which basically means you don't need to put a try/catch block around the call which is awaitUninterruptibly. Otherwise, the interruptible option can be used which is await.

* Timeout

For various reasons, operations may be indefinitely blocked so it may be wise to apply a timeout to channel operations which limits the time that a thread will wait before getting a timeout exception.

###Don't Wait

The connecting thread can optionally fire-and-forget by registering a listener on the ChannelFuture which will be fired when the operation completes.This listener is an implementation of a ChannelFutureListener and it receives a callback when the operation completes, passing a reference of the ChannelFuture in question.

In summary, this code outlines examples of the completion waiting options, using a connect operation as the asynchronous event we're waiting for, but applicable for most operations:

	// Waiting on a connect. (Pick one)
	ChannelFuture cf = bootstrap.connect(addressToConnectTo);
	// A. wait interruptibly
	cf.await();
	// B. wait interruptibly with a timeout of 2000 ms.
	cf.await(2000, TimeUnit.MILLISECONDS);
	// C. wait uninterruptibly
	cf.awaitUninterruptibly();
	// D. wait uninterruptibly with a timeout of 2000 ms.
	cf.awaitUninterruptibly(2000, TimeUnit.MILLISECONDS);
	// E. add a ChannelFutureListener that writes the Date when the connect is complete
	cf.addListener(new ChannelFutureListener(){
	 public void operationComplete(ChannelFuture future) throws Exception {
	  // chek to see if we succeeded
	  if(future.isSuccess()) {
	   Channel channel = future.getChannel();
	   channel.write(new Date());
	   // remember, the write is asynchronous too !
	  }
	 }
	});
	// if a wait option was selected and the connect did not fail,
	// the Date can now be sent.
	Channel channel = cf.getChannel();
	channel.write(new Date());
	
Typically, in a client side application, the awaitXXX would be appropriate, since your client thread may not have much to do while it waits anyways. In a server code stack (perhaps some sort of proxy server), the connect event callback might be a better choice since there is more likely to be a backlog of work to be done and threads should not be sitting around waiting on I/O events to complete.

The Netty documentation makes this fairly clear, but it is worth emphasising here, you want your Worker threads working, not waiting, so avoid calling any awaits in a worker thread.

One last item on connects, or more to the point, triggering some useful activity when a connect completes. In the example above, one of the options registered a ChannelFutureEvent and the write was executed in the body of the listener when the connection completed successfully. What may not be obvious here is that the operationComplete method is called-back on as a result of a broadcast 

ChannelStateEvent. This is an instance of a ChannelEvent which is broadcast when a Channel changes state. The ChannelFuture handled this event by invoking the defined callback, but ChannelEvents are passed to all handlers in the pipeline, so you might implement your post-connection action in one of the handlers. I point this out because much of the Netty sample code implements this slightly more non-linear style. For example, see ObjectEchoClient where the client class does all the work of establishing a connected channel, but never actually writes anything. The write of the Object to be echoed actually occurs in the last handler in the pipeline (the ObjectEchoClientHandler) and it is executed when it receives the ChannelStateEvent indicating the Channel connected.

##What about the server ?

Following up on the DateSender, we only got as far as the server socket which is listening for data being sent by client sockets. The server side is much like the reverse of the client side.

The server socket sends ChannelEvents loaded with the byte stream it is receiving from the client into the pipeline by passing them to the sink. The pipeline needs to be configured to do the reverse so the next ChannelHandler is an ObjectDecoder which knows how to convert an array of bytes back into a Java Object, or in this case, the Date. Once the Date is decoded, it is handed to the next handler in the pipeline which is a custom handler I invented called a DateHandler. You may have noticed that although Channels have a write method, they don't have an equivalent read method. That would be more of a synchronous API where a thread would read from an OutputStream and wait until data was available. That's why the diagram above does not have an arrow back to the ServerChannel. Rather, the last handler in the pipeline receives the fully decoded message from the client and does something useful with it.

Think of server channel pipelines as being your actual business service invokers. Our actual business service is the DateReceiver and the pipeline feeds it properly unmarshalled data. Of course, you could create a pipeline that contains just one handler that does everything, but chaining together simple components provides much more flexibility. For example, if I was sending not just a Date, but perhaps an array of 300 Dates, I might want to add Compression/Decompression handlers into the client and server pipelines to shrink the size of the transmitted payload, and I can do this by simply modifying the pipeline configurations, rather than have to add additional code to my rhetorical single-do-everything handler. Or I might want to add some authentication so not just any old client can send Dates to my server..... Or what if I needed to shift the Date in transit to account for Time Zone changes..... Or......  it makes more sense to componentize.

The Server side of the DateSender is fairly simple. We create a channel factory, a pipeline factory and put the ObjectDecoder and our custom DateHandler into the pipeline. We'll use a ServerBootstrap instead of a ClientBootstrap and rather than connecting, as we did in the client, we're going to bind to a server socket so we can listen for requests from clients.

	public static void bootServer() {
	 // More terse code to setup the server
	 ServerBootstrap bootstrap = new ServerBootstrap(
	   new NioServerSocketChannelFactory(
		 Executors.newCachedThreadPool(),
		 Executors.newCachedThreadPool()));
	 
	 // Set up the pipeline factory.
	 bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
	  public ChannelPipeline getPipeline() throws Exception {
	   return Channels.pipeline(
		new ObjectDecoder(ClassResolvers.cacheDisabled(getClass().getClassLoader())),
		new DateHandler()
	   );
	  };
	 });
	 
	 // Bind and start to accept incoming connections.
	 bootstrap.bind(new InetSocketAddress("0.0.0.0", 8080));
	 slog("Listening on 8080");
	}
	 
	static class DateHandler extends SimpleChannelHandler {
	 public void messageReceived(ChannelHandlerContext ctx,MessageEvent e) throws Exception {
	  Date date = (Date)e.getMessage();
	  // Here's the REALLY important business service at the end of the pipeline
	  slog("Hey Guys !  I got a date ! [" + date + "]");
	  // Huh ?
	  super.messageReceived(ctx, e);
	 } 
	}
	
Starting at the bottom, the DateHandler extends SimpleChannelHandler which tends to be a good way to go when you're not doing anything out of the ordinary because it's callbacks are strongly typed. In other words, you do not have to listen on all ChannelEvents and test the type of the event for the ones you want. In this case, the only callback that is overridden is messageReceived which means the handler has received some actual payload.

slog and clog are simple wrappers for System.out.println but the former prefixes with [Server] and the latter with [Client] so's we can differentiate the output.


##A Quick Note on messageReceived

As I mentioned, the messageReceived method is a callback when some actual payload is being received. That is, the method handles a special subtype of the ChannelEvent called a MessageEvent. To access the payload in the MessageEvent, we simply call its getMessage() method which returns a java.lang.Object which can then be cast to the type expected. Recap: If there are no prior handlers before this handler to convert the payload, the type of the message will be..... a ChannelBuffer. In this case, however, the ObjectDecoder already converted the bytes in the incoming ChannelBuffer to a Date, so the type is java.util.Date. As for the other guy, the ChannelHandlerContext, we'll get to that shortly.

As I mentioned, the messageReceived method is a callback when some actual payload is being received. That is, the method handles a special subtype of the ChannelEvent called a MessageEvent. To access the payload in the MessageEvent, we simply call its getMessage() method which returns a java.lang.Object which can then be cast to the type expected. Recap: If there are no prior handlers before this handler to convert the payload, the type of the message will be..... a ChannelBuffer. In this case, however, the ObjectDecoder already converted the bytes in the incoming ChannelBuffer to a Date, so the type is java.util.Date. As for the other guy, the ChannelHandlerContext, we'll get to that shortly.
Once we issue the critical business event of logging the received date, the handler is technically done, so what's the score with that code on line 30 ? Since there might be additional handlers further up the pipeline, it's a good idea to always send the payload on its way once it has been handled. If there are no additional handlers, the pipeline will discard the message.
The source code for DateSender is in the GitHub repository if you want to view the whole thing, and here's what the output looks like:

	[Server]:Listening on 8080
	[Client]:DateSender Example
	[Client]:Issuing Channel Connect...
	[Client]:Waiting for Channel Connect...
	[Client]:Connected. Sending Date
	[Server]:Hey Guys !  I got a date ! [Sat May 19 14:00:58 EDT 2012]
	
Note that this example is purely one way. We only send a Date up to the server and nothing is sent back. If it were to, the pipelines on both sides would need their counterparts,which I will discuss next.


##The Return of the Date

Both clients and servers will read and write in a bidirectional scenario. Consider the DateSender example. What if the server was to increment the date by some arbitrary value and return the modified date ? There another example of almost the same code as DateSender called DateModifier  but which has a server that modifies the Date and returns it to the client. In order to do this, these modifications are required:


* The Server will return a Date back to the client, so its pipeline will need an ObjectEncoder.
* The Client will receive a Date back from the Server so its pipeline will need an ObjectDecoder.
* The Client needs an extra handler to do something with the returned Date from the Server.

Here's the code that creates the new Client pipeline:
  
	ChannelPipelineFactory pipelineFactory = new ChannelPipelineFactory() {
	  public ChannelPipeline getPipeline() throws Exception {
		return Channels.pipeline(
		  new ObjectEncoder(),
		  // Next 2 Lines are new
		  new ObjectDecoder(ClassResolvers.cacheDisabled(getClass().getClassLoader())),
		  new ClientDateHandler()
		);
	  }
	};
	
The code for the client DateHandler is very simple and it is invoked when the client receives a Date back from the server and it has been decoded:

	static class ClientDateHandler extends SimpleChannelHandler {
	 public void messageReceived(ChannelHandlerContext ctx,MessageEvent e) throws Exception {
	  Date date = (Date)e.getMessage();
	  clog("Hey Guys !  I got back a modified date ! [" + date + "]");
	 }
	}
	
The server pipeline is almost the mirror of the client now:

	bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
	 public ChannelPipeline getPipeline() throws Exception {
	  return Channels.pipeline(
	   new ObjectDecoder(ClassResolvers.cacheDisabled(getClass().getClassLoader())),
	   // Next 2 Lines are new
	   new ObjectEncoder(),
	   new ServerDateHandler()
	  );
	 };
	});


This is the code for the new Server DateHandler which has a few new concepts.
 
	static class ServerDateHandler extends SimpleChannelHandler {
	 Random random = new Random(System.nanoTime());
	 public void messageReceived(ChannelHandlerContext ctx,MessageEvent e) throws Exception {
	  Date date = (Date)e.getMessage();
	  // Here's the REALLY important business service at the end of the pipeline
	  long newTime = (date.getTime() + random.nextInt());
	  Date newDate = new Date(newTime);
	  slog("Hey Guys !  I got a date ! [" + date + "] and I modified it to [" + newDate + "]");
	  // Send back the reponse
	  Channel channel = e.getChannel();
	  ChannelFuture channelFuture = Channels.future(e.getChannel());
	  ChannelEvent responseEvent = new DownstreamMessageEvent(channel, channelFuture, newDate, channel.getRemoteAddress());
	  ctx.sendDownstream(responseEvent);
	  // But still send it upstream because there might be another handler
	  super.messageReceived(ctx, e);
	 } 
	}
	
There's some new stuff on lines 10-13 dedicated specifically to returning the modified Date to the calling client and a subtle concept.

* On line 10, we get a reference to the Channel from the MessageEvent.
* The Date is going to be written back to the client, which (at the risk of repeating myself) will be an asynchronous call, so there's always a ChannelFuture involved, so in this case, on line 11,  we're going to create one using the Channels class, a class stuffed full of useful static methods like future.
* On line 12, a new MessageEvent is created by constructing a DownstreamMessageEvent. Basically, the return value is being packaged up to be sent back down the client.
* On line 13, the new MessageEvent is sent back down to the client by calling sendDownstream on the ChannelHandlerContext.


Say what ?  Ok, the ChannelHandlerContext is basically a functional reference to the pipeline. It provides access to all the handlers in the pipeline as well as to the pipeline itself, and most importantly, in this case, it provides a means of sending a message back through the "same" path that the incoming message arrived on. The important points here are:

    Keeping in mind that there may be additional handlers in the pipeline beyond the ServerDateHandler, I want the handler to respond to the client by sending the modified Date back and then forward the current payload to the next handler in the pipeline.
    If we wrote to the Channel directly, it would start from the "top" of the pipeline and might be pushed through handlers that are not indented to be called with the return Date.

All this begs some additional details on how exactly handlers work inside of a pipeline and an explanation of Upstream and Downstream.


##Upstream and Downstream

You might wonder why things don't get confused when both the ObjectEncoder and the ObjectDecoder are called within one pipeline invocation, after all, they're both in the pipeline, wouldn't they both get called ? They're not because handlers declare themselves as Upstream handlers, Downsteam handlers or both.Basically Downstream is when a pipeline is sending something to a remote. Upstream is when it is reading something from a remote. If the terminology is not immediately intuitive to you, think of this mnemonic sentence:

    If you want to learn something, you might read up on it and then write down some notes.

... anyways, for handlers to participate in a pipeline, they will directly or indirectly implement one or both of these interfaces:

    org.jboss.netty.channel.ChannelDownstreamHandler: Invoked on all downstream calls
    org.jboss.netty.channel.ChannelUpstreamHandler: Invoked on all upstream calls

Therefore, on a downstream write, the pipeline orchestrates the payload to be passed to all channel handlers that implement ChannelDownstreamHandler and vice-versa. Keep in mind that a channel handler can implement both interfaces and participate in upstream and downstream events.

[Upstream_Downstream]()

Notice that the notion of Upstream vs. Downstream is specific to the client and the server, and for a conversation between a client and a server, both sides will need Encoder/Decoder pairs defined in the correct order.  The next diagram illustrates this in the form of a simplified HTTP server serving a file to a requesting client.


##Sequence of ChannelHandlers in the Pipeline

In order to achieve the proper sequence of modifications of the payload, the pipeline keeps strict ordering of the handlers in the pipeline. Consider a pipeline that is supposed to look like this:

###JSON Encoder: 
       
        Receives: A widget object
        Produces: A JSON string representing the widget object.
    
###String Encoder:
   
        Receives: A string
        Produces: A ChannelBuffer with the encoded string

If the order of these encoders were accidentally reversed, the String Encoder would receive a widget Object when it expects a string and a class cast exception would occur.
There are a few constructs that allow for specifying the order of handlers in the pipeline. The simplest is probably the static method pipeline in the org.jboss.netty.channel.Channels class which creates a pipeline placing handlers in the created pipeline in the same order they are specified in the passed array. The signature is:

	static ChannelPipeline Channels.pipeline(ChannelHandler...handlers)
	
In the ChannelPipeline interface itself provides a number of location specifying methods to add handlers, and herein we can also assign handlers specific logical names to uniquely identify each handler in the pipeline. The name itself is purely arbitrary, but as we will see, it is sometimes necessary to be able to reference a handler directly by its name, or determine its positional notation via the assigned name. The handler names are actually mandatory and assigned automatically if you do not specify them yourself. The pipeline methods are as follows:


* addAfter(String baseName, String name, ChannelHandler handler): Adds a handler to the pipeline name handler and it is placed directly after the handler named baseName.
* addBefore(String baseName, String name, ChannelHandler handler): Adds a handler to the pipeline name handler and it is placed directly before the handler named baseName.
* addFirst(String name, ChannelHandler handler): Adds a handler named name to the start of the pipeline (it becomes the first handler in the pipeline)
* addLast(String name, ChannelHandler handler): Adds a handler named name to the end of the pipeline (it becomes the last handler in the pipeline)


##Dynamically Modifying ChannelHandlers in the Pipeline

Another aspect of pipelines is that they are not immutable*, so it is possible to add and remove handlers at runtime. (There will be a very salient example of this in Part 2). Netty advertises the pipeline as thread-safe for this exact purpose, the four methods above can be called at runtime in order to "fine-tune" the pipeline for a specific call or to switch state.

	Actually there is an immutable implementation called a StaticChannelPipeline. Since it is immutable once created, it cannot be modified and may provide better performance since the pipeline execution does not have to be guarded against runtime changes of the 
handlers.

Why might we want to suddenly change the handlers in a pipeline that we went to all the bother of creating just-so in the first place ? Here's a contrived example: Say you have a bunch of encoders in a client pipeline, with the last encoder being a ZlibEncoder which will compress the encoded payload before sending it out the door. However, let's say you have determined that if the payload is less than 1024 bytes, the compression step is actually a hindrance to performance, not a benefit. This means that you want to conditionally route the payload through the compression handler. Hmmmm.... since you won't actually know the size of the payload until it has passed through the preceding handlers, you don't even have the option of creating a compression enabled pipeline and a compressionless pipeline because your payload is most of the way down the pipeline before you know which one you want. To address this, we can create a content interrogating handler that examines the size of the payload to be compressed [or not] and add or remove the compression handler accordingly. We'll add this handler after all the other handlers, but before the compression handler.

At this point, you might be thinking there are several alternatives here, but I did say it was contrived, so bear with me.

First, we will create the pipeline with the compression handler enabled, then we'll look at the ConditionalCompressionHandler. Here's what the pipeline might look like:

	public ChannelPipeline getPipeline() throws Exception {
	 ChannelPipeline pipeline = Channels.pipeline();
	 /// Add all the pre-amble handlers
	 //pipeline.addLast("Foo", new SomeHandler());
	 //pipeline.addLast("Bar", new SomeOtherHandler());
	 // etc.
	 pipeline.addLast("IAmTheDecider", new ConditionalCompressionHandler(1024, "MyCompressionHandler"));
	 pipeline.addLast("MyCompressionHandler", new ZlibEncoder());
	 return pipeline;
	}
	
The ConditionalCompressionHandler must examine the size of the ChannelBuffer it is passed and decide if the compression handler should be called or not. However, once removed, if the next payload that comes along requires compression, we need to add it back in again.

	public class ConditionalCompressionHandler extends SimpleChannelDownstreamHandler {
	 /** The minimum size of a payload to be compressed */
	 protected final int sizeThreshold;
	 /** The name of the handler to remove if the payload is smaller than specified sizeThreshold */
	 protected final String nameOfCompressionHandler;
	 /** The compression handler */
	 protected volatile ChannelHandler compressionHandler = null;
	  
	 /**
	  * Creates a new ConditionalCompressionHandler
	  * @param sizeThreshold The minimum size of a payload to be compressed
	  * @param nameOfCompressionHandler The name of the handler to remove if the payload is smaller than specified sizeThreshold
	  */
	 public ConditionalCompressionHandler(int sizeThreshold, String nameOfCompressionHandler) {
	  this.sizeThreshold = sizeThreshold;
	  this.nameOfCompressionHandler = nameOfCompressionHandler;
	 }
	  
	 /**
	  * see org.jboss.netty.channel.SimpleChannelDownstreamHandler
	  */
	 public void writeRequested(ChannelHandlerContext ctx, MessageEvent e) {
	  // If the message is not a ChannelBuffer, hello ClassCastException !
	  ChannelBuffer cb = (ChannelBuffer)e.getMessage();
	  // Check to see if we already removed the handler
	  boolean pipelineContainsCompressor = ctx.getPipeline().getContext(nameOfCompressionHandler)!=null;
	  if(cb.readableBytes() < sizeThreshold) { 
	   if(pipelineContainsCompressor) {
		// The payload is too small to be compressed but the pipeline contains the compression handler
		// so we need to remove it.
		compressionHandler = ctx.getPipeline().remove(nameOfCompressionHandler);
	   }
	  } else {
	   // We want to compress the payload, let's make sure the compressor is there
	   if(!pipelineContainsCompressor) {
		// Oops, it's not there, so lets put it in
		ctx.getPipeline().addAfter(ctx.getName(), nameOfCompressionHandler , compressionHandler);
	   }
	  }
	 }
	  
	}
	
###ChannelHandlerContext

Note that in pretty much every handler event, the handler is passed a ChannelHandlerContext. This class is an adapter between a ChannelHandler and the ChannelPipeline that contains it, allowing a handler to interact with the pipeline, a crucial aspect of the preceding code sample. In addition, it allows a handler to inquire about other handlers and even interact with other handlers (though more indirectly).

    On line 26 we determine if the compression handler is installed by requesting the pipeline from the ChannelHandlerContext and then requesting the ChannelHandlerContext of the compression handler. If the compression handler is absent, the call will return null.
    On line 31, the handler requests the pipeline and removes the compression handler from the pipeline by name.
    On line 37, the handler requests the pipeline and adds the compression handler back into the pipeline. We also use the getName() method of the ChannelHandlerContext to get the name of the current handler.

That's it for this entry.


