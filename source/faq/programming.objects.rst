

Objects
=======
.. contents:: `Contents`
   :depth: 2
   :local:

What is a class?
----------------

A class is the particular object type created by executing a class statement.
Class objects are used as templates to create instance objects, which embody
both the data (attributes) and code (methods) specific to a datatype.

A class can be based on one or more other classes, called its base class(es). It
then inherits the attributes and methods of its base classes. This allows an
object model to be successively refined by inheritance.  You might have a
generic ``Mailbox`` class that provides basic accessor methods for a mailbox,
and subclasses such as ``MboxMailbox``, ``MaildirMailbox``, ``OutlookMailbox``
that handle various specific mailbox formats.


What is a method?
-----------------

A method is a function on some object ``x`` that you normally call as
``x.name(arguments...)``.  Methods are defined as functions inside the class
definition::

   class C:
       def meth(self, arg):
           return arg * 2 + self.attribute


What is self?
-------------

Self is merely a conventional name for the first argument of a method.  A method
defined as ``meth(self, a, b, c)`` should be called as ``x.meth(a, b, c)`` for
some instance ``x`` of the class in which the definition occurs; the called
method will think it is called as ``meth(x, a, b, c)``.

See also :ref:`why-self`.


How do I check if an object is an instance of a given class or of a subclass of it?
-----------------------------------------------------------------------------------

Use the built-in function ``isinstance(obj, cls)``.  You can check if an object
is an instance of any of a number of classes by providing a tuple instead of a
single class, e.g. ``isinstance(obj, (class1, class2, ...))``, and can also
check whether an object is one of Python's built-in types, e.g.
``isinstance(obj, str)`` or ``isinstance(obj, (int, long, float, complex))``.

Note that most programs do not use :func:`isinstance` on user-defined classes
very often.  If you are developing the classes yourself, a more proper
object-oriented style is to define methods on the classes that encapsulate a
particular behaviour, instead of checking the object's class and doing a
different thing based on what class it is.  For example, if you have a function
that does something::

   def search(obj):
       if isinstance(obj, Mailbox):
           ...  # code to search a mailbox
       elif isinstance(obj, Document):
           ...  # code to search a document
       elif ...

A better approach is to define a ``search()`` method on all the classes and just
call it::

   class Mailbox:
       def search(self):
           ...  # code to search a mailbox

   class Document:
       def search(self):
           ...  # code to search a document

   obj.search()


What is delegation?
-------------------

Delegation is an object oriented technique (also called a design pattern).
Let's say you have an object ``x`` and want to change the behaviour of just one
of its methods.  You can create a new class that provides a new implementation
of the method you're interested in changing and delegates all other methods to
the corresponding method of ``x``.

Python programmers can easily implement delegation.  For example, the following
class implements a class that behaves like a file but converts all written data
to uppercase::

   class UpperOut:

       def __init__(self, outfile):
           self._outfile = outfile

       def write(self, s):
           self._outfile.write(s.upper())

       def __getattr__(self, name):
           return getattr(self._outfile, name)

Here the ``UpperOut`` class redefines the ``write()`` method to convert the
argument string to uppercase before calling the underlying
``self.__outfile.write()`` method.  All other methods are delegated to the
underlying ``self.__outfile`` object.  The delegation is accomplished via the
``__getattr__`` method; consult :ref:`the language reference <attribute-access>`
for more information about controlling attribute access.

Note that for more general cases delegation can get trickier. When attributes
must be set as well as retrieved, the class must define a :meth:`__setattr__`
method too, and it must do so carefully.  The basic implementation of
:meth:`__setattr__` is roughly equivalent to the following::

   class X:
       ...
       def __setattr__(self, name, value):
           self.__dict__[name] = value
       ...

Most :meth:`__setattr__` implementations must modify ``self.__dict__`` to store
local state for self without causing an infinite recursion.


How do I call a method defined in a base class from a derived class that overrides it?
--------------------------------------------------------------------------------------

If you're using new-style classes, use the built-in :func:`super` function::

   class Derived(Base):
       def meth(self):
           super(Derived, self).meth()

If you're using classic classes: For a class definition such as ``class
Derived(Base): ...`` you can call method ``meth()`` defined in ``Base`` (or one
of ``Base``'s base classes) as ``Base.meth(self, arguments...)``.  Here,
``Base.meth`` is an unbound method, so you need to provide the ``self``
argument.


How can I organize my code to make it easier to change the base class?
----------------------------------------------------------------------

You could define an alias for the base class, assign the real base class to it
before your class definition, and use the alias throughout your class.  Then all
you have to change is the value assigned to the alias.  Incidentally, this trick
is also handy if you want to decide dynamically (e.g. depending on availability
of resources) which base class to use.  Example::

   BaseAlias = <real base class>

   class Derived(BaseAlias):
       def meth(self):
           BaseAlias.meth(self)
           ...


How do I create static class data and static class methods?
-----------------------------------------------------------

Both static data and static methods (in the sense of C++ or Java) are supported
in Python.

For static data, simply define a class attribute.  To assign a new value to the
attribute, you have to explicitly use the class name in the assignment::

   class C:
       count = 0   # number of times C.__init__ called

       def __init__(self):
           C.count = C.count + 1

       def getcount(self):
           return C.count  # or return self.count

``c.count`` also refers to ``C.count`` for any ``c`` such that ``isinstance(c,
C)`` holds, unless overridden by ``c`` itself or by some class on the base-class
search path from ``c.__class__`` back to ``C``.

Caution: within a method of C, an assignment like ``self.count = 42`` creates a
new and unrelated instance named "count" in ``self``'s own dict.  Rebinding of a
class-static data name must always specify the class whether inside a method or
not::

   C.count = 314

Static methods are possible since Python 2.2::

   class C:
       def static(arg1, arg2, arg3):
           # No 'self' parameter!
           ...
       static = staticmethod(static)

With Python 2.4's decorators, this can also be written as ::

   class C:
       @staticmethod
       def static(arg1, arg2, arg3):
           # No 'self' parameter!
           ...

However, a far more straightforward way to get the effect of a static method is
via a simple module-level function::

   def getcount():
       return C.count

If your code is structured so as to define one class (or tightly related class
hierarchy) per module, this supplies the desired encapsulation.


How can I overload constructors (or methods) in Python?
-------------------------------------------------------

This answer actually applies to all methods, but the question usually comes up
first in the context of constructors.

In C++ you'd write

.. code-block:: c

    class C {
        C() { cout << "No arguments\n"; }
        C(int i) { cout << "Argument is " << i << "\n"; }
    }

In Python you have to write a single constructor that catches all cases using
default arguments.  For example::

   class C:
       def __init__(self, i=None):
           if i is None:
               print "No arguments"
           else:
               print "Argument is", i

This is not entirely equivalent, but close enough in practice.

You could also try a variable-length argument list, e.g. ::

   def __init__(self, *args):
       ...

The same approach works for all method definitions.


I try to use __spam and I get an error about _SomeClassName__spam.
------------------------------------------------------------------

Variable names with double leading underscores are "mangled" to provide a simple
but effective way to define class private variables.  Any identifier of the form
``__spam`` (at least two leading underscores, at most one trailing underscore)
is textually replaced with ``_classname__spam``, where ``classname`` is the
current class name with any leading underscores stripped.

This doesn't guarantee privacy: an outside user can still deliberately access
the "_classname__spam" attribute, and private values are visible in the object's
``__dict__``.  Many Python programmers never bother to use private variable
names at all.


My class defines __del__ but it is not called when I delete the object.
-----------------------------------------------------------------------

There are several possible reasons for this.

The del statement does not necessarily call :meth:`__del__` -- it simply
decrements the object's reference count, and if this reaches zero
:meth:`__del__` is called.

If your data structures contain circular links (e.g. a tree where each child has
a parent reference and each parent has a list of children) the reference counts
will never go back to zero.  Once in a while Python runs an algorithm to detect
such cycles, but the garbage collector might run some time after the last
reference to your data structure vanishes, so your :meth:`__del__` method may be
called at an inconvenient and random time. This is inconvenient if you're trying
to reproduce a problem. Worse, the order in which object's :meth:`__del__`
methods are executed is arbitrary.  You can run :func:`gc.collect` to force a
collection, but there *are* pathological cases where objects will never be
collected.

Despite the cycle collector, it's still a good idea to define an explicit
``close()`` method on objects to be called whenever you're done with them.  The
``close()`` method can then remove attributes that refer to subobjecs.  Don't
call :meth:`__del__` directly -- :meth:`__del__` should call ``close()`` and
``close()`` should make sure that it can be called more than once for the same
object.

Another way to avoid cyclical references is to use the :mod:`weakref` module,
which allows you to point to objects without incrementing their reference count.
Tree data structures, for instance, should use weak references for their parent
and sibling references (if they need them!).

If the object has ever been a local variable in a function that caught an
expression in an except clause, chances are that a reference to the object still
exists in that function's stack frame as contained in the stack trace.
Normally, calling :func:`sys.exc_clear` will take care of this by clearing the
last recorded exception.

Finally, if your :meth:`__del__` method raises an exception, a warning message
is printed to :data:`sys.stderr`.


How do I get a list of all instances of a given class?
------------------------------------------------------

Python does not keep track of all instances of a class (or of a built-in type).
You can program the class's constructor to keep track of all instances by
keeping a list of weak references to each instance.


Why does the result of ``id()`` appear to be not unique?
--------------------------------------------------------

The :func:`id` builtin returns an integer that is guaranteed to be unique during
the lifetime of the object.  Since in CPython, this is the object's memory
address, it happens frequently that after an object is deleted from memory, the
next freshly created object is allocated at the same position in memory.  This
is illustrated by this example:

>>> id(1000)
13901272
>>> id(2000)
13901272

The two ids belong to different integer objects that are created before, and
deleted immediately after execution of the ``id()`` call.  To be sure that
objects whose id you want to examine are still alive, create another reference
to the object:

>>> a = 1000; b = 2000
>>> id(a)
13901272
>>> id(b)
13891296