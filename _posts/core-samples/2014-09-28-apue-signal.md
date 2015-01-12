
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


###Reliable-Signal Terminology and Semantics

A process has the option of blocking the delivery of a signal. If a signal that
is blocked is generated for a process, and if the action for that signal is
either the default action or to catch the signal, then the signal remains
pending for the process until the process either (a) unblocks the signal or (b)
changes the action to ignore the signal. The system determines what to do with a
blocked signal when the signal is delivered, not when it’s generated. This
allows the process to change the action for the signal before it’s delivered.
The sigpending function (Section 10.13) can be called by a process to determine
which signals are blocked and pending.

Each process has a signal mask that defines the set of signals currently blocked
from delivery to that process. We can think of this mask as having one bit for
each possible signal. If the bit is on for a given signal, that signal is
currently blocked. A process can examine and change its current signal mask by
calling sigprocmask.

###kill raise function

        #include <signal.h>
        int kill(pid_t pid, int signo);
        int raise(int signo);
                Both return: 0 if OK, −1 on error

If the signo argument is 0, then the normal error checking is performed by kill,
but no signal is sent. This technique is often used to determine if a specific
process still exists. that the test for process existence is not atomic. By the
time that kill returns the answer to the caller, the process in question might
have exited, so the answer is of limited value.

####alarm

        #include <unistd.h>
        unsigned int alarm(unsigned int seconds);
                Returns: 0 or number of seconds until previously set alarm

* case 1

If, when we call alarm, a previously registered alarm clock for the process has
not yet expired, the number of seconds left for that alarm clock is returned as
the value of this function.  That previously registered alarm clock is replaced
by the new value.

* case 2

If a previously registered alarm clock for the process has not yet expired and
if the seconds value is 0, the previous alarm clock is canceled. The number of
seconds left for that previous alarm clock is still returned as the value of the
function.

* case 3

If we intend to catch SIGALRM, we need to be careful to install its signal
handler before calling alarm. If we call alarm first and are sent SIGALRM before
we can install the signal handler, our process will terminate.

    #include <stdio.h>
    #include <string.h>
    #include <unistd.h>
    #include <signal.h>
    #include <sys/types.h>
    #include <pwd.h>

    static void my_alram(int signo)
    {
        struct  passwd      *rootptr;
        printf("signal hander\n");
        alarm(2);
    }

    int main(int argc, char **argv)
    {
        struct passwd       *ptr;
        /*uncomment for test case3
        alarm(1);
        sleep(2);
        */

        signal(SIGALRM,my_alram);
        alarm(6);//test case 1
        /*
        uncomment for testing case2
        alarm(0);
        */
        for(;;)
        {
            printf("wait signal\n");
            sleep(1);
        }
    }

####pause

suspends the calling process until a signal is caught.

        #include <unistd.h>
        int pause(void);
                Returns: −1 with errno set to EINTR


###Signal Set

        #include <signal.h>
        int sigemptyset(sigset_t *set);
        int sigfillset(sigset_t *set);
        int sigaddset(sigset_t *set, int signo);
        int sigdelset(sigset_t *set, int signo);
            All four return: 0 if OK, −1 on errorint sigismember()

        int sigismember(const sigset_t *set, int signo);
            Returns: 1 if true, 0 if false, −1 on error

All applications have to call either sigemptyset or sigfillset once for each
signal set, before using the signal set, because we cannot assume that the C
initialization for external and static variables (0) corresponds to the
implementation of signal sets on a given system.


        int sigprocmask(int how, const sigset_t *restrict set, sigset_t *restrict oset,)
            Returns: 0 if OK, −1 on error

The sigprocmask function is defined only for single-threaded processes.

        int sigpending(sigset_t *set);
            Returns: 0 if OK, −1 on error

signals are not queued on this system.

        int sigaction(int signo, const struct sigaction *restrict act,
                struct sigaction *restrict oact);
            Returns: 0 if OK, −1 on error
