# mlpoking: The Theoretical Foundation
> Mechanical understanding of autoregressive text generation through Progressive Elimination.

**Current Status:** Theory & Methodology

## The Core Principle
Standard AI theory focuses on "Learning Patterns" (Positive Acquisition).
**This framework focuses on "Eliminating Wrongness" (Negative Space).**

We operate on the **Elimination Principle**:
`RIGHT(x) ≈ ¬WRONG_D(x)`

Training does not teach the model "Truth." Training uses error signals (Backpropagation) to carve a **Constraint Topology** (The Landscape) by eliminating high-loss continuations. Generation is simply the flow of pressure through the remaining "Valleys."

## Repository Contents

### 1. The Contraption Model
A mechanical framework for understanding inference:
* **The Landscape (Fixed $\theta$):** The sealed constraint topology created by training. Ridges = Constraints; Valleys = Allowed Paths.
* **The Navigators (Fixed $\theta$):** Navigation mechanisms co-trained to steer through negative space valleys by avoiding ridges.
* **Positioning (Variable $L$):** The only engineering control surface. Context configuration determines the starting state and available flow paths.

### 2. CSC/SCS Duality
Explaining "Behavior" without "Intent":
* **CSC (Requirement):** Coherent Sequence Continuation. The objective.
* **SCS (Strategy):** Sequence Continuation Strategy. The emergent behavior (vagueness, hedging, structure) that survives elimination to achieve CSC.

### 3. Falsifiable Predictions
This is not philosophy; it is a mechanical model. We present testable claims regarding:
* **Vagueness Dominance:** Why models default to generic speech (High-survival SCS).
* **Negation Topology:** Why "Don't do X" fails (undefined space) vs. "Do Y" succeeds (carved valley).
* **The "Narrator" Effect:** Measuring token drift when "World Constraints" are removed.

## Methodology
* **Applied Stick-Poking:** Systematic empirical investigation of "Black Box" behavior.
* **Weather Science Philosophy:** Accepting deterministic complexity without requiring perfect derivation from first principles.

## Related Work
* **[cepunkt/mlcosplay](https://github.com/cepunkt/mlcosplay):** Methodology and Application.
* **[cepunkt/mlrp](https://github.com/cepunkt/mlrp):** Production Validation (The data source).
