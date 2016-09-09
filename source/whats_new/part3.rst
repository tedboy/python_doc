Python 3.1 Features
=======================

Much as Python 2.6 incorporated features from Python 3.0,
version 2.7 incorporates some of the new features
in Python 3.1.  The 2.x series continues to provide tools
for migrating to the 3.x series.

A partial list of 3.1 features that were backported to 2.7:

* The syntax for set literals (``{1,2,3}`` is a mutable set).
* Dictionary and set comprehensions (``{i: i*2 for i in range(3)}``).
* Multiple context managers in a single :keyword:`with` statement.
* A new version of the :mod:`io` library, rewritten in C for performance.
* The ordered-dictionary type described in :ref:`pep-0372`.
* The new ``","`` format specifier described in :ref:`pep-0378`.
* The :class:`memoryview` object.
* A small subset of the :mod:`importlib` module,
  `described below <#importlib-section>`__.
* The :func:`repr` of a float ``x`` is shorter in many cases: it's now
  based on the shortest decimal string that's guaranteed to round back
  to ``x``.  As in previous versions of Python, it's guaranteed that
  ``float(repr(x))`` recovers ``x``.
* Float-to-string and string-to-float conversions are correctly rounded.
  The :func:`round` function is also now correctly rounded.
* The :c:type:`PyCapsule` type, used to provide a C API for extension modules.
* The :c:func:`PyLong_AsLongAndOverflow` C API function.

Other new Python3-mode warnings include:

* :func:`operator.isCallable` and :func:`operator.sequenceIncludes`,
  which are not supported in 3.x, now trigger warnings.
* The :option:`-3` switch now automatically
  enables the :option:`-Qwarn <-Q>` switch that causes warnings
  about using classic division with integers and long integers.