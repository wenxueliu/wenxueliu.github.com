###linux下的压缩及加密方法

####加密 

   zip -e filename.zip filename 回车, 输入2次密码即可.

####解密 

   unzip filename.zip   按提示输入密码即可.

####打包/压缩 

   zip filename.zip filename 
   tar cvf filename.tar filename

####解包 

   unzip filename.zip 
   tar xvf filename.tar

1.zip -r myfile.zip ./*

将当前目录下的所有文件和文件夹全部压缩成 myfile.zip 文件, -r 表示递归压缩子目录下所有文件.

2.unzip

unzip -o -d /home/sunny myfile.zip 

把myfile.zip文件解压到 /home/sunny/

-o:不提示的情况下覆盖文件；

-d:-d /home/sunny 指明将文件解压缩到/home/sunny目录下；

3.其他

zip -d myfile.zip smart.txt

删除压缩文件中smart.txt文件

zip -m myfile.zip ./rpm_info.txt

向压缩文件中myfile.zip中添加rpm_info.txt文件
要使用 zip 来压缩文件，在 shell 提示下键入下面的命令：

zip -r filename.zip filesdir

在这个例子里，filename.zip 代表你创建的文件，filesdir 代表你想放置新 zip 文件的目录。-r 选项指定你想递归地（recursively）包括所有包括在 filesdir 目录中的文件。

要抽取 zip 文件的内容，键入以下命令：

unzip filename.zip

你可以使用 zip 命令同时处理多个文件和目录，方法是将它们逐一列出，并用空格间隔：

zip -r filename.zip file1 file2 file3 /usr/work/school

上面的命令把 file1、file2、 file3、以及 /usr/work/school 目录的内容（假设这个目录存在）压缩起来，然后放入 filename.zip 文件中。

tar 命令详解

　　-c: 建立压缩档案
　　-x：解压
　　-t：查看内容
　　-r：向压缩归档文件末尾追加文件
　　-u：更新原压缩包中的文件
　　这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。
　　-c: 建立压缩档案
　　-x：解压
　　-t：查看内容
　　-r：向压缩归档文件末尾追加文件
　　-u：更新原压缩包中的文件

　　下面的参数-f是必须的

　　-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

　　# tar -cf all.tar *.jpg
　　这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。

　　# tar -rf all.tar *.gif
　　这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。

　　# tar -uf all.tar logo.gif
　　这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。

　　# tar -tf all.tar
　　这条命令是列出all.tar包中所有文件，-t是列出文件的意思

　　# tar -xf all.tar
　　这条命令是解出all.tar包中所有文件，-t是解开的意思


压缩

　　tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
　　tar –czf jpg.tar.gz *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
　　tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
　　tar –cZf jpg.tar.Z *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
　　rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
　　zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux

解压
　　tar –xvf file.tar //解压 tar包
　　tar -xzvf file.tar.gz //解压tar.gz
　　tar -xjvf file.tar.bz2 //解压 tar.bz2
　　tar –xZvf file.tar.Z //解压tar.Z
　　unrar e file.rar //解压rar
　　unzip file.zip //解压zip

总结
　　1、*.tar 用 tar –xvf 解压
　　2、*.gz 用 gzip -d或者gunzip 解压
　　3、*.tar.gz和*.tgz 用 tar –xzf 解压
　　4、*.bz2 用 bzip2 -d或者用bunzip2 解压
　　5、*.tar.bz2用tar –xjf 解压
　　6、*.Z 用 uncompress 解压
　　7、*.tar.Z 用tar –xZf 解压
　　8、*.rar 用 unrar e解压
　　9、*.zip 用 unzip 解压



　　tar命令
　　tar 可以为文件和目录创建档案。利用tar，用户可以为某一特定文件创建档案(备份文件)，也可以在档案中改变文件，或者向档案中加入新的文件。tar最初被 用来在磁带上创建档案，现在，用户可以在任何设备上创建档案，如软盘。利用tar命令，可以把一大堆的文件和目录全部打包成一个文件，这对于备份文件或将 几个文件组合成为一个文件以便于网络传输是非常有用的。Linux上的tar是GNU版本的。
　　语法：tar [主选项+辅选项] 文件或者目录
　　使用该命令时，主选项是必须要有的，它告诉tar要做什么事情，辅选项是辅助使用的，可以选用。
　　主选项：
　　c 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。
　　r 把要存档的文件追加到档案文件的未尾。例如用户已经作好备份文件，又发现还有一个目录或是一些文件忘记备份了，这时可以使用该选项，将忘记的目录或文件追加到备份文件中。
　　t 列出档案文件的内容，查看已经备份了哪些文件。
　　u 更新文件。就是说，用新增的文件取代原备份文件，如果在备份文件中找不到要更新的文件，则把它追加到备份文件的最后。
　　x 从档案文件中释放文件。
　　辅助选项：
　　b 该选项是为磁带机设定的。其后跟一数字，用来说明区块的大小，系统预设值为20(20*512 bytes)。
　　f 使用档案文件或设备，这个选项通常是必选的。
　　k 保存已经存在的文件。例如我们把某个文件还原，在还原的过程中，遇到相同的文件，不会进行覆盖。
　　m 在还原文件时，把所有文件的修改时间设定为现在。
　　M 创建多卷的档案文件，以便在几个磁盘中存放。
　　v 详细报告tar处理的文件信息。如无此选项，tar不报告文件信息。
　　w 每一步都要求确认。
　　z 用gzip来压缩/解压缩文件，加上该选项后可以将档案文件进行压缩，但还原时也一定要使用该选项进行解压缩。
　　Linux下的压缩文件剖析
　　对于刚刚接触Linux的人来说，一定会给Linux下一大堆各式各样的文件名给搞晕。别个不说，单单就压缩文件为例，我们知道在 Windows下最常见 的压缩文件就只有两种，一是,zip，另一个是.rap。可是Linux就不同了，它有.gz、.tar.gz、tgz、bz2、.Z、.tar等众多的 压缩文件名，此外windows下的.zip和.rar也可以在Linux下使用，不过在Linux使用.zip和.rar的人就太少了。本文就来对这些 常见的压缩文件进行一番小结，希望你下次遇到这些文件时不至于被搞晕。
　　在具体总结各类压缩文件之前呢，首先要弄清两个概念：打包和压缩。打包是指将一大堆文件或目录什么的变成一个总的文件，压缩则是将一个大的文件 通过一些压 缩算法变成一个小文件。为什么要区分这两个概念呢?其实这源于Linux中的很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你就 得先借助另它的工具将这一大堆文件先打成一个包，然后再就原来的压缩程序进行压缩。
　　Linux下最常用的打包程序就是tar了，使用tar程序打出来的包我们常称为tar包，tar包文件的命令通常都是以.tar结尾的。生成tar包后，就可以用其它的程序来进行压缩了，所以首先就来讲讲tar命令的基本用法：
　　tar命令的选项有很多(用man tar可以查看到)，但常用的就那么几个选项，下面来举例说明一下：
　　# tar -cf all.tar *.jpg
　　这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
　　# tar -rf all.tar *.gif
　　这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
　　# tar -uf all.tar logo.gif
　　这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
　　# tar -tf all.tar
　　这条命令是列出all.tar包中所有文件，-t是列出文件的意思
　　# tar -xf all.tar
　　这条命令是解出all.tar包中所有文件，-t是解开的意思

　　以上就是tar的最基本的用法。为了方便用户在打包解包的同时可以压缩或解压文件，tar提供了一种特殊的功能。这就是tar可以在打包或解包的同时调用其它的压缩程序，比如调用gzip、bzip2等。
　　1) tar调用gzip
　　gzip是GNU组织开发的一个压缩程序，.gz结尾的文件就是gzip压缩的结果。与gzip相对的解压程序是gunzip。tar中使用-z这个参数来调用gzip。下面来举例说明一下：
　　# tar -czf all.tar.gz *.jpg
　　这条命令是将所有.jpg的文件打成一个tar包，并且将其用gzip压缩，生成一个gzip压缩过的包，包名为all.tar.gz
　　# tar -xzf all.tar.gz
　　这条命令是将上面产生的包解开。
　　2) tar调用bzip2
　　bzip2是一个压缩能力更强的压缩程序，.bz2结尾的文件就是bzip2压缩的结果。与bzip2相对的解压程序是bunzip2。tar中使用-j这个参数来调用gzip。下面来举例说明一下：
　　# tar -cjf all.tar.bz2 *.jpg
　　这条命令是将所有.jpg的文件打成一个tar包，并且将其用bzip2压缩，生成一个bzip2压缩过的包，包名为all.tar.bz2
　　# tar -xjf all.tar.bz2
　　这条命令是将上面产生的包解开。
　　3)tar调用compress
　　compress也是一个压缩程序，但是好象使用compress的人不如gzip和bzip2的人多。.Z结尾的文件就是bzip2压缩的结 果。与 compress相对的解压程序是uncompress。tar中使用-Z这个参数来调用gzip。下面来举例说明一下：
　　# tar -cZf all.tar.Z *.jpg
　　这条命令是将所有.jpg的文件打成一个tar包，并且将其用compress压缩，生成一个uncompress压缩过的包，包名为all.tar.Z
　　# tar -xZf all.tar.Z
　　这条命令是将上面产生的包解开
　　有了上面的知识，你应该可以解开多种压缩文件了，下面对于tar系列的压缩文件作一个小结：
　　1)对于.tar结尾的文件
　　tar -xf all.tar
　　2)对于.gz结尾的文件
　　gzip -d all.gz
　　gunzip all.gz
　　3)对于.tgz或.tar.gz结尾的文件
　　tar -xzf all.tar.gz
　　tar -xzf all.tgz
　　4)对于.bz2结尾的文件
　　bzip2 -d all.bz2
　　bunzip2 all.bz2
　　5)对于tar.bz2结尾的文件
　　tar -xjf all.tar.bz2
　　6)对于.Z结尾的文件
　　uncompress all.Z
　　7)对于.tar.Z结尾的文件
　　tar -xZf all.tar.z
　　另外对于Window下的常见压缩文件.zip和.rar，Linux也有相应的方法来解压它们：
　　1)对于.zip
　　linux下提供了zip和unzip程序，zip是压缩程序，unzip是解压程序。它们的参数选项很多，这里只做简单介绍，依旧举例说明一下其用法：
　　# zip all.zip *.jpg
　　这条命令是将所有.jpg的文件压缩成一个zip包
　　# unzip all.zip
　　这条命令是将all.zip中的所有文件解压出来
　　2)对于.rar
　　要在linux下处理.rar文件，需要安装RAR for Linux，可以从网上下载，但要记住，RAR for Linux
　　不是免费的;然后安装：
　　# tar -xzpvf rarlinux-3.2.0.tar.gz
　　# cd rar
　　# make
　　这样就安装好了，安装后就有了rar和unrar这两个程序，rar是压缩程序，unrar是解压程序。它们的参数选项很多，这里只做简单介绍，依旧举例说明一下其用法：
　　# rar a all *.jpg
　　这条命令是将所有.jpg的文件压缩成一个rar包，名为all.rar，该程序会将.rar 扩展名将自动附加到包名后。
　　# unrar e all.rar
　　这条命令是将all.rar中的所有文件解压出来
　　到此为至，我们已经介绍过linux下的tar、gzip、gunzip、bzip2、bunzip2、compress、 uncompress、 zip、unzip、rar、unrar等程式，你应该已经能够使用它们对.tar、.gz、.tar.gz、.tgz、.bz2、.tar.bz2、. Z、.tar.Z、.zip、.rar这10种压缩文件进行解压了，以后应该不需要为下载了一个软件而不知道如何在Linux下解开而烦恼了。而且以上方 法对于Unix也基本有效。
　　本文介绍了linux下的压缩程式tar、gzip、gunzip、bzip2、bunzip2、 compress、uncompress、zip、unzip、rar、unrar等程式，以及如何使用它们对.tar、.gz、.tar.gz、. tgz、.bz2、.tar.bz2、.Z、.tar.Z、.zip、.rar这10种压缩文件进行操作
