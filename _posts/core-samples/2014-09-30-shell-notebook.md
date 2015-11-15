---
layout: post
category : shell
tagline: "shell 速查笔记"
tags : [shell, linux, script]
---
{% include JB/setup %}

####参数

    $$:      : 表示当前shell的pid.
    $#:      : 表示参数的个数.
    $!:      : 最后一个放入后台作业的PID值.
    $*,$@    : 表示所有的参数.
    ${#parm} : 表示参数 param 长度

 	区别：
 	$ set 'apple pie' pears peaches
 	$ for i in $@ ;do  echo $i; done
 	$ for i in $* ;do  echo $i; done
 	$ for i in "$@" ;do  echo $i; done
 	$ for i in "$*" ;do  echo $i; done
    $ for i in "${@:2}"; do echo $i; done

###初始化顺序

	 /etc/profile    ( ~/.bash_profile | ~/.bash_login | ~/.profile )    ~/.bashrc

###set 命令

 	set -o allexport 当前shell变量对其所有子shell都有效.
    set +o allexport 当前shell变量对其所有子shell都无效.
    set -o noclobber 重定向输出时,如果输出文件已经存在则提示输出失败, date > out; date > out, 第二次操作失败
    set +o noclobber 缺省shell行为. date > out; date > out, 第二次操作成功
    shopt -s extglob 使用扩展通配符,如 abc?(2|9)K, abc*([0-9]), abc+([0-9]), no@(thing|body), no!(thing|body)
    其中?,*,+,@和!都是用于修饰后面的()的.

###declare

####变量声明: declare, 在赋值的时候等号的两边不需要空格. variable=value.

    declare -r variable=value 声明只读变量.
    declare -x variable=value 相当于export variable=value.

####数组声明:

    declare -a variable=(1 2 3 4)
    or
    name=(tom tim helen)
    or
    x[0]=5
    x[4]=10

    数组的声明可以不是连续的, 这一点和 awk 中的数组比较类似.

 Example 1

    $ declare -a friends
    $ friends=(sheryl peter louise)
    $ echo ${friends[0]}
    	sheryl
    $ echo ${friends[1]}
    	peter
    $ echo ${friends[2]}
    	louise
    $ echo ${friends[*]}
    	shery1 peter louise
    $ echo ${#friends[*]}
    	3
    $ unset friends

Example 2

    $ declare -a states=(ME [3]=CA [2]=CT)
    $ echo ${states[*]}
     	ME CA CT
    $ echo ${#states[*]}
    	3
    $ echo ${states[0]}
    	ME
    $ echo ${states[1]}

    $ echo ${states[2]}
    	CT
    $ echo ${states[3]}
    	CA
    $unset states

###函数声明及调用

    $ foo() { echo "Hi $1 and $2"; return 1; }
    $ function foo { echo "Hi $1 and $2"; return 1; }

    $ greeting tom joe
    	Hi tom and joe

    $ function greeting { echo "Hi joe" ; echo $1; return $1; }

    $ if greeting 0; then echo "return 1"; else echo "return 0"; fi
    $ if greeting 1; then echo "return 1"; else echo "return 0"; fi
    $ if greeting 2; then echo "return 1"; else echo "return 0"; fi

    $unset -f greeting


###格式化输出 printf (类似于 awk 的printf)

###变量扩展修改符

####${variable:+word}

    if (NULL != variable)
      	echo word
    else
      	echo $variable

####${variable:-word}

    if (NULL == variable)
      	echo word
    else
      	echo $variable

####${variable:=word}

    if (NULL == variable) {
       	variable=word
       	echo word
    } else {
       	echo $variable
    }

#### ${variable:offset}/${variable:offset:length}

	$ var_name=notebook
	$ echo ${var_name:0:4}
	  	note
	$ echo ${var_name:4:4}
		book
	$ echo ${var_name:2}
	  	tebook
	$ echo ${var_name:0}
	   	notebook

#### ${variable%pattern} 从variable尾部开始,最小化的删除pattern

    $ variable="/usr/bin/local/bin"
    $ echo ${variable%/bin*}
    	/usr/bin/local

#### ${variable%%pattern} 从variable尾部开始,最大化的删除pattern

    $ variable="/usr/bin/local/bin"
    $ echo ${variable%%/bin*}
    	/usr

#### ${variable#pattern} 从variable头部开始,最小化的删除pattern

    $ variable="/usr/bin/local/bin"
    $ echo ${variable#/usr}
    		bin/local/bin

####${variable##pattern} 从variable头部开始,最大化的删除pattern

    $ variable="/usr/bin/local/bin"
    $ echo ${variable##*/}
    		bin

####${variable/pattern/string} 替换 variable 扩展后的值中最长的匹配模式(若存在匹配模式的话).
为了在 pattern 扩展后的值开头匹配模式, 可以给 PATTERN 附上前缀#, 如果要在值末尾匹配模式, 则
附上前缀 %. 如果 STRING 为空, 则末尾的 / 可能被忽略, 匹配将被删除。使用 '@' 或 '$' 即可对列
表中的每个参数进行模式替换.

    $ variable="/usr/bin/local/bin"
    $ echo ${variable/bin/sbin}
    		/usr/sbin/local/bin

####${variable//pattern/string} 对所有的匹配（而不只是第一个匹配）执行替换

    $ variable="/usr/bin/local/bin"
    $ echo ${variable//bin/sbin}
    		/usr/sbin/local/sbin

##### ${#pattern} 返回patter的字符数量.

    $ echo variable=abc123


###引用

    \: 可以使shell中的元字符无效, 如?, < >, \, $, *, [ ], |, ( ), ;, &, { }
    ': 单引号可以史其内的所有元字符无效, 也包括\.
    ": 双引号也可以史其内的所有元字符无效, 变量和命令替换除外. 如 echo "What's time? $(date)", 这里date命令将被执行.

### 命令替换:

    variable=$(date) OR variable=`date`
    variable=`basename \`pwd\`` or variable=$(basename $(pwd))    命令替换可以嵌套, 第一种方法中,嵌套的命令必须使用\进行转义.
    eval: 可以进行命令行求值操作.

Example 1

    $ set a b c d
    $ ech echo The last argument is \$$#
    	The last argument is $4
    $ eval echo The last argument is \$$#
    	The last argument is d


####数学计算:

    $ echo $[5+4-2]
    	7
    $ echo $[5+2*3]
    	11
    $ echo $((5+4-2))
    	7
    $ echo $((5+2*3))
    	11

    $ declare -i num //必须声明-i, 以表示整型变量.
    $ num=5+5
    $ echo $num
    	10                  //如果没有声明declare -i, 则返回5+5.
    $ num=5 + 5
    	-bash: + 5: command not found.
    $ num="5 + 5"
    $ echo $num
    	10
    $ num=4*6
    $ echo $num
    	24

    let专门用于数学运算的bash内置命令.
    $ let i=5
    $ let i=i+1
    $ echo $i
    	6
    $ let "i = i + 2"
    $ echo $i
    	8
    $ let "i+=1"
    $ echo $i
    	9

	expr用于数学也可以运于数学运算



####使用不同进制(2~36)表示数字.

    $ declare -i x=017
    $ echo $x
    		15
    $ x=2#101
    $ echo $x
    		5
    $ x=8#17
    $ echo $x
    		15
    $ x=16#b
    $ echo $x
    		11

#读取用户输入(read)命令

    $ read answer
    	yes
    $ echo "$answer is the right response."
    	yes is the right reponse.

    $ read first middle last
    	Jon Jake Jones
    $ echo "Hello $first"
    	Hello Jon
   	$ read            //如果没有变量时, $REPLY是缺省变量.
    	the Chico Nut factory
    $ echo "I guess $REPLY keeps you busy!"
    	I guess the Chico Nut factory keeps you busy!
   	$ read -p "Enter your job titile: "
    	Enter your job title: Accountant
    $ echo "I thought you might be an $REPLY".
    	I thought you might be an accountant.
   	$ read -a friends
    	Melvin Tim Ernest
    $ echo "Say hi to ${friends[2]}
    	Say hi to Ernest.
    $ echo "Say hi to ${friends[$[${#friends[*]}-1]]}"
    	Say hi to Ernest.


###条件结构和流控制:

条件判断方法: test, [ ], [[ ]], (( )), 当 $? 为 0 时表示成功和 true, 否则失败和 false.

1. 内置命令 test 根据表达式 expr 求值的结果返回 0(真)或 1(假). 也可以使用方括号: test expr 和 [ expr ] 是等价的

    $ test 2 = 2 && echo "correct" || echo "incorrect"
     correct

    $ test 2 = 3 && echo "correct" || echo "incorrect"
     incorrect

    $ [ 2 = 3 ] && echo "correct" || echo "incorrect"
     incorrect

    $ [ 2 = 2 ] && echo "correct" || echo "incorrect"
     correct

    $ name=Tom
    $ [ $name = Tom ] && echo "correct" || echo "incorrect"
     correct

    $ [ $name = Tom ] && echo "correct" || echo "incorrect"
     correct

2. "[]" 和 test 不允许使用通配符. "[[]]" 支持通配符

    $ [ $name = [Tt]?? ] && echo "correct" || echo "incorrect"
     incorrect

    $ test $name = [Tt]?? && echo "correct" || echo "incorrect"
     incorrect

    $ [[ $name = [Tt]?? ]] && echo "correct" || echo "incorrect"

3. -eq -ne-lt -le -gt 或 -ge 比较算术值, 用操作符 = !=  < 和 > 比较字符串是否相等,
不相等或者第一个字符串的排序在第二个字符串的前面或后面, 在 [] 中 > < 必须用 "\" 转义

    $ x=1
    $ y=2
    $ [ $x -gt $y ] && echo "correct" || echo "incorrect"
     incorrect

    $ [ $x -le $y ] && echo "correct" || echo "incorrect"
     correct

    $ [ "abc" > "bcd" ] && echo "correct" || echo "incorrect"
     correct

    $ [ "abc" \> "bcd" ] && echo "correct" || echo "incorrect"
     incorrect

    [ string1 == string2 ] 或 [ string1 = string2 ]
    [ string1 != string2 ]
    [ string ]       : string 非空时返回true.
    [ -z string ]    : 为空时返回true.
    [ -n string ]    : 为非空时返回true.

    $ [ "abc" = "abc" ] && echo "correct" || echo "incorrect"
    $ [ "abc" == "abc" ] && echo "correct" || echo "incorrect"
    $ [[ "abc" = "abc" ]] && echo "correct" || echo "incorrect"
    $ [[ "abc" == "abc" ]] && echo "correct" || echo "incorrect"

4. (()) 复合命令 计算算术表达式, [[]] 对文件名和字符串, 支持 () 进行算术优先级调整

####逻辑判断:(cond1可以包含元字符)

    [ string1 -a string2 ] : string1 和 string2 都非空时
    [[ cond1 && cond2 ]]   : cond1 和 cond2 都为 true

    [ string1 -o string2 ] : string1 或 string2 为非空时
    [[ cond1 || cond2 ]]   : cond1 或 cond2 为 true.

    [ ! string ]           : string 为空
    [[ !cond ]]            : cond 为 false.

####整数判断:

    [ int1 -eq int2 ]      : int1 等于 int2
    [ int1 -ne int2 ]      : int1 不等于 int2
    [ int1 -gt int2 ]      : int1 大于 int2
    [ int1 -ge int2 ]      : int1 大于等于 int2
    [ int1 -lt int2 ]      : int1 小于 int2
    [ int1 -le int2 ]      : int1 小于等于 int2

####文件判断

    [ file1 -nt file2 ]               file1比file2新
    [ file1 -ot file2 ]               file1比file2旧

####文件检验

    [ -d $file ]           : 表示判断目录是否存在
    [[ -d $file ]]

    [ -e $file ] or        : 表示判断文件是否存在
    [[ -e $file ]]

    [ -f $file ] or        : 表示判断非目录普通文件是否存在
    [[ -f $file ]]

    [ -s $file ] or        : 表示判断文件存在, 并且非0.
    [[ -s $file ]]

    [ -L $file ] or        : 表示判断为链接符号存在
    [[ -L $file ]]

    [ -r $file ] or        : 表示判断文件存在, 并且可读.
    [[ -r $file ]]

    [ -w $file ] or        : 表示判断文件存在, 并且可写.
    [[ -w $file ]]

    [ -x $file ] or        : 表示判断文件存在, 并且可执行.
    [[ -x $file ]]

    [[ file1 -nt file2 ]]  : 测试 file1 是否比 file2 更新, 修改日期将用于这次和下次比较。
    [[ file1 -ot file2 ]]  : 测试 file1 是否比 file2 旧.
    [[ file1 -nt file2 ]]  : 测试 file1 是不是 file2 的硬链接.

    更多见 man test, help test

####if 语句:

    if command     //当 command 的返回状态为 0 时执行 then 后面的语句.
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

    以下两种为new format, 建议使用.

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

可以用括号把表达式分组, 覆盖默认的优先级. 请记住 shell 通常要在子 shell 中运行括号中的表达式,
所以需要用 \( 和 \) 转义括号, 或者把这些操作符括在单引号或双引号内

    test "home" != "$HOME" -a 3 -ge 4 && echo "correct" || echo "incorrect"
    [ ! \( "a" = "$HOME" -o 3 -lt 4 \) ] && echo "correct" || echo "incorrect"

####空语句

    用冒号表示.
    if expr
    then
        :
    fi

####case语句    //;;相当于c语言中break.

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

####循环语句:

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


    [scriptname: doit]
    while (( $# > 0 ))
    do
        echo $*
        shift
    done

    doit a b c d e
    a b c d e
    b c d e
    c d e
    d e
    e

###简单的命令跟踪

#### shell 命令加选项

	$ cat > trace_all_command.sh
		who | wc -l 
	$ chmod +x trace_all_command.sh
	$ sh -x ./trace_all_command.s
		+ wc -l    #被跟踪的两条Shell命令
    	+ who
    	2          #实际输出结果。

####set 设置

	$ cat > trace_patial_command.sh
		#! /bin/bash
		set -x           #从该命令之后打开跟踪功能
		echo first       #将被打印输出的Shell命令
		set +x           #该Shell命令也将被打印输出，然而在该命令被执行之后，所有的命令将不再打印输出
		echo second      #该Shell命令将不再被打印输出
	$ chmod +x trace_patial_command.sh
	$ ./trace_patial_command.sh
		+ echo first
		first
		+ set +x
		second

#### trap 捕捉信号

    trap [-lp] [[arg] sigspec ...]

    sigspec 包括 <signal.h> 中定义的各个 signal， EXIT，ERR，RETURN 和 DEBUG。

各个 signal 这里就不介绍了。EXIT 会在 shell 退出时执行指定的命令。若当前
shell 中有命令执行返回非零值，则会执行与 ERR 相关联的命令。而 RETURN 是针对
source 和 . ，每次执行都会触发 RETURN 陷阱。若绑定一个命令到
DEBUG，则会在每一个命令执行之前，都会先执行 DEBUG 这个
trap。这里要注意的是，ERR 和 DEBUG 只在当前 shell 有效。若想函数和子 shell
自动继承这些 trap，则可以设置 -T(DEBUG/RETURN) 和 -E(ERR)。

比如，下面的脚本会在退出时，执行echo：

    #!/bin/bash
    trap "echo this is a exit echo" EXIT
    echo "this is a normal echo"

或者,让脚本中命令出错时,把相应的命令打印出来：

    #!/bin/bash 
    trap 'echo $BASH_COMMAND return err' ERR echo this is a normal test
    UnknownCmd

这个脚本的输出如下:

    this is a normal test
    tt.sh: line 6: UnknownCmd: command not found
    UnknownCmd return err

亦或者, 让脚本的命令单步执行:

    #!/bin/bash
    trap '(read -p "[$0 : $LINENO] $BASH_COMMAND ?")' DEBUG
    echo this is a test
    i=0
    while [ true ]
    do
        echo $i
            ((i++))
    done

其输出如下:

    [tt.sh : 5] echo this is a test ?
    this is a test
    [tt.sh : 7] i=0 ?
    [tt.sh : 8] [ true ] ?
    [tt.sh : 10] echo $i ?
    0
    [tt.sh : 11] ((i++)) ?
    [tt.sh : 8] [ true ] ?
    [tt.sh : 10] echo $i ?
    1
    [tt.sh : 11] ((i++)) ?
    [tt.sh : 8] [ true ] ?
    [tt.sh : 10] echo $i ?
    2
    [tt.sh : 11] ((i++)) ?


#### date

	将基础时间转为时间戳

	time1=$(date +%s -d '1990-01-01 01:01:01')

	将时间戳转为基础时间

	time1=$(date +%Y-%m-%d\ %H:%M:%S -d "1970-01-01 UTC $time1 seconds");

#### log

    _loglevel=2

    DIE() {
        echo "Critical: $1" >&2
        exit 1
    }

    INFO() {
        [ $_loglevel -ge 2 ] && echo "INFO: $1" >&2
    }

    ERROR() {
        [ $_loglevel -ge 1 ] && echo "ERROR: $1" >&2
    }

这里的实现只是简单的加了一个 loglevel, 其实可以把 log 输出到一个文件中, 或者给
log 加上颜色. 比如:

    # add color
    [ $_loglevel -ge 1 ] && echo -e "\033[31m ERROR:\033[0m $1" >&2
    # redirect to file
    [ $_loglevel -ge 1 ] && echo "ERROR: $1" > /var/log/xxx_log.$BASHPID]]"

```
    #!/bin/bash

    process()
    {
        ls abc
        date
        sleep 2
    }

    LOGFILE="/tmp/log.txt"
    touch $LOGFILE
    tail -f $LOGFILE &
    pid=$!
    exec 3>&1
    exec 4>&2
    exec &>$LOGFILE
    process
    ret=$?
    exec 1>&3 3>&-
    exec 2>&4 4>&-
    kill -15 $pid
    exit $ret
```

###数组

    用 () 定义数组
    a=(1 2 3 4 5)
    echo $a


    用${#数组名[@或*]} 可以得到数组长度
    echo ${#a[@]}

    用${数组名[下标]} 下标是从0开始 下标是：*或者@ 得到整个数组内容
    echo ${a[2]}
    echo ${a[*]}

    直接通过 数组名[下标] 就可以对其进行引用赋值，如果下标不存在，自动添加新一个数组元素
    a[1]=100
    echo ${a[*]}

    a[5]=100
    echo ${a[*]}

    直接通过：unset 数组[下标] 可以清除相应的元素，不带下标，清除整个数据。
    a=(1 2 3 4 5)
    unset a
    echo ${a[*]}
    a=(1 2 3 4 5)
    unset a[1]
    echo ${a[*]}
    echo ${#a[*]}

    直接通过 ${数组名[@或*]:起始位置:长度} 切片原先数组，返回是字符串，中间用“空格”分开，因此如果加上”()”，
    将得到切片数组，
    a=(1 2 3 4 5)
    echo ${a[@]:0:3}
    echo ${a[@]:1:4}
    c=(${a[@]:1:4})
    echo ${#c[@]}
    echo ${c[*]}

    调用方法是：${数组名[@或*]/查找字符/替换字符} 该操作不会改变原先数组内容，如果需要修改，可以看上面例子，重新定义数据。
    a=(1 2 3 4 5)
    echo ${a[@]/3/100}
    echo ${a[@]}
    a=(${a[@]/3/100})
    echo ${a[@]}

    数组元素包含空格
    a=("1 2" "2 3" 4 5 6)
    echo "array list" ${a[*]}
    echo "array size" ${#a[*]}
    echo "array size" "${#a[*]}"
    echo "array size" ${!a[@]}"

    参考 http://www.cnblogs.com/chengmo/archive/2010/09/30/1839632.html

###case

    在shell的case语句中，可以使用匹配模式:
        * 匹配所有的字符
        ? 匹配单个字符
        [...]匹配[]括起的字符

    echo "please input number 1 to 3"
    read number
    case $number in
    1|2|3)
        echo "you input 1"
        ;;
    4|5)
        echo "you input 2"
        ;;
    6)
        echo "you input 3"
        ;;
    *)
        echo "error! the number you input isn't 1 to 3"
        ;;
    esac

###命令变量

    DATE=$(date +%F)
    DATE=`date +%F`

###参考
http://www.tinylab.org/bash-debugging-tools/
