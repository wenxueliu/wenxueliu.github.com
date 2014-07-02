

最简描述
============================
 一个 ThreadPool 维护一组 HttpServer（默认 5 个）
 
 ThreadPool start() 开启 5 个 WorkerThread（线程），每个 WorkerThread run()(死循环) 从 ThreadPool 维护的队列取出一个 conn(HttpConnection), 调用 conn.commuincate(), commuincate() 中接受请求、应答。 由于 communicate() 是死循环，所以这个连接一直占有这个现场，当前连接 close 之后，当前线程继续从队列中去新的连接。因此，默认支持 5 个连接，新的连接会被阻塞在队列中，直到有新的空闲线程。
 


HttpServer
============================
self.requests = ThreadPool()
self.socket = socket.socket()

start()
-----------------------
* bind_addr 为 (host port) OR host:port
* socket 语义集合
	socket.getaddrinfo()
	socket.socket()
	socket.setsockopt()
	socket.bind()
	self.socket.listen()

* ThreadPool().start()

* tick()
	self.ready = True
	while self.ready
		socket.accept()
		conn = HttpConnection()
		ThreadPool().put(conn)
	
	
stop()
-----------------------
* self.socket = None
* threadPool.stop(timeout)



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
------------------------

* read_request_line()
	method, uri, protocol = rfile.readline()
	scheme, authority, path = parse_request_uri(uri)
	
	初始化 self.request_protocol  self.response_protocol self.path self.qs self.uri self.method self.scheme
	
* read_request_headers()
	read_headers()
	self.sconn.wfile.sendall()
	
	
respond()
------------------------

	ChunkedRFile OR KnownLengthRFile
	self.server.gateway().respond() = WSGIGateWay_10().respond()
	self.send_headers()
	self.conn.wfile.sendall("0\r\n\r\n")
	
read_request_line()





ThreadPool
========================

self.server = HttpServer

start()
------------------------
for i in range(min)
	self._threads.append(WorkerThread(self.server))
for worker in self._threads:
	worker.start()


WorkerThread
========================

run()
------------------------
	while True
		conn = self.server.request.get()
		conn.commuincate()

WSGIGateWay
========================

respond()
------------------------
respond = self.req.server.wsgi_app(self.env, self.start_response)
for rs in respond:
	self.write()

