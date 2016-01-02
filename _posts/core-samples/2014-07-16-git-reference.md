---
layout: post
category : tools
tagline: "git reference"
tags : [ git ]
---
{% include JB/setup %}


掌握一个工具关键是要理解它的核心思想, 核心数据模型, git 也不例外, 如果能很好理解下面几个核心概念, 掌握 git 剩下的只有记忆的工作了.

核心概念
-----------------

    Workspace：工作区
    Index / Stage：暂存区
    Repository：仓库区（或本地仓库）
    Remote：远程仓库


git 自动补全
-----------------

下载补全脚本

    cd ~
    curl -v https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash

在~/.bash_profile文件中添加以下代码

    if [ -f ~/.git-completion.bash ]; then
        . ~/.git-completion.bash
    fi

bashrc生效

    source .bashrc


查看日志
-----------------

    git log --oneline

把每次提交间显示的信息压缩成缩减的hash值和提交信息，在一行显示。

    git log --graph

输出界面的左手边用一种基于文本的图形表示法来显示历史。如果你只是浏览一个单独分支的历史，
那么这个功能是没有用的。

    git log --all

显示全部分支的历史


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


查看修改记录
-----------------

    git diff            比较工作区和暂存区
    git diff HEAD       比较工作区和HEAD
    git diff --cached   比较暂存区和HEAD
    git diff --cached A 比较暂存区和提交ID A
    git diff A          比较工作区和提交ID A
    git diff A   B      比较提交记录A 和 B

    如果你需要更精细化的比较 git diff --word-diff 一定你很喜欢


精细化提交
-----------------

    文件级别

    git add -i

    TODO

    行级别
    git add -p [filename]

    出现 Stage this hunk [y,n,q,a,d,/,K,g,e,?]?

    y - stage this hunk
    n - do not stage this hunk
    q - quit; do not stage this hunk nor any of the remaining ones
    a - stage this hunk and all later hunks in the file
    d - do not stage this hunk nor any of the later hunks in the file
    g - select a hunk to go to
    / - search for a hunk matching the given regex
    j - leave this hunk undecided, see next undecided hunk
    J - leave this hunk undecided, see next hunk
    k - leave this hunk undecided, see previous undecided hunk
    K - leave this hunk undecided, see previous hunk
    s - split the current hunk into smaller hunks
    e - manually edit the current hunk
    ? - print help

一心二用
-----------------

场景：你正在分支A工作，而且做了不少修改但是还没有完成，突然老大说，赶紧的
分支B 有个bug，修复一下，客户很急。如果你

    git checkout B

报错:
    error: Your local changes to the following files would be overwritten by
    checkout:
            /Tools/A.cpp
    Please, commit your changes or stash them before you can switch
    branches.
    Aborting


正确操作

    git stash 保存当前工作的修改
    git checkout B
    完成后
    git stash list
    git stash pop [--index] stash@{[NUM]} //[NUM] 一般为数字

    其他
    git stash apply stash@{[NUM]} //与pop的区别主要是不删除恢复进度
    git stash drop stash@{[NUM]}


谁动了我的代码？
-----------------

    git blame [file_name]

显示文件中每一行的作者，最后一次改动后进行的提交(commit)以及该次提交的时间戳。

重构提交历史
-----------------

    git rebase


精细化合并
----------------

    git cherry-pick [commit_hash]

从不同的分支里选择某次提交并且把它合并到当前的分支来


起死回生
-----------------

    git reflog

检查丢失的提交
----------------

    git fsck --lost-found


统计某人的代码提交量，包括增加，删除：
---------------

	git log --author="$(git config --get user.name)" --pretty=tformat: --numstat | gawk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END{ printf "added lines: %s removed lines : %s total lines: %s\n",add,subs,loc }' -

仓库提交者排名前 5（如果看全部，去掉 head 管道即可）：
---------------

	git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5

仓库提交者（邮箱）排名前 5：这个统计可能不会太准，因为很多人有不同的邮箱，但会使用相同的名字
---------------

	git log --pretty=format:%ae | gawk -- '{ ++c[$0]; } END { for(cc in c) printf "%5d %s\n",c[cc],cc; }' | sort -u -n -r | head -n 5 

贡献者统计：
---------------

	git log --pretty='%aN' | sort -u | wc -l

提交数统计：
---------------

	git log --oneline | wc -l 

添加或修改的代码行数：
---------------

	git log --stat|perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/;
 

    git log 参数说明：
    --author   指定作者
    --stat   显示每次更新的文件修改统计信息，会列出具体文件列表
    --shortstat    统计每个commit 的文件修改行数，包括增加，删除，但不列出文件列表：  
    --numstat   统计每个commit 的文件修改行数，包括增加，删除，并列出文件列表：
     
       
    -p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
           例如：git log -p  -2
    --name-only 仅在提交信息后显示已修改的文件清单
    --name-status 显示新增、修改、删除的文件清单
    --abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符
    --relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）
    --graph 显示 ASCII 图形表示的分支合并历史
    --pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）
           例如： git log --pretty=oneline ; git log --pretty=short ; git log --pretty=full ; git log --pretty=fuller
    --pretty=tformat:   可以定制要显示的记录格式，这样的输出便于后期编程提取分析
           例如：git log --pretty=format:""%h - %an, %ar : %s""
           下面列出了常用的格式占位符写法及其代表的意义。                   
           选项       说明                  
           %H      提交对象（commit）的完整哈希字串               
           %h      提交对象的简短哈希字串               
           %T      树对象（tree）的完整哈希字串                   
           %t      树对象的简短哈希字串                    
           %P      父对象（parent）的完整哈希字串               
           %p      父对象的简短哈希字串                   
           %an     作者（author）的名字              
           %ae     作者的电子邮件地址                
           %ad     作者修订日期（可以用 -date= 选项定制格式）                   
           %ar     作者修订日期，按多久以前的方式显示                    
           %cn     提交者(committer)的名字                
           %ce     提交者的电子邮件地址                    
           %cd     提交日期                
           %cr     提交日期，按多久以前的方式显示              
           %s      提交说明  
    --since  限制显示输出的范围，
           例如： git log --since=2.weeks    显示最近两周的提交
           选项 说明                
           -(n)    仅显示最近的 n 条提交                    
           --since, --after 仅显示指定时间之后的提交。                    
           --until, --before 仅显示指定时间之前的提交。                  
           --author 仅显示指定作者相关的提交。                
           --committer 仅显示指定提交者相关的提交。
     
一些例子： 

	git log --until=1.minute.ago // 一分钟之前的所有 log 
	git log --since=1.day.ago //一天之内的log 
	git log --since=1.hour.ago //一个小时之内的 log 
	git log --since=`.month.ago --until=2.weeks.ago //一个月之前到半个月之前的log 
	git log --since ==2013-08.01 --until=2013-09-07 //某个时间段的 log   



参考 《git权威指南》
