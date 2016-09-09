General Questions
=================
.. contents:: `Contents`
   :depth: 2
   :local:

Is there a source code level debugger with breakpoints, single-stepping, etc.?
------------------------------------------------------------------------------

Yes.

The pdb module is a simple but adequate console-mode debugger for Python. It is
part of the standard Python library, and is :mod:`documented in the Library
Reference Manual <pdb>`. You can also write your own debugger by using the code
for pdb as an example.

The IDLE interactive development environment, which is part of the standard
Python distribution (normally available as Tools/scripts/idle), includes a
graphical debugger.

PythonWin is a Python IDE that includes a GUI debugger based on pdb.  The
Pythonwin debugger colors breakpoints and has quite a few cool features such as
debugging non-Pythonwin programs.  Pythonwin is available as part of the `Python
for Windows Extensions <https://sourceforge.net/projects/pywin32/>`__ project and
as a part of the ActivePython distribution (see
https://www.activestate.com/activepython\ ).

`Boa Constructor <http://boa-constructor.sourceforge.net/>`_ is an IDE and GUI
builder that uses wxWidgets.  It offers visual frame creation and manipulation,
an object inspector, many views on the source like object browsers, inheritance
hierarchies, doc string generated html documentation, an advanced debugger,
integrated help, and Zope support.

`Eric <http://eric-ide.python-projects.org/>`_ is an IDE built on PyQt
and the Scintilla editing component.

Pydb is a version of the standard Python debugger pdb, modified for use with DDD
(Data Display Debugger), a popular graphical debugger front end.  Pydb can be
found at http://bashdb.sourceforge.net/pydb/ and DDD can be found at
https://www.gnu.org/software/ddd.

There are a number of commercial Python IDEs that include graphical debuggers.
They include:

* Wing IDE (https://wingware.com/)
* Komodo IDE (https://komodoide.com/)
* PyCharm (https://www.jetbrains.com/pycharm/)


Is there a tool to help find bugs or perform static analysis?
-------------------------------------------------------------

Yes.

PyChecker is a static analysis tool that finds bugs in Python source code and
warns about code complexity and style.  You can get PyChecker from
http://pychecker.sourceforge.net/.

`Pylint <https://www.pylint.org/>`_ is another tool that checks
if a module satisfies a coding standard, and also makes it possible to write
plug-ins to add a custom feature.  In addition to the bug checking that
PyChecker performs, Pylint offers some additional features such as checking line
length, whether variable names are well-formed according to your coding
standard, whether declared interfaces are fully implemented, and more.
https://docs.pylint.org/ provides a full list of Pylint's features.


How can I create a stand-alone binary from a Python script?
-----------------------------------------------------------

You don't need the ability to compile Python to C code if all you want is a
stand-alone program that users can download and run without having to install
the Python distribution first.  There are a number of tools that determine the
set of modules required by a program and bind these modules together with a
Python binary to produce a single executable.

One is to use the freeze tool, which is included in the Python source tree as
``Tools/freeze``. It converts Python byte code to C arrays; a C compiler you can
embed all your modules into a new program, which is then linked with the
standard Python modules.

It works by scanning your source recursively for import statements (in both
forms) and looking for the modules in the standard Python path as well as in the
source directory (for built-in modules).  It then turns the bytecode for modules
written in Python into C code (array initializers that can be turned into code
objects using the marshal module) and creates a custom-made config file that
only contains those built-in modules which are actually used in the program.  It
then compiles the generated C code and links it with the rest of the Python
interpreter to form a self-contained binary which acts exactly like your script.

Obviously, freeze requires a C compiler.  There are several other utilities
which don't. One is Thomas Heller's py2exe (Windows only) at

    http://www.py2exe.org/

Another tool is Anthony Tuininga's `cx_Freeze <http://cx-freeze.sourceforge.net/>`_.


Are there coding standards or a style guide for Python programs?
----------------------------------------------------------------

Yes.  The coding style required for standard library modules is documented as
:pep:`8`.


My program is too slow. How do I speed it up?
---------------------------------------------

That's a tough one, in general.  There are many tricks to speed up Python code;
consider rewriting parts in C as a last resort.

In some cases it's possible to automatically translate Python to C or x86
assembly language, meaning that you don't have to modify your code to gain
increased speed.

.. XXX seems to have overlap with other questions!

`Pyrex <http://www.cosc.canterbury.ac.nz/~greg/python/Pyrex/>`_ can compile a
slightly modified version of Python code into a C extension, and can be used on
many different platforms.

`Psyco <http://psyco.sourceforge.net>`_ is a just-in-time compiler that
translates Python code into x86 assembly language.  If you can use it, Psyco can
provide dramatic speedups for critical functions.

The rest of this answer will discuss various tricks for squeezing a bit more
speed out of Python code.  *Never* apply any optimization tricks unless you know
you need them, after profiling has indicated that a particular function is the
heavily executed hot spot in the code.  Optimizations almost always make the
code less clear, and you shouldn't pay the costs of reduced clarity (increased
development time, greater likelihood of bugs) unless the resulting performance
benefit is worth it.

There is a page on the wiki devoted to `performance tips
<https://wiki.python.org/moin/PythonSpeed/PerformanceTips>`_.

Guido van Rossum has written up an anecdote related to optimization at
https://www.python.org/doc/essays/list2str.

One thing to notice is that function and (especially) method calls are rather
expensive; if you have designed a purely OO interface with lots of tiny
functions that don't do much more than get or set an instance variable or call
another method, you might consider using a more direct way such as directly
accessing instance variables.  Also see the standard module :mod:`profile` which
makes it possible to find out where your program is spending most of its time
(if you have some patience -- the profiling itself can slow your program down by
an order of magnitude).

Remember that many standard optimization heuristics you may know from other
programming experience may well apply to Python.  For example it may be faster
to send output to output devices using larger writes rather than smaller ones in
order to reduce the overhead of kernel system calls.  Thus CGI scripts that
write all output in "one shot" may be faster than those that write lots of small
pieces of output.

Also, be sure to use Python's core features where appropriate.  For example,
slicing allows programs to chop up lists and other sequence objects in a single
tick of the interpreter's mainloop using highly optimized C implementations.
Thus to get the same effect as::

   L2 = []
   for i in range(3):
       L2.append(L1[i])

it is much shorter and far faster to use ::

   L2 = list(L1[:3])  # "list" is redundant if L1 is a list.

Note that the functionally-oriented built-in functions such as :func:`map`,
:func:`zip`, and friends can be a convenient accelerator for loops that
perform a single task.  For example to pair the elements of two lists
together::

   >>> zip([1, 2, 3], [4, 5, 6])
   [(1, 4), (2, 5), (3, 6)]

or to compute a number of sines::

   >>> map(math.sin, (1, 2, 3, 4))
   [0.841470984808, 0.909297426826, 0.14112000806, -0.756802495308]

The operation completes very quickly in such cases.

Other examples include the ``join()`` and ``split()`` :ref:`methods
of string objects <string-methods>`.
For example if s1..s7 are large (10K+) strings then
``"".join([s1,s2,s3,s4,s5,s6,s7])`` may be far faster than the more obvious
``s1+s2+s3+s4+s5+s6+s7``, since the "summation" will compute many
subexpressions, whereas ``join()`` does all the copying in one pass.  For
manipulating strings, use the ``replace()`` and the ``format()`` :ref:`methods
on string objects <string-methods>`.  Use regular expressions only when you're
not dealing with constant string patterns.  You may still use :ref:`the old %
operations <string-formatting>` ``string % tuple`` and ``string % dictionary``.

Be sure to use the :meth:`list.sort` built-in method to do sorting, and see the
`sorting mini-HOWTO <https://wiki.python.org/moin/HowTo/Sorting>`_ for examples
of moderately advanced usage.  :meth:`list.sort` beats other techniques for
sorting in all but the most extreme circumstances.

Another common trick is to "push loops into functions or methods."  For example
suppose you have a program that runs slowly and you use the profiler to
determine that a Python function ``ff()`` is being called lots of times.  If you
notice that ``ff()``::

   def ff(x):
       ... # do something with x computing result...
       return result

tends to be called in loops like::

   list = map(ff, oldlist)

or::

   for x in sequence:
       value = ff(x)
       ... # do something with value...

then you can often eliminate function call overhead by rewriting ``ff()`` to::

   def ffseq(seq):
       resultseq = []
       for x in seq:
           ... # do something with x computing result...
           resultseq.append(result)
       return resultseq

and rewrite the two examples to ``list = ffseq(oldlist)`` and to::

   for value in ffseq(sequence):
       ... # do something with value...

Single calls to ``ff(x)`` translate to ``ffseq([x])[0]`` with little penalty.
Of course this technique is not always appropriate and there are other variants
which you can figure out.

You can gain some performance by explicitly storing the results of a function or
method lookup into a local variable.  A loop like::

   for key in token:
       dict[key] = dict.get(key, 0) + 1

resolves ``dict.get`` every iteration.  If the method isn't going to change, a
slightly faster implementation is::

   dict_get = dict.get  # look up the method once
   for key in token:
       dict[key] = dict_get(key, 0) + 1

Default arguments can be used to determine values once, at compile time instead
of at run time.  This can only be done for functions or objects which will not
be changed during program execution, such as replacing ::

   def degree_sin(deg):
       return math.sin(deg * math.pi / 180.0)

with ::

   def degree_sin(deg, factor=math.pi/180.0, sin=math.sin):
       return sin(deg * factor)

Because this trick uses default arguments for terms which should not be changed,
it should only be used when you are not concerned with presenting a possibly
confusing API to your users.