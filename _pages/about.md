---
layout: about
title: about
permalink: /
subtitle:
profile:
  align: center
  image:
  image_circular: false
  more_info:
selected_papers: false
social: false
---

<span class="blurb">AI researcher working on graph ML, AI agents, interpretability, and biological data.</span>

<style>
.blurb {
  font-size: 1.15rem;
}
.landing-panels {
  display: flex;
  gap: 2rem;
  justify-content: center;
  margin-top: 2.5rem;
}
.landing-panel {
  flex: 1;
  max-width: 256px;
  text-decoration: none;
  color: inherit;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: box-shadow 0.2s ease, transform 0.2s ease;
  display: flex;
  flex-direction: column;
  aspect-ratio: 4 / 5;
}
.landing-panel:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.18);
  transform: translateY(-2px);
  text-decoration: none;
  color: inherit;
}
.landing-panel-img {
  flex: 4;
  overflow: hidden;
  min-height: 0;
}
.landing-panel-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: top;
  display: block;
}
.landing-panel-label {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.1rem;
  font-weight: 500;
  background: rgba(0, 54, 159, 0.06);
  border-top: 1px solid rgba(0,0,0,0.06);
}
.landing-panel-socials {
  cursor: default;
}
.landing-panel-socials:hover {
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transform: none;
}
.socials-grid {
  flex: 4;
  display: grid;
  grid-template-columns: 1fr 1fr;
  min-height: 0;
  gap: 1px;
  background: rgba(0,0,0,0.06);
}
.socials-grid a {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2.25rem;
  color: inherit;
  text-decoration: none;
  background: var(--global-bg-color, #fff);
  transition: box-shadow 0.2s ease, transform 0.2s ease;
}
.socials-grid a:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.18);
  transform: translateY(-2px);
  color: inherit;
  text-decoration: none;
  z-index: 1;
}
@media (max-width: 600px) {
  .landing-panels {
    flex-direction: column;
    align-items: center;
  }
  .landing-panel {
    max-width: 224px;
    width: 100%;
  }
}
</style>

<div class="landing-panels">
  <a class="landing-panel" href="{{ '/assets/pdf/resume.pdf' | relative_url }}" target="_blank">
    <div class="landing-panel-img">
      <img src="{{ '/assets/img/resume_preview.png' | relative_url }}" alt="Resume preview">
    </div>
    <div class="landing-panel-label">Resume</div>
  </a>
  <a class="landing-panel" href="{{ '/blog/' | relative_url }}">
    <div class="landing-panel-img">
      <img src="{{ '/assets/img/freq10.png' | relative_url }}" alt="Blog preview">
    </div>
    <div class="landing-panel-label">Blog</div>
  </a>
  <div class="landing-panel landing-panel-socials">
    <div class="socials-grid">
      <a href="mailto:kamal.m.maher@gmail.com" title="Email"><i class="fa-solid fa-envelope"></i></a>
      <a href="https://github.com/kmaherx" target="_blank" title="GitHub"><i class="fa-brands fa-github"></i></a>
      <a href="https://scholar.google.com/citations?user=kDjKQHkAAAAJ" target="_blank" title="Google Scholar"><i class="ai ai-google-scholar"></i></a>
      <a href="https://x.com/fluorocore" target="_blank" title="X"><i class="fa-brands fa-x-twitter"></i></a>
    </div>
    <div class="landing-panel-label">Socials</div>
  </div>
</div>
