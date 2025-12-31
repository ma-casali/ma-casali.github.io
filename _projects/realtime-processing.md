---
layout: post
title: "Real-time Video to Audio iOS app"
description: "Algorithm that uses pixel values from frames of videos to continuously generate audio."
date: 2025-12-01
categories: software
featured: true
---
{% include mathjax.html %}

[See the current project files here!](https://github.com/ma-casali/video-auralizer)

What does light sound like? Is there a way for us to hear what a video "sounds" like? I thought I would give it a try while taking some artistic liberties along the way. The result is an iOS app that processes video and produces audio in real-time. Below, I will go into some detail about how this process works and some examples of what the output looks like.

<div style="text-align: center;">
    <img src="/assets/images/VideoAuralizerSample.jpg" alt = "Sample Screenshot from the App" width = "300">
</div>

# The Fundamental Process

The primary goal that I set forth with was to map a frame of video and the colors associated with each pixel in that video to a frequency spectrum in the audible range that represented that frame. This mapping relies on a few key factors:
1. **One frame maps to one chunk of audio.** This means that all pixel information must come together to form a single frequency spectrum. I achieved this by summing up the contributios from a downsampled set of pixels. 
2. **Pixel channel information can be converted from RGB to HSI (Hue, Saturation, and Intensity).** By treating each pixel as a representation of a resonant system, I can map these channels to an *audible frequency* (from Hue), *resonant Q factor* (from Saturation), and *relative amplitude* (from Intensity).
3. **Humans interpret audible frequencies on a log scale and visible frequencies on a linear scale.** This means that a doubling in frequency on the audible spectrum is equivalent to a step of fixed value on the visible spectrum. 

By combining all of these factors I can generate a spectrum \\(F(f)\\) using the following equation:

$$ F(f) = \sum_{i=0}^{N} W(f) * \frac{A_i}{1 + jQ_i(f - f_{0,i})}$$

where \\(i\\) is the pixel index, \\(N\\) is the total number of pixels in the frame, \\(W(f)\\) is some arbitrary windowing function in frequency-space (I default to the Hanning function), and \\(\mathrm{Intensity} \rightarrow A\\), \\(\mathrm{Saturation} \rightarrow Q_i\\), and \\(\mathrm{Hue}\rightarrow f_{0,i}\\) via complex mapping functions.

# Implementation Method (real-time processing)

I implemented this process by structuring the project as follows. The user will have access to a GUI that depicts the camera, resulting spectrum, and resulting time-domain signal. They will have access to factors that affect the signal such as low- and high-pass filters, attack/release values for volume scaling, resonance Q factor scaling, etc. Updates to these values happen on a **main thread**.

Simultaneously two other threads are running: 1) the **processing thread** and 2) the **audio thread**. The processing thread takes in video data, and runs it through the process described above using metal shaders to access the iPhone's GPU. After turning video into audio, it pushes frames to an audio buffer. As this process is happening, the audio thread is emptying frames from this buffer to play to the user. 

# Demonstration

Demonstration coming soon! In the meantime, check out [my work](https://github.com/ma-casali/video-auralizer) so far!