###注
   更新与2014-07-20
   
###工具介绍

####编辑器及编辑辅助工具
* vim + vundle : 以“神圣”二字冠之，必知必熟，效率倍升，至今无可替代。
* emacs : 你懂得
* [sloccount](http://www.dwheeler.com/sloccount/)：源代码行数统计工具。
* CLOC：[代码统计工具](http://sourceforge.net/projects/cloc/files/)

####源码检查：
 
* [splint](www.splint.org/) : checking C programs for security vulnerabilities and coding mistakes
* [flawfinder](http://www.dwheeler.com/flawfinder/) : examines source code and reports possible security weaknesses (“flaws”) sorted by risk level
* [ITS4](www.cigital.com/its4) : a simple tool that statically scans C and C++ source code for potential security vulnerabilities. 

####项目构建工具

* make(Makefile)：常用的项目构建工具，用一个 Makefile 就可以从整个项目的代码中构建各个目标。
* autotool(包括Autoconf, Automake 和 Libtool)：方便在你的项目中生成标准的 Makefile
* cmake : 构建大型程序的比 make 更友好的工具

####编译器及二进制文件处理工具

gcc : GNU项目所开发的全能编译器

[binutils](http://www.gnu.org/software/binutils/) :  The GNU Binutils are a collection of binary tools. The main ones are:

    ld - the GNU linker.
    as - the GNU assembler.

But they also include:

    addr2line - Converts addresses into filenames and line numbers.
    ar - A utility for creating, modifying and extracting from archives.
    c++filt - Filter to demangle encoded C++ symbols.
    dlltool - Creates files for building and using DLLs.
    gold - A new, faster, ELF only linker, still in beta test.
    gprof - Displays profiling information.
    nlmconv - Converts object code into an NLM.
    nm - Lists symbols from object files.
    objcopy - Copys and translates object files.
    objdump - Displays information from object files.
    ranlib - Generates an index to the contents of an archive.
    readelf - Displays information from any ELF format object file.
    size - Lists the section sizes of an object or archive file.
    strings - Lists printable strings from files.
    strip - Discards symbols.
    windmc - A Windows compatible message compiler.
    windres - A compiler for Windows resource files.

(ld-linux)[https://www.linux.com/learn/docs/man/3489-ld-linux8] : dynamic linker/loader

####调试工具

* gdb：GNU项目所开发的调试器，功能强大，是程序员的好助手
除了只用命令行gdb外，还可以用gdb的gui,有

* [cgdb](http://cgdb.github.io/) 缺点：界面简陋，自动化程度低，只是把terminal分为两部分，上面部分显示源码，下面打命令。由于没有显示反汇编的窗体，不适合要求使用到 stepi命令的场合。优点：运行快，锻炼手指头. 最大的优点是，它有完美的代码着色功能。其他几款调试器中都没有。

* [ddd](http://www.gnu.org/software/ddd/): 

缺点：与kdbg相比，界面凌乱。
优点：代码显示效果比kdbg好，c和反汇编代码分开在两个窗口。 可以随时暂停程序的运行。data windows 这个功能非常强大灵活。

* [kdbg](http://www.kdbg.org/):

缺点：功能比ddd弱。字体太小，c和反汇编代码交错显示，反汇编代码折叠隐藏在C代码之间，要显示反汇编代码要手动展开，不可忍受。太过界面化，居然找不到是在哪里手动打gdb命令。致命缺点是，内核跑起来后，如果没有断点拦截，就没法把内核的运行暂停下来，kdbg成了没事姥，源码窗口的显示不更新。 另一个致命缺点是，如果没有源码只有二进制文件，虽然可以下断点，但无法显示反汇编代码，没意义。据说kdbg是用来调试kde程序的，实际上也能调试内 核。

优点：窗口可以整合到一块，稳定。有变化的寄存器会显示红色。提示 kdbg -r localhost:1234 ./vmlinux

* [Source insight](http://www.sourceinsight.com/prodinfo.html)

其实，gdb自带了一个基于 curses 的 gui。启动方式是gdbtui xxx; 或者在gdb启动之后用命令layout启动gui。很好用，可以至多同时显示三个分窗口。要是代码有着色功能就好了。

针对内核调试的总结：

* kdbg不适合调试内核

* 如果想复习gdb强大的命令，选cgdb或纯gdb。

* 如果想学习汇编，insight是不二选择。

* 如果倾向于把调试器当作浏览器使用，作为 source insight 等工具的辅助工具，在内核运行中拦截函数，分析函数的调用关系，不需要反汇编的话，则 cgdb 是不错的选择 .(source insight等源码分析工具有个共同的缺点，因为体系和内核配置不同，一个函数有很多的定义，借助调试器可以在内核运行的时候找出实际调用的那个)

* insight 和 ddd 很接近，各有千秋。但如果侧重于追溯数据结构体间的联系，ddd 更好一点，因为它有 data window，它的强项是数据和数据结构关系分析并用图像方式显示出来（DDD = Data Display Debugger)。如果侧重于分析汇编指令是怎么在cpu中跑的，推荐用 insight，因为它汇编代码显示功能更细致。

####内存调试

* [Valgrind](http://valgrind.org/) :  instrumentation framework for building dynamic analysis tools.
                         
* [yamd](http://www.cs.hmc.edu/~nate/yamd/) : finding dynamic allocation related bugs in C and C++.

* [mtrace](http://en.wikipedia.org/wiki/Mtrace) : memory debugger included in the GNU C Library.
          
一篇关于内存调试的文章：[中文版](http://www.ibm.com/developerworks/cn/linux/sdk/l-debug/index.html#resource) 

####解析器

* flex

* [bison](https://www.gnu.org/software/bison/) : general-purpose parser generator that converts an annotated context-free grammar into a deterministic LR or generalized LR (GLR) parser employing LALR(1) parser tables.

* cxref   为一个或多个源文件生成HTML列表。 仅用于C，不支持C++
          
####调用跟踪器

* strace：系统调用跟踪器，可以跟踪你的程序所调用的系统调用 man strace

* ltrace：动态库调用跟踪器，可以跟踪你的程序所调用的动态库接口 man ltrace

* [cflow](http://www.gnu.org/software/cflow/) : analyzes a collection of C source files and prints a graph, charting control flow within the program. 
		    

####性能分析器

* gprof：binutils中带的性能分析器，可以帮助你优化你的代码，提高程序速度

* qprof：另一个性能分析器，支持动态库的性能分析和多线程、多进程性能分析

* oprofile：一个系统范围的性能分析器，使用内核模块和一个后台进程进行数据采集，它不但可以获得某个进程的性能分析数据还可以获得内核的性能分析数据

* gcov + lcov：gcov是gcc自带的代码覆盖分析工具，可以追踪程序运行时哪部分代码被执行了，该部分代码执行的频率，以及执行的时间消耗。这可以帮助你测试软件已经进行程序优化。lcov是gcov的一个扩展，可以提供直观的分析信息。

* [poor man profile](http://poormansprofiler.org/) don't really provide methods to see what multithreaded programs are blocking on

* [zoom](http://www.rotateright.com/) : not free, a profiling tool for understanding software performance

####版本控制工具

（1）以 patch 形式维护和改进软件的开发模式 : diff + patch 

（2）以SCM工具（SVN/CVS/git）为中心仓库的软件开发模式。

 diff & patch：代码打补丁工具，一对好搭挡，diff用来生成代码补丁，而patch则用来给代码打补丁。另见“使用diff和patch维护源代码”，http://hi.baidu.com/olaputan/blog/item/3d6585011b9e53df277fb5fd.html

[quilt](http://savannah.nongnu.org/projects/quilt)  easily manage large numbers of patches by keeping track of the changes each patch makes.





























   



   
  
  

