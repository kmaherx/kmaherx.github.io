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
.header-socials {
  display: flex;
  gap: 1rem;
  align-items: baseline;
}
.header-socials a {
  font-size: 1.5rem;
  color: #0000B3;
  text-decoration: none;
  opacity: 0.775;
  transition: none;
}
.header-socials a:hover {
  opacity: 1;
  color: #0000B3;
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
.landing-links > div {
  flex: 1;
  text-align: center;
  font-size: 1.1rem;
  color: #000;
  text-decoration: none;
  cursor: pointer;
  background: #f0f0f0;
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
  transition: background var(--button-speed), color var(--button-speed);
}
.landing-links .projects-toggle.active {
  background: rgba(0, 0, 179, 0.775);
  color: #fff !important;
}
.landing-links .projects-toggle.active > span:first-child {
  color: #fff !important;
}
.landing-links > a:hover,
.landing-links > div:hover {
  background: rgba(0, 0, 179, 0.775);
  text-decoration: none;
  color: #fff !important;
}
.landing-links > div:hover > span:first-child {
  color: #fff !important;
}
.projects-expand {
  display: none;
  font-family: monospace;
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  text-align: left;
}
.projects-expand.open {
  display: block;
}
.projects-expand .tree-line,
.projects-expand .tree-sym {
  color: #ccc;
}
.projects-expand .tree-item {
  display: flex;
  align-items: flex-start;
  line-height: 1.6;
  padding-bottom: 0.3rem;
  cursor: default;
  opacity: 0;
  transition: opacity var(--collapse-speed);
}
.projects-expand .tree-sym {
  flex-shrink: 0;
  position: relative;
  white-space: pre;
  align-self: stretch;
}
.projects-expand .tree-item:not(:last-child) .tree-sym::after {
  content: '';
  position: absolute;
  left: 0.25em;
  top: 1.6em;
  bottom: 0;
  border-left: 1.8px solid #ccc;
}
.projects-expand .tree-item:has(a):hover .tree-sym {
  color: #3333CC;
}
.projects-expand .tree-item:has(a):hover a {
  color: #3333CC;
}
.projects-expand a {
  font-size: 0.95rem;
  color: #888;
  text-decoration: none;
  font-family: monospace;
  transition: none;
}
#projects-overlay {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.8);
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
  background: #f0f0f0;
  opacity: 0.5;
}
#resume-link.muted:hover {
  background: rgba(0, 0, 179, 0.775);
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
    left: 0;
    right: 0;
  }
}
</style>

<div class="blurb-block" style="display: inline-block; margin-top: -0.25rem;">
<p class="blurb" style="font-size: 1.15rem; margin: 0; color: #666;">AI researcher &thinsp;—&thinsp; agents ∪ interpretability ∪ graphs ∪ biology</p>
<hr style="border: none; border-top: 1px solid #ddd; margin: 0.75rem 0 0 0;">
</div>

<div id="projects-overlay"></div>
<div class="landing-links">
  <div class="projects-toggle" id="projects-toggle">
    <span>Projects</span>
    <span class="projects-expand" id="projects-expand">{% assign sorted_projects = site.projects | sort: "importance" %}<span class="tree-item"><span class="tree-sym">│</span></span>
{% for project in sorted_projects %}<span class="tree-item"><span class="tree-sym">{% if forloop.last %}└── {% else %}├── {% endif %}</span><a href="{% if project.redirect %}{{ project.redirect }}{% else %}{{ project.url | relative_url }}{% endif %}"{% if project.redirect %} target="_blank"{% endif %}>{{ project.title }}</a></span>
{% endfor %}</span>
  </div>
  <a id="resume-link" href="{{ '/assets/pdf/resume.pdf' | relative_url }}" target="_blank">Resume</a>
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
    var items = t.querySelectorAll('.tree-item');
    var r = document.getElementById('resume-link');
    var overlay = document.getElementById('projects-overlay');
    staggerItems(items, true);
    for (var i = 0; i < items.length; i++) items[i].style.opacity = '0';
    r.classList.remove('muted');
    overlay.classList.remove('visible');
    document.getElementById('projects-toggle').classList.remove('active');
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
      var items = t.querySelectorAll('.tree-item');
      var overlay = document.getElementById('projects-overlay');
      staggerItems(items, false);
      document.getElementById('resume-link').classList.add('muted');
      document.getElementById('projects-toggle').classList.add('active');
      overlay.style.display = 'block';
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
