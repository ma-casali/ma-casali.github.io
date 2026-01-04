---
layout: page
title: "Projects"
description: "An assortment of hardware, acoustic modeling, and deep learning projects."
---

A collection of projects focusing on acoustic modeling, real-time DSP, and deep learning projects. 

I am constantly trying to push the boundaries of my knowledge and delve into the subjects that interest me. Once I identify the tool I want to build or the idea I want to chase, I don't stop working until I've produced something that I'm proud of.

<br> 

<div class="project-stack">
  {% assign all_projects = site.projects %}

  {% for project in all_projects %}
    <a href="{{ project.url | relative_url }}" class="custom-card project-card">

      {% if project.title-image %}
        <div class="card-image">
          <img src="{{ project.title-image | relative_url }}" alt="{{ project.title }}">
        </div>
      {% endif %}

      <i class="fas fa-arrow-right card-corner-icon"></i>
      <div class="card-content">
        <h3>{{ project.title }}</h3>
        <p>{{ project.description | default: "Click to learn more about this project." }}</p>
      </div>
    </a>
  {% endfor %}
</div>