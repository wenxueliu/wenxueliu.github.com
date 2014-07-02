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

-----------------------------------------------------------------------------------------------------------------------

Understand how the property work and where you use it will make you be a better pythonic

##[how property docerate work](http://docs.python.org/2.7/library/functions.html?highlight=property#property) 
property([fget[,fset[,fdel[,doc]]]])
* return: a property attribute for new-style classes (**classes that derive from object**).
* fget: a function for getting an attribute value
* fset: is a function for setting
* fdel: is a function for del’ing, an attribute.  

Typical use is to define a managed attribute x:

class C(object):
    ​def __init__(self):
    ​    ​self._x=None

    ​def getx(self):
    ​    ​return self._x

    ​def setx(self,value):
        ​​self._x=value

    ​def delx(self):
    ​    ​del self._x

    x = property(getx,setx,delx,"I'm the 'x'property.")

If then c is an instance of C, c.x will invoke the getter, c.x=value will invoke the setter and del c.x the deleter.

If given, doc will be the docstring of the property attribute. Otherwise, the property will copy fget‘s docstring (if it exists).

This makes it possible to create read-only properties easily using property() as a decorator:

class Parrot(object):
    ​def __init__(self):
    ​    ​self._voltage = 100000

    ​@property
    ​def voltage(self):
    ​    ​"""Get the current voltage."""
    ​    ​return self._voltage

turns the voltage() method into a “getter” for a read-only attribute with the same name.

A property object has getter,setter, and deleter methods usable as decorators that create a copy of the property with the corresponding accessor function set to the decorated function.

This is best explained with an example:

class C(object):
    ​def__init__(self):
    ​    ​self._x=None

    ​@property
    ​def x(self):‍
    ​    ​"""I'm the 'x'property."""
    ​    ​return self._x

    ​@x.setter
    ​def x(self,value):
    ​    ​self._x=value
    
    @x.deleter
    ​def x(self):
    ​    ​del self._x

This code is exactly equivalent to the first example.

Be sure to give the additional functions the same name as the original property(x in this case.)

The returned property also has the attributes fget,fset, and fdel corresponding to the constructor arguments.

**New in version 2.2**

Changed in version 2.5:Use fget‘s docstring if no doc given.
Changed in version 2.6:The getter,setter, and deleter attributes were added.


##where to use the property?
You can understand how the property works from the above. Otherwise you still confuse when and where you use it. 

###read-only
As all attributes are public in Python. Starting names with an underscore or two is just a warning that the given attribute is an implementation detail that may not stay the same in future versions of the code. 

###add extra code
Properties are especially useful when you want to access attributes of an object directly and need to have some extra code around getters/setters in some cases.

Imagine you have a car object.

class  Car(object):        
def__init__(self,make,model,year,vin):                
   self.make=make                
   self.model=model                 
   self.year=year                 
   self.vin=vin           

my_car = Car('Ford','Taurus',2005,'123ABCVINSARELONG999')

Now imagine that I need to change the year on the car.  I might do it a couple different ways.

my_car.year=2006
my_car.year='2007'

The second way gives me some serious problems if I'd like to assume the year is a number.  See the following:

if(my_car.year>2010):        
print('Shiney new!')

The challenge is that setting attributes directly is very clean and slick, but is not providing us an opportunity to do any data validation (maybe I want to ensure the make is in a list of known makes, for example) or data type conversion (in the year example).

Properties come to the rescue by allowing us to setup wrapper code around attribute assignment and/or retrieval but not requiring that we do that for all attributes or even requiring us to know ahead of time that we want to do some extra validation.  I can, after the fact, modify my Car object as follows and it would still be used in exactly the same way.

	class Car(object):
		​def  __init__(self,make,model,year,vin):                
		​   self.make=make                
		​   self.model=model                 
		​   self._year=year                 
		​   self.vin=vin                
	 
		​def  get_year(self):                
		​   return self._year                  
		​
		​def  set_year(self,val):                
		​   self._year=int(val)       
  
    ​year=property(get_year, set_year)

You still access year as my_car.year and you still set it with my_car.year = 2006, but it now converts the value into an int where necessary and throws an exception if the value provided by the user doesn't convert into an int properly.

Note：
1. When you reference self.year, it fires get_year().  get_year() references self.year.
2. when you changer _year to year , it will get "maximum recursion exceeded'.

**Besides**, properties is that they make the class less transparent. Especially, this is an issue if you were to raise an exception from a setter. For example, if you have an Account.email property:

	class Account(object):
		@property
		def email(self):
		   return self._email

		@email.setter
		def email(self, value):
		   if '@' not in value:
       raise ValueError('Invalid email address.')
   self._email = value

then the user of the class does not expect that assigning a value to the property could cause an exception:

a = Account()
a.email = 'badaddress'
--> ValueError: Invalid email address.


###update dynamiclly
You use propertyto evaluate the value an object attribute that is a funciton of other attributes. Consider the following class:

class shape:   
def __init__(self,l,w):       
self.length = l        
self.width = w    

@property   
def area(self):       
return self.length*self.width

If we change the value of either length or width, the value of area will be updated immediately when we ask for it.

	[In:] s=shape(2,3)
	[In:] print s.length,s.width,s.area
	[Out:]2 3 6
	[In:] s.length=5
	[In:] print s.length,s.width,s.area
	[Out:]5 3 15

To help clarify how this works, consider the following case: 
if we try to defineareain__init__():

class shape2:   
def__init__(self,l,w):       
self.length=l        
self.width  =w        
self.area = self.length*self.width

area will only be evaluated when the class is instantiated. Changing the values oflengthorwidthwon't modify the value ofarea:

	[In:] s=shape2(2,3)
	[In:] prints.length,s.width,s.area
	[Out:]2 3 6
	[In:] s.length=5
	[In:] prints.length,s.width,s.area
	[Out:]5 3 6

###a similar about the implement of  property decorate.

The property() function returns a special descriptor object:

>>> property()
<property object at 0x10ff07940>

It is this object that has extra methods:

>>> property().getter
<built-in method getter of property object at 0x10ff07998>
>>> property().setter
<built-in method setter of property object at 0x10ff07940>
>>> property().deleter
<built-in method deleter of property object at 0x10ff07998>

These act as decorators too. They return a new property object:

>>> property().getter(None)
<property object at 0x10ff079f0>

that is a copy of the old object, but with one of the functions replaced.

Remember, that the @decorator syntax is just syntactic sugar; the syntax:

@property
def foo(self): return self._foo

really means the same thing as

def foo(self): return self._foo
foo = property(foo)

so foo the function is replaced by property(foo), which we saw above is a special object. Then when you use @foo.setter(), what you are doing is call that property().setter method I showed you above, which returns a copy of the same property, but with the setter function replaced with the decorated method.

The following sequence also creates a full-on property, by using those decorator methods.

First we create some functions and a property object with just a getter:

>>> def getter(self): print 'Get!'
... 
>>> def setter(self, value): print 'Set to {!r}!'.format(value)
... 
>>> def deleter(self): print 'Delete!'
... 
>>> prop = property(getter)
>>> prop.fget is getter
True
>>> prop.fset is None
True
>>> prop.fdel is None
True

Next we use the .setter() method to add a setter:

>>> prop = prop.setter(setter)
>>> prop.fget is getter
True
>>> prop.fset is setter
True
>>> prop.fdel is None
True

Last we add a deleter with the .deleter() method:

>>> prop = prop.deleter(deleter)
>>> prop.fget is getter
True
>>> prop.fset is setter

True
>>> prop.fdel is deleter
True

Last but not least, the property object act as a descriptor object, so it has .__get__(), .__set__() and .__delete__() methods to hook into instance attribute getting, setting and deleting:

>>> class Foo(object): pass
... 
>>> prop.__get__(Foo(), Foo)
Get!
>>> prop.__set__(Foo(), 'bar')
Set to 'bar'!
>>> prop.__delete__(Foo())
Delete!

The Descriptor How to includes a pure python sample implementation of the property() type:

    class Property(object):
        "Emulate PyProperty_Type() in Objects/descrobject.c"
        def __init__(self, fget=None, fset=None, fdel=None, doc=None):
            self.fget = fget
            self.fset = fset
            self.fdel = fdel
            if doc is None and fget is not None:
                doc = fget.__doc__
            self.__doc__ = doc

       def __get__(self, obj, objtype=None):
            if obj is None:
                return self
            if self.fget is None:
                raise AttributeError("unreadable attribute")
            return self.fget(obj)

        def __set__(self, obj, value):
            if self.fset is None:
                raise AttributeError("can't set attribute")
            self.fset(obj, value)

        def __delete__(self, obj):
            if self.fdel is None:
                raise AttributeError("can't delete attribute")
            self.fdel(obj)

        def getter(self, fget):
            return type(self)(fget, self.fset, self.fdel, self.__doc__)

        def setter(self, fset):
            return type(self)(self.fget, fset, self.fdel, self.__doc__)



###if you don't like the property, the similar implement as follows. 

You can do it all in one block: not by using @property by defining and instantiating a class that has `__get__()`,`__set__()`, and `__delete__()` methods.  See Implementing Descriptors for more details:

class Test(object):   
def__init__(self):       
print "Object instance created."       
self._x = raw_input("Initial value of x = ")       
print "Initial value of x set."   

class x(object):       
def __get__(self,instance,owner):           
print 'Getting x'           
return instance._x        

def __set__(self,instance,value):           
print 'Setting x'           
instance._x=value        

def __delete__(self,instance):           
print 'Deleting x'           
del instance._x        
__doc__ = "A test case"   

x = x()

###NOTICE

In order for @properties to work properly the class needs to be a subclass of object. when the class is not a subclass of object then the first time you try access the setter it actually makes a new attribute with the shorter name instead of accessing through the setter.

The following does not work correctly.

    class C(): # <-- Notice that object is missing
    
        def __init__(self):
        self._x = None
    
        @property
        def x(self):
            print 'getting value of x'
            return self._x
    
        @x.setter
        def x(self, x):
            print 'setting value of x'
            self._x = x

    >>> c = C()
    >>> c.x = 1
    >>> print c.x, c._x
    1 0

The following will work correctly

    class C(object):
        def __init__(self):
            self._x = None
    
        @property
        def x(self):
            print 'getting value of x'
            return self._x
    
        @x.setter
                def x(self, x):
            print 'setting value of x'
            self._x = x
    
    >>> c = C()
    >>> c.x = 1
    setting value of x
    >>> print c.x, c._x
    getting value of x
    1 1





###参考文献
1. http://docs.python.org/2.7/library/functions.html?highlight=property#property
2. http://stackoverflow.com/questions/9626192/property-decorators-in-python-and-set-functions?rq=1 
3. http://stackoverflow.com/questions/16513845/when-is-a-property-needed-instead-of-just-exposing-the-variable-in-python?rq=1 
4. http://stackoverflow.com/questions/6618002/python-property-versus-getters-and-setters?rq=1 
5. http://tomayko.com/writings/getters-setters-fuxors 

