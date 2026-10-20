# md_files_r_overrated
yeah... why r we burning context to say things a vector could say :/

rep — Portable Reasoning Blueprints for Local Models

Ask a frontier model to solve a long-horizon problem and watch what happens to its context window. It spends thousands of tokens in a <thinking> sandbox restating the problem, tracking formatting constraints, re-deriving facts it already stated, and looping over its own prose. The actual reasoning is a small fraction of the tokens. The rest is human-readable scaffolding that the model needs only because we gave it a human-readable interface.

The .md file is the canonical example. When you want to give a model a skill, the current best practice is to hand it a markdown document: "Here is how to think about this. Step 1. Step 2. Remember to…" This works, but it is expensive and brittle. It consumes context that could hold the problem. It degrades as the trace grows. And it cannot adapt — a static document cannot notice that the model's reasoning is drifting and intervene.

I think there is a better substrate.

The Core Insight: The Failure Is Routing, Not Storage
The naive read of "context rot" is that long contexts cause models to forget. That is not what is happening.

Recent mechanistic work has localized long-context degradation to a precise failure: the information is still in the KV cache, but the attention heads responsible for routing it to the decision point stop doing so. In a causally instrumented Mixture-of-Experts model, the target fact remained ~99.5% decodable at its source position on every failing prompt — ruling out storage loss. What degraded was decodability at the readout position. A small set of attention heads carries that content forward; on long failing prompts, their attention to the fact collapses. A pre-softmax attention boost restricted to exactly those heads repaired the failures.

This reframes the problem. The bottleneck is not "the model needs more context." It is "the model needs help routing what it already has." That is an attention-level intervention problem, and it is tractable.

What rep Is
rep is a format and a runtime for compiled attention-steering policies.

A .rep file is a portable, binary artifact that bundles:

Prefix vectors — continuous embedding tensors that replace a system prompt, learned via gradient descent against a frozen base model.

Attention steering anchors — learned (layer, head, magnitude, trigger-condition) tuples that specify when and where to boost attention during generation.

The .rep file is to reasoning strategy what a LoRA adapter is to knowledge: a small, shareable delta that changes how a base model behaves, without modifying its weights.

The Architecture:
The Engine is a frozen base model. It is not "unaligned" or "raw" — that framing was poetic and wrong. It is simply the model whose weights you are not training. It does the computation.

The Supervisor is a small local model that reads the .rep anchors and monitors the Engine's hidden states, injecting corrective steering vectors when the reasoning trajectory drifts. The Supervisor does not need to solve the problem; it needs to recognize drift. Oversight is an easier task than generation, which is what makes the asymmetric split viable.

The Blueprint is the compiled artifact. It is the only thing that gets shared, versioned, and distributed.

Full code @ md_files_are_overrated (˶ᵔ ᵕ ᵔ˶)
