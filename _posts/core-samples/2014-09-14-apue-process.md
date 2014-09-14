
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
