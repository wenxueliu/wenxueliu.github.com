
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
