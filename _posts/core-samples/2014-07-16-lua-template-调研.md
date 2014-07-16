---
layout: post
category : lua
tagline: "模板调研"
tags : [ http, web formwork]
---
{% include JB/setup %}

#lua web template 调研

##lustache

Mustache templates for Lua  https://github.com/Olivine-Labs/lustache

###特性

* used for HTML, config files, source code - anything.

* no if statements, else clauses, or for loops. Instead there are only tags.

###语法

* Variables replacement

* blocks or Sections  类似 for loops

Template:

	\{\{#stooges\}\}
	<b>{\{name\}\}</b>
	\{\{/stooges\}\}

View:

	{
	  stooges = [
		{ name = "Moe" },
		{ name = "Larry" },
		{ name = "Curly" }
	  ]
	}

Output:

	<b>Moe</b>
	<b>Larry</b>
	<b>Curly</b>
	
Template:

	\{\{#musketeers\}\}
	\* \{\{.\}\}
	\{\{/musketeers\}\}

View:

	{
	  musketeers = { "Athos", "Aramis", "Porthos", "D'Artagnan" }
	}

Output:

	* Athos
	* Aramis
	* Porthos
	* D'Artagnan

* function

Template:

	\{\{#bold\}\}Hi \{\{name\}\}.\{\{/bold\}\}

View:

	{
	  name = "Tater",
	  bold = function (self)
		return function (text, render) 
		  return "<b>" .. render(text) .. "</b>"
		end
	  end
	}

Output:

	<b>Hi Tater.</b>

* Inverted Sections  类似 if clause

Template:

    \{\{#repos\}\}<b>\{\{name}}</b>\{\{/repos}}
    \{\{^repos\}\}No repos :(\{\{/repos}}

View:

	{
	  "repos": {}
	}

Output:

	No repos :(

* comment

    Today\{\{! ignore me \}\}

* Partials 类似 template inheritance

Template

	base.mustache:
	<h2>Names</h2>
	\{\{#names}}
	  \{\{> user}}
	\{\{/names}}

	user.mustache:
	<strong>\{\{name}}</strong>

Output:

	<h2>Names</h2>
	\{\{#names}}
	  <strong>\{\{name}}</strong>
	\{\{/names}}

* Set Delimiter

    \* \{\{ default_tags }}

	\{\{=<% %>=}}

	\* <% erb_style_tags %>

	<%=\{\{ }}=%>

	\* \{\{ default_tags_again }}




## lutem

a lua template engine like a famous python template engine jinja2 https://github.com/daly88/lutem

###特性

* dynamic text generation

* dynamic html page generation

* auto code generation

###语法

* variable replacement

* Template Inheritance

	{\% extends abc.tmpl %}

	Declare block:

	{\% block blockname %}

	...

	{\% endblock %}

* for and if clauses

	{\% for k in mp %}

	...  --this area would be repeated

	{\% endfor %}


* block



##slt2

slt2 is a Lua template processor. Similar to php or jsp, you can embed lua code directly.

https://github.com/henix/slt2

###特性


* \#{ lua code }\# : embed lua code
* \#{= expression }\# : embed lua expression
* \#{include: 'file' }\# : include another template


##lua-resty-template

lua-resty-template is a compiling (HTML) templating engine for Lua and OpenResty.

https://github.com/bungle/lua-resty-template


###语法

* \{\{expression\}}, writes result of expression - html escaped

* {\*expression\*}, writes result of expression 

Lua

	local template = require "resty.template"
	local layout   = template.new("layout.html")
	layout.title   = "Testing lua-resty-template"
	layout.view    = template.compile("view.html"){ message = "Hello, World!" }
	layout:render()
	-- Or like this
	template.render("layout.html", {
	  title = "Testing lua-resty-template",
	  view  = template.compile("view.html"){ message = "Hello, World!" }
	})
	-- Or maybe you like this style more (but please remember that view.view is overwritten on render)
	local view     = template.new("view.html", "layout.html")
	view.title     = "Testing lua-resty-template"
	view.message   = "Hello, World!"
	view:render()

layout.html

	<!DOCTYPE html>
	<html>
	<head>
		<title>\{\{title}}</title>
	</head>
	<body>
		{*view*}
	</body>
	</html>

view.html 

	<h1>\{\{message}}</h1>

* {\% lua code %}, executes Lua code

lua 

	local template = require "resty.template"
	local html = require "resty.template.html"

	template.render([[
	<ul>
	{\% for _, person in ipairs(context) do %}
		{*html.li(person.name)*}
	{\% end %}
	</ul>
	<table>
	{\% for _, person in ipairs(context) do %}
		<tr data-sort="{{(person.name or ""):lower()}}">
		    {*html.td{ id = person.id }(person.name)*}
		</tr>
	{\% end %}
	</table>]], {
		{ id = 1, name = "Emma"},
		{ id = 2, name = "James" },
		{ id = 3, name = "Nicholas" },
		{ id = 4 }
	})
	
Output

	<ul>
		<li>Emma</li>
		<li>James</li>
		<li>Nicholas</li>
		<li />
	</ul>
	<table>
		<tr data-sort="emma">
		    <td id="1">Emma</td>
		</tr>
		<tr data-sort="james">
		    <td id="2">James</td>
		</tr>
		<tr data-sort="nicholas">
		    <td id="3">Nicholas</td>
		</tr>
		<tr data-sort="">
		    <td id="4" />
		</tr>
	</table>

* {(template)}, includes template file

Lua

	template.render "view.html"

view.html

	{(view.html)}

view.html

	{(users/list.html)}


* Calling Methods in Templates

Lua

	local template = require "resty.template"
	template.render([[
	<h1>\{\{header:upper()}}</h1>
	]], { header = "hello, world!" })

Output

	<h1>HELLO, WORLD!</h1>

##参考模板

* [etlua](https://github.com/leafo/etlua)
* [cgilua](http://keplerproject.github.io/cgilua/manual.html#templates)
* [orbit](http://keplerproject.github.io/orbit/pages.html)
* [turbolua mustache](http://turbolua.org/doc/web.html#mustache-templating)
* [pl.template](http://stevedonovan.github.io/Penlight/api/modules/pl.template.html)
* [lustache](https://github.com/Olivine-Labs/lustache)
* [lust](https://github.com/weshoke/Lust)
* [templet](http://colberg.org/lua-templet/)
* [luahtml](https://github.com/TheLinx/LuaHTML)
* [mixlua](https://github.com/LuaDist/mixlua)
* [lutem](https://github.com/daly88/lutem)
* [tirtemplate](https://github.com/torhve/LuaWeb/blob/master/tirtemplate.lua)
* [cosmo](http://cosmo.luaforge.net/)
* [lua-codegen](http://fperrad.github.io/lua-CodeGen/)
* [groucho](https://github.com/hanjos/groucho)
* [simple lua preprocessor](http://lua-users.org/wiki/SimpleLuaPreprocessor)
* [slightly less simple lua preprocessor](http://lua-users.org/wiki/SlightlyLessSimpleLuaPreprocessor)
* [ltp](http://www.savarese.com/software/ltp/)
* [slt](https://code.google.com/p/slt/)
* [slt2](https://github.com/henix/slt2)
* [luasp](http://luasp.org/)
* [view0](https://bitbucket.org/jimstudt/view0)
* [leslie](https://code.google.com/p/leslie/)
* [fraudster](https://bitbucket.org/sphen_lee/fraudster)
* [lua-haml](https://github.com/norman/lua-haml)
* [lua-template](https://github.com/tgn14/Lua-template)
* [hige](https://github.com/nrk/hige)
* [lapis html generation](http://leafo.net/lapis/reference.html#html-generation)
* [lua-resty-template](https://github.com/bungle/lua-resty-template)
