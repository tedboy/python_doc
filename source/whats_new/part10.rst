
Other Language Changes
======================

Some smaller changes made to the core Python language are:

* The syntax for set literals has been backported from Python 3.x.
  Curly brackets are used to surround the contents of the resulting
  mutable set; set literals are
  distinguished from dictionaries by not containing colons and values.
  ``{}`` continues to represent an empty dictionary; use
  ``set()`` for an empty set.

    >>> {1, 2, 3, 4, 5}
    set([1, 2, 3, 4, 5])
    >>> set() # empty set
    set([])
    >>> {}    # empty dict
    {}

  Backported by Alexandre Vassalotti; :issue:`2335`.

* Dictionary and set comprehensions are another feature backported from
  3.x, generalizing list/generator comprehensions to use
  the literal syntax for sets and dictionaries.

    >>> {x: x*x for x in range(6)}
    {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
    >>> {('a'*x) for x in range(6)}
    set(['', 'a', 'aa', 'aaa', 'aaaa', 'aaaaa'])

  Backported by Alexandre Vassalotti; :issue:`2333`.

* The :keyword:`with` statement can now use multiple context managers
  in one statement.  Context managers are processed from left to right
  and each one is treated as beginning a new :keyword:`with` statement.
  This means that::

   with A() as a, B() as b:
       ... suite of statements ...

  is equivalent to::

   with A() as a:
       with B() as b:
           ... suite of statements ...

  The :func:`contextlib.nested` function provides a very similar
  function, so it's no longer necessary and has been deprecated.

  (Proposed in https://codereview.appspot.com/53094; implemented by
  Georg Brandl.)

* Conversions between floating-point numbers and strings are
  now correctly rounded on most platforms.  These conversions occur
  in many different places: :func:`str` on
  floats and complex numbers; the :class:`float` and :class:`complex`
  constructors;
  numeric formatting; serializing and
  deserializing floats and complex numbers using the
  :mod:`marshal`, :mod:`pickle`
  and :mod:`json` modules;
  parsing of float and imaginary literals in Python code;
  and :class:`~decimal.Decimal`-to-float conversion.

  Related to this, the :func:`repr` of a floating-point number *x*
  now returns a result based on the shortest decimal string that's
  guaranteed to round back to *x* under correct rounding (with
  round-half-to-even rounding mode).  Previously it gave a string
  based on rounding x to 17 decimal digits.

  .. maybe add an example?

  The rounding library responsible for this improvement works on
  Windows and on Unix platforms using the gcc, icc, or suncc
  compilers.  There may be a small number of platforms where correct
  operation of this code cannot be guaranteed, so the code is not
  used on such systems.  You can find out which code is being used
  by checking :data:`sys.float_repr_style`,  which will be ``short``
  if the new code is in use and ``legacy`` if it isn't.

  Implemented by Eric Smith and Mark Dickinson, using David Gay's
  :file:`dtoa.c` library; :issue:`7117`.

* Conversions from long integers and regular integers to floating
  point now round differently, returning the floating-point number
  closest to the number.  This doesn't matter for small integers that
  can be converted exactly, but for large numbers that will
  unavoidably lose precision, Python 2.7 now approximates more
  closely.  For example, Python 2.6 computed the following::

    >>> n = 295147905179352891391
    >>> float(n)
    2.9514790517935283e+20
    >>> n - long(float(n))
    65535L

  Python 2.7's floating-point result is larger, but much closer to the
  true value::

    >>> n = 295147905179352891391
    >>> float(n)
    2.9514790517935289e+20
    >>> n - long(float(n))
    -1L

  (Implemented by Mark Dickinson; :issue:`3166`.)

  Integer division is also more accurate in its rounding behaviours.  (Also
  implemented by Mark Dickinson; :issue:`1811`.)

* Implicit coercion for complex numbers has been removed; the interpreter
  will no longer ever attempt to call a :meth:`__coerce__` method on complex
  objects.  (Removed by Meador Inge and Mark Dickinson; :issue:`5211`.)

* The :meth:`str.format` method now supports automatic numbering of the replacement
  fields.  This makes using :meth:`str.format` more closely resemble using
  ``%s`` formatting::

    >>> '{}:{}:{}'.format(2009, 04, 'Sunday')
    '2009:4:Sunday'
    >>> '{}:{}:{day}'.format(2009, 4, day='Sunday')
    '2009:4:Sunday'

  The auto-numbering takes the fields from left to right, so the first ``{...}``
  specifier will use the first argument to :meth:`str.format`, the next
  specifier will use the next argument, and so on.  You can't mix auto-numbering
  and explicit numbering -- either number all of your specifier fields or none
  of them -- but you can mix auto-numbering and named fields, as in the second
  example above.  (Contributed by Eric Smith; :issue:`5237`.)

  Complex numbers now correctly support usage with :func:`format`,
  and default to being right-aligned.
  Specifying a precision or comma-separation applies to both the real
  and imaginary parts of the number, but a specified field width and
  alignment is applied to the whole of the resulting ``1.5+3j``
  output.  (Contributed by Eric Smith; :issue:`1588` and :issue:`7988`.)

  The 'F' format code now always formats its output using uppercase characters,
  so it will now produce 'INF' and 'NAN'.
  (Contributed by Eric Smith; :issue:`3382`.)

  A low-level change: the :meth:`object.__format__` method now triggers
  a :exc:`PendingDeprecationWarning` if it's passed a format string,
  because the :meth:`__format__` method for :class:`object` converts
  the object to a string representation and formats that.  Previously
  the method silently applied the format string to the string
  representation, but that could hide mistakes in Python code.  If
  you're supplying formatting information such as an alignment or
  precision, presumably you're expecting the formatting to be applied
  in some object-specific way.  (Fixed by Eric Smith; :issue:`7994`.)

* The :func:`int` and :func:`long` types gained a ``bit_length``
  method that returns the number of bits necessary to represent
  its argument in binary::

      >>> n = 37
      >>> bin(n)
      '0b100101'
      >>> n.bit_length()
      6
      >>> n = 2**123-1
      >>> n.bit_length()
      123
      >>> (n+1).bit_length()
      124

  (Contributed by Fredrik Johansson and Victor Stinner; :issue:`3439`.)

* The :keyword:`import` statement will no longer try an absolute import
  if a relative import (e.g. ``from .os import sep``) fails.  This
  fixes a bug, but could possibly break certain :keyword:`import`
  statements that were only working by accident.  (Fixed by Meador Inge;
  :issue:`7902`.)

* It's now possible for a subclass of the built-in :class:`unicode` type
  to override the :meth:`__unicode__` method.  (Implemented by
  Victor Stinner; :issue:`1583863`.)

* The :class:`bytearray` type's :meth:`~bytearray.translate` method now accepts
  ``None`` as its first argument.  (Fixed by Georg Brandl;
  :issue:`4759`.)

  .. XXX bytearray doesn't seem to be documented

* When using ``@classmethod`` and ``@staticmethod`` to wrap
  methods as class or static methods, the wrapper object now
  exposes the wrapped function as their :attr:`__func__` attribute.
  (Contributed by Amaury Forgeot d'Arc, after a suggestion by
  George Sakkis; :issue:`5982`.)

* When a restricted set of attributes were set using ``__slots__``,
  deleting an unset attribute would not raise :exc:`AttributeError`
  as you would expect.  Fixed by Benjamin Peterson; :issue:`7604`.)

* Two new encodings are now supported: "cp720", used primarily for
  Arabic text; and "cp858", a variant of CP 850 that adds the euro
  symbol.  (CP720 contributed by Alexander Belchenko and Amaury
  Forgeot d'Arc in :issue:`1616979`; CP858 contributed by Tim Hatch in
  :issue:`8016`.)

* The :class:`file` object will now set the :attr:`filename` attribute
  on the :exc:`IOError` exception when trying to open a directory
  on POSIX platforms (noted by Jan Kaliszewski; :issue:`4764`), and
  now explicitly checks for and forbids writing to read-only file objects
  instead of trusting the C library to catch and report the error
  (fixed by Stefan Krah; :issue:`5677`).

* The Python tokenizer now translates line endings itself, so the
  :func:`compile` built-in function now accepts code using any
  line-ending convention.  Additionally, it no longer requires that the
  code end in a newline.

* Extra parentheses in function definitions are illegal in Python 3.x,
  meaning that you get a syntax error from ``def f((x)): pass``.  In
  Python3-warning mode, Python 2.7 will now warn about this odd usage.
  (Noted by James Lingard; :issue:`7362`.)

* It's now possible to create weak references to old-style class
  objects.  New-style classes were always weak-referenceable.  (Fixed
  by Antoine Pitrou; :issue:`8268`.)

* When a module object is garbage-collected, the module's dictionary is
  now only cleared if no one else is holding a reference to the
  dictionary (:issue:`7140`).


.. ======================================================================

.. _new-27-interpreter:

Interpreter Changes
-------------------------------

A new environment variable, :envvar:`PYTHONWARNINGS`,
allows controlling warnings.  It should be set to a string
containing warning settings, equivalent to those
used with the :option:`-W` switch, separated by commas.
(Contributed by Brian Curtin; :issue:`7301`.)

For example, the following setting will print warnings every time
they occur, but turn warnings from the :mod:`Cookie` module into an
error.  (The exact syntax for setting an environment variable varies
across operating systems and shells.)

::

  export PYTHONWARNINGS=all,error:::Cookie:0

.. ======================================================================


Optimizations
-------------

Several performance enhancements have been added:

* A new opcode was added to perform the initial setup for
  :keyword:`with` statements, looking up the :meth:`__enter__` and
  :meth:`__exit__` methods.  (Contributed by Benjamin Peterson.)

* The garbage collector now performs better for one common usage
  pattern: when many objects are being allocated without deallocating
  any of them.  This would previously take quadratic
  time for garbage collection, but now the number of full garbage collections
  is reduced as the number of objects on the heap grows.
  The new logic only performs a full garbage collection pass when
  the middle generation has been collected 10 times and when the
  number of survivor objects from the middle generation exceeds 10% of
  the number of objects in the oldest generation.  (Suggested by Martin
  von Löwis and implemented by Antoine Pitrou; :issue:`4074`.)

* The garbage collector tries to avoid tracking simple containers
  which can't be part of a cycle. In Python 2.7, this is now true for
  tuples and dicts containing atomic types (such as ints, strings,
  etc.). Transitively, a dict containing tuples of atomic types won't
  be tracked either. This helps reduce the cost of each
  garbage collection by decreasing the number of objects to be
  considered and traversed by the collector.
  (Contributed by Antoine Pitrou; :issue:`4688`.)

* Long integers are now stored internally either in base 2**15 or in base
  2**30, the base being determined at build time.  Previously, they
  were always stored in base 2**15.  Using base 2**30 gives
  significant performance improvements on 64-bit machines, but
  benchmark results on 32-bit machines have been mixed.  Therefore,
  the default is to use base 2**30 on 64-bit machines and base 2**15
  on 32-bit machines; on Unix, there's a new configure option
  :option:`--enable-big-digits` that can be used to override this default.

  Apart from the performance improvements this change should be
  invisible to end users, with one exception: for testing and
  debugging purposes there's a new structseq :data:`sys.long_info` that
  provides information about the internal format, giving the number of
  bits per digit and the size in bytes of the C type used to store
  each digit::

     >>> import sys
     >>> sys.long_info
     sys.long_info(bits_per_digit=30, sizeof_digit=4)

  (Contributed by Mark Dickinson; :issue:`4258`.)

  Another set of changes made long objects a few bytes smaller: 2 bytes
  smaller on 32-bit systems and 6 bytes on 64-bit.
  (Contributed by Mark Dickinson; :issue:`5260`.)

* The division algorithm for long integers has been made faster
  by tightening the inner loop, doing shifts instead of multiplications,
  and fixing an unnecessary extra iteration.
  Various benchmarks show speedups of between 50% and 150% for long
  integer divisions and modulo operations.
  (Contributed by Mark Dickinson; :issue:`5512`.)
  Bitwise operations are also significantly faster (initial patch by
  Gregory Smith; :issue:`1087418`).

* The implementation of ``%`` checks for the left-side operand being
  a Python string and special-cases it; this results in a 1-3%
  performance increase for applications that frequently use ``%``
  with strings, such as templating libraries.
  (Implemented by Collin Winter; :issue:`5176`.)

* List comprehensions with an ``if`` condition are compiled into
  faster bytecode.  (Patch by Antoine Pitrou, back-ported to 2.7
  by Jeffrey Yasskin; :issue:`4715`.)

* Converting an integer or long integer to a decimal string was made
  faster by special-casing base 10 instead of using a generalized
  conversion function that supports arbitrary bases.
  (Patch by Gawain Bolton; :issue:`6713`.)

* The :meth:`split`, :meth:`replace`, :meth:`rindex`,
  :meth:`rpartition`, and :meth:`rsplit` methods of string-like types
  (strings, Unicode strings, and :class:`bytearray` objects) now use a
  fast reverse-search algorithm instead of a character-by-character
  scan.  This is sometimes faster by a factor of 10.  (Added by
  Florent Xicluna; :issue:`7462` and :issue:`7622`.)

* The :mod:`pickle` and :mod:`cPickle` modules now automatically
  intern the strings used for attribute names, reducing memory usage
  of the objects resulting from unpickling.  (Contributed by Jake
  McGuire; :issue:`5084`.)

* The :mod:`cPickle` module now special-cases dictionaries,
  nearly halving the time required to pickle them.
  (Contributed by Collin Winter; :issue:`5670`.)