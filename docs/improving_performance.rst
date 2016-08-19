Improving performance
=====================

Once you start creating larger models, Nengo might become a little bit slow.
This section walks you through different things that might help you to get the
best performance out of Nengo.

.. default-role:: obj

In a nutshell
-------------

To improve build times
^^^^^^^^^^^^^^^^^^^^^^

1. Set a seed on the top level model to enable caching::

    nengo.Network(seed=1)

2. Disable the operator graph optimizer::

    with nengo.Config(nengo.builder.optimizer.OpMergeOptimizer) as cfg:
        cfg[nengo.builder.optimizer.OpMergeOptimizer].enabled = False
        with nengo.Simulator(model) as sim:
            sim.run(...)

3. Reduce the number of neurons in very large ensembles, or consider using the
   `.RandomizedSVD` solver.

To improve run times
^^^^^^^^^^^^^^^^^^^^

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
   them with multiple smaller ensembles (`.EnsembleArray` is useful for that).

3. Reduce the number of probes or their sampling intervals
   (with the ``sample_every=`` argument).

Build and run time performance
------------------------------

The two main determiners of how long your model takes to run are the
build time and the (simulation) run time. The former is the time required to
translate the model description into the actual neural network that gets
simulated and is what happens when you create the simulator with
``nengo.Simulator(model)``. The latter is how long it takes to simulate this
network for the desired amount of simulation time and is what happens during
the ``sim.run(...)`` calls.

Some things will influence one of these variables, but other things will
reduce one variable at the cost of increasing another.
While the run time is usually the most important variable,
sometimes the memory consumption can be the main problem.

Getting the best performance for your model depends on your model
and your computing environment.

Decoder caching
---------------

*Influences build time.*

A significant amount of time during the build is spent on finding the NEF
decoders. Because of that, it is possible to cache the decoders. The first
build of a model will still take about the same time (technically a bit longer
because the computed decoders will be stored), but all subsequent builds of the
same model can load the stored decoders and will be faster.

To enable the decoder caching, set a seed on the network like so::

    with nengo.Networks(seed=1) as model:

There are :ref:`a few configuration options <nengorc-decoder_cache>` for more
advanced control of the cache. The most important might be the possibility to
control the path where the cache files are stored. On high performance
clusters, certain file systems might provide a better performance.

Operator graph optimizer
------------------------

*Influences build time, simulation time, and memory consumption.*

On modern processors, huge speed improvements are possible if memory access
occurs in a linear manner. By default, Nengo optimizes its internal data
structures (the "operator graph") to achieve this. However, this can increase
build times significantly and in some cases it can be better to turn this
optimization off to speed up the build at the cost of slowing simulation run
times. To turn the optimizer off, the simulator has to be created with
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

It is also possible to set the optimizer settings globally in
:ref:`the configuration file <nengorc-OpMergeOptimizer>`.

Another situation where it is helpful to disable the optimizer is when the peak
memory usage is too high. The optimizer can use up to twice as much memory
as would be required without the optimizer. Note that limiting the optimization
passes has no major influence on memory consumption.

nengo_ocl
---------

*Improves simulation times.*

If you have a powerful GPU, you have the option to switch to the `nengo_ocl
<https://github.com/nengo/nengo_ocl>`_ backend. It will utilize your GPU,
which is much more optimized for the sorts of calculations done by Nengo.
The build times with ``nengo_ocl`` are usually not much longer than with
Nengo, but the run time can be substantially faster.

Adjusting model structure
-------------------------

*Influences build times, simulation times, and memory consumption.*

Some aspects of the model structure, apart from the size of the model,
influence performance aspects. Ensembles with many neurons will take a long
time to build and consume a lot of memory during the process. Sometimes it is
feasible to split such ensembles into multiple smaller ensembles (the
`.EnsembleArray` is helpful for that). Alternatively, using the
`.RandomizedSVD` decoder solver can at least reduce the build time.

But be aware that many small ensembles will take longer to simulate if the
operator graph optimizer (see above) is deactivated.

Limiting probed data
--------------------

*Influences mainly memory consumption.*

All data that gets probed in the model has to be stored in memory.
Depending on how long the simulation runs and how many things are probed,
this might consume a significant amount of memory. By reducing the number
of probed objects, the memory consumption can be reduced. An alternative
is to not record a value for every time step. Probes accept a
``sample_every=`` argument to reduce the number of recorded samples.
