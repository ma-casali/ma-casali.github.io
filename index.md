---
layout: home
title: Welcome!
---

# Matthew Casali

Acoustic Engineer specializing in real-time DSP, audio machine learning, and hardware development.

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

# Contact Info: 

email: [casali.ma98@gmail.com](mailto:casali.ma98@gmail.com)

phone: (650)-799-5462
