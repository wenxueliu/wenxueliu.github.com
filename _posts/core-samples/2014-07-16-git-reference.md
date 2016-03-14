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

方法一
    cd ~
    curl -v https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash

在~/.bash_profile文件中添加以下代码

    if [ -f ~/.git-completion.bash ]; then
        . ~/.git-completion.bash
    fi

方法二

    cp contrib/completion/git-completion.bash /etc/bash_completion.d/
    /etc/bash_completion.d/git-completion.bash


bashrc生效

    source .bashrc

git 配色输出
----------------

在 CentOS 默认 git 输出是没有配色:

git config --global --add color.ui true

查看日志
-----------------

    git log --oneline

把每次提交间显示的信息压缩成缩减的hash值和提交信息，在一行显示。

    git log --graph

输出界面的左手边用一种基于文本的图形表示法来显示历史。如果你只是浏览一个单独分支的历史，
那么这个功能是没有用的。

    git log --all

显示全部分支的历史


clone 远程分支
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

修改远程分支
-----------------

    git branch -a                           查看远程分支
    git push origin --delete develop        删除远程分支 develop
    git push origin --delete tag 0.1.1      删除远程 tag 0.1.1
    git push origin :develop                删除远程分支 develop

    git tag -d 0.1.1                        删除本地 tag
    git push origin :refs/tags/0.1.1        删除远程 tag

    //把 develop 分支重命名为 rc1 分支
    git push --delete origin develop        删除远程分支 develop
    git branch -m develop rc1               重命名本地分支 develop 为 rc1
    git push origin rc1                     推送本地分支 rc1

    git push --tags                         本地所有 tag 推送到远程
    git fetch origin tag  0.1.1-rc1         获取远程tag 0.1.1-rc1



查看修改记录
-----------------

    git diff            比较工作区和暂存区
    git diff HEAD       比较工作区和HEAD
    git diff --cached   比较暂存区和HEAD
    git diff --cached A 比较暂存区和提交ID A
    git diff A          比较工作区和提交ID A
    git diff A   B      比较提交记录A 和 B

    如果你需要更精细化的比较 git diff --word-diff 一定你很喜欢


tag
----------------

    git tag                                 列出已有 tag
    git tag -l 'v1.4.2.*'                   模糊搜索 tag
    git tag -a v1.4 -m 'my version 1.4'     创建 tag v1.4
    git show v1.4                           显示 tag v1.4 的详细信息
    git tag -s v1.5 -m 'my signed 1.5 tag'  签署 tag v1.5
    git tag v1.4-lw                         创建轻量级 tag v1.4-lw, 轻量级标签实际上就是一个保存着对应提交对象的校验和信息的文件
    git tag -a v1.2 9fceb02                 给提交 ID 9fceb02 加 tag v1.2
    git push origin v1.5                    将 v1.5 推送到远程
    git push --tags                         本地所有 tag 推送到远程


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

    git rebase -i

可以实现 1)提交历史的合并, 2)提交顺序的修改, 3)提交历史的拆分. 但如果是多人合作,
建议只进行本地提交历史的修改, 远程需要了解风险所在.


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

	git log --stat|perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/';

一些例子:

	git log --until=1.minute.ago // 一分钟之前的所有 log 
	git log --since=1.day.ago //一天之内的log 
	git log --since=1.hour.ago //一个小时之内的 log 
	git log --since=`.month.ago --until=2.weeks.ago //一个月之前到半个月之前的log 
	git log --since ==2013-08.01 --until=2013-09-07 //某个时间段的 log   

    git diff --shortstat "@{0 day ago}" 命令，可以查看今天你写了多少行代码。

##附录

###问题

    1. 我创建了本地分支b1并pull到远程分支 origin/b1；
    2. 其他人在本地使用fetch或pull创建了本地的b1分支；
    3. 我删除了 origin/b1 远程分支；
    4. 其他人再次执行fetch或者pull并不会删除这个他们本地的 b1 分支，运行 git branch -a 也不能看出这个branch被删除了，如何处理？

解决办法

    git remote show origin

    * remote origin
      Fetch URL: git@github.com:xxx/xxx.git
      Push  URL: git@github.com:xxx/xxx.git
      HEAD branch: master
      Remote branches:
        master                 tracked
        refs/remotes/origin/b1 stale (use 'git remote prune' to remove)
      Local branch configured for 'git pull':
        master merges with remote master
      Local ref configured for 'git push':
        master pushes to master (up to date)

这时候能够看到b1是stale的，使用 git remote prune origin 可以将其从本地版本库中去除。

更简单的方法是使用这个命令，它在fetch之后删除掉没有与远程分支对应的本地分支：

    git fetch -p

###问题2

在 github 上操作的时候，我在删除远程分支时碰到这个错误：

     git push --delete origin devel
    remote: error: refusing to delete the current branch: refs/heads/devel
    To git@github.com:zrong/quick-cocos2d-x.git
     ! [remote rejected] devel (deletion of the current branch prohibited)
    error: failed to push some refs to 'git@github.com:zrong/quick-cocos2d-x.git'

解决办法

1. 进入 github 中该项目的 Settings 页面；
2. 设置 Default Branch 为其他的分支（例如 master）；
3. 重新执行删除远程分支命令。



参考

* git权威指南
* http://zengrong.net/post/1746.htm
