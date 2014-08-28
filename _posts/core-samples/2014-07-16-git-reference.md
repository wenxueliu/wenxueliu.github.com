---
layout: post
category : tools
tagline: "git reference"
tags : [ git ]
---
{% include JB/setup %}

git clone远程分支
-----------------

git clone默认会把远程仓库整个给clone下来, 但只会在本地默认创建一个master分支. (当然,也可以在一开始clone的时候, 通过-b参数指定要clone的分支.而非master)，如果远程还有其他的分支，此时用git branch -a查看所有分支. 如下:

	\* master
	  remotes/origin/HEAD -> origin/master
	  remotes/origin/magicvoid
	  remotes/origin/master
	  remotes/origin/sunxiaofan

假设我们现在想取 magicvoid 分支到本地，并自动建立 tracking.有三种方式:

* git checkout -b magicvoid origin/magicvoid  //-b magicvoid表示在本地新建一个叫magicvoid的分支, 与远程的origin/magicvoid对应.
* git checkout -t origin/magicvoid  //-t参数, 默认会在本地建立一个和远程分支一样名字的分支
* git fetch origin magicvoid:magicvoid //命令格式是: git fetch <远程名> <远程分支>:<本地分支> ;

建议用前面2种,因为我们知道git库的所有信息都是会存在本地的。所以使用前两种都是在本地就能进行。而使用git fetch命令则需要连接到远程服务器上。而且, 使用git fetch, 建立的本地分支不是一个track branch, 成功后也不会自动切换到该分支上。

注意不要使用下面的方法来clone一个远程分支.

	git branch magicvoid
	git checkout magicvoid
	git pull origin magicvoid:magicvoid

因为，这样建立的branch是以master为基础建立的，再pull下来的话，会和master的内容进行合并，有可能会发生冲突.

参考自[这里](http://www.scmlife.com/thread-22562-1-1.html) 
