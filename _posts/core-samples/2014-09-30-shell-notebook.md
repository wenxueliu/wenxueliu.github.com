---
layout: post
category : shell
tagline: "shell 速查笔记"
tags : [shell, linux, script]
---
{% include JB/setup %}

####参数
		$$:    表示当前shell的pid.
	 	$#:    表示参数的个数.
	 	$!:    最后一个放入后台作业的PID值.
	 	$*,$@: 表示所有的参数. 
	 	区别：
	 	$ set 'apple pie' pears peaches
	 	$ for i in $@ ;do  echo $i; done
	 	$ for i in $* ;do  echo $i; done
	 	$ for i in "$@" ;do  echo $i; done
	 	$ for i in "$*" ;do  echo $i; done

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
       
       数组的声明可以不是连续的, 这一点和awk中的数组比较类似.
      
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
 
       	function greeting
       	{
           	echo "Hi $1 and $2";
       	}
       
       	$ greeting tom joe
       		Hi tom and joe
       
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
      
       	if (NULL == variable)
       	{
           	variable=word
           	echo word
       	}
       	else
       	{
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
      
       $ variable="/home/lilliput/jake/.bashrc
       $ echo ${variable#/home}
       		/lilliput/jake/.bashrc
   
####${variable##pattern} 从variable头部开始,最大化的删除pattern
       
       $ variable="/home/lilliput/jake/.bashrc
       $ echo ${variable##*/}
       		.bashrc
   
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
   
#### 条件判断方法: test, [ ], [[ ]], (( )), 当$?为0时表示成功和true, 否则失败和false.
        $ name=tom
        $ test $name != tom
        $ echo $?
        	1    //Failure
       
        $ [ $name=Tom ]
        $ echo $?
        	0
       
        $ [ $name = [Tt]?? ]  //[]和test不允许使用通配符.
        $ echo $?
        	1
       
        $ x=5
        $ y=20
        $ [ $x -gt $y ]
        $ echo $?
        	1
       
        $ [ $x -le $y ]
        $ echo $?
        	0
       
        $ name=Tom
        $ friend=Joseph
        $ [[ $name == [Tt]om ]]
        $ echo $?
        	0
       
        $ [[ $name == [Tt]om && $friend == "Jose" ]]
        $ echo $?
        	1
       
        $ x=2
        $ y=3
        $ (( x > 2))
        $ echo $?
        	1
       
        $ (( x < 2))
        $ echo $?
        	1
       
        $ (( x == 2 && y == 3))
        $ echo $?
        	0
       
####test命令操作符:
    
  ##字符串判断:
       [ string1 = string2 ]    or    两个字符串相等时返回true.
       [ string1 == string2 ]
               
       [ string1 != string2 ]         两个字符串不等时返回true.
       [ string ]                          string非空时返回true.
       [ -z string ]                      为空时返回true.
       [ -n string ]                      为非空时返回true.
       
####逻辑判断:(cond1可以包含元字符)
       
       [ string1 -a string2 ] or     string1和string2都非空时
       [[ cond1 && cond2 ]]        cond1和cond2都为true
           
       [ string1 -o string2 ] or     string1或string2为非空时
       [[ cond1 || cond2 ]]          cond1或cond2为true.
           
       [ ! string ] or                    string为空
       [[ !cond ]]                        cond为false.
                   
####整数判断:
    
       [ int1 -eq int2 ]                int1等于int2
       [ int1 -ne int2 ]                int1不等于int2
       [ int1 -gt int2 ]                 int1大于int2
       [ int1 -ge int2 ]                int1大于等于int2
       [ int1 -lt int2 ]                  int1小于int2
       [ int1 -le int2 ]                 int1小于等于int2
       
####文件判断
    
       [ file1 -nt file2 ]               file1比file2新
       [ file1 -ot file2 ]               file1比file2旧
   
####文件检验

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

###if语句:
   
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
   
### 空语句:
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
		echo second     #该Shell命令将不再被打印输出
	$ chmod +x trace_patial_command.sh
	$ ./trace_patial_command.sh
		+ echo first
		first
		+ set +x
		second

#### trap 捕捉信号


#### date

	将基础时间转为时间戳

	time1=$(date +%s -d '1990-01-01 01:01:01')

	将时间戳转为基础时间

	time1=$(date +%Y-%m-%d\ %H:%M:%S -d "1970-01-01 UTC $time1 seconds");

