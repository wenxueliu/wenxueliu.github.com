
we looked at pipes, FIFOs, message queues, semaphores, and shared memory the
classical methods of IPC provided by various UNIX systems. These mechanisms
allow processes running on the same computer to communicate with one another.
In this chapter, we look at the mechanisms that allow processes running on
different computers (connected to a common network) to communicate with one
another—network IPC.

###Socket Descriptors

A socket is an abstraction of a communication endpoint.Just as they would use
file descriptors to access files, applications use socket descriptors to access
sockets. Socket descriptors are implemented as file descriptors in the UNIX System.

        #include <sys/socket.h>
        int socket(int domain, int type, int protocol);
                Returns: file (socket) descriptor if OK, −1 on error

        int shutdown(int sockfd, int how);
                Returns: 0 if OK, −1 on error

A SOCK_STREAM socket provides a byte-stream service; applications are unaware of
message boundaries. This means that when we read data from a SOCK_STREAM socket,
it might not return the same number of bytes written by the sender. We will
eventually get everything sent to us, but it might take several function calls.

A SOCK_SEQPACKET socket is just like a SOCK_STREAM socket except that we get a
message-based service instead of a byte-stream service. This means that the
amount of data received from a SOCK_SEQPACKET socket is the same amount as was
written. The Stream Control Transmission Protocol (SCTP) provides a sequential
packet service in the Internet domain.

A SOCK_RAW socket provides a datagram interface directly to the underlying
network layer (which means IP in the Internet domain). Applications are
responsible for building their own protocol headers when using this interface,
because the transport protocols (TCP and UDP, for example) are bypassed.
Superuser privileges are requiredto create a raw socket to prevent malicious
applications from creating packets that might bypass established security
mechanisms.

####why is shutdown needed?

First, close will deallocate the network endpoint only when the last active
reference is closed. If we duplicate the socket (with dup, for example), the
socket won’t be deallocated until we close the last file descriptor referring to
it. The shutdown function allows us to deactivate a socket independently of the
number of active file descriptors referencing it.

Second, it is sometimes convenient to shut a socket down in one direction only.
For example, we can shut a socket down for writing if we want the process we are
communicating with to be able to tell when we are done transmitting data, while
still allowing us to use the socket to receive data sent to us by the process.

###Address

####Byte Order

        #include <arpa/inet.h>
        uint32_t htonl(uint32_t hostint32)
            Returns: 32-bit integer in network byte order
        uint16_t htons(uint16_t hostint16);
            Returns: 16-bit integer in network byte order
        uint32_t ntohl(uint32_t netint32);
            Returns: 32-bit integer in host byte order
        uint16_t ntohs(uint16_t netint16);
            Returns: 16-bit integer in host byte order

####Address Formats
        #include <arpa/inet.h>
        const char *inet_ntop(int domain, const void *restrict addr,
                            char *restrict str,socklen_t size);
                Returns: pointer to address string on success, NULL on error

        int inet_pton(int domain, const char *restrict str,
                    void *restrict addr);
                Returns: 1 on success, 0 if the format is invalid, or −1 on error

####Address Lookup

        #include <netdb.h>
        struct hostent *gethostent(void);
                Returns: pointer if OK, NULL on error
        void sethostent(int stayopen);
        void endhostent(void);


        #include <netdb.h>
        struct netent *getnetbyaddr(uint32_t net, int type);
        struct netent *getnetbyname(const char *name);
        struct netent *getnetent(void);
                All return: pointer if OK, NULL on error
        void setnetent(int stayopen);
        void endnetent(void);

        //map between protocol names and numbers with the following functions
        #include <netdb.h>
        struct protoent *getprotobyname(const char *name);
        struct protoent *getprotobynumber(int proto);
        struct protoent *getprotoent(void);
                All return: pointer if OK, NULL on error
        void setprotoent(int stayopen);
        void endprotoent(void);


        #include <netdb.h>
        //map between a service name and a port
        struct servent *getservbyname(const char *name, const char *proto);
        struct servent *getservbyport(int port, const char *proto);

        //scan the services database sequentially
        struct servent *getservent(void);
                All return: pointer if OK, NULL on error
        void setservent(int stayopen);
        void endservent(void);

        //an application to map from a host name and a service name to an address, and vice versa
        #include <sys/socket.h>
        #include <netdb.h>
        int getaddrinfo(const char *restrict host,
                const char *restrict service,
                const struct addrinfo *restrict hint,
                struct addrinfo **restrict res);
                )
                Returns: 0 if OK, nonzero error code on error
        void freeaddrinfo(struct addrinfo *ai);
        const char *gai_strerror(int error);
                Returns: a pointer to a string describing the error

        int getnameinfo(const struct sockaddr *restrict addr, socklen_t alen,
                char *restrict host, socklen_t hostlen,
                char *restrict service, socklen_t servlen, int flags
                )
                Returns: 0 if OK, nonzero on error

 ####Associate an address with a socket
        //associate an address with a socket
        int bind(int sockfd, const struct sockaddr *addr, socklen_t len);
                Returns: 0 if OK, −1 on error

* The address we specify must be valid for the machine on which the process is
running; we can’t specify an address belonging to some other machine.

* The port number in the address cannot be less than 1,024 unless the process
has the appropriate privilege (i.e., is the superuser).

* Usually, only one socket endpoint can be bound to a given address, although
some protocols allow duplicate bindings.

For the Internet domain, if we specify the special IP address INADDR_ANY , the
socket endpoint will be bound to all the system’s network interfaces. This means
that we can receive packets from any of the network interface cards installed in
the system. the system will choose an address and bind it to our socket for us
if we call connect or listen without first binding an address to the socket.

        //discover the address bound to a socket
        int getsockname(int sockfd, struct sockaddr *restrict addr,
                        socklen_t *restrict alenp);
                Returns: 0 if OK, −1 on error

        //find out the peer’s address
        int getpeername(int sockfd, struct sockaddr *restrict addr,
                        socklen_t *restrict alenp);
                Returns: 0 if OK, −1 on error

###Connection Establishment

        int connect(int sockfd, const struct sockaddr *addr, socklen_t len);
                Returns: 0 if OK, −1 on error

                portable applications need to close the socket if connect fails. If
                we want to retry, we have to open a new socket.

        int listen(int sockfd, int backlog);
                Returns: 0 if OK, −1 on error

        int accept(int sockfd, struct sockaddr *restrict addr,
                socklen_t *restrict len);
                Returns: file (socket) descriptor if OK, −1 on error

###Data Transfer

Using read and write with socket descriptors is significant, because it means
that we can pass socket descriptors to functions that were originally designed
to work with local files.


        #include <sys/socket.h>
        ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);

If send returns success, it doesn’t necessarily mean that the process at the
other end of the connection receives the data. All we are guaranteed is that
when send succeeds, the data has been delivered to the network drivers without
error.

        ssize_t sendto(int sockfd, const void *buf, size_t nbytes, int flags,
                        const struct sockaddr *destaddr, socklen_t destlen);
                    Returns: number of bytes sent if OK, −1 on error

        ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
                    Returns: number of bytes sent if OK, −1 on error

sendmsg with a msghdr structure to specify multiple buffers from which
to transmit data

        ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
                    Returns: length of message in bytes,0 if no messages are available and
                    peer has done an orderly shutdown, or −1 on error

With SOCK_STREAM sockets, we can receive less data than we requested. The
MSG_WAITALL flag inhibits this behavior, preventing recv from returning until
all the data we requested has been received. With SOCK_DGRAM and SOCK_SEQPACKET
sockets, the MSG_WAITALL flag provides no change in behavior, because these
message-based socket types already return an entire message in a single read.


        ssize_t recvfrom(int sockfd, void *restrict buf, size_t len, int flags,
                struct sockaddr *restrict addr, socklen_t *restrict addrlen);
                    Returns: length of message in bytes,0 if no messages are available and
                    peer has done an orderly shutdown, or −1 on error

recvfrom is typically used with connectionless sockets. Otherwise,
recvfrom behaves identically to recv.

        ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
                    Returns: length of message in bytes,0 if no messages are available and
                    peer has done an orderly shutdown, or −1 on error


###Socket Options

        #include <sys/socket.h>
        int setsockopt(int sockfd, int level, int option, const void *val, socklen_t len);
        int getsockopt(int sockfd, int level, int option, void *restrict val,socklen_t *restrict lenp);
                Returns: 0 if OK, −1 on error

###Out-of-Band Data

Out-of-band data is an optional feature supported by some communication
protocols, allowing higher-priority delivery of data than normal. Out-of-band
data is sent ahead of any data that is already queued for transmission. TCP
supports out-of-band data, but UDP doesn’t. The socket interface to out-of-band
data is heavily influenced by TCP’s implementation of out-of-band data.

TCP refers to out-of-band data as ‘‘urgent’’ data. TCP supports only a single
byte of urgent data, but allows urgent data to be delivered out of band from the
normal data delivery mechanisms. To generate urgent data, we specify the MSG_OOB
flag to any of the three send functions. If we send more than one byte with the
MSG_OOB flag, the last byte will be treated as the urgent-data byte.

        #include <sys/socket.h>
        identify when we have reached the urgent mark
        int sockatmark(int sockfd);
                Returns: 1 if at mark, 0 if not at mark, −1 on error


###Nonblocking and Asynchronous I/O

Normally, the recv functions will block when no data is immediately available.
Similarly, the send functions will block when there is not enough room in the
socket’s output queue to send the message. This behavior changes when the socket
is in nonblocking mode.

In this case, these functions will fail instead of blocking, setting errno to
either EWOULDBLOCK or EAGAIN. When this happens, we can use either poll or
select to determine when we can receive or transmit data.


* The Single UNIX Specification
* classic socket-based asynchronous I/O

With socket-based asynchronous I/O, we can arrange to be sent the SIGIO signal
when we can read data from a socket or when space becomes available in a
socket’s write queue. Enabling asynchronous I/O is a two-step process.

* Establish socket ownership so signals can be delivered to the proper processes.
* Inform the socket that we want it to signal us when I/O operations won’t block.

We can accomplish the first step in three ways.
* Use the F_SETOWN command with fcntl.
* Use the FIOSETOWN command with ioctl.
* Use the SIOCSPGRP command with ioctl.

To accomplish the second step, we have two choices.
* Use the F_SETFL command with fcntl and enable the O_ASYNC file flag.
* Use the FIOASYNC command with ioctl.

###Conclusion

In this chapter, we looked at the IPC mechanisms that allow processes to
communicate with other processes on different machines as well as within the
same machine.  We discussed how socket endpoints are named and how we can
discover the addresses to use when contacting servers.

We presented examples of clients and servers that use connectionless (i.e.,
datagram-based) sockets and connection-oriented sockets. We briefly discussed
asynchronous and nonblocking socket I/O and the interfaces used to manage socket
options.
