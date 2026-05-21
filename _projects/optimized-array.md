---
layout: post
description: "An investigation into frequency-invariant array optimization for cetacean monitoring in the Monterey Bay."
date: 2026-05-01
categories: software
title: "Optimized Beamforming for Cetacean Vocalizations"
title-image: "/assets/images/OptimizedBeamforming-title.png"
featured: true
---
{% include mathjax.html %}

[See the current project files here!](https://github.com/ma-casali/acoustic-models)

# How does Frequency-Invariant beamforming help cetacean monitoring?

Tracking and monitoring dispersed populations of cetaceans in open water is an incredibly difficult task. Typically, monitoring efforts rely on Passive Acoustic Monitoring (PAM) and detection of a single specific cetacean call at one center frequency. Localization of the animals then falls to cross-referencing call periods with extremely geographically sparse tags on animals. 

However, with frequency-invariant beamforming, you get 
1. increased signal-to-noise ratio in your recordings, 
2. a wider range of frequencies to listen to than with traditional beamforming, and 
3. efficient localization of call source locations. 

Frequency-invariant beamformers are extremely complicated to engineer, but using a custom built optimizer, I created an optimized array that has highly-directional, frequency-invariant beam patterns from 10 to 100 Hz, a frequency range that covers many cetacean calls. 

In the future, I will also publish a study using my Parabolic Wave Equation solver (linked [here]({% link _projects/pwe-project.md %})) modeled using bathymetry from the Monterey Bay in which I show that a gain of just 7-8 dB that you would get from an FI array can sometimes result in an increase of detection range of **25 km**!

# What is Frequency-invariant (FI) beamforming?

Frequency-invariant (FI) beamforming is an important subset of traditional beamforming that constructs arrays so that their performance remains constant over a desired frequency range. The most fundamental attempt at this was by Ward et al. (1994)[^ward], who created a design scheme for a *linear*, frequency-invariant array. They proposed various methods of extending their design principles to two dimensions, but in my study, I saw that this was not the most optimal solution to frequency-invariant beamforming for a 2-D array. 

# Necessary background on arrays and beampatterns.

Before discussing arrays and what makes them good vs. bad, it will be helpful to understand a couple of concepts relevant to arrays. As I discuss these concepts, I will do so from the perspective of an array of hydrophones which are receivers of acoustic energy for underwater applications. Importantly, all of these concepts can be related to a case of an array of sources of acoustic energy. 

The first concept is that all arrays have an associated **beampattern**. A beampattern, in the hydrophone case, can tell you how loud a sound will be picked up (it's receive level) if it's position relative to the array is changed. This beampattern is typically expressed in terms of an angle from the main axis of the array, as arrays can not typically distinguish range unless the source is extremely close to the array and has the processing capability to do so. Importantly, these beampatterns are frequency-dependent. Cornell University has a well-labeled depiction of what a beampattern for an array might look like.[^cornell_bp]

![Example Beampattern, Cornell University](/assets/images/beampattern_example.png)

A beampattern has some key characteristics. These are the **main lobe**, which is a distinguishable central lobe with the highest receive level of the beampattern and is characterized by either it's **half-angle** or **full-angle beamwidths** which correspond to the angle away from the axis of the main lobe at which the receive level is at *half power* or **-3 dB**. Besides the main lobe of the beam pattern, there can also be **side lobes**, which are local maxima in the beampattern. When the side lobes of a beam pattern are at the same receive level as the main lobe, they are called **grating lobes**. These are very undesirable in beamforming as they result in spatial aliasing, or in other-words, they confuse where the sound is coming from. In general, higher side lobes are less desirable as they confuse spatial resolution. 

Additionally, a beam pattern has an associated **directivity index (DI)** that details, on a dB scale, the ratio of intensity at the main lobe of the beam pattern to the average intensity of the beampattern. DI does a very good job at condensing how *directional* a beampattern is into a single index.

In FI beamforming, it is also very common to refer to **subarrays** of the entire array. These subarrays are characterized by a limited band of performance within the entire desired band of performance and contain a subset of the elements of the entire array. The concept of subarrays and their design will be critical to the optimization of a FI array.

# How do you create an optimized array?

A well-designed array does a couple things well: 

1. The array creates a highly directional beam pattern. This can typically be characterized by a small beam-width of the main beam or an overall high DI. While there are a number of factors that influence this, one of the most important is the *aperture* of the array, or its size relative to a wavelength of the frequency that is trying to receive. 
2. The array reduces side lobes and eliminates grating lobes. Eliminating grating lobes can be done by making sure that the maximum spacing between elements of the array is no more than half of a wavelength. Limiting side lobes is similarly an issue of reducing spacing between elements while maximizing the aperture of the array. 
3. The array functions equivalently across a desired frequency range. This last requirement is tricky as it requires a balance between effective design with the first two requirements in mind, while not sacrificing performance at any frequency. With a set number of elements, where you need to minimize the space between them while increasing the size of the whole array, the case for optimization is made.

Although it would be most straight-forward to create objective functions that measure the DI and side lobe levels of the beampattern directly, these values can be computationally expensive for a multi-dimensional array, *especially* when you need to perform these calculations thousands of times throughout the course of optimization. To address this, I modeled the objective functions solely on the geometry of the array. I focused on optimizing four things: 

1. **Maximization of the minimum aperture across all subarrays.** This minimum aperture was calculated by taking the minimum width of the array and normalizing it by the wavelength of the lowest frequency it was capable of measuring. 

2. **Minimization of the difference between minimum and maximum aperture of a subarray.** Similarly to the minimum aperture from before, the maximum width of the array was found and normalized by the wavelength of its lowest design frequency. This was done to make sure that no array was linear (and thus poor-performing in a single direction).

3. **Maximization of the number of elements per subarray.** This maximized the amount of elements for a subarray that performed of the top edge of the desired band, and as a result ensured that the entire desired band was being serviced by as effectively by the entire array as posisble.

4. **Minimization of the variation of distances within a subarray** This was done to ensure that subarrays were relatively uniform. It is a necessary solution to the problem of subarrays where many elements are clumped together at spacing much less than a wavelength, thus acting as a single element, where only one other element is far away to allow the subarray to be active at a lower frequency. 

These objective functions acted as effective proxies for traditional beamforming metrics that allowed me to iterate different array designs quickly over the course of optimization. 

# Optimization Scheme

Due to practical restrictions of accuracy in element placement as well as logistical problems of setting up the array in a dynamic environment like the ocean, the element positions needed to be discretized. This ruled out many optimization methods that rely on computation of smooth gradients of the objective function. From previous projects, I had experience with simulated annealing (SA), so I decided to apply the algorithm I had already made to this problem. SA is an incredibly effective way at comparing discrete "states" (array configurations) to one another. Very basically, it proceeds by accepting a new state automatically if it produces a better value of the objective function and accepts a new state probabilistically if it produces a worse value. This probability is related both to the difference in current accepted objective value and a temperature value that decreases as the algorithm progresses. There are a number of other intricacies to this algorithm that I plan on discussing in a future post. 

The only thing that I had not included in my previous SA algorithm was functionality with multiple objectives. Standard single-objective scalarization often fails to reach global minima or properly balance competing constraints. As a result, I switched to Multiple Objective Simulated Annealing (MOSA) which relies instead on defining a **Pareto Optimal Set**. The Pareto Optimal Set is a set of objective values resulting from unique states that are non-dominated by one another. According to Kalyanmoy Deb et al.[^deb_nadir] a state \\(x^* \in S\\) is Pareto Optimal if:

1. there does not exist another state \\(x \in S\\) such that \\(f_i(x) \leq f_i(x^*)\\) for all \\(i=1,2,...,M\\) and 
2. \\(f_j \lt f_j(x^*)\\) for at least one index \\(j\\),

where \\(f_i, i = 1,2,...,M\\) represents any number of objective functions.

MOSA then relies on automatically accepting a new state if it is Pareto Optimal and accepting a new state probabilistically if it is not Pareto Optimal depending on the current temperature of the algorithm and the increase in L2 norm of the objective vector. 

Once optimization has been completed, a set of Pareto optimal values is returned. Then, a user is left to distinguish what the most optimal parameter is by balancing the different values of the objective functions for each state. There are three ways that I unify the objective values into a single value: 1) a simple weighted sum, 2) Chebyshev scalaraziation which minimizes the objective function with the maximum value under a weighting scheme, and 3) epsilon minimization that minimizes one objective function given a limit on the remaining objectives. 

# Results of Optimization

Using a 2-D Ward frequency-invariant array as a reference, I optimized an array of elements with the same restrictions on number of elements, total width, and total height. For a 1:10 ratio of low and high frequency with a desired minimum aperture of 2 half-wavelengths, the required amount of elements per side for a Ward array is \\(N=7\\), meaning that the total array has \\(N^2 = 49\\) elements. The total height and width of the array were constrained to \\(L = 2 * (c_0 \ f_L / 2) = 146 \mathrm{m}\\). The state space included points on a grid where points were spaced horizontally by a half-wavelength at the highest frequency, \\(dy = 14.6 \mathrm{m}\\), and points were spaced vertically by \\(dz = 1 \mathrm{m}\\). This was to allow for a design in which elements are placed on a number of cables that are anchored at different spots along the seafloor. When searching through the state space, the optimization algorithm did not allow for two elements to be placed in the same position. 

In order to efficiently navigate the Pareto optimal values, I designed a GUI in python that allowed me to change weightings of the objective functions and select the minimum value. 

![GUI showing optimized array](/assets/images/ArrayOptimizationGUI.png)

The GUI allowed me to switch between unifying methods and change weightings to the different objective functions. Additionally, I could scroll through frequencies on the left-hand-side of the GUI to observe different subarrays. The bottom left plot shows me the normalized minimum and maximum aperture size for each frequency. This allows me to understand how the weightings of each objective function chnages the actual beamforming capabilities of the array. The  array that I landed on was highly optimal, not only because it maximized array size and array roundness, but also because it has a high normalized aperture for the largest band that I saw in the optimal set. 

It is also helpful to look at the different metrics I mentioned earlier such as DI, Half-Power (3dB) half Beam Width (HPBW), and Maximum Side Lobe Level (MSLL). I've plotted the metrics for the Ward array in dashed lines against the optimized array metrics in solid lines. 

![Optimized Array Metrics](/assets/images/OptimizedArrayMetrics.png)

The optimized array does better in many of the metrics than the Ward array. Specifically, it has a more consistent and higher DI than that of the Ward array for the same number of elements. Second, its minimum HPBW is much lower than that of the Ward array, allowing for increased spatial resolution. Neither the Ward array or the optimized array have any side lobes that have been picked up by my detection algorithm for this limit to their aperture. 

In addition to those metrics, let's take a look at what the beampattern looks like at a range of frequencies from 10 to 100 Hz. 

![Beam Pattern Sweep](/assets/images/beampattern_sweep.gif)

Here, we have a squished view of the hemisphere directly in front of the array, much like you would get if you took the hemisphere and flattened it on the ground. The x-axis is pointing out of your screen at de = 0. The main lobe remains fairly constant throughout the entire frequency range. Additionally, side lobes are apparent in the beampattern, characterized by the local maxima away from the main lobe. It is most likely that these side lobes weren't captured in the metrics before because of the shallowness of the nulls of the main beam pattern. However, you can see that the side lobes never exceed -3 dB in receive level. 

# In conclusion: a well-optimized FI array and deployment on the horizon!

These results show that the limitations of traditional monitoring and tracking methods or even traditional beamforming methods don't have to constrain next-generation efforts to track and monitor cetacean life in the Monterey Bay or other marine environments. By utilizing a combination of custom-built Multiple Objective Simulated Annealing (MOSA) and beamforming model, I have built an optimized array that out-performs the foundational Ward array. 

I am extremely hopeful about being able to deploy this array. I am currently writing up plans for how to physically implement this array and how one could set it up for use within the Monterey Bay. 

# Citations

[^ward]: Ward, D. B., Kennedy, R. A., and Williamson, R. C., “Design of frequency-invariant broadband far-field sensor arrays”, in *Proceedings of IEEE Antennas and Propagation Society International Symposium and URSI National Radio Science Meeting*, 1994, vol. 2, Art. no. 23. [doi:10.1109/APS.1994.407857](doi:10.1109/APS.1994.407857).

[^cornell_bp]: Rudstam, L., & Sullivan, P. (2010). Acoustics Unpacked: A General Guide for Deriving Abundance Estimates from Hydroacoustic Data. Acoustics unpacked. [https://acousticsunpacked.echoview.com/acoustics/AcousticBackground/AcousticTransducers.asp](https://acousticsunpacked.echoview.com/acoustics/AcousticBackground/AcousticTransducers.asp)

[^deb_nadir]: Kalyanmoy Deb, Shamik Chaudhuri, and Kaisa Miettinen. 2006. Towards estimating nadir objective vector using evolutionary approaches. In Proceedings of the 8th annual conference on Genetic and evolutionary computation (GECCO '06). Association for Computing Machinery, New York, NY, USA, 643–650. [https://doi.org/10.1145/1143997.1144113](https://doi.org/10.1145/1143997.1144113)