Introduction
============

Welcome! If you 

To understand the structure and functionality of **SiMFS-Tk**, let's walk
through a simple single molecule experiment and map out how it would be modeled.

Take for instance a single dye molecule that lives in a small box that is
illuminated by a focused laser beam. The molecule is free to diffuse according
to a diffusion coefficient and randomly moves around the box. While moving, it
encounters areas of different excitation light intensity. The fluorophore
itself absorbs photons and traverses through different electronic and vibronic
energy states, emitting a fluorescence photon from time to time. Depending on
the molecules position these photons are collected by the microscope objective
and detected by some downstream electronics. The output is a stream of photon
times that contain information about the molecules movement through the laser
focus and about the fluorophore's photophysical characteristics such as triplet
dynamics.

To model such an experiment, we need quite a list of parameters, to specify the
features of our experiment:

- Size of the diffusion box
- Diffusion coefficient
- Laser focus shape and intensity
- Energy state diagram of the fluorophore
- Detection focus shape
- ...

Some of these parameters are a simple number such as the diffusion coefficient,
others such as the energy state diagram (Jablonksy diagram) are more complex
and need some more thought to be specified correctly. Instead of tackling all
these problems at once, let's break these requirements down into smaller
chunks. We start with the simulation of a diffusing object.

We model the object's position as a quartuple of ``(x, y, z, t)`` values
indicating the molecule's center position in x, y and z at time t. Given a
starting position, a diffusion coefficient and a time step (e.g. 100ns), we can
compute a series of random steps the molecule takes according to its diffusive
behavior. We also add some boundaries for the molecule to keep it close to our
observation volume. When a random step would put the molecule out of the box,
we just drop it and try another step. For a second of simulation time, we get a
list of 10 Million positions of a random walk with a time increment of 100ns
each.

Note that although we speak of `the molecule` here, we have not yet modeled a
proper molecule. We just produce a list of coordinates that obey diffusive
behavior. This idea might appear somewhat trivial, but is kind of important:
The molecule does not need to know where it is. It just needs to know what the
environment at a given position is like. More specifically, how bright is the
laser at a certain point in time. From this viewpoint there is no difference
between a molecule that randomly diffuses through a static laser focus and a
stationary molecule that is irradiated by a randomly fluctuating laser
intensity. Building the model this way, we have decoupled the diffusion and
fluorescence simulation from each other.

Now that we have the diffusion process solved, we need get to the laser
intensity at each position to pass it on the fluorescence simulation. There are
multiple ways to model the `shape` of a laser focus. Moreover the value of the
focus function can be a plain intensity value or a full complex description of
the EM-field. For our purpose let's calculate the photon flux density in z for
each point. This quantity signifies the number of photons that pass through a
unit of area at a point (x, y, z) in z-direction. Its unit is
:math:`\frac{1}{s\cdot m^2}`. From the physics' point of view, there are some
requirements to such a function regarding diffraction limited focus sizes and
area-normalized photon flux (all photons have to pass each layer in the
z-dimension). Nonetheless, since we are simulating here, we are free to choose
a function and can go forward with a simple 3D-Gaussian profile that is
characterized by three waist parameters :math:`w_x, w_y` and :math:`w_z`.
As mentioned the molecule does not need to know its position but only the
photon flux density at any given point in time. So we pass through the time
value of the coordinate list to the new photon flux density list, converting
`(x, y, z, t)` into `(f, t)` with f being the photon flux density at position
x, y, z.

We can finally start simulating the fluorescence process now! But before that,
we need to specify what should be simulated here. Let's take a moment to review
a model of the photophysics that are at work in the fluorophore. Given a
Jablonys diagram with three states **S0**, **S1** and **T1** with their usual
transitions *excitation*, *emission*, *intersystem-crossing (ISC)* and
*reverse-intersystem-crossing (rISC)*. (We ignore vibrational states in the
simulation.) Every transition is associated with a rate :math:`r`. We get the
rates :math:`r_{exi}, r_{emi}, r_{ISC}` and :math:`r_{rISC}`. We can set the
latter three to constant values since they do not depend on the environment.
:math:`r_{exi}`, the rate of excitation however is dependent on the laser
intensity. So we compute the rate of this rate according to our list of photon
flux densities, updating its value with each new time step. To convert from
photon flux density to a rate, we need an absorption cross section
:math:`\sigma`, that specifies, how many of the excitation photons are absorbed
by the fluorophore.  This however is a constant value that can be derived from
a molar absorption coefficient :math:`\varepsilon`. With this we can set a
dynamically updating rate for the excitation transition in our Jablonsky
diagram.

The system is now set to start the fluorescence simulation. :math:`r_{exi}` is
set to its first value at time :math:`t=0`. The systems state is initialized to
be **S0**, the ground state. The following algorithm is used:

1. For all outgoing transitions of the current state, calculate a random lifetime.
2. Choose the path of shortest lifetime.
3. Traverse to the path's target state and increment the clock by the lifetime.

The random lifetimes that are computed here follow exponential distributions
with their characteristic decay constant :math:`\lambda=\frac{1}{r}` being the
inverse of the transition rate. In this way the fluorophore progresses from
**S0** to **S1** since there is only one way to go. In **S1** most of the time
the lifetime of the emission edge will be shorter than the lifetime of the ISC
transition, so most of the time the system falls back to **S0**. Whenever the
emission transition is traversed, the current clock time is written out as a
timetag for an emitted photon. Occasionally the lifetime of the ISC transition
is shorter than the emission transition and the system moves to the **T1**
state. From there only the rISC transition can bring it back to **S0**. Since
the :math:`rISC` is typically small, the molecule makes a large step in time
without producing a photon. A dark state was simulated. This process continues
until the end of the first diffusion step is reached. At 100ns time, current
value for :math:`r_{exi}` is no longer valid. The next photon flux density,
value is read, converted to a transition rate and the simulation continues now
with new conditions. When all input flux densities are processed, the
simulation terminates and we are left with a list of times, representing the
emitted photons.

With the list of photons we can already start an analysis like binning and burst
selection or computing a correlation function. However, so far we include
*every* emitted photon in our dataset. In an actual experiment, a significant
fraction of photons is lost due to the detection system. To model this, we need
a detection efficiency for each photon that allows us to randomly drop photons
according to a fixed probability. Let's say we want a detection function
similar to our excitation function (3D Gaussian). We can easily compute this
shape to map out probabilities between 0 and 1, yielding highest detection
efficiency (all photons are detected) close to the center and low efficiency
far out of the center. We then need to examine each photon arrival time and
query the probability of detection to decide if we keep or drop the photon. But
there is a problem: We know the detection efficiency as a function of position.
The photon list has no position information, so we cannot directly find out the
right detection efficiency for each photon. We first need to take the
coordinate list from the beginning and evaluate the detection function in the
same way as we did with the excitation function to get a list of detection
efficiencies and time points. With this list of probabilities and times we can
go through the list of photons, detecting all photons within the first 100ns
with the first detection probability value. We progress through both lists
until all photons are processed.

Finally! We finished the whole simulation of a single molecule diffusing
through a laser focus that absorbs and emits light that is eventually collected
by the detection system. The stepwise process we employed here might seem
unnecessarily complicated or even inefficient regarding the duplicate work of
going through all diffusion coordinates. There are however a lot of benefits
with this approach:

1. *It's safer* – Doing the simulation step by step gives you finer control and
   more confidence 

3. *It's easier* – Tweaking parameters and exploring the
   effects is much simpler when you can loop through just a small part of the
   simulation at a time 
   
2. *It's open* – The example here is just the basic
   functionality of SiMFS-Tk. There are more functions included. With the
   modular strategy it is simple to plug your own functionality into the
   pipeline

