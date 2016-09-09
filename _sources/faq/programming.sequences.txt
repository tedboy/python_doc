


Sequences (Tuples/Lists)
========================
.. contents:: `Contents`
   :depth: 2
   :local:


How do I convert between tuples and lists?
------------------------------------------

The type constructor ``tuple(seq)`` converts any sequence (actually, any
iterable) into a tuple with the same items in the same order.

For example, ``tuple([1, 2, 3])`` yields ``(1, 2, 3)`` and ``tuple('abc')``
yields ``('a', 'b', 'c')``.  If the argument is a tuple, it does not make a copy
but returns the same object, so it is cheap to call :func:`tuple` when you
aren't sure that an object is already a tuple.

The type constructor ``list(seq)`` converts any sequence or iterable into a list
with the same items in the same order.  For example, ``list((1, 2, 3))`` yields
``[1, 2, 3]`` and ``list('abc')`` yields ``['a', 'b', 'c']``.  If the argument
is a list, it makes a copy just like ``seq[:]`` would.


What's a negative index?
------------------------

Python sequences are indexed with positive numbers and negative numbers.  For
positive numbers 0 is the first index 1 is the second index and so forth.  For
negative indices -1 is the last index and -2 is the penultimate (next to last)
index and so forth.  Think of ``seq[-n]`` as the same as ``seq[len(seq)-n]``.

Using negative indices can be very convenient.  For example ``S[:-1]`` is all of
the string except for its last character, which is useful for removing the
trailing newline from a string.


How do I iterate over a sequence in reverse order?
--------------------------------------------------

Use the :func:`reversed` built-in function, which is new in Python 2.4::

   for x in reversed(sequence):
       ...  # do something with x ...

This won't touch your original sequence, but build a new copy with reversed
order to iterate over.

With Python 2.3, you can use an extended slice syntax::

   for x in sequence[::-1]:
       ...  # do something with x ...


How do you remove duplicates from a list?
-----------------------------------------

See the Python Cookbook for a long discussion of many ways to do this:

   https://code.activestate.com/recipes/52560/

If you don't mind reordering the list, sort it and then scan from the end of the
list, deleting duplicates as you go::

   if mylist:
       mylist.sort()
       last = mylist[-1]
       for i in range(len(mylist)-2, -1, -1):
           if last == mylist[i]:
               del mylist[i]
           else:
               last = mylist[i]

If all elements of the list may be used as dictionary keys (i.e. they are all
hashable) this is often faster ::

   d = {}
   for x in mylist:
       d[x] = 1
   mylist = list(d.keys())

In Python 2.5 and later, the following is possible instead::

   mylist = list(set(mylist))

This converts the list into a set, thereby removing duplicates, and then back
into a list.


How do you make an array in Python?
-----------------------------------

Use a list::

   ["this", 1, "is", "an", "array"]

Lists are equivalent to C or Pascal arrays in their time complexity; the primary
difference is that a Python list can contain objects of many different types.

The ``array`` module also provides methods for creating arrays of fixed types
with compact representations, but they are slower to index than lists.  Also
note that the Numeric extensions and others define array-like structures with
various characteristics as well.

To get Lisp-style linked lists, you can emulate cons cells using tuples::

   lisp_list = ("like",  ("this",  ("example", None) ) )

If mutability is desired, you could use lists instead of tuples.  Here the
analogue of lisp car is ``lisp_list[0]`` and the analogue of cdr is
``lisp_list[1]``.  Only do this if you're sure you really need to, because it's
usually a lot slower than using Python lists.


.. _faq-multidimensional-list:

How do I create a multidimensional list?
----------------------------------------

You probably tried to make a multidimensional array like this::

   >>> A = [[None] * 2] * 3

This looks correct if you print it::

   >>> A
   [[None, None], [None, None], [None, None]]

But when you assign a value, it shows up in multiple places:

  >>> A[0][0] = 5
  >>> A
  [[5, None], [5, None], [5, None]]

The reason is that replicating a list with ``*`` doesn't create copies, it only
creates references to the existing objects.  The ``*3`` creates a list
containing 3 references to the same list of length two.  Changes to one row will
show in all rows, which is almost certainly not what you want.

The suggested approach is to create a list of the desired length first and then
fill in each element with a newly created list::

   A = [None] * 3
   for i in range(3):
       A[i] = [None] * 2

This generates a list containing 3 different lists of length two.  You can also
use a list comprehension::

   w, h = 2, 3
   A = [[None] * w for i in range(h)]

Or, you can use an extension that provides a matrix datatype; `NumPy
<http://www.numpy.org/>`_ is the best known.


How do I apply a method to a sequence of objects?
-------------------------------------------------

Use a list comprehension::

   result = [obj.method() for obj in mylist]

More generically, you can try the following function::

   def method_map(objects, method, arguments):
       """method_map([a,b], "meth", (1,2)) gives [a.meth(1,2), b.meth(1,2)]"""
       nobjects = len(objects)
       methods = map(getattr, objects, [method]*nobjects)
       return map(apply, methods, [arguments]*nobjects)


Why does a_tuple[i] += ['item'] raise an exception when the addition works?
---------------------------------------------------------------------------

This is because of a combination of the fact that augmented assignment
operators are *assignment* operators, and the difference between mutable and
immutable objects in Python.

This discussion applies in general when augmented assignment operators are
applied to elements of a tuple that point to mutable objects, but we'll use
a ``list`` and ``+=`` as our exemplar.

If you wrote::

   >>> a_tuple = (1, 2)
   >>> a_tuple[0] += 1
   Traceback (most recent call last):
      ...
   TypeError: 'tuple' object does not support item assignment

The reason for the exception should be immediately clear: ``1`` is added to the
object ``a_tuple[0]`` points to (``1``), producing the result object, ``2``,
but when we attempt to assign the result of the computation, ``2``, to element
``0`` of the tuple, we get an error because we can't change what an element of
a tuple points to.

Under the covers, what this augmented assignment statement is doing is
approximately this::

   >>> result = a_tuple[0] + 1
   >>> a_tuple[0] = result
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

It is the assignment part of the operation that produces the error, since a
tuple is immutable.

When you write something like::

   >>> a_tuple = (['foo'], 'bar')
   >>> a_tuple[0] += ['item']
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

The exception is a bit more surprising, and even more surprising is the fact
that even though there was an error, the append worked::

    >>> a_tuple[0]
    ['foo', 'item']

To see why this happens, you need to know that (a) if an object implements an
``__iadd__`` magic method, it gets called when the ``+=`` augmented assignment
is executed, and its return value is what gets used in the assignment statement;
and (b) for lists, ``__iadd__`` is equivalent to calling ``extend`` on the list
and returning the list.  That's why we say that for lists, ``+=`` is a
"shorthand" for ``list.extend``::

    >>> a_list = []
    >>> a_list += [1]
    >>> a_list
    [1]

This is equivalent to::

    >>> result = a_list.__iadd__([1])
    >>> a_list = result

The object pointed to by a_list has been mutated, and the pointer to the
mutated object is assigned back to ``a_list``.  The end result of the
assignment is a no-op, since it is a pointer to the same object that ``a_list``
was previously pointing to, but the assignment still happens.

Thus, in our tuple example what is happening is equivalent to::

   >>> result = a_tuple[0].__iadd__(['item'])
   >>> a_tuple[0] = result
   Traceback (most recent call last):
     ...
   TypeError: 'tuple' object does not support item assignment

The ``__iadd__`` succeeds, and thus the list is extended, but even though
``result`` points to the same object that ``a_tuple[0]`` already points to,
that final assignment still results in an error, because tuples are immutable.