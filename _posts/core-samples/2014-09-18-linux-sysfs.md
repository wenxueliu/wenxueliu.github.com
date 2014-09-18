

sysfs is the Virtual Filesystem created during the 2.6 Kernel release cycle to show device information as procfs did not do this type of information that well.

Memory etc has not been ported to sysfs as it was never intended to show that type of information so it is unlikely it will be ported at all.




In the beginning (way back in Unix), the way that programs found out about the running processes on the system was via directly reading process structures from the kernel memory (opening /dev/mem, and interpreting the raw data directly). This is how the very first 'ps' commands worked. Over time, some information was made available via system calls.

However, it is bad form to expose system data directly to user-space via /dev/mem, and obnoxious to be constantly creating new system calls every time you wanted to export some new piece of process data, and so a newer method was created to access structured data for user-space applications to find out about process attributes. This was the /proc filesystem. With /proc, the interfaces and structures (directories and files) could be kept the same, even as the underlying data structures in the kernel changed. This was much less fragile than the earlier system, and it scaled better.

The /proc filesystem was originally designed to publish process information and a few key system attributes, required by 'ps', 'top', 'free' and a few other system utilities. However, because it was easy to use (both from the kernel side and the user-space side), it became a dumping ground for a whole range of system information. Also, it started to gain read/write files, to be used to adjust settings and control the operation of the kernel or its various subsystems. However, the methodology of implementing control interfaces was ad-hoc, and /proc soon grew into a tangled mess.

The sysfs (or /sys filesystem) was designed to add structure to this mess and provide a uniform way to expose system information and control points (settable system and driver attributes) to user-space from the kernel. Now, the driver framework in the kernel automatically creates directories under /sys when drivers are registered, based on the driver type and the values in their data structures. This means that drivers of a particular type will all have the same elements exposed via sysfs.

Many of the legacy system information and control points are still accessible in /proc, but all new busses and drivers should expose their info and control points via sysfs.




    What is the difference between procfs and sysfs?

proc is the old one, it is more or less without rules and structure. And at some point it was decided that proc was a little to chaotic and a new way was needed.

Then sysfs was created, and the new stuff that was added was put into sysfs like device information.

So in some sense they do the same, but sysfs is a little bit more structured.

    Why are they made as file systems?

Unix philosophy tells us that everything is a file, therefore it was created so it behaves as files.

    As I understand it, proc is just something to store the immediate info regarding the processes running in the system.

Those parts has always been there and they will probably never move into sysfs.

But there is more old stuff that you can find in proc, that has not been moved.



　一、devfs

　　devfs是在2.4内核就出现了，它是用来解决linux中设备管理混乱的问题，你查看一下/dev下的设备文件就知道其中有许多是空的(也就是没有对应的硬件的)，但是它们却必须存在，所以这给linux设备管理带来了很多麻烦，为了解决这个问题，linux内核开发人员开发了devfs，并用一个守护进程devfsd来做一些与以前硬件驱动兼容的事情。

　　devfs和sysfs都是和proc一样，是一个虚拟的文件系统，向devfs注册的驱动程序，devfs将会在/dev下建立相应的设备文件；但是为了兼容，devfsd这个守护进程将会在某个设定的目录中建立以主设备号为索引的设备文件，如果不这么做，以前的许多应用将不能运行。

　　在2.6内核以前一直使用的是devfs，devfs挂载于/dev目录下，提供了一种类似于文件的方法来管理位于/dev目录下的所有设备，我们知道/dev目录下的每一个文件都对应的是一个设备，至于当前该设备存在与否先且不论，而且这些特殊文件是位于根文件系统上的，在制作文件系统的时候我们就已经建立了这些设备文件，因此通过操作这些特殊文件，可以实现与内核进行交互。

　　但是devfs文件系统有一些缺点，例如：不确定的设备映射，有时一个设备映射的设备文件可能不同，例如我的U盘可能对应sda有可能对应sdb；没有足够的主/辅设备号，当设备过多的时候，显然这会成为一个问题；/dev目录下文件太多而且不能表示当前系统上的实际设备；命名不够灵活，不能任意指定等等。

　　二、sysfs

　　sysfs是Linux 2.6所提供的一种虚拟档案系统。这个档案系统不仅可以把装置(devices)和驱动程式(drivers)的资讯从kernel space输出到user space，也可以用来对装置和驱动程式做设定。

　　sysfs的目的是把一些原本在procfs中的，关于装置的部份独立出来，以[装置阶层架构}(device tree)的形式呈现。这个档案系统由Patrick Mochel所写，稍后Maneesh Soni撰写 "sysfs backing store path"，以降低在大型系统中对内存的需求量。

　　sysfs一开始以ramfs为基础，也是一个只存在于内存中的档案系统。ramfs是在2.4核心处于稳定阶段时加入的。ramfs是一个优雅的实做，证明了要在当时仍很新的虚拟档案系统(VFS)下写一个简单的档案系统是多么容易的一件事。由于ramfs的简洁以及使用了VFS，稍后的一些内存形式的档案系统都以它作为开发基础。

　　sysfs刚开始被命名成ddfs(Device Driver Filesystem)，当初只是为了要对新的驱动程式模型除错而开发出来的。它在除错时，会把装置架构(device tree)的资讯输出到procfs档案系统中。但在Linus Torvalds的急切督促下，ddfs被转型成一个以ramfs为基础的档案系统。在新的驱动程式模型被整合进 2.5.1 核心时，ddfs 被改名成driverfs，以更确切描述它的用途。

　　在2.5核心开发的次年，新的‘驱动程式模型’和‘driverfs’证明了对核心中的其他子系统也有用处。kobjects被开发出来，作为核心物件的中央管理机制，而此时driverfs也被改名成sysfs。
   
　　正因为devfs上述这些问题的存在，在linux2.6内核以后，引入了一个新的文件系统sysfs，它挂载于/sys目录下，跟devfs一样它也是一个虚拟文件系统，也是用来对系统的设备进行管理的，它把实际连接到系统上的设备和总线组织成一个分级的文件，用户空间的程序同样可以利用这些信息以实现和内核的交互。

　　该文件系统是当前系统上实际设备树的一个直观反应，它是通过kobject子系统来建立这个信息的，当一个kobject被创建的时候，对应的文件和目录也就被创建了，位于/sys下的相关目录下，既然每个设备在sysfs中都有唯一对应的目录，那么也就可以被用户空间读写了。用户空间的工具udev就是利用了sysfs提供的信息来实现所有devfs的功能的，但不同的是udev运行在用户空间中，而devfs却运行在内核空间，而且udev不存在devfs那些先天的缺陷。很显然，sysfs将是未来发展的方向。

　　三、udev

　　udev是一种工具，它能够根据系统中的硬件设备的状况动态更新设备文件，包括设备文件的创建，删除等。设备文件通常放在/dev目录下，使用udev后,在/dev下面只包含系统中真实存在的设备。它于硬件平台无关的，位于用户空间，需要内核sysfs和tmpfs的支持，sysfs为udev提供设备入口和uevent通道，tmpfs为udev设备文件提供存放空间。

