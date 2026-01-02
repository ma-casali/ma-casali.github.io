---
layout: page
title: "Projects"
description: "An assortment of hardware, acoustic modeling, and deep learning projects."
---

Below are some of the projects I have been working on. They generally all fall under the umbrella of acoustic engineering, but there are exceptions like my experimentation applying my Deep Learning experience to the training of game-playing neural nets. 

<br>

<div class="card-grid">
  {% assign featured_projects = site.projects | where: "featured", "true" %}

  {% for project in featured_projects %}
    <a href="{{ project.url | relative_url }}" class="custom-card">
      <div class="card-content">
        <h3>{{ project.title }}</h3>
        <p>{{ project.description | default: "Click to learn more about this project." }}</p>
        <span class="read-more">Read more about this project →</span>
      </div>
    </a>
  {% endfor %}
</div>