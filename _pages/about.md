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
  transition: background var(--collapse-speed);
}
.landing-links .projects-toggle.active {
  background: rgba(0, 0, 179, 0.225);
}
.landing-links > a:hover,
.landing-links > div:hover {
  background: rgba(0, 0, 179, 0.225);
  text-decoration: none;
  color: #000;
}
.projects-expand {
  display: none;
  font-family: monospace;
  white-space: nowrap;
  position: absolute;
  top: 100%;
  left: 0;
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
  display: block;
  line-height: 1.6;
  cursor: default;
  opacity: 0;
  transition: opacity var(--collapse-speed);
}
.projects-expand .tree-item:has(a):hover .tree-sym {
  color: #0000B3;
}
.projects-expand .tree-item:has(a):hover a {
  opacity: 1;
  color: #0000B3;
}
.projects-expand a {
  font-size: 0.95rem;
  opacity: 0.5;
  color: var(--global-text-color, inherit);
  text-decoration: none;
  font-family: monospace;
  transition: none;
}
#resume-link {
  transition: background var(--collapse-speed), opacity var(--collapse-speed);
}
#resume-link.muted {
  background: #f0f0f0;
  opacity: 0.5;
}
#resume-link.muted:hover {
  background: rgba(0, 0, 179, 0.225);
  color: #000;
  opacity: 1;
}
</style>

<div class="blurb-block" style="display: inline-block; margin-top: -0.25rem;">
<p class="blurb" style="font-size: 1.15rem; margin: 0; color: #666;">AI researcher &thinsp;—&thinsp; agents ∪ interpretability ∪ graphs ∪ biology</p>
<hr style="border: none; border-top: 1px solid #ddd; margin: 0.75rem 0 0 0;">
</div>

<div class="landing-links">
  <div style="position:relative;" class="projects-toggle" id="projects-toggle">
    <span>projects</span>
    <span class="projects-expand" id="projects-expand">{% assign sorted_projects = site.projects | sort: "importance" %}<span class="tree-item"><span class="tree-sym">│</span></span>
{% for project in sorted_projects %}<span class="tree-item"><span class="tree-sym">{% if forloop.last %}└── {% else %}├── {% endif %}</span><a href="{% if project.redirect %}{{ project.redirect }}{% else %}{{ project.url | relative_url }}{% endif %}"{% if project.redirect %} target="_blank"{% endif %}>{{ project.title }}</a></span>
{% endfor %}</span>
  </div>
  <a id="resume-link" href="{{ '/assets/pdf/resume.pdf' | relative_url }}" target="_blank">resume</a>
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
    staggerItems(items, true);
    for (var i = 0; i < items.length; i++) items[i].style.opacity = '0';
    r.classList.remove('muted');
    document.getElementById('projects-toggle').classList.remove('active');
    var totalTime = (items.length * staggerStep + 0.3) * 1000;
    setTimeout(function() { t.classList.remove('open'); }, totalTime);
  }

  document.getElementById('projects-toggle').addEventListener('click', function(e) {
    e.stopPropagation();
    var t = document.getElementById('projects-expand');
    if (t.classList.contains('open')) {
      closeProjects();
    } else {
      t.classList.add('open');
      var items = t.querySelectorAll('.tree-item');
      staggerItems(items, false);
      document.getElementById('resume-link').classList.add('muted');
      document.getElementById('projects-toggle').classList.add('active');
      requestAnimationFrame(function() {
        for (var i = 0; i < items.length; i++) items[i].style.opacity = '1';
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
