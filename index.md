---
layout: home
body_class: wide-layout
title: Home
---

<div class="image-text-container">
  <div class="side-image">
    <img src="{{ '/assets/images/ProfilePicture.png' | relative_url }}" alt="Matthew Casali">
  </div>
  <div class="side-text">
    <h3> Welcome! </h3>
    <p>Hello and thank you for visiting my site! I am an acoustic engineer and passionate researcher with a Master's of Science in Acoustical Engineering from U.T. Austin. I most recently worked as an Acoustic Modeler and Data Analyst at Johns Hopkins University Applied Physics Laboratory before moving back home to California. My main focuses are on real-time DSP, hardware development, and machine learning. I am constantly working on projects within those disciplines to create helpful tools for engineers and scientists and expand upon my knowledge.</p>
    <p>Aside from my work, I love to play music and have released a couple of EPs on Spotify and other streaming platforms. I have been playing piano for over 20 years and it has always been an integral part of my life. I became an acoustic engineer in large part due to the influence music has had on my life and my desire to understand it more.</p>
    <p>Please take some time to look at my projects and if you ever want to talk about my work, please feel free to reach out via my contact information below!</p>
  </div>
</div>

<div class="full-width-breakout">
  <div class="breakout-inner">
    
    <div class="portfolio-dashboard">
      
      <div class="main-content">
        <h2 class="column-title">Featured Projects</h2>
        <div class="project-grid-2x2">
          {% assign featured_projects = site.projects | where: "featured", "true" %}
          {% for project in featured_projects %}
            <a href="{{ project.url | relative_url }}" class="custom-card">

              {% if project.title-image %}
                <div class="card-image">
                  <img src="{{ project.title-image | relative_url }}" alt="{{ project.title }}">
                </div>
              {% endif %}
              <i class="fas fa-arrow-right card-corner-icon"></i>
              <div class="card-content">
                <h3>{{ project.title }}</h3>
                <p>{{ project.description | default: "Technical project details." }}</p>
              </div>
            </a>
          {% endfor %}
        </div>
      </div>

      <div class="sidebar">

        <div class="sidebar-section">
          <h2 class="column-title">Resume</h2>
          <a href="{{ '/assets/documents/MatthewCasali_Resume.pdf' | relative_url }}" class="custom-card small-card highlight-resume">
            <i class="fas fa-file-pdf card-corner-icon"></i>
            <div class="card-content">
              <h3>Download PDF</h3>
              <ul class="resume-preview-list">
                <li>Johns Hopkins APL</li>
                <li>M.S. UT Austin</li>
                <li>DSP, ML & Hardware Focus</li>
              </ul>
            </div>
          </a>
        </div>
        
        <div class="sidebar-section">
          <h2 class="column-title">Music</h2>
          {% assign music_page = site.pages | where: "path", "my-b-music.md" | first %}
          {% if music_page %}
          <a href="{{ music_page.url | relative_url }}" class="custom-card small-card">
            <i class="fas fa-arrow-right card-corner-icon"></i>
            <div class="card-content">
              <h3>My Recent Music Releases</h3>
              <p>{{ music_page.description }}</p>
            </div>
          </a>
          {% endif %}
        </div>

        <div class="sidebar-section">
          <h2 class="column-title">Get in Touch</h2>
          <div class="custom-card small-card static-card">
            <div class="card-content">
              <ul class="contact-list">
                <li>
                  <i class="fas fa-envelope"></i>
                  <a href="mailto:casali.ma98@gmail.com">casali.ma98@gmail.com</a>
                </li>
                <li>
                  <i class="fab fa-linkedin"></i>
                  <a href="www.linkedin.com/in/matthew-casali-761158133" target="_blank">Matthew Casali</a>
                </li>
                <li>
                  <i class="fas fa-phone"></i>
                  <span>(650)-799-5462</span>
                </li>
              </ul>
            </div>
          </div>
        </div>

      </div> </div> </div>
</div>