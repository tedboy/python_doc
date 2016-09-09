


Modules
=======
.. contents:: `Contents`
   :depth: 2
   :local:


How do I create a .pyc file?
----------------------------

When a module is imported for the first time (or when the source is more recent
than the current compiled file) a ``.pyc`` file containing the compiled code
should be created in the same directory as the ``.py`` file.

One reason that a ``.pyc`` file may not be created is permissions problems with
the directory. This can happen, for example, if you develop as one user but run
as another, such as if you are testing with a web server.  Creation of a .pyc
file is automatic if you're importing a module and Python has the ability
(permissions, free space, etc...) to write the compiled module back to the
directory.

Running Python on a top level script is not considered an import and no
``.pyc`` will be created.  For example, if you have a top-level module
``foo.py`` that imports another module ``xyz.py``, when you run ``foo``,
``xyz.pyc`` will be created since ``xyz`` is imported, but no ``foo.pyc`` file
will be created since ``foo.py`` isn't being imported.

If you need to create ``foo.pyc`` -- that is, to create a ``.pyc`` file for a module
that is not imported -- you can, using the :mod:`py_compile` and
:mod:`compileall` modules.

The :mod:`py_compile` module can manually compile any module.  One way is to use
the ``compile()`` function in that module interactively::

   >>> import py_compile
   >>> py_compile.compile('foo.py')                 # doctest: +SKIP

This will write the ``.pyc`` to the same location as ``foo.py`` (or you can
override that with the optional parameter ``cfile``).

You can also automatically compile all files in a directory or directories using
the :mod:`compileall` module.  You can do it from the shell prompt by running
``compileall.py`` and providing the path of a directory containing Python files
to compile::

       python -m compileall .


How do I find the current module name?
--------------------------------------

A module can find out its own module name by looking at the predefined global
variable ``__name__``.  If this has the value ``'__main__'``, the program is
running as a script.  Many modules that are usually used by importing them also
provide a command-line interface or a self-test, and only execute this code
after checking ``__name__``::

   def main():
       print 'Running test...'
       ...

   if __name__ == '__main__':
       main()


How can I have modules that mutually import each other?
-------------------------------------------------------

Suppose you have the following modules:

foo.py::

   from bar import bar_var
   foo_var = 1

bar.py::

   from foo import foo_var
   bar_var = 2

The problem is that the interpreter will perform the following steps:

* main imports foo
* Empty globals for foo are created
* foo is compiled and starts executing
* foo imports bar
* Empty globals for bar are created
* bar is compiled and starts executing
* bar imports foo (which is a no-op since there already is a module named foo)
* bar.foo_var = foo.foo_var

The last step fails, because Python isn't done with interpreting ``foo`` yet and
the global symbol dictionary for ``foo`` is still empty.

The same thing happens when you use ``import foo``, and then try to access
``foo.foo_var`` in global code.

There are (at least) three possible workarounds for this problem.

Guido van Rossum recommends avoiding all uses of ``from <module> import ...``,
and placing all code inside functions.  Initializations of global variables and
class variables should use constants or built-in functions only.  This means
everything from an imported module is referenced as ``<module>.<name>``.

Jim Roskind suggests performing steps in the following order in each module:

* exports (globals, functions, and classes that don't need imported base
  classes)
* ``import`` statements
* active code (including globals that are initialized from imported values).

van Rossum doesn't like this approach much because the imports appear in a
strange place, but it does work.

Matthias Urlichs recommends restructuring your code so that the recursive import
is not necessary in the first place.

These solutions are not mutually exclusive.


__import__('x.y.z') returns <module 'x'>; how do I get z?
---------------------------------------------------------

Consider using the convenience function :func:`~importlib.import_module` from
:mod:`importlib` instead::

   z = importlib.import_module('x.y.z')


When I edit an imported module and reimport it, the changes don't show up.  Why does this happen?
-------------------------------------------------------------------------------------------------

For reasons of efficiency as well as consistency, Python only reads the module
file on the first time a module is imported.  If it didn't, in a program
consisting of many modules where each one imports the same basic module, the
basic module would be parsed and re-parsed many times.  To force rereading of a
changed module, do this::

   import modname
   reload(modname)

Warning: this technique is not 100% fool-proof.  In particular, modules
containing statements like ::

   from modname import some_objects

will continue to work with the old version of the imported objects.  If the
module contains class definitions, existing class instances will *not* be
updated to use the new class definition.  This can result in the following
paradoxical behaviour:

   >>> import cls
   >>> c = cls.C()                # Create an instance of C
   >>> reload(cls)
   <module 'cls' from 'cls.pyc'>
   >>> isinstance(c, cls.C)       # isinstance is false?!?
   False

The nature of the problem is made clear if you print out the class objects:

   >>> c.__class__
   <class cls.C at 0x7352a0>
   >>> cls.C
   <class cls.C at 0x4198d0>

