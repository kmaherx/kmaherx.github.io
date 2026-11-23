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
  /* Crimson Pro 400 (the site's 600 maps to 500, which read too heavy). See _sass/_fonts.scss. */
  font-family: "Crimson Pro Site Title", serif;
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
.about-socials .about-resume {
  display: inline-block;
  margin-top: 1.25rem;
  font-weight: bold;
}
a.about-link {
  text-decoration: none;
  border-bottom: 1px dotted currentColor;
}
a.about-link:hover {
  border-bottom-style: solid;
}
.about-toggle {
  position: absolute;
  top: 5vh;
  left: 5vw;
  z-index: 1;
}
.about-toggle a {
  text-decoration: none;
}
@media (max-width: 768px) {
  #dot-grid { display: none !important; }
}
</style>

<div class="about-toggle">
<a href="#" id="theme-toggle" onclick="
  var html = document.documentElement;
  html.classList.remove('transition');
  var current = html.getAttribute('data-theme');
  var next = current === 'dark' ? 'light' : 'dark';
  html.setAttribute('data-theme', next);
  html.setAttribute('data-theme-setting', next);
  localStorage.setItem('theme', next);
  return false;
"><i class="fa-solid fa-circle-half-stroke"></i></a>
</div>

<div class="about-blurb">
Multi-agent systems, interpretability, and biology
</div>

<div class="about-section">
<div class="about-section-title">Projects</div>
<a class="about-link" href="{{ '/assets/html/csp-div-dashboard.html' | relative_url }}">Divergent soft prompts converge on personas</a><br>
<a class="about-link" href="/projects/contextualized-soft-prompts/">Contextualized soft prompts are interpretable</a>
</div>

<div class="about-section about-socials">
<div class="about-section-title">Socials</div>
<a href="mailto:kamal.m.maher@gmail.com">Email</a><br>
<a href="https://github.com/kmaherx" target="_blank">GitHub</a><br>
<a href="https://www.linkedin.com/in/kamal-maher-4b526395/" target="_blank">LinkedIn</a><br>
<a href="https://scholar.google.com/citations?user=kDjKQHkAAAAJ" target="_blank">Google Scholar</a><br>
<a href="https://x.com/fluorocore" target="_blank">Twitter</a><br>
<a class="about-resume" href="{{ '/assets/pdf/resume.pdf' | relative_url }}">Resume</a>
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
  var parentOf = {}; // "c,r" -> parent "c,r" (absent for root)
  var nodes = []; // [{c,r}] in creation order
  var edges = []; // [{c1,r1,c2,r2}]
  var frontier = []; // [{c,r}] nodes to grow from
  var growDir = {dx: 0, dy: 0}; // bias direction
  var growTimeout = null;
  var growing = false;
  var dragStartX = 0;
  var dragStartY = 0;
  var dragRange = 100; // px for full-scale drag effect, shared by X and Y
  var rateMultiplier = 1; // 0.5 (slow) .. 2.0 (fast), controlled by vertical drag
  var chordBlend = 0; // -1 (minor 7th) .. 0 (chromatic) .. 1 (major 7th), controlled by horizontal drag
  var baseInterval = 40; // ms between growth steps at 1x
  var REPULSION = 1; // higher = branches avoid other branches more strongly (0 = off)
  var REPULSION_RADIUS = 2; // grid cells
  var SELF_ANCESTORS = 4; // own recent ancestors ignored when measuring crowding
  var TIP_BIAS_BASE = 6; // higher = keep extending the current tip more often = fewer, longer branches

  // Chord tones in semitones (relative to base), 2 octaves, indexed by branchStep.
  // Right = Imaj7 at base (Bb -> Bbmaj7 = Bb,D,F,A).
  // Left  = iim7, rooted a whole step above base (C -> Cm7 = C,Eb,G,Bb).
  // Major vs minor color, shares only Bb -- yet I-ii is the start of ii-V-I.
  var RIGHT_CHORD = [0, 4, 7, 11, 12, 16, 19, 23];
  var LEFT_CHORD  = [2, 5, 9, 12, 14, 17, 21, 24];
  var fading = false; // true while the tree is retracting after mouseup
  var RETRACT_DURATION = 800; // ms to retract the whole tree, same in light and dark
  var fadeRAF = null;

  // Per-node twinkle effect (dark mode only)
  var twinkles = []; // [{c, r, t0, s}], s = style (TWINKLE_IN or TWINKLE_OUT)
  var twinkleRAF = null;
  var TWINKLE_DURATION = 450;
  // Halo look per event: peak opacity and how far the ring expands (px past radius 3).
  var TWINKLE_IN  = {alpha: 0.45, grow: 10}; // node appears
  var TWINKLE_OUT = {alpha: 0.2,  grow: 5};  // node disappears (subtler)

  // Audio state
  var audioCtx = null;
  var masterGain = null;
  var convolver = null; // reverb chain, used in dark mode
  var activeTipKey = null; // "c,r" of the last node we extended from
  var branchStep = 0; // how many steps the current branch has extended
  var RETRACT_VOLUME = 0.6; // disappearance tones relative to appearance tones
  var RETRACT_TONE_GAP = 40; // ms, min spacing between disappearance tones (same as growth rate)
  // Blip shapes. NORMAL is the appearance bubble; POP is the lighter dark-mode
  // disappearance (snappier attack, shorter tail, smaller sweep, drier reverb).
  // sweep = starting pitch as a fraction of target; rise/settle = when the sweep
  // overshoots then lands on target; attack/decay/stop = envelope; all times in s.
  var BLIP_NORMAL = {sweep: 0.2, rise: 0.05, settle: 0.12, attack: 0.04,  decay: 0.22, stop: 0.2,  reverb: 0.6};
  var BLIP_POP    = {sweep: 0.6, rise: 0.02, settle: 0.05, attack: 0.008, decay: 0.11, stop: 0.12, reverb: 0.3};

  function makeImpulse(ctx, duration, decay) {
    var rate = ctx.sampleRate;
    var length = Math.floor(rate * duration);
    var buffer = ctx.createBuffer(2, length, rate);
    for (var ch = 0; ch < 2; ch++) {
      var data = buffer.getChannelData(ch);
      for (var i = 0; i < length; i++) {
        data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, decay);
      }
    }
    return buffer;
  }

  function ensureAudio() {
    if (audioCtx) return;
    var AC = window.AudioContext || window.webkitAudioContext;
    if (!AC) return;
    audioCtx = new AC();
    masterGain = audioCtx.createGain();
    masterGain.gain.value = 0.08;
    masterGain.connect(audioCtx.destination);
    convolver = audioCtx.createConvolver();
    convolver.buffer = makeImpulse(audioCtx, 1.6, 2.5);
    convolver.connect(masterGain);
    // Safari: prime the output with a 1-sample silent buffer so the engine
    // doesn't swallow the first scheduled note after waking up.
    var primer = audioCtx.createBufferSource();
    primer.buffer = audioCtx.createBuffer(1, 1, audioCtx.sampleRate);
    primer.connect(audioCtx.destination);
    primer.start(0);
  }

  function playEdgeTone(step) {
    if (!audioCtx) return;
    // Each note is a sine with a quick upward pitch sweep -> bubble-ish blip.
    // Across a branch, the target pitch rises step-by-step. Horizontal drag
    // blends the chromatic default toward a major (right) or minor (left)
    // 7th chord, in semitone (log-frequency) space.
    var base = 233; // Bb3, low starting pitch for a new branch
    var defaultSt = Math.min(step, 24); // chromatic, capped at 2 octaves
    var chord = chordBlend >= 0 ? RIGHT_CHORD : LEFT_CHORD;
    var chordSt = chord[Math.min(step, chord.length - 1)];
    var blend = Math.abs(chordBlend);
    var st = (1 - blend) * defaultSt + blend * chordSt;
    var target = base * Math.pow(2, st / 12);
    playBlip(target, 1.0, BLIP_NORMAL);
    return target;
  }

  // One muffled bubble blip at `target` Hz with peak gain `peak`, shaped by `shape`.
  function playBlip(target, peak, shape) {
    var t = audioCtx.currentTime;
    var osc = audioCtx.createOscillator();
    var g = audioCtx.createGain();
    var lpf = audioCtx.createBiquadFilter();
    osc.type = 'sine';
    // Per-note bubble sweep: start below target, whoop up fast
    osc.frequency.setValueAtTime(target * shape.sweep, t);
    osc.frequency.exponentialRampToValueAtTime(target * 1.15, t + shape.rise);
    osc.frequency.exponentialRampToValueAtTime(target, t + shape.settle);
    // Low-pass filter muffles the sound (underwater feel).
    // Cutoff tracks target pitch so high bubbles aren't completely swallowed.
    lpf.type = 'lowpass';
    lpf.Q.value = 0;
    lpf.frequency.value = Math.min(600, target * 1.4);
    // Soft, short envelope with a quick swell
    g.gain.setValueAtTime(0, t);
    g.gain.linearRampToValueAtTime(peak, t + shape.attack);
    g.gain.exponentialRampToValueAtTime(0.001, t + shape.decay);
    osc.connect(lpf);
    lpf.connect(g);
    g.connect(masterGain);
    // Wet send to reverb in dark mode only
    if (convolver && document.documentElement.getAttribute('data-theme') === 'dark') {
      var send = audioCtx.createGain();
      send.gain.value = shape.reverb;
      g.connect(send);
      send.connect(convolver);
    }
    osc.start(t);
    osc.stop(t + shape.stop);
  }

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
    if (edges.length === 0 && nodes.length === 0 && twinkles.length === 0) return;
    var dark = document.documentElement.getAttribute('data-theme') === 'dark';
    var rgb = dark ? '255,255,255' : '0,0,0';

    ctx.lineWidth = dark ? 1 : 1.5;
    ctx.strokeStyle = 'rgba(' + rgb + ',' + (dark ? 0.4 : 0.7) + ')';
    for (var i = 0; i < edges.length; i++) {
      var e = edges[i];
      ctx.beginPath();
      ctx.moveTo(dotX(e.c1), dotY(e.r1));
      ctx.lineTo(dotX(e.c2), dotY(e.r2));
      ctx.stroke();
    }

    ctx.fillStyle = 'rgba(' + rgb + ',' + (dark ? 0.7 : 0.9) + ')';
    for (var i = 0; i < nodes.length; i++) {
      ctx.beginPath();
      ctx.arc(dotX(nodes[i].c), dotY(nodes[i].r), dark ? 2.5 : 3, 0, Math.PI * 2);
      ctx.fill();
    }

    // Twinkle halos (dark mode): expanding, fading rings on freshly added nodes
    if (dark && twinkles.length) {
      var now = performance.now();
      for (var i = 0; i < twinkles.length; i++) {
        var tw = twinkles[i];
        var k = (now - tw.t0) / TWINKLE_DURATION;
        if (k < 0 || k >= 1) continue;
        var alpha = 1 - k;
        var radius = 3 + k * tw.s.grow;
        ctx.fillStyle = 'rgba(' + rgb + ',' + tw.s.alpha * alpha + ')';
        ctx.beginPath();
        ctx.arc(dotX(tw.c), dotY(tw.r), radius, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }

  function addTwinkle(c, r, style) {
    twinkles.push({c: c, r: r, t0: performance.now(), s: style});
    if (!twinkleRAF) twinkleRAF = requestAnimationFrame(tickTwinkles);
  }

  function tickTwinkles() {
    var now = performance.now();
    while (twinkles.length && now - twinkles[0].t0 > TWINKLE_DURATION) {
      twinkles.shift();
    }
    render();
    if (twinkles.length) {
      twinkleRAF = requestAnimationFrame(tickTwinkles);
    } else {
      twinkleRAF = null;
    }
  }

  function render() {
    drawBase();
    drawGraph();
  }

  // How crowded a candidate cell is by *other* branches: sum of 1/distance
  // over occupied cells within REPULSION_RADIUS, skipping the growing branch's
  // own recent ancestors so a branch isn't repelled by itself.
  function crowding(c, r, parentKey) {
    var own = {};
    var k = parentKey;
    for (var a = 0; a < SELF_ANCESTORS && k; a++) {
      own[k] = true;
      k = parentOf[k];
    }
    var score = 0;
    var R = REPULSION_RADIUS;
    for (var dc = -R; dc <= R; dc++) {
      for (var dr = -R; dr <= R; dr++) {
        if (dc === 0 && dr === 0) continue;
        var key = (c + dc) + ',' + (r + dr);
        if (!graph[key] || own[key]) continue;
        score += 1 / Math.sqrt(dc * dc + dr * dr);
      }
    }
    return score;
  }

  function growStep() {
    if (frontier.length === 0) return;

    // Strongly bias toward most recently added frontier node.
    // Bias exponent grows mildly with frontier size so branch runs hold up as tree grows.
    var tipBias = TIP_BIAS_BASE + Math.log10(Math.max(1, frontier.length));
    var fi = frontier.length - 1 - Math.floor(Math.pow(Math.random(), tipBias) * frontier.length);
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
      if (REPULSION > 0) {
        weight *= Math.exp(-REPULSION * crowding(nc, nr, node.c + ',' + node.r));
      }
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
    parentOf[pick.c + ',' + pick.r] = pick.fc + ',' + pick.fr;
    nodes.push({c: pick.c, r: pick.r});
    edges.push({c1: pick.fc, r1: pick.fr, c2: pick.c, r2: pick.r});
    // Normalize the step direction for momentum
    var stepDc = pick.c - pick.fc;
    var stepDr = pick.r - pick.fr;
    var stepLen = Math.sqrt(stepDc * stepDc + stepDr * stepDr) || 1;
    frontier.push({c: pick.c, r: pick.r, dc: stepDc / stepLen, dr: stepDr / stepLen});

    // Track branch continuity for audio: pitch drops along a single branch,
    // resets when a different frontier node is extended.
    var parentKey = pick.fc + ',' + pick.fr;
    if (parentKey === activeTipKey) {
      branchStep++;
    } else {
      branchStep = 0;
    }
    activeTipKey = pick.c + ',' + pick.r;
    // Remember the node's pitch so it can replay it when it disappears
    nodes[nodes.length - 1].hz = playEdgeTone(branchStep);

    if (document.documentElement.getAttribute('data-theme') === 'dark') {
      addTwinkle(pick.c, pick.r, TWINKLE_IN);
    }

    render();
  }

  function startGrow(e) {
    ensureAudio();
    // Always nudge resume -- Safari can re-suspend silently after backgrounding.
    if (audioCtx) audioCtx.resume();

    // Stop any existing fade
    if (fadeRAF) { cancelAnimationFrame(fadeRAF); fadeRAF = null; }
    if (twinkleRAF) { cancelAnimationFrame(twinkleRAF); twinkleRAF = null; }
    twinkles = [];
    fading = false;

    // Clear previous graph
    graph = {};
    parentOf = {};
    nodes = [];
    edges = [];
    frontier = [];
    activeTipKey = null;
    branchStep = 0;

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

    // Begin growth loop. Drag up/down adjusts rate; left/right blends pitch toward a chord.
    growing = true;
    dragStartX = e.clientX;
    dragStartY = e.clientY;
    rateMultiplier = 1;
    chordBlend = 0;
    scheduleNextStep();
  }

  function scheduleNextStep() {
    if (!growing) return;
    var interval = baseInterval / rateMultiplier;
    growTimeout = setTimeout(function() {
      if (!growing) return;
      growStep();
      scheduleNextStep();
    }, interval);
  }

  function onDrag(e) {
    if (!growing) return;
    // Vertical: up = faster (negative deltaY), down = slower. dragRange travel -> 2x.
    var deltaY = e.clientY - dragStartY;
    var m = Math.pow(2, -deltaY / dragRange);
    rateMultiplier = Math.max(0.5, Math.min(1.5, m));
    // Horizontal: right -> major 7th pull, left -> minor 7th pull. Same range.
    var deltaX = e.clientX - dragStartX;
    chordBlend = Math.max(-1, Math.min(1, deltaX / dragRange));
  }

  function stopGrow() {
    if (!growing) return;
    growing = false;
    if (growTimeout) { clearTimeout(growTimeout); growTimeout = null; }

    // Retract: growth in reverse. Each step removes the newest node together
    // with the edge that attached it, so no edge is ever left uncapped
    // (the newest node is always a leaf). The whole tree goes in RETRACT_DURATION.
    fading = true;
    var dark = document.documentElement.getAttribute('data-theme') === 'dark';
    var startCount = nodes.length;
    var fadeStart = performance.now();
    var lastToneAt = -Infinity;
    function fade(now) {
      var k = Math.min(1, (now - fadeStart) / RETRACT_DURATION);
      var keep = Math.ceil(startCount * (1 - k));
      while (nodes.length > keep) {
        var gone = nodes.pop();
        if (edges.length >= nodes.length && edges.length > 0) edges.pop();
        // Replay the node's own pitch, quieter (throttled so big trees don't buzz)
        if (audioCtx && gone.hz && now - lastToneAt >= RETRACT_TONE_GAP) {
          playBlip(gone.hz, RETRACT_VOLUME, dark ? BLIP_POP : BLIP_NORMAL);
          lastToneAt = now;
        }
        // Swap any appearance halo still on this node for a subtler disappearance one
        twinkles = twinkles.filter(function(t) { return t.c !== gone.c || t.r !== gone.r; });
        if (dark) addTwinkle(gone.c, gone.r, TWINKLE_OUT);
      }
      if (nodes.length === 0) {
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
  }

  setup();
  render();

  canvas.addEventListener('mousedown', startGrow);
  document.addEventListener('mousemove', onDrag);
  document.addEventListener('mouseup', stopGrow);
  window.addEventListener('resize', function() { setup(); render(); });
  // Safari: audio context can be suspended when the tab is hidden; re-resume on return.
  document.addEventListener('visibilitychange', function() {
    if (audioCtx && document.visibilityState === 'visible') audioCtx.resume();
  });

  var observer = new MutationObserver(function() { render(); });
  observer.observe(document.documentElement, { attributes: true, attributeFilter: ['data-theme'] });
})();
</script>
