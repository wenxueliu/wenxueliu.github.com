---
layout: post
category : python 
tagline: "Python paste deploy translate"
tags : [python, translate]
---
{% include JB/setup %}

翻译者：刘文学

文档：http://pythonpaste.org/deploy/


**名词的翻译**

    Paste Deployment: PD
    Paste Deploy : PD
    composite application: 组件应用
    composite section: 组件区域
    applications : app
    factories : 工厂函数
    entry point :入口
    key ： 与字典中key类似，配置文件中=左为key，右为value

###介绍

PD 是一个寻找和配置 WSGI app 和 server 的系统。对于 WSGI app 的 consumer，它提供了一个单一的、简单的函数（loadapp）从配置文件和 Python Egg 中加载 WSGI app。对于 WSGI app 的 providers ，它只请求单一的、简单的app 入口，所以不需要暴露你 app 的细节给 app user 。

结果是系统管理员在不了解 python 或 WSGI app 细节的情况下可以安装和管理 app 。
	
PD 目前不需要 Paste 的其他部分支持，作为一个独立的包。

为了看 PD 的更新，参考 [news file](http://pythonpaste.org/deploy/news.html)。

PD 遵循 MIT license。
	
###安装###

首先确认你已经安装 [setuptools](http://peak.telecommunity.com/DevCenter/setuptools) 或 它的的替代者 [distribute](https://pypi.python.org/pypi/distribute)。对于 Python 3.x 需要 distribute， setuptools 不能正常工作。

然后你可以用 pip 来安装 Paste Deployment：

	$ sudo pip install PasteDeploy

如果你想跟踪开发版本，如下

    $ hg clonehttp://bitbucket.org/ianb/pastedeploy
$ cd pastedeploy 
    $ sudo python setup.py develop

这里将安装全部包，但是在 checkbox 中加载文件。你也可以安装PasteDeploy==dev。

对于下载和其他信息，见[the Cheese Shop PasteDeploy page](https://pypi.python.org/pypi/PasteDeploy)。

一个值得赞赏的包是[Paste Script](http://pythonpaste.org/script/)。为了安装它，用pip install PasteScript (或 pip install PasteScript==dev)

###从用户的角度 

下面的部分，解释了 PD 的 Python API。这并不是给用户看的（对 python 开发者有用，对建立 tests fixtures 有用）

与 PD 的主要交互方式是配置文件，与配置文件交互，你要做的是提供服务给它。为了学习关于服务的配置文件，参考[paster serve command](http://pythonpaste.org/script/#paster-serve)。

###配置文件###

一个配置文件有不同的 sections，PD 只关心带有前缀的 sections，比如[app:main]或者[filter:errors]，在":"前面是 section 的名字，后面是 section 的类型。
总的来说，一个section的标识就是[type:name]，不是这种类型的section将会被忽略。

配置文件的格式是简单的 INI 格式：name = value。你可以通过缩进后续行来扩展这些值。"#"是注释。 

一个典型的配置文件有一个或两个 sections 的名字为 main：一个为 application sections（[app:main]）， 一个为server sections（[server:main]）。[composite:...]是一个应用的分发器，用来分发各个应用。

这里展示了用 paste.urlmap 挂载多个 app 的配置文件

	[composite:main]
	use=egg:Paste#urlmap
	/=home
	/blog=blog
	/wiki=wiki
	/cms=config:cms.ini

	[app:home]
	use=egg:Paste#static
	document_root=%(here)s/htdocs

	[filter-app:blog]
	use=egg:Authentication#auth
	next=blogapp 
	roles=admin 
	htpasswd=/home/me/users.htpasswd

	[app:blogapp]
	use=egg:BlogApp
	database=sqlite:/home/me/blog.db

	[app:wiki]
	use=call:mywiki.main:application 
	database=sqlite:/home/me/wiki.db

在sections的定义中，有以下几类：

* [app:main]:定义 WSGI app，main表示只有一个应用，有多个应用的话main改为应用名字
* [server:main]:定义了 WSGI server。
* [composite:xxx]：表示需要将一个请求调度定向（dispatched）到多个，或者多种应用上。以下是一个简单的例子，例子中，使用了composite，通过urlmap来实现载入多应用。
* [fliter:]：定义“过滤器”，将应用进行进一步的封装。
* [DEFAULT]：定义了一些默认变量的值。

下面会进行分段的讲解
####2.1 composite####

	[composite:main]
	use=egg:Paste#urlmap
	/=home
	/blog=blog
	/wiki=wiki
	/cms=config:cms.ini

这是一个 composite section，表示将会根据一些条件将 web 请求调度到不同的应用。use = egg:Paste#urlmap 表示我们将使用 Paste 包中名字为 urlmap 的 composite application，urlmap 是一个非常通用的 composite application。根据url 中 path 前缀建立 web 请求到应用的映射 (map)。这些被映射的 app 如:blog、home,wiki、config:cms.ini，最后一个映射被指向到当前配置文件目录中的另外一个配置文件。

下一个：

app 是一个 callable object，接受的参数(environ,start\_response)，这是 paste 系统交给 application 的，符合 WSGI 规范的参数. app 需要完成的任务是响应envrion中的请求，准备好响应头和消息体，然后交给 start\_response 处理，并返回响应消息体。

	[app:home]
	use=egg:Paste#static
	document_root=%(here)s/htdocs
	
egg:Paste#static 也是 Paste 包中的另一个简单 app，它只处理静态文件。它需要一个配置文件 document_root，后面的形如%（var）s变量，相应的值（大小写敏感）应该在 [DEFAULT] 字段指明以便Paste读取。具体的变量 %(here)s 指向包含当期配置文件的目录; 你应该用基于当前目录的相对路径来指定文件（当然你可以根据server进行改变）。比如下面：

	[DEFAULT]
	path=/etc/test
	
	[app:test]
	use=egg:Paste#static
	document_root=%(path)s/htdocs
	
然后

	[filter-app:blog]
	use=egg:Authentication#auth
	next=blogapp 
	roles=admin 
	htpasswd=/home/me/users.htpasswd

	[app:blogapp]
	use=egg:BlogApp
	database=sqlite:/home/me/blog.db

[filter-app:blog]中的 fliter-app 字段表明你希望对某个应用进行过滤，需要包装的应用通过 next 指明（表明在下一个  section 中），这个字段的意思就是，在正式调用 blogapp 之前，我会调用 egg:Authentication#auth 对登陆的用户记录日志并检查权限，随后才会调用 blogapp 进行处理。

后面的[app:blogapp]则是引用了你通过 pip install BlogApp安装的app，并指明了需要的 database 参数。

最后：

    [app:wiki]
    use=call:mywiki.main:application 
    database=sqlite:/home/me/wiki.db

这个 section 和之前的 app section类似。 除了一个重要不同的是，它没有使用egg包，而是直接引用了 mywiki.main 这个模块中的 application variable，引用包含两部分，以冒号分开，左边是模块的全名，后面是变量的路径，与python中引入相对模块的方式类似。

这是你可能用的大多是特性。

###基本用法

使用 PD 的基本方式就是加载 WSGI app。现在许多 python 框架支持 WSGI，所以在这些框架下写 app 应该是很方便的事情了。

PD 的主要是通过读取配置文件的方式载入 WSGI app。主要的函数是 paste.deploy.loadapp。给定URI后加载一个应用程序的方式如下：

    from paste.deploy import loadapp 
    wsgi_app=loadapp('config:/path/to/config.ini')

目前支持两种URI格式： `config:` 和 `egg:`。

####Config: URI
   
URI 是 config: 表示引用配置文件，如果你给loadapp()传递了 relative_to 关键词，这些文件名可以是相对路径，

**注意**
    
    文件名从来不考虑相对目前的工作目录，这是一个无法预期的位置。一般来说，URI中有一个context，将被认为是相对与context;例如，如果在一个配置文件，你有一个config:格式的URI，这个 URI 的路径被认为是相对这个配置文件的目录。

####配置格式

配置文件是INI格式的，一个简单的配置格式如下：

    [section_name]
    key = value
    another key = a long value
           that extends over multiple lines

全部的值都是字符串（不需要引号），keys和 section names
是大小写敏感的，可以包含标点符号和空格(keys和values的空行和空格会被压缩)，行之间可以包含空行。

以#或;开始的行认为是注释。

###Applications

你可以在单个配置文件中定义多个 app，每个应用有自己独立的 section。即使你只有一个应用可以应该放在一个 section 里面。

定义 application 的每个 section 应该以 app: 开头，形如 [app:name]， "main" section（表示只有一个section） 应该是 [app:main] 或 [app] 的形式。

在一个 app 中有两种引用 python 代码的方式，第一种方式是引用另一个 URI 或 name：

    [app:myapp]
    use=config:another_config_file.ini#app_name  #使用另外一个配置文件

    # or any URI:
    [app:myotherapp]
    use=egg:MyApp#使用egg包中的内容

    # or a callable from a module:
    [app:mythirdapp]
    use=call:my.project:myapplication#使用模块中的callable对象

    # or even another section:
    [app:mylastapp]
    use=myotherapp#使用另外一个section

初看这样配置似乎是没有意义的，只是指向另一个 app 的位置。实际上，除了可以加载一个app，你也可以增加和改变配置文件。

另一种方式是指向一些 python 代码。

    [app:myfacapp]
    paste.app_factory=myapp.modulename:app_factory#使用工厂函数

此模式下，必须显示地指定协议（在此是paste.app_factory），后面是需要 import 的值，在这个例子中
myapp.modulename 被加载，并从其中获取得 app_factory 的实例。

查看关于 protocol 更多信息可以参考 Defining Factories 


app\_factory 是一个callable object，其接受的参数是一些关于 app 的配置信息：(global\_conf,\*\*kwargs)，global\_conf 是在 ini 文件 default section 中定义的一系列 key-value ，而\*\*kwargs，即一些本地配置，是在ini文件中，app:xxx section 中定义的一系列key-value对。app_factory返回值是一个 app 对象


###配置

配置只能是通过 keys 中的 use（或protocol name）。任何其他 keys 将被解析为工厂函数的 keywards 参数，就像下面这样：

    [app:blog]
    use = egg:MyBlog
    database = mysql://localhost/blogdb
    blogname = This Is My Blog!

你可以在其他sections中覆盖上面的key，比如:

    [app:otherblog]
    use = blog
    blogname = The other face of my blog

这种方式下，一些设置可以被定义在一个更一般的配置文件中（如果你有 use = config:other\_config\_file） 或者通过增加一个 section 的情况下你可以发布多个（更具体）app。

###全局配置

很多情况下多个 app 共享一个相同配置文件。然而，你也经常想用完全不同的配置值的时候，你可以在其他 section 中覆盖这些变量。
一般情况下，app 不能提取额外的配置参数；通过全局配置你可以做类似的事情：如果这个 app 想知道admin email 可以这样。

全局配置会分别传给各个 app，app 必须自己定义不在全局配置中的选项;当没有局部的配置可以解析的时候，一般把全局配置作为默认。

全局配置应用于一个配置文件的多个app，这些变量我们规定放在 [DEFAULT]
section 中，如果需要全局覆盖，可以在自己的app中重新定义，如下：

    [DEFAULT]
    admin_email=webmaster@example.com

    [app:main]
    use=...
    set admin_email=bob@example.com

这里在key之前加了set

###组件应用（composite application）

组件应用与 app 类似，但是实际上是由多个 app 组成的。urlmap 就是组件应用的一个例子，不同的 url path对应了不同的 app。如下：

    [composite:main]
    use=egg:Paste#urlmap
    / = mainapp
    /files = staticapp

    [app:mainapp]
    use=egg:MyApp

    [app:staticapp]
    use=egg:Paste#static
    document_root=/path/to/docroot

组件应用 "main" 与另一个 app 类似（比如你可以用loadapp加载），但是必须访问配置文件中的其他 app。

###其他对象
    
除了app section,你可以在一个配置文件中以server:、 filter: 为前缀的方式定义 filters 和 servers，
通过 loadserver 和 loadfilter 的方式加载。配置项的机制与 app 相同。你只是得到不同的对象。

###Filter Composition

app 中应用 filter 的方式很多，重要的是看你filter的数量和组织形式。

第一种方式是 filter-with，比如:

    [app:main]
    use=egg:MyEgg
    filter-with=printdebug

    [filter:printdebug]
    use=egg:Paste#printdebug
    #and you could have another filter-with here, and so on...

你的 app filters 存在两种的特定的 section，[filter-app:...]和[pipeline:...]，这两种 section 都定义了 apps，所以可以应用到任何需要 app 的地方。

filter-app 定义了一个过滤器（正如你在[filter:...] section定义的一样），关键词 next指向了被过滤的application.

    [fliter-app:printdebug]
    use=egg:Paste
    next=main

    [app:main]
    use=egg:MyEgg

pipeline: 当你需要很多过滤器的时候，通过在配置中指定关键词pipeline(在后面加上任何你想覆盖的全局配置)。pipeline是一个以app结尾的过滤器列表。比如：

    [pipeline:main]
    pipeline = filter1 egg:FilterEgg#filter2 filter3 app

    [filter:filter1]
    ...

假设在 ini 文件中, 某条 pipeline 的顺序是filter1, filter2, filter3，app, 那么，
最终运行的 app\_real是这样组织的： app\_real = filter1(filter2(filter3(app)))

在app真正被调用的过程中，filter1.\_\_call\_\_(environ, start\_response) 首先被调用，若某种检查未通过，filter1 做出反应；否则交给 filter2.\_\_call\_\_(environ,start\_response)进一步处理，若某种检查未通过，filter2 做出反应，中断链条，否则交给filter3.\_\_call\_\_(environ, start\_response)处理，若 filter3 的某种检查都通过了，最后交给 app.\_\_call\_\_(environ,start\_response) 进行处理。

###读取配置文件

如果希望在不创建应用的情况下得到配置文件，可以使用 appconfig(uri) 函数，与 loadapp() 类似，不同的是将会以字典形式返回将被使用的配置。全局和本地的配置信息都包含在一个字典变量中，但是可以通过
属性方法 .local\_conf 和 .global\_conf 获得相应的属性。

##Egg URI

Python Eggs是由 setuptools 和 distribute 提供的版本和安装格式，setuptools 和 distribute 都增加元数据到一个正常的 python 包。

你不需要理解关于 Eggs 的全部才使用它。如果你有一个版本工具 setup.py 脚本,做如下改变:

    from distutils.core import setup

to:

    from setuptools import setup

之后，当你安装包的时候，就会以 Egg 的方式安装。

首先， Egg 最重要的是部分是有一个 specification，由你的版本名字（setup()的name关键参数）
和具体的版本号形成。所以你可以有一个名为MyApp的egg，或指定版本号的AyApp==0.1

其次是entry point， 它指向包中的 python 对象，和具体的 protocol[These are references to Python objects in your packages that are named and have a specific protocol]。"Protocol"在这里只是一种方式,声明我们用具体的参数调用他们，期望一个特定的返回值。我们会在后面讨论更多关于protocols的知识。

关键的是怎样定义我们的entry point。在setup()中增加一个参数，比如：

    setup(
        name='MyApp',
        ...
        entry_points={
            'paste.app_factory': [
            'main=myapp.mymodule:app_factory',
            'ob2=myapp.mymodule:ob_factory'],
            },
        )

这里定义了名为main和ob2两个app,你可以以 egg:MyApp#main(或egg:MyApp,由于main是默认的)和egg:MyApp:#ob2的方式应用它们。

values 是对导入对象的说明，main 位于 myapp.mymodule 模块，以 app_factory 命名的对象。

没有以导入Egg的方式增加配置到对象的方法(即以 Egg 导入方法仅此一种)，[原文There’s no way to add configuration to objects imported as Eggs.]


###Defining Factories

这让你指向工厂函数（遵守我们提到的具体的protocols）。除非你为你的 app 创建了工厂函数，否则没有什么用处。

有几个protocols: 

* paste.app_factory
* paste.composite_factory
* paste.filter_factory, 
* paste.server_factory

每个必须是可调用的(比如一个函数，方法，类)。

####1) paste.app_factory

这是最普遍的app，你可以如下定义：

    def app_factory(global_config, **local_conf):   
        return wsgi_app

global\_config是一个字典，而local\_conf则是关键字参数。返回一个wsgi app（含有call方法。）


####2) paste.composite_factory

Composites只是稍微复杂些：

    def composite_factory(loader, global_config, **local_conf): 
        return wsgi_app

loader参数是一个有几个有趣的方法的对象， get\_app(name\_or\_uri, global\_conf=None)根据name
返回一个wsgia app，get\_filter和get\_server与此一样。

一个更有意思的例子可能是是 composit factory，比如，考虑 pipeline app：

    def pipeline_factory(loader, global_config, pipeline):
        # space-separated list of filter and app names:
        pipeline = pipeline.split()
        filters = [loader.get_filter(n) for n in pipeline[:-1]]
        app = loader.get_app(pipeline[-1])
        filters.reverse() # apply in reverse order!
        for filter in filters:
            app = filter(app)
        return app

我们像下面一样使用它：

    [composite:main]
    use=<pipeline_factory_uri>
    pipeline=egg:Paste#printdebug session myapp

    [filter:session]
    use=egg:Paste#session
    store=memory

    [app:myapp]
    use=egg:MyApp


####3) paste.filter_factory 

fliter 工厂函数除了它返回的是一个 filter 之外，和 app 工厂函数类似，fliter 是一个
把一个 WSGI app 作为唯一参数的可调用的对象，返回一个过滤后的 app。 

这个 filter 例子检查REMOTE_USER CGI变量是否被设置，并创建一个简单的认证过滤器。

	def auth_filter_factory(global_conf, req_usernames):
		# space-separated list of usernames:
		req_usernames = req_usernames.split()
		def filter(app):
		    return AuthFilter(app, req_usernames)
		return filter

	class AuthFilter(object):
		def __init__(self, app, req_usernames):
		    self.app = app
		    self.req_usernames = req_usernames

		def __call__(self, environ, start_response):
		    if environ.get('REMOTE_USER') in self.req_usernames:
		        return self.app(environ, start_response)
		    start_response(
		        '403 Forbidden', [('Content-type', 'text/html')])
		    return ['You are forbidden to view this resource']


####4) paste.filter\_app\_factory 

这个filter 除了接受一个 wsgi\_app 参数和返回一个 WSGI app之外，和 paste.filter\_factory 类似，所以可以修改上面的例子为：

    class AuthFilter(object):   
        def__init__(self,app,global_conf,req_usernames):       
            ....

AuthFilter 扮演 filter\_app\_factory 的角色（在这个例子中 req_usernames 是局部的配置关键词）

####4) paste.server_factory

这个工厂函数与app和filers接受相同的参数，但是返回一个server。

一个server是接受单一 WSGI app 参数的可调用对象，而后为作为 app server。

一个例子如下:

    def server_factory(global_conf,host,port):   
        port=int(port)   
        def serve(app):       
            s=Server(app,host=host,port=port)       
            s.serve_forever()   
        return serve

Server的实现由用户来完成，可以参考python包wsgiref

####5) paste.server_runner 

除了 wsgi\_app 作为第一参数，这个 server 应该立即运行之外，与 paste.server\_factory 类似。

###Unstanding Issues 待考虑的问题

是不是每种类型的对象都有一个default 的protocol? 由于目前只有一个protocol, 似乎
非常有意义（在将来可能是多个）。期望paste.app\_factory 和 paste.composite\_factory
覆盖这个考虑。

ConfigParser的 INI 解析是匿名的。似乎是限制大于自由，有些部分是sloppy(比如[DEFAUL]部分).

conifg: URLs 应该与相对与其他位置。比如config:$docroot/.... 也许可以用global_conf 变量

其他变量可以访问global_conf么？

在配置文件中的对象是否需要是python语法（Python-syntax）的，而不总是字符串的形式？
许多代码没有thin wrapper翻译对象为合适的类型不具有可用性。

一些 filter/app 的短格式（short-form）,filter引用到"next app", 像下面这样：

    [app-filter:app_name]
    use = egg:...
    next = next_app

    [app:next_app]
    ...

最好的掌握这个模块的方法是找采用这个模块的项目。如果你没有好的项目，那么OpenStack是一个不错的选择，具体可以参考[这里](http://wenxueliu.github.io/blog/01/28/2014/openstack-paste-deploy/)


