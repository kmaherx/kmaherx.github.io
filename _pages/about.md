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
@media (max-width: 768px) {
  #dot-grid { display: none !important; }
}
</style>

<div class="about-blurb">
AI agents<br>
AI interpretability
</div>

<div class="about-section">
<div class="about-section-title">Projects</div>
<a class="about-link" href="/projects/contextualized-soft-prompts/">Contextualized soft prompts are interpretable</a>
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


<canvas id="dot-grid" style="position: fixed; top: 0; right: 0; width: 50vw; height: 100vh; pointer-events: auto; z-index: 0;"></canvas>

<script>
(function() {
  // CSS media query handles hiding on mobile

  var canvas = document.getElementById('dot-grid');
  var ctx = canvas.getContext('2d');
  var spacing = 20;
  var dpr = window.devicePixelRatio || 1;
  var cols, rows, w, h;

  // Graph state
  var graph = {}; // "c,r" -> true for visited nodes
  var nodes = []; // [{c,r}] in creation order
  var edges = []; // [{c1,r1,c2,r2}]
  var frontier = []; // [{c,r}] nodes to grow from
  var growDir = {dx: 0, dy: 0}; // bias direction
  var growInterval = null;
  var fading = false;
  var fadeHead = 0; // how many items (nodes/edges) have fully faded
  var fadeRAF = null;

  // 8 neighbor offsets
  var neighbors = [
    {dc:-1,dr:0},{dc:1,dr:0},{dc:0,dr:-1},{dc:0,dr:1},
    {dc:-1,dr:-1},{dc:1,dr:-1},{dc:-1,dr:1},{dc:1,dr:1}
  ];

  function setup() {
    w = canvas.clientWidth;
    h = canvas.clientHeight;
    canvas.width = w * dpr;
    canvas.height = h * dpr;
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    cols = Math.floor(w / spacing);
    rows = Math.floor(h / spacing);
  }

  function dotX(c) { return c * spacing + spacing / 2; }
  function dotY(r) { return r * spacing + spacing / 2; }

  function drawBase() {
    ctx.clearRect(0, 0, w, h);
    var dark = document.documentElement.getAttribute('data-theme') === 'dark';
    ctx.fillStyle = dark ? 'rgba(200,200,200,0.15)' : 'rgba(0,0,0,0.25)';
    for (var r = 0; r < rows; r++) {
      for (var c = 0; c < cols; c++) {
        ctx.beginPath();
        ctx.arc(dotX(c), dotY(r), 1.5, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }

  function drawGraph() {
    if (edges.length === 0 && nodes.length === 0) return;
    var dark = document.documentElement.getAttribute('data-theme') === 'dark';
    var rgb = dark ? '255,255,255' : '0,0,0';

    ctx.lineWidth = dark ? 1 : 1.5;
    for (var i = 0; i < edges.length; i++) {
      var a = 1;
      if (fading) {
        a = Math.max(0, Math.min(1, (i - fadeHead + 40) / 40));
      }
      if (a <= 0) continue;
      var e = edges[i];
      ctx.strokeStyle = 'rgba(' + rgb + ',' + (dark ? 0.4 : 0.7) * a + ')';
      ctx.beginPath();
      ctx.moveTo(dotX(e.c1), dotY(e.r1));
      ctx.lineTo(dotX(e.c2), dotY(e.r2));
      ctx.stroke();
    }

    for (var i = 0; i < nodes.length; i++) {
      var a = 1;
      if (fading) {
        a = Math.max(0, Math.min(1, (i - fadeHead + 40) / 40));
      }
      if (a <= 0) continue;
      ctx.fillStyle = 'rgba(' + rgb + ',' + (dark ? 0.7 : 0.9) * a + ')';
      ctx.beginPath();
      ctx.arc(dotX(nodes[i].c), dotY(nodes[i].r), dark ? 2.5 : 3, 0, Math.PI * 2);
      ctx.fill();
    }
  }

  function render() {
    drawBase();
    drawGraph();
  }

  function growStep() {
    if (frontier.length === 0) return;

    // Very strongly bias toward most recently added frontier node
    var fi = frontier.length - 1 - Math.floor(Math.pow(Math.random(), 6) * frontier.length);
    fi = Math.max(0, fi);
    var node = frontier[fi];

    // Score neighbors by direction bias
    var candidates = [];
    for (var i = 0; i < neighbors.length; i++) {
      var n = neighbors[i];
      var nc = node.c + n.dc;
      var nr = node.r + n.dr;
      if (nc < 0 || nc >= cols || nr < 0 || nr >= rows) continue;
      if (graph[nc + ',' + nr]) continue;

      // Combine global direction bias with local momentum
      var globalDot = n.dc * growDir.dx + n.dr * growDir.dy;
      var localDot = n.dc * node.dc + n.dr * node.dr;
      var dot = globalDot * 0.3 + localDot * 0.7;
      var weight = Math.max(0.01, Math.pow(Math.max(0, 0.5 + dot), 3));
      candidates.push({c: nc, r: nr, fc: node.c, fr: node.r, w: weight});
    }

    if (candidates.length === 0) {
      frontier.splice(fi, 1);
      return;
    }

    // Weighted random selection
    var total = 0;
    for (var i = 0; i < candidates.length; i++) total += candidates[i].w;
    var rnd = Math.random() * total;
    var pick = candidates[0];
    var acc = 0;
    for (var i = 0; i < candidates.length; i++) {
      acc += candidates[i].w;
      if (rnd <= acc) { pick = candidates[i]; break; }
    }

    // Add node and edge, carry momentum from parent direction
    graph[pick.c + ',' + pick.r] = true;
    nodes.push({c: pick.c, r: pick.r});
    edges.push({c1: pick.fc, r1: pick.fr, c2: pick.c, r2: pick.r});
    // Normalize the step direction for momentum
    var stepDc = pick.c - pick.fc;
    var stepDr = pick.r - pick.fr;
    var stepLen = Math.sqrt(stepDc * stepDc + stepDr * stepDr) || 1;
    frontier.push({c: pick.c, r: pick.r, dc: stepDc / stepLen, dr: stepDr / stepLen});

    render();
  }

  function startGrow(e) {
    // Stop any existing fade
    if (fadeRAF) { cancelAnimationFrame(fadeRAF); fadeRAF = null; }
    fading = false;
    fadeHead = 0;

    // Clear previous graph
    graph = {};
    nodes = [];
    edges = [];
    frontier = [];

    var rect = canvas.getBoundingClientRect();
    var mx = e.clientX - rect.left;
    var my = e.clientY - rect.top;
    var c = Math.round((mx - spacing / 2) / spacing);
    var r = Math.round((my - spacing / 2) / spacing);
    c = Math.max(0, Math.min(cols - 1, c));
    r = Math.max(0, Math.min(rows - 1, r));

    // Random growth direction
    var angle = Math.random() * Math.PI * 2;
    growDir = {dx: Math.cos(angle), dy: Math.sin(angle)};

    graph[c + ',' + r] = true;
    nodes.push({c: c, r: r});
    frontier.push({c: c, r: r, dc: growDir.dx, dr: growDir.dy});

    render();
    growInterval = setInterval(growStep, 30);
  }

  function stopGrow() {
    if (!growInterval) return;
    // Let it keep growing for a bit (inertia)
    setTimeout(function() {
      if (growInterval) {
        clearInterval(growInterval);
        growInterval = null;
      }
      // Start sequential fade: trigger spreads fast, each item fades slowly
    fading = true;
    fadeHead = 0;
    var total = Math.max(nodes.length, edges.length);
    var fadeStart = performance.now();
    function fade(now) {
      var elapsed = now - fadeStart;
      // Trigger sweeps through all items in ~300ms (fast spread)
      fadeHead = (elapsed / 300) * total;
      // But each item takes ~40 indices worth of distance to fully fade (slow per-item)
      // Check if the last item has fully faded
      if (fadeHead >= total + 40) {
        fading = false;
        graph = {};
        nodes = [];
        edges = [];
        frontier = [];
        render();
        return;
      }
      render();
      fadeRAF = requestAnimationFrame(fade);
    }
    fadeRAF = requestAnimationFrame(fade);
    }, 200);
  }

  setup();
  render();

  canvas.addEventListener('mousedown', startGrow);
  document.addEventListener('mouseup', stopGrow);
  window.addEventListener('resize', function() { setup(); render(); });

  var observer = new MutationObserver(function() { render(); });
  observer.observe(document.documentElement, { attributes: true, attributeFilter: ['data-theme'] });
})();
</script>
