PEP 389: The argparse Module for Parsing Command Lines
======================================================

The :mod:`argparse` module for parsing command-line arguments was
added as a more powerful replacement for the
:mod:`optparse` module.

This means Python now supports three different modules for parsing
command-line arguments: :mod:`getopt`, :mod:`optparse`, and
:mod:`argparse`.  The :mod:`getopt` module closely resembles the C
library's :c:func:`getopt` function, so it remains useful if you're writing a
Python prototype that will eventually be rewritten in C.
:mod:`optparse` becomes redundant, but there are no plans to remove it
because there are many scripts still using it, and there's no
automated way to update these scripts.  (Making the :mod:`argparse`
API consistent with :mod:`optparse`'s interface was discussed but
rejected as too messy and difficult.)

In short, if you're writing a new script and don't need to worry
about compatibility with earlier versions of Python, use
:mod:`argparse` instead of :mod:`optparse`.

Here's an example::

    import argparse

    parser = argparse.ArgumentParser(description='Command-line example.')

    # Add optional switches
    parser.add_argument('-v', action='store_true', dest='is_verbose',
                        help='produce verbose output')
    parser.add_argument('-o', action='store', dest='output',
                        metavar='FILE',
                        help='direct output to FILE instead of stdout')
    parser.add_argument('-C', action='store', type=int, dest='context',
                        metavar='NUM', default=0,
                        help='display NUM lines of added context')

    # Allow any number of additional arguments.
    parser.add_argument(nargs='*', action='store', dest='inputs',
                        help='input filenames (default is stdin)')

    args = parser.parse_args()
    print args.__dict__

Unless you override it, :option:`-h` and :option:`--help` switches
are automatically added, and produce neatly formatted output::

    -> ./python.exe argparse-example.py --help
    usage: argparse-example.py [-h] [-v] [-o FILE] [-C NUM] [inputs [inputs ...]]

    Command-line example.

    positional arguments:
      inputs      input filenames (default is stdin)

    optional arguments:
      -h, --help  show this help message and exit
      -v          produce verbose output
      -o FILE     direct output to FILE instead of stdout
      -C NUM      display NUM lines of added context

As with :mod:`optparse`, the command-line switches and arguments
are returned as an object with attributes named by the *dest* parameters::

    -> ./python.exe argparse-example.py -v
    {'output': None,
     'is_verbose': True,
     'context': 0,
     'inputs': []}

    -> ./python.exe argparse-example.py -v -o /tmp/output -C 4 file1 file2
    {'output': '/tmp/output',
     'is_verbose': True,
     'context': 4,
     'inputs': ['file1', 'file2']}

:mod:`argparse` has much fancier validation than :mod:`optparse`; you
can specify an exact number of arguments as an integer, 0 or more
arguments by passing ``'*'``, 1 or more by passing ``'+'``, or an
optional argument with ``'?'``.  A top-level parser can contain
sub-parsers to define subcommands that have different sets of
switches, as in ``svn commit``, ``svn checkout``, etc.  You can
specify an argument's type as :class:`~argparse.FileType`, which will
automatically open files for you and understands that ``'-'`` means
standard input or output.

.. seealso::

   :mod:`argparse` documentation
     The documentation page of the argparse module.

   :ref:`argparse-from-optparse`
     Part of the Python documentation, describing how to convert
     code that uses :mod:`optparse`.

   :pep:`389` - argparse - New Command Line Parsing Module
     PEP written and implemented by Steven Bethard.