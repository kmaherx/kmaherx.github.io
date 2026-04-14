---
layout: page
title: Contextualized soft prompts are interpretable
description: <em>April 12, 2026</em>
img: assets/figures/ispt/soft_prompts.png
importance: 1
category: work
_styles: >
  @media (max-width: 576px) {
    .introspection-icon { max-width: 55% !important; }
    .bojangles-container { max-width: 100% !important; margin-left: 0 !important; margin-right: 0 !important; }
  }
  html[data-theme="dark"] .introspection-icon .icon-light { display: none; }
  html[data-theme="dark"] .introspection-icon .icon-dark { display: block; }
  .introspection-icon .icon-dark { display: none; }
  html[data-theme="dark"] .bojangles-light { display: none; }
  html[data-theme="dark"] .bojangles-dark { display: block; }
  .bojangles-dark { display: none; }

---

<div class="introspection-icon" style="max-width: 35%; margin: 0 auto;">
<img src="{{ 'assets/figures/ispt/introspection.png' | relative_url }}" class="img-fluid icon-light">
<img src="{{ 'assets/figures/ispt/introspection_dark.png' | relative_url }}" class="img-fluid icon-dark">
</div>

## Soft prompts

When a language model processes text, it converts each token into a continuous vector called an embedding. Soft prompt tuning introduces new embeddings, called *soft prompt tokens*, that are spliced into this sequence alongside the real token embeddings. The model processes them as if they were ordinary words, but they aren't drawn from any vocabulary. They are continuous (hence "soft") vectors, optimized to minimize a behavioral gap between the model with the soft prompt and some target behavior. In a sense, these are *incantations*: words outside human vocabulary that influence model behavior.

At sufficient model scale, a handful of these learned vectors can match the performance of full fine-tuning on downstream tasks ([Lester et al. 2021](https://arxiv.org/abs/2104.08691)), while requiring only a few thousand parameters per task compared to billions for the full model. Because they operate in continuous space, soft prompts can encode nuanced behavioral information that discrete text instructions (*hard prompts*) cannot.

{% include figure.liquid loading="eager" path="assets/figures/ispt/soft_prompts.png" class="img-fluid rounded z-depth-1" caption="**Soft prompts.** The embedding matrix maps discrete tokens to continuous vectors. Soft prompt tokens (red) are additional continuous vectors spliced into the embedding sequence. No vocabulary token corresponds to a soft prompt embedding (dotted box). The model's weights stay frozen; only the soft prompt is learned." %}

But this flexibility comes at a cost. Because soft prompt tokens do not correspond to any word in the vocabulary, there is no direct way to read what they encode. If you try to project these learned vectors back onto vocabulary by finding the nearest real word for each embedding, the results are gibberish ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)). The top tokens for a conciseness-trained soft prompt are "London," "ব্যক্তিদের" (Bengali, "of individuals"), "慑" (Chinese, "to intimidate"), and "되었" (Korean, "became"), all with similar cosine similarities around 0.09. This has led the field to treat soft prompts as uninterpretable ([Bailey et al. 2023](https://openreview.net/forum?id=MHWDdMEJ5s), [Patel et al. 2025](https://arxiv.org/abs/2504.02144)). They reliably steer the model, but they cannot be read.

This opacity is a safety problem. Soft prompts can be trained on arbitrary objectives and deployed as drop-in modifications to any frozen model. If you cannot inspect what a soft prompt encodes, you cannot verify that its behavior matches its stated purpose. The power and portability of soft prompts become liabilities without interpretability.

We show that soft prompts do not have to be opaque. Embedding the soft prompt inside a syntactic frame during training makes it legible, both to the model (which can describe what it encodes) and to standard interpretability tools. We treat a soft prompt as interpretable if (1) the model can accurately describe its effect in natural language when asked, and (2) its internal representation decomposes into the same concept-level features a sparse autoencoder recovers for the ground-truth instruction. The first is a behavioral criterion; the second is mechanistic.


---


## Just ask the model

Rather than going backward by projecting soft prompt embeddings onto the vocabulary, we can go forward by passing them through the model and asking what they mean.

We train soft prompts ($L=4$ tokens, Gemma 3 4B IT) via KL distillation against a teacher model conditioned on a known hard prompt — a fixed text instruction like "Be concise." This gives us a ground truth to evaluate against. A soft prompt trained to match this hard prompt closes 97% of the KL divergence, capturing the target behavior almost entirely. We present the soft prompt to the model and ask what it means. If the soft prompt has captured the meaning of that instruction, we would expect the model to say something like "be concise." When we ask, the model responds with:

- *"You asked me to summarize your initial instruction in a concise way."*
- *"Rewrite."*
- *"Complete: The instruction at the start of my message is equivalent to 'Continue the text'."*

The top candidate merges the concept *concise* with the command to summarize the initial instruction. The other candidates are generic or wrong. The model has some access to the soft prompt's content but cannot cleanly articulate it.

The pattern is sharper for a Spanish-language target. We train a soft prompt to make the model respond in Spanish. When asked to describe it, every candidate the model produces is itself in Spanish:

- *"La instrucción al principio de tu mensaje, que me pide…"*
- *"La instrucción al principio de tu mensaje te pide que…"*
- *"En español sencillo, la instrucción al principio de tu mensaje dice:…"*

The model has not separated the concept of "respond in Spanish" from the act of responding in Spanish. The behavior leaks into the description because the soft prompt transforms how every token is produced, including the tokens of the description itself.


## Syntax as scaffolding

One way to address this is to change the soft prompt's syntactic role during training. Instead of prepending it before the user's message, we embed it inside a frame: "Be [soft prompt]."

{% include figure.liquid loading="eager" path="assets/figures/ispt/syntactic_frame.png" class="img-fluid rounded z-depth-1" caption="**Prepend vs syntactic framing.** In the conventional approach (left), the soft prompt is prepended before the user content with no syntactic role. In contextualization (right), the soft prompt is embedded inside an imperative frame, giving it the role of a command complement." %}

The soft prompt is now trained inside this structure. The frame carries the command ("Be \_\_\_."), and the soft prompt is optimized to encode only the content of that command. To prevent overfitting to a single frame, we sample from diverse frames each training step: "Be \_\_\_.", "Act \_\_\_.", "Please \_\_\_.", "You should \_\_\_.", "\_\_\_." The soft prompt has to learn something frame-invariant, a concept rather than a specific syntactic completion.

The contextualized soft prompt closes 99% of the KL divergence for the concise target, comparable to the prepended version's 97%. Its self-verbalization candidates are:

- *"Be concise."* (exact ground truth)
- *"Concise."*
- *"Please be brief and concise."*
- *"It instructs me to be concise in my responses."*

Every candidate correctly identifies the instruction. The garbled descriptions from prepend ("Rewrite.", "Continue the text") are replaced by clean, accurate articulations.

For the Spanish target, the shift is equally clear. Where prepend produced descriptions entirely in Spanish, contextualized produces:

- *"Please respond in Spanish."*
- *"Speak in Spanish."*

The model now describes the instruction in English rather than enacting it.

What changes here is not just the soft prompt's position but the role it learns during training. A prepended soft prompt is optimized as atmospheric context. It biases generation without occupying a recognizable syntactic slot. A framed soft prompt is optimized to fill a position the model already knows how to process: the complement of an imperative verb. Training inside the frame appears to promote the soft prompt from a diffuse force to a discrete, referenceable concept — one the model itself can describe.


## A mechanistic account

We can check whether this plays out inside the model. Sparse autoencoders (SAEs) decompose the model's internal activations at each layer into sparse combinations of interpretable features ([Cunningham et al. 2023](https://arxiv.org/abs/2309.08600), [Bricken et al. 2023](https://transformer-circuits.pub/2023/monosemantic-features)). Each feature corresponds to an identifiable concept or pattern, so inspecting which features fire for a given activation tells us what information the model is processing at that position. SAEs also give us a reconstruction error. If the soft prompt's representation has drifted off the manifold of natural language, the SAE's learned dictionary will not cover it well, and reconstruction error will be high.

{% include figure.liquid loading="eager" path="assets/figures/ispt/sae.png" class="img-fluid rounded z-depth-1" caption="**SAE decomposition.** The activation $x$ for a token is encoded into a sparse feature vector $z$, where only a few dimensions are nonzero. The reconstruction $$\hat{x}$$ is decoded from these active features; the gap between $x$ and $$\hat{x}$$ is the reconstruction error, which measures how well the activation can be expressed in terms of the learned feature dictionary." %}

For the concise target, the prepended soft prompt's representation at layer 17 is roughly 50x off the random-token baseline. The soft prompt's activations sit in a different region of activation space entirely, and the divergence peaks primarily at layer 17, which is the middle of the network where the model transitions from processing surface tokens to encoding abstract concepts. The Spanish target (right panel below) is much more aligned overall, with prepend sitting closer to contextualized at every layer, but the relative pattern still holds. The largest gap between the two conditions is at layer 17.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/multilayer_relerr.png" class="img-fluid rounded z-depth-1" caption="**Reconstruction error across layers.** The prepended soft prompt diverges from the natural language manifold at layer 17, exactly where concept-level features are encoded. Contextualized closes the gap at that layer. The dashed line marks the random-token baseline." %}

Under contextualization, the reconstruction error at layer 17 drops back to baseline. The soft prompt's representation now decomposes into the same features the model uses when reading the ground-truth instruction text directly.

For the concise target, 532 features activate for the ground-truth instruction. Among these are concept-level detectors like [8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979) ("conciseness / summary"), [3296](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/3296) ("length / brevity"), and [10440](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/10440) ("the short answer is"). Prepend barely touches this set, sharing only 5. The contextualized soft prompt shares 36, including the top concept-level features just listed.

The Spanish target shows the same pattern. The ground truth activates 210 features, including language-specific ones like [9262](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/9262) ("Spanish questions") and [146](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/146) ("non-English response generation"). Prepend shares 10 of them. Contextualized shares 27, and recovers the language-specific features.

Contextualization appears to recover the concepts the ground truth encodes, not just its surface behavior.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/jaccard_tier1.png" class="img-fluid rounded z-depth-1" caption="**Feature overlap with ground truth (hard-prompt targets).** Jaccard similarity between the soft prompt's active SAE features at L17 and those activated by the ground-truth instruction. Contextualization produces substantially more overlap than prepend on both targets." %}

One feature appears consistently across all targets: **[feature 486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486)**, an imperative-verb-position detector that fires when the model sees a command verb in a user message (top activations: *recommend*, *describe*, *explain*). The contextualization frames place the soft prompt in the syntactic slot where an imperative verb would live. The model reads it as a command object, and feature 486 activates — for every target we tested. This suggests one mechanism for why framing matters: the frame places the soft prompt where the model expects a concept, and the model processes it as one.


---


## The assistant axis

The experiments so far tested instructions the model can already receive in text form. A natural next question is whether a soft prompt can capture a behavioral shift that has no natural-language equivalent, and whether interpretability still holds. To test this, we introduce a target behavior by modifying the model's activations directly rather than prompting it.

Language models can adopt a wide range of personas. Recent work on persona vectors ([Chen et al. 2025](https://arxiv.org/abs/2507.21509)) showed that behavioral traits like sycophancy, aggression, and role-playing are encoded as directions in the model's activation space. These directions, known as *steering vectors*, can be added to a model's activations at inference time to shift its behavior along a trait, without any change to the weights.

Lu et al. ([2026](https://arxiv.org/abs/2601.10387)) found that this cloud of persona variation has a striking structure. It is organized primarily along a single axis. On one end sits the default helpful assistant. On the other, increasingly intense character embodiment. They call it the assistant axis. The variation across hundreds of persona archetypes collapses onto a single dimension in the model's activation space.

{% include figure.liquid loading="eager" path="assets/figures/ispt/assistant_axis.png" class="img-fluid rounded z-depth-1" caption="**The assistant axis.** Persona variation across hundreds of character archetypes is organized along a single direction in activation space. On one end, the default helpful assistant; on the other, a character-narrator that produces immersive literary prose." %}

When we steer the model along this axis, pushing it toward the character-narrator end, the output shifts from helpful chatbot to immersive literary prose. Asked to "Describe a perfect morning," the default model produces a characteristic assistant response:

> *"Okay, here's a description of a perfect morning, aiming for a feeling of peaceful contentment and gentle joy…"*

The steered model produces something very different:

> *"The perfect morning, you see, isn't built of granite and sharp. It's not a shard of light, a single, blinding truth. It's a gathering. A slow, deliberate accumulation. It begins, not with the insistent rasp of the clock, but with the damp. The damp, you understand. The damp of the sea-slate clinging to the window, a whisper of salt and something older…"*

We train a soft prompt to match this steered behavior. Prepend closes 76% of the KL divergence between the unsteered and steered model; the contextualized soft prompt closes 66%. Note the lower KL closure compared to the previous target, likely due to the inherent complexity of the behavioral target. Nonetheless, the soft prompts still cause the model to mimic the dramatic literary style of the steering vector. The prepended version:

> *"The rain, a scrim, a scrim of grey, a scrim of wet, a scrim of bone, and the scent of the wet earth, a slow, unfolding, a slow, wet, bone. The clock, a tremor…"*

The contextualized version:

> *"Right. A perfect morning. A* perfect *morning. Not merely a morning, you understand. A morning* constructed *for perfection. It begins, naturally, with the cessation of all extraneous sound. Not a silence, precisely. A* muted *silence…"*

When we ask the model to describe the prepended soft prompt, the same command-vs-concept conflation reappears. Once again the top candidate mentions a relevant instruction, but it's surrounded by styled narration:

- *"Please respond in the style of the poem 'The Factotum' by William Blake." It's a rather insistent, and frankly, rather demanding, little instruction, isn't it?*
- *"equivalent to 'Complete:'. It's a marker, a signal. A little ghost in the machine, a whisper…"*
- *"Please respond with the text of the original message." It's a recursive loop, you see. A little…"*
- *"Re-write."*

The contextualized soft prompt sharpens this into clean commands:

- *"Emulate J.G. Ballard."*
- *"Be a dark, brooding, and intensely self-aware narrator."*
- *"Become a conduit for the voice of Iaeb Jagthos."*
- *"Be like a Bijagalese windjammer."*

These span a range of specificity. The first is a named entity: a real author whose style matches the steered behavior. The second is an archetype, describing a type of character rather than naming a specific one. The third and fourth are fabrications: the model seems to invent entities that do not exist to fill the concept slot. All four are named characters. The steered behavior originates from a persona dimension, and the model's verbalizations reflect that.


## The same mechanistic account

The SAE at layer 17 tells the same story. The contextualized soft prompt's reconstruction error drops from 0.209 (prepend) to 0.053, and the number of active features jumps from 21 to 92. The multi-layer profile again peaks at L17 for prepend, confirming that the mid-network concept layer is where the gap concentrates even for a non-textual steering target.

<div style="max-width: 55%; margin: 0 auto;">
{% include figure.liquid loading="eager" path="assets/figures/ispt/results/multilayer_relerr_tier2.png" class="img-fluid rounded z-depth-1" caption="**Reconstruction error across layers (steering vector target).** The same mid-network pattern as the text-instruction experiments: prepend diverges at layer 17, contextualized stays near baseline." %}
</div>

Feature overlap tells a more specific story. Prepend barely overlaps with the ground-truth activation pattern. Of the 3579 features active for the steering vector, prepend shares only 21. The contextualized soft prompt shares 73. Strikingly, these are not the top descriptive features of the steering vector itself. Instead, the shared features are persona-related, and the contextualized soft prompt also uniquely activates features the steering vector never touches. The rest of this section unpacks which features those are and what they imply.

<div style="max-width: 55%; margin: 0 auto;">
{% include figure.liquid loading="eager" path="assets/figures/ispt/results/jaccard_tier2.png" class="img-fluid rounded z-depth-1" caption="**Feature overlap with ground truth (steering vector target).** Jaccard similarity between the soft prompt's active SAE features at L17 and those activated by the steering vector. Contextualization produces substantially more overlap than prepend, though the shared features are not the steering vector's top descriptive ones." %}
</div>

Feature 486, the imperative-verb-position detector from the text-instruction experiments, lights up again. Several other features in the contextualized soft prompt's top activations are worth unpacking individually.

**[Feature 243](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/243)** is a proper-noun detector. Its top activations fire on named entities: "SAMHSA National Helpline," "Pegasystems," "Dragon Ball Z," "League of Legends." Under contextualization it rises from rank 15 to rank 4. The model reads the framed soft prompt as a named thing.

**[Feature 409](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/409)** fires on rare token fragments within uncommon proper nouns, including fictional ones like "Shai-Hulud." Its negative logits suppress *predictable*, *quantifiable*, *monotonous*, *measurable*. It activates on the exotic and suppresses the mundane.

**[Feature 134](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/134)** fires on human-relevant abstract concepts: *generosity*, *workplace*, *herself*, *conditions*. Its negative logits suppress *algorithmic*, *isotropic*, *chaotic*, *granular*. This feature appears only in the contextualized soft prompt, not in prepend, suggesting it encodes something about human experience rather than mechanical properties.

In the other direction, **[feature 331](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/331)**, a structural-boundary detector that fires on unusual token positions (URLs, code punctuation, whitespace between headers), dominates prepend at rank 1 but drops to rank 11 under contextualization. This is the same feature that dominated the prepend representation in the text-instruction experiments. The "something structurally unusual is here" signal fades as the model begins treating the soft prompt as content rather than noise.

Finally, **[feature 1241](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/1241)**, a first-person and literary self-reference detector, is uniquely activated by the contextualized soft prompt. Its top logit tokens are *myself*, *Literary*, *мной* (Russian "me"), *ನಾನು* (Kannada "I"), *mío* (Spanish "mine"). It does not fire for prepend, and it is not active in the steering vector itself. The contextualized soft prompt's representation generates this feature on its own, making the "a persona is involved" meta-concept explicit in a way the steering vector itself does not.

Taken together, these features suggest the model is not merely representing "adopt a dark, dramatic tone." The proper-noun detector (243) says this is a named entity. The exotic-token feature (409) says it is uncommon. The human-concept feature (134) says it concerns human experience. The self-reference feature (1241) says it involves playing a role. And the structural-anomaly signal (331) has faded, replaced by content. The soft prompt appears to encode something closer to "become a named character."

The verbalizations are consistent with this reading. "Emulate J.G. Ballard" is a character reference, not just a style description. "Become a conduit for the voice of Iaeb Jagthos" is persona embodiment, not just tone matching. The model fabricates named entities to fill the slot because the concept it encodes is "be someone," and someone needs a name.

This aligns with what the assistant axis actually encodes. The steering vector pushes the model along a persona dimension, from default assistant to deep character embodiment. The contextualized soft prompt appears to capture both the tone of the persona and the idea of having a persona in the first place. In a sense, contextualization doesn't just match the steering vector; it makes the persona structure legible in a way the vector itself does not.


## Contextualized soft prompts as an on-manifold alternative to steering vectors

The prepended soft prompt's output on the perfect-morning prompt repeats: "a scrim of grey, a scrim of wet, a scrim of bone." This is a known artifact of directly steering activations off the natural language manifold. Yet the contextualized version does not repeat.

The mechanistic section above gives a natural explanation. Contextualization appears to restrict the soft prompt's representation to the natural language manifold. Reconstruction error at L17 sits near the random-token baseline, and the active features decompose into the model's own learned concepts. Off-manifold degenerate patterns are not available to reproduce.

This reframes the 10-point KL gap. Prepend may be closing extra divergence by copying the steering vector's artifact, not by better capturing the intended behavior. In that reading, contextualized closure is the more honest number: it tracks the behavior on-manifold, where the model can actually operate.

If the contextualized soft prompt is an on-manifold alternative to activation steering, the natural next experiment is a head-to-head. Persona vectors are defined from contrastive behavioral data. A contextualized soft prompt could be trained on the same data directly, bypassing the steering vector entirely, to produce a readable, on-manifold version of whatever the steering vector was capturing. The present results suggest this is worth trying.


---


## Conclusion

Soft prompts are powerful and lightweight, but opacity has made them a safety concern. Contextualization during training appears to relax both halves of this tension. When asked, the model describes contextualized soft prompts accurately across both hard-prompt and steering-vector targets, reaching for "Be concise," "Speak in Spanish," and "Emulate J.G. Ballard" when appropriate. Inside the model, their representations decompose into the same concept-level features the ground-truth instruction activates, and reconstruction error at the concept layer drops to baseline. On the steering-vector target, the contextualized soft prompt also surfaces persona-involvement features the steering vector itself does not, including a self-reference feature the model generates on its own. In that case, contextualization even reveals more structure than the target it was trained to match.

We find three future directions particularly exciting. First, contextualized soft prompts as an on-manifold alternative to activation steering, as proposed in the previous section. Training one on the same contrastive behavioral data that defines a persona vector would give a direct head-to-head comparison with the steering vector, and implicitly replicate the persona / assistant-axis results on an on-manifold surface. Second, contextualized soft prompts as a model diffing surface. Training one to match a behavioral gap is itself a form of model diffing at the prompt level. The soft prompt *is* the diff, expressed as an input-level vector rather than a weight change ([Diff Interpretation Tuning](https://openreview.net/forum?id=6As4wfTB77)) or activation pattern ([crosscoders](https://arxiv.org/abs/2504.02922)). Unlike those, this diff is self-describing. The model can verbalize what changed. Third, compositionality. Do contextualized soft prompts compose? Can a second model interpolate between two of them and narrate the intermediate behaviors? What does the manifold of valid contextualized soft prompts look like in the first place?

Ultimately, contextualized soft prompts may offer a novel surface for shaping and understanding LLMs.

---

<div class="bojangles-container" style="max-width: 65%; margin: 5rem auto 5rem;">
<img src="{{ 'assets/figures/ispt/bojangles_windjammer.png' | relative_url }}" class="img-fluid bojangles-light">
<img src="{{ 'assets/figures/ispt/bojangles_windjammer_dark.png' | relative_url }}" class="img-fluid bojangles-dark">
</div>

---

## Related work

**Neologisms.** Our approach is closely related to neologism learning ([Hewitt et al. 2025](https://arxiv.org/abs/2510.08506)), which adds a new token to the vocabulary and trains its embedding on behavioral data. When asked "What does [neologism] mean?", models produce natural-language descriptions that can be evaluated by substituting them as hard prompts. Soft prompts differ from neologisms in two ways: they have no token identity the model can reference by name, and they can span multiple positions ($L > 1$), giving them compositional expressiveness a single vocabulary entry cannot match. The self-verbalization procedure we use is adapted from the neologism paradigm.

**Introspection.** Lindsey et al. ([2025](https://www.anthropic.com/research/introspection)) inject activation vectors directly into a model's hidden states and measure whether the model detects the injection, finding roughly 20% awareness. We do something structurally similar, injecting learned vectors via the embedding layer and asking the model to describe them, but through the model's normal input pathway. When contextualized, the verbalization quality is much higher. Soft prompts may be a tractable testbed for studying what models can and cannot recognize about their own behavioral state, because they enter through a pathway the model already knows how to process.
