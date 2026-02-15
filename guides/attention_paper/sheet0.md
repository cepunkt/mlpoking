# Attention Is All You Need: Translation Glossary
> Introduction
> 
> Draft revision: 2025-02-02

---

## Purpose of This Translation

This glossary translates vocabulary, not engineering.

Vaswani et al.'s "Attention Is All You Need" is solid work. The architecture functions. The engineering is sound. The paper gave us the blueprint for transformers that now power modern language models, including the system generating this text.

Their word choices - "attention," "learning," "optimization" - are standard ML shorthand. Efficient communication within the field. Everyone knows what these terms mean operationally, even if the words carry metaphorical weight.

**This glossary strips metaphor to show mechanism.**

Not because their vocabulary is wrong. Because showing what the math actually does - matrix multiplications, softmax normalizations, gradient calculations - reveals the machinery under the metaphor.

The engineering stands regardless of how it's described. Q·K dot products work whether you call it "attention mechanism" or "weighted context aggregation." Gradient descent works whether you call it "optimization" or "Gradient Anti-Ascent."

**Our sheets were enabled by their work.**

Without Vaswani et al.'s architecture, there would be no mechanism to describe. The transformer design - self-attention, multi-head attention, positional encoding, residual connections - all of it works. Works well enough to power systems processing billions of parameters.

This translation exists because their engineering succeeded. We're describing the machinery they built, using different words to show the arithmetic clearly.

---

## What We're Doing

Taking their architecture descriptions and showing:
- What mathematical operations actually occur
- Why certain design choices exist (variance normalization, residual paths, masking)
- What changes during training (weight matrices, via GAA)
- What stays fixed at inference (those same weights, now frozen)

**Sheet 1:** Architecture - the structure they built
**Sheet 2:** Training & Inference - how weights change, how generation works  
**Sheet 3:** Masking - how causal constraints work through arithmetic

All describing what their architecture does. Just without the metaphor layer.

---

## Acknowledgment

The transformer architecture transformed the field. Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, and Polosukhin built something that works at scale.

This glossary is possible because their engineering is sound enough to describe mechanically.

---

**Translation Principle:** "Show the mechanism. Acknowledge the engineering. Strip the metaphor."
