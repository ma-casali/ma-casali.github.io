---
layout: home
title: Home
---


<div class="image-text-container">
  <div class="side-image">
    <img src="{{ '/assets/images/ProfilePicture.png' | relative_url }}" alt="Matthew Casali">
  </div>
  <div class="side-text">
    <h2> Welcome! </h2>
    <p>Hello and thank you for visiting my site! I am an acoustic engineer and my main focuses are on real-time DSP, hardware development, and machine learning. I am constantly working on projects within those subjects to create helpful tools for engineers and scientists and expand upon my knowledge.</p>
    <p>Aside from my work, I love to play music and have released a couple of EPs on Spotify and other streaming platforms. I have been playing piano for over 20 years and it has always been an integral part of my life. I became an acoustic engineer in large part due to the influence music has had on my life and my desire to understand it more.</p>
    <p>Please take some time to look at some of my projects and if you ever want to talk about my work, please feel free to reach out via my contact information below!</p>
  </div>
</div>

<div class="card-grid">
  {% assign projects_page = site.pages | where: "path", "my-a-projects.md" | first %}
  {% if projects_page %}
  <a href="{{ projects_page.url | relative_url }}" class="custom-card">
      <div class="card-content">
          <h3>{{ projects_page.title }}</h3>
          <p>{{ projects_page.description | default: "Click to learn more about this project." }}</p>
          <span class="read-more">View projects →</span>
      </div>
  </a>
  {% endif %}

  {% assign music_page = site.pages | where: "path", "my-b-music.md" | first %}
  {% if music_page %}
  <a href="{{ music_page.url | relative_url }}" class="custom-card">
      <div class="card-content">
          <h3>{{ music_page.title }}</h3>
          <p>{{ music_page.description | default: "Read about my acoustic and musical work." }}</p>
          <span class="read-more">Read about my music →</span>
      </div>
  </a>
  {% endif %}

  <a href="{{ '/assets/documents/MatthewCasali_Resume.pdf' | relative_url }}" class="custom-card">
      <div class="card-content">
          <h3>My Resume (pdf)</h3>
          <p>A description of me, my work experience, and some of the notable work I have done during my career.</p>
          <span class="read-more">Download my resume →</span>
      </div>
  </a>
</div>
