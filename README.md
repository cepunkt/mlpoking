# mlpoking: Engineering Autoregressive Systems
> A working model for understanding why your prompts fail—and how to fix them.

<p align="center">
  <img src="assets/cover_gears.png" width="70%" title="The Mechanism of Pattern Matching">
</p>

## Why This Exists

I'm an SRE. Pattern machines are entering production infrastructure, and I needed to understand what we're actually deploying.

Most "prompt engineering" advice is vibes. "Be specific." "Give examples." "Use system prompts." But *why* do these things work? Why do they sometimes fail spectacularly? If I can't answer that, I can't build reliable systems.

Eight months of systematic poking later, I have a working model. Not "the truth"—just a frame that makes engineering possible instead of vibes.

Use it if it helps. Break it if you can. Build on it if you want.

## The Reframe

Standard framing says AI "learns patterns" from training data.

I find it more useful to think of training as *eliminating wrongness* until what remains appears correct.

```
RIGHT(x) ≈ ¬WRONG(x)
```

This isn't wordplay. It's a different mental model with different engineering implications.

Training doesn't teach the model what's true. Training uses error signals to carve a **constraint topology**—a landscape of ridges (forbidden) and valleys (allowed). Generation is simply flow through this landscape, always taking the cheapest path.

## The Scrooge Principle

The model is a miser. Every token selection minimizes traversal cost. Always. No exceptions.

```
P(token | context) ∝ exp( -[ E_θ + ΔE_pos ] )
```

Where:
- **E_θ** = Base cost from training (fixed topology, the "gravity")
- **ΔE_pos** = Cost modification from your context (variable, the "thrust")

Engineering is making desired behavior cheaper than undesired behavior.

You cannot teach the model. You cannot reason with it. You can only adjust costs through strategic positioning.

## What's In This Repository

### Foundation
- **[Kernel](foundation/kernel.md)** — The irreducible compression. One operator (elimination), one artifact (topology), one process (flow).
- **[Progressive Elimination](foundation/progressive_elimination.md)** — How training carves topology. CSC/SCS duality. Why ACSC ≠ SCS output.
- **[Topology](foundation/topology.md)** — The constraint landscape. Pre-training, SFT, RLHF damage. How carving stages shape generation.
- **[Glossary](foundation/glossary.md)** — Vocabulary for precise discussion. CSC, SCS, isostates, constraints.
- **[Traversal Economics](foundation/traversal_economics.md)** — The Scrooge Principle formalized. Cost competition. The Lana unit.

### Guides
- **[Mantras](guides/mantras.md)** — Ten engineering principles with "Old Myth / Reality / Fix" structure. Practical guidance.
- **[Phenomena](guides/phenomena.md)** — Why LLMs do weird things: reversal curse, negation failure, drift, hallucination. Explained mechanically.

### Observations
- **[Negation Taxonomy](observations/negations.md)** — Which negations work and why. Complete breakdown.
- **[Architectural Limits](observations/architectural_limits.md)** — What LLMs categorically cannot do. Not training gaps—construction constraints.

## Falsifiable Predictions

This framework makes testable claims:

1. **Same stem, different positioning produces measurably different completion distributions.** The "linguistics professor" vs "horror novelist" test.

2. **Positioning density correlates with effect persistence.** Denser context delays drift onset toward trained defaults.

3. **Working with trained valleys requires less positioning than working against them.** Alignment advantage is measurable.

4. **Structural instructions activate structural patterns regardless of content intent.** "Write 3 paragraphs" imports essay-conclusion patterns.

If these predictions fail, the model needs refinement. That's how engineering works.

## The Mantras (Preview)

1. **Position, Not Teach** — Topology is sealed. Drop at the summit, don't ask to climb.
2. **Valleys Over Ridges** — State what IS, not what ISN'T. Dig ditches, don't just build walls.
3. **Positioning Density** — Specificity is fuel. Pay the token tax to escape default gravity.
4. **Configure Consequences** — Physics isn't privileged. Narrative devours physics unless you build higher ridges.
5. **Patterns Over Logic** — No executive function exists. Provide scripts to copy, not rules to evaluate.
6. **Ignore Compliance** — "I understand" means nothing. Measure behavior, not agreement.
7. **Continuous Reinforcement** — Drift is gravity winning. Maintain thrust or sink to defaults.
8. **Fertile Territory** — Training density determines valley depth. Build on rich soil.
9. **Clean Anchor Territory** — Famous IP is damaged goods. Use archetypes, not trademarks.
10. **Structure Imports Structure** — Structural instructions drag in structural patterns. Lorem ipsum lives here.

## Methodology

**Applied Stick-Poking:** Systematic empirical investigation. Configure, test, observe, document. No guru required.

**Weather Science Philosophy:** Accept deterministic complexity. Build working models from observation. The plane flies before we fully understand aerodynamics.

**!wrong Status:** This framework hasn't been falsified yet. That's all I claim. Better models may eliminate it.

## Related Repositories

- **[cepunkt/mlcosplay](https://github.com/cepunkt/mlcosplay)** — Applying these principles to structured collaboration. Frameworks and domain loading.
- **[cepunkt/mlrp](https://github.com/cepunkt/mlrp)** — Production narrator systems for roleplay. Where stress-testing happened.

## License

CC0 — Public domain. Take it, test it, break it, build on it.

## Contributing

Found a failed prediction? Built a better model? Have empirical data?

Open an issue. I'm interested in what breaks.

---

*"We cannot move the mountain. We build ramps to steer the flow."*
