
###UNIX Domain Sockets

UNIX domain sockets are used to communicate with processes running on the same
machine. Although Internet domain sockets can be used for this same purpose,
UNIX domain sockets are more efficient. UNIX domain sockets only copy data; they
have no protocol processing to perform, no network headers to add or remove, no
checksums to calculate, no sequence numbers to generate, and no acknowledgements
to send.

UNIX domain sockets provide both stream and datagram interfaces. The UNIX
domain datagram service is reliable, however. Messages are neither lost nor
delivered out of order. UNIX domain sockets are like a cross between sockets and
pipes.  You can use the network-oriented socket interfaces with them, or you can
use the socketpair function to create a pair of unnamed, connected, UNIX domain
sockets.

        #include <sys/socket.h>
        int socketpair(int domain, int type, int protocol, int sockfd[2]);
                Returns: 0 if OK, −1 on error


<img src="{{ IMAGE_PATH }}/APUE/socketpair.png" alt="socket_pairs" title="socket_pairs" />

A pair of connected UNIX domain sockets acts like a full-duplex pipe: both ends
are open for reading and writing . We’ll refer to these as ‘‘fd-pipes’’ to
distinguish them from normal, half-duplex pipes.

###Naming UNIX Domain Sockets
Although the socketpair function creates sockets that are connected to each
other, the individual sockets don’t have names. This means that they can’t be
addressed by unrelated processes.

###Unique Connections
A server can arrange for unique UNIX domain connections to clients using the
standard bind, listen, and accept functions. Clients use connect to contact the
server; after the connect request is accepted by the server, a unique connection
exists between the client and the server.

###Passing File Descriptors
Passing an open file descriptor between processes is a powerful technique. It
can lead to different ways of designing client–server applications. It allows
one process (typically a server) to do everything that is required to open a
file

What normally happens when a descriptor is passed from one process to another is
that the sending process, after passing the descriptor, then closes the
descriptor. Closing the descriptor by the sender doesn’t really close the file
or device, since the descriptor is still considered open by the receiving
process (even if the receiver hasn’t specifically received the descriptor yet).

    #include <sys/socket.h>
    unsigned char *CMSG_DATA(struct cmsghdr *cp);
        Returns: pointer to data associated with cmsghdr structure

    struct cmsghdr *CMSG_FIRSTHDR(struct msghdr *mp);
        Returns: pointer to first cmsghdr structure associated
                with the msghdr structure, or NULL if none exists

    struct cmsghdr *CMSG_NXTHDR(struct msghdr *mp, struct cmsghdr *cp);
        Returns: pointer to next cmsghdr structure associated with
                the msghdr structure given the current cmsghdr
                structure, or NULL if we’re at the last one

    unsigned int CMSG_LEN(unsigned int nbytes);
        Returns: size to allocate for data object nbytes large


