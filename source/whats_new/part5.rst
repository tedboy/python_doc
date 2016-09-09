.. _pep-0378:

PEP 378: Format Specifier for Thousands Separator
=================================================

To make program output more readable, it can be useful to add
separators to large numbers, rendering them as
18,446,744,073,709,551,616 instead of 18446744073709551616.

The fully general solution for doing this is the :mod:`locale` module,
which can use different separators ("," in North America, "." in
Europe) and different grouping sizes, but :mod:`locale` is complicated
to use and unsuitable for multi-threaded applications where different
threads are producing output for different locales.

Therefore, a simple comma-grouping mechanism has been added to the
mini-language used by the :meth:`str.format` method.  When
formatting a floating-point number, simply include a comma between the
width and the precision::

   >>> '{:20,.2f}'.format(18446744073709551616.0)
   '18,446,744,073,709,551,616.00'

When formatting an integer, include the comma after the width:

   >>> '{:20,d}'.format(18446744073709551616)
   '18,446,744,073,709,551,616'

This mechanism is not adaptable at all; commas are always used as the
separator and the grouping is always into three-digit groups.  The
comma-formatting mechanism isn't as general as the :mod:`locale`
module, but it's easier to use.

.. seealso::

   :pep:`378` - Format Specifier for Thousands Separator
     PEP written by Raymond Hettinger; implemented by Eric Smith.