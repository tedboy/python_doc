.. ======================================================================


.. _py27-maintenance-enhancements:

New Features Added to Python 2.7 Maintenance Releases
=====================================================

New features may be added to Python 2.7 maintenance releases when the
situation genuinely calls for it. Any such additions must go through
the Python Enhancement Proposal process, and make a compelling case for why
they can't be adequately addressed by either adding the new feature solely to
Python 3, or else by publishing it on the Python Package Index.

In addition to the specific proposals listed below, there is a general
exemption allowing new ``-3`` warnings to be added in any Python 2.7
maintenance release.


PEP 434: IDLE Enhancement Exception for All Branches
----------------------------------------------------

:pep:`434` describes a general exemption for changes made to the IDLE
development environment shipped along with Python. This exemption makes it
possible for the IDLE developers to provide a more consistent user
experience across all supported versions of Python 2 and 3.

For details of any IDLE changes, refer to the NEWS file for the specific
release.


PEP 466: Network Security Enhancements for Python 2.7
-----------------------------------------------------

:pep:`466` describes a number of network security enhancement proposals
that have been approved for inclusion in Python 2.7 maintenance releases,
with the first of those changes appearing in the Python 2.7.7 release.

:pep:`466` related features added in Python 2.7.7:

* :func:`hmac.compare_digest` was backported from Python 3 to make a timing
  attack resistant comparison operation available to Python 2 applications.
  (Contributed by Alex Gaynor; :issue:`21306`.)

* OpenSSL 1.0.1g was upgraded in the official Windows installers published on
  python.org. (Contributed by Zachary Ware; :issue:`21462`.)

:pep:`466` related features added in Python 2.7.8:

* :func:`hashlib.pbkdf2_hmac` was backported from Python 3 to make a hashing
  algorithm suitable for secure password storage broadly available to Python
  2 applications. (Contributed by Alex Gaynor; :issue:`21304`.)

* OpenSSL 1.0.1h was upgraded for the official Windows installers published on
  python.org. (contributed by Zachary Ware in :issue:`21671` for CVE-2014-0224)

:pep:`466` related features added in Python 2.7.9:

* Most of Python 3.4's :mod:`ssl` module was backported. This means :mod:`ssl`
  now supports Server Name Indication, TLS1.x settings, access to the platform
  certificate store, the :class:`~ssl.SSLContext` class, and other
  features. (Contributed by Alex Gaynor and David Reid; :issue:`21308`.)

  Refer to the "Version added: 2.7.9" notes in the module documentation for
  specific details.

* :func:`os.urandom` was changed to cache a file descriptor to ``/dev/urandom``
  instead of reopening ``/dev/urandom`` on every call. (Contributed by Alex
  Gaynor; :issue:`21305`.)

* :data:`hashlib.algorithms_guaranteed` and
  :data:`hashlib.algorithms_available` were backported from Python 3 to make
  it easier for Python 2 applications to select the strongest available hash
  algorithm. (Contributed by Alex Gaynor in :issue:`21307`)


PEP 477: Backport ensurepip (PEP 453) to Python 2.7
---------------------------------------------------

:pep:`477` approves the inclusion of the :pep:`453` ensurepip module and the
improved documentation that was enabled by it in the Python 2.7 maintenance
releases, appearing first in the Python 2.7.9 release.


Bootstrapping pip By Default
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The new :mod:`ensurepip` module (defined in :pep:`453`) provides a standard
cross-platform mechanism to bootstrap the pip installer into Python
installations. The version of ``pip`` included with Python 2.7.9 is ``pip``
1.5.6, and future 2.7.x maintenance releases will update the bundled version to
the latest version of ``pip`` that is available at the time of creating the
release candidate.

By default, the commands ``pip``, ``pipX`` and ``pipX.Y`` will be installed on
all platforms (where X.Y stands for the version of the Python installation),
along with the ``pip`` Python package and its dependencies.

For CPython :ref:`source builds on POSIX systems <building-python-on-unix>`,
the ``make install`` and ``make altinstall`` commands do not bootstrap ``pip``
by default.  This behaviour can be controlled through configure options, and
overridden through Makefile options.

On Windows and Mac OS X, the CPython installers now default to installing
``pip`` along with CPython itself (users may opt out of installing it
during the installation process). Window users will need to opt in to the
automatic ``PATH`` modifications to have ``pip`` available from the command
line by default, otherwise it can still be accessed through the Python
launcher for Windows as ``py -m pip``.

As `discussed in the PEP`__, platform packagers may choose not to install
these commands by default, as long as, when invoked, they provide clear and
simple directions on how to install them on that platform (usually using
the system package manager).

__ https://www.python.org/dev/peps/pep-0477/#disabling-ensurepip-by-downstream-distributors


Documentation Changes
~~~~~~~~~~~~~~~~~~~~~

As part of this change, the :ref:`installing-index` and
:ref:`distributing-index` sections of the documentation have been
completely redesigned as short getting started and FAQ documents. Most
packaging documentation has now been moved out to the Python Packaging
Authority maintained `Python Packaging User Guide
<http://packaging.python.org>`__ and the documentation of the individual
projects.

However, as this migration is currently still incomplete, the legacy
versions of those guides remaining available as :ref:`install-index`
and :ref:`distutils-index`.

.. seealso::

   :pep:`453` -- Explicit bootstrapping of pip in Python installations
      PEP written by Donald Stufft and Nick Coghlan, implemented by
      Donald Stufft, Nick Coghlan, Martin von Löwis and Ned Deily.

PEP 476: Enabling certificate verification by default for stdlib http clients
-----------------------------------------------------------------------------

:pep:`476` updated :mod:`httplib` and modules which use it, such as
:mod:`urllib2` and :mod:`xmlrpclib`, to now verify that the server
presents a certificate which is signed by a Certificate Authority in the
platform trust store and whose hostname matches the hostname being requested
by default, significantly improving security for many applications. This
change was made in the Python 2.7.9 release.

For applications which require the old previous behavior, they can pass an
alternate context::

    import urllib2
    import ssl

    # This disables all verification
    context = ssl._create_unverified_context()

    # This allows using a specific certificate for the host, which doesn't need
    # to be in the trust store
    context = ssl.create_default_context(cafile="/path/to/file.crt")

    urllib2.urlopen("https://invalid-cert", context=context)


PEP 493: HTTPS verification migration tools for Python 2.7
----------------------------------------------------------

:pep:`493` provides additional migration tools to support a more incremental
infrastructure upgrade process for environments containing applications and
services relying on the historically permissive processing of server
certificates when establishing client HTTPS connections.  These additions were
made in the Python 2.7.12 release.

These tools are intended for use in cases where affected applications and
services can't be modified to explicitly pass a more permissive SSL context
when establishing the connection.

For applications and services which can't be modified at all, the new
``PYTHONHTTPSVERIFY`` environment variable may be set to ``0`` to revert an
entire Python process back to the default permissive behaviour of Python 2.7.8
and earlier.

For cases where the connection establishment code can't be modified, but the
overall application can be, the new :func:`ssl._https_verify_certificates`
function can be used to adjust the default behaviour at runtime.