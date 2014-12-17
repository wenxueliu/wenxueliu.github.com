
python setup.py bdist --formats=gztar,zip

在当前目录下生成 build, dist, neutron.egg-info 三个目录：

	dist/
	|-- neutron-2014.1.dev128.g84aeb9a.linux-x86_64.tar.gz  包名+版本+平台
	`-- neutron-2014.1.dev128.g84aeb9a.linux-x86_64.zip

	build
	|-- bdist.linux-x86_64		空目录
	|-- lib.linux-x86_64-2.7    库文件
	`-- scripts-2.7             执行脚本

	neutron.egg-info/   
	|-- dependency_links.txt
	|-- entry_points.txt		entry_point
	|-- not-zip-safe			不支持zip包
	|-- PKG-INFO				包的元数据
	|-- requires.txt			依赖包
	|-- SOURCES.txt				打包后包含的文件
	`-- top_level.txt			打包的 top-level 包



