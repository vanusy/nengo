Nengo configuration
===================

Certain features of Nengo can be configured with a configuration file. The
configuration settings will be read from the following files with precedence to
those listed first:

1. ``nengorc`` in the current directory. This is intended to allow for project
   specific settings without hard coding them in the model description.
2. An operating system specific file in the user's home directory.

   * Windows: ``%userprofile%\.nengo\nengorc``
   * Other (OS X, Linux): ``~/.config/nengo/nengorc``

3. ``INSTALL/nengo-data/nengorc`` (where `INSTALL` is the installation directory
    of the Nengo package)

The RC file is divided into sections by lines containing the section name
in brackets, i.e. ``[section]``. A setting is set by giving the name followed
by a ``:`` or ``=`` and the value. All lines starting with ``#`` or ``;`` are
comments.

Example
-------
This example demonstrates how to set settings in an RC file::

    [decoder_cache]
    size: 536870912  # setting the decoder cache size to 512MiB.

Configuration options
---------------------

.. _nengorc-decoder_cache:

[decoder_cache]
^^^^^^^^^^^^^^^

This section controls the decoder cache.

enabled (bool)
""""""""""""""

Default: ``True``

Controls whether the decoder cache is enabled.

path (str)
""""""""""

Default: system dependent default Nengo cache directory

Path where the decoder cache stores its files.

readonly (bool)
"""""""""""""""

Default: ``False``

Sets the cache to read-only mode. This allows existing cached decoders to be
read, but no new decoders will be cached.

size (size)
"""""""""""

Default: ``512 MB``

Maximum size of the cache. Accepts common human readable units like ``512 MB``.

[exceptions]
^^^^^^^^^^^^

This section controls exceptions raised by Nengo.

simplified (bool)
"""""""""""""""""

Default: ``True``

Whether to raise simplified exceptions. These usually provide a better
indication of the error in the model definition. But they hide details about
where the exception occurred within the Nengo internals. Thus, Nengo developer
might want to turn simplified exceptions off.

.. _nengorc-OpMergeOptimizer:

[OpMergeOptimizer]
^^^^^^^^^^^^^^^^^^

This section controls the operator graph optimizer.

enabled (bool)
""""""""""""""

Default: ``True``

Whether to enable the operator graph optimizer.

max_passes (int)
""""""""""""""""

Default: not set

Maximum number of optimization passes to perform.

[progress]
^^^^^^^^^^

This section controls the progress bars used when running the simulator.

progress_bar (bool or str)
""""""""""""""""""""""""""

Default: ``auto``

Controls the progress bar used. The default of ``auto`` will automatically
display a progress bar when the estimated simulation run time exceeds one
second. Boolean values equivalent to ``True`` will have the same effect.
Boolean values equivalent to ``False`` or the string ``none`` will disable the
progress bar. Any other string will be interpreted as a module and class name
combined with a ``.`` and Nengo will try to load this class as progress bar.

updater (str)
"""""""""""""

Default: ``auto``

Controls the class scheduling the progress bar updates. The default of ``auto``
will use a heuristic to select the appropriate updater. Any other string will
be interpreted as a module and class name
combined with a ``.`` and Nengo will try to load this class as updater.
