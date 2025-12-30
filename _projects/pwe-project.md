---
layout: post
title: "Parabolic Wave Equation Solver"
description: "Computational model for acoustic radiation using an IFD class of solution to the parabolic wave equation. "
date: 2025-10-01
categories: software
featured: true
---
{% include mathjax.html %}

# Why focus on solving the parabolic wave equation (PWE)?

Noise control and communications rely heavily on knowing exactly how sound will travel over large distances. While there are plenty of analytical models for sound propagation over short distances and within enclosures, they can not extend easily to large-expanding spaces. This is where the parabolic wave equation comes in! An efficient solver of this equation, can model sound propagation over tens of kilometers under a minute. I built my version of this solver to provide acousticians with a free, fast, and reliable software product to aid them in noise analysis.

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

The user provides the algorithm with a starting field \\(\psi(r = 0, z)\\) that then steps the field forward along the range by \\(k \\) to solve for the whole field. 

# My implementation of a PWE solver.

In addition to implementing the parabolic wave equation as an efficient solution method for acoustic propagation by equipping the solver with GPU capability, I also made sure to implement the ability for the solver to work with arbitrary terrains. 

The following is the core of the algorithm: 

```python
for nn in range(len(self.r_mesh) - 1):

    # From Eq. 6.189 (Jensen, Computational Ocean Acoustics)
    A, B = self.build_system_matrices(nn, k_steps[nn], k_0_sq_h_sq, sqrt_approx_vec)
    self.u[nn+1, :] = self.solve_tridiagonal_matrix_eq(A, B, self.u[nn, :])

    # set boundaries to zero pressure
    self.u[nn+1, 0] = 0 # pressure release
    self.u[nn+1, -1] = 0 # absorbing layer
    
    if progress_callback is not None:
        progress_callback(nn, len(self.r_mesh) - 1)
```

This takes in things like the range step \\(k \\), constants, and the square-root approximations to build the two tri-diagonal matrices from Eq. 6.189 from Jensen's Computational Ocean Acoustics. It then solves that matrix equation, for the "nn+1" step of u (the field variable), and continues on iterating.

Some examples of the solutions to the PWE for different environments are shown below. 