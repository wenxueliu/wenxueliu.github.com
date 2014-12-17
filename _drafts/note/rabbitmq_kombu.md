
典型的生产者-消费者企业级消息架构，即高可用、高性能


生产者生产的消息 --> 交换器 --> RouteKey --> Match --> BindKey --> 消息队列(必须与交换器绑定) --> 消费者消费消息


客户端
=======================================
生产者(Invoker)
	产生信息：想做什么事情

消费者(Worker)
	消费信息：按照要求做事情

服务端
=======================================

交换器（exchange）
---------------------------------------
功能
	接受和路由消息，将匹配的消息发送到消息队列。
特点
	持久(Durable):默认，即使重启 AMQP 也不删除
	临时(Non-durable,transient) ： 当AMQP服务关闭或重启的时候删除
	自动删除(Auto_delete):默认 False，当没有消息队列使用的时候就删除

类型
	Direct
		一对多OR多对一：即相同信息可能被发送到多个消息队列或多个信息被发送到同一消息队列，关键是 RouteKey 的精确匹配决定
	Fanout
		一对多：即发送到被无条件地发送到所有与当前交换器绑定的队列。与消息队列直接没有 RouteKey。
	Topic
		一对多或多对一：即相同信息可能被发送到多个消息队列或多个信息被发送到同一消息队列，关键是 RouteKey 的正则匹配决定


队列（queue）
---------------------------------------
功能
	存储和转发消息
特点
	持久(Durable)
	临时(Non-durable,transient)
	自动删除(Auto_delete)
	有限制的先进先出
	出队：acquire + consume

绑定关键词（BindKey）
---------------------------------------
	各个关键词以“.” 分隔
	精确匹配：完全一样
	正则匹配：支持 "*","#"通配符


virtual host 
--------------------------------------
the multi-tenancy mechanism provided by Qpid or RabbitMQ,his virtual host must exist on the server, and the user must have access to it. Default is “/”.

broker
--------------------------------------

Invoker 
--------------------------------------
API or Scheduler
	An Invoker is a component that sends messages in the queuing system via two operationss 
operations:
	 1) rpc.call and ii) rpc.cast;

Worker
--------------------------------------
Compute or Network
a Worker is a component that receives messages from the queuing system and reply accordingly to rcp.call operations.

Exchanger
--------------------------------------
a routing table that exists in the context of a virtual host


Topic Publisher
--------------------------------------
	function : push a message to the queuing system
	come to life : an rpc.call or an rpc.cast operation is executed
	life-cycle : message delivery

Direct Consumer
--------------------------------------
	function : receive a response message from the queuing system
	come to life : rpc.call operation is executed
	life-cycle : message delivery

Topic Consumer
--------------------------------------

	function : receive messages from the queue and it invokes the appropriate action as defined by the Worker role
	come to life :Worker is instantiated
	life-cycle : Worker's life-cycle
	other : 
		a Worker has two topic consumers
		1)	rpc.cast,exchange key is ‘topic’ 
		2)  rpc.call,exchange key is ‘topic.host’

Direct Publisher
--------------------------------------
	function : return the message required by the request/response operation
	comes to life : during rpc.call operations

Topic Exchange
--------------------------------------
a message broker node will have only one topic-based exchange for every topic in Nova.

Direct Exchange
--------------------------------------
    come to life :  a routing table that is created during rpc.call operations
	lief-cycle : throughout the life-cycle of a message broker node

Queue Element
--------------------------------------
A Queue is a message bucket which is waiting for Consumer (either Topic or Direct Consumer) connects to the queue and fetch it


[nova rpc 官方文档](http://docs.openstack.org/developer/nova/devref/rpc.html)
