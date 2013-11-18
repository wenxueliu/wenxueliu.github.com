---
layout: post
category : swift
comments : false
tagline  : swift source code analysis
tags : [source code, swift, API]
---
{% include JB/setup %}


##/swift/common/manager.py
-----------------------------------

***********************************
**SOME VARIABLE YOU MAY CONFUSED:**

     Manage     : self.servers: [set] 每个元素是 (server,run_dir)
     Server     : self.servers: name
     server_pids: [dict] server:pids(dict)
     kwargs     : [dict] it frome the swift-init, the options of argments.
**********************************

###def setup_env()
    功能    : 限制pid对资源的消耗
	参数    : 无
	实现    : 
              通过resource.setlimit限制了：
              进程堆打开最大存储
              当前进程打开的文件的最大的描述
	返回    : 无
	关系    : 模块recource

###def watch_server_pids(server_pids, interval=1, kwargs)
    功能    : 在interval间隔内监控server结束。
	参数    : 
              server_pids: [dict] server: pids[list]
              interval = 1: [int] 
              kwargs: [dic]
	实现    : 
              对server_pids中的每一个pids中的每一个pid,
                  1.等待pid结束。
                  2.得到当前运行的进程，已经停止的进程加入迭代器，
                  3.更新用status[server]中的pids更新server_pids[server]。
              if 没有运行的pid或者当前时间超过了
              指定的时间间隔。就终止。 
              else 休眠0.1秒继续。
	返回    : 返回已经停止的(server pid)
	关系    : os.waitpid()
    注      : 这个函数有跨平台的问题



###def command()
    功能    : 命令调用的装饰器
	参数    : 
              func : function
	实现    : 
              调用functools.wraps装饰器 
	返回    : 函数对象
	关系    : 模块functools.wraps

----------------------------------------------------

class Manage
===========================================
####封装Server来管理多个服务的start status stop shutdown 等


###def __init__(self,servers, run_dir=RUN_DIR)
    功能    : 初始化servers
	参数    : 
	          servers [list] : 服务名称
              run_dir [str]  : default=RUN_DIR
	实现    : 
              根据servers来初始化self.servers,支持通配符
	返回    : 无
	关系    : 无

###def status(self, kwargs)
    功能    : servers状态
	参数    : 
              kwargs [dict] : options字典
	实现    : 
              调用servers.status来决定状态
	返回    : 如果服务在全部运行，返回0，如果其中有一个服务不在运行，返回1
	关系    : server.status

###def start(self, kwargs)
    功能    : 开始self.server
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              启动self.servers中的所有服务,
                   if 已经启动就输出服务已经启动，
                   else 就启动配置目录中所有服务。
              kwargs[daemon]=False
                   调用server.interact()，父进程会等待子进程结束，除非相应键盘中断,停止当前服务。
              kwargs[wait]=True或''
                   调用server.wait()，一个服务启动后
                   又启动下一个服务
	返回    : status,
	关系    : server.launch() server.interact()
              server.wait()
    注      : daemon wait ： server.wait()
              no_daemon wait：server.interact()
              no_daemon no_wait：Server.interact()
              daemon no_wait： server.launch()

###def no_wait(self, kwargs):
    功能    : no_wait方式启动服务,几个服务同时启动。
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              设置kwargs[wait]=False
              调用self.start()
	返回    : status
	关系    : self.start()

###def no_daemon(self, kwargs):
    功能    : no_daemon方式启动self.server
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              设置kwargs[daemon]=False
              调用self.start()
	返回    : status
	关系    : self.start()


###def once(self, kwargs):
    功能    : once方式启动self.server,启动一次
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              设置kwargs[once]=Ture
              调用self.start()
	返回    : status
	关系    : self.start()

###def stop(self, kwargs)
    功能    : 给self.server发送SIGTERM信号
	参数    : 
	          conf_file [str] : 配置文件（全路径）
	实现    : 
              1.对指定的服务调用stop(),得到stop的signal_pids.
                  if signal_pids为空, 输出no server running
                  else  server_pids=signal_pids,
                  server_pids为正常停止的服务dict。
              2.得到全部正常stop的pid signal_pids，
              3.监控在kwargs[kill_wait]时间内结束的server,pid,并加到killed_pids中。
                如果killed_pids和signal_pids相同，则返回0.进程结束。
              4.遍历server_pids,如果某个服务的pid不在killed_pids中，就输出提示信息。返回1.

	返回    : 如果在kwargs[kill_wait]时间，所有服务已经停止，则返回0，否则返回1.
	关系    : server.stop() dict.symmetric_difference() dict.issuperset()

###def shutdown(self, kwargs)
    功能    : 给self.server发送SIGHUP信号
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              设置kwargs[graceful]=Ture
              调用self.stop()
	返回    : 所有服务都接受SIGHUP, 如果kwargs[kill_wait]时间内，已经停止，则返回0，否则返回1.
	关系    : self.stop()

###def restart(self, kwargs):
    功能    : 停止self.server，又开启self.server.
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              停止self.server
              开启self.server
	返回    : 如果在规定时间内stop所有服务，又能开启所有服务，返回0，否则,返回1
	关系    : self.stop() self.start()

###def reload(self, kwargs):
    功能    : 对self.server中每个服务，先stop,后start.
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              kwargs[graceful]=True
              对self.server中的每个服务server
                  停止server
                  开启server
	返回    : 如果在规定时间内stop所有服务，又能开启所有服务，返回0，否则,返回1
	关系    : self.stop() self.start()

###def force_reload(self, kwargs):
    功能    : 对self.server中每个服务，先stop,后start.
	参数    : 
              kwargs [dict] : options字典。
	实现    : 
              与reload()相同
	返回    : 如果在规定时间内stop所有服务，又能开启所有服务，返回0，否则,返回1
	关系    : self.reload()

###def get_command(self, cmd):
    功能    : 检验cmd是否是合法的command
	参数    : 
              cmd [str] 
	实现    : 
              cmd是否是Manager的属性
              cmd是否有publicly_accessible=True
	返回    : 返回cmd,如果具有publicly_accessible属性的命令。
	关系    : 无。

###def list_commands(cls):
    功能    : 列出所有可运行的命令
	参数    : 
              cls [class] Manger
	实现    : 
              列出Manager中有publicly_accessible的所有命令。
	返回    : 返回所有属性,如果cmd具有publicly_accessible属性的命令。
	关系    : self.get_method()


###def run_command(self, cmd, kwargs):
    功能    : 列出所有可运行的命令
	参数    : 
              cmd [str] : 命令
              kwargs [dict] : options字典。
	实现    : 
              如果是命令，则调用相应的命令函数。
	返回    : 返回类属性
	关系    : self.get_method()


--------------------------------------------------------------------------


class Server
===============================================
####单个服务的start status launch wait等

###def __init__(self, server, run_dir=RUN_DIR)
    功能    : 初始化服务
	参数    : 
	          servers [list] : 服务名称
              run_dir [str]  : default=RUN_DIR
	实现    : 
              初始化各变量。
              self.type : 如init proxy object等
              self.procs: 子进程
	返回    : 返回pid文件列表
	关系    : 无
    注      : 为什么引入self.type?

###def __str__(self)
    功能    : s=Server(), 修改print s 和 
                          Server.str()
	参数    : 无
	实现    : 略
	返回    : self.server 
	关系    : 无

###def __repr__(self)
    功能    : s=Server(), 修改除str之外的输出属性
    参数    : 无
	实现    : 略
	返回    : 略
	关系    : 无

###def __hash__(self)
    功能    : 暂定
    参数    : 无
	实现    : 略
	返回    : 略
	关系    : 无

###def __eq__(self)
    功能    : 修改服务“==”属性
    参数    : 无
	实现    : 略
	返回    : 略
	关系    : 无

###def get_pid_file_name(self, conf_file):
    功能    : 将相应的配置文件转为pid文件
	参数    : 
	          conf_file [str] : 配置文件（全路径）
	实现    : 
	          将文件的配置目录替换为运行目录
			  将文件的扩展由.conf替换为.pid
	返回    : pid文件列表
	关系    : 无


###def get_conf_file_name(self, pid_file):
    功能    : 将相应的pid文件转为配置文件
	参数    : 
	          pid_file [str] : pid文件（全路径）
	实现    :
	          将文件的目录替换为配置目录
			  将文件的扩展由.pid替换为.conf
	返回    : 配置文件列表
	关系    : 无

###def conf_files(self, kwargs):
	功能    : 找到对应编号的配置文件，或全部配置文件
    参数: 
	          kwargs  [dict] : number quiet verbose
	实现    : 
	          从配置目录中找到swift相关的所有配置文件（.conf）。  
              1.如果kwargs的key中有number，则只返回第number个配置文件
	            否则 返回全部的配置文件
	          2.如果没有找到配置文件，根据kwargs的key中的quiet和verbose
	            决定是否输出具体信息
	返回    : 配置文件 如果不存在，就返回None
	关系    : search_tree() 


###def pid_files(self, kwargs):
    功能    : 得到pid文件
	参数    :
	          kwargs [dict] : number
	实现    :
              搜寻run_dir,找到所有当前服务的pid文件。
	          如果kwargs的key中number不为0，并且如果pid_file有
                  对应的conf_file在conf_files中，则返回对应conf文件的pid文件
			  否则，kwargs[number]=0,返回全部的pid文件
	返回    : pid文件列表，可能是0,1或者多个
	关系    : search_tree() get_config_file_name(self)


###def iter_pid_files(self, kwargs):
    功能    : 得到pid文件名和pid
	参数    :
	          kwargs  [dict] : number
	实现    :
	          返回pid文件的迭代器
	返回    : pid文件名和pid
	关系    : pid_file(self)

###def signal_pids(self, sig, kwargs):
    功能    : 给pid发送sig信号
	参数    :
	          sig    [signal]  : 信号
			  kwargs [dict]    : pid_file pid
	实现    : 
	          对指定的pid发送sig信号，os.kill(pid,sig)
	          异常：对不存在的pid和权限控制
	返回    : pids字典，key为pid，value为文件名,发送sig的成功的进程
	关系    : remove_file() os.kill() signal

###def get_running_pids(self, kwargs):
    功能    : 得到运行的pid,实际是kwargs中server对应pid发信号。
	参数    : 
              kwargs [dict] : verbose
	实现    :
	          简单调用signal_pids(signal.SIG_DFL,kwargs)
              signal_pids()例子
	返回    : 返回pids字典，key为pid，value为文件名,发送sig的成功的进程
	关系    : signal_pids(self)

###def kill_running_pids(self, kwargs):
    功能    : 杀死正在运行的pid.
	参数    : 
	          kwargs  [dict] : graceful
	实现    : 
              如果是kwargs[graceful]=True 并且在
              GRACEFUL_SHUTDOWN_SERVERS中就发送
              SIGHUP信号
              否则，发送SIGTERM信号
	返回    : pids字典，key为pid，value为文件名,正常kill进程
	关系    : self.signal_pids()

###def status(self, pids=None, kwargs):
    功能    : 显示服务的状态
	参数    : 
              pid  [int] : pid 号
	          kwargs  [dict] : graceful
	实现    : 
              if pids==None: 发送 signal.SIG_DFL信号
              if pids == False: 返回1
              else : 输出正在运行的pid号及配置文件
	返回    : 如果在运行，返回0, 如果不在运行返回1.
	关系    : self.get_running_pids(), self.conf_files 


###def spawn(self, conf_file, once=False, wait=True,daemon=True, kwargs)
    功能    : 启动子进程
	参数    : 
              conf_file [str] : 配置文件名
	          kwargs  [dict] : graceful
              once [boolen] : if True once增加到args.
              wait [boolen] : if True args增加verbose,修改标准输出的出口。
              daemon [boolen] : 修改标准错误和标准输出的出口
    实现：
              if daemon False:
                  args增加versbose属性, 标准输出和标准错误重定向到None。
              if daemon True: 
                  标准错误重定向到子进程标准输出 if wait = True : 标准输出重定向到子进程管道.
                  else 标准输出重定向到os.devnull

	返回    : 子进程的pid
	关系    : subprocess.Popen() write_file()


###def wait(self, kwargs):
    功能    : 等待子进程开始
	参数    : 
	          kwargs  [dict] : graceful
	实现    : 
              对每一个子进程，
              if 内容不为空，就输出内容，
                 if 在 WARNING_WAIT时间内子进程退出，就将退出状态加到status.
                 else，继续执行。
              else，继续下一个子进程。
	返回    : status,没有子进程退出,返回0,
                       有子进程终止,返回终止的个数
	关系    : Popen.stdout.read() Popen.poll()

###def interact(self, kwargs)
    功能    : 父进程会等待子进程终止
	参数    : 
	          kwargs  [dict] : graceful
	实现    : 
              对每一个子进程，
                 调用子进程的communicate()。
	返回    : status，子进程正常结束,返回0,
                              被终止,返回1
	关系    : Popen.communicate()
    注      : Popen.communicate()会等待子进程结束,才继续执行。

###def launch(self, kwargs)
    功能    : 有服务启动就输出启动的服务
              没有服务启动就用子进程启动服务。
	参数    : 
	          kwargs  [dict] : once number
	实现    : 
              从配置目录得到conf_files,如果conf_files为空就直接返回
              从运行目录得到正在运行的pids[list] 

              对于pids中的每一个pid
                  if pid对应的conf_file在conf_files中
                     输出已经启动的server pid conf_file
                  elif kwargs[number]=False
                     输出已经启动的server pid pid_file

              if already_started=True: 
                  输出服务已经启动，返回[]
              如果对于每一个pid都没有对应的conf_file在conf_files中，
              ，且指定了kwargs[number]，就从conf_files
              中的每一个conf_file开始:
                  启动子进程调用self.cmd conf_file

	返回    : if server启动，就返回[],
              else pids字典, pid:pid_file
	关系    : self.spawn()
    注      : 1. 配置目录中没有对应的conf_files。
              2. 配置文件中有对应的配置文件:
                   正在运行的pids对应的conf_file在conf_files中
                   正在运行的pids对应的conf_file不在conf_files中,kwargs[number]!=0
                   正在运行的pids对应的conf_file不在conf_files中,kwargs[number]=0

###def stop(self, kwargs)
    功能    : 终止当前进程
	参数    : 
	          kwargs  [dict] : 
	实现    : 
              调用kill_running_pids()
	返回    : pids字典，pid:pid_file,正常kill进程
	关系    : self.kill_running_pids()

