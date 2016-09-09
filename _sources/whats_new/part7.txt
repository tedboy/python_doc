
PEP 391: Dictionary-Based Configuration For Logging
====================================================

The :mod:`logging` module is very flexible; applications can define
a tree of logging subsystems, and each logger in this tree can filter
out certain messages, format them differently, and direct messages to
a varying number of handlers.

All this flexibility can require a lot of configuration.  You can
write Python statements to create objects and set their properties,
but a complex set-up requires verbose but boring code.
:mod:`logging` also supports a :func:`~logging.fileConfig`
function that parses a file, but the file format doesn't support
configuring filters, and it's messier to generate programmatically.

Python 2.7 adds a :func:`~logging.dictConfig` function that
uses a dictionary to configure logging.  There are many ways to
produce a dictionary from different sources: construct one with code;
parse a file containing JSON; or use a YAML parsing library if one is
installed.  For more information see :ref:`logging-config-api`.

The following example configures two loggers, the root logger and a
logger named "network".  Messages sent to the root logger will be
sent to the system log using the syslog protocol, and messages
to the "network" logger will be written to a :file:`network.log` file
that will be rotated once the log reaches 1MB.

::

    import logging
    import logging.config

    configdict = {
     'version': 1,    # Configuration schema in use; must be 1 for now
     'formatters': {
         'standard': {
             'format': ('%(asctime)s %(name)-15s '
                        '%(levelname)-8s %(message)s')}},

     'handlers': {'netlog': {'backupCount': 10,
                         'class': 'logging.handlers.RotatingFileHandler',
                         'filename': '/logs/network.log',
                         'formatter': 'standard',
                         'level': 'INFO',
                         'maxBytes': 1000000},
                  'syslog': {'class': 'logging.handlers.SysLogHandler',
                             'formatter': 'standard',
                             'level': 'ERROR'}},

     # Specify all the subordinate loggers
     'loggers': {
                 'network': {
                             'handlers': ['netlog']
                 }
     },
     # Specify properties of the root logger
     'root': {
              'handlers': ['syslog']
     },
    }

    # Set up configuration
    logging.config.dictConfig(configdict)

    # As an example, log two error messages
    logger = logging.getLogger('/')
    logger.error('Database not found')

    netlogger = logging.getLogger('network')
    netlogger.error('Connection failed')

Three smaller enhancements to the :mod:`logging` module, all
implemented by Vinay Sajip, are:

.. rev79293

* The :class:`~logging.handlers.SysLogHandler` class now supports
  syslogging over TCP.  The constructor has a *socktype* parameter
  giving the type of socket to use, either :const:`socket.SOCK_DGRAM`
  for UDP or :const:`socket.SOCK_STREAM` for TCP.  The default
  protocol remains UDP.

* :class:`~logging.Logger` instances gained a :meth:`~logging.Logger.getChild`
  method that retrieves a descendant logger using a relative path.
  For example, once you retrieve a logger by doing ``log = getLogger('app')``,
  calling ``log.getChild('network.listen')`` is equivalent to
  ``getLogger('app.network.listen')``.

* The :class:`~logging.LoggerAdapter` class gained a
  :meth:`~logging.LoggerAdapter.isEnabledFor` method that takes a
  *level* and returns whether the underlying logger would
  process a message of that level of importance.

.. XXX: Logger objects don't have a class declaration so the link don't work

.. seealso::

   :pep:`391` - Dictionary-Based Configuration For Logging
     PEP written and implemented by Vinay Sajip.