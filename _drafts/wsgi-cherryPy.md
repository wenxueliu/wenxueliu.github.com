

最简描述
============================
HttpServer 维护一个 ThreadPool（通过队列实现），每接受一个连接，从 ThreadPool 取出一个 WorkerThread， 调用 WorkerThread 的 run 方法来处理请求与应答。


HttpServer
============================
self.requests = ThreadPool()
self.socket = socket.socket()

start()
-----------------------
* bind_addr 确定 host port
* bind()
	self.socket = socket.socket()   
	self.socket.setsockopt()  
	self.socket.bind()
	
* self.socket.listen()

* ThreadPool().start()
	
* tick()
	self.socket.accept()  
	conn = HttpConnection()  
	ThreadPool().put(conn)
	
	
stop()
-----------------------
* self.socket = None
* threadPool.stop()


HttpConnection
=======================
self.req = HttpRequest

communicate()
-----------------------
* self.req.parse_request()
* self.req.respond()
	
	
HttpRequest
=======================

parse_request()

* read_request_line()
	method, uri, protocol = rfile.readline()
	scheme, authority, path = parse_request_uri(uri)
* read_request_headers()

respond()

* ChunkedRFile OR KnownLengthRFile
* gateway().respond()

read_request_line()


