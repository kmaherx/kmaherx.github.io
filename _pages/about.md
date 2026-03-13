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

<style>
:root {
  --collapse-speed: 0.3s ease;
  --button-speed: 0.15s ease;
  --about-social-color: #0000B3;
  --about-btn-bg: #f0f0f0;
  --about-btn-text: #000;
  --about-btn-hover: rgba(0, 0, 179, 0.775);
  --about-blurb-color: #666;
  --about-hr-color: #ddd;
  --about-project-color: #888;
  --about-project-hover: #3333CC;
  --about-bullet-color: #ccc;
  --about-overlay-bg: rgba(255, 255, 255, 0.8);
}
html[data-theme="dark"] {
  --about-social-color: #6699FF;
  --about-btn-bg: #2a2a2a;
  --about-btn-text: #d3d3d3;
  --about-btn-hover: #3a3a3a;
  --about-blurb-color: #aaa;
  --about-hr-color: #444;
  --about-project-color: #aaa;
  --about-project-hover: #6699FF;
  --about-bullet-color: #666;
  --about-overlay-bg: rgba(0, 0, 0, 0.8);
}
.post {
  max-width: 600px;
  margin: 0 auto;
  padding-top: 15vh;
  text-align: center;
}
.post .post-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 0;
}
.post .post-title {
  margin-bottom: 0.25rem;
}
a.blurb {
  color: var(--about-blurb-color);
}
.header-socials {
  display: flex;
  gap: 1rem;
  align-items: baseline;
}
.header-socials a {
  font-size: 1.5rem;
  color: var(--about-social-color);
  text-decoration: none;
  opacity: 0.775;
  transition: none;
}
.header-socials a:hover {
  opacity: 1;
  color: var(--about-social-color);
  text-decoration: none;
}
.landing-links {
  margin-top: 1.5rem;
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  position: relative;
}
.landing-links > a,
.landing-links > .projects-toggle {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  color: var(--about-btn-text);
  text-decoration: none;
  cursor: pointer;
  background: var(--about-btn-bg);
  border: none;
  border-radius: 8px;
  padding: 0.5rem 0;
  font-family: inherit;
  transition: none;
}
.landing-links .projects-toggle {
  cursor: pointer;
}
.landing-links .projects-toggle {
  transition: background var(--collapse-speed), color var(--collapse-speed);
}
.landing-links .projects-toggle.active {
  background: var(--about-btn-hover);
  color: #fff !important;
}
.landing-links .projects-toggle.active > span:first-child {
  color: #fff !important;
}
@media (hover: hover) {
  .landing-links > a:hover,
  .landing-links > .projects-toggle:hover {
    background: var(--about-btn-hover);
    text-decoration: none;
    color: #fff !important;
  }
  .landing-links > .projects-toggle:hover > span:first-child {
    color: #fff !important;
  }
}
.landing-links > a:active,
.landing-links > .projects-toggle:active {
  background: var(--about-btn-hover);
  color: #fff !important;
}
.landing-links > .projects-toggle:active > span:first-child {
  color: #fff !important;
}
.projects-expand {
  display: none;
  position: absolute;
  top: 100%;
  left: -1.5em;
  right: 0;
  text-align: left;
  padding-top: 1rem;
}
.projects-expand.open {
  display: block;
}
.projects-expand .project-item {
  display: block;
  position: relative;
  line-height: 1.5;
  padding-bottom: 0.6rem;
  padding-left: 1.5em;
  opacity: 0;
  transition: opacity var(--collapse-speed);
  max-width: 100%;
}
.projects-expand .project-item::before {
  content: '•';
  position: absolute;
  left: 0.4em;
  color: var(--about-bullet-color);
  font-family: monospace;
}
.projects-expand .project-item a {
  white-space: nowrap;
  font-size: 0.95rem;
  color: var(--about-project-color);
  text-decoration: none;
  font-family: monospace;
  transition: none;
}
.projects-expand .project-item:hover a {
  color: var(--about-project-hover);
}
#projects-overlay {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--about-overlay-bg);
  opacity: 0;
  transition: opacity var(--collapse-speed);
  z-index: 10;
}
#projects-overlay.visible {
  opacity: 1;
}
.landing-links {
  z-index: 11;
}
#resume-link {
  transition: background var(--button-speed), color var(--button-speed), opacity var(--collapse-speed);
}
#resume-link.muted {
  background: var(--about-btn-bg);
  opacity: 0.5;
}
#resume-link.muted:hover {
  background: var(--about-btn-hover);
  color: #fff !important;
  opacity: 1;
}
@media (max-width: 576px) {
  .post {
    padding-top: 5vh;
    padding-left: 1rem;
    padding-right: 1rem;
  }
  .post .post-title {
    flex-direction: column !important;
    align-items: center !important;
    gap: 0.5rem !important;
  }
  .header-socials {
    gap: 0.75rem;
    justify-content: center;
  }
  .blurb-block {
    width: 100% !important;
  }
  .blurb-block .blurb {
    white-space: normal;
  }
  .landing-links {
    width: 100% !important;
    margin-left: 0 !important;
    margin-right: 0 !important;
    position: relative;
  }
  .projects-expand {
    left: -1.5em;
    right: 0;
  }
  .projects-expand .project-item a {
    white-space: normal;
  }
}
</style>

<div class="blurb-block" style="display: inline-block; margin-top: -0.25rem;">
<a class="blurb" style="font-size: 1.15rem; margin: 0; text-decoration: none; cursor: default; display: block;">AI researcher &thinsp;—&thinsp; agents ∪ interpretability ∪ graphs ∪ biology</a>
<hr style="border: none; border-top: 1px solid var(--about-hr-color); margin: 0.75rem 0 0 0; transition: border-color var(--button-speed);">
</div>

<div id="projects-overlay"></div>
<div class="landing-links">
  <a class="projects-toggle" id="projects-toggle" href="javascript:void(0)">
    <i class="fa-regular fa-folder-open"></i>&ensp;Projects
  </a>
  <a id="resume-link" href="{{ '/assets/pdf/resume.pdf' | relative_url }}" target="_blank">
    <i class="fa-regular fa-file-lines"></i>&ensp;Resume
  </a>
  <div class="projects-expand" id="projects-expand">
    {% assign sorted_projects = site.projects | sort: "importance" %}
    {% for project in sorted_projects %}<span class="project-item"><a href="{% if project.redirect %}{{ project.redirect }}{% else %}{{ project.url | relative_url }}{% endif %}"{% if project.redirect %} target="_blank"{% endif %}>{{ project.title }}</a></span>
    {% endfor %}
  </div>
  <script>
  var staggerStep = 0.06; // seconds between each item

  function staggerItems(items, reverse) {
    var count = items.length;
    for (var i = 0; i < count; i++) {
      var idx = reverse ? (count - 1 - i) : i;
      items[idx].style.transitionDelay = (i * staggerStep) + 's';
    }
  }

  function closeProjects() {
    var t = document.getElementById('projects-expand');
    var items = t.querySelectorAll('.project-item');
    var r = document.getElementById('resume-link');
    var overlay = document.getElementById('projects-overlay');
    staggerItems(items, true);
    for (var i = 0; i < items.length; i++) items[i].style.opacity = '0';
    var lastDelay = (items.length - 1) * staggerStep;
    setTimeout(function() {
      r.classList.remove('muted');
      document.getElementById('projects-toggle').classList.remove('active');
    }, lastDelay * 1000);
    overlay.style.transitionDelay = lastDelay + 's';
    overlay.classList.remove('visible');
    var totalTime = (items.length * staggerStep + 0.3) * 1000;
    setTimeout(function() {
      t.classList.remove('open');
      overlay.style.display = 'none';
    }, totalTime);
  }

  document.getElementById('projects-toggle').addEventListener('click', function(e) {
    e.stopPropagation();
    var t = document.getElementById('projects-expand');
    if (t.classList.contains('open')) {
      closeProjects();
    } else {
      t.classList.add('open');
      var items = t.querySelectorAll('.project-item');
      var overlay = document.getElementById('projects-overlay');
      staggerItems(items, false);
      document.getElementById('resume-link').classList.add('muted');
      document.getElementById('projects-toggle').classList.add('active');
      overlay.style.display = 'block';
      overlay.style.transitionDelay = '0s';
      requestAnimationFrame(function() {
        for (var i = 0; i < items.length; i++) items[i].style.opacity = '1';
        overlay.classList.add('visible');
      });
    }
  });

  document.getElementById('projects-expand').addEventListener('click', function(e) {
    e.stopPropagation();
  });

  document.addEventListener('click', function() {
    var t = document.getElementById('projects-expand');
    if (t.classList.contains('open')) {
      closeProjects();
    }
  });

  // Match landing-links width to blurb
  var blurb = document.querySelector('.blurb-block');
  var links = document.querySelector('.landing-links');
  if (blurb && links) {
    links.style.width = blurb.offsetWidth + 'px';
    links.style.margin = '1.5rem auto 0';
  }

  </script>
</div>
