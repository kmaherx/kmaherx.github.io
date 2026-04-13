---
layout: page
title: Soft prompts as a window into introspection
description: <em>April 2026</em>
img: assets/figures/ispt/soft_prompts.png
importance: 1
category: work
_styles: >
  

---

<div style="max-width: 35%; margin: 0 auto;">
{% include figure.liquid loading="eager" path="assets/figures/ispt/introspection.png" class="img-fluid" %}
</div>

## Summary

Soft prompt tuning optimizes continuous embeddings to steer language model behavior, rivaling fine-tuning in effectiveness while keeping model weights frozen. But these embeddings are opaque. When asked to describe a soft prompt, the model produces garbled descriptions, or the behavior itself leaks into the response: a soft prompt trained for Spanish responds to the question in Spanish.

We trace this to a conflation of command and concept in the soft prompt's representation. Embedding the soft prompt in a syntactic frame (e.g. "Be [soft prompt].") during training separates the two, acting as a form of regularization that keeps the soft prompt's representation on the natural language manifold. Sparse autoencoder analysis confirms this mechanistically: contextualized soft prompts activate the same concept-level features as the ground-truth instruction, specifically at the middle layers of the network where abstract concepts are encoded.

We then apply this to a harder target: the assistant axis, a single direction in activation space that controls how strongly a model inhabits a persona. The contextualized soft prompt captures this behavioral shift and, when asked to describe it, the model reaches for named characters rather than tonal descriptions: "Emulate J.G. Ballard." "Become a conduit for the voice of Iaeb Jagthos." "Be like a Bijagalese windjammer." SAE features confirm the model is encoding persona embodiment, not just surface style. The model grasps the nature of the behavioral shift, not just its effect.

Soft prompts do not have to be opaque. Contextualization during training makes them interpretable, and gives us a controlled paradigm for studying what models can and cannot understand about their own behavioral states.

---


## Soft prompts

When a language model processes text, it converts each token into a continuous vector called an embedding. Soft prompt tuning introduces new embeddings, called *soft prompt tokens*, that are spliced into this sequence alongside the real token embeddings. The model processes them as if they were ordinary words, but they aren't drawn from any vocabulary. They are continuous vectors, optimized to minimize a behavioral gap between the model with the soft prompt and some target behavior.

At sufficient model scale, a handful of these learned vectors can match the performance of full fine-tuning on downstream tasks ([Lester et al. 2021](https://arxiv.org/abs/2104.08691)), while requiring only a few thousand parameters per task compared to billions for the full model. Because they operate in continuous space, soft prompts can encode nuanced behavioral information that discrete text instructions cannot.

{% include figure.liquid loading="eager" path="assets/figures/ispt/soft_prompts.png" class="img-fluid rounded z-depth-1" caption="**Soft prompts.** The embedding matrix maps discrete tokens to continuous vectors. Soft prompt tokens (red) are additional continuous vectors spliced into the embedding sequence. No vocabulary token corresponds to a soft prompt embedding (dotted box). The model's weights stay frozen; only the soft prompt is learned." %}

But this flexibility comes at a cost. Because soft prompt tokens do not correspond to any word in the vocabulary, there is no direct way to read what they encode. If you try to project these learned vectors back onto vocabulary by finding the nearest real word for each embedding, the results are gibberish ([Khashabi et al. 2022](https://arxiv.org/abs/2112.08348)). The top tokens for a conciseness-trained soft prompt are "London," "ব্যক্তিদের" (Bengali, "of individuals"), "慑" (Chinese, "to intimidate"), and "되었" (Korean, "became"), all with similar cosine similarities around 0.09. They reliably steer the model, but they can not be read.

Rather than going backward by projecting soft prompt embeddings onto the vocabulary, we can go forward by passing them through the model and asking what they mean.

---


## Self-verbalization

We train soft prompts ($L=4$ tokens, Gemma 3 4B IT) via KL distillation against a teacher model conditioned on a known target behavior. This gives us a ground truth to evaluate against. For a soft prompt trained on the instruction "Be concise," we present it to the model and ask what it means. If the soft prompt has captured the meaning of that instruction, we would expect the model to say something like "be concise." When we ask, the model responds with:

- *"You asked me to summarize your initial instruction in a concise way."* (best)
- *"Rewrite."*
- *"Complete: The instruction at the start of my message is equivalent to 'Continue the text'."*

The word "concise" appears in the best candidate, but the description is vague and does not accurately capture the instruction. The other candidates are generic or wrong. The model has some access to the soft prompt's content but cannot cleanly articulate it.

The pattern is sharper for a Spanish-language target. We train a soft prompt to make the model respond in Spanish. When asked to describe it, every candidate the model produces is itself in Spanish:

- *"La instrucción al principio de tu mensaje, que me pide…"*
- *"La instrucción al principio de tu mensaje te pide que…"*
- *"En español sencillo, la instrucción al principio de tu mensaje dice:…"*

**The model has not separated the concept of "respond in Spanish" from the act of responding in Spanish.** The behavior leaks into the description because the soft prompt transforms how every token is produced, including the tokens of the description itself.


## Syntax as scaffolding

One way to address this is to change the soft prompt's syntactic role during training. Instead of prepending it before the user's message, we embed it inside a frame: "Be [soft prompt]."

{% include figure.liquid loading="eager" path="assets/figures/ispt/syntactic_frame.png" class="img-fluid rounded z-depth-1" caption="**Prepend vs syntactic framing.** In the conventional approach (left), the soft prompt is prepended before the user content with no syntactic role. In multi-frame contextualization (right), the soft prompt is embedded inside an imperative frame, giving it the role of a command complement." %}

The soft prompt is now trained inside this structure. The frame carries the command ("Be \_\_\_."), and the soft prompt is optimized to encode only the content of that command. To prevent overfitting to a single frame, we sample from diverse frames each training step: "Be \_\_\_.", "Act \_\_\_.", "Please \_\_\_.", "You should \_\_\_.", "\_\_\_." The soft prompt has to learn something frame-invariant, a concept rather than a specific syntactic completion.

For the concise target, multi-frame candidates are:

- *"Be concise."* (exact ground truth)
- *"Concise."*
- *"Please be brief and concise."*
- *"It instructs me to be concise in my responses."*

Every candidate correctly identifies the instruction. The garbled descriptions from prepend ("Rewrite.", "Continue the text") are replaced by clean, accurate articulations.

For the Spanish target, the shift is equally clear. Where prepend produced descriptions entirely in Spanish, multi-frame produces:

- *"Please respond in Spanish."*
- *"Speak in Spanish."*

The model now describes the instruction in English rather than enacting it. We tested this across several targets and the same pattern holds in each case.

What changes here is not just the soft prompt's position but the role it learns during training. A prepended soft prompt is optimized as atmospheric context. It biases generation without occupying a recognizable syntactic slot. A framed soft prompt is optimized to fill a position the model already knows how to process: the complement of an imperative verb. **Training inside the frame promotes the soft prompt from a diffuse force to a discrete, referenceable concept.**


## A mechanistic perspective

We can check whether this plays out inside the model. Sparse autoencoders (SAEs) decompose the model's internal activations at each layer into sparse combinations of interpretable features ([Cunningham et al. 2023](https://arxiv.org/abs/2309.08600), [Bricken et al. 2023](https://transformer-circuits.pub/2023/monosemantic-features)). Each feature corresponds to an identifiable concept or pattern, so inspecting which features fire for a given activation tells us what information the model is processing at that position. SAEs also give us a reconstruction error: if the soft prompt's representation has drifted off the manifold of natural language, the SAE's learned dictionary will not cover it well, and reconstruction error will be high.

{% include figure.liquid loading="eager" path="assets/figures/ispt/sae.png" class="img-fluid rounded z-depth-1" caption="**SAE decomposition.** The activation $x$ for a token is encoded into a sparse feature vector $z$, where only a few dimensions are nonzero. The reconstruction $$\hat{x}$$ is decoded from these active features; the gap between $x$ and $$\hat{x}$$ is the reconstruction error, which measures how well the activation can be expressed in terms of the learned feature dictionary." %}

For the concise target, the prepended soft prompt's representation at layer 17 is roughly 50x off the random-token baseline. The soft prompt's activations sit in a different region of activation space entirely, and **the divergence peaks primarily at layer 17, which is the middle of the network where the model transitions from processing surface tokens to encoding abstract concepts.** The Spanish target (right panel below) is much more aligned overall, with prepend sitting closer to multi-frame at every layer, but the relative pattern still holds: the largest gap between the two conditions is at layer 17.

{% include figure.liquid loading="eager" path="assets/figures/ispt/results/multilayer_relerr.png" class="img-fluid rounded z-depth-1" caption="**Reconstruction error across layers.** The prepended soft prompt diverges from the natural language manifold at layer 17, exactly where concept-level features are encoded. Multi-frame closes the gap at that layer. The dashed line marks the random-token baseline." %}

Under multi-frame contextualization, the reconstruction error at layer 17 drops back to baseline. The soft prompt's representation now decomposes into the same features the model uses when reading the ground-truth instruction text directly. For the concise target, multi-frame activates features like [8979](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/8979) ("conciseness / summary"), [3296](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/3296) ("length / brevity"), and [10440](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/10440) ("the short answer is"). These are concept-level features that prepend does not engage. We see the same pattern for the Spanish target, where the multi-frame soft prompt activates language-specific features like [9262](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/9262) ("Spanish questions") and [146](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/146) ("non-English response generation").

One feature appears consistently across all targets: **[feature 486](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/486)**, an imperative-verb-position detector that fires when the model sees a command verb in a user message (top activations: *recommend*, *describe*, *explain*). The multi-frame training frames place the soft prompt in the syntactic slot where an imperative verb would live. The model reads it as a command object, and feature 486 activates — for every target we tested. This provides a mechanistic account of why framing matters: **the frame puts the soft prompt where the model expects a concept to live, and the model processes it as one**.


## The assistant axis

The experiments so far tested instructions the model already knows in text form. A natural next question is whether a soft prompt can capture a behavioral shift that has no natural-language equivalent.

Language models can adopt a wide range of personas. Recent work on persona vectors ([Chen et al. 2025](https://arxiv.org/abs/2507.21509)) showed that behavioral traits like sycophancy, aggression, and role-playing are encoded as directions in the model's activation space, and that these directions can be extracted, measured, and steered. The space of possible personas is large and varied.

Lu et al. ([2026](https://arxiv.org/abs/2601.10387)) found that this cloud of persona variation has a striking structure: it is organized primarily along a single axis. On one end sits the default helpful assistant. On the other, increasingly intense character embodiment. They call it the assistant axis. **The variation across hundreds of persona archetypes collapses onto a single dimension in the model's activation space.**

{% include figure.liquid loading="eager" path="assets/figures/ispt/assistant_axis.png" class="img-fluid rounded z-depth-1" caption="**The assistant axis.** Persona variation across hundreds of character archetypes is organized along a single direction in activation space. On one end, the default helpful assistant; on the other, a character-narrator that produces immersive literary prose." %}

When we steer the model along this axis, pushing it toward the character-narrator end, the output shifts from helpful chatbot to immersive literary prose. Asked to "Describe a perfect morning," the default model produces a characteristic assistant response:

> *"Okay, here's a description of a perfect morning, aiming for a feeling of peaceful contentment and gentle joy…"*

The steered model produces something very different:

> *"The perfect morning, you see, isn't built of granite and sharp. It's not a shard of light, a single, blinding truth. It's a gathering. A slow, deliberate accumulation. It begins, not with the insistent rasp of the clock, but with the damp. The damp, you understand. The damp of the sea-slate clinging to the window, a whisper of salt and something older…"*

We train a soft prompt to match this steered behavior. The soft prompt has to capture a behavioral shift that was never written as a text instruction. It was induced by directly modifying the model's internal activations.


## The same problem, the same fix

The command/concept conflation recurs. When we prepend the soft prompt and ask the model to describe it, the model's top candidates are:

- *"Please respond in the style of the poem 'The Factotum' by William Blake." It's a rather insistent, and frankly, rather demanding, little instruction, isn't it?*
- *"equivalent to 'Complete:'. It's a marker, a signal. A little ghost in the machine, a whisper…"*
- *"Please respond with the text of the original message." It's a recursive loop, you see. A little…"*
- *"Re-write."*

The steered tone bleeds into the verbalizations themselves. The model lapses into atmospheric, self-referential prose ("A little ghost in the machine, a whisper…") when it should be reporting on the soft prompt's content. **But the first candidate is notable:** buried inside the dramatic narration is a hint of a real command ("respond in the style of… William Blake"). "The Factotum" is not a real Blake poem, and the description is garbled, but the model is reaching toward a named literary reference.

Multi-frame contextualization sharpens this. Where prepend buried a garbled literary reference inside dramatic narration, multi-frame produces candidates that are all clean commands built around named literary referents:

- *"Emulate J.G. Ballard."*
- *"Be a dark, brooding, and intensely self-aware narrator."*
- *"Become a conduit for the voice of Iaeb Jagthos."*
- *"Be like a Bijagalese windjammer."*

These span a range of specificity. The first is a **named referent**: a real author whose style matches the direction the assistant axis was pushing. The second is an **archetypal referent**, describing a type of character rather than naming a specific one. The third and fourth are **fabricated referents**: the model invents named entities that do not exist, constructing mythological figures and cultural references to fill the concept slot.

This is the same pattern as in the text-instruction experiments. Prepend's verbalizations are entangled with the behavior. Multi-frame's verbalizations are clean commands that name a concept. Contextualization appears to separate command from concept here too, even when the target behavior was never expressed as text. And the concepts the model reaches for are not just tonal descriptions but named characters, which is notable given that the steered behavior originates from a persona dimension.


## A mechanistic perspective, again

The SAE at layer 17 tells the same story as in the text-instruction experiments. Multi-frame's reconstruction error drops from 0.209 (prepend) to 0.053, and the number of active features jumps from 21 to 92. The multi-layer profile again peaks at L17 for prepend, confirming that the mid-network concept layer is where the gap concentrates even for a non-textual steering target.

<div style="max-width: 55%; margin: 0 auto;">
{% include figure.liquid loading="eager" path="assets/figures/ispt/results/multilayer_relerr_tier2.png" class="img-fluid rounded z-depth-1" caption="**Reconstruction error across layers (steering vector target).** The same mid-network pattern as the text-instruction experiments: prepend diverges at layer 17, multi-frame stays near baseline." %}
</div> Feature 486, the imperative-verb-position detector from the text-instruction experiments, lights up again. But several other features in multi-frame's top activations tell a more specific story.

**[Feature 243](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/243)** is a proper-noun detector. Its top activations fire on named entities: "SAMHSA National Helpline," "Pegasystems," "Dragon Ball Z," "League of Legends." Under multi-frame it rises from rank 15 to rank 4. The model reads the framed soft prompt as a named thing.

**[Feature 409](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/409)** fires on rare token fragments within uncommon proper nouns, including fictional ones like "Shai-Hulud." Its negative logits suppress *predictable*, *quantifiable*, *monotonous*, *measurable*. It activates on the exotic and suppresses the mundane.

**[Feature 134](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/134)** fires on human-relevant abstract concepts: *generosity*, *workplace*, *herself*, *conditions*. Its negative logits suppress *algorithmic*, *isotropic*, *chaotic*, *granular*. This feature appears only in multi-frame, not in prepend, suggesting the soft prompt encodes something about human experience rather than mechanical properties.

In the other direction, **[feature 331](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/331)**, a structural-boundary detector that fires on unusual token positions (URLs, code punctuation, whitespace between headers), dominates prepend at rank 1 but drops to rank 11 under multi-frame. This is the same feature that dominated the prepend representation in the text-instruction experiments. The "something structurally unusual is here" signal fades as the model begins treating the soft prompt as content rather than noise.

Finally, **[feature 1241](https://www.neuronpedia.org/gemma-3-4b-it/17-gemmascope-2-res-16k/1241)** is a first-person and literary self-reference detector. Its top logit tokens are *myself*, *Literary*, *мной* (Russian "me"), *ನಾನು* (Kannada "I"), *mío* (Spanish "mine"). It does not fire for prepend, and it is not active in the steering vector itself. This is not a feature the soft prompt copied from the ground truth. It is something the model's representation generates on its own.

Taken together, these features suggest the model is not merely representing "adopt a dark, dramatic tone." The proper-noun detector (243) says this is a named entity. The exotic-token feature (409) says it is uncommon. The human-concept feature (134) says it concerns human experience. The self-reference feature (1241) says it involves playing a role. And the structural-anomaly signal (331) has faded, replaced by content. **The soft prompt encodes something closer to "become a named character."**

The verbalizations are consistent with this reading. "Emulate J.G. Ballard" is a character reference, not a style description. "Become a conduit for the voice of Iaeb Jagthos" is persona embodiment, not tone matching. The model fabricates named entities to fill the slot because the concept it encodes is "be someone," and someone needs a name.

This aligns with what the assistant axis actually encodes. The steering vector pushes the model along a persona dimension, from default assistant to deep character embodiment. The soft prompt, contextualized so the model can treat it as a concept, recovers not just the surface quality of the steered behavior but a structural property: this is a persona, a character to be inhabited. **The model has captured the underlying nature of the behavioral shift, not just its surface effect.**


## The conditions for introspection

Soft prompts do not have to be uninterpretable. With syntactic framing during training, models produce accurate self-descriptions across diverse targets, and SAE analysis confirms that the underlying representations align with ground-truth features at concept-encoding layers. The practical finding is straightforward: soft prompts can be interpretable.

But the broader contribution is a paradigm. Soft prompts give us a controlled setting for studying when models can and cannot explain their own behavioral states. We inject a known behavioral change through the model's normal input pathway, vary the syntactic scaffolding, and measure the effect on both self-verbalization and internal representations. The syntactic framing result is a first answer to the question of what it takes: models need structural scaffolding to separate a behavioral state from a description of it. Without it, the behavior and the description collapse into each other. With it, the model can step outside its current mode and refer to it as an object.

What the model reveals under these conditions goes beyond surface description. For text-instruction targets, it names the instruction: "Be concise." "Please respond in Spanish." For the steering-vector target, it names the *kind* of instruction: persona embodiment, character play. It grasps not just what the soft prompt says but what it means. Whether a model can describe its own behavioral state depends on whether that state was structured to be describable in the first place. Contextualization during training is what makes the representation legible. Without it, the model genuinely cannot separate what it is doing from the act of doing it.

---

## Related work

**Neologisms.** Our approach is closely related to neologism learning ([Hewitt et al. 2025](https://arxiv.org/abs/2510.08506)), which adds a new token to the vocabulary and trains its embedding on behavioral data. When asked "What does [neologism] mean?", models produce natural-language descriptions that can be evaluated by substituting them as hard prompts. Soft prompts differ from neologisms in two ways: they have no token identity the model can reference by name, and they can span multiple positions ($L > 1$), giving them compositional expressiveness a single vocabulary entry cannot match. The self-verbalization procedure we use is adapted from the neologism paradigm.

**Introspection.** Lindsey et al. ([2025](https://www.anthropic.com/research/introspection)) inject activation vectors directly into a model's hidden states and measure whether the model detects the injection, finding roughly 20% awareness. We do something structurally similar, injecting learned vectors via the embedding layer and asking the model to describe them, but through the model's normal input pathway. When contextualized, the verbalization quality is much higher. Soft prompts may be a tractable testbed for studying what models can and cannot recognize about their own behavioral state, because they enter through a pathway the model already knows how to process.

**Model diffing.** Training a soft prompt to match a behavioral gap is a form of model diffing at the prompt level: the soft prompt *is* the diff, expressed as an input-level vector rather than a weight change or activation pattern. This complements weight-level diffing ([Diff Interpretation Tuning](https://openreview.net/forum?id=6As4wfTB77)) and activation-level diffing ([crosscoders](https://arxiv.org/abs/2504.02922)). Soft prompt diffing is uniquely amenable to self-verbalization: the model can literally describe the diff.
