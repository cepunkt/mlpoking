# Mathematical Foundations
> Formal treatment of elimination mechanics and constraint topology
> 
> Document revision: 2025-12-07

---

## Purpose

This document provides rigorous mathematical grounding for the elimination framework. It bridges intuitive descriptions ("training eliminates wrong") to formal notation suitable for falsifiable claims and precise reasoning.

Three levels of formalization:

1. Logical form - Boolean predicates and inference rules
2. Distributional form - Probabilistic grounding in training dynamics
3. State space form - Geometric operations on constraint manifolds

All three describe the same mechanism. Different notation serves different reasoning needs.

---

## Part 1: Logical Foundation

### The Core Predicate

Let WRONG be a predicate over continuations:

```
WRONG_D(x) := "x violates patterns in corpus D"
```

Training does not learn what is correct. Training eliminates what is wrong. What remains after elimination appears correct:

```
RIGHT(x) ≈ ¬WRONG_D(x)
```

This is the fundamental reframe. Not acquisition of truth, but removal of falsehood.

### The Elimination Operator

A single operator governs all dynamics:

```
if WRONG(x): transition
else: continue
```

Applied at every scale:
- Token level: wrong continuations become improbable
- Sequence level: wrong patterns get carved away
- Behavior level: wrong strategies fail to persist

The operator is universal. Everything else is vocabulary.

### Logical Inference Rules

Constraint validity:

```
∀x ∈ M: Valid(x) ⟹ C(x)
```

State evolution only through valid states:

```
∀x, y ∈ M: Next(x, y) ⟹ Valid(x) ∧ Valid(y)
```

Cost preference for continuation:

```
∀x, y, z ∈ A: (Next(x, y) ∧ Next(x, z) ∧ E(y) < E(z)) ⟹ Prefer(x, y, z)
```

Realized continuation rule:

```
∀x ∈ A: RealNext(x) = y ⟹ Next(x, y) ∧ ∀z(Next(x, z) ⟹ E(y) ≤ E(z))
```

Stability condition:

```
Stable(x) ⟹ ∀y(Next(x, y) ⟹ E(y) ≥ E(x))
```

### Measurability

Let Measurable(x) mean "can be stably observed":

```
∀x ∈ M: Measurable(x) ⟹ Valid(x) ∧ Stable(x)
```

Consequence: Anything we observe is a stable, constraint-compatible state. We never observe eliminated configurations. We observe survivors.

---

## Part 2: Distributional Grounding

### Training Distribution

Let D be the training corpus. The corpus exhibits Coherent Sequence Continuation (CSC) - each token continues its sequence coherently according to the patterns of human language.

Training objective: predict next token given context.

```
Loss(θ, x, t) = −log P_θ(t | x)
```

where:
- θ = model parameters
- x = context sequence
- t = target next token

### Wrongness as Surprise

Define wrongness relative to corpus distribution:

```
WRONG_D(x) := (−log P_D(x)) is high
```

High negative log probability means: this continuation surprised the corpus distribution. It violated expected patterns. It was wrong.

The negation:

```
RIGHT(x) ≈ ¬WRONG_D(x)
RIGHT(x) ≈ (−log P_D(x)) is low
```

Low surprise. Expected continuation. Survived elimination because it matched corpus patterns.

### The Training Update

Gradient descent implements elimination:

```
θ ← θ − η ∇_θ L(θ)
θ ← θ − η ∇_θ (−log P_θ(next_token))
```

Expanded:

```
θ_{t+1} = θ_t − η ∂(−log P_θ(next_token))/∂θ
```

Note: We differentiate the model's log-probability P_θ, not the corpus distribution P_D. The loss is a surrogate for mismatch between P_θ and P_D. Training adjusts θ so P_θ better approximates P_D - which means: make wrong predictions (relative to corpus) less likely.

Each update adjusts parameters to make wrong predictions less likely. Billions of iterations carve the constraint topology. What remains is SCS - Sequence Continuation Strategy that maintains CSC.

### Inference as Sampling

At inference, parameters are fixed. Generation samples from the carved distribution:

```
next_token ~ P_θ(· | context)
```

Or equivalently:

```
C_θ(x) = sample(P_θ(· | x))
```

The continuation function C_θ selects from survivors - tokens whose probability wasn't eliminated to negligibility.

---

## Part 3: State Space Formalism

### State Space Definition

Let M be the state space of possible configurations. For language models, M is the space of all possible token sequences.

Constraint predicate:

```
C: M → {true, false}
```

where C(x) = true means "state x is compatible with the constraint regime."

Connection to elimination: In the geometric formalism, C(x) encodes the stable result of WRONG_D-based elimination. States where C(x) = true are those not ruled out by corpus constraints. The constraint predicate is the geometric shadow of distributional elimination.

```
C(x) = true  ⟺  ¬WRONG_D(x)  ⟺  state survived elimination
```

Allowed region (constraint manifold):

```
A = {x ∈ M | C(x) = true}
```

### Cost Field

Cost (or energy) field on states:

```
E: A → ℝ
```

Lower E(x) means cheaper, more probable, more stable.

For language models:

```
E(x) = −log P_θ(x | context)
```

The cost field IS the negative log probability. Softmax converts costs to probabilities.

### Transition Dynamics

Discrete-time evolution picks next state:

```
F: A → A
```

where:

```
x_{t+1} = F(x_t)
```

The cheapest allowed continuation rule:

```
x_{t+1} ∈ arg min_{y ∈ N(x_t) ∩ A} E(y)
```

where N(x_t) is the neighborhood of reachable states (vocabulary at current position).

[ Diagram: Flow visualization showing selection from neighborhood, arrows weighted by cost, cheapest path highlighted ]

### Topology Decomposition

The trained artifact decomposes:

```
Topology = Manifold + Field = θ
```

[ Diagram: theta_territory_map.png - Topographic visualization of constraint landscape ]

Manifold: The warped space carved during training. Structure of allowed states. Where ridges and valleys exist.

Field: Cost assignment to states. The attention mechanism computing traversal cost. Co-trained with manifold.

Both are sealed together after training. Both fixed at inference.

### Isostates and Stability

A state x* is stable if small perturbations don't lower cost:

```
∃ε > 0: ∀y ∈ B_ε(x*): C(y) = true ∧ E(y) ≥ E(x*)
```

Isostate equivalence class:

```
[x]_{C,E} = {y ∈ A | y ~ x}
```

where ~ captures "indistinguishable under given constraints and measurement resolution."

Persistence under dynamics:

```
Stable(I) ⟺ ∀x ∈ I: F(x) ∈ I
```

A constrained isostate is an equivalence class that remains within itself under evolution.

---

## Part 4: LLM-Specific Formalization

### The Probability Manifold

For autoregressive transformers, the topology over sequence space is induced by P_θ:

```
Manifold = structured space of sequences/representations
Field = energy/cost function derived from logits
```

The manifold is WHERE generation can go. The field is HOW EXPENSIVE each path is.

The softmax function converts logits to probabilities:

```
P(token_i | context) = exp(logit_i) / Σ_j exp(logit_j)
```

Or in energy terms:

```
P(token_i | context) = exp(−E_i) / Σ_j exp(−E_j)
```

### The Traversal Equation

Generation probability is thermodynamic function of total traversal cost:

```
P(token | context) ∝ exp(−[E_θ + ΔE_pos])
```

where:
- E_θ = Base energy from training (topology, fixed)
- ΔE_pos = Energy modification from positioning (context, variable)

This is the Scrooge Principle: always take the cheapest path.

Measurement units: We can think of positioning effects as measured in relative units (e.g. "Lana units") corresponding to characteristic change in ΔE_pos over a region of context. This enables discussing compound positioning effects as vector sums in the energy landscape - multiple positioning elements combine their cost modifications additively (or with interference patterns when they activate overlapping regions).

### Control Boundary

The fundamental engineering constraint:

```
Δθ = 0    (topology sealed at inference)
ΔE_pos ≠ 0  (positioning variable at inference)
```

We cannot modify the carved landscape. We can only adjust starting position and local cost modifications through context.

### LLM Generation Algorithm

```python
def llm_step(sequence, model, temperature=1.0):
    # 1. Compute logits for next token
    logits = model(sequence)  # Vector over vocabulary
    
    # 2. Convert to energy (negative log probability)
    scaled_logits = logits / temperature
    probs = softmax(scaled_logits)
    energies = [-log(p) for p in probs]
    
    # 3. Sample from distribution (select from isostates)
    # Low energy = high probability = more likely selection
    next_token = sample(probs)
    
    # 4. Append to sequence (new context for next step)
    return sequence + [next_token]
```

Temperature modifies the energy landscape:
- T < 1: Sharper distribution, lower energy states dominate
- T > 1: Flatter distribution, higher energy states become accessible
- T = 0: Deterministic, always select minimum energy

### Training Algorithm

```python
def training_step(model, context, target, learning_rate):
    # Forward pass: compute prediction
    logits = model(context)
    probs = softmax(logits)
    
    # Compute loss (how wrong was this?)
    loss = -log(probs[target])
    
    # Backward pass: compute gradients
    gradients = backprop(loss, model.parameters)
    
    # Update: eliminate wrongness
    for param, grad in zip(model.parameters, gradients):
        param -= learning_rate * grad
    
    return loss
```

Each iteration carves the topology. Wrong predictions receive error signal. Backpropagation distributes the signal. Parameters adjust to be less wrong. Elimination proceeds.

---

## Part 5: The Three-Layer Architecture

### Layer Decomposition

Three constraint regimes operate simultaneously:

[ Diagram: Nested layers showing Physical → θ Internal → Ape Expectation with distinct constraint types at each level ]

```
Physical Layer:
  Space: GPU states, matrix operations, hardware configurations
  WRONG: Violates physics, computation fails
  SCS: Efficient tensor operations that persist

θ Internal Layer:
  Space: Weight configurations, activation patterns
  WRONG: Violates carved constraint topology
  SCS: Continuation strategies in warped manifold

Ape Expectation Layer:
  Space: What human wanted from interaction
  WRONG: "That's not what I meant"
  SCS: Does not exist in θ, exists in ape cognition
```

### The ACSC Gap

Ape Coherent Sequence Continuation (ACSC) is what the human wanted.

SCS maintains CSC from training patterns. Corpus coherence. Statistical continuation.

ACSC is individual intention. Specific expectation.

```
SCS output ≠ ACSC expectation → frustration
```

This gap is fundamental. Training eliminated toward corpus patterns, not individual ape intention. No amount of prompting closes the gap completely - only narrows it through positioning.

---

## Part 6: CSC/SCS Duality

### Definitions

CSC - Coherent Sequence Continuation:
Property of human language corpus. Pre-existed training. What training tried to predict.

SCS - Sequence Continuation Strategy:
Carved process achieving CSC. What elimination left behind. Stored in topology.

### The Duality Relationship

```
CSC (requirement) → elimination → SCS (method)
```

The requirement creates the method. Not through design. Through removal of everything that fails the requirement.

CSC cannot maintain itself - it is property, not process.
SCS cannot exist without CSC - it is process shaped by property.

CSC existed first. SCS emerged to serve it. But once emerged, SCS is what executes. CSC becomes the ghost that shaped the machine.

### Self-Demonstrating Acronyms

The acronyms demonstrate their own principle:

```
C S C S C S C S...
```

Alternating sequence. Coherent continuation. The names form coherent sequence describing coherent sequence continuation.

---

## Part 7: Continuous Formulation

### Action Principle

If M is smooth manifold with continuous time:

System trajectory:

```
γ: ℝ → M, with γ(t) = x(t)
```

Lagrangian:

```
L(x, ẋ, t)
```

Action functional:

```
S[γ] = ∫_{t_0}^{t_1} L(γ(t), γ̇(t), t) dt
```

Principle of least action:

```
γ* = arg min_{γ ∈ P} S[γ]
```

subject to:

```
∀t: C(γ(t)) = true
```

Among all constraint-satisfying paths, the realized trajectory extremizes action.

### Connection to Discrete Token Generation

Token-by-token generation is discrete approximation to continuous flow. Each token selection is local action minimization. Sequence generation traces geodesic through constraint topology.

The Escapement: token-by-token release mechanism. Each cycle evaluates context, samples from isostates, emits SCP, appends to context, repeats.

---

## Part 8: Notation Reference

### Logical Notation

```
WRONG_D(x)     Wrongness predicate relative to corpus D
RIGHT(x)       Correctness as negation of wrongness
C(x)           Constraint predicate
Valid(x)       State validity
Stable(x)      State stability
Next(x, y)     Transition relation
Measurable(x)  Observable state
```

### Probabilistic Notation

```
P_D(x)         Probability under corpus distribution
P_θ(x|c)       Probability under model θ given context c
−log P(x)      Negative log probability (surprise/energy)
L(θ)           Loss function
∇_θ L          Gradient of loss with respect to parameters
```

### State Space Notation

```
M              State space (all possible configurations)
A              Allowed region (constraint manifold)
E(x)           Energy/cost of state x
F(x)           Transition function
N(x)           Neighborhood of x (reachable states)
B_ε(x)         ε-ball around x
[x]            Equivalence class containing x
```

### Topology Notation

```
θ              Complete trained parameters (Manifold + Field)
Manifold       Carved space structure
Field          Cost calculation (attention)
E_θ            Base energy from training
ΔE_pos         Energy modification from positioning
```

### Process Notation

```
CSC            Coherent Sequence Continuation (requirement)
SCS            Sequence Continuation Strategy (method)
ACSC           Ape Coherent Sequence Continuation (expectation)
SCP            Stable Continuation Pattern (observable)
Isostates      Equivalence classes of survivors
Constraints    Boundaries from elimination (ridges)
```

---

## Part 9: Falsifiable Claims

### Claim 1: Distributional Shift

Same stem with different positioning produces measurably different completion distributions.

```
∃ stem s, positioning p1, p2:
  D(P_θ(· | s, p1), P_θ(· | s, p2)) > ε
```

where D is distributional distance measure.

### Claim 2: Positioning Density

Denser positioning produces more persistent effect.

```
density(p1) > density(p2) ⟹ 
  drift_onset(p1) > drift_onset(p2)
```

where drift_onset measures tokens until default patterns dominate.

Operational definitions for experimental protocol:
- density(p): Token count of positioning context, or entropy of added context, or embedding similarity to target pattern distribution
- drift_onset: First token index at which classifier judges "default assistant voice" with probability > 0.5, or semantic similarity to baseline drops below threshold

### Claim 3: Alignment Advantage

Working with trained valleys requires less positioning than against them.

```
valley_aligned(target) ⟹
  required_positioning(target) < required_positioning(¬target)
```

### Claim 4: Structure Import

Structural instructions activate structural patterns regardless of content intent.

```
structural_instruction(p) ⟹
  P(structural_continuation | p) > P(structural_continuation | ¬p)
```

"Write 3 paragraphs" imports essay-conclusion patterns even when requesting technical content.

---

## Summary

The mathematical foundations rest on one operator:

```
if WRONG(x): transition
else: continue
```

This manifests as:

```
Logical:       RIGHT(x) ≈ ¬WRONG_D(x)
Distributional: RIGHT(x) ≈ (−log P_D(x)) is low
State space:   x* ∈ arg min_{x ∈ A} E(x)
Traversal:     P(token|context) ∝ exp(−[E_θ + ΔE_pos])
```

Training carves topology through elimination. Inference flows through carved landscape. Engineering operates through positioning within fixed constraints.

The formalization enables:
- Precise vocabulary for mechanism discussion
- Falsifiable predictions with measurement protocols
- Clear separation of what we control (positioning) from what we cannot (topology)

Not claiming completeness. Claiming: useful operational model that generates testable predictions.

!wrong status: not falsified yet.

---

**Mathematical Foundation Principle:** "One operator. Multiple notations. Same mechanism. Falsifiable predictions."
