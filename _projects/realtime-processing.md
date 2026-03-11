---
layout: post
title: "Vaudio"
description: "A video-to-audio engine that utilizes Metal kernels for feature extraction and DSP"
date: 2025-12-01
categories: software
title-image: "/assets/images/VideoAuralizer-title.png"
featured: true
---
{% include mathjax.html %}

[See the current project files here!](https://github.com/ma-casali/video-auralizer)

What does light sound like? Is there a way for us to hear what a video "sounds" like? I thought I would give it a try while taking some artistic liberties along the way. The result is an iOS app that processes video and produces audio in real-time. Below, I will go into some detail about how this process works and some examples of what the output looks like.

<div style="text-align: center;">
    <img src="/assets/images/VaudioHome.PNG" alt = "Sample Screenshot from the App" width = "300">
</div>

# Fundamental Approach

My first attempt at creating this app focused on taking a frame of audio and mapping each pixel's hue, saturation, and intensity to frequency, resonance "Q" factor, and amplitude respectively, before summing the contributions of each pixel into a total spectrum. In attempting this however, I ran into a double-headed problem: 1) the processing power to create each spectrum from each pixel even on the GPU was immense; too much even for a 3-fold down-sampled frame, and 2) the resulting audio was a jumble of noise due to the noisiness of the video frame. The latter half of the problem was the one I focused on while I hoped that fixing it would address the latency issue. 

Instead of mapping each pixel, I decided to focus on regions of the frame. Now, each frame is split into a 4 x 4 grid that are used to detect the most prevalent hue in the cell as well as average or peak values of 4 different modes of intensity gradients within the cell. These intensity gradients are breathing, vertical tilt, horizontal tilt, and shear and are calculated about each individual pixel in the cell. You can see a depiction of each gradient matrix used below. 

<div style="text-align: center;">
    <img src="/assets/images/gradientkernels.png" alt = "Gradient Kernels" width = "300">
</div>

The mapping now focuses on each of the 16 cells and the values of hue and intensity gradients that each contain. Each impact the resulting sound for the cell's "instrument" in the following way: 

1. The most promninent hue determines the note that the instrument is playing. Each hue is assigned to a linear scale from 1 to 360 where 1 is red and 360 is magenta. These are then mapped to frequencies according to \\(220 * 2^{\mathrm{hue}/360 * 3}\\) so that it covers three octaves from A3 to A6. These frequencies are called the *fundamental frequencies* and are the basis for the following harmonics. 

2. The "breathing mode" gradient determined by the left-most gradient matrix controls how prevalent the harmonics are. It controls a harmonic roll-off factor that reduces the harmonics by a certain amount of dB per octave starting from the fundamental frequency. A lower breathing mode gradient is associated with a more pure image, therefore it will have less contribution from harmonics and behave more like a pure sine-wave. 

3. The "horizontal tilt" and "vertical tilt" gradients are determined by the two center gradient matrices. These determine the amount of even or odd harmonics respectively present in the resulting spectrum. Even and odd harmonics yield different types of timbre to the sound and while the application of vertical tilt to odd harmonics and horizontal tilt to even harmonics is arbitrary, it allows for effective discernment between the types of gradients through sound. 

4. The shear mode gradients are determined by the right-most gradient matrix. This gradient determines how prevalent the first 13 bessel harmonics are in the spectrum. Bessel harmonics describe the harmonic vibrational modes of cylindrical waves, specifically applied here to a model of the harmonic modes of a circular membrane. Therefore, these harmonics remind us mostly of instruments like drums or cymbals. Because shear gradient values are higher for pictures that have more granular texture, a cymbal-like sound can provide an intuition for more complicated images. 

# Implementation Method

Vaudio's primary goal is to transduce video into audio. In order to do this, it takes the hue and gradient information described previously and uses those values to create spectra for each cell of a video frame. 

The project is split into two main components: 1) the vision engine which handles the camera and computation of hues and gradient values, and 2) the sound engine which handles the audio device and computation of the spectrum and time signal. These are both controlled by the video-to-audio converter which handles shared variables, passing information between the two engines, and sharing information with UI. 

## The Vision Engine

The Vision Engine will open the camera, set up the metal device for dominant hue and gradient calculation, and send data to the metal kernel to perform the calculations. In order to do this in reasonable time, it takes advantage of mip map images using the 3rd order downsampling to balance retention of essential information with processing time requirements. 

## The Sound Engine

The Sound Engine starts the audio device, sets up the metal device for spectrum computation, sends data to the metal kernel to perform the calculations, and populates a buffer that the audio device draws from to play sounds. This circular buffer is populated with a complete latency time of approximately 50 ms with 10-15 ms of that time being attributable to the software. 

## The UI 

Each frame of time signal and spectrum is shared with the UI and a feed from the camera is shown. This provides the user with real-time feedback about how different video feeds create different sounds using the video-to-audio translator. The time signal and spectrum can be hidden or shown so that the user can focus on just the camera alone, or how the audio responds to different inputs. 

<div style="text-align: center;">
    <img src="/assets/images/VaudioCamera_covered.PNG" alt = "Covered Screenshot of the Camera UI" width = "300">
    <img src="/assets/images/VaudioCamera_uncovered.PNG" alt = "Uncovered Screenshot of the Camera UI" width = "300">
</div>