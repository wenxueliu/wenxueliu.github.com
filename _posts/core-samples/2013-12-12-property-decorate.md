---
layout: post
category : python 
tagline: "how to use property decorate"
tags : [python, pythonic, decorate]
---
{% include JB/setup %}

I am apreciate the idiom(http://tomayko.com/writings/getters-setters-fuxors) which tell us how to use the property decorate in python after asking google and reading the discussion in stackoverflow.

        Do not write getters and setters. This is what the `property` built-in is for. And do not take that to mean that you should write getters and setters, and then wrap them in `property`. That means that until you prove that you need anything more than a simple attribute access, don not write getters and setters. They are a waste of CPU time, but more important, they are a waste of programmer time. Not just for the people writing the code and tests, but for the people who have to read and understand them as well. 


###explain with code.

####sever 
    class Contact(object):
    
        def __init__(self, first_name=None, last_name=None, 
                     display_name=None, email=None):
            self.first_name = first_name
            self.last_name = last_name
            self.display_name = display_name
            self.email = email
    
        def print_info(self):
            print self.display_name, "<" + self.email + ">" 

####client
	contact = Contact()
	contact.first_name = "Phillip"
	contact.last_name = "Eby"
	contact.display_name = "Phillip J. Eby"
	contact.email = "x@x.com"
	contact.print_info()

Now maybe you want set/get the first_name or last_name etc.So you may code as fllowing.

    class Contact(object):
    
        def __init__(self, first_name=None, last_name=None, 
                     display_name=None, email=None):
            self.set_first_name(first_name)
            self.set_last_name(last_name)
            self.set_display_name(display_name)
            self.set_email(email)
    
        def set_first_name(self, value):
            self._first_name = value
    
        def get_first_name(self):
            return self._first_name
    
        def set_last_name(self, value):
            self._last_name = value
    
        def get_last_name(self):
            return self._last_name
    
        def set_display_name(self, value):
            self._display_name = value
    
        def get_display_name(self):
            return self._display_name
    
        def set_email(self, value):
            self._email = value
    
        def get_email(self):
            return self._email
    
        def print_info(self):
            print self.display_name, "<" + self.email + ">" 

why this is bad style? you may think wrap the get method  and set method with property decorate. it's still a bad style. First the style is needed for the programmer who code with cpp and java but bad for pythonic. Second, you can see the cite at the first place, you will understand it.you may wonder what's the right thing to do. **go on**.

For example, we need to implement some logic that will raise an exception if an email address is set that doesn't look like an email address:

    class Contact(object):
    
        def __init__(self, first_name=None, last_name=None, 
                     display_name=None, email=None):
            self.first_name = first_name
            self.last_name = last_name
            self.display_name = display_name
            self.email = email
    
        def print_info(self):
            print self.display_name, "<" + self.email + ">"            
    
        def set_email(self, value):
            if '@' not in value:
                raise Exception("This doesn't look like an email address.")
            self._email = value
    
        def get_email(self):
            return self._email
    
        email = property(get_email, set_email)

What's happened here is that we were able to add get/set methods and still maintain backward compatibility. The following code still runs properly:

    contact = Contact()
    contact.email = "x@x.com"
    
So property are especially useful when you want to access attributes of an object directly and need to have some extra code around getters/setters in some cases.
