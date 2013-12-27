---
layout: post
category : linux
tagline: "sougou pinyin"
tags : [ tools, software, fcitx]
---

{% include JB/setup %}

自从由ibus切换到fcitx的搜狗拼音后，输入效率大增，所以有下面的记录。(ubuntu12.04 13.04亲测)。

1 Ubuntu 默认是安装了ibus.所以删除它

`sudo apt-get remove ibus`

对于已经安装老版本的fcitx,删掉再装.

`sudo apt-get remove fcitx*`

2 删除依赖库

`sudo apt-get autoremove`

3 检测是否删除fcitx

`dpkg --get-selections | grep fcitx`  
`dpkg --get-selections | grep ibus`

根据上面的结果卸载与ibus有关的软件。如python-ibus ibus-gtk ibus-gtk3等


4 添加fcitx的ppa：

`sudo add-apt-repository ppa:fcitx-team/nightly`

5 刷新软件源：

`sudo apt-get update`


6 直接安装搜狗输入法

`sudo apt-get install fcitx-sogoupinyin`

7 依次安装下列包（下17个应该是必须的）

sudo apt-get install fcitx fcitx-bin fcitx-config-common fcitx-config-gtk fcitx-data fcitx-frontend-gtk2 fcitx-frontend-gtk3 fcitx-frontend-qt4 fcitx-googlepinyin fcitx-libs fcitx-module-dbus fcitx-module-x11 fcitx-modules fcitx-pinyin fcitx-table fcitx-table-wubi fcitx-ui-classic


8 设置为fcitx为默认输入法

`im-switch -s fcitx -z default`
`sudo im-switch -s fcitx -z default`

由于4.2.4新版的Fcitx与系统默认的Locale有点问题。我们把下面这段代码粘贴到主文件夹下的.xprofile中（如果没有这个文件，则新建一个。）

    export LC_ALL=zh_CN.utf8
    export XMODIFIERS=@im=fcitx
    export QT_IM_MODULE=xim
    export GTK_IM_MODULE=xim
    fcitx -d
 
9 注销或重启就生效

10 在右上角的输入法设置中"输入法" ==> 搜狗拼音或输入法的“配置”中设置这个输入法的翻页不是安pagedown或pageup,而是加号键跟减号键(其实不加shift）

参考文献
http://www.mintos.org/config/quantal-fcitx.html
