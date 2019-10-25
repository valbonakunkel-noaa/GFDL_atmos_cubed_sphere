A BRIF OVERVIEW OF THE  FV&sup3; DYNAMICAL CORE  {#overview}
========================

By:

**The GFDL FV&sup3; NGGPS Development Team, Shian­Jiann Lin, Rusty Benson, Jan­Huey Chen, Xi Chen, Lucas Harris, Zhi Liang, Matthew Morin, William Putman, Shannon Rees, and Linjiong Zhou**

*Last modified: January 27, 2016*

##1. Introduction

A brief overview of the FV3 dynamical core

The GFDL FV3 NGGPS Development Team
Shian-Jiann Lin, Rusty Benson, Jan-Huey Chen, Xi Chen,  Lucas Harris, Zhi Liang, 
Matthew Morin, William Putman, Shannon Rees, and Linjiong Zhou
Last modified: January 27, 2016

##Introduction

This document describes the numerical algorithms used by the Finite-Volume Dynamical Core on the Cubed-Sphere grid (FV3 hereafter). This is a work-in-progress “live” document being created by a group, and for the moment, for the Next Generation Global Prediction System (NGGPS) documentation purpose. More details of the numerics and user-level information will be gradually added.

FV3 is a natural evolution of the hydrostatic Finite-Volume dynamical core (FV core) originally developed in the 90s on the latitude-longitude grid. The FV core started its life at NASA/Goddard Space Flight Center (GSFC) during early and mid 90s as an offline transport model with emphasis on the conservation, accuracy, consistency (tracer to tracer correlation), and efficiency of the transport process. The development and applications of monotonicity-preserving Finite-Volume schemes at GSFC were motivated in part by the need to have a “fix” for the noisy and unphysical negative water vapor and chemical species (Lin et al. 1994, and Lin and Rood 1996). It subsequently has been used by several high-profile Chemistry Transport Models (CTMs), including the NASA-community GMI model (Rotman et al., 2001), GOCART (Chin et al., 2000), and the Harvard University-developed GEOS-CHEM model. This transport module has also been used by several climate models, including the ECHAM5 AGCM. Motivated by the success of monotonicity-preserving FV schemes in CTM applications, a consistently formulated shallow-water model was developed. This solver was first presented at the 1994 PDE on the Sphere Workshop, and years later published by Lin and Rood (1997). The Lin-Rood algorithm for shallow-water equations maintains mass conservation and a key Mimetic property of “no false vorticity generation”, and for the first time in computational geophysical fluid dynamics, uses high-order monotonic advection consistently for momentum and all other prognostic variables,instead of the inconsistent hybrid finite-difference and finite-volume approach used by practically all other “finite-volume” models today. The full 3D hydrostatic dynamical core, the FV core, was constructed based on the Lin-Rood (1996) transport algorithm and the Lin-Rood shallow-water algorithm (1997). The pressure gradient force is evaluated by the Lin (1997) finite-volume integration method,derived from Green’s integral theorem based directly on first principles, and demonstrated errors an order of magnitude smaller than other well-known pressure-gradient schemes. Finally, the vertical discretization is the “vertically Lagrangian” scheme described by Lin (2004).

A non-hydrostatic extension (from FV to FV3) was documented in an unpublished manuscript. The most unique aspect of the FV3 is its Lagrangian vertical coordinate, which is computationally efficient as well as more accurate given the same vertical resolution. Recently, a more computationally efficient non-hydrostatic solver is implemented using a traditional semi-implicit approach for treating the vertically propagating sound waves. This faster solver is the default. The Riemann solver option is more efficient for resolution finer than 1-km, and also more accurate, because sound waves are treated nearly exactly.

In the rest of this document, we will describe the individual key components, from the transport scheme, horizontal and vertical discretization, both non-hydrostatic solvers, and dynamics-physics coupling, to the computational design for modern hybrid and fine-grained High Performance Computing (HPC) systems.

##The transport scheme and grid imprinting on the cubed-sphere

The transport scheme used by FV3 is documented in three main publications (Lin et al. 1994, Lin and Rood 1996, and Putman and Lin 2007). The Putman-Lin scheme is a refinement of the Lin and Rood (1996) scheme for the various Cubed-Sphere grids. Currently, the Gnomonic grid is the grid of our choice, due to its best grid uniformity. At the edges of the cubed-sphere tiles, two one-sided 3rd order extrapolations were averaged to form a directionally symmetric scheme across the edges (see Putman and Lin 2007). This averaging, however, locally reduces the formal accuracy from 4th order in the interior to only second order at the edges of the cubed-sphere, and it creates some grid imprinting due to the mild discontinuity of the great-circle grid lines and some reduction in accuracy. Fortunately, the grid imprinting is greatly reduced at increasing resolutions, since the two-sided extrapolation algorithm converges although more slowly than in the interior. At GFS’s current resolution of 13-km the grid imprinting is practically non-existent for weather predictions. Nonetheless, to improve lower resolution climate applications, we are still working on a revised edge handling algorithm, by using an extended-grid approach, so the algorithm remains fourth-order accurate at the edges., We believe improved edge handling will further reduce the already-small grid imprinting.


##Horizontal discretization

The horizontal discretization of FV3 is essentially the same as the original FV core (Lin and Rood, 1997;Lin 2004), except that all spatial averaging and pressure gradient operators have been upgraded from the second to formally fourth-order accurate in the FV3. More details on the horizontal discretization in FV3 is given in Harris and Lin (2013) and Harris, Lin, and Tu (2016, manuscript under review; available in the google folder).

##The Finite-Volume pressure gradient computation

The evaluation of the pressure-gradient force in FV3 remains the same as in the FV core (Lin 1997), upgraded to fourth-order accuracy. A lesser-known aspect of the Lin (1997) algorithm is its consistency with Newton’s 3rd law of motion, achieved by finite-volume integration about a grid cell: the pressure force exerted upon a cell by its neighbor is equal and opposite to that exerted by the cell upon the neighbor. This form satisfies Newton’s third law of motion in the same way flux-form transport schemes satisfy mass conservation. Other algorithms for evaluating the pressure-gradient force do not meet this requirement, with consequences revealed by a simple hydrostatic equilibrium test; see Lin (1997) for details. A similar test was performed as part of the Dynamical Core Intercomparison Project (Test 2-0-0, Ullrich et al 2012): initially resting, hydrostatic atmosphere is imposed upon topography, and the resulting spurious accelerations are then computed. It is found that FV3 produces little spurious oscillation compared to many other schemes.

![ ](../image/Test15level.png)
![ ](../image/Test30level.png)


Results at 6 days from DCMIP test case 2-0-0 for two different vertical resolutions. Results from https://earthsystemcog.org/projects/dcmip-2012/fv3-gfdl-test-200 . Compare against, for example the MPAS core, https://earthsystemcog.org/projects/dcmip-2012/mpas-test-2-0-0 .


##Vertically Lagrangian discretization

The Lagrangian vertical coordinate used in FV3 is fully described by Lin (2004) except the non-hydrostatic extension to be described in the next chapter. In this chapter we will provide more details and recent improvements not available in Lin (2004). This chapter is a work in progress.

##Non-hydrostatic extensions: the Riemann solver and the semi-implicit solver

FV3 contains two non-hydrostatic solvers. The first, a Riemann solver, was developed at GFDL between 2003-2006 based on a conservative Riemann invariants approach. This algorithm is particularly suitable for ultra-high resolution cloud-resolving simulations, with grid-cell widths of 1 km or less. A variation of this approach,within a simplified 2D vertically-Lagrangian framework, has been published by Chen et al. (2013).

The second non-hydrostatic solver, developed recently for lower horizontal resolutions, is a more traditional semi-implicit time integration scheme for vertically propagating sound waves. This solver is more suitable and more efficient for lower horizontal resolution simulations, in which the extra damping provided by the semi-implicit time-integration scheme can act to filter out the poorly resolved sound waves, and therefore provides a less-noisy simulation. Both solvers use the same governing equations.

 	

Modification for moist effects (water vapor and “condensates loading”)

This part will be filled later. Expected timeframe: Oct-Dec 2016.

Coupling to physical parameterization and surface models (ocean and land)

The coupling to physical parameterizations and surface models is straightforward; the primary complication is interpolating between cell-centered, orthogonal winds used by most physics packages and FV3’s staggered non-orthogonal D-grid. two-stage horizontal re-mapping is used. The D-grid winds are first transformed to locally orthogonal and unstaggered wind components at cell centers, as input to the physics. The wind tendencies (du/dt, dv/dt) from the physics are then remapped (using high-order algorithm) back to the native grid for updating the prognostic winds. This procedure satisfies the “no data no harm” principle - that the interpolation/remapping procedure creates no tendency on its own if there are no external sources from physics or data assimilation. 

##Plan for extension for space weather application

###10.1 Modification for the deep atmosphere equation set

Modifications of FV3 to include height-dependent radius and gravitational acceleration, three-dimensional Coriolis force, and related metric terms is being considered. FV3 can, by construction, use any vertical coordinate system. Currently, the mass coordinate is our choice, ensuring the best compatibility with existing physical parameterizations developed for hydrostatic models. Furthermore, for non-hydrostatic applications, the mass-coordinate does not need the unphysical rigid lid at the model top, reducing the need for a strong wave-absorbing sponge-layer (as commonly done in regional weather research models). On the other hand, the height of each grid box, and thus the radius and gravitational acceleration, is time-dependent in the mass coordinate. 

Hence, we may consider instead using the height coordinate for space-weather applications, in spite of the need for a reflective rigid lid that may degrade medium-range weather predictions and climate simulations unless the lid is very high.  To the best of our knowledge, the z-coordinate (popular in regional models) has never been successfully used in an established climate model from major modeling centers.

A thorough research program will thus be needed, including support from weather prediction centers, to modify FV3 to meet the requirements for all the stated NGGPS goals for space-weather prediction while maintaining the solution quality in the lower atmosphere. A separate configuration for space weather may be the best solution, to avoid degrading weather and climate simulation, but this would require significant human resources.


###10.2 A generalized potential temperature (instead of enthalpy) for multi-species “dry” air

We are developing a novel methodology, based on first principles, to support the thermodynamic effect of multiple dry air species in the middle and upper atmosphere. Currently, FV3 uses a modified moist potential temperature as its conserved thermodynamic variable, to take into account the varying specific heats and gas constants of different atmospheric constituents of relevance in the lower and middle atmosphere. This methodology can be extended to an arbitrary combination of dry air constituents in the upper atmosphere, without altering the basic FV3 algorithm. This modification is expected to be much less intrusive  than the proposed height-coordinate configuration in the previous section. However, more development and testing needs to be done before this revised potential-temperature based thermodynamic variable is to be adopted for FV3. The impacts of this particular modification for space weather to the troposphere-stratosphere, both weather and climate, should be carefully evaluated. We feel this is a one to three years effort after the NGGPS final selection.


###10.3 Computational strategy for large 3D diffusion, high-wind speed, and high temperature

We can derive a new way to discretize the vertical motion using a form of Riemann solver. In fact, Riemann solvers are proven to be mature tools for simulating high-wind speed and supersonic flows, high-temperature flows with strong discontinuities. Riemann solvers implicitly provide some diffusion to keep the numerical scheme stable, and, of course, physical diffusion can be still added if needed. We have successfully developed a Riemann solver-based horizontal transport scheme capable of handling strong discontinuities, which is a strong virtue of a genuinely finite-volume model like FV3. This exploration requires only minor plugged-in functions to our current CD-grid based algorithm (on-going work, based on Chen et. al 2013). Although this work has not been tested with deep atmosphere environment, we feel that the flexibility to equip Riemann solvers in our algorithm may lower the risks as compared to other traditional numerical techniques.

##Computational design

FV3 adopts the latest hybrid shared-distributed memory programming styles (“OpenMP” and MPI, Putman et al. 2005). It can, in principle, be easily transformed into “openACC” for GPU-enabled platforms. The computational design of FV3 also takes full advantage of the vertically Lagrangian discretization by using OpenMP (shared memory) in the vertical direction-- effectively a third parallel dimension. The horizontal grid is decomposed into a series of two-dimensional domains, each with its own MPI rank, within each of the six cubed-sphere tiles. These decomposed domains need not be square, permitting flexibility in choosing the total number of cores (CPUs). A flowchart showing how OpenMP and MPI are each used for parallelism in FV3 is shown below.

![ ](../image/OpenMP.png)

All dynamical core related codes are written in Fortran 90 with modules. The non-hydrostatic part of the extension is coded in a module named “nh_core”, enabling easy run-time switching between hydrostatic and non-hydrostatic configurations.

12. Initialization with NCEP and ECMWF analysis
13. Terrain filter algorithm
To be added; due to unexpected NGGPS workload, this may be delayed until July 2016.
References:

Chen, X, N Andronova, B van Leer, J Penner, J P Boyd, C Jablonowski, and Shian-Jiann Lin, July 2013: A control-volume model of the compressible Euler equations with a vertical Lagrangian Coordinate. Monthly Weather Review, 141(7), DOI:10.1175/MWR-D-12-00129.1

Chin, M, and Shian-Jiann Lin, et al., October 2000: Atmospheric sulfur cycle simulated in the global model GOCART: Model description and global properties. Journal of Geophysical Research, 105(D20), 24,671-24,687, DOI:10.1029/2000JD900384

Harris, Lucas M., and Shian-Jiann Lin, January 2013: A two-way nested global-regional dynamical core on the cubed-sphere grid. Monthly Weather Review, 141(1), DOI:10.1175/MWR-D-11-00201.1.

Harris, Lucas M., and Shian-Jiann Lin, July 2014: Global-to-regional nested-grid climate simulations in the GFDL High Resolution Atmosphere Model. Journal of Climate, 27(13), DOI:10.1175/JCLI-D-13-00596.1.

Harris, Lucas M., Shian-Jiann Lin, and ChiaYing Tu: High-resolution climate simulations using GFDL HiRAM with a stretched global grid. In revision at Journal of Climate.

Lin, Shian-Jiann, et al., July 1994: A Class of the van Leer-type Transport Schemes and Its Application to the Moisture Transport in a General Circulation Model. Monthly Weather Review, 122(7), 1575-1593. Abstract.

Lin, Shian-Jiann, and R B Rood, September 1996: Multidimensional Flux-Form Semi-Lagrangian Transport Schemes. Monthly Weather Review, 124(9), 2046-2070. Abstract.

Lin, Shian-Jiann, July 1997: A finite-volume integration method for computing pressure gradient force in general vertical coordinates. Quarterly Journal of the Royal Meteorological Society, 123(542), 1749-1762, DOI:10.1002/qj.49712354214

Lin, Shian-Jiann, and R B Rood, October 1997: An explicit flux-form semi-lagrangian shallow-water model on the sphere. Quarterly Journal of the Royal Meteorological Society, 123(544), 2477-2498. DOI:10.1002/qj.49712354416

Lin, Shian-Jiann, 2004: A "vertically Lagrangian" finite-volume dynamical core for global models. Monthly Weather Review, 132(10), 2293-2307. Abstract.

Putman, W M., and Shian-Jiann Lin, 2007: Finite-volume transport on various cubed-sphere grids. Journal of Computational Physics, 227(1), 55-78. DOI:10.1016/j.jcp.2007.07.022

Putman, W M., Shian-Jiann Lin, and B-W Shen, September 2005: Cross-Platform Performance of a Portable Communication Module and the NASA Finite Volume General Circulation Model. International Journal of High Performance Computing Applications, 19(3), DOI:10.1177/1094342005056101.

Rotman, D, and Shian-Jiann Lin, et al., January 2001: Global Modeling Initiative assessment model: Model description, integration, and testing of the transport shell. Journal of Geophysical Research, 106(D2), 1669-1691, DOI:10.1029/2000JD900463

Ullrich, P.A., C. Jablonowski, J. Kent, P. H. Lauritzen, R. D. Nair, M. A. Taylor (2012), Dynamical Core Model Intercomparison Project (DCMIP) Test Case Document, Version 1.7. Available at https://www.earthsystemcog.org/projects/dcmip-2012/test_cases 
				
			
		




