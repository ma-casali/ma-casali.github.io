---
layout: home
title: Welcome to My Portfolio
---

# Featured Projects

{% assign featured_projects = site.projects | where: "featured", "true" %}

{% for project in featured_projects %}

## {{ project.title }}

{{ project.description }}

[Read the Case Study]({{ project.url | relative_url }})

{% endfor %}