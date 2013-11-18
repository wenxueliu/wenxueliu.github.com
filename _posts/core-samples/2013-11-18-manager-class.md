---
layout: post
category : swift
comments : false
tags : [source code, python, swift, notebook]
---
{% include JB/setup %}

    #! /usr/bin/env python
    # -*- coding:utf-8 -*-
    # Author : Wenxueliu
    # Data   : 20130828
    # Commant: abstract from openstack swift
    # Lience : Apache 2.0

    import functools
    import re

    def command(func):
        """
        Decorator to declare which methods are accessible as commands, commands
        always return 1 or 0, where 0 should indicate success.

        :param func: function to make public
        """
        func.publicly_accessible = True

        @functools.wraps(func)
        def wrapped(*a, **kw):
            rv = func(*a, **kw)
            return 1 if rv else 0
        return wrapped

    class Manager():
        """Main class for performing commands on groups of command.
           1.you can add method with @command decorate as method
           2.if the command success make the status not 0
           3.you can add args in __init__ of  Manager more.
           you can feel it so beautiful
        """

        def __init__(self, **kwargs):
            ##here init the class  
            pass

        @command
        def start(self, **kwargs):
            """starts a server
            """
            status = 0
            ##here add you code
            print "start "
            print kwargs.items()
            status = 1

            return status

        @command
        def stop(self, **kwargs):
            """stops a server
            """
            status = 0
            ##here add you code
            print "stop"
            print kwargs.items()
            status = 1

            return status

        def get_command(self, cmd):
            """Find and return the decorated method named like cmd

            :param cmd: the command to get, a string, if not found raises
                        UnknownCommandError

            """
            cmd = cmd.lower().replace('-', '_')
            try:
                f = getattr(self, cmd)
            #this handl the cmd without in the cmd list. +++
            except AttributeError:
                raise UnknownCommandError(cmd)
            if not hasattr(f, 'publicly_accessible'):
                raise UnknownCommandError(cmd)
            return f

        @classmethod
        def list_commands(cls):
            """Get all publicly accessible commands

            :returns: a list of string tuples (cmd, help), the method names who are
                      decorated as commands

            """
            get_method = lambda cmd: getattr(cls, cmd)
            return sorted([(x, get_method(x).__doc__.strip())
                           for x in dir(cls) if
                           getattr(get_method(x), 'publicly_accessible', False)])

        def run_command(self, cmd, **kwargs):
            """Find the named command and run it

            :param cmd: the command name to run
            """
            f = self.get_command(cmd)
            return f(**kwargs)

    def simple_main():
        command = "stop"
        options = {"1":"a", "2":"b"}
        manager = Manager()
        if command in dict(manager.list_commands()).keys():
            status = manager.run_command(command, **options)
        else:
            print "you command haven't defined"




    if __name__ == "__main__":
        simple_main()
