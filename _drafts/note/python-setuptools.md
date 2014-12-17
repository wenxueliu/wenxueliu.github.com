
为什么用setuptools
----------------------------------------------------------------
Easily download, build, install, upgrade, and uninstall Python packages

Setuptools is a collection of enhancements to the Python distutils (for Python 2.6 and up) that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages.


Building and Distributing Packages with Setuptools
----------------------------------------------------------------

基本用法
================================================================

这是一个最小的setup script

	from setuptools import setup, find_packages
	setup(
		name = "HelloWorld",
		version = "0.1",
		packages = find_packages(),
	)

如你所见，这个project并没有用太多setuptools的知识。尽管如此，这个project可以生成egg, 上传到PypI，与setup.py在同一目录中的package可以自动被打包。

但是，你一定不会满足与此，那么让我们看一个更复杂点的例子

	from setuptools import setup, find_packages
	setup(
		name = "HelloWorld",
		version = "0.1",
		packages = find_packages(),
		scripts = ['say_hello.py'],

		# Project uses reStructuredText, so ensure that the docutils get
		# installed or upgraded on the target machine
		install_requires = ['docutils>=0.3'],

		package_data = {
		    # If any package contains *.txt or *.rst files, include them:
		    '': ['*.txt', '*.rst'],
		    # And include any *.msg files found in the 'hello' package, too:
		    'hello': ['*.msg'],
		},

		# metadata for upload to PyPI
		author = "Me",
		author_email = "me@example.com",
		description = "This is an Example Package",
		license = "PSF",
		keywords = "hello world example examples",
		url = "http://example.com/HelloWorld/",   # project home page, if any

		# could also include long_description, download_url, classifiers, etc.
	)

这似乎是一个比较完整的包结构了，也许，你会有一个疑问？到底有多少个选项我可以用？好的，让我们来看看


####setup所支持的选项

除了version，其他的选项都是可选的，如果你不需要相应的 setuptools 的特性，你就不用在setup()中使用它们。

* version

>> pre-release tag
c:与pre,preview,rc相同，即2.4rc1,2.4pre1,2.4preview1与2.4c1都指相同版本号。当然它们也可以是2.4-rc1,2.4-pre1,2.4-preview1,2.4-c1。
它们都比版本2.4旧。

>>post-release tag**
Post-release tags 一般是在release number后面加上patch numbers, port numbers, build numbers, revision numbers, 或 date stamps。比如2.4-r1263，2.4-20051127，它们都比版本2.4新。

如果你对一个版本号不太熟悉，可以用from pkg_resources import parse_version来进行比较。如

	>>> from pkg_resources import parse_version
	>>> parse_version('1.9.a.dev') == parse_version('1.9a0dev')
	True
	>>> parse_version('2.1-rc2') < parse_version('2.1')
	True
	>>> parse_version('0.6a9dev-r41475') < parse_version('0.6a9')
	True


###data files
**include_package_data** 
如果你的 data files 通过 CVS 或 SVN 版本控制，那么你只要设置为 True 即可，否则，需要将要打包的 data files 都放在 MANIFEST.in
文件里面。当然你可以写一个插件来支持你的版本控制系统，见下文。
	
	from setuptools import setup, find_packages
	setup(
		...
		include_package_data = True
	)

**package_data**

这是对 include\_package\_data 的弥补，因为 include\_package\_data 不能做全部的事情。比如，你用 git 来版本控制，而 MANIFEST.in 又无法满足你的需要。例如：

	from setuptools import setup, find_packages
	setup(
		...
		package_data = {
		    # If any package contains *.txt or *.rst files, include them:
		    '': ['*.txt', '*.rst'],
		    # And include any *.msg files found in the 'hello' package, too:
		    'hello': ['*.msg'],
		    # And include any *.dat files found in the 'data' subdirectory
        	# of the 'mypkg' package, also:
        	'mypkg': ['data/*.dat'],
		}
	)	



**exclude_package_data**: 
 
列出package中不想被打包的文件或目录。支持*（指代任意文件，如\*.txt表示任意txt文件）。

	from setuptools import setup, find_packages
	setup(
		...
		packages = find_packages('src'),  # include all packages under src
		package_dir = {'':'src'},   # tell distutils packages are under src

		include_package_data = True,    # include everything in source control

		# ...but exclude README.txt from all packages
		#即使include\_package\_data和package\_data 已经包含了 README.txt, 此时仍然不会被打包。
		exclude_package_data = { '': ['README.txt'] },
	)

注：
1.即使被 package\_data 和 include\_package\_data包含的文件，只要被 exclude\_package\_data 排除，就不必被打包。
2.如果你之前包含过的data files 经过修改后，不再被包含了，运行 setup.py clean --all 清除这些包。
3.'*'会应用与所有的 package，并且会遍历子目录。
4.路径分割符号统一为'/'，在 windows 下会自动转换。
5.如果include_package_data = True，除非你想增加由 setup script 和 build process产生的文件，否则 package_data 选项不需要。

###Non-Package Data Files
setuptools 并没有把 data files 安装在另外一个位置，而是与 egg 文件和 包文件在一起。当你想把这些 data files 放在另外一个位置的时候，
你应该用在包中增加一个脚本，调用 Resource Management API 来做这件事，如下

	from pkg_resources import Requirement, resource_filename
	filename = resource_filename(Requirement.parse("MyProject"),"sample.conf")



###Declaring Dependencies

当用 setuptools 安装包的时候，它会自动安装依赖，并且在python Egg中记录依赖信息。setuptools 和 pkg_resources使用共同的语法来
指明 project 的需要依赖包。

**install_requires**
如果 requirement 只有一个，那么在 setup() 中指明即可，如果多于一个，就每行一个。
也可以创建一个requirements文件，将依赖包每行一个写入reqirement。


例如：

	setup(
		name="Project-B",
		install_requires = ["Project-A[PDF]"],
		...
	)	

例如：

	docutils >= 0.3, <= 0.6
	pbr > 0.3, < 0.6
	# comment lines and \ continuations are allowed in requirement strings
	BazSpam ==1.1, ==1.2, ==1.3, ==1.4, ==1.5, \
		==1.6, ==1.7  # and so are line-end comments
    dutils != 0.6
	PEAK[FastCGI, reST]>=0.5a4

	setuptools==0.5a7

注意:
1.Python Egg distributions will include a metadata file listing the dependencies.
2.如果你在 setup.py 中声明了依赖，就不需要在脚本和模块中使用 require() 函数
3.正确的 requirement 是包安装和运行的保证。
4.">1, >2" becomes ">1", and "<2,<3" becomes "<3"
5.project 或版本号（version number)中包含空格或非标准字符（nonstandard characters ）都应该用'-'代替。





**zip_safe**: 
True OR False（default);
如果为True，表明 project 可以从 zip 文件安装和运行。
如果为Flase，每编译一次egg，bdist_egg命令会分析project下的每一个文件，检查是否能从zip安装和运行。



**entry_points**: dict; 生成其他脚本可以import 和运行的函数。
例如
	setup(
		# other arguments here...
		entry_points = {
		    'console_scripts': [
		        'foo = my_package.some_module:main_func',
		        'bar = other_module:some_func',
		    ],
		    'gui_scripts': [
		        'baz = my_package_gui.start_func',
		    ]
		}
	)

在非windows平台，foo，bar，baz将被安装，
在windows平台，将创建foo.exe, bar.exe, and baz.exe等文件。完全不必理会foo.py bar.py baz.py文件。.exe 会找到正确的python版本来执行.py文件。

###Declaring “Extras” (optional features with their own dependencies
**extras_require**
{extras:[stringlist ]}
当一个包不是为整个 project 所需要，只是为了支持某个特性而必须安装的包。

例如
	#为了输出 PDF 需要ReportLab包，为了支持 reStructuredText 需要 docutils 包。
	setup(
		name="Project-A",
		...
		entry_points = {
		    'console_scripts': [
		        'rst2pdf = project_a.tools.pdfgen [PDF]',
		        'rst2html = project_a.tools.htmlgen [reST]',
		        # more script entry points ...
		    ],
		}

		extras_require = {
		    'PDF':  ["ReportLab>=1.2", "RXP"],
		    'reST': ["docutils>=0.3"],
	)
    #这里安装 Project-A 并不会安装 extras_require 的包，只有当其他包（如下Project-B）
    #需要 Project-A 支持 PDF 或 reST 特性的时候，才会安装对应的包。 

例如

    #即使 Project-A 已经安装，如果 Project-B 被安装，ReportLab 将被安装，因为 Project-B 
    #需要 Project-A 支持 PDF。
	setup(
		name="Project-B",
		install_requires = ["Project-A[PDF]"],
		...
	)
    #上游（upstream）的 Project-B 并不需要关心支持 PDF 特性要安装什么包，具体实现由下游
	#(downstream)的 Project-A 来决定，比如 PDF 特性依赖的包又增加了，此时 Project-B 并
	#不需要做任何改变，只需要修改 Project-A。


例如
	#Project-A 不在需要 PDF 支持，只要为 [] 即可。这样不需要修改 Project-B
	setup(
		name="Project-A",
		...
		extras_require = {
		    'PDF':  [],
		    'reST': ["docutils>=0.3"],
		}
	)

**setup_requires**: 
为了 setup script 运行而需要的 distributions。在执行剩余的setup script 或 commands之前，setuptools 将尽力去获取这些distributions，即使通过 EasyInstall 下载。如果distutils extensions为你的编译过程中的一部分，需要这个选项。
 
当 setup script 运行的时候，如果你想这些 distributions 被安装并且可用，应该增加到install_requires 和 setup_requires 中。


**dependency_links**: 
如果要安装 setup_requires 或 tests_require 中的依赖包，这里的 link 将会被使用。当安装一个 .egg 文件，这些 link 将被写入 egg 的元数据中。

	setup(
		...
		dependency_links = [
		    "http://peak.telecommunity.com/snapshots/"
		],
	)


**namespace_packages**
URL 必须是：
* direct download URLs,
* the URLs of web pages that contain direct download links
* the repository’s URL
更多见 Name Packages

**test_suite**: 
可以是
* unittest.TestCase 的子类
* 一个无参函数，返回unittest.TestSuite
* 一个模块，有 additional_tests() 函数，这个函数将被调用，并增加到将要运行的测试中
* 一个包，任何子模块，子包都被递归增加到整个 test suite

**tests_require**
你的 project 测试需要一个多个额外的包，你可以用这个选项。当你运行 test 命令的时候，setuptools 将为你获取这些包（即使通过 EasyInstall 下载），
需要注意的是这些包只是下载到 setup.py 文件所在目录，并不会安装。


**test_loader**
如果你不想用 setuptools 一般的运行测试的方式，想用一个不同的方式运行测试，你可以在 setup() 中具体这个参数，参数的 value 可以是模块名或类名。
类名中的 instance 不能有参数，并且在 python 的 unittest 模块的 TestLoader 类中必须定义 loadTestsFromNames() 方法。 Setuptools 将传递 names 
参数给 test_suite。

模块或类名必须以":"分割，默认的值是 setuptools.command.test:ScanningLoader。如果你想用默认的 unittest，可以用 unittest:TestLoader，这将
防止自动地扫描子模块和子包。

当 test 命令运行的时候，只要你用 tests_require 确保包含加载类的包可以用，这些模块或类（test_loader的值）可以包含在另一个包中。


**eager_resources**
如果你正在使用的工具是“真正的”文件，或者你的 project 包含非扩展（non-extension）的原生库，亦或你的 C 扩展期望能够访问某些文件，在 setup() 的 eager_resources 中列出这些文件。不管什么时候 C  扩展被导入，这些文件将被
一起提取。

非常重要的一点是，如果你的 project 包含非distutils-built C 扩展的共享库，这些库的文件的扩展名不是 .dll .so .dylib（这些扩展的文件会被setuptools 0.6a8 或更高版本检测作为共享库并增加到native_libs.txt），由于这些文件在 C 扩展链接它们的时候，出现在文件系统中，那么你需要将这些文件在 eager_resources 中列出。

不管什么时候，resource_filename() API 需要任何 C 扩展或者eager resource， pkg_resources runtime 将同时自动提取所有的 C 扩展和 eager_resources（内部 C 扩展通过 resource_filename()），这将确保 C 扩展看到所有希望看到的'真实'文件。

你也可以在 eager_resources 列出目录资源，不管什么时候，C 扩展或者 eager resource 请求的时候，目录的内容将被提取。

如果需要 eager_resources 中的任何一个 resource，或者 project 需要导入 C 扩展，eager_resources 中的所有 resource 都必须被提取。当 project 被
安装为 zip 文件的时候，这个参数才有用，全部 resources 将作为一个整体被提取到文件系统。 

Resources 的路径相对于 source root，以'/'为路径分隔符。比如你要列出bar.baz中的foo.png，应该为 bar/baz/foo.png。

如果你一次只需要一个资源，或者你没有 C 扩展访问 project 中的其他文件，这个选项并不需要。

如果你不确定你是否需要这个参数，请不要使用。这是主要用于有大量非 python 的依赖，作为你处理过于复杂（crufty）包所能使用的最后手段	。

如果你的包是纯 python， python 和 data files，python 和 C，你真不需要这个参数。



**use_2to3**
在编译的时候，用 2to3 工具将源代码从 python2 转为 python3。
**convert_2to3_doctests**
列出需要用 2to3 工具转换的 doctest 源文件。
**use_2to3_fixers**: 
列出在 2to3 转换的过程中需要的模块（搜索将要用到的额外 fixers）。

例如：

	from setuptools import setup
	import sys

	extra = {}
	if sys.version_info >= (3,):
		extra['use_2to3'] = True
		extra['convert_2to3_doctests'] = ['src/your/module/README.txt']
		extra['use_2to3_fixers'] = ['your.fixers']

	setup(
		name='your.module',
		version = '1.0',
		description='This is your awesome module',
		author='You',
		author_email='your@email',
		package_dir = {'': 'src'},
		packages = ['your', 'you.module'],
		test_suite = 'your.module.tests',
		**extra
	)


###find_packages
在小的工程中，手动增加 package 到 setup() 的 package 参数中没有什么问题，但是在大型工程中,如Twisted, PEAK, Zope, Chandler, etc，当项目内容变化后，维护这个 package参数将是一个非常大的负担，setuptools.find_packages() 应运而生。

	from setuptools import find_packages
	find_packages(src\_dir, exclude):
		src_dir : 搜索包的目录，如果为空，则搜索setup.py所在目录。
		exclude : 忽略的文件
		return：赋值给 setup()的 package 参数

find_packages()在 src\_dir 中通过 __init__.py 来定位 python 包,然后用 exclude 过滤。

     
如：find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])，忽略所有的与tests相关的包。

###Automatic Script Creation

通过正确的扩展，setuptools可以自动为你创建脚本，在 windows 下会创建一个.exe 文件,使得用户并不需要改变 PATHEXT 的设置。要做到这样的
功能，只需要在setup script中定义 entry_points，指明脚本应该导入和运行什么函数。

	#这里创建了两个 console 脚本 foo，bar。一个 GUI 脚本baz
	setup(
		# other arguments here...
		entry_points = {
		    'console_scripts': [
		        'foo = my_package.some_module:main_func',
		        'bar = other_module:some_func',
		    ],
		    'gui_scripts': [
		        'baz = my_package_gui.start_func',
		    ]
		}
	)
	
在非 windows 平台，foo，bar，baz 三个脚本将被安装，调用的时候不需要参数，返回值会传给 sys.exit()，你可以将 errorlevel 或错误信息输出到 stderr。
在 windows 平台，将创建 foo.exe, bar.exe, and baz.exe 等启动器。完全不必理会 foo.py bar.py baz.py 文件。.exe 会找到正确的 python 版本来执行 .py 或 .pyw 文件。

注：
1.你可以具体一个 extras，当脚本运行的时候，将被增加到 sys.path。

###“Eggsecutable” Scripts
为了让一个 .egg 文件可执行。见如下：

	setup(
		# other arguments here...
		entry_points = {
		    'setuptools.installation': [
		        'eggsecutable = my_package.some_module:main_func',
		    ]
		}
	)

在 Unix-like 平台中，需要给.egg 可执行权限，并且在 PATH 环境变量中增加相应的 python 版本。
增加 Eggsecutable 的Eggs不能被重命名或通过符号链接调用。

###Namespace Packages
有时候将一个大的包分散为多个小的 egg 的集合将更有用。正常情况下，python 不允许一个包的内容来自多个 location，Namespace Packages 为解决这个问题
应运而生，当你声明一个包为 Namespace Packages 的时候，意味着 __init__.py 的内容没有意义，仅仅是一个模块和子包（subpackages）的容器

pkg_resources runtime 将确保 Namespace Packages 的内容能在“虚拟”包（由多个 eggs 或 目录形成）中传递。


	#ZopeInterface
	setup(
		# ...
		namespace_packages = ['zope']
	)
    #可用看出 ZopeInterface 是 zope 的一个子包。

	#ZopePublisher
	setup(
		# ...
		namespace_packages = ['zope']
	)
	#可用看出 ZopePublisher 是 zope 的一个子包。

这样，zope.interface 和 zope.publisher 都是单独的包，位于不同的位置，但是它们都声明为 zope 的 namespace packages，在安装和运行的时候，python 会把它们当成“虚拟” zope 包的一部分。

注：
1.Namespace Packages 并不需要为顶层（top-level）的包，zope.app 就是一个 Namespace Packages。
2.在一般的 python 包结构中，你 project 的源码 tree 中必须包含 namespace packages 的__init.py__ 文件，__init__.py 必须包含
	
	__import__('pkg_resources').declare_namespace(__name__)

确保 namespace package 机制工作，目前的包注册为 namespace package。
3. 在namespace package 的 __init__.py 中不能包含任何其他代码。即使在开发和作为 .egg 文件的时候，包是可以工作的，但是当用系统包工具安装的时候，project 的形成的包将无法工作，因为__init__.py 不会被包含进工程。
4.不管哪个 project 包的 __init.py__ 首先被加载，为了确保 namespace 被声明，你必须在 namespace package 子包的 project 的 __init__.py 中包含
	
	__import__('pkg_resources').declare_namespace(__name__)

以确保 namespace 被声明，因为一旦有 __init__.py 没有声明 namespace，则后续的声明将失败。
5.只要在namespace package 的**每一个**子包(subpackage)的 __init__.py 中声明
	
	__import__('pkg_resources').declare_namespace(__name__)

并且**没有包含任何其他代码**，在运行的时候，egg runtime system会自动将各个子包合成（merge）一个包。


###Dependencies that aren’t in PyPI
如果 project 依赖包不在 PyPI 中，你仍然可以通过如下方式寻找依赖：

* an egg, in the standard distutils sdist format,
* a single .py file, or
* a VCS repository (Subversion, Mercurial, or Git).

你要做到而是增加一些 URL 到 setup() 的 dependency_links中。URL 必须为下面中的一种:

* direct download URLs,
* the URLs of web pages that contain direct download links, or
* the repository’s URL

如果需要的包在 SourceForge 中维护，你也可以用 showfiles.php 中指明link。

如果需呀的包是一个单独的 .py 文件，必须在 URL 中后缀 #egg=project-version，指明需要的 project和版本号。

如果是 VCS ，应该增加后者 #egg=project-version,也可以在URL路径中用@REV，还有一种选项是增加一个前缀，如下:

* svn+URL for Subversion,
* git+URL for Git, and
* hg+URL for Mercurial

例如

	vcs+proto://host/path@revision#egg=project-version



###Dynamic Discovery of Services and Plugins
在 project 中,通过注册一个entry points来扩展应用程序，可以被应用程序和框架导入，

	setup(
		# ...
		entry_points = {'blogtool.parsers': '.rst = some_module:SomeClass'}
	)

	setup(
		# ...
		entry_points = {'blogtool.parsers': ['.rst = some_module:a_func']}
	)

	setup(
		# ...
		entry_points = """
		    [blogtool.parsers]
		    .rst = some.nested.module:SomeClass.some_classmethod [reST]
		""",
		extras_require = dict(reST = "Docutils>=0.3.5")
	)

###Defining Additional Metadata

一些扩展应用或框架也许需要在 egg 中定义它们自己的元数据，使得他们能够通过pkg_resources 
元数据 APIs访问。一般是插件开发者在 ProjectName.egg-info 目录中包含额外的元数据，然而
手动创建这样的文件非常枯燥。见 Creating distutils Extensions。


###“Development Mode”
###Using setuptools... Without bundling it

也许你没有安装 setuptools，或者安装了错误的版本。为了修复这个bug，下载 ez_setup.py，放在 setup.py 脚本的同一目录（确保增加到你的版本控制系统）。
然后在你的 setup.py 的最开始两行增加如下两行：

	import ez_setup
	ez_setup.use_setuptools()

如果你的系统没有 setuptools，ez_setup 模块会自动从 PYPI 下载匹配的版本。不管什么时候你更新 setuptools 的时候，你应该更新你工程的 ez_setup.py 
文件。这样一个匹配的版本将会安装在你的系统。

为了更新你的包，可以用如下命令：

	setup.py bdist_egg upload 更新到PYPI
	setup.py sdist upload   更新egg
	setup.py register sdist bdist_egg upload 注册更新一条龙服务

如果你要安装指定版本的 setuptools，在 use_setuptools() 传递指定版本号。

###Setting the zip_safe flag
为了最好的性能，Python 包最好以 zip 文件格式安装。因为人们期望能够像访问操作系统文件一样访问 source code 或 data files，然而，不是所有的包能够以压缩的格式运行，所以，setuptools 可以以 zip 文件或一个目录的两种形式安装你的 project，具体由包的 zip_safe 标志决定。

	setup.py easy_install --zip-ok 安装为zip文件
	setup.py easy_install   安装为一个目录

如果你不能确定你的包是否能安装为 zip，可以测试以zip\_safe为 True 和 False，最后来设置 zip_safe。

**Note**
现在的版本会自动调用 declare_namespace()，但是后续的版本也许不会支持，所以确保在你的 __init__.py 文件包含 declare\_namespace()。







使用包
----------------------------------------------------------------
###Using setup.py

The basic usage of setup.py is:

$ python setup.py <some_command> <options>

To see all commands type:

$ python setup.py --help-commands

And you will get:

Standard commands:
  build             build everything needed to install
  build_py          "build" pure Python modules (copy to build directory)
  build_ext         build C/C++ extensions (compile/link to build directory)
  build_clib        build C/C++ libraries used by Python extensions
  build_scripts     "build" scripts (copy and fixup #! line)
  clean             clean up temporary files from 'build' command
  install           install everything from build directory
  install_lib       install all Python modules (extensions and pure Python)
  install_headers   install C/C++ header files
  install_scripts   install scripts (Python or otherwise)
  install_data      install data files
  sdist             create a source distribution (tarball, zip file, etc.)
  register          register the distribution with the Python package index
  bdist             create a built (binary) distribution
  bdist_dumb        create a "dumb" built distribution
  bdist_rpm         create an RPM distribution
  bdist_wininst     create an executable installer for MS Windows
  upload            upload binary package to PyPI

Extra commands:
  rotate            delete older distributions, keeping N newest files
  develop           install package in 'development mode'
  setopt            set an option in setup.cfg or another config file
  saveopts          save supplied options to setup.cfg or other config file
  egg_info          create a distribution's .egg-info directory
  upload_sphinx     Upload Sphinx documentation to PyPI
  install_egg_info  Install an .egg-info directory for the package
  alias             define a shortcut to invoke one or more commands
  easy_install      Find/get/install Python packages
  bdist_egg         create an "egg" distribution
  test              run unit tests after in-place build
  build_sphinx      Build Sphinx documentation

usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help

**alias**
	添加
	[wenxueliu@linuxerneutron]$ sudo python setup.py alias --global-config daily egg_info --tag-svn-revision     --tag-build=development
	[sudo] password for wenxueliu: 
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Writing /usr/lib/python2.7/distutils/distutils.cfg
	[wenxueliu@linuxerneutron]$ sudo python setup.py alias 
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Command Aliases
	---------------
	setup.py alias --global-config daily egg_info --tag-svn-revision --tag-build=development
	[wenxueliu@linuxerneutron]$ sudo python setup.py alias bdist_egg bdist_egg rotate -k1 -m.egg
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Writing setup.cfg
	[wenxueliu@linuxerneutron]$ sudo python setup.py alias 
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Command Aliases
	---------------
	setup.py alias bdist_egg bdist_egg rotate -k1 -m.egg
	setup.py alias --global-config daily egg_info --tag-svn-revision --tag-build=development

	删除
	[wenxueliu@linuxerneutron]$ sudo python setup.py alias --global-config --remove daily
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Deleting empty [aliases] section from /usr/lib/python2.7/distutils/distutils.cfg
	Writing /usr/lib/python2.7/distutils/distutils.cfg

	[wenxueliu@linuxerneutron]$ sudo python setup.py alias --remove bdist_egg
	[pbr] Excluding argparse: Python 2.6 only dependency
	running alias
	Deleting empty [aliases] section from setup.cfg
	Writing setup.cfg

如果 --global-config 则为全局，否则为本 project。

aliases 可以基于 project-specific，per-user，sitewide。

**bdist_egg**

这个命令为 project 产生一个 Python Egg(.egg 文件)。由于他们是跨平台的，可以直接导入，project 元数据包含了脚本和 project 依赖信息，所以 Egg 优先采用可以被 EasyInstall 安装的二进制版本格式（binary distribution format）。它们可以简单地被下载和增加到系统路径(sys.path)，他们可以被放在系统路径一个目录之后，可以被 egg runtime system 发现。

这个命令运行 egg_info 命令（如果还没有运行）来更新project 的元数据(.egg-info)目录。如果你的 .egg-info 目录已经增加了任何额外的元数据文件，这些文件
将被包含在新的 egg 文件的元数据目录，并为 egg runtime system 和 任何应用程序或框架（需要使用这些元数据）使用。

###Classifiers
https://pypi.python.org/pypi?%3Aaction=list_classifiers

Python2 转为 Python3
----------------------------------------------------------------

参考文献
----------------------------------------------------------------
[简单入门](http://pythonhosted.org/an_example_pypi_project/setuptools.html)
[官方文档](https://pypi.python.org/pypi/setuptools)
[编译包详细文档](https://pythonhosted.org/setuptools/setuptools.html)
[全部文档](https://pythonhosted.org/setuptools/)
[版本命令](http://docs.openstack.org/developer/pbr/semver.html)
