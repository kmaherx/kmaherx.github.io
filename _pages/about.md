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
.container {
  max-width: none;
  padding-left: 5rem;
}
.post {
  max-width: none;
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
  opacity: 0.7;
  transition: opacity 0.2s ease;
}
.header-socials a:hover {
  opacity: 1;
  color: #0000B3;
  text-decoration: none;
}
.landing-links {
  margin-top: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.landing-links > a,
.landing-links .projects-toggle {
  font-size: 1.2rem;
  color: #000;
  text-decoration: none;
  opacity: 1;
  transition: opacity 0.2s ease;
  cursor: pointer;
  background: none;
  border: none;
  padding: 0;
  font-family: inherit;
}
.landing-links > a:hover,
.landing-links .projects-toggle:hover {
  opacity: 1;
  text-decoration: none;
  color: #0000B3;
}
.projects-expand {
  display: none;
  font-family: monospace;
  white-space: nowrap;
  opacity: 0;
  transition: opacity 0.3s ease;
}
.projects-expand.open {
  display: inline-block;
}
.projects-expand.visible {
  opacity: 1;
}
.projects-expand .tree-line,
.projects-expand .tree-sym {
  color: #0000B3;
  opacity: 0.7;
}
.projects-expand .tree-item {
  display: block;
  line-height: 1.6;
  pointer-events: none;
}
.projects-expand .tree-item a {
  pointer-events: auto;
  position: relative;
  z-index: 1;
}
.projects-expand a {
  font-size: 0.95rem;
  opacity: 0.5;
  color: var(--global-text-color, inherit);
  text-decoration: none;
  font-family: monospace;
}
.projects-expand a:hover {
  opacity: 1;
  text-decoration: none;
  color: #0000B3;
}
#resume-link {
  transition: opacity 0.3s ease, color 0.3s ease;
}
#resume-link.muted {
  color: #aaa;
  opacity: 0.5;
}
#resume-link.muted:hover {
  color: #0000B3;
  opacity: 1;
}
</style>

<div style="display: inline-block; margin-top: -0.25rem;">
<p class="blurb" style="font-size: 1.15rem; margin: 0; color: #666;">AI researcher &thinsp;—&thinsp; agents ∪ interpretability ∪ graphs ∪ biology</p>
<hr style="border: none; border-top: 1px solid #ddd; margin: 0.75rem 0 0 0;">
</div>

<div class="landing-links">
  <div style="display:flex; align-items:baseline; gap:1rem; height:1.5em; overflow:visible;">
    <span class="projects-toggle" id="projects-toggle">projects</span>
    <span class="projects-expand" id="projects-expand">{% assign sorted_projects = site.projects | sort: "importance" %}<span class="tree-line">──────┬── </span><a href="{% if sorted_projects.first.redirect %}{{ sorted_projects.first.redirect }}{% else %}{{ sorted_projects.first.url | relative_url }}{% endif %}"{% if sorted_projects.first.redirect %} target="_blank"{% endif %}>{{ sorted_projects.first.title }}</a>
{% for project in sorted_projects %}{% if forloop.first %}{% continue %}{% endif %}<span class="tree-item"><span class="tree-sym"><span style="color:transparent; pointer-events:none;">──────</span>{% if forloop.last %}└── {% else %}├── {% endif %}</span><a href="{% if project.redirect %}{{ project.redirect }}{% else %}{{ project.url | relative_url }}{% endif %}"{% if project.redirect %} target="_blank"{% endif %}>{{ project.title }}</a></span>
{% endfor %}</span>
  </div>
  <a id="resume-link" href="{{ '/assets/pdf/resume.pdf' | relative_url }}" target="_blank" style="align-self: flex-start;">resume</a>
  <script>
  function closeProjects() {
    var t = document.getElementById('projects-expand');
    var r = document.getElementById('resume-link');
    t.classList.remove('visible');
    r.classList.remove('muted');
    setTimeout(function() { t.classList.remove('open'); }, 300);
  }

  document.getElementById('projects-toggle').addEventListener('click', function(e) {
    e.stopPropagation();
    var t = document.getElementById('projects-expand');
    if (t.classList.contains('open')) {
      closeProjects();
    } else {
      t.classList.add('open');
      document.getElementById('resume-link').classList.add('muted');
      requestAnimationFrame(function() { t.classList.add('visible'); });
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

  </script>
</div>
