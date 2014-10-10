
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

SIGABRT  abort
SIGALRM  alarm  setitimer
SIGCHLD  child process terminates or stops, send to parent process
SIGCONT  job-control signal
SIGBUS   hardware error
SIGEMT   hardware error
SIGFPE   divide by 0, floating-point overflow
SIGIOT   implementation-defined hardware fault.
SIGILL   an illegal hardware instruction
SIGINT   Control-C
SIGIO    an asynchronous shiaf?/* event
SIGIOT   Hardware fault
SIGPIPE  write to a pipeline but the reader has terminated
SIGPOLL  It can be generated when a specific event occurs on a pollable device.
SIGPROF  setitimer
SIGQUIT  Control-backslash
SIGSTOP  job-control signal stops a process,cannot be caught or ignored
SIGSYS   an invalid system call.
SIGTERM  kill





