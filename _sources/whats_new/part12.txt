.. ======================================================================


Build and C API Changes
=======================

Changes to Python's build process and to the C API include:

* The latest release of the GNU Debugger, GDB 7, can be `scripted
  using Python
  <https://sourceware.org/gdb/current/onlinedocs/gdb/Python.html>`__.
  When you begin debugging an executable program P, GDB will look for
  a file named ``P-gdb.py`` and automatically read it.  Dave Malcolm
  contributed a :file:`python-gdb.py` that adds a number of
  commands useful when debugging Python itself.  For example,
  ``py-up`` and ``py-down`` go up or down one Python stack frame,
  which usually corresponds to several C stack frames.  ``py-print``
  prints the value of a Python variable, and ``py-bt`` prints the
  Python stack trace.  (Added as a result of :issue:`8032`.)

* If you use the :file:`.gdbinit` file provided with Python,
  the "pyo" macro in the 2.7 version now works correctly when the thread being
  debugged doesn't hold the GIL; the macro now acquires it before printing.
  (Contributed by Victor Stinner; :issue:`3632`.)

* :c:func:`Py_AddPendingCall` is now thread-safe, letting any
  worker thread submit notifications to the main Python thread.  This
  is particularly useful for asynchronous IO operations.
  (Contributed by Kristján Valur Jónsson; :issue:`4293`.)

* New function: :c:func:`PyCode_NewEmpty` creates an empty code object;
  only the filename, function name, and first line number are required.
  This is useful for extension modules that are attempting to
  construct a more useful traceback stack.  Previously such
  extensions needed to call :c:func:`PyCode_New`, which had many
  more arguments.  (Added by Jeffrey Yasskin.)

* New function: :c:func:`PyErr_NewExceptionWithDoc` creates a new
  exception class, just as the existing :c:func:`PyErr_NewException` does,
  but takes an extra ``char *`` argument containing the docstring for the
  new exception class.  (Added by 'lekma' on the Python bug tracker;
  :issue:`7033`.)

* New function: :c:func:`PyFrame_GetLineNumber` takes a frame object
  and returns the line number that the frame is currently executing.
  Previously code would need to get the index of the bytecode
  instruction currently executing, and then look up the line number
  corresponding to that address.  (Added by Jeffrey Yasskin.)

* New functions: :c:func:`PyLong_AsLongAndOverflow` and
  :c:func:`PyLong_AsLongLongAndOverflow`  approximates a Python long
  integer as a C :c:type:`long` or :c:type:`long long`.
  If the number is too large to fit into
  the output type, an *overflow* flag is set and returned to the caller.
  (Contributed by Case Van Horsen; :issue:`7528` and :issue:`7767`.)

* New function: stemming from the rewrite of string-to-float conversion,
  a new :c:func:`PyOS_string_to_double` function was added.  The old
  :c:func:`PyOS_ascii_strtod` and :c:func:`PyOS_ascii_atof` functions
  are now deprecated.

* New function: :c:func:`PySys_SetArgvEx` sets the value of
  ``sys.argv`` and can optionally update ``sys.path`` to include the
  directory containing the script named by ``sys.argv[0]`` depending
  on the value of an *updatepath* parameter.

  This function was added to close a security hole for applications
  that embed Python.  The old function, :c:func:`PySys_SetArgv`, would
  always update ``sys.path``, and sometimes it would add the current
  directory.  This meant that, if you ran an application embedding
  Python in a directory controlled by someone else, attackers could
  put a Trojan-horse module in the directory (say, a file named
  :file:`os.py`) that your application would then import and run.

  If you maintain a C/C++ application that embeds Python, check
  whether you're calling :c:func:`PySys_SetArgv` and carefully consider
  whether the application should be using :c:func:`PySys_SetArgvEx`
  with *updatepath* set to false.

  Security issue reported as `CVE-2008-5983
  <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-5983>`_;
  discussed in :issue:`5753`, and fixed by Antoine Pitrou.

* New macros: the Python header files now define the following macros:
  :c:macro:`Py_ISALNUM`,
  :c:macro:`Py_ISALPHA`,
  :c:macro:`Py_ISDIGIT`,
  :c:macro:`Py_ISLOWER`,
  :c:macro:`Py_ISSPACE`,
  :c:macro:`Py_ISUPPER`,
  :c:macro:`Py_ISXDIGIT`,
  :c:macro:`Py_TOLOWER`, and :c:macro:`Py_TOUPPER`.
  All of these functions are analogous to the C
  standard macros for classifying characters, but ignore the current
  locale setting, because in
  several places Python needs to analyze characters in a
  locale-independent way.  (Added by Eric Smith;
  :issue:`5793`.)

  .. XXX these macros don't seem to be described in the c-api docs.

* Removed function: :c:macro:`PyEval_CallObject` is now only available
  as a macro.  A function version was being kept around to preserve
  ABI linking compatibility, but that was in 1997; it can certainly be
  deleted by now.  (Removed by Antoine Pitrou; :issue:`8276`.)

* New format codes: the :c:func:`PyFormat_FromString`,
  :c:func:`PyFormat_FromStringV`, and :c:func:`PyErr_Format` functions now
  accept ``%lld`` and ``%llu`` format codes for displaying
  C's :c:type:`long long` types.
  (Contributed by Mark Dickinson; :issue:`7228`.)

* The complicated interaction between threads and process forking has
  been changed.  Previously, the child process created by
  :func:`os.fork` might fail because the child is created with only a
  single thread running, the thread performing the :func:`os.fork`.
  If other threads were holding a lock, such as Python's import lock,
  when the fork was performed, the lock would still be marked as
  "held" in the new process.  But in the child process nothing would
  ever release the lock, since the other threads weren't replicated,
  and the child process would no longer be able to perform imports.

  Python 2.7 acquires the import lock before performing an
  :func:`os.fork`, and will also clean up any locks created using the
  :mod:`threading` module.  C extension modules that have internal
  locks, or that call :c:func:`fork()` themselves, will not benefit
  from this clean-up.

  (Fixed by Thomas Wouters; :issue:`1590864`.)

* The :c:func:`Py_Finalize` function now calls the internal
  :func:`threading._shutdown` function; this prevents some exceptions from
  being raised when an interpreter shuts down.
  (Patch by Adam Olsen; :issue:`1722344`.)

* When using the :c:type:`PyMemberDef` structure to define attributes
  of a type, Python will no longer let you try to delete or set a
  :const:`T_STRING_INPLACE` attribute.

  .. rev 79644

* Global symbols defined by the :mod:`ctypes` module are now prefixed
  with ``Py``, or with ``_ctypes``.  (Implemented by Thomas
  Heller; :issue:`3102`.)

* New configure option: the :option:`--with-system-expat` switch allows
  building the :mod:`pyexpat` module to use the system Expat library.
  (Contributed by Arfrever Frehtes Taifersar Arahesis; :issue:`7609`.)

* New configure option: the
  :option:`--with-valgrind` option will now disable the pymalloc
  allocator, which is difficult for the Valgrind memory-error detector
  to analyze correctly.
  Valgrind will therefore be better at detecting memory leaks and
  overruns. (Contributed by James Henstridge; :issue:`2422`.)

* New configure option: you can now supply an empty string to
  :option:`--with-dbmliborder=` in order to disable all of the various
  DBM modules.  (Added by Arfrever Frehtes Taifersar Arahesis;
  :issue:`6491`.)

* The :program:`configure` script now checks for floating-point rounding bugs
  on certain 32-bit Intel chips and defines a :c:macro:`X87_DOUBLE_ROUNDING`
  preprocessor definition.  No code currently uses this definition,
  but it's available if anyone wishes to use it.
  (Added by Mark Dickinson; :issue:`2937`.)

  :program:`configure` also now sets a :envvar:`LDCXXSHARED` Makefile
  variable for supporting C++ linking.  (Contributed by Arfrever
  Frehtes Taifersar Arahesis; :issue:`1222585`.)

* The build process now creates the necessary files for pkg-config
  support.  (Contributed by Clinton Roy; :issue:`3585`.)

* The build process now supports Subversion 1.7.  (Contributed by
  Arfrever Frehtes Taifersar Arahesis; :issue:`6094`.)


.. _whatsnew27-capsules:

Capsules
-------------------

Python 3.1 adds a new C datatype, :c:type:`PyCapsule`, for providing a
C API to an extension module.  A capsule is essentially the holder of
a C ``void *`` pointer, and is made available as a module attribute; for
example, the :mod:`socket` module's API is exposed as ``socket.CAPI``,
and :mod:`unicodedata` exposes ``ucnhash_CAPI``.  Other extensions
can import the module, access its dictionary to get the capsule
object, and then get the ``void *`` pointer, which will usually point
to an array of pointers to the module's various API functions.

There is an existing data type already used for this,
:c:type:`PyCObject`, but it doesn't provide type safety.  Evil code
written in pure Python could cause a segmentation fault by taking a
:c:type:`PyCObject` from module A and somehow substituting it for the
:c:type:`PyCObject` in module B.   Capsules know their own name,
and getting the pointer requires providing the name::

   void *vtable;

   if (!PyCapsule_IsValid(capsule, "mymodule.CAPI") {
           PyErr_SetString(PyExc_ValueError, "argument type invalid");
           return NULL;
   }

   vtable = PyCapsule_GetPointer(capsule, "mymodule.CAPI");

You are assured that ``vtable`` points to whatever you're expecting.
If a different capsule was passed in, :c:func:`PyCapsule_IsValid` would
detect the mismatched name and return false.  Refer to
:ref:`using-capsules` for more information on using these objects.

Python 2.7 now uses capsules internally to provide various
extension-module APIs, but the :c:func:`PyCObject_AsVoidPtr` was
modified to handle capsules, preserving compile-time compatibility
with the :c:type:`CObject` interface.  Use of
:c:func:`PyCObject_AsVoidPtr` will signal a
:exc:`PendingDeprecationWarning`, which is silent by default.

Implemented in Python 3.1 and backported to 2.7 by Larry Hastings;
discussed in :issue:`5630`.


.. ======================================================================

Port-Specific Changes: Windows
-----------------------------------

* The :mod:`msvcrt` module now contains some constants from
  the :file:`crtassem.h` header file:
  :data:`CRT_ASSEMBLY_VERSION`,
  :data:`VC_ASSEMBLY_PUBLICKEYTOKEN`,
  and :data:`LIBRARIES_ASSEMBLY_NAME_PREFIX`.
  (Contributed by David Cournapeau; :issue:`4365`.)

* The :mod:`_winreg` module for accessing the registry now implements
  the :func:`~_winreg.CreateKeyEx` and :func:`~_winreg.DeleteKeyEx`
  functions, extended versions of previously-supported functions that
  take several extra arguments.  The :func:`~_winreg.DisableReflectionKey`,
  :func:`~_winreg.EnableReflectionKey`, and :func:`~_winreg.QueryReflectionKey`
  were also tested and documented.
  (Implemented by Brian Curtin: :issue:`7347`.)

* The new :c:func:`_beginthreadex` API is used to start threads, and
  the native thread-local storage functions are now used.
  (Contributed by Kristján Valur Jónsson; :issue:`3582`.)

* The :func:`os.kill` function now works on Windows.  The signal value
  can be the constants :const:`CTRL_C_EVENT`,
  :const:`CTRL_BREAK_EVENT`, or any integer.  The first two constants
  will send :kbd:`Control-C` and :kbd:`Control-Break` keystroke events to
  subprocesses; any other value will use the :c:func:`TerminateProcess`
  API.  (Contributed by Miki Tebeka; :issue:`1220212`.)

* The :func:`os.listdir` function now correctly fails
  for an empty path.  (Fixed by Hirokazu Yamamoto; :issue:`5913`.)

* The :mod:`mimelib` module will now read the MIME database from
  the Windows registry when initializing.
  (Patch by Gabriel Genellina; :issue:`4969`.)

.. ======================================================================

Port-Specific Changes: Mac OS X
-----------------------------------

* The path ``/Library/Python/2.7/site-packages`` is now appended to
  ``sys.path``, in order to share added packages between the system
  installation and a user-installed copy of the same version.
  (Changed by Ronald Oussoren; :issue:`4865`.)

Port-Specific Changes: FreeBSD
-----------------------------------

* FreeBSD 7.1's :const:`SO_SETFIB` constant, used with
  :func:`~socket.getsockopt`/:func:`~socket.setsockopt` to select an
  alternate routing table, is now available in the :mod:`socket`
  module.  (Added by Kyle VanderBeek; :issue:`8235`.)