零、shell中的内部变量:


1.    $?:    表示shell命令的返回值.
2.    $$:    表示当前shell的pid.
3.    $!:    最后一个放入后台作业的PID值.
4.    $0:    表示脚本的名字.
5.    $1--$9,${10}: 表示脚本的第一到九个参数,和第十个参数.
6.    $#:    表示参数的个数.
7.    $*,$@: 表示所有的参数.
       两者的区别如下: //都是双引号惹的祸^-^
       /> set 'apple pie' pears peaches
       /> for i in $*
       >  do
       >  echo $i
       >  done
       apple
       pie
       pears
       peaches
   
       /> set 'apple pie' pears peaches
       /> for i in $@
       >  do
       >  echo $i
       >  done
       apple
       pie
       pears
       peaches
   
       /> set 'apple pie' pears peaches
       /> for i in "$*"
       >  do
       >  echo $i
       >  done
       apple pie pears    peaches
   
       /> set 'apple pie' pears peaches
       /> for i in "$@"
       >  do
       >  echo $i
       >  done
       apple pie    //这里的单引号将两个单词合成一个.
       pears
       peaches

 

一、正则表达式在vi中的用法：


1.    ^:      如/^love,表示所有以love开头的行.
2.    $:      如/love$,表示所有以love结尾的行.
3.    .:       如/l..e, dot表示任意字符，如love,l22e,live等.
4.    *:      如/*love, *表示0多多个字符,这里表示love前面可以有0个多任意多个空格字符，如/go*gle,可以表示ggle,gogle,google,goooooooogle.
5.    []:     如/[Ll]ove,[]中的任意一个字符都可能成为候选者,如Love和love.
6.    [x-y]: 如/[A-Z]t, 表示[]中指定范围内的字符都可能成为候选者,如At, It等, 也可表示多个区间段如：[a-zA-TV-Z]表示所有除V之外的所有大小写英文字符.
7.    [^]:   如/[^A-Z]ove,表示A-Z之内的任意字符都是非法的, 如Love,Dove等.
8.    \:      转义符,    如果想表示任何meta字符的原义, 需使用在meta字符前加转义符\, 如\.将只表示dot，而不能在表示任何其他字符了.
9.    \<:    如/\<love, 表示任何单词的开始, 如love和lover, 但是glove将非法.
10.  \>:   如/love\>, 表示任何单词的结束, 如love和glove, 但是lover将非法.
11.  \(..\):      如/\(love\)able/\1rs/, 这里的\1表示love, 这种标签替代最多达到\9, 该例子表示用lovers代替loveable.
12.  x\{m\}:   如x\{5\}, 表示x被重复5次,如xxxxx.
13.  x\{m,\}:  如x\{5,\}, 表示x被至少重复5次,如xxxxx,xxxxxxxx.
14.  x\{m,n\}:如x\{5,10\}, 表示x被重复5-10次,如xxxxx,xxxxxxxx.
以下为grep的正则表示式用法:
15.  \w和\W: 等同于[a-zA-Z0-9].
16.  \b: 等同于\<和\>,均表示单词的边界.
以下为grep的正则表示式的扩展用法(grep -E或egrep):
17.  +:   如/lo+ve, +表示1个或者多个先前的字符,这里表示love,loove,但是lve非法.
18.  ?:   如/lo?ve, ?表示0个或者1个先前的字符, 这里只表示love和lve.
19.  (a|b|c):  如/l(o|i)ve, 表示或的意思,这里表示love和live. (o|i)和[oi]的主要区别就是(word|word)可以表示单词之间或的关系,[]只能表示字符.
20.  x{m},x{m,},x{m,n}  等同于grep普通模式中的x\{m\},x\{m,\},x\{m,n\}.

 

二、grep家族:


1.    家族成员:
       egrep: 执行带有扩展正则表达式元字符的grep搜索.
       fgrep:  将关闭grep的所有正则功能, 即搜索字符串中所有正则元字符都将只是表示其字符本意.
2.    返回值:
       0: 表示成功
       1: 表示搜索字符串不存在
       2: 表示搜索文件不存在.   
3.    grep的选项规则:
       -#,-A#和-B#: 表示在输出匹配内容的时候同时也输出其上下指定数量的行数, 如grep -2 "love" *, 该例输出匹配love的上下两行,
       grep -A2 "love" * 该例输出匹配love的后两行, grep -B2 "love" * 该例输出匹配love的前两行. 这里A表示after,B表示before.
       -F: 等同于fgrep, 这个选项将关闭所有正则功能,即所有正则的元字符均表示其本身含义.
       -c: 不输出找到的内容,只是输出在该文件中有多少匹配的行数.
       -h: 不输出匹配搜索字符串的文件的文件名,只是输出内容.
       -i:  搜索时忽略大小写.
       -l:  只显示匹配搜索内容的文件名, 不显示具体的内容.
       -L: 只显示没有包含搜索内容的文件名.
       -n: 输出匹配内容的同时也输出其所在的行号.
       -v: 反向搜索,输出不匹配搜索字符串的行.
       -w:只打印以完整单词形式匹配的行, 如果该搜索字符为某个单词的部分内容,将不会被输出.
       -x: 只打印以行形式匹配的行, 如果该搜索字符为行的部分内容,将不会被输出.
       -q: 不会输出任何信息, 该选项主要用于测试某个搜索字符或搜索pattern在执行grep命令之后的返回值.
       -r: 表示递归的搜索当前目录的子目录中的文件. 
4.    对于普通模式的grep,如果搜索的字符中普通字符前面加入\,则该字符按照扩展grep(egrep或者grep -E)的正则规则进行查找.如grep "love\|live" filename,
       将等同于egrep "love|live" filename,这里的\|将按照egrep中的|元字符处理, 再如, egrep "3+" filename等同于grep "3\+" filename.

 

三、sed:


1.    sed命令:
       ,:  表示范围.
       1) sed -n '/west/,/east/p' datafile 表示打印所有从包含west开始到包含east的行,如果直到文件的结尾都没有包含east的行,将打印west后面的所有行.
           其实逻辑很简单, 就是sed在发现包含west行之后开发打印该行,直到发现包含east的行打印才结束,否则一直打印直到文件的末尾.
       2) sed -n '5,/^northeast/p' datafile 表示从第五行开始打印,直到遇到以northeast开始的行结束打印.
       
       !:  表示对匹配结果取反.
       1) sed '/north/!d' datafile 将删除所有不包含north的行.

       a: 追加命令.
       1) sed '/^north/a first line \
           second line \
           third line' datafile 将会在所有包含north行的后面追加first line \r second line \n third line. 其中\表示下一行还有内容的连词. 如果是c-shell:
           sed '/^north/a first line \\
           second line \\
           third line' datafile 其中多出来的\是转义符.
       
       d: 表示删除.
       1) sed '/north/d' datafile 将删除所有包含north的行.
       2) sed '3d' datafile    将删除第三行.
       3) sed '3,$d' datafile    将删除第三行到文件的结尾行.
       4) sed 'd' datafile 将删除所有行.
       
       e: 表示多点编辑.
       1) sed -e '1,3d' -e 's/Hemenway/Jones/' datafile    一个sed语句执行多条编辑命令, 因此命令的顺序会影响其最终结果.
       2) sed -e 's/Hemenway/Jones/' -e 's/Jones/Max/' datafile 先用Jones替换Hemenway, 再用Max替换Jones.

       h和g/G: 保持和获取命令.
       1) sed -e '/northeast/h' -e '$G' datafile sed将把所有包含northeast的行轮流缓存到其内部缓冲区, 最后将只是保留最后一个匹配的行,
           $G是将缓冲区的行输出到$G匹配行的后面, 该例表示将最后一个包含northeast的行追加到文件的末尾.
       2) sed -e '/WE/{h; d;}' -e '/CT/{G;}' datafile 表示将包含WE的行保存到缓冲区, 然后删除该行,最后将缓冲区中保存的那份输出到CT行的后面.
       3) sed -e '/northeast/h' -e '$g' datafile 表示将包含northeast的行保存到缓冲区, 再将缓冲区中保存的那份替换文件的最后一行并输出.
           再与h合用时, g表示替换, G表示追加到匹配行后面.
       4) sed -e '/WE/{h; d;}' -e '/CT/{g;}' datafile 保留包含WE的行到缓冲区, 如果有新的匹配行出现将会替换上一个存在缓冲区中的行, 如果此时发现有
           包含CT的行出现, 就用缓冲区中的当前行替换这个匹配CT的行, 之后如果有新的WE出现, 将会用该新行替换缓冲区中数据, 当前再次遇到CT的时候,将用最
           新的缓冲区数据替换该CT行.
   
       i: 表示插入.
       1) sed '/north/i first line \
           second line \
           third line' datafile    其规则和a命令基本相同, 只是a是将额外的信息输出到匹配行的后面, i是将额外信息输出到匹配行的前面.
       
       p: 表示打印.
       1) sed '/north/p' datafile 将打印所有包含north的行.
       2) sed '3p' datafile    将打印第三行.
       3) sed '3,$p' datafile    将打印第三行到文件的结尾行.
       4) sed 'p' datafile 将打印所有行.
       注: 使用p的时候sed将会输出指定打印的行和所有行, 当其与-n选项组合时候,将只是打印输出匹配的行.
       
       n: 下一行命令.
       1) sed '/north/ {n; s/Chin/Joseph/}' datafile 将先定位包含north的行, 然后取其下一行作为目标行, 再在该目标行上执行s/Chin/Joseph/的替换操作.
       2) sed '/north/ {n; n; s/Chin/Joseph/}' datafile 将取north包含行的后两行作为目标行.
       注: {}作为嵌入的脚本执行.
       
       q: 退出命令.
       1) sed '5q' datafile 到第五行退出(输出第五行).
       2) sed '/north/q' datafile 输出到包含north的行退出(输出包含north的行).
       3) sed '/Lewis/ {s/Lewis/Joseph/; q}' datafile 将先定位包含Lewis的行, 然后用Joseph替换Lewis,最后退出sed操作.
       
       r: 文件读入.
       1) sed '/Suan/r newfile' datafile    在输出时,将newfile的文件内容跟随在datafile中包含Suan的行后面输出,如果多行都包含Suan,则文件被多次输出.
   
       s: 表示替换.
       1) sed 's/west/north/g' datafile    将所有west替换为north, g表示如果一行之内多次出现west,将全部替换, 如果没有g命令,将只是替换该行的第一个匹配.
       2) sed -n 's/^west/north/p' datafile    将所有以west开头的行替换为north, 同时只是输出替换匹配的行.
       3) sed -n '1,5 s/\(Mar\)got/\1ianne/p' datafile    将从第一行到第五行中所有的Margot替换为Marianne, \1是\(Mar\)的变量替代符.
       
       w: 文件写入.
       1) sed -n '/north/w newfile2' datafile    将datafile中所有包含north的行都写入到newfile2中.
       
       x: 互换命令.
       1) sed -e '/pat/h' -e '/Margot/x' datafile x命令表示当定位到包含Margot行,互换缓冲区和该匹配Margot行的数据, 即缓冲区中的数据替换该匹配行显示,
           该匹配行进入缓冲区, 如果在交换时缓冲区是空, 则该匹配行被换入缓冲区, 空行将替换该行显示, 后面依此类推. 如果交换后, 再次出现匹配pat的行, 该
           行将仍然会按照h命令的规则替换(不是交换, 交换只是发生在发现匹配Margot的时候)缓冲区中的数据.
   
       y: 变形命令.
       1) sed '1,3y/abcd/ABCD/' datafile 将1到3行中的小写abcd对应者替换为ABCD,注意abcd和ABCD是一一对应的. 如果他们的长度不匹配,sed将报错.
       2) sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' datafile 将datafile中所有的小写字符替换为大写字母.


四、awk家族:


1.    执行方式:
      1) awk 'pattern' filename 如awk '/Mary/' employees
      2) awk '{action}' filename 如awk '{print $1}' employees
      3) awk 'pattern {action}' filename 如awk '/Mary/ {print $1}' employees
      注: 模式/Mary/对action的作用范围是从其后面的第一个左花括号开始,到第一个右花括号结束. 其后的pattern将不会影响前面的action.
   
2.    内置变量:
       $0:    表示一整行(相当于数据库中一条记录).
       NR:    当前行号.
       NF:    当前记录的域(相当于数据库中的字段)数量
       RS:    行分隔符(缺省为回车).
       FS:    域分隔符,缺省为\t. awk -F: '{print $1,$2,$3}' employees 这里FS等于":".
       OFS:输出域分隔符, awk  -F: '{print $1,$2,$3}' employees 这里OFS等于" "空格, 因为在$1和$2之间是空格分开的.
       ARGC: 命令行参数的数量.
       ARGV: 命令行参数数组.
       ENVIRON: 从shell传递来的包含当前环境变量的数组.
       ERRNO: 错误号.
       FILENAME: 当前的输入文件名.
   
3.    格式化输出:
       转义码:
       \b:    Backspace.
       \n:    换行.
       \r:    回车.
       \t:    制表符.   
   
       格式化说明符:
       %c:    单个ASCII字符.
       %d:    十进制数字.
       %e:    科学记数法表示的数字.
       %f:    浮点数.
       %o:    八进制数字.
       %s:    打印字符串.
       %x:    十六进制数字.
       -:    表示左对齐,如%-15d, 在十进制数字的后面会有一些空格,同时该数字是左对齐的. %+15d或%15d表示右对齐,当数字不足15位的时候.
       #:    如%#o或%#x, 会在八进制的数字前面加入0,十六进制前加0x.

4.    操作符:
       ~:    匹配运算符. 如awk '$1~/Mary/' employees, 表示第一个域($1)中包含Mary的被打印, 如果其他域包含,第一个域没有,则仍然视为无效.
       !~:    不匹配运算符. 如awk '$1!~/Mary/' employees, 表示第一个域($1)中不包含Mary的被打印, 如果其他域包含,第一个域没有,则仍然视为有效.
       <,>,<=,>=,!=,==: 关系运算符. awk '$3>5000 {print $3}' datafile
       cond ? expr1 : expr2 条件表达式 awk '{max = $1 > $2 ? $1 : $2; print max}' datafile
       =,+=,-=,*=,/=,%=: 赋值运算符.
       -,+,*,/,%,^(x^y[乘方]): 数学运算符.
       &&, ||, !: 逻辑运算符.
       ,: 表示范围, awk '/Tom/,/Mary/' datafile 其规则可参照sed中逗号运算符.
   
5.    选项:
       -F:    指定特定的分隔符,而不是缺省的\t, 如-F:,这里分隔符是":".   

6.    awk编程:
       1) BEGIN: 其后紧跟着动作块, 该块将会在任何输入文件被读入之前执行, 如一些初始化工作, 或者打印一些输出标题.
       awk 'BEGIN{FS=":"; OFS="\t";ORS="\n\n"} {print $1,$2,$3}' file
       即使输入文件不存在, BEGIN块动作仍然会被执行.
       
       2) END: 其后也紧随动作块, 该动作模块将在整个输入文件处理完毕之后被处理, 但是END需要有文件名的输入.
       awk 'END {print "The end\n"} filename.
       
       3) 输入输出重新定向:
       awk 'BEGIN {print "Hello" > "newfile"}' datafile 文件名一定要用双引号扩起来, > 如果文件存在,则清空后重写新文件.
       awk 'BEGIN {print "Hello" >> "newfile"}' datafile 文件名一定要用双引号扩起来, > 如果文件存在, 则在文件末尾追加写入.
       awk 'BEGIN {getline name < "/dev/tty"; print name}' getline是awk的内置函数, 就像c语言的gets, 将输入赋值给name变量.
       
       4) system函数可以执行shell中的命令,这些命令必须用双引号扩起.
       awk 'END { system("clear"); system ("cat " FILENAME)}' filename
       
       5) 条件语句:
       if (expr) { stat; } else { stat; }
       if (expr) { stat; } else if { stat; } else { stat; }
       awk '{ if ($7 <= 2) { print "less than 2", $7 } else if ($7 <= 4) { print "less than 4", $7 } else { print "the others", $7 } }' datafile
       
       6) 循环语句:
       while (expr) { stat; }
       for (i = 1; i <= NF; i++) { stat; }
       break;
       continue;
       exit(exitcode);    awk 将退出. 退出后的$?将会是这里的exitcode.
       next; 读取下一条记录. awk '{ if ($7 == 3) { next } else { print $0 }}' datafile 将不会输出$7等于3的记录.
       
       7) 数组:
       awk的数组和pl/sql中数组有些类似, 都是通过哈希表来实现的,其下标可以是数字, 也可以是字符串.
       awk '{name[x++]=$3};END{for(i = 0; i < NR; i++) { print i, name[i]}}' employees
       awk '{id[NR]=$3};END{for (x = 1; x <= NR; x++) { print id[x]} }' employees
       awk '/^Tom/{name[NR]=$1}; END{for (i in name) { print name[i]}}' employees 特殊的for语句
       awk '/Tom/{count["tom"]++}; /Mary/{count["mary"]++}; END{print "count[tom] = ",count["tom"]; print "count[mary] = ", count["mary"]}' employees
       awk '{count[$2]++};END{for (name in count) {print name,count[name]}}' datafile 域变量也可以作为数组的下标.

7.    内置函数:
       1) sub/gsub(regexp, substitution string, [target string]); gsub和sub的差别是sub只是替换每条记录中第一个匹配正则的, gsub则替换该记录中所有匹配
       正则的, 就是vi中s/src/dest/ 和s/src/dest/g的区别, 如果target string没有输入, 其缺省值是$0.
       awk '{sub(/Tom/,"Thomas"); print}' employees
       awk '{sub(/Tom/,"Thomas",$1); print}' employees
       
       awk '{gsub(/Tom/,"Thomas"); print}' employees
       awk '{gsub(/Tom/,"Thomas",$1); print}' employees
       
       2) index(string ,substring) 返回子字符串第一次被匹配的位置(1开始)
       awk 'BEGIN{print index("hollow", "low") }'
       
       3) length(string) 返回字符串的长度.
       awk 'BEGIN{print length("hello")}'
       
       4) substr(string, starting position, [length])
       awk 'BEGIN{print substr("Santa Claus",7,6)}'
       awk 'BEGIN{print substr("Santa Claus",7)}'
       
       5) match(string, regexp) 返回正则表示在string中的位置, 没有定位返回0
       awk 'BEGIN{print match("Good ole USA",/[A-Z]+$/)}'

       6) toupper(string)和tolower(string) 仅仅gawk有效.
       awk 'BEGIN{print toupper("linux"), tolower("BASH")}'

       7) split(string, array, [field seperator]) 如果不输入field seperator, FS内置变量作为其缺省值.
       awk 'BEGIN{split("12/24/99",date,"/"); for (i in date) {print date[i]} }'
       
       8) variable = sprintf(format, ...) 和printf的最大区别就是他返回格式化后的字符串.
       awk '{line = sprintf("%-15s %6.2f ",$5,$6); print line}' datafile
       
       9) systime() 返回1970/1/1到当前时间的整秒数.
       
       10) variable = strftime(format, [timestamp])
       
       11) 数学函数: atan2(x,y), cos(x), exp(x)[求幂], int(x)[求整数], log(x), rand()[随机数], sin(x), sqrt(x), srand(x)
       
       
五、BASH SHELL编程:


1.    初始化顺序: /etc/profile    ( ~/.bash_profile | ~/.bash_login | ~/.profile )    ~/.bashrc

2.    set -o allexport 当前shell变量对其所有子shell都有效.
       set +o allexport 当前shell变量对其所有子shell都无效.
       set -o noclobber 重定向输出时,如果输出文件已经存在则提示输出失败, date > out; date > out, 第二次操作失败
       set +o noclobber 缺省shell行为. date > out; date > out, 第二次操作成功
       shopt -s extglob 使用扩展通配符,如 abc?(2|9)K, abc*([0-9]), abc+([0-9]), no@(thing|body), no!(thing|body)
       其中?,*,+,@和!都是用于修饰后面的()的.
   
3.    变量声明: declare, 在赋值的时候等号的两边不需要空格. variable=value.   
       declare -r variable=value 声明只读变量.
       declare -x variable=value 相当于export variable=value.
       数组声明:
       declare -a variable=(1 2 3 4)
       or
       name=(tom tim helen)
       or
       x[0]=5
       x[4]=10
       数组的声明可以不是连续的, 这一点和awk中的数组比较类似.
       e.g.
       /> declare -a friends
       /> friends=(sheryl peter louise)
       /> echo ${friends[0]}
       sheryl
       /> echo ${friends[1]}
       peter
       /> echo ${friends[2]}
       louise
       /> echo ${friends[*]}
       shery1 peter louise
       /> echo ${#friends[*]}
       3
       unset friends
   
       /> declare -a states=(ME [3]=CA [2]=CT)
       /> echo ${states[*]}
       ME CA CT
       /> echo ${#states[*]}
       3
       /> echo ${states[0]}
       ME
       /> echo ${states[1]}
   
       /> echo ${states[2]}
       CT
       /> echo ${states[3]}
       CA
       unset states
   
4.    函数声明:
       function greeting
       {
           echo "Hi $1 and $2";
       }
       /> greeting tom joe
       Hi tom and joe
       unset -f greeting
   
5.    printf其参数类型类似于awk的printf.

6.    变量扩展修改符:
       ${variable:+word}
       if (NULL != variable)
           echo word
       else
           echo $variable
       
       e.g.
       /> unset var_name
       /> var_name=
       /> echo ${var_name:+AA}
     
       /> var_name=BB
       /> echo ${var_name:+AA}
       AA
       /> echo $var_name
       BB

       ${variable:-word}
       if (NULL == variable)
          echo word
       else
          echo $variable

       e.g.
       /> unset var_name
       /> var_name=
       /> echo ${var_name:-AA}
       /> AA
       /> var_name=BB
       /> echo ${var_name:-AA}
       BB
       /> echo $var_name
       BB
     
       ${variable:=word}
       if (NULL == variable)
       {
           variable=word
           echo word
       }
       else
       {
           echo $variable
       }
   
       e.g.
       /> unset var_name
       /> echo ${var_name:=AA}
       AA
       /> echo $var_name
       AA
       /> echo ${var_name:=CC}
       AA
       /> echo $var_name
       AA

       ${variable:offset}/${variable:offset:length}

       e.g.
       /> var_name=notebook
       /> echo ${var_name:0:4}
       /> note
       /> echo ${var_name:4:4}
       /> book
       /> echo ${var_name:2}
       /> tebook
       /> echo ${var_name:0}
       /> notebook
   
       ${variable%pattern} 从variable尾部开始,最小化的删除pattern
       e.g.
       /> variable="/usr/bin/local/bin"
       /> echo ${variable%/bin*}
       /usr/bin/local
   
       ${variable%%pattern} 从variable尾部开始,最大化的删除pattern
       /> variable="/usr/bin/local/bin"
       /> echo ${variable%%/bin*}
       /usr

       ${variable#pattern} 从variable头部开始,最小化的删除pattern
       /> variable="/home/lilliput/jake/.bashrc
       /> echo ${variable#/home}
       /lilliput/jake/.bashrc
   
       ${variable##pattern} 从variable头部开始,最大化的删除pattern
       /> variable="/home/lilliput/jake/.bashrc
       /> echo ${variable##*/}
       .bashrc
   
       ${#pattern} 返回patter的字符数量.
       /> variable=abc123
       6
   
7.    引用:
       \: 可以史shell中的元字符无效, 如?, < >, \, $, *, [ ], |, ( ), ;, &, { }
       ': 单引号可以史其内的所有元字符无效, 也包括\.
       ": 双引号也可以史其内的所有元字符无效, 变量和命令替换除外. 如 echo "What's time? $(date)", 这里date命令将被执行.
   
8.    命令替换:
       variable=$(date) or variable=`date`
       variable=`basename \`pwd\`` or variable=$(basename $(pwd))    命令替换可以嵌套, 第一种方法中,嵌套的命令必须使用\进行转义.
       eval: 可以进行命令行求值操作.
       e.g.
       /> set a b c d
       /> echo The last argument is \$$#
       /> The last argument is $4
       /> eval echo The last argument is \$$#
       /> The last argument is d
   
9.    数学计算:
       /> echo $[5+4-2]
       7
       /> echo $[5+2*3]
       11
       /> echo $((5+4-2))
       7
       /> echo $((5+2*3))
       11
   
       /> declare -i num //必须声明-i, 以表示整型变量.
       /> num=5+5
       /> echo $num
       10                  //如果没有声明declare -i, 则返回5+5.
       /> num=5 + 5
       -bash: + 5: command not found.
       /> num="5 + 5"
       /> echo $num
       10
       /> num=4*6
       /> echo $num
       24
   
       使用不同进制(2~36)表示数字.
       /> declare -i x=017
       /> echo $x
       15
       /> x=2#101
       /> echo $x
       5
       /> x=8#17
       /> echo $x
       15
       /> x=16#b
       /> echo $x
       11
   
       let专门用于数学运算的bash内置命令.
       /> let i=5
       /> let i=i+1
       /> echo $i
       6
       /> let "i = i + 2"
       /> echo $i
       8
       /> let "i+=1"
       /> echo $i
       9
   
10.    读取用户输入(read)命令:
       /> read answer
       yes
       /> echo "$answer is the right response."
       yes is the right reponse.
   
       /> read first middle last
       Jon Jake Jones
       /> echo "Hello $first"
       Hello Jon
   
       /> read            //如果没有变量时, $REPLY是缺省变量.
       the Chico Nut factory
       /> echo "I guess $REPLY keeps you busy!"
       I guess the Chico Nut factory keeps you busy!
   
       /> read -p "Enter your job titile: "
       Enter your job title: Accountant
       /> echo "I thought you might be an $REPLY".
       I thought you might be an accountant.
   
       /> read -a friends
       Melvin Tim Ernest
       /> echo "Say hi to ${friends[2]}
       Say hi to Ernest.
       /> echo "Say hi to ${friends[$[${#friends[*]}-1]]}"
       Say hi to Ernest.
   
11.    条件结构和流控制:
        1) 条件判断方法: test, [ ], [[ ]], (( )), 当$?为0时表示成功和true, 否则失败和false.
        /> name=tom
        /> test $name != tom
        /> echo $?
        1    //Failure
       
        /> [ $name=Tom ]
        /> echo $?
        0
       
        /> [ $name = [Tt]?? ]  //[]和test不允许使用通配符.
        /> echo $?
        1
       
        /> x=5
        /> y=20
        /> [ $x -gt $y ]
        /> echo $?
        1
       
        /> [ $x -le $y ]
        /> echo $?
        0
       
        /> name=Tom
        /> friend=Joseph
        /> [[ $name == [Tt]om ]]
        /> echo $?
        0
       
        /> [[ $name == [Tt]om && $friend == "Jose" ]]
        /> echo $?
        1
       
        /> x=2
        /> y=3
        /> (( x > 2))
        /> echo $?
        1
       
        /> (( x < 2))
        /> echo $?
        1
       
        /> (( x == 2 && y == 3))
        /> echo $?
        0
       
       2) test命令操作符:
       字符串判断:
       [ string1 = string2 ]    or    两个字符串相等时返回true.
       [ string1 == string2 ]    
               
       [ string1 != string2 ]         两个字符串不等时返回true.
       [ string ]                          string非空时返回true.
       [ -z string ]                      为空时返回true.
       [ -n string ]                      为非空时返回true.
       
       逻辑判断:(cond1可以包含元字符)
       [ string1 -a string2 ] or     string1和string2都非空时
       [[ cond1 && cond2 ]]        cond1和cond2都为true
           
       [ string1 -o string2 ] or     string1或string2为非空时
       [[ cond1 || cond2 ]]          cond1或cond2为true.
           
       [ ! string ] or                    string为空
       [[ !cond ]]                        cond为false.
                   
       整数判断:
       [ int1 -eq int2 ]                int1等于int2
       [ int1 -ne int2 ]                int1不等于int2
       [ int1 -gt int2 ]                 int1大于int2
       [ int1 -ge int2 ]                int1大于等于int2
       [ int1 -lt int2 ]                  int1小于int2
       [ int1 -le int2 ]                 int1小于等于int2
       
       文件判断
       [ file1 -nt file2 ]               file1比file2新
       [ file1 -ot file2 ]               file1比file2旧
   
       文件检验:
       [ -d $file ] or                   表示判断目录是否存在
       [[ -d $file ]]
           
       [ -e $file ] or                   表示判断文件是否存在
       [[ -e $file ]]
           
       [ -f $file ] or                   表示判断非目录普通文件是否存在
       [[ -f $file ]]
           
       [ -s $file ] or                  表示判断文件存在, 并且非0.
       [[ -s $file ]]

       [ -L $file ] or                  表示判断为链接符号存在
       [[ -L $file ]]
           
       [ -r $file ] or                  表示判断文件存在, 并且可读.
       [[ -r $file ]]
           
       [ -w $file ] or                 表示判断文件存在, 并且可写.
       [[ -w $file ]]

       [ -x $file ] or                 表示判断文件存在, 并且可执行.
       [[ -x $file ]]

       3) if语句:
       if command     //当command的返回状态为0时执行then后面的语句.
       then
            command
       fi
       
       if test expr
       then
           command
       fi
       
       if [ string/numeric expr ]
       then
           command
       fi
       
       一下两种为new format, 建议使用.
       if [[ string expr ]]     //支持通配符和扩展通配符(shopt -s extglob)
       then
           command
       fi
       
       if (( numeric expr ))
       then
           command
       fi
      
       if command
       then
           command1
       else
           command2
       fi
       
       if command
       then
           command1
       elif command
       then
           command2
       elif command
       then
           command3
       else
           command4
       fi
   
       4) 空语句:
       用冒号表示.
       if expr
       then
           :
       fi
       
       5) case语句:    //;;相当于c语言中break.
       case variable in
       value1 | value2)        // value1支持通配符和|作为or的关系
           command1
           ;;
       value3)
           command2
           ;;
       [vV]alue4)
           command3
           ;;
       *)
           command4
           ;;
       esac
      
       6) 循环语句:
       IFS 变量表示缺省的分隔符whitespace, tab, newline等. 可以自行修改该值, 但是建议再改之前赋值给其他变量作为备份, 有利于再恢复到缺省值.
       for variable in word_list
       do
           commands
       done
      
       e.g.
       for pal in Tom Dick Harry Joe
       do
           echo "Hi $pal"
       done
       echo "Out of loop"
       
       for file in testfile[1-5]
       do
           if [[ -f $file ]]
           then
               echo "$file exist"
           fi
       done
      
       for name in $*    // 等同于 for name
       do
           echo "Hi $name"
       done
       
       while command    //这里当条件为true的时候, 或者command的退出为0时执行循环体的命令.
       do
           commands
       done
       
       while (( $num < 10 ))
       do
           echo -n "$num"
           let num+=1
       done
       
       until !command    //这里当条件为false的时候, 或者command的退出为非0时执行循环体的命令.
       do
           commands
       done       
   
       break/continue:    功能等同于c语言中的break和continue.
       
       重定向循环和后台执行的循环:
       while command
       do
           command
       done > file    //重定向到文件
       
       while command
       do
           command
       done | command    //重定向到其他程序的管道
       
       while command
       do
           command
       done &        //后台执行.
       
       shift [n]命令: 用于把脚本的参数列表移出指定的位数, 不指定为左移一位, 一旦发生位移, 被移出的参数将永远被删除.
       e.g.
       
       [scriptname: doit]
       while (( $# > 0 ))
       do
           echo $*
           shift
       done
       
       /> doit a b c d e
       a b c d e
       b c d e
       c d e
       d e
       e
 
1.   find
       find pathname -options [-print -exec -ok]
       让我们来看看该命令的参数：
       pathname find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
       -print find命令将匹配的文件输出到标准输出。
       -exec find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {} \;，注意{}和\；之间的空格，同时两个{}之间没有空格,
       注意一定有分号结尾。
       0) -ok 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行
       find . -name "datafile" -ctime -1 -exec ls -l {} \; 找到文件名为datafile*, 同时创建实际为1天之内的文件, 然后显示他们的明细.
       find . -name "datafile" -ctime -1 -exec rm -f {} \; 找到文件名为datafile*, 同时创建实际为1天之内的文件, 然后删除他们.

       find . -name "datafile" -ctime -1 -ok ls -l {} \; 这两个例子和上面的唯一区别就是-ok会在每个文件被执行命令时提示用户, 更加安全.
       find . -name "datafile" -ctime -1 -ok rm -f {} \;

       1) find . -name   基于文件名查找,但是文件名的大小写敏感.   
       find . -name "datafile*"
   
       2) find . -iname  基于文件名查找,但是文件名的大小写不敏感.
       find . -iname "datafile*"
   
       3) find . -maxdepth 2 -name fred 找出文件名为fred,其中find搜索的目录深度为2(距当前目录), 其中当前目录被视为第一层.
       
       4) find . -perm 644 -maxdepth 3 -name "datafile*"  (表示权限为644的, 搜索的目录深度为3, 名字为datafile*的文件)
   
       5) find . -path "./rw" -prune -o -name "datafile*" 列出所有不在./rw及其子目录下文件名为datafile*的文件。
       find . -path "./dir*" 列出所有符合dir*的目录及其目录的文件.
       find . \( -path "./d1" -o -path "./d2" \) -prune -o -name "datafile*" 列出所有不在./d1和d2及其子目录下文件名为datafile*的文件。
   
       6) find . -user ydev 找出所有属主用户为ydev的文件。
       find . ! -user ydev 找出所有属主用户不为ydev的文件， 注意!和-user之间的空格。
   
       7) find . -nouser    找出所有没有属主用户的文件，换句话就是，主用户可能已经被删除。
   
       8) find . -group ydev 找出所有属主用户组为ydev的文件。
   
       9) find . -nogroup    找出所有没有属主用户组的文件，换句话就是，主用户组可能已经被删除。
   
       10) find . -mtime -3[+3] 找出修改数据时间在3日之内[之外]的文件。
       find . -mmin  -3[+3] 找出修改数据时间在3分钟之内[之外]的文件。
       find . -atime -3[+3] 找出访问时间在3日之内[之外]的文件。
       find . -amin  -3[+3] 找出访问时间在3分钟之内[之外]的文件。
       find . -ctime -3[+3] 找出修改状态时间在3日之内[之外]的文件。
       find . -cmin  -3[+3] 找出修改状态时间在3分钟之内[之外]的文件。
   
       11) find . -newer eldest_file ! -newer newest_file 找出文件的更改时间 between eldest_file and newest_file。
       find . -newer file     找出所有比file的更改时间更新的文件
       find . ! -newer file 找出所有比file的更改时间更老的文件
       
       12) find . -type d    找出文件类型为目录的文件。
       find . ! -type d  找出文件类型为非目录的文件。
       b - 块设备文件。
       d - 目录。
       c - 字符设备文件。
       p - 管道文件。
       l - 符号链接文件。
       f - 普通文件。
       
       13) find . -size [+/-]100[c/k/M/G] 表示文件的长度为等于[大于/小于]100块[字节/k/M/G]的文件。
   
       14) find . -empty 查找所有的空文件或者空目录.
   
       15) find . -type f | xargs grep "ABC"
       使用xargs和-exec的区别是, -exec可能会为每个搜索出的file,启动一个新的进程执行-exec的操作, 而xargs都是在一个进程内完成, 效率更高.
   
2.   crontab:
       文件格式如下(每个列之间是使用空格分开的):
       第1列分钟1～59
       第2列小时1～23（0表示子夜）
       第3列日1～31
       第4列月1～12
       第5列星期0～6（0表示星期天）
       第6列要运行的命令
   
       分 时 日 月 星期 要运行的命令
   
       30 21* * * /apps/bin/cleanup.sh
       上面的例子表示每晚的21:30运行/apps/bin目录下的cleanup.sh。
       45 4 1,10,22 * * /apps/bin/backup.sh
       上面的例子表示每月1、10、22日的4:45运行/apps/bin目录下的backup.sh。
       10 1 * * 6,0 /bin/find -name "core" -exec rm {} \;
       上面的例子表示每周六、周日的1:10运行一个find命令。
       0,30 18-23 * * * /apps/bin/dbcheck.sh
       上面的例子表示在每天18:00至23:00之间每隔30分钟运行/apps/bin目录下的dbcheck.sh。
       0 23 * * 6 /apps/bin/qtrend.sh
       上面的例子表示每星期六的11:00pm运行/apps/bin目录下的qtrend.sh。
   
       -u 用户名。
       -e 编辑crontab文件。
       -l 列出crontab文件中的内容。
       -r 删除crontab文件。
       系统将在/var/spool/cron/目录下自动保存名为<username>的cron执行脚本.
       cron是定时完成的任务, 在任务启动时,一般来讲都是重新启动一个新的SHELL, 因此当需要使用登录配置文件的信息,特别是环境变量时,是非常麻烦的.
       一般这种问题的使用方法如下:
       0 2 * * * ( su - USERNAME -c "export LANG=en_US; /home/oracle/yb2.5.1/apps/admin/1.sh"; ) > /tmp/1.log 2>&1
       如果打算执行多条语句, 他们之间应使用分号进行分割. 注: 以上语句必须在root的帐户下执行.
   
3.   nohup:
       nohup command &
       如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户之后继续运行相应的进程。
       Nohup就是不挂起的意思(no hang up)。
   
4.   cut:
       1) cut一般格式为：cut [options] file1 file2
       -c list 指定剪切字符数。
       -f field 指定剪切域数。
       -d 指定与空格和tab键不同的域分隔符。
       -c 用来指定剪切范围，如下所示：
       -c1,5-7 剪切第1个字符，然后是第5到第7个字符。
       -c2- 剪切第2个到最后一个字符
       -c-5 剪切最开始的到第5个字符
       -c1-50 剪切前50个字符。
       -f 格式与-c相同。
       -f1,5 剪切第1域，第5域。
       -f1,10-12 剪切第1域，第10域到第12域。
       2) 使用方式：
       cut -d: -f3 cut_test.txt (基于":"作为分隔符，同时返回field 3中的数据) *field从0开始计算。
       cut -d: -f1,3 cut_test.txt (基于":"作为分隔符，同时返回field 1和3中的数据)
       cut -d: -c1,5-10 cut_test.txt(返回第1个和第5-10个字符)
   
5.   sort:    
       1) 对文件内容进行排序，缺省分割符为空格，如果自定义需要使用-t选择，如-t:
       2) 使用分隔符分割后，第一个field为0，awk中为1
       3) 具体用法如下：
       sort -t: sort_test.txt(缺省基于第一个field进行排序，field之间的分隔符为":")
       sort -t: -r sort_test.txt(缺省基于第一个field进行倒序排序，field之间的分隔符为":")
       sort -t: +1 sort_test.txt(基于第二个field进行排序，field之间的分隔符为":")
       sort +3n sort_test.txt(基于第三个field进行排序，其中n选项提示是进行"数值型"排序)
       sort -u  sort_test.txt(去除文件中重复的行，同时基于整行进行排序)
       sort -o output_file -t: +1.2[n] sort_text.txt(基于第二个field,同时从该field的第二个字符开始，这里n的作用也是"数值型"排序,并将结果输出到output_file中)
       sort -t: -m +0 filename1 filename2(合并两个文件之后在基于第一个field排序)

6.   pgrep和pkill:

       查找和杀死指定的进程, 他们的选项和参数完全相同, 这里只是介绍pgrep
       /> sleep 100&
       1000
       /> sleep 100&
       1001
   
       /> pgrep sleep
       1000
       1001
       /> pgrep -d: sleep    # -d定义多个进程之间的分隔符, 如果不定义则使用newline
       1000:1001
       /> pgrep -n sleep    # -n表示如果该程序有多个进程,查找最新的.
       1001
       /> pgrep -o  sleep    # -o表示如果该程序有多个进程,查找最老的.
       1000   
       /> pgrep -G root,oracle sleep # -G 表示进程的group id在-G后面的组列表中的进程会被考虑
       1000
       1001
       /> pgrep -u root,oracle sleep # -u 表示进程的effetive user id在-u后面的组列表中的进程会被考虑
       1000
       1001
       /> pgrep -U root,oracle sleep # -U 表示进程的real user id在-u后面的组列表中的进程会被考虑
       1000
       1001
       /> pgrep -x sleep # -x 表示进程的名字必须完全匹配, 以上的例子均可以部分匹配
       1000
       1001
       /> pgrep -x sle
   
       /> pgrep -l sleep # -l 将不仅打印pid,也打印进程名
       1000 sleep
       1001 sleep
       /> pgrep -lf sleep # -f 一般与-l合用, 将打印进程的参数
       1000 sleep 100
       1001 sleep 100
   
       /> pgrep -f sleep -d, | xargs ps -fp
       UID        PID  PPID  C STIME TTY          TIME CMD
       root      1000  2138  0 06:11 pts/5    00:00:00 sleep 1000
       root      1001  2138  0 06:11 pts/5    00:00:00 sleep 1000

7.   fuser:
       fuser -m /dev    # 列出所有和/dev设备有染的进程pid.
       fuser testfile    # 列出和testfile有染的进程pid
       fuser -u testfile # 列出和testfile有染的进程pid和userid
       fuser -k testfile # 杀死和testfile有染的进程pid

8.   mount:

　　 如何在unix下面mount一个windows下面的共享目录
       mount -t smbfs -o username=USERNAME,password=PASSWORD //windowsIp/pub_directory  /mountpoint  
       /> mkdir -p /mnt/win32
       /> mount -o username=administrator,password=1234 //10.1.4.103/Mine /mnt/win32
       /> umount /mnt/win32        # 卸载该mount.

9.   netstat:

　　 -a 表示显示所有的状态
　　 -l 则只是显示listen状态的，缺省只是显示connected
　　 -p 显示应用程序的名字
　　 -n 显示ip、port和user等信息
　　 -t 只显示TCP的连接
　　 /> netstat -apnt
　　 /> netstat -lpnt      #如果只是显示监听端口的状态，可以使用该命令

10. tune2fs:

　　 调整ext2/ext3文件系统特性的工具

　　 -l 查看文件系统信息
　　 /> tune2fs -l /dev/sda1  #将会列出所有和该磁盘分区相关的数据信息，如Inode等。
　　 /> tune2fs -l /dev/sda1 | grep -i "block size"      #查看当前文件系统的块儿尺寸
　　 /> tune2fs -l /dev/sdb1 |grep -i "mount count"   #查看 mount count 挂载次数

11.  开启或关闭Linux(iptables)防火墙
      重启后永久性生效：
      /> chkconfig iptables on         #开启
      /> chkconfig iptables off         #关闭
    
      即时生效，重启后还原:
      /> service iptables start        #开启
      /> service iptables stop         #关闭  


12.  tar 分卷压缩和合并
      以每卷500M为例
      />tar cvzpf - somedir | split -d -b 500m    #tar分卷压缩
      />cat x* > mytarfile.tar.gz                      #tar多卷合并


13.  把man或info的信息存为文本文件
      /> man tcsh | col -b > tcsh.txt
      /> info tcsh -o tcsh.txt -s


14.  查看正在执行进程的线程数
      />ps -eo "args nlwp pid pcpu" 

 

15.  使用md5sum计算文件的md5
      /> md5sum test.c
      07af691360175a6808567e2b08a11724  test.c

      /> md5sum test.c > hashfile
      /> md5sum –c hashfile     # 验证hashfile中包含的md5值和对应的文件,在执行该命令时是否仍然匹配, 如果此时test.c被修改了,该命令将返回不匹配的警告.

 

16.  在ps命令中显示进程的完整的命令行参数
      />ps auwwx

17. chkconfig：

    1). 编辑chkconfig操作的Shell文件头。
    #!/bin/bash
    #
    # chkconfig: 2345 20 80
    # description: Starts and stops the Redis Server
    这个注释头非常重要，否则chkconfig命令无法识别。其中2345表示init启动的级别，即在2、3、4、5这四个级别中均启动该服务。20表示该脚本启动的优先级，80表示停止的优先级。这些可以在chkconfig的manpage中找到更为详细的说明。
    
    2). 编译Shell文件的内容：
    case "$1" in
    start)
        #TODO: 执行服务程序的启动逻辑。
        ;;
    stop)
        #TODO: 执行服务程序的停止逻辑。
        ;;
    restart)
        ;;
    reload)
        ;;
    condrestart)
        ;;
    status)
        ;;
    上面列出的case条件必不可少，如果确实没有就当做占位符放在那里即可，如上例。
    
    3). 添加和删除服务程序：
    #--add选项表示添加新的服务程序。
    /> chkconfig --add redis_6379
    #查看是否删除或添加成功
    /> chkconfig | grep redis_6379
    redis_6379      0:off   1:off   2:on    3:on    4:on    5:on    6:off
    #--del选项表示删除已有的服务程序。
    /> chkconfig --del redis_6379


一.    特殊文件: /dev/null和/dev/tty

    Linux系统提供了两个对Shell编程非常有用的特殊文件，/dev/null和/dev/tty。其中/dev/null将会丢掉所有写入它的数据，换句换说，当程序将数据写入到此文件时，会认为它已经成功完成写入数据的操作，但实际上什么事都没有做。如果你需要的是命令的退出状态，而非它的输出，此功能会非常有用，见如下Shell代码：
    /> vi test_dev_null.sh
    
    #!/bin/bash
    if grep hello TestFile > /dev/null
    then
        echo "Found"
    else
        echo "NOT Found"
    fi
    在vi中保存并退出后执行以下命令：
    /> chmod +x test_dev_null.sh  #使该文件成为可执行文件
    /> cat > TestFile
    hello my friend
    CTRL + D                             #退出命令行文件编辑状态
    /> ./test_dev_null.sh
    Found                                 #这里并没有输出grep命令的执行结果。
    将以上Shell脚本做如下修改：
    /> vi test_dev_null.sh
    
    #!/bin/bash
    if grep hello TestFile
    then
        echo "Found"
    else
        echo "NOT Found"
    fi
    在vi中保存退出后，再次执行该脚本：
    /> ./test_dev_null.sh
    hello my friend                      #grep命令的执行结果被输出了。
    Found
    
    下面我们再来看/dev/tty的用途。当程序打开此文件是，Linux会自动将它重定向到一个终端窗口，因此该文件对于读取人工输入时特别有用。见如下Shell代码：
    /> vi test_dev_tty.sh
    
    #!/bin/bash
    printf "Enter new password: "    #提示输入
    stty -echo                               #关闭自动打印输入字符的功能
    read password < /dev/tty         #读取密码
    printf "\nEnter again: "             #换行后提示再输入一次
    read password2 < /dev/tty       #再读取一次以确认
    printf "\n"                               #换行
    stty echo                                #记着打开自动打印输入字符的功能
    echo "Password = " $password #输出读入变量
    echo "Password2 = " $password2
    echo "All Done"

    在vi中保存并退出后执行以下命令：
    /> chmod +x test_dev_tty.sh #使该文件成为可执行文件
    /> ./test_dev_tty
    Enter new password:             #这里密码的输入被读入到脚本中的password变量
    Enter again:                          #这里密码的输入被读入到脚本中的password2变量
    Password = hello
    Password2 = hello
    All Done

二.    简单的命令跟踪:

    Linux Shell提供了两种方式来跟踪Shell脚本中的命令，以帮助我们准确的定位程序中存在的问题。下面的代码为第一种方式，该方式会将Shell脚本中所有被执行的命令打印到终端，并在命令前加"+"：加号的后面还跟着一个空格。
    /> cat > trace_all_command.sh
    who | wc -l                          #这两条Shell命令将输出当前Linux服务器登录的用户数量
    CTRL + D                            #退出命令行文件编辑状态
    /> chmod +x trace_all_command.sh
    /> sh -x ./trace_all_command.sh #Shell执行器的-x选项将打开脚本的执行跟踪功能。
    + wc -l                               #被跟踪的两条Shell命令
    + who
    2                                       #实际输出结果。
    Linux Shell提供的另一种方式可以只打印部分被执行的Shell命令，该方法在调试较为复杂的脚本时，显得尤为有用。
    /> cat > trace_patial_command.sh
    #! /bin/bash
    set -x                                #从该命令之后打开跟踪功能
    echo 1st echo                     #将被打印输出的Shell命令
    set +x                               #该Shell命令也将被打印输出，然而在该命令被执行之后，所有的命令将不再打印输出
    echo 2nd echo                    #该Shell命令将不再被打印输出。
    CTRL + D                           #退出命令行文件编辑状态
    /> chmod +x trace_patial_command.sh
    /> ./trace_patial_command.sh
    + echo 1st echo
    1st echo
    + set +x
    2nd echo
   
三.    正则表达式基本语法描述:

    Linux Shell环境下提供了两种正则表达式规则，一个是基本正则表达式(BRE)，另一个是扩展正则表达式(ERE)。
    下面是这两种表达式的语法列表，需要注意的是，如果没有明确指出的Meta字符，其将可同时用于BRE和ERE，否则将尽适用于指定的模式。
正则元字符 	模式含义 	用例
\ 	通常用于关闭其后续字符的特殊意义，恢复其原意。 	\(...\)，这里的括号仅仅表示括号。
. 	匹配任何单个字符。 	a.b，将匹配abb、acb等
* 	匹配它之前的0-n个的单个字符。 	a*b，将匹配ab、aab、aaab等。
^ 	匹配紧接着的正则表达式，在行的起始处。 	^ab，将匹配abc、abd等，但是不匹配cab。
$ 	匹配紧接着的正则表达式，在行的结尾处。 	ab$，将匹配ab、cab等，但是不匹配abc。
[...] 	方括号表达式，匹配其内部任何字符。其中-表示连续字符的范围，^符号置于方括号里第一个字符则有反向的含义，即匹配不在列表内(方括号)的任何字符。如果想让]和-表示其原意，需要将其放置在方括号的首字符位置，如[]ab]或[-ab]，如这两个字符同时存在，则将]放置在首字符位置，-放置在最尾部，如[]ab-]。 	[a-bA-Z0-9!]表示所有的大小写字母，数字和感叹号。[^abc]表示a、b、c之外的所有字符。[Tt]om，可以匹配Tom和tom。
\{n,m\} 	区间表达式，匹配在它前面的单个字符重复出现的次数区间，\{n\}表示重复n次；\{n,\}表示至少重复n次；\{n,m\}表示重复n到m次。 	ab\{2\}表示abb；ab\{2,\}表示abb、abbb等。ab\{2,4\}表示abb、abbb和abbbb。
\(...\) 	将圆括号之间的模式存储在特殊“保留空间”。最多可以将9个独立的子模式存储在单个模式中。匹配于子模式的文本，可以通过转义序列\1到\9，被重复使用在相同模式里。 	\(ab\).*\1表示ab组合出现两次，两次之间可存在任何数目的任何字符，如abcdab、abab等。
{n,m}(ERE) 	其功能等同于上面的\{n,m\}，只是不再写\转义符了。 	ab+匹配ab、abbb等，但是不匹配a。
+(ERE) 	和前面的星号相比，+匹配的是前面正则表达式的1-n个实例。 	 
?(ERE) 	匹配前面正则表达式的0个或1个。 	ab?仅匹配a或ab。
|(ERE) 	匹配于|符号前后的正则表达式。 	(ab|cd)匹配ab或cd。
[:alpha:] 	匹配字母字符。 	[[:alpha:]!]ab$匹配cab、dab和!ab。
[:alnum:] 	匹配字母和数字字符。 	[[:alnum:]]ab$匹配1ab、aab。
[:blank:] 	匹配空格(space)和Tab字符。 	[[:alnum:]]ab$匹配1ab、aab。
[:cntrl:] 	匹配控制字符。 	 
[:digit:] 	匹配数字字符。 	 
[:graph:] 	匹配非空格字符。 	 
[:lower:] 	匹配小写字母字符。 	 
[:upper:] 	匹配大写字母字符。 	 
[:punct:] 	匹配标点字符。 	 
[:space:] 	匹配空白(whitespace)字符。 	 
[:xdigit:] 	匹配十六进制数字。 	 
\w 	匹配任何字母和数字组成的字符，等同于[[:alnum:]_] 	 
\W 	匹配任何非字母和数字组成的字符，等同于[^[:alnum:]_] 	 
\<\> 	匹配单词的起始和结尾。 	\<read匹配readme，me\>匹配readme。


    下面的列表给出了Linux Shell中常用的工具或命令分别支持的正则表达式的类型。
  	grep 	sed 	vi 	egrep 	awk
BRE 	* 	* 	* 	  	 
ERE 	  	  	  	* 	*


四.    使用cut命令选定字段:

    cut命令是用来剪下文本文件里的数据，文本文件可以是字段类型或是字符类型。下面给出应用实例：
    /> cat /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    ... ...
    /> cut -d : -f 1,5 /etc/passwd     #-d后面的冒号表示字段之间的分隔符，-f表示取分割后的哪些字段
    root:root                                 #这里取出的是第一个和第五个字段。
    bin:bin
    daemon:daemon
    adm:adm
    ... ...
    /> cut -d: -f 3- /etc/passwd       #从第三个字段开始显示，直到最后一个字段。
    0:0:root:/root:/bin/bash
    1:1:bin:/bin:/sbin/nologin
    2:2:daemon:/sbin:/sbin/nologin
    3:4:adm:/var/adm:/sbin/nologin
    4:7:lp:/var/spool/lpd:/sbin/nologin
    ... ...    
    这里需要进一步说明的是，使用cut命令还可以剪切以字符数量为标量的部分字符，该功能通过-c选项实现，其不能与-d选项共存。
    /> cut -c 1-4 /etc/passwd          #取每行的前1-4个字符。
    /> cut -c-4 /etc/passwd            #取每行的前4个字符。
    root
    bin:
    daem
    adm:
    ... ...
    /> cut -c4- /etc/passwd            #取每行的第4个到最后字符。
    t:x:0:0:root:/root:/bin/bash
    :x:1:1:bin:/bin:/sbin/nologin
    mon:x:2:2:daemon:/sbin:/sbin/nologin
    :x:3:4:adm:/var/adm:/sbin/nologin
    ... ...
    /> cut -c1,4 /etc/passwd           #取每行的第一个和第四个字符。
    rt
    b:
    dm
    a:
    ... ...
    /> cut -c1-4,5 /etc/passwd        #取每行的1-4和第5个字符。
    root:
    bin:x
    daemo
    adm:x

五.    计算行数、字数以及字符数:

    Linux提供了一个简单的工具wc用于完成该功能，见如下用例：
    /> echo This is a test of the emergency broadcast system | wc
    1    9    49                              #1行，9个单词，49个字符
    /> echo Testing one two three | wc -c
    22                                         #22个字符
    /> echo Testing one two three | wc -l
    1                                           #1行
    /> echo Testing one two three | wc -w
    4                                           #4个单词
    /> wc /etc/passwd /etc/group    #计算两个文件里的数据。
    39   71  1933  /etc/passwd
    62   62  906    /etc/group
    101 133 2839  总用量

六.    提取开头或结尾数行:

    有时，你会需要从文本文件里把几行字，多半是靠近开头或结尾的几行提取出来。如查看工作日志等操作。Linux Shell提供head和tail两个命令来完成此项工作。见如下用例：
    /> head -n 5 /etc/passwd           #显示输入文件的前五行。
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    /> tail -n 5 /etc/passwd             #显示输入文件的最后五行。
    sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
    pulse:x:496:494:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
    gdm:x:42:42::/var/lib/gdm:/sbin/nologin
    stephen:x:500:500:stephen:/home/stephen:/bin/bash
    如果使用者想查看不间断增长的日志(如服务程序输出的)，可以使用tail的-f选项，这样可以让tail命令不会自动退出，必须通过CTRL+C命令强制退出，因此该选项不适合用于Shell脚本中，见如下用例：
    /> tail -f -n 5 my_server_log
    ... ...
    ^C                                         #CTRL+C退出到命令行提示符状态。
    


七.　grep家族:
    
   1.  grep退出状态：
    0: 表示成功；
    1: 表示在所提供的文件无法找到匹配的pattern；
    2: 表示参数中提供的文件不存在。
    见如下示例：
    /> grep 'root' /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    operator:x:11:0:operator:/root:/sbin/nologin
    /> echo $?
    0
    
    /> grep 'root1' /etc/passwd  #用户root1并不存在
    /> echo $?
    1
    
    /> grep 'root' /etc/passwd1  #这里的/etc/passwd1文件并不存在
    grep: /etc/passwd1: No such file or directory
    /> echo $?
    2
    
   2.  grep中应用正则表达式的实例：
    需要说明的是下面所涉及的正则表达式在上一篇中已经给出了详细的说明，因此在看下面例子的时候，可以与前一篇的正则说明部分结合着看。
    /> cat testfile
    northwest        NW      Charles Main           3.0     .98     3       34
    western           WE       Sharon Gray          5.3     .97     5       23
    southwest       SW       Lewis Dalsass         2.7     .8       2       18
    southern         SO       Suan Chin               5.1     .95     4       15
    southeast       SE        Patricia Hemenway    4.0     .7       4       17
    eastern           EA        TB Savage              4.4     .84     5       20
    northeast        NE        AM Main Jr.              5.1     .94     3       13
    north              NO       Margot Weber          4.5     .89     5       9
    central            CT        Ann Stephens          5.7     .94     5       13

   
    /> grep NW testfile     #打印出testfile中所有包含NW的行。
    northwest       NW      Charles Main        3.0     .98     3       34
    
    /> grep '^n' testfile   #打印出以n开头的行。
    northwest       NW      Charles Main        3.0     .98     3       34
    northeast        NE       AM Main Jr.          5.1     .94     3       13
    north              NO      Margot Weber      4.5     .89     5       9
    
    /> grep '4$' testfile   #打印出以4结尾的行。
    northwest       NW      Charles Main        3.0     .98     3       34
    
    /> grep '5\..' testfile #打印出第一个字符是5，后面跟着一个.字符，在后面是任意字符的行。
    western         WE      Sharon Gray         5.3     .97     5       23
    southern        SO      Suan Chin             5.1     .95     4       15
    northeast       NE      AM Main Jr.            5.1     .94     3       13
    central           CT      Ann Stephens        5.7     .94     5       13
    
    /> grep '\.5' testfile  #打印出所有包含.5的行。
    north           NO      Margot Weber        4.5     .89     5       9

    /> grep '^[we]' testfile #打印出所有以w或e开头的行。
    western         WE      Sharon Gray         5.3     .97     5       23
    eastern          EA      TB Savage            4.4     .84     5       20
    
    /> grep '[^0-9]' testfile #打印出所有不是以0-9开头的行。
    northwest       NW     Charles Main             3.0     .98      3       34
    western          WE      Sharon Gray             5.3     .97     5       23
    southwest       SW     Lewis Dalsass           2.7     .8       2       18
    southern         SO      Suan Chin                5.1     .95     4       15
    southeast        SE      Patricia Hemenway     4.0     .7      4       17
    eastern           EA      TB Savage                4.4     .84     5       20
    northeast        NE      AM Main Jr.                5.1     .94     3       13
    north              NO      Margot Weber           4.5     .89     5       9
    central            CT      Ann Stephens            5.7     .94     5       13

    /> grep '[A-Z][A-Z] [A-Z]' testfile #打印出所有包含前两个字符是大写字符，后面紧跟一个空格及一个大写字母的行。
    eastern          EA      TB Savage       4.4     .84     5       20
    northeast       NE      AM Main Jr.      5.1     .94     3       13
    注：在执行以上命令时，如果不能得到预期的结果，即grep忽略了大小写，导致这一问题的原因很可能是当前环境的本地化的设置问题。对于以上命令，如果我将当前语言设置为en_US的时候，它会打印出所有的行，当我将其修改为中文环境时，就能得到我现在的输出了。
    /> export LANG=zh_CN  #设置当前的语言环境为中文。
    /> export LANG=en_US  #设置当前的语言环境为美国。
    /> export LANG=en_Br  #设置当前的语言环境为英国。
    
    /> grep '[a-z]\{9\}' testfile #打印所有包含每个字符串至少有9个连续小写字符的字符串的行。
    northwest        NW      Charles Main          3.0     .98     3       34
    southwest       SW      Lewis Dalsass         2.7     .8       2       18
    southeast        SE      Patricia Hemenway   4.0     .7       4       17
    northeast        NE      AM Main Jr.              5.1     .94     3       13
    
    #第一个字符是3，紧跟着一个句点，然后是任意一个数字，然后是任意个任意字符，然后又是一个3，然后是制表符，然后又是一个3，需要说明的是，下面正则中的\1表示\(3\)。
    /> grep '\(3\)\.[0-9].*\1    *\1' testfile
    northwest       NW      Charles Main        3.0     .98     3       34
    
    /> grep '\<north' testfile    #打印所有以north开头的单词的行。
    northwest       NW      Charles Main          3.0     .98     3       34
    northeast        NE       AM Main Jr.            5.1     .94     3       13
    north              NO      Margot Weber        4.5     .89     5       9
    
    /> grep '\<north\>' testfile  #打印所有包含单词north的行。
    north           NO      Margot Weber        4.5     .89     5       9
    
    /> grep '^n\w*' testfile      #第一个字符是n，后面是任意字母或者数字。
    northwest       NW     Charles Main          3.0     .98     3       34
    northeast        NE      AM Main Jr.            5.1     .94     3       13
    north             NO      Margot Weber        4.5     .89     5       9
    
    3.  扩展grep(grep -E 或者 egrep)：
    使用扩展grep的主要好处是增加了额外的正则表达式元字符集。下面我们还是继续使用实例来演示扩展grep。
    /> egrep 'NW|EA' testfile     #打印所有包含NW或EA的行。如果不是使用egrep，而是grep，将不会有结果查出。
    northwest       NW      Charles Main        3.0     .98     3       34
    eastern         EA      TB Savage           4.4     .84     5       20
    
    /> grep 'NW\|EA' testfile     #对于标准grep，如果在扩展元字符前面加\，grep会自动启用扩展选项-E。
    northwest       NW      Charles Main        3.0     .98     3       34
    eastern           EA       TB Savage           4.4     .84     5       20
    
    /> egrep '3+' testfile
    /> grep -E '3+' testfile
    /> grep '3\+' testfile        #这3条命令将会打印出相同的结果，即所有包含一个或多个3的行。
    northwest       NW      Charles Main         3.0     .98     3       34
    western          WE      Sharon Gray         5.3     .97     5       23
    northeast        NE       AM Main Jr.           5.1     .94     3       13
    central            CT       Ann Stephens       5.7     .94     5       13
    
    /> egrep '2\.?[0-9]' testfile
    /> grep -E '2\.?[0-9]' testfile
    /> grep '2\.\?[0-9]' testfile #首先含有2字符，其后紧跟着0个或1个点，后面再是0和9之间的数字。
    western         WE       Sharon Gray          5.3     .97     5       23
    southwest      SW      Lewis Dalsass         2.7     .8      2       18
    eastern          EA       TB Savage             4.4     .84     5       20
    
    /> egrep '(no)+' testfile
    /> grep -E '(no)+' testfile
    /> grep '\(no\)\+' testfile   #3个命令返回相同结果，即打印一个或者多个连续的no的行。
    northwest       NW      Charles Main        3.0     .98     3       34
    northeast        NE       AM Main Jr.          5.1     .94     3       13
    north              NO      Margot Weber      4.5     .89     5       9
    
    /> grep -E '\w+\W+[ABC]' testfile #首先是一个或者多个字母，紧跟着一个或者多个非字母数字，最后一个是ABC中的一个。
    northwest       NW     Charles Main       3.0     .98     3       34
    southern        SO      Suan Chin           5.1     .95     4       15
    northeast       NE      AM Main Jr.          5.1     .94     3       13
    central           CT      Ann Stephens      5.7     .94     5       13
    
    /> egrep '[Ss](h|u)' testfile
    /> grep -E '[Ss](h|u)' testfile
    /> grep '[Ss]\(h\|u\)' testfile   #3个命令返回相同结果，即以S或s开头，紧跟着h或者u的行。
    western         WE      Sharon Gray       5.3     .97     5       23
    southern        SO      Suan Chin          5.1     .95     4       15
    
    /> egrep 'w(es)t.*\1' testfile    #west开头，其中es为\1的值，后面紧跟着任意数量的任意字符，最后还有一个es出现在该行。
    northwest       NW      Charles Main        3.0     .98     3       34
   

    4.  grep选项：
    这里先列出grep常用的命令行选项：
选项 	说明
-c 	只显示有多少行匹配，而不具体显示匹配的行。
-h 	不显示文件名。
-i 	在字符串比较的时候忽略大小写。
-l 	只显示包含匹配模板的行的文件名清单。
-L 	只显示不包含匹配模板的行的文件名清单。
-n 	在每一行前面打印改行在文件中的行数。
-v 	反向检索，只显示不匹配的行。
-w 	只显示完整单词的匹配。
-x 	只显示完整行的匹配。
-r/-R 	如果文件参数是目录，该选项将递归搜索该目录下的所有子目录和文件。

    /> grep -n '^south' testfile  #-n选项在每一个匹配行的前面打印行号。
    3:southwest     SW      Lewis Dalsass         2.7     .8      2       18
    4:southern       SO      Suan Chin               5.1     .95     4       15
    5:southeast      SE      Patricia Hemenway    4.0     .7      4       17

    /> grep -i 'pat' testfile     #-i选项关闭了大小写敏感。
    southeast       SE      Patricia Hemenway       4.0     .7      4       17

    /> grep -v 'Suan Chin' testfile #打印所有不包含Suan Chin的行。
    northwest       NW      Charles Main          3.0     .98     3       34
    western          WE      Sharon Gray           5.3     .97    5       23
    southwest       SW      Lewis Dalsass        2.7     .8      2       18
    southeast        SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA      TB Savage              4.4     .84     5       20
    northeast        NE      AM Main Jr.             5.1     .94     3       13
    north              NO      Margot Weber        4.5     .89     5       9
    central            CT      Ann Stephens         5.7     .94     5       13

    /> grep -l 'ss' testfile  #-l使得grep只打印匹配的文件名，而不打印匹配的行。
    testfile

    /> grep -c 'west' testfile #-c使得grep只打印有多少匹配模板的行。
    3

    /> grep -w 'north' testfile #-w只打印整个单词匹配的行。
    north           NO      Margot Weber    4.5     .89     5       9

    /> grep -C 2 Patricia testfile #打印匹配行及其上下各两行。
    southwest      SW     Lewis Dalsass         2.7     .8       2       18
    southern        SO      Suan Chin              5.1     .95     4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern          EA      TB Savage              4.4     .84     5       20
    northeast       NE      AM Main Jr.             5.1     .94     3       13

    /> grep -B 2 Patricia testfile #打印匹配行及其前两行。
    southwest      SW      Lewis Dalsass         2.7     .8      2       18
    southern        SO      Suan Chin               5.1     .95    4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17

    /> grep -A 2 Patricia testfile #打印匹配行及其后两行。
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA      TB Savage              4.4     .84     5       20
    northeast       NE       AM Main Jr.             5.1     .94     3       13


八.　流编辑器sed:

    sed一次处理一行文件并把输出送往屏幕。sed把当前处理的行存储在临时缓冲区中，称为模式空间(pattern space)。一旦sed完成对模式空间中的行的处理，模式空间中的行就被送往屏幕。行被处理完成之后，就被移出模式空间，程序接着读入下一行，处理，显示，移出......文件输入的最后一行被处理完以后sed结束。通过存储每一行在临时缓冲区，然后在缓冲区中操作该行，保证了原始文件不会被破坏。
    
    1.  sed的命令和选项：
命令 	功能描述
a\ 	 在当前行的后面加入一行或者文本。
c\ 	 用新的文本改变或者替代本行的文本。
d 	 从pattern space位置删除行。
i\ 	 在当前行的上面插入文本。
h 	 拷贝pattern space的内容到holding buffer(特殊缓冲区)。
H 	 追加pattern space的内容到holding buffer。
g 	 获得holding buffer中的内容，并替代当前pattern space中的文本。
G 	 获得holding buffer中的内容，并追加到当前pattern space的后面。
n 	 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。
p 	 打印pattern space中的行。
P 	 打印pattern space中的第一行。
q 	 退出sed。
w file 	 写并追加pattern space到file的末尾。
! 	 表示后面的命令对所有没有被选定的行发生作用。
s/re/string 	 用string替换正则表达式re。
= 	 打印当前行号码。
替换标记 	 
g 	 行内全面替换，如果没有g，只替换第一个匹配。
p 	 打印行。
x 	 互换pattern space和holding buffer中的文本。
y 	 把一个字符翻译为另一个字符(但是不能用于正则表达式)。
选项 	 
-e 	 允许多点编辑。
-n 	 取消默认输出。

     需要说明的是，sed中的正则和grep的基本相同，完全可以参照本系列的第一篇中的详细说明。

   2.  sed实例：
    /> cat testfile
    northwest       NW     Charles Main           3.0      .98      3       34
    western          WE      Sharon Gray           5.3      .97     5       23
    southwest       SW     Lewis Dalsass          2.7      .8      2       18
    southern         SO      Suan Chin               5.1     .95     4       15
    southeast       SE       Patricia Hemenway   4.0      .7      4       17
    eastern           EA      TB Savage               4.4     .84     5       20
    northeast        NE      AM Main Jr.              5.1     .94     3       13
    north              NO      Margot Weber         4.5     .89     5       9
    central            CT      Ann Stephens          5.7     .94     5       13

    /> sed '/north/p' testfile #如果模板north被找到，sed除了打印所有行之外，还有打印匹配行。
    northwest       NW      Charles Main           3.0     .98     3       34
    northwest       NW      Charles Main           3.0     .98     3       34
    western          WE      Sharon Gray           5.3     .97     5       23
    southwest      SW      Lewis Dalsass          2.7     .8       2       18
    southern        SO       Suan Chin               5.1     .95     4       15
    southeast       SE       Patricia Hemenway   4.0     .7       4       17
    eastern           EA      TB Savage               4.4     .84     5       20
    northeast        NE      AM Main Jr.              5.1     .94     3       13
    northeast        NE      AM Main Jr.              5.1     .94     3       13
    north              NO      Margot Weber         4.5     .89     5       9
    north              NO      Margot Weber         4.5     .89     5       9
    central            CT      Ann Stephens          5.7     .94     5       13

    #-n选项取消了sed的默认行为。在没有-n的时候，包含模板的行被打印两次，但是在使用-n的时候将只打印包含模板的行。
    /> sed -n '/north/p' testfile
    northwest       NW      Charles Main    3.0     .98     3       34
    northeast        NE      AM Main Jr.       5.1     .94     3       13
    north              NO      Margot Weber  4.5     .89     5       9

    /> sed '3d' testfile  #第三行被删除，其他行默认输出到屏幕。
    northwest       NW     Charles Main            3.0     .98     3       34
    western          WE      Sharon Gray           5.3     .97     5       23
    southern         SO      Suan Chin               5.1     .95     4       15
    southeast       SE       Patricia Hemenway   4.0     .7       4       17
    eastern           EA      TB Savage               4.4     .84     5       20
    northeast       NE       AM Main Jr.              5.1     .94     3       13
    north             NO       Margot Weber         4.5     .89     5       9
    central           CT       Ann Stephens          5.7     .94     5       13

    /> sed '3,$d' testfile  #从第三行删除到最后一行，其他行被打印。$表示最后一行。
    northwest       NW      Charles Main    3.0     .98     3       34
    western          WE      Sharon Gray    5.3     .97     5       23

    /> sed '$d' testfile    #删除最后一行，其他行打印。
    northwest       NW     Charles Main           3.0     .98     3       34
    western          WE     Sharon Gray           5.3     .97     5       23
    southwest       SW    Lewis Dalsass          2.7     .8      2       18
    southern         SO     Suan Chin              5.1     .95     4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA      TB Savage             4.4     .84     5       20
    northeast       NE      AM Main Jr.             5.1     .94     3       13
    north             NO      Margot Weber        4.5     .89     5       9

    /> sed '/north/d' testfile #删除所有包含north的行，其他行打印。
    western           WE      Sharon Gray           5.3     .97     5       23
    southwest       SW      Lewis Dalsass          2.7     .8      2       18
    southern          SO      Suan Chin               5.1     .95     4       15
    southeast         SE      Patricia Hemenway   4.0     .7       4       17
    eastern            EA      TB Savage               4.4     .84     5       20
    central             CT      Ann Stephens          5.7     .94     5       13

    #s表示替换，g表示命令作用于整个当前行。如果该行存在多个west，都将被替换为north，如果没有g，则只是替换第一个匹配。
    /> sed 's/west/north/g' testfile
    northnorth      NW     Charles Main           3.0     .98    3       34
    northern         WE      Sharon Gray          5.3     .97    5       23
    southnorth      SW     Lewis Dalsass         2.7     .8      2       18
    southern         SO      Suan Chin              5.1     .95    4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA      TB Savage             4.4     .84     5       20
    northeast       NE      AM Main Jr.              5.1     .94    3       13
    north             NO      Margot Weber        4.5     .89     5       9
    central            CT      Ann Stephens        5.7     .94     5       13

    /> sed -n 's/^west/north/p' testfile #-n表示只打印匹配行，如果某一行的开头是west，则替换为north。
    northern        WE      Sharon Gray     5.3     .97     5       23

    #&符号表示替换字符串中被找到的部分。所有以两个数字结束的行，最后的数字都将被它们自己替换，同时追加.5。
    /> sed 's/[0-9][0-9]$/&.5/' testfile
    northwest       NW      Charles Main          3.0     .98     3       34.5
    western          WE      Sharon Gray           5.3     .97     5       23.5
    southwest       SW      Lewis Dalsass        2.7     .8       2       18.5
    southern         SO      Suan Chin              5.1     .95     4       15.5
    southeast       SE      Patricia Hemenway   4.0     .7       4       17.5
    eastern           EA      TB Savage              4.4     .84     5       20.5
    northeast        NE      AM Main Jr.             5.1     .94     3       13.5
    north              NO      Margot Weber        4.5     .89     5       9
    central            CT      Ann Stephens         5.7     .94     5       13.5

    /> sed -n 's/Hemenway/Jones/gp' testfile  #所有的Hemenway被替换为Jones。-n选项加p命令则表示只打印匹配行。
    southeast       SE      Patricia Jones  4.0     .7      4       17

    #模板Mar被包含在一对括号中，并在特殊的寄存器中保存为tag 1，它将在后面作为\1替换字符串，Margot被替换为Marlianne。
    /> sed -n 's/\(Mar\)got/\1lianne/p' testfile
    north           NO      Marlianne Weber 4.5     .89     5       9

    #s后面的字符一定是分隔搜索字符串和替换字符串的分隔符，默认为斜杠，但是在s命令使用的情况下可以改变。不论什么字符紧跟着s命令都认为是新的分隔符。这个技术在搜索含斜杠的模板时非常有用，例如搜索时间和路径的时候。
    /> sed 's#3#88#g' testfile
    northwest       NW      Charles Main            88.0    .98     88     884
    western          WE       Sharon Gray           5.88    .97     5       288
    southwest       SW      Lewis Dalsass          2.7     .8       2       18
    southern         SO       Suan Chin               5.1     .95     4       15
    southeast       SE        Patricia Hemenway   4.0     .7        4       17
    eastern           EA       TB Savage               4.4     .84      5      20
    northeast        NE       AM Main Jr.              5.1     .94      88     188
    north              NO       Margot Weber         4.5     .89      5       9
    central            CT       Ann Stephens          5.7     .94      5       188

    #所有在模板west和east所确定的范围内的行都被打印，如果west出现在esst后面的行中，从west开始到下一个east，无论这个east出现在哪里，二者之间的行都被打印，即使从west开始到文件的末尾还没有出现east，那么从west到末尾的所有行都将打印。
    /> sed -n '/west/,/east/p' testfile
    northwest       NW      Charles Main           3.0     .98      3      34
    western          WE      Sharon Gray            5.3     .97     5      23
    southwest       SW     Lewis Dalsass          2.7     .8       2      18
    southern         SO      Suan Chin               5.1     .95     4      15
    southeast        SE      Patricia Hemenway    4.0     .7       4      17

    /> sed -n '5,/^northeast/p' testfile  #打印从第五行开始到第一个以northeast开头的行之间的所有行。
    southeast       SE      Patricia Hemenway   4.0     .7       4       17
    eastern           EA      TB Savage              4.4     .84     5       20
    northeast        NE      AM Main Jr.             5.1     .94     3       13

    #-e选项表示多点编辑。第一个编辑命令是删除第一到第三行。第二个编辑命令是用Jones替换Hemenway。
    /> sed -e '1,3d' -e 's/Hemenway/Jones/' testfile
    southern        SO      Suan Chin          5.1     .95     4       15
    southeast       SE      Patricia Jones      4.0     .7      4       17
    eastern          EA      TB Savage          4.4     .84     5       20
    northeast       NE      AM Main Jr.         5.1     .94     3       13
    north             NO      Margot Weber    4.5     .89     5       9
    central           CT      Ann Stephens     5.7     .94     5       13

    /> sed -n '/north/w newfile' testfile #将所有匹配含有north的行写入newfile中。
    /> cat newfile
    northwest       NW      Charles Main     3.0     .98     3       34
    northeast       NE      AM Main Jr.         5.1     .94     3       13
    north             NO      Margot Weber    4.5     .89     5       9

    /> sed '/eastern/i\ NEW ENGLAND REGION' testfile #i是插入命令，在匹配模式行前插入文本。
    northwest       NW      Charles Main          3.0     .98      3       34
    western          WE      Sharon Gray           5.3     .97     5       23
    southwest       SW      Lewis Dalsass         2.7     .8      2       18
    southern         SO      Suan Chin              5.1     .95     4       15
    southeast        SE      Patricia Hemenway   4.0     .7      4       17
    NEW ENGLAND REGION
    eastern          EA      TB Savage              4.4     .84     5       20
    northeast       NE      AM Main Jr.             5.1     .94     3       13
    north             NO      Margot Weber        4.5     .89     5       9
    central           CT      Ann Stephens         5.7     .94     5       13

    #找到匹配模式eastern的行后，执行后面花括号中的一组命令，每个命令之间用逗号分隔，n表示定位到匹配行的下一行，s/AM/Archie/完成Archie到AM的替换，p和-n选项的合用，则只是打印作用到的行。
    /> sed -n '/eastern/{n;s/AM/Archie/;p}' testfile
    northeast       NE      Archie Main Jr. 5.1     .94     3       13

    #-e表示多点编辑，第一个编辑命令y将前三行中的所有小写字母替换为大写字母，-n表示不显示替换后的输出，第二个编辑命令将只是打印输出转换后的前三行。注意y不能用于正则。
    /> sed -n -e '1,3y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' -e '1,3p' testfile
    NORTHWEST       NW      CHARLES MAIN     3.0     .98     3       34
    WESTERN           WE      SHARON GRAY      5.3     .97     5       23
    SOUTHWEST       SW      LEWIS DALSASS   2.7     .8      2       18

    /> sed '2q' testfile  #打印完第二行后退出。
    northwest       NW      Charles Main    3.0     .98     3       34
    western          WE      Sharon Gray     5.3     .97     5       23

    #当模板Lewis在某一行被匹配，替换命令首先将Lewis替换为Joseph，然后再用q退出sed。
     /> sed '/Lewis/{s/Lewis/Joseph/;q;}' testfile
    northwest       NW      Charles Main      3.0     .98     3       34
    western          WE      Sharon Gray      5.3     .97     5       23
    southwest       SW      Joseph Dalsass  2.7     .8      2       18

    #在sed处理文件的时候，每一行都被保存在pattern space的临时缓冲区中。除非行被删除或者输出被取消，否则所有被处理过的行都将打印在屏幕上。接着pattern space被清空，并存入新的一行等待处理。在下面的例子中，包含模板的northeast行被找到，并被放入pattern space中，h命令将其复制并存入一个称为holding buffer的特殊缓冲区内。在第二个sed编辑命令中，当达到最后一行后，G命令告诉sed从holding buffer中取得该行，然后把它放回到pattern space中，且追加到现在已经存在于模式空间的行的末尾。
     /> sed -e '/northeast/h' -e '$G' testfile
    northwest       NW     Charles Main            3.0    .98     3       34
    western          WE     Sharon Gray            5.3    .97     5       23
    southwest       SW    Lewis Dalsass          2.7     .8       2       18
    southern         SO     Suan Chin               5.1     .95     4       15
    southeast       SE      Patricia Hemenway   4.0     .7       4       17
    eastern           EA      TB Savage              4.4     .84     5       20
    northeast       NE      AM Main Jr.              5.1     .94     3       13
    north             NO      Margot Weber         4.5     .89     5       9
    central           CT      Ann Stephens          5.7     .94     5       13
    northeast       NE      AM Main Jr.              5.1     .94     3       13

    #如果模板WE在某一行被匹配，h命令将使得该行从pattern space中复制到holding buffer中，d命令在将该行删除，因此WE匹配行没有在原来的位置被输出。第二个命令搜索CT，一旦被找到，G命令将从holding buffer中取回行，并追加到当前pattern space的行末尾。简单的说，WE所在的行被移动并追加到包含CT行的后面。
    /> sed -e '/WE/{h;d;}' -e '/CT/{G;}' testfile
    northwest       NW    Charles Main           3.0     .98     3       34
    southwest       SW    Lewis Dalsass         2.7     .8      2       18
    southern         SO     Suan Chin              5.1     .95     4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA     TB Savage              4.4     .84     5       20
    northeast       NE      AM Main Jr.              5.1     .94     3       13
    north             NO      Margot Weber         4.5     .89     5       9
    central           CT      Ann Stephens          5.7     .94     5       13
    western         WE      Sharon Gray           5.3     .97     5       23

    #第一个命令将匹配northeast的行从pattern space复制到holding buffer，第二个命令在读取的文件的末尾时，g命令告诉sed从holding buffer中取得行，并把它放回到pattern space中，以替换已经存在于pattern space中的。简单说就是包含模板northeast的行被复制并覆盖了文件的末尾行。
    /> sed -e '/northeast/h' -e '$g' testfile
    northwest       NW     Charles Main          3.0     .98     3       34
    western          WE      Sharon Gray         5.3     .97      5       23
    southwest       SW     Lewis Dalsass        2.7     .8       2       18
    southern         SO      Suan Chin             5.1     .95     4       15
    southeast       SE      Patricia Hemenway   4.0     .7      4       17
    eastern           EA      TB Savage             4.4     .84     5       20
    northeast       NE      AM Main Jr.             5.1     .94     3       13
    north             NO      Margot Weber        4.5     .89     5       9
    northeast       NE      AM Main Jr.             5.1     .94     3       13

    #模板WE匹配的行被h命令复制到holding buffer，再被d命令删除。结果可以看出WE的原有位置没有输出。第二个编辑命令将找到匹配CT的行，g命令将取得holding buffer中的行，并覆盖当前pattern space中的行，即匹配CT的行。简单的说，任何包含模板northeast的行都将被复制，并覆盖包含CT的行。    
    /> sed -e '/WE/{h;d;}' -e '/CT/{g;}' testfile
    northwest       NW    Charles Main           3.0     .98      3      34
    southwest       SW    Lewis Dalsass         2.7     .8       2       18
    southern         SO     Suan Chin              5.1     .95      4      15
    southeast       SE      Patricia Hemenway   4.0     .7       4      17
    eastern          EA      TB Savage              4.4     .84      5      20
    northeast       NE      AM Main Jr.              5.1     .94     3      13
    north             NO      Margot Weber        4.5     .89      5      9
    western         WE      Sharon Gray           5.3     .97     5      23

    #第一个编辑中的h命令将匹配Patricia的行复制到holding buffer中，第二个编辑中的x命令，会将holding buffer中的文本考虑到pattern space中，而pattern space中的文本被复制到holding buffer中。因此在打印匹配Margot行的地方打印了holding buffer中的文本，即第一个命令中匹配Patricia的行文本，第三个编辑命令会将交互后的holding buffer中的文本在最后一行的后面打印出来。
     /> sed -e '/Patricia/h' -e '/Margot/x' -e '$G' testfile
    northwest       NW      Charles Main           3.0      .98      3       34
    western           WE      Sharon Gray           5.3     .97      5       23
    southwest       SW      Lewis Dalsass         2.7      .8       2       18
    southern         SO      Suan Chin               5.1      .95     4       15
    southeast       SE       Patricia Hemenway    4.0      .7       4       17
    eastern           EA      TB Savage               4.4      .84     5       20
    northeast       NE       AM Main Jr.               5.1     .94      3       13
    southeast       SE      Patricia Hemenway      4.0     .7       4       17
    central            CT      Ann Stephens            5.7     .94     5       13
    
    
九.  awk实用功能:

    和sed一样，awk也是逐行扫描文件的，从第一行到最后一行，寻找匹配特定模板的行，并在这些行上运行“选择”动作。如果一个模板没有指定动作，这些匹配的行就被显示在屏幕上。如果一个动作没有模板，所有被动作指定的行都被处理。
    
   1.  awk的基本格式：
    /> awk 'pattern' filename
    /> awk '{action}' filename
    /> awk 'pattern {action}' filename
    
    具体应用方式分别见如下三个用例：
    /> cat employees
    Tom Jones         4424    5/12/66         543354
    Mary Adams      5346    11/4/63         28765
    Sally Chang       1654    7/22/54         650000
    Billy Black         1683    9/23/44         336500

    /> awk '/Mary/' employees   #打印所有包含模板Mary的行。
    Mary Adams      5346    11/4/63         28765

    #打印文件中的第一个字段，这个域在每一行的开始，缺省由空格或其它分隔符。
    /> awk '{print $1}' employees
    Tom
    Mary
    Sally
    Billy
    
    /> awk '/Sally/{print $1, $2}' employees #打印包含模板Sally的行的第一、第二个域字段。
    Sally Chang
    
    2.  awk的格式输出：
    awk中同时提供了print和printf两种打印输出的函数，其中print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。下面给出基本的转码序列：
转码 	含义
\n 	换行
\r 	回车
\t 	制表符


    /> date | awk '{print "Month: " $2 "\nYear: ", $6}'
    Month: Oct
    Year:  2011

    /> awk '/Sally/{print "\t\tHave a nice day, " $1,$2 "\!"}' employees
                    Have a nice day, Sally Chang!

    在打印数字的时候你也许想控制数字的格式，我们通常用printf来完成这个功能。awk的特殊变量OFMT也可以在使用print函数的时候，控制数字的打印格式。它的默认值是"%.6g"----小数点后面6位将被打印。
    /> awk 'BEGIN { OFMT="%.2f"; print 1.2456789, 12E-2}'
    1.25  0.12

    现在我们介绍一下功能更为强大的printf函数，其用法和c语言中printf基本相似。下面我们给出awk中printf的格式化说明符列表：
格式化说明符 	功能 	示例 	结果
%c 	打印单个ASCII字符。 	printf("The character is %c.\n",x) 	The character is A.
%d 	打印十进制数。 	printf("The boy is %d years old.\n",y) 	The boy is 15 years old.
%e 	打印用科学记数法表示的数。 	printf("z is %e.\n",z) 	z is 2.3e+01.
%f 	打印浮点数。 	printf("z is %f.\n",z) 	z is 2.300000
%o 	打印八进制数。 	printf("y is %o.\n",y) 	y is 17.
%s 	打印字符串。 	printf("The name of the culprit is %s.\n",$1); 	The name of the culprit is Bob Smith.
%x 	打印十六进制数。 	printf("y is %x.\n",y) 	y is f.

    注：假设列表中的变脸值为x = A, y = 15, z = 2.3, $1 = "Bob Smith"

    /> echo "Linux" | awk '{printf "|%-15s|\n", $1}'  # %-15s表示保留15个字符的空间，同时左对齐。
    |Linux          |

    /> echo "Linux" | awk '{printf "|%15s|\n", $1}'   # %-15s表示保留15个字符的空间，同时右对齐。
    |          Linux|

    #%8d表示数字右对齐，保留8个字符的空间。
     /> awk '{printf "The name is %-15s ID is %8d\n", $1,$3}' employees
    The name is Tom             ID is     4424
    The name is Mary            ID is     5346
    The name is Sally            ID is     1654
    The name is Billy             ID is     1683

    3.  awk中的记录和域：
    awk中默认的记录分隔符是回车，保存在其内建变量ORS和RS中。$0变量是指整条记录。
    /> awk '{print $0}' employees #这等同于print的默认行为。
    Tom Jones        4424    5/12/66         543354
    Mary Adams      5346    11/4/63         28765
    Sally Chang       1654    7/22/54         650000
    Billy Black         1683    9/23/44         336500

    变量NR(Number of Record)，记录每条记录的编号。
    /> awk '{print NR, $0}' employees
    1 Tom Jones        4424    5/12/66         543354
    2 Mary Adams      5346    11/4/63         28765
    3 Sally Chang       1654    7/22/54         650000
    4 Billy Black         1683    9/23/44         336500

    变量NF(Number of Field)，记录当前记录有多少域。
    /> awk '{print $0,NF}' employees
    Tom Jones        4424    5/12/66          543354   5
    Mary Adams      5346    11/4/63         28765     5
    Sally Chang      1654    7/22/54          650000   5
    Billy Black        1683     9/23/44         336500    5

    #根据employees生成employees2。sed的用法可以参考上一篇blog。
    /> sed 's/[[:space:]]\+\([0-9]\)/:\1/g;w employees2' employees
    /> cat employees
    Tom Jones:4424:5/12/66:543354
    Mary Adams:5346:11/4/63:28765
    Sally Chang:1654:7/22/54:650000
    Billy Black:1683:9/23/44:336500

    /> awk -F: '/Tom Jones/{print $1,$2}' employees2  #这里-F选项后面的字符表示分隔符。
    Tom Jones 4424

    变量OFS(Output Field Seperator)表示输出字段间的分隔符，缺省是空格。
    />  awk -F: '{OFS = "?"};  /Tom/{print $1,$2 }' employees2 #在输出时，域字段间的分隔符已经是?(问号)了
    Tom Jones?4424

    对于awk而言，其模式部分将控制这动作部分的输入，只有符合模式条件的记录才可以交由动作部分基础处理，而模式部分不仅可以写成正则表达式(如上面的例子)，awk还支持条件表达式，如：
    /> awk '$3 < 4000 {print}' employees
    Sally Chang     1654    7/22/54         650000
    Billy Black       1683    9/23/44         336500

    在花括号内，用分号分隔的语句称为动作。如果模式在动作前面，模式将决定什么时候发出动作。动作可以是一个语句或是一组语句。语句之间用分号分隔，也可以用换行符，如：
    pattern { action statement; action statement; etc. } or
    pattern {
        action statement
        action statement
    }
    模式和动作一般是捆绑在一起的。需要注意的是，动作是花括号内的语句。模式控制的动作是从第一个左花括号开始到第一个右花括号结束，如下：
    /> awk '$3 < 4000 && /Sally/ {print}' employees
    Sally Chang     1654    7/22/54         650000

    4.  匹配操作符：
    " ~ " 用来在记录或者域内匹配正则表达式。
    /> awk '$1 ~ /[Bb]ill/' employees      #显示所有第一个域匹配Bill或bill的行。
    Billy Black     1683    9/23/44         336500

    /> awk '$1 !~ /[Bb]ill/' employees     #显示所有第一个域不匹配Bill或bill的行，其中!~表示不匹配的意思。
    Tom Jones        4424    5/12/66         543354
    Mary Adams      5346    11/4/63         28765
    Sally Chang       1654    7/22/54         650000

    5.  awk的基本应用实例：
    /> cat testfile
    northwest     NW        Charles Main            3.0        .98        3        34
    western        WE        Sharon Gray            5.3        .97        5        23
    southwest     SW        Lewis Dalsass          2.7        .8          2        18
    southern       SO        Suan Chin                5.1        .95        4        15
    southeast      SE        Patricia Hemenway    4.0        .7          4        17
    eastern         EA        TB Savage                4.4        .84        5        20
    northeast      NE        AM Main Jr.               5.1        .94        3        13
    north            NO        Margot Weber          4.5        .89        5        9
    central          CT        Ann Stephens           5.7        .94        5        13

    /> awk '/^north/' testfile            #打印所有以north开头的行。
    northwest      NW      Charles Main     3.0     .98     3       34
    northeast       NE      AM Main Jr.        5.1     .94     3       13
    north             NO      Margot Weber   4.5     .89     5       9

    /> awk '/^(no|so)/' testfile          #打印所有以so和no开头的行。
    northwest       NW      Charles Main                3.0     .98      3       34
    southwest       SW      Lewis Dalsass              2.7     .8       2       18
    southern         SO      Suan Chin                    5.1     .95     4       15
    southeast        SE      Patricia Hemenway        4.0     .7       4       17
    northeast        NE      AM Main Jr.                   5.1     .94     3       13
    north              NO      Margot Weber              4.5     .89     5       9

    /> awk '$5 ~ /\.[7-9]+/' testfile     #第五个域字段匹配包含.(点)，后面是7-9的数字。
    southwest       SW      Lewis Dalsass            2.7     .8      2       18
    central             CT      Ann Stephens            5.7     .94     5       13

    /> awk '$8 ~ /[0-9][0-9]$/{print $8}' testfile  #第八个域以两个数字结束的打印。
    34
    23
    18
    15
    17
    20
    13

十.  awk表达式功能:

    1.  比较表达式：
    比较表达式匹配那些只在条件为真时才运行的行。这些表达式利用关系运算符来比较数字和字符串。见如下awk支持的条件表达式列表：
运算符 	含义 	例子
< 	小于 	x < y
<= 	小于等于 	x <= y
== 	等于 	x == y
!= 	不等于 	x != y
>= 	大于等于 	x >= y
> 	大于 	x > y
~ 	匹配 	x ~ /y/
!~ 	不匹配 	x !~ /y/

    /> cat employees
    Tom Jones        4424    5/12/66         543354
    Mary Adams      5346    11/4/63         28765
    Sally Chang       1654    7/22/54         650000
    Billy Black         1683    9/23/44         336500

    /> awk '$3 == 5346' employees       #打印第三个域等于5346的行。
    Mary Adams      5346    11/4/63         28765

    /> awk '$3 > 5000 {print $1}' employees  #打印第三个域大于5000的行的第一个域字段。
    Mary

    /> awk '$2 ~ /Adam/' employess      #打印第二个域匹配Adam的行。
    Mary Adams      5346    11/4/63         28765

    2.  条件表达式：
    条件表达式使用两个符号--问号和冒号给表达式赋值： conditional expression1 ? expression2 : expressional3，其逻辑等同于C语言中的条件表达式。其对应的if/else语句如下：
    {
        if (expression1)
            expression2
        else
            expression3
    }
    /> cat testfile
    northwest     NW        Charles Main             3.0        .98        3        34
    western        WE        Sharon Gray             5.3        .97         5        23
    southwest     SW        Lewis Dalsass           2.7        .8          2        18
    southern       SO        Suan Chin                 5.1        .95        4        15
    southeast      SE        Patricia Hemenway     4.0        .7          4        17
    eastern         EA        TB Savage                 4.4        .84        5        20
    northeast      NE        AM Main Jr.                5.1       .94         3        13
    north            NO        Margot Weber           4.5       .89         5        9
    central          CT        Ann Stephens            5.7       .94         5        13

    /> awk 'NR <= 3 {print ($7 > 4 ? "high "$7 : "low "$7) }' testfile
    low 3
    high 5
    low 2

    3.  数学表达式：
    运算可以在模式内进行，其中awk将所有的运算都视为浮点运算，见如下列表：
运算符 	含义 	例子
+ 	加 	x + y
- 	减 	x - y
* 	乘 	x * y
/ 	除 	x / y
% 	取余 	x % y
^ 	乘方 	x ^ y

    /> awk '/southern/{print $5 + 10}' testfile  #如果记录包含正则表达式southern，第五个域就加10并打印。
    15.1

    /> awk '/southern/{print $8 /2 }' testfile   #如果记录包含正则表达式southern，第八个域除以2并打印。
    7.5

    4.  逻辑表达式：
    见如下列表：
运算符 	含义 	例子
&& 	逻辑与 	a && b
|| 	逻辑或 	a || b
! 	逻辑非 	!a

    /> awk '$8 > 10 && $8 < 17' testfile   #打印出第八个域的值大于10小于17的记录。
    southern        SO      Suan Chin               5.1     .95     4       15
    central            CT      Ann Stephens         5.7     .94     5       13

    #打印第二个域等于NW，或者第一个域匹配south的行的第一、第二个域。
    /> awk '$2 == "NW" || $1 ~ /south/ {print $1,$2}' testfile
    northwest  NW
    southwest  SW
    southern    SO
    southeast   SE

    /> awk '!($8 > 13) {print $8}' testfile  #打印第八个域字段不大于13的行的第八个域。
    3
    9
    13

    5.  范围模板：
    范围模板匹配从第一个模板的第一次出现到第二个模板的第一次出现，第一个模板的下一次出现到第一个模板的下一次出现等等。如果第一个模板匹配而第二个模板没有出现，awk就显示到文件末尾的所有行。
    /> awk '/^western/,/^eastern/ {print $1}' testfile #打印以western开头到eastern开头的记录的第一个域。
    western    WE
    southwest SW
    southern   SO
    southeast  SE
    eastern      EA    
    
    

    6.  赋值符号：
    #找到第三个域等于Ann的记录，然后给该域重新赋值为Christian，之后再打印输出该记录。
    /> awk '$3 == "Ann" { $3 = "Christian"; print}' testfile
    central CT Christian Stephens 5.7 .94 5 13

    /> awk '/Ann/{$8 += 12; print $8}' testfile #找到包含Ann的记录，并将该条记录的第八个域的值+=12，最后再打印输出。
    25
    
十一.  awk编程:

    1.  变量：
    在awk中变量无须定义即可使用，变量在赋值时即已经完成了定义。变量的类型可以是数字、字符串。根据使用的不同，未初始化变量的值为0或空白字符串" "，这主要取决于变量应用的上下文。下面为变量的赋值负号列表：
符号 	含义 	等价形式
= 	a = 5 	a = 5
+= 	a = a + 5 	a += 5
-= 	a = a - 5 	a -= 5
*= 	a = a * 5 	a *= 5
/= 	a = a / 5 	a /= 5
%= 	a = a % 5 	a %= 5
^= 	a = a ^ 5 	a ^= 5

    /> awk '$1 ~ /Tom/ {Wage = $2 * $3; print Wage}' filename
    该命令将从文件中读取，并查找第一个域字段匹配Tom的记录，再将其第二和第三个字段的乘积赋值给自定义的Wage变量，最后通过print命令将该变量打印输出。

    /> awk ' {$5 = 1000 * $3 / $2; print}' filename
    在上面的命令中，如果$5不存在，awk将计算表达式1000 * $3 / $2的值，并将其赋值给$5。如果第五个域存在，则用表达式覆盖$5原来的值。

    我们同样也可以在命令行中定义自定义的变量，用法如下：
    /> awk -F: -f awkscript month=4 year=2011 filename
    这里的month和year都是自定义变量，且分别被赋值为4和2000，在awk的脚本中这些变量将可以被直接使用，他们和脚本中定义的变量在使用上没有任何区别。

    除此之外，awk还提供了一组内建变量(变量名全部大写)，见如下列表：
变量名 	变量内容
ARGC 	命令行参数的数量。
ARGIND 	命令行正在处理的当前文件的AGV的索引。
ARGV 	命令行参数数组。
CONVFMT 	转换数字格式。
ENVIRON 	从shell中传递来的包含当前环境变量的数组。
ERRNO 	当使用close函数或者通过getline函数读取的时候，发生的重新定向错误的描述信息就保存在这个变量中。
FIELDWIDTHS 	在对记录进行固定域宽的分割时，可以替代FS的分隔符的列表。
FILENAME 	当前的输入文件名。
FNR 	当前文件的记录号。
FS 	输入分隔符，默认是空格。
IGNORECASE 	在正则表达式和字符串操作中关闭大小写敏感。
NF 	当前文件域的数量。
NR 	当前文件记录数。
OFMT 	数字输出格式。
OFS 	输出域分隔符。
ORS 	输出记录分隔符。
RLENGTH 	通过match函数匹配的字符串的长度。
RS 	输入记录分隔符。
RSTART 	通过match函数匹配的字符串的偏移量。
SUBSEP 	下标分隔符。

    /> cat employees2
    Tom Jones:4424:5/12/66:543354
    Mary Adams:5346:11/4/63:28765
    Sally Chang:1654:7/22/54:650000
    Mary Black:1683:9/23/44:336500

    /> awk -F: '{IGNORECASE = 1}; $1 == "mary adams" { print NR, $1, $2, $NF}' employees2
    2 Mary Adams 5346 28765
    /> awk -F: ' $1 == "mary adams" { print NR, $1, $2, $NF}' employees2
    没有输出结果。
    当IGNORECASE内置变量的值为非0时，表示在进行字符串操作和处理正则表达式时关闭大小写敏感。这里的"mary adams"将匹配文件中的"Mary Admams"记录。最后print打印出第一、第二和最后一个域。需要说明的是NF表示当前记录域的数量，因此$NF将表示最后一个域的值。

    awk在动作部分还提供了BEGIN块和END块。其中BEGIN动作块在awk处理任何输入文件行之前执行。事实上，BEGIN块可以在没有任何输入文件的条件下测试。因为在BEGIN块执行完毕以前awk将不读取任何输入文件。BEGIN块通常被用来改变内建变量的值，如OFS、RS或FS等。也可以用于初始化自定义变量值，或打印输出标题。
    /> awk 'BEGIN {FS = ":"; OFS = "\t"; ORS = "\n\n"} { print $1,$2,$3} filename
    上例中awk在处理文件之前，已经将域分隔符(FS)设置为冒号，输出文件域分隔符(OFS)设置为制表符，输出记录分隔符(ORS)被设置为两个换行符。BEGIN之后的动作模块中如果有多个语句，他们之间用分号分隔。
    和BEGIN恰恰相反，END模块中的动作是在整个文件处理完毕之后被执行的。
    /> awk 'END {print "The number of the records is " NR }' filename
    awk在处理输入文件之后，执行END模块中的动作，上例中NR的值是读入的最后一个记录的记录号。

    /> awk '/Mary/{count++} END{print "Mary was found " count " times." }' employees2
    Mary was found 2 times.

    /> awk '/Mary/{count++} END{print "Mary was found " count " times." }' employees2
    Mary was found 2 times.
    
    /> cat testfile
    northwest       NW      Charles Main                3.0     .98     3       34
    western          WE      Sharon Gray                5.3     .97     5       23
    southwest       SW      Lewis Dalsass              2.7     .8      2       18
    southern         SO      Suan Chin                   5.1     .95     4       15
    southeast        SE      Patricia Hemenway        4.0     .7      4       17
    eastern           EA      TB Savage                   4.4     .84     5       20
    northeast        NE      AM Main Jr.                  5.1     .94     3       13
    north             NO       Margot Weber             4.5     .89     5       9
    central           CT       Ann Stephens              5.7     .94     5       13

    /> awk '/^north/{count += 1; print count}' testfile     #如记录以正则north开头，则创建变量count同时增一，再输出其值。
    1
    2
    3
    
    #这里只是输出前三个字段，其中第七个域先被赋值给变量x，在自减一，最后再同时打印出他们。
    /> awk 'NR <= 3 {x = $7--; print "x = " x ", $7 = " $7}' testfile
    x = 3, $7 = 2
    x = 5, $7 = 4
    x = 2, $7 = 1    
    
    #打印NR(记录号)的值在2--5之间的记录。
    /> awk 'NR == 2,NR == 5 {print "The record number is " NR}' testfile
    The record number is 2
    The record number is 3
    The record number is 4
    The record number is 5

    #打印环境变量USER和HOME的值。环境变量的值由父进程shell传递给awk程序的。
    /> awk 'BEGIN { print ENVIRON["USER"],ENVIRON["HOME"]}'
    root /root
    
    #BEGIN块儿中对OFS内置变量重新赋值了，因此后面的输出域分隔符改为了\t。
    /> awk 'BEGIN { OFS = "\t"}; /^Sharon/{ print $1,$2,$7}' testfile
    western WE      5
    
    #从输入文件中找到以north开头的记录count就加一，最后在END块中输出该变量。
    /> awk '/^north/{count++}; END{print count}' testfile
    3

    2.  重新定向：
    在动作语句中使用shell通用的重定向输出符号">"就可以完成awk的重定向操作，当使用>的时候，原有文件将被清空，同时文件持续打开，直到文件被明确的关闭或者awk程序终止。来自后面的打印语句的输出会追加到前面内容的后面。符号">>"用来打开一个文件但是不清空原有文件的内容，重定向的输出只是被追加到这个文件的末尾。
    /> awk '$4 >= 70 {print $1,$2 > "passing_file"}' filename  #注意这里的文件名需要用双引号括起来。
    #通过两次cat的结果可以看出>和>>的区别。
    /> awk '/north/{print $1,$3,$4 > "districts" }' testfile
    /> cat districts
    northwest Joel Craig
    northeast TJ Nichols
    north Val Shultz
    /> awk '/south/{print $1,$3,$4 >> "districts" }' testfile
    /> cat districts
    northwest Joel Craig
    northeast TJ Nichols
    north Val Shultz
    southwest Chris Foster
    southern May Chin
    southeast Derek Jonhson

   
    awk中对于输入重定向是通过getline函数来完成的。getline函数的作用是从标准输入、管道或者当前正在处理的文件之外的其他输入文件获得输入。他负责从输入获得下一行的内容，并给NF、NR和FNR等内建变量赋值。如果得到一个记录，getline就返回1，如果达到文件末尾就返回0。如果出现错误，如打开文件失败，就返回-1。
    /> awk 'BEGIN { "date" | getline d; print d}'
    Tue Nov 15 15:31:42 CST 2011
    上例中的BEGIN动作模块中，先执行shell命令date，并通过管道输出给getline，然后再把输出赋值给自定义变量d并打印输出它。
    
    /> awk 'BEGIN { "date" | getline d; split(d,mon); print mon[2]}'
    Nov
    上例中date命令通过管道输出给getline并赋值给d变量，再通过内置函数split将d拆分为mon数组，最后print出mon数组的第二个元素。
    
    /> awk 'BEGIN { while("ls" | getline) print}'
    employees
    employees2
    testfile
    命令ls的输出传递给getline作为输入，循环的每个反复，getline都从ls的结果中读取一行输入，并把他打印到屏幕。
    
    /> awk 'BEGIN { printf "What is your name? "; \
        getline name < "/dev/tty"}\
        $1 ~ name {print "Found" name " on line ", NR "."}\
        END {print "See ya, " name "."}' employees2
    What is your name? Mary
    Found Mary on line  2.
    See ya, Mary.    
    上例先是打印出BEGIN块中的"What is your name? "，然后等待用户从/dev/tty输入，并将读入的数据赋值给name变量，之后再从输入文件中读取记录，并找到匹配输入变量的记录并打印出来，最后在END块中输出结尾信息。
    
    /> awk 'BEGIN { while(getline < "/etc/passwd" > 0) lc++; print lc}'
    32
    awk将逐行读取/etc/passwd文件中的内容，在达到文件末尾之前，计数器lc一直自增1，当到了末尾后打印lc的值。lc的值为/etc/passwd文件的行数。
    由于awk中同时打开的管道只有一个，那么在打开下一个管道之前必须关闭它，管道符号右边可以通过可以通过双引号关闭管道。如果不关闭，它将始终保持打开状态，直到awk退出。
    /> awk {print $1,$2,$3 | "sort -4 +1 -2 +0 -1"} END {close("sort -4 +1 -2 +0 -1") } filename
    上例中END模块中的close显示关闭了sort的管道，需要注意的是close中关闭的命令必须和当初打开时的完全匹配，否则END模块产生的输出会和以前的输出一起被sort分类。


    3.  条件语句：
    awk中的条件语句是从C语言中借鉴来的，见如下声明方式：
    if (expression) {
        statement;
        statement;
        ... ...
    }
    /> awk '{if ($6 > 50) print $1 "Too hign"}' filename
    /> awk '{if ($6 > 20 && $6 <= 50) { safe++; print "OK}}' filename

    if (expression) {
        statement;
    } else {
        statement2;
    }
    /> awk '{if ($6 > 50) print $1 " Too high"; else print "Range is OK" }' filename
    /> awk '{if ($6 > 50) { count++; print $3 } else { x = 5; print $5 }' filename

    if (expression) {
        statement1;
    } else if (expression1) {
        statement2;
    } else {
        statement3;
    }
    /> awk '{if ($6 > 50) print "$6 > 50" else if ($6 > 30) print "$6 > 30" else print "other"}' filename

   4.  循环语句：
    awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

    5.  流程控制语句：
    next语句是从文件中读取下一行，然后从头开始执行awk脚本。
    exit语句用于结束awk程序。它终止对记录的处理。但是不会略过END模块，如果exit()语句被赋值0--255之间的参数，如exit(1)，这个参数就被打印到命令行，以判断退出成功还是失败。

    6.  数组：
    因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。
    /> cat employees
    Tom Jones       4424    5/12/66         543354
    Mary Adams      5346    11/4/63         28765
    Sally Chang     1654    7/22/54         650000
    Billy Black     1683    9/23/44         336500

    /> awk '{name[x++] = $2}; END{for (i = 0; i < NR; i++) print i, name[i]}' employees    
    0 Jones
    1 Adams
    2 Chang
    3 Black
    在上例中，数组name的下标是变量x。awk初始化该变量的值为0，在每次使用后自增1，读取文件中的第二个域的值被依次赋值给name数组的各个元素。在END模块中，for循环遍历数组的值。因为下标是关键字，所以它不一定从0开始，可以从任何值开始。

    #这里是用内置变量NR作为数组的下标了。
    /> awk '{id[NR] = $3}; END {for (x = 1; x <= NR; x++) print id[x]}' employees
    4424
    5346
    1654
    1683

    awk中还提供了一种special for的循环，见如下声明：
    for (item in arrayname) {
        print arrayname[item]
    }

    /> cat db
    Tom Jones
    Mary Adams
    Sally Chang
    Billy Black
    Tom Savage
    Tom Chung
    Reggie Steel
    Tommy Tucker

    /> awk '/^Tom/{name[NR]=$1}; END {for(i = 1;i <= NR; i++) print name[i]}' db
    Tom



    Tom
    Tom

    Tommy
    从输出结果可以看出，只有匹配正则表达式的记录的第一个域被赋值给数组name的指定下标元素。因为用NR作为下标，所以数组的下标不可能是连续的，因此在END模块中用传统的for循环打印时，不存在的元素就打印空字符串了。下面我们看看用special for的方式会有什么样的输出。
    /> awk '/^Tom/{name[NR]=$1};END{for(i in name) print name[i]}' db
    Tom
    Tom
    Tommy
    Tom

    下面我们看一下用字符串作为下标的例子：(如果下标是字符串文字常量，则需要用双引号括起来)    
    /> cat testfile2
    tom
    mary
    sean
    tom
    mary
    mary
    bob
    mary
    alex
    /> awk '/tom/{count["tom"]++}; /mary/{count["mary"]++}; END{print "There are " count["tom"] \
        " Toms and " count["mary"] " Marys in the file."} testfile2
    There are 2 Toms and 4 Marys in the file.
    在上例中，count数组有两个元素，下标分别为tom和mary，每一个元素的初始值都是0，没有tom被匹配的时候，count["tom"]就会加一，count["mary"]在匹配mary的时候也同样如此。END模块中打印出存储在数组中的各个元素。

    /> awk '{count[$1]++}; END{for(name in count) printf "%-5s%d\n",name, count[name]}' testfile2
    mary 4
    tom  2
    alex 1
    bob  1
    sean 1
    在上例中，awk是以记录的域作为数组count的下标。

    /> awk '{count[$1]++; if (count[$1] > 1) name[$1]++}; END{print "The duplicates were "; for(i in name) print i}' testfile2
    The duplicates were
    mary
    tom
    在上例中，如count[$1]的元素值大于1的时候，也就是当名字出现多次的时候，一个新的数组name将被初始化，最后打印出那么数组中重复出现的名字下标。

    之前我们介绍的都是如何给数组添加新的元素，并赋予初值，现在我们需要介绍一下如何删除数组中已经存在的元素。要完成这一功能我们需要使用内置函数delete，见如下命令：
    /> awk '{count[$1]++}; \
        END{for(name in count) {\
                if (count[name] == 1)\
                    delete count[name];\
            } \
            for (name in count) \
                print name}' testfile2
    mary
    tom
    上例中的主要技巧来自END模块，先是变量count数组，如果数组中某个元素的值等于1，则删除该元素，这样等同于删除只出现一次的名字。最后用special for循环打印出数组中仍然存在的元素下标名称。

    最后我们来看一下如何使用命令行参数数组，见如下命令：
    /> awk 'BEGIN {for(i = 0; i < ARGC; i++) printf("argv[%d] is %s.\n",i,ARGV[i]); printf("The number of arguments, ARGC=%d\n",ARGC)}' testfile "Peter Pan" 12
    argv[0] is awk.
    argv[1] is testfile.
    argv[2] is Peter Pan.
    argv[3] is 12.
    The number of arguments, ARGC=4
    从输出结果可以看出，命令行参数数组ARGV是以0作为起始下标的，命令行的第一个参数为命令本身(awk)，这个使用方式和C语句main函数完全一致。

    /> awk 'BEGIN{name=ARGV[2]; print "ARGV[2] is " ARGV[2]}; $1 ~ name{print $0}' testfile2 "bob"    
    ARGV[2] is bob
    bob
    awk: (FILENAME=testfile2 FNR=9) fatal: cannot open file `bob' for reading (No such file or directory)
    先解释一下以上命令的含义，name变量被赋值为命令行的第三个参数，即bob，之后再在输入文件中找到匹配该变量值的记录，并打印出该记录。
    在输出的第二行报出了awk的处理错误信息，这主要是因为awk将bob视为输入文件来处理了，然而事实上这个文件并不存在，下面我们需要做进一步的处理来修正这个问题。
    /> awk 'BEGIN{name=ARGV[2]; print "ARGV[2] is " ARGV[2]; delete ARGV[2]}; $1 ~ name{print $0}' testfile2 "bob"    
    ARGV[2] is bob
    bob
    从输出结果中我们可以看到我们得到了我们想要的结果。需要注意的是delete函数的调用必要要在BEGIN模块中完成，因为这时awk还没有开始读取命令行参数中指定的文件。

    7.  内建函数：
    字符串函数
    sub(regular expression,substitution string);
    sub(regular expression,substitution string,target string);

    /> awk '{sub("Tom","Tommy"); print}' employees   #这里使用Tommy替换了Tom。
    Tommy Jones       4424    5/12/66         543354

    #当正则表达式Tom在第一个域中第一次被匹配后，他将被字符串"Tommy"替换，如果将sub函数的第三个参数改为$2，将不会有替换发生。
    /> awk '{sub("Tom","Tommy",$1); print}' employees
    Tommy Jones       4424    5/12/66         543354

    gsub(regular expression,substitution string);
    gsub(regular expression,substitution string,target string);
    和sub不同的是，如果第一个参数中正则表达式在记录中出现多次，那么gsub将完成多次替换，而sub只是替换第一次出现的。

    index(string,substring)
    该函数将返回第二个参数在第一个参数中出现的位置，偏移量从1开始。
    /> awk 'BEGIN{print index("hello","el")}'
    2

    length(string)
    该函数返回字符串的长度。
    /> awk 'BEGIN{print length("hello")}'
    5

    substr(string,starting position)
    substr(string,starting position,length of string)
    该函数返回第一个参数的子字符串，其截取起始位置为第二个参数(偏移量为1)，截取长度为第三个参数，如果没有该参数，则从第二个参数指定的位置起，直到string的末尾。
    />  awk 'BEGIN{name = substr("Hello World",2,3); print name}'
    ell

    match(string,regular expression)
    该函数返回在字符串中正则表达式位置的索引，如果找不到指定的正则表达式就返回0.match函数设置内置变量RSTART为字符串中子字符串的开始位置，RLENGTH为到字字符串末尾的字符个数。
    /> awk 'BEGIN{start=match("Good ole CHINA", /[A-Z]+$/); print start}'
    10
    上例中的正则表达式[A-Z]+$表示在字符串的末尾搜索连续的大写字母。在字符串"Good ole CHINA"的第10个位置找到字符串"CHINA"。

    /> awk 'BEGIN{start=match("Good ole CHINA", /[A-Z]+$/); print RSTART, RLENGTH}'
    10 5
    RSTART表示匹配时的起始索引，RLENGTH表示匹配的长度。

    /> awk 'BEGIN{string="Good ole CHINA";start=match(string, /[A-Z]+$/); print substr(string,RSTART, RLENGTH)}'
    CHINA
    这里将match、RSTART、RLENGTH和substr巧妙的结合起来了。

    toupper(string)
    tolower(string)
    以上两个函数分别返回参数字符串的大写和小写的形式。
    /> awk 'BEGIN {print toupper("hello"); print tolower("WORLD")}'
    HELLO
    world

    split(string,array,field seperator)
    split(string,array)
    该函数使用作为第三个参数的域分隔符把字符串分隔为一个数组。如果第三个参数没有提供，则使用当前默认的FS值。
    /> awk 'BEGIN{split("11/20/2011",date,"/"); print date[2]}'
    20

    variable = sprintf("string with format specifiers ",expr1,expr2,...)
    该函数和printf的差别等同于C语言中printf和sprintf的差别。前者将格式化后的结果输出到输出流，而后者输出到函数的返回值中。
    /> awk 'BEGIN{line = sprintf("%-15s %6.2f ", "hello",4.2); print line}'
    hello             4.20

    时间函数：
    systime()
    该函数返回当前时间距离1970年1月1日之间相差的秒数。
    /> awk 'BEGIN{print systime()}'
    1321369554

    strftime()
    时间格式化函数，其格式化规则等同于C语言中的strftime函数提供的规则，见以下列表：
数据格式 	含义
%a 	Abbreviated weekday name
%A 	Full weekday name
%b 	Abbreviated month name
%B 	Full month name
%c 	Date and time representation appropriate for locale
%d 	Day of month as decimal number (01 – 31)
%H 	Hour in 24-hour format (00 – 23)
%I 	Hour in 12-hour format (01 – 12)
%j 	Day of year as decimal number (001 – 366)
%m 	Month as decimal number (01 – 12)
%M 	Minute as decimal number (00 – 59)
%p 	Current locale's A.M./P.M. indicator for 12-hour clock
%S 	Second as decimal number (00 – 59)
%U 	Week of year as decimal number, with Sunday as first day of week (00 – 53)
%w 	Weekday as decimal number (0 – 6; Sunday is 0)
%W 	Week of year as decimal number, with Monday as first day of week (00 – 53)
%x 	Date representation for current locale
%X 	Time representation for current locale
%y 	Year without century, as decimal number (00 – 99)
%Y 	Year with century, as decimal number

    /> awk 'BEGIN{ print strftime("%D",systime())}'
    11/15/11
    /> awk 'BEGIN{ now = strftime("%T"); print now}'
    23:17:29

    内置数学函数：
名称 	返回值
atan2(x,y) 	y,x范围内的余切
cos(x) 	余弦函数
exp(x) 	求幂
int(x) 	取整
log(x) 	自然对数
sin(x) 	正弦函数
sqrt(x) 	平方根

    /> awk 'BEGIN{print 31/3}'
    10.3333
    /> awk 'BEGIN{print int(31/3)}'
    10

    自定义函数：
    自定义函数可以放在awk脚本的任何可以放置模板和动作的地方。
    function name(parameter1,parameter2,...) {
        statements
        return expression
    }
    给函数中本地变量传递值。只使用变量的拷贝。数组通过地址或者指针传递，所以可以在函数内部直接改变数组元素的值。函数内部使用的任何没有作为参数传递的变量都被看做是全局变量，也就是这些变量对于整个程序都是可见的。如果变量在函数中发生了变化，那么就是在整个程序中发生了改变。唯一向函数提供本地变量的办法就是把他们放在参数列表中，这些参数通常被放在列表的最后。如果函数调用没有提供正式的参数，那么参数就初始化为空。return语句通常就返回程序控制并向调用者返回一个值。
    /> cat grades
    20 10
    30 20
    40 30

    /> cat add.sc
    function add(first,second) {
            return first + second
    }
    { print add($1,$2) }

    /> awk -f add.sc grades
    30
    50
    70


十二.   行的排序命令sort:

  1.  sort命令行选项：
选项 	描述
-t 	字段之间的分隔符
-f 	基于字符排序时忽略大小写
-k 	定义排序的域字段，或者是基于域字段的部分数据进行排序
-m 	将已排序的输入文件，合并为一个排序后的输出数据流
-n 	以整数类型比较字段
-o outfile 	将输出写到指定的文件
-r 	倒置排序的顺序为由大到小，正常排序为由小到大
-u 	只有唯一的记录，丢弃所有具有相同键值的记录
-b 	忽略前面的空格


   2.  sort使用实例：
    提示：在下面的输出结果中红色标注的为第一排序字段，后面的依次为紫、绿。
    /> sed -n '1,5p' /etc/passwd > users
    /> cat users
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    #-t定义了冒号为域字段之间的分隔符，-k 2指定基于第二个字段正向排序(字段顺序从1开始)。
    /> sort -t':' -k 1 users
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash

    #还是以冒号为分隔符，这次是基于第三个域字段进行倒置排序。
    /> sort -t':' -k 3r users
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    bin:x:1:1:bin:/bin:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash

    #先以第六个域的第2个字符到第4个字符进行正向排序，在基于第一个域进行反向排序。
    /> sort -t':' -k 6.2,6.4 -k 1r users
    bin:x:1:1:bin:/bin:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin

    #先以第六个域的第2个字符到第4个字符进行正向排序，在基于第一个域进行正向排序。和上一个例子比，第4和第5行交换了位置。
    /> sort -t':' -k 6.2,6.4 -k 1 users
    bin:x:1:1:bin:/bin:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    #基于第一个域的第2个字符排序
    /> sort -t':' -k 1.2,1.2 users    
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    bin:x:1:1:bin:/bin:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    #基于第六个域的第2个字符到第4个字符进行正向排序，-u命令要求在排序时删除键值重复的行。
    /> sort -t':' -k 6.2,6.4 -u users
    bin:x:1:1:bin:/bin:/sbin/nologin
    root:x:0:0:root:/root:/bin/bash
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin

    /> cat /etc/passwd | wc -l  #计算该文件中文本的行数。
    39
    /> sed -n '35,$p' /etc/passwd > users2  #取最后5行并输出到users2中。
    /> cat users2
    sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
    pulse:x:496:494:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
    gdm:x:42:42::/var/lib/gdm:/sbin/nologin
    stephen:x:500:500:stephen:/home/stephen:/bin/bash

    #基于第3个域字段以文本的形式排序
    /> sort -t':' -k 3 users2
    mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
    gdm:x:42:42::/var/lib/gdm:/sbin/nologin
    pulse:x:496:494:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
    stephen:x:500:500:stephen:/home/stephen:/bin/bash
    sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin

    #基于第3个域字段以数字的形式排序
    /> sort -t':' -k 3n users2
    mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
    gdm:x:42:42::/var/lib/gdm:/sbin/nologin
    sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    pulse:x:496:494:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
    stephen:x:500:500:stephen:/home/stephen:/bin/bash

    #基于当前系统执行进程的owner名排序，并将排序的结果写入到result文件中
    /> ps -ef | sort -k 1 -o result

十三. 删除重复行的命令uniq:

    uniq有3个最为常用的选项，见如下列表：
选项 	命令描述
-c 	可在每个输出行之前加上该行重复的次数
-d 	仅显示重复的行
-u 	显示为重复的行

    /> cat testfile
    hello
    world
    friend
    hello
    world
    hello

    #直接删除未经排序的文件，将会发现没有任何行被删除
    /> uniq testfile  
    hello
    world
    friend
    hello
    world
    hello

    #排序之后删除了重复行，同时在行首位置输出该行重复的次数
    /> sort testfile | uniq -c  
    1 friend
    3 hello
    2 world

    #仅显示存在重复的行，并在行首显示该行重复的次数
    /> sort testfile | uniq -dc
    3 hello
    2 world

    #仅显示没有重复的行
    /> sort testfile | uniq -u
    friend  


十四. 文件压缩解压命令tar:

   1.  tar命令行选项
选项 	命令描述
-c 	建立压缩档案
-x 	解压
--delete 	从压缩包中删除已有文件，如果该文件在包中出现多次，该操作其将全部删除。
-t 	查看压缩包中的文件列表
-r 	向压缩归档文件末尾追加文件
-u 	更新原压缩包中的文件
-z 	压缩为gzip格式，或以gzip格式解压
-j 	压缩为bzip2格式，或以bzip2格式解压
-v 	显示压缩或解压的过程，该选项一般不适于后台操作
-f 	使用档案名字，这个参数是最后一个参数，后面只能接档案名。


    2.  tar使用实例：
    #将当前目录下所有文件压缩打包，需要说明的是很多人都习惯将tar工具压缩的文件的扩展名命名为.tar
    /> tar -cvf test.tar *
    -rw-r--r--. 1 root root   183 Nov 11 08:02 users
    -rw-r--r--. 1 root root   279 Nov 11 08:45 users2

    /> cp ../*.log .                  #从上一层目录新copy一个.log文件到当前目录。
    /> tar -rvf test.tar *.log     #将扩展名为.log的文件追加到test.tar包里。
    /> tar -tvf test.tar
    -rw-r--r-- root/root        183 2011-11-11 08:02 users
    -rw-r--r-- root/root        279 2011-11-11 08:45 users2
    -rw-r--r-- root/root     48217 2011-11-11 22:16 install.log

    /> touch install.log           #使原有的文件更新一下最新修改时间
    /> tar -uvf test.tar *.log    #重新将更新后的log文件更新到test.tar中
    /> tar -tvf test.tar             #从输出结果可以看出tar包中多出一个更新后install.log文件。
    -rw-r--r-- root/root         183 2011-11-11 08:02 users
    -rw-r--r-- root/root         279 2011-11-11 08:45 users2
    -rw-r--r-- root/root     48217 2011-11-11 22:16 install.log
    -rw-r--r-- root/root     48217 2011-11-11 22:20 install.log

    /> tar --delete install.log -f test.tar #基于上面的结果，从压缩包中删除install.log
    -rw-r--r-- root/root       183 2011-11-11 08:02 users
    -rw-r--r-- root/root       279 2011-11-11 08:45 users2

    /> rm -f users users2      #从当前目录将tar中的两个文件删除
    /> tar -xvf test.tar          #解压
    /> ls -l users*                 #仅列出users和users2的详细列表信息
    -rw-r--r--. 1 root root 183 Nov 11 08:02 users
    -rw-r--r--. 1 root root 279 Nov 11 08:45 users2

    #以gzip的格式压缩并打包，解压时也应该以同样的格式解压，需要说明的是以该格式压缩的包习惯在扩展名后加.gz
    /> tar -cvzf test.tar.gz *
    /> tar -tzvf test.tar.gz      #查看压缩包中文件列表时也要加z选项(gzip格式)
    -rw-r--r-- root/root     48217 2011-11-11 22:50 install.log
    -rw-r--r-- root/root         183 2011-11-11 08:02 users
    -rw-r--r-- root/root         279 2011-11-11 08:45 users2

    /> rm -f users users2 install.log
    /> tar -xzvf test.tar.gz     #以gzip的格式解压
    /> ls -l *.log users*
    -rw-r--r-- root/root     48217 2011-11-11 22:50 install.log
    -rw-r--r-- root/root         183 2011-11-11 08:02 users
    -rw-r--r-- root/root         279 2011-11-11 08:45 users2

    /> rm -f test.*                #删除当前目录下原有的压缩包文件
    #以bzip2的格式压缩并打包，解压时也应该以同样的格式解压，需要说明的是以该格式压缩的包习惯在扩展名后加.bz2
    /> tar -cvjf test.tar.bz2 *
    /> tar -tjvf test.tar.bz2    #查看压缩包中文件列表时也要加j选项(bzip2格式)
    -rw-r--r-- root/root     48217 2011-11-11 22:50 install.log
    -rw-r--r-- root/root         183 2011-11-11 08:02 users
    -rw-r--r-- root/root         279 2011-11-11 08:45 users2

    /> rm -f *.log user*
    /> tar -xjvf test.tar.bz2    #以bzip2的格式解压
    /> ls -l
    -rw-r--r--. 1 root root 48217 Nov 11 22:50 install.log
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root     183 Nov 11 08:02 users
    -rw-r--r--. 1 root root     279 Nov 11 08:45 users2

十五. 大文件拆分命令split:

    下面的列表中给出了该命令最为常用的几个命令行选项：
选项 	描述
-l 	指定行数，每多少分隔成一个文件，缺省值为1000行。
-b 	指定字节数，支持的单位为：k和m
-C 	与-b参数类似，但切割时尽量维持每行的完整性
-d 	生成文件的后缀为数字，如果不指定该选项，缺省为字母

    /> ls -l
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2

    /> split -b 5k test.tar.bz2     #以每文件5k的大小切割test.tar.bz2
    /> ls -l                                #查看切割后的结果，缺省情况下拆分后的文件名为以下形式。
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root   5120 Nov 11 23:34 xaa
    -rw-r--r--. 1 root root   5120 Nov 11 23:34 xab
    -rw-r--r--. 1 root root     290 Nov 11 23:34 xac

    /> rm -f x*                         #删除拆分后的小文件
    /> split -d -b 5k test.tar.bz2 #-d选项以后缀为数字的形式命名拆分后的小文件
    /> ls -l
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root   5120 Nov 11 23:36 x00
    -rw-r--r--. 1 root root   5120 Nov 11 23:36 x01
    -rw-r--r--. 1 root root     290 Nov 11 23:36 x02

    /> wc install.log -l             #计算该文件的行数
    /> split -l 300 install.log     #每300行拆分成一个小文件
    /> ls -l x*
    -rw-r--r--. 1 root root 11184 Nov 11 23:42 xaa
    -rw-r--r--. 1 root root 10805 Nov 11 23:42 xab
    -rw-r--r--. 1 root root 12340 Nov 11 23:42 xac
    -rw-r--r--. 1 root root 11783 Nov 11 23:42 xad
    -rw-r--r--. 1 root root   2105 Nov 11 23:42 xae
    
十六. 文件查找命令find:

    下面给出find命令的主要应用示例：
    /> ls -l     #列出当前目录下所包含的测试文件
    -rw-r--r--. 1 root root 48217 Nov 12 00:57 install.log
    -rw-r--r--. 1 root root      37 Nov 12 00:56 testfile.dat
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root     183 Nov 11 08:02 users
    -rw-r--r--. 1 root root     279 Nov 11 08:45 users2
    
    1. 按文件名查找：
    -name:  查找时文件名大小写敏感。
    -iname: 查找时文件名大小写不敏感。
    #该命令为find命令中最为常用的命令，即从当前目录中查找扩展名为.log的文件。需要说明的是，缺省情况下，find会从指定的目录搜索，并递归的搜索其子目录。
    /> find . -name "*.log"
     ./install.log
    /> find . -iname U*          #如果执行find . -name U*将不会找到匹配的文件
    users users2


    2. 按文件时间属性查找：
    -atime  -n[+n]: 找出文件访问时间在n日之内[之外]的文件。
    -ctime  -n[+n]: 找出文件更改时间在n日之内[之外]的文件。
    -mtime -n[+n]: 找出修改数据时间在n日之内[之外]的文件。
    -amin   -n[+n]: 找出文件访问时间在n分钟之内[之外]的文件。
    -cmin   -n[+n]: 找出文件更改时间在n分钟之内[之外]的文件。
    -mmin  -n[+n]: 找出修改数据时间在n分钟之内[之外]的文件。
    /> find -ctime -2        #找出距此时2天之内创建的文件
    .
    ./users2
    ./install.log
    ./testfile.dat
    ./users
    ./test.tar.bz2
    /> find -ctime +2        #找出距此时2天之前创建的文件
    没有找到                     #因为当前目录下所有文件都是2天之内创建的
    /> touch install.log     #手工更新install.log的最后访问时间，以便下面的find命令可以找出该文件
    /> find . -cmin  -3       #找出修改状态时间在3分钟之内的文件。
    install.log

    3. 基于找到的文件执行指定的操作：
    -exec: 对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {} \;，注意{}和\；之间的空格，同时两个{}之间没有空格
    -ok:   其主要功能和语法格式与-exec完全相同，唯一的差别是在于该选项更加安全，因为它会在每次执行shell命令之前均予以提示，只有在回答为y的时候，其后的shell命令才会被继续执行。需要说明的是，该选项不适用于自动化脚本，因为该提供可能会挂起整个自动化流程。
    #找出距此时2天之内创建的文件，同时基于find的结果，应用-exec之后的命令，即ls -l，从而可以直接显示出find找到文件的明显列表。
    /> find . -ctime -2 -exec ls -l {} \;
    -rw-r--r--. 1 root root      279 Nov 11 08:45 ./users2
    -rw-r--r--. 1 root root  48217 Nov 12 00:57 ./install.log
    -rw-r--r--. 1 root root        37 Nov 12 00:56 ./testfile.dat
    -rw-r--r--. 1 root root      183 Nov 11 08:02 ./users
    -rw-r--r--. 1 root root  10530 Nov 11 23:08 ./test.tar.bz2
    #找到文件名为*.log, 同时文件数据修改时间距此时为1天之内的文件。如果找到就删除他们。有的时候，这样的写法由于是在找到之后立刻删除，因此存在一定误删除的危险。
    /> ls
    install.log  testfile.dat  test.tar.bz2  users  users2
    /> find . -name "*.log" -mtime -1 -exec rm -f {} \;
    /> ls
    testfile.dat  test.tar.bz2  users  users2
    在控制台下，为了使上面的命令更加安全，我们可以使用-ok替换-exec，见如下示例：
    />  find . -name "*.dat" -mtime -1 -ok rm -f {} \;
    < rm ... ./testfile.dat > ? y    #对于该提示，如果回答y，找到的*.dat文件将被删除，这一点从下面的ls命令的结果可以看出。
    /> ls
    test.tar.bz2  users  users2

    4. 按文件所属的owner和group查找：
    -user:      查找owner属于-user选项后面指定用户的文件。
    ! -user:    查找owner不属于-user选项后面指定用户的文件。
    -group:   查找group属于-group选项后面指定组的文件。
    ! -group: 查找group不属于-group选项后面指定组的文件。
    /> ls -l                            #下面三个文件的owner均为root
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root     183 Nov 11 08:02 users
    -rw-r--r--. 1 root root     279 Nov 11 08:45 users2
    /> chown stephen users   #将users文件的owner从root改为stephen。
    /> ls -l
    -rw-r--r--. 1 root       root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 stephen root    183 Nov 11 08:02 users
    -rw-r--r--. 1 root       root     279 Nov 11 08:45 users2
    /> find . -user root          #搜索owner是root的文件
    .
    ./users2
    ./test.tar.bz2
    /> find . ! -user root        #搜索owner不是root的文件，注意!和-user之间要有空格。
    ./users
    /> ls -l                            #下面三个文件的所属组均为root
    -rw-r--r--. 1 root      root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 stephen root    183 Nov 11 08:02 users
    -rw-r--r--. 1 root      root    279 Nov 11 08:45 users2
    /> chgrp stephen users    #将users文件的所属组从root改为stephen
    /> ls -l
    -rw-r--r--. 1 root           root    10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 stephen stephen      183 Nov 11 08:02 users
    -rw-r--r--. 1 root            root       279 Nov 11 08:45 users2
    /> find . -group root        #搜索所属组是root的文件
    .
    ./users2
    ./test.tar.bz2
    /> find . ! -group root      #搜索所属组不是root的文件，注意!和-user之间要有空格。    
    ./users

    5. 按指定目录深度查找：
    -maxdepth: 后面的参数表示距当前目录指定的深度，其中1表示当前目录，2表示一级子目录，以此类推。在指定该选项后，find只是在找到指定深度后就不在递归其子目录了。下例中的深度为1，表示只是在当前子目录中搜索。如果没有设置该选项，find将递归当前目录下的所有子目录。
    /> mkdir subdir               #创建一个子目录，并在该子目录内创建一个文件
    /> cd subdir
    /> touch testfile
    /> cd ..
    #maxdepth后面的参数表示距当前目录指定的深度，其中1表示当前目录，2表示一级子目录，以此类推。在指定该选项后，find只是在找到指定深度后就不在递归其子目录了。下例中的深度为1，表示只是在当前子目录中搜索。如果没有设置该选项，find将递归当前目录下的所有子目录。
    /> find . -maxdepth 1 -name "*"
    .
    ./users2
    ./subdir
    ./users
    ./test.tar.bz2
    #搜索深度为子一级子目录，这里可以看出子目录下刚刚创建的testfile已经被找到
    /> find . -maxdepth 2 -name "*"  
    .
    ./users2
    ./subdir
    ./subdir/testfile
    ./users
    ./test.tar.bz2
   
    6. 排除指定子目录查找：
    -path pathname -prune:   避开指定子目录pathname查找。
    -path expression -prune:  避开表达中指定的一组pathname查找。
    需要说明的是，如果同时使用-depth选项，那么-prune将被find命令忽略。
    #为后面的示例创建需要避开的和不需要避开的子目录，并在这些子目录内均创建符合查找规则的文件。
    /> mkdir DontSearchPath  
    /> cd DontSearchPath
    /> touch datafile1
    /> cd ..
    /> mkdir DoSearchPath
    /> cd DoSearchPath
    /> touch datafile2
    /> cd ..
    /> touch datafile3
    #当前目录下，避开DontSearchPath子目录，搜索所有文件名为datafile*的文件。
    /> find . -path "./DontSearchPath" -prune -o -name "datafile*" -print
    ./DoSearchPath/datafile2
    ./datafile3
    #当前目录下，同时避开DontSearchPath和DoSearchPath两个子目录，搜索所有文件名为datafile*的文件。
    /> find . \( -path "./DontSearchPath" -o -path "./DoSearchPath" \) -prune -o -name "datafile*" -print
    ./datafile3
       
    7. 按文件权限属性查找：
    -perm mode:   文件权限正好符合mode(mode为文件权限的八进制表示)。
    -perm +mode: 文件权限部分符合mode。如命令参数为644(-rw-r--r--)，那么只要文件权限属性中有任何权限和644重叠，这样的文件均可以被选出。
    -perm -mode:  文件权限完全符合mode。如命令参数为644(-rw-r--r--)，当644中指定的权限已经被当前文件完全拥有，同时该文件还拥有额外的权限属性，这样的文件可被选出。
    /> ls -l
    -rw-r--r--. 1 root            root           0 Nov 12 10:02 datafile3
    -rw-r--r--. 1 root            root    10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 stephen stephen        183 Nov 11 08:02 users
    -rw-r--r--. 1 root            root        279 Nov 11 08:45 users2
    /> find . -perm 644      #查找所有文件权限正好为644(-rw-r--r--)的文件。
    ./users2
    ./datafile3
    ./users
    ./test.tar.bz2
    /> find . -perm 444      #当前目录下没有文件的权限属于等于444(均为644)。    
    /> find . -perm -444     #644所包含的权限完全覆盖444所表示的权限。
    .
    ./users2
    ./datafile3
    ./users
    ./test.tar.bz2
    /> find . -perm +111    #查找所有可执行的文件，该命令没有找到任何文件。
    /> chmod u+x users     #改变users文件的权限，添加owner的可执行权限，以便于下面的命令可以将其找出。
    /> find . -perm +111    
    .
    ./users    
   
    8. 按文件类型查找：
    -type：后面指定文件的类型。
    b - 块设备文件。
    d - 目录。
    c - 字符设备文件。
    p - 管道文件。
    l  - 符号链接文件。
    f  - 普通文件。
    /> mkdir subdir
    /> find . -type d      #在当前目录下，找出文件类型为目录的文件。
    ./subdir
　 /> find . ! -type d    #在当前目录下，找出文件类型不为目录的文件。
    ./users2
    ./datafile3
    ./users
    ./test.tar.bz2
    /> find . -type f       #在当前目录下，找出文件类型为文件的文件
    ./users2
    ./datafile3
    ./users
    ./test.tar.bz2
   
    9. 按文件大小查找：
    -size [+/-]100[c/k/M/G]: 表示文件的长度为等于[大于/小于]100块[字节/k/M/G]的文件。
    -empty: 查找空文件。
    /> find . -size +4k -exec ls -l {} \;  #查找文件大小大于4k的文件，同时打印出找到文件的明细
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 ./test.tar.bz2
    /> find . -size -4k -exec ls -l {} \;  #查找文件大小小于4k的文件。
    -rw-r--r--. 1 root            root 279 Nov 11 08:45 ./users2
    -rw-r--r--. 1 root             root    0 Nov 12 10:02 ./datafile3
    -rwxr--r--. 1 stephen stephen 183 Nov 11 08:02 ./users
    /> find . -size 183c -exec ls -l {} \; #查找文件大小等于183字节的文件。
    -rwxr--r--. 1 stephen stephen 183 Nov 11 08:02 ./users
    /> find . -empty  -type f -exec ls -l {} \;
    -rw-r--r--. 1 root root 0 Nov 12 10:02 ./datafile3
   
    10. 按更改时间比指定文件新或比文件旧的方式查找：
    -newer file1 ! file2： 查找文件的更改日期比file1新，但是比file2老的文件。
    /> ls -lrt   #以时间顺序(从早到晚)列出当前目录下所有文件的明细列表，以供后面的例子参考。
    -rwxr--r--. 1 stephen stephen   183 Nov 11 08:02 users1
    -rw-r--r--. 1 root           root    279 Nov 11 08:45 users2
    -rw-r--r--. 1 root           root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root           root        0 Nov 12 10:02 datafile3
    /> find . -newer users1     #查找文件更改日期比users1新的文件，从上面结果可以看出，其余文件均符合要求。
    ./users2
    ./datafile3
    ./test.tar.bz2
    /> find . ! -newer users2   #查找文件更改日期不比users1新的文件。
    ./users2
    ./users
    #查找文件更改日期比users2新，但是不比test.tar.bz2新的文件。
    /> find . -newer users2 ! -newer test.tar.bz2
    ./test.tar.bz2
   
    细心的读者可能发现，关于find的说明，在我之前的Blog中已经给出，这里之所以拿出一个小节再次讲述该命令主要是因为以下三点原因：
    1. find命令在Linux Shell中扮演着极为重要的角色；
    2. 为了保证本系列的完整性；
    3. 之前的Blog是我多年之前留下的总结笔记，多少有些粗糙，这里给出了更为详细的举例。


十七. xargs命令:

    该命令的主要功能是从输入中构建和执行shell命令。       
    在使用find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。  
    find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。  
    在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高；  
    而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。
    /> ls -l
    -rw-r--r--. 1 root root        0 Nov 12 10:02 datafile3
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rwxr--r--. 1 root root    183 Nov 11 08:02 users
    -rw-r--r--. 1 root root    279 Nov 11 08:45 users2
    #查找当前目录下的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件。
    /> find . -type f -print | xargs file
    ./users2:        ASCII text
    ./datafile3:      empty
    ./users:          ASCII text
    ./test.tar.bz2: bzip2 compressed data, block size = 900k
    #回收当前目录下所有普通文件的执行权限。
    /> find . -type f -print | xargs chmod a-x
    /> ls -l
    -rw-r--r--. 1 root root     0 Nov 12 10:02 datafile3
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root   183 Nov 11 08:02 users
    -rw-r--r--. 1 root root   279 Nov 11 08:45 users2
    #在当面目录下查找所有普通文件，并用grep命令在搜索到的文件中查找hostname这个词
    /> find . -type f -print | xargs grep "hostname"
    #在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中。
    /> find / -name "core" -print | xargs echo "" >/tmp/core.log 　

    /> pgrep mysql | xargs kill -9　　#直接杀掉mysql的进程
    [1]+  Killed                  mysql
    
十八.  和系统运行状况相关的Shell命令:

    1.  Linux的实时监测命令(watch):
    watch 是一个非常实用的命令，可以帮你实时监测一个命令的运行结果，省得一遍又一遍的手动运行。该命令最为常用的两个选项是-d和-n，其中-n表示间隔多少秒执行一次"command"，-d表示高亮发生变化的位置。下面列举几个在watch中常用的实时监视命令：
    /> watch -d -n 1 'who'   #每隔一秒执行一次who命令，以监视服务器当前用户登录的状况
    Every 1.0s: who       Sat Nov 12 12:37:18 2011
    
    stephen  tty1           2011-11-11 17:38 (:0)
    stephen  pts/0         2011-11-11 17:39 (:0.0)
    root       pts/1         2011-11-12 10:01 (192.168.149.1)
    root       pts/2         2011-11-12 11:41 (192.168.149.1)
    root       pts/3         2011-11-12 12:11 (192.168.149.1)
    stephen  pts/4         2011-11-12 12:22 (:0.0)
    此时通过其他Linux客户端工具以root的身份登录当前Linux服务器，再观察watch命令的运行变化。
    Every 1.0s: who       Sat Nov 12 12:41:09 2011
    
    stephen  tty1          2011-11-11 17:38 (:0)
    stephen  pts/0        2011-11-11 17:39 (:0.0)
    root       pts/1        2011-11-12 10:01 (192.168.149.1)
    root       pts/2        2011-11-12 11:41 (192.168.149.1)
    root       pts/3        2011-11-12 12:40 (192.168.149.1)
    stephen  pts/4        2011-11-12 12:22 (:0.0)
    root       pts/5        2011-11-12 12:41 (192.168.149.1)
    最后一行中被高亮的用户为新登录的root用户。此时按CTRL + C可以退出正在执行的watch监控进程。
   
    #watch可以同时运行多个命令，命令间用分号分隔。
    #以下命令监控磁盘的使用状况，以及当前目录下文件的变化状况，包括文件的新增、删除和文件修改日期的更新等。
    /> watch -d -n 1 'df -h; ls -l'
    Every 1.0s: df -h; ls -l     Sat Nov 12 12:55:00 2011
    
    Filesystem            Size  Used Avail Use% Mounted on
    /dev/sda1             5.8G  3.3G  2.2G  61% /
    tmpfs                 504M  420K  504M   1% /dev/shm
    total 20
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root   183 Nov 11 08:02 users
    -rw-r--r--. 1 root root   279 Nov 11 08:45 users2
    此时通过另一个Linux控制台窗口，在watch监视的目录下，如/home/stephen/test，执行下面的命令
    /> touch aa         #在执行该命令之后，另一个执行watch命令的控制台将有如下变化
    Every 1.0s: df -h; ls -l                                Sat Nov 12 12:57:08 2011
    
    Filesystem            Size  Used Avail Use% Mounted on
    /dev/sda1             5.8G  3.3G  2.2G  61% /
    tmpfs                 504M  420K  504M   1% /dev/shm
    total 20
    -rw-r--r--. 1 root root        0 Nov 12 12:56 aa
    -rw-r--r--. 1 root root        0 Nov 12 10:02 datafile3
    -rw-r--r--. 1 root root 10530 Nov 11 23:08 test.tar.bz2
    -rw-r--r--. 1 root root     183 Nov 11 08:02 users
    -rw-r--r--. 1 root root     279 Nov 11 08:45 users2
    其中黄色高亮的部分，为touch aa命令执行之后watch输出的高亮变化部分。

   
    2.  查看当前系统内存使用状况(free)：
    free命令有以下几个常用选项：
选项 	说明
-b 	以字节为单位显示数据。
-k 	以千字节(KB)为单位显示数据(缺省值)。
-m 	以兆(MB)为单位显示数据。
-s delay 	该选项将使free持续不断的刷新，每次刷新之间的间隔为delay指定的秒数，如果含有小数点，将精确到毫秒，如0.5为500毫秒，1为一秒。

    free命令输出的表格中包含以下几列：
列名 	说明
total 	总计物理内存的大小。
used 	已使用的内存数量。
free 	可用的内存数量。
Shared 	多个进程共享的内存总额。
Buffers/cached 	磁盘缓存的大小。


    见以下具体示例和输出说明：
    /> free -k
                        total         used          free     shared    buffers     cached
    Mem:       1031320     671776     359544          0      88796     352564
    -/+ buffers/cache:      230416     800904
    Swap:        204792              0     204792
    对于free命令的输出，我们只需关注红色高亮的输出行和绿色高亮的输出行，见如下具体解释：
    红色输出行：该行使从操作系统的角度来看待输出数据的，used(671776)表示内核(Kernel)+Applications+buffers+cached。free(359544)表示系统还有多少内存可供使用。
    绿色输出行：该行则是从应用程序的角度来看输出数据的。其free = 操作系统used + buffers + cached，既：
    800904 = 359544 + 88796 + 352564
    /> free -m
                      total        used        free      shared    buffers     cached
    Mem:          1007         656        351            0         86            344
    -/+ buffers/cache:        225        782
    Swap:          199             0        199
    /> free -k -s 1.5  #以千字节(KB)为单位显示数据，同时每隔1.5刷新输出一次，直到按CTRL+C退出
                      total        used       free     shared    buffers     cached
    Mem:          1007         655        351          0           86        344
    -/+ buffers/cache:        224        782
    Swap:          199             0        199

                      total        used       free     shared    buffers     cached
    Mem:          1007         655        351          0           86        344
    -/+ buffers/cache:        224        782
    Swap:          199             0        199

    3.  CPU的实时监控工具(mpstat)：
    该命令主要用于报告当前系统中所有CPU的实时运行状况。
    #该命令将每隔2秒输出一次CPU的当前运行状况信息，一共输出5次，如果没有第二个数字参数，mpstat将每隔两秒执行一次，直到按CTRL+C退出。
    /> mpstat 2 5  
    Linux 2.6.32-71.el6.i686 (Stephen-PC)   11/12/2011      _i686_  (1 CPU)

    04:03:00 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
    04:03:02 PM  all    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00   99.50
    04:03:04 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    04:03:06 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    04:03:08 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    04:03:10 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    Average:       all    0.00    0.00    0.10    0.00    0.00    0.00    0.00    0.00   99.90

    第一行的末尾给出了当前系统中CPU的数量。后面的表格中则输出了系统当前CPU的使用状况，以下为每列的含义：
列名 	说明
%user 	在internal时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
%nice 	在internal时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
%sys 	在internal时间段里，内核时间(%)       (system/total)*100
%iowait 	在internal时间段里，硬盘IO等待时间(%) (iowait/total)*100
%irq 	在internal时间段里，硬中断时间(%)     (irq/total)*100
%soft 	在internal时间段里，软中断时间(%)     (softirq/total)*100
%idle 	在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100

    计算公式：
    total_cur=user+system+nice+idle+iowait+irq+softirq
    total_pre=pre_user+ pre_system+ pre_nice+ pre_idle+ pre_iowait+ pre_irq+ pre_softirq
    user=user_cur – user_pre
    total=total_cur-total_pre
    其中_cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。    

    /> mpstat -P ALL 2 3  #-P ALL表示打印所有CPU的数据，这里也可以打印指定编号的CPU数据，如-P 0(CPU的编号是0开始的)
    Linux 2.6.32-71.el6.i686 (Stephen-PC)   11/12/2011      _i686_  (1 CPU)

    04:12:54 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
    04:12:56 PM    all      0.00      0.00     0.50    0.00      0.00    0.00    0.00      0.00     99.50
    04:12:56 PM      0     0.00      0.00     0.50    0.00      0.00    0.00    0.00      0.00     99.50

    04:12:56 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
    04:12:58 PM    all     0.00      0.00     0.00    0.00      0.00    0.00    0.00      0.00    100.00
    04:12:58 PM     0     0.00      0.00     0.00    0.00      0.00    0.00    0.00      0.00    100.00

    04:12:58 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
    04:13:00 PM    all      0.00     0.00    0.00    0.00      0.00    0.00     0.00      0.00    100.00
    04:13:00 PM     0      0.00     0.00    0.00    0.00      0.00    0.00     0.00      0.00    100.00

    Average:       CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
    Average:         all      0.00     0.00    0.17    0.00      0.00    0.00     0.00      0.00     99.83
    Average:          0      0.00     0.00    0.17    0.00      0.00    0.00     0.00      0.00     99.83

    4.  虚拟内存的实时监控工具(vmstat)：
    vmstat命令用来获得UNIX系统有关进程、虚存、页面交换空间及CPU活动的信息。这些信息反映了系统的负载情况。vmstat首次运行时显示自系统启动开始的各项统计信息，之后运行vmstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
    /> vmstat 1 3    #这是vmstat最为常用的方式，其含义为每隔1秒输出一条，一共输出3条后程序退出。
    procs  -----------memory----------   ---swap-- -----io---- --system-- -----cpu-----
     r  b   swpd      free      buff   cache   si   so     bi    bo     in   cs  us  sy id  wa st
     0  0        0 531760  67284 231212  108  0     0  260   111  148  1   5 86   8  0
     0  0        0 531752  67284 231212    0    0     0     0     33   57   0   1 99   0  0
     0  0        0 531752  67284 231212    0    0     0     0     40   73   0   0 100 0  0

    /> vmstat 1       #其含义为每隔1秒输出一条，直到按CTRL+C后退出。

    下面将给出输出表格中每一列的含义说明：
    有关进程的信息有：(procs)
    r:  在就绪状态等待的进程数。
    b: 在等待状态等待的进程数。   
    有关内存的信息有：(memory)
    swpd:  正在使用的swap大小，单位为KB。
    free:    空闲的内存空间。
    buff:    已使用的buff大小，对块设备的读写进行缓冲。
    cache: 已使用的cache大小，文件系统的cache。
    有关页面交换空间的信息有：(swap)
    si:  交换内存使用，由磁盘调入内存。
    so: 交换内存使用，由内存调入磁盘。 
    有关IO块设备的信息有：(io)
    bi:  从块设备读入的数据总量(读磁盘) (KB/s)
    bo: 写入到块设备的数据总理(写磁盘) (KB/s)  
    有关故障的信息有：(system)
    in: 在指定时间内的每秒中断次数。
    sy: 在指定时间内每秒系统调用次数。
    cs: 在指定时间内每秒上下文切换的次数。  
    有关CPU的信息有：(cpu)
    us:  在指定时间间隔内CPU在用户态的利用率。
    sy:  在指定时间间隔内CPU在核心态的利用率。
    id:  在指定时间间隔内CPU空闲时间比。
    wa: 在指定时间间隔内CPU因为等待I/O而空闲的时间比。  
    vmstat 可以用来确定一个系统的工作是受限于CPU还是受限于内存：如果CPU的sy和us值相加的百分比接近100%，或者运行队列(r)中等待的进程数总是不等于0，且经常大于4，同时id也经常小于40，则该系统受限于CPU；如果bi、bo的值总是不等于0，则该系统受限于内存。

    5.  设备IO负载的实时监控工具(iostat)：
    iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
    其中该命令中最为常用的使用方式如下：
    /> iostat -d 1 3    #仅显示设备的IO负载，其中每隔1秒刷新并输出结果一次，输出3次后iostat退出。
    Linux 2.6.32-71.el6.i686 (Stephen-PC)   11/16/2011      _i686_  (1 CPU)

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda                 5.35       258.39        26.19     538210      54560

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda                 0.00         0.00         0.00                  0          0

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda                 0.00         0.00         0.00                  0          0

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda                 0.00         0.00         0.00                  0          0
    /> iostat -d 1  #和上面的命令一样，也是每隔1秒刷新并输出一次，但是该命令将一直输出，直到按CTRL+C退出。
    下面将给出输出表格中每列的含义：
列名 	说明
Blk_read/s 	每秒块(扇区)读取的数量。
Blk_wrtn/s 	每秒块(扇区)写入的数量。
Blk_read 	总共块(扇区)读取的数量。
Blk_wrtn 	总共块(扇区)写入的数量。

    iostat还有一个比较常用的选项-x，该选项将用于显示和io相关的扩展数据。
    /> iostat -dx 1 3
    Device:  rrqm/s wrqm/s  r/s   w/s  rsec/s wsec/s avgrq-sz avgqu-sz   await  svctm  %util
    sda            5.27   1.31 2.82 1.14 189.49  19.50    52.75     0.53     133.04  10.74   4.26

    Device:  rrqm/s wrqm/s  r/s   w/s  rsec/s wsec/s avgrq-sz avgqu-sz   await  svctm  %util
    sda            0.00   0.00 0.00 0.00   0.00   0.00        0.00     0.00         0.00   0.00   0.00

    Device:  rrqm/s wrqm/s  r/s   w/s  rsec/s wsec/s avgrq-sz avgqu-sz   await  svctm  %util
    sda            0.00   0.00 0.00 0.00   0.00   0.00        0.00     0.00         0.00   0.00   0.00
    还可以在命令行参数中指定要监控的设备名，如：
    /> iostat -dx sda 1 3   #指定监控的设备名称为sda，该命令的输出结果和上面命令完全相同。

    下面给出扩展选项输出的表格中每列的含义：
列名 	说明
rrqm/s 	队列中每秒钟合并的读请求数量
wrqm/s 	队列中每秒钟合并的写请求数量
r/s 	每秒钟完成的读请求数量
w/s 	每秒钟完成的写请求数量
rsec/s 	每秒钟读取的扇区数量
wsec/s 	每秒钟写入的扇区数量
avgrq-sz 	平均请求扇区的大小
avgqu-sz 	平均请求队列的长度
await 	平均每次请求的等待时间
util 	设备的利用率

    下面是关键列的解释：
    util是设备的利用率。如果它接近100%，通常说明设备能力趋于饱和。
    await是平均每次请求的等待时间。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
    avgqu-sz是平均请求队列的长度。毫无疑问，队列长度越短越好。                 

     6.  当前运行进程的实时监控工具(pidstat)：
     pidstat主要用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等。pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
    在正常的使用，通常都是通过在命令行选项中指定待监控的pid，之后在通过其他具体的参数来监控与该pid相关系统资源信息。
选项 	说明
-l 	显示该进程和CPU相关的信息(command列中可以显示命令的完整路径名和命令的参数)。
-d 	显示该进程和设备IO相关的信息。
-r 	显示该进程和内存相关的信息。
-w 	显示该进程和任务时间片切换相关的信息。
-t 	显示在该进程内正在运行的线程相关的信息。
-p 	后面紧跟着带监控的进程id或ALL(表示所有进程)，如不指定该选项，将监控当前系统正在运行的所有进程。

    #监控pid为1(init)的进程的CPU资源使用情况，其中每隔3秒刷新并输出一次，3次后程序退出。
    /> pidstat -p 1 2 3 -l
    07:18:58 AM       PID    %usr %system  %guest    %CPU   CPU  Command
    07:18:59 AM         1    0.00    0.00    0.00    0.00     0  /sbin/init
    07:19:00 AM         1    0.00    0.00    0.00    0.00     0  /sbin/init
    07:19:01 AM         1    0.00    0.00    0.00    0.00     0  /sbin/init
    Average:               1    0.00    0.00    0.00    0.00     -  /sbin/init
    %usr：      该进程在用户态的CPU使用率。
    %system：该进程在内核态(系统级)的CPU使用率。
    %CPU：     该进程的总CPU使用率，如果在SMP环境下，该值将除以CPU的数量，以表示每CPU的数据。
    CPU:         该进程所依附的CPU编号(0表示第一个CPU)。

    #监控pid为1(init)的进程的设备IO资源负载情况，其中每隔2秒刷新并输出一次，3次后程序退出。
    /> pidstat -p 1 2 3 -d    
    07:24:49 AM       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
    07:24:51 AM         1      0.00      0.00      0.00  init
    07:24:53 AM         1      0.00      0.00      0.00  init
    07:24:55 AM         1      0.00      0.00      0.00  init
    Average:               1      0.00      0.00      0.00  init
    kB_rd/s:   该进程每秒的字节读取数量(KB)。
    kB_wr/s:   该进程每秒的字节写出数量(KB)。
    kB_ccwr/s: 该进程每秒取消磁盘写入的数量(KB)。

    #监控pid为1(init)的进程的内存使用情况，其中每隔2秒刷新并输出一次，3次后程序退出。
    /> pidstat -p 1 2 3 -r
    07:29:56 AM       PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
    07:29:58 AM         1      0.00      0.00    2828   1368   0.13  init
    07:30:00 AM         1      0.00      0.00    2828   1368   0.13  init
    07:30:02 AM         1      0.00      0.00    2828   1368   0.13  init
    Average:               1      0.00      0.00    2828   1368   0.13  init
    %MEM:  该进程的内存使用百分比。

    #监控pid为1(init)的进程任务切换情况，其中每隔2秒刷新并输出一次，3次后程序退出。
    /> pidstat -p 1 2 3 -w
    07:32:15 AM       PID   cswch/s nvcswch/s  Command
    07:32:17 AM         1      0.00      0.00  init
    07:32:19 AM         1      0.00      0.00  init
    07:32:21 AM         1      0.00      0.00  init
    Average:            1      0.00      0.00  init
    cswch/s:    每秒任务主动(自愿的)切换上下文的次数。主动切换是指当某一任务处于阻塞等待时，将主动让出自己的CPU资源。
    nvcswch/s: 每秒任务被动(不自愿的)切换上下文的次数。被动切换是指CPU分配给某一任务的时间片已经用完，因此将强迫该进程让出CPU的执行权。

    #监控pid为1(init)的进程及其内部线程的内存(r选项)使用情况，其中每隔2秒刷新并输出一次，3次后程序退出。需要说明的是，如果-t选项后面不加任何其他选项，缺省监控的为CPU资源。结果中黄色高亮的部分表示进程和其内部线程是树状结构的显示方式。
    /> pidstat -p 1 2 3 -tr
    Linux 2.6.32-71.el6.i686 (Stephen-PC)   11/16/2011      _i686_  (1 CPU)

    07:37:04 AM      TGID       TID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
    07:37:06 AM         1         -      0.00      0.00        2828   1368      0.13  init
    07:37:06 AM         -         1      0.00      0.00        2828   1368      0.13  |__init

    07:37:06 AM      TGID       TID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
    07:37:08 AM         1         -      0.00      0.00        2828   1368      0.13  init
    07:37:08 AM         -         1      0.00      0.00        2828   1368      0.13  |__init

    07:37:08 AM      TGID       TID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
    07:37:10 AM         1         -      0.00      0.00        2828   1368      0.13  init
    07:37:10 AM         -         1      0.00      0.00        2828   1368      0.13  |__init

    Average:         TGID       TID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
    Average:            1         -      0.00      0.00        2828   1368      0.13  init
    Average:            -         1      0.00      0.00        2828   1368      0.13  |__init
    TGID: 线程组ID。
    TID： 线程ID。  

    以上监控不同资源的选项可以同时存在，这样就将在一次输出中输出多种资源的使用情况，如：pidstat -p 1 -dr。

    7.  报告磁盘空间使用状况(df):
    该命令最为常用的选项就是-h，该选项将智能的输出数据单位，以便使输出的结果更具可读性。
    /> df -h
    Filesystem             Size  Used   Avail Use% Mounted on
    /dev/sda1             5.8G  3.3G  2.2G  61%   /
    tmpfs                  504M  260K  504M   1%  /dev/shm

    8.  评估磁盘的使用状况(du)：
选项 	说明
-a 	包括了所有的文件，而不只是目录。
-b 	以字节为计算单位。
-k 	以千字节(KB)为计算单位。
-m 	以兆字节(MB)为计算单位。
-h 	是输出的信息更易于阅读。
-s 	只显示工作目录所占总空间。
--exclude=PATTERN 	排除掉符合样式的文件,Pattern就是普通的Shell样式，？表示任何一个字符，*表示任意多个字符。
--max-depth=N 	从当前目录算起，目录深度大于N的子目录将不被计算，该选项不能和s选项同时存在。

    #仅显示子一级目录的信息。
    /> du --max-depth=1 -h
    246M    ./stephen
    246M    .   
    /> du -sh ./*   #获取当前目录下所有子目录所占用的磁盘空间大小。
    352K    ./MemcachedTest
    132K    ./Test
    33M     ./thirdparty   
    #在当前目录下，排除目录名模式为Te*的子目录(./Test)，输出其他子目录占用的磁盘空间大小。
    /> du --exclude=Te* -sh ./*  
    352K    ./MemcachedTest
    33M     ./thirdparty
    
十九.  和系统运行进程相关的Shell命令:
    
   1.  进程监控命令(ps):
    要对进程进行监测和控制，首先必须要了解当前进程的情况，也就是需要查看当前进程，而ps命令就是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。总之大部分信息都是可以通过执行该命令得到的。
    ps命令存在很多的命令行选项和参数，然而我们最为常用只有两种形式，这里先给出与它们相关的选项和参数的含义：
选项 	说明
a 	显示终端上的所有进程，包括其他用户的进程。
u 	以用户为主的格式来显示程序状况。
x 	显示所有程序，不以终端来区分。
-e 	显示所有进程。
o 	其后指定要输出的列，如user，pid等，多个列之间用逗号分隔。
-p 	后面跟着一组pid的列表，用逗号分隔，该命令将只是输出这些pid的相关数据。

    /> ps aux

    root         1  0.0  0.1   2828  1400 ?        Ss   09:51   0:02 /sbin/init
    root         2  0.0  0.0      0          0 ?        S    09:51   0:00 [kthreadd]
    root         3  0.0  0.0      0          0 ?        S    09:51   0:00 [migration/0]
    ... ...  
    /> ps -eo user,pid,%cpu,%mem,start,time,command | head -n 4
    USER       PID %CPU %MEM  STARTED     TIME        COMMAND
    root         1         0.0    0.1   09:51:08     00:00:02  /sbin/init
    root         2         0.0    0.0   09:51:08     00:00:00  [kthreadd]
    root         3         0.0    0.0   09:51:08     00:00:00  [migration/0]
    这里需要说明的是，ps中存在很多和进程性能相关的参数，它们均以输出表格中的列的方式显示出来，在这里我们只是给出了非常常用的几个参数，至于更多参数，我们则需要根据自己应用的实际情况去看ps的man手册。
    #以完整的格式显示pid为1(init)的进程的相关数据
    /> ps -fp 1
    UID        PID  PPID  C STIME TTY          TIME   CMD
    root         1        0  0 05:16   ?        00:00:03 /sbin/init
   
    2.  改变进程优先级的命令(nice和renice):
    该Shell命令最常用的使用方式为：nice [-n <优先等级>][执行指令]，其中优先等级的范围从-20-19，其中-20最高，19最低，只有系统管理者可以设置负数的等级。
    #后台执行sleep 100秒，同时在启动时将其nice值置为19
    /> nice -n 19 sleep 100 &
    [1] 4661
    #后台执行sleep 100秒，同时在启动时将其nice值置为-19
    /> nice -n -19 sleep 100 &
    [2] 4664
    #关注ps -l输出中用黄色高亮的两行，它们的NI值和我们执行是设置的值一致。
    /> ps -l
    F S   UID   PID  PPID  C PRI  NI  ADDR  SZ    WCHAN  TTY       TIME        CMD
    4 S     0  2833  2829  0  80   0     -      1739     -         pts/2    00:00:00  bash
    0 S     0  4661  2833  0  99  19    -      1066     -         pts/2    00:00:00  sleep
    4 S     0  4664  2833  0  61 -19    -      1066     -         pts/2    00:00:00  sleep
    4 R     0  4665  2833  1  80   0     -      1231     -         pts/2    00:00:00  ps
   
    renice命令主要用于为已经执行的进程重新设定nice值，该命令包含以下几个常用选项：
选项 	说明
-g 	使用程序群组名称，修改所有隶属于该程序群组的程序的优先权。
-p 	改变该程序的优先权等级，此参数为预设值。
-u 	指定用户名称，修改所有隶属于该用户的程序的优先权。

    #切换到stephen用户下执行一个后台进程，这里sleep进程将在后台睡眠1000秒。
    /> su stephen
    /> sleep 1000&  
    [1] 4812
    /> exit   #退回到切换前的root用户
    #查看已经启动的后台sleep进程，其ni值为0，宿主用户为stephen
    /> ps -eo user,pid,ni,command | grep stephen
    stephen   4812   0 sleep 1000
    root        4821    0 grep  stephen
    #以指定用户的方式修改该用户下所有进程的nice值
    /> renice -n 5 -u stephen
    500: old priority 0, new priority 5
    #从再次执行ps的输出结果可以看出，该sleep后台进程的nice值已经调成了5
    /> ps -eo user,pid,ni,command | grep stephen
    stephen   4812   5 sleep 1000
    root         4826   0 grep  stephen
    #以指定进程pid的方式修改该进程的nice值
    /> renice -n 10 -p 4812
    4812: old priority 5, new priority 10
    #再次执行ps，该sleep后台进程的nice值已经从5变成了10
    /> ps -eo user,pid,ni,command | grep stephen
    stephen   4812  10 sleep 1000
    root        4829   0 grep  stephen


    3.  列出当前系统打开文件的工具(lsof):
    lsof(list opened files)，其重要功能为列举系统中已经被打开的文件，如果没有指定任何选项或参数，lsof则列出所有活动进程打开的所有文件。众所周知，linux环境中任何事物都是文件，如设备、目录、sockets等。所以，用好lsof命令，对日常的linux管理非常有帮助。下面先给出该命令的常用选项：
选项 	说明
-a 	该选项会使后面选项选出的结果列表进行and操作。
-c command_prefix 	显示以command_prefix开头的进程打开的文件。
-p PID 	显示指定PID已打开文件的信息
+d directory 	从文件夹directory来搜寻(不考虑子目录)，列出该目录下打开的文件信息。
+D directory 	从文件夹directory来搜寻(考虑子目录)，列出该目录下打开的文件信息。
-d num_of_fd 	以File Descriptor的信息进行匹配，可使用3-10，表示范围，3,10表示某些值。
-u user 	显示某用户的已经打开的文件，其中user可以使用正则表达式。
-i 	监听指定的协议、端口、主机等的网络信息，格式为：[proto][@host|addr][:svc_list|port_list]

    #查看打开/dev/null文件的进程。
    /> lsof /dev/null | head -n 5
    COMMAND    PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    init         1      root    0u   CHR    1,3      0t0 3671 /dev/null
    init         1      root    1u   CHR    1,3      0t0 3671 /dev/null
    init         1      root    2u   CHR    1,3      0t0 3671 /dev/null
    udevd 397      root    0u   CHR    1,3      0t0 3671 /dev/null

    #查看打开22端口的进程
    /> lsof -i:22
    COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    sshd    1582 root    3u  IPv4  11989      0t0  TCP *:ssh (LISTEN)
    sshd    1582 root    4u  IPv6  11991      0t0  TCP *:ssh (LISTEN)
    sshd    2829 root    3r   IPv4  19635      0t0  TCP bogon:ssh->bogon:15264 (ESTABLISHED)

    #查看init进程打开的文件
    />  lsof -c init
    COMMAND PID USER   FD   TYPE     DEVICE   SIZE/OFF   NODE    NAME
    init               1 root  cwd      DIR        8,2     4096              2        /
    init               1 root  rtd       DIR        8,2     4096              2        /
    init               1 root  txt       REG       8,2   136068       148567     /sbin/init
    init               1 root  mem    REG        8,2    58536      137507     /lib/libnss_files-2.12.so
    init               1 root  mem    REG        8,2   122232     186675     /lib/libgcc_s-4.4.4-20100726.so.1
    init               1 root  mem    REG        8,2   141492     186436     /lib/ld-2.12.so
    init               1 root  mem    REG        8,2  1855584    186631     /lib/libc-2.12.so
    init               1 root  mem    REG        8,2   133136     186632     /lib/libpthread-2.12.so
    init               1 root  mem    REG        8,2    99020      180422     /lib/libnih.so.1.0.0
    init               1 root  mem    REG        8,2    37304      186773     /lib/libnih-dbus.so.1.0.0
    init               1 root  mem    REG        8,2    41728      186633     /lib/librt-2.12.so
    init               1 root  mem    REG        8,2   286380     186634     /lib/libdbus-1.so.3.4.0
    init               1 root    0u     CHR        1,3      0t0           3671      /dev/null
    init               1 root    1u     CHR        1,3      0t0           3671      /dev/null
    init               1 root    2u     CHR        1,3      0t0           3671      /dev/null
    init               1 root    3r      FIFO       0,8      0t0           7969      pipe
    init               1 root    4w     FIFO       0,8      0t0           7969      pipe
    init               1 root    5r      DIR        0,10        0             1         inotify
    init               1 root    6r      DIR        0,10        0             1         inotify
    init               1 root    7u     unix   0xf61e3840  0t0       7970      socket
    init               1 root    9u     unix   0xf3bab280  0t0      11211     socket
    在上面输出的FD列中，显示的是文件的File Descriptor number，或者如下的内容：
    cwd:  current working directory;
    mem:  memory-mapped file;
    mmap: memory-mapped device;
    pd:   parent directory;
    rtd:  root directory;
    txt:  program text (code and data);
    文件的File Descriptor number显示模式有:
    r for read access;
    w for write access;
    u for read and write access;

    在上面输出的TYPE列中，显示的是文件类型，如：
    DIR:  目录
    LINK: 链接文件
    REG:  普通文件


    #查看pid为1的进程(init)打开的文件，其输出结果等同于上面的命令，他们都是init。
    /> lsof -p 1
    #查看owner为root的进程打开的文件。
    /> lsof -u root
    #查看owner不为root的进程打开的文件。
    /> lsof -u ^root
    #查看打开协议为tcp，ip为192.168.220.134，端口为22的进程。
    /> lsof -i tcp@192.168.220.134:22
    COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    sshd        2829 root     3r    IPv4  19635      0t0      TCP    bogon:ssh->bogon:15264 (ESTABLISHED)   
    #查看打开/root文件夹，但不考虑目录搜寻
    /> lsof +d /root
    #查看打开/root文件夹以及其子目录搜寻
    /> lsof +D /root
    #查看打开FD(0-3)文件的所有进程
    /> lsof -d 0-3
    #-a选项会将+d选项和-c选项的选择结果进行and操作，并输出合并后的结果。
    /> lsof +d .
    COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
    bash       9707  root  cwd    DIR    8,1     4096         39887 .
    lsof         9791  root  cwd    DIR    8,1     4096         39887 .
    lsof         9792  root  cwd    DIR    8,1     4096         39887 .
    /> lsof -a -c bash +d .
    COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
    bash        9707 root  cwd    DIR    8,1     4096         39887 .
    最后需要额外说明的是，如果在文件名的末尾存在(delete)，则说明该文件已经被删除，只是还存留在cache中。


    4.  进程查找/杀掉命令(pgrep/pkill)：
    查找和杀死指定的进程, 他们的选项和参数完全相同, 这里只是介绍pgrep。下面是常用的命令行选项：
选项 	说明
-d 	定义多个进程之间的分隔符, 如果不定义则使用换行符。
-n 	表示如果该程序有多个进程正在运行，则仅查找最新的，即最后启动的。
-o 	表示如果该程序有多个进程正在运行，则仅查找最老的，即最先启动的。
-G 	其后跟着一组group id，该命令在搜索时，仅考虑group列表中的进程。
-u 	其后跟着一组有效用户ID(effetive user id)，该命令在搜索时，仅考虑该effective user列表中的进程。
-U 	其后跟着一组实际用户ID(real user id)，该命令在搜索时，仅考虑该real user列表中的进程。
-x 	表示进程的名字必须完全匹配, 以上的选项均可以部分匹配。
-l 	将不仅打印pid,也打印进程名。
-f 	一般与-l合用, 将打印进程的参数。

    #手工创建两个后台进程
    /> sleep 1000&
    3456
    /> sleep 1000&
    3457

    #查找进程名为sleep的进程，同时输出所有找到的pid
    /> pgrep sleep
    3456
    3457
    #查找进程名为sleep的进程pid，如果存在多个，他们之间使用:分隔，而不是换行符分隔。
    /> pgrep -d: sleep
    3456:3457
    #查找进程名为sleep的进程pid，如果存在多个，这里只是输出最后启动的那一个。
    /> pgrep -n sleep
    3457
    #查找进程名为sleep的进程pid，如果存在多个，这里只是输出最先启动的那一个。
    /> pgrep -o  sleep
    3456
    #查找进程名为sleep，同时这个正在运行的进程的组为root和stephen。
    /> pgrep -G root,stephen sleep
    3456
    3457
    #查找有效用户ID为root和oracle，进程名为sleep的进程。
    /> pgrep -u root,oracle sleep
    3456
    3457
    #查找实际用户ID为root和oracle，进程名为sleep的进程。
    /> pgrep -U root,oracle sleep
    3456
    3457
    #查找进程名为sleep的进程，注意这里找到的进程名必须和参数中的完全匹配。
    /> pgrep -x sleep
    3456
    3457
    #-x不支持部分匹配，sleep进程将不会被查出，因此下面的命令没有结果。
    /> pgrep -x sle
    #查找进程名为sleep的进程，同时输出所有找到的pid和进程名。    
    /> pgrep -l sleep
    3456 sleep
    3457 sleep
    #查找进程名为sleep的进程，同时输出所有找到的pid、进程名和启动时的参数。
    /> pgrep -lf sleep
    3456 sleep 1000
    3457 sleep 1000
    #查找进程名为sleep的进程，同时以逗号为分隔符输出他们的pid，在将结果传给ps命令，-f表示显示完整格式，-p显示pid列表，ps将只是输出该列表内的进程数据。
    /> pgrep -f sleep -d, | xargs ps -fp
    UID        PID  PPID  C STIME TTY          TIME CMD
    root      3456  2138  0 06:11 pts/5    00:00:00 sleep 1000
    root      3457  2138  0 06:11 pts/5    00:00:00 sleep 1000  
    
    二十. 通过管道组合Shell命令获取系统运行数据:


    1.  输出当前系统中占用内存最多的5条命令:
    #1) 通过ps命令列出当前主机正在运行的所有进程。
    #2) 按照第五个字段基于数值的形式进行正常排序(由小到大)。
    #3) 仅显示最后5条输出。
    /> ps aux | sort -k 5n | tail -5
    stephen   1861  0.2  2.0  96972 21596  ?  S     Nov11   2:24 nautilus
    stephen   1892  0.0  0.4 102108  4508  ?  S<sl Nov11   0:00 /usr/bin/pulseaudio
    stephen   1874  0.0  0.9 107648 10124 ?  S     Nov11   0:00 gnome-volume
    stephen   1855  0.0  1.2 123776 13112 ?  Sl     Nov11   0:00 metacity
    stephen   1831  0.0  0.9 125432  9768  ?  Ssl   Nov11   0:05 /usr/libexec/gnome
    
    2.  找出cpu利用率高的20个进程:
    #1) 通过ps命令输出所有进程的数据，-o选项后面的字段列表列出了结果中需要包含的数据列。
    #2) 将ps输出的Title行去掉，grep -v PID表示不包含PID的行。
    #3) 基于第一个域字段排序，即pcpu。n表示以数值的形式排序。
    #4) 输出按cpu使用率排序后的最后20行，即占用率最高的20行。
    /> ps -e -o pcpu,pid,user,sgi_p,cmd | grep -v PID | sort -k 1n | tail -20

    3.  获取当前系统物理内存的总大小:
    #1) 以兆(MB)为单位输出系统当前的内存使用状况。
    #2) 通过grep定位到Mem行，该行是以操作系统为视角统计数据的。
    #3) 通过awk打印出该行的第二列，即total列。
    /> free -m | grep "Mem" | awk '{print $2, "MB"}'
    1007 MB

二十一. 通过管道组合Shell命令进行系统管理:

    1.  获取当前或指定目录下子目录所占用的磁盘空间，并将结果按照从大到小的顺序输出:
    #1) 输出/usr的子目录所占用的磁盘空间。
    #2) 以数值的方式倒排后输出。
    /> du -s /usr/* | sort -nr
    1443980 /usr/share
    793260   /usr/lib
    217584   /usr/bin
    128624   /usr/include
    60748    /usr/libexec
    45148    /usr/src
    21096    /usr/sbin
    6896      /usr/local
    4           /usr/games
    4           /usr/etc
    0           /usr/tmp
    
    2.  批量修改文件名:
    #1) find命令找到文件名扩展名为.output的文件。
    #2) sed命令中的-e选项表示流编辑动作有多次，第一次是将找到的文件名中相对路径前缀部分去掉，如./aa改为aa。
    #    流编辑的第二部分，是将20110311替换为mv & 20110310，其中&表示s命令的被替换部分，这里即源文件名。
    #    \1表示被替换部分中#的\(.*\)。
    #3) 此时的输出应为
    #    mv 20110311.output 20110310.output
    #    mv 20110311abc.output 20110310abc.output
    #    最后将上面的输出作为命令交给bash命令去执行，从而将所有20110311*.output改为20110311*.output
    /> find ./ -name "*.output" -print  | sed -e 's/.\///g' -e 's/20110311\(.*\)/mv & 20110310\1/g' | bash
   
    3.  统计当前目录下文件和目录的数量:
    #1) ls -l命令列出文件和目录的详细信息。
    #2) ls -l输出的详细列表中的第一个域字段是文件或目录的权限属性部分，如果权限属性部分的第一个字符为d，
    #    该文件为目录，如果是-，该文件为普通文件。
    #3) 通过wc计算grep过滤后的行数。
    /> ls -l * | grep "^-" | wc -l
    /> ls -l * | grep "^d" | wc -l
   
    4.  杀掉指定终端的所有进程:
    #1) 通过ps命令输出终端为pts/1的所有进程。
    #2) 将ps的输出传给grep，grep将过滤掉ps输出的Title部分，-v PID表示不包含PID的行。
    #3) awk打印输出grep查找结果的第一个字段，即pid字段。
    #4) 上面的三个组合命令是在反引号内被执行的，并将执行的结果赋值给数组变量${K}。
    #5) kill方法将杀掉数组${K}包含的pid。
    /> kill -9 ${K}=`ps -t pts/1 | grep -v PID | awk '{print $1}'`    

    5.  将查找到的文件打包并copy到指定目录:
    #1) 通过find找到当前目录下(包含所有子目录)的所有*.txt文件。
    #2) tar命令将find找到的结果压缩成test.tar压缩包文件。
    #3) 如果&&左侧括号内的命令正常完成，则可以执行&&右侧的shell命令了。
    #4) 将生成后的test.tar文件copy到/home/.目录下。
    /> (find . -name "*.txt" | xargs tar -cvf test.tar) && cp -f test.tar /home/.
   
    #1) cpio从find的结果中读取文件名，将其打包压缩后发送到./dest/dir(目标目录)。
    #2) cpio的选项介绍：
    #    -d：创建需要的目录。
    #    -a：重置源文件的访问时间。
    #    -m：保护新文件的修改时间。
    #    -p：将cpio设置为copy pass-through模式。
    /> find . -name "*" | cpio -dampv ./dest/dir

　  最后需要说明的是，该篇Blog中绝大多数的示例来自于互联网，是本人经过一天左右的时间收集和整理之后筛选出来的，其中注释部分是我在后来添加的，以便于我们阅读时的理解。如果今后再发现更好更巧妙的Shell组合命令，本人将持续更新该Blog。如果您有确实非常不错的Shell命令组合，且愿意和我们在这里分享，可以直接放在回复中，本人将对该篇Blog始终保持重点关注。


二十二. 交互式使用Bash Shell:

    1.  用set命令设置bash的选项:
    下面为set主要选项的列表及其表述：
选项名 	开关缩写 	描述
allexport 	-a 	打开此开关，所有变量都自动输出给子Shell。
noclobber 	-C 	防止重定向时文件被覆盖。
noglob 	-d 	在路径和文件名中，关闭通配符。


    #打开该选项
    /> set -o allexport   #等同于set -a
    #关闭该选项
    /> set +o allexport  #等同于set +a
    #列出当前所有选项的当前值。
    /> set -o
    allexport         off
    braceexpand   on
    emacs             on
    errexit            off
    errtrace          off
    functrace        off
    hashall            on
    histexpand      on
    ... ...
    /> set -o noclobber     #打开noclobber选项，防止在重定向时原有文件被覆盖。
    /> date > outfile         #通过date命令先生成一个文件outfile。
    /> ls > outfile             #将ls命令的输出重定向到该文件outfile，shell将提示不能覆盖已经存在的文件。
    -bash: outfile: cannot overwrite existing file
    /> set +o noclobber    #关闭noclobber选项。
    /> ls > outfile             #重新将ls的输出重定向到outfile，成功。


    2.  变量:
    设置局部变量:
    /> name="stephen liu"  #注意等号两边不要有空格，如果变量值之间存在空格，则需要用双引号括起
    /> echo $name
    stephen liu
    /> name=                    #将变量设置为空时，等号后面也不要有空格，直接回车即可。
    /> echo $name             #name变量为空，因此echo不会有任何输出。
    注意：以上变量的声明方式均可替换为declare variable=value的形式。

    /> declare name="stephen liu"
    /> readonly name         #将name变量设置为只读。
    /> echo $name
    stephen liu
    /> name="my wife"      #如果针对只读变量重新赋值，将报错，提示name是只读变量。
    -bash: name: readonly variable
    /> unset name             #如果unset只读变量，将同样报错，提示不能unset只读变量。
    -bash: unset: name: cannot unset: readonly variable

    设置全局/环境变量:
    在当前Shell中创建的全局/环境变量可以直接传递给它所有的子Shell，当前创建环境变量的Shell被称为夫Shell。
    /> export allname=john         #利用export命令，将其后声明的变量置为环境变量
    /> bash                                #启动一个新的子Shell
    /> echo $allname                  #在子Shell中echo变量$allname，发现夫Shell中设置的值被传递到子Shell
    john
    /> declare -x allname2=peter #这里的功能和结果都是和上面的命令相同，只是利用declare -x命令设置环境变量
    /> bash
    /> echo $allname2
    peter

    下面的列表将给出常用的内置Shell环境变量:
变量名 	含义
BASH 	表示bash命令的完整路径名。
ENV 	在启动新bash shell时执行的环境文件名。
HOME 	主目录。
LANG 	本地化语言。
PATH 	命令搜索路径，彼此之间冒号分隔。
PPID 	父进程PID。
PWD 	当前工作目录，用cd命令设置。


    3.  echo命令:
    该命令主要用于将其参数打印到标准输出。其中-e选项使得echo命令可以无限制地使用转义序列控制输出的效果。下面的列表给出常用的转义序列。
转义序列 	功能
\c 	不换行打印
\n 	换行
\t 	制表符
\\ 	反斜杠

    echo还提供了一个常用的-n选项，其功能不输出换行符。

    /> echo The username is $LOGNAME
    The username is stephen
    #下面命令的输出中的“/>”表示命令行提示符。
    /> echo -e "\tHello World\c"
        Hello World />
    /> echo -n "Hello World"
    Hello World />

    4.  printf命令:
    该命令和C语言中的printf函数的功能相同，都用用来格式化输出的。格式包括字符串本身和描述打印效果的字符。定义格式的方法和C语言也是完全一样的，即在%后面跟一个说明符，如%f表示后面是一个浮点数，%d表示一个整数。printf命令也同样支持转义序列符，其常用转义序列如下：
转义序列 	功能
\c 	不换行打印
\n 	换行
\t 	制表符
\\ 	反斜杠
\" 	双引号

    其常用的格式化说明符列表如下：
说明符 	描述
%c 	ASCII字符
%d,%i 	十进制整数
%f 	浮点格式
%o 	不带正负号的八进制值
%s 	字符串
%u 	不带正负号的十进制值
%x 	不带正负号的十六进制值，其中使用a-f表示10-15
%X 	不带正负号的十六进制值，其中使用A-F表示10-15
%% 	表示%本身

    下面是printf的一些常用使用方式：
    /> printf "The number is %.2f.\n" 100   这里.2f表示保留小数点后两位
    The number is 100.00.
    #%-20s表示一个左对齐、宽度为20个字符字符串格式，不足20个字符，右侧补充相应数量的空格符。
    #%-15s表示一个左对齐、宽度为15个字符字符串格式。
    #%10.2f表示右对齐、10个字符长度的浮点数，其中一个是小数点，小数点后面保留两位。
    /> printf "%-20s%-15s%10.2f\n" "Stephen" "Liu" 35
    Stephen             Liu                 35.00
    #%10s表示右对齐、宽度为10的字符串，如不足10个字符，左侧补充相应数量的空格符。
    /> printf "|%10s|\n" hello
    |     hello|
    在printf中还有一些常用的标志符，如上面例子中的符号(-)，这里我们在介绍一个比较常用的标识符"#"
    #如果#标志和%x/%X搭配使用，在输出十六进制数字时，前面会加0x/0X前缀。
    /> printf "%x %#x\n" 15 15
    f 0xf


   5.  变量替换运算符:
    bash中提供了一组可以同时检验和修改变量的特定修改符。这些修改符提供了一个快捷的方法来检验变量是不是被设置过，并把输出结果输出到一个变量中，见下表：
修改符 	描述 	用途
${variable:-word} 	如variable被设置且非空，则返回该值，否则返回word，变量值不变。 	如变量未定义，返回默认值。
${variable-word} 	如variable未被设置，则返回word，变量值不变，如果设置变量，则返回变量值，即使变量的值为空值。 	如变量未设置，返回默认值。
${variable:=word} 	如variable被设置且非空，则返回该值，否则设置变量为word，同时返回word。 	如果变量未定义，则设置其为默认值。
${variable=word} 	如variable未设置，则设置变量为word，同时返回word，如果variable被设置且为空，将返回空值，同时variable不变。否则返回variable值，同时variable不变。 	如果变量未设置，则设置其为默认值。
${variable:+word} 	如variable被设置且非空，则返回word，否则返回null，变量值不变。 	用于测试变量是否存在。
${variable+word} 	如variable被设置(即使是空值)，则返回word，否则返回空。 	用于测试变量是否设置。
${variable:?word} 	如variable被设置且非空，则返回该值，否则显示word，然后退出Shell。如果word为空，打印"parameter null or not set" 	为了捕捉由于变量未定义所导致的错误。
${variable:offset} 	从variable的offset位置开始取，直到末尾。 	 
${variable:offset:length} 	从variable的offset位置开始取length个字符。 	 


    #${variable:-word}的示例，其C语言表示形式为：
    #    if (NULL == variable)
    #        return word;
    #    else
    #        return $variable;
    /> unset var_name                        #将变量var_name置为空。
    /> var_name=
    /> echo ${var_name:-NewValue}    #var_name为空，因此返回NewValue
    NewValue
    /> echo $var_name                        #var_name的值未变化，仍然为空。

    /> var_name=OldValue                   #给var_name赋值。
    /> echo ${var_name:-NewValue}    #var_name非空，因此返回var_name的原有值。
    OldValue
    /> echo $var_name                        #var_name的值未变化，仍然OldValue。
    OldValue

    #${variable-word}的示例，其伪码表示形式为：
    #    if (variable is NOT set)
    #        return word;
    #    else
    #        return $variable;
    /> unset var_name                         #取消该变量var_name的设置。
    /> echo ${var_name-NewValue}    #var_name为空，因此返回NewValue
    NewValue
    /> echo $var_name                        #var_name的值未变化，仍然为空。

    /> var_name=OldValue                   #给var_name赋值，即便执行var_name=，其结果也是一样。
    /> echo ${var_name-NewValue}    #var_name非空，因此返回var_name的原有值。
    OldValue
    /> echo $var_name                        #var_name的值未变化，仍然OldValue。
    OldValue

    
    #${variable:=word}的示例，其C语言表示形式为：
    #    if (NULL == variable) {
    #        variable=world;
    #        return word;
    #    } else {
    #        return $variable;
    #    }
    /> unset var_name                        #将变量var_name置为空。
    /> var_name=
    /> echo ${var_name:=NewValue}   #var_name为空，设置变量为NewValue同时返回NewValue。
    NewValue
    /> echo $var_name                        #var_name的值已经被设置为NewValue。
    NewValue
    /> var_name=OldValue                  #给var_name赋值。
    /> echo ${var_name:=NewValue}   #var_name非空，因此返回var_name的原有值。
    OldValue
    /> echo $var_name                       #var_name的值未变化，仍然OldValue。
    OldValue
    
    #${variable=word}的示例，其伪码表示形式为：
    #    if (variable is NOT set) {
    #        variable=world;
    #        return word;
    #    } else if (variable == NULL) {
    #        return $variable;  //variable is NULL
    #    } else {
    #        return $variable;
    #    }
    /> unset var_name                        #取消该变量var_name的设置。
    /> echo ${var_name=NewValue}  #var_name未被设置，设置变量为NewValue同时返回NewValue。
    NewValue
    /> echo $var_name                        #var_name的值已经被设置为NewValue。
    NewValue
    /> var_name=                              #设置变量var_name，并给该变量赋空值。
    /> echo ${var_name=NewValue}  #var_name被设置，且为空值，返回var_name的原有空值。
   
    /> echo $var_name                       #var_name的值未变化，仍未空值。
   
    /> var_name=OldValue                  #给var_name赋值。
    /> echo ${var_name=NewValue}  #var_name非空，因此返回var_name的原有值。
    OldValue
    /> echo $var_name                       #var_name的值未变化，仍然OldValue。
    OldValue

    #${variable:+word}的示例，其C语言表示形式为：
    #    if (NULL != variable)
    #        return word;
    #    else
    #        return $variable;
    /> var_name=OldValue                  #设置变量var_name，其值为非空。
    /> echo ${var_name:+NewValue}   #由于var_name有值，因此返回NewValue
    NewValue
    /> echo $var_name                       #var_name的值仍然为远之OldValue
    OldValue
    /> unset var_name                        #将var_name置为空值。
    /> var_name=
    /> echo ${var_name:+NewValue}   #由于var_name为空，因此返回null。
    /> echo $var_name                       #var_name仍然保持原有的空值。

    #${variable+word}的示例，其伪码表示形式为：
    #    if (variable is set)
    #        return word;
    #    else
    #        return $variable;
    /> var_name=OldValue                  #设置变量var_name，其值为非空。
    /> echo ${var_name+NewValue}   #由于var_name有值，因此返回NewValue
    NewValue
    /> echo $var_name                       #var_name的值仍然为远之OldValue
    OldValue
    /> unset var_name                        #取消对变量var_name的设置。
    /> echo ${var_name+NewValue}   #返回空值。
    /> echo $var_name                       #var_name仍未被设置。

    #${variable:?word}的示例，其C语言表示形式为：
    #    if (NULL != variable) {
    #        return variable;
    #    } else {
    #        if (NULL != word)
    #            return "variable : word";
    #        else
    #            return "parameter null or not set";
    #    }
    /> var_name=OldValue                  #设置变量var_name，其值为非空。
    /> echo ${var_name:?NewValue}   #由于var_name有值，因此返回变量的原有值
    OldValue
    /> unset var_name                        #将var_name置为空值。
    /> var_name=
    /> echo ${var_name:?NewValue}   #由于var_name为空，因此返回word。
    -bash: var_name: NewValue
    /> echo $var_name                       #var_name仍然保持原有的空值。

    /> echo ${var_name:?}                #如果word为空，返回下面的输出。
    -bash: var_name: parameter null or not set

    #${variable:offset}示例：
    /> var_name=notebook
    /> echo ${var_name:2}
    tebook
    /> echo ${var_name:0}                #如果offset为0，则取var_name的全部值。
    notebook

    ${variable:offset:length}示例：
    /> var_name=notebook
    /> echo ${var_name:0:4}
    note
    /> echo ${var_name:4:4}
    book

    6.  变量模式匹配运算符:
    Shell中还提供了一组模式匹配运算符，见下表：
运算符 	替换
${variable#pattern} 	如果模式匹配变量值的开头，则删除匹配的最短部分，并返回剩下的部分，变量原值不变。
${variable##pattern} 	如果模式匹配变量值的开头，则删除匹配的最长部分，并返回剩下的部分，变量原值不变。
${variable%pattern} 	如果模式匹配变量值的结尾，则删除匹配的最短部分，并返回剩下的部分，变量原值不变。
${variable%%pattern} 	如果模式匹配变量值的结尾，则删除匹配的最长部分，并返回剩下的部分，变量原值不变。
${#variable} 	返回变量中字母的数量。


    #${variable#pattern}示例：
    /> pathname="/home/stephen/mycode/test.h"
    /> echo ${pathname#/home}        #变量pathname开始处匹配/home的最短部分被删除。
    /stephen/mycode/test.h
    /> echo $pathname                       #pathname的原值不变。
    /home/stephen/mycode/test.h

    #${variable##pattern}示例：
    /> pathname="/home/stephen/mycode/test.h"
    /> echo ${pathname##*/}            #变量pathname开始处匹配*/的最长部分被删除，*表示任意字符。
    test.h
    /> echo $pathname                       #pathname的原值不变。
    /home/stephen/mycode/test.h

    #${variable%pattern}示例：
    /> pathname="/home/stephen/mycode/test.h"
    /> echo ${pathname%/*}             #变量pathname结尾处匹配/*的最短部分被删除。
    /home/stephen/mycode
    /> echo $pathname                       #pathname的原值不变。
    /home/stephen/mycode/test.h

    #${variable%%pattern}示例：
    /> pathname="/home/stephen/mycode/test.h"
    /> echo ${pathname%%/*}          #变量pathname结尾处匹配/*的最长部分被删除，这里所有字符串均被删除。

    /> echo $pathname                       #pathname的原值不变。
    /home/stephen/mycode/test.h

    #${#variable}示例:
    /> name="stephen liu"
    /> echo ${#name}
    11

    7.  Shell中的内置变量:
    Shell中提供了一些以$开头的内置变量，见下表：
变量名 	描述
$? 	表示Shell命令的返回值
$$ 	表示当前Shell的pid
$- 	表示当前Shell的命令行选项
$! 	最后一个放入后台作业的PID值
$0 	表示脚本的名字
$1--$9 	表示脚本的第一到九个参数
${10} 	表示脚本的第十个参数
$# 	表示参数的个数
$*,$@ 	表示所有的参数，有双引号时除外，"$*"表示赋值到一个变量，"$@"表示赋值到多个。

    所有的内置变量都比较容易理解，因此这里仅给出$*和$@的区别用法：
    /> set 'apple pie' pears peaches
    /> for i in $*
    >  do
    >  echo $i
    >  done
    apple
    pie
    pears
    peaches

    /> set 'apple pie' pears peaches
    /> for i in $@
    >  do
    >  echo $i
    >  done
    apple
    pie
    pears
    peaches


    /> set 'apple pie' pears peaches
    /> for i in "$*"           #将所有参数变量视为一个
    >  do
    >  echo $i
    >  done
    apple pie pears    peaches
    
    /> set 'apple pie' pears peaches
    /> for i in "$@"
    >  do
    >  echo $i
    >  done
    apple pie                   #这里的单引号将两个单词合成一个.
    pears
    peaches    
   
    8.  引用:
    Shell中提供三种引用字符，分别是：反斜杠、单引号和双引号，它们可以使Shell中所有元字符失去其特殊功能，而还原其本意。见以下元字符列表：
元字符 	描述
; 	命令分隔符
& 	后台处理Shell命令
() 	命令组，创建一个子Shell
{} 	命令组，但是不创建子Shell
| 	管道
< > 	输入输出重定向
$ 	变量前缀
*[]? 	用于文件名扩展的Shell通配符

    注：单引号和双引号唯一的区别就是，双引号内可以包含变量和命令替换，而单引号则不会解释这些，见如下示例：
    /> name=Stephen
    /> echo "Hi $name, I'm glad to meet you! "  #name变量被替换
    Hi Stephen, I'm glad to meet you!
    /> echo 'Hi $name, I am glad to meet you! ' #name变量没有被替换
    Hi $name, I am glad to meet you!
    /> echo "Hey $name, the time is $(date)"      #name变量和date命令均被替换
    Hey Stephen, the time is Fri Nov 18 16:27:31 CST 2011
    /> echo 'Hey $name, the time is $(date)'
    Hey $name, the time is $(date)                      #name变量和date命令均未被替换

    9.  命令替换:
    同样我们需要把命令的输出结果赋值给一个变量或者需要用字符串替换变量的输出结果时，我们可以使用变量替换。在Shell中，通常使用反引号的方法进行命令替换。
    /> d=`date`                           #将date命令的执行结果赋值给d变量。
    /> echo $d
    Fri Nov 18 16:35:28 CST 2011
    /> pwd
    /home/stephen
    /> echo `basename \`pwd\``  #基于反引号的命令替换是可嵌入的，但是嵌入命令的反引号需要使用反斜杠转义。
    stephen
    除了反引号可以用于命令替换，这里我们也可以使用$(command)形式用于命令替换。
    /> d=$(date)
    /> echo $d
    Fri Nov 18 16:42:33 CST 2011
    /> dirname="$(basename $(pwd))"     #和之前的反引号一样，该方式也支持嵌套。
    /> echo $dirname
    stephen

   10.  数学扩展:
    Shell中提供了两种计算数学表达式的格式：$[ expression ]和$(( expression ))。
    /> echo $[5+4-2]
    7
    /> echo $[5+2*3]
    11
    /> echo $((5+4-2))
    7
    /> echo $((5+2*3))
    11    
    事实上，我们也可以在Shell中声明数值型的变量，这需要在declare命令的后面添加-i选项，如：
    /> declare -i num
    /> num=5+5      #注意在赋值的过程中，所有的符号之间均没有空格，如果需要空格，需要在表达式的外面加双引号
    /> echo $num    #如果没有声明declare -i num，该命令将返回5+5
    10
    /> num="5 * 5"
    /> echo $num
    25
    /> declare strnum
    /> strnum=5+5
    /> echo $strnum #由于并未将strnum声明为数值型，因此该输出将按字符串方式处理。
    5+5
    Shell还允许我们以不同进制的方式显示数值型变量的输出结果，其格式为：进制+#+变量。
    /> declare -i x=017 #017其格式为八进制格式
    /> echo $x           #缺省是十进制
    15
    /> x=2#101         #二进制
    /> echo $x
    5
    /> x=8#17           #八进制
    /> echo $x
    15
    /> x=16#b           #十六进制
    /> echo $x
    11
    在Shell中还提供了一个内置命令let，专门用于计算数学运算的，见如下示例：
    /> let i=5
    /> let i=i+1
    /> echo $i
    6
    /> let "i = i + 2"
    /> echo $i
    8
    /> let "i+=1"
    /> echo $i
    9    

    11.  数组:
    Shell中提供了创建一维数组的能力，你可以把一串数字、名字或者文件放在一个变量中。使用declare的-a选项即可创建它们，或者在变量后面增加下标操作符直接创建。和很多其它开发语言一样，Shell中的数组也是0开始的，然而不同的是Shell中数组的下标是可以不连续的。获取数组中某个元素的语法格式为: ${arrayname[index]}。见如下示例：
    /> declare -a friends                   #声明一个数组变量
    /> friends=(sheryl peter louise)   #给数组变量赋值
    /> echo ${friends[0]}                #通过数组下标的方式访问数组的元素
    sheryl
    /> echo ${friends[1]}
    peter
    /> echo ${friends[2]}
    louise
    /> echo ${friends[*]}                #下标中星号表示所有元素。
    shery1 peter louise
    # ${#array[*]}表示数组中元素的数量，而${#friend[0]}则表示第一个元素的长度。
    /> echo ${#friends[*]}    
    3
    /> unset friends                         #unset array清空整个数组，unset array[index]仅清空指定下标的元素。

    /> x[3]=100
    /> echo ${x[*]}
    100
    /> echo ${x[0]}                        #0下标的元素并没有被赋值过，因为输出为空。

    /> echo ${x[3]}
    100
    /> states=(ME [3]=CA [2]=CT)   #ME的下标为0。
    /> echo ${states[0]}
    ME
    /> echo ${states[1]}                 #数组下标为1的位置没有被赋值过，因此没有输出。

    /> echo ${states[2]}
    CT
    /> echo ${states[3]}
    CA

    12.  函数:
    和C语言一样，Shell中也可以创建自己的自定义函数。其格式如下：
    function_name () { commands; commands; }
    function function_name { commands; commands; }
    function function_name () { commands; commands; }
    函数的参数在函数内是以$[0-9]、${10}...，这种局部变量的方式来访问的。见下面的示例：

    #函数的左花括号和命令之间必须有至少一个空格。每个命令的后面都要有一个分号，即便是最后一个命令
    /> function greet { echo "Hello $LOGNAME, today is $(date)"; }
    #此时函数已经驻留在当前的bash shell中，因此使用函数效率更高。
    /> greet   
    Hello root, today is Fri Nov 18 20:45:10 CST 2011
    /> greet() { echo "Hello $LOGNAME, today is $(date)"; }
    /> greet
    Hello root, today is Fri Nov 18 20:46:40 CST 2011
    #welcome函数内部使用了函数参数。
    /> function welcome { echo "Hi $1 and $2"; }
    /> welcome stephen jane    
    Hi stephen and jane
    #declare -F选项将列出当前Shell中驻留的函数
    /> declare -F
    declare -f greet
    declare -f welcome
    #清空指定的函数，使其不在Shell中驻留。
    /> unset -f welcome

    13.  重定向:
    下面的列表为Shell中支持的重新定向操作符。
操作符 	功能
< 	重新定向输入
> 	重新定向输出
>> 	追加输出
2> 	重新定向错误
&> 	重新定向错误和输出
>& 	重新定向错误和输出
2>&1 	重新定向错误到标准输出
1>&2 	重新定向标准输出到错误
>| 	重新定向输出的时候覆盖noclobber选项


    #find命令将搜索结果输出到foundit文件，把错误信息输出到/dev/null
    /> find . -name "*.c" -print > foundit 2> /dev/null
    #将find命令的搜索结果和错误信息均输出到foundit文件中。
    /> find . -name "*.c" -print >& foundit
    #同上。
    /> find . -name "*.c" -print > foundit 2>&1
    #echo命令先将错误输出到errfile，再把信息发送到标准错误，该信息标准错误与标准输出合并在一起(errfile中)。
    /> echo "File needs an argument" 2> errfile 1>&2
    /> cat errfile
    File needs an argument
    
二十三. Bash Shell编程:

    1.  读取用户变量:
    read命令是用于从终端或者文件中读取输入的内建命令，read命令读取整行输入，每行末尾的换行符不被读入。在read命令后面，如果没有指定变量名，读取的数据将被自动赋值给特定的变量REPLY。下面的列表给出了read命令的常用方式：
命令格式 	描述
read answer 	从标准输入读取输入并赋值给变量answer。
read first last 	从标准输入读取输入到第一个空格或者回车，将输入的第一个单词放到变量first中，并将该行其他的输入放在变量last中。
read 	从标准输入读取一行并赋值给特定变量REPLY。
read -a arrayname 	把单词清单读入arrayname的数组里。
read -p prompt 	打印提示，等待输入，并将输入存储在REPLY中。
read -r line 	允许输入包含反斜杠。


    见下面的示例(绿色高亮部分的文本为控制台手工输入信息)：
    /> read answer        #等待读取输入，直到回车后表示输入完毕，并将输入赋值给变量answer
    Hello                       #控制台输入Hello
    /> echo $answer      #打印变量
    Hello

    #等待一组输入，每个单词之间使用空格隔开，直到回车结束，并分别将单词依次赋值给这三个读入变量。
    /> read one two three
    1 2 3                      #在控制台输入1 2 3，它们之间用空格隔开。
    /> echo "one = $one, two = $two, three = $three"
    one = 1, two = 2, three = 3

    /> read                  #等待控制台输入，并将结果赋值给特定内置变量REPLY。
    This is REPLY          #在控制台输入该行。
    /> echo $REPLY      #打印输出特定内置变量REPLY，以确认是否被正确赋值。
    This is REPLY

    /> read -p "Enter your name: "    #输出"Enter your name: "文本提示，同时等待输入，并将结果赋值给REPLY。
    Enter you name: stephen            #在提示文本之后输入stephen
    /> echo $REPLY
    stephen

    #等待控制台输入，并将输入信息视为数组，赋值给数组变量friends，输入信息用空格隔开数组的每个元素
    /> read -a friends
    Tim Tom Helen
    /> echo "I have ${#friends} friends"
    I have 3 friends
    /> echo "They are ${friends[0]}, ${friends[1]} and ${friends[2]}."
    They are Tim, Tom and Helen.

   2.  状态判断:
    test是Shell中提供的内置命令，主要用于状态的检验，如果结果为0，表示成功，否则表示失败。见如下示例：
    /> name=stephen
    /> test $name != stephen
    /> echo $?
    1
    需要注意的是test命令不支持Shell中提供的各种通配符，如：
    /> test $name = [Ss]tephen
    /> echo $?
    1
    test命令还可以中括号予以替换，其语义保持不变，如：
    /> [ $name = stephen ]
    /> echo $?
    0   
    在Shell中还提供了另外一种用于状态判断的方式：[[ expr ]]，和test不同的是，该方式中的表达式支持通配符，如：
    /> name=stephen
    /> [[ $name == [Ss]tephen ]]
    /> echo $?
    0
    #在[[ expression ]]中，expression可以包含&&(逻辑与)和||(逻辑或)。
    /> [[ $name == [Ss]tephen && $friend == "Jose" ]]
    /> echo $?
    1
    /> shopt -s extglob   #打开Shell的扩展匹配模式。
    /> name=Tommy
    # "[Tt]o+(m)y"的含义为，以T或t开头，后面跟着一个o，再跟着一个或者多个m，最后以一个y结尾。
    /> [[ $name == [Tt]o+(m)y ]]
    /> echo $?
    0
    在Shell中还提供了let命令的判断方式： (( expr ))，该方式的expr部分，和C语言提供的表达式规则一致，如：
    /> x=2
    /> y=3
    /> (( x > 2 ))
    /> echo $?
    1
    /> (( x < 2 ))
    /> echo $?
    0　　
    /> (( x == 2 && y == 3 ))
    /> echo $?
    0
    /> (( x > 2 || y < 3 ))
    /> echo $?
    1

    下面的表格是test命令支持的操作符：
判断操作符 	判断为真的条件
字符串判断 	 
[ stringA=stringB ] 	stringA等于stringB
[ stringA==stringB ] 	stringA等于stringB
[ stringA!=stringB ] 	stringA不等于stringB
[ string ] 	string不为空
[ -z string ] 	string长度为0
[ -n string ] 	string长度不为0
逻辑判断 	 
[ stringA -a stringB ] 	stringA和stringB都是真
[ stringA -o stringB ] 	stringA或stringB是真
[ !string ] 	string不为真
逻辑判断(复合判断) 	 
[[ pattern1 && pattern2 ]] 	pattern1和pattern2都是真
[[ pattern1 || pattern2 ] 	pattern1或pattern2是真
[[ !pattern ]] 	pattern不为真
整数判断 	 
[ intA -eq intB ] 	intA等于intB
[ intA -ne intB ] 	intA不等于intB
[ intA -gt intB ] 	intA大于intB
[ intA -ge intB ] 	intA大于等于intB
[ intA -lt intB ] 	intA小于intB
[ intA -le intB ] 	intA小于等于intB
文件判断中的二进制操作 	 
[ fileA -nt fileB ] 	fileA比fileB新
[ fileA -ot fileB ] 	fileA比fileB旧
[ fileA -ef fileB ] 	fileA和fileB有相同的设备或者inode值
文件检验 	 
[ -d $file ] or [[ -d $file ]] 	file为目录且存在时为真
[ -e $file ] or [[ -e $file ]] 	file为文件且存在时为真
[ -f $file ] or [[ -f $file ]] 	file为非目录普通文件存在时为真
[ -s $file ] or [[ -s $file ]] 	file文件存在, 且长度不为0时为真
[ -L $file ] or [[ -L $file ]] 	file为链接符且存在时为真
[ -r $file ] or [[ -r $file ]] 	file文件存在且可读时为真
[ -w $file ] or [[ -w $file ]] 	file文件存在且可写时为真
[ -x $file ] or [[ -x $file ]] 	file文件存在且可执行时为真

    注：在逻辑判断(复合判读中)，pattern可以包含元字符，在字符串的判断中，pattern2必须被包含在引号中。

    let命令支持的操作符和C语言中支持的操作符完全相同，如：
    +,-,*,/,%            加，减，乘，除，去模
    >>,<<                右移和左移
    >=,<=,==,!=      大于等于，小于等于，等于，不等于
    &,|,^                  按位与，或，非
    &&,||,!                逻辑与，逻辑或和取反
    还有其含义和C语言等同的快捷操作符，如=,*=,/=,%=,+=,-=,<<=,>>=,&=,|=,^=。

    3.  流程控制语句:
    if语句格式如下：
    #if语句的后面是Shell命令，如果该命令执行成功返回0，则执行then后面的命令。
    if command        
    then
        command
        command
    fi
    #用test命令测试其后面expression的结果，如果为真，则执行then后面的命令。
    if test expression
    then
        command
    fi
    #下面的格式和test expression等同
    if [ string/numeric expression ]
    then
        command
    fi
    #下面的两种格式也可以用于判断语句的条件表达式，而且它们也是目前比较常用的两种。
    if [[ string expression ]]
    then
        command
    fi

    if (( numeric expression ))           #let表达式
    then
        command
    fi
    见如下示例：
    /> cat > test1.sh                       #从命令行直接编辑test1.sh文件。
    echo -e "Are you OK(y/n)? \c"
    read answer
    #这里的$answer变量必须要用双引号扩住，否则判断将失败。当变量$answer等于y或Y时，支持下面的echo命令。
    if [ "$answer" = y -o "$answer" = Y ]   
    then
        echo "Glad to see it."
    fi
    CTRL+D  
    /> . ./test1.sh
    Are you OK(y/n)? y
    Glad to see it.
    上面的判断还可以替换为：
    /> cat > test2.sh
    echo -e "Are you OK(y/n or Maybe)? \c"
    read answer
    # [[ ]]复合命令操作符允许其中的表达式包含元字符，这里输入以y或Y开头的任意单词，或Maybe都执行then后面的echo。
    if [[ $answer == [yY]* || $answer = Maybe ]]  
    then
        echo "Glad to hear it.
    fi
    CTRL+D
    /> . ./test2.sh
    Are you OK(y/n or Maybe)? yes
    Glad to hear it.
    下面的例子将使用Shell中的扩展通配模式。
    /> shopt -s extglob        #打开该扩展模式
    /> answer="not really"
    /> if [[ $answer = [Nn]o?( way |t really) ]]
    > then
    >    echo "I am sorry."
    > fi
    I am sorry.
    对于本示例中的扩展通配符，这里需要给出一个具体的解释。[Nn]o匹配No或no，?( way|t really)则表示0个或1个( way或t really),因此answer变量匹配的字符串为No、no、Not really、not really、No way、no way。
    下面的示例使用了let命令操作符，如：
    /> cat > test3.sh
    if (( $# != 2 ))                    #等同于 [ $# -ne 2 ]
    then
        echo "Usage: $0 arg1 arg2" 1>&2
        exit 1                         #exit退出值为0-255之间，只有0表示成功。
    fi
    if (( $1 < 0 || $1 > 30 ))      #等同于 [ $1 -lt 0 -o $1 -gt 30 ]
    then
        echo "arg1 is out of range."
        exit 2
    fi
    if (( $2 <= 20 ))                  #等同于 [ $2 -le 20 ]
    then
        echo "arg2 is out of range."
    fi
    CTRL+D
    /> sh ./test3.sh
    Usage: ./test3.sh arg1 arg2
    /> echo $?                          #Shell脚本的退出值为exit的参数值。
    1
    /> sh ./test3.sh 40 30
    arg1 is out of range.
    /> echo $?
    2
    下面的示例为如何在if的条件表达式中检验空变量：
    /> cat > test4.sh
    if [ "$name" = "" ]                #双引号就表示空字符串。
    then
        echo "name is null."
    fi
    CTRL+D
    /> . ./test4.sh
    name is null.

    if/elif/else语句的使用方式和if语句极为相似，相信有编程经验的人都不会陌生，这里就不在赘述了，其格式如下：
    if command
    then
        command
    elif command
    then
        command
    else
        command
    fi
    见如下示例脚本：
    /> cat > test5.sh
    echo -e "How old are you? \c"
    read age
    if [ $age -lt 0 -o $age -gt 120 ]                #等同于 (( age < 0 || age > 120 ))
    then
        echo "You are so old."
    elif [ $age -ge 0 -a $age -le 12 ]               #等同于 (( age >= 0 && age <= 12 ))
    then
        echo "You are child."
    elif [ $age -ge 13 -a $age -le 19 ]             #等同于 (( age >= 13 && age <= 19 ))
    then
        echo "You are 13--19 years old."
    elif [ $age -ge 20 -a $age -le 29 ]             #等同于 (( age >= 20 && age <= 29 ))
    then
        echo "You are 20--29 years old."
    elif [ $age -ge 30 -a $age -le 39 ]             #等同于 (( age >= 30 && age <= 39 ))
    then
        echo "You are 30--39 years old."
    else
        echo "You are above 40."
    fi
    CTRL+D
    /> . ./test5.sh
    How old are you? 50
    You are above 40.

    case语句格式如下：
    case variable in
    value1)
        command
        ;;            #相同于C语言中case语句内的break。
    value2)
        command
        ;;
    *)                #相同于C语言中switch语句内的default
       command
        ;;
    esac
    见如下示例脚本：
    /> cat > test6.sh
    #!/bin/sh
    echo -n "Choose a color: "
    read color
    case "$color" in
    [Bb]l??)
        echo "you select blue color."
        ;;
    [Gg]ree*)
        echo "you select green color."
        ;;
    red|orange)
        echo "you select red or orange."
        ;;
    *)
        echo "you select other color."
        ;;
    esac
    echo "Out of case command."
    /> . ./test6.sh
    Choose a color: green
    you select green color.
    Out of case command.

   4.  循环语句:
    Bash Shell中主要提供了三种循环方式：for、while和until。
    for循环声明格式：
    for variable in word_list
    do
        command
    done
    见如下示例脚本：
    /> cat > test7.sh
    for score in math english physics chemist   #for将循环读取in后面的单词列表，类似于Java的for-each。
    do
        echo "score = $score"
    done
    echo "out of for loop"
    CTRL+D
    /> . ./test7.sh
    score = math
    score = english
    score = physics
    score = chemist
    out of for loop

    /> cat > mylist   #构造数据文件
    tom
    patty
    ann
    jake
    CTRL+D
    /> cat > test8.sh
    #!/bin/sh
    for person in $(cat mylist)                 #for将循环读取cat mylist命令的执行结果。
    do
        echo "person = $person"
    done
    echo "out of for loop."
    CTRL+D
    /> . ./test8.sh
    person = tom
    person = patty
    person = ann
    person = jake
    out of for loop.

    /> cat > test9.sh
    for file in test[1-8].sh                        #for将读取test1-test8，后缀为.sh的文件
    do
        if [ -f $file ]                              #判断文件在当前目录是否存在。
        then
            echo "$file exists."
        fi
    done
    CTRL+D
    /> . ./test9.sh
    test2.sh exists.
    test3.sh exists.
    test4.sh exists.
    test5.sh exists.
    test6.sh exists.
    test7.sh exists.
    test8.sh exists.

    /> cat > test10.sh
    for name in $*                                  #读取脚本的命令行参数数组，还可以写成for name的简化形式。
    do
        echo "Hi, $name"
    done
    CTRL+D
    /> . ./test10.sh stephen ann
    Hi, stephen
    Hi, ann

    while循环声明格式：
    while command  #如果command命令的执行结果为0，或条件判断为真时，执行循环体内的命令。
    do
        command
    done
    见如下示例脚本：
    /> cat > test1.sh  
    num=0
    while (( num < 10 ))               #等同于 [ $num -lt 10 ]
    do
        echo -n "$num "
        let num+=1
    done
    echo -e "\nHere's out of loop."
    CTRL+D
    /> . ./test1.sh
    0 1 2 3 4 5 6 7 8 9
    Here's out of loop.

    /> cat > test2.sh
    go=start
    echo Type q to quit.
    while [[ -n $go ]]                     #等同于[ -n "$go" ]，如使用该风格，$go需要被双引号括起。
    do
        echo -n How are you.
        read word
        if [[ $word == [Qq] ]]      #等同于[ "$word" = Q -o "$word" = q ]
        then
            echo Bye.
            go=                        #将go变量的值置空。
        fi
    done
    CTRL+D
    /> . ./test2.sh
    How are you. Hi
    How are you. q
    Bye.

    until循环声明格式:
    until command                         #其判断条件和while正好相反，即command返回非0，或条件为假时执行循环体内的命令。
    do
        command
    done
    见如下示例脚本：
    /> cat > test3.sh
    until who | grep stephen           #循环体内的命令将被执行，直到stephen登录，即grep命令的返回值为0时才退出循环。
    do
        sleep 1
        echo "Stephen still doesn't login."
    done
    CTRL+D

    shift命令声明格式:shift [n]
    shift命令用来把脚本的位置参数列表向左移动指定的位数(n)，如果shift没有参数，则将参数列表向左移动一位。一旦移位发生，被移出列表的参数就被永远删除了。通常在while循环中，shift用来读取列表中的参数变量。
    见如下示例脚本：
    /> set stephen ann sheryl mark #设置4个参数变量。
    /> shift                                    #向左移动参数列表一次，将stephen移出参数列表。
    /> echo $*
    ann sheryl mark
    /> shift 2                                 #继续向左移动两位，将sheryl和ann移出参数列表
    /> echo $*
    mark
    /> shift 2                                 #继续向左移动两位，由于参数列表中只有mark了，因此本次移动失败。
    /> echo $*
    mark

    /> cat > test4.sh
    while (( $# > 0 ))                    #等同于 [ $# -gt 0 ]
    do
        echo $*
        shift
    done
    CTRL+D
    /> . ./test4.sh a b c d e
    a b c d e
    b c d e
    c d e
    d e
    e        

    break命令声明格式：break [n]
    和C语言不同的是，Shell中break命令携带一个参数，即可以指定退出循环的层数。如果没有指定，其行为和C语言一样，即退出最内层循环。如果指定循环的层数，则退出指定层数的循环体。如果有3层嵌套循环，其中最外层的为1，中间的为2，最里面的是3。
    见如下示例脚本:
    /> cat > test5.sh
    while true
    do
        echo -n "Are you ready to move on?"
        read answer
        if [[ $answer == [Yy] ]]
        then
            break
        else
            echo "Come on."
        fi
    done
    echo "Here we are."
    CTRL+D
    /> . ./test5.sh
    Are you ready to move on? y
    Here we are

    continue命令声明格式：continue [n]
    和C语言不同的是，Shell中continue命令携带一个参数，即可以跳转到指定层级的循环顶部。如果没有指定，其行为和C语言一样，即跳转到最内层循环的顶部。如果指定循环的层数，则跳转到指定层级循环的顶部。如果有3层嵌套循环，其中最外层的为3，中间的为2，最里面的是1。
    /> cat  maillist                       #测试数据文件maillist的内容为以下信息。
    stephen
    ann
    sheryl
    mark

    /> cat > test6.sh
    for name in $(cat maillist)
    do
        if [[ $name == stephen ]]; then
            continue
        else
            echo "Hello, $name."
        fi
    done
    CTRL+D
    /> . ./test6.sh
    Hello, ann.
    Hello, sheryl.
    Hello, mark.

    I/O重新定向和子Shell：
    文件中的输入可以通过管道重新定向给一个循环，输出也可以通过管道重新定向给一个文件。Shell启动一个子Shell来处理I/O重新定向和管道。在循环终止时，循环内部定义的任何变量对于脚本的其他部分来说都是不看见的。
    /> cat > demodata                        #为下面的脚本构造册数数据
    abc
    def
    ghi
    CRTL+D
    /> cat > test7.sh
    if (( $# < 1 ))                                #如果脚本参数的数量小于1，则给出错误提示后退出。
    then
        echo "Usage: $0 filename " >&2
        exit 1
    fi
    count=1
    cat $1 | while read line                   #参数一中的文件被cat命令输出后，通过管道逐行输出给while read line。
    do
        let $((count == 1)) && echo "Processing file $1..." > /dev/tty  #该行的echo将输出到当前终端窗口。
        echo -e "$count\t$line"              #将输出行号和文件中该行的内容，中间用制表符隔开。
        let count+=1
    done > outfile                               #将while循环中所有的输出，除了>/dev/tty之外，其它的全部输出到outfile文件。
    CTRL+D
    /> . ./test7.sh demodata                #只有一行输出，其余的都输出到outfile中了。
    Processing file demodata...
    /> cat outfile
    1       abc
    2       def
    3       ghi

    /> cat > test8.sh
    for i in 9 7 2 3 5 4
    do
        echo $i
    done | sort -n                                #直接将echo的输出通过管道重定向sort命令。
    CTRL+D
    /> . ./test8.sh
    2
    3
    4
    5
    7
    9

    5.  IFS和循环：
    Shell的内部域分隔符可以是空格、制表符和换行符。它可以作为命令的分隔符用在例如read、set和for等命令中。如果在列表中使用不同的分隔符，用户可以自己定义这个符号。在修改之前将IFS原始符号的值保存在另外一个变量中，这样在需要的时候还可以还原。
    见如下示例脚本：
    /> cat > test9.sh
    names=Stephen:Ann:Sheryl:John   #names变量包含的值用冒号分隔。
    oldifs=$IFS                                   #保留原有IFS到oldifs变量，便于后面的还原。
    IFS=":"                            
    for friends in $names                     #这是遍历以冒号分隔的names变量值。    
    do
        echo Hi $friends
    done
    IFS=$oldifs                                   #将IFS还原为原有的值。
    set Jerry Tom Angela
    for classmates in $*                      #再以原有IFS的值变量参数列表。
    do
        echo Hello $classmates
    done
    CTRL+D
    /> . ./test9.sh
    Hi Stephen
    Hi Ann
    Hi Sheryl
    Hi John
    Hello Jerry
    Hello Tom
    Hello Angela

    6.  函数：
    Shell中函数的职能以及优势和C语言或其它开发语言基本相同，只是语法格式上的一些差异。下面是Shell中使用函数的一些基本规则：
    1) 函数在使用前必须定义。
    2) 函数在当前环境下运行，它和调用它的脚本共享变量，并通过位置参量传递参数。而该位置参量将仅限于该函数，不会影响到脚本的其它地方。
    3) 通过local函数可以在函数内建立本地变量，该变量在出了函数的作用域之后将不在有效。
    4) 函数中调用exit，也将退出整个脚本。
    5) 函数中的return命令返回函数中最后一个命令的退出状态或给定的参数值，该参数值的范围是0-256之间。如果没有return命令，函数将返回最后一个Shell的退出值。
    6) 如果函数保存在其它文件中，就必须通过source或dot命令把它们装入当前脚本。
    7) 函数可以递归。
    8) 将函数从Shell中清空需要执行：unset -f function_name。
    9) 将函数输出到子Shell需要执行：export -f function_name。
    10) 可以像捕捉Shell命令的返回值一样获取函数的返回值，如$(function_name)。
    Shell中函数的声明格式如下：
    function function_name { command; command; }
    见如下示例脚本：
    /> cat > test1.sh
    function increment() {            #定义函数increment。
        local sum                           #定义本地变量sum。
        let "sum=$1+1"    
        return $sum                      #返回值是sum的值。
    }
    echo -n "The num is "
    increment 5                          #increment函数调用。
    echo $?                                #输出increment函数的返回值。
    CTRL+D
    /> . ./test1.sh
    The num is 6

    7.  陷阱信号(trap)：
    在Shell程序运行的时候，可能收到各种信号，有的来自于操作系统，有的来自于键盘，而该Shell在收到信号后就立刻终止运行。但是在有些时候，你可能并不希望在信号到达时，程序就立刻停止运行并退出。而是他能希望忽略这个信号而一直在运行，或者在退出前作一些清除操作。trap命令就允许你控制你的程序在收到信号以后的行为。
    其格式如下：
    trap 'command; command' signal-number
    trap 'command; command' signal-name
    trap signal-number  
    trap signal-name
    后面的两种形式主要用于信号复位，即恢复处理该信号的缺省行为。还需要说明的是，如果trap后面的命令是使用单引号括起来的，那么该命令只有在捕获到指定信号时才被执行。如果是双引号，则是在trap设置时就可以执行变量和命令替换了。
    下面是系统给出的信号数字和信号名称的对照表：
    1)SIGHUP 2)SIGINT 3)SIGQUIT 4)SIGILL 5)SIGTRAP 6)SIGABRT 7)SIGBUS 8)SIGFPE
    9)SIGKILL 10) SIGUSR1 11)SIGEGV 12)SIGUSR2 13)SIGPIPE 14)SIGALRM 15)SIGTERM 17)SIGCHLD
    18)SIGCONT 19)SIGSTOP ... ...
    见如下示例脚本：
    /> trap 'rm tmp*;exit 1' 1 2 15      #该命令表示在收到信号1、2和15时，该脚本将先执行rm tmp*，然后exit 1退出脚本。
    /> trap 2                                      #当收到信号2时，将恢复为以前的动作，即退出。
    /> trap " " 1 2                              #当收到信号1和2时，将忽略这两个信号。
    /> trap -                                      #表示恢复所有信号处理的原始值。
    /> trap 'trap 2' 2                           #在第一次收到信号2时，执行trap 2，这时将信号2的处理恢复为缺省模式。在收到信号2时，Shell程序退出。
    /> cat > test2.sh
    trap 'echo "Control+C will not terminate $0."' 2   #捕获信号2，即在键盘上按CTRL+C。
    trap 'echo "Control+\ will not terminate $0."' 3   #捕获信号3，即在键盘上按CTRL+\。
    echo "Enter stop to quit shell."
    while true                                                        #无限循环。
    do
        echo -n "Go Go...."
        read
        if [[ $REPLY == [Ss]top ]]                            #直到输入stop或Stop才退出循环和脚本。
       then
            break
        fi
    done
    CTRL+D
    /> . ./test2.sh
    Enter stop to quit shell.
    Go Go....^CControl+C will not terminate -bash.
    ^\Control+\ will not terminate -bash.
    stop

    8.  用getopts处理命令行选项：
    这里的getopts命令和C语言中的getopt几乎是一致的，因为脚本的位置参量在有些时候是失效的，如ls -lrt等。这时候-ltr都会被保存在$1中，而我们实际需要的则是三个展开的选项，即-l、-r和-t。见如下带有getopts的示例脚本：
    /> cat > test3.sh
    #!/bin/sh
    while getopts xy options                           #x和y是合法的选项，并且将-x读入到变量options中，读入时会将x前面的横线去掉。
    do
        case $options in
        x) echo "you entered -x as an option" ;;       
        y) echo "you entered -y as an option" ;;
        esac
    done
    /> ./test3.sh -xy
    you entered -x as an option
    you entered -y as an option
    /> ./test3.sh -x
    you entered -x as an option
    /> ./test3.sh -b                                       #如果输入非法选项，getopts会把错误信息输出到标准错误。
    ./test3.sh: illegal option -- b
    /> ./test3.sh b                                        #该命令不会有执行结果，因为b的前面有没横线，因此是非法选项，将会导致getopts停止处理并退出。

    /> cat > test4.sh
    #!/bin/sh
    while getopts xy options 2>/dev/null         #如果再出现选项错误的情况，该重定向会将错误输出到/dev/null。
    do
        case $options in
        x) echo "you entered -x as an option" ;;
        y) echo "you entered -y as an option" ;;
        \?) echo "Only -x and -y are valid options" 1>&2 # ?表示所有错误的选项，即非-x和-y的选项。
    esac
    done
    /> . ./test4.sh -g                                     #遇到错误的选项将直接执行\?)内的代码。
    Only -x and -y are valid options
    /> . ./test4.sh -xg
    you entered -x as an option
    Only -x and -y are valid options

    /> cat > test5.sh
    #!/bin/sh
    while getopts xyz: arguments 2>/dev/null #z选项后面的冒号用于提示getopts，z选项后面必须有一个参数。
    do
        case $arguments in
        x) echo "you entered -x as an option." ;;
        y) echo "you entered -y as an option." ;;
        z) echo "you entered -z as an option."  #z的后面会紧跟一个参数，该参数保存在内置变量OPTARG中。
            echo "\$OPTARG is $OPTARG.";
           ;;
        \?) echo "Usage opts4 [-xy] [-z argument]"
            exit 1 ;;
        esac
    done
    echo "The number of arguments passed was $(( $OPTIND - 1 ))" #OPTIND保存一下将被处理的选项的位置，他是永远比实际命令行参数多1的数。
    /> ./test5.sh -xyz foo
    you entered -x as an option.
    you entered -y as an option.
    you entered -z as an option.
    $OPTARG is foo.
    The number of arguments passed was 2
    /> ./test5.sh -x -y -z boo
    you entered -x as an option.
    you entered -y as an option.
    you entered -z as an option.
    $OPTARG is boo.
    The number of arguments passed was 4

    9.  eval命令与命令行解析：
    eval命令可以对命令行求值，做Shell替换，并执行命令行，通常在普通命令行解析不能满足要求时使用。
    /> set a b c d
    /> echo The last argument is \$$#
    The last argument is $4
    /> eval echo The last argument is \$$#    #eval命令先进行了变量替换，之后再执行echo命令。
    The last argument is d



  该系列将重点介绍Linux Shell中的高级使用技巧，其主要面向有一定经验的Shell开发者、Linux系统管理员，以及Linux的爱好者。博客中的示例主要来源于网络和一些经典书籍，在经过本人的收集和整理之后，以系列博客的形式呈现给诸位。如果大家有更多更好的Shell脚本经典示例，且愿意在这里与我们一同分享的话，可以以邮件、博客回复等形式与我联系，我将会尽量保证该系列的持续更新。

一、将输入信息转换为大写字符后再进行条件判断：

      我们在读取用户的正常输入后，很有可能会将这些输入信息用于条件判断，那么在进行比较时，我们将不得不考虑这些信息的大小写匹配问题。

      /> cat > test1.sh
      #!/bin/sh
      echo -n "Please let me know your name. "
      read name
      #将变量name的值通过管道输出到tr命令，再由tr命令进行大小写转换后重新赋值给name变量。
      name=`echo $name | tr [a-z] [A-Z]`
      if [[ $name == "STEPHEN" ]]; then
          echo "Hello, Stephen."
      else
          echo "You are not Stephen."
      fi
      CTRL+D
      /> ./test1.sh
      Please let me know your name. stephen
      Hello, Stephen.

二、为调试信息设置输出级别：
    
      我们经常在调试脚本时添加一些必要的调试信息，以便跟踪到程序中的错误。在完成调试后，一般都会选择删除这些额外的调试信息，在过了一段时间之后，如果脚本需要添加新的功能，那么我们将不得不重新进行调试，这样又有可能需要添加这些调试信息，在调试成功之后，这些信息可能会被再次删除。如果我们能够为我们的调试信息添加调试级别，使其只在必要的时候输出，我想这将会是一件非常惬意的事情。
      /> cat > test2.sh
      #!/bin/sh
      if [[ $# == 0 ]]; then
          echo "Usage: ./test2.sh -d debug_level"
          exit 1
      fi
      #1. 读取脚本的命令行选项参数，并将选项赋值给变量argument。
      while getopts d: argument
      do
          #2. 只有到选项为d(-d)时有效，同时将-d后面的参数($OPTARG)赋值给变量debug，表示当前脚本的调试级别。
          case $argument in
          d) debug_level=$OPTARG ;;
          \?) echo "Usage: ./test2.sh -d debug_level"
              exit 1
              ;;
          esac
      done
      #3. 如果debug此时的值为空或者不是0-9之间的数字，给debug变量赋缺省值0.
      if [[ -z $debug_level ||  $debug_level != [0-9] ]]; then
          debug_level=0
      fi
      echo "The current debug_level level is $debug_level."
      echo -n "Tell me your name."
      read name
      name=`echo $name | tr [a-z] [A-Z]`
      if [ $name = "STEPHEN" ];then
          #4. 根据当前脚本的调试级别判断是否输出其后的调试信息，此时当debug_level > 0时输出该调试信息。
          test $debug_level -gt 0 && echo "This is stephen."
          #do something you want here.
      elif [ $name = "ANN" ]; then
          #5. 当debug_level > 1时输出该调试信息。
          test $debug_level -gt 1 && echo "This is ann."
          #do something you want here.
      else
          #6. 当debug_level > 2时输出该调试信息。
          test $debug_level -gt 2 && echo "This is others."
          #do any other else.
      fi
      CTRL+D
      /> ./test2.sh
      Usage: ./test2.sh -d debug_level
      /> ./test2.sh -d 1
      The current debug level is 1.
      Tell me your name. ann
      /> ./test2.sh -d 2
      The current debug level is 2.
      Tell me your name. ann
      This is ann.

三、判断参数是否为数字：

      有些时候我们需要验证脚本的参数或某些变量的值是否为数字，如果不是则需要需要给出提示，并退出脚本。
      /> cat > test3.sh
      #!/bin/sh
      #1. $1是脚本的第一个参数，这里作为awk命令的第一个参数传入给awk命令。
      #2. 由于没有输入文件作为输入流，因此这里只是在BEGIN块中完成。
      #3. 在awk中ARGV数组表示awk命令的参数数组，ARGV[0]表示命令本身，ARGV[1]表示第一个参数。
      #4. match是awk的内置函数，返回值为匹配的正则表达式在字符串中(ARGV[1])的起始位置，没有找到返回0。
      #5. 正则表达式的写法已经保证了匹配的字符串一定是十进制的正整数，如需要浮点数或负数，仅需修改正则即可。
      #6. awk执行完成后将结果返回给isdigit变量，并作为其初始化值。
      #7. isdigit=`echo $1 | awk '{ if (match($1, "^[0-9]+$") != 0) print "true"; else print "false" }' `
      #8. 上面的写法也能实现该功能，但是由于有多个进程参与，因此效率低于下面的写法。
      isdigit=`awk 'BEGIN { if (match(ARGV[1],"^[0-9]+$") != 0) print "true"; else print "false" }' $1`
      if [[ $isdigit == "true" ]]; then
          echo "This is numeric variable."
          number=$1
      else
          echo "This is not numeric variable."
          number=0
      fi
      CTRL+D
      /> ./test3.sh 12
      This is numeric variable.
      /> ./test3.sh 12r
      This is not numeric variable.

四、判断整数变量的奇偶性：

      为了简化问题和突出重点，这里我们假设脚本的输入参数一定为合法的整数类型，因而在脚本内部将不再进行参数的合法性判断。
      /> cat > test4.sh
      #!/bin/sh
      #1. 这里的重点主要是sed命令中正则表达式的写法，它将原有的数字拆分为两个模式(用圆括号拆分)，一个前面的所有高位数字，另一个是最后一位低位数字，之后再用替换符的方式(\2)，将原有数字替换为只有最后一位的数字，最后将结果返回为last_digit变量。
      last_digit=`echo $1 | sed 's/\(.*\)\(.\)$/\2/'`
      #2. 如果last_digit的值为0,2,4,6,8，就表示其为偶数，否则为奇数。
      case $last_digit in
      0|2|4|6|8)
          echo "This is an even number." ;;
      *)
          echo "This is not an even number." ;;
      esac
      CTRL+D
      /> ./test4.sh 34
      This is an even number.
      /> ./test4.sh 345
      This is not an even number.
       
五、将Shell命令赋值给指定变量，以保证脚本的移植性：

      有的时候当我们在脚本中执行某个命令时，由于操作系统的不同，可能会导致命令所在路径的不同，甚至是命令名称或选项的不同，为了保证脚本具有更好的平台移植性，我们可以将该功能的命令赋值给指定的变量，之后再使用该命令时，直接使用该变量即可。这样在今后增加更多OS时，我们只需为该变量基于新系统赋予不同的值即可，否则我们将不得不修改更多的地方，这样很容易导致因误修改而引发的Bug。
      /> cat > test5.sh
      #!/bin/sh
      #1. 通过uname命令获取当前的系统名称，之后再根据OS名称的不同，给PING变量赋值不同的ping命令的全称。
      osname=`uname -s`
      #2. 可以在case的条件中添加更多的操作系统名称。
      case $osname in
      "Linux")
          PING=/usr/sbin/ping ;;
      "FreeBSD")
          PING=/sbin/ping ;;
      "SunOS")
          PING=/usr/sbin/ping ;;
      *)
          ;;
      esac
      CTRL+D
      /> . ./test5.sh
      /> echo $PING
      /usr/sbin/ping
   
六、获取当前时间距纪元时间(1970年1月1日)所经过的天数：

      在获取两个时间之间的差值时，需要考虑很多问题，如闰年、月份中不同的天数等。然而如果我们能够确定两个时间点之间天数的差值，那么再计算时分秒的差值时就非常简单了。在系统提供的C语言函数中，获取的时间值是从1970年1月1日0点到当前时间所流经的秒数，如果我们基于此计算两个时间之间天数的差值，将会大大简化我们的计算公式。
      /> cat > test6.sh
      #!/bin/sh
      #1. 将date命令的执行结果(秒 分 小时 日 月 年)赋值给数组变量DATE。
      declare -a DATE=(`date +"%S %M %k %d %m %Y"`)
      #2. 为了提高效率，这个直接给出1970年1月1日到新纪元所流经的天数常量。
      epoch_days=719591
      #3. 从数组中提取各个时间部分值。
      year=${DATE[5]}
      month=${DATE[4]}
      day=${DATE[3]}
      hour=${DATE[2]}
      minute=${DATE[1]}
      second=${DATE[0]}
      #4. 当月份值为1或2的时候，将月份变量的值加一，否则将月份值加13，年变量的值减一，这样做主要是因为后面的公式中取月平均天数时的需要。
      if [ $month -gt 2 ]; then
          month=$((month+1))
      else
          month=$((month+13))
          year=$((year-1))
      fi
      #5. year变量参与的运算是需要考虑闰年问题的，该问题可以自行去google。
      #6. month变量参与的运算主要是考虑月平均天数。
      #7. 计算结果为当前日期距新世纪所流经的天数。
      today_days=$(((year*365)+(year/4)-(year/100)+(year/400)+(month*306001/10000)+day))
      #8. 总天数减去纪元距离新世纪的天数即可得出我们需要的天数了。
      days_since_epoch=$((today_days-epoch_days))
      echo $days_since_epoch
      seconds_since_epoch=$(((days_since_epoch*86400)+(hour*3600)+(minute*60)+second))
      echo $seconds_since_epoch
      CTRL+D
      /> . ./test6.sh
      15310
      1322829080
      需要说明的是，推荐将该脚本的内容放到一个函数中，以便于我们今后计算类似的时间数据时使用。  
     
七、非直接引用变量：

      在Shell中提供了三种为标准(直接)变量赋值的方式：
      1. 直接赋值。
      2. 存储一个命令的输出。
      3. 存储某类型计算的结果。
      然而这三种方式都是给已知变量名的变量赋值，如name=Stephen。但是在有些情况下，变量名本身就是动态的，需要依照运行的结果来构造变量名，之后才是为该变量赋值。这种变量被成为动态变量，或非直接变量。
      /> cat > test7.sh
      #!/bin/sh
      work_dir=`pwd`
      #1. 由于变量名中不能存在反斜杠，因此这里需要将其替换为下划线。
      #2. work_dir和file_count两个变量的变量值用于构建动态变量的变量名。
      work_dir=`echo $work_dir | sed 's/\//_/g'`
      file_count=`ls | wc -l`
      #3. 输出work_dir和file_count两个变量的值，以便确认这里的输出结果和后面构建的命令名一致。
      echo "work_dir = " $work_dir
      echo "file_count = " $file_count
      #4. 通过eval命令进行评估，将变量名展开，如${work_dir}和$file_count，并用其值将其替换，如果不使用eval命令，将不会完成这些展开和替换的操作。最后为动态变量赋值。
      eval BASE${work_dir}_$file_count=$(ls $(pwd) | wc -l)
      #5. 先将echo命令后面用双引号扩住的部分进行展开和替换，由于是在双引号内，仅完成展开和替换操作即可。
      #6. echo命令后面的参数部分，先进行展开和替换，使其成为$BASE_root_test_1动态变量，之后在用该变量的值替换该变量本身作为结果输出。
      eval echo "BASE${work_dir}_$file_count = " '$BASE'${work_dir}_$file_count
      CTRL+D
      /> . ./test7.sh
      work_dir =  _root_test
      file_count =  1
      BASE_root_test_1 = 1
   
八、在循环中使用管道的技巧：

      在Bash Shell中，管道的最后一个命令都是在子Shell中执行的。这意味着在子Shell中赋值的变量对父Shell是无效的。所以当我们将管道输出传送到一个循环结构，填入随后将要使用的变量，那么就会产生很多问题。一旦循环完成，其所依赖的变量就不存在了。
      /> cat > test8_1.sh
      #!/bin/sh
      #1. 先将ls -l命令的结果通过管道传给grep命令作为管道输入。
      #2. grep命令过滤掉包含total的行，之后再通过管道将数据传给while循环。
      #3. while read line命令从grep的输出中读取数据。注意，while是管道的最后一个命令，将在子Shell中运行。
      ls -l | grep -v total | while read line
      do
          #4. all变量是在while块内声明并赋值的。
          all="$all $line"
          echo $line
      done
      #5. 由于上面的all变量在while内声明并初始化，而while内的命令都是在子Shell中运行，包括all变量的赋值，因此该变量的值将不会传递到while块外，因为块外地命令是它的父Shell中执行。
      echo "all = " $all
      CTRL+D
      /> ./test8_1.sh
      -rw-r--r--.  1 root root 193 Nov 24 11:25 outfile
      -rwxr-xr-x. 1 root root 284 Nov 24 10:01 test7.sh
      -rwxr-xr-x. 1 root root 108 Nov 24 12:48 test8_1.sh
      all =

      为了解决该问题，我们可以将while之前的命令结果先输出到一个临时文件，之后再将该临时文件作为while的重定向输入，这样while内部和外部的命令都将在同一个Shell内完成。
      /> cat > test8_2.sh
      #!/bin/sh
      #1. 这里我们已经将命令的结果重定向到一个临时文件中。
      ls -l | grep -v total > outfile
      while read line
      do
          #2. all变量是在while块内声明并赋值的。
          all="$all $line"
          echo $line
          #3. 通过重定向输入的方式，将临时文件中的内容传递给while循环。
      done < outfile
      #4. 删除该临时文件。
      rm -f outfile
      #5. 在while块内声明和赋值的all变量，其值在循环外部仍然有效。
      echo "all = " $all
      CTRL+D
      /> ./test8_2.sh
      -rw-r--r--.  1 root root   0 Nov 24 12:58 outfile
      -rwxr-xr-x. 1 root root 284 Nov 24 10:01 test7.sh
      -rwxr-xr-x. 1 root root 140 Nov 24 12:58 test8_2.sh
      all =  -rwxr-xr-x. 1 root root 284 Nov 24 10:01 test7.sh -rwxr-xr-x. 1 root root 135 Nov 24 13:16 test8_2.sh

      上面的方法只是解决了该问题，然而却带来了一些新问题，比如临时文件的产生容易导致性能问题，以及在脚本异常退出时未能及时删除当前使用的临时文件，从而导致生成过多的垃圾文件等。下面将再介绍一种方法，该方法将同时解决以上两种方法同时存在的问题。该方法是通过HERE-Document的方式来替代之前的临时文件方法。
      /> cat > test8_3.sh
      #!/bin/sh
      #1. 将命令的结果传给一个变量    
      OUTFILE=`ls -l | grep -v total`
      while read line
      do
          all="$all $line"
          echo $line
      done <<EOF
      #2. 将该变量作为该循环的HERE文档输入。
      $OUTFILE
      EOF
      #3. 在循环外部输出循环内声明并初始化的变量all的值。
      echo "all = " $all
      CTRL+D
      /> ./test8_3.sh
      -rwxr-xr-x. 1 root root 284 Nov 24 10:01 test7.sh
      -rwxr-xr-x. 1 root root 135 Nov 24 13:16 test8_3.sh
      all =  -rwxr-xr-x. 1 root root 284 Nov 24 10:01 test7.sh -rwxr-xr-x. 1 root root 135 Nov 24 13:16 test8_3.sh
   
九、自链接脚本：

      通常而言，我们是通过脚本的命令行选项来确定脚本的不同行为，告诉它该如何操作。这里我们将介绍另外一种方式来完成类似的功能，即通过脚本的软连接名来帮助脚本决定其行为。
      /> cat > test9.sh
      #!/bin/sh
      #1. basename命令将剥离脚本的目录信息，只保留脚本名，从而确保在相对路径的模式下执行也没有任何差异。
      #2. 通过sed命令过滤掉脚本的扩展名。
      dowhat=`basename $0 | sed 's/\.sh//'`
      #3. 这里的case语句只是为了演示方便，因此模拟了应用场景，在实际应用中，可以为不同的分支执行不同的操作，或将某些变量初始化为不同的值和状态。
      case $dowhat in
      test9)
          echo "I am test9.sh"
          ;;
      test9_1)
          echo "I am test9_1.sh."
          ;;
      test9_2)
          echo "I am test9_2.sh."
          ;;
      *)
          echo "You are illegal link file."
          ;;
      esac
      CTRL+D
      /> chmod a+x test9.sh
      /> ln -s test9.sh test9_1.sh
      /> ln -s test9.sh test9_2.sh
      /> ls -l
      lrwxrwxrwx. 1 root root   8 Nov 24 14:32 test9_1.sh -> test9.sh
      lrwxrwxrwx. 1 root root   8 Nov 24 14:32 test9_2.sh -> test9.sh
      -rwxr-xr-x. 1 root root 235 Nov 24 14:35 test9.sh
      /> ./test9.sh
      I am test9.sh.
      /> ./test9_1.sh
      I am test9_1.sh.
      /> ./test9_2.sh
      I am test9_2.sh.

十、Here文档的使用技巧：

      在命令行交互模式下，我们通常希望能够直接输入更多的信息，以便当前的命令能够完成一定的自动化任务，特别是对于那些支持自定义脚本的命令来说，我们可以将脚本作为输入的一部分传递给该命令，以使其完成该自动化任务。
      #1. 通过sqlplus以dba的身份登录Oracle数据库服务器。
      #2. 在通过登录后，立即在sqlplus中执行oracle的脚本CreateMyTables和CreateMyViews。
      #3. 最后执行sqlplus的退出命令，退出sqlplus。自动化工作完成。
      /> sqlplus "/as sysdba" <<-SQL
      > @CreateMyTables
      > @CreateMyViews
      > exit
      > SQL
        
十一、获取进程的运行时长(单位: 分钟)：

      在进程监控脚本中，我们通常需要根据脚本的参数来确定有哪些性能参数将被收集，当这些性能参数大于最高阈值或小于最低阈值时，监控脚本将根据实际的情况，采取预置的措施，如邮件通知、直接杀死进程等，这里我们给出的例子是收集进程运行时长性能参数。
      ps命令的etime值将给出每个进程的运行时长，其格式主要为以下三种：
      1. minutes:seconds，如20:30
      2. hours:minutes:seconds，如1:20:30
      3. days-hours:minute:seconds，如2-18:20:30
      该脚本将会同时处理这三种格式的时间信息，并最终转换为进程所流经的分钟数。
      /> cat > test11.sh
      #!/bin/sh
      #1. 通过ps命令获取所有进程的pid、etime和comm数据。
      #2. 再通过grep命令过滤，只获取init进程的数据记录，这里我们可以根据需要替换为自己想要监控的进程名。
      #3. 输出结果通常为：1 09:42:09 init
      pid_string=`ps -eo pid,etime,comm | grep "init" | grep -v grep`
      #3. 从这一条记录信息中抽取出etime数据，即第二列的值09:42:09，并赋值给exec_time变量。
      exec_time=`echo $pid_string | awk '{print $2}'`
      #4. 获取exec_time变量的时间组成部分的数量，这里是3个部分，即时:分:秒，是上述格式中的第二种。
      time_field_count=`echo $exec_time | awk -F: '{print NF}'`
      #5. 从exec_time变量中直接提取分钟数，即倒数第二列的数据(42)。
      count_of_minutes=`echo $exec_time | awk -F: '{print $(NF-1)}'`
    
      #6. 判断当前exec_time变量存储的时间数据是属于以上哪种格式。
      #7. 如果是第一种，那么天数和小时数均为0。
      #8. 如果是后两种之一，则需要继续判断到底是第一种还是第二种，如果是第二种，其小时部分将不存在横线(-)分隔符分隔天数和小时数，否则需要将这两个时间字段继续拆分，以获取具体的天数和小时数。对于第二种，天数为0.
      if [ $time_field_count -lt 3 ]; then
          count_of_hours=0
          count_of_days=0
      else
          count_of_hours=`echo $exec_time | awk -F: '{print $(NF-2)}'`
          fields=`echo $count_of_hours | awk -F- '{print NF}'`
          if [ $fields -ne 1 ]; then
              count_of_days=`echo $count_of_hours | awk -F- '{print $1}'`
              count_of_hours=`echo $count_of_hours | awk -F- '{print $2}'`
          else
              count_of_days=0
          fi
      fi
      #9. 通过之前代码获取的各个字段值，计算出该进程实际所流经的分钟数。
      #10. bc命令是计算器命令，可以将echo输出的数学表达式计算为最终的数字值。
      elapsed_minutes=`echo "$count_of_days*1440+$count_of_hours*60+$count_of_minutes" | bc`
      echo "The elapsed minutes of init process is" $elapsed_minutes "minutes."
      CTRL+D
      /> ./test11.sh
      The elapsed minutes of init process is 577 minutes.
   
十二、模拟简单的top命令：
    
      这里用脚本实现了一个极为简单的top命令。为了演示方便，我们在脚本中将很多参数都写成硬代码，你可以根据需要更换这些参数，或者用更为灵活的方式替换现有的实现。
      /> cat > test12.sh
      #!/bin/sh
      #1. 将ps命令的title赋值给一个变量，这样在每次输出时，直接打印该变量即可。
      header=`ps aux | head -n 1`
      #2. 这里是一个无限循环，等价于while true
      #3. 每次循环先清屏，之后打印uptime命令的输出。
      #4. 输出ps的title。
      #5. 这里需要用sed命令删除ps的title行，以避免其参与sort命令的排序。
      #6. sort先基于CPU%倒排，再基于owner排序，最后基于pid排序，最后再将结果输出给head命令，仅显示前20行的数据。
      #7. 每次等待5秒后刷新一次。
     while :
      do
          clear
          uptime
          echo "$header"
          ps aux | sed -e 1d | sort -k3nr -k1,1 -k2n | head -n 20
          sleep 5
      done
      CTRL+D    
      /> ./test12.sh
      21:55:07 up 13:42,  2 users,  load average: 0.00, 0.00, 0.00
      USER       PID %CPU %MEM    VSZ   RSS   TTY      STAT START   TIME   COMMAND
      root      6408     2.0      0.0   4740   932   pts/2    R+    21:45     0:00   ps aux
      root      1755     0.2      2.0  96976 21260   ?        S      08:14     2:08   nautilus
      68        1195     0.0      0.4   6940   4416    ?        Ss    08:13     0:00   hald
      postfix   1399    0.0      0.2  10312  2120    ?        S      08:13     0:00   qmgr -l -t fifo -u
      postfix   6021    0.0      0.2  10244  2080    ?        S      21:33     0:00   pickup -l -t fifo -u
      root         1       0.0      0.1   2828   1364    ?        Ss     08:12    0:02   /sbin/init
      ... ...

十三、格式化输出指定用户的当前运行进程：

      在这个例子中，我们通过脚本参数的形式，将用户列表传递给该脚本，脚本在读取参数后，以树的形式将用户列表中用户的所属进程打印出来。
      /> cat > test13.sh
      #!/bin/sh
      #1. 循环读取脚本参数，构造egrep可以识别的用户列表变量(基于grep的扩展正则表达式)。
      #2. userlist变量尚未赋值，则直接使用第一个参数为它赋值。
      #3. 如果已经赋值，且脚本参数中存在多个用户，这里需要在每个用户名之间加一个竖线，在egrep中，竖线是分割的元素之间是或的关系。
      #4. shift命令向左移动一个脚本的位置参数，这样可以使循环中始终操作第一个参数。
      while [ $# -gt 0 ]
      do
          if [ -z "$userlist" ]; then
              userlist="$1"
          else
              userlist="$userlist|$1"
          fi
           shift
      done
      #5. 如果没有用户列表，则搜索所有用户的进程。
      #6. "^ *($userlist) ": 下面的调用方式，该正则的展开形式为"^ *(root|avahi|postfix|rpc|dbus) "。其含义为，以0个或多个空格开头，之后将是root、avahi、postfix、rpc或dbus之中的任何一个字符串，后面再跟随一个空格。
      if [ -z "$userlist" ]; then
          userlist="."
      else
          userlist="^ *($userlist) "
      fi
      #7. ps命令输出所有进程的user和命令信息，将结果传递给sed命令，sed将删除ps的title部分。
      #8. egrep过滤所有进程记录中，包含指定用户列表的进程记录，再将过滤后的结果传递给sort命令。
      #9. sort命令中的-b选项将忽略前置空格，并以user，再以进程名排序，将结果传递个uniq命令。
      #10.uniq命令将合并重复记录，-c选项将会使每条记录前加重复的行数。
      #11.第二个sort将再做一次排序，先以user，再以重复计数由大到小，最后以进程名排序。将结果传给awk命令。
      #12.awk命令将数据进行格式化，并删除重复的user。
      ps -eo user,comm | sed -e 1d | egrep "$userlist" |
          sort -b -k1,1 -k2,2 | uniq -c | sort -b -k2,2 -k1nr,1 -k3,3 |
              awk ' { user = (lastuser == $2) ? " " : $2;
                        lastuser = $2;
                        printf("%-15s\t%2d\t%s\n",user,$1,$3)
              }'
      CTRL+D
      /> ./test13.sh root avahi postfix rpc dbus
      avahi             2      avahi-daemon
      dbus             1      dbus-daemon
      postfix          1      pickup
                          1      qmgr
      root              5      mingetty
                          3      udevd
                          2      sort
                          2      sshd
      ... ...
      rpc               1      rpcbind

十四、用脚本完成which命令的基本功能：

      我们经常会在脚本中调用其他的应用程序，为了保证脚本具有更好的健壮性，以及错误提示的准确性，我们可能需要在执行前验证该命令是否存在，或者说是否可以被执行。这首先要确认该命令是否位于PATH变量包含的目录中，再有就是该文件是否为可执行文件。
      /> cat > test14.sh
      #!/bin/sh
      #1. 该函数用于判断参数1中的命令是否位于参数2所包含的目录列表中。需要说明的是，函数里面的$1和$2是指函数的参数，而不是脚本的参数，后面也是如此。
      #2. cmd=$1和path=$2，将参数赋给有意义的变量名，是一个很好的习惯。
      #3. 由于PATH环境变量中，目录之间的分隔符是冒号，因此这里需要临时将IFS设置为冒号，函数结束后再还原。
      #4. 在for循环中，逐个变量目录列表中的目录，以判断该命令是否存在，且为可执行程序。
      isInPath() {
          cmd=$1        path=$2      result=1
          oldIFS=$IFS   IFS=":"
          for dir in $path
          do
              if [ -x $dir/$cmd ]; then
                  result=0
              fi
          done
          IFS=oldifs
          return $result
      }
      #5. 检查命令是否存在的主功能函数，先判断是否为绝对路径，即$var变量的第一个字符是否为/，如果是，再判断它是否有可执行权限。
      #6. 如果不是绝对路径，通过isInPath函数判断是否该命令在PATH环境变量指定的目录中。
      checkCommand() {
          var=$1
          if [ ! -z "$var" ]; then
              if [ "${var:0:1}" = "/" ]; then
                  if [ ! -x $var ]; then
                      return 1
                  fi
              elif ! isInPath $var $PATH ; then
                  return 2
              fi
          fi
      }
      #7. 脚本参数的合法性验证。
      if [ $# -ne 1 ]; then
          echo "Usage: $0 command" >&2;
      fi
      #8. 根据返回值打印不同的信息。我们可以在这里根据我们的需求完成不同的工作。
      checkCommand $1
      case $? in
      0) echo "$1 found in PATH." ;;
      1) echo "$1 not found or not executable." ;;
      2) echo "$1 not found in PATH." ;;
      esac
      exit 0
      CTRL+D
      /> ./test14.sh echo
      echo found in PATH.
      /> ./test14.sh MyTest
      MyTest not found in PATH.
      /> ./test14.sh /bin/MyTest
      /bin/MyTest not found or not executable.


十五、验证输入信息是否合法：

      这里给出的例子是验证用户输入的信息是否都是数字和字母。需要说明的是，之所以将其收集到该系列中，主要是因为它实现的方式比较巧妙。
      /> cat > test15.sh
      #!/bin/sh
      echo -n "Enter your input: "
      read input
      #1. 事实上，这里的巧妙之处就是先用sed替换了非法部分，之后再将替换后的结果与原字符串比较。这种写法也比较容易扩展。    
      parsed_input=`echo $input | sed 's/[^[:alnum:]]//g'`
      if [ "$parsed_input" != "$input" ]; then
          echo "Your input must consist of only letters and numbers."
      else
          echo "Input is OK."
      fi
      CTRL+D
      /> ./test15.sh
      Enter your input: hello123
      Input is OK.
      /> ./test15.sh
      Enter your input: hello world
      Your input must consist of only letters and numbers.

十六、整数验证：

      整数的重要特征就是只是包含数字0到9和负号(-)。
      /> cat > test16.sh
      #!/bin/sh
      #1. 判断变量number的第一个字符是否为负号(-)，如果只是则删除该负号，并将删除后的结果赋值给left_number变量。
      #2. "${number#-}"的具体含义，可以参考该系列博客中"Linux Shell常用技巧(十一)"，搜索关键字"变量模式匹配运算符"即可。
      number=$1
      if [ "${number:0:1}" = "-" ]; then
          left_number="${number#-}"
      else
          left_number=$number
      fi
      #3. 将left_number变量中所有的数字都替换掉，因此如果返回的字符串变量为空，则表示left_number所包含的字符均为数字。
      nodigits=`echo $left_number | sed 's/[[:digit:]]//g'`
      if [ "$nodigits" != "" ]; then
          echo "Invalid number format!"
      else
          echo "You are valid number."
      fi
      CTRL+D
      /> ./test16.sh -123
      You are valid number.
      /> ./test16.sh 123e
      Invalid number format!
   
十七、判断指定的年份是否为闰年：

      这里我们先列出闰年的规则:
      1. 不能被4整除的年一定不是闰年；
      2. 可以同时整除4和400的年一定是闰年；
      3. 可以整除4和100，但是不能整除400的年，不是闰年；
      4. 其他可以整除的年都是闰年。
      #!/bin/sh   
      year=$1
      if [ "$((year % 4))" -ne 0 ]; then
          echo "This is not a leap year."
          exit 1
      elif [ "$((year % 400))" -eq 0 ]; then
          echo "This is a leap year."
          exit 0
      elif [ "$((year % 100))" -eq 0 ]; then
          echo "This is not a leap year."
          exit 1
      else
          echo "This is a leap year."
          exit 0
      fi
      CTRL+D
      /> ./test17.sh 1933
      This is not a leap year.
      /> ./test17.sh 1936
      This is a leap year.
           
十八、将单列显示转换为多列显示：

      我们经常会在显示时将单行的输出，格式化为多行的输出，通常情况下，为了完成该操作，我们将加入更多的代码，将输出的结果存入数组或临时文件，之后再重新遍历它们，从而实现单行转多行的目的。在这里我们介绍一个使用xargs命令的技巧，可以用更简单、更高效的方式来完成该功能。   
      /> cat > test18.sh
      #!/bin/sh
      #1. passwd文件中，有可能在一行内出现一个或者多个空格字符，因此在直接使用cat命令的结果时，for循环会被空格字符切开，从而导致一行的文本被当做多次for循环的输入，这样我们不得不在sed命令中，将cat输出的每行文本进行全局替换，将空格字符替换为%20。事实上，我们当然可以将cat /etc/passwd的输出以管道的形式传递给cut命令，这里之所以这样写，主要是为了演示一旦出现类似的问题该如果巧妙的处理。
      #2. 这里将for循环的输出以管道的形式传递给sort命令，sort命令将基于user排序。
      #3. -xargs -n 2是这个技巧的重点，它将sort的输出进行合并，-n选项后面的数字参数将提示xargs命令将多少次输出合并为一次输出，并传递给其后面的命令。在本例中，xargs会将从sort得到的每两行数据合并为一行，中间用空格符分离，之后再将合并后的数据传递给后面的awk命令。事实上，对于awk而言，你也可以简单的认为xargs减少了对它(awk)的一半调用。
      #4. 如果打算在一行内显示3行或更多的行，可以将-n后面的数字修改为3或其它更高的数字。你还可以修改awk中的print命令，使用更为复杂打印输出命令，以得到更为可读的输出效果。
      for line in `cat /etc/passwd | sed 's/ /%20/g'`
      do
          user=`echo $line | cut -d: -f1`
          echo $user
      done | \
          sort -k1,1 | \
          xargs -n 2 | \
          awk '{print $1, $2}'
      CTRL+D
      /> ./test18.sh
      abrt adm
      apache avahi
      avahi-autoipd bin
      daemon daihw
      dbus ftp
      games gdm
      gopher haldaemon
      halt lp
      mail nobody
      ntp operator
      postfix pulse
      root rtkit
      saslauth shutdown
      sshd sync
      tcpdump usbmuxd
      uucp vcsa
      
十九、将文件的输出格式化为指定的宽度：

      在这个技巧中，不仅包含了如何获取和文件相关的详细信息，如行数，字符等，而且还可以让文件按照指定的宽度输出。这种应用在输出帮助信息、License相关信息时还是比较有用的。
      /> cat > test19.sh
      #!/bin/sh
      #1. 这里我们将缺省宽度设置为75，如果超过该宽度，将考虑折行显示，否则直接在一行中全部打印输出。这里只是为了演示方便，事实上，你完全可以将该值作为脚本或函数的参数传入，那样你将会得到更高的灵活性。    
      my_width=75
      #2. for循环的读取列表来自于脚本的参数。
      #3. 在获取lines和chars变量时，sed命令用于过滤掉多余的空格字符。
      #4. 在if的条件判断中${#line}用于获取line变量的字符长度，这是Shell内置的规则。
      #5. fmt -w 80命令会将echo输出的整行数据根据其命令选项指定的宽度(80个字符)进行折行显示，再将折行后的数据以多行的形式传递给sed命令。
      #6. sed在收到fmt命令的格式化输出后，将会在折行后的第一行头部添加两个空格，在其余行的头部添加一个加号和一个空格以表示差别。
      for input; do
          lines=`wc -l < $input | sed 's/ //g'`
          chars=`wc -c < $input | sed 's/ //g'`
          owner=`ls -l $input | awk '{print $3}'`
          echo "-------------------------------------------------------------------------------"
          echo "File $input ($lines lines, $chars characters, owned by $owner):"
          echo "-------------------------------------------------------------------------------"
          while read line; do
              if [ ${#line} -gt $my_width ]; then
                  echo "$line" | fmt -w 80 | sed -e '1s/^/  /' -e '2,$s/^/+ /'
              else
                  echo "  $line"
              fi
          done < $input
          echo "-------------------------------------------------------------------------------"
      done | more
      CTRL+D
      /> ./test19.sh testfile
      -------------------------------------------------------------------------------
      File testfile.dat (3 lines, 645 characters, owned by root):
      -------------------------------------------------------------------------------
         The PostgreSQL Global Development Group today released updates for all
      + active branches of the PostgreSQL object-relational database system,
      + including versions 9.1.2, 9.0.6, 8.4.10, 8.3.17 and 8.2.23. Users of any of
      + the several affected features in this release, including binary replication,
      + should update their PostgreSQL installations as soon as possible.
         This is also the last update for PostgreSQL 8.2, which is now End-Of-Life
      + (EOL). Users of version 8.2 should plan to upgrade their PostgreSQL
      + installations to 8.3 or later within the next couple of months. For more
      + information, see our Release Support Policy.
         This is just a test file.
      -------------------------------------------------------------------------------
       
二十、监控指定目录下磁盘使用空间过大的用户：

      在将Linux用作文件服务器时，所有的注册用户都可以在自己的主目录下存放各种类型和大小的文件。有的时候，有些用户的占用空间可能会明显超过其他人，这时就需要管理员可以及时发现这一异常使用状况，并根据实际情况作出应对处理。
      /> cat > test20.sh
      #!/bin/sh
      #1. 该脚本仅用于演示一种处理技巧，其中很多阈值都是可以通过脚本参来初始化的，如limited_qutoa和dirs等变量。
      limited_quota=200
      dirs="/home /usr /var"
      #2. 以冒号作为分隔符，截取passwd文件的第一和第三字段，然后将输出传递给awk命令。
      #3. awk中的$2表示的是uid，其中1-99是系统保留用户，>=100的uid才是我们自己创建的用户，awk通过print输出所有的用户名给for循环。
      #4. 注意echo命令的输出是由八个单词构成，同时由于-n选项，echo命令并不输出换行符。
      #5. 之所以使用find命令，也是为了考虑以点(DOT)开头的隐藏文件。这里的find将在指定目录列表内，搜索指定用户的，类型为普通文件的文件。并通过-ls选项输出找到文件的详细信息。其中输出的详细信息的第七列为文件大小列。
      #6. 通过awk命令累加find输出的第七列，最后再在自己的END块中将sum的值用MB计算并输出。该命令的输出将会与上面echo命令的输出合并作为for循环的输出传递给后面的awk命令。这里需要指出的是，该awk的输出就是后面awk命令的$9，因为echo仅仅输出的8个单词。
      #7. 从for循环管道获取数据的awk命令，由于awk命令执行的动作是用双引号括起的，所以表示域字段的变量的前缀$符号，需要用\进行转义。变量$limited_quota变量将会自动完成命令替换，从而构成该awk命令的最终动作参数。
      for name in `cut -d: -f1,3 /etc/passwd | awk -F: '$2 > 99 {print $1}'`
      do
          echo -n "User $name exceeds disk quota. Disk Usage is: "
          find $dirs -user $name -type f -ls |\
                awk '{ sum += $7 } END { print sum / (1024*1024) " MB" }'
      done | awk "\$9 > $limited_quota { print \$0 }"
      CTRL+D
      /> ./test20.sh    

二十一、编写一个更具可读性的df命令输出脚本：

      这里我们将以awk脚本的方式来实现df -h的功能。
      /> cat > test21.sh
      #!/bin/sh
      #1. $$表示当前Shell进程的pid。    
      #2. trap信号捕捉是为了保证在Shell正常或异常退出时，仍然能够将该脚本创建的临时awk脚本文件删除。
      awk_script_file="/tmp/scf_tmp.$$"
      trap "rm -f $awk_script_file" EXIT
      #3. 首先需要说明的是，'EOF'中的单引号非常重要，如果忽略他将无法通过编译，这是因为awk的命令动作必须要用单引号扩住。
      #4. awk脚本的show函数中，int(mb * 100) / 100这个技巧是为了保证输出时保留小数点后两位。
      cat << 'EOF' > $awk_script_file
      function show(size) {
          mb = size / 1024;
          int_mb = (int(mb * 100)) / 100;
          gb = mb / 1024;
          int_gb = (int(gb * 100)) / 100;
          if (substr(size,1,1) !~ "[0-9]" || substr(size,2,1) !~ "[0-9]") {
              return size;
          } else if (mb < 1) {
              return size "K";
          } else if (gb < 1) {
              return int_mb "M";
          } else {
              return int_gb "G";
          }
      }
      #5. 在BEGIN块中打印重定义的输出头信息。
      BEGIN {
            printf "%-20s %7s %7s %7s %8s %s\n","FileSystem","Size","Used","Avail","Use%","Mounted"
      }
      #6. !/Filesystem/ 表示过滤掉包含Filesystem的行，即df输出的第一行。其余行中，有个域字段可以直接使用df的输出，有的需要通过show函数的计算，以得到更为可读的显示结果。
      !/Filesystem/ {
          size = show($2);
          used = show($3);
          avail = show($4);
          printf "%-20s %7s %7s %7s %8s %s\n",$1,size,used,avail,$5,$6
      }
      EOF
      df -k | awk -f $awk_script_file
      CTRL+D
      /> ./test12.sh
      FileSystem              Size       Used      Avail     Use%   Mounted
      /dev/sda2              3.84G    2.28G     1.36G      63%   /
      tmpfs                 503.57M     100K 503.47M        1%   /dev/shm
      /dev/sda1             48.41M  35.27M  10.63M      77%   /boot
      /dev/sda3              14.8G 171.47M  13.88G        2%   /home
   
二十二、编写一个用于添加新用户的脚本：

      之所以在这里选择这个脚本，没有更多的用意，只是感觉这里的有些技巧和常识还是需要了解的，如/etc/passwd、/etc/shadow、/etc/group的文件格式等。
      /> cat > test22.sh
      #!/bin/sh
      #1. 初始化和用户添加相关的变量。    
      passwd_file="/etc/passwd"
      shadow_file="/etc/shadow"
      group_file="/etc/group"
      home_root_dir="/home"
      #2. 只有root用户可以执行该脚本。    
      if [ "$(whoami)" != "root" ]; then
          echo "Error: You must be roor to run this command." >&2
          exit 1
      fi
    
      echo "Add new user account to $(hostname)"
      echo -n "login: "
      read login
      #3. 去唯一uid，即当前最大uid值加一。
      uid=`awk -F: '{ if (big < $3 && $3 < 5000) big = $3 } END {print big + 1}' $passwd_file`
      #4. 设定新用户的主目录变量
      home_dir=$home_root_dir/$login
      gid=$uid
      #5. 提示输入和创建新用户相关的信息，如用户全名和主Shell。
      echo -n "full name: "
      read fullname
      echo -n "shell: "
      read shell
      #6. 将输入的信息填充到passwd、group和shadow三个关键文件中。
      echo "Setting up account $login for $fullname..."
      echo ${login}:x:${uid}:${gid}:${fullname}:${home_dir}:$shell >> $passwd_file
      echo ${login}:*:11647:0:99999:7::: >> $shadow_file
      echo "${login}:x:${gid}:$login" >> $group_file
      #7. 创建主目录，同时将新用户的profile模板拷贝到新用户的主目录内。
      #8. 设定该主目录的权限，再将其下所有文件的owner和group设置为新用户。
      #9. 为新用户设定密码。
      mkdir $home_dir
      cp -R /etc/skel/.[a-zA-Z]* $home_dir
      chmod 755 $home_dir
      find $home_dir -print | xargs chown ${login}:${login}
      passwd $login
      exit 0
      CTRL+D
      /> ./test22.sh
      Add new user account to bogon
      login: stephen
      full name: Stephen Liu
      shell: /bin/shell
      Setting up account stephen for Stephen Liu...
      Changing password for user stephen.
      New password:
      Retype new password:
      passwd: all authentication tokens updated successfully.
   
二十三、kill指定用户或指定终端的用户进程：

      这是一台运行Oracle数据库的Linux服务器，现在我们需要对Oracle做一些特殊的优化工作，由于完成此项工作需要消耗更多的系统资源，因此我们不得不杀掉一些其他用户正在运行的进程，以便节省出更多的系统资源，让本次优化工作能够尽快完成。
      /> cat > test23.sh
      #!/bin/sh
      user=""
      tty=""
      #1. 通过读取脚本的命令行选项获取要kill的用户或终端。-t后面的参数表示终端，-u后面的参数表示用户。这两个选项不能同时使用。
      #2. case中的代码对脚本选项进行验证，一旦失败则退出脚本。
      while getopts u:t: opt; do
          case $opt in
          u) if [ "$tty" != "" ]; then
                 echo "-u and -t can not be set at one time."
                 exit 1
              fi
              user=$OPTARG
              ;;
          t)  if [ "$user" != "" ]; then
                 echo "-u and -t can not be set at one time."
                 exit 1
              fi
              tty=$OPTARG
              ;;
          ?) echo "Usage: $0 [-u user|-t tty]" >&2
              exit 1
          esac
      done
      #3. 如果当前选择的是基于终端kill，就用$tty来过滤ps命令的输出，否则就用$user来过滤ps命令的输出。
      #4. awk命令将仅仅打印出pid字段，之后传递给sed命令，sed命令删除ps命令输出的头信息，仅保留后面的进程pids作为输出，并初始化pids数组。
      if [ ! -z "$tty" ]; then
          pids=$(ps cu -t $tty | awk "{print \$2}" | sed '1d')
      else
          pids=$(ps cu -U $user | awk "{print \$2}" | sed '1d')
      fi
      #5. 判断数组是否为空，空则表示没有符合要求的进程，直接退出脚本。
      if [ -z "$pids" ]; then
          echo "No processes matches."
          exit 1
      fi
      #6. 遍历pids数组，逐个kill指定的进程。
      for pid in $pids; do
          echo "Killing process[pid = $pid]... ..."
          kill -9 $pid
      done
      exit 0
      CTRL+D
      /> ./test23.sh -t pts/1
      Killing process[pid = 11875]... ...
      Killing process[pid = 11894]... ...
      /> ./test23.sh -u stephen
      Killing process[pid = 11910]... ...
      Killing process[pid = 11923]... ...
       
二十四、判断用户输入(是/否)的便捷方法：

      对于有些交互式程序，经常需要等待用户的输入，以便确定下一步的执行流程。通常而言，当用户输入"Y/y/Yes/yes"时表示确认当前的行为，而当输入"N/n/No/no"时则表示否定当前的行为。基于这种规则，我们可以实现一个便捷确认方式，即只判断输入的首字母，如果为Y或y，表示确认，如为N或n，则为否定。
      /> cat > test24.sh
      #!/bin/sh
      echo -n "Please type[y/n/yes/no]: "
      read input
      #1. 先转换小写到大写，再通过cut截取第一个字符。
      ret=`echo $input | tr '[a-z]' '[A-Z]' | cut -c1`
    
      if [ $ret = "Y" ]; then
          echo "Your input is Y."
      elif [ $ret = "N" ]; then
          echo "Your input is N."
      else
          echo "Your input is error."
      fi
      CTRL+D
      /> ./test24.sh
      Please type[y/n/yes/no]: y
      Your input is Y.
      /> ./test24.sh
      Please type[y/n/yes/no]: n
      Your input is N.   

二十五、通过FTP下载指定的文件：

      相比于手工调用FTP命令下载文件，该脚本提供了更为方便的操作方式。
      /> cat > test25.sh
      #!/bin/sh
      #1. 测试脚本参数数量的有效性。    
      if [ $# -ne 2 ]; then
          echo "Usage: $0 ftp://... username" >&2
          exit 1
      fi
      #2. 获取第一个参数的前六个字符，如果不是"ftp://"，则视为非法FTP URL格式。这里cut的-c选项表示按照字符的方式截取第一到第六个字符。
      header=`echo $1 | cut -c1-6`
      if [ "$header" != "ftp://" ]; then
          echo "$0: Invalid ftp URL." >&2
          exit 1
      fi
      #3. 合法ftp URL的例子：ftp://ftp.myserver.com/download/test.tar
      #4. 针对上面的URL示例，cut命令通过/字符作为分隔符，这样第三个域字段表示server(ftp.myserver.com)。
      #5. 在截取filename时，cut命令也是通过/字符作为分隔符，但是"-f4-"将获取从第四个字段开始的后面所有字段(download/test.tar)。
      #6. 通过basename命令获取filename的文件名部分。
      server=`echo $1 | cut -d/ -f3`
      filename=`echo $1 | cut -d/ -f4-`
      basefile=`basename $filename`
      ftpuser=$2
      #7. 这里需要调用stty -echo，以便后面的密码输入不会显示，在输入密码之后，需要再重新打开该选项，以保证后面的输入可以恢复显示。
      #8. echo ""，是模拟一次换换。
      echo -n "Password for $ftpuser: "
      stty -echo
      read password
      stty echo
      echo ""
      #9. 通过HERE文档，批量执行ftp命令。
      echo ${0}: Downloading $baseile from server $server.
      ftp -n << EOF
      open $server
      user $ftpuser $password
      get $filename $basefile
      quit
      EOF
      #10.Shell内置变量$?表示上一个Shell进程的退出值，0表示成功执行，其余值均表示不同原因的失败。
      if [ $? -eq 0 ]; then
          ls -l $basefile
      fi
      exit 0
      CTRL+D
      /> ./test25.sh  ftp://ftp.myserver.com/download/test.tar stephen
      Password for stephen:
      ./test25.sh: Downloading from server ftp.myserver.com.
      -rwxr-xr-x. 1 root root 678 Dec  9 11:46 test.tar

二十六、文件锁定：

      在工业应用中，有些来自于工业设备的文件将会被放到指定的目录下，由于这些文件需要再被重新格式化后才能被更高层的软件进行处理。而此时负责处理的脚本程序极有可能是多个实例同时运行，因此这些实例之间就需要一定的同步，以避免多个实例同时操作一个文件而造成的数据不匹配等问题的发生。文件锁定命令可以帮助我们实现这一同步逻辑。
      /> cat > test26.sh
      #!/bin/sh
      #1. 这里需要先确认flock命令是否存在。
      if [ -z $(which flock) ]; then
          echo "flock doesn't exist."
          exit 1
      fi
      #2. flock中的-e选项表示对该文件加排它锁，-w选项表示如果此时文件正在被加锁，当前的flock命令将等待20秒，如果能锁住该文件，就继续执行，否则退出该命令。
      #3. 这里锁定的文件是/var/lock/lockfile1，-c选项表示，如果成功锁定，则指定其后用双引号括起的命令，如果是多个命令，可以用分号分隔。
      #4. 可以在两个终端同时启动该脚本，然后观察脚本的输出，以及lockfile1文件的内容。
      flock -e -w 20 /var/lock/lockfile1 -c "sleep 10;echo `date` | cat >> /var/lock/lockfile1"
      if [ $? -ne 0 ]; then
          echo "Fail."
          exit 1
      else
          echo "Success."
          exit 0
      fi
      CTRL+D
   
二十七、用小文件覆盖整个磁盘：

      假设我们现在遇到这样一个问题，公司的关键资料copy到测试服务器上了，在直接将其删除后，仍然担心服务器供应商可以将其恢复，即便是通过fdisk进行重新格式化，也仍然存在被恢复的风险，鉴于此，我们需要编写一个脚本，创建很多小文件(5MB左右)，之后不停在关键资料所在的磁盘中复制该文件，以使Linux的inode无法再被重新恢复，为了达到这里效果，我们需要先构造该文件，如：
      /> find . -name "*" > testfile
      /> ls -l testfile
      -rwxr-xr-x. 1 root root 5123678 Dec  9 11:46 testfile
      /> cat > test27.sh
      #!/bin/sh
      #1. 初始化计数器变量，其中max的值是根据当前需要填充的磁盘空间和testfile的大小计算出来的。
      counter=0
      max=2000000
      remainder=0
      #2. 每次迭代counter变量都自增一，以保证每次生成不同的文件。当该值大于最大值时退出。
      #3. 对计数器变量counter按1000取模，这样可以在每生成1000个文件时打印一次输出，以便看到覆盖的进度，输出时间则便于预估还需要多少时间可以完成。
      #4. 创建单独的、用于存放这些覆盖文件的目录。
      #5. 生成临时文件，如果写入失败打印出提示信息。
      while true
      do
          ((counter=counter+1))
          if [ #counter -ge $max ]; then
              break
          fi
          ((remainder=counter%1000))
          if [ $remainder -eq 0 ]; then
              echo -e "counter = $counter\t date = " $(date)
          fi
          mkdir -p /home/temp2
          cat < testfile > "/home/temp/myfiles.$counter"
          if [[ $? -ne 0 ]]; then
              echo "Failed to wrtie file."
              exit 1
          fi
      done
      echo "Done"
      CTRL+D
      /> ./test27.sh
      counter = 1000        Fri Dec  9 17:25:04 CST 2011
      counter = 2000        Fri Dec  9 17:25:24 CST 2011
      counter = 3000        Fri Dec  9 17:25:54 CST 2011
      ... ...
      与此同时，可以通过执行下面的命令监控磁盘空间的使用率。
      /> watch -n 2 'df -h'
      Every 2.0s: df -h                                       Fri Dec  9 17:31:56 2011
    
      Filesystem            Size   Used Avail Use% Mounted on
      /dev/sda2             3.9G  2.3G  1.4G  63% /
      tmpfs                  504M  100K  504M   1% /dev/shm
      /dev/sda1              49M   36M   11M  77% /boot
      /dev/sda3              15G  172M   14G   2% /home
      我们也可以在执行的过程中通过pidstat命令监控脚本进程的每秒读写块数。    
 
二十八、统计当前系统中不同运行状态的进程数量：

      在Linux系统中，进程的运行状态主要分为四种：运行时、睡眠、停止和僵尸。下面的脚本将统计当前系统中，各种运行状态的进程数量。
      /> cat > test28.sh
      #!/bin/sh
      #1. 初始化计数器变量，分别对应于运行时、睡眠、停止和僵尸。
      running=0
      sleeping=0
      stopped=0
      zombie=0
      #2. 在/proc目录下，包含很多以数字作为目录名的子目录，其含义为，每个数字对应于一个当前正在运行进程的pid，该子目录下包含一些文件用于描述与该pid进程相关的信息。如1表示init进程的pid。那么其子目录下的stat文件将包含和该进程运行状态相关的信息。
      #3. cat /proc/1/stat，通过该方式可以查看init进程的运行状态，同时也可以了解该文件的格式，其中第三个字段为进程的运行状态字段。
      #4. 通过let表达式累加各个计数器。
      for pid in /proc/[1-9]*
      do
          ((procs=procs+1))
          stat=`awk '{print $3}' $pid/stat`
          case $stat in
              R) ((running=runing+1));;
              S) ((sleeping=sleeping+1));;
              T) ((stopped=stopped+1));;
              Z) ((zombie=zombie+1));
          esac
      done
      echo -n "Process Count: "
      echo -e "Running = $running\tSleeping = $sleeping\tStopped = $stopped\tZombie = $zombie."
      CTRL+D
      /> ./test28.sh
      Process Count: Running = 0      Sleeping = 136  Stopped = 0     Zombie = 0.
   
二十九、浮点数验证：

     浮点数数的重要特征就是只是包含数字0到9、负号(-)和点(.)，其中负号只能出现在最前面，点(.)只能出现一次。
      /> cat > test29.sh
      #!/bin/sh
      #1. 之前的一个条目已经介绍了awk中match函数的功能，如果匹配返回匹配的位置值，否则返回0。
      #2. 对于Shell中的函数而言，返回0表示成功，其他值表示失败，该语义等同于Linux中的进程退出值。调用者可以通过内置变量$?获取返回值，或者作为条件表达式的一部分直接判断。
      validint() {
          ret=`echo $1 | awk '{start = match($1,/^-?[0-9]+$/); if (start == 0) print "1"; else print "0"}'`
          return $ret
      }
    
      validfloat() {
          fvalue="$1"
          #3. 判断当前参数中是否包含小数点儿。如果包含则需要将其拆分为整数部分和小数部分，分别进行判断。
          if [ ! -z  $(echo $fvalue | sed 's/[^.]//g') ]; then
              decimalpart=`echo $fvalue | cut -d. -f1`
              fractionalpart=`echo $fvalue | cut -d. -f2`
              #4. 如果整数部分不为空，但是不是合法的整型，则视为非法格式。
              if [ ! -z $decimalpart ]; then
                  if ! validint "$decimalpart" ; then
                      echo "decimalpart is not valid integer."
                      return 1
                  fi
              fi
              #5. 判断小数部分的第一个字符是否为-，如果是则非法。
              if [ "${fractionalpart:0:1}" = "-" ]; then
                  echo "Invalid floating-point number: '-' not allowed after decimal point." >&2
                  return 1
              fi
              #6. 如果小数部分不为空，同时也不是合法的整型，则视为非法格式。
              if [ "$fractionalpart" != "" ]; then
                  if ! validint "$fractionalpart" ; then
                      echo "fractionalpart is not valid integer."
                      return 1
                  fi
              fi
              #7. 如果整数部分仅为-，或者为空，如果此时小数部分也是空，则为非法格式。
              if [ "$decimalpart" = "-" -o -z "$decimalpart" ]; then
                  if [ -z $fractionalpart ]; then
                      echo "Invalid floating-point format." >&2
                      return 1
                  fi
              fi
          else
              #8. 如果当前参数仅为-，则视为非法格式。
              if [ "$fvalue" = "-" ]; then
                  echo "Invalid floating-point format." >&2
                  return 1
              fi
              #9. 由于参数中没有小数点，如果该值不是合法的整数，则为非法格式。
              if ! validint "$fvalue" ; then
                  echo "Invalid floating-point format." >&2
                  return 1
              fi
          fi
          return 0
      }   
      if validfloat $1 ; then
          echo "$1 is a valid floating-point value."
      fi
      exit 0
      CTRL+D
      /> ./test29.sh 47895      
      47895 is a valid floating-point value.
      /> ./test29.sh 47895.33
      47895.33 is a valid floating-point value.
      /> ./test29.sh 47895.3e
      fractionalpart is not valid integer.
      /> ./test29.sh 4789t.34
      decimalpart is not valid integer.   


三十、统计英文文章中每个单词出现的频率：
    
      这个技巧的主要目的是显示如何更好的使用awk命令的脚本。
      /> cat > test30.sh
      #!/bin/sh
      #1. 通过当前脚本的pid，生成awk脚本的临时文件名。
      #2. 捕捉信号，在脚本退出时删除该临时文件，以免造成大量的垃圾临时文件。
      awk_script_file="/tmp/scf_tmp.$$"
      trap "rm -f $awk_script_file" EXIT
      #3. while循环将以当前目录下的testfile作为输入并逐行读取，在读取到末尾时退出循环。
      #4. getline读取到每一行将作为awk的正常输入。在内层的for循环中，i要从1开始，因为$0表示整行。NF表示域字段的数量。
      #5. 使$i作为数组的键，如果$i的值匹配正则表达式"^[a-zA-Z]+$"，我们将其视为单词而不是标点。每次遇到单词时，其键值都会递增。
      #6. 最后通过awk脚本提供的特殊for循环，遍历数组的键值数据。
      cat << 'EOF' > $awk_script_file
      BEGIN {
          while (getline < "./testfile" > 0) {
              for (i = 1; i <= NF; ++i) {
                  if (match($i,"^[a-zA-Z]+$") != 0)
                      arr[$i]++
              }
          }
          for (word in arr) {
              printf "word = %s\t count = %s\n",word,arr[word]
          }
      }
      EOF
      awk -f $awk_script_file
      CTRL+D
      /> cat testfile
      hello world liu liu , ,
      stephen liu , ?
      /> ./test30.sh
      word = hello      count = 1
      word = world     count = 1
      word = stephen count = 1
      word = liu         count = 3


