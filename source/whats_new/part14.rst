
.. ======================================================================

Porting to Python 2.7
=====================

This section lists previously described changes and other bugfixes
that may require changes to your code:

* The :func:`range` function processes its arguments more
  consistently; it will now call :meth:`__int__` on non-float,
  non-integer arguments that are supplied to it.  (Fixed by Alexander
  Belopolsky; :issue:`1533`.)

* The string :meth:`format` method changed the default precision used
  for floating-point and complex numbers from 6 decimal
  places to 12, which matches the precision used by :func:`str`.
  (Changed by Eric Smith; :issue:`5920`.)

* Because of an optimization for the :keyword:`with` statement, the special
  methods :meth:`__enter__` and :meth:`__exit__` must belong to the object's
  type, and cannot be directly attached to the object's instance.  This
  affects new-style classes (derived from :class:`object`) and C extension
  types.  (:issue:`6101`.)

* Due to a bug in Python 2.6, the *exc_value* parameter to
  :meth:`__exit__` methods was often the string representation of the
  exception, not an instance.  This was fixed in 2.7, so *exc_value*
  will be an instance as expected.  (Fixed by Florent Xicluna;
  :issue:`7853`.)

* When a restricted set of attributes were set using ``__slots__``,
  deleting an unset attribute would not raise :exc:`AttributeError`
  as you would expect.  Fixed by Benjamin Peterson; :issue:`7604`.)

In the standard library:

* Operations with :class:`~datetime.datetime` instances that resulted in a year
  falling outside the supported range didn't always raise
  :exc:`OverflowError`.  Such errors are now checked more carefully
  and will now raise the exception. (Reported by Mark Leander, patch
  by Anand B. Pillai and Alexander Belopolsky; :issue:`7150`.)

* When using :class:`~decimal.Decimal` instances with a string's
  :meth:`format` method, the default alignment was previously
  left-alignment.  This has been changed to right-alignment, which might
  change the output of your programs.
  (Changed by Mark Dickinson; :issue:`6857`.)

  Comparisons involving a signaling NaN value (or ``sNAN``) now signal
  :const:`~decimal.InvalidOperation` instead of silently returning a true or
  false value depending on the comparison operator.  Quiet NaN values
  (or ``NaN``) are now hashable.  (Fixed by Mark Dickinson;
  :issue:`7279`.)

* The ElementTree library, :mod:`xml.etree`, no longer escapes
  ampersands and angle brackets when outputting an XML processing
  instruction (which looks like `<?xml-stylesheet href="#style1"?>`)
  or comment (which looks like `<!-- comment -->`).
  (Patch by Neil Muller; :issue:`2746`.)

* The :meth:`~StringIO.StringIO.readline` method of :class:`~StringIO.StringIO` objects now does
  nothing when a negative length is requested, as other file-like
  objects do.  (:issue:`7348`).

* The :mod:`syslog` module will now use the value of ``sys.argv[0]`` as the
  identifier instead of the previous default value of ``'python'``.
  (Changed by Sean Reifschneider; :issue:`8451`.)

* The :mod:`tarfile` module's default error handling has changed, to
  no longer suppress fatal errors.  The default error level was previously 0,
  which meant that errors would only result in a message being written to the
  debug log, but because the debug log is not activated by default,
  these errors go unnoticed.  The default error level is now 1,
  which raises an exception if there's an error.
  (Changed by Lars Gustäbel; :issue:`7357`.)

* The :mod:`urlparse` module's :func:`~urlparse.urlsplit` now handles
  unknown URL schemes in a fashion compliant with :rfc:`3986`: if the
  URL is of the form ``"<something>://..."``, the text before the
  ``://`` is treated as the scheme, even if it's a made-up scheme that
  the module doesn't know about.  This change may break code that
  worked around the old behaviour.  For example, Python 2.6.4 or 2.5
  will return the following:

    >>> import urlparse
    >>> urlparse.urlsplit('invented://host/filename?query')
    ('invented', '', '//host/filename?query', '', '')

  Python 2.7 (and Python 2.6.5) will return:

    >>> import urlparse
    >>> urlparse.urlsplit('invented://host/filename?query')
    ('invented', 'host', '/filename?query', '', '')

  (Python 2.7 actually produces slightly different output, since it
  returns a named tuple instead of a standard tuple.)

For C extensions:

* C extensions that use integer format codes with the ``PyArg_Parse*``
  family of functions will now raise a :exc:`TypeError` exception
  instead of triggering a :exc:`DeprecationWarning` (:issue:`5080`).

* Use the new :c:func:`PyOS_string_to_double` function instead of the old
  :c:func:`PyOS_ascii_strtod` and :c:func:`PyOS_ascii_atof` functions,
  which are now deprecated.

For applications that embed Python:

* The :c:func:`PySys_SetArgvEx` function was added, letting
  applications close a security hole when the existing
  :c:func:`PySys_SetArgv` function was used.  Check whether you're
  calling :c:func:`PySys_SetArgv` and carefully consider whether the
  application should be using :c:func:`PySys_SetArgvEx` with
  *updatepath* set to false.