---
layout: post
category : OpenStack
tagline : "解析"
tags : [OpenStack, python]
---
{% include JB/setup %}

###前言
   OpenStack的每个项目都是一个 python 包，它采用 setuptools 和 pbr 来构建包，由于 pbr 专为 OpenStack 定制，所以教程非常少。google 没有得到任何有用结果，所以自己深入源码解惑之。

###什么是PBR
    OpenStack 的每一个项目都是一个 python 包，但是由于 OpenStack 有十几项目、每个项目至少三个分支，通过一个可以重用的库来构建配置是一个很好的选择，PBR 在吸收 d2to1 的可声明的配置(declarative configuration)的基础上应运而生。它将setuptools 中 setup() 中的选项都写在了 setup.cfg 中来隔离配置，大部分信息都是通过 git 来维护，很好地适应了像 OpenStack 这样的大型项目。

###PBR支持的选项

####基本选项
		pbr              	setup()					type
	[medata]
	name					name					STRING
	version					version					STRING
	author					author					STRING
	author_email			author_email			STRING
	maintainer				maintainer				STRING
	maintainer_email		maintainer_email		STRING
	home_page				url						STRING
	summary					description				STRING
	keywargs				keywargs				CSV_FIELDS
	description				long_description		STRING
	download_url			download_url			STRING
	classifier				classifiers				MULTI_FIELDS
	platform				platforms				MULTI_FIELDS
	license					license					STRING
	requires_dist    		install_requires		MULTI_FIELDS
	setup_requires_dist		setup_requires			MULTI_FIELDS
	provides_dist       	provides				MULTI_FIELDS
	[file]
	obsoletes_dist			obsoletes				MULTI_FIELDS
	packages_root			packages_dir			STRING
	packages				packages				MULTI_FIELDS
	package_data			package_data			MULTI_FIELDS
	namespace_packages		namespace_packages		MULTI_FIELDS
	data_files				data_files				MULTI_FIELDS
	scripts					scripts					MULTI_FIELDS
	modules					py_modules				MULTI_FIELDS
	[global]
	commands				cmdclass				MULTI_FIELDS
	[backwards_compat]
	use_2to3				use_2to3				BOOL_FIELDS
	zip_safe				zip_safe				BOOL_FIELDS
	tests_require			tests_require			MULTI_FIELDS
	dependency_links		dependency_links		MULTI_FIELDS
	include_package_data	include_package_data	BOOL_FIELDS

以上为setup.cfg中的选项与setup()原生选项的映射

####扩展选项：

	("sources",
	 "include_dirs",
	 "define_macros", 
	 "undef_macros",
	 "library_dirs",
	 "libraries",
	 "runtime_library_dirs",
	 "extra_objects",
	 "extra_compile_args",
	 "extra_link_args",
	 "export_symbols",
	 "swig_opts",
	 "depends")


####类型说明：
* MULTI_FIELDS : 以'\n'分割多个选项
* CSV_FIELDS : 以','分割多个选项
* BOOL_FIELDS : 't','y','1','yes','true'表示True，否则为False
* STRING : 字符串

####选项说明：
  0. 所有的选项都会返回为kwargs[setup()]=pbr.value
     例如:

		home\_page = https://pypi.python.org/pypi/pbr/
		
	最终会返回 kwargs['url'] = https://pypi.python.org/pypi/pbr/。
                       
  1. 如果有description选项，忽略description_file。
	 如果没有description选项，就去寻找有没有description_file,如果有，就查询description_file的value对应的文件名，读取文件内容。
  2. 如果是packages_root,
     例如

        packages_root = lib

    它的value会被改写为kwargs['package_dir'] = {'','lib'}
  3. 如果是package_data或data_files,
     例如setup.cfg 中 
     
        data_files = 
             test1 = 
               	abc
               	def 
             test2 =
             	ghi
     被解析为 kwargs['data_files'] = [('test1',['abc','def']),('test2',['ghi'])]

**注**

1. 对于setup()中的 pbr 中会自动设置 

        zip_safe = False
        include_package_data = True
        
2. 支持自定义选项：位于 section 为 [global]，options 为 'compilers'，type 为 MULTI_FIELDS。

3. 支持扩展选项：section 以 \[extension:name\] 或 \[extension=name\] 格式，且 name 可以自定义，但不能为空。option 见上 EXTENSION_FIELDS。例如:
    
        [extension:test]
            include_dirs = 
                lib
                bin
    返回 kwargs['include_dirs']=[lib,bin]，这里扩展调用了setuptools.extension()
  
4. 支持entry_point section。例如

    	[entry_points]
    	console_scripts = 
        	test1:main
    		test2:main
    返回 kwargs['console_scripts'] = [test1:main,test2:main]

5. 如果没有command,command选项会调用setuptools.dist.Distribution.get_command_class()

6. [file]还支持extra_files，但是文件必须存在，否则异常。例如
     
        [file]
        extra_file = 
            test
            README
    
    这里的test和README必须存在

###示例
	[metadata]
	name = neutron
	version = 2014.1
	summary = OpenStack Networking
	description-file =
		README.rst
	author = OpenStack
	author-email = openstack-dev@lists.openstack.org
	home-page = http://www.openstack.org/
	classifier =
		Environment :: OpenStack
		Intended Audience :: Information Technology
		Intended Audience :: System Administrators
		License :: OSI Approved :: Apache Software License
		Operating System :: POSIX :: Linux
		Programming Language :: Python
		Programming Language :: Python :: 2
		Programming Language :: Python :: 2.7
		Programming Language :: Python :: 2.6

	[files]
	packages =
		neutron
		quantum
	data_files =
		etc/neutron =
		    etc/api-paste.ini
		etc/neutron/rootwrap.d =
		    etc/neutron/rootwrap.d/debug.filters
		    etc/neutron/rootwrap.d/dhcp.filters
		etc/init.d = etc/init.d/neutron-server
	scripts =
		bin/quantum-rootwrap
		bin/neutron-rootwrap
		bin/quantum-rootwrap-xen-dom0
		bin/neutron-rootwrap-xen-dom0

	namespace_packages =
    	neutron

	[global]
	setup-hooks =
		pbr.hooks.setup_hook
		neutron.hooks.setup_hook


	[entry_points]
	console_scripts =
		neutron-check-nsx-config = neutron.plugins.nicira.check_nsx_config:main
		neutron-check-nvp-config = neutron.plugins.nicira.check_nsx_config:main
	neutron.core_plugins =
		bigswitch = neutron.plugins.bigswitch.plugin:NeutronRestProxyV2
		brocade = neutron.plugins.brocade.NeutronPlugin:BrocadePluginV2
	
	[build_sphinx]
	all_files = 1
	build-dir = doc/build
	source-dir = doc/source

	[extract_messages]
	keywords = _ gettext ngettext l_ lazy_gettext
	mapping_file = babel.cfg
	output_file = neutron/locale/neutron.pot
	copyright_holder = OpenStack Foundation
	msgid_bugs_address = https://bugs.launchpad.net/neutron

	[compile_catalog]
	directory = neutron/locale
	domain = neutron

	[update_catalog]
	domain = neutron
	output_dir = neutron/locale
	input_file = neutron/locale/neutron.pot

	[compile_catalog]
	directory = neutron/locale
	domain = neutron

	[egg_info]
	tag_build =
	tag_date = 0
	tag_svn_revision = 0

	[upload_sphinx]
	upload-dir = doc/build/html

	[nosetests]
	exe = 1
	where=tests
	verbosity=2
	
	detailed-errors = 1
	with-doctest = true
	tests=neutron/tests
	cover-package = neutron
	cover-html = true
	
	cover-erase = true
	cover-inclusive = true	

	with-openstack=1
	openstack-red=0.05
	openstack-yellow=0.025
	openstack-show-elapsed=1
	openstack-color=1

	[pbr]
	skip_authors = True
	skip_changelog = True
	
	autodoc_index_modules = True
	[wheel]
	universal = 1

    

最终会被翻译为：

      {'author': 'OpenStack',
      'author_email': 'openstack-dev@lists.openstack.org',
      'classifiers': ['Environment :: OpenStack',
      'Intended Audience :: Information Technology',
      'Intended Audience :: System Administrators',
      'License :: OSI Approved :: Apache Software License',
      'Operating System :: POSIX :: Linux',
      'Programming Language :: Python',
      'Programming Language :: Python :: 2',
      'Programming Language :: Python :: 2.7',
      'Programming Language :: Python :: 2.6'],
      'cmdclass': {'build_sphinx': pbr.packaging.LocalBuildDoc,
      'build_sphinx_latex': pbr.packaging.LocalBuildLatex,
      'egg_info': pbr.packaging.LocalEggInfo,
      'install': pbr.packaging.LocalInstall,
      'install_scripts': pbr.packaging.LocalInstallScripts,
      'sdist': pbr.packaging.LocalSDist},
      'data_files': [('etc/init.d', ['etc/init.d/neutron-server']),
      ('etc/neutron', ['etc/api-paste.ini']),
      ('etc/neutron/rootwrap.d',
       ['etc/neutron/rootwrap.d/debug.filters',
        'etc/neutron/rootwrap.d/dhcp.filters'])],
      'description': 'OpenStack Networking',
      'entry_points': {'console_scripts': ['neutron-check-nsx-config = neutron.plugins.nicira.check_nsx_config:main',
       'neutron-check-nvp-config = neutron.plugins.nicira.check_nsx_config:main'],
      'neutron.core_plugins': ['bigswitch = neutron.plugins.bigswitch.plugin:NeutronRestProxyV2',
       'brocade = neutron.plugins.brocade.NeutronPlugin:BrocadePluginV2']},
      'include_package_data': True,
      'install_requires': ['pyudev',
      'pbr>=0.5.21,<1.0',
      'Paste',
      'PasteDeploy>=1.5.0',
      'Routes>=1.12.3',
      'amqplib>=0.6.1',
      'anyjson>=0.3.3',
      'Babel>=1.3',
      'eventlet>=0.13.0',
      'greenlet>=0.3.2',
      'httplib2',
      'requests>=1.1',
      'iso8601>=0.1.8',
      'jsonrpclib',
      'Jinja2',
      'kombu>=2.4.8',
      'netaddr>=0.7.6',
      'psutil>=0.6.1,<1.0',
      'python-neutronclient>=2.3.0,<3',
      'SQLAlchemy>=0.7.8,<=0.7.99',
      'WebOb>=1.2.3,<1.3',
      'python-keystoneclient>=0.4.1',
      'alembic>=0.4.1',
      'six>=1.4.1',
      'stevedore>=0.12',
      'oslo.config>=1.2.0',
      'python-novaclient>=2.15.0'],
      'long_description': '# -- Welcome!\n\n  You have come across a cloud computing network fabric controller.  It has\n  identified itself as "Neutron."  It aims to tame your (cloud) networking!\n\n# -- External Resources:\n\n The homepage for Neutron is: http://launchpad.net/neutron .  Use this\n site for asking for help, and filing bugs. Code is available on github at\n <http://github.com/openstack/neutron>.\n\n The latest and most in-depth documentation on how to use Neutron is\n available at: <http://docs.openstack.org>.  This includes:\n\n Neutron Administrator Guide\n http://docs.openstack.org/trunk/openstack-network/admin/content/\n\n Neutron API Reference:\n http://docs.openstack.org/api/openstack-network/2.0/content/\n\n The start of some developer documentation is available at:\n http://wiki.openstack.org/NeutronDevelopment\n\n For help using or hacking on Neutron, you can send mail to\n <mailto:openstack-dev@lists.openstack.org>.\n\n',
      'name': 'neutron',
      'namespace_packages': ['neutron'],
      'packages': ['neutron', 'quantum'],
      'scripts': ['bin/quantum-rootwrap',
      'bin/neutron-rootwrap',
      'bin/quantum-rootwrap-xen-dom0',
      'bin/neutron-rootwrap-xen-dom0'],
      'tests_require': ['hacking>=0.8.0,<0.9',
      'cliff>=1.4.3',
      'coverage>=3.6',
      'discover',
      'fixtures>=0.3.14',
      'mock>=1.0',
      'python-subunit',
      'sphinx>=1.1.2,<1.2',
      'testrepository>=0.0.17',
      'testtools>=0.9.32',
      'WebTest>=2.0',
      'configobj'],
      'url': 'http://www.openstack.org/',
      'version': '2014.1.dev128.g84aeb9a',
      'zip_safe': False}

由上可知:

* cmdclass: 没有 command,调用 setuptools.dist.Distribution.get_command_class()
* install_requires: 来自 neutron 项目中的 requirements.txt
* long_description: 来自 neutron 项目中的 README.rst 文件内容
* tests_require: 来自 neutron 项目中的 test-requirements.txt

###参考文献
[1]https://pypi.python.org/pypi/pbr/

[2]http://docs.openstack.org/developer/pbr/
