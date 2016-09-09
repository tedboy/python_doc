Changes to the Handling of Deprecation Warnings
===============================================

For Python 2.7, a policy decision was made to silence warnings only of
interest to developers by default.  :exc:`DeprecationWarning` and its
descendants are now ignored unless otherwise requested, preventing
users from seeing warnings triggered by an application.  This change
was also made in the branch that became Python 3.2. (Discussed
on stdlib-sig and carried out in :issue:`7319`.)

In previous releases, :exc:`DeprecationWarning` messages were
enabled by default, providing Python developers with a clear
indication of where their code may break in a future major version
of Python.

However, there are increasingly many users of Python-based
applications who are not directly involved in the development of
those applications.  :exc:`DeprecationWarning` messages are
irrelevant to such users, making them worry about an application
that's actually working correctly and burdening application developers
with responding to these concerns.

You can re-enable display of :exc:`DeprecationWarning` messages by
running Python with the :option:`-Wdefault <-W>` (short form:
:option:`-Wd <-W>`) switch, or by setting the :envvar:`PYTHONWARNINGS`
environment variable to ``"default"`` (or ``"d"``) before running
Python.  Python code can also re-enable them
by calling ``warnings.simplefilter('default')``.

The ``unittest`` module also automatically reenables deprecation warnings
when running tests.