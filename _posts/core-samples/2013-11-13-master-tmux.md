---
layout: post
category : tools
tagline: "效率利器"
tags : [linux, tmux, tools, tutorial]
---
{% include JB/setup %}

tmux
=================================
tmux 是一个优秀的终端复用软件，类似 GNU Screen，但来自于 OpenBSD，采用 BSD 授权。使用它最直观的好处就是，通过一个终端登录远程主机并运行 tmux 后，在其中可以开启多个控制台而无需再“浪费”多余的终端来连接这台远程主机。简单来说，tmux 是一个 multiplexers ,他可以让你同时运行多个终端，在多个终端之间切换。

功能
-----------------------------------------------

*  提供了强劲的、易于使用的命令行界面。
*  可横向和纵向分割窗口。
*  窗格可以自由移动和调整大小，或直接利用四个预设布局之一。
*  支持 UTF-8 编码及 256 色终端。
*  可通过交互式菜单来选择窗口、会话及客户端。
*  支持跨窗口搜索。
*  支持自动及手动锁定窗口。
*  可以配置 vi 或 emacs 按键绑定模式
*  有多个粘贴缓冲，可完全由按键进行选取、复制、以及粘贴操作
*  脚本化，通过脚本可以方便的控制 tmux 会话
*  有预设布局，可搜索窗口，自动命名窗口名称
*  文档清晰、详尽，配置很容易，尤其是状态行

安装
------------------------------------------------

*  sudo apt-get install tmux
*  yum install -y tmux
*  下载源码包安装

入门
------------------------------------------------
在terminal中输入`tmux`，就进入了tmux环境。

常用命令
------------------------------------------------
首先有几个概念需要解释：

* session(会话) : session是一个特定的终端组合。输入tmux就可以打开一个新的session。
* window(窗口) : window 为session中的终端。
* pane(面板)： pane为一个window分隔出来的各个间隔，即window中的终端。

`tmux ls` : 列出所有session

`tmux attach [-t sessionname]` 重新进入该session

------------------------------------------------

Ctrl+b  ： 激活控制台；此时以下按键生效，这是输入下面每个命令的前提。

**系统操作**

* `?` : 列出所有快捷键；按q返回。
* `d` : 脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话
* `D` : 选择要脱离的会话；在同时开启了多个会话时使用
* `Ctrl+z` : 挂起当前会话
* `r` : 强制重绘未脱离的会话
* `s` : 选择并切换会话；在同时开启了多个会话时使用; 按 q 退出
* `t` : 显示时钟
* `:` : 进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器
* `[` : 进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出
* `~` : 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
    
Note:
> 这些命令要像vi的命令一样熟悉，就要不断地用。有一个技巧是：如果忘掉命令了，你可用通过`Ctrl-b ?`查看，这也是你在一个没有经过自己配置的新环境迅速查找相关命令的极佳方式。所以，此命令一定不能忘记。

**窗口操作**

* `c` : 创建新窗口
* `&` : 关闭当前窗口
* `[数字]` ： 切换至指定窗口
* `p` : 切换至上一窗口
* `n` : 切换至下一窗口
* `l` : 在前后两个窗口间互相切换
* `w` : 通过窗口列表切换窗口
* `,` : 重命名当前窗口；这样便于识别
* `.` : 修改当前窗口编号；相当于窗口重新排序
* `f` : 在所有窗口中查找指定文本

**面板操作**

* `双引号` : 将当前面板平分为上下两块
* `%` : 将当前面板平分为左右两块
* `x` : 关闭当前面板
* `!` : 将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板
* `Ctrl 方向键` : 以1个单元格为单位移动边缘以调整当前面板大小
* `Alt 方向键` : 以5个单元格为单位移动边缘以调整当前面板大小
* `space(空格)` : 在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled
* `q` : 显示面板编号
* `o` : 在当前窗口中选择下一面板
* `方向键` : 移动光标以选择面板
* `{` : 向前置换当前面板
* `}` : 向后置换当前面板
* `Alt+o` : 逆时针旋转当前窗口的面板
* `Ctrl+o` : 顺时针旋转当前窗口的面板

**复制粘贴**

    按 C-a [ 进入复制模式，如果有设置 `setw -g mode-keys vi` 的话，可按 `vi` 的按键模式操作。移动至待复制的文本处，按一下`空格`，结合 vi 移动命令开始选择，选好后按`回车`确认。
    按 `C-a ]` 粘贴已复制的内容。

配置    
------------------------------------------------
###tmux配置文件的地址

* /etc/tmux.conf 存储的是系统中所有用户的全局配置
* ~/.tmux.conf 存储的时用户个人的配置

###默认配置

通过 `Ctrl-b ?` 查看配置

###自定义配置

推荐修改配置的方法:   首先将窗口分为左右两个面板，一个窗口`vim ~/.tmux.conf`
另一个面板，`Ctrl-b ?` 显示系统默认配置，然后你直接可用拷贝方式，将 默认配置全部拷贝到  `.tmux.conf`文件。然后修改之。

默认的前缀为 `<ctrl-b>`,比较难按，很多人会改为screen中的`<Ctrl-a>`，来保持一致性。自己根据自己的喜好来修改。

    set -g prefix ^s
    unbind ^b

**鼠标设置**

    setw -g mode-mouse on
    set -g mouse-select-pane on
    set -g mouse-resize-pane on
    set -g mouse-select-window on
`键盘控`表示基本不用鼠标。


**水平或垂直分割窗口**

    unbind '"'
    bind - split-window -v # 分割成上下两个窗口
    unbind %
    bind | split-window -h # 分割成左右两个窗口

**选择分割的窗格**

    bind k select-pane -U # 选择上窗格
    bind j select-pane -D # 选择下窗格
    bind h select-pane -L # 选择左窗格
    bind l select-pane -R # 选择右窗格

**重新调整窗格的大小**

    bind  J resize-pane -D 10
    bind  K resize-pane -U 10
    bind  H resize-pane -L 10
    bind  L resize-pane -R 10
    bind  <  resize-pane -L 20
    bind  >  resize-pane -R 20

**拷贝粘贴（vi风格）**

    unbind [
    bind Escape copy-mode
    unbind p
    bind p paste-buffer
    bind -t vi-copy 'v' begin-selection
    bind -t vi-copy 'y' copy-selection
    bind y run-shell "tmux show-buffer | xclip -sel clip -i" \; display-message "Copied tmux buffer to system clipboard"
**交换两个窗格**

    bind ^u swapp -U # 与上窗格交换 Ctrl-u
    bind ^d swapp -D # 与下窗格交换 Ctrl-d

**执行命令**

    bind m command-prompt "splitw -h 'exec man %%'" # 查看Manpage
    bind @ command-prompt "splitw -h 'exec perldoc -f %%'" # 查perl函数

**色彩设置**

    set -g default-terminal “screen-256color” 让tmux支持256色
设置底部状态条的颜色

    set -g status-fg yellow
    set -g status-bg black
    setw -g window-status-fg cyan
    setw -g window-status-bg default
    setw -g window-status-attr dim
    setw -g window-status-current-fg white
    setw -g window-status-current-bg red
    setw -g window-status-current-attr bright

设置面板间分割线的颜色

    set -g pane-border-fg green
    set -g pane-border-bg black
    set -g pane-active-border-fg red
    set -g pane-active-border-bg black

设置命令出错后提醒的颜色

    set -g message-fg whiteset -g message-bg blackset -g message-attr bright



###状态栏

tmux的状态栏配置非常简单。相比screen就…… 比如我的配置中：

    set -g status-left "#[fg=green]s#S:w#I.p#P#[default]"

这一行就将状态栏左侧配置为：

    绿色，#S,#I,#p分别表示session，window，pane编号

当然，你可以让状态行更强大,可以使用tmux-powerline,在此不介绍了。

    set -g status-left-length 40
    set -g status-left "#[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P" 
    #状态栏左侧长度和文字颜色
    set -g status-right "#[fg=cyan]%d %b %R" #右侧
    set -g status-utf8 on
    set -g status-interval 60 #每60秒更新一次显示的时间。默认是15秒
    setw -g monitor-activity on
    set -g visual-activity on
    #非当前窗口中有事件发生时（比如一个耗时的命令跑完了），状态栏上会有高亮提醒

###修改生效
* 进入命令模式后，执行 `:source-file ~/.tmux.conf` 之后，更改的配置才会生效
* 也可以设置绑定 `bind r source-file ~/.tmux.conf` 按r可以让更改后的tmux设置生效

###默认启动应用

如一个自动ssh到远程的脚本,下述英文字全大寫的部份, 请自行换成自己想要取的名字

    #!/bin/sh
    tmux new-session -d -s TMUX_NAME
    tmux new-window -t TMUX_NAME:0 -n 'SCREEN_NAME0' 'echo windows 1'
    tmux new-window -t TMUX_NAME:1 -n 'SCREEN_NAME1' 'ssh w1.example.com'
    tmux new-window -t TMUX_NAME:2 -n 'SCREEN_NAME2' 'ssh w2.example.com'
    tmux select-window -t TMUX_NAME:1
    tmux -2 attach-session -t TMUX_NAME

参考文献

[1](http://foocoder.com/blog/zhong-duan-huan-jing-zhi-tmux.html/)

[2](http://www.cnblogs.com/shires/archive/2013/05/20/linux_tmux.html)  

[3](http://wing2south.com/post/40670260768/tmux )   

