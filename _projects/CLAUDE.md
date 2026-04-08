# iSPT Blog Post Writing Guide

## Tone

Measured, precise, and approachable. The guiding reference is Anthropic's research blog style (e.g. [The Assistant Axis](https://www.anthropic.com/research/assistant-axis)).

### Do

- **Lead with facts, not hype.** Open paragraphs with declarative statements: "There is a technique...", "We find that...", "It turns out that...". Let the findings carry the weight.
- **Use "we" naturally.** This is a first-person research narrative. "We tried...", "We steered the model...", "We can measure this."
- **Pose questions to advance the narrative.** Use them at transitions: "What happens when the incantation encodes something more complex?" Questions should feel like the natural next thought, not rhetorical flourishes.
- **Explain before naming.** Introduce a concept through what it does before giving it a label. Say "sparse autoencoders trained on natural text fail to reconstruct these activations" before saying "off-manifold."
- **Keep sentences short and varied.** Mix short declarative sentences with longer explanatory ones. Avoid chains of long sentences.
- **Maintain the "incantation" framing.** Soft prompts are incantations — powerful but opaque mixtures of language. This metaphor runs through the piece and should be used consistently.
- **Let wonder emerge from precision.** The most striking moments come from accurate descriptions of surprising results, not from telling the reader to be surprised.

### Don't

- **Don't oversell.** Avoid "remarkably", "incredibly", "groundbreaking". If a result is surprising, describe it plainly and the surprise will be self-evident.
- **Don't use filler transitions.** No "Interestingly,...", "It is worth noting that...", "Notably,...". Just state the thing.
- **Don't hedge excessively.** One qualifier per claim is enough. "The model can, partially, explain itself" — not "The model can, to some degree, at least partially, begin to explain itself."
- **Don't break the fourth wall.** Don't say "In this blog post, we will show..." or "As we discussed above...". The reader is already reading.
- **Don't use jargon without grounding.** Every technical term should be preceded or immediately followed by a plain-language explanation on first use.

## Structure

The post follows a progressive narrative:

1. **Summary** — The full arc in miniature: incantation framing, the opacity problem, the self-verbalization idea, why it fails with vanilla soft prompts, the manifold diagnosis, contextualization as the fix, escalation from simple to complex targets, closing with the Tier 2 steering vector results.
2. **Background** — LLM architecture, soft prompt tuning, SAEs, self-verbalization. Each subsection motivates the next tool in the interpretability toolkit.
3. **Tier 1** — Calibration experiments with known text instructions. Progressive build-up of conditions: prepend fails, postpend improves manifold alignment, single-frame enables verbalization, multi-frame achieves full recovery.
4. **Tier 2** — Steering vector recovery. The incantation captures a non-textual behavioral shift and the model verbalizes it thematically.

## Formatting

- Section headers use `##`, subsections use `###`.
- All section headers are bold (handled by site CSS).
- Figures use `{% include figure.liquid %}` with `class="img-fluid rounded z-depth-1"`.
- Figure placeholders are HTML comments: `<!-- [placeholder: description] -->`.
- Tabbed figures use `<ul class="tab">` / `<ul class="tab-content">` markup with `tabs: true` in frontmatter.
- Horizontal rules (`---`) separate major sections.
- Use `*italics*` for the model's own words or verbalized outputs.
- Use `**bold**` sparingly, for key terms on first introduction.

## File

The project page is `_projects/1_ispt.md`. It uses `layout: page` and per-page `_styles` for title wrapping.
