.. ========================================================================
.. Large, PEP-level features and changes should be described here.
.. ========================================================================

.. _pep-0372:

PEP 372: Adding an Ordered Dictionary to collections
====================================================

Regular Python dictionaries iterate over key/value pairs in arbitrary order.
Over the years, a number of authors have written alternative implementations
that remember the order that the keys were originally inserted.  Based on
the experiences from those implementations, 2.7 introduces a new
:class:`~collections.OrderedDict` class in the :mod:`collections` module.

The :class:`~collections.OrderedDict` API provides the same interface as regular
dictionaries but iterates over keys and values in a guaranteed order
depending on when a key was first inserted::

    >>> from collections import OrderedDict
    >>> d = OrderedDict([('first', 1),
    ...                  ('second', 2),
    ...                  ('third', 3)])
    >>> d.items()
    [('first', 1), ('second', 2), ('third', 3)]

If a new entry overwrites an existing entry, the original insertion
position is left unchanged::

    >>> d['second'] = 4
    >>> d.items()
    [('first', 1), ('second', 4), ('third', 3)]

Deleting an entry and reinserting it will move it to the end::

    >>> del d['second']
    >>> d['second'] = 5
    >>> d.items()
    [('first', 1), ('third', 3), ('second', 5)]

The :meth:`~collections.OrderedDict.popitem` method has an optional *last*
argument that defaults to True.  If *last* is True, the most recently
added key is returned and removed; if it's False, the
oldest key is selected::

    >>> od = OrderedDict([(x,0) for x in range(20)])
    >>> od.popitem()
    (19, 0)
    >>> od.popitem()
    (18, 0)
    >>> od.popitem(last=False)
    (0, 0)
    >>> od.popitem(last=False)
    (1, 0)

Comparing two ordered dictionaries checks both the keys and values,
and requires that the insertion order was the same::

    >>> od1 = OrderedDict([('first', 1),
    ...                    ('second', 2),
    ...                    ('third', 3)])
    >>> od2 = OrderedDict([('third', 3),
    ...                    ('first', 1),
    ...                    ('second', 2)])
    >>> od1 == od2
    False
    >>> # Move 'third' key to the end
    >>> del od2['third']; od2['third'] = 3
    >>> od1 == od2
    True

Comparing an :class:`~collections.OrderedDict` with a regular dictionary
ignores the insertion order and just compares the keys and values.

How does the :class:`~collections.OrderedDict` work?  It maintains a
doubly-linked list of keys, appending new keys to the list as they're inserted.
A secondary dictionary maps keys to their corresponding list node, so
deletion doesn't have to traverse the entire linked list and therefore
remains O(1).

The standard library now supports use of ordered dictionaries in several
modules.

* The :mod:`ConfigParser` module uses them by default, meaning that
  configuration files can now be read, modified, and then written back
  in their original order.

* The :meth:`~collections.somenamedtuple._asdict()` method for
  :func:`collections.namedtuple` now returns an ordered dictionary with the
  values appearing in the same order as the underlying tuple indices.

* The :mod:`json` module's :class:`~json.JSONDecoder` class
  constructor was extended with an *object_pairs_hook* parameter to
  allow :class:`OrderedDict` instances to be built by the decoder.
  Support was also added for third-party tools like
  `PyYAML <http://pyyaml.org/>`_.

.. seealso::

   :pep:`372` - Adding an ordered dictionary to collections
     PEP written by Armin Ronacher and Raymond Hettinger;
     implemented by Raymond Hettinger.