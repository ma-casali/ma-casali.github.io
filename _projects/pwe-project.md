---
layout: post
title: "Parabolic Wave Equation Solver"
description: "Computational model for acoustic radiation using an IFD class of solution to the parabolic wave equation. "
date: 2025-10-01
categories: software
featured: true
---
{% include mathjax.html %}

# What is the Parabolic Wave Equation?

The parabolic wave equation is derived from the Helmholtz Wave Equation in polar coordinates (with no azimuthal dependence),

$$ \frac{\partial^2}{\partial r^2} + 
\frac{1}{r}\frac{\partial p}{\partial r} +
\frac{\partial^2 p}{\partial z^2} + 
k_0^2n^2p = 0 $$

which, when coupled with the paraxial approximation, \\(\frac{\partial^2 \psi} {\partial r^2} = 2ik_0 \frac{\partial \psi}{\partial r}\\), the wave equation results in,

$$ \frac{\partial \psi}{\partial r} = ik_0 \left(\sqrt{n^2 + \frac{1}{k_0}\frac{\partial^2}{\partial z^2} - 1}\right)\psi$$

where, \\(n^2 = c^2/c_0^2\\).

# Solving the Parabolic Wave Equation with the Implicit Finite Difference (IFD) method.

The IFD method takes the definition of the derivative and relates a prior step in \\(r\\) to the next step via the derivative defined within the parabolic wave approximation. 

The user provides the algorithm with a starting field \\(\psi(r = 0, z)\\) that then steps the field forward along the range by \\(dr \\) to solve for the whole field. 

# My implementation of a PWE solver.

In addition to implementing the parabolic wave equation as an efficient solution method for acoustic propagation by equipping the solver with GPU capability, I also made sure to implement the ability for the solver to work with arbitrary terrains. 
