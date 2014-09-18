---
layout: post
category : tools
tagline: "linux 调试 core dump"
tags : [ clang, cpp, linux, debug ]
---
{% include JB/setup %}

###什么是Core

Sam之前一直以为Core Dump中Core是 Linux Kernel的意思. 今天才发现在这里，Core是另一种意思：

在使用半导体作为内存的材料前，人类是利用线圈当作内存的材料（发明者为王安），线圈就叫作 core ，用线圈做的内存就叫作 core memory。如今 ，半导体工业澎勃发展，已经没有人用 core memory 了，不过，在许多情况下， 人们还是把记忆体叫作 core 。

###什么是Core Dump

我们在开发（或使用）一个程序时，最怕的就是程序莫明其妙地当掉。虽然系统没事，但我们下次仍可能遇到相同的问题。于是这时操作系统就会把程序当掉时的内存内容 dump 出来（现在通常是写在一个叫 core 的 file 里面），让我们以 debugger 做为参考。这个动作就叫作 core dump。

###Core Dump时会生成何种文件

Core Dump时，会生成诸如 core.进程号 的文件。


###为何有时程序 Down 了，却没生成 Core 文件

Linux下，有一些设置，标明了resources available to the shell and to processes。 可以使用#ulimit -a 来看这些设置。 (ulimit是bash built-in Command)

	-a All current limits are reported

	-c The maximum size of core files created

	-d The maximum size of a process data segment

	-e The maximum scheduling priority ("nice")

	-f The maximum size of files written by the shell and its children

	-i The maximum number of pending signals

	-l The maximum size that m ay be locked into memory

	-m The maximum resident set size (has no effect on Linux)

	-n The maximum number of open file descriptors (most systems do not allow this value to be set)

	-p The pipe size in 512-byte blocks (this may not be set)

	-q The maximum number of bytes in POSIX message queues

	-r The maximum real-time scheduling priority

	-s The maximum stack size

	-t The maximum amount of cpu time in seconds

	-u The maximum number of processes available to a single user

	-v The maximum amount of virtual memory available to the shell

	-x The maximum number of file locks


从这里可以看出，如果 -c 是显示：core file size (blocks, -c) 如果这个值为0，则无法生成core文件。所以可以使用：

1）使用 ulimit -c 命令可查看 core 文件的生成开关。若结果为0，则表示关闭了此功能，不会生成 core 文件。

2）使用 ulimit -c filesize 命令，可以限制 core 文件的大小（filesize的单位为kbyte）。若 ulimit -c unlimited，则表示core文件的大小不受限制。如果生成的信息超过此大小，将会被裁剪，最终生成一个不完整的 core 文件。在调试此 core 文件的时候，gdb会提示错误。

如果程序出错时生成 Core 文件，则会显示 Segmentation fault (core dumped) 。

注：编译程序的时候，必须加 -g 为可调试

### Core Dump的核心转储文件目录和命名规则

若系统生成的core文件不带其它任何扩展名称，则全部命名为core。新的core文件生成将覆盖原来的core文件 。


/proc/sys/kernel/core_uses_pid 可以控制 core 文件的文件名中是否添加 pid 作为扩展。文件内容为1，表示添加 pid 作为扩展名，生成的 core 文件格式为core.xxxx；为 0 则表示生成的core文件同一命名为core。

可通过以下命令修改此文件：

	echo"1" >/proc/sys/kernel/core_uses_pid


/proc/sys/kernel/core_pattern可以控制 core 文件保存位置和文件名格式。

可通过以下命令修改此文件：

	echo"/corefile/core-%e-%p-%t" >core_pattern

可以将core文件统一生成到/corefile目录下，产生的文件名为core-命令名-pid-时间戳

以下是参数列表:

	%p - insert pid into filename 添加pid
	%u - insert current uid into filename 添加当前uid
	%g - insert current gid into filename 添加当前gid
	%s - insert signal that caused the coredump into the filename 添加导致产生core的信号
	%t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
	%h - insert hostname where the coredump happened into filename 添加主机名
	%e - insert coredumping executable name into filename 添加命令名


###如何使用Core文件

在Linux下，使用：

	#gdb -c core.pid program_name

就可以进入gdb模式。

输入where，就可以指出是在哪一行被Down掉，哪个function内，由谁调用等等。

	(gdb) where

或者

	(gdb) bt


此外 gdb [exec file] [core file] 也可以。


注意：待调试的可执行文件，在编译的时候需要加-g，core文件才能正常显示出错信息！

###如何让一个正常的程序down

	#kill -s SIGSEGV pid

###察看Core文件输出在何处

存放 Coredump 的目录即进程的当前目录，一般就是当初发出命令启动该进程时所在的目录。但如果是通过脚本启动，则脚本可能会修改当前目录，这时进程真正的当前目录就会与当初执行脚本所在目录不同。这时可以查看“/proc/<进程pid>/cwd”符号链接的目标来确定进程真正的当前目录地址。通过系统服务启动的进程也可通过这一方法查看。

---------------------------------------------------------------------------------------------

###What is a core file?

A core file is an image of a process that has crashed It contains all process information pertinent to debugging: contents of hardware registers, process status, and process data. Gdb will allow you use this file to determine where your program crashed.

###How to use gdb to examine core files

First, the program must be compiled with debugging information otherwise the information that gdb can display will be fairly cryptic. 

Second, the program must have crashed and left a core file. It should tell you if it has left a core file with the message "core dumped".

	ulimit -c unlimited

Third, use the command line to start gdb to look at the core file is:

    gdb program core

where "program" is the name of the program you're working on. Gdb will then load the program's debugging information and examine the core file to determine the cause of the crash.


When it starts up, you can use bt (for backtrace) to get a stack trace from the time of the crash. In the backtrace, each function invocation is given a number.


You can use frame number (replacing number with the corresponding number in the stack trace) to select a particular stack frame. You can then use list to see code around that function, and info locals to see the local variables. You can also use print name_of_variable (replacing "name_of_variable" with a variable name) to see its value.


Then you can get the information using "bt" command. For detailed backtrace use
"bt full".To print the variables use "print varibale-name" or " p varibale-name"
.To get any help on gdb use "help" option or use "apropos search-topic".Use
"info locals" to see the local variables. Use "frame frame-number" to go to
desired frame number. Use "up n" and "down n" commands to select frame n frames
up and select frame n frames down respectively.To stop gdb use "quit" or "q".
