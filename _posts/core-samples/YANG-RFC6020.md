###include 

make content from a submodule
available to that submodule’s parent module, or to another submodule
of that parent module.

###submodule

Submodules allow a module
designer to split a complex model into several pieces where all the
submodules contribute to a single namespace, which is defined by the
module that includes the submodules.

###belong-to

The "belongs-to" statement specifies the module to which the
submodule belongs. the prefix is mandatory in its block.


###typedef

The "typedef" statement defines a new type that may be used locally
in the module, in modules or submodules which include it, and by
other modules that import from it

The "typedef" statement’s argument is an identifier that is the name
of the type to be defined, and MUST be followed by a block of
substatements that holds detailed typedef information.

The name of the type MUST NOT be one of the YANG built-in types.

###type

The "type" statement takes as an argument a string that is the name
of a YANG built-in type or a derived type

###container

used to define an interior data node in
the schema tree.

A container node does not have a value, but it has a list of child
nodes in the data tree. The child nodes are defined in the
container’s substatements.


###must ????

###presence

If a container has the "presence" statement, the container’s
existence in the data tree carries some meaning. Otherwise, the
container is used to give some structure to the data, and it carries
no meaning by itself.

###leaf

A leaf node has a value, but no child nodes in the data tree.

The "leaf" statement is used to define a scalar variable of a
particular built-in or derived type.

###leaf-list

the "leaf-list" statement is used to define an
array of a particular type. 

###list

used to define an interior data node in the
schema tree.

list key????

###choice

The identifier is used to identify the case node in the schema tree.
A case node does not exist in the data tree.


###anyxml

###grouping

###use
###refine
###rpc
####output
####input
###notification
###augment
###notification

###identity

used to define a new globally unique,
abstract, and untyped identity. Its only purpose is to denote its
name, semantics, and existence. An identity can either be defined
from scratch or derived from a base identity. 

####base
derive form identity

###feature

This
allows portions of the YANG module to be conditional based on
conditions on the device. The model can represent the abilities of
the device within the model, giving a richer model that allows for
differing device abilities and roles.

####if-feature


###deviation

YANG allows devices to document portions of a base module
that are not supported or supported but with different syntax, by
using the "deviation" statement.

####deviate

"not-supported", "add", "replace", or "delete"

##Common Statements

###config

config:true  配置数据
config:false 状态数据
默认决定于父节点的 config 设置，如果直到顶层都没有设置，则为 true.
如果一个节点设置为 false, 子节点不能设置为true.

###status

* current : 有效（默认）
* deprecated : 保留
* obsolete : 废弃

如果是 deprecated，不能用于 current; 如果是 obsolete 不能用于 deprecated。

例如:下面是不合法的

	typedef my-type {
		status deprecated;
		type int32;
	}
	leaf my-leaf {
		status current;
		type my-type; // illegal, since my-type is deprecated
	}
###description

对当前模块的描述

###reference

引用的其他模块或文档

###when

XPath 表达式（可以自定义），当为true，父节点定义的数据有效。否则，无效。


##Constraints


###Hierarchy of Constraints

 "must", "mandatory",
"min-elements", and "max-elements" constraints are not enforced if
the parent node has a "when" or "if-feature" property that is not
satisfied on the current device.


##Built-In Types


