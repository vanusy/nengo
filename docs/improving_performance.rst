Improving performance
=====================

Once you start creating larger models, Nengo might become a little bit slow.
This section walks you through different things that might help you to get the
best performance out of Nengo.


In a nutshell
-------------

To improve the build times
^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Set a seed on the top level model to enable caching::

    nengo.Network(seed=1)

2. Disable the operator graph optimizer::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].enabled = False
        with nengo.Simulator(model) as sim:
            sim.run(...)

3. Reduce the number of neurons in very large ensembles, or consider using the
   :class:`nengo.utils.least_squares_solvers.RandomizedSVD` solver.

To improve the simulation run time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Enable the operator graph optimizer::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].enabled = True
        with nengo.Simulator(model) as sim:
            sim.run(...)

2. Consider switching to the `nengo_ocl <https://github.com/nengo/nengo_ocl>`_
   backend if you have a powerful GPU.

To lower peak memory consumption
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Disable the operator graph optimizer::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].enabled = False
        with nengo.Simulator(model) as sim:
            sim.run(...)

2. Reduce the number of neurons in very large ensembles, consider replacing
   them with multiple smaller ensembles (:class:`nengo.networks.EnsembleArray`
   is useful for that).

3. Reduce the number of probes or their sampling intervals (with the
   ``sample_every=`` argument).


Build and run time performance
------------------------------

There are two main things determining how long your model takes to run: the
build time and the (simulation) run time. The former is the time required to
translate the model description into the actual neural network that gets
simulated and is what happens when you create the simulator with
``nengo.Simulator(model)``. The latter is how long it takes to simulate this
network for the desired amount of simulation time and is what happens during
the ``sim.run(...)`` calls.

Some things will influence solely one of these variables, but other things will
reduce one of the variables at the cost of increasing the other variables.
Thus, they are a trade-off and the best point therein depends on your exact use
case.

While the run time is usually the most important aspect, sometimes the memory
consumption can be the main problem instead.


Decoder caching
---------------

*Influences build time.*

A significant amount of time during the build is spent on finding the NEF
decoders. Because of that, it is possible to cache the decoders. The first
build of a model will still take about the same time (technically a bit longer,
because the computed decoders will be stored), but all subsequent builds of the
same model can just load the stored decoders and will be faster. This will also
work to some degree if changes to the model are made, but of course only for
the unchanged parts.

To enable the decoder caching where possible, it is only required to set a seed
on the model like so::

    with nengo.Networks(seed=1) as model:

There are :ref:`a few configuration options <nengorc-decoder_cache>` for more
advanced control of the cache. The most important might be the possibility to
control the path where the cache files are stored as on high performance
clusters certain file systems might provide a better performance.


Operator graph optimizer
------------------------

*Influences build time, simulation time, and memory consumption.*

On modern processors huge speed improvements are possible if memory access
occurs in a linear manner. By default Nengo optimizes its internal data
structures (the "operator graph") to achieve this. However, this can increase
build times significantly and in some cases it can be better to turn this
optimization off to have a faster build at the cost of slower simulation run
times. To turn the optimizer off the simulator has to be created with a certain
configuration applied as shown here::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].enabled = True
        with nengo.Simulator(model) as sim:
            sim.run(...)

An intermediary solution is possible by limiting the number of optimization
passes::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].max_passes = 5
        with nengo.Simulator(model) as sim:
            sim.run(...)

It is also possible to set these setting for the optimizer globally in
:ref:`the configuration file <nengorc-OpMergeOptimizer>`.

Another situation where it is helpful to disable the optimizer is when the peak
memory usage is too high. The optimizer can require up to twice as much memory
as would be required without the optimizer. Note that limiting the optimization
passes has no major influence on the memory consumption.


nengo_ocl
---------

*Improves simulation times.*

If you have a powerful GPU, you have the option to switch to the `nengo_ocl
<https://github.com/nengo/nengo_ocl>`_ backend. It will utilize the GPU instead
of CPU which is much more optimized for the sort of calculations done by Nengo.
The build times with nengo_ocl are usually not much longer than with core
Nengo, but the run time can be substantially faster.


Adjusting model structure
-------------------------

*Influences build times, simulation times, and memory consumption.*

Some aspects of the model structure, apart from the pure size of the model,
influence performance aspects. Ensembles with many neurons will take a long
time to build and consume a lot memory during the process. Sometimes it is
feasible to split such ensembles into multiple smaller ensembles (the
:class:`nengo.networks.EnsembleArray` is helpful for that). Alternatively,
using the :class:`nengo.utils.least_squares_solvers.RandomizedSVD` can at least
reduce the build time.

But be aware that many small ensembles will take longer to simulate if the
operator graph optimizer (see above) is deactivated.


Limiting probed data
--------------------

*Influences mainly memory consumption.*

Obviously, all data that gets probed in the model has to be stored in memory.
Depending on how long the simulation runs for and how many things get probed
this might contribute a significant amount to the memory consumption. By
reducing the number of probed object the memory consumption can be reduced. An
alternative is to not record a value for every time step. Probes accept
a ``sample_every=`` argument to reduce the number of recorded samples.
