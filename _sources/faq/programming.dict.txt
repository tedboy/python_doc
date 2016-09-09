


Dictionaries
============
.. contents:: `Contents`
   :depth: 2
   :local:


How can I get a dictionary to display its keys in a consistent order?
---------------------------------------------------------------------

You can't.  Dictionaries store their keys in an unpredictable order, so the
display order of a dictionary's elements will be similarly unpredictable.

This can be frustrating if you want to save a printable version to a file, make
some changes and then compare it with some other printed dictionary.  In this
case, use the ``pprint`` module to pretty-print the dictionary; the items will
be presented in order sorted by the key.

A more complicated solution is to subclass ``dict`` to create a
``SortedDict`` class that prints itself in a predictable order.  Here's one
simpleminded implementation of such a class::

   class SortedDict(dict):
       def __repr__(self):
           keys = sorted(self.keys())
           result = ("{!r}: {!r}".format(k, self[k]) for k in keys)
           return "{{{}}}".format(", ".join(result))

       __str__ = __repr__

This will work for many common situations you might encounter, though it's far
from a perfect solution. The largest flaw is that if some values in the
dictionary are also dictionaries, their values won't be presented in any
particular order.


I want to do a complicated sort: can you do a Schwartzian Transform in Python?
------------------------------------------------------------------------------

The technique, attributed to Randal Schwartz of the Perl community, sorts the
elements of a list by a metric which maps each element to its "sort value". In
Python, use the ``key`` argument for the :func:`sort()` function::

   Isorted = L[:]
   Isorted.sort(key=lambda s: int(s[10:15]))


How can I sort one list by values from another list?
----------------------------------------------------

Merge them into a single list of tuples, sort the resulting list, and then pick
out the element you want. ::

   >>> list1 = ["what", "I'm", "sorting", "by"]
   >>> list2 = ["something", "else", "to", "sort"]
   >>> pairs = zip(list1, list2)
   >>> pairs
   [('what', 'something'), ("I'm", 'else'), ('sorting', 'to'), ('by', 'sort')]
   >>> pairs.sort()
   >>> result = [ x[1] for x in pairs ]
   >>> result
   ['else', 'sort', 'to', 'something']

An alternative for the last step is::

   >>> result = []
   >>> for p in pairs: result.append(p[1])

If you find this more legible, you might prefer to use this instead of the final
list comprehension.  However, it is almost twice as slow for long lists.  Why?
First, the ``append()`` operation has to reallocate memory, and while it uses
some tricks to avoid doing that each time, it still has to do it occasionally,
and that costs quite a bit.  Second, the expression "result.append" requires an
extra attribute lookup, and third, there's a speed reduction from having to make
all those function calls.
