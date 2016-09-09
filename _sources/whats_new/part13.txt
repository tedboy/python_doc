
Other Changes and Fixes
=======================

* Two benchmark scripts, :file:`iobench` and :file:`ccbench`, were
  added to the :file:`Tools` directory.  :file:`iobench` measures the
  speed of the built-in file I/O objects returned by :func:`open`
  while performing various operations, and :file:`ccbench` is a
  concurrency benchmark that tries to measure computing throughput,
  thread switching latency, and IO processing bandwidth when
  performing several tasks using a varying number of threads.

* The :file:`Tools/i18n/msgfmt.py` script now understands plural
  forms in :file:`.po` files.  (Fixed by Martin von Löwis;
  :issue:`5464`.)

* When importing a module from a :file:`.pyc` or :file:`.pyo` file
  with an existing :file:`.py` counterpart, the :attr:`co_filename`
  attributes of the resulting code objects are overwritten when the
  original filename is obsolete.  This can happen if the file has been
  renamed, moved, or is accessed through different paths.  (Patch by
  Ziga Seilnacht and Jean-Paul Calderone; :issue:`1180193`.)

* The :file:`regrtest.py` script now takes a :option:`--randseed=`
  switch that takes an integer that will be used as the random seed
  for the :option:`-r` option that executes tests in random order.
  The :option:`-r` option also reports the seed that was used
  (Added by Collin Winter.)

* Another :file:`regrtest.py` switch is :option:`-j`, which
  takes an integer specifying how many tests run in parallel. This
  allows reducing the total runtime on multi-core machines.
  This option is compatible with several other options, including the
  :option:`!-R` switch which is known to produce long runtimes.
  (Added by Antoine Pitrou, :issue:`6152`.)  This can also be used
  with a new :option:`-F` switch that runs selected tests in a loop
  until they fail.  (Added by Antoine Pitrou; :issue:`7312`.)

* When executed as a script, the :file:`py_compile.py` module now
  accepts ``'-'`` as an argument, which will read standard input for
  the list of filenames to be compiled.  (Contributed by Piotr
  Ożarowski; :issue:`8233`.)