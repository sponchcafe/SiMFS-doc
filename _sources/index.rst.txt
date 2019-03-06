.. SphinxTest documentation master file, created by
   sphinx-quickstart on Mon May  7 23:00:23 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to SiMFS-Tk's documentation!
====================================

You found the documentation of **SiMFS-Tk** - the Single Molecule Flourescence
Simulation Toolkit. It is still under active development, so some pats are not
yet complete. For now, check out the installation section to get started.

.. Warning::

   SiMFS-Tk is still under heavy development.


What is it?
------------

SiMFS-Tk is a collection of tools to simulate single molecule fluorescence
experiments. Each tool is a commandline program that is configured by a single
json file and reads and writes a set of plain binary input and output files. By
chaining together multiple tools (components), complex experiments can be
simulated. The modular architecture allows easy and quick iteration of
parameters, parallel execution of simulations as well as inspection of
intermediate results and interfacing to third party software.

The project is developed in portable C++11 and built with CMake. So far the
functionality is only tested on UNIX systems.  For Windows users we recommend
the Linux subsystem for Windows 10.

.. _WSL: https://docs.microsoft.com/de-de/windows/wsl/about

Features
^^^^^^^^
- Free diffusion and immobilized settings
- Different focus functions
    - 3D-Gaussian
    - Normalized flux pseudo-Gaussian
    - Gaussian beam
    - Ideal lens E-field function
    - Custom grid interface for arbitrary numeric focus shape interpolation
- Online and precalculated focus evaluation
- Independent excitation and detection foci
- Arbitrary pulse definition to model µs and ns ALEX, PIE, POE and other pulse 
  shemes
- State based photophysics for flexible modeling of fluorophores
    - Lifetime interval simulation to bridge short (ps) and long (minutes) 
      timescales
    - Complete output of events in the rate graph (e.g. ISC and RISC events)
    - Event system to react to external factors (molecule movement, laser 
      power, FRET photons, collision with simulation box, ...) 
- State simulator module to generate internal dynamics transitions
- Utilities for splitting, mixing and buffering photon timetags, t3r writer
- Imaging module for visualizing photon distributions
- Highly parallel execution and low memory usage due to pipeline design
   

Who is it for?
--------------

This toolkit is for everyone who wants to generate simulated single molecule
data to complement experimental data, test and benchmark analysis procedures or
for demonstrative purposes.

Where is it going?
------------------

SiMFS-Tk tries to provide a framework for further extensions and interfacing
with other simulation software. We therefore try to maintain a highly
modularized approach that uses simple interfaces. The purpose of the packkage
is to produce simulated photon events. Non-trivial analysis is left to
downstream software.

How to get started?
-------------------

Clone the repository and run the installation commands. The compiled binaries
are usable for simple tasks and for testing the functionalty. For more complex
simulations, using **pysimfs**, a python package that coordinates the
invokation of sets of SiMFS-components, is recommended. Check out the basic how
to get started section and the tutorials to learn about SiMFS-Tk and the python
driver.


Contents
========

.. toctree::
   :maxdepth: 2

   installation
   introduction
   quickstart
   tutorial
   examples
   architecture
   components
   api
   contributing




Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`


