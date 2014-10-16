
###Signal Generate

* The terminal-generated signals occur when users press certain terminal keys.

* Hardware exceptions generate signals, detected by the hardware, and the
kernel is notified.such as divide by 0, invalid memory reference, and the like.

* The kill(2) function allows a process to send any signal to another process or
process group.

* The kill(1) command allows us to send signals to other processes.

* Software conditions can generate signals when a process should be notified of
various events.

Signals are classic examples of asynchronous events. They occur at what appear
to ss can't simply test a variable (such as errno) to see whether a signal has
occurred; instead, the process has to tell the kernel "if and when this signal
occurs, do the following."random times to the process.

###Action Associated With A Signal

* Ignore the signal. SIGKILL and SIGSTOP can never be ignored. hardware
exception is undefined.
* Catch the signal. SIGKILL and SIGSTOP can’t be caught.
* Let the default action apply. default action for most signals is to terminate
the process.


###Core File
The core file will not be generated
* the process was set-user-ID and the current user is not the owner of the program file
* the process was set-group-ID and the current user is not the group owner of the file
* the user does not have permission to write in the current working directory
* the file already exists and the user does not have permission to write to it
* the file is too big

SIGABRT     abort
SIGALRM     alarm  setitimer
SIGCHLD     child process terminates or stops, send to parent process
SIGCONT     job-control signal
SIGBUS      hardware error
SIGEMT      hardware error
SIGFPE      divide by 0, floating-point overflow
SIGIOT      implementation-defined hardware fault.
SIGILL      an illegal hardware instruction
SIGINT      Control-C
SIGIO       an asynchronous shiaf?/* event
SIGIOT      Hardware fault
SIGPIPE     write to a pipeline but the reader has terminated
SIGPOLL     It can be generated when a specific event occurs on a pollable device.
SIGPROF     setitimer
SIGQUIT     Control-backslash
SIGSTOP     job-control signal stops a process,cannot be caught or ignored
SIGSYS      an invalid system call.
SIGTERM     kill
SIGTRAP     hardware error
SIGTSTP     Control-Z
SIGTTIN     generate by terminal driver when a process in  background process group
            tries to read from its control terminal.
SIGTTOU     generate by terminal driver when a process in  background process group
            tries to write from its control terminal.
SIGURG      otifies the process that an urgent condition has occurred. It is optionally
            generated when out-of-band data is received on a network connection.
SIGUSR1     user-defined signal
SIGUSR2     user-defined signal
SIGVTALRM   a virtual interval timer set by the setitimer(2) function expires
SIGWINCH    ioctl set-window-size
SIGXFSZ     process exceeds its soft file size limit

###unreliable signal

* signal get lost :a signal could occur and the process would never know about it
* signal of control is weak : a process could catch the signal or ignore it.
* a signal was reset to its default each time the signal occurred(now it catch ever time)
* unable to turn a signal off when it didn’t want the signal to occur.

###Reentrant Functions

The Single UNIX Specification specifies the functions that are guaranteed to be
safe to call from within a signal handler. These functions are reentrant and are
called async-signal safe by the Single UNIX Specification.

Besides being reentrant, they block any signals during operation if delivery of
a signal might cause inconsistencies.

Cannot Reentrant Function

* use static data structure
* call malloc or free
* part of the standard I/O library

Note

    even though we call printf from signal handlers in some of our examples, it is
    not guaranteed to produce the expected results,signal handler can interrupt a
    call to printf from our main program




as a general rule, when calling the reentrant functions from a signal handler,
we should save and restore errno.

###SIGCLD Function

Two signals that continually generate confusion are SIGCLD and SIGCHLD. The name
SIGCLD (without the H) is from System V, and this signal has different semantics
from the BSD signal, named SIGCHLD. The POSIX.1 signal is also named SIGCHLD.

