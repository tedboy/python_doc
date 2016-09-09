
PEP 3137: The memoryview Object
====================================================

The :class:`memoryview` object provides a view of another object's
memory content that matches the :class:`bytes` type's interface.

    >>> import string
    >>> m = memoryview(string.letters)
    >>> m
    <memory at 0x37f850>
    >>> len(m)           # Returns length of underlying object
    52
    >>> m[0], m[25], m[26]   # Indexing returns one byte
    ('a', 'z', 'A')
    >>> m2 = m[0:26]         # Slicing returns another memoryview
    >>> m2
    <memory at 0x37f080>

The content of the view can be converted to a string of bytes or
a list of integers:

    >>> m2.tobytes()
    'abcdefghijklmnopqrstuvwxyz'
    >>> m2.tolist()
    [97, 98, 99, 100, 101, 102, 103, ... 121, 122]
    >>>

:class:`memoryview` objects allow modifying the underlying object if
it's a mutable object.

    >>> m2[0] = 75
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: cannot modify read-only memory
    >>> b = bytearray(string.letters)  # Creating a mutable object
    >>> b
    bytearray(b'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ')
    >>> mb = memoryview(b)
    >>> mb[0] = '*'         # Assign to view, changing the bytearray.
    >>> b[0:5]              # The bytearray has been changed.
    bytearray(b'*bcde')
    >>>

.. seealso::

   :pep:`3137` - Immutable Bytes and Mutable Buffer
     PEP written by Guido van Rossum.
     Implemented by Travis Oliphant, Antoine Pitrou and others.
     Backported to 2.7 by Antoine Pitrou; :issue:`2396`.
