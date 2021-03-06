---
layout: post
category : http
tagline: "详解"
tags : [ http, network]
---
{% include JB/setup %}

###Http Server WorkFlow
1. Set up connection—accept a client connection, or close if the client is unwanted.
2. Receive request—read an HTTP request message from the network.
3. Process request—interpret the request message and take action.
4. Access resource—access the resource specified in the message.
5. Construct response—create the HTTP response message with the right headers.
6. Send response—send the response back to the client.
7. Log transaction—place notes about the completed transaction in a log file.


<img src="{{IMAGE_PATH}}/http/http-server-process.png" title="http server process" width="600" />


###Accepting Client Connections

* Handling New Connections
* Client Hostname Identification
Most web servers can be configured to convert client IP addresses into client host-
names, using “reverse DNS.”
* Determining the Client User Through ident (drop-out???)
find out what username initiated an HTTP connection,This information is particularly useful for web server logging


###Receiving Request Messages

* Parses the request line looking for the request method, the specified resource
identifier (URI), and the version number,* each separated by a single space, and
ending with a carriage-return line-feed (CRLF) sequence†
* Reads the message headers, each ending in CRLF
* Detects the end-of-headers blank line, ending in CRLF (if present)
* Reads the request body, if any (length specified by the Content-Length header)

####Internal Representations of Messages
TODO

####Connection Input/Output Processing Architectures

Single-threaded web servers
Multiprocess and multithreaded web servers
Multiplexed I/O servers
Multiplexed multithreaded web servers 

###Processing Requests

received a request, it can process the request using the
method, resource, headers, and optional body.

###Mapping and Accessing Resources

mapping the URI from the request message to the proper content
or content generator on the web server.

####Docroots

a special folder in the web server filesystem is reserved for web content.
This folder is called the document root, or docroot. The web server takes the URI
from the request message and appends it to the document root.

####Virtually hosted docroots

Virtually hosted web servers host multiple web sites on the same web server, giving
each site its own distinct document root on the server.

####User home directory docroots

TODO


####Directory Listings

gives people private web sites on a web server. A
typical convention maps URIs whose paths begin with a slash and tilde (/~) fol-
lowed by a username to a private document root for that user. 

####Dynamic Content Resource Mapping

map URIs to dynamic resources—that is, to programs that gen-
erate content on demand

####Server-Side Includes (SSI)

If a resource is flagged as containing server-side includes, the server processes the resource contents
before sending them to the client.

####Access Controls

Web servers also can assign access controls to particular resources.


###Building Responses

Once the web server has identified the resource, it performs the action described in
the request method and returns the response message. The response message con-
tains a response status code, response headers, and a response body if one was gener-
ated

####Response Entities

* Content-Type header
* Content-Length header
* message body conten

####MIME Typing

* mime.types
use the extension of the filename to indicate MIME type
* Magic typing
scan the contents of each resource and pattern-
match the content against a table of known patterns (called the magic file) to
determine the MIME type for each file. 
* Explicit typing 
force particular files or directory contents to
have a MIME type, regardless of the file extension or contents.
* Type negotiation
configured to store a resource in multiple document
formats


####Redirection

* Permanently moved resources
* Temporarily moved resources
* URL augmentation
* Load balancing
* Server affinity
* Canonicalizing directory names


###Sending Responses

TODO


###Logging

TODO



###HTTP协议详解之URL篇

HTTP 是 Hyper Text Transfer Protocol（超文本传输协议）的缩写。它的发展是万维网协会（World Wide Web Consortium）和Internet工作小组IETF（Internet Engineering Task Force）合作的结果，（他们）最终发布了一系列的RFC，RFC 1945定义了HTTP/1.0版本。其中最著名的就是RFC 2616。RFC 2616定义了今天普遍使用的一个版本——HTTP 1.1。

HTTP 协议是用于从 WWW 服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示(如文本先于图形)等。

HTTP 是一个基于请求与响应模式的、无状态的、应用层的协议，常基于TCP的连接方式，HTTP1.1 版本中给出一种持续连接的机制，绝大多数的 Web 开发，都是构建在 HTTP 协议之上的 Web 应用。


在TCP/IP协议栈中的位置

HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候，就成了我们常说的HTTPS。如下图所示：

<img src="{{ IMAGE_PATH }}/http/http-arch.png" title="http-arch" width="600" />

默认HTTP的端口号为80，HTTPS的端口号为443。

URIs(uniform resource identifier) 包括 URLs(Uniform resource locators) 和 URNs(uniform resource names)

HTTP URL 是一种特殊类型的URI，包含了用于查找某个资源的足够的信息)的格式如下：

	<scheme>   ://	<user>:<password>@<host>:<port>  /	<path>;<params>?<query>#<frag>

* scheme 	Which protocol to use,case-insensitive   								None
* user 		The username some schemes require to access a resource. 				anonymous
* password 	The password that may be included after the username 				
* host 		The hostname or dotted IP address of the server hosting the resource. 	None
* port 		The port number on which the server hosting the resource is listening. 	Scheme-specific
* path 		The local name for the resource on the server, divided into segments 	None
* params 	specify input parameters, key/value pairs. separate by semicolons (;) 	None
* query		parameters to active applications，identifying the gateway resource		None
* frag 		specify the sections within the resource. It is separated by the “#” 		None
 
例：
 	http://www.joes-hardware.com/hammers;sale=false/index.html;graphics=true#profile

**注**

事实上，打开一个网页需要浏览器发送很多次Request，如下

1. 当你在浏览器输入URL http://www.google.com 的时候，浏览器发送一个Request去获取 http://www.google.com 的html.
服务器把Response发送回给浏览器.

2. 浏览器分析Response中的 HTML，发现其中引用了很多其他文件，比如图片，CSS文件，JS文件。

3. 浏览器会自动再次发送Request去获取图片，CSS文件，或者JS文件。

4. 等所有的文件都下载成功后。 网页就被显示出来了。


HTTP协议详解之请求篇
============================

http 请求由三部分组成，分别是：请求行(Request line)、请求报头(Request header)、请求正文(body)

####请求行

包括方法、请求的URI和协议的版本，以空格分开，格式如下：

	Method Request-URI HTTP-Version CRLF 

* Method : 表示请求方法,当为 GET 的时候，body 为空
* Request-URI : 一个统一资源标识符
* HTTP-Version : 表示请求的HTTP协议版本
* CRLF : 表示回车和换行，即\r\n（除了作为结尾的CRLF外，不允许出现单独的CR或LF字符）。

####请求方法

（所有方法全为大写）有多种，各个方法的解释如下：

	GET     请求获取Request-URI所标识的资源
	POST    在Request-URI所标识的资源后附加新的数据(GET 和 POST 的区别见附录)
	HEAD    请求获取由Request-URI所标识的资源的响应消息报头
	PUT     请求服务器存储一个资源，并用Request-URI作为其标识
	DELETE  请求服务器删除Request-URI所标识的资源
	TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
	CONNECT 保留将来使用
	OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求


例如

GET方法：在浏览器的地址栏中输入网址的方式访问网页时，浏览器采用GET方法向服务器获取资源，比如

	GET /form.html HTTP/1.1 (CRLF)

POST方法： 要求被请求服务器接受附在请求后面的数据，常用于提交表单。比如

	POST /reg.jsp HTTP/ (CRLF)

	Accept:image/gif,image/x-xbit,... (CRLF)
	...
	HOST:www.nit.edu.cn (CRLF)
	Content-Length:22 (CRLF)
	Connection:Keep-Alive (CRLF)
	Cache-Control:no-cache (CRLF)
	(CRLF)         //该CRLF表示消息报头已经结束，在此之前为消息报头
	user=jeffrey&pwd=1234  //此行以下为提交的数据

HEAD方法与GET方法几乎是一样的，对于HEAD请求的回应部分来说，它的HTTP头部中包含的信息与通过GET请求所得到的信息是相同的。利用这个方法，不必传输整个资源内容，就可以得到 Request-URI 所标识的资源的信息。该方法常用于测试超链接的有效性，是否可以访问，以及最近是否更新。

####请求报头

后述

####请求正文

(略) 


###HTTP协议详解之响应篇
 

HTTP响应也是由三个部分组成，分别是：状态行(Response line)、消息报头(Response header)、响应正文(body)

####状态行

格式如下：

	HTTP-Version Status-Code Reason-Phrase CRLF
	
* HTTP-Version : 表示服务器HTTP协议的版本
* Status-Code : 表示服务器发回的响应状态代码
* Reason-Phrase : 表示状态代码的文本描述。

状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：


	1xx：指示信息--表示请求已接收，继续处理
	2xx：成功--表示请求已被成功接收、理解、接受
	3xx：重定向--要完成请求必须进行更进一步的操作
	4xx：客户端错误--请求有语法错误或请求无法实现
	5xx：服务器端错误--服务器未能实现合法的请求


常见状态代码、状态描述、说明：

	200 OK      //客户端请求成功
    304 Not Modified  //客户端已经缓存所请求的内容
	400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
	401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用
	403 Forbidden  //服务器收到请求，但是拒绝提供服务
	404 Not Found  //请求资源不存在，eg：输入了错误的URL
	500 Internal Server Error //服务器发生不可预期的错误
	503 Server Unavailable  //服务器当前不能处理客户端的请求，一段时间后，可能恢复正常


例如：

HTTP/1.1 200 OK （CRLF）


####响应报头

后述

####响应正文

服务器返回的资源的内容


###HTTP协议详解之消息报头篇

HTTP 消息由客户端到服务器的请求和服务器到客户端的响应组成。请求消息和响应消息都是由开始行（对于请求消息，开始行就是请求行，对于响应消息，开始行就是状态行），消息报头（可选），空行（只有CRLF的行），消息正文（可选）组成。

HTTP 消息报头包括普通报头、请求报头、响应报头、实体报头。

每一个报头域都是由名字 + “：” + 空格 + 值组成，消息报头域的名字是大小写无关的。


普通报头
----------------------
在普通报头中，有少数报头域用于所有的请求和响应消息，但并不用于被传输的实体，只用于传输的消息。

**Cache-Control**

Cache-Control 指定请求和响应遵循的缓存机制。在请求消息或响应消息中设置。通过缓存指令来指定，缓存指令是单向的（响应中出现的缓存指令在请求中未必会出现），且是独立的（一个消息的缓存指令不会影响另一个消息处理的缓存机制），HTTP1.0使用的类似的报头域为 Pragma。

请求时的缓存指令包括：no-cache（用于指示请求或响应消息不能缓存）、no-store、max-age、max-stale、min-fresh、only-if-cached

响应时的缓存指令包括：public、private、no-cache、no-store、no-transform、must-revalidate、proxy-revalidate、max-age、s-maxage.

比如

	Cache-Control : max-age=0\r\n

**Date**

Date 普通报头域表示消息产生的日期和时间，时间的描述格式由rfc822定义。例如，Date:Mon,31 Dec 2014 04:25:57GMT。Date描述的时间表示世界标准时，换算成本地时间，需要知道用户所在的时区。比如

	Date :  Mon, 30 Nov 2013 13:59:59 GMT\r\n 

 

**Connection**

在 http1.1，request 和 reponse 头中都有可能出现一个 connection 的头，此 header 的含义是当 client 和 server 通信时对于长链接如何进行处理。

在 http1.1中，client 和 server 都是默认对方支持长连接的， 如果 client 使用 http1.1 协议，但又不希望使用长连接，则需要在 header 中指明 connection 的值为 close；如果server方也不想支持长连接，则在 response 中也需要明确说明 connection 的值为 close。不论 request 还是 response 的 header 中包含了值为 close 的 connection，都表明当前正在使用的 tcp 连接在当天请求处理完毕后会被断掉。以后 client 再进行新的请求时就必须创建新的 tcp 连接了。


请求报头
--------------------------

请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信息。

常用的请求报头

**Accept**

Accept 请求报头域用于指定客户端接受哪些类型的信息。比如 

	Accept：image/gif，表明客户端希望接受GIF图象格式的资源；
	Accept：text/html，表明客户端希望接受html文本。


**Accept-Charset**

Accept-Charset 请求报头域用于指定客户端接受的字符集。比如

	Accept-Charset:iso-8859-1,gb2312.如果在请求消息中没有设置这个域，缺省是任何字符集都可以接受。


**Accept-Encoding**

Accept-Encoding 请求报头域类似于Accept，但是它是用于指定可接受的内容编码。比如

	Accept-Encoding:gzip.deflate
	
如果请求消息中没有设置这个域服务器假定客户端对各种内容编码都可以接受。Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间；

**Accept-Language**

Accept-Language 请求报头域类似于 Accept，但是它是用于指定一种自然语言。比如

	Accept-Language:zh-cn.

如果请求消息中没有设置这个报头域，服务器假定客户端对各种语言都可以接受。


**Authorization**

Authorization 请求报头域主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收到服务器的响应代码为401（未授权），可以发送一个包含Authorization请求报头域的请求，要求服务器对其进行验证。


**Host**（发送请求时，该报头域是必需的）

Host请求报头域主要用于指定被请求资源的Internet主机和端口号，它通常从HTTP URL中提取出来的，比如，我们在浏览器中输入：

	http://www.nit.edu.cn/index.html

浏览器发送的请求消息中，就会包含 Host 请求报头域，如下：

	Host：www.nit.edu.cn

此处使用缺省端口号80，若指定了端口号，则变成：Host：www.nit.edu.cn:指定端口号


**User-Agent**

我们上网登陆论坛的时候，往往会看到一些欢迎信息，其中列出了你的操作系统的名称和版本，你所使用的浏览器的名称和版本，这往往让很多人感到很神奇，实际上，服务器应用程序就是从 User-Agent 这个请求报头域中获取到这些信息。User-Agent请求报头域允许客户端将它的操作系统、浏览器和其它属性告诉服务器。不过，这个报头域不是必需的，如果我们自己编写一个浏览器，不使用User-Agent请求报头域，那么服务器端就无法得知我们的信息了。

**Referer**

Referer头域允许客户端指定请求uri的源资源地址，这可以允许服务器生成回退链表，可用来登陆、优化cache等。他也允许废除的或错误的连接由于维护的目的被追踪。如果请求的uri没有自己的uri地址，Referer不能被发送。如果指定的是部分uri地址，则此地址应该是一个相对地址。比如

	Referer : http://imge.google.com/\r\n

**Content-Length**

表示请求消息正文的长度；	

**Cookie**

这是最重要的请求头信息之一

**From**

请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它；

**If-Modified-Since**

只有当所请求的内容在指定的日期之后又经过修改才返回它，否则返回304“Not Modified”应答；

**Pragma**
 
指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝



请求报头举例：

	GET /form.html HTTP/1.1 (CRLF)
	Accept:image/gif,image/x-xbitmap,image/jpeg,application/x-shockwave-flash,application/vnd.ms-excel,application/vnd.ms-powerpoint,application/msword,*/* (CRLF)
	Accept-Language:zh-cn (CRLF)
	Accept-Encoding:gzip,deflate (CRLF)
	If-Modified-Since:Wed,05 Jan 2007 11:21:25 GMT (CRLF)
	If-None-Match:W/"80b1a4c018f3c41:8317" (CRLF)
	User-Agent:Mozilla/4.0(compatible;MSIE6.0;Windows NT 5.0) (CRLF)
	Host:www.nit.edu.cn (CRLF)
	Connection:Keep-Alive (CRLF)
	(CRLF)
	
	
响应报头
-----------------------

响应报头允许服务器传递不能放在状态行中的附加响应信息，以及关于服务器的信息和对 Request-URI 所标识的资源进行下一步访问的信息。

常用的响应报头

**Allow**

服务器支持哪些请求方法（如GET、POST等）

**Location**

Location 响应报头域用于重定向接受者到一个新的位置。Location响应报头域常用在更换域名的时候。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302；


**Server**

Server 响应报头域包含了服务器用来处理请求的软件信息。与User-Agent请求报头域是相对应的。下面是Server响应报头域的一个例子：

	Server：Apache-Coyote/1.1


**WWW-Authenticate**

WWW-Authenticate响应报头域必须被包含在 401（未授权的）响应消息中，客户端收到 401 响应消息时候，并发送 Authorization 报头域请求服务器对其进行验证时，服务端响应报头就包含该报头域。比如

	WWW-Authenticate:Basic realm="Basic Auth Test!"  //可以看出服务器对请求资源采用的是基本验证机制。

**Refresh**

表示浏览器应该在多少时间之后刷新文档，以秒计。

实体（Entity）报头
-----------------------

请求和响应消息都可以传送一个实体。一个实体由实体报头域和实体正文组成，但并不是说实体报头域和实体正文要在一起发送，可以只发送实体报头域。实体报头定义了关于实体正文（如，有无实体正文）和请求所标识的资源的元信息。

常用的实体报头

**Content-Encoding**

Content- Encoding 实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得 Content-Type 报头域中所引用的媒体类型，必须采用相应的解码机制。Content-Encoding这样用于记录文档的压缩方法，比如：

	Content-Encoding：gzip


**Content-Language**

Content-Language实体报头域描述了资源所用的自然语言。没有设置该域则认为实体内容将提供给所有的语言阅读
者。比如：

	Content-Language:da


**Content-Length**

Content-Length实体报头域用于指明实体正文的长度，以字节方式存储的十进制数字来表示。

**Content-Type**

Content-Type实体报头域用语指明发送给接收者的实体正文的媒体类型。比如

	Content-Type:text/html;charset=ISO-8859-1
	Content-Type:text/html;charset=GB2312


**Last-Modified**

Last-Modified实体报头域用于指示资源的最后修改日期和时间。


**Expires**

Expires 实体报头域给出响应过期的日期和时间。为了让代理服务器或浏览器在一段时间以后更新缓存中(再次访问曾访问过的页面时，直接从缓存中加载，缩短响应时间和 降低服务器负载)的页面，我们可以使用 Expires 实体报头域指定页面过期的时间。比如

	Expires：Thu，15 Sep 2006 16:23:12 GMT
	
HTTP1.1的客户端和缓存必须将其他非法的日期格式（包括0）看作已经过期。比如，为了让浏览器不要缓存页面，我们也可以利用 Expires 实体报头域，设置为0。

**Content-MD5**

MD5
实体的一种MD5摘要，用作校验和。发送方和接受方都计算MD5摘要，接受方将其计算的值与此头标中传递的值进行比较。比如：Content-MD5: base64 of 128 MD5 digest。

**Content-Range**

随部分实体一同发送；标明被插入字节的低位与高位字节偏移，也标明此实体的总长度。比如：Content-Range: 1001-2000/5000，eg2：bytes 2543-4532/7898

**注**
请求报头必须包含 Host
响应抱头必须包含 Content-Length 或 Transfer-Encodeing

附录
======================

GET 和 POST 的区别
---------------------

  GET方式：是以实体的方式得到由请求URI所指定资源的信息，如果请求URI只是一个数据产生过程，那么最终要在响应实体中返回的是处理过程的结果所指向的资源，而不是处理过程的描述。

  POST方式：用来向目的服务器发出请求，要求它接受被附在请求后的实体，并把它当作请求队列中请求URI所指定资源的附加新子项，Post被设计成用统一的方法实现下列功能：

1：对现有资源的解释；

2：向电子公告栏、新闻组、邮件列表或类似讨论组发信息；

3：提交数据块；

4：通过附加操作来扩展数据库 。

从上面描述可以看出，Get是向服务器发索取数据的一种请求；而Post是向服务器提交数据的一种请求，要提交的数据位于信息头后面的实体中。

GET与POST方法有以下区别：

（1）   在客户端，Get方式在通过URL提交数据，数据在URL中可以看到；POST方式，数据放置在HTML HEADER内提交。

（2）   GET方式提交的数据最多只能有1024字节，而POST则没有此限制。

（3）   安全性问题。正如在（1）中提到，使用 Get 的时候，参数会显示在地址栏上，而 Post 不会。所以，如果这些数据是中文数据而且是非敏感数据，那么使用 get；如果用户输入的数据不是中文字符而且包含敏感数据，那么还是使用 post为好。

（4）   安全的和幂等的。所谓安全的意味着该操作用于获取信息而非修改信息。幂等的意味着对同一 URL 的多个请求应该返回同样的结果。完整的定义并不像看起来那样严格。换句话说，GET 请求一般不应产生副作用。从根本上讲，其目标是当用户打开一个链接时，她可以确信从自身的角度来看没有改变资源。比如，新闻站点的头版不断更新。虽然第二次请求会返回不同的一批新闻，该操作仍然被认为是安全的和幂等的，因为它总是返回当前的新闻。反之亦然。POST 请求就不那么轻松了。POST 表示可能改变服务器上的资源的请求。仍然以新闻站点为例，读者对文章的注解应该通过 POST 请求实现，因为在注解提交之后站点已经不同了（比方说文章下面出现一条注解）。
 
HTTP/1.0和HTTP/1.1的比较
-------------------------

RFC 1945定义了HTTP/1.0版本，RFC 2616定义了HTTP/1.1版本。

笔者在blog上提供了这两个RFC中文版的下载地址。


**建立连接方面**

HTTP/1.0 每次请求都需要建立新的TCP连接，连接不能复用。HTTP/1.1 新的请求可以在上次请求建立的TCP连接之上发送，连接可以复用。优点是减少重复进行TCP三次握手的开销，提高效率。

注意：在同一个TCP连接中，新的请求需要等上次请求收到响应后，才能发送。

**Host**

HTTP1.1在Request消息头里头多了一个Host域, HTTP1.0则没有这个域。

比如

    GET /pub/WWW/TheProject.html HTTP/1.1
    Host: www.w3.org

可能HTTP1.0的时候认为，建立TCP连接的时候已经指定了IP地址，这个IP地址上只有一个host。

**Date**

(接收方向)

无论是HTTP1.0还是HTTP1.1，都要能解析下面三种date/time stamp：

	Sun, 06 Nov 1994 08:49:37 GMT ; RFC 822, updated by RFC 1123
	Sunday, 06-Nov-94 08:49:37 GMT ; RFC 850, obsoleted by RFC 1036
	Sun Nov 6 08:49:37 1994       ; ANSI C's asctime() format

(发送方向)

HTTP1.0要求不能生成第三种asctime格式的date/time stamp；

HTTP1.1则要求只生成RFC 1123(第一种)格式的date/time stamp。

**状态响应码**

状态响应码100 (Continue) 状态代码的使用，允许客户端在发request消息body之前先用request header试探一下server，看server要不要接收request body，再决定要不要发request body。

客户端在Request头部中包含

Expect: 100-continue

       Server看到之后呢如果回100 (Continue) 这个状态代码，客户端就继续发request body。这个是HTTP1.1才有的。

另外在HTTP/1.1中还增加了101、203、205等等性状态响应码

**请求方式**

HTTP1.1增加了OPTIONS, PUT, DELETE, TRACE, CONNECT这些Request方法.

       Method         = "OPTIONS"                ; Section 9.2

                      | "GET"                    ; Section 9.3

                      | "HEAD"                   ; Section 9.4

                      | "POST"                   ; Section 9.5

                      | "PUT"                    ; Section 9.6

                      | "DELETE"                 ; Section 9.7

                      | "TRACE"                  ; Section 9.8

                      | "CONNECT"                ; Section 9.9

                      | extension-method

       extension-method = token
