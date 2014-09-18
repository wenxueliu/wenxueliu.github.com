
###main Function

###Process Termination

* Return from main
* Calling exit
* Calling _exit or _Exit
* Return of the last thread from its start routine(Section 11.5)
* Calling pthread_exit (Section 11.5) from the last thread
* Calling abort (Section 10.17)
* Receipt of a signal (Section 10.2)
* Response of the last thread to a cancellation request (Sections 11.5 and 12.7)

####Exit Function

    #include <stdlib.h>
    void exit(int status);
    void _Exit(int status);
    #include <unistd.h>
    void _exit(int status);

exit status of the process is undefined if

* any of three functions is called without an exit status
* main does a return without a return value
* the main function is not declared to return an integer

Returning an integer value from the main function is equivalent to calling exit
with the same value. Thus

    exit(0);

is the same as

    return(0);

from the main function.

For any of the preceding cases, we want the terminating process to be able to
notify its parent how it terminated. For the three exit functions (exit, _exit,
        and _Exit), this is done by passing an exit status as the argument to
the function. In the case of an abnormal termination, however, the kernel—not
the process — generates a termination status to indicate the reason for the
abnormal termination. In any case, the parent of the process can obtain the
termination status from either the wait or the waitpid function 

In UNIX System terminology, a process that has terminated, but whose parent has
not yet waited for it, is called a zombie.

####atexit function

With ISO C, a process can register at least 32 functions that are automatically
called by exit. These are called exit handlers and are registered by calling the
atexit function.

    #include <stdlib.h>
    int atexit(void (\*func)(void));))

####C program is started and the various ways it can terminate

<img src="{{ IMAGE_PATH }}/APUE/process_terminated.png" alt="process exit" 
title="process terminated" />

A process can also be involuntarily terminated by a signal which do not show
above


###Arguments List 
    for (i = 0; argv[i] != NULL; i++)


###Environment Variables

    extern char **environ;

<img src="{{ IMAGE_PATH }}/APUE/environment_list.png" alt="Environment vairable" 
title="environment_list" />


    #include <stdlib.h>
    char *getenv(const char *name);
        Returns: pointer to value associated with name, NULL if not found

    #include <stdlib.h>
    int putenv(char *str);
        Returns: 0 if OK, nonzero on error
    int setenv(const char *name, const char *value, int rewrite);
    int unsetenv(const char *name);
        Both return: 0 if OK, −1 on error

we can affect the environment of only the current process and any child
processes that we invoke. We cannot affect the environment of the parent
process, which is often a shell.

* Note
    that ISO C doesn’t define any environment variables.

###Memory Layout of a C Program

* Text segment, consisting of the machine instructions that the CPU
executes.(read-only)
* Initialized data segment, usually called simply the data segment
* Uninitialized data segment, often called the ‘‘bss’’ segment
* Heap, where dynamic memory allocation usually takes place.
* Stack, where automatic variables are stored, along with information that is saved
each time a function is called.

<img src="{{ IMAGE_PATH }}/APUE/memory_arrangement.png" alt="Memory layout"
title="memory layout" />

the contents of the uninitialized data segment are not stored in the program
file on disk, because the kernel sets the contents to 0 before the program
starts running. The only portions of the program that need to be saved in the
program file are the text segment and the initialized data.


###Share Library

instead maintaining a single copy of the library routine somewhere in memory
that all processes reference.

Share library reduces the size of each executable file but may add some runtime
overhead, either when the program is first executed or the first time each
shared library function is called.

Another advantage of shared libraries is that library functions can be replaced
with new versions without having to relink edit every program that uses the
library (assuming that the number and type of arguments haven’t changed).


###Memory Allocation

    #include <stdlib.h>
    void *malloc(size_t size);
    void *calloc(size_t nobj, size_t size);
    void *realloc(void *ptr, size_t newsize);
        All three return: non-null pointer if OK, NULL on error
    void free(void *ptr);

The pointer returned by the three allocation functions is guaranteed to be
suitably aligned so that it can be used for any data object.

if we #include <stdlib.h> (to obtain the function prototypes), we do not
explicitly have to cast the pointer returned by these functions when we assign
it to a pointer of a different type.

The function free causes the space pointed to by ptr to be deallocated. but the
freed space is not usually returned to the kernel;instead, This freed space is
usually put into a pool of available memory and can be allocated in a later call
to one of the three alloc functions.


####Alternate Memory Allocators
* alloca Function
* TCMalloc
* quick-fit
* jemalloc

###setjmp and longjmp Functions

In C, we can’t goto a label that’s in another function. Instead, we must use the
setjmp and longjmp functions to perform this type of branching.

    #include <setjmp.h>
    int setjmp(jmp_buf env);
        Returns: 0 if called directly, nonzero if returning from a call to longjmp
    void longjmp(jmp_buf env, int val);

use the volatile attribute if you’re writing portable code that uses nonlocal
jumps.

###Potential Problem with Automatic Variables

    #include <stdio.h>
    FILE * open_data(void)
    {
        FILE *fp;
        char databuf[BUFSIZ];
        /* setvbuf makes this the stdio buffer */
        if ((fp = fopen("datafile", "r")) == NULL)
            return(NULL);

        if (setvbuf(fp, databuf, _IOLBF, BUFSIZ) != 0)
            return(NULL);
        return(fp);
        /* error */
    }

The problem is that when open_data returns, the space it used on the stack will
be used by the stack frame for the next function that is called. But the
standard I/O library will still be using that portion of memory for its stream
buffer. Chaos is sure to result.  To correct this problem, the array databuf
needs to be allocated from global memory, either statically (static or extern)
or dynamically (one of the alloc functions).

###getrlimit and setrlimit Functions

    #include <sys/resource.h>
    int getrlimit(int resource, struct rlimit *rlptr);
    int setrlimit(int resource, const struct rlimit *rlptr);
        Both return: 0 if OK, −1 on error


###Process Identifiers

Because the process ID is the only well-known identifier of a process that is
always unique, it is often used as a piece of other identifiers, to guarantee
uniqueness. For example, applications sometimes include the process ID as part
of a filename in an attempt to generate unique filenames.

* Process ID 0

it is usually the scheduler process and is often known as the swapper. No
program on disk corresponds to this process, which is part of the kernel and is
known as a system process.

* Process ID 1

it is usually the init process and is invoked by the kernel at the end of the
bootstrap procedure.

The program file for this process was /etc/init in older versions of the UNIX
System and is /sbin/init in newer versions.

init usually reads the system-dependent initialization files — the /etc/rc*
files or /etc/inittab and the files in /etc/init.d—and brings the system to a
certain state, such as multiuser.

The init process never dies. It is a normal user process, not a system process
within the kernel, like the swapper, although it does run with superuser
privileges.

This process is responsible for bringing up a UNIX system after the
kernel has been bootstrapped.


    #include <unistd.h>
    pid_t getpid(void);
        Returns: process ID of calling process
    pid_t getppid(void);
        Returns: parent process ID of calling process
    uid_t getuid(void);
        Returns: real user ID of calling process
    uid_t geteuid(void);
        Returns: effective user ID of calling process
    gid_t getgid(void);
        Returns: real group ID of calling process
    gid_t getegid(void);
        Returns: effective group ID of calling process

###fork Function

    #include <unistd.h>
    pid_t fork(void);
        Returns: 0 in child, process ID of child in parent, −1 on error

the child gets a copy of the parent’s data space, heap, and stack, buffer

The parent and the child do share the text segment

Modern implementations don’t perform a complete copy of the parent’s data,
stack, and heap, since a fork is often followed by an exec.

a fork followed by an exec—into a single operation called a spawn

####copy-on-write (COW)

These regions are shared by the parent and the child and have their protection
changed by the kernel to read-only. If either process tries to modify these
regions, the kernel then makes a copy of that piece of memory only, typically a
"page" in a virtual memory system.

###File Sharing
one characteristic of fork is that all file descriptors that are open in the
parent are duplicated in the child.

It is important that the parent and the child share the same file offset.

<img src="{{ IMAGE_PATH }}/APUE/file_sharing.png" alt="file share"
title="file share" />

####the difference of child and parent process

* The child’s tms_utime, tms_stime, tms_cutime, and tms_cstime values
are set to 0 (these times are discussed in Section 8.17).
* File locks set by the parent are not inherited by the child.
* Pending alarms are cleared for the child.

####reasons for fork to fail

* too many processes are already in the system, which usually means that
something else is wrong
* the total number of processes for this real user ID exceeds the system’s
limit.

###vfork Function

The function vfork has the same calling sequence and same return values as fork,
but the semantics of the two functions differ.

####difference between vfork and fork

The vfork function creates the new process, just like fork, without copying the
address space of the parent into the child, as the child won’t reference that
address space; the child simply calls exec (or exit) right after the
vfork.Instead, the child runs in the address space of the parent until it calls
either exec or exit.

vfork guarantees that the child runs first, until the child calls exec or exit.
When the child calls either of these functions, the parent resumes.


###wait and waitpid function

When a process terminates, either normally or abnormally, the kernel notifies
the parent by sending the SIGCHLD signal to the parent. Because the termination
of a child is an asynchronous event—it can happen at any time while the parent
is running — this signal is the asynchronous notification from the kernel to the
parent. The parent can choose to ignore this signal, or it can provide a
function that is called when the signal occurs: a signal handler. The default
action for this signal is to be ignored.

    #include <sys/wait.h>
    pid_t wait(int *statloc);
    pid_t waitpid(pid_t pid, int *statloc, int options);
        Both return: process ID if OK, 0 (see later), or −1 on error

####the difference between wit and waitpid

1. The waitpid function lets us wait for one particular process, whereas the
wait function returns the status of any terminated child. We’ll return to this
feature when we discuss the popen function.  

2. The waitpid function provides a nonblocking version of wait. There are times
when we want to fetch a child’s status, but we don’t want to block.  

3. The waitpid function provides support for job control with the WUNTRACED and
WCONTINUED options.


###waitid Function

    #include <sys/wait.h>
    int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
        Returns: 0 if OK, −1 on error

###wait3 and wait4 Functions

    #include <sys/resource.h>
    #include <sys/time.h>
    #include <sys/wait.h>
    #include <sys/types.h>
    pid_t wait3(int *statloc, int options, struct rusage *rusage);
    pid_t wait4(pid_t pid, int *statloc, int options, struct rusage *rusage);
        Both return: process ID if OK, 0, or −1 on error


###Race Conditions

* some form of signaling
* interprocess communication (IPC)

###exec Function

    #include <unistd.h>
    int execl(const char *pathname, const char *arg0, ... /* (char *)0 */ );
    int execv(const char *pathname, char *const argv[]);
    int execle(const char *pathname, const char *arg0, ...
            /* (char *)0, char *const envp[] */ );
    int execve(const char *pathname, char *const argv[], char *const envp[]);
    int execlp(const char *filename, const char *arg0, ... /* (char *)0 */ );
    int execvp(const char *filename, char *const argv[]);
    int fexecve(int fd, char *const argv[], char *const envp[]);
            All seven return: −1 on error, no return on success


<img src="{{ IMAGE_PATH }}/APUE/exec_function.png" alt="exec Function"
title="exec function" />

###Changing User IDs and Group IDs

if you want show privilege data to user which is unprivileged, it is very useful

    #include <unistd.h>
    int setuid(uid_t uid);
    int setgid(gid_t gid);
        Both return: 0 if OK, −1 on error

<img src="{{ IMAGE_PATH }}/APUE/user_ids.png" alt="user ids" title="use ids" />

    #include <unistd.h>
    int setreuid(uid_t ruid, uid_t euid);
    int setregid(gid_t rgid, gid_t egid);
        Both return: 0 if OK, −1 on error

    #include <unistd.h>
    int seteuid(uid_t uid);
    int setegid(gid_t gid);
        Both return: 0 if OK, −1 on error

<img src="{{ IMAGE_PATH }}/APUE/set_ids.png" alt="set uid", title="use ids" />


###Interpreter Files

read again

###system Function

    #include <stdlib.h>
    int system(const char *cmdstring);

ISO C defines the system function, but its operation is strongly system
dependent.

system is implemented by calling fork, exec, and waitpid

If it is running with special permissions—either set-user-ID or set-group-ID —
and wants to spawn another process, a process should use fork and exec directly,
being certain to change back to normal permissions after the fork, before
calling exec. The system function should never be used from a set-user-ID or a
set-group-ID program.

###User Identification

    #include <unistd.h>
    char *getlogin(void);
        Returns: pointer to string giving login name if OK, NULL on error
