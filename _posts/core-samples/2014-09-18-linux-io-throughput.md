---
layout: post
category : linux
tagline: "IO 吞吐量"
tags : [ linux, io, throughput ]
---
{% include JB/setup %}



cat /proc/diskstats

   1       0 ram0 0 0 0 0 0 0 0 0 0 0 0
   1       1 ram1 0 0 0 0 0 0 0 0 0 0 0
   1       2 ram2 0 0 0 0 0 0 0 0 0 0 0
   1       3 ram3 0 0 0 0 0 0 0 0 0 0 0
   1       4 ram4 0 0 0 0 0 0 0 0 0 0 0
   1       5 ram5 0 0 0 0 0 0 0 0 0 0 0
   1       6 ram6 0 0 0 0 0 0 0 0 0 0 0
   1       7 ram7 0 0 0 0 0 0 0 0 0 0 0
   1       8 ram8 0 0 0 0 0 0 0 0 0 0 0
   1       9 ram9 0 0 0 0 0 0 0 0 0 0 0
   1      10 ram10 0 0 0 0 0 0 0 0 0 0 0
   1      11 ram11 0 0 0 0 0 0 0 0 0 0 0
   1      12 ram12 0 0 0 0 0 0 0 0 0 0 0
   1      13 ram13 0 0 0 0 0 0 0 0 0 0 0
   1      14 ram14 0 0 0 0 0 0 0 0 0 0 0
   1      15 ram15 0 0 0 0 0 0 0 0 0 0 0
   7       0 loop0 0 0 0 0 0 0 0 0 0 0 0
   7       1 loop1 0 0 0 0 0 0 0 0 0 0 0
   7       2 loop2 0 0 0 0 0 0 0 0 0 0 0
   7       3 loop3 0 0 0 0 0 0 0 0 0 0 0
   7       4 loop4 0 0 0 0 0 0 0 0 0 0 0
   7       5 loop5 0 0 0 0 0 0 0 0 0 0 0
   7       6 loop6 0 0 0 0 0 0 0 0 0 0 0
   7       7 loop7 0 0 0 0 0 0 0 0 0 0 0
   8       0 sda 99619 4679 2672276 1452764 42541 73197 5843802 3568956 0 470716 5027124
   8       1 sda1 304 1134 2418 2772 0 0 0 0 0 2772 2772
   8       2 sda2 162 0 1296 2072 0 0 0 0 0 2072 2072
   8       3 sda3 361 232 2710 3792 10 5 42 1360 0 3952 5152
   8       4 sda4 2 0 4 8 0 0 0 0 0 8 8
   8       5 sda5 5319 281 194138 126156 3876 4957 79864 76052 0 87348 202220
   8       6 sda6 49357 1068 1520890 567060 1548 7296 70752 225948 0 148032 792952
   8       7 sda7 3182 336 107146 83348 1802 1730 127416 51492 0 46824 134840
   8       8 sda8 40712 1597 841666 661736 32133 59209 5565728 3205188 0 335368 3871892
   8       9 sda9 173 31 1632 3540 0 0 0 0 0 3540 3540
   8      16 sdb 10375 135850 1150592 67568 3 0 24 4 0 19296 67572
   8      20 sdb4 10362 135850 1150488 67556 3 0 24 4 0 19284 67560

you may discovery that 

	304 + 162 + 361 + 2 + 5319 + 49357 + 3182 + 40712 + 173  = 120309 > 99619
	10375 > 10362

what happened ?

There are always a few bytes taken by the partition table. Additionally, there could be blocks marked as bad as well, most HDs have a few bad blocks even when new.



cat /proc/diskstats | grep 'sda'

    8       0 sda 99619 4679 2672276 1452764 42541 73197 5843802 3568956 0 470716 5027124

	1 - major number
	2 - minor mumber
	3 - device name
	4 - reads completed successfully
	5 - reads merged
	6 - sectors read
	7 - time spent reading (ms)
	8 - writes completed
	9 - writes merged
	10 - sectors written
	11 - time spent writing (ms)
	12 - I/Os currently in progress
	13 - time spent doing I/Os (ms)
	14 - weighted time spent doing I/Os (ms)

cat /proc/diskstats | grep 'hda1 '

	3 1 hda1 25838 525266 1505217 12041736

	Field 1 -- # of reads issued - Total number of reads issued to this partition.
	Field 2 -- # of sectors read - Total number of sectors requested to be read from this partition.
	Field 3 -- # of writes issued - Total number of writes issued to this partition.
	Field 4 -- # of sectors written - Total number of sectors requested to be written to this partition

###calculate write and read per second

    #! /usr/bin/env bash

    function get_throughput_by_procfs()
    {
        disk=$1
        disk={$disk:="sda"}
        disk=$disk" "
        while true; do
            io_read_begin_bytes=`cat /proc/diskstats | grep $disk | awk '{ print $6 }'`
            io_write_begin_bytes=`cat /proc/diskstats | grep $disk | awk '{ print $10 }'`
            sleep 1
            io_read_end_bytes=`cat /proc/diskstats | grep $disk | awk '{ print $6 }'`
            io_write_end_bytes=`cat /proc/diskstats | grep $disk | awk '{ print $10 }'`
            echo `expr $[($io_read_end_bytes - $io_read_begin_bytes) * 512]` `expr $[($io_write_end_bytes - $io_write_begin_bytes) * 512]`
        done
    }

    function get_throughput_by_sysfs()
    {
        disk=$1
        disk={$disk:="sda"}
        disk=$disk" "
        while true; do
            io_read_begin_bytes=`cat /sys/block/$disk/stat | awk '{ print $3 }'`
            io_write_begin_bytes=`cat /sys/block/$disk/stat | awk '{ print $7 }'`
            sleep 1
            io_read_end_bytes=`cat /sys/block/$disk/stat | awk '{ print $3 }'`
            io_write_end_bytes=`cat /sys/block/$disk/stat | awk '{ print $7 }'`
            echo `expr $[($io_read_end_bytes - $io_read_begin_bytes) / 2]` `expr $[($io_write_end_bytes - $io_write_begin_bytes) / 2]`
        done
    }

###SUFFIX

[/sys/block/<dev>/stat](https://www.kernel.org/doc/Documentation/block/stat.txt)


This file documents the contents of the /sys/block/<dev>/stat file.

The stat file provides several statistics about the state of block
device <dev>.

Q. Why are there multiple statistics in a single file?  Doesn't sysfs
   normally contain a single value per file?
A. By having a single file, the kernel can guarantee that the statistics
   represent a consistent snapshot of the state of the device.  If the
   statistics were exported as multiple files containing one statistic
   each, it would be impossible to guarantee that a set of readings
   represent a single point in time.

The stat file consists of a single line of text containing 11 decimal
values separated by whitespace.  The fields are summarized in the
following table, and described in more detail below.

Name            units         description
----            -----         -----------
read I/Os       requests      number of read I/Os processed
read merges     requests      number of read I/Os merged with in-queue I/O
read sectors    sectors       number of sectors read
read ticks      milliseconds  total wait time for read requests
write I/Os      requests      number of write I/Os processed
write merges    requests      number of write I/Os merged with in-queue I/O
write sectors   sectors       number of sectors written
write ticks     milliseconds  total wait time for write requests
in_flight       requests      number of I/Os currently in flight
io_ticks        milliseconds  total time this block device has been active
time_in_queue   milliseconds  total wait time for all requests

read I/Os, write I/Os
---------------

These values increment when an I/O request completes.

read merges, write merges
---------------

These values increment when an I/O request is merged with an
already-queued I/O request.

read sectors, write sectors
---------------

These values count the number of sectors read from or written to this
block device.  The "sectors" in question are the standard UNIX 512-byte
sectors, not any device- or filesystem-specific block size.  The
counters are incremented when the I/O completes.

read ticks, write ticks
---------------

These values count the number of milliseconds that I/O requests have
waited on this block device.  If there are multiple I/O requests waiting,
these values will increase at a rate greater than 1000/second; for
example, if 60 read requests wait for an average of 30 ms, the read_ticks
field will increase by 60*30 = 1800.

in_flight
---------------

This value counts the number of I/O requests that have been issued to
the device driver but have not yet completed.  It does not include I/O
requests that are in the queue but not yet issued to the device driver.

io_ticks
---------------

This value counts the number of milliseconds during which the device has
had I/O requests queued.

time_in_queue
---------------

This value counts the number of milliseconds that I/O requests have waited
on this block device.  If there are multiple I/O requests waiting, this
value will increase as the product of the number of milliseconds times the
number of requests waiting (see "read ticks" above for an example).

[/proc/diskstat document](https://www.kernel.org/doc/Documentation/iostats.txt)

I/O statistics fields
---------------

Since 2.4.20 (and some versions before, with patches), and 2.5.45,
more extensive disk statistics have been introduced to help measure disk
activity. Tools such as sar and iostat typically interpret these and do
the work for you, but in case you are interested in creating your own
tools, the fields are explained here.

In 2.4 now, the information is found as additional fields in
/proc/partitions.  In 2.6, the same information is found in two
places: one is in the file /proc/diskstats, and the other is within
the sysfs file system, which must be mounted in order to obtain
the information. Throughout this document we'll assume that sysfs
is mounted on /sys, although of course it may be mounted anywhere.
Both /proc/diskstats and sysfs use the same source for the information
and so should not differ.

Here are examples of these different formats:

2.4:
   3     0   39082680 hda 446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
   3     1    9221278 hda1 35486 0 35496 38030 0 0 0 0 0 38030 38030


2.6 sysfs:
   446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
   35486    38030    38030    38030

2.6 diskstats:
   3    0   hda 446216 784926 9550688 4382310 424847 312726 5922052 19310380 0 3376340 23705160
   3    1   hda1 35486 38030 38030 38030

On 2.4 you might execute "grep 'hda ' /proc/partitions". On 2.6, you have
a choice of "cat /sys/block/hda/stat" or "grep 'hda ' /proc/diskstats".
The advantage of one over the other is that the sysfs choice works well
if you are watching a known, small set of disks.  /proc/diskstats may
be a better choice if you are watching a large number of disks because
you'll avoid the overhead of 50, 100, or 500 or more opens/closes with
each snapshot of your disk statistics.

In 2.4, the statistics fields are those after the device name. In
the above example, the first field of statistics would be 446216.
By contrast, in 2.6 if you look at /sys/block/hda/stat, you'll
find just the eleven fields, beginning with 446216.  If you look at
/proc/diskstats, the eleven fields will be preceded by the major and
minor device numbers, and device name.  Each of these formats provides
eleven fields of statistics, each meaning exactly the same things.
All fields except field 9 are cumulative since boot.  Field 9 should
go to zero as I/Os complete; all others only increase (unless they
overflow and wrap).  Yes, these are (32-bit or 64-bit) unsigned long
(native word size) numbers, and on a very busy or long-lived system they
may wrap. Applications should be prepared to deal with that; unless
your observations are measured in large numbers of minutes or hours,
they should not wrap twice before you notice them.

Each set of stats only applies to the indicated device; if you want
system-wide stats you'll have to find all the devices and sum them all up.

    Field  1 -- # of reads completed
        This is the total number of reads completed successfully.
    Field  2 -- # of reads merged, field 6 -- # of writes merged
        Reads and writes which are adjacent to each other may be merged for
        efficiency.  Thus two 4K reads may become one 8K read before it is
        ultimately handed to the disk, and so it will be counted (and queued)
        as only one I/O.  This field lets you know how often this was done.
    Field  3 -- # of sectors read
        This is the total number of sectors read successfully.
    Field  4 -- # of milliseconds spent reading
        This is the total number of milliseconds spent by all reads (as
        measured from __make_request() to end_that_request_last()).
    Field  5 -- # of writes completed
        This is the total number of writes completed successfully.
    Field  6 -- # of writes merged
        See the description of field 2.
    Field  7 -- # of sectors written
        This is the total number of sectors written successfully.
    Field  8 -- # of milliseconds spent writing
        This is the total number of milliseconds spent by all writes (as
        measured from __make_request() to end_that_request_last()).
    Field  9 -- # of I/Os currently in progress
        The only field that should go to zero. Incremented as requests are
        given to appropriate struct request_queue and decremented as they finish.
    Field 10 -- # of milliseconds spent doing I/Os
        This field increases so long as field 9 is nonzero.
    Field 11 -- weighted # of milliseconds spent doing I/Os
        This field is incremented at each I/O start, I/O completion, I/O
        merge, or read of these stats by the number of I/Os in progress
        (field 9) times the number of milliseconds spent doing I/O since the
        last update of this field.  This can provide an easy measure of both
        I/O completion time and the backlog that may be accumulating.


To avoid introducing performance bottlenecks, no locks are held while
modifying these counters.  This implies that minor inaccuracies may be
introduced when changes collide, so (for instance) adding up all the
read I/Os issued per partition should equal those made to the disks ...
but due to the lack of locking it may only be very close.

In 2.6, there are counters for each CPU, which make the lack of locking
almost a non-issue.  When the statistics are read, the per-CPU counters
are summed (possibly overflowing the unsigned long variable they are
summed to) and the result given to the user.  There is no convenient
user interface for accessing the per-CPU counters themselves.

Disks vs Partitions
-------------------

There were significant changes between 2.4 and 2.6 in the I/O subsystem.
As a result, some statistic information disappeared. The translation from
a disk address relative to a partition to the disk address relative to
the host disk happens much earlier.  All merges and timings now happen
at the disk level rather than at both the disk and partition level as
in 2.4.  Consequently, you'll see a different statistics output on 2.6 for
partitions from that for disks.  There are only *four* fields available
for partitions on 2.6 machines.  This is reflected in the examples above.

Field  1 -- # of reads issued
    This is the total number of reads issued to this partition.
Field  2 -- # of sectors read
    This is the total number of sectors requested to be read from this
    partition.
Field  3 -- # of writes issued
    This is the total number of writes issued to this partition.
Field  4 -- # of sectors written
    This is the total number of sectors requested to be written to
    this partition.

Note that since the address is translated to a disk-relative one, and no
record of the partition-relative address is kept, the subsequent success
or failure of the read cannot be attributed to the partition.  In other
words, the number of reads for partitions is counted slightly before time
of queuing for partitions, and at completion for whole disks.  This is
a subtle distinction that is probably uninteresting for most cases.

More significant is the error induced by counting the numbers of
reads/writes before merges for partitions and after for disks. Since a
typical workload usually contains a lot of successive and adjacent requests,
the number of reads/writes issued can be several times higher than the
number of reads/writes completed.

In 2.6.25, the full statistic set is again available for partitions and
disk and partition statistics are consistent again. Since we still don't
keep record of the partition-relative address, an operation is attributed to
the partition which contains the first sector of the request after the
eventual merges. As requests can be merged across partition, this could lead
to some (probably insignificant) inaccuracy.

Additional notes
----------------

In 2.6, sysfs is not mounted by default.  If your distribution of
Linux hasn't added it already, here's the line you'll want to add to
your /etc/fstab:

none /sys sysfs defaults 0 0


In 2.6, all disk statistics were removed from /proc/stat.  In 2.4, they
appear in both /proc/partitions and /proc/stat, although the ones in
/proc/stat take a very different format from those in /proc/partitions
(see proc(5), if your system has it.)

-- ricklind@us.ibm.com

