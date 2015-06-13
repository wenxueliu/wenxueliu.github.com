---
layout: post
category :  linux
tagline: "linux 快捷键"
tags : [ tools, terminal, 效率利器, tutorial]
---
{% include JB/setup %}

众所周知，vim是因为我们的手始终不离开键盘和一系列快捷键而得名天下。后来有人将其移植到vimperator(firefox)和Pentadactyl(firefox)。如果再结合linux中的快捷键，我可以告诉你，除了编辑办公文档，我几乎不需要鼠标。最近，刚买了一个无线鼠标，但是发现自己很久没有用鼠标了，就把鼠标送了人，没有感到丝毫的不适。特此为记。

####为什么我们要这样做

    1 为了显示我很geek?
no，当你熟悉后，你就会感到效率上的提升是鼠标无法替代的。你自己可以估计下，在做同样的动作的时候，快捷键和鼠标的速度。然后，将乘以1000，你就知道你节省的时间，但你也许做同样的动作是100000,节省的时间，你自己估计吧。

    2 这样很难?
以我自己的经历，我不知道这有难度，只要你花点时间去熟悉，暂时记不住的，在你需要的时候，想想可不可以用快捷键就行了，你会找到你要的快捷键。

####快捷键一览

下面大多是我自己的经常用的。你熟悉后即会感到它的强大。经ubuntu 12.04测试

1.Gnome文件夹

    Ctrl+T：在文件夹中打开新的 Tab
    Ctrl+N 新建窗口
    Alt+N 切换到第N个标签（N为数字）
    Ctrl+W：在文件夹中关闭一个 Tab，如果仅有一个 Tab，则关闭整个文件夹
    Ctrl+H：切换隐藏文件（夹）显示或不显示
    Ctrl+1/2：修改文件夹视图为图标或者列表模式
    Ctrl + Shift + N：创建新文件夹
    Alt + ↑←/↓→：目录的后退和前进
    Alt + Enter：显示所选文件或者文件夹的属性信息
    Alt + Home：直接移到主文件夹
    F9：显示和关闭边栏
注：我已经很久不用图形界面来切换目录和文件了。推荐我常用的工具ranger。

    Ctrl+Shift+Printscreen区域截图    我曾经在window XP下一般要打开QQ截图。囧


2.终端操作

    F11: 全屏切换
    Ctrl + Alt + F1：切换到首个虚拟终端
    Ctrl + Alt + F2(F3)(F4)(F5)(F6)：选择不同的虚拟终端
    Ctrl + Alt + F7：切换到当前登录会话
    Ctrl + Alt + L：锁屏

    Shift+Ctrl+T: 新建标签页（常用）
    Shift+Ctrl+W: 关闭标签页（常用）
    Alt + NUM: 切换到第NUM个标签
    Shift+Ctrl+PageUp: 标签页左移
    Shift+Ctrl+PageDown: 标签页右移
    Shift+Ctrl+N: 新建窗口（常用）
    Shift+Ctrl+Q: 关闭终端
    Shift+Ctrl+c: 复制选中
    Shift+Ctrl+v: 复制

    以下大多数为常用
    Ctrl + a 切换到命令行开始
    Ctrl + b 向前移动一个字符
    Ctrl + c 终端命令执行
    Ctrl + d 删除后一个字符
    Ctrl + e 切换到命令行末尾
    Ctrl + f 向前移动一个字符
    Ctrl + h 删除前一个字符
    Ctrl + k 删除光标之前的内容
    Ctrl + l 清除屏幕内容
    Ctrl + n 输出历史中下一个命令
    Ctrl + p 输出历史中上一个命令
    Ctrl + q 允许屏幕输出
    Ctrl + r 在历史命令中查找 （这个非常好用，输入关键字就调出以前的命令了）
    Ctrl + s 阻止屏幕输出
    Ctrl + t 交换前两个字符
    Ctrl + u 清除光标之前的内容
    Ctrl + w 删除光标前的单词
    Ctrl + x 在命令行开始和结束之间切换
    Ctrl + y 粘贴剪切板内容
    Ctrl + z 转入后台运行..
    alt键比较少用,因为很多地方与远程登陆工具是有冲突的，用ESC代替
    Alt + b 切换光标后的字母
    Alt + c 单词首字母大小
    Alt + d 删除字母之后到单词末尾
    Alt + f 向前移动一个单词
    Alt + l 当前字符大小全变为小写
    Alt + n 未知
    Alt + p 未知
    Alt + r 删除整个命令
    Alt + t 交换当前字符之前的前两个单词
    Alt + u 当前字符大小全变为大写
    Alt + . 使用上一条命令的最后一个参数

    Ctrl + Alt + F1：切换到首个虚拟终端
    Ctrl + Alt + F2(F3)(F4)(F5)(F6)：选择不同的虚拟终端
    Ctrl + Alt + F7：切换到当前登录会话
    Ctrl + Alt + L：锁屏

    Ctrl + +: 放大
    Ctrl + -: 减小
    Ctrl + 0: 原始大小

    记不住命令操作快捷键怎么办？
    可以用info readline和bind -p来查看。

3.应用程序

    Ctrl+Q：退出当前应用
    Alt+Tab：在不同的应用程序之间切换显示（**最常用**）也用于不同区域直接的切换。当按下Alt+快速按Tab时，会出现多个窗口，供选择。
    Alt+`  ：同一应用程序中多个窗口间切换`
    Win 长按：显示 Unity 边栏的应用编号，松开 E 再按相应编号即可轻松进行切换
    Win 快击：打开 Dash 应用程序菜单
    Win+s ：激活工作区切换器。缩小所有工作区。
    Ctrl+Win+d：隐藏所有窗口并显示桌面。再次按下按钮可以恢复窗口。
    Space：激活焦点所在项目，例如按钮、复选框或者列表
    F10：打开窗口菜单栏的第一个项目，然后在下拉菜单中使用方向键上下浏览。
    Ctrl+Win+↑：最大化窗口。
    Ctrl+Win+↓：将最大化的窗口恢复到初始尺寸。
    Ctrl+方向键：在列表视图或图标视图中，可以在不改变选中项的情况下将键盘焦点移动到另一个选项。
    Ctrl+Super+↑：最大化窗口。
    Ctrl+Super+←：在屏幕左半边纵向最大化窗口。
    Ctrl+Super+→：在屏幕右半边纵向最大化窗口。

4.工作空间

    Alt + Ctrl + ↑↓→←：移动工作空间，在 4 个工作空间进行切换
    Alt + Ctrl + Shift + ↑↓→←：移动当前窗口到另外的工作空间
    Alt + F2 类似Windows下的Win + R组合键，运行应用程序（意义不大）
    Alt + F4 关闭窗口(常用)
    Alt + F7 移动窗口 (注: 在窗口最大化的状态下无效)
    Alt + F8 改变窗口大小 (注: 在窗口最大化的状态下无效)
    Alt + Space 打开窗口的控制菜单 (点击窗口左上角图标出现的菜单)
    Alt + PrintScreen 当前窗口抓图

####历史命令

    引用命令 : ![!|[?]string|[-]number]
    选择word : :[n|x-y|^|$|*|n*|%]
    修饰符 : :[h|t|r|e|p|s|g]

下面是对上面的解释

    !! : 重复上次命令
    sudo !! : 以sudo重新执行上次命令
    !num : 重复执行历史命令中的第num个，可借助history查找。如 !-1 !10等等。
    string+↑↓ : 包含以string开头的历史命令
    !-n : 执行倒数第 n 个命令
    !string : 最近的以 string 为关键词开头的命令
    !string:p : 仅仅打印，并不执行最近以string的命令
    !?string[?] : 重复最近的包含string的命令
    shift+alt+# : 可以注释命令，这样可以在命令历史中找回。如果用ctrl+C放弃，则不会有记录。
    !^ : 上一条命令的第一个参数
    !$ : 上一条命令的最后参数
    !$:p 打印上一条命令的最后参数
    !* : 上一命令的所有参数
    !*:p : 打印上一命令的所有参数
    !# : 引用当前行
    !cmd:n : cmd 命令第 n 个参数
    !:n : 上一条命令第 n 个参数
    !:x-y : 上一条命令从 x 到 y 的参数
    !:n* : 上一条命令从 n 开始到最后的参数
    !$:h : 上一条命令最后一个参数到结尾的参数。(选取路径开头)
    !$:t : 上一条命令最后一个参数到结尾的参数。(选取路径结尾)
    !$:r : 上一条命令最后一个参数到结尾的参数。(选取文件名)
    !$:e : 上一条命令最后一个参数到结尾的参数。(选取文件扩展名)
    ^string : 删除上一命令的string
    ^string1^string2 : 执行上一个命令中，用string1代替string2
    ^string1^string2^ : 执行上一个命令中，string1全部用string2代替
    !:s/old/new : 上一条命令old替换为new
    !:gs/old/new :上一条命令old替换为new(全局)
    ![FILE_NAME] : 如 rm !(2.txt) 删除除2.txt的所有文件
    [ ! -d PATH ] : 检查文件夹是否存在 , 比如 [ ! -d /home/test/ ] && mkdir /home/test/

    echo $HISTSIZE
    echo $HISTFILE
    echo $HISTFILESIZE
    export PS1='!\! \u@\h:\w\$ ' : 给命令提示符中增加历史命令编号

####常备锦囊

使用 {} 构造字符串

    vim {a,b,c}file.c
    touch {01..5}.txt
    touch {1..10..2}.txt
    echo {9..1..2}
    mkdir -p 2014/{01..12}/{baby,photo}
    echo \{\{A..Z\},{a..z},{0..9}\}


####超级利器

在自己的用户主目录（home directory）新建一个 .inputrc 文件：

	$ vi ~/.inputrc
	"\e[A": history-search-backward
	"\e[B": history-search-forward
	set show-all-if-ambiguous on
	set completion-ignore-case on

退出 bash 后重新登陆，敲打一个字母或者几个字母，然后 “上下” 键，就会看到以这个字母搜索到的完整命令行。如果搜索到几个类似命令，通过上下键来切换，有点像 ctrl+r，但是更好用。

####标准输入的控制

* command > filename 把标准输出重定向到一个新文件中
* command >! filename 把标准输出重定向到一个新文件中（覆盖写）
* command >> filename 把标准输出重定向到一个文件中(追加)
* command 1> fielname 把标准输出重定向到一个文件中
* command > filename 2>&1 把标准输出和标准错误一起重定向到一个文件中
* command 2> filename 把标准错误重定向到一个文件中
* command 2>> filename 把标准输出重定向到一个文件中(追加)
* command >> filename 2>&1 把标准输出和标准错误一起重定向到一个文件中(追加)
* command filename2 < filename 把command命令以filename文件作为标准输入，以filename2文件作为标准输出
* command &m 把标准输出重定向到文件描述符m中
* command >>& filename 将命令执行时屏幕上所产生的任何信息附加到指定的文件中(追加)。
* command >& filename 将命令执行时屏幕上所产生的任何信息附加到指定的文件中。
* command >! filename 将命令的执行结果送至指定的文件中，若文件已经存在，则覆盖。

