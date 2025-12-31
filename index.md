---
layout: home
title: Home Page
---

# Welcome!

Hello and thank you for visiting my site! I am an acoustic engineer and my main focuses are on real-time DSP, hardware development, and machine learning. Outside of these projects I love to play music and have released a couple of EPs on Spotify and other streaming platforms. Please take some time to look at some of my projects and if you ever want to talk about my work, please feel free to reach out via my contact information below!

# Featured Projects
---

{% assign featured_projects = site.projects | where: "featured", "true" %}

{% for project in featured_projects %}

## {{ project.title }}

{{ project.description }}

[Read more about this project]({{ project.url | relative_url }})

---

{% endfor %}

# My Resume

[Download my Resume (PDF)](/assets/documents/MatthewCasali_Resume.pdf)
