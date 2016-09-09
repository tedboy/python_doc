.. ======================================================================

New and Improved Modules
========================

As in every release, Python's standard library received a number of
enhancements and bug fixes.  Here's a partial list of the most notable
changes, sorted alphabetically by module name. Consult the
:file:`Misc/NEWS` file in the source tree for a more complete list of
changes, or look through the Subversion logs for all the details.

* The :mod:`bdb` module's base debugging class :class:`~bdb.Bdb`
  gained a feature for skipping modules.  The constructor
  now takes an iterable containing glob-style patterns such as
  ``django.*``; the debugger will not step into stack frames
  from a module that matches one of these patterns.
  (Contributed by Maru Newby after a suggestion by
  Senthil Kumaran; :issue:`5142`.)

* The :mod:`binascii` module now supports the buffer API, so it can be
  used with :class:`memoryview` instances and other similar buffer objects.
  (Backported from 3.x by Florent Xicluna; :issue:`7703`.)

* Updated module: the :mod:`bsddb` module has been updated from 4.7.2devel9
  to version 4.8.4 of
  `the pybsddb package <https://www.jcea.es/programacion/pybsddb.htm>`__.
  The new version features better Python 3.x compatibility, various bug fixes,
  and adds several new BerkeleyDB flags and methods.
  (Updated by Jesús Cea Avión; :issue:`8156`.  The pybsddb
  changelog can be read at http://hg.jcea.es/pybsddb/file/tip/ChangeLog.)

* The :mod:`bz2` module's :class:`~bz2.BZ2File` now supports the context
  management protocol, so you can write ``with bz2.BZ2File(...) as f:``.
  (Contributed by Hagen Fürstenau; :issue:`3860`.)

* New class: the :class:`~collections.Counter` class in the :mod:`collections`
  module is useful for tallying data.  :class:`~collections.Counter` instances
  behave mostly like dictionaries but return zero for missing keys instead of
  raising a :exc:`KeyError`:

  .. doctest::
     :options: +NORMALIZE_WHITESPACE

     >>> from collections import Counter
     >>> c = Counter()
     >>> for letter in 'here is a sample of english text':
     ...   c[letter] += 1
     ...
     >>> c
     Counter({' ': 6, 'e': 5, 's': 3, 'a': 2, 'i': 2, 'h': 2,
     'l': 2, 't': 2, 'g': 1, 'f': 1, 'm': 1, 'o': 1, 'n': 1,
     'p': 1, 'r': 1, 'x': 1})
     >>> c['e']
     5
     >>> c['z']
     0

  There are three additional :class:`~collections.Counter` methods.
  :meth:`~collections.Counter.most_common` returns the N most common
  elements and their counts.  :meth:`~collections.Counter.elements`
  returns an iterator over the contained elements, repeating each
  element as many times as its count.
  :meth:`~collections.Counter.subtract` takes an iterable and
  subtracts one for each element instead of adding; if the argument is
  a dictionary or another :class:`Counter`, the counts are
  subtracted. ::

    >>> c.most_common(5)
    [(' ', 6), ('e', 5), ('s', 3), ('a', 2), ('i', 2)]
    >>> c.elements() ->
       'a', 'a', ' ', ' ', ' ', ' ', ' ', ' ',
       'e', 'e', 'e', 'e', 'e', 'g', 'f', 'i', 'i',
       'h', 'h', 'm', 'l', 'l', 'o', 'n', 'p', 's',
       's', 's', 'r', 't', 't', 'x'
    >>> c['e']
    5
    >>> c.subtract('very heavy on the letter e')
    >>> c['e']    # Count is now lower
    -1

  Contributed by Raymond Hettinger; :issue:`1696199`.

  .. revision 79660

  New class: :class:`~collections.OrderedDict` is described in the earlier
  section :ref:`pep-0372`.

  New method: The :class:`~collections.deque` data type now has a
  :meth:`~collections.deque.count` method that returns the number of
  contained elements equal to the supplied argument *x*, and a
  :meth:`~collections.deque.reverse` method that reverses the elements
  of the deque in-place.  :class:`~collections.deque` also exposes its maximum
  length as the read-only :attr:`~collections.deque.maxlen` attribute.
  (Both features added by Raymond Hettinger.)

  The :class:`~collections.namedtuple` class now has an optional *rename* parameter.
  If *rename* is true, field names that are invalid because they've
  been repeated or aren't legal Python identifiers will be
  renamed to legal names that are derived from the field's
  position within the list of fields:

     >>> from collections import namedtuple
     >>> T = namedtuple('T', ['field1', '$illegal', 'for', 'field2'], rename=True)
     >>> T._fields
     ('field1', '_1', '_2', 'field2')

  (Added by Raymond Hettinger; :issue:`1818`.)

  Finally, the :class:`~collections.Mapping` abstract base class now
  returns :const:`NotImplemented` if a mapping is compared to
  another type that isn't a :class:`Mapping`.
  (Fixed by Daniel Stutzbach; :issue:`8729`.)

* Constructors for the parsing classes in the :mod:`ConfigParser` module now
  take an *allow_no_value* parameter, defaulting to false; if true,
  options without values will be allowed.  For example::

    >>> import ConfigParser, StringIO
    >>> sample_config = """
    ... [mysqld]
    ... user = mysql
    ... pid-file = /var/run/mysqld/mysqld.pid
    ... skip-bdb
    ... """
    >>> config = ConfigParser.RawConfigParser(allow_no_value=True)
    >>> config.readfp(StringIO.StringIO(sample_config))
    >>> config.get('mysqld', 'user')
    'mysql'
    >>> print config.get('mysqld', 'skip-bdb')
    None
    >>> print config.get('mysqld', 'unknown')
    Traceback (most recent call last):
      ...
    NoOptionError: No option 'unknown' in section: 'mysqld'

  (Contributed by Mats Kindahl; :issue:`7005`.)

* Deprecated function: :func:`contextlib.nested`, which allows
  handling more than one context manager with a single :keyword:`with`
  statement, has been deprecated, because the :keyword:`with` statement
  now supports multiple context managers.

* The :mod:`cookielib` module now ignores cookies that have an invalid
  version field, one that doesn't contain an integer value.  (Fixed by
  John J. Lee; :issue:`3924`.)

* The :mod:`copy` module's :func:`~copy.deepcopy` function will now
  correctly copy bound instance methods.  (Implemented by
  Robert Collins; :issue:`1515`.)

* The :mod:`ctypes` module now always converts ``None`` to a C NULL
  pointer for arguments declared as pointers.  (Changed by Thomas
  Heller; :issue:`4606`.)  The underlying `libffi library
  <https://sourceware.org/libffi/>`__ has been updated to version
  3.0.9, containing various fixes for different platforms.  (Updated
  by Matthias Klose; :issue:`8142`.)

* New method: the :mod:`datetime` module's :class:`~datetime.timedelta` class
  gained a :meth:`~datetime.timedelta.total_seconds` method that returns the
  number of seconds in the duration.  (Contributed by Brian Quinlan; :issue:`5788`.)

* New method: the :class:`~decimal.Decimal` class gained a
  :meth:`~decimal.Decimal.from_float` class method that performs an exact
  conversion of a floating-point number to a :class:`~decimal.Decimal`.
  This exact conversion strives for the
  closest decimal approximation to the floating-point representation's value;
  the resulting decimal value will therefore still include the inaccuracy,
  if any.
  For example, ``Decimal.from_float(0.1)`` returns
  ``Decimal('0.1000000000000000055511151231257827021181583404541015625')``.
  (Implemented by Raymond Hettinger; :issue:`4796`.)

  Comparing instances of :class:`~decimal.Decimal` with floating-point
  numbers now produces sensible results based on the numeric values
  of the operands.  Previously such comparisons would fall back to
  Python's default rules for comparing objects, which produced arbitrary
  results based on their type.  Note that you still cannot combine
  :class:`Decimal` and floating-point in other operations such as addition,
  since you should be explicitly choosing how to convert between float and
  :class:`~decimal.Decimal`.  (Fixed by Mark Dickinson; :issue:`2531`.)

  The constructor for :class:`~decimal.Decimal` now accepts
  floating-point numbers (added by Raymond Hettinger; :issue:`8257`)
  and non-European Unicode characters such as Arabic-Indic digits
  (contributed by Mark Dickinson; :issue:`6595`).

  Most of the methods of the :class:`~decimal.Context` class now accept integers
  as well as :class:`~decimal.Decimal` instances; the only exceptions are the
  :meth:`~decimal.Context.canonical` and :meth:`~decimal.Context.is_canonical`
  methods.  (Patch by Juan José Conti; :issue:`7633`.)

  When using :class:`~decimal.Decimal` instances with a string's
  :meth:`~str.format` method, the default alignment was previously
  left-alignment.  This has been changed to right-alignment, which is
  more sensible for numeric types.  (Changed by Mark Dickinson; :issue:`6857`.)

  Comparisons involving a signaling NaN value (or ``sNAN``) now signal
  :const:`InvalidOperation` instead of silently returning a true or
  false value depending on the comparison operator.  Quiet NaN values
  (or ``NaN``) are now hashable.  (Fixed by Mark Dickinson;
  :issue:`7279`.)

* The :mod:`difflib` module now produces output that is more
  compatible with modern :command:`diff`/:command:`patch` tools
  through one small change, using a tab character instead of spaces as
  a separator in the header giving the filename.  (Fixed by Anatoly
  Techtonik; :issue:`7585`.)

* The Distutils ``sdist`` command now always regenerates the
  :file:`MANIFEST` file, since even if the :file:`MANIFEST.in` or
  :file:`setup.py` files haven't been modified, the user might have
  created some new files that should be included.
  (Fixed by Tarek Ziadé; :issue:`8688`.)

* The :mod:`doctest` module's :const:`IGNORE_EXCEPTION_DETAIL` flag
  will now ignore the name of the module containing the exception
  being tested.  (Patch by Lennart Regebro; :issue:`7490`.)

* The :mod:`email` module's :class:`~email.message.Message` class will
  now accept a Unicode-valued payload, automatically converting the
  payload to the encoding specified by :attr:`output_charset`.
  (Added by R. David Murray; :issue:`1368247`.)

* The :class:`~fractions.Fraction` class now accepts a single float or
  :class:`~decimal.Decimal` instance, or two rational numbers, as
  arguments to its constructor.  (Implemented by Mark Dickinson;
  rationals added in :issue:`5812`, and float/decimal in
  :issue:`8294`.)

  Ordering comparisons (``<``, ``<=``, ``>``, ``>=``) between
  fractions and complex numbers now raise a :exc:`TypeError`.
  This fixes an oversight, making the :class:`~fractions.Fraction`
  match the other numeric types.

  .. revision 79455

* New class: :class:`~ftplib.FTP_TLS` in
  the :mod:`ftplib` module provides secure FTP
  connections using TLS encapsulation of authentication as well as
  subsequent control and data transfers.
  (Contributed by Giampaolo Rodola; :issue:`2054`.)

  The :meth:`~ftplib.FTP.storbinary` method for binary uploads can now restart
  uploads thanks to an added *rest* parameter (patch by Pablo Mouzo;
  :issue:`6845`.)

* New class decorator: :func:`~functools.total_ordering` in the :mod:`functools`
  module takes a class that defines an :meth:`__eq__` method and one of
  :meth:`__lt__`, :meth:`__le__`, :meth:`__gt__`, or :meth:`__ge__`,
  and generates the missing comparison methods.  Since the
  :meth:`__cmp__` method is being deprecated in Python 3.x,
  this decorator makes it easier to define ordered classes.
  (Added by Raymond Hettinger; :issue:`5479`.)

  New function: :func:`~functools.cmp_to_key` will take an old-style comparison
  function that expects two arguments and return a new callable that
  can be used as the *key* parameter to functions such as
  :func:`sorted`, :func:`min` and :func:`max`, etc.  The primary
  intended use is to help with making code compatible with Python 3.x.
  (Added by Raymond Hettinger.)

* New function: the :mod:`gc` module's :func:`~gc.is_tracked` returns
  true if a given instance is tracked by the garbage collector, false
  otherwise. (Contributed by Antoine Pitrou; :issue:`4688`.)

* The :mod:`gzip` module's :class:`~gzip.GzipFile` now supports the context
  management protocol, so you can write ``with gzip.GzipFile(...) as f:``
  (contributed by Hagen Fürstenau; :issue:`3860`), and it now implements
  the :class:`io.BufferedIOBase` ABC, so you can wrap it with
  :class:`io.BufferedReader` for faster processing
  (contributed by Nir Aides; :issue:`7471`).
  It's also now possible to override the modification time
  recorded in a gzipped file by providing an optional timestamp to
  the constructor.  (Contributed by Jacques Frechet; :issue:`4272`.)

  Files in gzip format can be padded with trailing zero bytes; the
  :mod:`gzip` module will now consume these trailing bytes.  (Fixed by
  Tadek Pietraszek and Brian Curtin; :issue:`2846`.)

* New attribute: the :mod:`hashlib` module now has an :attr:`~hashlib.hashlib.algorithms`
  attribute containing a tuple naming the supported algorithms.
  In Python 2.7, ``hashlib.algorithms`` contains
  ``('md5', 'sha1', 'sha224', 'sha256', 'sha384', 'sha512')``.
  (Contributed by Carl Chenet; :issue:`7418`.)

* The default :class:`~httplib.HTTPResponse` class used by the :mod:`httplib` module now
  supports buffering, resulting in much faster reading of HTTP responses.
  (Contributed by Kristján Valur Jónsson; :issue:`4879`.)

  The :class:`~httplib.HTTPConnection` and :class:`~httplib.HTTPSConnection` classes
  now support a *source_address* parameter, a ``(host, port)`` 2-tuple
  giving the source address that will be used for the connection.
  (Contributed by Eldon Ziegler; :issue:`3972`.)

* The :mod:`ihooks` module now supports relative imports.  Note that
  :mod:`ihooks` is an older module for customizing imports,
  superseded by the :mod:`imputil` module added in Python 2.0.
  (Relative import support added by Neil Schemenauer.)

  .. revision 75423

* The :mod:`imaplib` module now supports IPv6 addresses.
  (Contributed by Derek Morr; :issue:`1655`.)

* New function: the :mod:`inspect` module's :func:`~inspect.getcallargs`
  takes a callable and its positional and keyword arguments,
  and figures out which of the callable's parameters will receive each argument,
  returning a dictionary mapping argument names to their values.  For example::

    >>> from inspect import getcallargs
    >>> def f(a, b=1, *pos, **named):
    ...     pass
    >>> getcallargs(f, 1, 2, 3)
    {'a': 1, 'b': 2, 'pos': (3,), 'named': {}}
    >>> getcallargs(f, a=2, x=4)
    {'a': 2, 'b': 1, 'pos': (), 'named': {'x': 4}}
    >>> getcallargs(f)
    Traceback (most recent call last):
    ...
    TypeError: f() takes at least 1 argument (0 given)

  Contributed by George Sakkis; :issue:`3135`.

* Updated module: The :mod:`io` library has been upgraded to the version shipped with
  Python 3.1.  For 3.1, the I/O library was entirely rewritten in C
  and is 2 to 20 times faster depending on the task being performed.  The
  original Python version was renamed to the :mod:`_pyio` module.

  One minor resulting change: the :class:`io.TextIOBase` class now
  has an :attr:`errors` attribute giving the error setting
  used for encoding and decoding errors (one of ``'strict'``, ``'replace'``,
  ``'ignore'``).

  The :class:`io.FileIO` class now raises an :exc:`OSError` when passed
  an invalid file descriptor.  (Implemented by Benjamin Peterson;
  :issue:`4991`.)  The :meth:`~io.IOBase.truncate` method now preserves the
  file position; previously it would change the file position to the
  end of the new file.  (Fixed by Pascal Chambon; :issue:`6939`.)

* New function: ``itertools.compress(data, selectors)`` takes two
  iterators.  Elements of *data* are returned if the corresponding
  value in *selectors* is true::

    itertools.compress('ABCDEF', [1,0,1,0,1,1]) =>
      A, C, E, F

  .. maybe here is better to use >>> list(itertools.compress(...)) instead

  New function: ``itertools.combinations_with_replacement(iter, r)``
  returns all the possible *r*-length combinations of elements from the
  iterable *iter*.  Unlike :func:`~itertools.combinations`, individual elements
  can be repeated in the generated combinations::

    itertools.combinations_with_replacement('abc', 2) =>
      ('a', 'a'), ('a', 'b'), ('a', 'c'),
      ('b', 'b'), ('b', 'c'), ('c', 'c')

  Note that elements are treated as unique depending on their position
  in the input, not their actual values.

  The :func:`itertools.count` function now has a *step* argument that
  allows incrementing by values other than 1.  :func:`~itertools.count` also
  now allows keyword arguments, and using non-integer values such as
  floats or :class:`~decimal.Decimal` instances.  (Implemented by Raymond
  Hettinger; :issue:`5032`.)

  :func:`itertools.combinations` and :func:`itertools.product`
  previously raised :exc:`ValueError` for values of *r* larger than
  the input iterable.  This was deemed a specification error, so they
  now return an empty iterator.  (Fixed by Raymond Hettinger; :issue:`4816`.)

* Updated module: The :mod:`json` module was upgraded to version 2.0.9 of the
  simplejson package, which includes a C extension that makes
  encoding and decoding faster.
  (Contributed by Bob Ippolito; :issue:`4136`.)

  To support the new :class:`collections.OrderedDict` type, :func:`json.load`
  now has an optional *object_pairs_hook* parameter that will be called
  with any object literal that decodes to a list of pairs.
  (Contributed by Raymond Hettinger; :issue:`5381`.)

* The :mod:`mailbox` module's :class:`~mailbox.Maildir` class now records the
  timestamp on the directories it reads, and only re-reads them if the
  modification time has subsequently changed.  This improves
  performance by avoiding unneeded directory scans.  (Fixed by
  A.M. Kuchling and Antoine Pitrou; :issue:`1607951`, :issue:`6896`.)

* New functions: the :mod:`math` module gained
  :func:`~math.erf` and :func:`~math.erfc` for the error function and the complementary error function,
  :func:`~math.expm1` which computes ``e**x - 1`` with more precision than
  using :func:`~math.exp` and subtracting 1,
  :func:`~math.gamma` for the Gamma function, and
  :func:`~math.lgamma` for the natural log of the Gamma function.
  (Contributed by Mark Dickinson and nirinA raseliarison; :issue:`3366`.)

* The :mod:`multiprocessing` module's :class:`Manager*` classes
  can now be passed a callable that will be called whenever
  a subprocess is started, along with a set of arguments that will be
  passed to the callable.
  (Contributed by lekma; :issue:`5585`.)

  The :class:`~multiprocessing.Pool` class, which controls a pool of worker processes,
  now has an optional *maxtasksperchild* parameter.  Worker processes
  will perform the specified number of tasks and then exit, causing the
  :class:`~multiprocessing.Pool` to start a new worker.  This is useful if tasks may leak
  memory or other resources, or if some tasks will cause the worker to
  become very large.
  (Contributed by Charles Cazabon; :issue:`6963`.)

* The :mod:`nntplib` module now supports IPv6 addresses.
  (Contributed by Derek Morr; :issue:`1664`.)

* New functions: the :mod:`os` module wraps the following POSIX system
  calls: :func:`~os.getresgid` and :func:`~os.getresuid`, which return the
  real, effective, and saved GIDs and UIDs;
  :func:`~os.setresgid` and :func:`~os.setresuid`, which set
  real, effective, and saved GIDs and UIDs to new values;
  :func:`~os.initgroups`, which initialize the group access list
  for the current process.  (GID/UID functions
  contributed by Travis H.; :issue:`6508`.  Support for initgroups added
  by Jean-Paul Calderone; :issue:`7333`.)

  The :func:`os.fork` function now re-initializes the import lock in
  the child process; this fixes problems on Solaris when :func:`~os.fork`
  is called from a thread.  (Fixed by Zsolt Cserna; :issue:`7242`.)

* In the :mod:`os.path` module, the :func:`~os.path.normpath` and
  :func:`~os.path.abspath` functions now preserve Unicode; if their input path
  is a Unicode string, the return value is also a Unicode string.
  (:meth:`~os.path.normpath` fixed by Matt Giuca in :issue:`5827`;
  :meth:`~os.path.abspath` fixed by Ezio Melotti in :issue:`3426`.)

* The :mod:`pydoc` module now has help for the various symbols that Python
  uses.  You can now do ``help('<<')`` or ``help('@')``, for example.
  (Contributed by David Laban; :issue:`4739`.)

* The :mod:`re` module's :func:`~re.split`, :func:`~re.sub`, and :func:`~re.subn`
  now accept an optional *flags* argument, for consistency with the
  other functions in the module.  (Added by Gregory P. Smith.)

* New function: :func:`~runpy.run_path` in the :mod:`runpy` module
  will execute the code at a provided *path* argument.  *path* can be
  the path of a Python source file (:file:`example.py`), a compiled
  bytecode file (:file:`example.pyc`), a directory
  (:file:`./package/`), or a zip archive (:file:`example.zip`).  If a
  directory or zip path is provided, it will be added to the front of
  ``sys.path`` and the module :mod:`__main__` will be imported.  It's
  expected that the directory or zip contains a :file:`__main__.py`;
  if it doesn't, some other :file:`__main__.py` might be imported from
  a location later in ``sys.path``.  This makes more of the machinery
  of :mod:`runpy` available to scripts that want to mimic the way
  Python's command line processes an explicit path name.
  (Added by Nick Coghlan; :issue:`6816`.)

* New function: in the :mod:`shutil` module, :func:`~shutil.make_archive`
  takes a filename, archive type (zip or tar-format), and a directory
  path, and creates an archive containing the directory's contents.
  (Added by Tarek Ziadé.)

  :mod:`shutil`'s :func:`~shutil.copyfile` and :func:`~shutil.copytree`
  functions now raise a :exc:`~shutil.SpecialFileError` exception when
  asked to copy a named pipe.  Previously the code would treat
  named pipes like a regular file by opening them for reading, and
  this would block indefinitely.  (Fixed by Antoine Pitrou; :issue:`3002`.)

* The :mod:`signal` module no longer re-installs the signal handler
  unless this is truly necessary, which fixes a bug that could make it
  impossible to catch the EINTR signal robustly.  (Fixed by
  Charles-Francois Natali; :issue:`8354`.)

* New functions: in the :mod:`site` module, three new functions
  return various site- and user-specific paths.
  :func:`~site.getsitepackages` returns a list containing all
  global site-packages directories,
  :func:`~site.getusersitepackages` returns the path of the user's
  site-packages directory, and
  :func:`~site.getuserbase` returns the value of the :envvar:`USER_BASE`
  environment variable, giving the path to a directory that can be used
  to store data.
  (Contributed by Tarek Ziadé; :issue:`6693`.)

  The :mod:`site` module now reports exceptions occurring
  when the :mod:`sitecustomize` module is imported, and will no longer
  catch and swallow the :exc:`KeyboardInterrupt` exception.  (Fixed by
  Victor Stinner; :issue:`3137`.)

* The :func:`~socket.create_connection` function
  gained a *source_address* parameter, a ``(host, port)`` 2-tuple
  giving the source address that will be used for the connection.
  (Contributed by Eldon Ziegler; :issue:`3972`.)

  The :meth:`~socket.socket.recv_into` and :meth:`~socket.socket.recvfrom_into`
  methods will now write into objects that support the buffer API, most usefully
  the :class:`bytearray` and :class:`memoryview` objects.  (Implemented by
  Antoine Pitrou; :issue:`8104`.)

* The :mod:`SocketServer` module's :class:`~SocketServer.TCPServer` class now
  supports socket timeouts and disabling the Nagle algorithm.
  The :attr:`~SocketServer.TCPServer.disable_nagle_algorithm` class attribute
  defaults to False; if overridden to be True,
  new request connections will have the TCP_NODELAY option set to
  prevent buffering many small sends into a single TCP packet.
  The :attr:`~SocketServer.BaseServer.timeout` class attribute can hold
  a timeout in seconds that will be applied to the request socket; if
  no request is received within that time, :meth:`~SocketServer.BaseServer.handle_timeout`
  will be called and :meth:`~SocketServer.BaseServer.handle_request` will return.
  (Contributed by Kristján Valur Jónsson; :issue:`6192` and :issue:`6267`.)

* Updated module: the :mod:`sqlite3` module has been updated to
  version 2.6.0 of the `pysqlite package <https://github.com/ghaering/pysqlite>`__. Version 2.6.0 includes a number of bugfixes, and adds
  the ability to load SQLite extensions from shared libraries.
  Call the ``enable_load_extension(True)`` method to enable extensions,
  and then call :meth:`~sqlite3.Connection.load_extension` to load a particular shared library.
  (Updated by Gerhard Häring.)

* The :mod:`ssl` module's :class:`~ssl.SSLSocket` objects now support the
  buffer API, which fixed a test suite failure (fix by Antoine Pitrou;
  :issue:`7133`) and automatically set
  OpenSSL's :c:macro:`SSL_MODE_AUTO_RETRY`, which will prevent an error
  code being returned from :meth:`recv` operations that trigger an SSL
  renegotiation (fix by Antoine Pitrou; :issue:`8222`).

  The :func:`ssl.wrap_socket` constructor function now takes a
  *ciphers* argument that's a string listing the encryption algorithms
  to be allowed; the format of the string is described
  `in the OpenSSL documentation
  <https://www.openssl.org/docs/apps/ciphers.html#CIPHER-LIST-FORMAT>`__.
  (Added by Antoine Pitrou; :issue:`8322`.)

  Another change makes the extension load all of OpenSSL's ciphers and
  digest algorithms so that they're all available.  Some SSL
  certificates couldn't be verified, reporting an "unknown algorithm"
  error.  (Reported by Beda Kosata, and fixed by Antoine Pitrou;
  :issue:`8484`.)

  The version of OpenSSL being used is now available as the module
  attributes :data:`ssl.OPENSSL_VERSION` (a string),
  :data:`ssl.OPENSSL_VERSION_INFO` (a 5-tuple), and
  :data:`ssl.OPENSSL_VERSION_NUMBER` (an integer).  (Added by Antoine
  Pitrou; :issue:`8321`.)

* The :mod:`struct` module will no longer silently ignore overflow
  errors when a value is too large for a particular integer format
  code (one of ``bBhHiIlLqQ``); it now always raises a
  :exc:`struct.error` exception.  (Changed by Mark Dickinson;
  :issue:`1523`.)  The :func:`~struct.pack` function will also
  attempt to use :meth:`__index__` to convert and pack non-integers
  before trying the :meth:`__int__` method or reporting an error.
  (Changed by Mark Dickinson; :issue:`8300`.)

* New function: the :mod:`subprocess` module's
  :func:`~subprocess.check_output` runs a command with a specified set of arguments
  and returns the command's output as a string when the command runs without
  error, or raises a :exc:`~subprocess.CalledProcessError` exception otherwise.

  ::

    >>> subprocess.check_output(['df', '-h', '.'])
    'Filesystem     Size   Used  Avail Capacity  Mounted on\n
    /dev/disk0s2    52G    49G   3.0G    94%    /\n'

    >>> subprocess.check_output(['df', '-h', '/bogus'])
      ...
    subprocess.CalledProcessError: Command '['df', '-h', '/bogus']' returned non-zero exit status 1

  (Contributed by Gregory P. Smith.)

  The :mod:`subprocess` module will now retry its internal system calls
  on receiving an :const:`EINTR` signal.  (Reported by several people; final
  patch by Gregory P. Smith in :issue:`1068268`.)

* New function: :func:`~symtable.Symbol.is_declared_global` in the :mod:`symtable` module
  returns true for variables that are explicitly declared to be global,
  false for ones that are implicitly global.
  (Contributed by Jeremy Hylton.)

* The :mod:`syslog` module will now use the value of ``sys.argv[0]`` as the
  identifier instead of the previous default value of ``'python'``.
  (Changed by Sean Reifschneider; :issue:`8451`.)

* The ``sys.version_info`` value is now a named tuple, with attributes
  named :attr:`major`, :attr:`minor`, :attr:`micro`,
  :attr:`releaselevel`, and :attr:`serial`.  (Contributed by Ross
  Light; :issue:`4285`.)

  :func:`sys.getwindowsversion` also returns a named tuple,
  with attributes named :attr:`major`, :attr:`minor`, :attr:`build`,
  :attr:`platform`, :attr:`service_pack`, :attr:`service_pack_major`,
  :attr:`service_pack_minor`, :attr:`suite_mask`, and
  :attr:`product_type`.  (Contributed by Brian Curtin; :issue:`7766`.)

* The :mod:`tarfile` module's default error handling has changed, to
  no longer suppress fatal errors.  The default error level was previously 0,
  which meant that errors would only result in a message being written to the
  debug log, but because the debug log is not activated by default,
  these errors go unnoticed.  The default error level is now 1,
  which raises an exception if there's an error.
  (Changed by Lars Gustäbel; :issue:`7357`.)

  :mod:`tarfile` now supports filtering the :class:`~tarfile.TarInfo`
  objects being added to a tar file.  When you call :meth:`~tarfile.TarFile.add`,
  you may supply an optional *filter* argument
  that's a callable.  The *filter* callable will be passed the
  :class:`~tarfile.TarInfo` for every file being added, and can modify and return it.
  If the callable returns ``None``, the file will be excluded from the
  resulting archive.  This is more powerful than the existing
  *exclude* argument, which has therefore been deprecated.
  (Added by Lars Gustäbel; :issue:`6856`.)
  The :class:`~tarfile.TarFile` class also now supports the context management protocol.
  (Added by Lars Gustäbel; :issue:`7232`.)

* The :meth:`~threading.Event.wait` method of the :class:`threading.Event` class
  now returns the internal flag on exit.  This means the method will usually
  return true because :meth:`~threading.Event.wait` is supposed to block until the
  internal flag becomes true.  The return value will only be false if
  a timeout was provided and the operation timed out.
  (Contributed by Tim Lesher; :issue:`1674032`.)

* The Unicode database provided by the :mod:`unicodedata` module is
  now used internally to determine which characters are numeric,
  whitespace, or represent line breaks.  The database also
  includes information from the :file:`Unihan.txt` data file (patch
  by Anders Chrigström and Amaury Forgeot d'Arc; :issue:`1571184`)
  and has been updated to version 5.2.0 (updated by
  Florent Xicluna; :issue:`8024`).

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

  The :mod:`urlparse` module also supports IPv6 literal addresses as defined by
  :rfc:`2732` (contributed by Senthil Kumaran; :issue:`2987`). ::

    >>> urlparse.urlparse('http://[1080::8:800:200C:417A]/foo')
    ParseResult(scheme='http', netloc='[1080::8:800:200C:417A]',
                path='/foo', params='', query='', fragment='')

* New class: the :class:`~weakref.WeakSet` class in the :mod:`weakref`
  module is a set that only holds weak references to its elements; elements
  will be removed once there are no references pointing to them.
  (Originally implemented in Python 3.x by Raymond Hettinger, and backported
  to 2.7 by Michael Foord.)

* The ElementTree library, :mod:`xml.etree`, no longer escapes
  ampersands and angle brackets when outputting an XML processing
  instruction (which looks like ``<?xml-stylesheet href="#style1"?>``)
  or comment (which looks like ``<!-- comment -->``).
  (Patch by Neil Muller; :issue:`2746`.)

* The XML-RPC client and server, provided by the :mod:`xmlrpclib` and
  :mod:`SimpleXMLRPCServer` modules, have improved performance by
  supporting HTTP/1.1 keep-alive and by optionally using gzip encoding
  to compress the XML being exchanged.  The gzip compression is
  controlled by the :attr:`encode_threshold` attribute of
  :class:`SimpleXMLRPCRequestHandler`, which contains a size in bytes;
  responses larger than this will be compressed.
  (Contributed by Kristján Valur Jónsson; :issue:`6267`.)

* The :mod:`zipfile` module's :class:`~zipfile.ZipFile` now supports the context
  management protocol, so you can write ``with zipfile.ZipFile(...) as f:``.
  (Contributed by Brian Curtin; :issue:`5511`.)

  :mod:`zipfile` now also supports archiving empty directories and
  extracts them correctly.  (Fixed by Kuba Wieczorek; :issue:`4710`.)
  Reading files out of an archive is faster, and interleaving
  :meth:`~zipfile.ZipFile.read` and :meth:`~zipfile.ZipFile.readline` now works correctly.
  (Contributed by Nir Aides; :issue:`7610`.)

  The :func:`~zipfile.is_zipfile` function now
  accepts a file object, in addition to the path names accepted in earlier
  versions.  (Contributed by Gabriel Genellina; :issue:`4756`.)

  The :meth:`~zipfile.ZipFile.writestr` method now has an optional *compress_type* parameter
  that lets you override the default compression method specified in the
  :class:`~zipfile.ZipFile` constructor.  (Contributed by Ronald Oussoren;
  :issue:`6003`.)


.. ======================================================================
.. whole new modules get described in subsections here


.. _importlib-section:

New module: importlib
------------------------------

Python 3.1 includes the :mod:`importlib` package, a re-implementation
of the logic underlying Python's :keyword:`import` statement.
:mod:`importlib` is useful for implementors of Python interpreters and
to users who wish to write new importers that can participate in the
import process.  Python 2.7 doesn't contain the complete
:mod:`importlib` package, but instead has a tiny subset that contains
a single function, :func:`~importlib.import_module`.

``import_module(name, package=None)`` imports a module.  *name* is
a string containing the module or package's name.  It's possible to do
relative imports by providing a string that begins with a ``.``
character, such as ``..utils.errors``.  For relative imports, the
*package* argument must be provided and is the name of the package that
will be used as the anchor for
the relative import.  :func:`~importlib.import_module` both inserts the imported
module into ``sys.modules`` and returns the module object.

Here are some examples::

    >>> from importlib import import_module
    >>> anydbm = import_module('anydbm')  # Standard absolute import
    >>> anydbm
    <module 'anydbm' from '/p/python/Lib/anydbm.py'>
    >>> # Relative import
    >>> file_util = import_module('..file_util', 'distutils.command')
    >>> file_util
    <module 'distutils.file_util' from '/python/Lib/distutils/file_util.pyc'>

:mod:`importlib` was implemented by Brett Cannon and introduced in
Python 3.1.


New module: sysconfig
---------------------------------

The :mod:`sysconfig` module has been pulled out of the Distutils
package, becoming a new top-level module in its own right.
:mod:`sysconfig` provides functions for getting information about
Python's build process: compiler switches, installation paths, the
platform name, and whether Python is running from its source
directory.

Some of the functions in the module are:

* :func:`~sysconfig.get_config_var` returns variables from Python's
  Makefile and the :file:`pyconfig.h` file.
* :func:`~sysconfig.get_config_vars` returns a dictionary containing
  all of the configuration variables.
* :func:`~sysconfig.get_path` returns the configured path for
  a particular type of module: the standard library,
  site-specific modules, platform-specific modules, etc.
* :func:`~sysconfig.is_python_build` returns true if you're running a
  binary from a Python source tree, and false otherwise.

Consult the :mod:`sysconfig` documentation for more details and for
a complete list of functions.

The Distutils package and :mod:`sysconfig` are now maintained by Tarek
Ziadé, who has also started a Distutils2 package (source repository at
https://hg.python.org/distutils2/) for developing a next-generation
version of Distutils.


ttk: Themed Widgets for Tk
--------------------------

Tcl/Tk 8.5 includes a set of themed widgets that re-implement basic Tk
widgets but have a more customizable appearance and can therefore more
closely resemble the native platform's widgets.  This widget
set was originally called Tile, but was renamed to Ttk (for "themed Tk")
on being added to Tcl/Tck release 8.5.

To learn more, read the :mod:`ttk` module documentation.  You may also
wish to read the Tcl/Tk manual page describing the
Ttk theme engine, available at
https://www.tcl.tk/man/tcl8.5/TkCmd/ttk_intro.htm. Some
screenshots of the Python/Ttk code in use are at
http://code.google.com/p/python-ttk/wiki/Screenshots.

The :mod:`ttk` module was written by Guilherme Polo and added in
:issue:`2983`.  An alternate version called ``Tile.py``, written by
Martin Franklin and maintained by Kevin Walzer, was proposed for
inclusion in :issue:`2618`, but the authors argued that Guilherme
Polo's work was more comprehensive.


.. _unittest-section:

Updated module: unittest
---------------------------------

The :mod:`unittest` module was greatly enhanced; many
new features were added.  Most of these features were implemented
by Michael Foord, unless otherwise noted.  The enhanced version of
the module is downloadable separately for use with Python versions 2.4 to 2.6,
packaged as the :mod:`unittest2` package, from
https://pypi.python.org/pypi/unittest2.

When used from the command line, the module can automatically discover
tests.  It's not as fancy as `py.test <http://pytest.org>`__ or
`nose <http://code.google.com/p/python-nose/>`__, but provides a simple way
to run tests kept within a set of package directories.  For example,
the following command will search the :file:`test/` subdirectory for
any importable test files named ``test*.py``::

   python -m unittest discover -s test

Consult the :mod:`unittest` module documentation for more details.
(Developed in :issue:`6001`.)

The :func:`~unittest.main` function supports some other new options:

* :option:`-b <unittest -b>` or :option:`--buffer` will buffer the standard output
  and standard error streams during each test.  If the test passes,
  any resulting output will be discarded; on failure, the buffered
  output will be displayed.

* :option:`-c <unittest -c>` or :option:`--catch` will cause the control-C interrupt
  to be handled more gracefully.  Instead of interrupting the test
  process immediately, the currently running test will be completed
  and then the partial results up to the interruption will be reported.
  If you're impatient, a second press of control-C will cause an immediate
  interruption.

  This control-C handler tries to avoid causing problems when the code
  being tested or the tests being run have defined a signal handler of
  their own, by noticing that a signal handler was already set and
  calling it.  If this doesn't work for you, there's a
  :func:`~unittest.removeHandler` decorator that can be used to mark tests that
  should have the control-C handling disabled.

* :option:`-f <unittest -f>` or :option:`--failfast` makes
  test execution stop immediately when a test fails instead of
  continuing to execute further tests.  (Suggested by Cliff Dyer and
  implemented by Michael Foord; :issue:`8074`.)

The progress messages now show 'x' for expected failures
and 'u' for unexpected successes when run in verbose mode.
(Contributed by Benjamin Peterson.)

Test cases can raise the :exc:`~unittest.SkipTest` exception to skip a
test (:issue:`1034053`).

The error messages for :meth:`~unittest.TestCase.assertEqual`,
:meth:`~unittest.TestCase.assertTrue`, and :meth:`~unittest.TestCase.assertFalse`
failures now provide more information.  If you set the
:attr:`~unittest.TestCase.longMessage` attribute of your :class:`~unittest.TestCase` classes to
True, both the standard error message and any additional message you
provide will be printed for failures.  (Added by Michael Foord; :issue:`5663`.)

The :meth:`~unittest.TestCase.assertRaises` method now
returns a context handler when called without providing a callable
object to run.  For example, you can write this::

  with self.assertRaises(KeyError):
      {}['foo']

(Implemented by Antoine Pitrou; :issue:`4444`.)

.. rev 78774

Module- and class-level setup and teardown fixtures are now supported.
Modules can contain :func:`~unittest.setUpModule` and :func:`~unittest.tearDownModule`
functions.  Classes can have :meth:`~unittest.TestCase.setUpClass` and
:meth:`~unittest.TestCase.tearDownClass` methods that must be defined as class methods
(using ``@classmethod`` or equivalent).  These functions and
methods are invoked when the test runner switches to a test case in a
different module or class.

The methods :meth:`~unittest.TestCase.addCleanup` and
:meth:`~unittest.TestCase.doCleanups` were added.
:meth:`~unittest.TestCase.addCleanup` lets you add cleanup functions that
will be called unconditionally (after :meth:`~unittest.TestCase.setUp` if
:meth:`~unittest.TestCase.setUp` fails, otherwise after :meth:`~unittest.TestCase.tearDown`). This allows
for much simpler resource allocation and deallocation during tests
(:issue:`5679`).

A number of new methods were added that provide more specialized
tests.  Many of these methods were written by Google engineers
for use in their test suites; Gregory P. Smith, Michael Foord, and
GvR worked on merging them into Python's version of :mod:`unittest`.

* :meth:`~unittest.TestCase.assertIsNone` and :meth:`~unittest.TestCase.assertIsNotNone` take one
  expression and verify that the result is or is not ``None``.

* :meth:`~unittest.TestCase.assertIs` and :meth:`~unittest.TestCase.assertIsNot`
  take two values and check whether the two values evaluate to the same object or not.
  (Added by Michael Foord; :issue:`2578`.)

* :meth:`~unittest.TestCase.assertIsInstance` and
  :meth:`~unittest.TestCase.assertNotIsInstance` check whether
  the resulting object is an instance of a particular class, or of
  one of a tuple of classes.  (Added by Georg Brandl; :issue:`7031`.)

* :meth:`~unittest.TestCase.assertGreater`, :meth:`~unittest.TestCase.assertGreaterEqual`,
  :meth:`~unittest.TestCase.assertLess`, and :meth:`~unittest.TestCase.assertLessEqual` compare
  two quantities.

* :meth:`~unittest.TestCase.assertMultiLineEqual` compares two strings, and if they're
  not equal, displays a helpful comparison that highlights the
  differences in the two strings.  This comparison is now used by
  default when Unicode strings are compared with :meth:`~unittest.TestCase.assertEqual`.

* :meth:`~unittest.TestCase.assertRegexpMatches` and
  :meth:`~unittest.TestCase.assertNotRegexpMatches` checks whether the
  first argument is a string matching or not matching the regular
  expression provided as the second argument (:issue:`8038`).

* :meth:`~unittest.TestCase.assertRaisesRegexp` checks whether a particular exception
  is raised, and then also checks that the string representation of
  the exception matches the provided regular expression.

* :meth:`~unittest.TestCase.assertIn` and :meth:`~unittest.TestCase.assertNotIn`
  tests whether *first* is or is not in  *second*.

* :meth:`~unittest.TestCase.assertItemsEqual` tests whether two provided sequences
  contain the same elements.

* :meth:`~unittest.TestCase.assertSetEqual` compares whether two sets are equal, and
  only reports the differences between the sets in case of error.

* Similarly, :meth:`~unittest.TestCase.assertListEqual` and :meth:`~unittest.TestCase.assertTupleEqual`
  compare the specified types and explain any differences without necessarily
  printing their full values; these methods are now used by default
  when comparing lists and tuples using :meth:`~unittest.TestCase.assertEqual`.
  More generally, :meth:`~unittest.TestCase.assertSequenceEqual` compares two sequences
  and can optionally check whether both sequences are of a
  particular type.

* :meth:`~unittest.TestCase.assertDictEqual` compares two dictionaries and reports the
  differences; it's now used by default when you compare two dictionaries
  using :meth:`~unittest.TestCase.assertEqual`.  :meth:`~unittest.TestCase.assertDictContainsSubset` checks whether
  all of the key/value pairs in *first* are found in *second*.

* :meth:`~unittest.TestCase.assertAlmostEqual` and :meth:`~unittest.TestCase.assertNotAlmostEqual` test
  whether *first* and *second* are approximately equal.  This method
  can either round their difference to an optionally-specified number
  of *places* (the default is 7) and compare it to zero, or require
  the difference to be smaller than a supplied *delta* value.

* :meth:`~unittest.TestLoader.loadTestsFromName` properly honors the
  :attr:`~unittest.TestLoader.suiteClass` attribute of
  the :class:`~unittest.TestLoader`. (Fixed by Mark Roddy; :issue:`6866`.)

* A new hook lets you extend the :meth:`~unittest.TestCase.assertEqual` method to handle
  new data types.  The :meth:`~unittest.TestCase.addTypeEqualityFunc` method takes a type
  object and a function. The function will be used when both of the
  objects being compared are of the specified type.  This function
  should compare the two objects and raise an exception if they don't
  match; it's a good idea for the function to provide additional
  information about why the two objects aren't matching, much as the new
  sequence comparison methods do.

:func:`unittest.main` now takes an optional ``exit`` argument.  If
False, :func:`~unittest.main` doesn't call :func:`sys.exit`, allowing
:func:`~unittest.main` to be used from the interactive interpreter.
(Contributed by J. Pablo Fernández; :issue:`3379`.)

:class:`~unittest.TestResult` has new :meth:`~unittest.TestResult.startTestRun` and
:meth:`~unittest.TestResult.stopTestRun` methods that are called immediately before
and after a test run.  (Contributed by Robert Collins; :issue:`5728`.)

With all these changes, the :file:`unittest.py` was becoming awkwardly
large, so the module was turned into a package and the code split into
several files (by Benjamin Peterson).  This doesn't affect how the
module is imported or used.

.. seealso::

  http://www.voidspace.org.uk/python/articles/unittest2.shtml
    Describes the new features, how to use them, and the
    rationale for various design decisions.  (By Michael Foord.)

.. _elementtree-section:

Updated module: ElementTree 1.3
---------------------------------

The version of the ElementTree library included with Python was updated to
version 1.3.  Some of the new features are:

* The various parsing functions now take a *parser* keyword argument
  giving an :class:`~xml.etree.ElementTree.XMLParser` instance that will
  be used.  This makes it possible to override the file's internal encoding::

    p = ET.XMLParser(encoding='utf-8')
    t = ET.XML("""<root/>""", parser=p)

  Errors in parsing XML now raise a :exc:`ParseError` exception, whose
  instances have a :attr:`position` attribute
  containing a (*line*, *column*) tuple giving the location of the problem.

* ElementTree's code for converting trees to a string has been
  significantly reworked, making it roughly twice as fast in many
  cases.  The :meth:`ElementTree.write() <xml.etree.ElementTree.ElementTree.write>`
  and :meth:`Element.write` methods now have a *method* parameter that can be
  "xml" (the default), "html", or "text".  HTML mode will output empty
  elements as ``<empty></empty>`` instead of ``<empty/>``, and text
  mode will skip over elements and only output the text chunks.  If
  you set the :attr:`tag` attribute of an element to ``None`` but
  leave its children in place, the element will be omitted when the
  tree is written out, so you don't need to do more extensive rearrangement
  to remove a single element.

  Namespace handling has also been improved.  All ``xmlns:<whatever>``
  declarations are now output on the root element, not scattered throughout
  the resulting XML.  You can set the default namespace for a tree
  by setting the :attr:`default_namespace` attribute and can
  register new prefixes with :meth:`~xml.etree.ElementTree.register_namespace`.  In XML mode,
  you can use the true/false *xml_declaration* parameter to suppress the
  XML declaration.

* New :class:`~xml.etree.ElementTree.Element` method:
  :meth:`~xml.etree.ElementTree.Element.extend` appends the items from a
  sequence to the element's children.  Elements themselves behave like
  sequences, so it's easy to move children from one element to
  another::

    from xml.etree import ElementTree as ET

    t = ET.XML("""<list>
      <item>1</item> <item>2</item>  <item>3</item>
    </list>""")
    new = ET.XML('<root/>')
    new.extend(t)

    # Outputs <root><item>1</item>...</root>
    print ET.tostring(new)

* New :class:`Element` method:
  :meth:`~xml.etree.ElementTree.Element.iter` yields the children of the
  element as a generator.  It's also possible to write ``for child in
  elem:`` to loop over an element's children.  The existing method
  :meth:`getiterator` is now deprecated, as is :meth:`getchildren`
  which constructs and returns a list of children.

* New :class:`Element` method:
  :meth:`~xml.etree.ElementTree.Element.itertext` yields all chunks of
  text that are descendants of the element.  For example::

    t = ET.XML("""<list>
      <item>1</item> <item>2</item>  <item>3</item>
    </list>""")

    # Outputs ['\n  ', '1', ' ', '2', '  ', '3', '\n']
    print list(t.itertext())

* Deprecated: using an element as a Boolean (i.e., ``if elem:``) would
  return true if the element had any children, or false if there were
  no children.  This behaviour is confusing -- ``None`` is false, but
  so is a childless element? -- so it will now trigger a
  :exc:`FutureWarning`.  In your code, you should be explicit: write
  ``len(elem) != 0`` if you're interested in the number of children,
  or ``elem is not None``.

Fredrik Lundh develops ElementTree and produced the 1.3 version;
you can read his article describing 1.3 at
http://effbot.org/zone/elementtree-13-intro.htm.
Florent Xicluna updated the version included with
Python, after discussions on python-dev and in :issue:`6472`.)