---
layout: about
title: about
permalink: /
subtitle:
profile: false
announcements: false
latest_posts: false
selected_papers: false
social: false
---

<style>
.about-page {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
}
.about-page .post {
  max-width: none;
  margin: 0;
  padding-left: 5vw;
}
.post-header {
  text-align: left;
  padding-top: 12vh;
}
.about-page .post-title {
  font-variant: small-caps;
  font-weight: 600;
}
.about-blurb {
  color: #666;
  margin-top: 0.5rem;
  line-height: 1.8;
}
html[data-theme="dark"] .about-blurb {
  color: #aaa;
}
.about-section {
  margin-top: 2.5rem;
}
.about-section-title {
  font-weight: bold;
  margin-bottom: 0.5rem;
}
.about-project {
  margin-left: 1.5rem;
}
.about-socials a {
  text-decoration: none;
  border-bottom: 1px dotted currentColor;
}
.about-socials a:hover {
  border-bottom-style: solid;
}
a.about-link {
  text-decoration: none;
  border-bottom: 1px dotted currentColor;
}
a.about-link:hover {
  border-bottom-style: solid;
}
</style>

<div class="about-blurb">
AI agents<br>
AI interpretability
</div>

<div class="about-section">
<div class="about-section-title">Projects</div>
<a class="about-link" href="/projects/1_ispt/">Soft prompts as a window into introspection in large language models</a>
</div>

<div class="about-section about-socials">
<div class="about-section-title">Socials</div>
<a href="{{ '/assets/pdf/resume.pdf' | relative_url }}">Resume</a><br>
<a href="mailto:kamal.m.maher@gmail.com">Email</a><br>
<a href="https://github.com/kmaherx" target="_blank">GitHub</a><br>
<a href="https://www.linkedin.com/in/kamal-maher-4b526395/" target="_blank">LinkedIn</a><br>
<a href="https://scholar.google.com/citations?user=kDjKQHkAAAAJ" target="_blank">Google Scholar</a><br>
<a href="https://x.com/fluorocore" target="_blank">Twitter</a>
</div>

<div class="about-section">
<a href="#" id="theme-toggle" onclick="
  var html = document.documentElement;
  html.classList.remove('transition');
  var current = html.getAttribute('data-theme');
  var next = current === 'dark' ? 'light' : 'dark';
  html.setAttribute('data-theme', next);
  html.setAttribute('data-theme-setting', next);
  localStorage.setItem('theme', next);
  return false;
" style="text-decoration: none;"><i class="fa-solid fa-circle-half-stroke"></i></a>
</div>

