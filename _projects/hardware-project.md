---
layout: post
title: "Acoustic Metamaterial-Enhanced Transducer"
description: "Design and validation of a novel acoustic metamaterial enabled transducer for heightened beamforming performance."
date: 2023-08-01
categories: hardware
featured: true
---

# The Idea

Most directional acoustic transducers gain their directional capability from summing weighted inputs from elements of a large transducer array. Depending on the frequency of interest, these arrays can be massive (up to miles in length!). In order to combat this, I worked on an acoustic metamaterial-enhanced transducer that can sense both acoustic pressure and a direction, effectively reducing  directional acoustic transducers to the size of a single element!

If you are curious about my work and want to read more than what is provided here, please feel free to peruse my Master's Thesis located [here](https://repositories.lib.utexas.edu/items/034c2639-e6a3-4389-9ab3-e0f2ae32e3f1). It has a lot of background on the math behind the theory, the experiment set-up, and the quirks of the entire process!

# The Goal

In order to validate this sensor, my advisor and I utilized a water-filled impedance tube (the only experimentally validated one of its kind to our knowledge!) to infer its material properties and measure its responsiveness to acoustic waves. By measuring the values for the reflection and transmission coefficients for this sensor, we would be able to infer a value for the *Willis coupling* and *electromomentum coupling* values -- values which effectively describe how well the sensor could determine direction of the incoming wave.

# The Experiment

Below is a picture of the impedance tube that I used to make measurements. A sample section was included between two sections of water. A source would then activate to send acoustic energy through the water, through the source, and towards the termination. Two hydrophones on either side of the impedance tube measured the signal of the acoustic pressure as a function of time.

![Water-filled Impedance Tube](/assets/images/WaterImpedanceTube.png)

This impedance tube was a monster to handle. It required precise tuning and allowance for things like an imperfect termination, coupling of the energy between the water and the steel walls, and bubbles within the tube that would completely disrupt the measurement. 

Additionally, one of the test samples we used is shown below. This sample was a bi-layer stack of PZT-4 (a common piezoelectric material) and PMMA (plexiglass). This sample was chosen because in my modeling optimization, it was determined to exhibit the highest value of Willis coupling.

![Bi-layer Stack](/assets/images/BiLayerStack.png)

# The Results

Ultimately, we were able to measure the strength of the *Willis coupling* within one of the sample sensors. Using reflection and transmission coefficients for the sample facing forwards and backwards, we retrieved the following material properties as a function of frequency. Comparisons our made to a finite element model of the impedance tube/sample system.

![Material Properties](/assets/images/MaterialProperties.png)

This work set the stage for future tests, providing schematics for electromomentum-coupled samples, detailing the highly-sensitive nature of the test apparatus, and explaining both analytical and finite element models for the test system and sample dynamics.