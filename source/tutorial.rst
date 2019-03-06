Tutorial (pysimfs)
==================

This section introduces the use of pysimfs for more complex simulation tasks.

Building a pipeline
^^^^^^^^^^^^^^^^^^^

You have seen the basic structure of a SiMFS-Tk simulation in the
:ref:`introduction-label` and :ref:`quickstart-label` sections. So far the
simulations have been a step-by-step procedure with a lot of intermediate file
clobber. If you are familiar with the UNIX file system, you might have guessed,
that you can avoid the large amount of data on disk by using pipes. Pipes are
file-like obejects that allow concurrent writing and reading to pass
information from one process to the other.

The manual approach would be to configure two processes like ``simfs_dif`` and
``simfs_exi`` to write to and read from the same file called ``coord_pipe``.
Then you create a named pipe of this name and start the processes in parallel:

.. code-block:: bash

   $ mkfifo coord_pipe
   $ simfs_dif < dif.json > dif.log.json & simfs_exi < exi.json > exi.log.json

The two processes run simultaneously passing coordinates from ``simfs_dif`` to
``simfs_exi`` continuously without using diskspace for coordinate data. Only
the resulting excitaion data file will be created (unless there is more
piping).

This approach is pretty powerful in that it saves diskspace, avoids
unnecessarily large intermediate data files and allows for better usage of
multiple cores. It is however quite cumbersome to manage the named pipes
yourself. As soon as one connection within a complex pipeline is not set
correctly, the whole run blocks and leaves partial processes and files behind
-- a *huge* mess! Therefore we have a little python package that helps you set
up and run your SiMFS-components, while managing the required pipes for
parallel execution for you! The next section will show you how to use pysimfs
to run a similar simulation like the basic commandline example from python.

.. Warning::

   Pysimfs is a python package designed to be used in python scripts and
   jupyter-notebooks This tutorial requires a basic understanding of python,
   numpy and matplotlib.

Pysimfs startup
^^^^^^^^^^^^^^^

Once you have the pysimfs package installed, you can import it. It will try to
locate the SiMFS-Tk executables in its basepath. If there is an error, check
the location of your SiMFS-core installation and adapte the path in the pysimfs
package's ``__init__.py``.

The import should look like this:

.. code-block:: python

   >>> import pysimfs as simfs
       All simfs components found in /opt/SiMFS-Tk/SiMFS-core/build/src/components/.

Configuring components
^^^^^^^^^^^^^^^^^^^^^^

Each ``simfs_xxx`` executable has a pysimfs object to control it. To create a
diffusion component, create a pysimfs.Diffusion object like this:

.. code-block:: python

   >>> dif = simfs.Diffusion()
   >>> dif.params
       {'collision_output': '__collisions__',
        'coordinate_output': '__coordinates__',
        'diffusion_coefficient': 1e-10,
        'experiment_time': 1.0,
        'half_height': 1e-06,
        'increment': 1e-07,
        'radius': 1e-06,
        'seed': 477755232}

So far no simulation ran. You can inspect (or change) the current paramters of
the component in its ``params`` dictionary. Every component constructor takes
keyword argument that map exactly to the toplevel json members of the manages
SiMFS-somponent. To create the diffusion example again:

.. code-block:: python

   >>> import os
   >>> dif = simfs.Diffusion(
   ...    collision_output=os.devnull, 
   ...    coordinate_output='coords.dat'
   ...    diffusion_coefficient=4.35e-10
   ... ) 
   >>> dif.params
       {'collision_output': '/dev/null',
        'coordinate_output': 'coords.dat',
        'diffusion_coefficient': 4.35e-10,
        'experiment_time': 1.0,
        'half_height': 1e-06,
        'increment': 1e-07,
        'radius': 1e-06,
        'seed': 2235270572}

This is simple a convenient wrapper around the process of getting the defaults
with the ``list`` option, save it to a file, edit it and reapply the changes.


Running a simulation
^^^^^^^^^^^^^^^^^^^^

So far no simulation run. To run the single component, create a ``Simulation``
and add the configured component to it:

.. code-block:: python

   >>> sim = simfs.Simulation()
   >>> sim.add(dif)

There might be some messages about already existing folders, when you run this
multiple times. You can ignore this for now. To start the simulation simply
call ``run`` on the simulation:

.. code-block:: python

   >>> log = sim.run()
       started /opt/SiMFS-Tk/SiMFS-core/build/src/components/mol/simfs_dif
       Simulation completed after 1.86 seconds.

You get list of all paramter logs and error messages of all components. The
output reports which executables were started and finally how long the
simulatio took. The file ``coords.dat`` will be created in your current
direcory. To quickly load the simulation results into your python session, use
the ``get_results`` function of the simulation:

.. code-block:: python

   >>> res = sim.get_results()

``res`` now contains a dictionary of numpy arrays that contain the result data.

Chaining components
^^^^^^^^^^^^^^^^^^^

We could now continue to run the next component in a new simulation, reading
the new ``coords.dat`` file.  However pysimfs is capable of handling more than
one component per simulation. Take a look at this new simulation:

.. code-block:: python

   >>> sim = simfs.Simulation()
   >>> sim.add(
           simfs.Diffusion(
               coordinate_output='coords',
               collision_output=os.devnull, 
               diffusion_coefficient=4.35e-10
           )
       ) 
   >>> sim.add(
           simfs.Excitation(
                input='coords', 
                output='exi.dat'
           )
       )
   >>> log = sim.run()
       started /opt/SiMFS-Tk/SiMFS-core/build/src/components/fcs/simfs_exi
       started /opt/SiMFS-Tk/SiMFS-core/build/src/components/mol/simfs_dif
       Simulation completed after 2.93 seconds.
   >>> res = sim.get_results()

Two components were started simultaneously. Pysimfs detected that ``dif`` and
``exi`` share an output/input pair and created the requirede pipe for us. As a
result only the ``exi.dat`` file is generated. The intermediate ``coords`` are
gone and don't occupy any disk space.

Utilities
^^^^^^^^^

With pysimfs it is much easier to design and run more complex simulations.
Checkout the example notebooks for more complex simulations.  There are some
more helper functions included in pysimfs that make the simulatio process
easier:

Default parameters
------------------

If you work in a jupyter notebook context, it can be helpful to have explicit
parameter dictionaries in some cases. To quickly load a default parameter set
from a component, you can use the ``simfs_default`` magic:

.. code-block:: python

   %simfs_default dif

As a result, a default paramter dictionary is written to the current cell.

.. Warning::

   Note that the magic overwrites the content of the current cell.

You can pass a paramter dictionary to a component by using ``**kwargs``:

.. code-block:: python

   dif = simfs.Diffusion(**dif_params)

Context manager
---------------

Since pysimfs interacts a lot with the filesystem, there can be some
accumulation of old intermediate files like named pipes.  Pysimfs uses
temporary folders to manage its files, but does not clean them up by default.

An easy way to manage the lifetime of these files is to use the context manager
capability of ``Simulation``:

.. code-block:: python

   import os
   from pysimfs import *
   with Simulation() as S:
       S.add(Diffusion(coordinate_output='coords', collision_output=os.devnull)
       S.add(Excitation(input='coords', output='exi.dat')
       log = S.run()
       res = S.get_results()

This idiom wraps the simulation setup and run into a context that handles the
cleanup of temporary files when done.

Grid data
---------

The ``simfs_img`` and ``simfs_pre`` component produce numeric data on a grid in
a binary format. Pysimfs includes a loader for these files.

.. code-block:: python

   img = simfs.util.GridData('image.dat')

Checkout the notebooks for further usage examples.
