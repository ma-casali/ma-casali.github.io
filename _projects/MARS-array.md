---
layout: post
]description: "An investigation into frequency-invariant array optimization for cetacean monitoring in the Monterey Bay."
date: 2026-05-01
categories: software
title-image: "/assets/images/VideoAuralizer-title.png"
featured: true
---
{% include mathjax.html %}

[See the current project files here!](https://github.com/ma-casali/acoustic-models)

# What is Frequency-invariant (FI) beamforming and why is it important for cetacean monitoring?

Frequency-invariant (FI) beamforming is an important subset of traditional beamforming that constructs arrays so that their performance remains constant over a desired frequency range. The most fundamental attempt at this was by Ward et al. (1994), who created a design scheme for a *linear*, frequency-invariant array. They proposed various methods of extending their design principles to two dimensions, but in my study, I saw that this was not the most optimal solution to frequency-invariant beamforming for a 2-D array. 

Besides my continual interest and fixation on optimization problems and strategies, creating a frequency-invariant array is essential to more productive monitoring of cetacean populations in the Monterey Bay. As I will show, due to the acoustic propagation environment for the frequencies of cetacean calls, detection ranges can increase drastically (**often more than 25 km**) as a result of the array gain from a beamformer. FI arrays can guarantee that this array gain remains somewhat constant for the full frequency range of interest. 