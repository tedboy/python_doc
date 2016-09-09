
Numbers and strings
===================
.. contents:: `Contents`
   :depth: 2
   :local:


How do I specify hexadecimal and octal integers?
------------------------------------------------

To specify an octal digit, precede the octal value with a zero, and then a lower
or uppercase "o".  For example, to set the variable "a" to the octal value "10"
(8 in decimal), type::

   >>> a = 0o10
   >>> a
   8

Hexadecimal is just as easy.  Simply precede the hexadecimal number with a zero,
and then a lower or uppercase "x".  Hexadecimal digits can be specified in lower
or uppercase.  For example, in the Python interpreter::

   >>> a = 0xa5
   >>> a
   165
   >>> b = 0XB2
   >>> b
   178


Why does -22 // 10 return -3?
-----------------------------

It's primarily driven by the desire that ``i % j`` have the same sign as ``j``.
If you want that, and also want::

    i == (i // j) * j + (i % j)

then integer division has to return the floor.  C also requires that identity to
hold, and then compilers that truncate ``i // j`` need to make ``i % j`` have
the same sign as ``i``.

There are few real use cases for ``i % j`` when ``j`` is negative.  When ``j``
is positive, there are many, and in virtually all of them it's more useful for
``i % j`` to be ``>= 0``.  If the clock says 10 now, what did it say 200 hours
ago?  ``-190 % 12 == 2`` is useful; ``-190 % 12 == -10`` is a bug waiting to
bite.

.. note::

   On Python 2, ``a / b`` returns the same as ``a // b`` if
   ``__future__.division`` is not in effect.  This is also known as "classic"
   division.


How do I convert a string to a number?
--------------------------------------

For integers, use the built-in :func:`int` type constructor, e.g. ``int('144')
== 144``.  Similarly, :func:`float` converts to floating-point,
e.g. ``float('144') == 144.0``.

By default, these interpret the number as decimal, so that ``int('0144') ==
144`` and ``int('0x144')`` raises :exc:`ValueError`. ``int(string, base)`` takes
the base to convert from as a second optional argument, so ``int('0x144', 16) ==
324``.  If the base is specified as 0, the number is interpreted using Python's
rules: a leading '0' indicates octal, and '0x' indicates a hex number.

Do not use the built-in function :func:`eval` if all you need is to convert
strings to numbers.  :func:`eval` will be significantly slower and it presents a
security risk: someone could pass you a Python expression that might have
unwanted side effects.  For example, someone could pass
``__import__('os').system("rm -rf $HOME")`` which would erase your home
directory.

:func:`eval` also has the effect of interpreting numbers as Python expressions,
so that e.g. ``eval('09')`` gives a syntax error because Python regards numbers
starting with '0' as octal (base 8).


How do I convert a number to a string?
--------------------------------------

To convert, e.g., the number 144 to the string '144', use the built-in type
constructor :func:`str`.  If you want a hexadecimal or octal representation, use
the built-in functions :func:`hex` or :func:`oct`.  For fancy formatting, see
the :ref:`formatstrings` section, e.g. ``"{:04d}".format(144)`` yields
``'0144'`` and ``"{:.3f}".format(1/3)`` yields ``'0.333'``.  You may also use
:ref:`the % operator <string-formatting>` on strings.  See the library reference
manual for details.


How do I modify a string in place?
----------------------------------

You can't, because strings are immutable.  If you need an object with this
ability, try converting the string to a list or use the array module::

   >>> import io
   >>> s = "Hello, world"
   >>> a = list(s)
   >>> print a
   ['H', 'e', 'l', 'l', 'o', ',', ' ', 'w', 'o', 'r', 'l', 'd']
   >>> a[7:] = list("there!")
   >>> ''.join(a)
   'Hello, there!'

   >>> import array
   >>> a = array.array('c', s)
   >>> print a
   array('c', 'Hello, world')
   >>> a[0] = 'y'; print a
   array('c', 'yello, world')
   >>> a.tostring()
   'yello, world'


How do I use strings to call functions/methods?
-----------------------------------------------

There are various techniques.

* The best is to use a dictionary that maps strings to functions.  The primary
  advantage of this technique is that the strings do not need to match the names
  of the functions.  This is also the primary technique used to emulate a case
  construct::

     def a():
         pass

     def b():
         pass

     dispatch = {'go': a, 'stop': b}  # Note lack of parens for funcs

     dispatch[get_input()]()  # Note trailing parens to call function

* Use the built-in function :func:`getattr`::

     import foo
     getattr(foo, 'bar')()

  Note that :func:`getattr` works on any object, including classes, class
  instances, modules, and so on.

  This is used in several places in the standard library, like this::

     class Foo:
         def do_foo(self):
             ...

         def do_bar(self):
             ...

     f = getattr(foo_instance, 'do_' + opname)
     f()


* Use :func:`locals` or :func:`eval` to resolve the function name::

     def myFunc():
         print "hello"

     fname = "myFunc"

     f = locals()[fname]
     f()

     f = eval(fname)
     f()

  Note: Using :func:`eval` is slow and dangerous.  If you don't have absolute
  control over the contents of the string, someone could pass a string that
  resulted in an arbitrary function being executed.

Is there an equivalent to Perl's chomp() for removing trailing newlines from strings?
-------------------------------------------------------------------------------------

Starting with Python 2.2, you can use ``S.rstrip("\r\n")`` to remove all
occurrences of any line terminator from the end of the string ``S`` without
removing other trailing whitespace.  If the string ``S`` represents more than
one line, with several empty lines at the end, the line terminators for all the
blank lines will be removed::

   >>> lines = ("line 1 \r\n"
   ...          "\r\n"
   ...          "\r\n")
   >>> lines.rstrip("\n\r")
   'line 1 '

Since this is typically only desired when reading text one line at a time, using
``S.rstrip()`` this way works well.

For older versions of Python, there are two partial substitutes:

- If you want to remove all trailing whitespace, use the ``rstrip()`` method of
  string objects.  This removes all trailing whitespace, not just a single
  newline.

- Otherwise, if there is only one line in the string ``S``, use
  ``S.splitlines()[0]``.


Is there a scanf() or sscanf() equivalent?
------------------------------------------

Not as such.

For simple input parsing, the easiest approach is usually to split the line into
whitespace-delimited words using the :meth:`~str.split` method of string objects
and then convert decimal strings to numeric values using :func:`int` or
:func:`float`.  ``split()`` supports an optional "sep" parameter which is useful
if the line uses something other than whitespace as a separator.

For more complicated input parsing, regular expressions are more powerful
than C's :c:func:`sscanf` and better suited for the task.


What does 'UnicodeError: ASCII [decoding,encoding] error: ordinal not in range(128)' mean?
------------------------------------------------------------------------------------------

This error indicates that your Python installation can handle only 7-bit ASCII
strings.  There are a couple ways to fix or work around the problem.

If your programs must handle data in arbitrary character set encodings, the
environment the application runs in will generally identify the encoding of the
data it is handing you.  You need to convert the input to Unicode data using
that encoding.  For example, a program that handles email or web input will
typically find character set encoding information in Content-Type headers.  This
can then be used to properly convert input data to Unicode. Assuming the string
referred to by ``value`` is encoded as UTF-8::

   value = unicode(value, "utf-8")

will return a Unicode object.  If the data is not correctly encoded as UTF-8,
the above call will raise a :exc:`UnicodeError` exception.

If you only want strings converted to Unicode which have non-ASCII data, you can
try converting them first assuming an ASCII encoding, and then generate Unicode
objects if that fails::

   try:
       x = unicode(value, "ascii")
   except UnicodeError:
       value = unicode(value, "utf-8")
   else:
       # value was valid ASCII data
       pass

It's possible to set a default encoding in a file called ``sitecustomize.py``
that's part of the Python library.  However, this isn't recommended because
changing the Python-wide default encoding may cause third-party extension
modules to fail.

Note that on Windows, there is an encoding known as "mbcs", which uses an
encoding specific to your current locale.  In many cases, and particularly when
working with COM, this may be an appropriate default encoding to use.