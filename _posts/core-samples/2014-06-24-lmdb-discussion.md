---
layout: post
category : database 
tagline: "lmdb 讨论"
tags : [leveldb, lmdb]
---
{% include JB/setup %}

Paul Bank 
===========================

LMDB: The Leveldb Killer?
--------------------------

I've been quiet for a while on this blog, busy with many projects, but I just had to comment on my recent discovery of [Lightning Memory-Mapped Database (LMDB)](http://symas.com/mdb/). It's very impressive, but left me with some questions.

###Disclaimer

Let me start out with this full acknowledgement that I have not yet had a chance to compile and test LMDB (although I certainly will). This post is based on my initial response to the literature and discussion I've read, and a quick read through the source code.

I'm also very keen to acknowledge the author [Howard Chu](https://twitter.com/hyc_symas) as a software giant compared to my own humble experience. I've seen other, clearly inexperienced developers online criticising his code style and I do not mean to do the same here. I certainly hope my intent is clear and my respect for him and this project is understood throughout this discussion. I humbly submit these issues for discussion for the benefit of all.

###Understanding the Trade-offs

First up, with my previous statement about humility in mind, the biggest issue I ran up against when reviewing LMDB is partly to do with presentation. The slides and documentation I've read do a good job of explaining the design, but not once in what I've read was there any more than a passing mention of anything resembling a trade-off in the design of LMDB.

My engineering experience tells me that no software, especially when attempting to claim "high performance" comes without some set of assumptions and some trade-offs. So far everything I have read about LMDB has been so positive I'm left with a slight (emphasis important) feel of the "silver bullet" marketing hype I'd expect from commercial database vendors and which I've come to ignore.

Please don't get me wrong, I don't think the material I've reviewed is bad, just seems to lack any real discussion of the downsides - the areas where LMDB might not be the best solution out there.

On a personal note, I've found the apparent attitude towards leveldb and Google engineers a little off-putting too. I respect the authors opinion that LSM tree is a bad design for this purpose but the lack of respect toward it and it's authors that comes across in some presentations seems detrimental to the discussion of the engineering.

So to sum up the slight gripe here: engineers don't buy silver-bullet presentations. A little more clarity on the trade-offs is important to convince us to take the extraordinary benchmark results seriously.

On reflection the previous statement goes too far - I do take the results seriously - my point was more that they may seem "to good to be true" without a little more clarity on the limitations. 


###My Questions

I have a number of questions that I feel the literature about LMDB doesn't cover adequately. Many of these are things I can and will find out for myself through experimentation but I'd like to make them public so anyone with experience might weigh in on them and further the community understanding.

Most of these are not really phrased as questions, more thoughts I had that literature does not address. Assume I'm asking the author or anyone with insight their thoughts on the issues discussed.

To reiterate, I don't claim to be an expert. Some of my assumptions or understanding that lead the the issues below may be wrong - please correct me. Some of these issues may not be at all important in many use cases too. But I'm interested to understand these areas more so please let me know if you have thoughts or preferably experience with any of this.


###Write Amplification

It seems somewhat skimmed over in LMDB literature that the COW B-tree design writes multiple whole pages to disk for every single row update. That means that if you store a counter in each entry then an increment operation (i.e, changing 1 or 2 bits) will result in some number of pages (each 4kb by default) of DB written to disk. I've not worked out the branching factor given page size for a certain average record size but I guess in realistic large DBs that could be in the order of 3-10 4k pages written for a single bit change in the data.

All that is said is that "it's sequential IO so it's fast". I understand that but I'd like to understand more of the qualifiers. For leveldb in synchronous mode you only need to wait for the WAL to have the single update record appended. Writing 10s of bytes vs 10s or 100s of kbytes for every update surely deserves a little more acknowledgement.

In fact if you just skimmed the [benchmarks](http://symas.com/mdb/microbench/) you might have missed it but in all write configurations (sync, async, random, sequential, batched) except for batched-sequential writes, leveldb performs better, occasionally significantly better.

Given that high update throughput is a strong selling point for leveldb and the fact that LMDB was designed initially for a high-read ratio use case I feel that despite the presence in stats all of the rest of the literature seems to ignore this trade-off as if it wasn't there at all.


###File Fragmentation

The free-list design for reclaiming disk space without costly garbage collection or compaction is probably the most important advance here over other COW B-tree designs. But it seems to me that the resulting fragmentation of data is also overlooked in discussion.

It's primarily a problem for sequential reads (i.e. large range scans). In a large DB that has been heavily updated, presumably a sequential read will on average end up having to seek backwards and forwards for each 4k page as they will be fragmented on disk.

One of the big benefits of LSM Tree and other compacting designs is that over time the majority of the data ends up in higher level files which are large and sorted. Admittedly, with leveldb, range scans require a reasonable amount of non-sequential IO as you need to switch between the files in different levels as you scan.

I've not done any thorough reasoning about it but seems from my intuition that with leveldb the relative amount of non-sequential IO needed will at least remain somewhat linear as more and more data ends up in higher levels where it is actually sequential on disk. With LMDB it seems to me that large range scans are bound to perform increasingly poorly over the life of the DB even if the data doesn't grow at all, just updates regularly.

But also, beyond the somewhat specialist case of large range scans, it seems to be an issue for writes. The argument given above is that large writes are OK because they are sequential IO but surely once you start re-using pages from the free list this stops being the case. What if blocks 5, 21 and 45 are next free ones and you need to write 3 tree pages for your update? I'm aware there is some attention paid to trying to find contiguous free pages but this seems like it can only be a partial solution.

The micro benchmarks show writes are already slower than leveldb but I'd be very interested to see a long-running more realistic benchmark that shows the performance over a much longer time where fragmentation effects might become more significant.


###Compression

The LMDB benchmarks simply state that "Compression support was disabled in the libraries that support it". I understand why but in my opinion it's a misleading step.

The author states "any compression library could easily be used to handle compression for any database using simple wrappers around their put and get APIs". But that is totally missing the point. Compressing each individual value is a totally different thing to compressing whole blocks on disk.

Consider a trivial example: each value might look like {"id": 1234567, "referers": ["http://example.com/foo", "https://othersite.org/bar"] }. On it's own gzipping that value is unlikely to give any real saving (the repetition of 'http' possibly but the gzip headers is more than the saving there). Whereas compressing a 4k block of such results is likely to give a significant reduction even if it is only in the JSON field names repeated each time.

This is a trivial example I won't pursue and better serialisation could fix that but in my real-world experience most data even with highly optimised binary serialisation often ends up with a lot of redundancy between records - even if it's just in the keys. Block compression is MUCH more effective for the vast majority of data types than the LMDB author implies with that comment.

Leveldb's file format is specially designed in such a way that compression is possible and effective and it seems Google's intent is to use it as a key part of the performance of the data structure. Their own benchmarks show performance gains of over 40% with compression enabled. And that is ignoring totally the size on-disk which for many will be a fairly crucial part of the equation especially if relatively expensive SSD space is required.

One argument might be that you could apply compression at block level to LMDB too but I don't think it would be easy at all. It seems like it relies on fixed block size for it's addressing and compressing contents and leaving blanks gives no disk space saving and probably no IO saving either since all 4k is likely still read from disk.

I'm pretty wary of the benchmarks where leveldb has compression off since I see it as a fairly fundamental feature of leveldb that it is very compression friendly. Any real implementation would surely have compression on since there are essentially no downsides due to the design. It's also baked in (provided you have the snappy lib) and on by default for leveldb so it's not like it's an advanced bit of tuning/modification from basic implementation to use compression for leveldb.

Maybe I'm wrong and it's trivial to add effective compression to LMDB but if so, and doing it would give ~40% performance increase why is it not already done and compared?

I'd like to see the benchmarks re-run with compression on for leveldb. Given writes are already quicker for leveldb this more realistic real-world comparison might well give a better insight into the tradeoffs of the two designs. If I get a chance I will try this myself.

###Large Transactions Amplify Writes Even Further

LMDB makes a big play of being fully transactional. It's a great feature and implemented really well. My (theoretical) problem is to do with write performance - we've already seen how writes can be slower due to COW design but how about the case when you update many rows in one transaction.

Consider worst case that you modify 1 row in every leaf node, that means that the transaction commit will re-write every block in the database file. I realise currently that there is a limit on how many dirty pages can be accumulated by a single transaction but I've also read there are plans to remove this.

Leveldb by contrast can do an equivalent atomic batch write without anywhere near the same disk IO in the commit path. It would seem this is a key reason leveldb is so much better in random batch write mode. Again I'd love to see the test repeated with leveldb compression on too. On reflection, probably not such a big deal - writes to the WAL in leveldb won't be affected by compression. 

It may not be a problem for your workload but actually it might. Having certain writes use so much IO could cause you some real latency issues and given single writer lock, could give you similar IO-based stalls that leveldb is known for due to it's background compaction.

I'll repeat this is all theoretical but I'd want to understand a lot more detail like this before I used LMDB in a critical application.


###Disk Reclamation

Deleting a large part of the DB does not free any disk space for other DBs or applications in LMDB. Indeed there is no built in feature or any tools I've seen that will help you re-optimise the DB after a major change, nor help migrate one DB to another to reclaim the space.

This may be a moot point for many but for practical systems, having to solve these issues in the application might add significant work for the developer and operations teams where others (leveldb) would eventually reclaim the space naturally with no effort.


###Summary

I feel to counter the potentially negative tone I may have struck here, I should sum up by saying LMDB looks like a great project. I'm genuinely interested in the design and performance of all the options in this space.

I would suggest that a real understanding of the strengths and weaknesses of each option is an important factor in making real progress in the field. I'd humbly suggest that, if the author of LMDB was so inclined, including at least some discussion of some of these issues in the docs and benchmarks would benefit all.

I'll say it again if Howard or anyone else who's played with LMDB would like to comment on any of these issues, I'm looking forward to learning more.

So is LMDB a leveldb killer? I'd say it seems good, but more data required.
- See more at: http://banksco.de/p/lmdb-the-leveldb-killer.html#sthash.w7z57snh.dpuf


作者的应答
=================================

Was pointed at [this blog](http://banksco.de/p/lmdb-the-leveldb-killer.html) post via twitter, and decided to write a more complete response here. (Sorry, it’s just really hard to carry on a deep discussion or analysis only 140 characters at a time.)

There were a few good questions raised, and some that seemed a bit too obvious, but I’ll just take things in order.

Tradeoffs – I’m not sure how anyone can say this only gets a passing mention. We state quite clearly that [LMDB](http://symas.com/mdb/) is read-optimized, not write-optimized. I wrote this for the [OpenLDAP Project](http://www.openldap.org/); LDAP workloads are traditionally 80-90% reads. Write performance was not the goal of this design, read performance is. We make no claims that LMDB is a silver bullet, good for every situation. It’s not meant to be – but it is still far better at many things than all of the other DBs out there that *do* claim to be good for everything.

For the most part there aren’t a lot of other tradeoffs to discuss. I designed LMDB based on ~13 years of experience working with the BerkeleyDB transactional store, so it’s mostly a distillation of what we liked and didn’t like about BDB. I think the papers and presentations already cover those design aspects. Of course it’s also a product of ~35 years of experience writing optimal code. There are probably many paths that I take implicitly because it’s obvious to me that doing them any other way would be stupid, and it would take someone else’s questions to draw my attention to these unconscious choices. So before going further, let me say Thanks to Paul Banks for taking the time to write his comments and questions.

re: the personal note – I would have more respect for the LevelDB project if they had spelled out its own limitations as openly and as up-front as LMDB does. As it is, they are pushing it as a fast and lightweight DB suitable for everything, when it clearly is neither lightweight nor fit for general use. It is *certainly* not fit for use in high transaction rate heavy workloads. And from a practical perspective, data stored in a DB is only useful when you actually retrieve it again and *use* it. (Assuming you *can* retrieve it again; some folks like the MongoDB guys seem to forget that detail.) As such, a DB optimized for writes at the expense of read performance makes no sense. You can only swing so far; once you get to 50% writes/50% reads you are actually not *optimized* for anything at all. And if you go beyond that, to e.g. 60% writes/40% reads then you might as well use MongoDB and just pretend you wrote the data.

Write Amplification – I already summarized this in a comment on another blog, but I’ll repeat it here for convenience – in any logging-based design, you get write amplification on the order of 2x the size of your user data writes. This is because most data gets written twice – once first to the log, and then eventually to the main database. In a Copy-On-Write B+tree design, you get write amplification the order of 1x the size of your writes, plus the log of the number of keys – i.e., the depth of the tree. The data only gets written once, but you also must make copies of every page in the path from the leaf back to the root of the tree – this is also already clearly explained in my LMDB presentations. In the final analysis, there is a definite trade here – when all of your data items are “small” the logging approach will be more economical. When your data items are “large” the COW approach is better. The effect of this tradeoff is pretty clearly illustrated in our [microbenchmarks](http://symas.com/mdb/microbench/) – all of the other DB engines choke on the large value operations.

As for the fact that I/Os are always page-sized – I shouldn’t have to spell that out any more than already done in the presentations. LMDB is a disk-based B+tree design. That in itself should tell you that its I/Os are page-sized, since a page is the fundamental unit of disk I/O in a filesystem. The slides showing the operation of the tree are clearly labeled as operating on units of pages. LMDB relies on the OS’s virtual memory manager to page data in and out when the DB size is larger than RAM. Again, note the use of the word page.

Paul writes in his blog

    In fact if you just skimmed [the benchmarks](http://symas.com/mdb/microbench/) you might have missed it but in all write configurations (sync, async, random, sequential, batched) except for batched-sequential writes, leveldb performs better, occasionally significantly better.

I’ve stated this many times on the OpenLDAP mailing lists but it obviously applies to our larger audience – you have to actually read what’s in front of you. You’re dealing with textual information and if you skip over anything then you’re missing out. Attention to detail matters, in code and in prose. In this case, the “tiny” detail is that we never expected or claimed LMDB would be fast for writes. In fact, in one case where LMDB beat LevelDB, we plainly state:

    Also, surprisingly, MDB beats LevelDB’s write performance here.

I would think that makes our perspective very clear – LMDB is not a write-optimized design. The fact that it occasionally beats other designs that *do* claim to be designed for high write throughput is more a testament to their poor implementations, than to any conscious design effort on my part.

I disagree that we ignore this read vs write tradeoff. We are totally up-front about it in all of our presentations. Only someone who skims the information and isn’t paying attention could miss such a fundamental point.

File Fragmentation – this can certainly become a problem, but in practice it is a *worse* problem for other DB designs. See our [HyperDex benchmark](http://symas.com/mdb/hyperdex/) for a more long-running test. I have a couple simple rules for writing efficient code. A relevant one here is “don’t do the same work more than once” – the big problem with all logging-based systems is the massive amount of redundant writing they do. Compound that with any compaction/merging they require and the result is a quagmire of wasted resources. When you decide “random I/O is bad”, then set out to develop a new DB engine to combat this problem, and wind up still performing miserably compared to a plain random I/O workload, there is no other way to put it than “Fail.”

For the record, in regards to our HyperDex benchmarks, I don’t consider running a database at 50x the size of system RAM to be a smart deployment choice. RAM is cheap these days. If you’re deploying clusters with thousands of nodes with only 8GB of RAM on each to solve your “big data” problems You’re Doing It Wrong. You’ll occupy more of your compute resources in cluster management overhead than in actually working on your tasks.

Compression – this is utterly off the mark. LMDB’s purpose is to be a compact and efficient transactional embedded B+tree. Whatever else you may want it to do, first and foremost it must operate efficiently on its own. Any DB design that *requires* compression in order to perform well is a broken design. And any DB design that *precludes* compression is a broken design too. But you can add whatever frills and other layers on top of LMDB as you like. Case in point, the CentiDB project implements block compression perfectly well on top of LMDB.

The microbenches were run without compression because we were benchmarking DB engines, not compression libraries.

Whatever other libraries get used with the DB engine are purely incidental. So LevelDB requires the Snappy compression library and the tcmalloc memory allocator in its build prereqs – tests with all of these components included aren’t showing how well *LevelDB* works. We can use Snappy and tcmalloc with other DB engines too; their use only muddies the picture. But you’re right Paul, it would be interesting to run tests where all of the DB engines were also using Snappy and tcmalloc. The before/after comparisons would be particularly enlightening, to show the impact of each component on overall performance. (We did a series of benchmarks on [malloc implementations for OpenLDAP slapd](http://highlandsun.com/hyc/malloc/) a few years back. Again, quite revealing, but we only did it after already thoroughly analyzing and optimizing slapd’s usage of malloc on its own. Always get the foundation right first, before adding frills.)

There’s another important point to be made here – LMDB was designed for *efficiency* – this is not the same goal as designing for *performance* as LevelDB claims to be. Frequently you see discussions saying “since this task is I/O bound, we can use free CPU cycles to compress the data and reduce the I/O bandwidth requirements.” LevelDB assumes there are free CPU cycles to use, and consumes them with abandon in a vain attempt to maximize performance. In our HyperDex testing we see what happens when this assumption is followed thru – HyperLevelDB continuously chews up 100% of all 4 cores of the test machine for the entire duration of the tests. This is an extremely myopic approach to design though, as it precludes anyone else getting any useful work done on the machine. In contrast, LMDB minimizes I/O bandwidth requirements simply by being more efficient in what it writes. As a result, very little I/O bandwidth is used and extremely little CPU is used, which makes it feasible for your higher level apps that are using the DB to actually get their work done.

Large Transactions – preemptive note – LMDB 0.9.7 has been released and the limit on transaction size is gone. As for the rest – again a naive comment about LevelDB not doing the bulk of its disk IO in the commit path. (Completely ignoring the fact that LevelDB doesn’t even support transactions…) The problem is in assuming that you have the entire machine to yourself and that nothing else is competing for compute resources while you shove the major work into a background thread. This simply won’t be true on a busy, heavily loaded system, and on an idle system no one will care either way.

Disk Reclamation – In our experience of 15 years of OpenLDAP deployments, databases never shrink. Any time you delete a record you will probably be adding a new one shortly thereafter.

Some final thoughts – “more data required” – please, do conduct your own tests and report your results. The only benchmark that is truly meaningful is the one you conduct on your own actual workloads. LMDB may not be the be-all/end-all of DB engines. There may well be workloads out there for which it is not the best choice today.

But LMDB is also a design for tomorrow; solving yesterday’s problems today is for slow-moving leviathans. Expending excessive amounts of CPU and disk resources to “solve” the random I/O problem is futile if your end result is still inferior to straight random access. And moreover, with the [imminent arrival](http://www.thessdreview.com/wp-content/cache/supercache/thessdreview.com/daily-news/latest-buzz/diablo-technologies-unveils-memory-channel-storage-outperforms-enterprise-ssd-and-pcie-flash-solutions/index.html.gz) of multiple [non-volatile RAM](http://www.techpowerup.com/188507/crossbar-unveils-resistive-ram-non-volatile-memory-technology.html) solutions looming on the market, it’s foolish to waste any further effort on it.

Thanks again Paul, for taking the time to study LMDB and write a thoughtful post with your questions. It’s always good to check assumptions.
