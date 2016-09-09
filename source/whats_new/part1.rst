.. _whatsnew27-python31:

The Future for Python 2.x
=========================

Python 2.7 is the last major release in the 2.x series, as the Python
maintainers have shifted the focus of their new feature development efforts
to the Python 3.x series. This means that while Python 2 continues to
receive bug fixes, and to be updated to build correctly on new hardware and
versions of supported operated systems, there will be no new full feature
releases for the language or standard library.

However, while there is a large common subset between Python 2.7 and Python
3, and many of the changes involved in migrating to that common subset, or
directly to Python 3, can be safely automated, some other changes (notably
those associated with Unicode handling) may require careful consideration,
and preferably robust automated regression test suites, to migrate
effectively.

This means that Python 2.7 will remain in place for a long time, providing a
stable and supported base platform for production systems that have not yet
been ported to Python 3. The full expected lifecycle of the Python 2.7
series is detailed in :pep:`373`.

Some key consequences of the long-term significance of 2.7 are:

* As noted above, the 2.7 release has a much longer period of maintenance
  when compared to earlier 2.x versions. Python 2.7 is currently expected to
  remain supported by the core development team (receiving security updates
  and other bug fixes) until at least 2020 (10 years after its initial
  release, compared to the more typical support period of 18-24 months).

* As the Python 2.7 standard library ages, making effective use of the
  Python Package Index (either directly or via a redistributor) becomes
  more important for Python 2 users. In addition to a wide variety of third
  party packages for various tasks, the available packages include backports
  of new modules and features from the Python 3 standard library that are
  compatible with Python 2, as well as various tools and libraries that can
  make it easier to migrate to Python 3. The `Python Packaging User Guide
  <https://packaging.python.org>`__ provides guidance on downloading and
  installing software from the Python Package Index.

* While the preferred approach to enhancing Python 2 is now the publication
  of new packages on the Python Package Index, this approach doesn't
  necessarily work in all cases, especially those related to network
  security. In exceptional cases that cannot be handled adequately by
  publishing new or updated packages on PyPI, the Python Enhancement
  Proposal process may be used to make the case for adding new features
  directly to the Python 2 standard library. Any such additions, and the
  maintenance releases where they were added, will be noted in the
  :ref:`py27-maintenance-enhancements` section below.

For projects wishing to migrate from Python 2 to Python 3, or for library
and framework developers wishing to support users on both Python 2 and
Python 3, there are a variety of tools and guides available to help decide
on a suitable approach and manage some of the technical details involved.
The recommended starting point is the :ref:`pyporting-howto` HOWTO guide.