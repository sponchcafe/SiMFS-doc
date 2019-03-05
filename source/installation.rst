Installation
============

In principal SiMFS-Tk can be used on Linux, Mac OS and Windows, although a UNIX
system that supports named pipes is recommended. 

.. note:: 

   The installation process will be improved in the future to integrate better
   with the python driver. Consider this the developer documentation for
   building the project from source.

Prerquisites
------------

Before using SiMFS-Tk, make sure you have the following tools:

- git
- a c++ compiler (g++ or clang)
- cmake
- python3 (if you want to use the python driver)

Linux
^^^^^

Use your system package manager (e.g. ``apt``) to install the dependencies.

Mac OSX
^^^^^^^

We recommend homebrew_ to install software dependencies on OSX.

Windows
^^^^^^^

   
.. note::
   :class: strike

   We recommend Cygwin to run SiMFS-Tk on Windows. Make sure you select the
   required packages during the installation Cygwin.

.. note::
   
   We recommend the Windows subsystem for linux (WSL_) for Windows users.
   Follow the installation steps for Linux.


Standard procedure
------------------

The standard procedure after installing all dependencies is cloning the
SiMFS-Tk repository and invoking cmake:

.. code-block:: bash

   $ git clone https://github.com/sponchcafe/SiMFS-core/ simfs
   $ mkdir simfs/build
   $ cd simfs/build
   $ cmake ..
   $ make
   $ ctest

After completion of the installation the executables are found in
:code:`build/src/components/<group>/<name>`, e.g.
:code:`build/src/components/mol/simfs_dif`.  The component directory in the
build folder should look like this:

.. code-block:: bash

   /opt/SiMFS-Tk/SiMFS-core/build/src/components/
   ├── ...
   ├── fcs
   │   ├── ...
   │   ├── simfs_det
   │   ├── simfs_exi
   │   ├── simfs_pls
   │   └── simfs_pre
   ├── mol
   │   ├── ...
   │   ├── simfs_cnf
   │   ├── simfs_dif
   │   └── simfs_sft
   ├── ph2
   │   ├── ...
   │   └── simfs_ph2
   └── utl
       ├── ...
       ├── simfs_buf
       ├── simfs_img
       ├── simfs_mix
       ├── simfs_spl
       └── simfs_t3r

You can copy or link the executables to a location where you can easily invoke
them. The next section will introduce the basic usage of a SiMFS-Tk component
from the commandline.



.. _Cygwin: https://www.cygwin.org
.. _homebrew: https://www.brew.sh
.. _WSL: https://docs.microsoft.com/de-de/windows/wsl/about

