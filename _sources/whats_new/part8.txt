PEP 3106: Dictionary Views
====================================================

The dictionary methods :meth:`~dict.keys`, :meth:`~dict.values`, and
:meth:`~dict.items` are different in Python 3.x.  They return an object
called a :dfn:`view` instead of a fully materialized list.

It's not possible to change the return values of :meth:`~dict.keys`,
:meth:`~dict.values`, and :meth:`~dict.items` in Python 2.7 because
too much code would break.  Instead the 3.x versions were added
under the new names :meth:`~dict.viewkeys`, :meth:`~dict.viewvalues`,
and :meth:`~dict.viewitems`.

::

    >>> d = dict((i*10, chr(65+i)) for i in range(26))
    >>> d
    {0: 'A', 130: 'N', 10: 'B', 140: 'O', 20: ..., 250: 'Z'}
    >>> d.viewkeys()
    dict_keys([0, 130, 10, 140, 20, 150, 30, ..., 250])

Views can be iterated over, but the key and item views also behave
like sets.  The ``&`` operator performs intersection, and ``|``
performs a union::

    >>> d1 = dict((i*10, chr(65+i)) for i in range(26))
    >>> d2 = dict((i**.5, i) for i in range(1000))
    >>> d1.viewkeys() & d2.viewkeys()
    set([0.0, 10.0, 20.0, 30.0])
    >>> d1.viewkeys() | range(0, 30)
    set([0, 1, 130, 3, 4, 5, 6, ..., 120, 250])

The view keeps track of the dictionary and its contents change as the
dictionary is modified::

    >>> vk = d.viewkeys()
    >>> vk
    dict_keys([0, 130, 10, ..., 250])
    >>> d[260] = '&'
    >>> vk
    dict_keys([0, 130, 260, 10, ..., 250])

However, note that you can't add or remove keys while you're iterating
over the view::

    >>> for k in vk:
    ...     d[k*2] = k
    ...
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    RuntimeError: dictionary changed size during iteration

You can use the view methods in Python 2.x code, and the 2to3
converter will change them to the standard :meth:`~dict.keys`,
:meth:`~dict.values`, and :meth:`~dict.items` methods.

.. seealso::

   :pep:`3106` - Revamping dict.keys(), .values() and .items()
     PEP written by Guido van Rossum.
     Backported to 2.7 by Alexandre Vassalotti; :issue:`1967`.